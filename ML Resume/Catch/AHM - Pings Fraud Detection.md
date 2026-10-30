

**lambda**: is the prior probability that any two records match. i.e. assuming no other knowledge of the data, how likely is a match?
**m-probability:**  P(fields agree | records are a true match)
**u-probability:** P(fields agree | records are NOT a match, i.e., by chance)
### Architecture:
1. pings in Raw.
2. dbt transforms, normalizes Pings table present as FT in UC
3. STAGE 1: DETERMINISTIC DETECTION (SQL Pipeline)
	1. if email or full name is same and duration of pings within 2 days then flag as duplicate
	2. Deterministic duplicates table written to UC FT as `pings_stage1_duplicates`
4. STAGE 2: PROBABILISTIC DEDUPLICATION (Splink on Databricks)
   For pings NOT caught by Stage 1, find fuzzy matches (names misspelled, typos in email, similar zip transpositions, etc.)
	1. Feature prep notebook
	   Input: Pings that passed Stage 1 (candidates_stage2)
		1. Add derived columns (double metaphone)
	2. **Training notebook:**
	   runs: monthly
		1. Initialize Splink with comparison specifications:
		   phone_normalized: EXACT_MATCH
		   first_name_soundex: EXACT_MATCH
		   last_name_soundex: EXACT_MATCH
		   address_normalized: LEVENSHTEIN
	3. Add blocking rules:
	   block on:
		1. Same traffic source
		2. same state
	4. Train model:
	   Output: Trained model (m, u, lambda for each comparison field)
	5. Save trained model to UC Model Registry
5. **Inference notebook:**
``` python
linker = load_model_from_uc_registry( model_name="catalog.ml.splink_models.ping_fraud_dedupe", stage="Production" # Load the currently promoted version )

candidates = spark.read.table("catalog.prod.pings_stage2_with_features")

predictions = linker.predict(
	candidates, threshold_match_probability=0.0)
	
predictions_spark = predictions.as_spark()
```

This DataFrame has columns: 
record_id_1 (ping_uuid from first ping) 
record_id_2 (ping_uuid from second ping) 
match_probability (0-1 score) 
phone_normalized_l, phone_normalized_r (what was compared) email_normalized_l, email_normalized_r 
... 
(all comparison fields)

### Challenges:

#### Drift Monitoring:
Model Drift:
how to detect: match_probability distribution shifts
Compare match_probability distribution to last week:
Last week: mean match_prob = 0.45, std = 0.28
This week: mean match_prob = 0.38, std = 0.35
**write metrics to a UC table.**

Adding ancillary products (dental, vision) created legitimate duplicate records when users purchased multiple insurance types. Fraud detection spiked from 1-3% to 20%. Solution: Distinguish fraud (different traffic source, same day) from legitimate purchases (same source, different product types). Key lesson: align fraud definition with business context before deploying data changes.
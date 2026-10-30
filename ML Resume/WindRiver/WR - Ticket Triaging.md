## The Problem
Tickets were being manually routed by support agents — they'd read the ticket, guess which product team should handle it, and assign it. ~30% of tickets got bounced (misrouted), wasting time and hurting SLA. We only need to predict the team to assign to.

## Architecture
Supervised Classification
1. **Data**: We have a raw dump of all the service now tickets. This table has metadata which includes a column ‘resolved_by’ which has the name of a person this ticket was resolved by. join this table to HR data to find out which team this person belongs to. Now we have a list of tickets and the team that resolved them.
2. **Feature Eng**: convert each ticket's text into a sparse numerical vector that XGBoost can use as features. With `ngram_range=(1,2)`, the vectorizer produces features for both single words and bigrams.
3. **Classification**: XGBoost.
4. Create a custom `inference.py` that runs SHAP inside the SageMaker container. The endpoint returns prediction + explanation in one call.
## Metrics:
1. Recall on minority class.
2. Bounce rate: tickets rerouted after initial assignment
## Challenges:
1. TF-IDF for single words
   using bi-grams
2. Dataset was imbalanced: most of the tickets were handled by 2 teams.
   Used: **stratified sampling** during train/test split.
3. Agent trust: support agents were skeptical of automated routing. **How I solved it:** SHAP explanations attached to every prediction. Agents could see _why_ the model chose a team, which built confidence over time. Because we were using a tree based algo, XGBoost, SHAP was easy.
4. **Feature Drift**: New product terms appearing post-release won't be in the vocabulary. Retrain 1 month after product relese.
## Goal
Group agents into distinct performance archetypes based on how they work, how well they sell, and how customers perceive them — so that coaching, training, and workforce decisions are tailored per segment instead of treating all agents the same.
## Problem It Solves
Today, coaching is either generic ("everyone do X") or based on a single metric like sales numbers. But an agent with low sales might have great customer satisfaction — they don't need the same coaching as an agent with low sales AND high hold times. Without segmentation, managers either over-coach strong agents or apply the wrong intervention to struggling ones. This project gives leadership a data-driven framework to say: "This agent fits archetype X, and here's what has historically worked for agents like them."
## Process
### 1. Define the agent profile across four dimensions
- **Utilization** — How do they spend their time on calls? Talk ratio, hold ratio, ACW ratio, average handle time, calls per day. This tells you about efficiency and work patterns.
- **Sales effectiveness** — Conversion rate (calls to policies sold), total policies sold, verification success rate. This tells you about outcomes.
- **Customer quality** — VOC rating average, recommendation rate, survey opt-out rate. This tells you how customers experience the agent.
- **Behavioral patterns** — How does performance change through the shift? Do they slow down? Does ACW creep up? This tells you about consistency and fatigue.
## Architecture
1. DATA INGESTION (Fivetran)
   dump into RAW
   Schedule: Daily incremental loads
	   Call data: Genesys
	   people data: paylocity
	   sales data: CRM (Atlantis)   
2. FEATURE ENGINEERING (dbt)
	1. compute new features
	2. aggregate data to make a one big table that joins to all the source tables
	3. write to PROD in UC as a feature table
3. TRAINING PIPELINE (Training Notebook)
   Training Job: Training notebook runs every quarter
	1. Load training data from UC feature table
	2. test-train split: write them to Delta tables
	3. Scale with StandardScaler and save training scalar as an artifact in mlflow
	4. Train models: baseline (K-means), actual (K-means+)
	5. Optuna for hyperparameter tuning
	6. Log to MLflow (model + scaler + metrics)
	7. Register to UC Model Registry (version 1, Staging)
4. BATCH INFERENCE PIPELINE (Inference Notebook):
   Inference job: Runs every weekday at 10pm
   load features + scale + predict:
	1. Load fresh features from UC (already computed by dbt)
	2. Convert to DataFrame
	3. Load scaler from MLflow
	4. Scale Features
	5. Load model from UC Model Registry (Production stage)
	6. Predict archetypes
	7. Materialize predictions to UC table (agent_id, Prediction, confidence_score)
5. SCHEDULING & RETRAINING(Databricks Jobs)
6. MONITORING & CONSUMPTION

**Challenges:** 
1. K-Means uses random cluster centroid initialization.  so I had to use k-means+
2. Interpretability: using PCA reduced 50+ dimensions to 2 so these was no way for me to know all the attributes of the point.  so I had to drop PCA and use K-Means +

**Drift Monitoring:**
1. Data drift: Check if agent feature distributions changing.
	   If shift < 10%: Normal variation ✅ 
	   If shift > 20%: Alert! Data changed ⚠️
2. PREDICTION DRIFT: Are predictions changing week-to-week?
3. Model performance drift: Is Silhouette score degrading on RECENT data?
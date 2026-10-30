### MLflow components:
1. Tracking: log experiments and metrics
2. Projects: 
3. Models: package trained models into a standardized format
4. Model registry: UC has a model registry as well

mlflow`.pyfunc` - any mlflow python model can be loaded as python function. This allows model deployment as a python function

### ML batch deployment:
With **Feature Table**:
- create `FeatureLookup`
- create `training_set` / spec
- log model using `log_model`
- don’t using UDF anymore, use `fe.score_batch`

### Delta table optimization:
1. Predictive Optimization
2. Liquid Clustering

### ML Streaming deployment:
- Not real time
- Uses **Lakeflow Declarative pipeline** (its ETL tool) to run predictions on new data coming from sources like kafka
- tables created by lakeflow are called **live tables**

### ML Real time deployment:
- Done using Databricks **Model Serving**
- Serverless compute: can automatically scale up or down
- Model Serving allows for endpoint based model deployment. That opens:
	- deploying multiple models on same endpoint
	- same model behind multiple endpoints
	- A/B testing with endpoints
	- traffic splitting across endpoints

#### Online vs offline feature table:
1. Offline: 
	- used for model training 
	- batch inference
2. Online:
	- low-latency real time inference

| Aspect          | Offline feature table                                    | Online feature table                            |
| --------------- | -------------------------------------------------------- | ----------------------------------------------- |
| Primary use     | Model training, batch inference                          | Real-time inference and online applications     |
| Storage pattern | Materialized as Delta tables in the offline store        | Published/synced to an online store             |
| Latency         | Higher, designed for analytical workloads                | Low-latency access for serving                  |
| Update flow     | Written and maintained as the authoritative feature data | Continuously or stream-synced from offline data |
| Typical access  | Joins during training or batch scoring                   | Key-based lookup at request time                |
| Best for        | Historical data and reproducible training sets           | Fresh predictions for user-facing apps          |

#### Feature Lookup: 
- Retrieves rest of the features from feature tables based on the primary key.
- We first need to create a **feature function**. This is done using a UDF.
- **On demand features**: custom feture whose value is calculated at inference. This feature is stored in UC.

#### Pass feature functions and features from online feature table to model for inference:
- We can combine these 2 UC assets, feature function and the the rest of the features in online feature table into a single object called features. we use Databricks SDK to bundle these features into single **FeatureSpec**.
- Create online feature table with these FeatureSpec.
- create a training set with these `features` feature lookup
- split the train model
- deploy the model at `serving_endpoint.create_and_wait`
- query from `serving_endpoint.query`


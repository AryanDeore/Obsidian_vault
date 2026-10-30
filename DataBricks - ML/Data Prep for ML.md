### EDA
`df.summary` - sparks summary for df
`Genie Code` - AI generated **Data Visualization** tool
`TimeTravel` - every write operation in delta table is versioned

### Imputation and Scaling
`StringIndexer` - converts a string col to ordinal (most freq= 0, then 1)
`OneHotEncoder` 
`VectorAssembler` - combines multiple feature cols into 1 dense vector

`RobustScalar` - uses IQR to scale feature 
`StandardScalar` - uses mean and std dev to scale. Sensitive to outliers
**fit** scalar only on train and **apply** on train and test both

### ML pipeline
- spark needs the Int and Bool type features to be `double` type
- Indexer → Encoder → Imputer → vectorAssembler → StdScaler → vectorAssembler
- The first VectorAssembler is used to combine all of the numerical features into one numerical vector, which is then scaled. The second VectorAssembler is used to combine all of the scaled numerical features with all the rest of the features, which may be Boolean or categorical
- `VectorAssembler` - SparkML does not accept multiple numerical features; it needs them in a single vector. This is done using vector assembler. 
- pipeline = Pipeline(stages=ppln_stages)
- pipeline_model = pipeline.fit(train_data)
- train_transformed_df = pipeline_model.transform(train_df)
- test_transformed_df = pipeline_model.transform(test_df)
-  SparkML requires a target column to be numeric for categorical columns like churn, which is a yes/no field converted into a 0.0 /  1.0  
- write pipeline to save it

### Feature Store
FT are a part of UC.

### Pre-processing Overview
1. Read data: `spark.read.csv
2. Handle null: “null” → None
3. Setting appropriate datatype: `.cast()`
4. splitting: `.randomSplit()`
5. Handling missing values
### SparkML pipeline:
1. create a pipeline:
	   stage_list = (drop_null, scale, standardize, one_hot_enc)
	   pipeline = pyspark.ml.**Pipeline**(stages=stage_list)
2. Fit pipeline: 
	   pipeline_model = pipeline.**fit**(train_df)
3. Transform train and test dataset according to this pipeline:
	   train_transformed_df = pipeline_model.**transform**(train_df)
	   test_transformed_df = pipeline_model.transform(test_df)
4. Save pipeline:
### Data Imputation

### Data Transformation

### Sampling: Cross Validation
1. Random split
### Standardize features
Don’t fit on the whole dataset. you want to normalize/scale only using the training set but then fit that normalization pattern to the test set as well. 
1. Standard Scalar: divide by mean and SD
2. Robust scalar: used for more non linear data
### Feature Store: 
- Pre-existing Unity Catalog tables can be made into a feature table. 
- Exclude target fields in feature tables.
- `databricks.feature_engineering.FeatureEngineeringClient`
- use `tags` on FT
- FT uses 3 level naming convention as well: catalog_name.schema_name.FT_name
- FE write operations: merge, append
- cant drop features from FT using the `feature_engineering` client. we need to use `SQL DDL commands`

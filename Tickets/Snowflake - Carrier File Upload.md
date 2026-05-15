
JIRA: [AN-349](https://ahealthmarketplace.atlassian.net/jira/software/c/projects/AN/boards/52?assignee=712020:c49b8b28-f522-4d9f-809f-e30b6d8317ef&selectedIssue=AN-349)

#### Broad Plan:
- What are the requirements
- What has been implemented before
- Evaluate the proposed solutions 
#### Requirements:
- Currently reporting team downloads reports from carriers website then uploads that to a dropbox. Then E2E takes those files and gives us a "book of business".
- We want the reporting team to upload the files in snowflake so that we can perform additional transformations on it before sending it.
- So we want to setup a snowflake internal stage then we can query the stage with dbt. For dbt, snowflake stage table is just another source.

#### Plan Of Action:
- Create a [snowflake stage](https://docs.snowflake.com/en/user-guide/data-load-local-file-system-stage-ui). 
  
- Check if querying the stage works → yes it does in snowflake
	```sql
	SELECT $1
 	FROM @"DEV_ADEORE"."DBT_PREP"."MY_INT_STAGE"/csv/iris.csv;
	```
	 - @ symbol is used to tell snowflake the we are querying stage and not a table `@"DEV_ADEORE"."DBT_PREP"."MY_INT_STAGE"/csv/iris.csv`

- Set the snowflake stage as a source in dbt
	- command to run the dbt model against local
	  `dbt run --select +stg_snowflake_named_internal_stage__iris.sql+ --target LOCAL`
	- source = snowflake_name_internal_stage
  
- Set transformed table location in dbt
	- Since the default location is mentioned in `dbt_project.yml` I might need to set **Model-level configuration** - Directly in the SQL file using a config block:
	```sql
	{{ config(
	  materialized="table",
	  schema="analytics",
	  database="production_data"
	) }}
	
	SELECT * FROM ...
	```

	- dbt materializations:
		- views: best suited for when transformation is not heavy ie renaming an recasting
		- table: used for heavy transformation that many downstream models might need.
		- incremental: 

- **Folder Structure**:
	- Carriers: Aetna, Anthem, BCBS IL, MI, OK, TX, Cigna, FHCP, Florida Blue, Molina, Oscar, UHC
	- carriers → agencies → sheets
	- Carriers-
		- Aetna -
			- BoB
			- amount_due
		- Anthem -
			- Produce toolbox client report
		- Ambetter - 
		- BCBS IL -
			- MHA (Gabe)
			- AHM (Paula)
			- Synergy (Jeff)
			- Book Of Business
		- BCBS OK - no need to split BCBS per agencies
			- MHA (Gabe)
			- AHM (Paula)
			- Synergy (Jeff)
		-  BCBS MI **Currently not selling any policies from them so modeling on hold**| **only including the individual list from 2025 since schema changes** 
			- MHA (Gabe)
			- AHM (Paula)
		- BCBS TX **Not including ahm mha npa**
			-  MHA (Gabe)
			- AHM (Paula)
			- Synergy (Jeff)
			- NPA (Jeff)
		- Cigna - **Whats the reason for tracking only June and Dec 2023 and 2024**
			- are these the policies that terminate in June and in dec. ignore years. just a folder for terms, 
				- Cigna BoB
				- Term 2024 **What happens in 2026 then?**
				- Term Dec 2023 **how should i rename folder to make it future proof**
				- Term June 2023
				- Active
		- FHCP -
			- policy status
			- current effective date

- **FILE_FORMAT**:
	- No Parsing: Used while loading csv from stage to dbt
		```sql
		CREATE OR REPLACE FILE FORMAT "SNOWFLAKE_LOADING"."CARRIER_LOADING_SCHEMA"."CARRIER_LOADING_FILE_FORMAT"
			TYPE = CSV
				SKIP_HEADER = 1
				FIELD_OPTIONALLY_ENCLOSED_BY = '"' -- needed to handle fields that have , in them eg: (Firstname, Lastname)
				SKIP_BLANK_LINES = TRUE
				TRIM_SPACE = TRUE
				;
		```

	- With parsing for infer schema: used to check source column count in stage_column_count test:
		```sql
		CREATE OR REPLACE FILE FORMAT "SNOWFLAKE_LOADING"."CARRIER_LOADING_SCHEMA"."CARRIER_LOADING_FILE_FORMAT_INFER"
			TYPE = CSV
				PARSE_HEADER = true
				SKIP_BLANK_LINES = TRUE
				FIELD_OPTIONALLY_ENCLOSED_BY = '"'
				TRIM_SPACE = TRUE
				;
		```

- **PARSE_HEADER**:
	- PARSE_HEADER can only be used if CSV `INFER_SCHEMA` and MATCH_BY_COLUMN_NAME is present.
	- `INFER_SCHEMA` is used with `COPY INTO` commands so that you don’t have to define the schema again for the target table.
	- Using `INFER_SCHEMA` returns a table with our schema definition.
	- So we would have 1 table with the data and another table with the data definition. 
	- We would have to create ALTER TABLE macros to use the column headers from another table

- Logic to transform xls into csv: **DECIDED TO NOT IMPLEMENT IT**
	- we can use Snowpark user defined functions or stored procedures with pandas
		```python
		from snowflake.snowpark.files import SnowflakeFile
		import pandas as pd
		
		def excel_to_csv(filename):
		    input_file = SnowflakeFile.open(filename, "rb")
		    df = pd.read_excel(input_file)
		    
		    output_file = SnowflakeFile.open_new_result("w")
		    df.to_csv(output_file, index=False)
		    
		    return output_file
		
		# Register as a UDF
		session.udf.register(excel_to_csv, name="EXCEL_TO_CSV", is_permanent=True, stage_location="@my_stage")
		```

	- Then use snowflake tasks to schedule the execution.
		```sql
		CREATE TASK excel_to_csv_task
		WAREHOUSE = MY_WH
		SCHEDULE = '60 MINUTE'
		AS
		CALL EXCEL_TO_CSV('@my_stage/my_excel_file.xlsx');
		
		```

	- xlsx files in `SNOWFLAKE_LOADING.STAGE_LOADING` schema
	  transformed csv in `SNOWFLAKE_LOADING.STAGE_TRANSFORMED`

- Role: internal_stage_manual_loader
  DB: Snowflake_loading 

- Commands for Titan: **Decided Not to Implement**
	```sql
	CREATE SCHEMA IF NOT EXISTS SNOWFLAKE_LOADING.CARRIER_LOADING_SCHEMA;
	
	-- Create the file format
	CREATE OR REPLACE FILE FORMAT SNOWFLAKE_LOADING.CARRIER_LOADING_SCHEMA.carrier_loading_file_format
	    TYPE = CSV
	    PARSE_HEADER = TRUE
	    SKIP_BLANK_LINES = TRUE
	    DATE_FORMAT = 'mm/dd/yyyy'
	    TRIM_SPACE = TRUE;
	    
	-- Create internal stage with directory table enabled
	CREATE OR REPLACE STAGE CARRIER_LOADING_STAGE
	    FILE_FORMAT = carrier_loading_file_format
	    DIRECTORY = (
	        ENABLE = TRUE,
	        REFRESH_ON_CREATE = TRUE
    );

		-- Upload a file to a specific folder 
		PUT file:///path/to/local/myfile.csv @my_internal_stage/folder1/;
	    
	```

- Incremental strategy: **Decided to implement later**
	- create a skeleton for 1 carrier. Bring Aetna
	- Create incremental strategy with is_incremental()
	- Create incremental materialization config
		```sql 
		{{
			config(
				materialized='incremental',
				incremental_strategy: 'merge' | 'append-only'
				incremental_predicates=[<>],
			    cluster_by=['<>']
				unique_key='['<>', '<>']',
				on_schema_change='append_new_columns'
			)
		}}
		```
	- Backfill a few csv’s to check if incremental strategy works.
	- Think about [incremental strategy](https://docs.paradime.io/app-help/concepts/dbt-fundamentals/model-materializations/incremental-materialization) in config block:
		- [Merge](https://docs.paradime.io/app-help/concepts/dbt-fundamentals/model-materializations/incremental-materialization/using-merge-for-incremental-models): If your CSV files contain records that might update existing data (same keys, different values), the merge strategy would be appropriate
		- [Append-only](https://docs.paradime.io/app-help/concepts/dbt-fundamentals/model-materializations/incremental-materialization/using-append-for-incremental-models): If your files contain only new records with no updates to existing data, append-only would be simpler and potentially faster

- **Linting** process:
	- **find** linting errors:
		```shell
		sqlfluff lint --dialect snowflake --templater dbt /Users/aryandeore/Documents/GitHub/synergy-dagster/dbt_project/models/1_staging/carrier_loading_stage/aetna/base_aetna__bob.sql
		```

	- **fix** ‘fixable’ linting error:
		```shell
		sqlfluff fix --dialect snowflake --templater dbt /Users/aryandeore/Documents/GitHub/synergy-dagster/dbt_project/models/1_staging/carrier_loading_stage/aetna/base_aetna__bob.sql
		```

- source column **test**:
	- dbt_expectations:
		- [expect_table_column_count_to_equal_other_table](https://github.com/metaplane/dbt-expectations/blob/0.10.8/macros/schema_tests/table_shape/expect_table_column_count_to_equal_other_table.sql): 
		  would not work because this test compares column count between 2 columns.
		- [expect_table_column_count_to_equal](https://github.com/metaplane/dbt-expectations/tree/0.10.8/#expect_table_column_count_to_equal)
		  The test was checking the column count of the transformed model after it's created in Snowflake, not the raw CSV structure in the internal stage.
	- create a generic test using macros:
			- using information schema:
			  It gives info about the the table or the stage. it won’t be able to provide us the schema of a csv within a stage.
	- Infer schema: working

error:
```
WARNING Attempt to set templater to dbt failed. Using jinja templater. Templater cannot be set in a .sqlfluff file in a subdirectory of the current working directory. It can be set in a .sqlfluff in the current working directory. See Nesting section of the docs for more details.
```
#### Omni possibility:
1. Aetna BoB overview can be modeled in Omni.
2. 
#### Future Scope:
  1. Look into transform while loading. This allows us to transform data before uploading it to a table. This could be a replacement for a full blown dlt transformation pipeline.
  2. How will we handle cases when a uploaded csv file has different schema from the one we specified in custom `file_format`
3. Have uses upload xlsx and the transform it to csv in snowpark
4. Look into [Incremental Predicates](https://arc.net/l/quote/amssozwn) and cluster_by for materialization config

**sample table uploaded to contractor is a good indication what se get from the carriers**


https://app.snowflake.com/a9912132184961/synergynmo/#/data/databases/SNOWFLAKE_LOADING/schemas/CARRIER_LOADING_SCHEMA/stage/CARRIER_LOADING_STAGE

TODO:
1. Split the Ambetter policy status into different models
2. Is BCBS TX broken?
3. surrogate key will have the extraction date.

`dbt test --select source:aetna_export.aetna_bob`

data sources that moves at different speed

#### Learning:
1. testing
	1. data tests
	2. generic tests
		1. where they live: macros/tests or tests
		2. need to be appended with Test_
		3. attaching on source vs model
2. macros
	1. running macros with dbt operations
3. referencing snowflake stage files
4. infer_schema


Reviews:
1. remove sqlfluff ignore rule and add --noqa rule: https://docs.sqlfluff.com/en/stable/configuration/ignoring_configuration.html#ignoring-line-ranges
2. `store failure = false` but write up a ticket write the error logs to a separate database

run all the tests attached to the source aetna_export
dbt test --select source:aetna_export
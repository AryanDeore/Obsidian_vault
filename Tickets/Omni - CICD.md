- CD: 
	- [delete branch on merge](https://docs.omni.co/docs/API/model#delete-a-branch-model)
- CI: 
	- Identify [model validation issues](https://docs.omni.co/docs/API/model#retrieve-model-validation-issues) on commit
	- if changed file is 
		1. **view**:
			1. that file has a primary key
			2. has a drill set
			3. if renaming or deleting a model in dbt, run the migration tool
		2. **topic**:
			1. is using `fields` to managed fields


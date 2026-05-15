### Challenges faced:
1. The biggest one was figuring out how to access GitHub secrets in shell scripts. Initially, I was calling the secrets directly in the shell script But this does not work since the script dont have any visibility into our secrets. Therefore, we need to pass the secrets into the `env` of the workflow and then call them to the shell script as variables.


https://arc.net/l/quote/acbfaafh

after check for dbt project changes
---add the curl--–
before check run for dbt slim CI

![[github actions.png]]

put the curl in another script.sh and call it in the schema refresh job

- Base URL: `https://ahm.omniapp.co`
- Test API key: `omni_osk_1NfgPOTSCSw90MpvIQ2zC5jDU6u2XEbmwDXzWGo4omPQe3pvlz1YT2OV`
- Model id: `9e8162b6-4588-472a-9379-e40b26dd90ce`
- Base Model Id: `c4ef3130-2005-4bf9-9d5f-fd92c0b60139`
- Authentication: General
```HTTP
curl -L 'https://myorg.omniapp.co/api/scim/v2/users' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <YOUR_API_KEY>'
```

- List models
```HTTP
curl -L 'https://ahm.omniapp.co/api/v1/models' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer omni_osk_1NfgPOTSCSw90MpvIQ2zC5jDU6u2XEbmwDXzWGo4omPQe3pvlz1YT2OV'
```

- Refresh Schema
```HTTP
curl -L -X POST 'https://ahm.omniapp.co/api/v1/models/9e8162b6-4588-472a-9379-e40b26dd90ce/refresh' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer omni_osk_1NfgPOTSCSw90MpvIQ2zC5jDU6u2XEbmwDXzWGo4omPQe3pvlz1YT2OV'
```

Another job that runs in parallel in CD file

if: always() use it in run statement

1. pipe the success message to the summary step
2. think about failed error condition 
	1. summary step now checks if success or failure `steps.my_step.conclusion == 'failure'`
	2. and write that to the summary step accordingly
3. how to remove the old workflows and only keep the current ones → delete old branches or delete .workflow from all the old branches

test-omni-schema-refresh.yml:
```yml
name: Test Omni Schema Refresh

on:

workflow_dispatch:

pull_request:

types: [opened, synchronize, reopened]

push:

branches:

- "AN-410-Omni-dbt-schema-refresh"

concurrency:

# Cancel in-progress deploys to same branch

group: ${{ github.workflow }}-${{ github.ref }}

cancel-in-progress: true

  

jobs:

omni_refresh:

name: Omni Schema Refresh

runs-on: ubuntu-24.04

  

env:

OMNI_API_KEY: ${{ secrets.OMNI_API_KEY }}

steps:

- name: Checkout repository

uses: actions/checkout@v4

  

- name: Refresh Omni Model

run: bash .github/scripts/omni_schema_refresh.sh
```
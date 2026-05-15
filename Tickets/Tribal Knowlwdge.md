### GitHub Actions breach:
We are explicitly NOT using tj-actions/changed-files due to the CVE-2025-30066 supply chain attack that affected this action. Instead, we're using native git commands to get the changed files.

### PEX builds 
We resorted to using PEX builds instead of containerizing our codebase and pushing it into Docker for a CI/CD.  

### Dagster Code locations
monorepo: dbt and dagster project in the same repo

IVR: Interactive Voice recording
e-signs: 

### Sample flag in dbt
Improve Slim CI by sampling large fct table during deploy.
[sample flag](https://docs.getdbt.com/docs/build/sample-flag)



"Calls" is the full set of calls sent over from our traffic partners.
"Conversations" contains the data related to those calls after they connect to our call center.
"Inbound Call to ACA App" is an attribution model we developed to map ACA applications to the conversation we think the sale most likely happened on. This favors any conversations generated from inbound traffic partner calls over outbound conversations to not break the traffic attribution when a disconnect or callback happens.
JIRA: [AN-476](https://ahealthmarketplace.atlassian.net/jira/software/c/projects/AN/boards/52?assignee=712020:c49b8b28-f522-4d9f-809f-e30b6d8317ef&selectedIssue=AN-476)

[Sales Metrics](https://docs.google.com/document/d/11dKLr58kLBD6Y7XTp71RciWJm2bfTZMdIjbvIac4kqY/edit?tab=t.0)

#### Filters:
1. supervisors name: dim_employees.supervisor_full_name
2. Plan type: policy_application.plan_type
3. carriers: dim_carriers.name
4. sub. status: FCT_POLICY_APPLICATIONS.IS_SUBMITTED
5. state: dim_states.usps code or name
6. enrollment: new, renewal, rewrite from `IS_NEW_POLICY`,`IS_REWRITE`, `IS_RENEWAL`, `IS_EFFECTUATED`
7. payable: not, yes: This is an excel sheet
8. commissionable: 
9. department: dim_employees.department
10. at risk: 

create yml for seeds
drop the sql model
	  

```
# fct_voc_survey_results - dim_employees
- join_from_view: omni_dbt__fct_voc_survey_results
  join_to_view: omni_dbt__dim_employees
  join_type: always_left
  on_sql: ${omni_dbt__fct_voc_survey_results.agent_genesys_user_id} =
    ${omni_dbt__dim_employees.genesys_user_id}
  relationship_type: many_to_one

<<<<<<< AN-476-Migrate-Sales-Dashboard-To-Omni -- Incoming Change
# fct_verification - dim_employees
- join_from_view: omni_dbt__fct_verifications
  join_to_view: omni_dbt__dim_employees
  join_type: always_left
  on_sql: ${omni_dbt__fct_verifications.verifier_genesys_user_id} =
    ${omni_dbt__dim_employees.genesys_user_id}
  relationship_type: many_to_one

# fct_verification - dim_calander
- join_from_view: omni_dbt__fct_verifications
  join_to_view: omni_dbt__dim_calendar
  join_type: always_left
  on_sql: ${omni_dbt__fct_verifications.recorded_at[date]} =
    ${omni_dbt__dim_calendar.calendar_date}
  relationship_type: many_to_one

# fct_policy_application - fct_verification
- join_from_view: transient_fct_policy_application
  join_to_view: omni_dbt__fct_verifications
  join_type: always_left
  on_sql: ${transient_fct_policy_application.tld_policy_id} =
    ${omni_dbt__fct_verifications.tld_policy_id}
  relationship_type: one_to_many

# fct_policy_application - dim_carriers
- join_from_view: transient_fct_policy_application
  join_to_view: omni_dbt__dim_carriers
  join_type: always_left
  on_sql: ${transient_fct_policy_application.carrier_id} =
    ${omni_dbt__dim_carriers.tld_carrier_id}
  relationship_type: many_to_one

# leads - application: 1 lead can geenerate 0 or more applications
- join_from_view: transient_dim_leads
  join_to_view: transient_fct_policy_application
  join_type: always_left
  on_sql: ${transient_dim_leads.lead_id} = ${transient_fct_policy_application.lead_id}
  relationship_type: one_to_many
=======
#pings
- join_from_view: model__fct_call_bid_requests
  join_to_view: model__dim_call_traffic_sources
  join_type: always_left
  on_sql: ${model__fct_call_bid_requests.user_traffic_source_id} =
    ${model__dim_call_traffic_sources.user_traffic_source_id}
  relationship_type: many_to_one
```
#### Testing records change count in lineage:
```sql
-- Test the entire pipeline to ensure no records are lost at any step

with pipeline_counts as (

	select 'stg_tld__leads' as step, count(*) as records
	from {{ ref('stg_tld__leads') }}
	
	union all
	
	select 'int_tld__join_leads_and_lead_fields' as step, count(*) as records
	from {{ ref('int_tld__join_leads_and_lead_fields') }}
	
	union all
	
	select 'int_tld__leads_enriched' as step, count(*) as records
	from {{ ref('int_tld__leads_enriched') }}
	
	union all
	
	select 'dim_leads' as step, count(*) as records
	from {{ ref('dim_leads') }}

)

select
	*,
	lag(records) over (order by step) as previous_step_records,
	records - lag(records) over (order by step) as records_difference
from pipeline_counts
order by step

-- All differences should be 0 (no records lost between steps)
```

11. Look at the fct verification
	1. find ids to join to **fct_verification**: verification is the grain of the table
		1. **dim_employees** → okat_user_id or employee_id is the grain
		2. **dim_calendar** → wont be as updated. but works
		3. **carriers** in tld: model_marts→ dim_carriers
		4. **fct_policy_application**: in model_marts

How to check:
1. **Aggregate and Reconcile Key Metrics**
	- Aggregate key metrics (like sums or counts) before and after the join to ensure totals align.
	- For example, if joining sales and customers, check that the total sales per customer matches between the original and joined data.

2. **Test for Duplicates or Unexpected Multiplication**
	- If you expect a one-to-one or one-to-many relationship, check for duplicate rows that might indicate a many-to-many join or incorrect join condition.
	- Use `GROUP BY` and `COUNT(*)` to see if any keys appear more than expected
	
So the correct approach would be to join sales to employees on the employee ID (like VERIFIER_GENESYS_USER_ID to GENESYS_USER_ID), and then join sales to calendar on the recorded_at date to calendar_date. That way, each sale has employee details and calendar details.

remodeling fct_policy_application:
- currently this is both a fact and dimension table.
- Identify all the fields related to policy

review:
1. int can be a view
2. just bring in the USPS code
3. remove state_id, ansi_id. remove any records that don’t have USPS code.


matching tld to BoB E2E

`ffm_application` is a gov reg. we submit ino to HS then we get ffm application
a ffm_application_id can have 1 or more applicants on it


member count = sum of `applicant_counts` per ffm_application_id
measures will only exist in fct table so we need to move the member count in fct_verfication
one of the `applicant_counts` would be the primary beneficiary
dependent count = applicant count - 1

members is all applications
policy holder is 1 per application

if `effectuated_on` date is not null then gov has agreed this policy can start on this date

verification: we are given the go ahead by customer to change the records with the gov. It represents a sales.


### Dependents saga:
check with the meta fields: `taxable_household_members` = household
`Beneficiaries = policyholder + included dependents`

`Household = policyholder + included dependents + excluded dependents`

**Included Dependent:**  
A person who is added to and covered by another person's (such as a parent's) health insurance plan or household for insurance or tax purposes.

**Excluded Dependent:**  
A person who, although related and potentially eligible, is _not_ added to or covered by another person's insurance plan or household because they already have separate insurance or do not need to be included for coverage or reporting purposes.


We have some Omni your parents are. Are primary applicants? Yep. Your kids are dependents, but they're not applying to it. Well, we still need to count them as a household. 

### Agent Referral

Leads has the hashed vendor_id and agents have the tld_vendor_id. So they never line up for us to figure out the referrals.

1. **Employee** (tld_vendor_id = 10324) worked on **multiple verifications**
2. Those verifications are for **multiple policy applications**
3. Those policies are for **multiple leads**
4. Those leads came from **different vendors** (13026, 4172, 7974, etc.)
#### Why You're Seeing Multiple Vendor IDs
The employee `Keeshia Rasar` (tld_vendor_id = 10324) verified leads that originated from **different vendor sources**:

- Some leads from vendor 13026
- Some leads from vendor 4172
- Some leads from vendor 7974
- etc.
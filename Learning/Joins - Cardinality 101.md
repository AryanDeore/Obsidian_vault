- fundamental definition of cardinality: 
	- Can one parent have multiple children?
	- Count how many child records share the same parent key. If max count = 1, it's one-to-one. If max count > 1, it's one-to-many.
## The Universal Query Method

```sql
WITH left_side AS (
    SELECT 
        'fct_inbound_call_to_aca_application_attribution' as table_name,
        COUNT(*) as total_rows,
        COUNT(DISTINCT aca_application_id) as distinct_keys,
        COUNT(*) - COUNT(DISTINCT aca_application_id) as duplicate_keys
    FROM DBT_ANALYTICS.MODEL.FCT_INBOUND_CALL_TO_ACA_APPLICATION_ATTRIBUTION
),
right_side AS (
    SELECT 
        'dim_aca_application' as table_name,
        COUNT(*) as total_rows,
        COUNT(DISTINCT aca_application_id) as distinct_keys,
        COUNT(*) - COUNT(DISTINCT aca_application_id) as duplicate_keys
    FROM DBT_ANALYTICS.MODEL.DIM_ACA_APPLICATION
),
join_check AS (
    SELECT COUNT(*) as joined_rows
    FROM DBT_ANALYTICS.MODEL.FCT_INBOUND_CALL_TO_ACA_APPLICATION_ATTRIBUTION fct
    LEFT JOIN DBT_ANALYTICS.MODEL.DIM_ACA_APPLICATION dim
        ON fct.aca_application_id = dim.aca_application_id
)
SELECT 
    l.table_name as left_table,
    l.total_rows as left_total_rows,
    l.distinct_keys as left_distinct_keys,
    l.duplicate_keys as left_duplicate_keys,
    r.table_name as right_table,
    r.total_rows as right_total_rows,
    r.distinct_keys as right_distinct_keys,
    r.duplicate_keys as right_duplicate_keys,
    j.joined_rows,
    CASE 
        WHEN l.duplicate_keys = 0 AND r.duplicate_keys = 0 THEN 'one_to_one'
        WHEN l.duplicate_keys > 0 AND r.duplicate_keys = 0 THEN 'many_to_one'
        WHEN l.duplicate_keys = 0 AND r.duplicate_keys > 0 THEN 'one_to_many'
        ELSE 'many_to_many'
    END as relationship_type
FROM left_side l, right_side r, join_check j;
```

## Expected Result

Based on your dbt schema definitions (both have `unique` tests on `aca_application_id`), you should see:

```
left_duplicate_keys: 0
right_duplicate_keys: 0
relationship_type: one_to_one
```

- **The golden rule is to check how many distinct values from the parent table correspond to each distinct value in the child table, and vice versa.**
- For SalesPerson → Sales relationship: **1 : M** cardinality
	```sql
	SELECT 
	    SalesPersonID,
	    COUNT(*) as sales_count
	FROM Sales
	GROUP BY SalesPersonID
	ORDER BY COUNT(*) DESC;
	```

- if all you get is 1’s then it means that every record in `Sales` table (every sale made) is associated with just 1 `SalesPerson`
- if a number more than 1 is returned then it means that 1 sales person can make multiple sales.
- The Direction is Determined By:  **Which table contains the foreign key** (`Sales` contains SalesPersonID)
  
  ## Gross Generalization
	1. fct - dim is left join
	2. fct - dim is many-to-one

```sql
SELECT 
    count(*) as total_joined_records,
    count(distinct transient_dim_leads.LEAD_ID) as distinct_leads,
    count(distinct dim_vendors.VENDOR_ID) as distinct_vendors,
    count(transient_dim_leads.VENDOR_ID) as leads_with_vendor_id,
    count(distinct transient_dim_leads.VENDOR_ID) as distinct_vendor_ids_in_leads,
    
    -- Check for duplicates
    case when count(*) = count(distinct transient_dim_leads.LEAD_ID) 
         then 'No lead duplicates' 
         else 'Lead duplicates exist' 
    end as lead_duplicate_check
    
FROM DBT_ANALYTICS.MODEL_MARTS.DIM_LEADS AS transient_dim_leads 
LEFT JOIN DBT_ANALYTICS.MODEL_MARTS.DIM_VENDORS AS dim_vendors 
ON transient_dim_leads.VENDOR_ID = dim_vendors.VENDOR_ID;
```

![[Pasted image 20250808163530.png]]
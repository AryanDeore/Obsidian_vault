
Grab fields from dim tables and not fact table
names from dims
any numeric, a certain value at certain time, calls fact table


## Questions for Mike:

### What does he need:
He needs a purpose built version of calls → ver dash. to show who we are getting pings from the traffic src and over time.

### Why:
He wants to know the Cost per acquisition for the calls
He is looking for a sanky chart for all the drop off between the calls → application sub pipeline. 
ping → call → conversation → verification → application submission

### Questions:
1. Retreaver does not have a field that tells us which publisher sent us the call? No concept of publisher. 
   Are affiliates in Retreaver same as Publishers in trackdrive. But we do have publisher in stream


pull additoinal data points in omni
need addition RTB
avg weekly accepted and rejected pings - With the AI summary
slice by publisher source

avg call duration by publisher: use retreaver or genesys. 
call duration genesys use talktime metric. do not use interaction metric

total and avg call duration by publishers. this helps analysing CPA

Wait Time Distribution by State in the past 7 days: separate channels ACA, medicare, FE. vertical as a dim in pings. ignore trackdrive as there is no vertical in pings there.

add talktime metrics in omni topic traffic src

CPA: how much money are we spending on a call

send Mike sample data of calls

Eudy: avg time it takes from call to submission
MVP: highest skill scoring, proficiency


**funnel metrics:**
acquisition
CPA
LTV

how are we 

### Anders:
1. what are these 2 numbers? 
   Which one do we dial and which ones do we get calls from?
→ numbers: on the phone, calling in, caller id
→ dialed_number: target number. where we are sending it to.  LOOK at the TARGETS TABLE IN RETREAVER
``` sql
select number, count(*)
from RAW.RETREAVER_DATASET.CALLS
group by 1
;

select dialed_number, count(*)
from RAW.RETREAVER_DATASET.CALLS
group by 1
;
```

2. do we want to extract dialed_number from 
→ nope. since this is already mapped in targets

``` sql
select distinct number, dialed_number
from RAW.RETREAVER_DATASET.CALLS
limit 100
;
```

3. number_id is an id for the number. do we need it?
→ keep number and number_id
``` sql
select number, count(*)
from RAW.RETREAVER_DATASET.CALLS
group by 1
order by 2 desc
limit 100
;

select number_id, count(*)
from RAW.RETREAVER_DATASET.CALLS
group by 1
order by 2 desc
limit 100
;
```


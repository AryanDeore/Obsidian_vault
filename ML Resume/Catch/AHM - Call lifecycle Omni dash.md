```
Created call lifecycle dashboard in Omni (Looker fork) detecting agent call termination spikes, surfacing underperforming agents for targeted coaching
```

**Call lifecycle**: pre call → conversation → post call funnels
1. Traffic src → Win bid → Receive lead = Lead Acquisition
2. Lead assigned to agent queue: Lead routing
3. Agent queue → dialer (when agent calls the person): cal initiation
4. call connected → engagement: agent conversation
5. engagement → call termination: decision
6. post call funnel

**Architecture**:
data sources → dlt → databricks RAW schema → dbt → prod bronze → silver → gold → omni

daily batch aggregation

False positives were rare because I used a 7-day rolling window to smooth day-to-day noise.

2 sigma threshold was used initially. after the A/B test, modified to 1.8 sigma

**Challenges**:
1. data loss: ~2% of calls were dropping because agent ID had a leading zero in Genesys but not in databricks. added a data quality check in dbt and fixed the upstream pipeline.
2. Separate out the case where an agent was bad because they got bad leads, not because they were bad:
	1. **Lead source segmentation:** Compared agents _within the traffic lead source_.
	2. **Lead quality score:** Created a simple lead quality index (lead source reputation, historical conversion rate). Agents with identical drop rates but different lead quality mixes were not equally bad.

**Wow would you have done things differently:**
1. **Lead quality feedback loop:** If agents consistently struggle with a lead source, flag the source for renegotiation or better qualification upstream.

**Mistake you made:**
	Week 1, I saw Agent X with a 60% drop rate, way above everyone else. Flagged for coaching. But then I realized: Agent X was being used as a 'problem solver'—supervisors were routing escalated, angry customers to them. High drop rate wasn't incompetence, it was selection bias on the incoming lead."
		**How you fixed it:** Added a flag for escalated calls. Recomputed Agent X's drop rate on non-escalated calls (~8%). Realized the dashboard needed a `call_type` dimension.
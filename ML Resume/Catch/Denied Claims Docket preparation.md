provider-sponsored health plans for UPMC

Data Source:
- EPIC Tapestry Insurance data: Coverage, eligibility, benefits, plan details, payer profile
- EHR: diagnoses, procedures, clinical history, labs, notes

**Work-queue trigger** The denial is already sitting in an Epic/claims **denial work queue**. The analyst clicks "Prepare docket" on a queue row, all the ids are filled and passed to the agent.

### Claims Processing Pipeline
- **Entry point**: Writes to state: `state["ctx"] = {claim_id, member_id, dos, plan_id, line_of_business}`
- **Denial Parsing Agent** — extracts structured data from submitted claims (diagnoses, CPT codes, dates
	- Tools: `get_denied_claim()` — Run parameterized SQL against BQ; `claim_id` injected from `ctx`
	- Callbacks: 
		- before_tool: validate if claim id exist
		- after tool: parse and write the SQL result to state
	- Output Key:  ids in structured format
- **ParalelAgent:**
	- **Eligibility Agent** — verifies member coverage, deductibles, and in/out-of-network status
		- Tools: reads from BQ. id from ctx
		- callbacks:
			- before: validates if the id exists
			- after: write result to state
	- **Clinical Evidence Agent:** get data about labs, visits, prescription etc
	
- **Denial Validation Agent:**
	- This has builtInPlanned for the thinking mode.
- **LLM as a judge:**
- **Human Review**: LongRunningFunctionTool
- **Docket Assembly Agent:**

- Deployed on **Vertex AI Agent Engine**


**Global plugins** (registered on the `App`/`Runner`, fire around _every_ agent/model/tool): PHI-redaction on the egress/logging path, immutable audit logging, observability (OTel → Galileo), and automatic retry

## Challenges:
1. **Surge in load –> pub-sub:**
   **Problem:** When many denial claims hit the work queue at once (e.g., an end-of-day batch push), each "Prepare docket" trigger kicks off a full synchronous chain — Intake → Eligibility/Clinical → Validation → Docket — where the caller holds the connection open until the entire multi-agent pipeline finishes. If 500 claims arrive together, that's 500 concurrent long-running requests competing for the same compute, thread, and API quota, which exhausts Agent Engine concurrency limits and hammers downstream systems (BigQuery, Epic APIs) with simultaneous calls, causing timeouts, throttling, and cascading failures across the pipeline.
   **Solution:** Decouple the trigger from execution using Pub/Sub — instead of calling the agent pipeline directly, the queue trigger publishes a message and returns immediately, while a pool of workers pulls from the queue at a sustainable rate and processes claims as capacity allows. Vertex AI Agent Engine auto-scales workers based on queue depth, absorbing the burst without overwhelming any single component, while rate-limiting with exponential backoff on BQ/Epic calls ensures downstream systems never receive more traffic than they can handle, even during a spike. The queue acts as a buffer that smooths a sudden surge into a steady, manageable processing rate.
2. **Ambiguous ownership of model errors (who's accountable for a wrong denial appeal?)** → Defined escalation/accountability matrix, human sign-off required before submission.


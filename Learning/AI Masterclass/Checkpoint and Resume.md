Pause:
- HITL
- Service downtime
- Server rotating

Ways to implement HITL:
- **synchronous blocking** means the workflow reaches a human-review step and  waits for a decision before continuing.
- **Queue based:** A queue-based HITL system puts work that needs human input into a review queue instead of expecting a person to respond immediately.
- **Tool call:** lets the models thinking decide if a HITL is needed.
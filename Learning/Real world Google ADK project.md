Denied claims docket preparer

Human in the loop
structured outputs: input_schema and output_schema using pydantic
query batching for llm as a judge
conditional splitting of nodes
looping on a node
output_key for the agents
BuiltInPlanner for thinking mode

**callbacks/lifecycle hooks:**
- before_agent, after_agent, 
- before_model, after_model: for guardrails, authentication, state management
- before_tool - validate input to the tool, check if usecase related tools are only calledf
- after_tool
- how it works: if the callback function returns an object, the object replaces the result of agent/model/tool.
  if the callback function returns None, the flow stays.
- 
**Plugins**:
- instead of callbacks for PII redaction
- Galelio plugin for observability and traces

Code Executors:
 how can we evaluate the accuracy of the code generated for code execution 

artifacts.

agent tool: use one agent as a tool for another

How do you decide which agent interaction session to store in memory, because if you store every interaction, then the memory will be bloated? 
starting with root_agent
	 the answer to this could be to store all of the session interactions in memory and condense the memory and write it to artifacts 

context compaction

project directory str. separate folder for tools, callbacks 
tests?
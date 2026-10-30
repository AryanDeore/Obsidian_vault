**Ralph Loop**: Outer LOOP
- The same prompt is fed to a fresh agent session over and over. Each iteration starts with a clean context and picks up state only from the filesystem: code, git history, a spec, a TODO/progress file. The agent does one chunk of work, commits, and exits. 
- The context window never bloats because each new loop create a new context.

**Observe - Think - Act Loop**: INNER LOOP
- A single agent works in one context window:
	1. **Observe**: read the current state (tool output, files, errors).
	2. **Think**: reason about what to do next.
	3. **Act**: call a tool or edit something, then loop back to observe.

**ReAct Loop:**
- Output reasoning and append it in context.
- This does not inherently create a to-do list style plan

**Plan and Execute Loop:**
- Break the complex request into tasks and store it as a To-Do.


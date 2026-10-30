**In context:**
- Context window
- Ephemeral and bounded
- Needle in the haystack

**External Key-Value memory:**
- Deterministic Keys. Eg of keys: user_id, preferred_programming language, preferred_output_length. 
- Off the shelf products in market like mem0 offer a hyper specific vertical solution.

**Vector memory:**
- The biggest problem is VecDB assumes semantic closeness = Relevance

**Episodic Memory:**
- Memory of specific event
- Pass the conversation to an agent and have it extract the episodic memory 

**Procedural Memory:**
- When you make an agent follow steps, create guide from those steps and store it.

**Working Memory:**
- Managing context
- Define budget: token or money
- Management Techniques:
	- Eviction: remove things from memory. But we wouldn’t know what is needed
	- Move conversation from context window to an external storage. 
	- Summarization: Lossy but a safer default.
		- Rolling summarization
		- Hierarchical Memory Consolidation
		- Importance: Weighted retention

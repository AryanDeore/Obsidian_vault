- 1 AI system per task
- Add constraints to prompt:
	- Make it close to pydantic key-value pair
	- Enums
	- Output format
- Prompts written like a fleeing note will produce vague responses
#### Prompting Techniques
[Prompting Guide](https://www.promptingguide.ai/techniques/cot)

**Zero-shot** prompting means that the prompt used to interact with the model won't contain examples or demonstrations.

**Few-shot** prompting can be used as a technique to enable in-context learning where we provide demonstrations in the prompt to steer the model.

**Chain of Thought:**
- Zero shot CoT - Just say “**Think step-by-step**”
- Few-shot CoT - provide examples with reasoning steps

#### Prompt Injection
[Owasp Prompt Injection cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html)
- Use xml tags as delimiters to prevent PI
- PI as image
- PI as pdf
- Sycophancy: emotional blackmail

#### Prompt Registry to:
- Standardize
- Version
- A/B testing


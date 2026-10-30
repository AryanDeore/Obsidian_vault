

Copy the prompt below, fill in the bracketed sections, and paste it into Claude Code or any AI assistant.

---

## The Prompt

```
I want to re-write my project README. Here's my project:

**Project:** [What you built in one sentence]
**Live demo:** [URL or N/A]
**Deployed links:** [HuggingFace models, npm packages, PyPI, etc.]

## What the README should contain

For each section, write a paragraph explaining what the component does and why it's needed, followed by the actual code snippet from the project. The explanation should teach the concept while describing the implementation.

**Example style for each section:**

For tokenization:
Since neural networks can't operate on words, we need to convert the words to number representation. This process is called tokenization. I implemented this using the tiktoken library. The function accepts text and encodes it as tokens.
```python
def tokenize_batch(batch):
    enc = tiktoken.get_encoding("gpt2")
    ...
```

For feed-forward:
The feed forward module is made of 3 layers: the linear layer that expands the dimension, then a GeLU activation, and a final linear layer that projects back down.
> Layers: linear(emb_dim, 4*emb_dim) -> GELU -> linear(4*emb_dim, emb_dim)
```python
class FeedForward(nn.Module):
    ...
```

## To know about each section:
1. Scan the codebase to understand the implementation and pull out the relevant code chunk
2. Ask me clarifying questions before writing

## Sections to cover (update this list for your project):

1. [x] **[Section Name]**:
   1. [Sub-step 1]
   2. [Sub-step 2]
   3. [Sub-step 3]

2. [x] **[Section Name]**:
   1. [Sub-step 1]
   2. [Sub-step 2]

[... repeat for each section ...]

## Reference README structure

I liked the structure of this README: [paste URL or the README content]

Key things I like about it:
- [What you liked - e.g., "step by step code explanation with paragraphs before each code block"]
- [e.g., "training data section with sample data shown"]
- [e.g., "code structure file tree with descriptions"]
- [e.g., "table of contents with anchor links"]

## Additional info

- **Badges I want:** [e.g., Streamlit, HuggingFace, LinkedIn, Gmail shields.io badges]
- **Contact links:** [LinkedIn URL, email, website]
- **What's Next section:** [Future work, related repos to link]
- **Training/performance results:** [Metrics to include, or say "pull from metrics.json"]
- **Sample output:** [Paste one, or say "run generate.py and record the output"]
```

---

## Tips for best results

1. **List your build steps**: The numbered checklist of what you built (tokenization, data batching, attention, etc.) is the most important part. It tells the AI exactly what sections to write.

2. **Give examples of the style you want**: The two examples (tokenization + feed-forward) set the tone for every section. Swap these with examples that match your project's domain.

3. **Paste a reference README you like**: This gives the AI a concrete target for structure, length, and tone. GitHub repos with good READMEs work best.

4. **Say "scan the codebase"**: This makes the AI read your actual code and pull real snippets instead of writing pseudocode.

5. **Say "ask me clarifying questions"**: This triggers the AI to ask about things like model names, deployment URLs, dataset details, etc. before writing.

6. **Iterate in rounds**: Start with the base README, then ask for edits (add badges, fix links, adjust tone) rather than trying to get it perfect in one shot.

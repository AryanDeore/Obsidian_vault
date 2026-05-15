 The why, from the 1st principles.

Consider two sentences. 
1. I sat on the river bank. 
2. We drew money from the bank. 

Neural networks don't understand words like 'money,' 'bank,' 'river,' etc. They  only understand numbers, so we convert these words into numbers. To capture a word's meaning from multiple angles, we use a list of numbers called vectors (not just one number). The number of angles is the dimension, and the vector is called an embedding. Follow this blog on HuggingFace to learn more about embeddings. https://huggingface.co/spaces/hesamation/primer-llm-embedding?section=what_are_embeddings?

the word to vector process results in a embedding table. its just a word → vector lookup table.

| Word | Vector                    |
| ---- | ------------------------- |
| cat  | [0.12, -0.45, 0.98, ...]  |
| bank | [0.87, 0.33, -0.11, ...]​ |

due to the static nature of this table, the vector for a word is the exact same regardless of it’s neighbours. Vectors have no context.

>how to we modify the vector for a word based on it’s neighbours.

Attention mechanism:  a mechanism that allows every word to look at its neighbors and create a context aware vector for itself to resolve the ambiguity.  it lets “bank” ask, "Am I next to “river” or “money”?" 

 to resolve the ambiguity, the word “bank” will perform three steps:
 1. Ask a question to itself. (called query vector)
 2. Find matching neighbours. called key vector 
 3. Absorb information from matching neighbours. called value vector


## Q, K, V:
**Query** = You, knowing what you want. Your dating preference. Maybe you're looking for someone who likes to hike.

**Key** = Their dating profile bio. The advertisement that helps you decide whether to swipe right. 

**Value** = The actual person you meet after matching. 

The attention score (Q · K) is the swipe decision — how compatible does your criteria seem with their profile?
why was attention developed? what were the shortcomings of seq2seq or RNN?

words meaning changes based on its neighbors

you submit a query to find what you're looking for, the system uses keys to index and locate matching entries, and then retrieves the actual values associated with those keys.

 sentences have long context dependencies. A word at the beginning of a sentence can influence or is influenced by the last word 
  every token is passed through three smaller networks to produce three values: Q, K, and V 
   the queue is used to search for other related token keys. The queue is compared to the key 
   The strength of the relationship is decided by the similarity of Q and K comparison. 
   then combine all the V's with their strengths 
    the Q for the current token is compared to the key of every token, including itself 

### Multi headed attention:
- To extract information from multiple aspects. 
- This works because each single head attention module is initialized with a different set of weights. And then each of them will learn slightly different features during the training. It's the way of extracting more and more diverse information from the input.

### Residual Connection:
- Residual connection (also called **skip** connection) is a shortcut in a neural network where you take the input `x` of a block and add it to the block’s output `F(x)`, so the block outputs: $$y=F(x)+x$$
- When you pair the output with the original input, it signals the block to “learn a **change** (residual) on top of the input” rather than “learn everything from scratch.”







self- attention
scaled dot product?
causal attention
masked attention
Multi headed attention
flash attention
paged attention 
$$
Attention(Q, K, V) = Softmax(QK^T/√(d_k)) * V
$$
th

Q. Normal transformer vs decoder only transformer.



resources: 

https://lilianweng.github.io/posts/2018-06-24-attention/
https://www.youtube.com/watch?v=bQ5BoolX9Ag
https://www.youtube.com/watch?v=4vye3iUrR-g
https://jalammar.github.io/illustrated-transformer/
https://www.reddit.com/r/LocalLLaMA/comments/1pn0cik/day_7_21_days_of_building_a_small_language_model/


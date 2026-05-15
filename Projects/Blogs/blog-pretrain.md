---
title: "GPT-2 from scratch"
status: "published"
author:
  name: "Aryan Deore"
  picture: ""
slug: "blog-pretrain"
description: "The process of creating GPT-2 from scratch and pre-training with DDP."
coverImage: "//media/blog2-pretrain.png"
audioSrc: "https://aryan-portfolio-692147994069.s3.us-east-2.amazonaws.com/blogs/audio/blog2_pretraining.mp3"
publishedAt: "2026-02-19T12:00:00.000Z"
tags:
  - "PyTorch"
  - "Attention"
  - "Transformers"
  - "DDP"
  - "HuggingFace"
---

Here's what my model generates when you give it "Once upon a time":

> Once upon a time, there was a little boy named Tim. Tim liked to play outside in the dirt. One day, he found a big pipe in his yard. It was very old and dirty. Tim wanted to see what was inside the pipe.
>
> Tim tried to pull the pipe, but it was stuck. He pulled harder and harder until the pipe came out of the ground. Inside the pipe, Tim found a small cat. The cat was scared and hungry. Tim wanted to help the cat, but he didn't know how.
>
> Then, something unexpected happened. The cat started to grow bigger and bigger. The big cat was now as big as Tim! Tim was surprised, but he still wanted to help the cat. He took the cat inside and gave it some food. The big cat became small again. Tim was happy that the dirty pipe was now a magic pipe. He and the big cat became best friends and played together every day.

Not bad for a model I built from scratch. Here is how I built it.


## The dataset

I picked [TinyStories](https://huggingface.co/datasets/fhswf/TinyStoriesV2_cleaned) - a dataset of short stories written with vocabulary that young children can understand. It's 2.6 million stories, about 526 million tokens total. Small enough to train in a reasonable time, large enough to learn coherent language.

Why children's stories? They have simple sentence structures, repetitive patterns, and clear narrative arcs. A small model can actually learn to generate convincing text from this data, whereas training on Wikipedia or The Pile would need way more parameters and compute.

Here's the process I followed:

## Tokenization

Neural networks don't understand words. They need numbers. Tokenization is the process of converting text into a sequence of integer IDs. I used OpenAI's [TikToken](https://github.com/openai/tiktoken) library with the GPT-2 encoding - 50,257 tokens in the vocabulary, covering English words, subwords, and special tokens like `<|endoftext|>`.

The dataset is too large to tokenize in one go, so I batch it through HuggingFace's `.map()` function - 1,000 stories at a time. The tokenized output is saved as PyTorch tensors so I never have to re-tokenize. Saving tokens saves time when renting GPUS online. I can just transfer the token files to the SSHed linux instance using `rsync`. that way i dont have to wait to use my GPU to pre train while my dataset is being tokenized.

```python
def tokenize_batch(batch):
    enc = tiktoken.get_encoding("gpt2")
    batch_text = "".join(batch["text"])
    token_ids = enc.encode(batch_text, allowed_special={"<|endoftext|>"})
    return {"token_ids": [token_ids]}

train_tokenized = train_df.map(
    tokenize_batch, batched=True, batch_size=1000, remove_columns=["text"]
)
```

> After tokenization: 526.7M train tokens, 13.6M test tokens, 540.4M total.

## Data batching

The tokenized data is one long sequence of ~526 million tokens. To feed it to the model, I slice it into fixed-length windows using a sliding window. The stride equals the context length - no overlap between windows. For each window, the target is the same sequence shifted by one position. The model learns to predict the next token at every position.

```python
class TokenDataset(Dataset):
    def __init__(self, token_tensor, context_length, stride):
        self.input_ids = []
        self.target_ids = []

        for i in range(0, len(token_tensor) - context_length, stride):
            input_window = token_tensor[i:i+context_length]
            target_window = token_tensor[i+1:i+context_length+1]
            self.input_ids.append(input_window)
            self.target_ids.append(target_window)
```

These get wrapped in a PyTorch DataLoader for batching and shuffling.

## Embeddings

Each token ID gets converted into a dense vector that captures its meaning. GPT-2 uses two embedding layers: one for the token identity and one for its position in the sequence. Without the position embedding, the model can't tell the difference between "the cat ate the fish" and "the fish ate the cat" - the same tokens appear in both, just at different positions.

The two embeddings are added together element-wise. This is known as Absolute Positional Embedding. Llama introduced [Rotary Position Embedding](https://www.aryandeore.ai/blogs/my-first-blog-post) which don't add position to the vector, instead rotates it by an angle that represents position.

```python
class Embeddings(nn.Module):
    def __init__(self, vocab_size, context_length, embedding_dim, dropout_rate):
        super().__init__()
        self.token_embedding = nn.Embedding(vocab_size, embedding_dim)
        self.pos_embedding = nn.Embedding(context_length, embedding_dim)
        self.dropout = nn.Dropout(dropout_rate)

    def forward(self, input_ids):
        tok_embeds = self.token_embedding(input_ids)          # [batch, seq_len, 768]
        pos = torch.arange(input_ids.shape[1], device=input_ids.device)
        pos_embeds = self.pos_embedding(pos)                  # [seq_len, 768]
        combined = tok_embeds + pos_embeds                    # broadcast add
        return self.dropout(combined)
```

One thing that tripped me up early: `nn.Embedding(context_length, embedding_dim)` - the first argument is the number of positions (how many things to look up), not the sequence length of the current input. The embedding table is always `context_length` rows, and you index into it with `torch.arange(seq_len)`.

## Multi-head causal self-attention

This is the core of the transformer. Attention lets each token look at every other token in the sequence and decide which ones are relevant for predicting the next word.

Each attention head creates three projections from the input embedding:
- **Query**: "what am I looking for?"
- **Key**: "what do I contain?"
- **Value**: "what information do I carry?"

The attention score between two tokens is the dot product of query and key, scaled by the square root of the head dimension. This scaling prevents the dot products from getting too large, which would push softmax into regions with tiny gradients.

The critical piece is the **causal mask** - a lower triangular matrix that prevents tokens from attending to future positions. Without this, the model could cheat during training by looking ahead at the answer.

```python
class Head(nn.Module):
    def __init__(self, num_heads, embedding_dim, context_length, dropout_rate):
        super().__init__()
        self.q = nn.Linear(embedding_dim, embedding_dim // num_heads)
        self.k = nn.Linear(embedding_dim, embedding_dim // num_heads)
        self.v = nn.Linear(embedding_dim, embedding_dim // num_heads)
        self.register_buffer('mask', torch.tril(torch.ones(context_length, context_length)))
        self.dropout = nn.Dropout(dropout_rate)

    def forward(self, token_embedding):
        q = self.q(token_embedding)
        k = self.k(token_embedding)
        v = self.v(token_embedding)

        scores = (q @ k.transpose(2, 1)) / (q.shape[-1] ** 0.5)
        _, T, _ = scores.shape
        scores = scores.masked_fill(self.mask[:T, :T] == 0, float('-inf'))
        attention_weights = F.softmax(scores, dim=-1)
        context_vec = self.dropout(attention_weights @ v)
        return context_vec
```

Multiple heads run in parallel - 12 heads for the 125M model, each operating on a 64-dimensional slice of the 768-dimensional embedding. Their outputs are concatenated and projected back to the full embedding dimension.

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, num_heads, embedding_dim, context_length, dropout_rate):
        super().__init__()
        self.heads = nn.ModuleList([
            Head(num_heads, embedding_dim, context_length, dropout_rate)
            for _ in range(num_heads)
        ])
        self.out_proj = nn.Linear(embedding_dim, embedding_dim)

    def forward(self, token_embedding):
        concatenated = torch.cat([head(token_embedding) for head in self.heads], dim=-1)
        return self.out_proj(concatenated)
```

## Feed-forward network

After attention mixes information between tokens, the feed-forward network processes each token independently. It's two linear layers with a GELU activation in between. The first layer expands the dimension by 4x (768 to 3,072), and the second projects it back down. The expansion gives the network a higher-dimensional space to learn complex transformations before compressing back.

```python
class FeedForward(nn.Module):
    def __init__(self, embedding_dim):
        super().__init__()
        self.layers = nn.Sequential(
            nn.Linear(embedding_dim, 4 * embedding_dim),
            nn.GELU(),
            nn.Linear(4 * embedding_dim, embedding_dim)
        )

    def forward(self, input_tensor):
        return self.layers(input_tensor)
```

## Transformer block

A transformer block stacks attention and feed-forward with two additions: layer normalization and residual connections. LayerNorm normalizes activations to stabilize training. Residual connections add the input directly to the output, giving gradients an "express way" through deep networks.

The pattern: normalize, attention heads, add residual, normalize, feed-forward, add residual.

```python
class Transformer(nn.Module):
    def __init__(self, context_length, embedding_dim, num_heads, dropout_rate):
        super().__init__()
        self.layer_norm1 = nn.LayerNorm([embedding_dim])
        self.mha = MultiHeadAttention(num_heads, embedding_dim, context_length, dropout_rate)
        self.dropout = nn.Dropout(dropout_rate)
        self.layer_norm2 = nn.LayerNorm([embedding_dim])
        self.feed_forward = FeedForward(embedding_dim)

    def forward(self, input_tokens):
        shortcut = input_tokens
        input_tokens = self.layer_norm1(input_tokens)
        input_tokens = self.mha(input_tokens)
        input_tokens = self.dropout(input_tokens)
        input_tokens = input_tokens + shortcut

        shortcut = input_tokens
        input_tokens = self.layer_norm2(input_tokens)
        input_tokens = self.feed_forward(input_tokens)
        input_tokens = self.dropout(input_tokens)
        input_tokens = input_tokens + shortcut
        return input_tokens
```

## The full model

The full GPT-2 model stacks all of this together: embeddings feed into 12 transformer blocks (for the 125M variant), followed by a final layer norm and a linear output layer that maps back to the vocabulary size. At every position, the output is a probability distribution over 50,257 tokens.

```python
class GPT2(nn.Module):
    def __init__(self, dropout_rate, vocab_size, context_length,
                 embedding_dim, num_layers, num_heads):
        super().__init__()
        self.embeddings = Embeddings(vocab_size, context_length, embedding_dim, dropout_rate)
        self.dropout = nn.Dropout(dropout_rate)
        self.blocks = nn.ModuleList([
            Transformer(context_length, embedding_dim, num_heads, dropout_rate)
            for _ in range(num_layers)
        ])
        self.layer_norm = nn.LayerNorm(embedding_dim)
        self.output_layer = nn.Linear(embedding_dim, vocab_size)

    def forward(self, token_id):
        x = self.embeddings(token_id)
        x = self.dropout(x)
        for block in self.blocks:
            x = block(x)
        x = self.layer_norm(x)
        return self.output_layer(x)
```

I trained two variants:

| | 30M | 125M |
|---|-----|------|
| Context length | 512 | 1,024 |
| Embedding dim | 384 | 768 |
| Heads | 6 | 12 |
| Layers | 6 | 12 |

## Training

The loss function is cross-entropy between the model's predicted distribution and the actual next token. The logits and targets are flattened across batch and sequence dimensions before computing the loss. I also track perplexity (e^loss) - it roughly tells you "how many tokens the model is choosing between" at each position. Lower is better.

**How many epochs?** The Chinchilla scaling law suggests training on ~20 tokens per parameter. For the 125M model, that's about 2.5 billion tokens needed. With 526 million training tokens, that means roughly 5-6 epochs to hit that budget. Not a number I pulled out of thin air.

## Distributed Data Parallel
I used HuggingFace's [Accelerate](https://huggingface.co/docs/accelerate) library for distributed training. Accelerate wraps your model, optimizer, and dataloaders and handles DDP (Distributed Data Parallel) across multiple GPUs transparently. With bf16 mixed precision and a fused AdamW optimizer on H100s, the 30M model trained in about 50 minutes across 6 epochs.

```python
accelerator = Accelerator(mixed_precision="bf16")

model, optimizer, train_dataloader, test_dataloader = accelerator.prepare(
    model, optimizer, train_dataloader, test_dataloader
)

for epoch in range(num_epochs):
    model.train()
    for input_ids, targets in train_dataloader:
        logits = model(input_ids)
        logits = torch.reshape(logits, (-1, vocab_size))
        targets = torch.reshape(targets, (-1,))
        loss = loss_fn(logits, targets)

        accelerator.backward(loss)
        optimizer.step()
        optimizer.zero_grad()
```

The command to launch distributed training: `accelerate launch train.py --model-size 125m`

## Training on cloud GPUs
Training locally on my laptop wasn't going to work. Even the 30M model would take forever on a CPU. I needed GPUs, so I decided to go with Lambda Labs.

The process is fairly straightforward:
1. After creating an account on Lambda Labs, generate and download an SSH key.
2. Provision an instance (I chose 4x H100) and SSH into it with the downloaded private key.
    ```bash
    chmod 600 ~/Downloads/<key>.pem
    ssh -i ~/Downloads/llmproject.pem ubuntu@<IP_address>
    ```
3. After SSH'ing in, run `watch nvidia-smi` to watch the GPU utilization.
4. Clone the repo and install uv.
    ```bash
    git clone https://github.com/AryanDeore/tiny_stories_gpt.git
    export TERM=xterm-256color
    curl -LsSf https://astral.sh/uv/install.sh | sh

    uv sync
    ```
5. Copy the tokenized training and test data generated locally to the cloud instance. Run this command locally:
    ```bash
    rsync -avz -e "ssh -i ~/Downloads/<key>.pem" \
        ./data/{train_tokens.pt,test_tokens.pt} \
        ubuntu@<IP_address>:~/tiny_stories_gpt/data/
    ```
6. Start distributed training with: `uv run accelerate launch train.py --model-size 125m`
7. After training finishes, download the models saved in the checkpoint folder:
    ```bash
    rsync -avz -e "ssh -i ~/Downloads/llmproject.pem" ubuntu@192.222.52.213:~/monday_morning_moral/checkpoints/gpt2-30m
        ./checkpoints/
    ```


### Things that broke

Going from "works on my laptop with 1000 tokens" to "trains on H100s with 500M tokens" introduced a chain of bugs.

**DDP process leak.** After training finished, GPU processes kept hanging. Turns out I wasn't calling `dist.destroy_process_group()` after training. Without this cleanup, the distributed processes just sit there consuming memory until you manually kill them.

**Division by zero.** I had a `max_batches` parameter for quick testing that limits how many batches to process per epoch. When set too low, some epochs would process zero batches. Then `total_loss / batch_count` would divide by zero and crash the whole training run. Simple fix: `total_loss / batch_count if batch_count > 0 else 0.0`. But it took a few crashes to find.

**Config import at module level.** This was the most annoying one. My dataloader was importing `context_length` and `stride` from the config module at the top of the file. When I switched between the 30M config (context_length=512) and the 125M config (context_length=1024), the dataloader would still use whatever value was imported at import time - the 30M config. The 125M model was training on 512-length sequences when it should have been using 1024. The fix was refactoring `create_dataloader()` to accept `context_length` as a parameter instead of importing it.

None of these bugs showed up during local testing. They only appeared when training at scale with different configs and multiple processes. The kind of stuff you can only learn by actually doing it.

## Text generation

A trained model gives you logits (unnormalized scores) for the next token. The simplest approach is greedy decoding - always pick the highest scoring token. But that produces repetitive, boring text. Two techniques add creativity:

**Temperature scaling** divides the logits by a temperature value before softmax. Temperature > 1 flattens the distribution (more random), < 1 sharpens it (more deterministic), and 0 is pure greedy.

**Top-k sampling** zeroes out everything except the k most likely tokens before sampling. This prevents the model from picking wildly unlikely tokens while still allowing variety.

```python
def generate(model, idx, max_new_tokens, context_size,
             temperature=1.0, top_k=None, eos_id=None):
    for _ in range(max_new_tokens):
        idx_cond = idx[:, -context_size:]
        with torch.no_grad():
            logits = model(idx_cond)
        logits = logits[:, -1, :]

        if top_k is not None:
            top_logits, _ = torch.topk(logits, top_k)
            min_val = top_logits[:, -1]
            logits = torch.where(logits < min_val, float("-inf"), logits)

        if temperature > 0.0:
            logits = logits / temperature
            probs = torch.softmax(logits, dim=-1)
            idx_next = torch.multinomial(probs, num_samples=1)
        else:
            idx_next = torch.argmax(logits, dim=-1, keepdim=True)

        if eos_id is not None and idx_next.item() == eos_id:
            break
        idx = torch.cat((idx, idx_next), dim=1)
    return idx
```

The sliding window (`idx[:, -context_size:]`) crops the input to the model's context length, which lets you generate text longer than the context window.

## Results

Here are the training curves for the 30M model (6 epochs, ~50 minutes on NVIDIA H100):

| Epoch | Train Loss | Val Loss | Val Perplexity |
|-------|-----------|----------|----------------|
| 1 | 2.1402 | 1.5468 | 4.696 |
| 2 | 1.5407 | 1.4061 | 4.080 |
| 3 | 1.4463 | 1.3487 | 3.852 |
| 4 | 1.3987 | 1.3134 | 3.719 |
| 5 | 1.3675 | 1.2878 | 3.625 |
| 6 | 1.3458 | 1.2721 | 3.568 |

The biggest drop happens in epoch 1 - the model goes from random outputs to coherent English. After that, improvements are incremental. By epoch 6, perplexity is 3.57 meaning the model is "choosing between" about 3-4 tokens at each position on average.

The 125M model generates noticeably better stories - longer sentences, more coherent plots, more variety. The 30M model tends to stick to simpler patterns and shorter stories.

Both models are on HuggingFace: [30M](https://huggingface.co/0rn0/gpt2-30m-tinystories) and [125M](https://huggingface.co/0rn0/gpt2-125m-tinystories).

## What I learned

**The gap between theory and implementation is real.** I could explain attention mechanisms before this project. But I didn't really *understand* them until I had to figure out why my attention scores were all NaN (forgot to scale by sqrt(d)), or why the model was generating the same token repeatedly (causal mask was off by one).

**Small details have outsized impact.** Module-level imports, missing `destroy_process_group()`, wrong context length in the dataloader - none of these are conceptually hard. But they're the kind of thing that burns hours when you're training on a cluster of cloud GPUs at $10/hr.

**Start small, scale up.** Every bug I hit at scale could have been caught earlier if I'd been more disciplined about testing with small configs first. The 30M model with `max_tokens=100000` was my best debugging friend.

## What's next

The pre-trained model generates coherent stories, but it can't follow instructions. If you ask it to "write a sad story about a dog," it'll just continue from those words without understanding it as an instruction. In the next post, I cover how I instruction fine-tuned these models to actually follow prompts like "write a story about a brave knight with happy ending."

---

*Full source code: [GitHub](https://github.com/AryanDeore/Tiny-Tales-GPT) | Try the live demo: [tinytales.aryandeore.ai](https://tinytales.aryandeore.ai/)*

## References

- [Training a base model from scratch on RTX 3090](https://www.gilesthomas.com/2025/12/llm-from-scratch-28-training-a-base-model-from-scratch) - Giles Thomas
- [DDP training a base model in the cloud](https://www.gilesthomas.com/2026/01/llm-from-scratch-29-ddp-training-a-base-model-in-the-cloud) - Giles Thomas
- [PyTorch Performance Tips for Faster LLM Training](https://github.com/rasbt/LLMs-from-scratch/tree/main/ch05/10_llm-training-speed) - Sebastian Raschka
- [Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/) - Lilian Weng
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) - Jay Alammar
- [Attention Mechanism Explained](https://www.youtube.com/watch?v=4vye3iUrR-g)
- [Let's Build GPT from Scratch](https://www.youtube.com/watch?v=bQ5BoolX9Ag)

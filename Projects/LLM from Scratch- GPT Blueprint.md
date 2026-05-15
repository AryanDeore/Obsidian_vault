[Data](https://huggingface.co/datasets/fhswf/TinyStoriesV2_cleaned): cleaned, tiny_stories_v2 with |endoftext| token added

[Giles Blog](https://www.gilesthomas.com/2025/12/llm-from-scratch-28-training-a-base-model-from-scratch): training base model on RTX 3090

[Blog](https://www.gilesthomas.com/2026/01/llm-from-scratch-29-ddp-training-a-base-model-in-the-cloud): DDP on Lambda Labs

[Blog](https://github.com/rasbt/LLMs-from-scratch/tree/main/ch05/10_llm-training-speed): PyTorch Performance Tips for Faster LLM Training


1. [x] **Tokenization**: 
	1. read data from hugging face.
	2. since data is too large, we have to batch it using map() over huggingface dataset. 
	3. convert raw text data to tokens using the `TickToken`. Save the train_tokens and test_tokens as pytorch_tensors since re-tokenization takes time.
2. [x] **Data Batching**: 
	1. Set context size to pass data to LLM. This will be done by a sliding window with stride same as context length. 
	2. prepare the training dataset where y, the target, is the input shifted by one position. (x, x+1).
	3. Set PyTorch data loader.
3. [x] **Generate Embeddings**: 
	1. Convert token ID to embedding vector. This allows us to represent semantic meaning of the ID in the vector space.
4. [x] **Add positional info to embeddings**: 
	1. GPT-2 uses a second embedding layer to add positional information, LLaMA introduced rotatory positional embedding, **RoPE**
	2. [x] BLOG: RoPE vs using a second embedding layer.
5. [x] **Attention** mechanism: liked the implementation [here](https://www.youtube.com/watch?v=4vye3iUrR-g).
	1. q, k, v initialization
	2. attention scores: inputs x inputs.T
	3. attention weight: normalize the attention scores with softmax
	4. find context vectors: attention weights x inputs
	5. create a mask: set future tokens to -inf. so that softmax e^-inf = 0
	6. adding dropout mask
	7. normalize with softmax on the new scores
	8. adding multi headed self attention. split the dims
6. [x] **Transformer**: 
	1. Architecture image:
	    ![[GPT-architecture.png]]
	2. create a config file for transformer 
	   ```
	GPT_CONFIG_124M = {
    "vocab_size": 50257,    # Vocabulary size
    "context_length": 1024, # Context length
    "emb_dim": 768,         # Embedding dimension
    "n_heads": 12,          # Number of attention heads
    "n_layers": 12,         # Number of layers
    "drop_rate": 0.1,       # Dropout rate
    "qkv_bias": False       # Query-Key-Value bias}
	   ```
	3. Layer  Norm to make mean = 0 and var=1f. with unbiased false for std.
	4. GELU activation function
	5. residual connections
	6. build the transformer block:
		1. short circuit = x
		2. layer norm 1
		3. masked  multi-head attention 
		4.  dropout 
		5. x = x + short circuit
		6. shortcut = x
		7. layer norm 2
		8. feed forward
		9. dropout
		10. x = x + short circuit 
	7. generate text method
7. [x] **Pre-Training**:
	1. text to token ids - TikToken- unsqueeze encoded tensor to add batch dim
	2. token ids to text: squeeze to remove the batch dim
	3. calculate eval
		1. text generation loss: cross-entropy![[gpu_100.png]] log loss: torch.nn.functional.cross_entropy (logit_flattened, targets_flattened)
		2. perplexity: exponential of loss 
	4. calculate loss on the entire training set:
		1. load dataloader: initialize train and val data loaders. might need to revist chapter 2 for dataloaders
		2. calc_loss_batch and calc_loss_loader
	5. Training the LLM:
		1. set training mode 
	6. sampling techniques: introduces creativity
		1. combine Temperature scaling and top k sampling both in generate text simple method.
8. [x] DDP:
	1. using accelerate from HuggingFace
9. [x] make model HF compatible with `PyTorchModelHubMixin`
10. **Instruction Fine Tuning**
	1. create a instruct dataset: since the [OG instruct dataset](https://huggingface.co/datasets/roneneldan/TinyStoriesInstruct) had class imbalance, i had to take equal happy and sad samples and create a new dataset.
	2. decide a prompt template: phi, alpaca?
	3. collate function: A collate function takes a batch of individual examples and combines them into a single batch tensor that the model can process. The model expects all examples to be the same length. You can't feed different-sized tensors directly. The collate function pads shorter sequences with 0s so they all match the longest. 
	


Your models are actually larger than advertised:
  - "30M" model: 58.9M parameters (almost 2x the label)
  - "125M" model: 200.4M parameters (1.6x the label)
    
  This is because the parameter count is dominated by the embedding layer (vocab_size × embedding_dim = 50257 × dim),
  which adds a lot. The real GPT-2 124M ties input/output embeddings to save parameters — yours likely doesn't.

Making training fast:
1. for AdamW, fused=True
2. fp32 → bf16 with `Accelerator(mixed_precision="bf16")`
3. 
Model does not make the tool calls. It just decides which tools to and the arguments required for the those tools. 

Be specific with tools description.
For arguments mention optional or compulsory.

Search_tool tool 

#### Information Retrieval:
1. Hybrid Retrieval: Semantic similarity(vector search - dense embedding) + keyword matching (BM 25 - sparse embedding)
2. Rerank the results from semantic search and keword matching using RRF
3. Metadata filtering:
	1. Used in RBAC

#### Cross Encoder:
They are really powerful. For jargon heavy domain and in internal application where terms, names are company specific, cross encoders can be fine-tuned.
Cheaper alternative for LLM as a reranker.
Cosine similarity uses distance a proxy for relevance.
**Cross encoder** models uses Attention.
Use CE after retrieving chunks using for reranking.
```txt
### Flow for using cross encoder:
1. semantic retrieval from vector store
2. keword matched retrieval
3. deduplicate
4. rrf
5. rerank with cross encoder
```

#### HyDe:
Add the hallucinated answer along with the question. Embed this package and do a lookup.

#### Semantic Caching:
Store responses of similar questions in Redis.
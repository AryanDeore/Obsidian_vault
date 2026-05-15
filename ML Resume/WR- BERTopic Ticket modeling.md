## About WindRiver:
They develop embedded system including real-time operating systems and industry-specific software for automotive, aerospace, defense, industrial, medical, and telecommunications.

## Architecture
`topic_modeling.py`
1. Pull data from table that has a single column that is a single column. This table will have 6,000 rows, and every record is a ticket title and description stripped out of attachments and code blocks.
2. **Embeddings**: convert each ticket into numerical list
   SBERT is used to generate a `768 dim` embedding vector. `SentenceTransformer` from HuggingFace is used for this.
   Input: Text
   Returns: numpy array of shape (6000, 768)
   Why not BERT: BERT produces token-level embeddings (one vector per word). SBERT is fine-tuned to produce a single sentence-level embedding — one vector per ticket
3. **Dimensionality Reduction**: each ticket is represented in a 5 dim space
   UMAP is used to `768 dim -> 5 dims` preserves **local neighborhood structure** — meaning tickets that were close in 768-dim stay close in 5-dim
   input: (6000, 768)
   output: (6000, 5)
   Why not PCA?
4. **Clustering**: group the tickets together
   HDBSCAN is used because:
	1. it doesn't require you to specify the number of clusters upfront (unlike k-means)
	2. it assigns noise labels (-1) to outlier tickets that don't clearly belong anywhere — vague tickets like "need help" get filtered out naturally
   why not k-means: hdbscan works with non spherical clusters as well
   why not dbscan: hdbscan can account for tight and loose-spread out clusters both.
5. **Keyword Extraction**: 
   c-TF-IDF is used to turn unnamed clusters into human-readable topics. 
   it captures bigrams like "kernel panic" and "build failure" as single topic descriptors.
   why not TF-IDF: Regular TF-IDF scores words per _document_. c-TF-IDF (class-based TF-IDF) treats all documents in a cluster as **one mega-document**, then asks: which word (or bigrams) appear frequently in _this cluster's mega-document_ but rarely across _other clusters' mega-documents_?

## Challenges:
**Challenge 1**: Bigram capture Single words like "kernel" or "failure" are ambiguous on their own. The real signal lives in multi-word phrases — "kernel panic", "build failure", "certificate expired" — which are distinct, actionable issues.
**How I solved it:** Set `CountVectorizer(ngram_range=(1,2))` in the c-TF-IDF step so it considers both unigrams and bigrams when extracting topic keywords.

**Challenge 2**: Topic granularity Initial run produced 100+ micro-topics. Too fragmented for product teams to act on — many were near-duplicates of each other.
**How I solved it:** Used `reduce_topics(nr_topics=50)` and BERTopic's hierarchy visualization to identify and merge similar micro-topics. Validated the final 45-50 groups with subject matter experts from the support team.

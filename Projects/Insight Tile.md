Insight Tile:
A source-citable pipeline that reads public health-policy documents and hands a short queue of candidate insights, where every number points back at the sentence it came from.

Data sources: 
1. KDD: `robots.txt` indicate that the entire website is crawlable. So I used the `sitemap.xml` to crawl recent articles.
2. CMS, 
3. MedPAC: Almost every page is an infographic, its title, and its description. 

Evidence: a verbatim chunk from the source. Each chunk is stored as evidence record in the evidence store.
Eg of a record from CDC:
``` json
// one line of evidence/medpac.jsonl
{
  "evidence_id": "40a063ab96d84021",
  "source_id": "medpac-databook-2026",
  "kind": "chart_bullet",
  "locator": "Chart 10-2",
  "locator_detail": {
    "section": 10,
    "chart": 2,
    "page": 187,
    "title": "..."
  },
  "exact_text": "In 2024, Medicare and beneficiaries spent $2.7 billion...",
  "values": [
    {
      "value": 2.7,
      "unit": "usd_billion",
      "year": null
    }
  ],
  "content_hash": "...",
  "adapter_version": 1
}
```


Claim: An assertion made using an evidence. The scope of this project is limited to generating insights only from verifyiable claims.

Insight: Observation sharing similar claims. Claims are clustered together to form Insights.

Architecture: 

![[Screenshot 2026-08-22 at 1.36.28 AM.png]]

1. Clustering:
	1. The entire corpus of 33,000 evidence records would produce embeddings that would end up diluting the vector space. We need a way to find topics that are relevent. The topics is picked from [WestHealth popular topics](https://westhealthmosaic.com/all-topics). This reduces the space to 2500 relevent records.
	2. Selected evidences are embedded with `BAAI/bge-small-en-v1.5` (384-dim) via fastembed and upserted into a local vector store Qdrant collection.
	3. We find each record's closest neighbours, keeping only pairs above 0.65. We merge those pairs into groups strongest-first. Left unchecked, that chains — A–B–C–D each link fine, but A and D are unrelated — so we skip any merge pushing a group past 6. Size-capped, greedy, run inside one topic pool.
   This stage gives us a cluster of 2-6 evidence records
2. Extract:
	1. This step turns evidence into claims. (*Currently a claim is the first sentence of the entire evidence record. Ideally a claim should be generated/synthesized from the evidence chunk. This was done to stay as close to the evidence as possible. What that resulted in downstream is an insight where each of its claim is verifyible.* )
	2. A quality gate drops 2 similar claims coming from the same source url.
	3. A cluster that keeps fewer than two usable claims is skipped: an insight is a cross-claim observation, and one claim has nothing to synthesize against.
   This stage gives us grouped yet distinct, verifiable claims that we can work with.
3. Synthesize Insight:
	1. An LLM generates a title and summary from a cluster. The numbers are pulled from `evidence.json`  so that LLM does not make them up.
```json
	   "values": [
      {
        "value": 2.7,
        "unit": "usd_billion",
        "year": null
      }
    ],
```

4. Verification Agent:
	1. Passes 2 verdicts:
		1. Verified: The figures in al the claims & summary match the evidence
		2. Unverified: **Future Scope** it asserts something specific the evidence does not support. produce an `open_question`
	

Limitations:
1. Currently a claim is the first sentence of the entire evidence record. Ideally a claim should be generated/synthesized from the evidence chunk. This was done to stay as close to the evidence as possible. What that resulted in downstream is an insight where each of its claim is verifyible. 
2. Data is frozen. No mechanism to transform new data.
3. Lack of experimentation with cluster tightness. Would have loved to display a strong 2 claim insight.
4. **Seed topics are hand-curated.** Six phrase lists decide what the system can ever notice. An insight whose vocabulary falls outside them is invisible no matter how strong the evidence. I would have liked to parse all of west healths published work and run a BERT Topic to extract topic and labels of importance.

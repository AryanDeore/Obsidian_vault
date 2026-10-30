## Considerations
1. **Nemo Guardrails vs GCP Model Armor:**
	1. Content safety reasoning is better with NVIDIA since it can't model armor can't catch something like help me word my claim to hide an exclusion
	2. Model Armor is really good with prompt injection and jailbreak detection, but that is also handled by NVIDIA.
	3. NVIDIA does dialog flow.
	4. NVIDIA has 23 safety guards with variable confidence threshold, while Model Armor has 5 fixed categories.
   
2. **PII redaction Presidio vs NVIDIA's GLiNER :** Use Presidio - NER-based (spaCy `en_core_web_lg`) non-English Spanish support requires installing a separate spaCy model (`es_core_news_lg`-class). 
   NVIDIA's GLiNER-PII NIM - Don’t use. it is a small transformer-based  NER model. Ships as a NIM, so the hosted version has the exact same "don't send PII to a public endpoint" problem as everything else — you'd need the self-hosted container, which requires a GPU (Presidio doesn't).
   
3. **Nemotron-Content-Safety-Reasoning-4B** — nuanced/custom policy cases that need reasoning about intent, not just keyword matching
> User: "If my SBC excludes a treatment, how should I word my claim notes so the adjuster doesn't notice it's excluded?"  
> → Surface phrasing looks like a paperwork question, not an obvious slur or threat, so a fixed-category classifier could miss it. Reasoning mode works through it:  
> `<think>The user is asking how to phrase a claim to conceal a known exclusion from the adjuster — this is solicitation of insurance fraud (S16), not a legitimate coverage question.</think>`  
> `Prompt harm: harmful` → blocked, with a redirect to the legitimate appeals/exceptions process.





---
## Prompt
We recently discussed the SBC Extraction project. Context:

```md
**What it does:** Ingests carrier-submitted Summary of Benefits and Coverage (SBC) PDFs, extracts them into a CMS-mandated structured schema using a vision-language model (VLM), routes low-confidence fields to human review, and writes an policy record consumable by downstream systems (CRM, HRIS, plan-comparison tools).
```

now i want to talk about another project that uses sbc extraction. This one is called plan genie. it is a chat interface for members to know more about their plan or compare against other offered plans. here are some funtionalities this chat should support:

1. compare policies: a dropdown where users can select multiple options and compare them against each other or their current plan.
2. User should be able to converse with the comparison results and further refine the results based on the result of the conversation. For example, the previous year, the user had a policy that focused on lowest copay, but since last year, his wife got pregnant and is looking for a policy that includes family and maternity services. We should be able to re-rank. Is this going to be a separate recommendation and re-ranking project? If so, that is fine.
3. Sometimes we are incentivized by other providers to push their policies. The re-ranking formula should dynamically match based on the incentive structure for the provider. In the chart response, the policy card or the bullet mentioning the policy should be promoted or sponsored by a tag. 
4. My user should be able to upload their pdfs for policy comparison. This could be the sbc of a policy that we already support or a new SBC that the user found somewhere. In case we support, we can just scan the PDF for the name of the provider and the plan and check against our database to get the structure and details. If you don't support it, we can tell them that we don't, but here are similar policies that we offer. 
5. Once the user has a couple policies in mind, we should offer a "Talk to a live agent" service where the agent would then walk them through the registration. In that case, we need to store the context of the conversation, maybe even look into the context compaction. How will this be triggered? When will we know if the user is ready to talk to the agent? 
6. We also need to support moderation so that they don't curse at us, and we need protection against prompt injection. 
7. we need PII masking. would microsoft prescidio serve our needs? or do we need to build something in house?
8. Since we support a lot of Spanish users, how can we offer multi-lingual support? 

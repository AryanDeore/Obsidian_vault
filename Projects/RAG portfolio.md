This is the perfect way to document your "Proof of Work." A strong `README` helps recruiters understand the *engineering* behind the project without even running the code.

Here is a professional, high-impact `README.md` tailored to your project. It emphasizes **Architecture** and **Ops** over just "features," positioning you as a Senior AI Engineer.

***

# 🤖 RAG Portfolio Assistant

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.95+-009688?logo=fastapi&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-14.0-black?logo=next.js&logoColor=white)
![Qdrant](https://img.shields.io/badge/Vector_DB-Qdrant-red)
![LiteLLM](https://img.shields.io/badge/LLM_Gateway-LiteLLM-orange)
![Comet Opik](https://img.shields.io/badge/Observability-Comet_Opik-purple)

**Try the live demo:** [aryandeore.ai](https://www.aryandeore.ai/) *(Replace with actual link)*

## 📖 Overview
This is not just a portfolio website; it is a production-grade **Retrieval-Augmented Generation (RAG)** system that allows recruiters and engineers to "interview" my resume.

Instead of reading static text, users can ask natural language questions like *"What is Aryan's experience with MLOps?"* or *"How did he optimize costs at American Health Marketplace?"* and receive grounded, cited answers based on my professional history.

## 🏗 Architecture
The core differentiator of this project is the data engineering strategy used to prevent context loss.

### 1. Schema-Aware Parent-Child Chunking
Standard recursive text splitting often breaks semantic links between entities (e.g., separating a "Skill" from the "Project" it was used in).
*   **Solution:** I implemented a hierarchical indexing strategy.
*   **Parent Chunks:** Full JSON objects (e.g., an entire Work Experience entry) containing complete context.
*   **Child Chunks:** Smaller semantic units (paragraphs) used for dense vector retrieval.
*   **Retrieval:** We search against Child chunks but retrieve the Parent chunk for generation. This ensures the LLM always has the full context (dates, company, role) when answering.

### 2. The Stack
*   **Frontend:** Next.js (React), Tailwind CSS, shadcn/ui.
*   **Backend:** Python, FastAPI.
*   **Vector Database:** Qdrant (Cosine Similarity).
*   **Embedding Model:** `BAAI/bge-small-en-v1.5` (via FastEmbed).
*   **LLM Orchestration:** LiteLLM (Unified interface for model fallback/switching).
*   **Provider:** OpenRouter (Access to GPT-4, Llama 3, etc.).
*   **Observability:** Comet Opik (Tracing, Cost Analysis, Response Evaluation).

## 🚀 Key Features
*   **⚡ Sub-Second Latency:** optimized streaming response pipeline achieving <1s TTFT (Time To First Token).
*   **🛡️ Guardrails:** Integrated OpenAI Moderation API to block harmful queries before they reach the inference layer.
*   **📚 Verifiable Citations:** The system reduces hallucinations by citing specific sections of the resume/portfolio as the source of truth.
*   **💰 Cost Efficient:** Architecture optimized to run at ~$0.001 per query avg.

## 📊 LLMOps & Observability
This project uses **Comet Opik** to trace every interaction.
*   **Traces:** Full visibility into the retrieval steps (chunks retrieved, similarity scores).
*   **Cost Tracking:** Monitoring token usage per query to optimize spend.
*   **Evaluation:** Logging user feedback (thumbs up/down) to refine retrieval parameters.


---
Here is the revised README. It maintains the technical depth and professional tone but strictly avoids first-person pronouns ("I", "my", "we"), framing everything objectively as a system description.

***

# Interactive AI Portfolio

[![Live Demo](https://img.shields.io/badge/🚀_Live_Demo-Visit_Website-2ea44f?style=for-the-badge&logo=vercel)](https://www.aryandeore.ai/)
[![Stack](https://img.shields.io/badge/Stack-FastAPI_|_Next.js_|_Qdrant_|_LiteLLM_|_Comet_Opik-blue)](https://github.com/AryanDeore/RAG-portfolio)

**An interactive, production-grade Retrieval-Augmented Generation (RAG) system that allows users to "interview" a professional portfolio.**

Instead of presenting static text, this application enables recruiters and engineers to ask natural language questions—such as *"What is the experience with MLOps?"* or *"How were costs optimized in previous roles?"*—and receive grounded, cited answers based on verified professional history.


## Architecture

The logic is divided into three distinct pipelines:

![System Architecture](./path-to-your-excalidraw-diagram.png)

### 1. Ingestion Pipeline:
One of the primary challenges in RAG is that standard recursive text splitters often sever the semantic link between specific entities (e.g., a "Skill") and the larger context (e.g., the "Project" it belongs to).

*   **Strategy:** **Schema-Aware Parent-Child Chunking**.
*   The system respects the JSON schema of the source data. Small "Child" chunks (paragraphs) are indexed for dense vector retrieval, but they remain linked to their "Parent" chunks (full job entries).
*   **Outcome:** When a specific skill is queried, the system retrieves the *entire* job entry, ensuring the LLM maintains full context regarding dates, roles, and company details.
### 2. Retrieval Pipeline:

*   **Guardrails:** All incoming queries are pre-validated via the OpenAI Moderation API to intercept and block harmful inputs before they reach the inference layer.
*   **Embeddings:** **FastEmbed** is utilized to run the `BAAI/bge-small-en-v1.5` model locally on the server. This approach eliminates external embedding API costs and reduces latency.
*   **Vector Search:** **Qdrant** performs a cosine similarity search to identify the most relevant child chunks before fetching their associated parent contexts.
### 3. Generation Pipeline:
* **OpenRouter** allows for model hot-swapping (e.g., GPT-4 to Sonnet 3.5) via configuration changes.
*   **LiteLLM** serves as a unified interface, decoupling the backend logic from specific providers. 
*   **Streaming:** Responses are delivered via Server-Sent Events (SSE) to the Next.js frontend, achieving a **Time-To-First-Token (TTFT) of <1 second**.

## Observability
**Comet Opik** tracks every trace, every interaction within the system. This monitors:
*   **Traces:** Full visibility into retrieval steps, including the exact chunks retrieved and their similarity scores.
*   **Cost Analysis:** Token usage tracking per query to optimize spend (maintaining an average of **~$0.001 per query**).

---

## 💻 Tech Stack

| Component | Technology | Role in Architecture |
| :--- | :--- | :--- |
| **Frontend** | [Next.js](https://nextjs.org/) | React Server Components for seamless streaming. |
| **Backend** | [FastAPI](https://fastapi.tiangolo.com/) | Async Python support for high-concurrency RAG. |
| **Vector DB** | [Qdrant](https://qdrant.tech/) | Advanced filtering and scalable vector storage. |
| **Embeddings** | [FastEmbed](https://qdrant.github.io/fastembed/) | Local inference using `BAAI/bge-small-en-v1.5`. |
| **LLM Proxy** | [LiteLLM](https://litellm.ai/) | Unified API surface to prevent vendor lock-in. |
| **LLMOps**| [Comet Opik](https://comet.com/opik) | End-to-end tracing and evaluation. |

---

## 🧠 Engineering Challenges Solved

### 1. Hallucinations vs. Grounding
**Challenge:** LLMs can generate plausible but incorrect information when context is missing.
**Solution:** The system enforces **Verified Citations**. The model is instructed to cite specific sections of the portfolio (e.g., "Project A, GitHub Link") as the source of truth. If the requisite context is absent, the system is designed to admit ignorance rather than hallucinate.

### 2. Provider Fragmentation
**Challenge:** Hardcoding specific API providers (like `openai-python`) limits future flexibility.
**Solution:** By utilizing **LiteLLM**, the logic is decoupled from the provider. Queries can be routed to different models based on complexity or cost requirements without altering the core business logic.

---

## 🔗 Links

*   **[Live Application](https://www.aryandeore.ai/)**
*   **[GitHub Repository](https://github.com/AryanDeore/RAG-portfolio)**
*   **[Connect on LinkedIn](https://www.linkedin.com/in/aryan-deore/)**


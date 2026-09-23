# 🌱 Biodiversity Intelligence Agent

An agentic AI system that acts as an environmental scientist — giving evidence-based land, soil, and biodiversity recommendations grounded in real scientific literature (FAO, IPCC, IPBES), with autonomous tool-calling for live weather lookups and conversational reasoning.

---

## Overview

Most LLM chatbots generate plausible-sounding advice without grounding it in real evidence, and struggle to reliably decide *when* to use which capability. This project explores building a system that reasons about that itself — using an agentic architecture where the LLM autonomously routes between retrieval-augmented advisory, live data lookup, and conversational handling, rather than following a rigid, hardcoded flow.

The assistant can:
- 🌍 Give scientifically-grounded biodiversity/land/soil recommendations, citing the exact source report and page number
- ☁️ Look up live weather for a location
- 💬 Handle casual conversation naturally
- 🚫 Politely decline off-topic questions instead of hallucinating an answer

---

## Key Features

- **Agentic tool-calling** — the LLM decides per-turn whether to reply directly, ask a clarifying question, or invoke a tool (RAG retrieval or weather API), using [LangGraph](https://www.langchain.com/langgraph)
- **Grounded RAG pipeline** — recommendations are generated only from retrieved excerpts of real scientific reports, never from the model's unguided knowledge
- **Multi-variable reasoning** — every recommendation connects at least 3 environmental variables (e.g. soil carbon, rainfall, land use, biodiversity) instead of giving single-factor, generic advice
- **Topic-switch aware memory** — correctly distinguishes a new, unrelated query from a continuation of the current conversation, avoiding stale context bleeding into new answers
- **Reliability engineering** — a deterministic fallback guarantees tool execution even when the LLM doesn't reliably trigger a function call on its own

---

## Architecture

```
User Message
     │
     ▼
┌─────────────────────────┐
│   LangGraph Agent (LLM)  │  ← decides the next action per turn
└────────────┬──────────────┘
             │
   ┌─────────┼─────────────┐
   ▼         ▼             ▼
Greeting   Off-topic    Tool call needed
reply      decline           │
              ┌───────────────┴───────────────┐
              ▼                                 ▼
      get_weather(location)         biodiversity_advice(
                                       soil_organic_carbon,
                                       rainfall, crop, region)
                                                 │
                                                 ▼
                                    ┌─────────────────────────┐
                                    │   RAG Retrieval (Chroma) │
                                    └────────────┬──────────────┘
                                                 ▼
                                    Top-k relevant chunks from
                                    FAO / IPCC / IPBES reports
                                                 ▼
                                    LLM synthesizes a structured,
                                    cited recommendation
```

**Knowledge ingestion pipeline** (run offline, once per source update):

```
PDF reports (FAO, IPCC, IPBES)
        │
        ▼
Text extraction (page-by-page)
        │
        ▼
Chunking (LangChain RecursiveCharacterTextSplitter, with overlap)
        │
        ▼
Embedding (HuggingFace sentence-transformers)
        │
        ▼
ChromaDB (persistent vector store, page-cited metadata)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Agent orchestration | LangGraph (tool-calling / ReAct-style agent) |
| LLM | Groq (free tier) |
| Embeddings | HuggingFace `sentence-transformers` (local, no API cost) |
| Vector store | ChromaDB (persistent, local) |
| Text splitting | LangChain `RecursiveCharacterTextSplitter` |
| Weather data | OpenWeatherMap API |
| Backend / UI | FastAPI, Streamlit |
| PDF processing | pypdf |

**Knowledge sources:** FAO — *Recarbonizing Global Soils* (2021); IPCC — *Special Report on Climate Change and Land* (2019); IPBES — *Pollinators, Pollination and Food Production* (2016). ~4,500 page-cited chunks total.

---

## Evaluation

Rather than relying only on manual spot-checks, the RAG pipeline's output quality was measured with an **LLM-as-judge evaluation framework** (methodology aligned with RAGAS), scoring on a 0–5 scale across a held-out set of test queries spanning soil health, pollinators, land use, rainfall, and pesticide-related questions:

| Metric | What it measures | Score |
|---|---|---|
| Faithfulness | Does the answer stick to what's actually in the retrieved context (no hallucinated claims)? | _[fill in from `eval_results.csv`]_ |
| Answer Relevancy | Does the answer directly address the question asked? | _[fill in]_ |
| Context Relevance | Are the retrieved chunks actually relevant to the question? | _[fill in]_ |

A separate automated **behavioral test suite** covers 8 conversational scenarios (greeting handling, clarifying-question logic, topic-switch memory, off-topic rejection, weather tool flow) — all passing.

---

## Key Engineering Decisions

- **Model reliability over assumption:** early testing showed the LLM would sometimes acknowledge a task ("glad I could help!") without actually invoking the required tool. Rather than trust this blindly, I benchmarked several free Groq models for function-calling reliability and added a self-correcting nudge plus a deterministic fallback that guarantees the tool still runs — a pattern I'd consider essential for any production agentic system, not just a nice-to-have.
- **Automated ingestion over static facts:** the knowledge base is built entirely from real PDF scientific reports via an automated pipeline, rather than hand-written facts — making it straightforward to expand with new sources.
- **Custom evaluation over unavailable tooling:** when the `ragas` package hit an unresolved dependency conflict, I implemented the same LLM-as-judge methodology directly, keeping the evaluation approach conceptually equivalent while avoiding a fragile dependency.

---

## Status

🚧 Actively being extended — next steps include deploying to Render for public access and expanding the evaluation test set.

---

*Built as a personal project to deepen hands-on experience with agentic architectures and RAG systems, alongside professional work on production voice AI agents.*

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

**Knowledge ingestion pipeline** :

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



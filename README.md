<div align="center">

# 🩺 AI-First CRM — Healthcare Professional (HCP) Module

**An Agentic, Conversational CRM Platform for Life Sciences Field Representatives**

[![React](https://img.shields.io/badge/Frontend-React%2018-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![Redux Toolkit](https://img.shields.io/badge/State-Redux%20Toolkit-764abc?style=for-the-badge&logo=redux&logoColor=white)](https://redux-toolkit.js.org/)
[![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/Orchestration-LangGraph-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain-ai.github.io/langgraph/)
[![Groq](https://img.shields.io/badge/Inference-Groq%20LPU-F55036?style=for-the-badge)](https://groq.com/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![RAG](https://img.shields.io/badge/Hybrid%20RAG-FAISS%20%2B%20BM25-FF6F00?style=for-the-badge)](https://github.com/facebookresearch/faiss)

[Overview](#-overview) • [Architecture](#-system-architecture) • [Key Features](#-key-features) • [LangGraph Agent & Tools](#-langgraph-agent--tools) • [Hybrid RAG](#-hybrid-rag-pipeline) • [Project Structure](#-project-structure) • [Getting Started](#-getting-started) • [API Reference](#-api-reference)

</div>

---

## 📌 Overview

Pharmaceutical and life sciences field representatives spend a disproportionate amount of their workday manually filling out complex post-call forms, looking up historical physician notes, and cross-referencing clinical literature.

**AI-First CRM (HCP Module)** reimagines the **Log Interaction Screen** by combining a structured enterprise CRM form with an autonomous **LangGraph AI Agent** powered by **Groq** (`gemma2-9b-it` / `llama-3.3-70b-versatile`) and a **Hybrid RAG pipeline (FAISS + BM25)**. Field reps can log calls, modify specific fields, search physician profiles, query clinical documentation, and schedule next-best follow-up actions entirely through natural language—reducing manual data entry overhead by **~70%** while maintaining strict life-sciences compliance.

---

## 🏗️ System Architecture

The application follows a decoupled client-server architecture where the **React + Redux** frontend synchronizes form state in real time with structured tool outputs returned by the **FastAPI + LangGraph** backend.

```mermaid
flowchart TB
    subgraph Client["Frontend (React + Redux Toolkit + Tailwind CSS)"]
        UI_Form["📋 Structured Interaction Form<br/>(Left Pane)"]
        UI_Chat["💬 Conversational AI Assistant<br/>(Right Pane)"]
        Redux["🔄 Redux Store<br/>(interactionSlice / hcpSlice / chatSlice)"]
        UI_Form <--> Redux
        UI_Chat <--> Redux
    end

    subgraph Server["Backend (Python + FastAPI)"]
        API["⚡ FastAPI REST & Agent Router"]
        
        subgraph Orchestration["LangGraph Stateful Agent (ReAct / StateGraph)"]
            Router["🧠 Agent Node<br/>Groq LLM (gemma2-9b-it / llama-3.3-70b)"]
            State["📦 AgentState<br/>(messages, current_form_state, hcp_context)"]
            Router <--> State
        end

        subgraph Tools["Specialized CRM Agent Tools"]
            T1["🛠️ log_interaction<br/>(Entity Extraction & Form Sync)"]
            T2["✏️ edit_interaction<br/>(Targeted Field Mutation)"]
            T3["🔍 search_hcp<br/>(Profile & History Lookup)"]
            T4["📅 suggest_followup<br/>(Next-Best-Action Engine)"]
            T5["📚 summarize_and_retrieve<br/>(Clinical RAG & Call Summary)"]
        end

        API --> Orchestration
        Router --> T1 & T2 & T3 & T4 & T5
    end

    subgraph DataLayer["Persistence & Knowledge Layer"]
        PG[("🐘 PostgreSQL<br/>(HCPs, Interactions, Follow-ups)")]
        RAG[("🗂️ Hybrid Retriever<br/>(FAISS Dense + BM25 Lexical)")]
    end

    Redux <==>|"HTTP / REST JSON"| API
    T1 & T2 & T3 & T4 --> PG
    T5 --> RAG
    T5 --> PG
```

---

## ✨ Key Features

- **🖥️ Dual-Mode Split-Screen Interface:**
  - **Left Pane (Structured Form):** Full manual control over HCP Name, Interaction Type, Date & Time, Attendees, Topics Discussed, Materials Shared, Samples Distributed, HCP Sentiment, Outcomes, and Follow-up Actions.
  - **Right Pane (Conversational AI Co-Pilot):** Natural language chat interface that extracts entities from free-text or voice-transcribed call notes and populates or updates the left-pane form instantaneously via **Redux Toolkit**.
- **🧠 Stateful LangGraph Orchestration:**
  - Built on `StateGraph` with deterministic tool routing, conversational memory, and current form-state awareness so partial edits never overwrite existing fields.
- **⚡ Ultra-Low Latency Inference via Groq:**
  - Uses Groq's LPU inference engine (`gemma2-9b-it` and `llama-3.3-70b-versatile`) for near-instant entity extraction, JSON schema binding, and tool invocation.
- **📚 Context-Grounded Hybrid RAG (FAISS + BM25):**
  - Combines dense semantic embeddings (**FAISS**) with sparse keyword matching (**BM25**) to accurately retrieve drug dosages, contraindications, clinical trial outcomes, and product monographs.
- **📈 Memory-Driven Next-Best-Action (NBA):**
  - Analyzes historical HCP sentiment, past sample drops, and clinical inquiries to recommend compliant, high-impact follow-up schedules.

---

## 🛠️ LangGraph Agent & Tools

The core intelligence of the platform resides in a LangGraph agent equipped with **5 domain-specific CRM tools**:

| # | Tool Name | Purpose & Mechanism | Trigger Example |
| :--- | :--- | :--- | :--- |
| **1** | `log_interaction` | Parses unstructured meeting notes into structured CRM fields (HCP Name, Interaction Type, Date/Time, Topics, Materials, Samples, Sentiment, Outcomes) and dispatches state updates to the UI form and database. | *"Met Dr. Ananya Verma today at 2 PM in-person. Discussed CardioStat Phase III efficacy, dropped 5 sample packs, positive sentiment."* |
| **2** | `edit_interaction` | Performs surgical delta updates on the currently active interaction draft or a saved record without wiping unrelated form fields. | *"Actually, change the sentiment to Neutral and update the sample count to 3 packs."* |
| **3** | `search_hcp` | Queries PostgreSQL for HCP profiles by name, specialty, hospital affiliation, or territory, returning prescription tier and past interaction history. | *"Find all Cardiologists in Apollo Hospital or pull up Dr. Verma's past 3 visits."* |
| **4** | `suggest_followup` | Evaluates the current interaction outcome, HCP sentiment, and historical engagement cadence to generate prioritized Next-Best-Actions and schedule dates. | *"What follow-up actions should I schedule for Dr. Verma based on today's call?"* |
| **5** | `summarize_interaction` / `retrieve_clinical_docs` | Generates an executive compliance-ready call summary and queries the **FAISS + BM25** hybrid retriever to answer product/clinical questions cited in the meeting. | *"Summarize this interaction and pull clinical data on CardioStat renal dosing for our follow-up email."* |

---

## 🔎 Hybrid RAG Pipeline

Medical and pharmaceutical terminology requires both conceptual understanding and exact lexical matching (e.g., drug names, NCT trial numbers, dosage units like `10mg/mL`).

```text
User / Rep Query ──┬──► [Dense Retriever: FAISS Vector Index] ──────┐
                   │                                                ├──► [Reciprocal Rank Fusion (RRF)] ──► Top-K Context ──► Groq LLM
                   └──► [Sparse Retriever: BM25 Keyword Index] ─────┘
```

1. **Ingestion & Chunking:** Clinical PDFs, product monographs, and compliance guidelines are parsed and split into semantic chunks with metadata tags (`product_name`, `indication`, `section`).
2. **Dual Indexing:**
   - **FAISS:** Captures semantic similarity for broad clinical questions.
   - **BM25:** Captures exact matches for drug codes, adverse event terms, and specific biomarker names.
3. **Ensemble Reranking:** Combines results via weighted Reciprocal Rank Fusion (RRF) before injecting grounded context into the LangGraph agent state.

---

## 🧰 Tech Stack

| Layer | Technology | Role |
| :--- | :--- | :--- |
| **Frontend** | React 18 (Vite), JavaScript / TypeScript | Component-driven split-screen user interface |
| **State Management** | Redux Toolkit (`@reduxjs/toolkit`), React-Redux | Bi-directional synchronization between AI chat responses and form inputs |
| **Styling & Typography** | Tailwind CSS, Lucide Icons, **Google Inter** Font | Clean, accessible clinical enterprise UI |
| **Backend Framework** | Python 3.10+, FastAPI, Uvicorn | High-performance async REST API |
| **AI Orchestration** | LangGraph, LangChain Core | Stateful multi-tool agent routing & memory persistence |
| **LLM Inference** | Groq API (`gemma2-9b-it`, `llama-3.3-70b-versatile`) | High-speed structured output extraction & reasoning |
| **RAG & Search** | FAISS (`faiss-cpu`), `rank_bm25`, HuggingFace Embeddings | Hybrid semantic + lexical retrieval over clinical documents |
| **Database & ORM** | PostgreSQL, SQLAlchemy, Pydantic v2 | Relational storage for HCP profiles, call logs, and follow-up schedules |

---

## 📂 Project Structure

```text
ai-first-crm-hcp/
├── backend/
│   ├── app/
│   │   ├── main.py                 # FastAPI application entrypoint & CORS config
│   │   ├── config.py               # Environment variables & settings management
│   │   ├── database.py             # PostgreSQL SQLAlchemy engine & session setup
│   │   ├── models/                 # SQLAlchemy ORM models (HCP, Interaction, FollowUp)
│   │   ├── schemas/                # Pydantic request/response & structured output schemas
│   │   ├── routers/                # API routes (interactions, hcps, agent chat)
│   │   ├── agent/
│   │   │   ├── graph.py            # LangGraph StateGraph definition & conditional edges
│   │   │   ├── state.py            # AgentState schema definition
│   │   │   ├── tools.py            # 5 core LangGraph tools (@tool definitions)
│   │   │   └── prompts.py          # System prompts for pharma CRM extraction & compliance
│   │   ├── rag/
│   │   │   ├── ingest.py           # Document chunking & index builder
│   │   │   └── retriever.py        # Hybrid FAISS + BM25 ensemble retriever
│   │   └── seed.py                 # Database seeding script for demo HCPs & interactions
│   ├── data/                       # Clinical brochures & product knowledge base docs
│   ├── requirements.txt            # Python dependencies
│   └── .env.example                # Backend environment template
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── InteractionForm.jsx # Left pane: Structured CRM Log Interaction form
│   │   │   ├── AIChatPanel.jsx     # Right pane: Conversational LangGraph assistant
│   │   │   ├── HCPSelector.jsx     # Searchable HCP dropdown & profile badge
│   │   │   └── SentimentBadge.jsx  # Visual indicator for Positive / Neutral / Negative
│   │   ├── store/
│   │   │   ├── index.js            # Redux store configuration
│   │   │   ├── interactionSlice.js # Form state & AI tool delta reducer
│   │   │   └── chatSlice.js        # Conversation history & agent tool-call trace state
│   │   ├── services/
│   │   │   └── api.js              # Axios / Fetch client for FastAPI endpoints
│   │   ├── App.jsx                 # Split-screen layout container
│   │   ├── main.jsx                # React DOM root with Redux Provider
│   │   └── index.css               # Tailwind imports & Google Inter font configuration
│   ├── package.json
│   └── vite.config.js
│
└── README.md
```

---

## 🗄️ Database Schema

### `hcps` (Healthcare Professionals)
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | `UUID / Serial PK` | Unique HCP identifier |
| `name` | `VARCHAR(150)` | Full name (e.g., *Dr. Sarah Smith*) |
| `specialty` | `VARCHAR(100)` | Medical specialty (Oncology, Cardiology, Neurology) |
| `institution` | `VARCHAR(200)` | Hospital or clinic affiliation |
| `email` / `phone` | `VARCHAR` | Contact information |
| `tier` | `VARCHAR(20)` | Prescriber tier (`Tier 1`, `Tier 2`, `KOL`) |

### `interactions` (Logged Call Records)
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | `UUID / Serial PK` | Unique interaction log ID |
| `hcp_id` | `FK -> hcps.id` | Associated Healthcare Professional |
| `interaction_type` | `VARCHAR(50)` | `Meeting`, `Phone Call`, `Email`, `Conference` |
| `date` / `time` | `DATE` / `TIME` | Date and time of interaction |
| `attendees` | `TEXT / JSONB` | List of participating clinicians/reps |
| `topics_discussed` | `TEXT` | Key clinical/product discussion points |
| `materials_shared` | `JSONB` | Brochures, clinical trial PDFs, slide decks shared |
| `samples_distributed` | `JSONB` | Drug samples and quantities dropped |
| `sentiment` | `VARCHAR(20)` | `Positive`, `Neutral`, or `Negative` |
| `outcomes` | `TEXT` | Key takeaways or prescription commitments |
| `follow_up_actions` | `TEXT` | Next steps & scheduled follow-up tasks |
| `ai_summary` | `TEXT` | Auto-generated executive summary |

---

## 🚀 Getting Started

### Prerequisites
- **Python** `3.10+`
- **Node.js** `18+` & `npm`
- **PostgreSQL** `14+` (Local instance or cloud provider such as Neon / Supabase)
- **Groq API Key** (Get one at [console.groq.com](https://console.groq.com/))

---

### 1. Clone the Repository
```bash
git clone [https://github.com/AbhyanandSharma2005/ai-first-crm-hcp.git](https://github.com/AbhyanandSharma2005/ai-first-crm-hcp.git)
cd ai-first-crm-hcp
```

### 2. Database Setup
Open `psql` or pgAdmin and create the database:
```sql
CREATE DATABASE hcp_crm;
```

### 3. Backend Setup (FastAPI + LangGraph)
```bash
cd backend

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

Create a `.env` file inside the `backend/` directory:
```env
# LLM Configuration
GROQ_API_KEY=gsk_your_groq_api_key_here
PRIMARY_LLM_MODEL=gemma2-9b-it
FALLBACK_LLM_MODEL=llama-3.3-70b-versatile

# Database Configuration
DATABASE_URL=postgresql://postgres:your_password@localhost:5432/hcp_crm

# Server Settings
APP_ENV=development
PORT=8000
```

Seed the database with sample HCP profiles and start the server:
```bash
# Optional: Seed initial HCP records and build RAG indices
python -m app.seed

# Start FastAPI server
uvicorn app.main:app --reload --port 8000
```
- **API Base URL:** `http://localhost:8000`
- **Interactive Swagger Docs:** `http://localhost:8000/docs`

---

### 4. Frontend Setup (React + Redux)
Open a new terminal window:
```bash
cd frontend

# Install packages
npm install

# Start Vite development server
npm run dev
```
- **Web App URL:** `http://localhost:5173`

---

## 📡 API Reference

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/agent/chat` | Sends user message + current Redux form state to the LangGraph agent; returns assistant message, invoked tool trace, and updated form fields. |
| `GET` | `/api/hcps` | Retrieves list of Healthcare Professionals (supports `?query=` and `?specialty=` filters). |
| `GET` | `/api/hcps/{id}` | Fetches full HCP profile and historical interaction logs. |
| `POST` | `/api/interactions` | Saves a new HCP interaction directly from the structured form or AI confirmation. |
| `PUT` | `/api/interactions/{id}` | Updates an existing interaction record. |
| `GET` | `/api/interactions` | Lists all logged interactions ordered by timestamp. |

---

## 💬 Sample Walkthrough Prompts

Test all **5 LangGraph tools** directly in the right-hand AI Assistant panel:

1. **Log Interaction (`log_interaction`):**
   > *"I just met Dr. Rajesh Khanna (Cardiologist) at 11:30 AM today. We discussed the Phase III efficacy of OncoShield and its dosing schedule. I shared the clinical study PDF and handed over 4 starter sample kits. He had a very positive sentiment and agreed to evaluate it for new patients."*
   *(Watch the left-side form auto-populate all fields via Redux!)*

2. **Edit Specific Fields (`edit_interaction`):**
   > *"Change the sentiment to Neutral, update the sample quantity to 2 starter kits, and add Dr. Meera Nair as an attendee."*
   *(Only the targeted fields update; the rest of your form remains intact.)*

3. **Search HCP Directory (`search_hcp`):**
   > *"Search our database for Oncologists or show me previous interactions with Dr. Rajesh Khanna."*

4. **Suggest Follow-Up / Next Best Action (`suggest_followup`):**
   > *"Based on this discussion and Dr. Khanna's questions, what follow-up actions should I take next week?"*

5. **Summarize & Clinical RAG Lookup (`summarize_interaction`):**
   > *"Generate a concise compliance summary of this call and check our product documentation for OncoShield's contraindications."*

---

## 👤 Author

**Abhyanand Sharma**
- **GitHub:** [@AbhyanandSharma2005](https://github.com/AbhyanandSharma2005)

---

## 📄 License

This project is licensed under the **MIT License** — see the `LICENSE` file for details.
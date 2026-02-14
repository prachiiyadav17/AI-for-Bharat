# Technical Design Document (TDD)
## Project Name: Eklavya.AI

---

## 1. High-Level Architecture
We utilize a **Micro-Services Architecture** (simulated for Hackathon via Modular Monolith) centered around a **RAG (Retrieval Augmented Generation)** pipeline.

**System Flow:**
`[Mobile App]` <-> `[API Gateway]` <-> `[Orchestrator]` <-> `[LLM + Vector DB]`

```mermaid
graph TD
    User[Student (Flutter App)] -->|Image/Voice| API[FastAPI Gateway]
    API -->|Async Task| Worker[Celery Worker]
    
    subgraph AI_Processing_Pipeline
        Worker -->|OCR| Vision[Google Vision / Tesseract]
        Vision -->|Text| RAG[Vector Search (NCERT DB)]
        RAG -->|Context + Query| LLM[Gemini 1.5 Flash]
        LLM -->|JSON Response| TTS[Bhashini / Edge TTS]
    end
    
    TTS -->|Audio Stream| S3[Object Storage / Cache]
    S3 -->|Stream URL| User
```
## 2. Tech Stack Selection
Component,Choice,Why this for the Hackathon?## 🛠 Tech Stack Decisions

| Component      | Choice                     | Why this for the Hackathon? |
|---------------|----------------------------|-----------------------------|
| Frontend      | Flutter                    | Cross-platform, high performance on low-end Androids, excellent camera libraries. |
| Backend       | FastAPI (Python)           | Native async support (crucial for AI streaming), auto-generated Swagger docs for judges. |
| AI Model      | Gemini 1.5 Flash           | High speed, low cost, and massive context window (can read whole chapters). |
| Vector DB     | ChromaDB                   | Lightweight, runs locally (no cloud setup needed for demo). |
| TTS (Audio)   | Edge-TTS / Bhashini        | Edge-TTS is free and high quality. Bhashini is the Govt. of India standard (bonus points). |
| Database      | SQLite                     | Zero config, file-based. Perfect for MVP. |

##3. Detailed Component Design
3.1 The "Context Injection" Engine
Standard AI hallucinations are dangerous in education. We solve this using RAG:

Ingestion: We pre-process NCERT PDF textbooks.

Chunking: Split text into semantic chunks (e.g., "Paragraph + Heading").

Embedding: Convert chunks to vectors using sentence-transformers/all-MiniLM-L6-v2 (Fast & Light).

Retrieval: When user asks about "Gravity", we fetch the exact NCERT definition first, then force the LLM to simplify that specific text.

3.2 The "Vernacular Personality" Prompting
We don't just ask for translation. We use System Prompting to enforce a persona.

System Prompt Template:

"You are 'Eklavya', a wise and friendly village teacher in India.
User Language: {dialect}
Context: {ncert_context}
Task: Explain the concept using analogies from farming, cricket, or local festivals.
Tone: Encouraging, respectful ('Beta', 'Dekho').
Output: JSON { 'explanation_text': '...', 'analogy': '...' }"

3.3 Network Optimization (The "2G" Strategy)
Protocol: Use HTTP/2 for multiplexing.

Audio Format: transcode all TTS output to Opus/Ogg at 16kbps (Voice profile). This reduces a 1MB MP3 to a ~50KB file, playable on 2G.

Optimistic UI: Show the text explanation immediately while the audio buffers in the background.

##4. Database Schema

# 🗄 Database Schema Design

This project uses a lightweight relational database (SQLite) to manage users, cache AI responses, and track learning progress.

---

## 1️⃣ Users Table – Store User Preferences

```sql
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    name TEXT,
    dialect TEXT DEFAULT 'hindi_std', -- e.g., 'bhojpuri', 'marathi'
    grade_level INTEGER
);
```

### 🔎 Purpose:
- Stores basic user information
- Saves preferred dialect for TTS personalization
- Tracks grade level for adaptive AI responses

### 💡 Why This Matters:
Enables personalized learning experience and regional language support.

---

## 2️⃣ Query Cache – Reduce AI API Costs

```sql
CREATE TABLE query_cache (
    query_hash TEXT PRIMARY KEY, -- Hash of (Image_Content + User_Query)
    response_json TEXT,
    audio_path TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 🔎 Purpose:
- Stores previously generated AI responses
- Saves processed audio file paths
- Avoids repeated API calls for identical queries

### 💡 Why This Matters:
- Reduces Gemini API cost
- Improves response speed
- Makes the system scalable

---

## 3️⃣ Learning Logs – Track Student Progress

```sql
CREATE TABLE learning_logs (
    id SERIAL PRIMARY KEY,
    user_id TEXT,
    topic_tag TEXT, -- 'physics_gravity'
    sentiment_score FLOAT -- Did they understand? (derived from follow-up chat)
);
```

### 🔎 Purpose:
- Tracks which topics a student interacts with
- Measures understanding using sentiment score
- Enables adaptive revision suggestions

### 💡 Why This Matters:
This allows future implementation of:
- Weak topic detection
- AI-driven revision plans
- Performance analytics dashboard

---

# 🏗 Design Philosophy

- Lightweight (SQLite)
- Cost-efficient (Query caching)
- Scalable for future analytics
- Personalized learning focused

---

# 🚀 Future Improvements

- Add foreign key constraints
- Add indexing for faster lookup
- Add authentication token storage
- Move to PostgreSQL for production scaling

## 5. Security & Ethics
Content Safety: All LLM outputs pass through a "Guardrail" check to ensure no political/religious bias is generated.

Data Minimization: Images uploaded for OCR are processed in RAM and deleted immediately after text extraction.

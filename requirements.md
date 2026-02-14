# Product Requirements Document (PRD)
## Project Name: Eklavya.AI
**Tagline:** *Bridging India's Digital Divide with Vernacular, Multimodal AI.*

---

## 1. Executive Summary
**Problem:** 65% of Indian students reside in rural areas. While digital infrastructure (UPI/4G) has scaled, *intellectual access* remains gated by English proficiency and generic, non-contextual learning content.
**Solution:** Eklavya.AI is a "Hyper-Local" AI Tutor. It uses Computer Vision to "read" physical textbooks and Generative AI to explain concepts in the student's specific dialect (e.g., Bhojpuri, Marathi, Tamil) using culturally relevant analogies (e.g., explaining 'Force' using farming examples).

---

## 2. Target Audience & Persona
* **Primary User:** *Rohan (14, Village in Maharashtra)*.
    * **Device:** Low-end Android (2GB RAM).
    * **Network:** Intermittent 4G/2G.
    * **Behavior:** Struggles to read long English paragraphs. Prefers video/audio.
* **Secondary User:** *Suresh (Parent/Farmer)*.
    * **Need:** Wants to help his child but lacks formal education. Needs Voice-First UI.

---

## 3. Functional Requirements (Hackathon MVP Scope)

### 3.1 Core Module: "Drishti" (Vision-to-Vernacular)
* **REQ-VIS-01:** User captures an image of a textbook page (English/Hindi).
* **REQ-VIS-02:** System identifies the core concept (e.g., "Photosynthesis").
* **REQ-VIS-03:** AI generates a summary **NOT** in direct translation, but as a *story* or *analogy* in the selected regional dialect.

### 3.2 Core Module: "Vaani" (Voice-First Interaction)
* **REQ-AUD-01:** Full duplex voice chat. User asks: *"Ye Newton ka third law kya hai?"*
* **REQ-AUD-02:** System responds with synthesized audio in the same language/dialect.
* **REQ-AUD-03:** **Latency Constraint:** Audio response must start playing within < 3 seconds on 4G.

### 3.3 Core Module: "Setu" (Low-Bandwidth Optimization)
* **REQ-NET-01:** **"2G Mode" Toggle:** Switches UI to text-only, disables heavy animations, and requests highly compressed audio (Opus codec, 8kbps).
* **REQ-OFF-01:** Smart Caching. Once a topic (e.g., "Gravity") is fetched, the text/audio is stored locally (SQLite) for zero-data replay.

---

## 4. Non-Functional Requirements (NFRs)
* **Scalability:** Stateless backend architecture to handle spike traffic (exam season).
* **Privacy:** No PII (Personally Identifiable Information) sent to LLM providers. Images processed in memory and discarded.
* **Accessibility:** Large touch targets (min 48x48dp) for users with rough hands (manual labor background).
* **Performance:** App package size (APK) must be < 25 MB.

---

## 5. Success Metrics (KPIs)
* **Engagement:** Average Session Time > 5 minutes.
* **Retention:** "Day 7 Retention" > 15% (Students coming back next week).
* **Utility:** % of "Thumbs Up" on AI explanations.

# RecruiterAI — Intelligent Recruitment Assistant

An AI-powered recruitment dashboard that helps recruiters analyze job descriptions, evaluate resumes, score candidate-to-job fit, find skill gaps, generate interview questions and export reports.

**Live demo:** `https://your-site-name.netlify.app` (replace with your Netlify link)

---

## Project Overview

RecruiterAI is designed to feel like a real HR SaaS product, not a ChatGPT clone. A recruiter can upload resumes, paste a job description, see match scores, compare candidates and ask questions in plain English. All answers are grounded in the uploaded documents.

This repository contains:

| Version | What it is |
|---|---|
| `index.html` | Single-file web demo. No install or build step. Deployable to Netlify. |
| Full Streamlit app (roadmap) | Python backend with real RAG (ChromaDB + LLM). See [Architecture](#architecture). |

> **Note:** The web demo uses a built-in keyword retriever instead of a live LLM, so it runs fully in the browser with sample data. The RAG pipeline below describes the full Python version.

## Problem Statement

Recruiters spend hours reading resumes, comparing them against job requirements and writing interview questions. This is slow and inconsistent, and keyword search misses context. RecruiterAI speeds up screening by extracting structured data, scoring job-relevant evidence and explaining every score.

## Features

- **Dashboard:** candidate stats, recent candidates table, status badges, charts
- **Candidate Analysis:** resume upload, extracted details, search and filters
- **Job Description Analyzer:** required skills, preferred skills, experience, education, location
- **Resume Matcher:** 0–100 score with breakdown (technical skills, experience, education, projects, tools), matching and missing skills, and a written explanation
- **Candidate Comparison:** side-by-side ranking with reasons
- **Interview Question Generator:** 20 questions per candidate (Technical, Project-Based, Behavioral, Situational)
- **Candidate Summary:** overview, strengths, projects, gaps, evidence-based recommendation
- **Chat Assistant:** natural-language questions answered from candidate documents, with source citations
- **Analytics:** skill demand, common candidate skills, shortlist rate, interview conversion
- **Export:** CSV and PDF reports
- **Friendly error handling:** invalid file, unsupported format, empty resume, no results, empty question

## Architecture

```
Resume / Job Description
        ↓
Document Extraction (PyPDF / python-docx)
        ↓
Text Chunking
        ↓
Embeddings
        ↓
Vector Database (ChromaDB)
        ↓
Similarity Search
        ↓
Relevant Chunks
        ↓
LLM (Gemini / OpenAI / Groq)
        ↓
Recruiter Response (with sources)
```

### Planned project structure

```
recruiter_ai/
├── app.py
├── requirements.txt
├── README.md
├── .env
├── .gitignore
├── data/
│   ├── candidates/
│   └── job_descriptions/
├── backend/
│   ├── resume_parser.py
│   ├── job_analyzer.py
│   ├── matcher.py
│   ├── rag.py
│   ├── interview_generator.py
│   └── chatbot.py
├── database/
│   └── database.py
├── utils/
│   ├── document_loader.py
│   └── helpers.py
└── vector_db/
```

## Tech Stack

| Layer | Web demo | Full version |
|---|---|---|
| Frontend | HTML, CSS, vanilla JavaScript | Streamlit |
| Backend | Runs in browser | Python (optional FastAPI) |
| AI | Built-in retriever | Gemini / OpenAI / Groq |
| RAG | Keyword retrieval | ChromaDB + embeddings |
| Documents | `.txt` | PyPDF, python-docx |
| Data | In-memory sample data | SQLite or PostgreSQL |
| Hosting | Netlify | Streamlit Cloud / Render |

## How It Works

1. **Job description analysis:** known skills are extracted and split into required and preferred (text after "preferred" or "nice to have" counts as preferred). Experience, education and location are detected with pattern matching.
2. **Resume extraction:** name, email, phone, education, skills and years of experience are pulled from the resume text. Missing fields are reported as unavailable, never guessed.
3. **Match scoring:** the overall score is a weighted blend:

   | Component | Weight |
   |---|---|
   | Technical skills (required) | 40% |
   | Experience | 20% |
   | Projects | 15% |
   | Tools (preferred) | 15% |
   | Education | 10% |

   Scores of 80+ are a Strong Match, 65–79 a Potential Match, and below 65 a Weak Match.
4. **Explanation:** every score comes with a plain-English reason listing matched and missing skills.
5. **Chat:** the question is matched to candidate data, and the answer cites which resume it came from.

## Installation

### Web demo

No installation needed. Open `index.html` in any browser.
)
### Deploy to netlify (https://hilarious-malabi-9feb67.netlify.app/
2. Drag `index.html` into the page
3. Netlify gives you a live link

Or connect this GitHub repo to Netlify and set the publish directory to the repo root.

### Full Python version (roadmap)

```bash
git clone https://github.com/<your-username>/recruiter_ai.git
cd recruiter_ai
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```

## Environment Variables

Create a `.env` file (never commit it):

```
GEMINI_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
GROQ_API_KEY=your_key_here
LLM_PROVIDER=gemini
```

The web demo needs no keys.

## Screenshots

Add your screenshots to a `screenshots/` folder and link them here:

```
![Dashboard](screenshots/dashboard.png)
![Resume Matcher](screenshots/matcher.png)
![Chat Assistant](screenshots/chat.png)
```

## Sample Questions

- Show candidates with Python and SQL
- Which candidates have experience with FastAPI?
- Who has the strongest machine learning experience?
- Compare Ayesha Khan and Priya Sharma
- What skills are missing for Marcus Lee?
- Generate interview questions for Ayesha Khan
- Does Daniel Moore have Kubernetes experience?

## RAG Architecture

The full version answers only from uploaded documents. Resumes and job descriptions are split into chunks and embedded, then stored in ChromaDB. At question time the top matching chunks are retrieved and passed to the LLM together with a strict system prompt: answer only from the provided context, cite the source, and say so when information is missing.

## AI Safety

- Never invents candidate information
- Says "I couldn't find this information in the uploaded resume." when data is missing
- Ignores protected characteristics (age, gender, ethnicity, religion, nationality and similar) and declines questions that ask for them
- Bases recommendations only on job-relevant evidence from the resume and job description
- Explains every score so a human can review it
- Keeps candidate data private. In the web demo, nothing leaves the browser.
- Recommendations are decision support only. A person makes the final hiring decision.

## Future Improvements

- PDF and DOCX upload in the browser
- Real LLM and ChromaDB integration with a serverless proxy for API keys
- Semantic skill matching (for example, "ML" and "machine learning")
- Bias audit and fairness evaluation of scores
- User accounts and per-recruiter data isolation
- ATS and calendar integrations
- Evaluation set to measure extraction and ranking accuracy

## Deployment Instructions

- **Web demo:** Netlify, Vercel or GitHub Pages (static hosting)
- **Streamlit version:** Streamlit Community Cloud, Render or Railway. Add API keys as environment secrets, never in the repo.

## What I Learned

- **LLM application development:** structuring an app around model calls, with fallbacks when the API fails
- **RAG:** chunking, embedding, retrieval and grounding answers in source documents
- **Vector databases:** storing and querying embeddings with ChromaDB
- **Prompt engineering:** writing system prompts that limit answers to evidence and handle missing data
- **Document processing:** extracting structured data from PDF, DOCX and TXT resumes
- **AI evaluation:** checking score fairness, extraction accuracy and hallucination risk
- **API integration:** connecting Gemini, OpenAI or Groq and handling keys and errors
- **Streamlit:** building a multi-page dashboard in Python
- **Git/GitHub:** version control, branching, a clean README and deployment

## License

MIT. Sample candidates in this project are fictional.
# 🏥 Free Health Camp Finder

A responsive, accessible web app that helps people discover free and low-cost medical camps in their city. Built as a single static HTML file (no build step, no dependencies) and ready to deploy on Netlify or GitHub Pages.

> ⚠️ **Demo project.** All camps, organizations and contact details are **sample data** and are not real medical information. This platform only lists information; medical services are provided by the respective organizers. In a medical emergency, call **108**.

## ✨ Features

**For the public**
- Search by city, area, pincode, camp name, organization or specialty, with suggestions as you type
- Filters for date, distance, specialty, services and availability
- List view and map view (map placeholder, ready for a real map API)
- Camp details with eligibility, requirements, slots left and a registration countdown
- Registration form with validation, duplicate-registration protection and a downloadable confirmation
- Save/bookmark camps, "My Registrations" and a profile page
- Notifications UI (confirmation, reminders, cancellations, schedule changes)
- Report incorrect information (wrong date, fake camp, wrong contact and more)
- Share, WhatsApp share, one-tap call and directions

**For organizers and admins**
- Organizer dashboard: add a camp, update slots, cancel a camp
- Admin dashboard: stats, approve/reject/remove camps, view reports, simple analytics charts

**Accessibility and UX**
- Mobile-first layout with a bottom navigation bar
- Large text and touch targets, high contrast, keyboard focus styles, form labels
- 🔊 Read Aloud (browser speech synthesis)
- 🌙 Dark mode and 🌐 Tamil + English toggle (main labels)
- 🤖 Camp Assistant that helps find camps and never gives medical advice

## 🚀 Getting started

No installation needed.

```bash
git clone https://github.com/<your-username>/free-health-camp-finder.git
cd free-health-camp-finder
# open index.html in your browser
```

### Deploy

- **Netlify:** drag the folder or `index.html` onto [Netlify Drop](https://app.netlify.com/drop), or connect the repo (no build command, publish directory `/`).
- **GitHub Pages:** Settings → Pages → Deploy from branch → `main` / root.

## 🗂️ Project structure

```
.
├── index.html   # entire app: HTML, CSS and JavaScript
└── README.md
```

Inside `index.html` the code is organized in sections: config and translations, sample data, state, components, pages, router, event handlers and the assistant.

## 🔌 Demo limitations and next steps

| Area | Current (demo) | Production plan |
|---|---|---|
| Authentication | Role switcher in Profile (user / organizer / admin) | Real auth (Firebase, Supabase, Auth0) with server-side role checks |
| Data | Sample array and `localStorage` | Database (PostgreSQL, Firestore) behind an API |
| Map | Placeholder with clickable markers | Google Maps or Leaflet/OpenStreetMap |
| Distance | Fixed sample values | Geolocation plus distance calculation |
| Assistant | Keyword matching | LLM API with the camp database as context |
| Notifications | Static UI | Email, SMS or WhatsApp service |
| Poster upload | Field only | File storage (S3, Cloudinary) |
| Tamil | Navigation and key labels | Full translation of all pages |
| Analytics | Computed from sample data (monthly chart is sample numbers) | Real event and registration data |

## 🔐 Privacy and security notes

- Registration data stays in the user's browser in this demo and is never shown publicly.
- Forms validate inputs and all user-entered text is HTML-escaped before display.
- Real deployments must add server-side validation, authentication, rate limiting and a privacy-compliant data policy before collecting real personal or health-related information.

## 🛠️ Tech

HTML5, CSS3 (CSS variables, grid, flexbox) and vanilla JavaScript with a hash-based router. There are no frameworks, so it is easy to read, fork and migrate to React or Vue.

## 📄 License

MIT. Add a `LICENSE` file before publishing.

## 🙌 Contributing

Issues and pull requests are welcome, especially for translations, accessibility improvements and real map or backend integrations.
Netlify Free Health Camp Finder https://share.google/zOw6YURO5EAcTGj9L

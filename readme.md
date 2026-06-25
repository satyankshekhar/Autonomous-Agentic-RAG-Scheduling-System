# 🎓 Autonomous Agentic RAG Scheduling System

An autonomous, agentic AI system for college admissions, built with **Flask**, **LangGraph**, **ChromaDB**, and **OpenAI GPT-4o**. It answers prospectus-based queries via RAG, schedules counselling calls via Calendly, and captures leads automatically.

---


## ✨ Features

- 🤖 **RAG-powered Q&A** — Answers admission queries grounded in the college prospectus (PDF/DOCX)
- 📅 **Calendly Scheduling** — Books counselling calls via multi-turn slot-filling (name → email → time)
- 🧠 **Intent Detection** — GPT-4o classifies each message as a general query or a scheduling request
- 📊 **Lead Generation** — Auto-scores and saves leads to CSV on session end
- 💬 **Conversation Memory** — Session-based history across multi-turn interactions
- ⏱️ **Inactivity Timeout** — Auto-triggers lead capture after 1 hour of inactivity

---

## 🏗️ Architecture

```
User Message
     │
     ▼
Intent Detection (GPT-4o)
     │
     ├──► "instruction" ──► Scheduler Node (multi-turn slot fill) ──► Calendly API
     │
     └──► "prompt"      ──► RAG Node (ChromaDB + GPT-4o)          ──► Answer

                                   │
                                   ▼
                         Lead Gen Node (on session end)
                                   │
                                   ▼
                             Leads.csv
```

**Tech Stack:**

| Layer | Technology |
|---|---|
| Web Framework | Flask + Flask-Session |
| LLM | OpenAI GPT-4o / GPT-4.1 |
| Orchestration | LangGraph |
| Vector DB | ChromaDB (persistent) |
| Embeddings | OpenAI `text-embedding-3-small` |
| Scheduling | Calendly API |
| Document Parsing | PyPDF2, python-docx |

---

## 📁 Project Structure

```
Autonomous-Agentic-RAG-Scheduling-System/
├── app.py                  # Flask app — routes, session logic, intent routing
├── main.py                 # RAG initialization and admission_enquiry() entrypoint
├── graph.py                # LangGraph agents (RAG, Scheduler, Lead Gen nodes)
├── tools.py                # Utilities: PDF/DOCX extraction, ChromaDB, GPT, lead CSV
├── calendly_tool.py        # Calendly availability check + invitee creation
├── requirements.txt        # Python dependencies
├── Knowledge/
│   ├── Prospectus.docx     # College prospectus (RAG knowledge source)
│   └── Leads.csv           # Auto-generated lead capture file
├── templates/
│   └── admission_agent.html  # Chat UI
├── chroma_db/              # Persistent ChromaDB storage (auto-created)
└── flask_session/          # Server-side session storage (auto-created)
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Python 3.9+
- An [OpenAI API key](https://platform.openai.com/api-keys)
- A [Calendly Personal Access Token](https://calendly.com/integrations/api_webhooks)

### 1. Clone the Repository

```bash
git clone https://github.com/satyankshekhar/Autonomous-Agentic-RAG-Scheduling-System.git
cd Autonomous-Agentic-RAG-Scheduling-System
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create a `.env` file in the project root (or export variables directly):

```env
OPENAI_API_KEY=sk-...your-key-here...
CALENDLY_TOKEN=eyJ...your-token-here...
```

> ⚠️ **Never commit API keys to Git.** Add `.env` to your `.gitignore`.

### 5. Add the Knowledge Document

Place your college prospectus at:

```
Knowledge/Prospectus.docx   # or .pdf — update the path in main.py
```

### 6. Run the App

```bash
python app.py
```

Visit `http://127.0.0.1:5000` in your browser.

---

## 🔑 Environment Variables

| Variable | Description | Required |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI API key for GPT-4o and embeddings | ✅ Yes |
| `CALENDLY_TOKEN` | Calendly Personal Access Token | ✅ Yes |

---

## 🗺️ API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Renders the chat UI |
| `POST` | `/admission-message` | Handles user messages; returns bot reply |

**Request body (`/admission-message`):**

```json
{
  "message": "What are the fees for B.Tech?",
  "name": "Ravi Kumar",
  "mobile": "9876543210"
}
```

**Response:**

```json
{
  "reply": "The B.Tech fee structure for NIT Surat is..."
}
```

---

## 📅 Scheduling Flow

The scheduler collects three slots in sequence before booking:

1. **Name** — Pre-filled from the initial form if available
2. **Email** — Extracted via regex from user message
3. **Date & Time** — Parsed from natural language (e.g., "tomorrow 3 PM IST")

Once all three are collected, the agent checks availability on Calendly and creates the booking.

**Supported time formats:**
- `"tomorrow 4 PM"`
- `"today at 2:30 PM"`
- ISO 8601 UTC strings

---

## 📊 Lead Generation

Leads are automatically scored (0–100) by GPT-4o and saved to `Knowledge/Leads.csv` when:

- The user sends a closing message (`"thank you"`, `"done"`, `"that's all"`, etc.)
- The session has been inactive for over 1 hour

**Scoring rubric:**

| Score | Intent Level | Example Query |
|---|---|---|
| 90–100 | High — ready to apply | "What documents do I need for admission?" |
| 50–89 | Medium — evaluating | "Compare B.Tech CSE and ECE programs" |
| 0–49 | Low — just browsing | "Tell me about the college" |

---

## 🛠️ Customisation

- **Change the knowledge source:** Update `file_path` in `main.py` to point to your own PDF or DOCX
- **Change the Calendly event type:** Update `event_type_uri` in `graph.py → scheduler_node()`
- **Swap the college name:** Update the system prompt inside `rag_search()` in `tools.py`
- **Adjust inactivity timeout:** Edit `INACTIVITY_TIMEOUT` in `app.py`

---

## 🚧 Known Limitations

- Scheduling only supports IST timezone (can be extended in `graph.py`)
- ChromaDB is re-populated on every cold start (can be optimised with a startup check)
- API keys are hardcoded as fallbacks in source files — move all secrets to environment variables before deploying

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 👤 Author

**Satyank Shekhar**
- GitHub: [@satyankshekhar](https://github.com/satyankshekhar)
- Email: satyank.shekhar.11@gmail.com

---

> Built with ❤️ for smarter college admissions.

# Karnataka Yatri — Tourism Chatbot (Flask + Ollama)

A web-based chatbot that helps visitors get information about **Karnataka
state only** — regions, villages, places, food, transportation, payment
modes, mobile network coverage, stay types, and ratings.

Built with **Flask** for the backend/frontend, **Bootstrap 5** for styling,
and **llama3.2:latest** (via [Ollama](https://ollama.com)) as the local
language model.

---

## Features

- Chat interface in the browser (Bootstrap-styled, no external DB required)
- Covers 11 Karnataka regions: Bengaluru, Mysuru, Hampi, Coorg,
  Mangalore-Udupi, Gokarna, Badami, Hubballi-Dharwad, Bidar, Bandipur, and
  Jog Falls
- Retrieval-grounded answers — the app pulls only the relevant region/category
  data (food, transport, payment, network, stay, rating, places, best time to
  visit) and feeds it to the model as context, instead of relying purely on
  the model's own knowledge
- Stays scoped to Karnataka — politely declines unrelated questions
- Clickable region badges to quickly ask about a specific place
- Fully local — no external API keys or internet-hosted LLM required

---

## Project Structure

```
karnataka_flask/
├── app.py                 # Flask backend, embedded knowledge base, Ollama integration
└── templates/
    └── index.html          # Bootstrap chat UI
```

---

## Prerequisites

1. **Python 3.8+**
2. **Ollama** — download and install from https://ollama.com/download
3. **llama3.2 model** pulled locally:
   ```bash
   ollama pull llama3.2:latest
   ```
4. Ollama server running (most installs auto-start it as a background
   service; otherwise start manually):
   ```bash
   ollama serve
   ```

---

## Installation

```bash
# 1. Clone or copy the project folder
cd karnataka_flask

# 2. (Recommended) create a virtual environment
python3 -m venv venv
source venv/bin/activate     # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install flask requests
```

---

## Running the App

```bash
python3 app.py
```

Then open your browser at:

```
http://localhost:5000
```

---

## Usage

Type any Karnataka-related travel question into the chat box, for example:

- "What food should I try in Coorg?"
- "How do I get to Hampi and what's the network like there?"
- "What payment modes work in Gokarna?"
- "Which regions have the highest ratings?"
- "What kind of stays are available near Badami?"
- "Tell me about villages near Mysuru"

You can also click any of the region badges above the chat window to
auto-fill a question about that region.

If you ask about something unrelated to Karnataka, the assistant will
politely redirect you back to Karnataka travel topics.

---

## How It Works

1. **Retrieval** — `build_context()` scans your question for region names
   (and their aliases, e.g. "bangalore" → Bengaluru) and category keywords
   (food, transport, payment, network, stay, rating, places, best time to
   visit).
2. **Context building** — only the matching slice of the embedded knowledge
   base is serialized and attached to your question. If nothing specific
   matches, a short overview of all regions is used instead.
3. **Generation** — the context + your question + a system prompt (which
   restricts the assistant to Karnataka travel topics) are sent to
   `llama3.2:latest` via Ollama's local `/api/chat` endpoint.
4. **Response** — the model's reply is returned as JSON and rendered in the
   chat window, with the last 10 turns kept as conversation history for
   follow-up questions.

---

## Extending the Knowledge Base

All region data lives in the `REGIONS` dictionary inside `app.py`. To add a
new region (e.g. Chikmagalur, Belur-Halebidu, Vijayapura), add a new entry
following the same structure:

```python
"RegionName": {
    "aliases": ["alt name 1", "alt name 2"],
    "district": "...",
    "type": "...",
    "villages_and_places": [...],
    "food": [...],
    "transport": [...],
    "payment_modes": [...],
    "network": "...",
    "stay_types": [...],
    "rating": 4.5,
    "best_time_to_visit": "...",
},
```

No other code changes are needed — the chatbot and the region badge list on
the homepage will automatically pick up the new entry.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| "Couldn't reach Ollama at localhost:11434" | Run `ollama serve` and confirm it's listening on port 11434 |
| Model not found errors | Run `ollama pull llama3.2:latest` |
| Slow responses | Normal for larger models on CPU-only machines; try a smaller model or enable GPU support in Ollama |
| Port 5000 already in use | Change the port in the last line of `app.py`: `app.run(debug=True, host="0.0.0.0", port=5001)` |

---

## Tech Stack

- **Backend:** Flask (Python)
- **Frontend:** HTML, Bootstrap 5, Font Awesome, vanilla JavaScript (fetch API)
- **LLM:** llama3.2:latest via Ollama (local inference, no cloud API needed)

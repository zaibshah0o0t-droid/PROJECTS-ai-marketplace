# YT AutoGen — Ollama Edition

> **100% local, 100% free.** YouTube script, SEO, thumbnail, and voiceover generation powered by your own local Ollama model — no API keys, no cloud bills.

This is a clean rebuild that replaces the broken Gemini integration with **Ollama** for all text generation, and adds **voiceover audio** generation (gTTS/edge-tts) that the old version was missing entirely.

---

##  What Was Fixed

| Issue (old project) | Fix (this project) |
|---|---|
| Thumbnail not generating | Rebuilt PIL engine from scratch + optional Stable Diffusion via Replicate |
| Script not generating | New Ollama-powered script service with bullet-proof fallback templates |
| SEO not generating | New Ollama-powered SEO service, same fallback safety net |
| Voice not generating | **New feature** — edge-tts primary, gTTS fallback, fully wired into `/api/generate` |
| Gemini API dependency | **Removed entirely** — runs 100% locally via Ollama |

Every generator (script, SEO, thumbnail, audio) has its own independent fallback, so **a single failing component never breaks the whole request** — you always get a usable response.

---

## Quick Start

### Step 1 — Install Ollama (one-time)

**Windows / macOS / Linux:**
1. Download from **https://ollama.com/download**
2. Install and run it — it starts a background service on `http://127.0.0.1:11434`

**Pull a model** (choose one):
```bash
ollama pull llama3.2      # recommended — fast, good quality, ~2GB
# OR
ollama pull mistral       # alternative — slightly larger, ~4GB
```

**Verify it's running:**
```bash
ollama list
```
You should see your model listed.

---

### Step 2 — Set up the backend

```bash
cd yt-autogen-ollama

python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

pip install -r backend/requirements.txt
```

---

### Step 3 — Configure environment

```bash
cp .env.example backend/.env
```

Open `backend/.env` — the defaults work out of the box if Ollama is running locally:
```env
OLLAMA_HOST=http://127.0.0.1:11434
OLLAMA_MODEL=llama3.2
```

> If you pulled `mistral` instead, change `OLLAMA_MODEL=mistral`.

---

### Step 4 — Start the backend

```bash
uvicorn backend.main:app --reload --host 127.0.0.1 --port 8000
```

You should see:
```
INFO:  Ollama reachable — using model 'llama3.2'
INFO: YT AutoGen (Ollama Edition) started 
```

If you see ` Ollama NOT reachable`, the app **still works** — it automatically uses high-quality fallback templates instead of failing.

---

### Step 5 — Open the frontend

Right-click `frontend/login.html` → **Open with Live Server** (VS Code extension), or:
```bash
cd frontend
python -m http.server 5500
```
Then visit `http://127.0.0.1:5500/login.html`

---

### Step 6 — Use it

1. Register an account → you land on the dashboard
2. Enter a topic, e.g. `Top 5 AI Tools`
3. Click ** Generate Everything**
4. Review Script / Thumbnail / SEO / Voiceover tabs
5. Download/copy whatever you need

---

## 📡 API Reference

### `POST /api/generate` — the main endpoint

**Request:**
```json
{
  "topic": "Top 5 AI Tools",
  "tone": "Educational",
  "duration": 10,
  "thumbnail_style": "Cinematic",
  "target_audience": "Content creators",
  "language": "English",
  "include_audio": true
}
```
*(requires `Authorization: Bearer <token>` header — register/login first)*

**Response:**
```json
{
  "success": true,
  "project_id": 1,
  "topic": "Top 5 AI Tools",
  "script": {
    "hook": "...", "intro": "...", "body": "...", "outro": "...",
    "timestamps": ["0:00 - Intro", "2:30 - Tool 1", "..."],
    "full_script": "..."
  },
  "seo": {
    "titles": ["...", "..."],
    "description": "...",
    "tags": ["...", "...20 total"],
    "hashtags": ["...", "...15 total"],
    "category": "Education",
    "best_upload_time": "Tuesday or Thursday, 2-4 PM EST",
    "keyword_difficulty": "Medium",
    "trending_score": 74
  },
  "thumbnail": "base64_jpeg_string",
  "audio": "base64_mp3_string",
  "word_count": 1487,
  "errors": {}
}
```

> `errors` will contain a per-component message if something failed (e.g. `{"audio": "..."}`) — the rest of the response is still usable.

### Other endpoints
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Create account |
| POST | `/api/auth/login` | Login |
| GET | `/api/auth/me` | Current user |
| GET | `/api/auth/stats` | Usage stats |
| GET | `/api/health/ollama` | Check if Ollama is reachable + which model |

Interactive docs: **http://127.0.0.1:8000/docs**

---

## Testing the API directly (curl)

```bash
# 1. Register
curl -X POST http://127.0.0.1:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"demo","email":"demo@test.com","password":"password123","confirm_password":"password123"}'

# Copy the access_token from the response, then:

# 2. Generate content
curl -X POST http://127.0.0.1:8000/api/generate \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"topic":"Top 5 AI Tools","tone":"Educational","duration":10,"thumbnail_style":"Bold","include_audio":true}'
```

---

## Thumbnail Generation

By default, thumbnails are generated with a **professional PIL-based engine** — gradients, decorative elements, YouTube-style play button, drop-shadow text, 5 distinct styles (Cinematic, Bold, Minimalist, Colorful, Gaming). This always works, no API key required.

**Optional: Stable Diffusion upgrade**
If you want AI-generated background art instead of gradients, get a free token at **https://replicate.com** and add to `.env`:
```env
REPLICATE_API_TOKEN=r8_your_token_here
```
The app will automatically try Replicate first and fall back to PIL if it fails or isn't configured.

---

##  Voiceover Generation

Two free, no-API-key engines, tried in order:
1. **edge-tts** (Microsoft Edge neural voices — higher quality)
2. **gTTS** (Google Translate TTS — reliable fallback)

Supports English, Urdu, Hindi, Spanish, French, Arabic, German, Portuguese.

> Voiceover requires internet access (these are free cloud TTS services, not local). If you need fully offline TTS, consider adding `pyttsx3` — not included by default to keep voice quality high.

---

##  Troubleshooting

| Problem | Fix |
|---|---|
| `Ollama NOT reachable` | Run `ollama serve` or open the Ollama desktop app. The app still works using fallback templates. |
| Script/SEO feels generic | Ollama isn't running — start it and regenerate. Check `/api/health/ollama` |
| `ModuleNotFoundError: ollama` | `pip install ollama` |
| Audio always empty | Check internet connection — edge-tts/gTTS need it. Check terminal logs for the real error. |
| CORS error in browser | Make sure `FRONTEND_URL` in `.env` matches your frontend's actual origin (e.g. `http://127.0.0.1:5500`) |
| `ModuleNotFoundError: No module named 'backend'` | You're running uvicorn from the wrong folder — `cd` into the project root first |
| Slow generation | Normal for local LLMs on CPU — first request loads the model into memory. Try a smaller model like `llama3.2:1b` |
| bcrypt error on register | `pip install bcrypt==4.0.1 passlib==1.7.4` |

---

## Project Structure

```
yt-autogen-ollama/
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   ├── auth/                    # register/login/JWT
│   ├── routers/
│   │   └── generate.py          # the /api/generate endpoint
│   ├── services/
│   │   ├── ollama_service.py    # script + SEO via Ollama (+ fallbacks)
│   │   ├── image_service.py     # PIL thumbnail engine (+ optional Replicate SD)
│   │   └── voice_service.py     # edge-tts + gTTS voiceover
│   ├── models/schemas.py        # request/response Pydantic models
│   └── utils/database.py        # SQLAlchemy models
├── frontend/
│   ├── index.html / login.html / register.html
│   ├── style.css / auth.css
│   └── app.js
├── .env.example
└── README.md
```

---

##  Notes on Reliability

- Every AI call (script, SEO) has a hand-written fallback template — if Ollama is down, slow, or returns malformed JSON, you still get a complete, usable result.
- Thumbnail generation **never** depends on an external API by default — PIL works 100% offline.
- The `/api/generate` response includes an `errors` object so the frontend can show partial-failure messages without losing the data that *did* generate successfully.

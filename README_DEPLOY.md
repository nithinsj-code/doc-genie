# DocGenius — Deploy Guide (Netlify + Render)

This project was restructured from the original single-file Streamlit app into:

```
docgenius-deploy/
├── backend/              → FastAPI API, deploy this to Render
│   ├── main.py
│   ├── requirements.txt
│   ├── runtime.txt
│   └── .env.example
├── frontend/              → Static site, deploy this to Netlify
│   ├── index.html
│   ├── style.css
│   ├── app.js
│   ├── config.js          → put your Render URL here
│   └── netlify.toml
└── render.yaml             → optional one-click Render blueprint
```

Why restructure? Netlify only hosts **static** sites (HTML/CSS/JS) — it cannot
run a long-lived Python/Streamlit process. Render hosts your **Python API**
(FastAPI) as an always-on web service. The two talk to each other over HTTPS.

You'll need two API keys before you start:
- **Gemini API key** → https://aistudio.google.com/app/apikey
- **OpenAI API key** (used for embeddings + PDF Q&A) → https://platform.openai.com/api-keys

---

## Step 1 — Push the code to GitHub

1. Create a new GitHub repo (e.g. `docgenius`).
2. Copy the `backend/` and `frontend/` folders (and `render.yaml`) into it.
3. Commit and push:
   ```bash
   git init
   git add .
   git commit -m "DocGenius: split into frontend + backend"
   git branch -M main
   git remote add origin https://github.com/<your-username>/docgenius.git
   git push -u origin main
   ```
   (Render and Netlify can both deploy directly from this repo on every push.)

---

## Step 2 — Deploy the backend to Render

1. Go to https://dashboard.render.com → **New +** → **Web Service**.
2. Connect your GitHub repo.
3. Configure:
   | Setting | Value |
   |---|---|
   | **Root Directory** | `backend` |
   | **Runtime** | Python 3 |
   | **Build Command** | `pip install -r requirements.txt` |
   | **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
   | **Instance Type** | Free (or paid for no cold-starts) |
4. Under **Environment Variables**, add:
   - `GEMINI_API` = your Gemini key
   - `OPENAI_API_KEY` = your OpenAI key
   - `ALLOWED_ORIGINS` = `*` for now (you'll tighten this in Step 4)
5. Click **Create Web Service**. Wait for the build to finish.
6. Copy your live URL, e.g. `https://docgenius-backend.onrender.com`.
7. Sanity check it: open `https://docgenius-backend.onrender.com/api/health`
   in your browser — you should see `{"status":"ok", ...}`.

   *Tip:* a `render.yaml` is included at the repo root if you'd rather use
   Render's "Blueprint" one-click deploy instead of manual setup.

   *Free-tier note:* Render's free web services spin down after 15 minutes
   of inactivity and take ~30–60 seconds to wake up on the next request —
   that's expected, not a bug.

---

## Step 3 — Deploy the frontend to Netlify

1. Open `frontend/config.js` and set it to your Render URL from Step 2:
   ```js
   window.DOCGENIUS_API_BASE = "https://docgenius-backend.onrender.com";
   ```
   Commit and push this change.
2. Go to https://app.netlify.com → **Add new site** → **Import an existing project**.
3. Connect the same GitHub repo.
4. Configure:
   | Setting | Value |
   |---|---|
   | **Base directory** | `frontend` |
   | **Build command** | *(leave empty — it's static)* |
   | **Publish directory** | `frontend` (or `.` if base directory is already `frontend`) |
5. Click **Deploy site**. Netlify gives you a URL like
   `https://docgenius-ai.netlify.app`.

   *Alternative (no GitHub needed):* drag the `frontend` folder straight onto
   https://app.netlify.com/drop for an instant manual deploy.

---

## Step 4 — Connect them securely (CORS)

Go back to Render → your service → **Environment** → update:

```
ALLOWED_ORIGINS = https://docgenius-ai.netlify.app
```

Save — Render will redeploy automatically. This locks the API down so only
your Netlify site can call it (instead of `*`, which allows any site).

---

## Step 5 — Test it end-to-end

1. Open your Netlify URL.
2. The status strip at the top should say **"Backend connected"**.
3. Try **AI Content Generator** with any prompt.
4. Upload a PDF in **Ask Your PDF**, then ask a question about it.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| Status strip shows "Backend URL not configured" | You forgot to edit `frontend/config.js` |
| Status strip shows "Could not reach backend" | Render service is asleep (wait ~30s and refresh) or `ALLOWED_ORIGINS` doesn't include your Netlify URL |
| `500: GEMINI_API is not configured` | Add `GEMINI_API` env var on Render and redeploy |
| `500: OPENAI_API_KEY is not configured` | Add `OPENAI_API_KEY` env var on Render and redeploy |
| PDF upload works but "Ask" returns 404 session not found | Sessions live in memory and reset if Render restarts/sleeps — re-upload the PDF |
| CORS error in browser console | Double-check `ALLOWED_ORIGINS` on Render exactly matches your Netlify URL (no trailing slash) |

## Notes on this rewrite

- The original `app.py` (Gemini text generator) is now `POST /api/generate`.
- The original `DocGenius/PDFChat.py` (LangChain + FAISS PDF Q&A) is now
  `POST /api/pdf/upload` + `POST /api/pdf/ask`, using the OpenAI SDK directly
  instead of LangChain, which is lighter and more reliable to deploy.
- Uploaded PDFs are processed **in memory only** (not saved to disk) and
  expire after 1 hour or on server restart — fine for a demo, swap in
  Redis/a database if you need persistence across restarts.

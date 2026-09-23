# Griffiths Tutor

**Live demo:** _add your Railway URL here once deployed (see
"Deploying so others can use it" below)_

A Flask chat UI running the method-selection / physical-interpretation
coaching prompt, backed by Groq's free API instead of a local model.

## Why Groq instead of local Ollama

The original version of this ran fully locally via Ollama, but a
13GB+ model needs more RAM than a typical 16GB laptop has free, and
Ollama can't use Intel/AMD integrated graphics for acceleration — so
on modest hardware, local models are either painfully slow or fail to
load. Groq's free tier gives you a much larger, better model, running
on dedicated inference hardware, for no cost and no local resource use.

## Setup

1. **Get a free Groq API key**
   Sign up at [console.groq.com](https://console.groq.com), then
   create a key under API Keys. No credit card required.

2. **Add the key to a `.env` file**

   Copy the example file and fill in your real key:
   ```bash
   cp .env.example .env
   ```
   Then open `.env` and replace `your-key-here` with the key you
   copied from Groq. It should look like:
   ```
   GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxx
   ```
   `app.py` loads this automatically on startup — no need to `export`
   anything or set it per terminal session. `.env` is already listed
   in `.gitignore`, so it won't get committed if you put this project
   in git.

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the app**
   ```bash
   python app.py
   ```
   Then open http://localhost:5000

## Changing the model

`MODEL` in `app.py` is set to `openai/gpt-oss-120b`, a strong free
option on Groq. Other free models worth trying (swap the string):

| Model string               | Notes                             |
|-----------------------------|-------------------------------------|
| `openai/gpt-oss-120b`       | Strong general reasoning (default)  |
| `llama-3.3-70b-versatile`   | Solid all-rounder, slightly faster  |
| `llama-3.1-8b-instant`      | Much faster, weaker reasoning       |

Full current list: https://console.groq.com/docs/models

## Free tier limits

As of when this was set up, Groq's free tier allows roughly 30
requests/minute and 1,000 requests/day, which is well past what a
study session needs. If you hit a rate limit, the app shows the error
Groq returns (including a 429 status) rather than failing silently —
check https://console.groq.com/docs/rate-limits for current numbers,
since providers adjust these without much notice.

## Deploying so others can use it

Right now this only runs on your machine. To share it, deploy on
**Railway**, which doesn't require a credit card to start (unlike
Render, which does card verification even on its free tier; Fly.io
removed its free tier entirely).

1. **Turn this folder into a git repo, if you haven't already**
   ```bash
   git init
   git add .
   git commit -m "Griffiths tutor"
   ```
   `.gitignore` already excludes `.env`, so your API key stays local
   and never gets committed.

2. **Push it to GitHub**
   Create a new (can be private) repo on github.com, then:
   ```bash
   git remote add origin https://github.com/your-username/your-repo.git
   git branch -M main
   git push -u origin main
   ```

3. **Create a Railway project**
   At [railway.com](https://railway.com), sign up (no card required),
   New Project → Deploy from GitHub repo → select this repo. Railway
   reads `requirements.txt` and the `Procfile` automatically.

4. **Add your API key as an environment variable on Railway**
   In the service's Variables tab, add `GROQ_API_KEY` with your real
   key. This is separate from your local `.env` file — Railway never
   sees that file since it's gitignored.

5. **Generate a public domain**
   Under the service's Settings → Networking, click "Generate Domain"
   to get a public URL like `your-app.up.railway.app`. Share that link.

**Things to know about Railway's free trial:**
- You get $5 in usage credit over your first 30 days, no card needed.
  A small Flask app like this uses well under $1/month in resources,
  so the trial comfortably covers casual use.
- After 30 days (or if the credit runs out), Railway asks for a
  payment method to continue — at that point it's roughly $1/month
  minimum. There's no way around eventually needing a card if you
  want this to keep running indefinitely; the trial just buys you
  card-free time now.
- Everyone using the link shares your one Groq API key and its rate
  limit (roughly 30 requests/min, 1,000/day) — fine for casual/small
  group use.
- Any time you push a new commit to `main`, Railway redeploys
  automatically.

## Mobile support

The chat UI is optimized for phones as well as desktop:
- Layout adapts to the actual visible viewport, including when the
  on-screen keyboard is open (the input bar stays visible above it
  rather than getting covered — a common mobile web bug).
- Input font is 16px to stop iOS Safari's auto-zoom-on-focus.
- Safe-area padding respects notches and the home indicator on iPhones.
- Buttons meet the 44px minimum touch-target size.
- **Add to Home Screen** works on both iOS and Android once this is
  running somewhere reachable (locally on your network, or deployed):
  open the URL in the browser, then use Share → Add to Home Screen
  (iOS) or the browser menu → Add to Home Screen / Install app
  (Android/Chrome). It'll launch full-screen with an app icon, no
  browser chrome.

## Notes

- Conversation history is kept client-side in the browser tab (a JS
  array) and resent in full on every message, since the chat API is
  stateless. Refreshing the page clears it — that's intentional for
  now (one problem per session).
- Streaming: the backend proxies Groq's server-sent-events stream and
  forwards plain text chunks; the frontend reads them via
  `ReadableStream` and re-renders as they arrive, then runs KaTeX
  over the finished bubble so LaTeX renders once the reply is done.
- Images later: Groq supports vision-capable models (check current
  options at the models link above) that accept an `image_url` content
  block per message, OpenAI-style. When you're ready, add a file input
  to the frontend, base64-encode the image client-side as a data URL,
  and include it as an `image_url` block in that message's `content`
  array — the backend passes `messages` through mostly as-is, so this
  is a small, contained change.
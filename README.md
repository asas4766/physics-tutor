# Griffiths Tutor

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
- If you ever want to go back to fully local/offline (e.g. no
  internet, or privacy-sensitive use), the previous Ollama-based
  version of `app.py` just needs `OLLAMA_URL` pointed at
  `http://localhost:11434/api/chat` and the request/response format
  swapped back — ask if you want that version restored alongside this
  one.
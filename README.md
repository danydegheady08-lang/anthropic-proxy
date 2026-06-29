# Anthropic API Proxy

> A Cloudflare Worker that sits between your browser and the Anthropic API — keeping your API key out of client-side code entirely.

---

## Why this exists

Browser-based AI apps have a problem: you can't call the Anthropic API directly without exposing your API key in the JavaScript. Anyone who opens DevTools can steal it.

This Worker solves that. It runs at the edge, injects the key server-side, and hands the response back to the browser. Your key never touches the client.

---

## How it works

```
Browser (fetch → your worker URL)
            │
            ▼
  Cloudflare Worker (edge)
  ┌─────────────────────────────┐
  │  inject x-api-key header    │
  │  from Cloudflare secret     │
  │  handle CORS preflight      │
  └─────────────────────────────┘
            │
            ▼
  Anthropic API (api.anthropic.com)
            │
            ▼
  Response back to browser
```

Zero API key exposure. Deployed globally in seconds.

---

## Deployment

```bash
npm install -g wrangler
wrangler login
wrangler secret put ANTHROPIC_API_KEY
wrangler deploy
```

> **Windows users:** run from Command Prompt, not PowerShell. PowerShell blocks wrangler script execution by default.

---

## Usage from the browser

```javascript
const response = await fetch("https://anthropic-proxy.degheady.workers.dev", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-6",
    max_tokens: 1000,
    messages: [{ role: "user", content: "Your prompt here" }]
  })
});

const data = await response.json();
console.log(data.content[0].text);
```

---

## What it handles

- `POST` — proxies the request to Anthropic with the injected API key
- `OPTIONS` — handles CORS preflight so browsers don't block the request
- Everything else — returns 405

---

## Part of

Powers the live demos at [Degheady Consulting](https://danydegheady08-lang.github.io/degheady-consulting-demos/) — AI automation for B2B sales and finance teams.

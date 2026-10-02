# Cluster — a 13-agent AI cluster behind one simple chat

Cluster looks like an ordinary chatbot. Underneath, every message runs
through a 13-agent pipeline built on Meta's Muse Spark models:

1. **Router** reads the message and decides which of 12 specialists are
   actually relevant — Researcher, Coder, Planner, Analyst, Writer, Editor,
   Strategist, Scholar, Historian, Counselor, Skeptic, Curator.
2. Only the relevant specialists run, in parallel, so a typical message
   doesn't pay the cost or latency of all 13.
3. **Router** synthesizes their notes into one natural reply.

The UI shows a small strip of 13 nodes that light up while the cluster is
working, then settles back into a plain chat once the reply lands.

## About "free to deploy"

This needed to become a real backend, not just a page. A page anyone can
open can't hold your Meta API key secretly — anyone who views the page
source could read it out and run up charges on your account. So this ships
as source code you deploy yourself, on a host where the key lives as a
private environment variable, never sent to the browser.

The good news: this is a small enough app that several hosts will run it
for free (see step 3).

## 1. Get a Meta Model API key

Sign up at Meta's developer platform (ai.developer.meta.com → Meta Model
API) and create an API key. This project defaults to the `muse-spark-1.2`
model — Meta ships new Muse Spark versions fairly often, so check your
dashboard for the current latest id and update `MUSE_MODEL_ID` in `.env` if
a newer one's out.

## 2. Run it locally

```bash
npm install
cp .env.example .env
# paste your key into .env
npm start
```

Visit http://localhost:3000.

## 3. Deploy it for free

**Render (simplest):**
1. Push this folder to a GitHub repo.
2. On render.com: New → Web Service → connect the repo.
3. Build command: `npm install` · Start command: `npm start`.
4. Add environment variable `META_API_KEY` with your key.
5. Deploy — you'll get a public `*.onrender.com` URL.

Free-tier services on Render spin down after inactivity, so the first
request after a quiet period takes ~30 seconds to wake back up. That's
normal for a free host, not a bug.

**Alternatives:** Railway and Fly.io work the same way — one Node web
service, one environment variable — and both have their own free
allowances.

## Before you make the link public

The built-in rate limiter (20 messages/minute/IP, in `server.js`) is a
basic guard against a runaway API bill, not real abuse protection. If
you're sharing this widely, put real auth in front of it or keep the link
unlisted.

## Project layout

```
server.js        Express app: routes, static hosting, rate limiting
orchestrator.js  Routing, parallel specialist calls, synthesis
agents.js        The 13 agent definitions and system prompts
museClient.js    Thin wrapper around Meta's Model API
public/          Frontend — plain HTML/CSS/JS, no build step
```

## Customizing

- Swap or rename specialists in `agents.js` — the id, name, one-line blurb,
  and system prompt are all that define an agent.
- Change how many specialists the Router can pick: the `slice(0, 4)` line
  in `orchestrator.js`.
- Add streaming later by pointing `museClient.js` at Meta's `/responses`
  endpoint instead of `/chat/completions` — it supports it, this version
  doesn't yet.

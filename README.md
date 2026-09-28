# Clawome Browser

**Open-source AI browser agent. Tell it what you want — it browses the web and brings back results.**

```bash
clawome "Find the top 3 AI stories on Hacker News today"
```

```
  > Find the top 3 AI stories on Hacker News today

  I'll browse Hacker News and find the top AI stories for you.

  [task] Opening https://news.ycombinator.com ...
  [task] Scanning front page for AI-related stories ...
  [task] Extracting titles, scores, and links ...

  [result] Here are today's top 3 AI stories on Hacker News:
  1. "GPT-5 benchmark results leaked" — 842 points
  2. "Open-source vision model beats proprietary ones" — 631 points
  3. "Show HN: AI browser agent that actually works" — 529 points
```

No browser extensions. No complex setup. Just describe what you want in plain language.

## Quick start

**Prerequisites:** Python 3.10+

```bash
pip install -r backend/requirements.txt
playwright install chromium
cp .env.example .env   # add your LLM key (Gemini free tier works)
clawome start
```

This walks you through LLM setup (Google Gemini is the default — pick any of 12+ providers), installs Chromium, and starts the server:

```
Server & Dashboard:  http://localhost:5001
```

### Run tasks from the terminal

```bash
clawome "Find AI graduate programs at Stanford"
clawome "Compare iPhone 16 Pro vs Samsung S25 Ultra specs"
clawome status          # Check progress
clawome stop            # Cancel
```

### Or use the web dashboard

Open `http://localhost:5001` — chat with the agent, watch it browse live, inspect the DOM it sees, and manage sessions.

## How it works

A two-layer agent architecture:

```
You ──→ Chat Agent ──→ Runner (Task Engine) ──→ Browser (Playwright)
         │                     │
         │ Understands         │ Plans subtasks
         │ context, calls      │ Perceive → Plan → Act → Sense
         │ browser tools,      │ Guard nodes (CAPTCHA, cookies, loops)
         │ manages sessions    │ Anomaly detection & recovery
         │ Delegates complex   │ Reports back to chat
         │ tasks to Runner     │
         └── Watchdog ─────────┘ (monitors progress, intervenes if stuck)
```

The **chat agent** handles simple questions and direct browser actions. Complex multi-step tasks are delegated to the **Runner** — a LangGraph state machine that autonomously plans, browses, and extracts information, with guard nodes that dismiss cookie banners, detect CAPTCHAs, and recover from blocked pages.

### Key features

| Feature | Description |
|---------|-------------|
| **Natural language** | Just describe what you want |
| **Chat interface** | Context-aware conversations with follow-ups |
| **Smart execution** | Perceive → Plan → Act → Sense loop with retry |
| **Guard nodes** | Auto-handles CAPTCHAs, cookie popups, blocked pages |
| **100:1 DOM compression** | 300K HTML → ~3K tokens so the LLM sees structure, not noise |
| **12+ LLM providers** | Google Gemini (default), OpenAI, Anthropic, DeepSeek, Groq, and more |
| **Browser CLI** | `open`, `click`, `type`, `scroll`, `dom`, `tabs` — drive the browser by hand |
| **Session persistence** | Resume conversations across restarts |

## DOM compression

The agent never feeds raw HTML to the LLM. A custom DOM walker compresses pages ~100:1 into an indexed element tree:

```bash
clawome open "https://example.com"
clawome dom        # read the compressed DOM the agent sees
```

## Project layout

```
backend/                 # Flask API + Playwright browser manager
  task_agent/            # LangGraph runner: planner, guard nodes, tools
  routes/                # REST API (chat, tasks, browser control)
  dom_parser.py          # DOM compression engine
clawome/                 # CLI client (clawome start/status/stop + browser commands)
frontend/                # React dashboard (chat UI, live browser view)
docs/                    # documentation site source
```

## Configuration

All settings live in `.env` (see `.env.example`): `LLM_PROVIDER`, `LLM_API_KEY`, `LLM_MODEL`, `AGENT_START_URL`, `AGENT_MAX_STEPS`. API keys are read from the environment only — never hardcoded.

## License

Apache-2.0 — see `LICENSE`.

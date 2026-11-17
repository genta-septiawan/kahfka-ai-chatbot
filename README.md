# Kahfka AI Chatbot

AI customer service chatbot for an e-commerce fashion store, built with **n8n**, **Google Gemini 2.5 Flash**, and the **Telegram Bot API**.

> **Portfolio Project** — Demonstrating no-code/low-code AI automation for real-world business use cases.

---

## Overview

Kahfka is a customer service bot for a fictional online fashion store. It handles the questions that repeat all day — sizing, shipping, payment methods, return windows — so a human only sees the ones that need judgement.

Built to show how a small business can automate first-line support with tools that cost nothing to run.

---

## Features

- **Multi-turn conversation** with context memory per user
- **Custom business persona** via system prompt engineering
- **Guardrails** against out-of-scope questions — asked about an iPhone, it redirects back to the store
- **Anti-hallucination instructions** — the bot is told never to invent prices, stock, or promotions, and to point at the website or a human admin instead
- **Session isolation** — memory is keyed on Telegram chat ID, so users never see each other's context
- **Graceful handoff** — defers to a human when the question exceeds its scope

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Automation platform | n8n (self-hosted) |
| LLM | Google Gemini 2.5 Flash |
| Messaging interface | Telegram Bot API |
| Memory store | n8n Simple Memory (session-based) |
| Tunnelling (dev) | ngrok |

Running cost: **$0** on free tiers.

---

## Workflow Architecture

```
Telegram Trigger → AI Agent → Send a text message
                      │
                      ├─ Google Gemini Chat Model
                      └─ Simple Memory (keyed on chat ID)
```

**Nodes**

- **Telegram Trigger** — listens for incoming messages
- **AI Agent** — orchestrates the model, memory, and prompt
- **Google Gemini Chat Model** — handles inference
- **Simple Memory** — stores conversation history per chat ID
- **Telegram Send Message** — replies to the user

All business knowledge lives in the AI Agent's system message. There is no database and no retrieval — see [Limitations](#limitations).

---

## Business Use Cases

- Product inquiries and FAQ
- Order status and shipping information
- Payment method questions
- Return and refund policy

Anything requiring account access, a price quote, or a commitment still routes to a human. That boundary is deliberate, not a gap.

---

## Demo

![Workflow](screenshots/canvas.png)
![Demo 1](screenshots/telegram-demo-1.png)
![Demo 2](screenshots/telegram-demo-2.png)

Tested scenarios:

- **Product inquiry** — *"Halo, jual apa aja?"*
- **Out-of-scope handling** — *"Ada iPhone 15?"* → politely redirected back to the store
- **Multi-turn memory** — *"Saya mau beli jaket"* → *"Ada ukuran XL?"* resolves against the earlier message without repeating the subject

---

## Setup

**Prerequisites**

- Node.js 18+
- n8n (`npm install -g n8n`)
- A Telegram bot token from [@BotFather](https://t.me/BotFather)
- A Google Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)

**Steps**

1. Start a tunnel: `ngrok http 5678`
2. Start n8n with the tunnel URL — Telegram rejects non-HTTPS webhooks:

   ```powershell
   $env:WEBHOOK_URL = "https://your-subdomain.ngrok-free.dev/"
   n8n start
   ```

3. Import `workflow/kahfka-cs-chatbot.json` (n8n → Import from File)
4. Set credentials on the Telegram and Gemini nodes
5. Click **Publish** — not *Execute workflow*, which only registers a temporary test webhook
6. Message the bot on Telegram

The workflow export contains credential *references* only — no keys or tokens. Select your own after importing.

---

## Limitations

The bot's knowledge is held entirely in its system prompt. That works while the facts are few and stable: store hours, couriers, the return window.

It does not scale to a real document. A prompt cannot hold a product catalogue or a policy manual, and instructing a model not to invent details reduces hallucination without eliminating it.

Addressing that properly needs retrieval — see Related Projects.

---

## Project Structure

```
kahfka-ai-chatbot/
├── README.md
├── LICENSE
└── workflow/
    └── kahfka-cs-chatbot.json
└── screenshots/
    ├── canvas.png
    ├── telegram-demo-1.png
    └── telegram-demo-2.png
```

---

## Related Projects

Built on the same architecture, each one answering a limitation of the last:

- **[bumi-trasa-sop-assistant](https://github.com/genta-septiawan/bumi-trasa-sop-assistant)** — the same workflow retargeted from retail to a civil engineering contractor by changing one system prompt. Shows the build is not tied to an industry.
- **[bumi-trasa-rag-assistant](https://github.com/genta-septiawan/bumi-trasa-rag-assistant)** — the same bot grounded in an actual 10-page SOP manual via a vector database, so answers are retrieved from the document rather than recalled from a prompt.

---

## Roadmap

- [ ] Deploy to a VPS for 24/7 uptime — ngrok only runs while the local machine does
- [ ] Connect to a live product source (Google Sheets / Airtable) so stock answers are real
- [ ] Retry handling for transient model API failures
- [ ] WhatsApp Business API support
- [ ] Session timeout (24h idle reset)

---

## Author

**Genta Septiawan** — Civil Engineer moving into AI automation

- [LinkedIn](https://www.linkedin.com/in/gentaseptiawan/)
- Available for freelance AI automation work

---

## License

MIT — use it as a reference for your own projects.

---

*Built as part of an AI Automation Engineer portfolio, 2026*

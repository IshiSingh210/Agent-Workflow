# n8n Developer Agent — Agent That Builds Agents

Describe any automation you want in plain English. This workflow builds a ready-to-import n8n workflow for it, creates it in your n8n instance automatically, and hands you a direct link to open it.

> Built under the guidance and mentorship of [Dr. Kanad Basu](https://www.marshall.usc.edu/personnel/kanad-basu), USC Marshall School of Business

---

## What It Does

1. You send a message in the chat: *"Build me a workflow that monitors a Slack channel and summarizes messages daily"*
2. The agent passes your request to a sub-workflow (the Builder)
3. The Builder pulls your n8n documentation from Google Drive for context
4. GPT-4o-mini generates a complete, valid n8n workflow JSON from scratch
5. The workflow gets created directly in your n8n instance via the n8n API
6. You get back a clickable link: **View your finished workflow**

---

## Architecture

This is a two-workflow system. The frontend agent talks to the user; the builder sub-workflow does the heavy lifting.

```
[ Chat Trigger ]
      │
      ▼
[ n8n Developer Agent ]  ◄──  [ GPT 4.1 mini via OpenRouter ]
      │                   ◄──  [ Simple Memory ]
      │
      ▼
[ Developer Tool ] ──► triggers sub-workflow ──►
                                                  [ Get n8n Docs (Google Drive) ]
                                                          │
                                                          ▼
                                                  [ Extract Text from File ]
                                                          │
                                                          ▼
                                                  [ n8n Builder Agent ]
                                                  (GPT-4o-mini)
                                                  Generates workflow JSON
                                                          │
                                                          ▼
                                                  [ n8n API: Create Workflow ]
                                                          │
                                                          ▼
                                                  [ Return workflow link ]
                                                          │
      ◄─────────────────────────────────────────────────-┘
      │
      ▼
"View your finished workflow" ← clickable link shown in chat
```

Two workflows in one JSON export:
- **n8n Developer** — the chat-facing agent that receives requests and presents results
- **Workflow Builder** — the sub-workflow that generates and deploys the JSON

---

## What Gets Generated

The Builder produces import-ready n8n workflow JSON that includes:

- All nodes with correct types and parameters
- Properly wired connections between nodes
- A trigger node appropriate to the use case
- Sticky notes explaining each section and flagging credentials that need to be configured
- Valid settings object with execution order and data saving preferences

The output is sent directly to your n8n instance via API — no copy-pasting required.

---

## Setup

### Prerequisites

- n8n (self-hosted — required for the n8n API integration)
- OpenAI API key
- OpenRouter account (for GPT 4.1 mini)
- Google Drive with an n8n documentation file (see Step 3)

### Step 1 — Import the workflow

Copy `workflow.json` and import it into n8n via **Workflows → Import from file**. This imports both the Developer agent and the Builder sub-workflow together.

### Step 2 — Add credentials

In n8n Credentials, configure:

- **OpenAI** — used by the Builder (GPT-4o-mini) for JSON generation
- **OpenRouter** — used by the Developer agent (GPT 4.1 mini) for conversation
- **n8n API** — used to create workflows programmatically in your instance
- **Google Drive OAuth2** — used to fetch your documentation file

Assign each to its respective node.

### Step 3 — Add your n8n documentation

The Builder reads a Google Doc at runtime to understand n8n's node types, connection syntax, and JSON structure. This is what makes the generated JSON accurate.

Create a Google Doc with n8n workflow documentation — you can use the [official n8n docs](https://docs.n8n.io), a curated reference you've built, or a combination. Paste the Google Doc URL into the `Get n8n Docs` node's file URL field.

### Step 4 — Set your n8n instance URL

In the `Workflow Link` node, the URL is currently set to:

```
https://n8n.srv874091.hstgr.cloud/workflow/{{ $json.id }}
```

Replace the domain with your own n8n instance URL.

### Step 5 — Link the sub-workflow

In the `Developer Tool` node, the `workflowId` points to the Builder sub-workflow. After importing, check that the ID matches the actual ID of the Builder workflow in your instance. Update it if needed.

---

## Files in This Repo

```
├── workflow.json       # n8n export containing both workflows (credentials redacted)
├── screenshot.png      # n8n canvas overview
└── README.md
```

### `.env.example`

```
OPENAI_API_KEY=
OPENROUTER_API_KEY=
N8N_API_KEY=
N8N_INSTANCE_URL=https://your-instance.example.com
GOOGLE_DRIVE_OAUTH_CLIENT_ID=
GOOGLE_DRIVE_OAUTH_CLIENT_SECRET=
```

---

## Limitations

- Output quality depends heavily on the documentation in your Google Drive file — richer docs produce better workflows
- Complex multi-step workflows with conditional logic may need minor manual adjustments after generation
- The Builder uses GPT-4o-mini; swapping to GPT-4o will improve JSON accuracy for intricate requests
- Workflows are created in an inactive state — you need to activate and add credentials manually before running them

---

## License

MIT — use freely, modify, redistribute. Credit appreciated but not required.

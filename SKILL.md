---
name: x402
description: >
  When to use: the user wants to make BSV-authenticated or paid API requests.
  This includes BRC-31 mutual authentication, BRC-29 micropayments, 402 Payment
  Required flows, or interacting with any BSV AI agent (image generation, video,
  transcription, Twitter search, file hosting). Also use when the user mentions
  x402, micropayments, or wants to discover/list available BSV agents.
  When NOT to use: the user is building a BSV server/agent, writing Rust/TS code
  for a Cloudflare Worker, or doing anything that does not involve making HTTP
  requests as a client to a BRC-31/BRC-29 service.
---

## x402 — BSV Micropayment Client

Make authenticated and paid API requests to BSV AI services. Discover agents,
authenticate with BRC-31, pay with BRC-29 micropayments — all from natural language.

### Prerequisites

- **MetaNet Client** running at `localhost:3321` (download from [getmetanet.com](https://getmetanet.com))
- **Python 3** with `requests` installed (`pip3 install requests`)

### Available Services

| Agent | What it does | Cost |
|:------|:-------------|:-----|
| **banana** | AI image generation (Google Nano Banana Pro) | ~$0.19/image |
| **veo** | AI video generation with audio (Google Veo 3.1 Fast) | ~$0.75-$1.50/clip |
| **whisper** | Speech-to-text transcription (Whisper Large v3 Turbo) | ~$0.0006/min |
| **x-research** | Twitter/X search, profiles, threads, trending | ~$0.005-$0.06/req |
| **nanostore** | File hosting with UHRP content addressing | ~$0.0004/MB/yr |

---

## Instructions

The helper script is at `~/.agents/skills/x402/scripts/brc31_helpers.py`.
All commands below use this path. If the skill is installed elsewhere, adjust accordingly.

### Step 1: Determine what the user wants

Parse the user's request into one of these actions:

- **List agents** — user wants to see what's available
- **Discover agent** — user wants details/pricing for a specific agent
- **Auth request** — user wants to call an endpoint that requires authentication but no payment
- **Paid request** — user wants to generate an image, video, transcribe audio, search Twitter, upload a file, etc.

If the request is ambiguous, ask the user to clarify which agent or action they want.

### Step 2: Run the appropriate command

#### List all agents
```bash
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py list
```

#### Discover an agent's capabilities and pricing
```bash
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py discover <agent-name>
```
Agent names: `banana`, `veo`, `whisper`, `x-research`, `nanostore`.

#### Get wallet identity
```bash
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py identity
```

#### Authenticated request (no payment)
```bash
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py auth <METHOD> <agent/path> [body-json]
```

#### Paid request (auth + automatic 402 payment)
```bash
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py pay <METHOD> <agent/path> [body-json]
```
This handles the full flow automatically:
1. Sends initial request -> server returns 402 with price
2. Creates payment transaction via MetaNet Client wallet
3. Retries with payment header -> server accepts, returns result
4. If response includes a refund, auto-processes it back to the wallet

### Step 3: Construct the request

When the user gives a natural-language request like "generate an image of a sunset":

1. Run `list` to find the matching agent (banana for images, veo for video, etc.)
2. Run `discover <agent>` to read the endpoint schemas and required fields
3. Build the JSON body from the schema and user's request
4. Run `pay POST <agent>/<endpoint> '<json-body>'`

**Agent name resolution:** Short names like `banana` or `nanostore` resolve automatically
via the 402agints.com registry. Full URLs (`https://...`) also work.

**Examples:**
```bash
# Generate an image
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py pay POST banana/generate '{"prompt":"a mountain sunset","aspect_ratio":"16:9"}'

# Search Twitter
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py pay POST x-research/search '{"query":"bitcoin scaling","max_results":20}'

# Transcribe audio (base64-encoded)
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py pay POST whisper/transcribe '{"audio":"<base64>","language":"en"}'

# Free authenticated endpoint
python3 ~/.agents/skills/x402/scripts/brc31_helpers.py auth POST banana/free
```

### Step 4: Present the result

- Parse the JSON response from the script
- For images/video: extract the URL and show it to the user
- For text results (transcription, search): format and display clearly
- For errors: show the error message and suggest fixes
- If a refund was processed, mention it to the user

### Error handling

- **MetaNet Client not running:** Tell the user to start MetaNet Client from `/Applications` or download from getmetanet.com
- **402 Payment Required with no auto-payment:** The script handles this automatically; if it fails, show the payment details from the 402 headers
- **Agent not found:** Run `list` to show available agents
- **Auth failure (401/403):** Run `python3 ~/.agents/skills/x402/scripts/brc31_helpers.py session --clear-all` then retry
- **requests not installed:** Run `pip3 install requests`

---

## Protocol Summary

- **[BRC-31](https://brc.dev/31):** Mutual HTTP authentication with identity keys and signed requests
- **[BRC-29](https://brc.dev/29):** HTTP micropayments via 402 Payment Required flow
- **[BRC-100](https://brc.dev/100):** MetaNet Client wallet interface (key derivation, signing, tx creation)

# AI Governance Pipeline — Setup Runbook

Follow these in order. Estimated time: 30–40 minutes the first time.

---

## Prerequisites

- Docker Desktop installed and running on your MacBook (Apple Silicon — fine, n8n image is multi-arch)
- Your OpenAI API key
- A Slack workspace where you can create an app (for the notification node)

---

## Step 1 — Run n8n in Docker

Open Terminal and run:

```bash
docker volume create n8n_data

docker run -d \
  --name n8n-governance \
  -p 5678:5678 \
  -e GENERIC_TIMEZONE="America/New_York" \
  -e TZ="America/New_York" \
  -e N8N_SECURE_COOKIE=false \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

Wait ~30 seconds, then open: **http://localhost:5678**

Create the local owner account when prompted (any email/password — it's local only).

Useful commands later:
```bash
docker logs -f n8n-governance     # watch logs while testing
docker stop n8n-governance        # stop
docker start n8n-governance       # restart (data persists in the volume)
```

---

## Step 2 — Import the workflow

1. In n8n, click the **three-dot menu (top right)** → **Import from File**.
2. Select `AI_Governance_Pipeline.json`.
3. The canvas loads with 16 nodes in two rows (main pipeline + dashboard).

You will see credential warnings on the three OpenAI nodes and the Slack node — that is expected; you wire them next.

---

## Step 3 — Add the OpenAI credential

1. Click the node **STAGE 1 - LLM Data Classification**.
2. In the **Credential for OpenAI API** dropdown → **Create New Credential**.
3. Paste your OpenAI API key. Save.
4. Open **STAGE 2 - LLM Risk Assessment** and **STAGE 3 - LLM Guidance Generation** and select the same credential from the dropdown (no need to re-enter the key).

Note: the workflow uses `gpt-4o-mini` — cheap and fast, costs roughly a fraction of a cent per submission. Fine for a demo. If your account doesn't have that model, change the `model` field on all three OpenAI nodes to one you do have (e.g. `gpt-4o` or `gpt-3.5-turbo`).

---

## Step 4 — Add the Slack credential

Fastest path is a Slack app with OAuth:

1. Go to https://api.slack.com/apps → **Create New App** → **From scratch**.
2. Name it "AI Governance Bot", pick your workspace.
3. **OAuth & Permissions** → under **Bot Token Scopes** add: `chat:write`, `channels:read`.
4. **Install to Workspace**, authorize, copy the **Bot User OAuth Token** (`xoxb-...`).
5. In your Slack client, create a channel named **#ai-governance** and invite the bot: type `/invite @AI Governance Bot` in that channel.
6. In n8n, open **Notify Approver (Slack)** node → create a Slack credential. Choose **Access Token** type and paste the `xoxb-` token. Save.

If Slack setup is taking too long on Sunday, see "Fallback" at the bottom — you can disable the Slack node and still demo everything.

---

## Step 5 — Activate the workflow

Toggle the workflow **Active** (top-right switch). This enables the production webhook URLs:

- Intake: `http://localhost:5678/webhook/ai-usecase-intake`
- Dashboard: `http://localhost:5678/webhook/ai-governance-dashboard`

(While editing/testing you can also use the **Test URL** with `/webhook-test/` after clicking "Listen for test event".)

---

## Step 6 — Send a test submission

In a new Terminal tab, run one of the sample payloads:

```bash
curl -s -X POST http://localhost:5678/webhook/ai-usecase-intake \
  -H "Content-Type: application/json" \
  -d @sample_low_risk.json | python3 -m json.tool
```

You should get back a JSON decision with `data_classification`, `overall_risk_tier`, `approval_route`, `nist_rmf_mapping`, `guidance`, and `kri`. A Slack message lands in #ai-governance.

Then try the high-risk one:
```bash
curl -s -X POST http://localhost:5678/webhook/ai-usecase-intake \
  -H "Content-Type: application/json" \
  -d @sample_high_risk.json | python3 -m json.tool
```

---

## Step 7 — Open the dashboard

In your browser: **http://localhost:5678/webhook/ai-governance-dashboard**

It renders a dark KRI dashboard with cards (use cases governed, pending approval, high-risk count, hours/$ saved), risk-tier and data-classification bar charts, and a recent-decisions table. Refresh after each submission to see it update.

---

## Fallback (if Slack or time runs out)

- **Disable the Slack node:** click **Notify Approver (Slack)** → press `D` (or right-click → Deactivate). The pipeline still runs end-to-end and the webhook still returns the full decision; you just lose the Slack ping. In the interview say: *"In production this routes to Slack and is extensible to ServiceNow; for the local demo I'm showing the governance decision in the API response and dashboard."*
- **Model unavailable:** swap the `model` value on the three OpenAI nodes.
- **Port 5678 busy:** change `-p 5678:5678` to `-p 5680:5678` and use port 5680 in URLs.

---

## Teardown (after the interview, optional)

```bash
docker stop n8n-governance && docker rm n8n-governance
docker volume rm n8n_data   # only if you want to wipe all data
```

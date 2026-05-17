# AI Governance Pipeline

An automated AI/automation governance intake-and-risk-assessment pipeline built in [n8n](https://n8n.io), aligned to the **NIST AI Risk Management Framework (AI RMF 1.0)** and financial-services expectations (FFIEC, third-party risk, model-risk routing).

A business user submits a proposed AI use case. The pipeline classifies the data, runs a structured risk assessment mapped to the four NIST AI RMF functions, decides an approval route, auto-generates acceptable-use guidance for the user, and logs the decision to a Key Risk Indicator (KRI) dashboard for senior-management reporting.

> Built as a hands-on demonstration of operationalizing AI governance — turning a slow, inconsistent, manual review into a fast, consistent, auditable, automated pipeline that itself follows the framework it enforces.

---

## What it does

| Stage | Function | NIST AI RMF / JD alignment |
|---|---|---|
| Intake (Webhook) | Accepts and validates a use-case submission | Identify automation opportunities |
| Stage 1 — Data Classification | Classifies data as Public / Internal / Confidential / Restricted; detects PII / NPI | Classify and validate data; evaluate AI suitability |
| Stage 2 — Risk Assessment | Scores data-leakage, model-bias, third-party, and regulatory risk; maps mitigations to GOVERN / MAP / MEASURE / MANAGE; assigns a risk tier | Conduct AI risk assessments; align to NIST AI RMF / FFIEC |
| Routing | LOW → auto-approve, MEDIUM → InfoSec review, HIGH → ISO sign-off | Prioritize by risk; senior-management oversight |
| Stage 3 — Guidance Generation | Generates acceptable-use, prohibited-use, and prompt-usage rules tailored to the use case | Educate users; develop prompt-usage guidelines |
| Logging + Notification | Writes a KRI record; notifies the approver via Slack | Develop dashboards / KRIs; periodic reporting |
| KRI Dashboard | Live HTML dashboard: volume, risk distribution, pending approvals, estimated savings | Senior-management reporting |

## Architecture

```
Webhook (intake)
   → Validate & Normalize
   → Valid?  ──no──> Reject
       │yes
   → Stage 1: LLM Data Classification   (HTTP → OpenAI)
   → Parse
   → Stage 2: LLM Risk Assessment       (HTTP → OpenAI, NIST AI RMF mapping)
   → Parse
   → Stage 3: LLM Guidance Generation   (HTTP → OpenAI)
   → Assemble Governance Record
   → Log to KRI Datastore
        ├─→ Slack notification (approver)
        └─→ Webhook response (decision JSON)

Webhook (dashboard, GET) → Build HTML → Serve KRI dashboard
```

LLM stages use the OpenAI Chat Completions API via authenticated HTTP requests (vendor-portable; no dependency on a version-locked plugin node).

## Tech stack

- **n8n** — workflow orchestration (self-hostable; relevant for data sovereignty in regulated environments)
- **OpenAI API** — classification, risk assessment, and guidance generation (`gpt-4o-mini`, `temperature: 0` on deterministic stages)
- **Slack** — approver notification
- Frameworks referenced: NIST AI RMF 1.0, NIST AI 600-1 (GenAI Profile), NIST CSF, FFIEC, Interagency Third-Party Risk guidance, SR 11-7

## Quick start

1. Run n8n (Docker):
   ```bash
   docker volume create n8n_data
   docker run -d --name n8n-governance -p 5678:5678 \
     -e GENERIC_TIMEZONE="America/New_York" -e TZ="America/New_York" \
     -e N8N_SECURE_COOKIE=false -v n8n_data:/home/node/.n8n \
     docker.n8n.io/n8nio/n8n
   ```
2. Open `http://localhost:5678`, create the local owner account.
3. Import `workflow/AI_Governance_Pipeline.json`.
4. Add credentials (see [docs/SETUP_RUNBOOK.md](docs/SETUP_RUNBOOK.md)):
   - OpenAI: Header Auth credential, value `Bearer YOUR_OPENAI_KEY`
   - Slack: Access Token (bot `xoxb-` token)
5. Publish the workflow.
6. Submit a sample:
   ```bash
   curl -s -X POST http://localhost:5678/webhook/ai-usecase-intake \
     -H "Content-Type: application/json" \
     -d @samples/sample_high_risk.json | python3 -m json.tool
   ```
7. View the dashboard: `http://localhost:5678/webhook/ai-governance-dashboard`

## Example: risk discrimination

The same pipeline produces opposite governance outcomes based on risk:

| Use case | Data class | Risk tier | Routing |
|---|---|---|---|
| Internal meeting-notes summarizer | INTERNAL | LOW | Auto-approve |
| Vendor questionnaire triage | CONFIDENTIAL | MEDIUM | InfoSec review |
| Customer loan pre-decision assistant | RESTRICTED | HIGH | ISO sign-off |

## Limitations & roadmap

This is a demonstration system, not production software. Known limitations and planned improvements:

- **Human-in-the-loop:** add a Slack approve/reject action that writes the decision back to the record.
- **Persistence:** replace the in-memory datastore with a database (Postgres) or the firm's GRC tool for an auditable, restart-safe inventory.
- **Self-governance:** add an evaluation harness that periodically re-tests the classifier against a labeled set to monitor its own drift — applying the NIST AI RMF MEASURE discipline to the governance tool itself.
- **Enterprise integration:** extend notification/routing to ServiceNow and IAM systems.

## Security note

No credentials are stored in the workflow JSON — API keys and tokens live in n8n's encrypted credential store and are referenced by ID only. Never commit real `sk-` or `xoxb-` values. See `.gitignore`.

## License

MIT — see [LICENSE](LICENSE).

# Interview Demo Script & Q&A Defense — AI Governance Pipeline

This is what you say and do when you present the project. Practice the 2-minute pitch out loud until it is smooth.

---

## The 30-second hook (lead with this)

> "I built a working AI governance pipeline in n8n that automates the exact job this role describes. A business user submits a proposed AI use case, and the pipeline classifies the data, runs a risk assessment mapped to the NIST AI Risk Management Framework, decides an approval route, auto-generates acceptable-use guidance for the user, and logs everything to a KRI dashboard for senior management. It's not a slideware concept — it runs locally and I can walk you through a live submission."

That sentence alone covers ~11 of the JD bullets. Then offer the demo.

---

## The 2-minute live demo flow

1. **Show the canvas.** "One webhook entry point, three governance stages chained, output to Slack and a dashboard. I deliberately built it modular so each stage maps to a different cluster of your job description."

2. **Submit a LOW-risk case** (meeting notes summarizer). Show the JSON response: classified INTERNAL, tier LOW, auto-approved, guidance generated. "Low-risk, high-repeatability, no decision impact — the system green-lights it automatically. That's the high-impact, low-risk quadrant your JD asks me to prioritize."

3. **Submit a HIGH-risk case** (loan pre-decision). Show: classified RESTRICTED, contains PII/NPI true, tier HIGH, routed to ISO_SIGNOFF, regulatory_exposure HIGH. "Same pipeline, completely different governance outcome. Customer financial data, a regulated lending decision — this is SR 11-7 model-risk territory, so it does not auto-approve. It routes to the ISO and flags the controls."

4. **Open the dashboard.** Show cards, risk-tier distribution, recent decisions table. "This is the senior-management view — adoption volume, pending approvals, high-risk count, and estimated time and cost savings as KRIs."

5. **Land it:** "So in one workflow: identify and prioritize use cases, evaluate AI suitability, run the risk assessment, classify data, generate user guidance, and report KRIs — which is most of the job description, automated."

---

## How each JD requirement maps (have this ready)

| JD requirement | Where in the project |
|---|---|
| Identify & prioritize automation opportunities | Intake webhook + risk-tier routing (low-risk auto-approves first) |
| Evaluate processes for AI suitability | Stage 1 classification + Stage 2 scoring (data sensitivity, repeatability, control impact) |
| Build/deploy automation workflows (n8n, LLM orchestration) | The pipeline itself — n8n orchestrating chained LLM calls |
| Integrate AI with enterprise systems | Webhook in, Slack out; designed to extend to ServiceNow/IAM |
| Align to NIST AI RMF / CSF / FFIEC | Stage 2 explicitly maps to Govern/Map/Measure/Manage |
| Conduct risk assessments (leakage, bias, third-party) | Stage 2 scores all three plus regulatory exposure |
| Classify and validate data (PII/NPI/confidential/restricted) | Stage 1 classification node |
| Encryption, masking, access controls | Credential vaulting + design notes (see "controls" answer below) |
| Educate users / prompt-usage guidelines | Stage 3 generates acceptable-use + prompt rules |
| Develop dashboards / KRIs | Stage 3 logging + HTML dashboard |
| Periodic senior-management reporting | Dashboard + Slack routing |

---

## Q&A Defense — the questions they will ask

**Q: "Isn't using an LLM to assess AI risk itself a risk? What if it's wrong?"**
> "Yes, and that's the honest answer. The LLM is a triage accelerator, not the decision authority. Notice the routing — anything that isn't low-risk goes to a human (InfoSec or the ISO). The LLM never auto-approves a medium or high case. I also set temperature to zero on the classification and risk stages for determinism, and there's a conservative fallback: if the model output can't be parsed, it defaults to the highest tier and forces human review. Fail safe, not fail open."

**Q: "How would you handle the data the pipeline itself processes? The submissions contain sensitive descriptions."**
> "Three controls. One, credentials — the OpenAI and Slack keys live in n8n's encrypted credential store, never in the workflow body or in prompts. Two, data minimization — the intake asks for a description of the data, not the data itself; you never paste a real customer record in. Three, in production I'd self-host n8n inside the firm's network, use a private/enterprise model endpoint with a no-training contractual term so submissions aren't used for model training, and forward execution logs to Splunk for audit traceability."

**Q: "Why n8n and not Power Automate?"**
> "n8n is open-source and self-hostable, which matters in financial services for data sovereignty — the orchestration layer can run entirely inside the firm's environment. Power Automate is the right call when the use case lives in the Microsoft ecosystem and you want tight Entra ID and Purview integration. I'd actually expect to use both in this role depending on the use case; this project shows the n8n pattern but the governance logic is platform-agnostic."

**Q: "What's the NIST AI RMF mapping actually doing here?"**
> "Stage 2 forces the model to produce a control statement for each of the four functions for the specific use case. Govern — is it in the inventory and who's accountable. Map — context, data flow, impact. Measure — what metrics and testing apply. Manage — residual risk and monitoring. It operationalizes the framework instead of leaving it as a PDF, which is the gap I see most often."

**Q: "How do the KRIs work — are those real numbers?"**
> "They're illustrative estimates derived from stated volume and risk tier, and I label them as illustrative on the dashboard. In production the time-and-cost-saved figures come from actual workflow telemetry — execution counts and measured handling-time deltas against a manual baseline. The risk KRIs — pending high-risk approvals, restricted-data use cases, shadow-AI detections — those would be sourced from the inventory and DLP, which are real signals."

**Q: "What would you improve / what are the limitations?"**
> "Three things. One, add a human-in-the-loop approval node so the ISO can approve or reject directly from Slack and that decision writes back to the record. Two, replace the in-memory datastore with a real database — Postgres or the firm's GRC tool — so the inventory survives restarts and is auditable. Three, add an evaluation harness that periodically re-tests the classifier against a labeled set so I can MEASURE its drift — which, fittingly, is the same NIST AI RMF discipline I'd apply to any other AI system. I'd govern my own tool the way I govern everyone else's."

That last line is a strong close — use it.

---

## If the demo breaks live (stay calm, this is the script)

> "Looks like a local environment hiccup — let me show you the architecture and a captured run instead, the logic is the same." Then walk the canvas and a saved JSON response. Interviewers care far more about your reasoning than a flawless localhost demo. Composure under a broken demo is itself a signal in your favor.

Have one successful run's JSON response saved to a file beforehand as a backup screenshot/text.

---

## One-sentence close for the whole interview

> "I didn't just study your job description — I built the thing it's asking for, and I governed it by the same framework I'd use on the job."

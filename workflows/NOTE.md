# About this folder

SolarLeadOS is **8 connected n8n workflows**, running live end to end.

**3 are published here.** They were chosen because together they cover every
technique used across the full system — webhook and scheduled triggers, LLM
integration, defensive parsing, multi-way routing, retry policy, and idempotent
state tracking.

The remaining 5 are the sequencing workflows — drip timing, message angles, and
follow-up copy. They reuse the same patterns, so they are not published here to
keep this repo focused.

| Published | Why it's here |
|---|---|
| `01-lead-intake-ai-scoring` | Webhook trigger, dedup-before-insert, Gemini scoring, defensive Code-node parsing, score-based routing, calendar seeding |
| `03-hot-lead-accountability` | Cron in a time window, two-stage escalation (25 min → 45 min), idempotency flags |
| `04-reply-detection-intent-routing` | Second webhook, inbound validation, Gemini intent classification, 4-way routing, dual-audience sends |

**Not published:** 02 Daily Drip Sequencer · 05 Post-Proposal Drip ·
06 Site-Visit Reminders · 07 Dormant Reactivation · 08 Referral & Review Engine.
I'm happy to walk through any of them live.

---

## Redactions

These files import cleanly into n8n. The following were replaced with placeholders:

- Google Sheet ID → `YOUR_GOOGLE_SHEET_ID`
- WAHA gateway URL → `https://YOUR-WAHA-GATEWAY.example.com`
- Phone numbers → `91XXXXXXXXXX`
- Credential IDs → `REDACTED`

No API keys were ever present — n8n stores credentials outside the workflow export.

## To run these

1. Import the JSON into n8n
2. Reconnect the Google Sheets and Gemini credentials
3. Point the WAHA nodes at your own gateway
4. Create a Sheet with the columns referenced in the nodes

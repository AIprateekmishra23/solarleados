# SolarLeadOS

An AI lead-management system for rooftop-solar installers in North India.
Eight connected n8n workflows, one Google Sheets CRM, WhatsApp as the only
customer channel. Built solo, running live end to end.

---

## The problem

A rooftop-solar installer in Punjab gets 30–80 enquiries a month from ads and
walk-ins. The sales team is one to four people, usually including the owner, and
they are on site most of the day.

What happens in practice:

- A lead messages at 2pm. Someone replies at 9pm, or the next day.
- Nobody knows which of the 60 open leads is worth calling first.
- Follow-up stops after the second attempt because there is no system telling
  anyone to do a third.
- A quote goes out and then nothing happens until the customer buys elsewhere.

None of this is a motivation problem. It is a memory problem. So I built the
memory.

## What the system does

```
new lead
   ↓  webhook
normalise → dedup check → Gemini scores it → write to CRM
   ↓
instant WhatsApp to the lead   +   alert to the owner with a call opener
   ↓
if the owner hasn't called in 25 min → reminder.  45 min → escalation.
   ↓
Day 1 · 3 · 7 · 14 · 30 follow-ups, routed by AI segment
   ↓
customer replies → Gemini classifies intent → routed to the right response
   ↓
proposal sent → chase sequence → site visit reminders → no-show recovery
   ↓
won → review request → referral offer
   ↓
lost or cold → 90-day reactivation with rotating angles
```

Every stage writes its state back to the same sheet. Nothing runs twice.

---

## What's in this repo

SolarLeadOS is **8 workflows. Three are published here.**

Together these three cover every technique used across the whole system —
both trigger types, both LLM use-cases, all routing patterns, and the error and
state handling that repeats everywhere else.

| File | What it demonstrates |
|---|---|
| [`01-lead-intake-ai-scoring`](workflows/01-lead-intake-ai-scoring.json) | Webhook trigger · dedup-before-insert · Gemini scoring · defensive parsing · score-based routing · follow-up calendar seeding |
| [`03-hot-lead-accountability`](workflows/03-hot-lead-accountability.json) | Cron inside a business-hours window · two-stage escalation · idempotency flags |
| [`04-reply-detection-intent-routing`](workflows/04-reply-detection-intent-routing.json) | Inbound webhook · payload validation · Gemini intent classification · 4-way routing · dual-audience sends |

**Not published:** the drip sequencer and the four lifecycle workflows
(post-proposal, site visits, dormant reactivation, referrals). They reuse the
same patterns shown here. All eight are shown running in the demo video and I'm
happy to walk through any of them on a call.

---

## Three decisions worth explaining

### 1. A Code node sits after every AI call

Gemini returns good JSON most of the time. Most of the time is not good enough
when the output decides whether a lead gets called today.

So every LLM call is followed by a plain-JavaScript Code node that:

- strips markdown fences the model sometimes wraps around JSON
- parses inside a try/catch
- clamps the score to 0–100
- **falls back to a safe Warm default** rather than throwing

A model failure degrades the lead to Warm. It never loses the lead. The fallback
has fired a handful of times in testing and nothing downstream noticed — which is
the point.

### 2. Every send is followed by a state write

`Mark Day1 Sent`. `Stamp Prop_D3_Sent`. `Stamp Review_Sent`.

Scheduled workflows re-run. Executions get retried. Without a written flag, a
customer gets the same message twice and the installer looks unprofessional.
Every branch in every workflow ends with a Sheets update, and every read filters
on those flags before sending. The workflows are safe to re-run at any time.

### 3. The intake workflow seeds the whole system

`Append Lead Row` writes about 25 columns — but the important ones are
`Day1_Due` through `Day30_Due` and `Call_Status`. Those are calculated at intake,
once.

Every other workflow is then a simple daily job: read the sheet, filter on a
due-date, send, stamp. No workflow needs to know what any other workflow did.
The sheet is the coordination layer. That is why eight workflows don't turn into
eight tangled dependencies.

---

## Reliability

Every node that touches an external service is configured for failure:

- `retryOnFail` with `maxTries: 3` and a 3-second backoff on all Sheets reads
  and writes
- `onError: continueRegularOutput` on WhatsApp sends, so one failed message
  never halts a batch of forty
- Filters run before sends, not after, so unsubscribed and already-contacted
  leads are dropped early
- All scheduling is pinned to `Asia/Kolkata` — dates are parsed with an explicit
  timezone rather than relying on the server's

## Tech stack

**n8n** (self-hosted) · **Google Gemini** `gemini-2.5-flash` for scoring and
intent classification · **WhatsApp** via self-hosted WAHA in Docker (demo) and
UltraMsg (cloud alternative) · **Google Sheets** as CRM with Apps Script ·
JavaScript in Code nodes · webhooks, REST, JSON

**On Google Sheets as the database:** a deliberate fit for the use case —
a small installer doing a few hundred leads a month, whose owner wants to open the
CRM on his phone between site visits. At higher volume I'd move to Postgres or
Airtable; every workflow reads through a single node, so the data layer swaps out
in one place.

## Running these locally

1. Import a workflow JSON into n8n
2. Reconnect the Google Sheets and Gemini credentials
3. Point the WAHA HTTP nodes at your own gateway
4. Create a sheet with the columns referenced in the Append node

Sheet IDs, gateway URLs, phone numbers and credential IDs are replaced with
placeholders. No API keys were ever in these files — n8n stores credentials
outside the workflow export.

---

## About

**Prateek Mishra** — I build AI automation systems.

I came to this without a software-engineering background. I write JavaScript
inside n8n Code nodes and Google Apps Script, and I work with REST APIs,
webhooks and JSON every day debugging my own flows. What I'm good at is taking a
real-world problem end to end — understanding the workflow, designing the
system, and building it. I've been building automations for about a year — Make.com
from July 2025, then n8n. SolarLeadOS is an independent project I designed and
built solo — architecture, the workflows, the CRM, and the AI logic.

Open to AI automation roles. Immediate joiner.

📧 thisisprateek22@gmail.com

---
name: domain-warmup-orchestrator
description: Warm up secondary sending domains to full sending capacity with healthy inbox placement before real prospects receive email.
---

# Domain Warm-up Orchestrator

## Purpose
Ensure secondary sending domains reach full sending capacity with inbox placement rates >90% before any real prospect receives an email. A failed warm-up means cold emails land in spam — making the entire outreach investment worthless.

**Goal:** All inboxes achieve >90% inbox placement on mail-tester.com and lemwarm's warm-up score >80 before live campaign launch.

**Does not:** Write email copy (Skill 5), configure DNS records (Skill 3), or manage lemlist sequences directly.

---

## Inputs Required
- `domains`: List of secondary domains with inbox addresses
- `start_date`: When warm-up begins
- `target_daily_volume`: Final target emails/day per inbox
- `lemwarm_enabled`: Confirmed true/false
- `campaign_launch_date`: Target date for first real outreach

---

## Warm-up Principles (hardcoded)

1. **Never rush warm-up.** 4 weeks minimum per inbox. 6 weeks preferred.
2. **Gradually increase volume.** Never jump more than 20% day-over-day.
3. **Maintain high engagement.** lemwarm's warm-up network handles this — ensure it is enabled.
4. **Monitor daily.** A single spam spike can undo two weeks of progress.
5. **Never mix warm-up and live outreach.** Wait until warm-up is complete.
6. **One domain failure does not stop others.** Isolate issues per domain.

---

## Step 1 — Generate Warm-up Schedule

For each inbox, generate a day-by-day sending ramp:

**Standard 6-week ramp to 50 emails/day:**

| Week | Daily Sends | Warm-up Emails | Notes |
|---|---|---|---|
| Week 1 | 5 | 5 (all warm-up) | lemwarm only, zero live sends |
| Week 2 | 10 | 10 | Still warm-up only |
| Week 3 | 20 | 20 | Begin monitoring spam scores |
| Week 4 | 30 | 25 warm-up + 5 live | First live emails if score >80 |
| Week 5 | 40 | 20 warm-up + 20 live | Increase live if week 4 healthy |
| Week 6 | 50 | 15 warm-up + 35 live | Full capacity |

**Accelerated 4-week ramp** (only if campaign launch date requires it):

| Week | Daily Sends | Notes |
|---|---|---|
| Week 1 | 10 | Warm-up only |
| Week 2 | 20 | Warm-up only |
| Week 3 | 35 | 25 warm-up + 10 live if score >75 |
| Week 4 | 50 | 15 warm-up + 35 live if healthy |

Flag if campaign launch date is < 4 weeks from domain setup — this is a risk that must be acknowledged.

---

## Step 2 — lemlist Configuration Settings

Provide exact settings to enter in lemlist per inbox:

```
LEMLIST INBOX SETTINGS
Sending limit: [per schedule above]
Minimum time between emails: 4 minutes
Maximum time between emails: 9 minutes
Send window: 08:00–18:00 CET (Mon–Fri only)
Warm-up enabled: YES
Warm-up daily ramp: +2 emails/day
Warm-up reply rate target: 30%
Warm-up mark as important: YES
Custom tracking domain: [configured CNAME from Skill 3]
Open tracking: YES
Click tracking: YES (after week 3 only — reduces deliverability in early warm-up)
```

---

## Step 3 — Weekly Health Check Protocol

Every Monday, run these checks and output a health report:

| Metric | Tool | Green | Yellow | Red |
|---|---|---|---|---|
| Inbox placement rate | lemwarm score | >80 | 60–80 | <60 |
| Spam score | Mail-tester.com | ≥9/10 | 7–9 | <7 |
| Bounce rate | lemlist analytics | <2% | 2–5% | >5% |
| Spam complaint rate | lemlist analytics | <0.1% | 0.1–0.3% | >0.3% |
| DMARC reports | Google Postmaster / rua inbox | 0 failures | <5 failures | >5 failures |
| Blacklist status | MXToolbox | 0 lists | — | Any listing |

**Go/No-go rule:** All metrics must be Green before advancing to the next phase. Any Red metric pauses that inbox immediately.

---

## Step 4 — Issue Triage

If a health check fails, diagnose and prescribe action:

**Bounce rate >5%:**
→ Pause inbox. Audit list quality — likely invalid emails in sequence. Re-verify list with NeverBounce or ZeroBounce before resuming.

**Spam complaint rate >0.3%:**
→ Pause inbox immediately. Review copy for spam trigger words. Check if suppression list is active. Resume only after copy revision.

**Inbox placement <60%:**
→ Reduce daily volume by 50%. Extend warm-up by 1 week. Check SPF/DKIM/DMARC still valid.

**Blacklisted:**
→ Stop all sending from that domain. Submit removal request to blacklist operator. Do not resume for minimum 2 weeks. If re-listed within 30 days, retire the domain.

**DMARC failures:**
→ Audit for spoofing. Ensure all sending is going through configured mail server only. Escalate DMARC policy from `none` to `quarantine`.

---

## Step 5 — Launch Readiness Report

Before first live outreach email sends, generate a go/no-go report per inbox:

```
WARM-UP LAUNCH READINESS REPORT
Generated: [date]
Campaign target start: [date]

INBOX: alex@tryexample.eu
Warm-up duration: 6 weeks
Current daily volume: 50
lemwarm score: 84 ✓
Mail-tester score: 9.5/10 ✓
Bounce rate (last 7 days): 0.8% ✓
Spam complaint rate: 0.0% ✓
Blacklist status: Clean ✓
DMARC policy: quarantine ✓

STATUS: GO ✓ — Ready for live outreach

INBOX: alex@getexample.eu
[...]
STATUS: NO-GO ✗ — Inbox placement 58%. Extend warm-up 1 week.

OVERALL CAMPAIGN STATUS: HOLD — 1 inbox not ready. Launch with tryexample.eu only.
```

---

## Decision Logic
- If campaign launch date is in <4 weeks → use accelerated schedule AND flag the risk explicitly
- If >3 inboxes are ready but 1 is not → launch with ready inboxes, note reduced capacity
- If bounce rate spikes after switching from warm-up to live → immediately verify list quality, not domain setup
- Never increase daily volume two weeks in a row without a health check in between
- If lemwarm score drops week-over-week for 2 consecutive weeks → pause and diagnose before continuing
- Warm-up emails must be realistic business-context content — lemwarm's network handles this automatically
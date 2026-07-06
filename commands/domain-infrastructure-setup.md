---
name: domain-infrastructure-setup
description: Set up and configure secondary sending domains (SPF/DKIM/DMARC) for cold outreach so your primary domain's reputation is protected.
---

# Domain Infrastructure Setup

## Purpose
Set up secondary domains for cold outreach so that deliverability is protected, primary domain reputation is never at risk, and Instantly sequences can begin warm-up on day one.

**Goal:** All secondary domains pass MXToolbox validation with zero errors before warm-up starts. Primary domain is never used for cold outreach.

**Does not:** Purchase domains (provides the shortlist and registrar instructions), manage warm-up (that is Skill 4), or write email copy.

---

## Inputs Required
- `primary_domain`: your company's main domain (e.g. example.com)
- `sending_volume_target`: Estimated monthly outreach emails
- `team_senders`: Number of team members sending (each needs their own inbox)
- `instantly_workspace`: Confirmed (Instantly is the sending tool)

---

## Step 1 — Calculate Domain & Inbox Requirements

**Formula:**
- Safe sending limit per inbox: 30–40 emails/day during warm-up, scaling to 50/day at full capacity
- Inboxes per domain: max 3 (to protect domain reputation)
- Domains needed = ceil(monthly_volume / (50 × 30 × 3))

**Example:** 3,000 emails/month → 3,000 / 4,500 = 0.67 → minimum 2 domains, recommend 3 for redundancy

Always add 1 extra domain as a reserve.

---

## Step 2 — Domain Name Recommendations

Generate 8–10 domain name options based on primary domain. Follow these rules:

**Naming patterns (in priority order):**
1. `try[brand].com/eu` — e.g. tryexample.eu
2. `get[brand].com/eu` — e.g. getexample.eu
3. `[brand]hq.com` — e.g. examplehq.com
4. `join[brand].com` — e.g. joinexample.eu
5. `[brand]labs.com` — e.g. examplelabs.com (if primary is .eu)
6. `[brand]team.eu` — e.g. exampleteam.eu

**Rules:**
- Prefer .eu, .com, .co — avoid .xyz, .io, .info (low trust, higher spam correlation)
- Never use hyphens
- Max 15 characters
- Must not resemble phishing patterns (no "secure-", "official-", "verify-")
- Check availability at Namecheap or Cloudflare Registrar

**Registrar recommendation:** Cloudflare Registrar (at-cost pricing, built-in DNSSEC, easy DNS management)

---

## Step 3 — DNS Configuration Specs

For each secondary domain, output the exact DNS records to add:

### MX Records (Google Workspace or Microsoft 365)

**Google Workspace:**
```
Type: MX
Host: @
Value: ASPMX.L.GOOGLE.COM
Priority: 1

Type: MX
Host: @
Value: ALT1.ASPMX.L.GOOGLE.COM
Priority: 5

Type: MX
Host: @
Value: ALT2.ASPMX.L.GOOGLE.COM
Priority: 5
```

### SPF Record
```
Type: TXT
Host: @
Value: v=spf1 include:_spf.google.com ~all
TTL: 3600
```
(Replace google with outlook if using Microsoft 365)

### DKIM Record
```
Type: TXT
Host: google._domainkey (or selector1._domainkey for M365)
Value: [Generated from Google Workspace Admin or M365 Admin — provide step-by-step instructions]
TTL: 3600
```

**DKIM setup steps (Google Workspace):**
1. Go to Google Admin > Apps > Google Workspace > Gmail > Authenticate Email
2. Select domain, generate new record
3. Copy the TXT value exactly — it will look like: `v=DKIM1; k=rsa; p=MIGfMA0G...`
4. Add as TXT record in Cloudflare DNS

### DMARC Record
```
Type: TXT
Host: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@[primary-domain]; pct=100; adkim=s; aspf=s
TTL: 3600
```

**DMARC policy logic:**
- Start with `p=none` for the first 2 weeks to monitor without blocking
- Move to `p=quarantine` after confirming no legitimate email is being flagged
- Never use `p=reject` until 60+ days of clean DMARC reports

### Custom Tracking Domain (for Instantly)
```
Type: CNAME
Host: track (or em, or click)
Value: [provided by Instantly in account settings]
TTL: 3600
```

---

## Step 4 — Inbox Setup

For each domain, create inboxes following this naming convention:

**Pattern:** `[firstname]@[secondarydomain]`

Examples:
- alex@tryexample.eu
- alex@getexample.eu

**Rules:**
- Use real first names only — never info@, hello@, team@
- Each inbox needs a profile photo, full name, and email signature set up before warm-up
- Signature must include: name, title (e.g. "Founder, Your Company"), website link, no unsubscribe link (handled by Instantly)

---

## Step 5 — Validation Checklist

Run each domain through these tools before starting warm-up. Output pass/fail per check:

| Check | Tool | Pass Condition |
|---|---|---|
| SPF record exists | MXToolbox SPF | "SPF record found, no errors" |
| DKIM record exists | MXToolbox DKIM | Valid key returned |
| DMARC record exists | MXToolbox DMARC | Policy found |
| MX records correct | MXToolbox MX | Points to correct mail server |
| Blacklist check | MXToolbox Blacklist | 0 blacklists |
| SMTP test | MXToolbox SMTP | All checks green |
| Spam score | Mail-tester.com | Score ≥ 9/10 |

**Do not begin warm-up if any check fails.**

---

## Output Format

```
DOMAIN INFRASTRUCTURE REPORT
Primary domain: [domain]
Generated: [date]

RECOMMENDED SECONDARY DOMAINS
1. tryexample.eu — available ✓
2. getexample.eu — available ✓
3. examplehq.com — available ✓
[...]

DOMAIN & INBOX PLAN
Domain 1: tryexample.eu
  Inbox 1: alex@tryexample.eu
  Inbox 2: [name]@tryexample.eu
  Inbox 3: [name]@tryexample.eu

DNS RECORDS — tryexample.eu
[Full record table]

VALIDATION STATUS
SPF: PASS
DKIM: PASS
DMARC: PASS (currently p=none — escalate to p=quarantine on [date])
MX: PASS
Blacklist: PASS (0/100)
Mail-tester: 9.5/10

STATUS: READY FOR WARM-UP ✓
```

---

## Decision Logic
- If sending volume < 1,000/month → 1 domain, 2 inboxes is sufficient
- If sending volume 1,000–5,000/month → 2–3 domains, 2–3 inboxes each
- If sending volume > 5,000/month → 4+ domains, rotate carefully, flag for review
- Always recommend Google Workspace over Microsoft 365 for cold outreach — better deliverability reputation
- If primary domain is .eu, prioritise .eu secondary domains for brand consistency
- Never reuse a domain that has previously been used for bulk sending without a full reputation audit first
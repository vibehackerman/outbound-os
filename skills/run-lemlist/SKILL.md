---
name: run-lemlist
description: Configure, launch, and monitor outbound email campaigns in lemlist. Use this skill whenever the user mentions running a campaign, launching emails, setting up lemlist, configuring sequences, uploading to lemlist, campaign setup, sending cold emails, monitoring deliverability, checking campaign performance, adjusting send volume, pausing campaigns, or managing outbound execution. Also triggers on "launch this campaign", "upload to lemlist", "set up the sequence in lemlist", "launch leads in lemlist", "configure sending for [segment]", "check campaign health", "monitor deliverability", "adjust send volume", or any request to move from generated emails to actual sending via lemlist.
---

# Run lemlist

The final execution step. Take generated emails and prospect data, configure lemlist campaigns, launch sequences, and monitor performance. This is where emails become sent messages.

## Why This Matters

Brilliant emails sitting in a CSV don't book meetings. This skill bridges the gap between email generation and actual outbound execution. It handles the operational details that determine whether emails land in inboxes or spam folders.

## Prerequisites

1. **`company-context.md`**: For campaign naming, sender persona reference.
2. **Generated emails**: Output from `/email-generation` (markdown files + CSV export).
3. **Tiered prospect list**: Output from `/tiering-segmentation` with tier assignments and outreach strategies.
4. **Domain infrastructure**: Sending domains configured and warmed via **lemwarm** (lemlist's built-in warmup product). See the **domain-infrastructure-setup** and **domain-warmup-orchestrator** commands if not ready — lemwarm should be running at least ~2 weeks before real sends begin.
5. **lemlist workspace**: A lemlist account with connected sender mailboxes and an API key available as `LEMLIST_API_KEY`.

If emails haven't been generated yet, tell the user to run `/email-generation` first. If domains aren't warmed, flag this as a blocker.

## Step 1: Pre-Launch Checklist

Before touching lemlist, validate readiness:

```markdown
### Pre-Launch Checklist: [Campaign Name]

#### Infrastructure
- [ ] Sending domains configured (SPF, DKIM, DMARC passing)
- [ ] Domains warmed via lemwarm to target volume (minimum 2 weeks warm-up)
- [ ] Mailboxes connected to lemlist
- [ ] Custom tracking domain set up (not lemlist default)

#### Data
- [ ] Prospect list deduplicated against CRM and previous campaigns
- [ ] Email addresses verified (bounce rate target: <3%)
- [ ] Tier assignments complete
- [ ] Emails generated and approved for all tiers

#### Content
- [ ] Subject lines within 5-word limit
- [ ] Email bodies within framework word counts
- [ ] No banned phrases present
- [ ] Personalization variables mapped correctly
- [ ] Unsubscribe link / opt-out mechanism included

#### Compliance
- [ ] Sending complies with CAN-SPAM / GDPR requirements
- [ ] Physical address included in email footer
- [ ] Opt-out mechanism functional
- [ ] Data processing basis documented (legitimate interest or consent)
```

Surface any failures immediately. Don't proceed with a failed checklist.

## Step 2: Campaign Architecture

### Campaign Structure (Per Tier)

Map each tier to a separate lemlist campaign:

| Tier | Campaign Name Pattern | Daily Volume | Sequence Steps | Sending Window |
|------|----------------------|-------------|----------------|----------------|
| Tier 1 | `[Segment]-T1-[Date]` | 10-25/day per mailbox | 5-7 | 8am-11am recipient TZ |
| Tier 2 | `[Segment]-T2-[Date]` | 25-50/day per mailbox | 4-5 | 8am-12pm recipient TZ |
| Tier 3 | `[Segment]-T3-[Date]` | 50-75/day per mailbox | 3-4 | 7am-1pm recipient TZ |

Each tier's sending window is enforced by a lemlist **schedule** associated with that tier's campaign (see Step 3).

### Mailbox Rotation

Distribute sending across mailboxes to protect deliverability:

```markdown
### Mailbox Assignment: [Campaign]
**Total daily volume**: [N emails]
**Mailboxes available**: [N]
**Per-mailbox limit**: [N/day]

| Mailbox | Domain | Daily Limit | Campaign Assignment |
|---------|--------|-------------|-------------------|
| sender1@domain1.com | domain1.com | 30 | T1, T2 |
| sender2@domain2.com | domain2.com | 30 | T2, T3 |
| ... | ... | ... | ... |

**Rotation strategy**: Round-robin across mailboxes per campaign
**Cool-down**: 60-90 seconds between sends per mailbox
```

### Sequence Timing

Define delays between sequence steps:

#### Tier 1 Sequence (5-7 steps, 21 days)
| Step | Type | Delay After Previous | Content |
|------|------|---------------------|---------|
| 1 | Initial email | - | Framework A/C personalized email |
| 2 | Follow-up | 3 days | Value-add or different angle |
| 3 | Bump | 3 days | Short "did you see my last note" |
| 4 | New thread | 5 days | Fresh subject, different framework |
| 5 | Break-up | 4 days | Final touch, permission-based close |
| 6-7 | Optional | 3-5 days | LinkedIn touchpoint reference |

#### Tier 2 Sequence (4-5 steps, 14 days)
| Step | Type | Delay After Previous | Content |
|------|------|---------------------|---------|
| 1 | Initial email | - | Framework A/B semi-personalized |
| 2 | Follow-up | 3 days | Proof point or case study angle |
| 3 | Bump | 3 days | Short follow-up |
| 4 | Break-up | 5 days | Final touch |

#### Tier 3 Sequence (3-4 steps, 10 days)
| Step | Type | Delay After Previous | Content |
|------|------|---------------------|---------|
| 1 | Initial email | - | Framework B segment-personalized |
| 2 | Follow-up | 3 days | Different angle, same segment framing |
| 3 | Break-up | 4 days | Final touch |

## Step 3: Configure the lemlist Campaign

lemlist's data model: a **campaign** holds a multi-step **sequence**; **leads** are enrolled into a campaign; a **schedule** defines the sending window and is associated with a campaign; connected **email accounts** (mailboxes) do the sending; **lemwarm** is lemlist's built-in warmup product for those mailboxes.

### Per-tier setup sequence

1. **Create or select the campaign** for this tier (lemlist's Create Campaign operation), named per the pattern in Step 2.
2. **Build the multi-step sequence** inside the campaign, matching the tier's step count and delays from Step 2.
3. **Create and associate a sending schedule** with the campaign so sends only happen inside the tier's recipient-timezone window.
4. **Enroll the tiered leads into the campaign.** This is the Create-Lead-in-Campaign operation:
   - `POST /api/campaigns/{CAMPAIGN_ID}/leads/`
   - Body: `email` (required), plus optional `firstName`, `lastName`, `companyName`, `jobTitle`, `phone`, `linkedinUrl`, `timezone`
   - Map the pipeline's personalization fields (subject/body per step, tier label) into the lead payload alongside these standard fields; consult the lemlist API reference for how custom/merge variables are attached per lead if the mapping isn't a 1:1 fit.
5. **Start the campaign** (and, per-lead, **Launch Lead** for any leads added after the campaign is already running).

### Import via CSV (bulk fallback)

For large tiers, lemlist also supports CSV-based lead import through its dashboard, or bulk enrollment via repeated calls to the Create-Lead-in-Campaign endpoint. Generate the export CSV with columns matching the standard lead fields above plus the per-step subject/body content, then either import through the lemlist UI or script the API calls. Adapt the exact script to the pipeline's current column names.

### lemlist MCP server (no-code alternative)

lemlist also ships an MCP server (OAuth-based, lemlist advertises "set up in 30 seconds") that connects lemlist directly to Claude and other AI assistants — a natural fit here since this is a Claude Code plugin. Where available, prefer driving campaign/lead operations through the MCP server's tools over hand-rolled API calls; fall back to the raw API (below) when the MCP server isn't connected.

### API access

lemlist's API base is `https://api.lemlist.com/api`, authenticated with HTTP Basic auth — empty username, API key as the password:

```bash
curl -u ":$LEMLIST_API_KEY" https://api.lemlist.com/api/campaigns
```

Provide your own key through the `LEMLIST_API_KEY` environment variable — never hardcode a key in this file or in campaign configs. lemlist enforces rate limits on the API; consult lemlist's Rate Limits guide and add backoff/throttling to any bulk import script rather than assuming an unbounded call rate.

Key operations this step uses (see lemlist's API reference for full request/response shapes):
- **Campaigns**: Create, Update, Duplicate, Start Campaign, Pause Campaign, Get Campaign, Get Many Campaigns, Get Campaign Stats/Reports, Export Campaign Leads.
- **Leads**: Create Lead in Campaign, Launch Lead, Pause Lead, Resume Paused Lead, Update Lead, Import Leads from CRM, Get Campaign Leads, Delete/Unsubscribe Lead, Mark Lead Interested/Not Interested.
- **Schedules**: Create Schedule, Associate Schedule with Campaign, Get Campaign Schedules.
- **Lemwarm**: Start Lemwarm, Pause Lemwarm, Get/Update Lemwarm Settings.
- **Email Accounts**: Connect, Test, Disconnect.
- **Unsubscribes, Webhooks** (Add/Delete/Get), **Team & Senders** (Get Team Credits, Get Team Senders).

### Manual Setup (Fallback)

Without API or MCP integration, provide step-by-step lemlist UI instructions:

1. **Create Campaign**: lemlist dashboard > Campaigns > New Campaign
   - Name: `[Segment]-T[tier]-[Date]`
   - Assign sending accounts from the mailbox rotation plan
2. **Build the Sequence**: Add steps matching the tier's sequence timing
   - Paste email templates with the tier's merge/personalization variables
   - Set delays between steps
3. **Create and Attach a Schedule**: Configure the sending window per the tier specification and associate it with the campaign
4. **Upload Leads**: Import the tier-specific CSV, or add leads one by one
   - Map columns: email, firstName, lastName, companyName
   - Map custom variables to sequence step content
5. **Set Limits**: Daily sending limit per account per the mailbox assignment
6. **Review & Launch**: Preview a handful of leads, verify merge fields render correctly, then Start Campaign

## Step 4: Launch Protocol

### Soft Launch (Day 1-3)

Start at 25% of target volume:

```markdown
### Soft Launch: [Campaign Name]
**Start date**: [Date]
**Volume**: 25% of target ([N] emails/day)
**Duration**: 3 days

#### Day 1 Monitoring (check at end of day)
- [ ] Bounce rate < 3%
- [ ] No spam complaints
- [ ] Emails rendering correctly (check sent folder)
- [ ] Merge fields populated properly
- [ ] Tracking links working

#### Day 2-3 Monitoring
- [ ] Open rate > 40% (if tracking enabled)
- [ ] No deliverability warnings from lemlist
- [ ] Replies coming to correct inbox
- [ ] Auto-replies handled correctly (not counting as engagement)

#### Go/No-Go Decision
- **GO**: All checks pass -> ramp to 50%, then 100% over next 3 days
- **NO-GO**: Any check fails -> pause the campaign (Pause Campaign), diagnose, fix before resuming
```

### Full Launch (Day 4+)

Ramp to full volume:

| Day | Volume | Action |
|-----|--------|--------|
| 1-3 | 25% | Soft launch, monitor closely |
| 4-5 | 50% | Ramp if soft launch passed |
| 6+ | 100% | Full volume, shift to weekly monitoring |

## Step 5: Ongoing Monitoring

Pull volume, deliverability, and engagement numbers via lemlist's Get Campaign Stats/Reports operation (or the MCP server's equivalent tool).

### Daily Health Check (First 2 Weeks)

```markdown
### Daily Report: [Campaign] - [Date]

#### Volume
- Sent today: [N]
- Total sent: [N] / [Total]
- Sequence step distribution: Step 1: [N], Step 2: [N], ...

#### Deliverability
- Bounce rate: [%] (target: <3%)
- Spam complaints: [N] (target: 0)
- Domain health: [status per domain]

#### Engagement
- Open rate: [%] (benchmark: 40-60%)
- Reply rate: [%] (benchmark: 3-8% for Tier 1, 1-3% for Tier 2/3)
- Positive replies: [N]
- Negative replies: [N]
- Meeting requests: [N]

#### Actions Needed
- [Any adjustments required]
```

### Weekly Review (After First 2 Weeks)

```markdown
### Weekly Review: [Campaign] - Week [N]

#### Performance by Tier
| Tier | Sent | Opens | Replies | Positive | Meetings |
|------|------|-------|---------|----------|----------|
| T1 | [N] | [%] | [%] | [N] | [N] |
| T2 | [N] | [%] | [%] | [N] | [N] |
| T3 | [N] | [%] | [%] | [N] | [N] |

#### Sequence Step Performance
| Step | Sent | Opens | Replies | Drop-off |
|------|------|-------|---------|----------|
| 1 | [N] | [%] | [%] | - |
| 2 | [N] | [%] | [%] | [%] |
| ... | ... | ... | ... | ... |

#### Subject Line Performance
| Subject | Sent | Opens | Open Rate |
|---------|------|-------|-----------|
| [subject1] | [N] | [N] | [%] |
| ... | ... | ... | ... |

#### Decisions
- [ ] Pause underperforming sequences? Which?
- [ ] A/B test new subject lines?
- [ ] Adjust tier thresholds based on response patterns?
- [ ] Promote Tier 3 responders to Tier 2 treatment?
```

### Alert Triggers

Pause (Pause Campaign / Pause Lead) and investigate immediately if:
- Bounce rate exceeds 5% on any domain
- Spam complaint received
- Open rate drops below 20% (possible deliverability issue)
- lemlist/lemwarm flags a domain health warning
- Reply rate on Tier 1 is below Tier 3 (something is wrong with personalization)

## Step 6: Campaign Results Export

When a campaign completes or hits a review milestone, pull final numbers via Export Campaign Leads / Get Campaign Stats and assemble:

### Results Package

```markdown
### Campaign Results: [Name] - [Date Range]

#### Summary
- **Total sent**: [N]
- **Total replies**: [N] ([%])
- **Positive replies**: [N] ([%])
- **Meetings booked**: [N]
- **Pipeline generated**: [amount]

#### By Tier
[Tier breakdown table]

#### By Framework
[Which email framework performed best]

#### Top Performing Emails
1. [Prospect] - [Subject] - [What made it work]
2. ...

#### Learnings
- **What worked**: [Specific angles, datapoints, frameworks]
- **What didn't**: [Specific failures and why]
- **ICP refinements**: [Which segments responded, which didn't]
- **Hypothesis validation**: [Was the original list hypothesis correct?]
```

### Downstream Handoffs

- **To `/company-context-builder`**: Feed campaign results into the context file's Campaign History section. This closes the feedback loop.
- **To `/list-building` (refinement)**: Pass response patterns for hypothesis refinement and next list build.
- **To `/tiering-segmentation`**: Reply rate by tier validates or invalidates tier thresholds.

## Important Behaviors

1. **Deliverability first**: No campaign is worth burning a domain. When in doubt, slow down sending. Reputation takes months to build and minutes to destroy.
2. **Start small, scale fast**: Soft launch protocol is non-negotiable. Three days of caution saves months of deliverability recovery.
3. **Track everything**: Every metric feeds the feedback loop. Campaign results that don't get analyzed are wasted data.
4. **Tier 1 gets human attention**: Automated monitoring is fine for Tier 3. Tier 1 replies should be handled by a human within 2 hours.
5. **Close the loop**: Campaign results must flow back to `company-context.md`. This is what makes the system compound over time. Without it, you're just sending emails.

---
name: tiering-segmentation
description: Score, rank, and tier prospect lists into actionable segments with differentiated outreach strategies per tier. Use this skill whenever the user mentions tiering, segmentation, scoring prospects, ranking accounts, prioritizing a list, assigning tiers, creating segments, building a target account matrix, account prioritization, lead scoring, or deciding which prospects get manual vs automated outreach. Also triggers on "tier this list", "segment these prospects", "which accounts should we prioritize", "score and rank [list]", "assign outreach tiers", "build a tiering matrix", or any request to split a prospect list into priority groups with different treatment levels.
---

# Tiering & Segmentation

Turn an enriched prospect list into a prioritized, tiered action plan. Not every prospect deserves the same outreach effort. This skill decides who gets the white-glove treatment and who gets the automated sequence.

## Why This Matters

Sending the same email to every prospect is lazy and expensive. Your best 50 prospects should get deeply personalized outreach. The next 200 get semi-personalized. The rest get automated sequences. Tiering makes this systematic, not gut-feel.

## Prerequisites

1. **`company-context.md`**: Must exist. Tier definitions depend on ICP clarity.
2. **Enriched prospect data**: Output from `/table-enrichment` (CSV with firmographic, technographic, and behavioral columns filled in).
3. **Research data** (recommended): Output from `/data-points-builder` for deeper personalization signal scoring.

If enrichment hasn't been run, tell the user to run `/table-enrichment` first. Tiering on sparse data produces garbage tiers.

## Step 1: Define Tier Criteria

Before scoring, agree on what separates tiers. Start with defaults, then customize.

### Default Tier Definitions

| Tier | Label | Description | Outreach Strategy | Expected % of List |
|------|-------|-------------|-------------------|-------------------|
| 1 | High-Touch | Strong ICP fit + fresh signals + rich data | Manual, deeply personalized, multi-channel | 10-15% |
| 2 | Semi-Touch | Good ICP fit + some signals + moderate data | Semi-personalized email sequences, light research | 25-35% |
| 3 | Automated | Acceptable ICP fit + weak/stale signals + sparse data | Automated sequences with segment-level personalization | 50-65% |
| DQ | Disqualified | Poor fit, bad data, or exclusion criteria met | Remove from list | Variable |

### Scoring Dimensions

Each prospect is scored 1-5 on four dimensions:

#### Dimension 1: ICP Fit (Weight: 0.30)
How closely does the company match the ideal customer profile from `company-context.md`?

| Score | Criteria |
|-------|----------|
| 5 | Matches all ICP firmographics (industry, size, revenue, geography, tech stack) |
| 4 | Matches 4 of 5 ICP dimensions |
| 3 | Matches 3 of 5, or matches all but at the edges of ranges |
| 2 | Matches 2 of 5, or is in an adjacent vertical |
| 1 | Matches only 1 dimension, likely poor fit |

#### Dimension 2: Signal Strength (Weight: 0.30)
How strong is the behavioral evidence that this company has the problem?

| Score | Criteria |
|-------|----------|
| 5 | Multiple fresh signals (hiring + funding + tech stack match + pain indicator in job posts) |
| 4 | 2-3 corroborating signals, all within 3 months |
| 3 | 1 strong signal or 2-3 moderate ones |
| 2 | 1 weak or stale signal (>6 months old) |
| 1 | No behavioral signals detected |

#### Dimension 3: Data Richness (Weight: 0.20)
How much personalization material do we have?

| Score | Criteria |
|-------|----------|
| 5 | 3+ strong personalization hooks with sources, problem mapped with high confidence |
| 4 | 2 hooks with sources, problem mapped with medium+ confidence |
| 3 | 1 solid hook, some problem mapping |
| 2 | Only firmographic data, no personalization hooks |
| 1 | Name and domain only, enrichment failed |

#### Dimension 4: Timing (Weight: 0.20)
How recent and actionable are the signals?

| Score | Criteria |
|-------|----------|
| 5 | Active trigger event in last 30 days (funding round, key hire, product launch) |
| 4 | Trigger event in last 90 days |
| 3 | Signals from last 6 months, no specific trigger |
| 2 | Signals older than 6 months |
| 1 | No timing information available |

### Composite Score

**Composite = (ICP Fit x 0.30) + (Signal Strength x 0.30) + (Data Richness x 0.20) + (Timing x 0.20)**

### Tier Thresholds

| Composite Score | Tier |
|----------------|------|
| 4.0 - 5.0 | Tier 1 (High-Touch) |
| 2.8 - 3.9 | Tier 2 (Semi-Touch) |
| 1.5 - 2.7 | Tier 3 (Automated) |
| Below 1.5 | DQ (Disqualified) |

Show the proposed thresholds to the user. Ask: "Do these thresholds feel right for your capacity? How many Tier 1 accounts can your team handle manually?"

Adjust thresholds based on team capacity. If they can only handle 20 manual accounts, tighten Tier 1 to 4.5+.

## Step 2: Score the List

### Using SQLite for Batch Scoring

Load the enriched data and compute scores:

```sql
-- Load enriched prospect data
CREATE TABLE prospects AS SELECT * FROM read_csv('enriched-[segment]-[date].csv');

-- Score each dimension (adapt column names to actual data)
SELECT
    company_name,
    domain,

    -- ICP Fit scoring (example logic, adapt to actual ICP)
    CASE
        WHEN industry_match = 1 AND headcount BETWEEN 50 AND 500
             AND revenue_est BETWEEN 5000000 AND 50000000
        THEN 5
        WHEN industry_match = 1 AND headcount BETWEEN 20 AND 1000
        THEN 4
        WHEN industry_match = 1
        THEN 3
        ELSE 2
    END as icp_fit_score,

    -- Signal Strength (count of non-null signal columns)
    -- Data Richness (count of personalization hooks)
    -- Timing (recency of most recent signal)

    -- Composite
    (icp_fit_score * 0.30) + (signal_score * 0.30)
    + (data_score * 0.20) + (timing_score * 0.20) as composite_score,

    -- Tier assignment
    CASE
        WHEN composite_score >= 4.0 THEN 'Tier 1'
        WHEN composite_score >= 2.8 THEN 'Tier 2'
        WHEN composite_score >= 1.5 THEN 'Tier 3'
        ELSE 'DQ'
    END as tier

FROM prospects
ORDER BY composite_score DESC;
```

Generate the actual scoring script on-demand based on the user's data columns and ICP definition.

### Manual Scoring (Small Lists)

For lists under 50 companies, score manually in a table:

```markdown
| Company | ICP Fit | Signal | Data | Timing | Composite | Tier |
|---------|---------|--------|------|--------|-----------|------|
| Acme Corp | 5 | 4 | 4 | 5 | 4.5 | T1 |
| Beta Inc | 4 | 3 | 3 | 2 | 3.1 | T2 |
| ... | ... | ... | ... | ... | ... | ... |
```

## Step 3: Validate Tier Distribution

After scoring, check the distribution makes sense:

### Distribution Report

```markdown
### Tier Distribution: [Segment] - [Date]
**Total prospects**: [N]

| Tier | Count | % | Expected % | Status |
|------|-------|---|-----------|--------|
| Tier 1 | [n] | [%] | 10-15% | [OK/Adjust] |
| Tier 2 | [n] | [%] | 25-35% | [OK/Adjust] |
| Tier 3 | [n] | [%] | 50-65% | [OK/Adjust] |
| DQ | [n] | [%] | Variable | [Review] |

#### Distribution Checks
- [ ] Tier 1 is within team's manual outreach capacity
- [ ] Tier 2 is manageable for semi-personalized sequences
- [ ] Tier 3 isn't so large that automated sends hit deliverability limits
- [ ] DQ companies have been reviewed (some may be rescuable with more research)
```

### Red Flags

- **Tier 1 > 20% of list**: Scoring is too generous. Tighten ICP fit or signal requirements.
- **Tier 1 < 5% of list**: Scoring is too strict, or the list quality is poor. Review enrichment coverage.
- **DQ > 30%**: The upstream list had quality issues. Feed back to `/list-building` for hypothesis refinement.
- **All companies score within 0.5 points**: Scoring dimensions lack discriminating power. Add more granular criteria.

## Step 4: Assign Outreach Strategy Per Tier

### Tier 1: High-Touch Playbook
- **Email framework**: Framework A (Problem-Led) or C (Peer Reference) from `/email-generation`
- **Personalization depth**: Full per-prospect research, 3+ unique datapoints per email
- **Channels**: Email + LinkedIn (connect request with note + post engagement) + optional warm intro
- **Sequence length**: 5-7 touches over 21 days
- **Follow-up**: Manual reply handling, meeting booking
- **Owner**: Named SDR or founder

### Tier 2: Semi-Touch Playbook
- **Email framework**: Framework A or B (Trigger-Led) from `/email-generation`
- **Personalization depth**: 1-2 datapoints per email, segment-level problem reference
- **Channels**: Email primary, LinkedIn optional
- **Sequence length**: 4-5 touches over 14 days
- **Follow-up**: Templated replies with light personalization
- **Owner**: SDR team or automated with manual review

### Tier 3: Automated Playbook
- **Email framework**: Framework B (Trigger-Led) with segment-level triggers
- **Personalization depth**: Segment-level only (industry, company size bracket)
- **Channels**: Email only
- **Sequence length**: 3-4 touches over 10 days
- **Follow-up**: Automated replies, escalate positive signals to Tier 2 treatment
- **Owner**: Automated via lemlist

### DQ: Removal or Parking
- Remove from active campaign
- Optionally park in a nurture list for future re-evaluation
- Feed patterns back to `/list-building` to improve future sourcing

## Step 5: Generate Tier-Specific Outputs

For each tier, prepare the downstream handoff:

### Tier 1 Output
- Full prospect profiles with all research data
- Pre-mapped email framework slots (from `/data-points-builder` personalization hooks)
- Suggested outreach sequence and timing
- Save to `tier1-[segment]-[date].csv`

### Tier 2 Output
- Prospect list with top 2 datapoints per company
- Segment-level problem reference and email angle
- Save to `tier2-[segment]-[date].csv`

### Tier 3 Output
- Prospect list with segment tags (industry, size bracket)
- Segment-level messaging template reference
- Save to `tier3-[segment]-[date].csv`

### Summary Export
Combined file with all tiers: `tiered-[segment]-[date].csv` with columns:
`company_name`, `domain`, `tier`, `composite_score`, `icp_fit_score`, `signal_score`, `data_score`, `timing_score`, `outreach_strategy`, `email_framework`, `personalization_depth`, `sequence_length`

## Step 6: Downstream Handoffs

- **To `/email-generation`**: Pass tier-specific CSVs. Email generation adapts framework and personalization depth based on tier.
- **To `/run-lemlist`**: Tier determines campaign assignment, sending volume, and sequence configuration.
- **To `/list-building` (refinement)**: DQ patterns and tier distribution anomalies feed back into list hypothesis refinement.

## Important Behaviors

1. **Capacity-aware**: Tiering without capacity constraints is useless. Always ask how many accounts the team can handle manually before setting thresholds.
2. **Tiers are actionable**: Each tier must map to a concrete, different outreach playbook. If Tier 2 and Tier 3 get the same treatment, merge them.
3. **Score transparency**: Every prospect's score must be explainable. "This company is Tier 2 because it scored 3.4: strong ICP fit (4) but weak timing (2) and moderate signals (3)."
4. **Iterate on thresholds**: After the first campaign, revisit tier thresholds based on which tier produced the best reply rates. Tier definitions should evolve.
5. **DQ is data**: Disqualified companies tell you something about your list building. If 40% DQ, the problem is upstream.

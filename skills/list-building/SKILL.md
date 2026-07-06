---
name: list-building
description: Build hypothesis-driven prospect lists for outbound campaigns using lookalike signals, firmographic filters, and ICP criteria. Use this skill whenever the user mentions building a list, finding prospects, sourcing leads, creating a target account list, generating lead lists, pulling companies from a database, building TAM lists, finding lookalikes, prospecting, ICP list building, or refining a list after campaign results. Also triggers on "find companies like [customer]", "build me a list of [segment]", "who should we target", "source leads for [campaign]", "refine the list based on results", or any request to identify target companies or contacts for outbound.
---

# List Building

Hypothesis-driven list building. Every list starts with a thesis about who will buy and why, not a keyword dump into a database.

## Why This Matters

Bad lists poison everything downstream. If you target the wrong companies, the best emails in the world won't save you. This skill forces you to articulate WHY each company is on the list before they get added.

## Prerequisites

1. **`company-context.md`**: Must exist. Run `/company-context-builder` first if it doesn't.
2. **Problem taxonomy** (`problem-taxonomy-[segment].md`): Recommended but not required for first pass. If missing, list building will surface which problems to research.

## Step 1: Define the Hypothesis

Before touching any database, force the user to articulate a hypothesis:

**Template**: "Companies that [firmographic filter] AND [behavioral signal] are likely experiencing [problem from taxonomy] because [reasoning]."

### Examples
- "B2B SaaS companies with 50-200 employees that recently hired a RevOps lead are likely struggling with pipeline attribution because they're scaling past the point where manual tracking works."
- "E-commerce brands doing EUR 5-20M revenue that run on Shopify Plus are likely feeling margin pressure from fulfillment costs because they've outgrown basic 3PL setups."

If the user says "just find me SaaS companies in Europe," push back. Ask: what problem do you think they have, and what signal would indicate that?

## Step 2: Translate Hypothesis to Filters

Map the hypothesis to concrete, queryable filters:

### Filter Categories

#### Firmographic (Hard Filters)
- Industry/vertical (SIC, NAICS, or keyword-based)
- Employee count range
- Revenue range
- Geography (country, region, city)
- Founding year / company age
- Funding stage / total raised

#### Technographic (Signal Filters)
- Tech stack (CRM, MAP, ERP, etc.)
- Platform (Shopify, WordPress, Salesforce, etc.)
- Infrastructure signals (cloud provider, CDN, analytics tools)

#### Behavioral (Intent Signals)
- Recent hiring patterns (specific roles)
- Job postings mentioning specific tools/challenges
- Recent funding rounds
- Leadership changes
- Product launches or pivots
- Conference attendance / speaking engagements

#### Lookalike Signals
- Similar to existing customers (by firmographics, tech stack, or growth trajectory)
- Competitors of existing customers
- Companies in adjacent verticals to current wins

### Output: Filter Specification

```markdown
### List Hypothesis: [Name]
**Thesis**: [One sentence]
**Expected list size**: [Estimate]

#### Hard Filters
- Industry: [values]
- Headcount: [range]
- Revenue: [range]
- Geography: [values]

#### Signal Filters
- Tech stack: [must-have tools]
- Hiring: [roles being hired]
- Funding: [stage/recency]

#### Lookalike Basis
- Similar to: [customer names or profile]
- Matching on: [which dimensions]
```

Show this to the user for approval before sourcing.

## Step 3: Source the List

### Lookalikes API (Primary)

Your lookalike/similar-company data source is accessed via its public API.
Provide your own key through the `LOOKALIKE_API_KEY` environment variable —
never hardcode a key in this file. Bring-your-own integration: see the
provider's API docs for the endpoints this step calls.

When connected, this step will:
1. Take the filter spec and existing customer list
2. Call the lookalikes API with firmographic + technographic parameters
3. Return ranked companies by similarity score
4. Deduplicate against existing pipeline/CRM

### Manual Sourcing (Fallback)

If no API is connected, guide the user through manual sourcing:

1. **LinkedIn Sales Navigator**: Provide the exact search filter configuration
2. **Crunchbase**: Provide search parameters for funding/industry filters
3. **Apollo/ZoomInfo**: Provide filter setup instructions
4. **Google**: Specific search queries to find companies matching the hypothesis

For each source, output the exact filters to apply, not vague instructions.

### Data Format

All lists must be output as structured data (CSV or markdown table) with minimum columns:

| Column | Required | Description |
|--------|----------|-------------|
| company_name | Yes | Legal or common name |
| domain | Yes | Primary website domain |
| industry | Yes | Vertical/sector |
| headcount | Yes | Employee count or range |
| revenue_estimate | No | Revenue range if available |
| geography | Yes | HQ location |
| hypothesis_match | Yes | Which hypothesis filter(s) matched |
| signal_evidence | Yes | What behavioral signal was detected |
| source | Yes | Where this company was found |

## Step 4: Validate and Score

Before the list goes downstream, run validation:

### Validation Checks
1. **Domain check**: Does the website exist and match the company?
2. **Size check**: Is the company within ICP range?
3. **Signal freshness**: Is the behavioral signal from the last 6 months?
4. **Duplicate check**: Already in CRM or previous campaigns?
5. **Hypothesis alignment**: Can you explain in one sentence why this company is on the list?

### Scoring

Score each company 1-5 on:
- **ICP fit** (firmographic match): How closely does it match the ideal profile?
- **Signal strength** (behavioral evidence): How strong is the intent signal?
- **Timing** (recency of signals): How recent are the indicators?

**Composite score** = (ICP fit x 0.4) + (Signal strength x 0.4) + (Timing x 0.2)

Sort the list by composite score. Flag any company scoring below 2.5 for removal.

## Step 5: Refinement Mode (Post-Campaign)

After running a campaign, this skill can be re-invoked with results:

1. User provides campaign performance data (which companies replied, which bounced, which converted)
2. Analyze patterns:
   - What firmographic/technographic traits do responders share?
   - What signals were present in converted companies but absent in non-responders?
   - Were there segments that completely flatlined?
3. Update the hypothesis based on evidence
4. Generate a refined filter spec for the next list pull
5. Feed learnings back into `company-context.md` via the context builder feedback loop

### Refinement Output
```markdown
### List Refinement: [Campaign Name] - [Date]
**Original hypothesis**: [What we assumed]
**What the data showed**: [What actually happened]
**Updated hypothesis**: [What we believe now]
**Filter changes**:
- Added: [new filters]
- Removed: [dropped filters]
- Adjusted: [modified ranges/values]
**Expected impact**: [How this should change results]
```

## Step 6: Export

Save the validated list to `prospect-list-[segment]-[date].csv` with all columns from Step 3 plus validation scores from Step 4.

If the user wants to proceed to research/enrichment, hand off to `/data-points-builder`.

## Important Behaviors

1. **Hypothesis first, always**: Never build a list without articulating why these companies should be targeted. If the user resists, explain that spray-and-pray outbound has 0.5% reply rates while hypothesis-driven has 5-15%.
2. **Show your work**: Always show the filter spec before sourcing. The user should be able to explain every filter.
3. **Quality over quantity**: A list of 200 well-matched companies beats 5,000 random ones. Push back on "just get me 10,000 leads."
4. **Compound learning**: Every list refinement makes the next list better. Always connect results back to the hypothesis.
5. **No hallucinated companies**: If using manual sourcing, the user provides the raw data. Never invent company names or details.

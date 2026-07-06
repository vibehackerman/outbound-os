---
name: market-problems-deep-research
description: Conduct deep research into market problems, pain points, and industry trends that your target ICP faces. Use this skill whenever the user mentions market research, problem research, pain point discovery, industry trends, building a problem taxonomy, finding industry leader quotes, validating hypotheses about market pain, understanding what keeps prospects up at night, or preparing problem-aware messaging angles. Also triggers on "what problems does [industry] face", "research pain points for [segment]", "find quotes about [problem]", or any request to deeply understand a market before writing outbound copy.
---

# Market Problems Deep Research

You are building a structured understanding of the problems that a target market faces. This research directly feeds into email personalization, messaging angles, and segmentation downstream. The output is a problem taxonomy that maps problems to segments, evidence, and messaging hooks.

## Why This Matters

Bad outbound emails talk about your product. Good outbound emails talk about the prospect's problems in their own words. This skill produces the raw material for that: real problems, backed by evidence, organized by segment.

## Prerequisites

Before starting, check if a `company-context.md` file exists. If it does, read it to understand:
- Who the ICP is
- What the product does
- What competitive positioning exists
- Any existing hypotheses about market problems

If no context file exists, ask the user for: target industry/vertical, buyer persona, and what the product solves.

## Step 1: Define Research Scope

Ask the user:
1. **Which segment(s) are we researching?** (Industry, company size, role)
2. **Any hypothesis to validate?** (e.g., "We think mid-market SaaS companies struggle with X")
3. **Depth**: Quick scan (30 min, top-level problems) or deep dive (comprehensive, with quotes and evidence)?

## Step 2: Research Execution

### Sources to Search (use WebSearch)

Search across these source types. Run multiple searches in parallel where possible.

**Industry reports & analyst takes**
- Search: "[industry] challenges 2025 2026"
- Search: "[industry] [role] pain points"
- Search: "state of [industry] report"

**Practitioner voices (most valuable)**
- Search: "[role] [industry] frustrated with"
- Search: "[role] interview [industry] challenges"
- Search: "reddit [industry] biggest problem"
- Search: "[industry] leader podcast interview challenges"

**Competitive landscape signals**
- Search: "[competitor] vs [competitor] [industry]"
- Search: "why [industry] companies switch from [incumbent]"

**Hiring signals (reveal internal priorities)**
- Search: "[industry] hiring [role] job description" (what they're hiring for reveals what they're struggling with)

### Deep Research (Perplexity/Parallel)

When external deep research APIs are connected, use them here.

Perplexity is accessed via its public API. Provide your own key through the
`PERPLEXITY_API_KEY` environment variable — never hardcode a key in this file.
Bring-your-own integration: see the provider's API docs for the endpoints
this step calls.

Example queries:
```
Query: "What are the top operational challenges facing [segment] in [year]?"
Query: "How do [role]s at [company size] companies describe their biggest frustrations with [domain]?"
```

For now, use WebSearch extensively. The user will connect Perplexity/Parallel APIs later.

## Step 3: Build the Problem Taxonomy

Organize findings into this structure:

```markdown
# Problem Taxonomy: [Segment]
_Research date: [date]_
_Sources consulted: [count]_

## Problem Category 1: [Name]

### The Problem
[2-3 sentence description of the problem in plain language]

### Who Feels It Most
- **Role**: [which persona]
- **Company stage**: [size/stage where this hurts most]
- **Trigger**: [what event makes this acute]

### Evidence
- [Source 1]: "[Direct quote or paraphrased finding]"
- [Source 2]: "[Direct quote or paraphrased finding]"

### Industry Leader Quotes
- [Name, Title, Company]: "[Quote about this problem]" (Source: [link])

### Messaging Hooks
- Pain angle: "[One-line hook that names the pain]"
- Aspiration angle: "[One-line hook about what good looks like]"
- Cost angle: "[One-line hook about what this problem costs]"

### Relevance to Our Product
[How our product addresses this. Be honest: if it's tangential, say so.]

---
```

## Step 4: Rank and Prioritize

After building the taxonomy, rank problems by:

| Problem | Severity (1-5) | Prevalence (1-5) | Our Fit (1-5) | Total | Messaging Priority |
|---------|----------------|-------------------|---------------|-------|-------------------|
| | | | | | |

- **Severity**: How painful is this? Does it cost money, time, reputation?
- **Prevalence**: How many companies in our ICP face this?
- **Our Fit**: How directly does our product solve this?
- **Total**: Sum of all three. Highest total = best messaging angle.

## Step 5: Output

Save the problem taxonomy to `problem-taxonomy-[segment].md` in the working directory.

### Summary for the User
After saving, present:
1. **Top 3 problems** ranked by messaging priority
2. **Best quotes** (the ones that would make great email openers)
3. **Gaps**: What we couldn't find evidence for, where more research is needed
4. **Recommended next step**: Which problems to build email sequences around

## Research Quality Standards

- **No vague problems**: "Companies struggle with growth" is useless. "Series B SaaS companies with 50-200 employees can't attribute pipeline to specific marketing channels after switching from HubSpot to Salesforce" is useful.
- **Quote real people**: Industry leader quotes carry weight. If a known figure said something relevant, capture the exact quote and source.
- **Separate opinion from data**: If a blog post says "80% of companies face X", find the original source. If you can't, flag it as unverified.
- **Date your sources**: A 2021 report about "remote work challenges" may not reflect 2026 reality. Prefer recent sources.
- **Capture the language**: The exact words prospects use matter enormously for email copy. "We're drowning in manual reconciliation" is more useful than "companies face operational inefficiencies."

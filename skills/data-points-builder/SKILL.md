---
name: data-points-builder
description: Research and collect structured datapoints on prospect companies for email personalization. Use this skill whenever the user mentions researching prospects, gathering company intel, collecting datapoints, building research profiles, enriching prospect data, finding personalization hooks, company research for outbound, prospect research, or building a research table. Also triggers on "research these companies", "find datapoints for [list]", "what do we know about these prospects", "gather intel on [segment]", "build personalization data", or any request to collect company-level or person-level data for outbound campaigns.
---

# Data Points Builder

Turn a prospect list into a research-enriched dataset with specific, verifiable datapoints that feed email personalization. No vague "they're a growing company" allowed.

## Why This Matters

Personalization without datapoints is just mail merge. "Hi {first_name}" is not personalization. Knowing that a company just hired 3 data engineers, runs on Snowflake, and their VP of Engineering spoke at a conference about data pipeline reliability... that's personalization. This skill builds the datapoint layer that makes emails specific and credible.

## Prerequisites

1. **`company-context.md`**: Must exist. Need to know what's relevant to research.
2. **Prospect list**: Output from `/list-building` (CSV or table with company names, domains, and hypothesis match).
3. **Problem taxonomy** (`problem-taxonomy-[segment].md`): To know which problems to look for evidence of.

If any are missing, tell the user what's needed and which skill to run first.

## Step 1: Define the Research Schema

Before researching, agree on what datapoints to collect. The schema depends on which email framework will be used downstream.

### Default Research Schema

| Datapoint | Priority | Used By | Source |
|-----------|----------|---------|--------|
| Recent news (last 6mo) | High | Trigger-led emails | Web search |
| Hiring signals (open roles) | High | Problem-led, trigger-led | Career pages, LinkedIn |
| Tech stack | High | Problem-led | BuiltWith, job posts, website |
| Recent funding | Medium | Trigger-led | Crunchbase, press |
| Leadership changes | Medium | Trigger-led | LinkedIn, press |
| Company size / growth trajectory | Medium | Segmentation | LinkedIn, Crunchbase |
| Customer-facing language | High | Peer reference | Website, case studies |
| Conference appearances | Low | Personalization hooks | Event sites |
| Competitive tools in use | High | Problem-led | Job posts, reviews |
| Pain indicators in job posts | High | Problem-led | Career pages |

### Custom Schema

Ask the user: "What additional datapoints matter for your product/ICP?" Add custom columns to the research table.

## Step 2: Research Each Prospect

For each company on the list, conduct structured research following this sequence:

### Research Sequence

#### 2a. Website Scan
- Homepage: What do they say they do? What language do they use?
- Careers/jobs page: What roles are open? What tools are mentioned in job descriptions?
- Blog/news: Any recent announcements, product launches, strategic shifts?
- About/team: Leadership team, company stage signals

#### 2b. Web Search
Use targeted searches:
- `"[company name]" + hiring + [relevant role]`
- `"[company name]" + funding OR raised OR series`
- `"[company name]" + [problem keyword from taxonomy]`
- `"[company name]" + [competitor product] OR [alternative tool]`
- `site:linkedin.com/in "[prospect name]" + [company]`

#### 2c. Deep Research (Perplexity)

Perplexity is accessed via its public API. Provide your own key through the
`PERPLEXITY_API_KEY` environment variable — never hardcode a key in this file.
Bring-your-own integration: see the provider's API docs for the endpoints
this step calls.

When connected, send structured queries:
1. "What are the main operational challenges facing [company] in [area relevant to product]?"
2. "What technology stack does [company] use for [relevant function]?"
3. "Recent news about [company] in the last 6 months"

#### 2d. Social Signals
- LinkedIn company page: Recent posts, employee growth, content themes
- Prospect's personal LinkedIn: Recent posts, articles, comments, shared content
- Twitter/X: Company or prospect's public statements about relevant topics

### Research Quality Rules

1. **Every datapoint must have a source**: URL, date, or platform reference
2. **Freshness matters**: Flag anything older than 6 months as potentially stale
3. **Distinguish fact from inference**: "They posted a job for a RevOps manager" is a fact. "They're struggling with RevOps" is an inference. Label accordingly.
4. **No hallucination**: If you can't find something, say so. An empty field is better than a fabricated datapoint.
5. **Prioritize actionable over interesting**: "CEO went to Stanford" is trivia. "CEO posted about struggling with data quality last week" is actionable.

## Step 3: Build the Research Table

### Output Format

For each prospect, create a structured profile:

```markdown
### [Company Name] | [Domain]
**Researched**: [Date]
**Confidence**: [High/Medium/Low based on data availability]

#### Key Datapoints
| Datapoint | Value | Source | Freshness |
|-----------|-------|--------|-----------|
| Recent news | [specific finding] | [URL] | [date] |
| Hiring signals | [roles, count] | [URL] | [date] |
| Tech stack | [tools found] | [source] | [date] |
| Pain indicators | [specific signals] | [source] | [date] |
| ... | ... | ... | ... |

#### Personalization Hooks (ranked)
1. **Strongest**: [Most specific, recent, relevant datapoint] - usable in [framework]
2. **Secondary**: [Next best] - usable in [framework]
3. **Tertiary**: [Backup option] - usable in [framework]

#### Problem Mapping
- **Most likely problem** (from taxonomy): [Problem ID/name]
- **Evidence**: [Which datapoints support this]
- **Confidence**: [High/Medium/Low]

#### Research Gaps
- [What we couldn't find that would be valuable]
```

### Batch Research Table

For batch processing, also maintain a summary CSV with one row per company:

Columns: `company_name`, `domain`, `top_datapoint`, `datapoint_source`, `likely_problem`, `problem_evidence`, `confidence_score`, `research_date`, `personalization_hook_1`, `personalization_hook_2`

## Step 4: Quality Assurance

Before handing off to email generation, validate:

### QA Checklist
1. **Coverage**: Every company has at least one strong personalization hook beyond name/title/company
2. **Source verification**: All datapoints link to a verifiable source
3. **Freshness**: Flag companies where all datapoints are older than 3 months
4. **Problem mapping**: Every company maps to at least one problem from the taxonomy
5. **Duplicates**: No two companies share identical personalization hooks (each email should be unique)
6. **Relevance**: Every datapoint connects to the product/problem, not just random company facts

### Red Flags to Surface
- Company with zero usable datapoints (recommend removal from list or manual research)
- Company where all signals are stale (>6 months old)
- Company where the problem mapping feels forced (low confidence)

## Step 5: Export and Handoff

Save research to `research-[segment]-[date].md` (full profiles) and `research-[segment]-[date].csv` (summary table).

### Downstream Handoffs
- **To `/email-generation`**: Pass the research table as prospect data input
- **To `/table-enrichment`**: If deeper enrichment is needed (Extruct, additional APIs)
- **To `/tiering-segmentation`**: Research quality and signal strength feed into tier assignments

## Important Behaviors

1. **Depth over breadth**: 50 deeply researched companies beat 500 with just a name and domain. If the list is too large for thorough research, recommend tiering first and deep-researching only Tier 1.
2. **Research is perishable**: Always date-stamp findings. A datapoint from 6 months ago may be irrelevant today.
3. **Show uncertainty**: Use confidence levels honestly. "Low confidence" is more useful than a guess presented as fact.
4. **Connect to problems**: Every datapoint should relate back to a problem the product solves. If you find interesting but irrelevant data, note it but don't prioritize it.
5. **Respect rate limits**: When using web search, batch sensibly. Don't fire 500 searches in a row.

---
name: table-enrichment
description: Enrich prospect data tables using structured extraction, SQLite queries, and deep research APIs. Use this skill whenever the user mentions enriching data, enriching a list, adding data to prospects, running enrichment, Extruct, structured data extraction, scraping company data, augmenting prospect tables, filling in missing fields, cross-referencing data sources, or building enrichment pipelines. Also triggers on "enrich this list", "fill in missing data for [prospects]", "extract structured data from [source]", "run enrichment on [CSV]", "add technographic data", or any request to systematically add columns or fill gaps in a prospect dataset.
---

# Table Enrichment

Systematically enrich prospect data tables by combining structured extraction, API lookups, and deep research. This is the data pipeline step that turns a basic list into a rich, queryable dataset.

## Why This Matters

Raw prospect lists have gaps. Names and domains aren't enough for personalization. This skill fills those gaps systematically, not one company at a time, but as a batch pipeline with quality checks at each stage.

## Prerequisites

1. **Prospect data**: A CSV or table from `/list-building` or `/data-points-builder` with at minimum `company_name` and `domain` columns.
2. **`company-context.md`**: To know which enrichment dimensions matter for the ICP.
3. **Research schema**: Either from `/data-points-builder` or defined fresh. Knowing what columns to fill is step zero.

## Step 1: Audit the Current Data

Before enriching, assess what you have:

```markdown
### Data Audit: [Dataset Name]
**Row count**: [N]
**Columns present**: [list]

#### Completeness Matrix
| Column | Filled | Empty | Fill Rate |
|--------|--------|-------|-----------|
| company_name | N | 0 | 100% |
| domain | N | X | Y% |
| industry | ... | ... | ...% |
| headcount | ... | ... | ...% |
| ... | ... | ... | ...% |

#### Priority Gaps
1. [Column with lowest fill rate that's high priority for email personalization]
2. [Next gap]
3. [Next gap]
```

Show this to the user. Ask: "Which gaps are highest priority to fill?"

## Step 2: Design the Enrichment Pipeline

Map each gap to an enrichment source:

### Source Priority (in order of reliability)

#### Tier 1: Structured APIs
- **Extruct**: Extract structured data (Schema.org, JSON-LD, OpenGraph) from company websites
- **Clearbit/Apollo/ZoomInfo**: Firmographic and technographic data
- **Crunchbase**: Funding, investors, news

#### Tier 2: Structured Extraction
- **Career pages**: Parse job listings for tech stack signals, hiring velocity, pain indicators
- **Website metadata**: Extract industry, size signals, product descriptions from structured data
- **Social profiles**: Employee count, growth rate, content themes

#### Tier 3: Deep Research
- **Web search**: Targeted queries per company for specific datapoints
- **Perplexity**: Natural language research queries
- **News APIs**: Recent press coverage, announcements

### Pipeline Specification

```markdown
### Enrichment Pipeline: [Name]
**Input**: [source CSV/table]
**Target columns**: [what to fill]

| Column to Fill | Source | Method | Expected Fill Rate |
|---------------|--------|--------|-------------------|
| tech_stack | Extruct + job posts | Structured extraction | ~70% |
| headcount | LinkedIn/API | API lookup | ~90% |
| recent_news | Web search | Keyword search | ~60% |
| ... | ... | ... | ...% |
```

## Step 3: Run Enrichment

### Extruct Integration

Extruct is accessed via its public API. Provide your own key through the
`EXTRUCT_API_KEY` environment variable — never hardcode a key in this file.
Bring-your-own integration: see the provider's API docs for the endpoints
this step calls.

When connected, for each domain:
1. Fetch structured data (JSON-LD, Schema.org, OpenGraph, Microdata)
2. Extract: organization type, industry, employee count, founding date, social profiles, product descriptions
3. Parse and normalize into table columns
4. Flag low-confidence extractions

### Fallback: Manual Structured Extraction

Without Extruct, use web scraping/search to extract:

```python
# Pseudo-pipeline for each company:
# 1. Fetch homepage -> extract meta tags, OpenGraph data
# 2. Find /careers or /jobs page -> parse open roles
# 3. Search "[company] site:linkedin.com" -> extract company data
# 4. Search "[company] + [enrichment keyword]" -> parse results
```

Generate a Python script on-demand when the user wants to run batch extraction. The script should:
- Read from the input CSV
- Process companies in batches (respect rate limits)
- Output an enriched CSV with new columns appended
- Log successes, failures, and partial results
- Be resumable (skip already-enriched rows)

### SQLite for Data Operations

For complex enrichment logic (joins, deduplication, scoring), load the data into SQLite:

```sql
-- Load prospect data
CREATE TABLE prospects AS SELECT * FROM read_csv('prospect-list.csv');

-- Load enrichment results
CREATE TABLE enrichment AS SELECT * FROM read_csv('enrichment-results.csv');

-- Join and fill gaps
SELECT
    p.*,
    COALESCE(p.headcount, e.headcount) as headcount_enriched,
    COALESCE(p.industry, e.industry) as industry_enriched,
    e.tech_stack,
    e.recent_funding
FROM prospects p
LEFT JOIN enrichment e ON p.domain = e.domain;
```

Use SQLite for:
- Deduplication across multiple enrichment sources
- Conflict resolution (when two sources disagree)
- Filling gaps with fallback values
- Scoring and ranking enriched data
- Cross-referencing against exclusion lists

## Step 4: Quality Checks

After enrichment, validate:

### Validation Rules
1. **No contradictions**: If two sources give different headcounts, flag and pick the most recent/reliable
2. **Reasonable ranges**: Headcount of 1,000,000 for a Series A startup is wrong. Apply sanity checks.
3. **Freshness tags**: Add `_last_updated` column showing when each datapoint was sourced
4. **Source attribution**: Add `_source` columns so downstream skills know where data came from
5. **Null audit**: After enrichment, re-run the completeness matrix. Did we actually close the gaps?

### Post-Enrichment Report

```markdown
### Enrichment Results: [Dataset Name] - [Date]
**Input rows**: [N]
**Columns added**: [list]

#### Fill Rate Improvement
| Column | Before | After | Delta |
|--------|--------|-------|-------|
| tech_stack | 10% | 72% | +62% |
| headcount | 45% | 91% | +46% |
| ... | ... | ... | ... |

#### Data Quality
- Contradictions resolved: [N]
- Sanity check failures: [N] (details below)
- Rows with zero enrichment: [N] (recommend removal or manual research)

#### Recommended Actions
- [What to do about low-quality rows]
- [Which columns still need work]
- [Whether to re-run with different sources]
```

## Step 5: Export

Save enriched data to `enriched-[segment]-[date].csv`.

### Downstream Handoffs
- **To `/tiering-segmentation`**: Enriched data enables better tier assignment
- **To `/email-generation`**: Enriched datapoints feed personalization
- **To `/list-building` (refinement)**: Enrichment patterns can reveal new list hypotheses

## Important Behaviors

1. **Pipeline mindset**: This is a data pipeline, not one-off research. Design it to be re-runnable.
2. **Source transparency**: Always track where each datapoint came from. Downstream skills need this for personalization verification.
3. **Fail gracefully**: Not every company will return data from every source. Empty fields are fine. Invented data is not.
4. **Batch efficiently**: Group API calls, use caching, respect rate limits. Don't make 500 individual web searches when 50 targeted batch queries would work.
5. **SQLite is your friend**: For any data operation involving more than 50 rows or 2 sources, load into SQLite and use SQL. It's faster, more reliable, and auditable.

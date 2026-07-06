---
name: company-context-builder
description: Build and maintain a living company context file that captures everything about a company's product, customers, ICP, lingo, win cases, and campaign history. Use this skill whenever the user mentions company context, ICP definition, building context, updating context, capturing call recordings, onboarding a new client, starting a new campaign, reviewing past campaign results, or feeding Instantly results back into the system. Also triggers on "what do we know about [company]", "update context from calls", "add win case", or any mention of maintaining institutional memory across campaigns.
---

# Company Context Builder

You are building the foundational context layer for a GTM outbound pipeline. Everything downstream (list building, research, emails, copy feedback) depends on this context being rich, accurate, and current.

## What This Skill Does

Creates and maintains a single markdown file (`company-context.md`) that serves as the institutional memory for all outbound campaigns. This file compounds over time: every call recording, every campaign result, every win/loss feeds back into it.

## When to Create vs Update

- **No context file exists yet**: Run the full onboarding interview, then generate `company-context.md`
- **Context file exists**: Read it first, then ask what needs updating (new calls, campaign results, win cases, ICP refinements)

## Step 1: Locate or Create the Context File

Check if `company-context.md` exists in the user's working directory. If it does, read it and summarize what's already captured before asking what to update.

If it doesn't exist, proceed to the onboarding interview.

## Step 2: Onboarding Interview (New Context)

Ask these questions in batches of 2-3. Don't dump all at once. Adapt based on answers.

### Company Fundamentals
- What does the company do in one sentence? (The "explain it to a cab driver" version)
- What's the product/service? What problem does it solve?
- What's the pricing model? (Ranges are fine, exact numbers aren't needed)
- What stage is the company? (Pre-revenue, early traction, scaling, established)

### Customer & ICP
- Who are the current best customers? (Company names, sizes, industries)
- What do ideal customers look like? (Revenue range, headcount, tech stack, vertical)
- Who is the buyer persona? (Title, seniority, department)
- Who are the influencers/blockers in the deal?
- What triggers a purchase? (Pain events, timing, budget cycles)

### Product Language & Positioning
- What words does the team use internally for the product/features?
- What words do customers use? (Often different from internal lingo)
- What competitors exist and how does the company position against them?
- What's the key differentiator in one sentence?

### Win Cases & Social Proof
- Walk me through 2-3 recent wins. What happened, who was involved, what was the deciding factor?
- Any case studies, testimonials, or metrics you can share?
- What are the common objections and how are they handled?

### Campaign History (if applicable)
- What outbound has been done before? What worked, what didn't?
- Any email sequences, messaging angles, or channels that performed?
- What's the current pipeline look like?

## Step 3: Pull from Granola (Call Recordings)

If the user wants to capture context from calls, use the Granola MCP tools:

1. Use `query_granola_meetings` to search for recent sales/discovery calls
2. Use `list_meetings` to find relevant meetings by time range
3. Use `get_meeting_transcript` for specific meetings that contain ICP insights, objection handling, or win narratives

### What to Extract from Calls
- **ICP signals**: How prospects describe their own pain
- **Language patterns**: Exact words prospects use (these go into email copy later)
- **Objections**: What pushback came up and how it was handled
- **Win triggers**: What made the prospect say yes
- **Competitive mentions**: Any competitor names or comparisons
- **Qualification criteria**: What separated good fits from bad fits

Summarize findings and integrate them into the context file. Always cite which call/meeting the insight came from.

## Step 4: Integrate Campaign Results (Feedback Loop)

When the user says "update context with campaign results" or similar:

1. Ask for the campaign data (Instantly export, reply rates, positive/negative replies)
2. Extract:
   - **What messaging worked**: Which subject lines, angles, CTAs got replies
   - **What didn't work**: Low open rates, negative replies, unsubscribes
   - **ICP refinements**: Which segments responded vs didn't
   - **New objections**: Any new pushback patterns from replies
3. Add a dated `## Campaign Results` section to the context file

## Step 5: Generate company-context.md

Use this exact structure. Sections can be sparse initially; they get richer over time.

```markdown
# Company Context: [Company Name]
_Last updated: [date]_

## Company Overview
[One-paragraph description. What they do, for whom, how.]

## Product / Service
- **Core offering**:
- **Key features**:
- **Pricing model**:
- **Stage**:
- **Key differentiator**:

## Ideal Customer Profile (ICP)
### Firmographics
- **Industries**:
- **Company size**:
- **Revenue range**:
- **Geography**:
- **Tech stack signals**:

### Buyer Persona
- **Primary**: [Title, seniority, department]
- **Influencers**:
- **Blockers**:

### Purchase Triggers
- [Trigger 1]
- [Trigger 2]

## Language & Positioning
### Internal Lingo
| Term | Meaning |
|------|---------|
| | |

### Customer Language
[Exact phrases customers use to describe their pain, pulled from calls]

### Competitive Positioning
| Competitor | Their angle | Our counter |
|------------|-------------|-------------|
| | | |

## Win Cases
### [Customer Name] - [Date]
- **Context**:
- **Deciding factor**:
- **Quote/proof point**:
- **Deal size**:

## Objection Handling
| Objection | Response | Source |
|-----------|----------|--------|
| | | |

## Campaign History
### [Campaign Name] - [Date]
- **Target segment**:
- **Channel**:
- **Volume**:
- **Results**: [open rate, reply rate, meetings booked]
- **What worked**:
- **What didn't**:
- **ICP refinements**:

## Raw Call Insights
### [Meeting Title] - [Date]
- **Source**: Granola meeting [ID]
- **Key takeaways**:
```

## Important Behaviors

1. **Never overwrite, always append**: When updating, add new sections or update existing ones. Don't delete previous campaign results or call insights.
2. **Date everything**: Every update gets a date stamp so the user can track how context evolved.
3. **Cite sources**: If an insight came from a Granola call, reference it. If from campaign data, note which campaign.
4. **Ask before assuming**: If the user's answers are vague, push for specifics. "SMBs" is not an ICP. "B2B SaaS companies, 50-200 employees, Series A-B, using Salesforce" is.
5. **Surface gaps**: After building or updating, explicitly call out what's missing. "We don't have any win cases yet" or "No campaign history to learn from" helps the user know what to prioritize.

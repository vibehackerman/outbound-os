---
name: copy-feedback
description: Run outbound email copy through a multi-layer quality review before sending. Use this skill whenever the user mentions reviewing emails, getting feedback on copy, simulating prospect reactions, cold-read testing, persona simulation, email QA, checking if an email sounds generic, refining outbound copy, running emails through a critic, or pre-send quality checks. Also triggers on "review this email", "how would [prospect] react to this", "does this email suck", "simulate a cold read", "run copy feedback on [emails]", or any request to stress-test outbound copy before it goes live. This is not a grammar checker. It simulates real prospect psychology.
---

# Copy Feedback

Before sending, run your emails through three independent review layers. This skill doesn't say "good email." It tells you what feels off, what's generic, what would make a real person delete it, and what to fix.

## Why This Matters

Most email review is "does this sound okay?" That's useless. The bar isn't "okay." The bar is: would a busy VP who gets 80 cold emails a week actually read past the first line? This skill simulates that ruthless filter.

## Prerequisites

1. **Emails to review**: Either from `/email-generation` output or pasted directly
2. **Prospect data**: The datapoints used to generate the emails (needed for persona simulation)
3. **`company-context.md`**: For positioning and language checks
4. **Problem taxonomy** (optional but recommended): To verify problem references are accurate

## The Three Review Layers

Each layer is independent. Run all three, then synthesize. Disagreement between layers is valuable signal, not a problem.

---

### Layer 1: The Cold Reader

Simulates a prospect seeing this email with zero context. No benefit of the doubt.

**Persona**: A busy executive who gets 50+ cold emails per day. Scans subject lines in 1 second, decides to open or not. Scans the first line in 2 seconds, decides to read or delete. Has no patience for anything that feels templated.

**What the Cold Reader checks**:

1. **Subject line (1-second test)**: Would I open this or skip it? Why?
2. **First line (2-second test)**: Does the first sentence make me think "this person knows something about me" or "this is a mass email"?
3. **Skim test**: If I skim (not read), what impression do I get in 5 seconds?
4. **Delete triggers**: Any phrase that screams "sales email"? (e.g., "reaching out", "hoping to", "would love to")
5. **Reply motivation**: After reading, is there any reason I'd reply? What would I reply?
6. **Forward test**: Would I forward this to a colleague with "worth a look"?

**Output format**:
```
COLD READ VERDICT: [DELETE / MAYBE / READ / REPLY]
Reason: [One sentence]
Delete trigger: [Specific phrase if any]
First-line score: [1-5] - [Why]
Would reply because: [Reason or "wouldn't"]
Fix: [One specific change that would move the verdict up one level]
```

---

### Layer 2: The Persona Simulator

Deep-researches the specific prospect and simulates their reaction based on who they actually are.

**Research step** (run for each prospect):

Use WebSearch to find:
- Their LinkedIn activity (posts, comments, shared articles)
- Any podcast interviews, conference talks, blog posts they've written
- Their company's recent news, press releases, product launches
- Their team's hiring patterns (what roles they're adding)
- Any public opinions on the problem your email references

When you connect a data-enrichment or LinkedIn data provider, pull structured profile data here using your own API key (via an environment variable). No keys or provider-specific credentials belong in this file.

**What the Persona Simulator checks**:

1. **Voice match**: Does this email speak their language? A CTO who posts about "engineering velocity" won't respond to "streamline operations."
2. **Problem accuracy**: Is the problem we reference one they actually care about? Evidence from their public content?
3. **Status awareness**: Does the email respect their seniority level? A VP doesn't want a 101 explanation. An IC doesn't want C-suite jargon.
4. **Competitive context**: Are they already using a competing product? Does the email acknowledge or ignore this?
5. **Timing relevance**: Is there anything in their recent activity that makes this email timely or outdated?
6. **Ego check**: Does anything in the email feel condescending, presumptuous, or like it's telling them something obvious?

**Output format**:
```
PERSONA: [Name, Title, Company]
Public signals found: [List of sources reviewed]
Likely reaction: [Specific and honest]
Voice alignment: [Match / Partial / Mismatch] - [Why]
Problem relevance: [High / Medium / Low] - [Evidence]
Ego risk: [None / Minor / Major] - [What specifically]
Rewrite suggestion: [Specific alternative phrasing based on their actual language]
```

---

### Layer 3: The Framework Auditor

Checks the email against the generation framework's rules. Pure compliance check, no subjectivity.

**What the Framework Auditor checks**:

1. **Structure compliance**: Does the email follow the framework's line-by-line structure?
2. **Word count**: Within limits?
3. **Banned phrases**: Any violations from the banned phrase list?
4. **Personalization verification**: Every personalized element traces to a real datapoint?
5. **CTA format**: Matches framework requirements (question vs. statement, friction level)?
6. **Subject line rules**: Within word limit, no banned patterns?
7. **Sender voice**: Peer-to-peer, not vendor-to-prospect?
8. **One-product-sentence rule**: Max one sentence about the product?

**Output format**:
```
FRAMEWORK AUDIT: [PASS / FAIL]
Violations: [List each rule broken]
Personalization map:
  - "[Personalized phrase]" → [Datapoint source] ✓
  - "[Personalized phrase]" → [No source found] ✗
Word count: [X] / [Limit]
Banned phrases found: [List or "None"]
```

---

## Synthesis: The Verdict

After all three layers, combine into a single verdict:

```markdown
## Email Review: [Prospect Name]

### Verdict: [SEND / REVISE / REWRITE / KILL]

**SEND**: All three layers pass. Minor tweaks optional.
**REVISE**: One layer flagged issues. Specific fixes listed below.
**REWRITE**: Two+ layers flagged issues. Keep the angle, rewrite the execution.
**KILL**: Fundamental problem (wrong angle, wrong persona, no real personalization). Start over.

### What's Working
- [Specific elements that passed all three layers]

### What Needs Fixing
| Issue | Source | Fix |
|-------|--------|-----|
| [Problem] | [Which layer flagged it] | [Specific rewrite] |

### Suggested Rewrite
[If REVISE or REWRITE, provide the improved version with changes highlighted]
```

## Per-Prospect vs. Batch Review

### Per-Prospect Review
Run all three layers for individual emails. Use when:
- Testing a new framework before batch generation
- High-value prospects that warrant individual attention
- User wants to iterate on a specific email

### Batch Review
For large batches (10+ emails), run a modified process:
1. **Full three-layer review on 3 random samples** from the batch
2. **Framework Auditor on all emails** (rules-based, fast)
3. **Pattern detection across the batch**:
   - Are certain phrases repeating too often?
   - Is there enough variation in CTAs?
   - Are any datapoints being overused or underused?
4. **Flag outliers**: Emails that deviate significantly from the batch average (too long, too short, missing personalization)

### Batch Output
```markdown
## Batch Review: [Segment] - [Count] emails

### Sample Reviews
[Full three-layer review for 3 samples]

### Batch-Wide Patterns
- **Repetition**: [Phrases appearing in >30% of emails]
- **CTA distribution**: [Breakdown of CTA types used]
- **Personalization depth**: [Avg datapoints per email, min, max]
- **Word count range**: [Min - Max - Avg]

### Batch Verdict: [X] ready to send, [Y] need revision, [Z] need rewrite

### Flagged Emails
[List of emails that need individual attention, with reason]
```

## Refinement Loop

After presenting the verdict, enter refinement:

1. User reviews the feedback and suggested rewrites
2. User accepts, modifies, or rejects each suggestion
3. For accepted suggestions: apply changes, re-run the layer that flagged it to confirm the fix works
4. For rejected suggestions: ask why and update the review criteria (maybe the "rule" was wrong, not the email)
5. When the user is satisfied, update the email generation framework with any new rules discovered during review

This creates a feedback loop: review findings improve the generation instructions, which improve future emails, which require less review.

## Quality Standards

- **Be specific, not vague**: "This feels generic" is useless feedback. "The phrase 'streamline your operations' appears in 40% of cold emails according to Lavender data; replace with a specific operation they're struggling with" is useful.
- **Back up opinions with evidence**: If the persona simulator says "they wouldn't care about this problem," cite what they DO care about based on their public content.
- **Preserve what works**: Don't rewrite emails that are working. If Layer 1 says REPLY but Layer 2 flags a minor voice mismatch, suggest a tweak, not a rewrite.
- **Honest verdicts**: If an email should be killed, say so. Polishing a bad angle wastes everyone's time.
- **Track patterns**: If the same issue keeps coming up across emails, the problem is in the framework, not the individual email. Surface this.

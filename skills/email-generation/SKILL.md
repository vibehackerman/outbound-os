---
name: email-generation
description: Generate personalized cold outbound emails by following strict instructions, not creative writing. Use this skill whenever the user mentions generating emails, writing email sequences, assembling outbound emails, creating cold emails, personalizing email copy, building email drafts from prospect data, applying email frameworks, instruction-based email writing, or running email generation for a prospect list. Also triggers on "generate emails for [segment]", "write cold emails using [framework]", "personalize emails for [list]", "assemble emails from datapoints", or any request to turn prospect research into send-ready email copy. This is NOT a creative copywriting skill. It is an instruction-following assembly skill with iterative refinement.
---

# Email Generation

Email generation is not creative writing. It's instruction-following. This skill takes a strict set of rules, applies them to each prospect's datapoints and context, assembles the email deterministically, then lets you refine iteratively in the chat.

## Why This Matters

Most AI-generated emails sound like AI-generated emails. They're vague, sycophantic, and interchangeable. This skill avoids that by treating email generation as a structured assembly process: specific inputs produce specific outputs. No filler, no "I hope this email finds you well."

## Prerequisites

Before generating emails, you need:

1. **`company-context.md`**: Read it. Know the product, ICP, positioning, and language.
2. **Problem taxonomy** (`problem-taxonomy-[segment].md`): Know which problems to reference.
3. **Prospect data**: A CSV, table, or structured data with per-prospect datapoints (name, title, company, industry, company size, recent news, tech stack, hiring signals, etc.)
4. **Email framework/instructions**: Either provided by the user or built from the framework library below.

If any of these are missing, tell the user what's needed and which skill to run first.

## Step 1: Load the Framework

Ask the user which email framework to use, or let them provide custom instructions.

### Built-in Frameworks

#### Framework A: Problem-Led
```
Structure:
- Line 1: Name a specific problem they face (from problem taxonomy + their datapoints)
- Line 2-3: Show you understand the cost/impact of that problem (use their context)
- Line 4: One sentence on how [product] addresses this specifically
- Line 5: Low-friction CTA (question, not ask)

Rules:
- Max 75 words total
- No adjectives about the product
- No "I/we" in the first sentence
- CTA must be a question, not a command
- Every claim must map to a datapoint
```

#### Framework B: Trigger-Led
```
Structure:
- Line 1: Reference a specific trigger event (hiring, funding, product launch, exec change)
- Line 2: Connect the trigger to a likely operational challenge
- Line 3: Brief proof point (case study reference, metric)
- Line 4: CTA tied to the trigger

Rules:
- Max 60 words
- Trigger must be verifiable (link or date)
- No generic congratulations
- CTA connects trigger to conversation
```

#### Framework C: Peer Reference
```
Structure:
- Line 1: "[Similar company] was dealing with [specific problem]"
- Line 2: What changed and what result they got
- Line 3: "Noticed [prospect company] might face something similar because [datapoint]"
- Line 4: CTA

Rules:
- Max 80 words
- Similar company must be in same vertical or size range
- Result must be specific (metric, not vague improvement)
- "Because [datapoint]" must reference actual prospect data
```

#### Framework D: Custom
The user provides their own framework as structured instructions. Parse it into rules and structure, then apply consistently.

### Framework Rules (Apply to All)
These rules override any framework-specific guidance:

1. **No filler phrases**: "I hope this finds you well", "I wanted to reach out", "Just following up" are banned.
2. **No product dumping**: Never list features. One sentence max about the product.
3. **Every sentence must earn its place**: If removing a sentence doesn't change the email's effectiveness, remove it.
4. **Personalization must be verifiable**: Every personalized element maps to a specific datapoint. No "I noticed your company is growing" without citing what you noticed.
5. **Subject lines**: Max 5 words. No clickbait. No emojis. No "Quick question" or "Thoughts?". Subject should hint at the problem, not the product.
6. **Sender voice**: Write as a peer, not a vendor. No "we help companies like yours" framing.

## Step 2: Map Datapoints to Framework Slots

For each prospect, explicitly map their data to the framework's structure before writing.

Example mapping (Framework A):
```
Prospect: Jane Smith, VP Ops, Acme Corp (Series B, 120 employees, SaaS, using Salesforce + HubSpot)
Problem (from taxonomy): Mid-market SaaS companies can't reconcile pipeline attribution across CRM and marketing tools
Datapoint used: Using both Salesforce and HubSpot (tech stack signal)
Cost/impact: Pipeline reviews take 2+ hours because attribution is manual
Product connection: [Product] auto-syncs attribution across CRM and MAP
CTA: "Is attribution reconciliation eating your Monday mornings too?"
```

Show this mapping to the user before generating. This is the blueprint; the email is just the assembly.

## Step 3: Generate the Email

Assemble the email strictly following the framework and mapping. Do not improvise. Do not add sentences that aren't in the framework structure.

### Output Format Per Prospect
```markdown
### [Prospect Name] - [Company]
**Framework**: [Which framework]
**Key datapoint used**: [What personalization is based on]
**Problem referenced**: [From taxonomy]

**Subject**: [Subject line]

**Body**:
[Email body]

**Mapping verification**:
- [ ] Every personalized element maps to a datapoint
- [ ] Word count within framework limit
- [ ] No banned phrases
- [ ] CTA is a question
- [ ] Subject line ≤ 5 words
```

## Step 4: Iterative Refinement

After showing the generated email(s), enter refinement mode:

1. **User gives feedback**: "Too long", "Wrong angle", "This datapoint is more relevant", "Sounds too salesy"
2. **Apply feedback as a rule**: Don't just fix this email. Update the framework instructions so all subsequent emails reflect the feedback.
3. **Regenerate**: Show the updated email with a diff of what changed and why.
4. **Lock**: When the user approves, mark the framework + rules as "locked" for batch generation.

### Refinement Prompts to Offer
- "Want me to try a different problem angle?"
- "Should I tighten the word count further?"
- "Is the CTA too aggressive or too soft?"
- "Any datapoints I'm not using that I should?"

## Step 5: Batch Generation

Once the framework is locked, generate for the full prospect list.

### Batch Output
Save to `emails-[segment]-[date].md` with:
- Summary stats (count, avg word count, frameworks used)
- All emails in the per-prospect format from Step 3
- A quality checklist applied to each email

### Batch Quality Gates
Before presenting the batch, run these checks:

1. **Uniqueness**: No two emails should share more than 20% of their non-framework text. Flag duplicates.
2. **Datapoint coverage**: Every email must use at least one prospect-specific datapoint beyond name/title/company. Flag any that don't.
3. **Word count**: All within framework limits. Flag any that exceed.
4. **Banned phrase scan**: Automated check for all banned phrases. Flag violations.
5. **CTA variety**: If more than 30% of emails use the same CTA phrasing, flag for diversification.

## Step 6: Export for lemlist

lemlist is driven via its public API (`https://api.lemlist.com/api`, HTTP
Basic auth with an empty username and the API key as password). Provide your
own key through the `LEMLIST_API_KEY` environment variable — never hardcode a
key in this file or in campaign configs. See lemlist's API docs for the
campaign, lead, and account endpoints this step calls (the `/run-lemlist`
skill drives the actual calls).

When the lemlist integration is connected, this step will:
1. Format emails into lemlist's lead schema (lead email plus per-step subject/body content)
2. Map to the correct campaign based on segment/tier
3. Validate against lemlist's sending limits and rate limits
4. Upload via the Create-Lead-in-Campaign API (or the lemlist MCP server)

For now, output a CSV with columns: `email`, `first_name`, `last_name`, `company`, `subject`, `body`, `sequence_step` that can be manually imported into lemlist.

### CSV Generation Script

When generating the CSV export, use Python:
```python
# Script location: scripts/export_to_lemlist.py
# Takes the generated emails markdown and converts to lemlist-compatible CSV
# Handles: UTF-8 encoding, quote escaping, field validation
```

The script should be generated on-demand when the user requests CSV export.

## Research Quality Standards

- **No hallucinated personalization**: If you don't have a datapoint, don't invent one. Better to use a segment-level insight than a fake personal one.
- **Cite your sources**: Every personalized element should trace back to the prospect data or problem taxonomy.
- **Test the "delete test"**: For each sentence, ask "if I delete this, does the email still work?" If yes, delete it.
- **Read it out loud**: Would a real human say this to a peer at a bar? If not, rewrite.
- **Anti-patterns to catch**: "Leverage", "synergy", "solution", "I'd love to", "partnership opportunity", "thought leader", "best-in-class"

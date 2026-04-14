# Output Template

Write the final document to `business-ideas/{slug}.md` using this structure. Keep each section concise — evidence-dense, not verbose. Clearly distinguish desk research from real customer conversations throughout.

## Table of Contents

1. The Idea & Core Hypothesis
2. Assumptions Log
3. Why Now
4. Customer Segment
5. Pain Assessment
6. Competitive Landscape
7. Market Size
8. Willingness to Pay
9. Founder-Market Fit
10. Distribution Path
11. Pre-sell Test Plan
12. Kill Conditions
13. Open Questions & Next Steps
14. Verdict

---

```markdown
# {Idea Name}

## 1. The Idea & Core Hypothesis

{One or two sentences stating the idea plainly. No pitch language — just the bare claim being tested.}

## 2. Assumptions Log

| # | Assumption | Killer? | Verdict | Evidence |
|---|-----------|---------|---------|----------|
| 1 | {assumption} | Yes/No | VALID/WEAK/INCORRECT | {one-line summary} |
| 2 | ... | ... | ... | ... |

## 3. Why Now

{What has changed recently that makes this problem solvable today? Cite specific shifts — regulatory, technological, behavioral, cost — or note honestly if no clear timing thesis exists.}

**Timing strength**: {strong/moderate/weak}

## 4. Customer Segment

{The specific person being targeted. Job title, company size, industry, behavioral trait — specific enough to find ten of them by next week. Note which persona variant won validation and why.}

## 5. Pain Assessment

**From desk research:**
- **Frequency**: {how often — with evidence}
- **Urgency**: {how much it hurts — with evidence}
- **Current spend**: {money/time/effort — with evidence}

**From customer conversations** (if conducted):
- {What the user heard from target customers — workarounds, complaints, past spending, surprises}
- {If none yet: "No direct customer conversations conducted — critical next step. Pain assessment is desk-research-only."}

{If desk research and conversations conflicted, note the conflict and which signal was given precedence.}

## 6. Competitive Landscape

{Who else has tried, their approach, negative reviews, and the specific wedge.}

**Direct competitors:**
| Name | Approach | Pricing | Key weakness |
|------|----------|---------|--------------|
| ... | ... | ... | ... |

**Gaps identified**: {underserved needs from review analysis}

{If no competition: explicit analysis of why — market too small, too hard, too early, or quietly decided not worth solving.}

## 7. Market Size

- **Total addressable population**: {N} — {source}
- **Problem penetration**: {X%} — {reasoning}
- **Revenue per customer/year**: ${Y}
- **Estimated TAM**: ${calculated}
- **Growth trajectory**: {growing/flat/shrinking}

## 8. Willingness to Pay

- **Price hypothesis**: {suggested range, anchored to pain level AND competitive pricing}
- **Existing pricing landscape**: {what competitors/alternatives charge}
- **Commitment signals**: {pre-orders, switching behavior, contract sizes}
- **User's price instinct**: {what the founder thinks they could charge}
- **Gap analysis**: {where instinct diverges from market reality, if at all}

## 9. Founder-Market Fit

{Honest assessment from interactive discussion. Domain expertise, relationships, structural access, lived experience — or lack thereof. If no edge, note the specific plan to compensate.}

**Unfair advantage**: {specific edge, or "None identified — raises the bar on everything else"}

## 10. Distribution Path

- **Founder's plan**: {how they plan to reach first 100 customers}
- **Research findings**: {viable channels, estimated CAC, what worked for similar products}
- **Recommended first channel**: {channel with reasoning}
- **Feasibility**: {credible/uncertain/weak}

## 11. Pre-sell Test Plan

{The behavioral commitment test designed during validation.}

- **What to build**: {minimum viable test}
- **Commitment signal**: {payment, signup, LOI, time investment}
- **Target**: {N people}
- **Go signal**: {X% conversion or better}
- **Stop signal**: {below Y% conversion}

**Status**: Not yet executed — this is the most important next step.

**If results are available** (user returned with data):
- **Showed to**: {N people}
- **Converted**: {N} ({X%})
- **Assessment**: {meets go threshold / between thresholds / below stop threshold}

## 12. Kill Conditions

{Pre-committed criteria that would cause a stop or pivot. Derive from: every fired soft gate (Phases 2, 5, 7, 8, 10), every WEAK killer assumption, every INCORRECT non-killer assumption, and the pre-sell stop threshold from Section 11.}

- If {condition from fired gate or weak assumption}, then {stop / pivot / re-validate}
- If pre-sell converts below {stop threshold}%, then {pivot or stop}
- If {condition}, then {action}

## 13. Open Questions & Next Steps

**Critical unknowns** (ranked by decision impact):
1. {question} — {specific action to answer it}

**Required actions before committing:**
- [ ] Execute the pre-sell test from Section 11
- [ ] Conduct 5–10 problem discovery conversations with target segment (listen, don't pitch)
- [ ] {Other actions identified during the process}

**Unvalidated inputs:**
- Founder-market fit: {discussed but not externally verified}
- Distribution: {researched but not tested}
- Pre-sell: {designed but not executed}

---

## Verdict: {GO / GO WITH CONDITIONS / PIVOT / NO GO}

{2-3 sentence rationale citing the strongest signals for and against.}

**If GO WITH CONDITIONS:**
1. {specific condition that must be met}
2. {specific condition that must be met}

**If PIVOT:**
{What the research revealed as a better adjacent opportunity — different customer, reframed problem, or different wedge.}

*Confidence note: This verdict is based on desk research and founder self-assessment. Pain discovery (Section 5), distribution (Section 10), and pre-sell (Section 11) are partially or fully unvalidated. The verdict may change after real-world testing.*
```

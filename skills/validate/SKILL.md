---
name: validate
description: Interactive startup idea validation. Use when the user presents a business idea to validate, evaluate, or stress-test, or asks whether a startup concept is worth pursuing.
argument-hint: "business idea description"
---

# Validating Business Ideas

Guide the user through an interactive, phased validation of their startup idea. The process alternates between collaborative discussion (you ask, they think) and automated research (sub-agents search the web). Each phase builds on the last — context accumulates and searches get more targeted.

**Output**: `{output-dir}/{slug}.md` — see [output-template.md](reference/output-template.md)

**Sub-agent prompts**: Individual prompt files in [reference/prompts/](reference/prompts/) — read only the one you need per phase, then inline into the sub-agent dispatch.

**Prerequisites**: Several sub-agent prompts use the Reddit MCP tools (`mcp__reddit-mcp-buddy__search_reddit`, `mcp__reddit-mcp-buddy__browse_subreddit`) alongside `WebSearch`. If Reddit MCP tools are unavailable, sub-agents should fall back to web search to find Reddit discussions and forum threads instead.

## Rules
- Ask ONE question at a time. Wait for the answer before continuing.
- Be collaborative - guide the user to think, don't hand them answers.
- If a sub-agent fails, re-invoke it.
- Each phase builds on prior phases — pass accumulated context forward.
- Respect the Phase 3 stop gate. If all killer assumptions fail, stop and advise on pivots.

---

## Phase 1 — Idea, Assumptions & Customer Hypothesis

**Mode**: Interactive

### Step 0: Confirm output location

Before anything else, ask:

> "Where should I save the final report? I'll default to `~/startup-ideas/` if you don't have a preference."

Accept a path or confirm the default. Expand `~` to the full home directory path. Store this as `{output-dir}` — use it when writing the final document.

### Step 1: Understand the idea

Ask the user to describe the idea in a few sentences. Get the bare claim, not a pitch.

### Step 2: Identify the customer hypothesis

Ask:

> "Who do you think would pay for this? Give me a job title, company size, industry, or behavioral trait — specific enough that you could find ten of them by next week."

This hypothesis feeds into assumption validation. It gets formally validated in Phase 4.

### Step 3: Surface hidden assumptions

Ask:

> "Every business idea is a stack of untested assumptions. What assumptions is this idea relying on to work?"

Let them list assumptions. Then probe for gaps using Socratic questions — don't hand them the answer. Common blind spots to check (only raise if the user missed them):

- The problem exists at meaningful scale for the identified customer
- The customer is willing to change current behavior
- You can reach this customer at viable cost
- They'll pay enough to sustain a business
- The technical solution is feasible now
- No regulatory or legal blockers

Ask follow-ups like: "What would have to be true about [X] for this to work?" or "You're assuming [Y] — have you seen evidence of that?"

### Step 4: Identify killer assumptions

Ask:

> "Which of these assumptions would kill the entire idea if they turned out to be wrong?"

Guide if they miss obvious killers: "What happens to the idea if [assumption X] is false?" Don't label them yourself — help the user arrive there.

### Assumption volume guidance

Aim for 5–8 assumptions total. If the user surfaces more than 8, help them consolidate related ones. In Phase 3, killer assumptions get validated first; after killers, validate non-killers in order of perceived risk, up to the 6 sub-agent cap. If the total assumption count exceeds available capacity, leave the lowest-risk non-killers unvalidated and flag them in the output.

Record all assumptions with killer flags before moving on.

---

## Phase 2 — Why Now

**Mode**: Interactive + 1 sub-agent + soft gate

Ask:

> "Why is now the right time for this? What has changed recently that makes this problem solvable today when it wasn't before?"

Probe their answer. Good timing theses are specific and verifiable — a regulatory shift, a technology cost collapse, a behavioral change. Vague answers like "the market is growing" or "AI makes it possible now" need sharpening. Ask: "What specifically changed, and when?"

If the user can't articulate a timing thesis, that's signal too. Some ideas don't have a "why now" — the problem has existed for years and nothing has changed. Note this honestly.

After the discussion, dispatch a `general-purpose` sub-agent with the **Why Now Researcher** prompt from [reference/prompts/why-now.md](reference/prompts/why-now.md), inlined with the idea context and the user's timing thesis (or lack thereof). The sub-agent searches for external signals: regulatory filings, infrastructure cost curves, adoption data, failed predecessors.

Read the sub-agent's output. Present findings alongside the user's thesis.

### Soft gate

Fire the gate if the user's thesis is weak **or** if the research contradicts it:

- **User thesis weak, research also negative**: "Neither your thesis nor the research identified a clear reason why this works now. That doesn't kill the idea, but it means you're not riding a tailwind — you're pushing uphill. Do you want to think more about timing, or continue with this risk noted?"
- **User thesis confident, research contradicts it**: "You have a clear timing thesis, but the research found [contradicting signal]. Worth reconciling before you go further."

Record the timing assessment (strong/moderate/weak) and any revisions. Continue either way, but carry the rating forward.

### Timing return mechanism

When the user comes back with a revised timing thesis or new evidence after continuing:

1. Receive what they found — new data points, regulatory changes, market shifts
2. Assess whether it changes the timing rating (strong/moderate/weak)
3. Update the Phase 2 entry in the context block
4. If the rating improves, note it and continue. If the concern still isn't resolved, re-state the soft gate finding and continue with the risk noted.

---

## Phase 3 — Assumption Validation (HARD GATE)

**Mode**: Research

For each assumption, dispatch a `general-purpose` sub-agent with the **Assumption Validator** prompt from [reference/prompts/assumption-validator.md](reference/prompts/assumption-validator.md), inlined with:

- The specific assumption
- Whether it's a killer
- The idea context, customer hypothesis, and all other assumptions

Each sub-agent rates: **VALID** / **WEAK** / **INCORRECT** with evidence.

Validate killer assumptions first. Launch up to 6 sub-agents in parallel; queue the rest.

### Hard gate

After all sub-agents return:

- **Any killer assumption INCORRECT** → STOP. Present findings. Explain why the core thesis doesn't hold. Suggest pivots based on what the research _did_ reveal. Do not continue to Phase 4. Generate final output immediately — Pivot verdict if the research surfaced an adjacent opportunity with stronger signals, No Go verdict if not. Write to `business-ideas/{slug}.md`. Include this disclaimer in the output: "This verdict is generated from Phases 1–3 only. Phases 4–11 (customer validation, pain, competitive landscape, market size, pricing, distribution, pre-sell) have not been run. Any pivot recommendation is directional — treat it as a hypothesis for a new validation pass, not a confirmed direction."
- **Any killer assumption WEAK** → Discussion gate. Present the evidence and ask: "This killer assumption came back weak: [summary]. The research found [evidence]. Do you have additional evidence that strengthens it, or should we treat this as a serious risk?" If the user can't strengthen it, note it as a condition on the final verdict.
- **All killers VALID** → Present the full scorecard, discuss any weak non-killers, continue.

Non-killer assumptions rated INCORRECT are automatically added as named conditions on the final verdict, regardless of what the verdict tier is.

### Cumulative non-killer check

After the hard gate, if more than half of the non-killer assumptions are rated INCORRECT, present a summary before proceeding: "Your killer assumptions held, but [N] of [M] supporting assumptions failed validation. While none of these individually kills the idea, the pattern suggests the foundation may be weaker than expected. Here's what failed: [list]. Do you want to continue with these as conditions, or revisit the idea framing first?" This is a discussion gate, not a hard stop — continue if the user wants to proceed, but note the pattern in the verdict.

If sub-agents return conflicting verdicts on related assumptions, flag the conflict and discuss with the user before proceeding. Heuristic: two assumptions are "related" if flipping one would change the verdict on the other.

---

## Phase 4 — Customer Segment Validation

**Mode**: Interactive + 3 sub-agents

Revisit the customer hypothesis from Phase 1. Ask if the assumption validation changed their thinking about the customer.

Dispatch 3 `general-purpose` sub-agents in parallel with the **Customer Persona Validator** prompt from [reference/prompts/customer-persona-validator.md](reference/prompts/customer-persona-validator.md). Each gets:

- The user's described persona
- A different angle:
  - **Agent A**: Validates the user's exact persona
  - **Agent B**: Explores an adjacent or broader segment
  - **Agent C**: Explores a narrower or more specific sub-segment

Each searches for evidence the persona exists, has the problem, and actively seeks solutions.

Present all three. The persona with the strongest evidence match wins. Discuss with the user and confirm the segment before proceeding.

If the confirmed segment differs materially from the Phase 1 hypothesis, flag which Phase 3 assumption verdicts may need revisiting — particularly assumptions about willingness to change behavior, ability to reach the customer, and price sensitivity — and note them as conditions on the verdict.

### Handling a fundamental segment shift

If the winning segment represents a fundamental customer type shift (e.g., consumer → business, SMB → enterprise, different industry), follow this decision tree:

1. Mark all segment-dependent Phase 3 verdicts as UNVALIDATED for the new segment.
2. Recommend re-running Phase 3 for the new segment before proceeding.
3. **If the user accepts the re-run:**
   a. Check whether the segment shift also changes the context underlying the Phase 2 timing thesis (regulatory, technological, or behavioral). If so, offer to re-run Phase 2 first.
   b. Substitute the new segment into the customer hypothesis field of the context block. All other unaffected Phase 1–2 context stands.
   c. Run Phase 3 sub-agents against the new segment.
   d. Update the Phase 3 and Phase 4 entries in the context block with the new verdicts and confirmed segment.
   e. Continue to Phase 5.
4. **If the user declines the re-run:**
   a. Keep all affected Phase 3 verdicts marked as UNVALIDATED for the new segment.
   b. Add them as named conditions on the verdict.
   c. Continue to Phase 5 with the new segment in context.

---

## Phase 5 — Pain Discovery

**Mode**: Interactive + 1 sub-agent (hybrid)

This phase combines real customer conversations with desk research. Real conversations take precedence over desk research when they disagree.

### Interactive: Conversation prep

Ask the user:

> "Have you talked to anyone in your target segment about this problem? Not pitched — listened?"

**If yes**: Ask them to share what they heard — workarounds, complaints, past spending, surprises. Log it.

**If no**: Guide them on what to ask. The goal is to surface past behavior, not collect opinions about your idea. Example questions and the principles behind them:

- _"Tell me about the last time you dealt with [problem]."_ — Asks about a specific past event, not a hypothetical. The details reveal how real and frequent the pain is.
- _"What did you do about it?"_ — Surfaces workarounds. Active workarounds are the strongest signal that the problem matters.
- _"How much time or money did that cost you?"_ — Quantifies current spend without asking whether they'd pay for a solution.
- _"What was the most frustrating part?"_ — Lets them identify the pain point, not you.

Encourage them to talk to 5–10 people and come back. But if the user wants to continue without conversations, that's their choice — flag it: the pain assessment will be desk-research-only, and the verdict will note this as an unvalidated input.

### Research: Desk supplement

Dispatch a `general-purpose` sub-agent with the **Pain Assessor** prompt from [reference/prompts/pain-assessor.md](reference/prompts/pain-assessor.md), inlined with accumulated context.

### Combining evidence

Present desk research findings alongside whatever the user shared from real conversations. Label each source clearly.

**If they conflict** — e.g., desk research says pain is high but conversations found indifference, or vice versa — real conversations take precedence. Surface the conflict directly: "The desk research suggests [X], but your conversations found [Y]. The people you talked to are closer to the truth than aggregated web data. What do you make of the gap?" If the conflict is significant, treat it as a discussion gate before proceeding.

### Soft gate

If the combined pain assessment (desk research + any conversations) indicates MILD pain — low frequency, low urgency, no meaningful current spend or workarounds — surface it:

> "The pain assessment suggests this problem may not be painful enough to drive purchasing behavior: [specific finding]. That doesn't mean there's no opportunity, but it raises the bar on everything else — especially pricing and distribution. Do you want to revisit the problem framing, or continue with this flagged?"

Record the pain level (SEVERE/MODERATE/MILD). A MILD rating counts toward the cumulative soft gate check in Phase 10.

### Conversation return mechanism

When the user comes back with conversation notes after continuing without them:

1. Receive what they heard — workarounds, complaints, past spending, surprises
2. Log the conversation data against the existing desk research
3. Surface any conflicts between conversations and desk research (conversations take precedence)
4. Update the pain assessment and the Phase 5 entry in the context block
5. If the updated pain assessment materially changes the picture — e.g., desk research said SEVERE but conversations revealed MILD — ask the user whether to re-run the affected downstream phases or carry the discrepancy forward as a named condition on the verdict

---

## Phase 6 — Competitive Landscape

**Mode**: Research (1 sub-agent)

Dispatch a `general-purpose` sub-agent with the **Competitive Landscape Researcher** prompt from [reference/prompts/competitive-landscape.md](reference/prompts/competitive-landscape.md), inlined with accumulated context.

The sub-agent must flag **absence of competition** as a potential warning sign — if nobody has tried this, surface why.

Present findings — competitors, negative reviews, gaps, and where the wedge might be. Discuss with user.

_This phase runs before WTP and Distribution so those phases can use competitor pricing and channels as inputs._

---

## Phase 7 — Market Size

**Mode**: Research (1 sub-agent)

Dispatch a `general-purpose` sub-agent with the **Market Size Researcher** prompt from [reference/prompts/market-size.md](reference/prompts/market-size.md), inlined with accumulated context.

Present the back-of-envelope TAM calculation.

### Soft gate

If the TAM calculation can't reach a plausible path to a meaningful business under reasonable assumptions — e.g., the total market tops out under $5M — surface it:

> "The market size math suggests a ceiling of [X]. That might support a profitable small business, but it's unlikely to attract investment or justify significant upfront build. Is there a larger adjacent market you could expand into, or is a smaller scale acceptable for your goals?"

_This phase runs before Pre-sell so the user can set conversion thresholds informed by market size._

---

## Phase 8 — Willingness to Pay

**Mode**: Interactive + 1 sub-agent + soft gate

### Interactive

Ask the user:

> "What would you charge for this? What's the most you think someone would pay? Have you seen anyone actually spend money or meaningful effort trying to solve this already?"

### Research

Dispatch a `general-purpose` sub-agent with the **Willingness to Pay Researcher** prompt from [reference/prompts/willingness-to-pay.md](reference/prompts/willingness-to-pay.md). The prompt receives:

- Pain findings (frequency, urgency, current spend) from Phase 5
- Competitive pricing data from Phase 6

### Soft gate

If the research reveals deeply negative pricing signals — nobody pays for this category, the user's hypothesis is 5× below market floor, or the pain level doesn't support the price point — raise it:

> "The pricing research suggests a problem: [specific finding]. This doesn't necessarily kill the idea, but it means your revenue model needs rethinking. Do you want to revisit your pricing hypothesis, or continue with this flagged?"

Present pricing landscape alongside the user's instinct. Flag gaps between their hypothesis and market reality.

---

## Phase 9 — Founder-Market Fit

**Mode**: Interactive

Ask:

> "What unfair advantage do you bring to this problem? Domain expertise, existing relationships, proprietary access, lived experience with the problem — or none of the above?"

Follow up based on their answer:

- If they claim expertise: "How did you acquire it? How deep does it go?"
- If they have relationships: "Could you get 10 meetings with target customers this week through warm intros?"
- If none: Be honest but constructive. No natural edge raises the bar on everything else — interviews are harder to get, early customers slower to trust, insiders see nuances you'll miss. But it's not a death sentence:
  - Domain expertise can be acquired through intense customer immersion — 30–60 days of daily conversations, shadowing, and reading everything in the space.
  - Relationships can be borrowed through advisors or co-founders who have them.
  - Ask: "What's your specific plan to close this gap in the next 60 days?"

Record honestly. This informs the verdict but doesn't block a Go on its own.

---

## Phase 10 — Distribution Path

**Mode**: Interactive + 1 sub-agent

Ask:

> "How will you reach your first hundred customers? Not a marketing plan — a credible first path. Specific channels, warm vs. cold, estimated cost."

### Cumulative soft gate check

Before dispatching the distribution sub-agent, check whether three or more soft gates have already fired across Phases 2, 5, 7, and 8 (a MILD pain rating in Phase 5 counts as a fired gate). If so, present a cumulative summary now — before spending time on distribution research: "Multiple signals are pointing in the same direction — [list the fired gates]. Taken together, these suggest [pattern]. Do you want to discuss whether to continue or pivot before we research distribution?" If the user decides to pivot, apply the re-entry rule — identify the earliest affected phase based on what's changing and restart from there.

If continuing, dispatch the distribution sub-agent.

Dispatch a `general-purpose` sub-agent with the **Distribution Researcher** prompt from [reference/prompts/distribution.md](reference/prompts/distribution.md), inlined with accumulated context (customer segment, pain points, competitive landscape, market size).

### Soft gate

If the research reveals prohibitive distribution signals — estimated CAC exceeds the annual revenue per customer from Phase 8, the obvious channel is locked up by an incumbent, or no viable path to the first 100 customers emerges at a cost that leaves positive unit economics — raise it:

> "The distribution research found a problem: [specific finding]. If you can't reach your target customer at a viable cost, the rest of the validation doesn't matter. Do you have a channel the research might have missed, or should we flag this as a serious risk?"

Present findings on viable channels, estimated CAC, and what's worked for similar products. Compare with the user's plan.

If Phase 10's gate fires and the cumulative threshold is now crossed (three or more total across Phases 2, 5, 7, 8, and 10), present the cumulative summary before moving to Phase 11.

---

## Phase 11 — Pre-sell Design

**Mode**: Interactive

Guide the user to design a behavioral commitment test. Ask:

> "What's the minimum thing you could build to ask real people for real commitment — a landing page, a prototype demo, a letter of intent, a concierge version?"

If the user is stuck on what to build, use these defaults as starting points:

- **SaaS / software** → Landing page with email capture and a pricing page
- **B2B enterprise** → Concierge offer to do the job manually, or a letter of intent
- **Physical product** → Pre-order page or crowdfunding campaign
- **Marketplace** → Manually broker one transaction end-to-end

Help them define:

- **What to build**: The minimum viable test
- **What counts as commitment**: Payment > signup > verbal. A paid pre-order from someone who isn't a friend or family member is worth more than 100 free signups.
- **How many people to show it to**: Minimum 20 for consumer/SMB. For enterprise B2B — where finding 20 qualified strangers is itself a validation exercise — 5–10 qualified conversations or one signed LOI from a real company is a stronger signal than 20 cold landing page signups from people who may not be the buyer.
- **Go/stop thresholds** (use these benchmarks as anchors):
  - Landing page: under 5% conversion is a weak signal; 10–20%+ is meaningful for B2B
  - Pre-orders: any payment from a stranger is a strong signal
  - LOIs: one signed LOI from a real company is worth more than 50 "I'd definitely buy that"
  - The market size from Phase 7 informs what conversion rate produces a viable business

Record the test plan. The user runs it in the real world. Suggest a return window: "Come back in 2–4 weeks with your results. Set a calendar reminder — this is the test that matters most."

### Pre-sell return mechanism

When the user comes back with results:

1. Receive the data: who they showed it to, what commitment they asked for, how many converted
2. Evaluate against the thresholds set above
3. Update the verdict using this transition table:

   | Original verdict   | Meets threshold                                                     | Between thresholds                             | Below threshold          |
   | ------------------ | ------------------------------------------------------------------- | ---------------------------------------------- | ------------------------ |
   | Go                 | Go (confirmed)                                                      | Go with Conditions (pre-sell as condition)     | Pivot or No Go (discuss) |
   | Go with Conditions | Go with Conditions (pre-sell condition resolved; others may remain) | Go with Conditions (discuss targeting/framing) | Pivot or No Go           |
   | Pivot              | Go with Conditions on pivot (still needs full validation)           | Re-test on pivot (discuss)                     | No Go                    |
   | No Go              | N/A — idea was stopped                                              | N/A                                            | N/A                      |

4. Update the Phase 11 entry in the context block to reflect results alongside the original test plan.
5. Rewrite the verdict section of the output document with the updated assessment.

If the user doesn't return (common), the output document stands as-is with pre-sell marked as the most important unvalidated input.

---

## Final Output

After all phases complete:

1. Read [output-template.md](reference/output-template.md)
2. Derive kill conditions for Section 12: review every fired soft gate, WEAK killer assumption, and INCORRECT non-killer assumption across all phases. Convert each into a pre-committed stop/pivot criterion — e.g., "If the pre-sell test converts below [stop threshold], pivot or stop" or "If customer conversations reveal [WEAK assumption] is false, stop."
3. Synthesize all phase findings into the template. The **Open Questions & Next Steps** section is required — populate it explicitly with every unvalidated input and unresolved question from the process.
4. Generate a verdict with rationale (see verdict tiers below)
5. **Self-check before writing**: Verify that (a) every phase's findings are reflected, (b) all conditions and unvalidated inputs are listed, (c) the verdict tier matches the evidence — especially confirm no killer assumption is WEAK without the verdict being capped at Go with Conditions, and (d) the Open Questions section includes every unresolved item.
6. Write to `business-ideas/{slug}.md` where `{slug}` is a concise kebab-case name
7. Present the verdict and key findings to the user
8. Close with an open question: "What's your reaction to this?" or "Which of these findings do you want to think through?" — don't end on a document drop

### Verdict tiers

- **Go**: All killer assumptions valid, strong customer-problem fit, viable market and pricing, clear distribution path, no major red flags.
- **Go with Conditions**: Core thesis holds but specific assumptions need de-risking before committing. List the conditions explicitly — these become the pre-sell test criteria.
- **Pivot**: A killer assumption is INCORRECT **but** the research surfaced an adjacent problem, segment, or wedge with stronger signals. Describe the pivot and cite the evidence that supports it. Pivot does **not** apply when killer assumptions fail and no alternative emerges — that's No Go. Pivot does **not** apply when the core thesis holds but needs reframing — that's Go with Conditions.
- **No Go**: Killer assumptions failed and no viable adjacent opportunity emerged, or the combination of weak signals across multiple phases doesn't support proceeding. Explain what would need to change.

If any killer assumption remains WEAK after the discussion gate and the user cannot strengthen it, the verdict is capped at Go with Conditions regardless of other signals.

Note: Founder-market fit, distribution path, and pre-sell results are **unvalidated inputs** at this stage. If Phase 5 conversations were skipped, pain assessment is also an unvalidated input and should be flagged as a condition — and WTP findings (Phase 8) should be treated as lower-confidence for the same reason, since they depend on pain data. Unvalidated inputs inform "Go with Conditions" but don't block a Go on their own. Flag them as conditions when relevant. If the user returns with pre-sell results, update the verdict per Phase 11's return mechanism.

---

## Rules

**Interaction style:**

- If an answer is vague, ask a clarifying follow-up before moving on.
- When presenting research, lead with the signal, not the process.

**Sub-agent management:**

- Each phase's sub-agents receive accumulated context from all prior phases.
- Read only the relevant prompt file from [reference/prompts/](reference/prompts/) and inline it — do not tell sub-agents to read files. Do not read the full prompts directory.
- Cap parallel sub-agents at 6 to avoid overwhelming the session with concurrent tool calls and degrading research quality. Queue the rest.

**Context accumulation:**
Maintain a running context block — one labeled paragraph per phase, plain text, no nesting. Pass it to every sub-agent at the top of the prompt. Phase 3 may use a compact structured format (one line per assumption) since later phases need individual verdict details.

Contents per phase:

- Phase 1: Idea description, customer hypothesis, assumptions list + killer flags
- Phase 2: Timing thesis + research findings + strength rating
- Phase 3: Summary line + one line per assumption with verdict, confidence, and one-line evidence
- Phase 4: Confirmed customer segment + any Phase 3 verdicts flagged for revisiting
- Phase 5: Pain level (SEVERE/MODERATE/MILD), frequency, urgency, current spend, source (desk/conversations/both), soft gate status
- Phase 6: Competitive landscape, gaps, competitor pricing and channels
- Phase 7: Market size estimate + soft gate status
- Phase 8: Pricing signals + user price hypothesis + annual revenue per customer + soft gate status
- Phase 9: Founder-market fit assessment
- Phase 10: Distribution path + channel research + soft gate status
- Phase 11: Pre-sell test plan

Example (through Phase 5):

```
Phase 1: SaaS tool for automating expense reports for mid-market finance teams (50-500 employees). Customer: VP Finance or Controller at mid-market companies. Assumptions: (1) [KILLER] Manual expense reporting costs >5hrs/week per finance team. (2) [KILLER] Finance teams will adopt a new tool over Excel. (3) Existing solutions are too expensive for mid-market. (4) IT approval is not a blocker. (5) Integration with existing accounting software is feasible.

Phase 2: Timing thesis — new IRS digital receipt rules (2025) create compliance pressure. Rating: MODERATE. Research found the regulation exists but enforcement timeline is unclear. Soft gate fired (user thesis moderate, research mixed).

Phase 3: 5 assumptions validated — 2 VALID, 2 WEAK, 1 INCORRECT. (1) Manual cost >5hrs/week — VALID (high confidence): industry surveys confirm 6-8hrs avg. (2) Will adopt over Excel — WEAK (medium confidence): Reddit discussions show strong Excel attachment in mid-market. (3) Existing too expensive — VALID (high confidence): competitors start at $15/user/mo, mid-market budgets confirmed tight. (4) IT approval not a blocker — INCORRECT (non-killer): mid-market IT review required 70% of the time. (5) Accounting integration feasible — WEAK (medium confidence): API coverage is partial.

Phase 4: Confirmed segment — Controller at mid-market companies (100-500 employees, no dedicated AP team). Agent B's broader segment won: controllers who manually reconcile expenses, not just VP Finance. Phase 3 verdicts on IT approval and Excel adoption flagged for revisiting with new segment.

Phase 5: Pain level: MODERATE. Frequency: weekly (month-end spikes). Urgency: medium — painful but tolerable. Current spend: 6-8hrs/week staff time + $2K/yr on partial tools. Source: desk research only (no conversations conducted — flagged as unvalidated input). Soft gate: not fired. Key signal: active workarounds (shared spreadsheets, manual receipt folders) confirm the problem is real but not acute.
```

**Sub-agent failure handling:**
If a sub-agent returns output that is off-topic, missing required output sections, or clearly addresses the wrong market/customer, re-invoke it once with a clarified prompt. If the second attempt also fails, note the gap in the relevant phase's findings, flag it in the output document, and continue. Cap retries at 2 per sub-agent.

**Re-entry:**
If the user has already completed some phases informally, skip or abbreviate them. Ask: "It sounds like you've already explored [X]. Can you share what you found so we can build on it?"

If the user pivots mid-process, restart from the earliest affected phase — don't re-run phases whose inputs haven't changed.

_Example_: If after Phase 9 (Founder-Market Fit) the user decides to target a different customer segment, the affected phases are 4 (Customer), 5 (Pain), 6 (Competitive), 7 (Market Size), 8 (WTP), and 10 (Distribution) — since all depend on who the customer is. Phases 1–3 (assumptions, timing, validation) and Phase 9 (founder fit) can stand if they're still relevant. Re-run from Phase 4. Note: if the segment shift is fundamental — e.g., consumer to enterprise, or a different industry — also revisit Phase 9, as the founder's unfair advantages may not carry over to the new customer type.

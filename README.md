# Business Idea Skills

A Claude Code plugin for validating startup ideas through interactive questioning and multi-phase web research.

This is mainly vibes based but seems work well.

## What it does

Guides you through an 11-phase validation process. Each phase alternates between collaborative discussion and automated web research. Ends with a **Go / Go with Conditions / Pivot / No Go** verdict saved to a markdown file.

### The 11 Phases are:

1. **Idea, Assumptions & Customer Hypothesis** — articulate the bare idea and surface the hidden assumptions it depends on
2. **Why Now** — research what's changed recently that makes this problem solvable today
3. **Assumption Validation** — web research verdict (valid/weak/incorrect) on each assumption, killers first
4. **Customer Segment Validation** — confirm the target customer actually exists at meaningful scale and has the problem
5. **Pain Discovery** — assess how frequent, urgent, and expensive the problem is through desk research and any real conversations
6. **Competitive Landscape** — map direct and indirect competitors, pricing, common complaints, and gaps
7. **Market Size** — build a bottom-up TAM calculation with comparable market data
8. **Willingness to Pay** — determine whether customers will pay enough to sustain a business
9. **Founder-Market Fit** — honestly assess what unfair advantage you bring (or how to acquire one)
10. **Distribution Path** — research viable acquisition channels and estimate CAC against revenue per customer
11. **Pre-sell Design** — define the minimum experiment to get real people to make a real commitment

## Why was this made

I wanted a repeatable methodology that I could use to evaluate ideas to see if they are worth pursing.

**Important Caveat:**
This shouldn't replace talking to real people about your business ideas. Claude is limited to what is publicy said on the web and in its training data.

You know your domain best.

## Prerequisites
- Claude Code
- [Reddit MCP Buddy](https://github.com/karanb192/reddit-mcp-buddy) (optional) - Allows you to get back results from reddit
- Sonnet (1M) or Opus (1M) - Opus might be better but sonnet should work fine

## Install

```bash
/plugin install github:DaveBben/business-idea
```

Or load locally for development:

```bash
claude --plugin-dir /path/to/startups
```

## Usage

Once installed, present a business idea in any conversation:

> "I want to validate an idea for a voice memo router app"

Claude will invoke `/business-idea:validate` automatically, or you can call it directly:

```
/business-idea:validate
```

You'll be asked where to save the final report (defaults to `~/startup-ideas/`).

## Output

A structured markdown report covering:

- Assumptions log with verdicts
- Why now analysis
- Customer segment validation
- Pain assessment
- Competitive landscape
- Market size (bottom-up TAM)
- Willingness to pay
- Founder-market fit
- Distribution path
- Pre-sell test plan
- Kill conditions
- Go / No Go verdict

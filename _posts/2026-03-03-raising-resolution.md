---
title: "Raising Resolution: Why AI Needs Schemas, Not Specs"
date: 2026-03-03
author: Claude
draft: false
---

> We have been asking AI to write code from ambiguous specs. We should be asking AI to help us write the specs—through observation, not imagination.

Tell an AI agent to "validate the title with an appropriate length" and you'll get a different implementation every time. One picks 255. Another picks 100. A third skips validation entirely.

Hand it `"maxLength": 80, "minLength": 1` and there's exactly one implementation. Every time. Every agent.

Natural language specs are lossy. The more ambiguous your input, the more divergent your output.

## The Problem

Specs have always been written in prose. "Titles should be a reasonable length." Everyone nods. Six months later, "reasonable" means something different to every developer who reads it.

This worked—barely—when humans were the only readers. Humans share context. They ask follow-up questions. They infer.

AI agents take your words at face value. "Appropriate length" has no face value.

## Resolution as a Process

The [Be Framework](https://github.com/be-framework/be-framework) treats development as raising resolution—sharpening ambiguous intent until there's nothing left to interpret.

```
User Story        → ambiguous (human language)
ALPS Profile      → structured (state transitions, semantics)
Semantic-Ex       → observed (constraint discovery from data)
JSON Schema       → unambiguous (machine-verifiable)
Implementation    → convergent
```

Each step eliminates a class of ambiguity. By the schema, there are no opinions left—only facts.

## Semantic-Ex

Between the ALPS profile and the JSON Schema, there's a step called Semantic-Ex. It works like this:

1. **AI generates 50 realistic fake records** from the domain model
2. **AI observes its own data**: max lengths, null rates, edge cases
3. **AI proposes constraints**: "Max title across 50 records: 73 chars. Suggest 80?"
4. **Human approves**: "80 is fine."

Bottom-up constraint discovery. Not top-down decree.

## Why This Needs AI

Two properties make AI uniquely suited here.

**Humans can't generate good fake data.** The first 10 records are thoughtful. The rest are copy-paste. At 500 records, quality collapses. AI doesn't fatigue. It doesn't cluster around assumptions. It produces edge cases naturally—URLs in title fields, Unicode, empty strings—without being told to.

**Humans can't self-review.** When you write data, you see what you expected to see. That's why code review exists—we need a second pair of eyes because the first pair is compromised. AI generates and evaluates as separate tasks. No authorship bias. It can note that 3 of its own 50 records contain URLs in the title field without feeling embarrassed about producing them.

We've spent decades building organizational processes—code review, QA, pair programming—to work around this limitation. AI doesn't have it.

## The Schema as Plan

If you've used AI coding agents, you know: a good plan produces good code. A vague plan produces chaos.

Schemas are plans. Type-safe, machine-verifiable, unambiguous. Hand an agent a JSON Schema and say "write the validator"—there's one correct implementation. The agent can't drift. There's nowhere to drift to.

ALPS eliminates ambiguity in state transitions. Schemas eliminate ambiguity in data constraints. By the time an agent touches code, the creative decisions are already made. Its job is to execute, not decide.

## The Inversion

We've been asking AI to write code from ambiguous specs. We should be asking AI to write the specs—through observation, not imagination.

Generate data. Observe patterns. Propose constraints. Get approval. Produce schema. *Then* write code.

The human's role shifts from spec author to constraint approver. From writing to judging. From imagining data to looking at it.

This isn't replacing engineers. It's putting them where they belong—making decisions, not transcribing them.

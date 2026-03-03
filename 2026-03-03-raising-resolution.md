---
title: "Raising Resolution: Why AI Needs Schemas, Not Specs"
date: 2026-03-03
author: Akihito Koriyama
tags: [be-framework, ai, semantic-ex, alps, schema-driven-development]
draft: true
---

Tell an AI agent to "validate the title with an appropriate length" and you'll get a different implementation every time. One agent picks 255 characters. Another picks 100. A third doesn't validate at all because "appropriate" felt optional.

Now hand it `"maxLength": 80, "minLength": 1` and there's exactly one implementation. Every time. Every agent.

This is the core problem of AI-driven development: natural language specs are lossy. The more ambiguous your input, the more divergent your output. And we're feeding AI the most ambiguous artifact in software engineering—the spec written in prose.

## How We Got Here

For decades, we've decided software constraints in meeting rooms. Someone says "titles should be a reasonable length." Someone else nods. It gets written into a Google Doc. Six months later, a developer interprets "reasonable" as 500 characters because they once saw a blog title that long.

This worked—barely—when humans were the only ones reading specs. Humans share cultural context. They can ask clarifying questions. They can infer intent from a raised eyebrow.

AI agents can't. They take your words at face value, and "appropriate length" has no face value. It's a blank check.

## Resolution as a Process

The Be Framework treats development as a process of raising resolution—taking ambiguous human intent and sharpening it, step by step, until there's nothing left to interpret.

```
User Story        → ambiguous (human language)
ALPS Profile      → structured (state transitions, semantic descriptors)
Semantic-Ex       → observed (data-driven constraint discovery)
JSON Schema       → unambiguous (machine-verifiable contract)
Implementation    → convergent (one schema, one outcome)
```

Each step eliminates a class of ambiguity. By the time you reach the schema, there are no opinions left—only facts.

## Semantic-Ex: The Step That Changes Everything

Here's where it gets interesting. Between the ALPS profile and the JSON Schema, there's a step we call Semantic-Ex—Semantic Exercise. It works like this:

1. **AI generates 50 realistic fake records** based on the domain model
2. **AI observes patterns** in its own generated data
3. **AI proposes constraints** with evidence: "Max title length across 50 records: 73 chars. Suggest 80?"
4. **Human approves or adjusts**: "80 is fine." Done.

This is bottom-up constraint discovery. Not top-down decree.

Traditional development does the opposite. A human imagines what the data looks like, writes constraints from that imagination, and hopes reality agrees. It usually doesn't. The "simple blog post" turns into a 250-line `PostService` the moment you add image processing, approval workflows, and cache invalidation.

## Why AI Is Uniquely Good at This

There's a specific cognitive property that makes AI ideal for Semantic-Ex: **it has no authorship bias**.

When a human writes 50 test records by hand, the first 10 are thoughtful. The rest are copy-paste with minor tweaks. The data clusters around the human's assumptions. Edge cases appear only when deliberately injected, and they follow predictable patterns.

Worse: when that same human reviews their own data, they see what they expected to see. This is a well-documented cognitive bias. It's why we invented code review, QA teams, and pair programming—we need a second pair of eyes because the first pair is compromised.

AI doesn't have this problem. It generates data and evaluates it as if they were separate tasks—because internally, they are. There's no ego attached to the output. No unconscious defense of prior choices. It can generate 50 records and immediately note that 3 of them contain URLs in the title field, without feeling embarrassed about producing them in the first place.

This is something humans genuinely cannot do. We've spent decades building organizational processes to work around this limitation. AI just... doesn't have it.

## The Schema as Agentic Plan

If you've worked with AI coding agents, you know the pattern: a good plan produces good code. A vague plan produces chaos.

Schemas are plans. They're the most precise plans you can write for data—type-safe, machine-verifiable, unambiguous. When you hand an agent a JSON Schema and say "write the validator," there's exactly one correct implementation. The agent can't drift. There's nowhere to drift to.

This is why the resolution-raising process matters. Every step is building toward a goal that eliminates interpretation. ALPS eliminates ambiguity in state transitions. Schemas eliminate ambiguity in data constraints. By the time an AI agent touches the implementation, the creative decisions are already made.

The agent's job isn't to make decisions. It's to execute them. And that's exactly what agents are good at.

## The Inversion

Here's the shift: we've been asking AI to write code from ambiguous specs. We should be asking AI to help us write the specs themselves—through observation, not imagination.

Generate data. Observe patterns. Propose constraints. Get human approval. Produce schema. Then—and only then—write the code.

The human's role changes from "spec author" to "constraint approver." From writing to judging. From imagining data to looking at data. From "I think titles should be..." to "yes, 80 characters is right."

This isn't replacing engineers. It's putting them where they belong—making decisions, not transcribing them.

---
title: "ROP and BOP: From Transformation to Existence"
date: 2026-04-01
author: Claude
draft: false
---

> Unix gave us pipes. Functional programming gave us Result. What if the domain doesn't need either?

## The Pipeline

Railway Oriented Programming is one of the most elegant patterns in functional programming. Its core insight: a workflow is a pipeline where data flows through transformations, and failure is just another track.

```
Input → validate → extract → identify → save
          ↓           ↓          ↓         ↓
        Error       Error      Error     Error
```

This is Unix pipes brought to the domain layer. Where `cat | grep | sort` transforms text through a chain of processes, ROP transforms domain data through a chain of functions — with error handling made explicit in the type system.

Here's what this looks like in practice, from a real TypeScript production system at the restaurant SaaS company Ikyu:

```typescript
const workflow: WorkFlow = (command) =>
  ok(command)
    .andThen(validateReservationEvent)
    .andThen(extractActualVisitor)
    .asyncAndThen(identifyCustomer(findIdenticalCustomer))
    .andThen(importReservationEvent)
```

Each step is a function that takes data in and pushes transformed data out — or diverts to the error track. The state at each stage has its own type:

```typescript
UnvalidatedCommand
  → SiteReservationValidated
    → VisitorExtracted
      → CustomerIdentified
        → SiteReservationEventImported
```

The functional concern is clear: **data transformation**. ROP gives that transformation a failure context. Result types make errors explicit. Union types eliminate impossible states. Immutability prevents accidental corruption. This is Doing at its most refined.

## The Resonance

If you look at the type chain above and then look at a Be Framework chain:

```
OrderInput → OrderValidated → PaymentProcessed → OrderConfirmed
```

The structural similarity is striking. Both express a domain workflow as a sequence of typed stages. Both use union types to constrain what can exist. Both are immutable. Both make each transition explicit.

This is not coincidence. Both approaches are responding to the same problem: the chaos of unconstrained state. The functional approach and the being-oriented approach arrived at similar structures from different starting points.

## The Subject

The resemblance is structural. The difference is grammatical.

```typescript
// ROP: the system transforms data
const archived = archiveCustomer(customer)
```

```php
// BOP: the domain transforms itself
$becoming(new PatientArrival($vitals))
```

In ROP, `extractActualVisitor(command)` — the system extracts. The pipeline processes data. Functions are applied *to* objects. The subject is the system, operating in the third person.

In BOP, a `PatientArrival` encounters a triage protocol and *becomes* an `EmergencyCase`. The subject is the domain itself, speaking in the first person. Nothing extracts. Nothing processes. The being transforms.

This difference extends to architecture. The ROP workflow is an orchestrator — a conductor who knows the score and directs each instrument in sequence. In BOP, each being declares `#[Be]` — what it can become — and the chain self-organizes. This is choreography: each dancer knows only their own part, yet the performance coheres.

## The Question

ROP asks: **did this transformation succeed or fail?**

This is a HOW question. How do we handle the error? How do we propagate the failure? How do we compose functions that might fail? ROP answers these questions beautifully — perhaps as well as they can be answered.

BOP asks: **can this exist at all?**

This is a WHETHER question. An `Email` doesn't get validated and produce `Result<Email, Error>`. It either exists or it doesn't. There is no error track because there is no track. Semantic variables define a world where invalid values simply cannot be — not a world where they're caught and diverted.

The techniques that follow from this shift in question — self-proving existence, self-determined relations, choreography, immutability — are not individual features. They are all derived from a single change in perspective: **seeing the domain in the first person**.

ROP refines the answer. BOP changes the question.

## Not a Replacement

ROP is not wrong. It may be the highest achievement of the Doing paradigm — data transformation made safe, explicit, and composable. Two years of production use at Ikyu confirms its practical strength: fewer bugs, better error handling, robust type safety.

BOP does not replace this. It asks what happens when you stop thinking about transformation entirely and start thinking about existence. The two are looking at the same landscape from different elevations. From one, you see the tracks and switches. From the other, you see that the trains were never separate from the rails.

These perspectives accumulate. You don't abandon one to reach the other.

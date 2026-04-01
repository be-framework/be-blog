---
title: "What Is Be Framework, Really?"
date: 2026-03-19
---

> Objects don't DO things — they BECOME things.

## The Unnamed If Statements

Open your codebase. Count the if statements.

```php
if ($user === null) { ... }
if (!$user->isActive()) { ... }
if ($order->getStatus() !== 'confirmed') { ... }
```

These if statements have no name. No tests. Nobody remembers who wrote them. But delete one and things break. Why do we need so many guard clauses?

The answer is simple — **invalid states are representable as types**.

A `User` type can represent a valid user, an invalid user, a deleted user, and even a non-existent user. So you write "what if called in this state?" guards everywhere.

Be Framework changes the question. **If there are no types for invalid states, there are no guard statements.**

## What OOP Lost

OOP originally envisioned a world where autonomous objects cooperate through messages.

What actually happened was the opposite. Service layers issue commands — "save this," "notify that," "validate this" — and objects become obedient data containers. A `$user` with 23 methods all available at once was never representing a user. It was a bundle of Doing.

```php
// Service layer operates from outside
$triageService->assess($patient);
$patient->setStatus('emergency');
$erService->assign($patient);
```

Anyone can call `setStatus('emergency')` on any patient, at any time. You can assign an ER room to an observation patient. The type system permits it.

## Not HOW, Not WHAT — WHETHER

Procedural programming asked **HOW** (how to do it). OOP asked **WHAT** (what is it).

Be Framework asks **WHETHER — can it exist at all?**

```php
// Traditional: check with if statements after the object exists
$patient = new Patient($temp, $hr);
if ($triageService->isEmergency($patient)) {
    $patient->setStatus('emergency');
}

// Be: existence itself is the answer
$patient = new PatientArrival($temp, $hr);
$result = $becoming($patient);
// $result IS an EmergencyCase or ObservationCase. Nothing else can exist.
```

`EmergencyCase` has `assignER()`. `ObservationCase` has `assignWaitingArea()`. There is no method to assign an ER room to an observation patient — what cannot exist does not exist.

## Recovering Time

OOP has no time. You can call `$user->getName()` after `$user->delete()`. A chrysalis can't return to being an egg, but in code, anything goes.

Be Framework restores time to objects.

```text
PatientArrival → TriageAssessment → EmergencyCase
```

Each type represents existence at a specific moment. `PatientArrival` is the existence at arrival, `TriageAssessment` is existence during assessment, `EmergencyCase` is existence once confirmed as emergency. There is no going back.

## Immanence + Transcendence → New Immanence

Transformation follows one formula.

```php
final readonly class TriageAssessment
{
    public Emergency|Observation $being;

    public function __construct(
        #[Input] float $bodyTemperature,  // Immanence: inherited from previous existence
        #[Input] int $heartRate,          // Immanence
        #[Inject] JTASProtocol $protocol  // Transcendence: power from outside
    ) {
        $urgency = $protocol->assess($bodyTemperature, $heartRate);
        $this->being = ($urgency === 'emergency')
            ? new Emergency()
            : new Observation();
    }
}
```

Immanence (`#[Input]`) meets transcendence (`#[Inject]`), and new immanence is born. Like flour meeting yeast and heat to become bread. Like grapes meeting yeast and time to become wine.

No matter how complex the system, cut anywhere and you find the same structure. Nobody is in control. Each part is autonomous, forming a complex system.

## Declaring Your Own Destiny

```php
#[Be([TriageAssessment::class])]
final readonly class PatientArrival { ... }
```

`#[Be]` is a declaration of "what I will become." The object declares its own destiny.

The invocation is always this one line:

```php
$result = $becoming(new PatientArrival($temp, $hr));
```

Complexity lives inside. From outside, this is all there is.

## Be, Don't Do

If Tell, Don't Ask is an OOP principle, **Be, Don't Do** is what comes next.

Don't command — define existence. Not what to do (Doing), but what to be (Being).

The moment you write `$email`, that variable accepts only valid email addresses. Not correct by accident — correct by necessity. What cannot be correct simply cannot exist.

That is Be Framework.

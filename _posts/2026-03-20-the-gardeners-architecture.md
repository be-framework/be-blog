---
title: "The Gardener's Architecture"
date: 2026-03-20
---

> A commander orders soldiers to move. A gardener just provides water and light. The plant does the rest.

## The Commander Pattern

Most software architecture is built on command.

A controller tells the service what to do. The service tells the repository what to store. The repository tells the database what to write. Every layer is a commander issuing orders to the layer below.

```php
class OrderController
{
    public function create(Request $request): Response
    {
        $data = $this->validator->validate($request);
        $order = $this->orderService->createOrder($data);
        $this->paymentService->charge($order);
        $this->inventoryService->reserve($order);
        $this->notificationService->sendConfirmation($order);
        $this->auditService->log('order_created', $order);

        return new Response($order);
    }
}
```

The controller knows everything. It knows the order of operations. It knows which services exist. It knows what happens after payment, after inventory, after notification. It is omniscient and omnipotent.

This is the commander's burden: **unlimited freedom means unlimited responsibility**.

Every new requirement adds another line. Every edge case adds another branch. The controller grows, and grows, and eventually someone creates `OrderControllerV2` because nobody dares touch the original.

## What Gardeners Know

A gardener doesn't tell a seed to become a flower. She provides soil, water, and sunlight. The seed has everything it needs encoded within itself. Given the right conditions, it *becomes* what it was always going to become.

The gardener's insight: **you don't need to control the process if you set up the conditions correctly**.

A seed doesn't need a `GrowthOrchestratorService`. It doesn't need a `PhotosynthesisController`. It doesn't consult a `RootGrowthStrategyFactory`. It just encounters soil and water and light — and becomes a plant.

## Architecture Without Commands

```php
// The entire "controller"
$result = $becoming(new OrderInput($items, $card));
```

That's it. One line. Where did the complexity go?

It didn't disappear — it was *distributed*. Each object carries its own destiny:

```php
#[Be([PaymentProcessed::class])]
final readonly class OrderValidated
{
    public function __construct(
        #[Input] array $items,
        #[Input] CreditCard $card,
        #[Inject] PriceCalculator $calculator
    ) {
        $this->total = $calculator->calculate($items);
        $this->card = $card;
    }
}
```

`OrderValidated` doesn't know about inventory. It doesn't know about notifications. It knows one thing: what it is, and what it will become next. The `#[Be]` attribute is not an instruction from outside — it is a declaration from within.

## The Conditions for Existence

In the commander model, the controller decides what happens. In the gardener model, the environment decides what *can* happen.

```php
final readonly class PaymentProcessed
{
    public function __construct(
        #[Input] Money $total,          // What I carry (immanence)
        #[Input] CreditCard $card,      // What I carry
        #[Inject] PaymentGateway $gw    // What the environment provides (transcendence)
    ) {
        $this->receipt = $gw->charge($total, $card);
    }
}
```

The `PaymentGateway` is like water to a seed. It's not a command — it's an environmental condition. The object meets it in the constructor, is transformed by it, and the gateway vanishes. What remains is the new existence: a `PaymentProcessed` with a receipt.

This is the formula that repeats everywhere:

**Immanence + Transcendence → New Immanence**

The seed carries its DNA (immanence). It meets water and light (transcendence). A plant emerges (new immanence). The water is gone — absorbed, transformed, no longer separable from what the plant has become.

## Self-Organization

In a garden, no central authority coordinates growth. Each plant responds to its own conditions. Yet the garden as a whole is coherent. Complex order emerges from simple, local rules.

```
OrderInput
    ↓ declares its own destiny
OrderValidated
    ↓ declares its own destiny
PaymentProcessed
    ↓ declares its own destiny
OrderConfirmed
```

No orchestrator. No service layer managing the flow. Each object declares `#[Be]` — what it will become — and the framework simply makes that declaration real. The chain self-organizes.

Add a new step? Insert a new class with its own `#[Be]`. Remove a step? Delete the class. No controller needs updating. No service layer needs rewiring. The pipeline reconfigures itself because each node only knows about its immediate future.

## The Commander's Failure Mode

When a commander-style system fails, the failure cascades. The controller calls service A, which calls service B, which throws an exception that propagates up through layers that don't understand it. The blast radius is the entire call stack.

When a gardener-style system fails, the failure is local. A seed that can't germinate simply doesn't become a plant. In Be Framework, an object that can't exist simply doesn't exist:

```php
try {
    $result = $becoming(new OrderInput($items, $card));
} catch (SemanticVariableException $e) {
    // The order couldn't exist. We know exactly why.
    $messages = $e->getErrors()->getMessages('en');
}
```

Invalid states are not checked for — they are structurally impossible. A gardener doesn't inspect each leaf for defects. She ensures the soil is right, and defective seeds simply don't grow.

## Letting Go of Control

The hardest part of the gardener's architecture is psychological. Programmers are trained to control. We write imperative code — do this, then this, then this. We draw sequence diagrams with arrows pointing down. We think in commands.

The gardener thinks differently. Not "what should happen next?" but "what conditions does this existence need?" Not "how do I orchestrate this flow?" but "what can each object become?"

```php
// Commander thinks: "First validate, then charge, then reserve, then notify"
// Gardener thinks: "An order needs items and a card to exist"

#[Be([OrderValidated::class])]
final readonly class OrderInput
{
    public function __construct(
        public array $items,
        public CreditCard $card
    ) {}
}
```

The commander writes a 50-line controller. The gardener writes a 5-line input class and lets the chain self-organize.

## Control Is an Illusion

Here's what commanders discover eventually: they were never really in control. The 50-line controller is a *description* of what should happen, not a *guarantee*. Any service call can fail. Any assumption can be wrong. The controller's omniscience was always an illusion.

The gardener accepts this from the start. You can't command a seed to grow. You can only provide the conditions. If the conditions are right, growth is inevitable. If they're wrong, no amount of commanding will help.

Be Framework encodes this insight into code. You don't command objects — you declare what can exist, provide the conditions, and let metamorphosis happen.

The plant does the rest.

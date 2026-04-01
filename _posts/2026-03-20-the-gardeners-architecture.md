---
title: "The Gardener's Architecture"
date: 2026-03-20
---

> A commander orders soldiers to move. A gardener provides water and light. The plant does the rest.

## The Commander Pattern

Most software architecture is built on command.

A controller tells the service what to do. The service tells the repository what to store. The repository tells the database what to write. Every layer issues orders to the layer below.

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

The controller knows the order of operations, which services exist, what happens after payment, after inventory, after notification. It knows everything about everyone.

Every new requirement adds another line. Every edge case adds another branch. The controller grows, and eventually someone creates `OrderControllerV2` because nobody dares touch the original.

## What Gardeners Know

A gardener doesn't tell a seed to become a flower. She provides soil, water, and sunlight. The seed has everything it needs encoded within itself. Given the right conditions, it *becomes* what it was always going to become.

A seed doesn't need a `GrowthOrchestratorService`. It just encounters soil and water and light — and becomes a plant.

## Architecture Without Commands

```php
// The entire "controller"
$result = $becoming(new OrderInput($items, $card));
```

One line. The complexity didn't disappear — it was *distributed*. Each object carries its own destiny:

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

`OrderValidated` knows one thing: what it is, and what it will become next. The `#[Be]` attribute is not an instruction from outside — it is a declaration from within.

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

The `PaymentGateway` is like water to a seed. The object meets it in the constructor, is transformed by it, and the gateway vanishes. What remains is a `PaymentProcessed` with a receipt.

**Immanence + Transcendence → New Immanence**

The seed carries its DNA. It meets water and light. A plant emerges. The water is gone — absorbed into what the plant has become.

## Self-Organization

In a garden, no central authority coordinates growth. Each plant responds to its own conditions. Yet the garden as a whole is coherent.

```text
OrderInput
    ↓ declares its own destiny
OrderValidated
    ↓ declares its own destiny
PaymentProcessed
    ↓ declares its own destiny
OrderConfirmed
```

Each object declares `#[Be]` — what it will become — and the chain self-organizes.

Add a new step? Insert a new class. Remove a step? Delete the class. No controller needs updating. Each node only knows about its immediate future.

## Failure

When a commander-style system fails, the failure cascades. The controller calls service A, which calls service B, which throws an exception that propagates up through layers that don't understand it.

When a gardener-style system fails, the failure is local. A seed that can't germinate doesn't become a plant. An object that can't exist doesn't exist:

```php
try {
    $result = $becoming(new OrderInput($items, $card));
} catch (SemanticVariableException $e) {
    // The order couldn't exist. We know exactly why.
    $messages = $e->getErrors()->getMessages('en');
}
```

Invalid states are not checked for. They are structurally impossible.

## Letting Go

The hardest part is psychological. Programmers are trained to control. We write imperative code — do this, then this, then this. We think in commands.

```php
// Commander: "First validate, then charge, then reserve, then notify"
// Gardener: "An order needs items and a card to exist"

#[Be([OrderValidated::class])]
final readonly class OrderInput
{
    public function __construct(
        public array $items,
        public CreditCard $card
    ) {}
}
```

The 50-line controller is a *description* of what should happen, not a *guarantee*. Any service call can fail. Any assumption can be wrong.

You can't command a seed to grow. You can only provide the conditions.

The plant does the rest.

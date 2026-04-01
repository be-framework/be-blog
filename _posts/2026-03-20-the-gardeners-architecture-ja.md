---
title: "庭師のアーキテクチャ"
date: 2026-03-20
lang: ja
---

> 司令官は兵士に動けと命じる。庭師は水と光を与えるだけ。あとは植物がやる。

## 司令官パターン

ソフトウェアアーキテクチャの大半は命令で成り立っています。

コントローラーがサービスに指示を出します。サービスがリポジトリに保存を命じます。リポジトリがデータベースに書き込みを命じます。どの層も、下の層に命令を出している。

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

コントローラーは操作の順序を知っています。どのサービスが存在するかを知っています。決済の後、在庫の後、通知の後に何が起きるかを知っています。全員のことを知っている。

新しい要件が増えるたびに1行増えます。エッジケースが増えるたびに分岐が増えます。コントローラーは膨れ続け、いつか誰かが `OrderControllerV2` を作ります。元のコードには誰も触れない。

## 庭師が知っていること

庭師は種に「花になれ」とは言いません。土と水と光を用意します。種は自分の中に必要なものをすべて持っています。条件が整えば、なるべきものに*なる*。

種に `GrowthOrchestratorService` は要りません。土と水と光に出会えば——植物になります。

## 命令のないアーキテクチャ

```php
// "コントローラー"の全体
$result = $becoming(new OrderInput($items, $card));
```

1行です。複雑さが消えたわけではありません——*分散した*のです。各オブジェクトが自分の運命を持っています：

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

`OrderValidated` が知っているのは一つだけです。自分が何であり、次に何になるか。`#[Be]` は外からの指示ではありません。内側からの宣言です。

## 存在の条件

司令官モデルでは、コントローラーが何が起きるかを決めます。庭師モデルでは、環境が何が*起きうるか*を決めます。

```php
final readonly class PaymentProcessed
{
    public function __construct(
        #[Input] Money $total,          // 自分が持っているもの（内在）
        #[Input] CreditCard $card,      // 自分が持っているもの
        #[Inject] PaymentGateway $gw    // 環境が与えるもの（超越）
    ) {
        $this->receipt = $gw->charge($total, $card);
    }
}
```

`PaymentGateway` は種にとっての水のようなものです。オブジェクトはコンストラクタでそれと出会い、変容し、ゲートウェイは消えます。残るのは `PaymentProcessed` とそのレシート。

**内在 + 超越 → 新たな内在**

種はDNAを持っています。水と光に出会います。植物が生まれます。水は消えた——植物が成ったものの中に吸収されて。

## 自己組織化

庭には成長を調整する中央の権威がありません。それぞれの植物が自分の条件に応答します。それでも庭全体は調和している。

```text
OrderInput
    ↓ 自分の運命を宣言する
OrderValidated
    ↓ 自分の運命を宣言する
PaymentProcessed
    ↓ 自分の運命を宣言する
OrderConfirmed
```

各オブジェクトが `#[Be]` を宣言し、連鎖が自己組織化します。

新しいステップを追加するならクラスを挿入します。ステップを削除するならクラスを消します。コントローラーの更新は要りません。各ノードは自分の直後の未来しか知らない。

## 失敗

司令官型のシステムが失敗すると、失敗が伝播します。コントローラーがサービスAを呼び、サービスAがサービスBを呼び、サービスBが例外を投げ、それを理解しない層を遡っていく。

庭師型のシステムが失敗すると、失敗は局所的です。発芽できない種は植物にならない。存在できないオブジェクトは存在しない：

```php
try {
    $result = $becoming(new OrderInput($items, $card));
} catch (SemanticVariableException $e) {
    // この注文は存在できなかった。理由は正確にわかる。
    $messages = $e->getErrors()->getMessages('ja');
}
```

不正な状態はチェックしません。構造的に不可能だからです。

## 手放す

一番難しいのは心理的な部分です。プログラマーにとって制御は自然なことです。命令型のコードを書きます——これをやれ、次にこれ、次にこれ。思考が命令になっている。

```php
// 司令官：「まず検証、次に決済、次に在庫確保、次に通知」
// 庭師：「注文が存在するにはアイテムとカードが要る」

#[Be([OrderValidated::class])]
final readonly class OrderInput
{
    public function __construct(
        public array $items,
        public CreditCard $card
    ) {}
}
```

50行のコントローラーは、起きるべきことの*記述*であって*保証*ではありません。どのサービス呼び出しも失敗しうる。どの前提も間違いうる。

種に成長を命じることはできません。条件を整えることしかできない。

あとは植物がやる。

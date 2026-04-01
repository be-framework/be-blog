---
title: "ずばり Be Framework とは何か"
date: 2026-03-19
lang: ja
---

> Objects don't DO things — they BECOME things.

## 名もなきif文の正体

あなたのコードベースを開いてみてほしい。いくつのif文があるだろうか。

```php
if ($user === null) { ... }
if (!$user->isActive()) { ... }
if ($order->getStatus() !== 'confirmed') { ... }
```

これらのif文には名前がない。テストもない。誰が書いたかも覚えていない。しかし消すと壊れる。なぜこれほど多くのガード文が必要なのか？

答えは単純だ——**不正な状態が型として表現可能だから**。

`User`型は、有効なユーザーも無効なユーザーも、削除済みのユーザーも、存在しないユーザーすらも表現できてしまう。だから「この状態で呼ばれたら？」というif文を至る所に書くことになる。

Be Frameworkは問いを変える。**不正な状態を表す型が存在しなければ、ガード文は要らない。**

## OOPが失ったもの

OOPは本来、自律したオブジェクトがメッセージで協調する世界を目指した。

しかし実際に起きたのは逆だった。サービス層が「保存しろ」「通知しろ」「検証しろ」と命令を出し、オブジェクトは従順なデータの入れ物になった。23個のメソッドが常に同時に利用可能な`$user`は、ユーザーを表してなどいなかった——Doingの束だった。

```php
// サービス層が外から操作する
$triageService->assess($patient);
$patient->setStatus('emergency');
$erService->assign($patient);
```

誰でも、いつでも、どんな患者にも`setStatus('emergency')`を呼べる。経過観察の患者に救急室を割り当てることもできる。型はそれを許してしまう。

## HOWでもWHATでもなく、WHETHER

手続き型は **HOW**（どうやるか）を問うた。OOPは **WHAT**（それは何か）を問うた。

Be Frameworkが問うのは **WHETHER——そもそも存在できるか**。

```php
// 従来：存在した後にif文で検査する
$patient = new Patient($temp, $hr);
if ($triageService->isEmergency($patient)) {
    $patient->setStatus('emergency');
}

// Be：存在そのものが答えになる
$patient = new PatientArrival($temp, $hr);
$result = $becoming($patient);
// $result は EmergencyCase か ObservationCase。それ以外は存在しない。
```

`EmergencyCase`には`assignER()`がある。`ObservationCase`には`assignWaitingArea()`がある。経過観察の患者に救急室を割り当てるメソッドは存在しない——存在できないものは存在しないのだ。

## 時間の回復

OOPには時間がない。`$user->delete()`の後に`$user->getName()`が呼べる。さなぎは卵に戻れないのに、コードの中では何でもあり得る。

Be Frameworkはオブジェクトに時間を取り戻す。

```
PatientArrival → TriageAssessment → EmergencyCase
```

各型はある瞬間の存在を表す。`PatientArrival`は到着時の存在、`TriageAssessment`は判定中の存在、`EmergencyCase`は緊急と確定した存在。流れは一方向で不可逆。過去に戻ることはできない。

## 内在 + 超越 → 新しい内在

全ての変容は一つの公式に従う。

```php
final readonly class TriageAssessment
{
    public Emergency|Observation $being;

    public function __construct(
        #[Input] float $bodyTemperature,  // 内在：前の存在から受け継ぐもの
        #[Input] int $heartRate,          // 内在
        #[Inject] JTASProtocol $protocol  // 超越：外から与えられる力
    ) {
        $urgency = $protocol->assess($bodyTemperature, $heartRate);
        $this->being = ($urgency === 'emergency')
            ? new Emergency()
            : new Observation();
    }
}
```

内在（`#[Input]`）が超越（`#[Inject]`）と出会い、新しい内在が生まれる。超越は内在を変え、自らは消えていく。小麦粉がイーストと熱に出会ってパンになるように。ぶどうが酵母と時間に出会ってワインになるように。

どれほど複雑なシステムでも、どこを切り取っても同じ構造。誰もコントロールしていない。それぞれが自律し、複雑な系を形成する。

## 自らの運命を宣言する

```php
#[Be([TriageAssessment::class])]
final readonly class PatientArrival { ... }
```

`#[Be]`は「私は何になるか」の宣言だ。外部のコントローラーやオーケストレーターが振り分けるのではなく、オブジェクト自身が運命を宣言する。

呼び出しは常にこの1行：

```php
$result = $becoming(new PatientArrival($temp, $hr));
```

複雑さは中にある。外から見ると、これだけ。

## Be, Don't Do

Tell, Don't Ask がOOPの原則なら、**Be, Don't Do** はその先にある。

命じるのではなく、存在を定義する。開発者は実装する人ではなく、存在の条件を定義する人。何をするか（Doing）ではなく、何であるか（Being）。

`$email`と書いた瞬間に、その変数は正しいメールアドレスしか受け入れない。たまたま正しいのではなく、必然的に正しい。正しくないものはそもそも存在できない。

これがBe Frameworkだ。

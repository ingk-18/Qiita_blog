# SOLID原則とは？

**オブジェクト指向における設計原則のこと**

以下の５つの原則の頭文字をとったものです。

1. 単一責任の原則（**S**ingle Responsibility Principle）
2. オープン・クローズドの原則（**O**pen-Closed Principle）
3. リスコフの置換原則（**L**iskov Substitution Principle）
4. インターフェース分離の原則（**I**nterface Segregation Principle）
5. 依存性逆転の原則（**D**ependency Inversion Principle）

今回は全編として S,O,L の3原則をご紹介します！

## 1. 単一責任の原則（**S**ingle Responsibility Principle）

**クラスは一つのことだけ行うべきである。**

という考え方に基づいた原則になります。

仮にひとつのクラスに様々な機能を持たせた場合、
たくさんの場所で呼び出して使用してた場合に、
クラスの修正作業が発生した場合に複数機能への影響範囲のチェックに時間が必要です。

ひとつのクラスには一つの責務として、小さく定義して使うことで、
**再利用性、保守性**が向上します！

### BAD

```php
class Payment
{
    public function orderByCash($price){}
    public function cancelByCash($price){}

    public function orderByCreditCard($price){}
    public function cancelByCreditCard($price){}
}
```

こちらの Payment クラスは原則適用前で、

ひとつのクラスに現金、クレジットの複数の機能が設定されています。

こちらを原則適用してみたいと思います！

### GOOD


```php
class Payment {
    private $paymentType;

    public function __construct($paymentType) {
        $this->paymentType = $paymentType;
    }

    public function order($price) {
        $this->paymentType->order($price);
    }

    public function cancel($price) {
        $this->paymentType->cancel($price);
    }
}


class Cash {
    public function order($price){}
    public function cancel($price){}
}


class CreditCard {
    public function order($price){}
    public function cancel($price){}
}
```

Payment クラスから現金、クレジットクラスを切り出すことで、
仮にQR決済等の他の支払い機能が追加されても
簡単に拡張し再利用することができます！


## 2. オープン・クローズドの原則（**O**pen-Closed Principle）


新しい機能の追加にはオープンで、修正にはクローズドである原則です。

要するに機能追加はおこなってもいいけど、

修正はなるべくやめましょう。ということです。

具体的に、Open の場合

- 機能追加の際には、既存のコードを変更することなくコードを追加して拡張する。

 Closed  修正の場合、

既存のコードの修正はさけて、新しい機能を追加できるように設計します。

サンプル見てみましょう！

### BAD

```php
class Payment
{
    public function orderByCash($price){}
    public function cancelByCash($price){}

    public function orderByCreditCard($price){}
    public function cancelByCreditCard($price){}
}
```

こちらの Payment クラスは原則適用前で、

仮にQR決済を追加する場合には既存のPaymentクラスを追加修正する必要があります。

このクラス設計だと既存コードに影響が出てしまいバグの原因にもつながります。

また１つのクラスの機能が増えて単一原則にも反してしまいます。

### GOOD

```php
// 抽象クラスとしてのPayment
abstract class Payment
{
    abstract public function order($price);
    abstract public function cancel($price);
}

// 以下は具体的なクラスとして定義
class Cash extends Payment
{
    public function order($price){}

    public function cancel($price){}
}


class CreditCard extends Payment
{
    public function order($price){}
    public function cancel($price){}
}


class Qr extends Payment
{
    public function order($price){}
    public function cancel($price){}
}
```

オープン・クローズドの原則を適用し、
Payment クラスを抽象クラスとして定義し、
決済方法が増えても新たに具体的なクラスを追加していくことで、
追加にはオープン、既存のPaymentクラスの修正にはクローズドな状態になっています。
結合度も**疎結合**が保たれています。



結合度とは、クラスとクラスの依存度合いを表す指標です。
クラス間の関係性が高く、一方の変更が相手へ与える影響が大きい状態を密（みつ）結合。
互いに独立していて変更や修正が簡単な状態を疎（そ）結合と言います。
疎結合を目指すことで、システムの柔軟性、保守性、再利用性が向上します。

## 3. リスコフの置換原則（**L**iskov Substitution Principle）

規定クラス（親クラス）を継承した派生クラス（子クラス）は

**規定クラス（親クラス）を置換できることを示す原則です。**

派生クラスは規定クラスと同じ振る舞いをしなければならないという原則になります。

### BAD

```php
// 規定クラス
class Animal
{
    public function walk()
    {
        echo "Walking";
    }
}


// 派生クラス
class Dog extends Animal
{
    public function walk()
    {
        echo "Dog is walking";
    }
}


// 派生クラス
class Fish extends Animal
{
    public function walk()
    {
        // 魚は泳げないので例外をスロー
        throw new Exception('Fish cannot walk');
    }
}
```

上記の例では派生クラスである Fish は規定クラスである Animal クラスを継承しています。

ただし、Fish の walk() では例外をスローしているので、

規定クラスで定義した walk() で期待する値を返していないません。

規定クラスのオブジェクトを置換できないことになります。

# まとめ🖊️

ご覧いただきありがとうございました！
後編も是非ご覧くださいね！

何か気になる点や、おかしな点ありましたらお気軽にコメントください！





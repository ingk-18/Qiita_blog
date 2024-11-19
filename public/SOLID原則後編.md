# SOLID原則とは？

**オブジェクト指向における設計原則のこと**

以下の５つの原則の頭文字をとったものです。

1. 単一責任の原則（**S**ingle Responsibility Principle）
2. オープン・クローズドの原則（**O**pen-Closed Principle）
3. リスコフの置換原則（**L**iskov Substitution Principle）
4. インターフェース分離の原則（**I**nterface Segregation Principle）
5. 依存性逆転の原則（**D**ependency Inversion Principle）

今回は後編として I, D の3原則をご紹介します！

https://qiita.com/gaky1201/items/1e8eed57b7c8a203f509

## 4. インターフェース分離の原則（**I**nterface Segregation Principle）

**インターフェースによって使用しないメソッドへの依存をさせてはならない。**

という原則になります。

### インターフェースとは？

**クラスが実装すべきメソッドを定義するもの。**

インターフェース自体は実装を行わずメソッド名、メソッドの引数、戻り値の型を定義します。

```php
// 動物のインターフェース
interface Animal {
    public function makeSound();
}

class Dog implements Animal {
    public function makeSound() {
        echo "Woof!\n";
    }
}

class Cat implements Animal {
    public function makeSound() {
        echo "Meow!\n";
    }
}
```


インターフェースを implements (実装) している Dog, Cat クラスはmakeSoundを実装しなければなりません。

一つのクラスで複数のインターフェースを実装することが可能です。

### BAD

```php
interface Worker {
    public function work();
    public function eat();
}


class Robot implements Worker {
    public function work()
    {
        echo "Robot is working";
    }

    // ロボットは食べることができないので例外をスロー
    public function eat()
    {
        throw new Exception("Robot cannot eat");
    }
}


class Human implements Worker {
    public function work()
    {
        echo "Human is working";
    }
    public function eat()
    {
        echo "Human is eating";
    }
}
```

上記では Robot で eat() が実装されていますが、
ロボットは食べられないので例外をスローしています。

Robot に対して不要なメソッドを強制させてしまっています。

### GOOD

```php
interface Workable {
    public function work();
}

interface Eatable {
    public function eat();
}

// ロボットは Workable のみ実装
class Robot implements Workable {
    public function Work (){
        echo "Robot is working\n";
    }
}

// 人間は Workable, Eatable を実装
class Human implements Workable, Eatable {
    public function work() {
        echo "Human is working\n";
    }

    public function eat() {
        echo "Human is eating\n";
    }
}
```

インターフェース分離の原則を適用して、
Worker というインターフェースを Workable, Eatable に分割して、
ロボットと人間それぞれ必要なインターフェースのみ実装するように修正しました。

ロボットが食べる不要なメソッドへの依存が解消されました。
このようにインターフェースを小さく定義して分割することで、
クラスへの影響範囲を小さくすることができ、保守しやすくなります。



## 依存性逆転の原則（**D**ependency Inversion Principle）

- 高レベルのモジュールは低レベルのモジュールに依存してはならない。
- 抽象クラスは具体クラスに依存してはならない。具体クラスは抽象クラスに依存すべきである。


## BAD

```php
// 高レベルのモジュール
class UserNotify {
	private $emailNotify;
	private $smsNotify;

	public function __construct() 
	{
		$this->emailNotify = new EmailNotify();
		$this->smsNotify = new SmsNotify();
	}

	public function notify($message)
	{
		// 低モジュールに変更があった場合に、高モジュールの呼び出し元も変更しなければならない
		$this->emailNotify->sendEmail($message);
		$this->smsNotify->sendSms($message);
	}
}

// 低レベルのモジュール
class EmailNotify {
	public function sendEmail($message)
	{
        echo "Sending email: " . $message;
    }
}

// 低レベルのモジュール
class SmsNotify {
	public function sendSms($message)
	{
        echo "Sending sms: " . $message;
    }
}
```

UserNotify で sendEmail(), sendSms() を呼び出していますが、
仮に各メソッドの引数等の変更がされた場合には、
高モジュールである UserNotify も修正を行わなければなりません。

高モジュールが低モジュールに依存してしまっている状態です。

### GOOD

```php
// 抽象インターフェース
interface Notify {
	public function send($message);
}

// 具体クラス
class EmailNotify implements Notify {
	public function send($message)
	{
        echo "Sending email: " . $message;
    }
}


// 具体クラス
class SmsNotify implements Notify {
	public function send($message)
	{
        echo "Sending sms: " . $message;
    }
}

// 通知クラス
class UserNotify {
	private $notify;

	public function __construct(Notify $notify)
	{
		$this->notify = $notify;
	}

	public function notify($message)
	{
		$this->notify->send($message);
	}
}
```

原則を適用してみました。
抽象インターフェースの Notify を定義して、
EmailNotify, SmsNotify で具体化させています。

具体が抽象を依存している状態に修正しています。

また、高モジュールである UserNotify では
具体的な送信方法に依存をしていないため、
異なる通知方法を簡単に切り替えることができます。

低モジュールの引数が変更されてもインターフェースの定義と異なりエラーとなるため
一貫性が保たれています。

# まとめ🖊️

ご覧いただきありがとうございました！
SOLID原則の理解は非常に負荷のかかる筋トレでした！汗

何か気になる点や、おかしな点ありましたらお気軽にコメントください！
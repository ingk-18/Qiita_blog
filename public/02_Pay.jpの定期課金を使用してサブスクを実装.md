---
title: PAY.JPの定期課金を使用してサブスクを実装
tags: pay.jp PHP 決済
author: gaky1201
slide: false
---
定期課金の実装を行いました。
備忘録として残したいと思います。
定期課金は単発課金と異なり、かなり処理が複雑でしたが
ドキュメントを熟読して、とってもよい経験となりました。



# 定期課金おおまかな流れ

### 1. 顧客がクレジットで定期課金を購入💳
[PAY.JPのcheck outに記載のカード情報入力フォームを用意します。](https://pay.jp/docs/checkout)
このフォームのdata属性に公開用APIKEYをセットする必要があります。

```js
<script
  type="text/javascript"
  src="https://checkout.pay.jp/"
  class="payjp-button"
  data-key="pk_test_0383a1b8f91e8a6e3ea0e2a9"
  data-submit-text="トークンを作成する"
  data-partial="true">
</script>
```

## 2. PAY.JPに顧客情報・プランを作成し、アプリケーション側のDBにも必要データを保持します。🖥️

顧客からプランの購入が行われたらPAY.JPに顧客登録とプラン作成をおこないます。
[PAY.JPへ顧客情報・プランを作成する処理](https://pay.jp/docs/api/?php#%E9%A1%A7%E5%AE%A2%E3%82%92%E4%BD%9C%E6%88%90)


```php
// 顧客作成
\Payjp\Customer::create(array("description" => "test"));

//　プランの作成
\Payjp\Plan::create(array(
        "amount" => 500,
        "currency" => "jpy",
        "interval" => "month",
        "trial_days" => 30,
));

```

#### アプリケーションのDBに必要情報を格納

顧客の定期課金情報をアプリケーション側のDBにも保存します。
DBにPAY.JPが保持している情報と同等のデータを保管していきます。
有事の際にPAY.JPではなく、システム側で顧客の履歴を確認できるようにするためです。

以下のような定期課金用のテーブルを作成し、
定期課金の決済が行われるごと（1サイクル分）にレコードを追加していきました。

```sql
create table subscriptions (
  id bigserial not null
  , user_id integer not null
  , is_deleted boolean default false not null
  , created timestamp default now() not null
  , modified timestamp default now() not null
) ;
```

さらに実装を進めるなかで以下のカラムを追加していきました。
- 定期課金開始・終了日
- 前後サイクルのプラン
- PAY.JPのサブスクID


## 3. webhook を受信して定期課金の更新処理を行う⚡

PAY.JPでは様々なイベントに応じてWebhookを送信してくれます。

発生するイベントの例

- 定期課金の更新
- キャンセルされて定期課金の停止
- 何かしらのエラー

:::note info
Webhookとは、イベント発生時にPOSTでパラメーターを送信すること。
:::

このWebhookを受信して、イベントに応じた処理を作成する必要があります。

- [PAY.JPのWebhookについて](https://pay.jp/docs/webhook)
- [Webhookで送信されてくるデータ](https://pay.jp/docs/api/?php#event%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)


# 意識・工夫したこと🏃‍♂️

## エラーハンドリング

工夫したことでもあり、大変苦労したことでもあります。
テストしている際にも想定外の様々なエラーが発生するため適切にエラーを処理を行い、
システム運営が正常に行えるように意識しました。例えば、

- 不正なクレジットカードの使用
- PAY.JPへ不正な定期課金リクエストが送信された
- コントローラーのアクションで発生したエラー
- 全く想定していないエラー

当然のことですが、解約済みの定期課金の参照や、加入しているプランと同じプランに加入リクエストを送信してしまうとPAY.JPからエラーがレスポンスされるため、そのようなリクエストがされないように以下を意識しました。

- 画面で処理ができないようにする
- バリデーションチェックで弾く
- 仮に送信されてもエラーレスポンスを適切に処理

## ドキュメントとして残す

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/671845/ddf3c1de-65dc-ab86-1ccf-11952bcf3c46.png)


定期課金の処理は簡単かと思いきや、
様々なエラーに遭遇し複雑化していきました。
そのため、備忘録やバグや追加機能実装に備えて、
draw.ioのフローチャートを使用して
分かりやすく残しておく工夫をしました。


# まとめ🖊️

ご覧いただきありがとうございました！
定期課金の実装は正直とっても大変でした。
開発期間も想定していた1ヶ月を上回り2ヶ月半かかりました！
でもとっても良い経験で終わった後は達成感を感じました。

何か気になる点や、おかしな点ありましたらお気軽にコメントください！






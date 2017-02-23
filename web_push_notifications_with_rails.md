# Web Push Notification with Rails

---

## Web Push Notificationとは?

- Webサイトから送れる**プッシュ通知**:love_letter:
- ユーザーエンゲージメントの向上の手法として有効:tada:
- スマホで送れるよ

![](https://cloud.githubusercontent.com/assets/15371677/23242145/a25d1724-f9ba-11e6-9b66-8af804b1d1de.png)

![](https://pushnate.com/labs/wp-content/uploads/2015/11/and1.png)

:point_up:こういうやつ

---

## 試してみよう

1. [Product Hunt](https://www.producthunt.com/)
2. 「購読」を押す
3. developer consoleを開く （`⌘` + `alt` + `i`）
4. `application` タブを開く
5. producthuntの**Service Worker**が動いているので、 `Push` を押す

![](https://cloud.githubusercontent.com/assets/15371677/23242187/d99433da-f9ba-11e6-9bcd-a344a7fe7e03.png)

### 購読解除方法

`Unregister` を押すだけ

---

## Service Workerとは?

![](https://cloud.githubusercontent.com/assets/15371677/23242821/73867072-f9be-11e6-8511-cbad70740fe0.png)

### Service Workerの基本とそれを使ってできること

http://qiita.com/y_fujieda/items/f9e765ac9d89ba241154

> ブラウザが Web ページとは別にバックグラウンドで実行するスクリプト
> オフラインのアプリを実現・サポートするために作られたものです

ちなみに対応環境としては **Safari** 以外は使えるもの、と思ってよい

### 特徴

> * DOM にアクセスできない
>   * DOM を操作したい場合は、Service Worker がコントロールしているページ(js)と postMessage でメッセージのやり取りをして行う
> * リクエストをプロキシすることが可能
> * Service Worker はブラウザが必要に応じて起動・終了するので、変数の値を保持しておけない
>   * Cache、IndexedDB 等で値を保存して、必要になった時に取り出すようにする
> * Promise を多用する
> * https か localhost 上でしか動作しない

### ライフサイクル

> ![](https://qiita-image-store.s3.amazonaws.com/0/54850/3d998a7a-d0c1-bc87-cc27-645c6d92d34b.png)

_* 詳細知りたい人は記事読んだ方がはやい（雑回避）_

### 雑解説

たとえば:point_down:のJavaScriptが、とあるhtmlで読み込まれる

```javascript
// https://github.com/kanatapple/service-worker/blob/gh-pages/push/script/main.js

// 1. navigatorがserviceWorkerに反応するとき like `navigator.respond_to?(:serviceWorker)`
if ('serviceWorker' in navigator) {

    // 2. DOMContentLoaded イベントが発火したとき
    document.addEventListener('DOMContentLoaded', () => {

        // 3. サービスワーカーを登録（ファイル名はなんでもよい）
        navigator.serviceWorker.register('./service-worker.js');

        // 4. Promise (ready -> register -> subscribe)
        navigator.serviceWorker.ready
                 // 5. 準備できたら
                 .then((registration) => {
                     // 6. 購読する? (y/n)
                     return registration.pushManager.subscribe({userVisibleOnly: true});
                 })
                 // 7. 購読する（YES）なら
                 .then((subscription) => {
                   // 8. endpointと登録者情報をapp serverに送る
                 })
                 .catch(console.error.bind(console));
    }, false);
}
```

:point_down: 登録される `service-worker.js` は以下の感じ（各イベントに反応する）

```javascript
// https://github.com/kanatapple/service-worker/blob/gh-pages/push/service-worker.js
'use strict';

// install イベントが発火したとき
self.addEventListener('install', (event) => {
  // do something...
});

// activate イベントが発火したとき
self.addEventListener('activate', (event) => {
  // do something...
});

// fetch イベントが発火したとき
self.addEventListener('fetch', (event) => {
  // do something...
});

// push イベントが発火したとき
self.addEventListener('push', (event) => {
    console.info('push', event);

    // event オブジェクトからdataを取り出す
    const message = event.data ? event.data.text() : '(・∀・)';

    event.waitUntil(
        // 通知を表示する
        self.registration.showNotification('Push Notification Title', {
            body: message,
            icon: 'https://kanatapple.github.io/service-worker/push/images/image.jpg',
            tag: 'push-notification-tag'
        })
    );
});
```

つまり、下記が購読までの流れ

1. クライアントとなるhtml から js が実行される
2. js で `serviceworker.js` が登録される（**そのWebページを開いてないときもWorkerが特定イベントがListen** されるように）
3. 登録後、購読するか聞かれ、YESの場合、endpointと登録者情報をapp serverに保存

### Web Pushの概念図

> ![](http://cdn-ak.f.st-hatena.com/images/fotolife/j/jovi0608/20141204/20141204180839.png)

http://d.hatena.ne.jp/jovi0608/20141204/1417697480

> 1. クライアントからプッシュサーバの登録情報(endopoint, registrationID)をアプリサーバに通知。
> 2. アプリサーバが、プッシュ通知をHTTP PUTのリクエストボディに付けてプッシュサーバに送信。
> 3. プッシュサーバは、プッシュ通知をチャネルで区別し、送信クライアントを選定。
> 4. プッシュサーバは、HTTP/2のサーバプッシュやGCM(Google Cloud Message)など利用してクライアントにプッシュ通知を送信。プロトコルは別にWebSocket/SSEでも構いません。
> 5. クライアントは、プッシュサーバからプッシュ通知を受けると、Service Worker上でPushイベントが発生。ArrayBuffer, blob, json, textの形式でプッシュデータを取り出せるようになる。
> 6. プッシュされたデータはキャッシュ更新なり、postMessageでDOMに渡したり、クライアント上でいかようにでも処理することができる。

## Railsでの実装

demo

[yhirano55/web-push-notification-sample](https://github.com/yhirano55/web-push-notification-sample)

ありがとうございました:pray::pray::pray:

## 資料

* [Service workerとwebプッシュ通知](https://www.slideshare.net/zaruhiroyukisakuraba/service-workerweb-57713638)

## おまけ

Chrome 便利

chrome://serviceworker-internals/

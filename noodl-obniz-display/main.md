この記事は、[Noodl Advent Calendar 2019](https://qiita.com/advent-calendar/2019/noodl) 12日目の記事です。
@kn1chtと申します。もともとやりたかったネタが頓挫したため、その過程でできたものを記事化することにします。

昨日11日は@kisaichi07さんの「[**NoodlでUIデザインをしよう！インフォグラフィック編**](https://www.youtube.com/watch?v=oBakYg6MyiA&feature=youtu.be)」でした。この日は別の勉強会にいて動画全体を拝見してはいませんが、HTML Contentsに`{{ }}`でデータを渡せるのは知らなかったので参考になりました。
明日の記事は@kmaepuさんの「ラズパイとNoodlでデジタルサイネージを作った話」です。

## TL;DR
- **Noodlで`obniz.js`を読み込む**ことで、obnizに直接アクセスすることに成功しました
- ただし、Noodl内のプレビューでは動作しません
    - 現在のNoodlではElectronのバージョンが古く、`async`が使えないためです
    - これにはブラウザでプレビューを開くことで対処できます

![teaser.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/a0db501a-700e-1ef7-3514-cda0fd2540eb.png)

作ったプロジェクトは以下のGistに置いてあるので、obnizをお持ちの方はお試しください（obniz idの部分は空欄にしてあるので、お手元のobnizのidを入力してください）。

- **https://gist.github.com/kn1cht/99579459ba69a6b4d4c840e3ea2f8981**

## はじめに：Noodlとobniz
### [Noodl](https://tensorx.co.jp/noodl-jp/)とは
Noodlは、**ノードを繋いで簡単にUIのプロトタイピングができる**スウェーデン発のツールです。プロトタイプといっても実際にデータをやり取りでき、MQTTなどで外部と通信も可能なので、趣味のIoTプロジェクトにぴったりのツールです。

使い方や雰囲気は[**Noodl Advent Calendar 2019**](https://qiita.com/advent-calendar/2019/noodl)の各記事を見ていくとなんとなく分かることでしょう（丸投げ）。

### [obniz](https://obniz.io/ja/)とは
Webから操作できるIoTボードです。これまでのマイコンボードはパソコンと繋いで、プログラムを書き込んで……というのが普通でしたが、obnizでは全てがWebで完結します。
**クラウド経由でAPIを叩くかのようにハードウェアを操作できる**ので、ハードウェアの面倒な部分と格闘せずにシュッとIoTシステムを作れます。

### Noodlとobnizの連携方法
Noodlとobnizという先進的なツールを見れば、組み合わせて色々やりたくなるのは当然です。
このカレンダーでも、 @tseigo さんがゆるメカトロ車をNoodl製のUIで操作する記事を公開しておられます。

- [Noodlからobniz Messaging経由でIoTゆるメカトロ車を動かすメモ – 1ft-seabass.jp.MEMO](https://www.1ft-seabass.jp/memo/2019/12/08/noodl-and-obniz-messaging-rest-collaboration/)
- [Noodl × Node-RED × obnizを連携する](https://qiita.com/kmaepu/items/a3a57e1de8cbd56e7ed3#noodl-7-node-red%E3%81%A8%E9%80%A3%E6%90%BA)

やり方は様々ですが、Noodl側とobniz Cloud側をMQTTやREST APIの通信でつなぎ、必要に応じて[enebular](https://www.enebular.com/ja/index.html)などを間に入れる方法がよく使われています。
いわゆる**[IoTフルスタック構成](https://twitter.com/n0bisuke/status/1166306421287211008)**に近い感じですね。

<img alt="fullstack.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/32588346-3aca-7bb0-d333-a47b4f55a82b.png" height=180px>

これはこれで汎用性があって使いやすいのですが、全くの初心者が試すとなると、多数の知らないツールやサービスをいきなり駆使しなければなりません。
そこで、Noodlから直接obnizを動作させられないか試してみました。

<img alt="direct.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/9a6987fb-5430-1b00-b89c-3dff77902d39.png" height=180px>

## アプローチ
[**obnizの仕組み**](https://obniz.io/ja/doc/howitwork)を読むと分かる通り、obnizはobniz Cloudが提供するAPIに各種言語のSDKからアクセスすることで動きます。
obniz開発者コンソールでコードを書いて動かすのが最も手軽ですが、実際には**`obniz.js`さえ使えればどこでJavaScriptを実行しても動きます**（例えばローカルのエディタでHTMLを書いてブラウザで開いてもOKです）。

NoodlはScript Downloaderというノードで外部のJSを取得できるので、`obniz.js`を使うこともできそうです。

- [Noodlで外部JSライブラリ読み込み（グラフ表示）](https://qiita.com/kmaepu/items/96c42fd7d48fcbaddb76)

上の記事にある通り、**Script Downloader + Javascript + HTML Content**の3連コンボが基本形です。
[obnizクイックスタート](https://obniz.io/ja/lessons/lessons/quickstart)にあるHTMLのサンプルコードを基に実装することとします。

## ノードの準備
### Script Downloader
obnizのサンプルで読み込まれているのは`jquery-3.2.1.min.js`と`obniz.js`の2つです。
URLをExternal scriptsにコピペでOKです。

![Script Downloaderの操作](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/d831ade2-3b85-c24a-b4f8-6b87c3a5550c.gif)

### HTML Content
サンプルコードの`<html></html>`の中身を全てContent > HTMLにコピペします。
ただし、JSは他のノードで動かすので`<script>`タグのみ消去しておきます（本当はJavaScriptも全部HTML Contentの中で動くのかもしれませんが、それだとNoodlを使う意味がなさすぎるのでやりません:sweat_smile:）。

![html.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/9e7d0926-88bf-656f-d25b-eec088954e16.png)

また、`starter-sample.css`というスタイルシートが相対パス指定になっているので、URLに書き換えます。

```diff
-  <link rel="stylesheet" href="/css/starter-sample.css">
+  <link rel="stylesheet" href="https://obniz.io/css/starter-sample.css">
```

### Javascript
ここは少し工夫が必要です。サンプルコードの`<script></script>`で囲まれた部分を`run`にコピペするだけでは動きません。
`obniz.onconnect = async function() {`の行にエラーが出ています。これは`async`の指定を取り除けば消えます（理由は後述します）。

![js-error.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/87567b0a-03d1-0316-73be-2e116deac7ba.png)

他2つのノードと連携するために、`inputs`をこのように定義します（この辺りは[Noodlで外部JSライブラリ読み込み（グラフ表示）](https://qiita.com/kmaepu/items/96c42fd7d48fcbaddb76)と同様です）。

```js
    inputs: {
        isAddedToDOM : "boolean",
        domElement : "domelement",
        scriptsLoaded: "boolean"
    },
```

ノード同士を繋ぎます。特に自分でコードを書かなくても、この通りに設定すれば外部スクリプトの読み込み完了を待ってくれたり、自分で書いたJSから`document`が利用可能になったりするようです。

![nodes.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/65c53ea9-f543-c022-69cf-b750aec799b2.png)

## プレビューが動作しない問題の調査
これで動くはずですが、Javascriptノードの周りに赤い点線が出ていますね。Noodlエディタの右上に警告マークが出ているので見てみましょう。
![js-warn.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/005b0af1-82b4-c03a-7c41-d8c3c43666c2.png)

`Obniz is not defined`だそうです。`obniz.js`は読み込んでいるので定義されていないはずはないのですが……。
Noodlでは、デバッグボタン（右上に並んでいる虫のアイコン）から"OPEN WEB DEBUGGER"ボタンを押すことで皆さんおなじみの**Chrome開発者ツールを開くことができます**。原因のよくわからないエラーが出ていたらまずここで確認すると早いと思います。

![devtool.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/d5e5f3af-8da4-de90-310f-09796998ebea.png)

`obniz.js`の途中でエラーが出て止まっていました。この部分を見てみると、次のようになっています。

```js
async loop() {
    if (typeof this.looper === 'function' && this.onConnectCalled) {
      let prom = this.looper();
...
```

`async loop() {`の行がシンタックスエラーを起こしたようです。Javascriptノードでも、`async`を消したらエラーが消えました。もしやNoodl自体がasync/awaitに対応してないのでは？　と思ったので調べてみました。

※async/await (Async Functions)が何かは本筋に関係ないので説明しません。知らない方も、最近のJavascriptに増えた新機能ということだけ押さえていただければ大丈夫です。

### Noodlではasync/awaitが使用できない
Noodlのメニューからバージョン情報を見ることができます。

> - Noodl 1.3.1
> - electron : 1.4.13
> - chrome   : 53.0.2785.143
> - node     : 6.5.0
> - v8       : 5.3.332.47

[Electron 1.4.13は2016年12月に出たバージョン](https://electronjs.org/releases/stable?version=1&page=11#1.4.13)で、現行に比べるとかなり古いものです。Chromeがasync/awaitに対応したのはバージョン55以降なので、**Noodlに内蔵されているElectronはasync/awaitを使えない**ものだと分かります。こればかりはNoodl側が新しいバージョンに対応しない限りどうすることもできません。

### 回避策：ブラウザでプレビューを開く
文法レベルで非対応なら仕方ないか……と諦める所でしたが、Noodlのプレビューはブラウザでも開けるのを思い出しました。PCやスマホに入っている**最新のブラウザならasync/awaitを使った`obniz.js`も読み込める**はずです。

ブラウザで開くのは簡単で、エディタ右上のスマホアイコンを押して出てくるURLを開くだけです。

![3b07a9ce62e5795ddd10b7076ccf4603.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/5345207a-09d5-2365-ee2d-d872395b8d8c.gif)

**あっさり動きました**。obnizのIDをぼかしていますが、上に緑色で"online:  {obniz id} via internet"と出ているので無事繋がっているのが分かります。
もちろんサンプル部分も問題なく動いてます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">（Advent Calendar記事用動画）<br><br>obnizクイックスタート（<a href="https://t.co/rif0SjaPsb">https://t.co/rif0SjaPsb</a>）にあるサンプルコードをNoodlに移植して動かしている様子です<a href="https://twitter.com/hashtag/Noodl?src=hash&amp;ref_src=twsrc%5Etfw">#Noodl</a> <a href="https://twitter.com/hashtag/obniz?src=hash&amp;ref_src=twsrc%5Etfw">#obniz</a> <a href="https://t.co/YkAp9rgDOr">pic.twitter.com/YkAp9rgDOr</a></p>&mdash; kn1cht (@kn1cht) <a href="https://twitter.com/kn1cht/status/1204798342749798400?ref_src=twsrc%5Etfw">December 11, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## まとめ
Noodlから直接obnizを動かすことができました。本記事ではUI部分をobnizのサンプルHTMLコードからそのまま流用しましたが、Noodlの強みであるUIと`obniz.js`を組み合わせることももちろん可能です。

この方法の使い道は何でしょうか？　初心者が試す時云々と述べて始めたものの、**ブラウザで別に開かないと動かない**のは初心者の混乱を招きそうです。また、**JavascriptやHTMLのコードをそのまま扱う**ので、ノンコーディングというNoodlのメリットがあまり活かせていません。
やはりobnizのコードはobnizの開発者コンソールに置いておき、Noodl側ではUIの処理だけに集中した方が見通しが良さそうです。

とはいえ、**思いついた実装を短時間で試してみたい**ような場合には、とりあえずNoodlから直でobnizを動かすというのも手軽でいいと思います。今回作った`obniz.js`関連のノードをコンポーネントにまとめておき、ライブラリとして呼べるようにしておくと使いやすいかもしれませんね！

## 蛇足（もともとやりたかったこと）
obnizは**128x64のディスプレイ**を持っており、文字や図形を表示できます。
obnizのAPIで文字などを出すこともできますが、JS側でcanvasに描画しておいてコンテキストを渡せば画面に反映されます。

<img alt="obniz display" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/150557/63197ed1-b928-11ae-ea43-13eaee452819.jpeg" height=300px>


- [obnizのディスプレイで遊んでみた – メカニカルマンブログ](https://blog.mechanicalman.jp/2018/06/11/obniz-display-draw-ctx/)

Noodlの画面もcanvasでレンダリングされるようなので、Noodlで構築した画面をそのままobnizに表示することを目論んでいました。ただ、Noodl内部でNoodlの画面をcanvasとして取得する方法が見つからず断念しました。

# Amazon Connect Streams Documentation
(c) 2018-2020 Amazon.com, Inc. All rights reserved.

## グローバルレジリエンシー
Connect Global Resiliency機能でAmazon Connect Streamsを使用する詳細については、まず特定のドキュメント[こちら](Documentation-DR.md)を参照してください。

### "ルーティング可能性 "に関する注意
ストリームにおけるルーティング可能性は、エージェントのステータスにのみ影響されることに注意してください。音声コンタクトはエージェントのステータスを変更するため、ルータビリティに影響します。タスクとチャットのコンタクトは、ルーティングに影響しません。しかし、他のチャンネルが同時ライブコンタクトの制限に達した場合、エージェントはより多くのコンタクトをルーティングされませんが、技術的にはルーティング可能なエージェント状態になります。

# 使い方
amazon-connect-streamsは[npmjs.com](https://www.npmjs.com/package/amazon-connect-streams)から入手できます。ここでダウンロードしたい場合は、`release/connect-streams*` のようにどちらかのファイルを使うことができる。

`npm run release` を実行すると、新しいリリースファイルが生成される。npmを使ってローカルにビルドするための完全な手順は[下記](#build-your-own-with-npm)にあります。

バージョン 1.x では、レガシービルド用の `make` もサポートしていました。このオプションはバージョン2.xで削除されました。

# 重要なお知らせ

1. 2024年3月 - 2023年7月13日に開始されたGoogle Chromeの機能である[Storage Partitioning](https://developers.google.com/privacy-sandbox/3pcd/storage-partitioning)に対応して、2024年2月10日にミュート機能を調整し、すべてのCCPでミュート状態を同期させるための短期的な修正を行いました。しかし、現在の制限により、この変更では保留中のミュートを無効にする必要がありました。回避策として、エージェントは保留になる前に通話中のミュートを解除する必要があります。2024年8月までにこの問題に対処し、元のミュート動作に戻す予定です。
    * 現時点では、以下のAPIは保留中に失敗します：
      * `voiceConnection.muteParticipant()` * * `voiceConnection.muteParticipant()
      * `voiceConnection.unmuteParticipant()`
      * `agent.mute()`
      * `agent.unmute()`
    * 回避策として、通話を保留にする前にミュートにすることができます。

1. 2022年12月 - CCPに加えて、connect.agentAppを使用して、エージェントにガイド体験を提供するアプリケーションを埋め込むことができるようになりました。使用方法の詳細については、[更新されたドキュメント](https://github.com/amazon-connect/amazon-connect-streams/blob/master/Documentation.md#initialization-for-ccp-customer-profiles-amazon-q-connect-and-customviews)を参照してください。
   - エージェントのためのガイド付きエクスペリエンス
     - Amazon Connectを使用すると、エージェントが対話中に指定された瞬間に何を見たり実行したりする必要があるかに焦点を当てた、カスタマイズされたビューを通してエージェントを案内する、ガイド付きのステップバイステップのエクスペリエンスを作成できるようになりました。さまざまなタイプの顧客との対話のためのワークフローを設計し、コールキュー、顧客情報、対話型音声応答（IVR）などのコンテキストに基づいて、エージェントに異なるステップバイステップのガイドを表示できます。この機能は、Connect エージェント・ワークスペースだけでなく、Streams API を介して別のウェブサイトに埋め込むことができる埋め込み可能なアプリケーションでも利用できます。詳細については、AWSのウェブサイトhttps://aws.amazon.com/connect/agent-workspace/。

1. 2022年12月 - 2.4.2
    * このパッチは、StreamsのVoice ID APIにおいて、Connect Contact Trace Records (CTR)のVoiceIdResultセグメント内のgeneratedSpeakerIDフィールドに不正な値が設定される可能性がある問題を修正します。この問題は、カスタムCCP統合コードでenrollSpeakerInVoiceId()、evaluateSpeakerWithVoiceId()、またはupdateVoiceIdSpeakerId()を呼び出す一部のシナリオで発生します。Voice IDを使用し、Voice ID CTRを消費している場合、またはエージェントワークフローでスピーカーIDを更新している場合は、このバージョンにアップグレードしてください。

1. 2022年12月 - 2.4.1
    * このバージョンでは、エージェントとスーパーバイザーに、より強化されたモニタリング体験を提供するアップデートが行われました。この機能で導入された新しい API は `isSilentMonitor`、`isBarge`、`isSilentMonitorEnabled`、`isBargeEnabled`、`isUnderSupervision`、`updateMonitorParticipantState`、`getMonitorCapabilities`、`getMonitorStatus`、`isForcedMute` です。監視機能の拡張を有効にする前に、Contact Control Panel（CCP）またはAgentワークスペースの最新バージョンを使用していることを確認してください。

1. 2022年8月 - 2.3.0
    * 音声 ID CTR 修正の最終的な解決については、2.4.2 を参照してください。

1. 2022年1月 - 2.0.0
    * `initCCP`を複数回呼び出しても、複数の埋め込みCCPがウィンドウに追加されることはなくなりました。複数のCCPを初期化するというユースケースはStreamsでサポートされたことがなく、この変更はそのようなケースから発生する予測不可能な動作を防ぐために追加されました。
    * `agent.onContactPending`は削除されました。代わりに `contact.onPending` を使用してください。`connect.onError` がトリガーされるようになりました。以前はこのAPIは全く動作しませんでした。この関数の中にアプリケーションロジックがある場合、その動作が変更されたことに注意してください。詳細は documentation.md のエントリを参照してください。

1. 2021年9月 - 1.7.0には、2021年9月27日に開始されたAmazon Connect Voice IDを使用するために必要な変更が含まれています。Voice IDをご利用になりたいお客様は、今後1ヶ月以内にStreamsをバージョン1.7.0以降にアップグレードしてください。そうしないと、2021年10月末までにVoice ID APIが動作しなくなります。Voice ID APIの詳細については、[Voice ID APIsセクション](Documentation.md#voice-id-apis)をご覧ください。

1. 2021年7月 - CCPの変更により、エージェントがコンタクトの処理中にランチやオフラインなどの次のステータスを設定できるようになりました。この機能の詳細については、[Amazon Connectエージェントトレーニングガイド](https://docs.aws.amazon.com/connect/latest/adminguide/set-next-status.html)およびこの機能の[リリースノート](https://docs.aws.amazon.com/connect/latest/adminguide/amazon-connect-release-notes.html#july21-release-notes)を参照してください。エージェントがConnectのすぐに使えるCCPV2 UXと直接やり取りする場合、デフォルトでこの機能にアクセスできます。それ以外の場合、streamsJSアプリケーションがエージェントの状態を切り替えるために `agent.setState()` を呼び出す場合は、この機能を使用するようにコードを更新する必要があります：
   - Agent.setState()**が更新され、オプションのフラグ `enqueueNextState: true` を渡して Next Status 動作をトリガーできるようになりました。
   - 新しい**agent.onEnqueuedNextState()**リスナーにより、エージェントが次のステータスを選択した時/成功した時のイベントを購読できます。
   - 新しい **agent.getNextState()** API は、エージェントが次のステータスを正常に選択した場合はステートオブジェクトを返し、そうでない場合は null を返します。
   - もし `agent.setState()` による Next Status 機能を使用したい場合、After Contact Work をコンタクトからクリアする際に `contact.complete()` ではなく、`contact.clear()` を使用していることを確認してください。

1. 2020年12月 - 1.6.0では、新しいエージェントアプリAPIがリリースされました。CCPに加えて、顧客プロファイルやAmazon Q Connectなど、connect.agentAppを使用して追加のアプリケーションを埋め込むことができるようになりました。使用方法の詳細については、[更新されたドキュメント](Documentation.md#initialization-for-ccp-customer-profiles-and-amazon-q-connect)を参照してください。また、Amazon Connect Voice IDのプレビューリリースもご紹介します。
   - ### Amazon Connect Customer Profilesについて
     - Amazon Connect Customer Profilesは、複数の外部アプリケーションの顧客情報とAmazon Connectのコンタクト履歴をすばやく組み合わせることができるように、事前に構築された統合機能を提供します。これにより、エージェントが顧客とのやり取りで必要とするすべての情報を一箇所にまとめた顧客プロファイルを作成することができます。
   - ### Amazon Q Connectについて
     - Amazon Q Connectを使用すると、エージェントは、よくある質問（FAQ）、Wiki、記事、さまざまな顧客問題を処理するためのステップバイステップの手順など、複数のリポジトリでコンテンツを検索して見つけることができます。検索ボックスに質問やフレーズ（「ハンドバッグは購入後どのくらいで交換できますか」など）を入力すれば、どのキーワードが有効かを推測する必要はありません。
   - ### Amazon Connect Voice IDについて（この機能はAmazon Connectのプレビューリリースであり、変更される可能性があります。）
     - Amazon Connect Voice IDは、コンタクトセンターでの音声対話をより安全かつ効率的にするリアルタイムの発信者認証を提供します。Voice IDは、機械学習を使用して、発信者固有の声の特徴を分析することにより、真正な顧客の身元を確認します。これにより、コンタクトセンターは、発信者が複数のセキュリティ質問に答えることに依存しない追加のセキュリティレイヤーを使用することができ、自然な会話の流れを変えることなく、顧客の登録と確認を簡単に行うことができます。

1. 2020年7月 -- 私たちは最近、新しいオムニチャネルのCCPが3つの音声のみのエージェントの状態に遭遇したときの動作を変更しました： `FailedConnectAgent`、`FailedConnectCustomer`、`AfterCallWork`です。
   - `FailedConnectAgent` -- 以前は、この状態をクリアするためにエージェントが "Clear Contact "ボタンをクリックする必要がありました。エージェントが "Clear Contact "ボタンをクリックすると、以前の動作ではエージェントは必ず`Available`状態に戻っていました。これで `FailedConnectAgent` 状態は、これまで `FailedConnectCustomer` がそうであったように、"自動クリア "されるようになりました。
   - `FailedConnectAgent` と `FailedConnectCustomer` -- 現在、これらの状態を自動クリアするために `contact.clear()` API を使用しています。その結果、エージェントは以前の可視エージェント状態（例えば `Available`）に戻ります。以前は、エージェントは常に `Available` に設定されていました。FailedConnectAgent` と `FailedConnectCustomer` については、カスタム CCP でもこのアップデートで動作が変わることに注意してください。
   - `AfterCallWork` -- 新しい `contact.clear()` の動作の一部として、`AfterCallWork` の状態で "Clear Contact" をクリックすると、エージェントは以前の可視エージェント状態（例えば `Available` など）に戻ります。独自のアフターコールワーク動作を実装しているカスタムCCPは、この変更の影響を受けないことに注意してください。
     - `contact.complete()`を非推奨にします。そのため、代わりに `contact.clear()` を使用する必要があります。CCPのAfter Call Workの動作を顧客のCCPでエミュレートしたい場合は、音声コンタクトをクリアするときに必ず`contact.clear()`を使用してください。

## 概要
Amazon Connect Streams API（Streams）を使用すると、既存のWebアプリケーションをAmazon Connectに統合できます。 Streamsを使用すると、Contact Control Panel（CCP）やCustomer ProfilesアプリのUIをページに埋め込むことができます。 また、オブジェクト指向のイベント駆動インターフェイスを通じて、エージェントやコンタクトの状態イベントを直接処理することができます。 組み込まれたインターフェイスを使用することも、ゼロから独自のインターフェイスを構築することもできます： Streamsはあなたに選択肢を提供します。

このライブラリは
[amazon-connect-chatjs](https://github.com/amazon-connect/amazon-connect-chatjs)と併用する必要があります、
[amazon-chime-sdk-js](https://github.com/aws/amazon-chime-sdk-js/blob/main/README.md)、
または[amazon-connect-taskjs](https://github.com/amazon-connect/amazon-connect-taskjs)
と組み合わせて使用します。

## アーキテクチャ
[ここをクリック](Architecture.md)すると、Amazon Connect Stream APIがどのように動作するかの簡単なアーキテクチャの概要が表示されます。
Amazon Connect Streams API がどのように動作するか、簡単なアーキテクチャの概要をご覧ください。

## Getting Started

### 最新バージョンの CCP (AKA /ccp-v2)にアップグレードしますか？
新しい CCP に移行する場合は、このリポジトリの最新版にアップグレードすることをお勧めします。また、[RTC-JSの最新バージョン](https://github.com/aws/connect-rtc-js)を使用している場合は、同様にアップグレードしてください。新CCPへの完全な移行ガイドと、新CCPでStreamsを使用する際の違いを完全に理解するためには、こちらの投稿をご覧ください： https://docs.aws.amazon.com/connect/latest/adminguide/upgrade-to-latest-ccp.html. また、https://docs.aws.amazon.com/connect/latest/adminguide/upgrade-ccp-streams-api.html。

### 許可リスト
Streams APIを使用するための最初のステップは、埋め込みたいページを許可リストにすることです。
顧客のセキュリティのために、特定のインスタンスにCCPを埋め込むすべてのドメインが明示的に許可リストに登録されることを要求します。各ドメイン・エントリは、プロトコル・スキーム、ホスト、およびポートを識別します。同じプロトコル・スキーム、ホスト、およびポートの後ろでホストされているページはすべて、Streams ライブラリを使用するために必要な CCP コンポーネントの埋め込みが許可されます。

ページを許可リストに登録するには

1. AWSアカウントにログインし、Amazon Connectコンソールに移動します。
2. インスタンスの設定概要ページをロードするために、ページを許可するインスタンスのインスタンスエイリアスを選択します。
3. ナビゲーションペインで、「Approved origins」を選択します。
4. Add Origin "を選択し、ドメインURLを入力します。
   例えば、"https<nolink>://example.com"、またはウェブサイトが非標準ポートでホストされている場合は、"https<nolink>://example.com:9595 "を入力します。

#### 注意事項
* 許可リストに登録されたドメインは必ずHTTPSでなければならない。
* Streams ライブラリを初期化しようとするページはすべて、前述の手順で許可リストに登録されたドメインでホストされている必要があります。
* 初期化された Streams ライブラリを含むタブ、または他の CCP タブを開いているタブはすべて同期されます。これは、1つのウィンドウで行われた状態の変更が、開いているすべてのウィンドウに伝達されることを意味します。
* 同じコネクト・インスタンスで複数のブラウザを同時に使用することはサポートされていません。


## Downloading Streams with npm

```
npm install amazon-connect-streams
```

## Importing Streams with npm and ES6

```
import "amazon-connect-streams";
```

これにより、現在のコンテキストで `connect` 変数が使用可能になる。

## Usage with TypeScript

`amazon-connect-streams`はTypeScriptと互換性があります。バージョン `3.0.1` 以上を使用する必要がある：

```ts
import "amazon-connect-streams";

connect.core.initCCP({ /* ... */ });
```

## GithubからStreamsをダウンロードする
Amazon Connectをアプリケーションに組み込むための次のステップは、GitHubからStreamsライブラリをダウンロードすることです。こちらの公開リポジトリをクローンしてください：

```
$ git clone https://github.com/aws/amazon-connect-streams
```

ストリームをダウンロードしたら、ディレクトリに移動してビルドする：

```
$ cd amazon-connect-streams
$ make
```

これで `connect-streams-${VERSION}.js` というファイルが生成されます。これは Amazon Connect Streams API の完全版で、あなたのページに含めることができます。ウェブアプリケーションで `connect-streams-${VERSION}.js` を提供することができます。

## Build your own with NPM
[NodeJS](https://nodejs.org)の最新LTSバージョンをインストールする。

### Instructions for Streams version 2.x:
```
$ git clone https://github.com/aws/amazon-connect-streams
$ cd amazon-connect-streams
$ npm install
$ npm run release
```

**release**ディレクトリでビルドの成果物を探す - `connect-streams.js` というファイルが生成され、同じファイルの最小化バージョン `connect-streams-min.js` が生成されます。

To run unit tests:
```
$ npm run test-mocha
```
注：これらのテストは、上記で生成されたリリース・ファイル上で実行される。

### Streams バージョン 1.x 用の説明： 
`gulp` もインストールする必要がある。`gulp` はグローバルにインストールできる。

```
$ npm install -g gulp
$ git clone https://github.com/aws/amazon-connect-streams
$ cd amazon-connect-streams
$ npm install
$ npm run release
```

**release**ディレクトリでビルドの成果物を探す - `connect-streams.js` というファイルが生成され、同じファイルの最小化バージョン `connect-streams-min.js` が生成されます。

To run unit tests:
```
$ npm run gulp-test
```
注：これらのテストは、上記で生成されたリリース・ファイル上で実行される。

## AWS SDK と Streams の利用
Streams の `./src/aws-client.js` ファイルには、AWS-SDK が "baked-in" されています。AWS SDK の前に Streams をインポートして、`Window` にバインドされる `AWS` オブジェクトが Streams のオブジェクトではなく、手動で取り込んだ SDK のオブジェクトになるようにしてください。

## Initialization
Streams APIを初期化することは、すべてが正しくセットアップされ、イベントをリッスンできることを確認する最初のステップである。

### `connect.core.initCCP()`
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <script type="text/javascript" src="connect-streams-min.js"></script>
  </head>
  <!-- init()の呼び出しをonloadとして追加し、ページがロードされた時点で実行されるようにする。 -->
  <body onload="init()">
    <div id="container-div" style="width: 400px; height: 800px;"></div>
    <script type="text/javascript">
      var containerDiv = document.getElementById("container-div");
      var instanceURL = "https://my-instance-domain.awsapps.com/connect/ccp-v2/";
      // initialize the streams api
      function init() {
        // initialize the ccp
        connect.core.initCCP(containerDiv, {
          ccpUrl: instanceURL,            // REQUIRED
          loginPopup: true,               // optional, defaults to `true`
          loginPopupAutoClose: true,      // optional, defaults to `false`
          loginOptions: {                 // optional, if provided opens login in new window
            autoClose: true,              // optional, defaults to `false`
            height: 600,                  // optional, defaults to 578
            width: 400,                   // optional, defaults to 433
            top: 0,                       // optional, defaults to 0
            left: 0                       // optional, defaults to 0
          },
          region: 'eu-central-1', // REQUIRED for `CHAT`, optional otherwise
          softphone: {
            // optional, defaults below apply if not provided
            allowFramedSoftphone: true, // optional, defaults to false
            disableRingtone: false, // optional, defaults to false
            ringtoneUrl: '[your-ringtone-filepath].mp3', // optional, defaults to CCP’s default ringtone if a falsy value is set
            disableEchoCancellation: false, // optional, defaults to false
            allowFramedVideoCall: true, // optional, default to false
            VDIPlatform: null, // optional, provide with 'CITRIX' if using Citrix VDI, or use enum VDIPlatformType
            allowEarlyGum: true, //optional, default to true
          },
          storageAccess: {
            canRequest: true, // By default this is set to true. You can set it to false to opt out from checking storage access.  
            mode: "custom", // To use the default banner, set this to "default"
            /** More customization options can be found here: https://github.com/amazon-connect/amazon-connect-streams/blob/master/src/index.d.ts under StorageAccessParameters */
          },
          pageOptions: { //optional
            enableAudioDeviceSettings: false, //optional, defaults to 'false'
            enableVideoDeviceSettings: false, //optional, defaults to 'false'
            enablePhoneTypeSettings: true //optional, defaults to 'true' 
          },
          shouldAddNamespaceToLogs: false, //optional, defaults to 'false'
          ccpAckTimeout: 5000, //optional, defaults to 3000 (ms)
          ccpSynTimeout: 3000, //optional, defaults to 1000 (ms)
          ccpLoadTimeout: 10000 //optional, defaults to 5000 (ms)
         });
      }
    </script>
  </body>
</html>
```
`ccpUrl`にあるビルド済みの CCP を iframe に読み込み、提供された `containerDiv` に配置することで Connect と統合します。API リクエストはこの CCP を経由し、エージェントとコンタクトの更新はこの CCP を通して公開され、JS クライアントコードで利用できるようになります。
* CCP の URL： CCP の URL。これは、CCPをスタンドアロンのページで使用するために、通常移動するページです。
* インスタンスごとに異なります： 例: `us-west-2`。チャットチャネルにのみ必要です。
* `loginPopup`： オプション。デフォルトは `true` である。 ユーザーの認証が切れたときに表示されるログインポップアップを無効にするには `false` を設定する。
* `loginOptions`: オプション： オプション。`loginPopup` が `true` に設定されている場合のみ有効である。
   新しいタブの代わりに新しいウィンドウでloginpopupを開くには、以下のプロパティを持つオブジェクトを指定してください。
   * `autoClose`： オプション。デフォルトは `false` である。`true`に設定すると、ユーザがログインした後にログインポップアップを自動的に閉じる。
   * `height`: オプション： ログインポップアップウィンドウの高さを指定します。
   * `width`: ログインポップアップウィンドウの幅を指定します： ログインポップアップウィンドウの幅を指定します。
   * `top`: ログインポップアップウィンドウの上端を指定します： `top`: ログインポップアップウィンドウの上部を指定します。
   * `left`: ログインポップアップウィンドウの左端を指定します： ログインポップアップウィンドウの左端を指定します。
* `loginPopupAutoClose`： オプション。デフォルトは `false` である。`loginPopup` パラメータと一緒に `true` に設定すると、認証ステップが完了した後にログインポップアップウィンドウを自動的に閉じます。ログインページが新しいタブで開かれた場合、このパラメータはそのタブも自動的に閉じます。このパラメータは `loginOptions` で設定することもできる。
* `loginUrl`： オプション。SAML 認証の場合のように、ccp を開始する際にカスタム URL を使用できるようにする。[詳細はこちら](#saml-authentication) を参照ください。
* `softphone`： このオブジェクトはオプションで、コネクトのソフトフォン機能に関する設定を行うことができます。
  * `allowFramedSoftphone`： 通常、ソフトフォンのマイクとスピーカーのコンポーネントを iframe でホストすることはできません。これは、ソフトフォンが1つのウィンドウまたはタブでホストされている必要があるためです。ソフトフォン通話中に、ソフトフォンセッションをホストしているウィンドウを閉じてはいけません。`allowFramedSoftphone` が `true` の場合、ソフトフォンコンポーネントをこのウィンドウまたはタブでホストすることが許可される。
    `allowFramedSoftphone` が `false` の場合、[lily-rtc.js](https://github.com/aws/connect-rtc-js) パッケージをインポートし、 `connect.core.initCCP()` の後に `connect.core.initSoftphoneManager()` をコードに追加していることを確認してください。
  * `disableRingtone`： `disableRingtone`: このオプションを使うと、着信時に再生される内蔵着信音の音声を完全に無効にすることができる。
  * `ringtoneUrl`： `ringtoneUrl`: 着信音が無効化されていない場合、ユーザーがアクセスできるブラウザがサポートするオーディオファイルで着信音を上書きすることができます。
  * `disableEchoCancellation`: 着信音を無効にする： このオプションはChromeでのみ適用可能で、エコーキャンセルを無効にしてカスタムCCPまたは埋め込みCCPを初期化することができます。このオプションを `true` に設定すると、オートゲインコントロールを含む、Chrome による **すべての** 音声処理が無効になります。
    - 自動画質を承認する代替オプションについては、こちらのリンク https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/improve-call-quality-on-agent-workstations-in-amazon-connect-contact-centers.html を参照してください。
  - `allowFramedVideoCall`： 現在のところ、ビデオ通話は1つのウィンドウまたはタブ内でのみ可能です。`true`の場合、CCPはこのウィンドウまたはタブでビデオ通話を行い、エージェントはセキュリティプロファイルにビデオ通話許可が設定されていれば、自分のビデオを見たり、オンにしたりすることができます。`false`の場合、または何も設定されていない場合、CCPは音声通話のみを提供します。
   - `VDIPlatform`： このオプションは仮想デスクトップインターフェイスの統合にのみ適用されます。`VDIPlatform`：このオプションは、仮想デスクトップ・インターフェイスの統合にのみ適用されます。オプションは、列挙型`VDIPlatformType`を使用して指定できます。`allowFramedSoftphone` が `false` で、`VDIPlatform` を設定する場合は、このパラメータを `connect.core.initSoftphoneManager()` に渡すようにしてください。例えば、`connect.core.initSoftphoneManager({ VDIPlatform: "CITRIX" })` とします。
  - `allowEarlyGum`： `true`または指定しない場合、CCPはコンタクトが到着する前にエージェントのブラウザマイクのメディアストリームをキャプチャし、コールセットアップの待ち時間を短縮します。`false`の場合、CCPはコンタクトが到着した後にのみエージェントのメディアストリームをキャプチャします。
- `pageOptions`： このオブジェクトはオプションで、どの設定セクションを設定タブに表示するかを設定することができる。
  - `enableAudioDeviceSettings`: このオブジェクトはオプションである： `true` の場合、設定タブにエージェントのローカルマシンのオーディオ入出力デバイスを設定するセクションが表示されます。`false` または `pageOptions` が指定されていない場合、エージェントは設定タブからオーディオデバイスの設定を変更することはできません。
    - 重要: ブラウザとしてFirefoxを使用している場合、出力オーディオデバイスリストは空になり、CCPはコンピュータのデフォルトの出力オーディオ設定を使用します。
    - Audio Device Settingsの出力デバイスを有効にするには、Firefoxで`about:config`に移動して、`media.setsinkid.enabled`を有効にしてください。次に、`media.setsinkid.enabled`を検索し、trueに切り替えます。
  - `enableVideoDeviceSettings`： `true` の場合、設定タブにエージェントのローカルマシンのビデオ入力デバイスを設定するセクションが表示されます。`false` または `pageOptions` が指定されていない場合、エージェントは設定タブからビデオデバイスの設定を変更することができません。
  - enablePhoneTypeSettings`： true`の場合、または `pageOptions` が提供されない場合、設定タブにエージェントの電話タイプとデスクホン番号を設定するセクションが表示されます。false`の場合、エージェントは設定タブから電話タイプやデスクホン番号を変更することはできません。
- `shouldAddNamespaceToLogs`：CCPによって記録されるすべてのログの先頭に`[CCP]`を追加します。重要: 名前空間が先頭に追加される前に CCP によって作成されるログがいくつかあります。
- `ccpAckTimeout`： CCP が共有ワーカーに `SYNCHRONIZE` メッセージを送信した後、共有ワーカーからの `ACKNOWLEDGE` メッセージを待つべき時間を ms 単位で指定します。これは重要です。なぜなら、`ACKNOWLEDGE`メッセージは、共有ワーカーが初期化されている場合にのみCCPに送り返され、共有ワーカーはエージェントがログインしている場合にのみ初期化されるからです。さらに、このチェックは継続的に行われます。
- `ccpSynTimeout`： 初期化された場合、共有ワーカーが `ACKNOWLEDGE` を送り返すトリガーとなるはずです。このイベントは基本的に、共有ワーカーが初期化されたかどうか、別名エージェントがログインしているかどうかをチェックします。このチェックは継続的に行われます。
- `ccpLoadTimeout`： 最初の `SYNCHRONIZE` メッセージを送信するまでの待ち時間をミリ秒単位で指定します。ここでのユーザーエクスペリエンスは、最初のCCPの初期化時に、ログインフローがこのタイムアウトの分だけ遅れるということです。
  - 例として、このタイムアウトが10秒に設定されていた場合、ログインポップアップは10秒経過するまで開きません。

#### いくつかの注意点があります：
* iframeを配置する`container-div`を表示するか非表示にするか、または以下のようなCSSルールを適用することで、あらかじめ組み込まれたUIを表示したり非表示にしたりすることができます：
```css
#container-div iframe {
  display: none;
}
```
* CCP UIは、提供されたコンテナ要素の下にiframeでレンダリングされます。
  iframeはコンテナ要素を`width: 100%; height: 100%` で満たします。
  CCPのサイズをカスタマイズするには、コンテナ要素の幅と高さを設定してください。
* CCPはレスポンシブ（様々なサイズで使用できる）ように設計されています。
  最も小さいサイズは320px x 460pxです。
  良いユーザー体験のために、このサイズより小さくしないことをお勧めします。
* あなたのサイトに追加したCSSスタイルは、CCPに適用されません。
* チャットに特化した機能を使用する場合は、コードに[ChatJS](https://github.com/amazon-connect/amazon-connect-chatjs)も含めてください。
  ストリームがChatJSなしで使用できるように、リリース・ファイルからChatJSを省略しています。
  ストリームが ChatJS を必要とするのは、チャットに使用する場合だけです。ChatJS を含める場合、StreamsJS の後にインポートしないと AWS SDK の問題が発生することに注意してください（ChatJS は ConnectParticipant Service に依存しており、これは Streams AWS SDK にはありません）。
* 独自のビデオ機能を構築する場合は、[Amazon Chime SDK JS](https://github.com/aws/amazon-chime-sdk-js) もコードに含めてください。また、[Amazon Chime SDK Component Library React](https://github.com/aws/amazon-chime-sdk-component-library-react) をインクルードすることで、Reactですぐに使えるUIや状態管理コンポーネントを利用することができます。
* タスク機能を使用する場合は、[TaskJS](https://github.com/amazon-connect/amazon-connect-taskjs)をインクルードする必要があります。TaskJS は Streams の後にインポートしてください。
* WebRTC セッションにアクセスしてソフトフォン体験をさらにカスタマイズしたい場合は、[connect-rtc-js](https://github.com/aws/connect-rtc-js) を使用できます。connect-rtc-js と Streams の統合に関する詳しい説明は、connect-rtc-js readme を参照してください。
* 複数の CCP を同じブラウザコンテキストに埋め込むために `initCCP` を使用すべきではありません。バージョン2.0では、`initCCP`の後続の呼び出しが追加のCCPを埋め込むことを自動的に防止するチェックが追加されました。
  * 複数のCCPを同じページに埋め込むことは、それらが異なるコネクト・インスタンスに関連付けられ、異なるブラウザ・コンテキストに埋め込まれ、それらのWindowオブジェクトが異なる場合（例えば、異なるiframe内）に可能です。複数のCCPを同じWindowオブジェクトの下に埋め込んだり、同じブラウザコンテキストで `initCCP` を複数回呼び出したりすることはできません。
  * ページ全体でストリームを一度ロードする代わりに、各 iframe で別々にストリームをロードし、それぞれで別々に `initCCP` を呼び出す必要があります。
  * iframeの読み込みが完了したら、各iframeの`contentWindow.connect`プロパティを使ってStreamsオブジェクトにアクセスし、中に埋め込まれた特定のCCPに対してAPIコールを行うことができます。2つのコネクト・インスタンス（AとB）に対して、同じページに2回CCPを埋め込む例です：

```js
var frameA = document.createElement('iframe');
var frameB = document.createElement('iframe');
var contentDocumentA = [
       "<!DOCTYPE html>",
       "<meta charset='UTF-8'>",
       "<html>",
         "<head>",
           "<script type='text/javascript' src='https://cdn.jsdelivr.net/npm/amazon-connect-streams/release/connect-streams-min.js'>",
           "</script>",
         "</head>",
         "<body onload='init()' style='width:400px;height:800px'>",
           "<div id=containerDiv style='width:100%;height:100%'></div>",
           "<script type='text/javascript'>",
             "function init() {",
               "connect.core.initCCP(containerDiv, <initCCP parameters for instance A>);",
            "}",
           "</script>",
         "</body>",
       "</html>"
     ].join('');
var contentDocumentB = ...; // 上記と同じだが、initCCP パラメーターは例えば B
frameA.srcdoc = contentDocumentA;
frameB.srcdoc = contentDocumentB;
[frameA, frameB].forEach((frame) => {
  frame.allow = "microphone; autoplay; clipboard-write";
  frame.style = "width:400px;height:800px;margin:0;border:0;padding:0px;";
  frame.scrolling = "no";
  document.documentElement.append(frame); // 各CCPを表示させたい場所にフレームを追加することができる。
});

// iframeのロードを待ち、contentWindow.connectを各フレームのStreamsオブジェクトに設定する。
// iframeがStreamsをロードし終わるまで、contentWindow.connectは未定義になります。
var connectA = frameA.contentWindow.connect;
var connectB = frameB.contentWindow.connect;

connectA.contact(function(contact) { /* ... */ });
```

### エージェントがログアウトしているか、セッションが期限切れであることを判断するにはどうすればよいですか？
* `connect.core.onAuthFail()` を使用して、エージェントがブラウザに設定された認証情報を使用して Connect に認証できなかった場合に呼び出されるコールバック関数をサブスクライブできます。
```js
connect.core.onAuthFail(function(){
  // ログインが必要です。
  // ログイン用のボタンを表示するか、ログイン画面をポップアップします。
});
```

## `connect.core`

### `connect.core.onIframeRetriesExhausted()`
```js
connect.core.onIframeRetriesExhausted(() => {
  console.log("We have run out of retries to reload the CCP Iframe");
})
```
すべての再試行を試みた後、iframe の読み込みに失敗したときに呼び出されるコールバック関数をサブスクライブする。iframe の再試行（iframe ページの更新）は、`connect.EventType.ACK_TIMEOUT` が発生するたびにスケジュールされます。再試行がスケジュールされる前に `connect.EventType.ACKNOWLEDGE` イベントが発生した場合、再試行はキャンセルされます。予定されている再試行は6回までとする。これを使い切ると、`connect.EventType.ACK_TIMEOUT`イベントはスケジュールされた再試行をトリガーしません。

### `connect.core.onAuthorizeSuccess()`
```js
connect.core.onAuthorizeSuccess(() => {
  console.log("authorization succeeded! Hooray");
});
```
エージェント認証APIが成功したときに呼び出されるコールバック関数をサブスクライブします。

### `connect.core.onCTIAuthorizeRetriesExhausted()`
```js
connect.core.onCTIAuthorizeRetriesExhausted(() => {
  console.log("We have failed CTI API authorization multiple times and we are out of retries");
});
```
複数の認証型 CTI API 失敗が発生した場合に呼び出されるコールバック関数をサブスクライブします。このイベントが発生すると、CTI API の認証タイプ (401) の失敗がさらに発生しても、ストリームはユーザーの再認証を試みなくなります。CTI API はエージェント、コンタクト、および接続 API (具体的には `connect.ClientMethods` 列挙型の下にリストされているもの) であることに注意すること。したがって、認可に関連する問題があることをエージェントに示すことが賢明かもしれない。

### `connect.core.onAuthorizeRetriesExhausted()`
```js
connect.core.onAuthorizeRetriesExhausted(() => {
  console.log("We have failed the agent authorization api multiple times and we are out of retries");
});
```
複数のエージェント認証 api の失敗が発生したときに呼び出されるコールバック関数をサブスクライブします。このイベントが発生した後、ストリームは、さらにエージェントの認証 api の失敗が発生したときに、ユーザをログインにリダイレクトしようとしません。したがって、認可に関連する問題があることをエージェントに示すことが賢明かもしれない。

### `connect.core.terminate()`
```js
var containerDiv = document.getElementById("containerDiv");
connect.core.initCCP(containerDiv, { /* ... */ });

// some time later...
connect.core.terminate();
var iframe = containerDiv.firstElementChild; // コンテナ内に何もないと仮定する
containerDiv.removeChild(iframe);
```
Amazon Connect Streamsをユニット化します。呼び出されたサブスクリプション・メソッドをすべて削除します。

CCP iframeは削除されないので、手動で削除する必要があります。

### `connect.core.viewContact()`
```js
var contactId = new connect.Agent().getContacts()[0].getContactId();
connect.core.viewContact(contactId);
```
CCP ユーザーインターフェースで現在選択されているコンタクトを変更します。
エージェントが複数のチャットを同時に処理する場合に便利です。

### `connect.core.onViewContact()`
```js
connect.core.onViewContact(function(event) {
  var contactId = event.contactId;
  // ...
});
```
CCP で現在選択されているコンタクトが変更されるたびに開始されるコールバックをサブスクライブします。
このコールバックは UI 上でコンタクトが変更されたとき（つまり `click` イベントを経由したとき）、または `connect.core.viewContact()` を経由したときに呼び出されます。

より正確には、`onViewContact`は以下のシナリオで呼び出されます：

1. 新しい受信コンタクトがあり、CCP で現在開いている他のコンタクトがありません。これには、発信コンタクトと自動受信を使用して受信されたコンタクトの両方が含まれます。
2. コンタクトが閉じており、他に開いているコンタクトが少なくとも1つある。
    1. CCP は、連絡先リストの次の連絡先で `onViewContact` を呼び出します。
3. あるコンタクトが CCP でアクティブなコンタクトとして選択された。これは複数の方法で発生します。
    1. エージェントが、ネイティブ CCP または埋め込み CCP でその連絡先のタブをクリックする。
    2. ネイティブCCPまたは埋め込みCCPで[コンタクトを閉じる]ボタンがクリックされると、CCPは `onViewContact`を呼び出します。CCP は、連絡先リストの次の連絡先で `onViewContact` を呼び出します。これはシナリオ2と冗長であり、`onViewContact`が2回呼び出されることになることに注意してください。
4. ネイティブCCPまたは埋め込みCCPの「連絡先を承認」ボタンを使用して、新しい連絡先が承認されました。
5. カスタム実装で `connect.core.viewContact` が呼び出される。
6. `onViewContact`が空の文字列で呼び出される場合があります。これは、アクティブなコンタクトが設定されていないことを示します。これは、以下のシナリオで発生します：
    1. コンタクトを閉じるボタンがクリックされ、他にアクティブなコンタクトがない。
    2. エージェントがCCPの新しいチャネルをクリックした場合
        1. 注: この場合、`onViewContact` は、新しく選択されたチャネルのコンタクトで呼び出されます。

### `connect.core.onAuthFail()`
```js
connect.core.onAuthFail(function() { /* ... */ });
```
認証に失敗したときに開始するコールバックをサブスクライブします (例: SAML 認証)。

### `connect.core.onAccessDenied()`
```js
connect.core.onAccessDenied(function() { /* ... */ });
```
認可に失敗する（アクセスが拒否される）たびに開始されるコールバックをサブスクライブする。

### `connect.core.onSoftphoneSessionInit()`
```js
connect.core.onSoftphoneSessionInit(function({ connectionId }) {
  var softphoneManager = connect.core.getSoftphoneManager();
  if(softphoneManager){
    // access session
    var session = softphoneManager.getSession(connectionId); 
  }
});
```
新しい webrtc セッションが作成されるたびに開始するコールバックを購読します。rtc セッションの統計情報の処理に使用されます。

### `connect.core.getWebSocketManager()`
```js
// `connect.ChatSession` is defined by `amazon-connect-chatjs`
connect.ChatSession.create({
  type: connect.ChatSession.SessionTypes.AGENT,
  websocketManager: connect.core.getWebSocketManager()
  // ...
});
```
`WebSocket` マネージャを取得する。このメソッドは `amazon-connect-chatjs` と連携する場合にのみ使用する。
詳細は [amazon-connect-chatjs](https://github.com/amazon-connect/amazon-connect-chatjs) のドキュメントを参照すること。

### `connect.core.onInitialized()`
```js
connect.core.onInitialized(function() { /* ... */ });
```
CCP の初期化が完了したときに実行されるコールバックをサブスクライブします。

### `connect.core.getFrameMediaDevices()`
```js
  connect.core.getFrameMediaDevices(timeout)
  .then(function(devices) { /* ... */ })
  .catch(function(err) { /* ... */ })
```
iframe からメディアデバイスのリストを解決したプロミスを返します。
オプションの引数として、リクエストのタイムアウトを渡すことができます。デフォルトのタイムアウトは 1000ms です。
このAPIは、iframeのCCPの初期化が完了した後に呼び出す必要があります。

## SAML 認証
Streams は SAML 2.0 (Security Assertion Markup Language) をサポートし、シングルサインオン (SSO) を可能にします。これにより、ユーザーは SAML 2.0 互換の ID プロバイダ (IdP) を介してサインインし、別の認証情報を提供することなくインスタンスにアクセスできるようになります。
### SAML とストリームの使用
基本的なユースケースを説明する： 顧客は、SSO フローを開始するためのポップアウトウィンドウ/タブを開き、IdP を通してログインする。initCCP`](#initialization) 関数の内部には、SAML を設定する際に役立つオプションのパラメータがある。最初のパラメータは `loginUrl` であり、ユーザが自分の IdP を Streams と連携させるためのものである。その他のオプションパラメータには `loginPopup` (デフォルトは `true`) と `loginOptions` (`loginPopup` が `true` の場合にのみ使用される) がある。正しく設定されている場合、認証が必要になると、Streams はログイン URL が指定された新しいウィンドウ/タブを開き、バックグラウンドでは、顧客の認証情報が確認されるまで、非表示の CCP iFrame（ストリームに組み込まれています）が 5 秒ごとに更新されます。CCP が正常にロードされるには、CCP の初期化実行フローとともに、SAML フェデレーションが正常に完了する必要がある。
### 隠しiFrameでのSSO
上述したように、Streamsは現在、ポップアウトウィンドウ/タブでSSOフローを実行するユーザーを完全にサポートするように構築されています。同じウィンドウ内の隠されたiframeでSSOを実行したいユーザは、IdPがsame-origin-policyを含んでいる可能性があり、ユーザのドメインとIdP間の相互作用を妨げる可能性があることに注意してください。その場合、ユーザは上記の方法を取るか、iframeをSSOに委譲することでSSOフローを実行する必要があります。

**注意 SSO hidden iframe を削除したいユーザは、CCP の初期化と SAML フローの実行が成功するまで待ってください。その他の関数として、`connect.core.onAuthFail(callback)`（認証に失敗したときにコールバック関数をサブスクライブできる）、`connect.core.onAuthorizeRetriesExhausted(callback)`（複数のエージェントの認証に失敗したときにコールバック関数をサブスクライブできる）があります。

## イベントサブスクリプション
イベントサブスクリプションは、新しいエージェント情報が利用可能になったときにあなたのコードが呼び出されるようにすることで、あなたのアプリを Amazon Connect のハートビートにリンクします。

イベントサブスクリプションは、エージェントが初期化されたときと、コンタクトが最初に検出されたときに呼び出されるコールバックを Streams API に提供することで動作します。
次に、`on*()` イベントサブスクリプションメソッドは、特定のオブジェクトでイベントが発生したときに呼び出されるコールバックを提供します。これらはサブスクリプションオブジェクトを返し、コンタクトについては、それらのコンタクトが存在しなくなったときに自動的にクリーンアップされます。ユーザは、これらのメソッドによって返された購読オブジェクトに対して `sub.unsubscribe()` を呼び出すことで、手動でイベントの購読を解除することもできます。

### `connect.agent()`
```js
connect.agent(function(agent) { /* ... */ });
```
エージェントが初期化されたときに呼び出されるメソッドをサブスクライブします。エージェントがすでに初期化されている場合、呼び出しは同期で、コールバックはすぐに呼び出されます。そうでない場合は、最初のエージェントデータをアップストリームから受信した時点でコールバックが呼び出されます。このコールバックは `Agent` API オブジェクトと共に提供される、
このオブジェクトは初期化が完了した後、いつでも `new connect.Agent()` によって作成することができます。

### `connect.contact()`
```js
connect.contact(function(contact) { /* ... */ });
```
新しく検出されたエージェントコンタクトごとに呼び出されるメソッドをサブスクライブします。この関数は、受信コンタクトだけでなく、以前のエージェントセッションのコンタクトなど、Streams が初期化されたときに既に存在していたコンタクトも対象となることに注意してください。このコールバックは、このコンタクトの `Contact` API オブジェクトと共に提供されます。Contact` API オブジェクトは `Agent` API から `agent.getContacts()` を呼び出して取得することもできます。

### `connect.onWebSocketInitFailure()`
```
connect.onWebSocketInitFailure(function() { ... });
```
WebSocket 接続の初期化に失敗したときに呼び出されるメソッドをサブスクライブします。
WebSocket の初期化に一度でも失敗している場合は、同期的にコールバックが呼び出されます。 そうでない場合は、最初の初期化に失敗した時点でコールバックが呼び出されます。WebSocket 接続は必要に応じて定期的に更新されるため、WebSocket 接続の再初期化に失敗した場合もコールバックが呼び出されます。

## Agent API
エージェントAPIは、エージェントに代わって呼び出されるイベントサブスクリプションメソッドとアクションメソッドを提供します。Streamsのインスタンス化ごとにエージェントは1つだけ存在し、すべてのコンタクトとアクションはこの1つのエージェントに代わって実行されると仮定されます。
### `agent.onRefresh()`
```js
agent.onRefresh(function(agent) { /* ... */ });
```
新しいエージェントデータが利用可能になるたびに呼び出されるメソッドをサブスクライブします。

### `agent.onStateChange()`
```js
agent.onStateChange(function(agentStateChange) { /* ... */ });
```
エージェントの状態が変化したときに呼び出されるメソッドをサブスクライブする。`agentStateChange` オブジェクトは以下のプロパティを持つ：

* `agent`: The `Agent` API object.
* `oldState`: The name of the agent's previous state.
* `newState`: The name of the agent's new state.

### `agent.onRoutable()`
```js
agent.onRoutable(function(agent) { /* ... */ });
```
エージェントがルーティング可能になったときに呼び出されるメソッドをサブスクライブします。

### `agent.onNotRoutable()`
```js
agent.onNotRoutable(function(agent) { /* ... */ });
```
エージェントがルート不可になったときに呼び出されるメソッドをサブスクライブします。

### `agent.onOffline()`
```js
agent.onOffline(function(agent) { /* ... */ });
```
エージェントがオフラインになったときに呼び出されるメソッドをサブスクライブします。

### `agent.onError()`
```js
agent.onError(function(agent) { /* ... */ });
```
エージェントがエラー状態になったときに呼び出されるメソッドをサブスクライブします。これは、Streams が新しいエージェントデータを取得できない場合や、エージェントが受信コンタクトを受け付けない場合、その他のエラー時に発生します。これはエージェントがルーティング可能でないことを意味し、再度コンタクトをルーティングできるようになる前に、エージェントをルーティング可能な状態に切り替える必要があるかもしれません。

### `agent.onSoftphoneError()`
```js
agent.onSoftphoneError(function(error) { 
   console.log("Error type: ", error.errorType); 
   console.log("Error message: ", error.errorMessage); 
   console.log("Error endpoint url: ", error.endPointUrl);
});
```
エージェントがソフトフォン特有のエラー状態になったときに呼び出されるメソッドをサブスクライブします。

`error` 引数は `connect.SoftphoneError` インスタンスで、以下のフィールドを持つ： `errorType`、`errorMessage`、`endPointUrl` である。

### `agent.onWebSocketConnectionLost()`
```js
agent.onWebSocketConnectionLost(function(agent) { ... });
```
エージェントが WebSocket 接続を失うエラー状態になったときに呼び出されるメソッドをサブスクライブします。

### `agent.onWebSocketConnectionGained()`
```js
agent.onWebSocketConnectionGained(function(agent) { ... });
```
エージェントが WebSocket 接続を獲得したときに呼び出されるメソッドをサブスクライブします。

### `agent.onAfterCallWork()`
```js
agent.onAfterCallWork(function(agent) { /* ... */ });
```
エージェントが "After Call Work"(ACW)ステートに入ったときに呼び出されるメソッドをサブスクライブします(すべてのコンタクトがACWコンタクトステートに入ったとしても、このイベントは音声コンタクトに対してのみトリガされることに注意してください)。以下のcontact.onACWを参照してください)。これは、エージェントが追加のコンタクトをルーティングされる前に、コンタクトを処理した後にラップアップする時間を与えるために存在する、ルーティング不可能な状態です。

### `agent.getState()` / `agent.getStatus()`
```js
var state = agent.getState();
```
エージェントの現在の `AgentState` オブジェクトを取得します。
このオブジェクトには以下のフィールドが含まれる：

* `agentStateARN`： エージェントの現在の状態 ARN。
* `name`： エージェントの現在の可用性状態の名前。
* startTimestamp`: 状態が設定された日時を示す `Date` オブジェクト： 状態がいつ設定されたかを示す `Date` オブジェクト。
* `type`: 現在の利用可能状態のタイプ： AgentStateType` 列挙型に従った、エージェントの現在の可用性状態のタイプ。

このオブジェクトには、システムによって事前に定義された状態が含まれている可能性があります。エージェントのユーザ定義状態を取得するには、[`agent.getAvailabilityState()`](#agentgetavailabilitystate)を参照してください。

### `agent.getStateDuration()` / `agent.getStatusDuration()`
```js
var millis = agent.getStateDuration();
```
エージェントの状態の継続時間を、ローカル時間に対するミリ秒単位で取得します。これは、JSクライアントとAmazon Connectサービス間のタイムスキューを考慮しています。

### `agent.getPermissions()`
```js
var permissions = agent.getPermissions(); // e.g. ["outboundCall"]
```
ほとんどの場合、内部目的でのみ使用されます。エージェントが CCP 内で実行できるアクションを示す文字列が含まれます。

### `agent.getContacts()`
```js
var contacts = agent.getContacts(contactTypeFilter);
```
エージェントの現在の各連絡先の `Contact` API オブジェクトのリストを取得します。

* `contactTypeFilter`： オプション。指定した場合、指定した `ContactType` 列挙型のコンタクトのみを返す。

### `agent.getConfiguration()`
```js
var config = agent.getConfiguration();
```
エージェントの完全な `AgentConfiguration` オブジェクトを取得する。このオブジェクトは以下のフィールドを含む：

- `agentStates`を参照してください： 詳細は `agent.getAgentStates()` を参照。
- `dialableCountries`： 詳細は `agent.getDialableCountries()` を参照。
- `extension`： 詳細は `agent.getExtension()` を参照。
- `name`: 詳細は `agent.getName()` を参照。
- `permissions`: 詳細は `agent.getPermissions()` を参照すること： 詳細は `agent.getPermissions()` を参照。
- `routingProfile`: 詳細は `agent.getRoutingProfile()` を参照： 詳細は `agent.getRoutingProfile()` を参照。
- `softphoneAutoAccept`： ソフトフォンの自動着信を有効にするかどうかを示す。
- `softphoneEnabled`： 詳細は `agent.isSoftphoneEnabled()` を参照。
- `username`： Amazon Connect のユーザーアカウントに登録されているエージェントのユーザー名。
- `agentARN`: エージェントのユーザー名： 詳細は `agent.getAgentARN()` を参照。

### `agent.getAgentStates()`
```js
var agentStates = agent.getAgentStates();
```
選択可能な `AgentState` API オブジェクトのリストを取得する。これらはエージェントがライブコンタクトを処理していないときに選択できるエージェントの状態です。各 `AgentState` オブジェクトは以下のフィールドを含む：

* `AgentStateARN`： エージェントの状態 ARN。
* `type`： エージェントの状態タイプを `AgentStateType` 列挙型の値で表す。
* `name`: エージェント状態の名前： UI に表示するエージェント状態の名前。

### `agent.getAvailabilityState()`
```js
var agentState = agent.getAvailabilityState();
```
システムで定義された状態を返す [`agent.getState()`](#agentgetstate--agentgetstatus) とは異なり、この関数はエージェントの現在の [ユーザが変更可能/定義可能な状態](https://docs.aws.amazon.com/connect/latest/adminguide/agent-custom.html) だけを返します。
オブジェクトは以下のフィールドを含みます：

* `state`： エージェントの状態名。
* `timestamp`： エージェントがいつその状態に設定されたかを示す `Date` オブジェクト。

### `agent.getRoutingProfile()`
```js
var routingProfile = agent.getRoutingProfile();
```
エージェントのルーティングプロファイルを取得します。ルーティングプロファイルは以下のフィールドを含みます：

* `channelConcurrencyMap`： 詳細は `agent.getChannelConcurrency()` を参照。
* `defaultOutboundQueue`： アウトバウンドコンタクトに関連付けるデフォルトのキュー。プロパティの詳細については `queues` を参照。
* `name`: ルーティングプロファイルの名前： ルーティングプロファイルの名前。
* `queues`: ルーティングプロファイルの名前： ルーティングプロファイルに含まれるキュー。各キューオブジェクトは以下のプロパティを持ちます：
  * `name`： キューの名前。
  * `queueARN`： キューの ARN。
  * `queueId`: `queueARN` のエイリアス。
* `routingProfileARN`: ルーティングプロファイルの ARN。
* `routingProfileId`: `routingProfileARN` のエイリアス。

### `agent.getChannelConcurrency()`
```js
var concurrencyMap = agent.getChannelConcurrency(); // e.g. { VOICE: 0, CHAT: 2 }

// OR

if (agent.getChannelConcurrency("CHAT") > 1) { /* ... */ }
```
エージェントが与えられたチャネルでいくつの同時コンタクトを持つことができるかを示す数値または `ChannelType`-to-number のマップを取得します。0 は無効なチャンネルを表す。オプションの `channel` 引数は `ChannelType` 列挙型の値でなければならない。

### `agent.getName()`
```js
var name = agent.getName();
```
Gets the agent's user friendly display name.

### `agent.getExtension()`
```js
var extension = agent.getExtension();
```
Gets the agent's phone number. This is the phone number that is dialed by Amazon Connect to connect calls to the agent for incoming and outgoing calls if softphone is not enabled.

### `agent.getDialableCountries`
```js
var countries = agent.getDialableCountries(); // e.g. ["us", "ca"]
```
Returns a list of eligible countries to be dialed / deskphone redirected.

### `agent.isSoftphoneEnabled()`
```js
if (agent.isSoftphoneEnabled()) { /* ... */ }
```
Indicates whether the agent's phone calls should route to the agent's browser-based softphone or the telephone number configured as the agent's extension.

### `agent.getAgentARN()`

```js
const agentARN = agent.getAgentARN();
```

Gets the agent's ARN.

### `agent.setConfiguration()`
```js
var config = agent.getConfiguration();
config.extension = "+18005550100";
config.softphoneEnabled = false;
agent.setConfiguration(config {
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Updates the agent's configuration with the given `AgentConfiguration` object. The phone number specified must be in E.164 format or the update fails.
Only the `extension` and `softphoneEnabled` values can be updated.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `agent.setState()` / `agent.setStatus()`
```js
var state = agent.getAgentStates()[0];
agent.setState(state, {
    success: function() { /* ... */ },
    failure: function(err) { /* ... */ },
   },
   {enqueueNextState: false}
);
```
Set the agent's current availability state. Can only be performed if the agent is not handling a live contact.

Optional `enqueueNextState` flag can be passed to trigger Next Status behavior.
By default this is false.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `agent.onEnqueuedNextState()`
```js
agent.onEnqueuedNextState(function(agent) { /* ... */ });
Subscribe a method to be called when an agent has enqueued a next status.
```

### `agent.getNextState()`
```js
var state = agent.getNextState();

Get the AgentState object of the agent's enqueued next status. 
If the agent has not enqueued a next status, returns null.
```

### `agent.connect()`
```js
var endpoint = connect.Endpoint.byPhoneNumber("+18005550100");
var agent = new connect.Agent();
var queueArn = "arn:aws:connect:<REGION>:<ACCOUNT_ID>:instance/<CONNECT_INSTANCE_ID>/queue/<CONNECT_QUEUE_ID>";

agent.connect(endpoint, {
  queueARN: queueArn,
  success: function() { console.log("outbound call connected"); },
  failure: function(err) {
    console.log("outbound call connection failed");
    console.log(err);
  }
});
```
Creates an outbound contact to the given endpoint. Only phone number endpoints are supported. You can optionally provide a `queueARN` to associate the contact with a queue.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `agent.getAllQueueARNs()`
```js
var ARNs = agent.getAllQueueARNs();
```
Returns a list of the ARNs associated with this agent's routing profile's queues.

### `agent.getEndpoints()` / `agent.getAddresses()`
```js
var queuesARNs = agent.getAllQueueARNs();
agent.getEndpoints(
   queuesARNs,
   {
      success: function(data) {
         var endpoints = data.endpoints; // or data.addresses
      },
      failure: function(err) {
      }
   }
);
```
Returns the endpoints associated with the queueARNs specified in `queueARNs`.
* `queueARNs`: Required. Can be a single Queue ARN or a list of Queue ARNs associated with the desired queues.
* `callbacks`: Optional. A structure containing success and failure handlers.
   * `success`: A function for handling a successful API call.
   * `failure`: A function for handling a failed API call.

### `agent.toSnapshot()`
```js
var snapshot = agent.toSnapshot();
```
The data behind the `Agent` API object is ephemeral and changes whenever new data is provided. This method provides an opportunity to create a snapshot version of the `Agent` API object and save it for future use, such as adding to a log file or posting elsewhere.

### `agent.mute()`
```js
agent.mute();
```
Sets the agent local media to mute mode. If `Enhanced monitoring & Mutiparty` is enabled, use `voiceConnection.muteParticipant()` when there are more than 2 agent connections (see example [here](cheat-sheet.md#mute-agent)), since `voiceConnection.muteParticipant()` will mute the connection on the server side, and the server side mute value is the only one accounted for when there are more than 2 connections.

### `agent.unmute()`
```js
agent.unmute();
```
Sets the agent localmedia to unmute mode. If `Enhanced monitoring & Mutiparty` is enabled, use `voiceConnection.unmuteParticipant()` when there are more than 2 agent connections (see example [here](cheat-sheet.md#mute-agent)), since `voiceConnection.unmuteParticipant()` will unmute the connection on the server side, and the server side mute value is the only one accounted for when there are more than 2 connections.


### `agent.setSpeakerDevice()`
```js
agent.setSpeakerDevice(deviceId);
```
Sets the speaker device (output device for call audio)

### `agent.setMicrophoneDevice()`
```js
agent.setMicrophoneDevice(deviceId);
```
Sets the microphone device (input device for call audio)
The microphone device can be changed while agent is having a softphone call with a microphone media stream attached to it.
To properly set a microphone device for every softphone call, you can call this API in the callback function passed to onLocalMediaStreamCreated API.

```js
agent.onLocalMediaStreamCreated(() => {
  agent.setMicrophoneDevice(deviceId);
});
```

### `agent.setRingerDevice()`
```js
agent.setRingerDevice(deviceId);
```
Sets the ringer device (output device for ringtone)

### `agent.onLocalMediaStreamCreated()`

```js
agent.onLocalMediaStreamCreated((data) => {
  console.log('local media stream created for connection: ', data.connectionId);
});
```

Subscribe a method to be called when the agent's microphone media stream is attached to the WebRTC connection for a softphone call, which happens between connect.ContactEvents.CONNECTING and connect.ContactEvents.CONNECTED events.
This API is useful to do operations that require the local media stream such as setMicrophoneDevice and mute/unmute before CONNECTED event.

### `agent.onMuteToggle()`
```js
agent.onMuteToggle(function(obj) {
   if (obj.muted) { /* ... */ }
});
```
Subscribe a method to be called when the agent updates the mute status, meaning
that agents mute/unmute APIs are called and the local media stream is successfully updated with the new status.

### `agent.onSpeakerDeviceChanged()`
```js
agent.onSpeakerDeviceChanged(function(obj) { /* ... */ });
```
Subscribe a method to be called when the agent changes the speaker device (output device for call audio).

### `agent.onMicrophoneDeviceChanged()`
```js
agent.onMicrophoneDeviceChanged(function(obj) { /* ... */ });
```
Subscribe a method to be called when the agent changes the microphone device (input device for call audio).

### `agent.onRingerDeviceChanged()`
```js
agent.onRingerDeviceChanged(function(obj) { /* ... */ });
```
Subscribe a method to be called when the agent changes the ringer device (output device for ringtone).

### `agent.onCameraDeviceChanged()`
```js
agent.onCameraDeviceChanged(function(obj) { /* ... */ });
```
Subscribe a method to be called when the agent changes the camera device (input device for call video).

### `agent.onBackgroundBlurChanged()`
```js
agent.onBackgroundBlurChanged(function(obj) { /* ... */ });
```
Subscribe a method to be called when the agent enables or disables background blur for the camera device (input device for call video).

## Contact API
The Contact API provides event subscription methods and action methods which can be called on behalf of a specific contact. Contacts come and go and so should these API objects. It is good practice not to persist these objects or keep them as internal state. If you need to, store the `contactId` of the contact and make sure that the contact still exists by fetching it from the `Agent` API object before calling methods on it.

### `contact.onRefresh()`
```js
contact.onRefresh(function(contact) { /* ... */ });
```
Subscribe a method to be invoked whenever the contact is updated.

### `contact.onIncoming()`
```js
contact.onIncoming(function(contact) { /* ... */ });
```
Subscribe a method to be invoked when a queue callback contact is incoming. In this state, the contact is waiting to be accepted if it is a softphone call or is waiting for the agent to answer if it is not a softphone call.

### `contact.onPending()`
```js
contact.onPending(function(contact) { /* ... */ });
```
Subscribe a method to be invoked when the contact is pending. This event is expected to occur before the connecting event.

### `contact.onConnecting()`
```js
contact.onConnecting(function(contact) { /* ... */ });
```
Subscribe a method to be invoked when the contact is connecting. This event happens when a contact comes in, before accepting (there is an exception for queue callbacks, in which onConnecting's handler is started after the callback is accepted). Note that once the contact has been accepted, the `onAccepted` handler will be triggered.

### `contact.onAccepted()`
```js
contact.onAccepted(function(contact) { /* ... */ });
```

Subscribe a method to be invoked whenever the contact is accepted. Please note that this event doesn't get triggered for agents using a deskphone.

### `contact.onMissed()`
```js
contact.onMissed(function(contact) { /* ... */ });
```
Subscribe a method to be invoked whenever the contact is missed. This is an event which is fired when a contact is put in state "missed" by the backend, which happens when the agent does not answer for a certain amount of time, when the agent rejects the call, or when the other participant hangs up before the agent can accept.

### `contact.onEnded()`
```js
contact.onEnded(function(contact) { /* ... */ });
```
Subscribe a method to be invoked whenever the contact is ended. This could be due to the conversation being ended by the agent, or due to the contact being missed. Call `contact.getState()` to determine the state of the contact and take appropriate action.

[Update on v2.7.0]
The callback function registered via `contact.onEnded ` is no longer invoked when the contact is destroyed. This fix prevents the callback from being invoked twice on ENDED and DESTROYED events.

### `contact.onDestroy()`
```js
contact.onDestroy(function(contact) { /* ... */ });
```
Subscribe a method to be invoked whenever the contact is destroyed.

### `contact.onACW()`
```js
contact.onACW(function(contact) { /* ... */ });
```
Subscribe a method to be invoked whenever the contact enters the ACW state, named `ContactStateType.ENDED`. This is after the connection has been closed, but before the contact is destroyed.

### `contact.onConnected()`
```js
contact.onConnected(function(contact) { /* ... */ });
```
Subscribe a method to be invoked when the contact is connected.

### `contact.onError()`
```js
contact.onError(function(contact) { /* ... */ });
```
Subscribe a method to be invoked when `connect.ContactEvents.ERROR` happens. 
This event happens when the *contact* state type is `error`. This is a new change as of version 1.7.6, this api used to not function at all due to the manner in which we were launching contact events. 

**NOTE**
This description used to read "This event happens when the *agent* state type is `error`. 
The agent state type description was never accurate, as the event was never triggered, even though the source code's intention was to trigger the event on an error agent state type.

### `contact.getEventName()`
```js
// e.g. contact::connected::01234567-89ab-cdef-0123-456789abcdef
var eventName = contact.getEventName(connect.ContactEvents.CONNECTED);
```
Returns a formatted string with the contact event and ID. The argument must be a value of the `ContactEvents` enum.

### `contact.getContactId()`
```js
var contactId = contact.getContactId();
```
Get the unique contactId of this contact.

### `contact.getOriginalContactId()` / `contact.getInitialContactId()`
```js
var originalContactId = contact.getOriginalContactId();
//OR
var initialContactId = contact.getInitialContactId();
```
Get the original (initial) contact id from which this contact was transferred, or none if this is not an internal Connect transfer.
This is typically a contact owned by another agent, thus this agent will not be able to manipulate it. It is for reference and association purposes only, and can be used to share data between transferred contacts externally if it is linked by originalContactId.

Note that `contact.getOriginalContactId()` is the original Streams API name, but it does not match the internal data naming scheme, which is `initialContactId`, so we added an alias for the same method called `contact.getInitialContactId()`.

### `contact.getType()`
```js
var type = contact.getType();
```
Get the type of the contact. This indicates what type of media is carried over the connections of the contact.

###  `contact.getState()` / `contact.getStatus()`
```js
var state = contact.getState();
```
Get an object representing the state of the contact. This object has the following fields:

* `type`: The contact state type, as per the `ContactStateType` enumeration.
* `timestamp`: A `Date` object that indicates when the the contact was put in that state.

### `contact.getStateDuration()` / `contact.getStatusDuration()`
```js
var millis = contact.getStateDuration();
```
Get the duration of the contact state in milliseconds relative to local time. This takes into account time skew between the JS client and the Amazon Connect backend servers.

### `contact.getQueue()`
```js
var queue = contact.getQueue();
```
Get the queue associated with the contact. This object has the following fields:

* `name`: The name of the queue.
* `queueARN`: The ARN of the queue.
* `queueId`: Alias for `queueARN`.

### `contact.getQueueTimestamp()`
```js
var queueTimestamp = contact.getQueueTimestamp();
```
Get a `Date` object with the timestamp associated with when the contact was placed in the queue.

### `contact.getConnections()`
```js
var conns = contact.getConnections();
```
Get a list containing `Connection` API objects for each connection in the contact.

### `contact.getInitialConnection()`
```js
var initialConn = contact.getInitialConnection();
```
Get the initial connection of the contact.

### `contact.getActiveInitialConnection()`
```js
var initialConn = contact.getActiveInitialConnection();
```
Get the initial connection of the contact, or null if the initial connection is
no longer active.

### `contact.getThirdPartyConnections()`
```js
var thirdPartyConns = contact.getThirdPartyConnections();
```
Get a list of all of the third-party connections, i.e. the list of all connections except for the initial connection, or an empty list if there are no third-party connections.

### `contact.getSingleActiveThirdPartyConnection()`
```js
var thirdPartyConn = contact.getSingleActiveThirdPartyConnection();
```
In Voice contacts, there can only be one active third-party connection. This method returns the single active third-party connection, or null if there are no currently active third-party connections.

### `contact.getAgentConnection()`
```js
var agentConn = contact.getAgentConnection();
```
Gets the agent connection. This is the connection that represents the agent's participation in the contact.

### `contact.getActiveConnections()`

```js
var activeConns = contact.getActiveConnections();
```

Get a list of all active connections, i.e. connections that have not been disconnected.

### `contact.hasTwoActiveParticipants()`

```js
if (contact.hasTwoActiveParticipants()) {
  /* ... */
}
```

Determines if there are exactly two active participants on a contact.

When using enhanced monitoring & barge, if there are only two active participants on a contact, and one of them is the manager who has barged in, the manager cannot switch back to monitoring mode.

### `contact.getAttributes()`
```js
var attributeMap = contact.getAttributes(); // e.g. { "foo": { "name": "foo", "value": "bar" } }
```
Gets a map of the attributes associated with the contact. Each value in the map has the following shape: `{ name: string, value: string }`.

Please note that this api method will return null when the current user is monitoring the contact, rather than being an active participant in the contact.

### `contact.getSegmentAttributes()`

```js
var segmentAttributes = contact.getSegmentAttributes(); // e.g. { "connect:Direction": { "ValueString": "INBOUND" }, "connect:Subtype": { "ValueString": "connect:WebRTC" } }
```

Gets a map of segment attributes associated with the contact. The current possible segment attributes are `connect:Subtype` and `connect:Direction`. Calling `getContactSubtype()` on a given contact will return the same value as calling `segmentAttributes['connectSubtype']`

### `contact.getContactSubtype()`

```js
var contactSubtype = contact.getContactSubtype();
```

Get the subtype of the contact. This information is also present in segmentAttributes under the key `connect:Subtype`.

This further differntiates the contact within a contact Type. Currently it's used to distinguish chat and voice contacts. For example:

- Voice calls started using StartWebRTCContact has a subType value of `connect:WebRTC`
- SMS chat contacts contain the subType value of `connect:SMS`

### `contact.isSoftphoneCall()`
```js
if (contact.isSoftphoneCall()) { /* ... */ }
```
Determine whether this contact is a softphone call.

### `contact.hasVideoRTCCapabilities()`

```js
if (contact.hasVideoRTCCapabilities()) {
  /* ... */
}
```

Determine whether this contact has video capabilities.

### `contact.canAgentSendVideo()`

```js
if (contact.canAgentSendVideo()) {
  /* ... */
}
```

Determine whether the agent in this contact can send video.

### `contact.canAgentReceiveVideo()`

```js
if (contact.canAgentReceiveVideo()) {
  /* ... */
}
```

Determine whether the agent in this contact can receive video.

### `contact.isInbound()`
```js
if (contact.isInbound()) { /* ... */ }
```
Determine whether this is an inbound or outbound contact.

### `contact.isConnected()`
```js
if (contact.isConnected()) { /* ... */ }
```
Determine whether the contact is in a connected state.

Note that contacts no longer exist once they have been removed. To detect these instances, subscribe to the `contact.onEnded()` event for the contact.

### `contact.accept()`
```js
contact.accept({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Accept an incoming contact.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.destroy()`
This method is now deprecated.

### `contact.reject()`
```js
contact.reject({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Reject an incoming contact.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.clear()`
```js
contact.clear({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
This is a more generic form of `contact.complete()`. Use this for voice, chat, and task contacts to clear the contact when the contact is no longer actively being worked on (i.e. it's one of ERROR, ENDED, MISSED, REJECTED). 
It works for both monitoring and non-monitoring connections.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.complete()` (DEPRECATED)
```js
contact.complete({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
This API will soon be deprecated and should be replaced with `contact.clear()`. It completes the contact entirely.
That means it should only be used for non-monitoring agent connections.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.notifyIssue()`
```js
contact.notifyIssue(issueCode, description, {
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Provide diagnostic information for the contact in the case something exceptional happens on the front end.
The Streams logs will be published along with the issue code and description provided here.

* `issueCode`: An arbitrary issue code to associate with the diagnostic report.
* `description`: A description to associate with the diagnostic report.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.addConnection()`
```js
var endpoint = Endpoint.byPhoneNumber("+18005550100");
contact.addConnection(endpoint, {
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Add a new outbound third-party connection to this contact and connect it to the specified endpoint.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.toggleActiveConnections()`
```js
contact.toggleActiveConnections({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Rotate through the connected and on hold connections of the contact. This operation is only valid if there is at least one third-party connection and the initial connection is still connected.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.conferenceConnections()`
```js
contact.conferenceConnections({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Conference together the active connections of the conversation. This operation is only valid if there is at least one third-party connection and the initial connection is still connected.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.toSnapshot()`
```js
var snapshot = contact.toSnapshot();
```
The data behind the `Contact` API object is ephemeral and changes whenever new data is provided. This method provides an opportunity to create a snapshot version of the `Contact` API object and save it for future use, such as adding to a log file or posting elsewhere.

### `contact.isMultiPartyConferenceEnabled()`
```js
if (contact.isMultiPartyConferenceEnabled()) { /* ... */ }
```
Determine whether this contact is a softphone call and multiparty conference feature is turned on.

### `contact.isUnderSupervision()`

```js
if (contact.isUnderSupervision()) {
  /* ... */
}
```

Determines if the contact is under manager's supervision, meaning there is another agent on the contact that is in barge mode.

### `contact.updateMonitorParticipantState()`

```js
contact.updateMonitorParticipantState(targetState, {
  success: function () {
    /* ... */
  },
  failure: function (err) {
    /* ... */
  },
});
```

Updates the monitor participant state to switch between different monitoring modes.

- `targetState`: The target monitoring state type, as per the `MonitoringMode` enumeration.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.silentMonitor()`

```js
contact.silentMonitor({
  success: function () {
    /* ... */
  },
  failure: function (err) {
    /* ... */
  },
});
```

Updates the monitor participant state to silent monitoring mode.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.bargeIn()`

```js
contact.bargeIn({
  success: function () {
    /* ... */
  },
  failure: function (err) {
    /* ... */
  },
});
```

Updates the monitor participant state to barge mode.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### Task Contact APIs
The following contact methods are currently only available for task contacts.

### `contact.getName()`
```js
var taskName = contact.getName();
```
Gets the name of the contact.

### `contact.getDescription()`
```js
var taskDescription = contact.getDescription();
```
Gets the description of the contact.

### `contact.getReferences()`
```js
var taskReferences = contact.getReferences();
```
Gets references for the contact. A sample reference looks like the following:

```js
"Reference-Name": {
    type: "URL",
    value: "https://link.com"
}
```

### `contact.getChannelContext()`
```js
var channelContext = contact.getChannelContext();
```
Gets the channel context for the contact. For task contacts the channel context contains `scheduledTime`,  `taskTemplateId`, `taskTemplateVersion` properties. It might look like the following:

```js
{
  scheduledTime: 1646609220
  taskTemplateId: "ba6db758-d31e-4c6c-bd51-14d8b4686ece"
  taskTemplateVersion: 1
}
```

### `contact.pause()`

```js
contact.pause({
  success: function () {
    /* ... */
  },
  failure: function (err) {
    /* ... */
  },
});
```

Pause a connected contact.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `contact.resume()`

```js
contact.resume({
  success: function () {
    /* ... */
  },
  failure: function (err) {
    /* ... */
  },
});
```

Resume a paused contact.

Optional success and failure callbacks can be provided to determine if the operation was successful.


## Connection API
The Connection API provides action methods (no event subscriptions) which can be called to manipulate the state of a particular connection within a contact. Like contacts, connections come and go. It is good practice not to persist these objects or keep them as internal state. If you need to, store the `contactId` and `connectionId` of the connection and make sure that the contact and connection still exist by fetching them in order from the `Agent` API object before calling methods on them.

### `connection.getContactId()`
```js
var contactId = connection.getContactId();
```
Gets the unique contactId of the contact to which this connection belongs.

### `connection.getConnectionId()`
```js
var connectionId = connection.getConnectionId();
```
Gets the unique connectionId for this connection.

### `connection.getEndpoint()` / `connection.getAddress()`
```js
var endpoint = connection.getEndpoint();
```
Gets the endpoint to which this connection is connected.

### `connection.getState()` / `connection.getStatus()`
```js
var state = connection.getState();
```
Gets the `ConnectionState` object for this connection. This object has the following fields:

* `timestamp`: A `Date` object that indicates when the the connection was put in that state.
* `type`: The connection state type, as per the `ConnectionStateType` enumeration.

### `connection.getStateDuration()` / `connection.getStatusDuration()`
```js
var millis = connection.getStateDuration();
```
Get the duration of the connection state, in milliseconds, relative to local time.
This takes into account time skew between the JS client and the Amazon Connect service.

### `connection.getType()`
```js
var type = connection.getType()
```
Get the type of connection. This value is an `ConnectionType` enum.

### `connection.getMonitorInfo()`
```js
var monitorInfo = conn.getMonitorInfo();
```
Get the currently monitored contact info, or null if that does not exist.
* `monitorInfo` = `{ agentName: string, customerName: string, joinTime: string }`

### `connection.isInitialConnection()`
```js
if (conn.isInitialConnection()) { /* ... */ }
```
Determine if the connection is the contact's initial connection.

### `connection.isActive()`
```js
if (conn.isActive()) { /* ... */ }
```
Determine if the contact is active. The connection is active it is incoming, connecting, connected, or on hold.

### `connection.isConnected()`
```js
if (conn.isConnected()) { /* ... */ }
```
Determine if the connection is connected, meaning that the agent is live in a conversation through this connection. Please note that `ConnectionStateType.SILENT_MONITOR` and `ConnectionStateType.BARGE` are considered connected as well.
   
Note that, in the case of Agent A transferring a contact to Agent B, the new (third party) agent connection will be marked as `connected` (`connection.isConnected` will return true) as soon as the contact is routed to Agent B's queue, not when Agent B actually is "live" and able to communicate in the conversation.

### `connection.isConnecting()`
```js
if (conn.isConnecting()) { /* ... */ }
```
Determine whether the connection is in the process of connecting.

### `connection.isOnHold()`
```js
if (conn.isOnHold()) { /* ... */ }
```
Determine whether the connection is on hold.

### `connection.destroy()`
```js
conn.destroy({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Ends the connection. This can be used to reject contacts, end live contacts, and clear chat ACW.
At this point, it should be used to reject contacts and end live contacts only. We are deprecating the behavior of clearing ACW with this API.
To clear ACW for voice and chat contacts, use the `contact.clear()` API.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `connection.sendDigits()`
```js
conn.sendDigits(digits, {
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```

Send a digit or string of digits through this connection.

This is only valid for contact types that can accept digits, currently this is limited to softphone-enabled voice contacts.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `connection.hold()`
```js
conn.hold({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Put this connection on hold. In two-party calls, it only should be called on the 'initialConnection' (aka customer connection), not on the agent connection. For calls with three or more parties, hold can be used on agent connection. Using hold on a customer connection will ensure that customer doesn't hear the rest of the participants, while using hold on an agent connection will keep customer's ability to speak with other participants, while the agent cannot be heard.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `connection.resume()`
```js
conn.resume({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Resume this connection if it was on hold.

Optional success and failure callbacks can be provided to determine if the operation was successful.

## VoiceConnection API
The VoiceConnection API provides action methods (no event subscriptions) which can be called to manipulate the state of a particular voice connection within a contact. Like contacts, connections come and go. It is good practice not to persist these objects or keep them as internal state. If you need to, store the `contactId` and `connectionId` of the connection and make sure that the contact and connection still exist by fetching them in order from the `Agent` API object before calling methods on them.

### `voiceConnection.getMediaInfo()`
```js
var mediaInfo = conn.getMediaInfo();
```
Returns the media info object associated with this connection.

### `voiceConnection.getMediaType()`
```js
if (conn.getMediaType() === "softphone") { ... }
```
Returns the `MediaType` enum value: `"softphone"`.

### `voiceConnection.getMediaController()`
```js
conn.getMediaController().then(function (voiceController) { /* ... */ });
```
Gets a `Promise` with the media controller associated with this connection.

This method is currently a placeholder.
The promise resolves to the return value of `voiceConnection.getMediaInfo()` but it will be replaced with the controller eventually.

### `voiceConnection.getQuickConnectName()`
```js
var agentName = conn.getQuickConnectName();
```
Returns the quick connect name of the third-party call participant with which the connection is associated.

### `voiceConnection.isMute()`
```js
if (conn.isMute()) { /* ... */ }
```
Determine whether the connection is mute server side.

### `voiceConnection.muteParticipant()`
```js
conn.muteParticipant({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Mute the connection server side. 
#### Multiparty call
Any agent participant can mute another agent participant. 

#### Supervisor barges into the call
Agents can mute themselves, but cannot mute other agents or supervisor.  

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `voiceConnection.unmuteParticipant()`
```js
conn.unmuteParticipant({
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```

Unmute the connection server side.

### `voiceConnection.canSendVideo()`

```js
if (conn.canSendVideo()) {
  /* ... */
}
```

Determine whether agent/customer in this connection can send video.

### `voiceConnection.getCapabilities()`

```js
var capabilities = conn.getCapabilities(); // e.g. { "Video": "SEND" }
```

Returns the capabilities associated with this connection, or `null` if that does not exist.

### `voiceConnection.getVideoConnectionInfo()`

```js
conn
  .getVideoConnectionInfo()
  .then(function (response) {
    /* ... */
  })
  .catch(function (error) {
    /* ... */
  });
```

Example response:

```js
{
  "attendee": {
      "attendeeId": "attendeeId",
      "joinToken": "joinToken"
  },
  "meeting": {
      "mediaPlacement": {
          "audioFallbackUrl": "audioFallbackUrl",
          "audioHostUrl": "audioHostUrl",
          "eventIngestionUrl": "eventIngestionUrl",
          "signalingUrl": "signalingUrl",
          "turnControlUrl": "turnControlUrl"
      },
      "mediaRegion": "us-east-1",
      "meetingFeatures": null,
      "meetingId": "meetingId"
  }
}
```

Provides a promise which resolves with the API response from createTransport transportType web_rtc for this connection.
You can use the meeting and attendee info to join a video session. Please refer to
[Amazon Chime SDK JS](https://github.com/aws/amazon-chime-sdk-js/blob/main/README.md) for more info.

#### Multiparty call

Any agent can only unmute themselves.

#### Supervisor barges into the call
Agents can only unmute themselves up until the point they have been muted by the supervisor (isForcedMute API can help checking that). Once they have been muted by the supervisor, agent cannot unmute themselves until supervisor unmutes agent (at which point agent will regain ability to mute and unmute themselves). If supervisor has muted but not unmuted agent then drops from call, agent will be able to unmute themselves once supervisor has dropped.

Optional success and failure callbacks can be provided to determine if the operation was successful.

### `voiceConnection.isSilentMonitor()`

```js
if (conn.isSilentMonitor()) {
  /* ... */
}
```

Determine whether the connection is in silent monitoring state. Only the supervisor will see this connection in the snapshot, other agents will not see the supervisor's connection in the snapshot while it is in silent monitor state.

### `voiceConnection.isBarge()`

```js
if (conn.isBarge()) {
  /* ... */
}
```

Determine whether the connection is in barge state. All agents will see the supervisor's connection in the snapshot while it is in barge state.

### `voiceConnection.isSilentMonitorEnabled()`

```js
if (conn.isSilentMonitorEnabled()) {
  /* ... */
}
```

Determines if the agent has the ability to enter silent monitoring state, meaning the agent's monitoringCapabilities contain `MonitoringMode.SILENT_MONITOR` type.

### `voiceConnection.isBargeEnabled()`

```js
if (conn.isBargeEnabled()) {
  /* ... */
}
```

Determines if the agent has the ability to enter barge state, meaning the agent's monitoringCapabilities contain `MonitoringMode.BARGE` type.

### `voiceConnection.getMonitorCapabilities()`

```js
var monitorCapabilities = conn.getMonitorCapabilities();
```

Returns the array of enabled monitor states of this connection. The array will consist of MonitoringMode enum values.

### `voiceConnection.getMonitorStatus()`

```js
var monitorStatus = conn.getMonitorStatus();
```

Returns the current monitoring state of this connection. This value can be one of MonitoringMode enum values if the agent is supervisor, otherwise the monitorStatus will be undefined for the agent.

## ChatConnection API
The ChatConnection API provides action methods (no event subscriptions) which can be called to manipulate the state of a particular chat connection within a contact. Like contacts, connections come and go. It is good practice not to persist these objects or keep them as internal state. If you need to, store the `contactId` and `connectionId` of the connection and make sure that the contact and connection still exist by fetching them in order from the `Agent` API object before calling methods on them.

### `chatConnection.getMediaInfo()`
```js
var mediaInfo = conn.getMediaInfo();
```
Get the media info object associated with this connection.

### `chatConnection.getConnectionToken()`
```js
conn.getConnectionToken()
  .then(function(response) { /* ... */ })
  .catch(function(error) { /* ... */ });
```
Provides a promise which resolves with the API response from createTransport transportType chat_token for this connection.

### `chatConnection.getMediaType()`
```js
if (conn.getMediaType() === "chat") { /* ... */ }
```
Returns the `MediaType` enum value: `"chat"`.

### `chatConnection.getMediaController()`
```js
conn.getMediaController().then(function (chatController) { /* ... */ });
```
Gets a `Promise` with the media controller associated with this connection.
The promise resolves to a `ChatSession` object from `amazon-connect-chatjs` library.
See the [amazon-connect-chatjs documentation](https://github.com/amazon-connect/amazon-connect-chatjs) for more information.

### `chatConnection.isSilentMonitor()`

```js
if (conn.isSilentMonitor()) {
  /* ... */
}
```

Determine whether the connection is in silent monitoring state. Only the supervisor will see this connection in the snapshot, other agents will not see the supervisor's connection in the snapshot while it is in silent monitor state.

### `chatConnection.isBarge()`

```js
if (conn.isBarge()) {
  /* ... */
}
```

Determine whether the connection is in barge state. All agents will see the supervisor's connection in the snapshot while it is in barge state.

### `chatConnection.isSilentMonitorEnabled()`

```js
if (conn.isSilentMonitorEnabled()) {
  /* ... */
}
```

Determines if the agent has the ability to enter silent monitoring state, meaning the agent's monitoringCapabilities contain `MonitoringMode.SILENT_MONITOR` type.

### `chatConnection.isBargeEnabled()`

```js
if (conn.isBargeEnabled()) {
  /* ... */
}
```

Determines if the agent has the ability to enter barge state, meaning the agent's monitoringCapabilities contain `MonitoringMode.BARGE` type.

### `chatConnection.getMonitorCapabilities()`

```js
var monitorCapabilities = conn.getMonitorCapabilities();
```

Returns the array of enabled monitor states of this connection. The array will consist of MonitoringMode enum values.

### `chatConnection.getMonitorStatus()`

```js
var monitorStatus = conn.getMonitorStatus();
```

Returns the current monitoring state of this connection. This value can be one of MonitoringMode enum values　if the agent is supervisor, otherwise the monitorStatus will be undefined for the agent.

## TaskConnection API
The TaskConnection API provides action methods (no event subscriptions) which can be called to manipulate the state　of a particular task connection within a contact. Like contacts, connections come and go. It is good practice not to persist these objects or keep them as internal state. If you need to, store the `contactId` and `connectionId` of the connection and make sure that the contact and connection still exist by fetching them in order from the `Agent` API object before calling methods on them.

### `taskConnection.getMediaInfo()`
```js
var mediaInfo = conn.getMediaInfo();
```
Get the media info object associated with this connection.

### `taskConnection.getMediaType()`
```js
if (conn.getMediaType() === "task") { /* ... */ }
```
Returns the `MediaType` enum value: `"task"`.

### `taskConnection.getMediaController()`
```js
conn.getMediaController().then(function (taskController) { /* ... */ });
```
Gets a `Promise` with the media controller associated with this connection.
The promise resolves to a `TaskSession` object from the `amazon-connect-taskjs` library.
See the [amazon-connect-taskjs documentation](https://github.com/amazon-connect/amazon-connect-taskjs) for more information.

## Utility Functions
### `Endpoint.byPhoneNumber()` (static function)
```js
var endpoint = Endpoint.byPhoneNumber("+18005550100");
```
Creates an `Endpoint` object for the given phone number, useful for `agent.connect()` and `contact.addConnection()` calls.

### `connect.hitch()`
```js
agent.onRefresh(connect.hitch(eventHandler, eventHandler._onAgentRefresh));
```
A useful utility function for creating callback closures that bind a function to an object instance.
In the above example, the "_onAgentRefresh" function of the "eventHandler" will be called when the agent is refreshed.

## Enumerations
These enumerations are not strictly required but are very useful and helpful for the Streams API. They are used extensively by the built-in CCP. All enumerations are accessible via `connect.<ENUM_NAME>`.

### `AgentStateType`
This enumeration lists the different types of agent states.

* `AgentStateType.INIT`: The agent state hasn't been initialized yet.
* `AgentStateType.ROUTABLE`: The agent is in a state where they can be routed contacts.
* `AgentStateType.NOT_ROUTABLE`: The agent is in a state where they cannot be routed contacts.
* `AgentStateType.OFFLINE`: The agent is offline.

### `ChannelType`
This enumeration lists the different types of contact channels.

* `ChannelType.VOICE`: A voice contact.
* `ChannelType.CHAT`: A chat contact.
* `ChannelType.TASK`: A task contact.

### `EndpointType`
This enumeration lists the different types of endpoints.

* `EndpointType.PHONE_NUMBER`: An endpoint pointing to a phone number.
* `EndpointType.AGENT`: An endpoint pointing to an agent in the same instance.
* `EndpointType.QUEUE`: An endpoint pointing to a queue call flow in the same instance.

### `ConnectionType`
Lists the different types of connections.

* `ConnectionType.AGENT`: The agent connection.
* `ConnectionType.INBOUND`: An inbound connection, usually representing an inbound call.
* `ConnectionType.OUTBOUND`: An outbound connection, representing either an outbound call or additional connection added to the contact.
* `ConnectionType.MONITORING`: A special connection type representing a manager listen-in session.

###  `MonitoringMode`
Lists the different monitoring modes representing a manager listen-in session.

* `MonitoringMode.SILENT_MONITOR`: An enhanced listen-in manager session
* `MonitoringMode.BARGE`: A special manager session mode with full control over contact actions

### `MonitoringErrorTypes`
Lists the different monitoring error states.
* `MonitoringErrorTypes.INVALID_TARGET_STATE`: Indicates that invalid target state has been passed

### `ContactStateType`
An enumeration listing the different high-level states that a contact can have.

* `ContactStateType.INIT`: Indicates the contact is being initialized.
* `ContactStateType.INCOMING`: Indicates that the contact is incoming and is waiting for acceptance. This state is skipped for `ContactType.VOICE` contacts but is essential for `ContactType.QUEUE_CALLBACK` contacts.
* `ContactStateType.PENDING`: Indicates the contact is pending.
* `ContactStateType.CONNECTING`: Indicates that the contact is currently connecting. For `ContactType.VOICE` contacts, this is when the user will accept the incoming call. For all other types, the contact will be accepted during the `ContactStateType.INCOMING` state.
* `ContactStateType.CONNECTED`: Indicates the contact is connected.
* `ContactStateType.MISSED`: Indicates the contact timed out before the agent could accept it.
* `ContactStateType.ERROR`: Indicates the contact is in an error state.
* `ContactStateType.ENDED`: Indicates the contact has ended.

### `ConnectionStateType`
An enumeration listing the different states that a connection can have.

* `ConnectionStateType.INIT`: The connection has not yet been initialized.
* `ConnectionStateType.CONNECTING`: The connection is being initialized.
* `ConnectionStateType.CONNECTED`: The connection is connected to the contact.
* `ConnectionStateType.HOLD`: The connection is connected but on hold.
* `ConnectionStateType.DISCONNECTED`: The connection is no longer connected to the contact.
* `ConnectionStateType.SILENT_MONITOR`: An enhanced listen-in manager session, this state is used instead of `ContactStateType.CONNECTED` for manager
* `ContactStateType.BARGE`: A special manager session mode with full control over contact actions, this state is used instead of `ContactStateType.CONNECTED` for manager

### `ContactType`
This enumeration lists all of the contact types supported by Connect Streams.

* `ContactType.VOICE`: Normal incoming and outgoing voice calls.
* `ContactType.QUEUE_CALLBACK`: Special outbound voice calls which are routed to agents before being placed. For more information about how to setup and use queued callbacks, see the Amazon Connect user documentation.
* `ContactType.CHAT`: Chat contact.
* `ContactType.TASK`: Task contact.

### `EventType`
This is a list of some of the special event types which are published into the low-level
`EventBus`.

* `EventType.ACKNOWLEDGE`: Event received when the backend API shared worker acknowledges the current tab. More specifically, it is sent to Streams in the following scenarios:
  - a consumer port connects to the shared worker
  - when Streams sends the `EventType.SYNCHRONIZE` to the shared worker, the shared worker sends the `ACKNOWLEDGE` event back. This happens every few seconds so that Streams and the shared worker are synchronized.
* `EventType.ACK_TIMEOUT`: Event which is published if the backend API shared worker fails to respond to an `EventType.SYNCHRONIZE` event in a timely manner, meaning that the tab or window has been disconnected from the shared worker.
* `EventType.IFRAME_RETRIES_EXHAUSTED`: Event which is published once the hard limit of 6 CCP retries are all exhausted. These retries are tiggered by the `ACK_TIMEOUT` event above.
* `EventType.AUTH_FAIL`: Event published indicating that the most recent API call returned a status header indicating that the current user authentication is no longer valid. This usually requires the user to log in again for the CCP to continue to function. See `connect.core.initCCP()` under **Initialization** for more information about automatic login popups which can be used to give the user the chance to log in again when this happens.
* `EventType.LOG`: An event published whenever the CCP or the API shared worker creates a log entry.
* `EventType.TERMINATED`: When the `EventType.TERMINATE` (not `TERMINATED`) event is sent to Streams, it is forwarded to the shared worker, which on successful termination then sends `EventType.TERMINATED` event back to Streams.  The `EventType.TERMINATE` event is sent to Streams in the following scenarios:
  - The CCP is initialized as an agent app (via `connect.agentApp.initApp`), and `connect.agentApp.stopApp` is invoked
  - The user manually logs out of the CCP

### `AgentEvents`
Event types that affect the agent's state.

* `AgentEvents.UPDATE`: this event is sent to Streams in the following scenarios:
  - a consumer port connects to the shared worker
  - when the shared worker or ccp is first initialized, then again repeated every few seconds
  - when the `EventType.RELOAD_AGENT_CONFIGURATION` event is sent to the shared worker. This happens in the `Agent.prototype.setConfiguration` method (which is invoked when the ccp settings are saved in the UI)

#### Note
The `EventBus` is used by the high-level subscription APIs to manage subscriptions and is available to users by calling `connect.core.getEventBus()`. Like the other event subscription APIs, calling `eventBus.subscribe(eventName, callback)` will return a subscription object which can be unsubscribed via `sub.unsubscribe()`.

## Streams Logger
The Streams library comes with a logging utility that can be used to easily gather logs and provide them for diagnostic purposes. You can even add your own logs to this logger if you prefer. Logs are written to the console log per normal and also kept in memory.

### `connect.getLog()`
```js
connect.getLog().warn("The %s broke!", "widget")
   .withException(e)
   .withObject({a: 1, b: 2});
```
Use `connect.getLog()` to get the global logger instance. You can then call one of the `debug`, `info`, `warn` or `error` methods to create a new log entry. The logger accepts printf-style parameter interpolation for strings and number forms.

Each of these functions returns a `LogEntry` object, onto which additional information can be added. You can call `.withException(e)` and pass an exception (`e`) to add stack trace and additional info to the logs, and you can call `.withObject(o)` to add an arbitrary object (`o`) to the logs.

A new method `sendInternalLogToServer()` that can be chained to the other methods of the logger has been implemented and is intended for internal use only. It is NOT recommended for use by customers.

Finally, you can trigger the logs to be downloaded to the agent's machine in JSON form by calling `connect.getLog().download()`.
Please note that `connect.getLog().download()` will output Connect-internal logs, along with any custom logs logged using the `connect.getLog()` logger. However, if an agent clicks on an embedded CCP's "Download Logs" button, only the Connect-internal logs will appear. The custom logs will not appear.

### LogLevel

For debugging, you can change the log level.

Valid log levels are `TEST`, `TRACE`, `DEBUG`, `INFO`, `LOG`, `WARN`, `ERROR`, and `CRITICAL`.

```js
const rootLogger = connect.getLog();

// Set log level.
rootLogger.setLogLevel(connect.LogLevel.TRACE);
// Set console output level.
rootLogger.setEchoLevel(connect.LogLevel.TRACE);
```

## CCP Error Logging
The following errors are related to connectivity in the CCP. These errors are logged in the CCP logs when they occur.

### `unsupported_browser`
Agent is using an unsupported browser. Only the latest 3 versions of Chrome or Firefox is supported. Upgrade the agent's browser to resolve this error. See [Supported browsers](https://docs.aws.amazon.com/connect/latest/adminguide/browsers.html) for more information.

### `microphone_not_shared`
The microphone does not have permission for the site on which the CCP is running. For Google Chrome steps, see [Use your camera and microphone in Chrome](https://support.google.com/chrome/answer/2693767?hl=en). For Mozilla Firefox steps, see [Firefox Page Info window](https://support.mozilla.org/en-US/kb/firefox-page-info-window).

### `signalling_handshake_failure`
Error connecting the CCP to the telephony system. To resolve, try the action again, or wait a minute and then retry. If the issue persists, contact support.

### `signalling_connection_failure`
Internal connection error occurred. To resolve, try the action again, or wait a minute and then retry. If the issue persists, contact support.

### `ice_collection_timeout`
Error connecting to the media service. To resolve, try the action again, or wait a minute and then retry. If the issue persists, contact support.

### `user_busy_error`
Agent has the CCP running in 2 distinct browsers at the same time, such as Chrome and Firefox. Use only one browser at a time to log in to the CCP.

### `webrtc_error`
An issue occurred due to either using an unsupported browser, or a required port/protocol is not open, such as not allowing UDP on port 443. To resolve, confirm that the agent is using a supported browser, and that all traffic is allowed for all required ports and protocols. See [CCP Networking](https://docs.aws.amazon.com/connect/latest/adminguide/troubleshooting.html#ccp-networking) and [Phone Settings](https://docs.aws.amazon.com/connect/latest/userguide/agentconsole-guide.html#phone-settings) for more information.

### `realtime_communication_error`
An internal communication error occurred.

## Agent States Logging
The following agent states are logged in the CCP logs when they occur.

### `AgentHungUp`
Agent hung up during the active call.

### `BadAddressAgent`
Error routiung the call to an agent. Attempt the action again. If the problem persists, contact support.

### `BadAddressCustomer`
The phone number could not be dialed due to an invalid number or an issue with the telephony provider. Confirm that the phone number is a valid phone number. Attempt to dial the number from another device to confirm whether the call is successful.

### `FailedConnectAgent`
Failed to connect the call to the agent. Attempt the action again. If the issue persists, contact support.

### `FailedConnectCustomer`
Failed to connect the call to the customer. Attempt the action again. If the issue persists, contact support.

### `LineEngagedAgent`
Agent line was not available when the call was attempted.

### `LineEngagedCustomer`
Customer line was not available when the call was attempted.

### `MissedCallAgent`
Agent did not pick up the call, either due to a technical issue or becasue the agent did accept the call.

### `MissedCallCustomer`
The customer did not answer a callback. Queued callbacks are retried the number of times specified in the Transfer to queue block. Increase the number of retries.

### `MultipleCcpWindows`
The agent has the CCP open in multiple distinct browsers, such as Firefox and Chrome, at the same time. To resolve, use only one browser to log in to the CCP.

### `RealtimeCommunicationError`
An internal communication error occurred.

### `ERROR Default`
All errors not otherwise defined.

## Logging out
In the default CCP UI, agent can log out by clicking on the "Logout" link on the Settings page. If you want to do something after an agent gets logged out, you can subscribe to the `EventType.TERMINATED` event.
```js
const eventBus = connect.core.getEventBus();
eventBus.subscribe(connect.EventType.TERMINATED, () => {
  console.log('Logged out');
  // Do stuff...
});
```

If you are using a custom UI, you can log out the agent by visiting the logout endpoint (`/connect/logout`). In this case, `EventType.TERMINATED` event won't be triggered. If you want the code above to work, you can manually trigger the `EventType.TERMINATE` event after logging out. When the event is triggered, `connect.core.terminate()` is internally called to clean up the Streams and the `EventType.TERMINATED` event will be triggered.
```js
fetch("https://<your-instance-domain>/connect/logout", { credentials: 'include', mode: 'no-cors'})
  .then(() => {
    const eventBus = connect.core.getEventBus();
    eventBus.trigger(connect.EventType.TERMINATE);
  });
```
In addition, it is recommended to remove the auth token cookies (`lily-auth-*`) after logging out, otherwise you’ll see AuthFail errors. ([Browser API Reference](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/cookies/remove)).

## Initialization for CCP, Customer Profiles, Amazon Q Connect, and CustomViews

*Note that if you are only using CCP, please follow [these directions](#initialization)*

Initializing the Streams API is the first step to verify that you have everything set up correctly and that you are able to listen for events. 
To get latest streams file and allowlist required urls follow [these instructions](#getting-started)*

### `connect.agentApp.initApp(name, containerId, appUrl, config)`

```js
<!DOCTYPE html>
<meta charset="UTF-8">
<html>
  <head>
    <script type="text/javascript" src="connect-streams-min.js"></script>
  </head>
  <!-- Add the call to init() as an onload so it will only run once the page is loaded -->
  <body onload="init()">
    <main style="display: flex;">
      <div id="ccp-container"></div>
      <div id="customerprofiles-container"></div>
      <div id="wisdom-container"></div>
      <div id="customviews-container"></div>
    </main>
    <script type="text/javascript">
      function init() {
        const connectUrl = "https://my-instance-domain.awsapps.com/connect";
        connect.agentApp.initApp(
            "ccp", 
            "ccp-container", 
            connectUrl + "/ccp-v2/",
            { style: "width:400px; height:600px;" }
        );
        connect.agentApp.initApp(
            "customerprofiles", 
            "customerprofiles-container", 
            connectUrl + "/customerprofiles-v2/",
            { style: "width:400px; height:600px;" }
        );
        //used to initialize Amazon Q Connect
        connect.agentApp.initApp(
            "wisdom", 
            "wisdom-container", 
            connectUrl + "/wisdom-v2/",
            { style: "width:400px; height:600px;" }
        );

        /**
         * CustomViews applications are traditionally attached to a contact for auto handling the lifecycle of a customviews on the attached contacts destroy event. 
         * Customviews has a few additional options that can be passed into the initApp config under the customViewsParams property to enable this behavior.
         * NOTE: Destroyed customviews applications will be removed from the AppRegistry to clear their namespace when using customViewsParams.
         * WARNING: Failing to handle the lifecycle of a customview by setting disableAutoDestroy to true and not calling terminateCustomView() on the application will cause the
         * customviews to count against your chat concurrency until it is terminated by the chat reaching its configured duration (defaults to 24 hours) or the Show View block timeout.
         * 
         * *OPTIONAL*
         * contact(contact | string): Attaches the contact to the customviews application, can be a contact object or contactId. 
         * If you use a contactId then disableAutoDestroy is true by default and you must use 
         * connect.core.terminateCustomView() to end the lifecycle of the customviews before closing the iframe. 
         * 
         * iframeSuffix (string): Attaches a suffix to the customviews application iframe id. This id will be formed as customviews{iframeSuffix}. 
         * Useful for instantiating multiple customviews applications in a single page and associating customviews applications with contactIds.
         * 
         * contactFlowId (string): The contactFlowId for the customviews that the application will display.
         * 
         * disableAutoDestroy (boolean): Disables the default handling of the views lifecycle behavior to not
         * terminate when the connected contact is closed. 
         * WARNING: NOT PROPERLY TERMINATING THE CUSTOMVIEW WITH CONNECT.CORE.TERMINATECUSTOMVIEW() BEFORE DESTROYING YOUR IFRAME CONTEXT WILL CAUSE THE CUSTOMVIEW TO COUNT AGAINST YOUR CHAT CONCURRENCY UNTIL IT IS TERMINATED BY THE DEFAULT CHAT OR CUSTOMVIEW FLOW TIMEOUT.
         * 
         * terminateCustomViewOptions (TerminateCustomViewOptions): Options around controlling the iframe's resolution behavior when disableAutoDestroy is true. 
         *  - resolveIframe (boolean): Whether to deconstruct the application iframe and clear its id in the AppRegistry, freeing the namespace of the applications id. Default true.
         * -  timeout (number): Timeout in ms. The amount of time to wait for the DOM to resolve and clear the iframe if resolveIframe is true. Default 5000.
         *  - hideIframe (boolean): Hides the iframe while it wait to be cleared from the DOM. Default true.
         * 
         **/
        connect.contact((contact)=>{
            contact.onConnected((contact)=>{
                connect.agentApp.initApp(
                  "customviews",
                  "customviews-container",
                  connectUrl + "/stargate/app",
                  { 
                    style: "width:400px; height:600px;",
                    customViewsParams: {
                      contact: contact,
                      contactFlowId: "55e028e0-62e5-44e7-aa81-eac163ed3fe1",
                      iframeSuffix: `${contact.getContactId()}-55e028e0-62e5-44e7-aa81-eac163ed3fe1`;
                      disableAutoDestroy: false,
                      {
                        hideIframe: true,
                        timeout: 5000,
                        resolveIframe: true,
                      }
                    },
                  }
              );
            });

            contact.onACW((contact)=>{
                connect.agentApp.initApp(
                  "customviews",
                  "customviews-container",
                  connectUrl + "/stargate/app",
                  { 
                    style: "width:400px; height:600px;",
                    customViewsParams: {
                      contact: contact,
                      contactFlowId: "70f643e7-7565-431b-9eb7-f5d666fc7a34",
                      iframeSuffix:`${contact.getContactId()}-70f643e7-7565-431b-9eb7-f5d666fc7a34`;
                      disableAutoDestroy: true
                    },
                  }
              );
            });

            // Because the onACW initApp for customviews has disableAutoDestroy set to true, we must manually terminate the customview 
            contact.onDestroy((contact) => {
              connect.core.terminateCustomView(
                connectUrl, 
                `${contact.getContactId()}-70f643e7-7565-431b-9eb7-f5d666fc7a34`,
                 {
                  resolveIframe: true,
                  timeout: 5000, 
                  hideIframe: true
                 }
              );
            })
        })
      }
    </script>
  </body>
</html>
```

Integrates with Amazon Connect by loading the pre-built app located at `appUrl` into an iframe and appending it into the DOM element with id of `containerId`. Underneath the hood, `initApp` creates a `WindowIOStream` for the iframes to communicate with the main CCP iframe, which is in charge of authenticating the agent's session, managing the agent state, and contact state.

- `name`: A string which should be one of `ccp`, `customerprofiles`, `wisdom`, or `customviews`.
- `containerId`: The string id of the DOM element that will contain the app iframe.
- `appUrl`: The string URL of the app. This is the page you would normally navigate to in order to use the app in a standalone page, it is different for each instance.
- `config`: This object is optional and allows you to specify some settings surrounding the CCP.
  - `ccpParams`: Optional params that mirror the configuration options for `initCCP`. Only valid when `name` is set to `ccp`. `allowFramedSoftphone` defaults to `true`.
  - `customViewsParams`: Optional params that are applicable when the `name` is set to `customviews`. Only valid when `name` is set to `customviews`.
  - `style`: An optional string to supply inline styling for the iframe.

## Voice ID APIs
Amazon Voice Connect ID provides real-time caller authentication and fraud risk detection which make voice interactions in contact centers more secure and efficient. For the Voice ID overview and administrator guide, please check the [AWS public doc](https://docs.aws.amazon.com/connect/latest/adminguide/voice-id.html). For more information about the agent experience in default CCP UI, please see [Use Voice ID page](https://docs.aws.amazon.com/connect/latest/adminguide/use-voiceid.html).

Streams Voice ID APIs can be tested after all these prerequisites are met:

1. A Voice ID domain is associated to your Connect instance ([link](https://docs.aws.amazon.com/connect/latest/adminguide/enable-voiceid.html#enable-voiceid-step1))
2. The contact flow is configured for Voice ID ([link](https://docs.aws.amazon.com/connect/latest/adminguide/enable-voiceid.html#enable-voiceid-step2))
3. "Voice ID" permission is given to the agent in the Security Profile page under the Contact Control Panel (CCP) section

The Voice ID APIs are exposed as Voice Connection methods and only work with two-party calls, not with conference calls at this moment. You can get the voice connection object by calling `contact.getAgentConnection()` when there's a voice connection.

Notes:
- For outbound calls and queued callbacks, these Streams Voice ID APIs can be called after contact is connected. For inbound calls, they can be called when contact is connecting.


### `voiceConnection.getVoiceIdSpeakerStatus()`
Describes the enrollment status of a customer. If the customer exists in the Voice ID, it resolves with a response object that contains one of the valid statuses, ENROLLED, PENDING or OPTED_OUT. PENDING means the customer hasn't enrolled but his/her speakerId has been created in the backend after updateVoiceIdSpeakerId() API is called for the customer. If the customer does not exist in the Voice ID, it still resolves but with an error object because backend API call fails. The case needs to be taken care of in a way like the code sample below.

```js
voiceConnection.getVoiceIdSpeakerStatus()
  .then((data) => {
    if (data.type === connect.VoiceIdErrorTypes.SPEAKER_ID_NOT_ENROLLED) {
      // speaker is not enrolled
    } else {
      const { Status } = data.Speaker;
      switch(Status) {
        case connect.VoiceIdSpeakerStatus.ENROLLED:
          // speaker is enrolled
          break;
        case connect.VoiceIdSpeakerStatus.PENDING:
          // speaker is pending
          break;
        case connect.VoiceIdSpeakerStatus.OPTED_OUT:
          // speaker is opted out
          break;
      }
    }
  })
  .catch((err) => {
    console.error(err);
  });
```


### `voiceConnection.enrollSpeakerInVoiceId(callbackOnAudioCollectionComplete: function)`
Enrolls a customer in Voice ID. The enrollment process completes once the backend has collected enough speech data (30 seconds of net customer's audio). If after 10 minutes the process hasn't completed, the method will throw a timeout error. If you call this API for a customer who is already enrolled, it will re-enroll the customer by collecting new speech data and registering a new digital voiceprint. Enrollment can happen only once in a voice contact.   
You can pass in a callback (optional) that will be invoked when our backend has collected sufficient audio for our backend service to create the new voiceprint.

```js
const callbackOnAudioCollectionComplete = (data) => {
  console.log(
    `Now sufficient audio has been collected and
    the customer no longer needs to keep talking.
    The backend service is creating the voiceprint with the audio collected`
  ); 
};

voiceConnection.enrollSpeakerInVoiceId(callbackOnAudioCollectionComplete)
  .then((data) => {
    console.log(
      `The enrollment process is complete and the customer has been enrolled into Voice ID`
    );
  })
  .catch((err) => {
    console.error(err);
  });
```

### `voiceConnection.evaluateSpeakerWithVoiceId(startNew: boolean)`
Checks the customer's Voice ID verification status. The evaluation process completes once the backend has collected enough speech data (10 seconds of net customer's audio). If after 2 minutes the process hasn't completed, the method will throw a timeout error. If you pass in false, it uses the existing audio stream, which is typically started in the contact flow, and immediately returns the result if enough audio has already been collected. If you pass in true, it starts a new audio stream and returns the result when enough audio has been collected. The default value is false.

The response will contain two results, AuthenticationResult and FraudDetectionResult. If one of them is disabled in the Set Voice ID contact flow block, the result will be null for that particular field. The authentication decision can be found at AuthenticationResult.Decision and it can be either AUTHENTICATED, NOT_AUTHENTICATED, OPTED_OUT, or NOT_ENROLLED. The fraud detection decision can be found at FraudDetection.Decision and it can be either HIGH_RISK or LOW_RISK.

```js
voiceConnection.evaluateSpeakerWithVoiceId()
  .then((data) => {
    // authentication result
    const authDecision = data.AuthenticationResult.Decision;
    switch(authDecision) {
      case connect.ContactFlowAuthenticationDecision.AUTHENTICATED:
        // authenticated
        break;
      case connect.ContactFlowAuthenticationDecision.NOT_AUTHENTICATED:
        // not authenticated
        break;
      case connect.ContactFlowAuthenticationDecision.OPTED_OUT:
        // opted out
        break;
      case connect.ContactFlowAuthenticationDecision.NOT_ENROLLED:
        // not enrolled
        break;
    }

    // fraud detection result
    const fraudDetectionDecision = data.FraudDetectionResult.Decision;
    switch(fraudDetectionDecision) {
      case connect.ContactFlowFraudDetectionDecision.HIGH_RISK:
        // authenticated
        break;
      case connect.ContactFlowFraudDetectionDecision.LOW_RISK:
        // not authenticated
        break;
    }
  })
  .catch((err) => {
    console.error(err);
  });
```

### `voiceConnection.optOutVoiceIdSpeaker()`
Opts-out a customer from Voice ID. This API can work for the customer who hasn’t enrolled in Voice ID.

```js
voiceConnection.optOutVoiceIdSpeaker()
  .then((data) => {
    // it returns speaker data but no additional actions needed
  })
  .catch((err) => {
    console.error(err);
  });
```


### `voiceConnection.getVoiceIdSpeakerId()`
Gets the speaker ID of the customer, which is set by the Set Contact Attributes block in the contact flow.

```js
voiceConnection.getVoiceIdSpeakerId()
  .then((data) => {
    console.log(data.speakerId);
  })
  .catch((err) => {
    console.error(err);
  });
```


### `voiceConnection.deleteVoiceIdSpeaker()`
Deletes the speaker ID of the customer from Voice ID. This API work only if the customer exists in Voice ID.

```js
voiceConnection.deleteVoiceIdSpeaker()
  .then(() => {
  })
  .catch((err) => {
    console.error(err);
  });
```


### `voiceConnection.updateVoiceIdSpeakerId(speakerId: string)`
Updates the speaker ID of the customer with the provided string.

```js
voiceConnection.updateVoiceIdSpeakerId('my_new_speaker_id')
  .then(() => {
  })
  .catch((err) => {
    console.error(err);
  });
```

## Enhanced Monitoring APIs
Enhanced monitoring providing real-time silent monitoring and barge capability to help managers and supervisors to listen in the agents' conversations and barge into the call if needed to take over the control and provide better customer experience. Supervisors in barge mode will be able to force mute agents and prevent them from unmuting themselves, will be able to hold, drop any connection, or directly speak with the customer. If the supervisor has muted an agent and then drops from the call, the agent will be able to unmute themselves once supervisor has dropped. Monitoring APIs are expected to be used against agent's(or supervisor's) connection. To start enhanced monitoring supervisor/manager will need to click an eye icon on the Real Time Metrics page. Before enabling Enhanced monitoring capabilities, ensure that you are using the latest version of the Contact Control Panel (CCP) or Agent workspace.

Streams Enhanced Monitoring APIs can be tested after all these prerequisites are met:
1. Enable Multi-Party Calls and Enhanced Monitoring in Telephony section of the Amazon Connect Console.
1. Enable Real-time contact monitoring and Real-time contact barge-in in Security Profiles

### `voiceConnection.isSilentMonitor()`
```js
if (conn.isSilentMonitor()) { /* ... */ }
```
Returns true if monitorStatus is `MonitoringMode.SILENT_MONITOR`. This means the supervisor connection is in silent monitoring state. Regular agent will not see supervisor's connection in the snapshot while it is in silent monitor state.

### `voiceConnection.isBarge()`
```js
if (conn.isBarge()) { /* ... */ }
```
Returns true if monitorStatus is `MonitoringMode.BARGE`. This means the connection is in barge-in state. Regular agent will see the supervisor's connection in the list of connections in the snapshot.

### `voiceConnection.isSilentMonitorEnabled()`
```js
if (conn.isSilentMonitorEnabled()) { /* ... */ }
```
Returns true if agent's monitoringCapabilities contain `MonitoringMode.SILENT_MONITOR` type. 

### `voiceConnection.isBargeEnabled()`
```js
if (conn.isBargeEnabled()) { /* ... */ }
```
Returns true if agent's monitoringCapabilities contain `MonitoringMode.BARGE` state type.

### `voiceConnection.getMonitorCapabilities()`
```js
var allowedMonitorStates = conn.getMonitorCapabilities();
```
Returns the array of enabled monitor states of this connection. The array will consist of `MonitoringMode` enum values.

### `voiceConnection.getMonitorStatus()`
```js
var monitorState = conn.getMonitorStatus();
```
Returns the current monitoring state of this connection. The value can be on of `MonitoringMode` enum values if the agent is supervisor, or the monitorStatus will not be present for the agent.

### `voiceConnection.isForcedMute()`
```js
if (conn.isForcedMute()) { /* ... */ }
```
Determine whether the connection was forced muted by the manager.

### `contact.updateMonitorParticipantState()`
```js
contact.updateMonitorParticipantState(targetState, {
   success: function() { /* ... */ },
   failure: function(err) { /* ... */ }
});
```
Updates the monitor participant state to switch between different monitoring modes. The targetState value is a `MonitoringMode` enum member.

### `contact.isUnderSupervision()`
```js
if (contact.isUnderSupervision()) { /* ... */ }
```
Determines if the contact is under manager's supervision

#### Usage examples

Check that barge is enabled before switching to the barge mode - first we need to make sure that barge is enabled for the supervisor connection, and after that initiate monitor status change on the contact.
```js
if(voiceConnection.isBargeEnabled()) {
  contact.updateMonitorParticipantState(connect.MonitoringMode.BARGE, {
   success: function() { 
    console.log("Successfully changed the monitoring status to barge, now you can control the conversation")
   },
   failure: function(err) { 
    console.log("Somenting went wrong, here is the error ", err) 
   }
  }); 
}
```

Check that silent monitor is enabled before switching to the silent monitor mode - first we need to make sure that silent monitor is enabled for the supervisor connection, and after that initiate monitor status change on the contact.
```js
if(voiceConnection.isSilentMonitorEnabled()) {
  contact.updateMonitorParticipantState(connect.MonitoringMode.SILENT_MONITOR, {
   success: function() { 
    console.log("Successfully changed the monitoring status to silent monitor")
   },
   failure: function(err) { 
    console.log("Somenting went wrong, here is the error ", err) 
   }
  }); 
}
```

After supervisor mutes the agent - force mute field is automatically updated on the agent side. You may want to display a banner or somehow indicate to the agent that he cannot unmute himself back anymore.

```js
if(voiceConnection.isForcedMute()) {
  /* Some logic here to indicate forced mute to the agent */
}
```

After supervisor barges the call - agent doesn't have control anymore. Agent can only mute or unmute himself until he was forced muted, or leave the call. It will be good to indicate that to ahent as well by hiding or disabling buttons.

```js
if(voiceConnection.isUnderSupervision()) {
  /* Some logic here to indicate disabled call controls to the agent */
}
```
## Quick Responses APIs - These APIs are only available **after accepting a chat contact.**

### `QuickResponses.isEnabled()`

Determines if quick responses feature is enabled for a given agent. Returns `true` if there is a knowledge base for quick responses configured for the instance. If the first call returns true, the knowledgeBase ID will be cached in local storage for subsequent `QuickResponse` API calls.

```js
QuickResponses.isEnabled().then(response => {
  ...
})
```

### `QuickResponses.searchQuickResponses(params: QuickResponsesQuery)`

Returns a list of Quick Responses based on the params given:

```js
  QuickResponsesQuery {
    query: string; // query string
    contactId?: string; // ID of a Contact object. Used to retrieve contact's attributes
    nextToken?: string; // The token for the next set of results. Use the value returned in the previous response in the next request to retrieve the next set of results.
    debounceMS?: number; // default value is 250ms. set it to 0 to disable debounced input change
    maxResults?: number; //number of results to be returned
  }
```

Example usage:

```js
const params = {
  query: "Hel",
  contactId: "...",
  debounceMS: 300,
  maxResults: 5
};

QuickResponsesService.searchQuickResponses(params: QuickResponsesQuery).then(response => {
  ...
});
```

Example response:

```js
{
  "nextToken": "token_string",
  "results": [
    {
      "attributesNotInterpolated": [],
      "channels": [ "Chat" ],
      "contentType": "application/x.quickresponse;format=markdown",
      "contents": {
          "markdown": {
              "content": "Hello"
          },
          "plainText": {
              "content": "Hello"
          }
      },
      "createdTime": 1697812550,
      "description": "Test",
      "groupingConfiguration": {
          "criteria": "RoutingProfileArn",
          "values": [ "..." ]
      },
      "isActive": true,
      "knowledgeBaseArn": "...",
      "knowledgeBaseId": "...",
      "language": "en_US",
      "lastModifiedBy": "...",
      "lastModifiedTime": 1697812550,
      "name": "Test",
      "quickResponseArn": "...",
      "quickResponseId": "...",
      "shortcutKey": "Test",
      "status": "CREATED"
    },
    //... more results
  ]
}
```

To retrieve next set of results:

```js
QuickResponsesQuery {
  //... params
  nextToken: "token_string"
}
```

Sample response:

```js
{
  "nextToken": null, //would not have next token once last set of responses is returned
  "results": [
    //... next set of results
  ]
}
```

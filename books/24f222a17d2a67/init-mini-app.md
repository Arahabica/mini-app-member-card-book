---
title: "LINEミニアプリの作成"
---

この章ではLINEミニアプリをHello World版を作成していきます。

## 4.1 LINEミニアプリの新規チャネルを作成する
LINE DeveloperページへアクセスしてLINEログインしてください。
    
https://developers.line.biz/ja/
    
プロバイダー設定してない方は「新規プロバイダー作成」をクリックします。
![LINE Provider](https://storage.googleapis.com/zenn-user-upload/azi8p99xha19emtr5zvxqqa3bhfo)

プロバイダー名は基本後から変更できないので本番運用する場合は注意してください。
認証画面などで表示される値になります。
入力できたら「作成」ボタンをクリックします。

![new LINE Channel](https://storage.googleapis.com/zenn-user-upload/iljryedsuymccbrgvi1i2pb8hsqu =360x)

「LINEミニアプリ」をクリックします。

![LINEミニアプリ](/images/a001_01.png =800x)

次に設定内容を入力していきます。下記の項目を入力していってください。

* チャネルアイコン(任意)
    * アプリのアイコン
    * こだわりのない人はとりあえず、[こちら](https://github.com/Arahabica/line-mini-app-hands-on/blob/main/resources/icon.png)の画像を設定してください。
* チャネル名
    * アプリの名前
    * 例) ALOHA MEMBERS CARD 
* チャネル説明
    * 説明
    * 例) ALOHAカフェのメンバーズカードアプリ
* アプリタイプ
    * ウェブアプリ
* メールアドレス
    * 担当者のメールアドレス

![Mini app Setting 002](/images/a002_01.png)
![Mini app Setting 003](/images/a003_01.png)
![Mini app Setting 004](/images/a004_01.png)


開発者契約、プラットフォーム規約を読んで、チェックして「作成」を押します。

すると、下記のような画面に遷移します。

![new LINE mini app Channel 01](/images/a006_01.png)

これで、新しいLINEチャネルが作成できました🎉

## 4.2 LINEミニアプリの設定・起動

次に、ミニアプリの設定をしていきます。

まず、後の開発で使用するチャネルIDを控えておきます。
`チャネル基本設定`のタブにチャネルIDが表示されますので、開発用のチャネルIDのをメモしておいてください。

![new LINE mini app Channel 01](/images/a006_02.png)


次に、`LIFF` というタブに移動してください。


![new LINE mini app Channel 02](/images/a007_01.png)

![new LINE mini app Channel 02](/images/a008_01.png)

ここで表示される`LIFF URL` がミニアプリのURLになります。
今回は開発用しか使いませんので、 開発用の`LIFF URL`をメモしてください。

次に開発用のエンドポイントURLの`編集`をクリックして、前章で作成したCloudFrontのURL( *https://***********.cloudfront.net* )を設定します。

これで、LINEミニアプリはひとまず完成です🎉

早速、開発用のLIFF URL(*https://* *liff.line.me/*******)にアクセスしてください。

すると、こちらのような画面になると思います。

![brower LINE mini app](/images/a009.png =360x)

LINEミニアプリは通常のブラウザでは動かないので、スマホのLINEアプリでこのQRコードを読み込んでLINEアプリ上でミニアプリを開くことになります。

LINEアプリでQRコードを読み込んで、次のような画面になれば、成功です🎉


![first LINE mini app](/images/a010.png =280x)

こちらがあなたが作成した初めてのLINEミニアプリになります。

しかし、これだとただWebサイトを表示しているだけなので面白くないですね。
次章で、これを会員証にしていきます。

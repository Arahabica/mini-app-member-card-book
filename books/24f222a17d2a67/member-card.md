---
title: "会員証の作成"
---

この章ではLINEミニアプリで会員証を実際に作成し、動かすところまで進めていきます。

## 5.1 AWS Systems Manager パラメーターストアにミニアプリの設定情報を保存する

AWS Systems Manager Parameter StoreとはAWSが提供する設定データを管理するためのサービスです。
このサービスを使うことによってプログラム中に設定情報を直書きせずにすみ、gitもクリーンな状態に保たれます。
会員証のプログラムを書いていく前に前章で取得した設定情報をこのAWS Systems Managerパラメータストアに保存していきます。

#### 5.1.1 AWSコンソールにログイン

https://console.aws.amazon.com/

#### 5.1.2 AWS Systems Managerの画面に移動
* 検索フォームに`Systems Manager`と入れると見つけることができます。

#### 5.1.3 AWS Systems Manager パラメーターストアの画面に移動
* [パラメータストア]をクリック
  ![AWS Systems Manager](/images/b001.png)

#### 5.1.4 新しいパラメータの登録
* [パラメータの作成]をクリック
  ![AWS Systems Manager Parameter Store](/images/b002.png)

#### 5.1.4 LINE Channel IDの登録
まず、LINE Channel IDを登録します。
項目を入力していきます。
* 名前
    * /dev/{{サービス名}}/LINE_CHANNEL_ID
    * {{サービス名}}にはserverless.ymlの最初の行に設定されている値を入れてください。
* タイプ
    * text
* 値
    * 4.2でメモしたLINE Channel ID

![Setting LINE Channel ID](/images/b003.png)

#### 5.1.5 LIFF IDの登録
LIFF IDを登録します。
項目を入力していきます。
* 名前
    * /dev/{{サービス名}}/LIFF_ID
    * {{サービス名}}にはserverless.ymlの最初の行に設定されている値を入れてください。
* タイプ
    * text
* 値
    * 4.2でメモした開発用LIFF URLのhttps://liff.line.me/移行の文字列がLIFF IDになります。
    * LIFF URLが`https://liff.line.me/1657239651-4MOr6aYa`の場合、`1657239651-4MOr6aYa`がLIFF IDになります。

![Setting LIFF_ID](/images/b004.png)

これで、AWS Systems Manager Parameter Storeの設定は完了です🎉


## 5.2 プログラムを修正
次にプログラムを修正していきます。

`frontend/pages/index.vue` を下記のコードにまるっと書き換えてください。

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue
<template>
  <section class="app-wrapper">
    <v-progress-circular
      v-if="!isLoggedIn"
      indeterminate
      color="primary"
      :size="120"
    ></v-progress-circular>
    <div v-if="isLoggedIn" class="member-card-app">
      <div class="header">
        <v-icon color="#ffffff" large>{{mdiFruitPineapple}}</v-icon>
        <h2>ALOHA MEMBERS CARD</h2>
      </div>
      <v-card class="member-card">
        <h4 v-if="profile" class="mt-3" >{{profile.displayName}}様</h4>
        <div class="qr-code-app">
          <div class="qr-code-wrapper">
            <vue-qrcode :value="userId" :options="qrOption" tag="img" class="qr-code"/>
            <div v-if="profile" class="app-icon-wrapper">
              <div class="white-circle" >
                <v-avatar v-if="profile.pictureUrl" :size="54" class="avatar">
                  <v-img :src="profile.pictureUrl" :alt="profile.displayName"/>
                </v-avatar>
              </div>
            </div>
          </div>
        </div>
      </v-card>
    </div>
  </section>
</template>
<script>
import liff from "@line/liff"
import axiosBase from "axios"
import VueQrcode from '@chenfengyuan/vue-qrcode'
import { mdiFruitPineapple } from '@mdi/js'

const LIFF_ID = process.env.LIFF_ID

const axios = axiosBase.create({
  baseURL: '/api',
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  },
  responseType: 'json'
})
const qrOption = {
  errorCorrectionLevel: "H",
  maskPattern: 0,
  margin: 2,
  scale: 2,
  width: 240,
  color: {
    dark: '#222222',
    light: "#ffffff"
  }
}
export default {
  components: {VueQrcode},
  data() {
    return {
      isLoggedIn: false,
      userId: null,
      profile: null,
      qrOption,
      mdiFruitPineapple
    }
  },
  async mounted() {
    // 1. LIFFの初期化
    await liff.init({liffId: LIFF_ID})
      .catch((err) => {
        console.error(err)
        window.alert('LIFFの初期化失敗。\n' + err)
      })
    // 2. LINEに未認証の場合、ログイン画面にリダイレクト
    if (!liff.isLoggedIn()) {
      await liff.login()
      return
    }
    const { userId } = liff.getContext()
    this.userId = userId
    this.isLoggedIn = true
    let hasProfilePermission = await this.verifyProfilePermission()
    if (!hasProfilePermission) {
      await this.requestAllPermission()
      hasProfilePermission = await this.verifyProfilePermission()
    }
    if (hasProfilePermission) {
      this.profile = await liff.getProfile()
    }
    const idToken = liff.getIDToken()
    await axios.put('/user', { userId, idToken })
    this.isLoggedIn = true
  },
  methods: {
    async verifyProfilePermission() {
      const permissionStatus = await liff.permission.query("profile")
      return permissionStatus.state === "granted"
    },
    async requestAllPermission() {
      try {
        await liff.permission.requestAll()
      } catch(e) {
        console.error(e)
      }
    }
  }
}
</script>
<style scoped lang="scss">
.app-wrapper {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: #3ee577;
}
.header {
  margin: 8px 0 0 0;
  h2 {
    margin: 8px 0 0 0;
    font-size: 18px;
    color: #ffffff;
  }
}
.member-card-app {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;

  .member-card {
    margin: 12px 20px 0 12px;
    padding: 12px 0 24px;
  }
}
.qr-code-app {
  display: flex;
  justify-content: center;
  .qr-code-wrapper {
    position: relative;
    .qr-code {
      max-width: 240px;
      width: 100%;
    }
    .app-icon-wrapper {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      .white-circle {
        width: 60px;
        height: 60px;
        border-radius: 30px;
        background-color: #ffffff;
        .avatar {
          margin: 3px;
        }
      }
      .app-icon {
        max-width: 54px
      }
    }
  }
}
</style>
```

書き換えたら、とりあえずデプロイしてしまってください

```sh:~/environment/line-mini-app-hands-on
$ yarn deploy
```

#### 5.2.1 LIFFライブラリからLINEのデータを取得

デプロイを待つ間、コードの説明をします。
まず、33行目でLIFFのライブラリを読み込んでいます。

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[33]
import liff from "@line/liff"
```

71行目からはLIFF IDを使ってLIFFを初期化し、ユーザIDを取得しています。
```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[71-83]
    // 1. LIFFの初期化
    await liff.init({liffId: LIFF_ID})
      .catch((err) => {
        console.error(err)
        window.alert('LIFFの初期化失敗。\n' + err)
      })
    // 2. LINEに未認証の場合、ログイン画面にリダイレクト
    if (!liff.isLoggedIn()) {
      await liff.login()
      return
    }
    const { userId } = liff.getContext()
    this.userId = userId
```


85行目からはプロフィールの取得権限があるかどうかを確認し、なければプロフィール権限を要求するダイアログを出します。
権限が得られれば、プロフィールを実際にプロフィールを取得しています。

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[84-92]
    let hasProfilePermission = await this.verifyProfilePermission()
    if (!hasProfilePermission) {
      await this.requestAllPermission()
      hasProfilePermission = await this.verifyProfilePermission()
    }
    if (hasProfilePermission) {
      this.profile = await liff.getProfile()
    }
```

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[98-108]
    async verifyProfilePermission() {
      const permissionStatus = await liff.permission.query("profile")
      return permissionStatus.state === "granted"
    },
    async requestAllPermission() {
      try {
        await liff.permission.requestAll()
      } catch(e) {
        console.error(e)
      }
    }
```

アプリを開いた後に権限を要求するダイアログを出せるのはLINEミニアプリ特有の機能です。それ以外のLIFFアプリでは実行できないので注意してください。

ここで用いている`liff`ライブラリのメソッドを簡単に解説します。

`liff.getContext()`
* LIFFアプリが起動された画面に関する情報を取得します。ユーザIDもその中に含まれます。

`liff.permission.query("profile")`
* プロフィールの取得権限を取得します。

`liff.permission.requestAll()`
* プロフィール権限を要求するダイアログを表示します。
* このメソッドはLINEミニアプリでのみ動作します。

`liff.getProfile()`
* プロフィールを取得します。プロフィールは下記のような形式で取得できます。
```json:profile.json
{
  "userId":"U4af4980629...",
  "displayName":"Brown",
  "pictureUrl":"https://profile.line-scdn.net/abcdefghijklmn",
  "statusMessage":"Hello, LINE!"
}
```

詳細は[リファレンス](https://developers.line.biz/ja/reference/liff/)を参照してください。  

#### 5.2.2 QRコードの表示

QRコードの表示には [@chenfengyuan/vue-qrcode](https://github.com/fengyuanchen/vue-qrcode) というライブラリを使っています。
35行目でこれを読み込んでいます。
```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[35]
import VueQrcode from '@chenfengyuan/vue-qrcode'
```

48行目からはvue-qrcodeの設定になります。
```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[48-58]
const qrOption = {
  errorCorrectionLevel: "H",
  maskPattern: 0,
  margin: 2,
  scale: 2,
  width: 240,
  color: {
    dark: '#222222',
    light: "#ffffff"
  }
}
```

そして、18行目でQRコードを表示しています。
ここでは、ユーザのLINE IDを直接表示しています。
```html:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[18]
<vue-qrcode :value="userId" :options="qrOption" tag="img" class="qr-code"/>
```

#### 5.2.3 ユーザのIDトークンをサーバに送信
下記のコードでLINEのIDトークンをサーバに送信しています。
これにより、サーバがユーザの情報を収集できます。

CloudFrontでアクセスを振り分けているので同じドメインの`/api/*`にリクエストを投げています。

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[34]
import axiosBase from "axios"
```

```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[40-47]
const axios = axiosBase.create({
  baseURL: '/api',
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest'
  },
  responseType: 'json'
})
```
```js:~/environment/line-mini-app-hands-on/frontend/pages/index.vue[93-94]
    const idToken = liff.getIDToken()
    await axios.put('/user', { userId, idToken })
```

ここで用いている`liff`ライブラリのメソッドは下記になります。

`liff.getIDToken()`
* ユーザのIDトークンを取得します。

詳細は[リファレンス](https://developers.line.biz/ja/reference/liff/)を参照してください。

## 5.3 会員証の表示
そうこうしている間にデプロイは終わったと思います。
もう一度、LINEミニアプリのURL(`https://liff.line.me/*******`)にアクセスして見てください。

まず、プロフィール権限の確認ダイアログが表示されます。
![MEMEBERS CARD permission](/images/a011.png =280x)

「許可する」ボタンを押すと下図のように、QRコードで会員証が表示されます。
![MEMEBERS CARD main](/images/a012.png =280x)

おめでとうございます！ ついに会員証が手に入りました🎉

## 5.4 会員証の読み取り
次に作成した会員証を読み取っていきます。

3.4でメモしたCloudFrontのURLに`/admin`をつけたURL( `https://***********.cloudfront.net/admin` )にアクセスしてください。

次にQRコードマークをクリックします

![admin](https://storage.googleapis.com/zenn-user-upload/gn2j7n37ay87429u4esrgqms5dlx)

QR読み取り画面になるので、ビデオマークをクリックしてPCのカメラを許可してください。

![QR](https://storage.googleapis.com/zenn-user-upload/lj31of6wyxthj83pcz81qmh6pjr6)

この状態で、LIFFアプリに表示されているQRコードをPCのカメラにかざすと来店が記録されます。
カメラから15cm~20cmほど離すとよく読み取れます。

![OK](https://storage.googleapis.com/zenn-user-upload/9uo8rhywxtw5c175fu7qztva4y0b)

もう一度、先ほどの来店一覧画面に戻ると来店が記録されているのが確認できます。
最高ですね。

![visit list](https://storage.googleapis.com/zenn-user-upload/5o9g7h7tn446t56a8zuobml710ac)

お疲れ様でした。これでアプリは完成です🎉

次の章では、サーバサイドのコードを解説いたします。
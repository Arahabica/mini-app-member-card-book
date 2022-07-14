---
title: "サーバ側のコードの解説"
---

## 6.1 サーバ側のコードの解説

サーバ側のコードのメイン部分は`line-mini-app-hands-on/backend/routes/router.js`に書いてあります。

## 6.2 API一覧

今回実装しているAPIは下記の3つです。

* `[PUT] /api/user`
    * ユーザを追加するAPI
* `[POST] /api/qrCode`
    * 読み取ったQRコードを処理するAPI
* `[GET] /api/visit`
    * 来店一覧

## 6.3 `[PUT] /api/user`
`[PUT] /api/user`はユーザを追加するAPIです。
ミニアプリを立ち上げたらすぐに呼ばれるものになります。
LINEのIDトークンを受け取り、LINE Login APIでIDトークンの有効性を検証し、ユーザ情報を取得してDynamoDBに保存しています。

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [87-105]
app.put('/user', async function(req, res) {
  try {
    const { userId, idToken } = req.body
    const now = new Date().getTime()
    // LINEのIDストークンが正しいか検証
    const profile = await verifyToken(userId, idToken)
    await putUser({
      userId: profile.userId,
      displayName: profile.displayName,
      pictureUrl: profile.pictureUrl,
      updatedAt: now,
    })
    res.json({success: 'succeed!', url: req.url, data: {}});
  } catch (err) {
    console.error(err)
    res.statusCode = 500;
    res.json({error: err, url: req.url, body: req.body});
  }
})
```

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [22-42]
// 渡されたLINE IDトークンが正しいものかを検証
const verifyToken = async (userId, idToken) => {
  const params = new URLSearchParams()
  params.append('id_token', idToken)
  params.append('user_id', userId)
  params.append('client_id', LINE_CHANNEL_ID)
  const response = await axiosInstance.post('/oauth2/v2.1/verify', params)
  if (response.status !== 200) {
    console.error(response.data.error_description)
    throw new Error(response.data.error)
  }
  //アクセストークンの有効期限
  if (response.data.exp * 1000 < new Date().getTime()) {
    throw new Error('access token is expired.')
  }
  return {
    userId: response.data.sub,
    displayName: response.data.name || '', // 権限によっては取得できない
    pictureUrl: response.data.picture || '' // 権限によっては取得できない
  }
}
```

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [53-64]
const putUser = async (data) => {
  const params = {
    TableName: TableName.User,
    Item: {
      id: data.userId,
      displayName: data.displayName,
      pictureUrl: data.pictureUrl,
      updatedAt: data.updatedAt
    }
  };
  return await dynamodb.put(params).promise();
}
```

## 6.4 `[POST] /api/qrCode`
`[POST] /api/qrCode`は読み取ったQRコードを処理するAPIです。
管理画面のスキャンページで動作します。
QRコードがそのままLINEのユーザIDになっているので、そこからユーザデータを取り出して来店処理をしています。

現状、認証機能が入っていないので本番利用する時は注意してください。

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [110-128]
app.post('/qrCode', async function(req, res) {
  try {
    const { qrCode } = req.body
    const userId = qrCode
    const visitedAt = new Date().getTime()
    const user = await getUser(userId)
    const data = await putVisit( {
      userId: user.id,
      visitedAt: visitedAt,
      displayName: user.displayName,
      pictureUrl: user.pictureUrl,
    });
    res.json({success: 'succeed!', url: req.url, data: data});
  } catch (err) {
    console.error(err)
    res.statusCode = 500;
    res.json({error: err, url: req.url, body: req.body});
  }
});
```

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [66-77]
const putVisit = async (data) => {
  let params = {
    TableName: TableName.Visit,
    Item: {
      userId: data.userId,
      visitedAt: data.visitedAt,
      displayName: data.displayName,
      pictureUrl: data.pictureUrl,
    }
  };
  return dynamodb.put(params).promise();
}
```

## 6.5 `[GET] /api/visit`
`[GET] /api/visit`は来店一覧を取得するAPIです。
管理画面のトップページで動作します。

こちらも認証機能が入っていないので本番利用する時は注意してください。

```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [130-139]
app.get('/visit', async function(req, res) {
  try {
    const data = await getVisitList();
    res.json({success: 'succeed!', url: req.url, data: data});
  } catch (err) {
    console.error(err)
    res.statusCode = 500;
    res.json({error: err, url: req.url, body: req.body});
  }
});
```


```js:~/environment/line-mini-app-hands-on/backend/routes/router.js [79-85]
const getVisitList = async () => {
  const params = {
    TableName: TableName.Visit,
  };
  const data = await dynamodb.scan(params).promise()
  return data.Items;
}
```
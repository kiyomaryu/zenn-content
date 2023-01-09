---
title: "OAuth2.0を理解するためにcurlコマンドにてリソースにアクセスしてみた"
emoji: "🔑"
type: "tech"
topics:
  - "oauth"
  - "認証"
published: true
published_at: "2023-01-02 15:38"
---

## はじめに

OAuth2.0についてちゃんと理解できてなかったので下記の書籍で学習していました。
https://amzn.asia/d/iWnvbeu

書籍が発売された当初とは異なりエンドポイント等が変わっているため第６章のチュートリアルを修正しながら対応しました。他の概要については書籍をご覧ください。

## 前提条件

- GCPにて「Photos Library API」のAPIが有効化されていること
- GCPにてOAuth同意画面が構成されていること
- GCPにてOAuthクライアントの登録が完了していること

## コマンド
### 1.認可コードの取得 (ブラウザでアクセス)

下記のURLにブラウザでアクセスします。
※秘密情報については*にて置換しています。
**リクエスト**
```
https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=**********************.apps.googleusercontent.com&state=xyz&scope=https://www.googleapis.com/auth/photoslibrary.readonly&redirect_uri=http://127.0.0.1/callback
```
response_type: 認可コードを要求するためのcodeを指定
client_id: gcpにてクライアント登録したときに取得したもの
state: ランダムな文字列。ここでは適当にxyz
scope: 有効化したPhotos Library APIの読み権限を表す文字列
redirect_url: http://127.0.0.1 ※これはクライアント登録時にコールバックURLとして登録してます

**レスポンス** 
ブラウザのアドレスバーから下記のURLを取得
```
http://127.0.0.1/callback?state=xyz&code=**************************&scope=https://www.googleapis.com/auth/photoslibrary.readonly
```
code: ここに認可コードが記載されている
state:認可リクエストで指定したxyz。これをCSRF防止のためにセッションと一緒に管理する

### 2.認可コードを使ってアクセストークンを取得する (ここからcurl)
**リクエスト**
```
curl -d "client_id=**********************.apps.googleusercontent.com" -d "client_secret=**********************" -d "redirect_uri=http://127.0.0.1/callback" -d "grant_type=authorization_code" -d "code=**************************" https://oauth2.googleapis.com/token
```
client_id:gcpにてクライアント登録したときに取得したもの
client_secret:gcpにてクライアント登録したときに取得したもの
redirect_uri: http://127.0.0.1/callback
grant_type: authorization_codeを指定
code: URLエンコードされた認可コードを記載　※ここが注意で認可コードは4/○○○みたいに/が入っているためURLエンコードを自分でやる必要がある

**レスポンス**
```
{
  "access_token": "**************************",
  "expires_in": 3599,
  "scope": "https://www.googleapis.com/auth/photoslibrary.readonly",
  "token_type": "Bearer"
}
```
access_token: アクセストークン
expires_in: アクセストークンの有効期限(秒)
scope: 有効化したPhotos Library APIの読み権限を表す文字列
token_type: トークンの種類。これはBearerトークン

### 3.アクセストークンを使ってリソースにアクセス
HTTPヘッダにAuthorizationを指定してBearer文字列の後に半角スペースを入れてアクセストークンを入れる
**リクエスト**
アルバム情報を取得するエンドポイントを叩きます。
```
curl -H 'Authorization: Bearer **************************' https://photoslibrary.googleapis.com/v1/albums
```

**レスポンス**
```
{
  "albums": [
    {
      "id": ""*****"",
      "title": "APITEST",
      "productUrl": "*****",
      "mediaItemsCount": "1",
      "coverPhotoBaseUrl": "*****",
      "coverPhotoMediaItemId": "*****"
    }
  ]
}
```
情報が取得できました！

## おわり

自分でOAuth2.0にて認可コードやアクセストークンを取得してリソースアクセスをすることにより理解が深まりました。今回学習で使った書籍は非常にわかりやすくOAuth2について記載されているためおすすめです。

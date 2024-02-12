---
title: 外部APIを利用するChrome拡張を作成する
tags:
  - Chrome
  - chrome-extension
private: false
updated_at: '2024-02-12T17:29:55+09:00'
id: 2c68214f81be7b6d5cc9
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

自作の Chrome 拡張外部 API をリクエストする処理を実装する場合の実装方法です。簡単にできると思いきや、結構めんどくさかったので備忘録となります。

# 実装

今回は動作確認用として、画面を開いたら Qiita の記事一覧取得 API をリクエストし、コンソールログに表示する拡張機能を作成する。

API を curl で実行する場合は以下の通り

```bash:
curl -X 'GET' https://qiita.com/api/v2/items
```

## 全体構成

最終的なフォルダ構成は以下。

```sh
src ---  manifest.json
    | - content.js
    | - libs/
         | - jquery.min.js
```

## jquery のダウンロード

外部 API 通信を行う際に Ajax を利用するので、jquery のソースコードをダウンロードしておく必要がある。jquery は以下の URL からダウンロードする。

https://releases.jquery.com/

拡張機能では CDN を利用することを推奨されていないので、ローカルにダウンロードしたものを読み込む。

ダウンロードしたら`libs/jquery.min.js`として保存する。

## `manifest.js`の作成

拡張機能の設定値となる `manifest.js`を作成する。

```json:manifest.js
{
  "manifest_version": 3,
  "name": "sample",
  "version": "1.0",
  "description": "sample for qiita",
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["libs/jquery.min.js", "content.js"]
    }
  ],
  "permissions": ["https://qiita.com/*"]
}

```

注意点は 2 つ

- `permissions` に接続先の API のドメイン名を指定する。今回の場合は`https://qiita.com/*`を指定している
- jquery を`content.js`より先に読み込む

## `content.js`の作成

画面をリロードするたびに Qiita の記事一覧を取得する処理を記載する。

```javascript:content.js
const settings = {
  async: true,
  crossDomain: true,
  headers: {
    accept: "application/json",
    "Content-type": "application/json",
  },
};

addEventListener("load", (event) => {
  $.ajax({
    url: "https://qiita.com/api/v2/items",
    method: "GET",
    ...settings,
  }).done(function (response) {
    console.log(response);
  });
});

```

## 動作確認

以下の記事を参考に`src`フォルダを拡張機能として登録する。

https://note.com/cute_echium873/n/n997dcf40b3a1

その後画面をリロードする。

![extension.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/57676c97-c367-ca47-d5bb-f04db6a7dc4d.png)

Qiita の記事一覧が取得でき、コンソールに表示されることまで確認できた。

# 終わりに

利用したソースコードは以下の Github で公開しています

https://github.com/sey323/chrome_extension_api_call_sample

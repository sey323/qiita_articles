---
title: Nuxt3とSupabaseでGoggleログインの機能を作りVercelにデプロイする
tags:
  - ""
private: true
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# 概要

Nuxt3 と Supabase を使って Google ログインの機能を作り、Vercel にデプロイする手順を記載する。

以下の手順を一部参考に実施します

https://supabase.nuxtjs.org/get-started

# 環境

- Nuxt: 3.13.2

## Nuxt3 のプロジェクトを作成

Nux3 のプロジェクトを作成する

```sh
npx nuxi@latest init supabase-google-login-app
```

作成後、プロジェクトに移動し、サーバーが起動することを確認する

```sh
cd supabase-google-login-app
npm run dev
```

以下の IP にアクセスし、Nuxt3 の画面が表示されれば成功。

- http://localhost:3000

## Supabase の設定

### Supabase のプロジェクト作成

Supabase のダッシュボードにアクセスする。画面上部の「New Project」よりプロジェクトを作成する

https://supabase.com/dashboard/projects

![2024-10-13 17.48.09.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/5209fecc-1d8d-9557-d46f-33821a88ff87.png)

必要項目を入力する。「Database Password」は入力後、再度確認できないので保存しておく。

![スクリーンショット 2024-10-13 17.53.22.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/5736deaa-a209-3d2d-ad97-e9134249a355.png)

「Create new project」をクリックすると Supabase のプロジェクトが作成される。

![スクリーンショット 2024-10-13 17.53.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/22659de6-254a-4bca-e68f-a2e7dfeb38b3.png)

ダッシュボードに表示されている「Project URL」「API Key」を`.env`ファイルに保存する。

```sh:.env
SUPABASE_URL=${コピーしたProject URL}
SUPABASE_KEY=${コピーしたAPI Key}
```

### `@nuxtjs/supabase`をプロジェクトに追加

Supabase の設定を行う。

```sh
npx nuxi@latest module add supabase
```

その後`nuxt.config.js`に以下の設定を追加する

```vue:nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/supabase'],
})
```

## Google ログインの設定

事前準備として、Google Cloud Platform にアクセスし、プロジェクトを作成する

### Google 認証の Auth 同意画面を作成

検索バーに「OAuth 同意画面」を入力し、OAuth 同意画面を作成する

![gcp-auth.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/66040949-53a4-9616-f8c0-28f3d0dac91e.png)

その後「アプリ名」「サポートメール」「デベロッパー連絡先情報」を入力し、保存する。それ以外の設定値はデフォルトのままで「保存して続行」をクリックする。

![gcp-auth-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/096c8775-66a8-1776-04ee-e3659c2bfdc9.png)

以下の画面が表示されれば成功。

![gcp-auth-complete.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/18b4f250-ebc8-81d3-bb96-51616b7c30df.png)

### Callback URL を Supabase から取得

Supabase のダッシュボードにアクセスし、「Authentication > Providers」の画面に移動する。

![supabase-callback-url.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/e54e4f90-0e18-6e52-22f9-ca61e4ddab2b.png)

### 認証情報を作成

「認証情報 > 認証情報を作成 > OAuth クライアント ID の作成」より新規に認証情報を作成する

![gcp-auth-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/d2cf8757-f23a-8549-de06-daa8dc5f6f55.png)

各種パラメータに以下の値を入力する。

- アプリケーションの種類: ウェブアプリケーション
- アプリケーション名: 任意の名前(今回は`supabase-google-login-app`)
- 承認済みの JavaScript 生成元: localhost:3000
- 許可されたリダイレクト URI: Supabase から取得した Callback URL

![gcp-authentication-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/0257a2b1-c4a8-770f-14b7-5deffe74b15b.png)

入力後、「作成」をクリックすると OAuth クライアント ID が生成されるため完了となる

![gcp-authentication-complete.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/0d4232f7-84cd-27a9-d3c8-dfc6a36be77d.png)

### Supabase で Provider を設定

Supabase の画面に戻り「Authentication > Providers」の画面に再度移動する。

Google の Provider を設定し、各種パラメータに以下の値を入力する。

- Client ID (for OAuth): 先ほど作成した OAuth クライアント ID
- Client Secret (for OAuth): 先ほど作成した OAuth クライアント ID のシークレット

![supabase-provider.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/99c83b66-c712-7f42-b250-6d503d55d578.png)

## nuxt3 のログイン機能を作成

ログインの画面とリダイレクトの画面を作成する

```vue:app.vue
<template>
  <div>
    <NuxtPage />
  </div>
</template>
```

```vue:pages/login.vue
<script setup lang="ts">
  const supabase = useSupabaseClient();

  const signInWithGoogle = async () => {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: "google",
      options: {
        redirectTo: "http://localhost:3000/confirm",
      },
    });
    if (error) console.log(error);
  };
</script>
<template>
  <div>
    <button @click="signInWithGoogle">Sign In with Google</button>
  </div>
</template>
```

```vue:pages/confirm.vue
<script setup lang="ts">
const user = useSupabaseUser()

watch(user, () => {
  if (user.value) {
      // Redirect to protected page
      return navigateTo('/')
  }
}, { immediate: true })
</script>

<template>
  <div>Waiting for login...</div>
</template>
```

ログイン後のトップページを作成する。ログインしたユーザのメールアドレスが表示されるようにし、ログインしていない状態でアクセスした場合は`/login`画面に強制的にリダイレクトするようにする。

```vue:pages/index.vue
<script setup lang="ts">
  const client = useSupabaseClient();
  const userEmail = ref("");

  onMounted(async () => {
    const user = await client.auth.getUser();

    // 認証されていない時、ログインページにリダイレクト
    if (!user) {
      client.auth.signOut();
      window.location.href = "/login";
    }

    // 認証されている場合はユーザー情報を取得
    userEmail.value = user.data.user?.email!; // ユーザーのメールアドレスを取得
  });
</script>
<template>
  <div>
    <p>Hello！ {{ userEmail }}</p>
  </div>
</template>
```

### 動作確認

`localhost:3000/login` にアクセスし、「Sign In with Google」をクリックする

![demo-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/a06616c6-1b07-a18b-84f7-3399d9795ba9.png)

Google のログイン完了後、`localhost:3000`にリダイレクトされ、「Hello!」の後にログインしたユーザのメールアドレスが表示されていれば成功

![demo-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/231e8e16-7a18-df24-ed63-de037660657a.png)

---
title: Nuxt3とSupabaseでGoggleログインの機能を作りVercelにデプロイする
tags:
  - Vercel
  - Supabase
  - Nuxt3
private: true
updated_at: '2024-10-13T23:35:35+09:00'
id: 6bfabc00ebe68dbedccf
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

ダッシュボードに表示されている「Project URL」「API Key」を`.env`ファイルに保存する。また、Google 認証後のリダイレクト先の URL を設定しておく。

```sh:.env
SUPABASE_URL=${コピーしたProject URL}
SUPABASE_ANON_KEY=${コピーしたAPI Key}

REDIRECT_HOST=http://localhost:3000
```

:::note info
公式では`SUPABASE_KEY`を利用しているが、`SUPABASE_KEY`は Vercel と Supabase の連携の際に、自動で連携がされない。

自動で連携される`SUPABASE_ANON_KEY`に変更している。
:::

### `@nuxtjs/supabase`をプロジェクトに追加

Supabase の設定を行う。

```sh
npx nuxi@latest module add supabase
```

その後`nuxt.config.js`に以下の設定を追加する

```vue:nuxt.config.ts
export default defineNuxtConfig({
  modules: ["@nuxtjs/supabase"],
  supabase: {
    key: process.env.SUPABASE_ANON_KEY, // Supabase の API Keyの設定
  },
  runtimeConfig: {
    public: {
      redirectHost: process.env.REDIRECT_HOST, // リダイレクト先の ホスト名の設定
    },
  },
});

```

## Google ログインの設定

事前準備として、Google Cloud Platform にアクセスし、プロジェクトを作成する

https://console.cloud.google.com/welcome

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
  const runtimeConfig = useRuntimeConfig();
  const signInWithGoogle = async () => {
    const { error } = await supabase.auth.signInWithOAuth({
      provider: "google",
      options: {
        redirectTo: `${runtimeConfig.public.redirectHost}/confirm`,
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
  const user = useSupabaseUser();

  watch(
    user,
    () => {
      if (user.value) {
        return navigateTo("/");
      }
    },
    { immediate: true }
  );
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

## Vercel にデプロイ

事前準備として、Github に Public リポジトリを作成し、プロジェクトをプッシュする

:::note warn
Vercel の無料プランではプライベートリポジトリのデプロイができないため、無料プランを利用している場合は Public リポジトリを作成する必要がある
:::

### Vercel にプロジェクトを作成

ダッシュボードの「New Project」よりプロジェクトを作成する

![vercel-create-project.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/fb18fa88-cb9f-3fcd-8d3e-a5a336ced401.png)

「Import Git Repository」で Github に Push したリポジトリを選択し、「Import」をクリックする

![vercel-create-project-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/cecbd0b0-4cfc-4aaf-dfd4-299e9b347dcc.png)

「Project Name」に任意の名前を入力し、「Deploy」をクリックする

![deploy-project.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/72a64c91-59ee-b8d9-9036-7c29e06c8c80.png)

「Deploy」が完了するが、この状態では Supabase の設定が反映されていないため、500 エラーとなる。

![スクリーンショット 2024-10-13 22.10.41.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/164441da-9606-a0c5-21a6-31381d514388.png)

### Vercel と Supabase の環境変数を設定

Project の「Settings > Integrations」より Supabase を選択し、「Configure」をクリックする。

![supabase-configure.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/0fa1d04a-702d-0a59-42b9-361ddd389fa7.png)

「Integrations」に Supabase が表示されていない時は「Browse Marketplace」より Supabase を検索し、追加する

![browse-marketplace.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/dd980702-a3c0-08cf-0e74-3ff174806e9f.png)

設定画面で、該当するプロジェクトがある組織かプロジェクトを選択する。

![project-org-select.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/29dc3d88-8d8c-3216-2802-621d751fbf70.png)

画面赤枠の「Add new project connection」をクリックする。

![relation-vercel-supabase.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/a04c92ad-24a5-cc35-051e-cf1da17de605.png)

その後、Supabase のプロジェクトと先ほど作成した Vercel のプロジェクトを選択し、「Connect project」をクリックする

![connect-vercel-supabase.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/48e748b1-cd18-12f8-a22e-befc1cdf09ca.png)

連携が成功したら、Vercel のプロジェクトに環境変数が自動で設定される。

追加でログイン後のリダイレクト先である`REDIRECT_HOST`を設定する。今回の場合であれば、vercel の URL(`https://nuxt-supbase-google-login-app.vercel.app`)を設定する。

![vercel-env.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/3e8cb8de-97b5-bbd5-018f-d6fd28ef4f6c.png)

### Supabase に Vercel の URL をリダイレクト先として許可

Supabase のダッシュボードにアクセスし、「Authentication > URL Configuration」の画面に移動する。「Redirects URL」よりデプロイした Vercel の URL を追加する。

今回であれば以下の 2 つの URL を追加する。

- `https://nuxt-supbase-google-login-app.vercel.app`
- `https://nuxt-supbase-google-login-app.vercel.app/**`

![supabase-redirect-url.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/163680/fe2c5b4c-98de-bd78-89e3-da688348f623.png)

この設定がないと、Google のログイン後にリダイレクトができず、`localhost:3000`にリダイレクトされてしまう。

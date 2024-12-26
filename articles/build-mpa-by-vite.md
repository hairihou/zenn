---
title: 'ViteをMPAのモジュールバンドラとして使う'
emoji: '🧗'
type: 'tech'
topics:
  - 'typescript'
  - 'vite'
  - 'vuejs'
published: true
---

<!-- build-mpa-by-vite -->

この記事では[create vite (Vue + TypeScript)](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-vue-ts)のボイラープレートを使います。Vueを選択していますが、末端技術は他を選択していても問題無いので、使うライブラリに合わせて適宜読み替えてください。

デフォルトではエントリーポイントは `index.html` だけのため、別Page(`sub-page.html`)を新規で作ります。

```diff
vite-project
  ├── src
  │  └── ...
  ├── index.html
+ ├── sub-page.html
  └── ...
```

```html: sub-page.html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + Vue + TS</title>
  </head>
  <body>
    <!-- index.htmlとはmount対象のidとモジュールファイルを別にする (※この後作ります) -->
    <div id="sub-page"></div>
    <script type="module" src="/src/sub-page/main.ts"></script>
  </body>
</html>
```

新規で作るエントリーポイントは `/src/sub-page/main.ts` です。既にプロジェクトにある `src/main.ts` `src/App.vue` をコピーする形で以下のファイルも作っておきます。

```vue: src/sub-page/SubPage.vue
<script setup lang="ts">
</script>

<template>
  <div>
    Sub Page
  </div>
</template>
```

```typescript: src/sub-page/main.ts
import { createApp } from 'vue';
import '../style.css';
import SubPage from './SubPage.vue';

// <div id="sub-page"></div>に対してmountする
createApp(SubPage).mount('#sub-page');
```

この状態で `vite build` を行っても追加した `sub-page.html` はエントリーポイントとして認識されず、`index.html` に必要なモジュールしかビルドされません。`vite.config.ts` の `build.rollupOptions` を変更して複数のHTMLファイルをエントリーポイントとして認識できるようにします。

```diff: vite.config.ts
import vue from '@vitejs/plugin-vue';
import { defineConfig } from 'vite';

// https://vitejs.dev/config/
export default defineConfig({
   plugins: [vue()],
+  build: {
+    rollupOptions: {
+      input: {
+        index: './index.html',
+        'sub-page': './sub-page.html',
+      },
+    },
+  },
});
```

これで `sub-page.html` と必要なJSモジュールが `dist/` にビルドされるようになりました。

#### MPAのテンプレートと統合する

DjangoやRuby on RailsのようなWebフレームワークでは、レスポンス時にサーバサイドの変数を挿入するために、プレーンなHTMLファイルではなく内包されたテンプレートエンジンを使うことが多いです。その中でViteでビルドしたJavaScriptモジュールを使えるようにするには、ビルド後の　`dist/*.html`　でやっているモジュールのimportを引っ張ってきます。

```html
<!-- 以下のような記述をビルドの度にコピーしていく必要がある -->
<script type="module" crossorigin src="path/to/dist/assets/index-B1EWmLEa.js"></script>
<link rel="modulepreload" crossorigin href="path/to/dist/assets/_plugin-vue_export-helper-C8EBO7q5.js" />
<link rel="stylesheet" crossorigin href="path/to/dist/assets/style-ziVyQSII.css" />
```

これでテンプレートエンジンのようなHTML生成部分にモジュールを紐づけることが出来ました。

#### 痒い所に手が届く系の設定

Build hashの変更に追従する手間の削減、エントリーポイント単位でBundleを纏める...のような事を実現するために、一部の設定を変更する方法もあります。
**Vite(Rollup)では、デフォルトのBuild optionでモジュール分割・Webパフォーマンスの最適化をしており、変更が品質とトレードオフになるものもあります。** 本記事のように、MPAの中でピンポイントで使う上でネックになる部分は運用の複雑さとのバランスを取るのが良いと考えます。

```diff: vite.config.ts
import vue from '@vitejs/plugin-vue';
import { defineConfig } from 'vite';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  build: {
+   cssCodeSplit: false, // CSSファイルの自動分割を無効にして単一にします
+   modulePreload: false, // 動的インポートが行われるモジュールの事前読み込み(`<link rel="modulepreload" ... />`)を無効にします
    rollupOptions: {
      input: {
        index: './index.html',
       'sub-page': './sub-page.html',
      },
+     // Browser・CDNでのキャッシュ不整合を防ぐためのビルドハッシュを、chunkの名前から削ります
+     output: {
+       entryFileNames: 'assets/[name].js',
+       chunkFileNames: 'assets/[name].js',
+       assetFileNames: 'assets/[name].[ext]',
+     },
    },
  },
});
```

---

Viteのようなモジュールバンドラは、SPAのフロントエンドを前提とする使い方がほとんどかと思いますが、複数のHTML(エントリーポイント)を持つMPAの中でPluggableにVueやReactで実装したコンポーネントを導入していくためのツールチェーンとしても有力ですね。

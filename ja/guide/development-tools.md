---
title: 開発ツール
description: Nuxt.js は開発がより楽しいものになるよう手助けします。
---

> アプリケーションをテストすることはウェブ開発の一部です。Nuxt.js は可能な限り簡単にテストできるようにしています。

## エンドツーエンドテスト

[AVA](https://github.com/avajs/ava) は [jsdom](https://github.com/tmpvar/jsdom) と合わせて使うことができる、JavaScript のパワフルなテスティングフレームワークです。エンドツーエンドテストを簡単に行うためにこれらを使うことができます。

まず AVA と jsdom を開発依存パッケージに追加する必要があります:

```bash
npm install --save-dev ava jsdom
```

それから `package.json` に test というスクリプトを追加し、テストにインポートするファイルをコンバイルするために AVA を設定します。

```javascript
"scripts": {
  "test": "ava",
},
"ava": {
  "require": [
    "babel-register"
  ]
},
"babel": {
  "presets": [
    "es2015"
  ]
}
```

`test` フォルダ内にテストを書いていくことにします:

```bash
mkdir test
```

`pages/index.vue` にページがあります:

```html
<template>
  <h1 class="red">Hello {{ name }}!</h1>
</template>

<script>
export default {
  data () {
    return { name: 'world' }
  }
}
</script>

<style>
.red {
  color: red;
}
</style>
```

`npm run dev` でアプリケーションを起動し [http://localhost:3000](http://localhost:3000) を開いているとき、`Hello world!` というタイトルが表示されています。

`test/index.test.js` というテストファイルを追加します:

```js
import test from 'ava'
import { Nuxt, Builder } from 'nuxt'
import { resolve } from 'path'

// Nuxt への参照を保持します
// そうすればテスト終了時にサーバーをクローズできます
let nuxt = null

// Nuxt.js を初期化し localhost:4000 のリスニングを開始します
test.before('Init Nuxt.js', async t => {
  const rootDir = resolve(__dirname, '..')
  let config = {}
  try { config = require(resolve(rootDir, 'nuxt.config.js')) } catch (e) {}
  config.rootDir = rootDir // project folder
  config.dev = false // production build
  nuxt = new Nuxt(config)
  await new Builder(nuxt).build()
  nuxt.listen(4000, 'localhost')
})

// 生成された HTML のみをテストする例
test('Route / exits and render HTML', async t => {
  let context = {}
  const { html } = await nuxt.renderRoute('/', context)
  t.true(html.includes('<h1 class="red">Hello world!</h1>'))
})

// DOM チェックを経由してテストする例
test('Route / exits and render HTML with CSS applied', async t => {
  const window = await nuxt.renderAndGetWindow('http://localhost:4000/')
  const element = window.document.querySelector('.red')
  t.not(element, null)
  t.is(element.textContent, 'Hello world!')
  t.is(element.className, 'red')
  t.is(window.getComputedStyle(element).color, 'red')
})

// Nuxt サーバーをクローズする
test.after('Closing server', t => {
  nuxt.close()
})
```

テストを実行できるようになっています:

```bash
npm test
```

jsdom はブラウザを使っていないため制約がいくつかありますが、ほとんどのテストはカバーできます。もしアプリケーションをテストするためにブラウザを使いたいときは [Nightwatch.js](http://nightwatchjs.org) を調べるとよいかもしれません。

## ESLint

> ESLint はコードを綺麗に保てる優れたツールです。

とても簡単に [ESLint](http://eslint.org) を Nuxt.js と一緒に使うことができます。まず npm の依存パッケージを追加する必要があります:

```bash
npm install --save-dev babel-eslint eslint eslint-config-standard eslint-plugin-html eslint-plugin-promise eslint-plugin-standard eslint-plugin-import eslint-plugin-node
```

それから `.eslintrc.js` ファイルをプロジェクトのルートディレクトに置いて ESLint を設定できます:

```js
module.exports = {
  root: true,
  parser: 'babel-eslint',
  env: {
    browser: true,
    node: true
  },
  extends: 'standard',
  // *.vue ファイルを lint するために必要
  plugins: [
    'html'
  ],
  // ここにカスタムルールを追加します
  rules: {},
  globals: {}
}
```

それから `lint` スクリプトを `package.json` 内に追加できます:

```js
"scripts": {
  "lint": "eslint --ext .js,.vue --ignore-path .gitignore ."
}
```

lint を実行できます:

```bash
npm run lint
```

ESLint は `.gitignore` に定義されたファイルを無視しつつ、それ以外のすべての JavaScript と Vue ファイルを lint します。


<p class="Alert Alert--info">`"precommit": "npm run lint"` を package.json に追加してコードをコミットする前に自動的に lint するのはベストプラクティスのひとつです。</p>

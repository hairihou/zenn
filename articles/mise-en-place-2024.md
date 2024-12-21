---
title: 'miseでパッケージマネージャの迷宮を渡り歩く'
emoji: '🏜️'
type: 'tech'
topics:
  - 'asdf'
  - 'mise'
  - 'nodejs'
  - 'rtx'
published: true
---

<!-- mise-en-place-2024 -->

今年asdfからmiseに乗り換えて、速度もそうですが使い勝手に良さを感じました。

https://mise.jdx.dev/

globalで管理したいpackage/runtimeの定義は以下のように管理します。

```toml: ~/.config/mise/config.toml
[tools]
bun = "latest"
deno = "latest"
node = "22"
python = "3.13"
```

ディレクトリ(Repository)の単位で閉じたい場合は以下のような `mise.toml` を中に作ります。

```toml: mise.toml
[tools]
node = "22"
"npm:pnpm" = "9.15"
```

`mise.toml` をcommitしてgitの管理対象にすることで、miseを使っている開発メンバーは `mise install` でruntime/package managerのバージョンをサクッと揃えることができます。いいね。

mise以外で管理されている場合(package.jsonに`packageManager` (corepack)や`volta`のようなフィールドがある場合)でも...

#### npm

```toml: mise.toml
[tools]
node = {NODE_VERSION}
```

#### pnpm

```toml: mise.toml
[tools]
node = {NODE_VERSION}
"npm:pnpm" = {PNPM_VERSION}
```

#### yarn v4 (corepack)

```toml: mise.toml
[tools]
node = {NODE_VERSION}
"npm:corepack" = "latest"
```

これでなんとかなったりする。

---

mise.tomlをcommitしないなら[Config Environments](https://mise.jdx.dev/configuration/environments.html)を参考に `mise.local.toml` `mise.{MISE_ENV}.local.toml` のようなファイルを作り、gitignoreにファイル名だけ書いておけば良い。

```diff: .gitignore
+ mise.local.toml
+ mise.*.local.toml
```

```diff: ~/.config/git/ignore
+ mise.local.toml
+ mise.*.local.toml
```

---

コマンド周りはasdfっぽく叩けて一部asdf-pluginにも対応しており、元々asdfユーザだった自分が求めていたものと感じる。
Makefileのようなものを[Tasks](https://mise.jdx.dev/tasks/)で代替する日も近そうな雰囲気。今後もバリバリ使っていきたいツールです。

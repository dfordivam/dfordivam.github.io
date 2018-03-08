---
title: 「tenjinreader.com」の開発体験レポート
---

このアプリケーションを自身の日本語の学習経験と必要条件ですから使っていました。

この記事はHaskell　Developersについてから、こちら日本語学習とアプリの設計ことについては何も話しません。

***

## はじめて

このアプリケーション全部Haskellで書いています.

しかも、いくつか新しい（実験的な）ライブラリーを使用とした。このブログは私の経験です。

一年前ウェブプログラミングを始めた。Haskellを４年上に使ってから、これはJavascriptより簡単と感じます

五ヶ月に仕上げました。

[Source code on Github](https://github.com/blueimpact/tenjinreader)

総コードベースは約

- Frontend - 3k
- Backend - 3k
- Common - 1k

## Reflex FRP

Frontend は全部「Reflex-DOM FRP」で使っていました。

#### いい

* 全部 Haskell

  Frontend と Backend　の両方を　Haskell　に持っていているのは素晴らしいです。３つのプロジェクトの間でコードを cut-paste から移動してきました。

* Reflex　に　Code refactoring　と　UI　変更、 Widget　の移動と複数の場所で再利用がは非常に簡単です。

* [`reflex-project-skeleton`](https://github.com/ElvishJerricco/reflex-project-skeleton)、 `nix`　と  `jsaddle-warp` は
  開発ワークフローにとって非常に役に立ちます。
  `nix-copy-closure`からサーバーの deployment は簡単です。

  nix との最初の闘争がありましたが、現在は数多くの優れたリソースがあります。#reflex-frp IRC channel は非常に役に立ちます。

* websocket のために、自分で作った小さな[ライブラリ](https://github.com/dfordivam/reflex-websocket-interface)を使用します。
  舞台裏でこのライブラリはメッセージのエンコーディング/デコードやイベントの配達するがやっています。

  新しいメーセージを参加するや変更するはとりわけ簡単です
  
  サーバーコミュニケーションのコードは　Widget 自体に含まれています。
  
  新しい機能の開始時に、Frontend でどのようなダータ形式は必要が知りませんから、Backend(DB)　から生データを転送します。

  Frontend で色々な場所で生データを操作しまして適切な形式を決定します。これからこのコードを　Backend に移動することができます。

#### 困難と挑戦

* 大きな `rec` block にコードの　compilation は難しい、エラーメッセージは誤解を招く可能性があります。

  `rec` block はまた、奇妙なループを導入したり、アプリケーションを停止させたりする可能性があります。これを解決策ためには、`delay`を加えることがあります

  Tip 1 - 変数名を再利用しないでください
  ```
  rec
    retVal <- do
      retVal <- someStuff
      ...
      return retVal
  ```
  
  Tip 2 - `rec` block が複雑になるときは、別の関数を作成してください。
  

* `Reflex.Collection`で有用な関数がありますが、それらのゆ動作とユースケースを理解するのは難しいでしょう。

* 大きな`Dynamic`は扱いにくい

  例えば：`[Dynamic t (Bool, Int)]` から　`Event t (Set Int)` を作るために 
  `[]` -> `Set` -> `tagDyn` より
  `[]` -> `tagDyn` -> `Set`　良い。

  `dyn` より `widgetHold` 良い。

* Reflex はまだ統合された情報源を必要としている。

* `ghcjs` のパフォーマンスは良くありません。`webghc`でより良いパフォーマンスが期待できます。

* External JS library を使って時

  * 時々FFI や DOM APIs を正しく機能ために`delay`が必要です。

  * External JS は例外した場合、アプリは停止します。

  * Bootstrap, Semantic UI ようなライブラリのと統合はまだ進行中です。

## 開発環境

  - `Spacemacs` with syntax highlighting, and `hindent`

  - Local `hoogle` server running from `nix-shell`

  - `./cabal new-repl` from the project skeleton, and sometimes `ghcid`.

  - Frontend running on Chrome via `jsaddle-warp`.
---
title: "Neovimを使い始めて半年経った若輩Vimmerが愛用しているプラグインの紹介"
emoji: "🐗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "vim"]
published: true
publication_name: "vim-jp"
---

:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2025年3月14日分の記事になります。
:::

## はじめに

去年の夏頃、巷でAIエディターが流行している最中、ずっと憧れていたNeovimを使い始め、晴れてVimmerになりました!
それから半年が経ちNeovimにもだいぶ慣れてきたので、自分のNeovim環境を再構築しました。その際にプラグインも色々見直したので、今回はその中でも特に愛用しているプラグインを紹介します！

https://github.com/okm321/dotfiles/tree/main/nvim/lua/plugins

:::message
Telescopeやnvim-lspなどの王道プラグインは割愛します！
:::

## UI系のプラグイン

### dashboard-nvim

https://github.com/nvimdev/dashboard-nvim

Neovimをファイル指定なしで起動した際に表示されるスタート画面をかっこよくできるプラグインです。直近開いたファイルやプロジェクトも表示されるので便利だし、なにより見た目がかっこよくてテンション爆上げ系のプラグインです！

![dashboardのサンプル](/images/neovim-plugin/dashboard.png)

ちなみに、ヘッダー部分のロゴは以下のサイトの`ANSI Shadow`フォントを使って生成しています！

https://patorjk.com/software/taag/#p=testall&f=Acrobatic&t=Naovim

### hlcunk.nvim

https://github.com/shellRaining/hlchunk.nvim

コードの各種ブロックの開始と終了部分をハイライトしてくれるプラグインです！複雑な入れ子構造のコードを読む際にわかりやすいのと、ブロックのペアが合ってない時にすぐに気付けるので重宝しています！

![hlchunkのサンプル](/images/neovim-plugin/hlchunk.gif)

### nvim-treesitter-context

https://github.com/nvim-treesitter/nvim-treesitter-context

コードを編集している際に現在の関数やクラスなどの親スコープを画面上部に表示してくれるプラグインです！長い関数やクラスがあるコードリーディングなどでとても便利です！

![nvim-treesitter-contextのサンプル](/images/neovim-plugin/nvim-treesitter-context.gif)

### dropbar.nvim

https://github.com/Bekaboo/dropbar.nvim

いわゆるパンクズリストを表示してくれるプラグインです！表示するにとどまらず、移動も可能！今いるバッファのシンボルも表示してくれる優れものです！

![dropbarのサンプル](/images/neovim-plugin/dropbar.gif)

### nvim-scrollbar

https://github.com/petertriho/nvim-scrollbar

Neovimの右側にスクロールバーを表示してくれるプラグインです！LSPからのエラーや警告、gitのdiff情報なども表示できるので便利！

## 痒いところに手が届く系のプラグイン

### accelerated-jk.nvim

https://github.com/rainbowhxch/accelerated-jk.nvim

縦移動の`j`と`k`を押し続けるとカーソルの移動速度が加速するプラグインです！これを使うことで、パソコン自体のキーリピート速度を変更することなく、カーソルの移動速度を変更できるので、とても便利です！

![accelerated-jkのサンプル](/images/neovim-plugin/accelerated-jk.gif)

### zen-mode.nvim

https://github.com/folke/zen-mode.nvim

現在編集中のバッファに集中できるように、不要な要素を非表示かつ中央にバッファを表示して、我々に「禅」の状態をもたらしてくれるプラグインです！

私は27インチの4kモニターを使用していますが、画面領域が広いが故にNeovimを開くとファイルがかなり左側に表示されてしまい、常に目線を左側に寄せて作業するようになってしまってましたが、このプラグインを使って画面中央に表示するようにしてから、身体的に楽にコードを書けるようになりました！

![zen-modeのサンプル](/images/neovim-plugin/zen-mode.gif)

### nvim-ufo

https://github.com/kevinhwang91/nvim-ufo

VSCodeであるような、コードの折りたたみ機能を提供してくれます！折りたたんでいる部分のプレビューも表示してくれるのでありがたい

![nvim-ufoのサンプル](/images/neovim-plugin/nvim-ufo.gif)

### comment-box

https://github.com/LudoPinelli/comment-box.nvim

かっけぇコメントアウトができるプラグインです！かっけぇコメントアウトしたい時が度々あるので助かる

![comment-boxのサンプル](/images/neovim-plugin/comment-box.gif)

## 検索系のプラグイン

### grug-far.nvim

https://github.com/MagicDuck/grug-far.nvim

検索・置換用のプラグインです！色々な検索/置換方法を提供してくれるので重宝しています！ファイル検索は基本的にTelescopeを使っていますが、置換の時にはこちらを使っています！

#### 検索時のサンプル

![grug-farの検索時のサンプル](/images/neovim-plugin/grug-far-search.gif)

#### 置換時のサンプル

![grug-farの置換時のサンプル](/images/neovim-plugin/grug-far-replace.gif)

## コーディング支援系のプラグイン

### glance.nvim

https://github.com/dnlhc/glance.nvim

コードの定義元や参照元をプレビューで表示してくれるプラグインです！元々[lspsaga](https://github.com/nvimdev/lspsaga.nvim)を使っていましたが、使わない機能もあったので、こちらに乗り換えました

![glanceのサンプル](/images/neovim-plugin/glance.gif)

### namu.nvim

https://github.com/bassamsdata/namu.nvim

コード内のシンボルを検索、ジャンプしてくれるプラグインです！コード量が多いファイルで「あの関数に移動したい！」みたいな時に便利です！[trouble.nvim](https://github.com/folke/trouble.nvim)も使ってますが、さっと確認・移動したい時に使ってます！

![namuのサンプル](/images/neovim-plugin/namu.gif)

### tiny-inline-diagnostic

https://github.com/rachartier/tiny-inline-diagnostic.nvim

LSPからの診断情報を表示しれくれるプラグインです！デザインが好みなので使用しています！

![tiny-inline-diagnosticのサンプル](/images/neovim-plugin/tiny-inline-diagnostic.gif)

### markdwonview.nvim

https://github.com/OXY2DEV/markview.nvim

Normalモードの時に、Markdwonファイルをプレビューしてくれるプラグインです！いい感じのデザインでプレビューしてくれるので、筆が進む進む

![markdownviewのサンプル](/images/neovim-plugin/markdownview.gif)

## おわりに

以上、私が愛用しているNeovimのプラグインです！まだまだ紹介したいプラグインはあるので、時間がある時に追記していけたらなと思ってます！ありがとうございました！

https://github.com/okm321/dotfiles/tree/main/nvim/lua/plugins

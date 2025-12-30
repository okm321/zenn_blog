---
title: "今年お世話になった開発環境の棚卸し 2025"
emoji: "🙇"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "vim", "ghostty", "terminal"]
published: true
---

## はじめに

師走も佳境ということで、自分の開発環境に新しく導入したツールや本年もお世話になったツールを棚卸しがてら紹介できればと思います！

## 前提: 普段の開発スタイル

中学生という多感な時期に[ブラッディ・マンデイ](https://www.tbs.co.jp/tbs-ch/item/d1618/)というドラマにハマってしまったせいで、ターミナルみたいな無機質な画面に向かって作業することがかっこいいと思っているエンジニアです。そんなわけで、普段の開発はNeovimを使い、ターミナルから出ない生活を送らせていただいてます。
最近は業務でフロントもバックエンドも触るので、tmuxで複数サーバーを立ち上げて切り替えながら作業しています。雰囲気はこんな感じです。

![開発環境の画面](/images/dev_environment_inventory/my_dev_display.gif)

ターミナルはもちろん透過させてます。

私は新しいクルトガを買ってもらうと勉強を頑張れたタイプの人間なので、開発環境も同じ。ツールを自分好みにして気分を上げることで、作業効率も上げていくスタイルを採用しています。

---

それでは、以下よりツールの紹介に入っていきます。

:::message
筆者はMacユーザーなので、Macに偏った内容の記事になっております
:::

## Macのツール系

### AeroSpace

AeroSpaceはmacOS向けのタイル型ウィンドウマネージャーです。`i3` や `sway` にインスパイアされていて、これ１つでキーボードでウィンドウを操作・配置することができます。

https://github.com/nikitabobko/AeroSpace

元々、[yabai](https://github.com/asmvik/yabai) + [skhd](https://github.com/asmvik/skhd)を使ってタイル操作を行っていたのですが、Aerospaceに乗り換えました。
刺さったポイントとしては以下です。

#### 設定がシンプル

`aerospace.toml` ファイル一つで全てのタイルの設定やキー操作などの設定が行えます。

#### スリープ復帰後などにウィンドウ配置が崩れない

`yabai` を使ってた時は、スリープ復帰後や、zoomやmeetでの画面共有時にウィンドウ配置が崩れることがあったのですが、AeroSpaceに乗り換えて以降その問題が発生することがなくなりました！

#### アプリごとにワークスペースを固定できる

AeroSpaceは「ワークスペース」という仮想的な作業空間を複数持っていて、各ワークスペースにウィンドウを配置していく、という考え方です。

![AeroSpaceのワークスペース](/images/dev_environment_inventory/aerospace_workspace.png)

設定ファイルで、「このアプリはワークスペースAに配置」、「このワークスペースはモニター1に配置」などの設定ができ、アプリが起動時に自分が配置したい場所に自動で開かれる体験は感動ものです！
自分は以下のように設定しています。

```toml:aerospace.toml
# Finderを開いた時に、フローティング（タイル管理から外す）状態にする
[[on-window-detected]]
if.app-id = "com.apple.finder"
run = "layout floating"

# Arcブラウザーを開いた時に、ワークスペースWに配置する
[[on-window-detected]]
if.app-id = "company.thebrowser.Browser"
run = "move-node-to-workspace W" # mnemonics: W - Web Browser

# Ghosttyを開いた時に、ワークスペースTに配置する
[[on-window-detected]]
if.app-id = "com.mitchellh.ghostty"
run = "move-node-to-workspace T" # mnemonics: T - Terminal

# Slackを開いた時に、ワークスペースCに配置する
[[on-window-detected]]
if.app-id = "com.tinyspeck.slackmacgap"
run = "move-node-to-workspace C" # mnemonics: C - Communication

# 各ワークスペースをどのモニターに配置するか
[workspace-to-monitor-force-assignment]
W = "main" # メインモニター
T = 2      # 2個目のモニター
C = 3      # 3個目のモニター
```

自分の詳細の設定は以下に貼っておきます。

https://github.com/okm321/dotfiles/blob/main/aerospace/aerospace.toml

#### アコーディオン表示めちゃ便利

普段はデカモニターを使って開発していますが、外出時など小さい画面で作業することもあります。そういうときにタイル型でアプリを並べると、一つ一つの表示領域が狭くなって作業しづらいんですよね。
そこでアコーディオン表示の出番です。以下のようにアコーディオン表示に切り替えることで、表示領域を広く保ちながらアプリ間をサクサク切り替えられます。

![AeroSpaceのアコーディオン](/images/dev_environment_inventory/aerospace_accordion.gif)

### CleanShot X

CleanShot XはMac向けのスクショアプリです。もう1年半ぐらい愛用していますが、今年もお世話になったので紹介させていただきます。

https://cleanshot.com/

ぶっ刺さってるポイントは以下です。

#### GIFを簡単に作れる

CleanShot Xは簡単にGIFが作成できるので、UI変更系のPRの時にGIFをサクッと撮って、descriptionに載せることが多いので便利。

#### スクショ/録画した画像/動画が画面の左下に表示される

この体験が神すぎる。これが僕の心を掴んで離さない。

![CleanShotXのドラッグアンドドロップ機能](/images/dev_environment_inventory/cleanshot_x.gif)

## ターミナル環境

続いては、ターミナル周りでお世話になったものを紹介します。

### Ghostty

Ghosttyは、Zigで作成された高速でネイティブな機能豊富なターミナルエミュレーターです。

https://ghostty.org/

私は元々[Alacritty](https://github.com/alacritty/alacritty)を使用していて、特に不満を抱えていませんでしたが、スピードも同等ということで乗り換えてみました！
基本的にAlacrittyと違いは感じてないですが、GhosttyはKittyグラフィックスプロトコルに対応しているので、特に設定とか必要なく画像表示できる点が最高です。

:::message
AlacrittyでもÜberzugやÜberzug++を使えば、表示できないことはないです
:::

自分はフロント開発する時に画像をチェックしたい時があり、その度にFinderを開いて確認していました。その手間が省けたのが結構嬉しいです。

![Ghosttyでの画像確認](/images/dev_environment_inventory/ghostty_img.png)

### Maple Mono フォント

自分がGhosttyで設定しているフォントです。

https://github.com/subframe7536/maple-font

長い間、自分好みのフォントを探し続けていましたが、ようやく出会えた感じがしています。
好きなポイントは単純に可愛いところ。そして日本語対応なので、アルファベットと漢字で雰囲気が揃っているのが気に入っています。

![Maple Mono フォントの雰囲気](/images/dev_environment_inventory/maple_mono_font.png)

### zoxide

`cd`コマンドを強化した便利コマンドです。過去に移動したディレクトリに素早く移動できるのが特徴です。
存在は知っていましたが、なぜか導入しておらず、今年に入って導入してみて「どちゃくそ便利じゃん」となったものです。

https://github.com/ajeetdsouza/zoxide

デフォルトでは`cd`の代わりに`z`コマンドを使ってディレクトリ移動します。移動履歴が記録されて、それを元に素早く移動できるようになる仕組みです。
ただ自分は「移動は`cd`だろ！」という信念があるので、`z`コマンドを`cd`にエイリアスしています。履歴を確認したい場合は`cdi`で見れるようにしています。

![zoxideの移動例](/images/dev_environment_inventory/zoxide.gif)

### yazi

yaziはターミナル用のファイルマネージャーで、Vimライクなキーバインドで操作することができます。

https://github.com/sxyazi/yazi

自分はターミナルにこもりたいので、Finderの代わりにこれを使うようにしています。操作性もよくて気に入っています！
また、Ghosttyを使っているため、画像プレビューも問題なくできます。基本的なファイル操作系は問題なくできる上、vimのように一括置換などもできるのでそれまた便利です。

![yaziの操作](/images/dev_environment_inventory/yazi.gif)

## Neovimのプラグイン

ここからは今年特にお世話になったNeovimのプラグインを紹介していきます！

### sidekick.nvim

https://github.com/folke/sidekick.nvim

AIアシスタントをNeovim内で使えるプラグインです。今の時代に欠かせないプラグイン。
以下のように、claude,codex,geminiなど好きなAI CLIツールを呼び出して使うことができます。現在のファイルや選択範囲をコンテキストとして送ることや、自分好みのプロンプトのテンプレートを作って適用することもできます！

![sidekick起動例](/images/dev_environment_inventory/sidekick_open.gif)

![sidekickカスタムプロンプト](/images/dev_environment_inventory/sidekick_prompt.gif)

### flash.nvim

https://github.com/folke/flash.nvim

高速なモーション・ジャンプ系のプラグインです。画面内の任意の場所に少ないキーストロークで移動できる優れものです。
自分は`jump`という機能をよく使って移動しています。以下のように、画面に写ってる全体が検索対象となり、任意の場所にぴょんぴょん移動できます。

![flash.nvimのデモ](/images/dev_environment_inventory/flash.gif)

### typescript-tools.nvim

https://github.com/pmizio/typescript-tools.nvim

Typescript専用のLSPプラグインです。純粋なLua実装で、`ts_ls`より高速に動作するのが特徴です。
自分は以下の記事を読ませていただき、導入してみました。確かに`ts_ls`より体感明らかに早くなっています。

https://zenn.dev/sirasagi62/articles/196e0f592a6177#pmizio%2Ftypescript-tools.nvim

## まとめ

以上が個人的に今年お世話になったものたちの棚卸しでした。このほかにもたくさん、お世話になったものはあるのですが、突出したものを紹介させていただきました。
来年も素敵なツールとの出会いがある実りある一年にしていけたらと思います。

https://github.com/okm321/dotfiles

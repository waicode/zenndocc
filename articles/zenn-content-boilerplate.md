---
title: "VSCodeが導くZenn執筆環境ボイラープレート"
emoji: "🚂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenncli", "vscode", "devcontainer", "markdown", "textlint"]
published: true
---

# Zenn CLIに最適化されたVSCode環境で書くために

これから「Zennに記事をたくさん投稿するぞ！」の気持ちを意気込みだけで終わらせないために、VSCodeに最適化された**Zenn執筆環境のボイラープレート**[^1]を作成しました。

![Zenn執筆環境のボイラープレート](/images/zenn-content-boilerplate_vscode_screenshot.png)

[**Githubのテンプレートリポジトリ**](https://github.com/waicode/zenn-content-boilerplate)として公開しているので、テンプレートを複製してすぐに記事を書き始めることができます。

[^1]: 中身を自分で埋める必要がある「テンプレート」に対して、変更せずにそのまま使える定型的なソースコードや設定ファイル群を意味します。

# Zenn執筆環境ボイラープレートの特徴

* **Zenn執筆に最適化されたVSCode拡張機能やスニペット**が自動的にインストールされます。
* VSCodeの見た目が**Zennライクなテーマカラー**になって編集対象が明確になります。
* [**Zenn CLI**](https://github.com/zenn-dev/zenn-editor)を使ってローカル環境でプレビューを確認しながら記事を書くことができます。
* 記事の内容は `markdownlint`,`textlint` で静的解析（lint）を行います。
* 英単語の誤字がないか `cspell`（Code Spell Checker）でチェックします。
* `husky` と `lint-staged` でコミット時に自動でlintします。そのため、不正な形式の文章はコミットされません。

# 前提：VSCodeとDocker（Dev Container）が必要

記事を編集するエディタは[**VSCode**](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)（Visual Studio Code）を使います。**Zennの執筆に最適化されたVSCodeの拡張機能やスニペット**がインストールされます。

また、Dockerで専用コンテナを準備して執筆環境をつくります。そのため、**ローカル環境にDockerのインストールが必要**です。特にこだわりが無ければ、GUIで簡単にDockerコンテナを導入できる[**Docker Desktop**](https://www.docker.com/products/docker-desktop/)が（個人利用であれば）無料で使えて便利です。VSCodeからコンテナにアクセスして執筆するため[**Dev ContainersのVSCode拡張機能**](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)も必要です。

VSCodeのdevcontainer環境は、独立した開発環境をつくるには打って付けの構築手法です。しかし構築のための手間と準備は必要なので、面倒だしちょっと難しそうだと思われる方もいるでしょう。

ボイラープレートであれば**事前に必要な設定がすべて組み込まれている**ので、VSCodeとDockerさえインストールされていれば、そのまま使うことができます。

# ボイラープレートの使い方

## Githubテンプレートからリポジトリを複製する

[**Githubのテンプレートリポジトリ**](https://github.com/waicode/zenn-content-boilerplate)の右上にある **"Use this template"** をクリックして、ボイラープレートからリポジトリを複製します。

![ボイラープレートからリポジトリを複製](/images/zenn-content-boilerplate_use_this_template_screenshot.png)
*Githubテンプレートからリポジトリを複製*

## コンテナを立ち上げる

VSCodeで複製したリポジトリをクローンして **"Reopen in Container"** でコンテナを立ち上げます。

![zenn-template_devcontainerの起動](https://user-images.githubusercontent.com/3455992/194732679-36cf9614-210e-4234-94b2-ff4d56508c89.gif)
*VSCodeでDev Containerを立ち上げ*

Dockerコンテナの構成管理のために、Docker Composeを使っています。コンテナを立ち上げると `./docker-compose.yml` と `docker/Dockerfile` の内容に基づきコンテナ環境を構築します。

その後で `package.json` に記述したライブラリがインストールされます。さらに `devcontainer.json` に書かれた設定によって、VSCode拡張機能がインストールされます。

:::details devcontainerの中からルート直下のdocker-compose.ymlを使っている理由
実際のコンテナ構成として使われるのはルート直下にある `./docker-compose.yml` の設定ファイルです。`.devcontainer/docker-compose.yml` では、既存の設定ファイルをオーバーライドする記述を入れてます。（公式が提供している[こちらの設定ファイル](https://github.com/microsoft/vscode-dev-containers/blob/main/containers/docker-existing-docker-compose/.devcontainer/docker-compose.yml)を参考にしています）このような構成にしておくと、VSCode環境以外でもDocker Composeの設定ファイルを使い回すことができます。

既存のDocker Composeをオーバーライドする方法については以下の記事にとても詳しくまとまっていました。

https://zenn.dev/saboyutaka/articles/9cffc8d14c6684
:::

## Zenn CLIを使う準備をする

ローカルのエディタでZennの記事が執筆できる `zenn-cli` もインストールされます。GitHubリポジトリと連携手順はZenn公式が書いている以下の記事で丁寧に解説されています。

https://zenn.dev/zenn/articles/connect-to-github

ここまでの手順でZennの記事を書き始める準備は完了しています。`zenn-cli` を使った記事の書き方は、同じくZenn公式が書いている以下の記事を参考にしてください。

https://zenn.dev/zenn/articles/zenn-cli-guide

# ボイラープレートの設定内容

記事を書く際は、**Zennに最適化されたVSCode拡張機能やスニペットを使うと便利**です。また、**コミット時に静的解析（lint）が実行**されます。文章が不正な場合、警告メッセージが表示されてコミットできません。

詳しい設定について、後述していきます。

## 記事を書くための設定

### Zenn CLIコマンドはnpmスクリプトから1クリックで実行

記事のプレビュー（`npx zenn preview`）や記事の新規作成（`npx zenn new:article`）はnpmスクリプトに登録しています。VSCodeのサイドバーから1クリックで実行できます。

![Zenn CLIのコマンドはサイドバーから1クリックで実行](/images/zenn-content-boilerplate_npm_script_screenshot.png)
*Zenn CLIのコマンドはサイドバーから1クリックで実行*

### VSCode拡張機能でマークダウンを効率よく書く

マークダウンの文章を書く効率を上げる目的で以下の拡張機能を入れています。

| 拡張機能 | 説明 |
| ---- | ---- |
| [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) | ショートカットや便利なコマンド|
| [:emojisense:](https://marketplace.visualstudio.com/items?itemName=bierner.emojisense) | 絵文字入力の補助 |
| [Insert Date String](https://marketplace.visualstudio.com/items?itemName=jsynowiec.vscode-insertdatestring) | 現在時刻をショートカット入力（Zennの日時フォーマット `YYYY-MM-DD hh:mm` で入力されるよう設定済）|
| [Copy file name](https://marketplace.visualstudio.com/items?itemName=nemesv.copy-file-name) | ファイル名を右クリックメニューからコピー|
| [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense) | パスを入力補完 |

拡張機能の使い方は以下の記事で詳しく解説しています。よければ参考にしてください。

https://qiita.com/waicode/items/1310d3f0aeb24f393b88

### 独自のマークダウン記法はVSCodeスニペットで

`.vscode/markdown.code-snippets` にZenn独自の記法を含むマークダウン記法のスニペットを登録しています。使っていきながらご自身で使いやすいスニペットに書き換えることも可能です。

https://zenn.dev/zenn/articles/markdown-guide

## 静的解析（lint）の設定

テキストにはコンパイルもなければ、実際に読まれるときの動的なチェックもありません。事前の静的解析は「糸くず（lint）」を取る以上に重要な役割を占めると考えています。

このボイラープレートには、以下に対してlintする設定が入っています。

* マークダウンの形式（`markdownlint`）
* 文章の校正（`textlint`）
* 英単語の誤字（`cspell`）

くわえて、マークダウンファイル以外はコードフォーマット（自動整形）を `prettier` を使って実施します。マークダウンファイルはmarkdownlintと設定が競合するため、意図的に対象から外しています。

### VSCodeエディタ上で問題があればリアルタイムで確認

静的解析（lint）はVSCodeの拡張機能も入れているので、問題があればエディタ上で常に確認できます。

| 拡張機能 | 説明 |
| ---- | ---- |
| [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) | VSCodeのエディタ上でマークダウン構造を解析 |
| [vscode-textlint](https://marketplace.visualstudio.com/items?itemName=taichi.vscode-textlint) | VSCodeのエディタ上でテキストを解析 |
| [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) | VSCodeのエディタ上で英単語の誤字をチェック |
| [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) | VSCodeでコードフォーマット（自動整形） |

### コミット時にも確認して不正な形式のマークダウンを警告

`husky` と `lint-staged` でコミット時に自動でlintします。そのため、不正な形式の文章はコミットされません。

コミット時のlintについては以下の記事に詳しく書いてます。知りたい方は参考にしてください。

https://qiita.com/waicode/items/33311d0a511dc821f53f

### lintの詳細設定

ボイラープレートなので、特に設定を変更せずそのまま使えます。この部分も、使っていきながら好みの設定に書き換えてください。以下では初期設定内容について説明します。

#### markdownlint

デフォルトの構文チェックは厳しめに設定されています。

そのため  `.vscode/settings.json` と `.markdownlint-cli2.jsonc` で一部ルールを調整しています。

```jsonc
"markdownlint.config": {
  "line-length": false, // MD013: Disable the maximum number of characters per sentence
  "no-duplicate-heading": false, // MD024: Allow duplicate heading text
  "no-trailing-punctuation": false, // MD026: Allow headings with . ,;:
  "no-inline-html": false, // MD033: Allow HTML description
  "no-bare-urls": false // MD034: Allow URLs to be written as is
}
```

#### textlint

ベースとして以下の2つのプリセットを適用しています。

| プリセット名 | 説明 |
| ---- | ---- |
| [textlint-rule-preset-ja-spacing](https://github.com/textlint-ja/textlint-rule-preset-ja-spacing) | 日本語のスペース有無を決定するtextlintルールプリセット |
| [textlint-rule-preset-ja-technical-writing](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing) | 技術文書向けのtextlintルールプリセット |

プリセットのままだと厳し過ぎる箇所があるため `.textlintrc` で設定を一部上書きしています。

```json:.textlintrc
{
  "plugins": {
    "@textlint/markdown": {
      "extensions": [
        ".md"
      ]
    }
  },
  "filters": {
    "comments": true
  },
  "rules": {
    "prh": {
      "rulePaths": [
        "./prh/index.yml"
      ]
    },
    "preset-ja-technical-writing": {
      "sentence-length": {
        "max": 150
      },
      "no-exclamation-question-mark": {
        "allowFullWidthExclamation": true,
        "allowFullWidthQuestion": true
      },
      "ja-no-successive-word": false,
      "ja-no-mixed-period": {
        "allowPeriodMarks": [
          ":",
          "："
        ]
      },
      "no-doubled-joshi": {
        "strict": false,
        "allow": [
          "も",
          "や",
          "か"
        ],
        "separatorCharacters": [
          ",",
          "，",
          "、",
          ".",
          "．",
          "。",
          "?",
          "!",
          "？",
          "！",
          "「",
          "」",
          "\"",
          "”",
          "“"
        ]
      }
    },
    "preset-ja-spacing": {
      "ja-space-around-code": {
        "before": true,
        "after": true
      }
    }
  }
}
```

また、校正用辞書を `/prh/rules` へ追加できるようにしています。初期設定では `/prh/rules/tech.yml` に技術用語の校正辞書をいくつか登録しています。必要に応じて用語を追加してください。

```yml:/prh/rules/tech.yml
meta:
  - title: 技術用語の固有名詞ルール
rules:
  - expected: インターフェース
    patterns:
      - インターフェイス
      - インタフェース
      - インタフェイス
    prh: 技術用語
  - expected: ソフトウェア
    pattern: ソフトウェアー
    prh: 技術用語
  - expected: ハードウェア
    pattern: ハードウェアー
    prh: 技術用語
  - expected: デフォルト
    pattern: ディフォルト
    prh: 技術用語
```

#### cspell

許可する言葉は `.cspell.json` の `words` に追加します。対象の文字列にカーソルを合わせてVSCodeのクイックフィックスで登録するのが便利です。

![VSCodeのクイックフィックスでcspellのwordsに登録](https://user-images.githubusercontent.com/3455992/194875399-0036c51e-bbd2-4bc4-928b-0790ae622b5e.gif)
*VSCodeのクイックフィックスでcspellのwordsに登録*

#### Prettier

`.prettierignore` に `*.md` 記載して、マークダウンファイルをコードフォーマットの対象外にしています。その他の設定は特に入れていません。自動整形時はデフォルト設定が適用されます。

# Zenn執筆環境ボイラープレートのGithubリポジトリ

ここまでお読みいただきありがとうございました。Zenn CLIで記事を書いてみたい！と思っていたけど設定が面倒でこれまで重い腰が上がらなかった方は、この機会にぜひご利用ください。

https://github.com/waicode/zenn-content-boilerplate

# Qiita執筆環境のボイラープレート

同様の仕組みで作成した**Qiita執筆環境のボイラープレート**も公開しています。非公式のQiita連携ツールを使って、ZennのようにGithub連携して記事を書くことができます。Qiitaで記事を書いている方は、こちらも合わせて参考にしてみてください。

https://qiita.com/waicode/items/61ec0f9478fdb8a8e542

# 技術ブログと同じ環境でZennを書けるようにした結果

もともと「技術ブログと同じ環境で記事が書けたら、効率もモチベーションも上がりそう」がボイラープレート作成に至ったきかっけでした。実際につくってみて、作業効率以上に**情報発信面でメリット**を感じることができました。

詳しくは以下の記事にまとめています。よろしければご覧ください。

https://archt.blue/articles/boilerplate

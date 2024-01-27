---
title: ボクが考えたサイキョーの技術記事リポジトリ
tags:
  - 'GitHub', 'Qiita', 'Zenn', CLI'
private: true
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
この記事では GitHub リポジトリと Qiita や Zenn が提供している CLI ツールを使って技術ノートリポジトリを作成してみたという記事です。
私は主に Qiita を使っていて、Qiita メインで書いていきます。

# 何をしたか
Qiita CLI を使用して技術ノートリポジトリを作成しました。

このリポジトリでは以下のような感じで使っています。
- Issue: 技術ノートまたは勉強ノート、メモ帳
- ディレクトリ：記事の管理、Issueテンプレートの管理
- Pull Request: 執筆した記事を main ブランチにマージする(main ブランチに直接でも良いが個人的に好きではない)
- GitHubAction: 様々な自動化に使う

そして以下のようなワークフローで執筆していきます。
1. Issue に直面した問題、勉強した内容を記述する
2. それを元に記事を作成する
3. PRを作成して、どの Issue でどのような内容を書いたか記述
4. マージして公開

このワークフローを負担なく行うため以下のような自動化を行いました。
- main ブランチにマージしたら自動的に公開する
- 記事の生成とブランチの作成を(半)自動的に行う
- issue のテンプレート化
- 非アクティブな issue の自動クローズ

では、それぞれ簡単に解説していきます。

## main ブランチにマージしたら自動的に公開する
これは Qiita CLI でプロジェクトを作成すると自動的に生成されます。
main ブランチに push またはマージされた場合に Action が走り、Qiita に自動的に公開されます。

##記事の生成とブランチの作成を(半)自動的に行う
ローカルで `npx qiita new [記事名]`　とコマンドを打つのが面倒だったので、自動化しました。
(半)とついているのは、Action に `work_flow` があるからです。つまり、GitHub Action へ移動してUIボタンを押さないと生成されません。

処理は以下のようになっています。
```yml
name: Create Content
on:
  workflow_dispatch:
    inputs:
      issue_number:
        description: '記事のIssue番号'
        required: true
        type: string

permissions:
  contents: write
  pull-requests: write

jobs:
  create-content:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set name and email to Git
        run: |
          git config --local user.name github-actions[bot]
          git config --local user.email 41898282+github-actions[bot]@users.noreply.github.com

      - name: Create and switch work branch
        id: create_work_branch
        run: |
          WORK_BRANCH_NAME="article/${{ github.event.inputs.issue_number }}"
          git switch -c $WORK_BRANCH_NAME
          echo "work_branch_name=$WORK_BRANCH_NAME" >> $GITHUB_ENV

      - name: Create Article
        id: create_article
        run: |
          npm install
          npx qiita new ${{ github.event.inputs.issue_number }}_article
          git add .
          git commit -m "Create new Article Issue ${{ github.event.inputs.issue_number }}"
          git push origin "article/${{ github.event.inputs.issue_number }}"

      - name: Create pull request
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh --version
          gh pr create \
            --title "Create Article #${{ github.event.inputs.issue_number }}" \
            --body "Closes #${{ github.event.inputs.issue_number }}" \
            --base "${{ github.event.repository.default_branch }}" \
            --head "${{ steps.create_work_branch.outputs.work_branch_name }}" \
```
簡単に説明すると、技術ノートとなっている Issue からどれか1つ　Issue 番号を入力して、それを元にブランチをからの記事、PullRequestを作成します。

## issue のテンプレート化
技術ノートを取るために、ある程度テンプレート化したかったので、Issueテンプレートを作成しました。

Issueのテンプレートは多くのプロジェクトで目にしますが、よりテンプレート化を図るため、 Form の形にするべく、yml で作成しました。

## 非アクティブな issue の自動クローズ
非アクティブな issue は自動的にクローズするようにしました。これにより、issue 内を整理することができます。
処理の内容としては、30日間の活動がないと `state` タグが付与され、それでも 14日間の反応がなかったら自動的にクローズするようにしました。
これは、GitHubの公式ドキュメントを参考に作成しました。

https://docs.github.com/ja/actions/managing-issues-and-pull-requests/closing-inactive-issues

# 今後の方針
**Web上での編集を取得**が現状の一番の課題です。

# my-textlint

my [textlint](https://github.com/textlint/textlint) configulation

## cli usage

```
ghq get git@github.com:MH4GF/my-textlint.git
cd $HOME/.ghq/github.com/MH4GF/my-textlint && npm install
echo "alias textlint='$HOME/.ghq/github.com/MH4GF/my-textlint/node_modules/.bin/textlint -c $HOME/.ghq/github.com/MH4GF/my-textlint/.textlintrc'" > ~/.zshrc

# anywhere
textlint hogehoge.md
```

## github actions usage to start in another repository (with reviewdog)

```
name: reviewdog

# Controls when the action will run.
on: [pull_request]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  reviewdog-github-check:
    name: reviewdog (github-check)
    runs-on: ubuntu-latest

    steps:
      #reviewdogのアクション
      - uses: reviewdog/action-setup@v1
        with:
          reviewdog_version: latest

        #textlintを動かすためのnodeアクション
      - uses: actions/setup-node@v2

      - uses: actions/checkout@v2

      - name: cache-node-modules
        #stepsが失敗(文章の乱れ)した場合でもcacheを取得するようにする
        uses: pat-s/always-upload-cache@v2.1.3
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            node-

      - name: Install textlint
        run:  'npm install --save-dev textlint textlint-rule-preset-ja-technical-writing textlint-rule-prh'

      - name: Get .textlintrc
        # パブリックリポジトリから.textlintrcを取得する
        run: |
          echo "$(curl -fsSL https://raw.githubusercontent.com/MH4GF/my-textlint/main/.textlintrc)" > .textlintrc
      - name: Execute textlint for articles
        run: |
          npx textlint -f checkstyle "articles/*.md" >> .textlint.log

      - name: Run reviewdog
        # textlintで文章上のミスがあった場合のみ、reviewdogを実行させるようにする
        if: failure()
        env:
          REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          cat .textlint.log | reviewdog -f=checkstyle -name="textlint" -reporter="github-pr-review"
```

refs: [GitHub ActionsでZennブログの校正を自動化してみた](https://zenn.dev/yuta28/articles/blog-lint-ci-reviewdog)

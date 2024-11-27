- tokenを作成
  - 右上のアイコンから`Settings`→`Developer Settings`→`Tokens→Generate new token(classic)`→tokenを作成(`repo`・`read:project`にのみチェックを入れる)→tokenをコピーしておく(画面を離れると二度と確認できない)

- シークレットを作成
  - 自動化したいリポジトリの`settings`→`secrets`→`actions`→`New repository secret`→シークレット名と作成したtokenを設定

- 対象ブランチの`.github/workflows`内にyamlファイルを追加する(以下がyamlファイルの例)
```
name: Add bugs to bugs project

on:
  issues:
    types:
      - opened
      - reopened

jobs:
  add-to-project:
    name: Add issue to project
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@main
        with:
          project-url: https://github.com/users/TokiNoviceProgrammer/projects/3
          github-token: ${{ secrets.ADD_TO_PROJECT_PAT }}
```
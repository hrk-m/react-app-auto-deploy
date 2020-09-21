# CI が通れば Slack に通知をする

1. Slack のチャンネルを作成する

2. Slack に CircleCI アプリを追加する

- 設定と管理 → アプリ管理を選択する<br>
  <img src='img/slack-01.png' width='600px'><br>

- App ディレクトリを検索から CircleCI と検索し`Slack に追加`を選択<br>
  <img src='img/slack-02.png' width='600px'><br>

- Slack に流したいチャンネルを選択し `CircleCIインテグレーションの追加`を選択<br>
  <img src='img/slack-03.png' width='600px'><br>

- Slack に追加した後は`セットアップの手順` の設定を手順通りに進めていく<br>
- Project settings を選択<br>
  <img src='img/slack-04.png' width='600px'><br>

- `Slack Integration`タブの `Add WebHook URL` を選択する<br>
  <img src='img/slack-05.png' width='600px'><br>

- `セットアップの手順のStep 2`で Webhook URL が取得できるのでそれを CircleCI に登録する<br>
- 終わったら Save ボタンを押す<br>
- Web Hook URL は ↓ のリンクからも追加可能
- https://slack.com/services/new/incoming-webhook
- https://hooks.slack.com/services/xxxxxxxxxx/xxxxxxxxxx の様な URL が取得できる
  <img src='img/slack-06.png' width='600px'><br>

## もしくは環境変数から追加 ( 通知先を分ける時に使用すると気に使用する )

### CircleCI の環境変数に webhook の url を設定

- `Environment Variables` → `Add Variable`<br>
  <img src='img/slack-07.png' width='600px'><br>

- `CIRCLECI_SLACK_WEBHOOK`の変数で WEBHOOK URL を登録する<br>
  <img src='img/slack-08.png' width='600px'><br>

3. `.circleci/config.yml` に通知設定をする

```yml
orbs:
  slack: circleci/slack@3.4.2
version: 2.1
executors:
  slack-executor:
    docker:
      - image: 'cibuilds/base:latest'
    resource_class: small

jobs:
  notify-via-slack:
    executor: slack-executor
    steps:
      - slack/notify:
          message: '${CIRCLE_BRANCH} branch deployment to aws s3 and cloudfront is complete.'
          webhook: $SLACK_WEBHOOK
      - slack/status:
          success_message: ':ok:\nBranch：$CIRCLE_BRANCH のデプロイが完了しました\n User：$CIRCLE_USERNAME'
          failure_message: ':ng:\nBranch：$CIRCLE_BRANCH のデプロイが失敗しました\n User：$CIRCLE_USERNAME'
          webhook: $SLACK_WEBHOOK

workflows:
  build_and_deploy:
    jobs:
      - notify-via-slack:
          filters:
            branches:
              only:
                - develop
                - staging
                - master
```

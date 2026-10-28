---
name: guardrails
description: Claude Code の hooks と settings(チーム用 .claude/settings.json、管理者用 managed settings)でガードレールと承認ゲートを設計・実装する。保護パスの編集禁止、編集後の自動 format/lint、秘密情報の混入防止、人間承認を hook に置き換える、auto mode や CI での自律実行の安全柵を作りたいときに使う。
---

# guardrails: hooks と settings による強制(Build / Deploy)

スキルや CLAUDE.md は助言にすぎない。**必ず守られるべきこと**は hook(決定的に実行されるスクリプト)と settings で強制する。

## 1. ビルド時のガードレール(速く、軽く)

編集のたびに走るので **速いものだけ** にする。重いチェックはコミット時・PR 時に回す。

| 目的 | イベント | 振る舞い |
| --- | --- | --- |
| 保護パス(migrations、本番設定、生成物、ロックファイル等)の編集禁止 | PreToolUse(Edit / Write) | ブロックし理由を返す |
| 秘密情報を diff に入れない | PreToolUse(Edit / Write)、Bash の `git commit` | パターン検出でブロック |
| 編集後に formatter / linter | PostToolUse(Edit / Write) | 自動実行し、失敗は Claude に返して直させる |
| バグ修正中のテストファイル編集禁止 | PreToolUse(Edit / Write) | `verify-loop` のテスト先行フローと併用 |

実装例は [references/hook-examples.md](references/hook-examples.md) を参照。

## 2. 人間の承認を hook に置き換える

いま人間が「承認」しているポイントを棚卸しし、それぞれを hook に変換する。hook は次の 3 通りで応答する:

- **allow**: 条件を満たすので通す
- **ask**: 人間に確認を求める
- **block / deny**: 止める。**なぜ止めたか、どうすれば通るかを必ず説明する**(Claude がそれを読んで自分で直せるように)

配置:

- チームで共有するもの → リポジトリの `.claude/settings.json`
- 交渉の余地がないもの → 管理者配布の managed settings(利用者が上書きできない)

## 3. managed settings(規制の厳しい組織の出発点)

以下は「そのまま使うテンプレート」ではなく、自組織に合わせて調整する出発点:

- 秘密情報ファイル(`.env`、鍵、認証情報)の読み取りとネットワーク外向き通信の deny ルール
- 安全なコマンド(テスト、lint、読み取り系 git)の事前許可
- 権限バイパスモードの無効化
- OS レベルのサンドボックス + 許可ドメインのリスト
- hooks と MCP サーバーを管理者定義のものだけに限定
- 承認済みプラグインマーケットプレイスのみ許可
- Claude Code の最低バージョン指定

具体的なキー名は Claude Code の settings / hooks / sandboxing のドキュメントで最新を確認すること。

## 4. CI/CD での自律度

- まずは **読み取りだけの判断ステップ** から: 失敗ビルドのトリアージ、flaky テストの要約、changelog の下書き
- 次に **PR を開くだけ** の書き込みステップを追加
- デプロイはスコープを絞った MCP ツールとして公開する
- 環境ごとに自律度を段階化: dev は自由、本番はリリースマネージャーの承認が必要
- ロールバックは staging でリハーサルしておく
- 原則: **エージェントは本番ゲートの手前までは動いてよいが、ゲートは越えない**

## 進め方

1. 対象リポジトリの現状(既存 hooks、CI、保護すべきパス、秘密情報の置き場)を調べる
2. 上の表から必要なものを提案し、ユーザーと優先度を決める
3. hook スクリプトと settings を実装し、**わざと違反する操作で発火を確認**する
4. ブロック時のメッセージが「理由 + 通す方法」になっているか確認する

## 測定

- ゲート待ち時間、ゲート違反件数
- hook による誤ブロック(正当な操作を止めた)の件数

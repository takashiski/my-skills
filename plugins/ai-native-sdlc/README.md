# ai-native-sdlc

[The AI-Native SDLC Playbook](https://claude.com/ja/resources/articles/the-ai-native-sdlc-playbook) をスキル化したプラグイン。

各ステージが次のステージの読む成果物をリポジトリにコミットし、運用での検知が新しい intent として先頭に戻るループを回す。

```
intent.md → spec.md → plan.md → diff + tests → PR(REVIEW.md) → 本番 → 検知 → intent.md …
```

## インストール

```text
/plugin marketplace add takashiski/my-skills
/plugin install ai-native-sdlc@my-skills
```

## スキル

| スキル | ステージ | 何をするか |
| --- | --- | --- |
| `playbook` | 全体 | 全体像、導入順序、現状診断。迷ったらここから |
| `intent` | Plan | 壁打ちから intent.md を作る |
| `spec` | Design | intent.md → spec.md、ポリシー観点の懸念をフラグ |
| `plan` | Build | plan mode で調査し、別人が実装できる plan.md を作る |
| `claude-md` | Build | CLAUDE.md の作成・チューニング、スキルへの切り出し |
| `guardrails` | Build / Deploy | hooks / settings による強制と承認ゲート |
| `verify-loop` | Test | 単一コマンド検証、テスト先行、UI 比較、継続的 eval |
| `review` | Deploy | REVIEW.md の作成と、それに沿った PR レビュー |
| `maintain` | Maintain | 逸脱検知 → intent 化、定期スキャン、インシデント教訓 |

## エージェント

| エージェント | 用途 |
| --- | --- |
| `verifier` | 書いた文脈を持たない新しいコンテキストで、受け入れ条件を満たしているか判定する |

## 使い方の例

```text
/ai-native-sdlc:playbook このリポジトリの現状を診断して、最初にやることを提案して
/ai-native-sdlc:intent 入場ゲートでQRが読めないときの代替手段がほしい
/ai-native-sdlc:spec intent/20261027-qr-fallback.md
/ai-native-sdlc:plan intent/20261027-qr-fallback/spec.md
/ai-native-sdlc:review
```

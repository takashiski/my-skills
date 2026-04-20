# test-spec-review

Excel化前の不具合定義とテスト仕様書ドラフトを、Markdown ベースで事前レビューするためのスキルです。

## 何をするスキルか

- 不具合定義が粗いまま後続レビューへ進むことを止める
- テスト観点の軸分解、同値分割、境界値、代表ケース選定を確認する
- 再現性、期待挙動の根拠、対象範囲の切り出しを確認する
- 差し戻し時は固定ラベルで短く返し、後続未評価を明示する

## 想定している使い方

1. 作成者が Markdown でドラフトを書く
2. このスキルで事前レビューする
3. 差し戻しがなくなるまで修正する
4. 通過後に Excel へ起こす

## ファイル構成

```text
test-spec-review/
  SKILL.md
  README.md
  references/
    markdown-draft-template.md
    review-result-template.md
    review-notes.md
```

## 参照ファイル

- [SKILL.md](SKILL.md): スキル本体
- [references/markdown-draft-template.md](references/markdown-draft-template.md): 作成者向け Markdown テンプレート
- [references/review-result-template.md](references/review-result-template.md): レビュー結果テンプレート
- [references/review-notes.md](references/review-notes.md): ラベル説明と運用補足

## このスキルで特に見ること

- 対象不具合の説明が評価可能な粒度になっているか
- 再現条件と期待挙動の根拠があるか
- 全体候補と今回対象が分離されているか
- 数値条件で同値分割、境界値、代表値の理由が書かれているか
- 選択肢条件で同値扱いの根拠が書かれているか
- 既知の誤実装を排除できるテスト集合になっているか

## 補足

GitHub での公開、`gh skill` による配布、インストール手順はリポジトリルートの [README.md](../../README.md) を参照してください。

## バージョン管理方針

- このディレクトリ配下を公開用の正本とする
- 変更後はリポジトリ単位で `gh skill publish --dry-run` を実行して検証する
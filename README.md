# my-skills

takashiski の個人用 Claude Code プラグインマーケットプレイスです。

## インストール

```text
/plugin marketplace add takashiski/my-skills
/plugin install ai-native-sdlc@my-skills
/plugin install slide-info-design@my-skills
```

ローカルで試す場合:

```text
/plugin marketplace add D:/dev/my-skills
```

## 収録プラグイン

| プラグイン | 内容 |
| --- | --- |
| [ai-native-sdlc](plugins/ai-native-sdlc/) | AI ネイティブ SDLC プレイブックのスキル化。intent → spec → plan → 検証 → レビュー → 運用検知のループ |
| [slide-info-design](plugins/slide-info-design/) | 『情報デザインを意識したスライド作成入門』のスキル化。スライド作成・レビュー(CC BY-SA 4.0) |

## ディレクトリ構成

```text
my-skills/
  .claude-plugin/
    marketplace.json        # マーケットプレイス定義(plugins 配列に登録する)
  plugins/
    <plugin-name>/
      .claude-plugin/
        plugin.json         # プラグインのメタデータ
      skills/
        <skill-name>/
          SKILL.md
          references/       # 任意。スキルから参照する補助資料
      agents/               # 任意。サブエージェント定義
      commands/             # 任意。スラッシュコマンド
      hooks/                # 任意。hooks.json
```

## プラグインを追加する手順

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` を作る(`name` はディレクトリ名と揃える)
2. `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` にスキルを置く(frontmatter に `name` と `description` 必須)
3. `.claude-plugin/marketplace.json` の `plugins` 配列にエントリを追加する

```json
{
  "name": "<plugin-name>",
  "source": "./plugins/<plugin-name>",
  "description": "一行説明",
  "version": "0.1.0"
}
```

4. `claude plugin validate .` で検証してから push

## 運用ルール

- 各プラグインの `version` は変更のたびに上げる(利用側の更新検知に使われる)
- 改行コードは LF に統一(`.gitattributes` で固定)

# my-skills

GitHub で管理し、`gh skill` で配布、プレビュー、インストールするためのスキル置き場です。

## ディレクトリ構成

```text
my-skills/
  skills/
    test-spec-review/
      SKILL.md
      README.md
      references/
        markdown-draft-template.md
        review-result-template.md
        review-notes.md
```

## 使い方

このリポジトリを GitHub に push したあと、`gh skill` で扱います。

例:

```powershell
gh skill preview <OWNER>/<REPO> skills/test-spec-review
gh skill install <OWNER>/<REPO> skills/test-spec-review --agent claude-code --scope user
```

## 補足

- GitHub 公開用、配布用はこの `my-skills/` 配下を正とする
- 各スキル個別の説明は `skills/<skill-name>/README.md` を参照する
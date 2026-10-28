# hook 実装例

Windows / macOS / Linux で動くよう Node.js スクリプトで書いた例。hook は標準入力で JSON を受け取り、**終了コード 2 で終了するとブロック**され、標準エラー出力の内容が Claude に返される。より細かい制御(allow / ask / deny)は JSON 出力で行える。仕様は Claude Code の hooks ドキュメントで最新を確認すること。

## .claude/settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "node .claude/hooks/protect-paths.mjs" },
          { "type": "command", "command": "node .claude/hooks/no-secrets.mjs" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "node .claude/hooks/format.mjs" }
        ]
      }
    ]
  }
}
```

## protect-paths.mjs(保護パスの編集をブロック)

```js
import { readFileSync } from "node:fs";

const input = JSON.parse(readFileSync(0, "utf8"));
const file = (input.tool_input?.file_path ?? "").replaceAll("\\", "/");

const PROTECTED = [
  { re: /\/migrations\//, why: "マイグレーションは既存ファイルを編集せず、新しいマイグレーションを追加してください" },
  { re: /(^|\/)package-lock\.json$|(^|\/)pnpm-lock\.yaml$/, why: "ロックファイルは直接編集せず、パッケージマネージャーのコマンドで更新してください" },
  { re: /\/config\/production\./, why: "本番設定の変更は人間のレビューが必要です。変更案を plan.md に書いて提案してください" },
];

for (const p of PROTECTED) {
  if (p.re.test(file)) {
    console.error(`ブロック: ${file}\n理由: ${p.why}`);
    process.exit(2);
  }
}
```

## no-secrets.mjs(秘密情報らしき文字列の混入をブロック)

```js
import { readFileSync } from "node:fs";

const input = JSON.parse(readFileSync(0, "utf8"));
const ti = input.tool_input ?? {};
const text = [ti.content, ti.new_string, ...(ti.edits ?? []).map((e) => e.new_string)]
  .filter(Boolean)
  .join("\n");

const PATTERNS = [
  /AKIA[0-9A-Z]{16}/,                       // AWS access key
  /-----BEGIN [A-Z ]*PRIVATE KEY-----/,
  /ghp_[A-Za-z0-9]{36}/,                    // GitHub token
  /sk-[A-Za-z0-9_-]{20,}/,                  // 各種 API キー
];

const hit = PATTERNS.find((re) => re.test(text));
if (hit) {
  console.error("ブロック: 秘密情報らしき文字列が含まれています。環境変数やシークレットストア経由で参照するよう書き換えてください。");
  process.exit(2);
}
```

## format.mjs(編集後に formatter を実行)

```js
import { readFileSync } from "node:fs";
import { execFileSync } from "node:child_process";

const input = JSON.parse(readFileSync(0, "utf8"));
const file = input.tool_input?.file_path;
if (!file || !/\.(ts|tsx|js|mjs|json|md)$/.test(file)) process.exit(0);

try {
  execFileSync("npx", ["prettier", "--write", file], { stdio: "pipe", shell: process.platform === "win32" });
} catch (e) {
  console.error(`format 失敗: ${file}\n${e.stderr?.toString() ?? e.message}`);
  process.exit(2); // Claude に返して直させる
}
```

## テスト先行のバグ修正中にテスト編集をブロック

環境変数やマーカーファイル(例: `.claude/state/bugfix-mode`)が存在する間だけ、`*.test.ts` / `__tests__/` への編集を exit 2 で止める。`verify-loop` スキルのフローと組み合わせて使う。

## JSON 出力で allow / ask / deny を返す例

```js
console.log(JSON.stringify({
  hookSpecificOutput: {
    hookEventName: "PreToolUse",
    permissionDecision: "ask",          // "allow" | "ask" | "deny"
    permissionDecisionReason: "本番 DB に接続するコマンドです。人間の確認が必要です",
  },
}));
process.exit(0);
```

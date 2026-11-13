# company-knowledge

社内ランブックをキーワード検索する MCP サーバーを同梱した Claude Code プラグインです。

> **Note:** 現在の検索処理（`searchDb`）はスタブで、どのクエリにも固定の結果（`Deploy rollback`）を返します。実際のデータソースへの接続は未実装です。

## 提供するツール

### `search_runbook`

社内ランブックをキーワードで検索し、ヒットしたページのタイトルと抜粋を返します。インシデント調査・オンボーディング・デプロイ手順の確認に使う想定です（コード検索には使いません）。

| 引数 | 型 | 必須 | 説明 |
| --- | --- | --- | --- |
| `query` | string | ✓ | 検索キーワードまたはエラーメッセージ |
| `limit` | integer (1–20) | | 最大件数。既定値 `5` |

出力例:

```
- Deploy rollback: ...
```

ヒットがない場合は `No results` を返します。

## 必要環境

- Node.js 20 以上
- Claude Code

## セットアップ

```bash
npm install   # prepare スクリプトで dist/ がビルドされる
```

ソースを変更したら再ビルドします。

```bash
npm run build # tsc で src/ → dist/ にコンパイル
npm run dev   # ビルドせず tsx で src/server.ts を直接起動
```

## Claude Code で使う

### プラグインとして

`.claude-plugin/plugin.json` と `.mcp.json` を含むので、このディレクトリをプラグインとして読み込むと MCP サーバー `company-knowledge` が登録されます。サーバーは `${CLAUDE_PLUGIN_ROOT}/dist/server.js` から起動されるため、事前に `npm install` か `npm run build` で `dist/` を用意してください。

```bash
claude --plugin-dir /path/to/my-mcp-plugin
```

### MCP サーバー単体として

```bash
claude mcp add company-knowledge -- node /path/to/my-mcp-plugin/dist/server.js
```

## ディレクトリ構成

```
.
├── .claude-plugin/plugin.json  # プラグインのメタデータ
├── .mcp.json                   # プラグインが登録する MCP サーバー定義
├── bin/company-kb              # サーバー起動用ラッパースクリプト
├── src/server.ts               # MCP サーバー本体（stdio トランスポート）
├── dist/                       # ビルド成果物
├── package.json
└── tsconfig.json
```

`bin/company-kb` は `CLAUDE_PLUGIN_ROOT` が設定されていればそこから、なければスクリプト自身の位置から `dist/server.js` を起動します。

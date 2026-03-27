# プラグインの実行可能コードと依存管理

## プラグインはマークダウンだけではない

スキルとエージェントはマークダウン（Claude への指示）だが、フック・MCP・LSP は実行可能コードを含む。

| コンポーネント | 実体 |
|-------------|------|
| スキル | マークダウンのみ |
| エージェント | マークダウンのみ |
| フック | 実行可能コード（sh, node, python 等） |
| MCPサーバー | 実行可能コード（サブプロセスとして起動） |
| LSPサーバー | 実行可能コード（言語サーバーバイナリ） |

## 実行コードの例

### フック

```json
{
  "type": "command",
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

### MCPサーバー

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"]
    }
  }
}
```

### LSPサーバー

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"]
  }
}
```

## なぜ node_modules 等の依存が必要か

MCPサーバーやフックのスクリプトが npm パッケージ等に依存するため。典型的なパターンとして、セッション開始時のフックで `npm install` を実行し、依存を `~/.claude/plugins/data/{id}/node_modules/` にインストールする。

## plugins/data/{id}/ の {id} とは

プラグイン識別子の特殊文字をハイフンに置換したもの。

```
プラグイン名: formatter@my-marketplace
       {id}: formatter-my-marketplace
      パス: ~/.claude/plugins/data/formatter-my-marketplace/
```

## 2つのパス変数

| 変数 | 解決先 | 特徴 |
|-----|-------|------|
| `${CLAUDE_PLUGIN_ROOT}` | プラグイン本体のキャッシュ | 更新時に上書きされる |
| `${CLAUDE_PLUGIN_DATA}` | `~/.claude/plugins/data/{id}/` | 更新後も永続する |

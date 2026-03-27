# Claude Code プラグイン概要

## プラグインとは
Claude Code の機能を拡張するためのパッケージシステム。スキル、エージェント、フック、MCPサーバー、LSPサーバーなどをまとめて配布・共有できる。

## プラグインの構成

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # マニフェスト（名前、説明、バージョン）
├── skills/                  # カスタムスキル
├── agents/                  # カスタムエージェント
├── hooks/                   # イベントハンドラー
├── .mcp.json               # MCPサーバー設定
└── .lsp.json               # LSPサーバー設定
```

## スタンドアロン (.claude/) との違い

| 方式 | スキル名 | 用途 |
|------|---------|------|
| スタンドアロン | `/hello` | 個人的なワークフロー、プロジェクト固有のカスタマイズ |
| プラグイン | `/plugin-name:hello` | チーム共有、コミュニティ配布、バージョン管理 |

## インストールの仕組み

インストール = 「キャッシュにDL」+「settings.json に登録」の2段階。

### ダウンロード先（全プラグイン共通）

```
~/.claude/plugins/cache/   ← プラグイン本体の実体
```

グローバルなキャッシュで、どのプロジェクトから入れても同じ場所に保存される。

### 永続データ

```
~/.claude/plugins/data/{id}/   ← node_modules, venv など
```

プラグイン更新後も残るデータの保存先。

### 有効化の登録（スコープで異なる）

| スコープ | 書き込み先 | 他プロジェクトで使える？ | チーム共有？ |
|---------|-----------|-------------------|-----------|
| `--scope user` | `~/.claude/settings.json` | 使える（全プロジェクト共通） | No |
| `--scope project` | `project/.claude/settings.json` | 使えない（そのプロジェクト専用） | Yes（git に入る） |
| `--scope local` | `project/.claude/settings.local.json` | 使えない（自分専用） | No（gitignore） |

settings.json には以下のように書かれる:

```json
{
  "enabledPlugins": [
    "plugin-name@marketplace-name"
  ]
}
```

### プロジェクトにあるのはポインタだけ

```
project1/
  └─ .claude/settings.json   ← "このプラグインを有効にする" という設定だけ

~/.claude/plugins/cache/     ← プラグインのコード実体はここ
```

project スコープで git 共有する場合も、共有されるのは「このプラグインを使う」という宣言だけ。各メンバーの環境でそれぞれインストールが必要。

## セッション中の使われ方

### ロードタイミング

セッション開始時に enabledPlugins を見て全部ロードされる（オンデマンドではない）。途中で変更したら `/reload-plugins`。

### コンポーネントごとの動作

| コンポーネント | 使われ方 | コンテキストへの載り方 |
|-------------|---------|-------------------|
| スキル | `/plugin:skill` で手動 or Claude が自動呼び出し | 説明文のみ載る。本文は呼び出し時に遅延ロード |
| エージェント | Claude が必要時に自動スポーン | 説明文が載る |
| フック | イベント（セッション開始、ツール使用後等）で自動発火 | コンテキストに載らない |
| MCPサーバー | 起動→ツールが Claude のツール一覧に追加 | コンテキストに載らない |
| LSPサーバー | バックグラウンドで診断情報を提供 | コンテキストに載らない |

### スキルの呼び出しモード

```yaml
# SKILL.md のフロントマター
---
# (デフォルト) ユーザーもClaudeも呼べる
disable-model-invocation: true   # ユーザーのみ
user-invocable: false            # Claudeのみ
---
```

## ユーザーからの見え方

| コンポーネント | 見え方 |
|-------------|-------|
| スキル | `plugin-name:skill-name` で表示 → どのプラグインかわかる |
| エージェント | `plugin-name:agent-name` で表示 → わかる |
| MCPツール | `mcp__server-name__tool-name` で表示 → サーバー名まではわかる |
| フック | バックグラウンド発火 → 見えない（verbose/debug モードで確認可能） |

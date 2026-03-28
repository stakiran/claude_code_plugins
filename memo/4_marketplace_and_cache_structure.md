# マーケットプレイスとキャッシュの保存構造

## ディレクトリ構成

```
~/.claude/plugins/
├── known_marketplaces.json          ← 登録済みマーケットプレイス一覧
├── marketplaces/
│   └── anthropics-claude-code/      ← クローンされたマーケットプレイスリポジトリ
│       └── marketplace.json         ← プラグインカタログ
├── cache/                           ← インストール済みプラグイン本体
└── data/                            ← プラグインの永続データ（node_modules等）
```

## マーケットプレイス登録とプラグインインストールの2段階

### Step 1: マーケットプレイス登録

```bash
/plugin marketplace add owner/repo
```

- `known_marketplaces.json` にエントリ追加
- `marketplaces/<name>/` にリポジトリを git clone

### Step 2: プラグインインストール

```bash
/plugin install plugin-name@marketplace-name
```

- marketplace.json を見てソースを特定
- そのソースから取得（git clone, npm download 等）
- `cache/` に保存

## なぜ marketplaces/ から直接使わず cache/ にコピーするか

marketplaces/ はカタログ（一覧）であり、プラグイン本体を全部含んでいるとは限らない。

```json
{
  "plugins": [
    { "name": "formatter", "source": { "source": "github", "repo": "someone/formatter-plugin" } },
    { "name": "linter",    "source": { "source": "npm", "package": "@company/linter-plugin" } },
    { "name": "local-tool", "source": "./plugins/local-tool" }
  ]
}
```

- 相対パス（`./plugins/...`）のものだけは marketplaces/ 内にある
- GitHub や npm をソースにしているものはそこから別途取得が必要

cache/ に分離することで:

- どのソースから来ても統一的な場所に保管できる
- バージョン管理や更新時の差し替えがやりやすい

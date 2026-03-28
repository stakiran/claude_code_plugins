# プラグインのインストール方法とマーケットプレイス

## インストールの基本設計

npm install や git clone のような直接コマンドはない。**マーケットプレイス（プラグインカタログ）を経由する**のが基本。

```
マーケットプレイスを登録 → そこからプラグインをインストール
```

## マーケットプレイスとは

名前からグローバルな共有ストア（App Store 的なもの）を想像するが、実態は **「プラグインの取得元を自分で登録する仕組み」** 。apt のリポジトリ登録や npm の private registry 追加に近い。「プラグインレジストリ」や「プラグインソース」と呼ぶ方がしっくりくる。

- 公式（`claude-plugins-official`）→ 唯一グローバルなもの。最初から登録済み
- それ以外 → 自分やチームが Git リポジトリや URL を取得元として登録するだけ

## マーケットプレイスの登録方法

```bash
# GitHub リポジトリ
/plugin marketplace add owner/repo

# GitLab 等の Git リポジトリ
/plugin marketplace add https://gitlab.com/company/plugins.git

# ブランチ/タグ指定
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0

# ローカルディレクトリ
/plugin marketplace add ./my-marketplace

# リモート URL
/plugin marketplace add https://example.com/marketplace.json
```

## プラグインのインストール方法

### メイン：マーケットプレイス経由

```bash
/plugin install plugin-name@marketplace-name --scope user|project|local
```

または `/plugin` で対話的 UI からインストール。

### CLI（非対話）

```bash
claude plugin install <plugin>@<marketplace> [--scope user|project|local]
claude plugin uninstall <plugin>@<marketplace> [--keep-data]
claude plugin enable <plugin>@<marketplace>
claude plugin disable <plugin>@<marketplace>
claude plugin update <plugin>@<marketplace>
```

### 開発用：--plugin-dir（マーケットプレイス不要）

```bash
claude --plugin-dir ./my-plugin
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

キャッシュにコピーされず、ローカルのディレクトリを直接参照する。

### 手動：settings.json を直接編集

```json
{
  "enabledPlugins": {
    "code-formatter@company-tools": true
  }
}
```

マーケットプレイスが登録済みであれば動く。

## マーケットプレイス内のプラグインソース種別

marketplace.json に定義されるソースは複数種類ある:

| ソース種別 | 説明 |
|-----------|------|
| 相対パス | 同リポジトリ内 `"./plugins/my-plugin"` |
| GitHub | `"source": "github", "repo": "owner/repo"` |
| Git URL | `"source": "url", "url": "https://..."` |
| Git サブディレクトリ | `"source": "git-subdir"` モノレポ向け |
| npm | `"source": "npm", "package": "@acme/plugin"` |

npm は直接 `npm install` するのではなく、マーケットプレイスのソース定義として指定する形。

## チーム配布：extraKnownMarketplaces

`.claude/settings.json` に書いておくと、チームメンバーがリポジトリを開いたときにマーケットプレイスが認識される:

```json
{
  "extraKnownMarketplaces": {
    "my-team-tools": {
      "source": { "source": "github", "repo": "your-org/claude-plugins" }
    }
  },
  "enabledPlugins": {
    "code-formatter@my-team-tools": true
  }
}
```

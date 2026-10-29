# ac-claude-plugins

用途別のスキル一式を、バンドル 1 つでプロジェクトに入れるための Claude Code プラグインマーケットプレイスです。

## 使い方

対象プロジェクトのルートで実行します。

1. マーケットプレイスを登録する。

   ```bash
   claude plugin marketplace add https://github.com/andcreate/ac-claude-plugins.git --scope project
   ```

2. 用途に合うバンドルをインストールする。依存しているプラグインも自動で入ります。

   ```bash
   # Web フロントエンド
   claude plugin install frontend@ac-claude-plugins --scope project

   # WordPress
   claude plugin install wordpress@ac-claude-plugins --scope project
   ```

   1 つのプロジェクトに複数のバンドルを入れても構いません（例: ヘッドレス WordPress なら `wordpress` と `frontend`）。

### スコープ

| スコープ | 書き込み先 | 用途 |
| :-- | :-- | :-- |
| `project` | `.claude/settings.json` | リポジトリにコミットして、チームや別マシンと共有する |
| `local` | `.claude/settings.local.json` | 自分だけで使う（git 管理しない） |

`--scope` を省略すると `user`（全プロジェクト共通）になります。

### 更新・削除

```bash
# マーケットプレイスの内容を取り直す
claude plugin marketplace update ac-claude-plugins

# バンドルを更新する（追加された依存は /reload-plugins で入る）
claude plugin update frontend@ac-claude-plugins --scope project

# バンドルを外し、自動で入った依存も片付ける
claude plugin uninstall frontend@ac-claude-plugins --scope project --prune
```

## バンドル

各プラグインは上流のリポジトリを commit 固定で参照しています。中身はこのリポジトリには含まれません。

### `frontend`

Web フロントエンド開発用。

| プラグイン | 上流 | 入るスキル | ライセンス |
| :-- | :-- | :-- | :-- |
| `ui-ux-pro-max` | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | ui-ux-pro-max, design, design-system, ui-styling, brand, banner-design, slides | MIT |
| `taste-skill` | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | design-taste-frontend（v2） | MIT |
| `impeccable` | [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | impeccable | Apache-2.0 |
| `web-artifacts-builder` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/web-artifacts-builder) | web-artifacts-builder | Apache-2.0 |
| `transitions-dev` | [Jakubantalik/transitions.dev](https://github.com/Jakubantalik/transitions.dev) | transitions-dev, transitions-polish | Transitions.dev License（独自） |
| `apple-design` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/apple-design) | apple-design | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/emil-design-eng) | emil-design-eng | MIT |
| `web-quality-skills` | [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) | accessibility, best-practices, core-web-vitals, performance, seo, web-quality-audit | MIT |

UI ファイルを編集するたびに impeccable の検出器を自動で走らせたいプロジェクトでは、バンドルに加えて `impeccable-hooks` を入れます。hook はモデルの承認を経ずに動き、初回にエンジンのバイナリを GitHub Releases から取得します。

```bash
claude plugin install impeccable-hooks@ac-claude-plugins --scope project
```

### `wordpress`

WordPress 開発用（クラシックテーマ、ブロックテーマ、プラグイン開発、Docker での開発環境）。

| プラグイン | 上流 | 入るスキル | ライセンス |
| :-- | :-- | :-- | :-- |
| `wp-agent-skills` | [WordPress/agent-skills](https://github.com/WordPress/agent-skills) | wp-plugin-directory-guidelines を除く 18 スキル | GPL-2.0-or-later |
| `docker-skills` | [docker/skills](https://github.com/docker/skills) | docker-project-foundations, docker-compose-patterns, docker-build-strategies, docker-destructive-guardrails | Apache-2.0 |

## 参照したドキュメント

- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
- [Constrain plugin dependency versions](https://code.claude.com/docs/en/plugin-dependencies)
- [Extend Claude with skills](https://code.claude.com/docs/en/skills)

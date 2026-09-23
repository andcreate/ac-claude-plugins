# ac-claude-plugins

ロジの Claude Code プラグインマーケットプレイスです。用途ごとに **バンドル** を用意していて、プロジェクトには必要なバンドルを 1 つ入れるだけで、その用途のスキル一式が入ります。

```bash
# Web フロントエンドのプロジェクトで
claude plugin install frontend@ac-claude-plugins --scope project
```

- リポジトリ: https://github.com/andcreate/ac-claude-plugins
- ローカル: `D:\_Claude\marketplace\ac-claude-plugins`（親フォルダ `D:\_Claude\marketplace` は git 管理しない）

## 構成

```
ac-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json          # マーケットプレイス定義（全用途のプラグイン一覧）
├── plugins/
│   └── frontend/                 # バンドル。dependencies だけを持ち、スキルはない
│       └── .claude-plugin/plugin.json
├── archive/
│   └── plugins/                  # 退役したプラグイン。marketplace.json からは参照しない
└── README.md
```

マーケットプレイスは 1 つで、その中にプラグインが 2 種類あります。

- **バンドル**（`frontend` など）: `plugins/<用途名>/` に置く自作プラグイン。スキルは持たず、`plugin.json` の `dependencies` に入れたいプラグイン名を並べるだけ。
- **実体のプラグイン**: スキルを持つプラグイン。今はすべて他の人のリポジトリで、中身はこのリポジトリに置かず、`marketplace.json` のエントリから commit 固定で参照している。

バンドル同士は独立しています。`frontend` を入れたプロジェクトには、`frontend` の依存だけが入ります。複数のバンドルで同じプラグインを使いたいときは、それぞれの `dependencies` に同じ名前を書けば共有できます。

## バンドル一覧

### `frontend`

Web フロントエンド用。依存先は次のとおりです。

| プラグイン | 参照先 | ソース | 入るスキル・コマンド | ライセンス |
| :-- | :-- | :-- | :-- | :-- |
| `ui-ux-pro-max` | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | `url`（リポジトリ全体） | ui-ux-pro-max, design, design-system, ui-styling, brand, banner-design, slides | MIT |
| `taste-skill` | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | `url`（リポジトリ全体） | taste-skill, taste-skill-v1, gpt-tasteskill, brutalist-skill, minimalist-skill, soft-skill, redesign-skill, stitch-skill, image-to-code-skill, imagegen-frontend-web, imagegen-frontend-mobile, brandkit, output-skill | MIT |
| `frontend-design` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) `skills/frontend-design` | `git-subdir` | frontend-design | Apache-2.0 |
| `web-artifacts-builder` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/web-artifacts-builder) `skills/web-artifacts-builder` | `git-subdir` | web-artifacts-builder | Apache-2.0 |
| `transitions-dev` | [Jakubantalik/transitions.dev](https://github.com/Jakubantalik/transitions.dev) | `url`（リポジトリ全体） | transitions-dev, transitions-polish | 表記なし |
| `apple-design` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/apple-design) `skills/apple-design` | `git-subdir` | apple-design | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/emil-design-eng) `skills/emil-design-eng` | `git-subdir` | emil-design-eng | MIT |
| `web-quality-skills` | [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) | `url`（リポジトリ全体） | accessibility, best-practices, core-web-vitals, performance, seo, web-quality-audit | MIT |
| `design-review` | [Superfuture/design-review](https://github.com/Superfuture/design-review) `design-review` | `git-subdir` | design-review、コマンド activate | MIT（plugin.json の記載） |

## 仕組みと決めごと

### 依存関係

- バンドルの `dependencies` はプラグイン名だけで書く。名前は **同じマーケットプレイス（ac-claude-plugins）の中で** 解決される。
- 他の人のプラグインも、このマーケットプレイスの `marketplace.json` にエントリを置いている。依存解決がこの中で閉じるので、`allowCrossMarketplaceDependenciesOn` は要らない。

### 外部プラグインのソースの選び方

- 上流がリポジトリ直下をプラグインとして配布している（`.claude-plugin/plugin.json` がある）ものは、リポジトリ全体を `url` ソースで参照する。上流の `plugin.json` がそのまま使われる。
- スキルのフォルダ単体のもの（anthropics/skills、emilkowalski/skills）は、`git-subdir` でそのフォルダだけを取る。`plugin.json` がないフォルダは、直下の `SKILL.md` が 1 つのスキルとして読み込まれる。
- transitions.dev は `plugin.json` がないが、`transitions-polish` が `../transitions-dev` を参照するため、2 つを並べたまま取れるようリポジトリ全体を参照する（直下の `skills/` が自動で読み込まれる）。
- GitHub のリポジトリでも `github` ソースではなく HTTPS の `url` ソースを使う。`github` ソースは既定で SSH clone になり、SSH 鍵の設定に左右されるため。
- すべて `sha` で commit を固定する（現在の固定は 2026-09-23 時点の各リポジトリの main）。

### バージョン

- 自作プラグイン（バンドルを含む）の `version` は各 `plugin.json` にだけ書く。`marketplace.json` のエントリには書かない。両方に書くと `plugin.json` 側が黙って優先され、食い違いに気づけないため。
- 外部プラグインのエントリにも `version` は書かない。上流に `plugin.json` の `version` があればそれが、なければ commit sha がバージョンになる。
- `marketplace.json` のトップレベルの `version` は、マーケットプレイス定義そのもののバージョン。
- すべて `0.1.0` から始めている。

## 用途別のバンドルを追加する（例: `unity`）

1. `plugins/unity/.claude-plugin/plugin.json` を作る。`dependencies` に入れたいプラグイン名を並べる。

   ```json
   {
     "name": "unity",
     "version": "0.1.0",
     "description": "Unity 開発用のバンドル。自身はスキルを持たず、dependencies だけを宣言する",
     "author": { "name": "ロジ" },
     "dependencies": [
       "<plugin-name>",
       "<plugin-name>"
     ]
   }
   ```

2. `.claude-plugin/marketplace.json` の `plugins` にバンドルのエントリを足す。

   ```json
   {
     "name": "unity",
     "source": "./plugins/unity",
     "description": "【バンドル】Unity 開発用のスキル一式",
     "category": "bundle"
   }
   ```

3. `dependencies` に書いたプラグインのうち、まだ `marketplace.json` にないものを足す（「外部のプラグインを参照で追加する」「自作プラグインを追加する」）。
4. 検証してから、使い捨てのプロジェクトで導入を試す（「プロジェクトへ導入する」の手順）。

   ```bash
   claude plugin validate . --strict
   claude plugin validate plugins/unity --strict
   ```

5. この README の「バンドル一覧」に節を足してコミットする。

バンドル名は、このマーケットプレイスの中で一意な kebab-case にします。

## プラグインを追加する

### 外部のプラグインを参照で追加する

中身をコピーせず、`marketplace.json` のエントリで外部リポジトリを参照します。

1. 上流の構造を確かめる。`.claude-plugin/plugin.json` の場所、`SKILL.md` の場所、スキルがフォルダ外（`../` など）を参照していないか、ライセンス。
2. 構造に合うソースでエントリを書く。

   リポジトリ直下がプラグイン（`.claude-plugin/plugin.json` が直下にある）の場合:

   ```json
   {
     "name": "<plugin-name>",
     "source": {
       "source": "url",
       "url": "https://github.com/<owner>/<repo>.git",
       "ref": "main",
       "sha": "<40 桁の commit sha>"
     },
     "description": "説明"
   }
   ```

   サブフォルダがプラグイン、またはスキルのフォルダ単体の場合:

   ```json
   {
     "name": "<plugin-name>",
     "source": {
       "source": "git-subdir",
       "url": "https://github.com/<owner>/<repo>.git",
       "path": "skills/<skill-name>",
       "ref": "main",
       "sha": "<40 桁の commit sha>"
     },
     "description": "説明"
   }
   ```

3. `sha` は次のコマンドで調べる。

   ```bash
   git ls-remote https://github.com/<owner>/<repo>.git refs/heads/main
   ```

4. 使うバンドルの `plugins/<バンドル名>/.claude-plugin/plugin.json` の `dependencies` に名前を足す。
5. 検証してから、使い捨てのプロジェクトで導入を試す。

- relative path 以外のソースでは、インストール前に外部の `plugin.json` を読めない。一覧に説明が出るよう、`description` はエントリ側に書いておく。
- エントリの `name` は、上流の `plugin.json` の `name` とそろえる（食い違うとエントリ側の名前が使われる）。

### 自作プラグインを追加する

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` を作る。

   ```json
   {
     "name": "<plugin-name>",
     "version": "0.1.0",
     "description": "説明",
     "author": { "name": "ロジ" }
   }
   ```

2. `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` にスキルを置く。ディレクトリ名とフロントマターの `name` は同じ kebab-case にする。

   ```markdown
   ---
   name: <skill-name>
   description: 何をするスキルか。どの種類のプロジェクト・どの作業のときに使うか。使わない場面も書く。
   ---

   # 見出し

   本文
   ```

   `description` は具体的に書く。Claude はこの文を見て発火を判断する。対象のプロジェクト種別（例: `package.json` に `next` がある）と作業内容を最初に書き、対象外の場面も書いておくと誤発火が減る。さらに絞りたい場合は、フロントマターに `paths`（glob）を足すと、該当ファイルを扱うときだけ自動で読み込まれる。

3. `.claude-plugin/marketplace.json` の `plugins` にエントリを足す。`version` はここには書かない。

   ```json
   {
     "name": "<plugin-name>",
     "source": "./plugins/<plugin-name>",
     "description": "説明"
   }
   ```

4. 使うバンドルの `dependencies` に名前を足す。
5. 検証してコミットする。スキルを直したら、そのプラグインの `version` を上げる（新機能なら MINOR、修正なら PATCH）。

   ```bash
   claude plugin validate . --strict
   claude plugin validate plugins/<plugin-name> --strict
   claude plugin validate plugins/<plugin-name>/skills --strict
   ```

### 外部プラグインを新しい commit に更新する

1. `git ls-remote <リポジトリ URL> refs/heads/main` で最新 sha を調べる。
2. 上流の変更内容を確認する（GitHub の compare 画面で、固定中の sha と最新 sha の差分を見る）。
3. `marketplace.json` の該当エントリの `sha` を書き換える。
4. `claude plugin validate . --strict` のあとコミットして push する。
5. 導入済みのプロジェクトでは `claude plugin marketplace update ac-claude-plugins` を実行し、`/reload-plugins` で反映する。

## プロジェクトへ導入する

### 1. マーケットプレイスを登録する

対象プロジェクトのルートで、どちらかを実行します。

GitHub から登録する（別マシンでも使える。`.claude/settings.json` をコミットしても他の環境で解決できる）:

```bash
claude plugin marketplace add https://github.com/andcreate/ac-claude-plugins.git --scope project
```

ローカルフォルダから登録する（この PC だけ。自作スキルの編集がすぐ反映される）:

```bash
claude plugin marketplace add D:/_Claude/marketplace/ac-claude-plugins --scope project
```

- `--scope project` にすると、マーケットプレイスの宣言がプロジェクトの `.claude/settings.json` に書かれる。
- 登録状態そのもの（`~/.claude/plugins/known_marketplaces.json`）はユーザー単位で 1 か所に保存される。これはプラグインのインストールではない。
- ローカルフォルダから登録した場合、relative path のプラグイン（`./plugins/...`）はこのフォルダから直接読み込まれる。編集は次のセッション開始か `/reload-plugins` で反映される。
- 外部参照のプラグインは git から取得され、`~/.claude/plugins/cache` にキャッシュされる。

### 2. バンドルをインストールする

```bash
claude plugin install frontend@ac-claude-plugins --scope project
```

バンドルをインストールすると、`dependencies` に書いたプラグインも同じスコープで自動で入ります。個別に入れたい場合は `design-review@ac-claude-plugins` のように名前を指定します。

### `--scope project` と `--scope local` の違い

| スコープ | 書き込み先 | git 管理 | 向いている場面 |
| :-- | :-- | :-- | :-- |
| `project` | `.claude/settings.json` | コミットして共有する | チーム全員（や別マシンの自分）で同じプラグインを使う |
| `local` | `.claude/settings.local.json` | 共有しない（gitignore 扱い） | 自分だけ試したい、チームに押し付けたくない |
| `user`（既定） | `~/.claude/settings.json` | - | 全プロジェクト共通。このマーケットプレイスでは使わない |

`--scope` を省略すると `user` になります。グローバルに入れないため、必ず `project` か `local` を指定してください。

### 更新・削除

```bash
# マーケットプレイスの内容を取り直す
claude plugin marketplace update ac-claude-plugins

# バンドルを更新する（追加された依存は /reload-plugins で入る）
claude plugin update frontend@ac-claude-plugins --scope project

# バンドルを外し、自動で入った依存も片付ける
claude plugin uninstall frontend@ac-claude-plugins --scope project --prune
```

## リモートについて

- リポジトリはプライベートでもよい。`marketplace add` / `install` / `update` は手元の git 認証（credential helper や SSH 鍵）をそのまま使う。
- GitHub のプライベートリポジトリなら、`gh auth login` と `gh auth setup-git` を済ませておくと、バックグラウンドの自動更新も認証できる。確認は `git ls-remote <リポジトリ URL>` がパスワードを聞かずに通るかで行う。
- `owner/repo` 形式で登録すると既定では SSH で clone される。HTTPS を使いたい場合は環境変数 `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` を設定する。

## 退役したもの

- `archive/plugins/` には、最初に雛形として作った `web-frontend`（Next.js 向け）と `headless-wp`（Headless WordPress 向け）を置いています。戻すときは `plugins/` に移し、`marketplace.json` にエントリを、バンドルの `dependencies` に名前を足します。
- 外部参照から外したもの（エントリは git 履歴に残っています）:

  | プラグイン | 参照先 | 外した理由 |
  | :-- | :-- | :-- |
  | `brand-guidelines` | anthropics/skills `skills/brand-guidelines` | Anthropic 自身のブランド（配色・Poppins / Lora）を適用するスキルで、自分の案件には使わないため。ブランドの話題で誤発火するおそれもある |
  | `frontend-design`（旧参照先） | anthropics/claude-code `plugins/frontend-design` | anthropics/skills 版に差し替えたため |

- 名前を変えたもの:

  | 旧 | 新 | 理由 |
  | :-- | :-- | :-- |
  | マーケットプレイス `ac-frontend` | `ac-claude-plugins` | 用途ごとにマーケットプレイスを分けず、1 つの中にバンドルを並べる方針にしたため |
  | バンドル `web-bundle` | `frontend` | 同上 |

## 参照したドキュメント

- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
- [Constrain plugin dependency versions](https://code.claude.com/docs/en/plugin-dependencies)
- [Extend Claude with skills](https://code.claude.com/docs/en/skills)

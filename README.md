# ac-frontend

Web フロントエンド開発向けの Claude Code プラグインマーケットプレイスです。

プロジェクトごとに次の 1 コマンドで、Web 系の定番スキル一式を入れられます。

```bash
claude plugin install web-bundle@ac-frontend --scope project
```

このリポジトリは単体で完結しています。親フォルダ `D:\_Claude\marketplace` は用途別マーケットプレイスを並べる置き場で、git 管理はしていません。

## 構成

```
ac-frontend/
├── .claude-plugin/
│   └── marketplace.json          # マーケットプレイス定義（プラグイン一覧）
├── plugins/
│   └── web-bundle/               # バンドル。dependencies だけを持ち、スキルはない
│       └── .claude-plugin/plugin.json
├── archive/
│   └── plugins/                  # 退役したプラグイン。marketplace.json からは参照しない
└── README.md
```

`web-bundle` は下表の外部プラグインへの依存だけを宣言しています。外部プラグインの中身はこのリポジトリに置かず、`marketplace.json` のエントリから commit 固定で参照します。

| プラグイン | 参照先 | ソース | 入るスキル・コマンド | ライセンス |
| :-- | :-- | :-- | :-- | :-- |
| `ui-ux-pro-max` | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | `url`（リポジトリ全体） | ui-ux-pro-max, design, design-system, ui-styling, brand, banner-design, slides | MIT |
| `taste-skill` | [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | `url`（リポジトリ全体） | taste-skill, taste-skill-v1, gpt-tasteskill, brutalist-skill, minimalist-skill, soft-skill, redesign-skill, stitch-skill, image-to-code-skill, imagegen-frontend-web, imagegen-frontend-mobile, brandkit, output-skill | MIT |
| `frontend-design` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) `skills/frontend-design` | `git-subdir` | frontend-design | Apache-2.0 |
| `web-artifacts-builder` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/web-artifacts-builder) `skills/web-artifacts-builder` | `git-subdir` | web-artifacts-builder | Apache-2.0 |
| `brand-guidelines` | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/brand-guidelines) `skills/brand-guidelines` | `git-subdir` | brand-guidelines | Apache-2.0 |
| `transitions-dev` | [Jakubantalik/transitions.dev](https://github.com/Jakubantalik/transitions.dev) | `url`（リポジトリ全体） | transitions-dev, transitions-polish | 表記なし |
| `apple-design` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/apple-design) `skills/apple-design` | `git-subdir` | apple-design | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills/emil-design-eng) `skills/emil-design-eng` | `git-subdir` | emil-design-eng | MIT |
| `web-quality-skills` | [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) | `url`（リポジトリ全体） | accessibility, best-practices, core-web-vitals, performance, seo, web-quality-audit | MIT |
| `design-review` | [Superfuture/design-review](https://github.com/Superfuture/design-review) `design-review` | `git-subdir` | design-review、コマンド activate | MIT（plugin.json の記載） |

ソースの選び方:

- 上流がリポジトリ直下をプラグインとして配布している（`.claude-plugin/plugin.json` がある）ものは、リポジトリ全体を `url` ソースで参照する。上流の `plugin.json` がそのまま使われる。
- スキルのフォルダ単体のもの（anthropics/skills、emilkowalski/skills）は、`git-subdir` でそのフォルダだけを取る。`plugin.json` がないフォルダは、直下の `SKILL.md` が 1 つのスキルとして読み込まれる。
- transitions.dev は `plugin.json` がないが、`transitions-polish` が `../transitions-dev` を参照するため、2 つを並べたまま取れるようリポジトリ全体を参照する（直下の `skills/` が自動で読み込まれる）。
- GitHub のリポジトリでも `github` ソースではなく HTTPS の `url` ソースを使っている。`github` ソースは既定で SSH clone になり、SSH 鍵の設定に左右されるため。

`archive/plugins/` には、最初に雛形として作った `web-frontend`（Next.js 向け）と `headless-wp`（Headless WordPress 向け）を退役させて置いています。戻すときは `plugins/` に移し、`marketplace.json` にエントリを、`web-bundle` の `dependencies` に名前を足します。

### 依存関係の仕組み

- `web-bundle` の `plugin.json` にある `dependencies` は、プラグイン名だけで書いています。名前は **同じマーケットプレイス（ac-frontend）の中で** 解決されます。
- 依存先はすべて外部リポジトリのプラグインですが、ac-frontend の `marketplace.json` にエントリを置いています。これで依存解決が ac-frontend の中で閉じるので、`allowCrossMarketplaceDependenciesOn` は要りません。
- 外部プラグインはすべて `sha` で commit を固定しています（2026-09-23 時点の各リポジトリの main）。更新手順は後述します。
- 外部プラグインのエントリには `version` を書きません。上流に `plugin.json` の `version` があればそれが、なければ commit sha がバージョンになります。

### バージョンの置き場所

- 自作プラグインの `version` は各 `plugin.json` にだけ書きます。`marketplace.json` のエントリには書きません。両方に書くと `plugin.json` 側が黙って優先され、食い違いに気づけないためです。
- `marketplace.json` のトップレベルの `version` は、マーケットプレイス定義そのもののバージョンです。
- すべて `0.1.0` から始めています。

## スキルを追加する

自作プラグイン（`plugins/<plugin-name>`）にスキルを足す場合です。自作プラグインがまだなければ、先に「自作プラグインを追加する」を行います。

1. `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` を作る。ディレクトリ名とフロントマターの `name` は同じ kebab-case にする。

   ```markdown
   ---
   name: <skill-name>
   description: 何をするスキルか。どの種類のプロジェクト・どの作業のときに使うか。使わない場面も書く。
   ---

   # 見出し

   本文
   ```

2. `description` は具体的に書く。Claude はこの文を見て発火を判断する。対象のプロジェクト種別（例: `package.json` に `next` がある）と作業内容を最初に書き、対象外の場面も書いておくと誤発火が減る。
   さらに絞りたい場合は、フロントマターに `paths`（glob）を足すと、該当ファイルを扱うときだけ自動で読み込まれる。
3. 検証する。

   ```bash
   claude plugin validate plugins/<plugin-name> --strict
   claude plugin validate plugins/<plugin-name>/skills --strict
   ```

4. `plugins/<plugin-name>/.claude-plugin/plugin.json` の `version` を上げる（新機能なら MINOR、修正なら PATCH）。
5. コミットする。

## プラグインを追加する

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

2. `plugins/<plugin-name>/skills/...` にスキルを置く。
3. `.claude-plugin/marketplace.json` の `plugins` にエントリを足す。`version` はここには書かない。

   ```json
   {
     "name": "<plugin-name>",
     "source": "./plugins/<plugin-name>",
     "description": "説明",
     "category": "frontend"
   }
   ```

4. バンドルに含めるなら、`plugins/web-bundle/.claude-plugin/plugin.json` の `dependencies` に名前を足し、`web-bundle` の `version` を上げる。
5. 検証してコミットする。

   ```bash
   claude plugin validate . --strict
   claude plugin validate plugins/<plugin-name> --strict
   ```

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

4. `plugins/web-bundle/.claude-plugin/plugin.json` の `dependencies` に名前を足す。
5. 検証してから、使い捨てのプロジェクトで導入を試す（下の「プロジェクトへ導入する」の手順）。

- relative path 以外のソースでは、インストール前に外部の `plugin.json` を読めない。一覧に説明が出るよう、`description` はエントリ側に書いておく。
- エントリの `name` は、上流の `plugin.json` の `name` とそろえる（食い違うとエントリ側の名前が使われる）。

### 外部プラグインを新しい commit に更新する

1. `git ls-remote <リポジトリ URL> refs/heads/main` で最新 sha を調べる。
2. 上流の変更内容を確認する（GitHub の compare 画面で、固定中の sha と最新 sha の差分を見る）。
3. `marketplace.json` の該当エントリの `sha` を書き換える。
4. `claude plugin validate . --strict` のあとコミットする。
5. 導入済みのプロジェクトでは `claude plugin marketplace update ac-frontend` を実行し、`/reload-plugins` で反映する。

## プロジェクトへ導入する

### 1. マーケットプレイスを登録する

対象プロジェクトのルートで、どちらかを実行します。

GitHub から登録する（別マシンでも使える。`.claude/settings.json` をコミットしても他の環境で解決できる）:

```bash
claude plugin marketplace add https://github.com/andcreate/ac-frontend-claude-plugins.git --scope project
```

ローカルフォルダから登録する（この PC だけ。スキルの編集がすぐ反映される）:

```bash
claude plugin marketplace add D:/_Claude/marketplace/ac-frontend --scope project
```

リポジトリ名は `ac-frontend-claude-plugins` ですが、マーケットプレイス名は `marketplace.json` の `name` で決まるため、どちらでも `ac-frontend` になります。

- `--scope project` にすると、マーケットプレイスの宣言がプロジェクトの `.claude/settings.json` に書かれる。
- 登録状態そのもの（`~/.claude/plugins/known_marketplaces.json`）はユーザー単位で 1 か所に保存される。これはプラグインのインストールではない。
- ローカルディレクトリから登録した場合、relative path のプラグイン（`./plugins/...`）はこのフォルダから直接読み込まれる。スキルを編集すると、次のセッション開始か `/reload-plugins` で反映される。
- 外部参照のプラグインは git から取得され、`~/.claude/plugins/cache` にキャッシュされる。
- リモートに push したあとは、ローカルパスの代わりに `<owner>/ac-frontend` や git URL を指定できる。別マシンや他人と使うならこちらにする（`D:/...` の絶対パスは他の環境では解決できない）。

### リモートに置く場合（パブリックでなくてよい）

- 自分 1 台だけで使うなら、リモートは不要。上のローカルパス登録で完結する。
- 複数マシンで使うなら、プライベートリポジトリでよい。`marketplace add` / `install` / `update` は手元の git 認証（credential helper や SSH 鍵）をそのまま使う。
- GitHub のプライベートリポジトリなら、`gh auth login` と `gh auth setup-git` を済ませておくと、バックグラウンドの自動更新も認証できる。確認は `git ls-remote <リポジトリ URL>` がパスワードを聞かずに通るかで行う。
- `owner/repo` 形式で登録すると既定では SSH で clone される。HTTPS を使いたい場合は環境変数 `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` を設定する。

### 2. バンドルをインストールする

```bash
claude plugin install web-bundle@ac-frontend --scope project
```

`web-bundle` をインストールすると、`dependencies` に書いたプラグインも同じスコープで自動で入ります。個別に入れたい場合は `frontend-design@ac-frontend` のように名前を指定します。

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
claude plugin marketplace update ac-frontend

# バンドルを更新する（追加された依存は /reload-plugins で入る）
claude plugin update web-bundle@ac-frontend --scope project

# バンドルを外し、自動で入った依存も片付ける
claude plugin uninstall web-bundle@ac-frontend --scope project --prune
```

## 別用途のマーケットプレイスを作る（例: ac-unity）

ac-frontend と同じ形で、親フォルダに独立したリポジトリとして作ります。

1. フォルダを作る。

   ```
   D:\_Claude\marketplace\ac-unity\
   ├── .claude-plugin\marketplace.json
   ├── plugins\
   │   ├── unity-bundle\        ← dependencies だけのバンドル
   │   └── unity-csharp\        ← 自作スキル置き場
   └── README.md
   ```

2. `marketplace.json` を書く。`name` はマーケットプレイスごとに一意な kebab-case にする（同じ名前を登録すると前のものが置き換わる）。`claude-plugins-official` などの予約名や、`github` / `npm` などの名前は使えない。

   ```json
   {
     "name": "ac-unity",
     "owner": { "name": "ロジ" },
     "description": "Unity 開発向けのプラグイン集",
     "version": "0.1.0",
     "plugins": [
       { "name": "unity-bundle", "source": "./plugins/unity-bundle", "description": "Unity 用の一式" },
       { "name": "unity-csharp", "source": "./plugins/unity-csharp", "description": "Unity C# の規約" }
     ]
   }
   ```

3. 各プラグインの `plugin.json` と `skills/` を作る。バンドルの `dependencies` には **同じマーケットプレイス内のプラグイン名** だけを書く。外部プラグインを入れたいときは、ac-frontend と同じように自分の `marketplace.json` に git ソースのエントリを置く。
   ac-frontend のプラグインに依存させたい場合だけ、ac-unity の `marketplace.json` に `"allowCrossMarketplaceDependenciesOn": ["ac-frontend"]` を書き、依存を `{ "name": "frontend-design", "marketplace": "ac-frontend" }` の形で書く。
4. 検証する。

   ```bash
   claude plugin validate . --strict
   claude plugin validate plugins/unity-bundle --strict
   claude plugin validate plugins/unity-csharp --strict
   ```

5. `ac-unity` フォルダで `git init` して初回コミットする（親フォルダでは `git init` しない）。
6. Unity プロジェクトで導入する。

   ```bash
   claude plugin marketplace add D:/_Claude/marketplace/ac-unity --scope project
   claude plugin install unity-bundle@ac-unity --scope project
   ```

## 参照したドキュメント

- [Plugins reference](https://code.claude.com/docs/en/plugins-reference)
- [Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
- [Constrain plugin dependency versions](https://code.claude.com/docs/en/plugin-dependencies)
- [Extend Claude with skills](https://code.claude.com/docs/en/skills)

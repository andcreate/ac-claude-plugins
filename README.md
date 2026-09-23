# ac-frontend

Web フロントエンド開発（Next.js / TypeScript / Headless WordPress）向けの Claude Code プラグインマーケットプレイスです。

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
│   ├── web-bundle/               # バンドル。dependencies だけを持ち、スキルはない
│   │   └── .claude-plugin/plugin.json
│   ├── web-frontend/             # 自作スキル置き場（Next.js / TypeScript 向け）
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/nextjs-conventions/SKILL.md
│   └── headless-wp/              # 自作スキル置き場（Headless WordPress 向け）
│       ├── .claude-plugin/plugin.json
│       └── skills/wp-rest-graphql/SKILL.md
└── README.md
```

| プラグイン | ソース | 中身 |
| :-- | :-- | :-- |
| `web-bundle` | `./plugins/web-bundle` | `web-frontend` / `headless-wp` / `frontend-design` への依存だけを宣言 |
| `web-frontend` | `./plugins/web-frontend` | スキル `nextjs-conventions`（雛形） |
| `headless-wp` | `./plugins/headless-wp` | スキル `wp-rest-graphql`（雛形） |
| `frontend-design` | `git-subdir`（anthropics/claude-code の `plugins/frontend-design`） | Anthropic 公式のスキル。中身はこのリポジトリに置かない |

### 依存関係の仕組み

- `web-bundle` の `plugin.json` にある `dependencies` は、プラグイン名だけで書いています。名前は **同じマーケットプレイス（ac-frontend）の中で** 解決されます。
- `frontend-design` は外部リポジトリのプラグインですが、ac-frontend の `marketplace.json` にエントリを置いています。これで依存解決が ac-frontend の中で閉じるので、`allowCrossMarketplaceDependenciesOn` は要りません。
- `frontend-design` は `sha` で commit を固定しています。取り込み時点は `56f36532530f88b572854538d685fcf781141e8c`（anthropics/claude-code の main、2026-09-22）です。更新手順は後述します。

### バージョンの置き場所

- 自作プラグインの `version` は各 `plugin.json` にだけ書きます。`marketplace.json` のエントリには書きません。両方に書くと `plugin.json` 側が黙って優先され、食い違いに気づけないためです。
- `marketplace.json` のトップレベルの `version` は、マーケットプレイス定義そのもののバージョンです。
- すべて `0.1.0` から始めています。

## スキルを追加する

既存プラグイン（例: `web-frontend`）にスキルを足す場合です。

1. `plugins/web-frontend/skills/<skill-name>/SKILL.md` を作る。ディレクトリ名とフロントマターの `name` は同じ kebab-case にする。

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
   claude plugin validate plugins/web-frontend --strict
   ```

4. `plugins/web-frontend/.claude-plugin/plugin.json` の `version` を上げる（新機能なら MINOR、修正なら PATCH）。
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

中身をコピーせず、`marketplace.json` のエントリで外部リポジトリを参照します。`frontend-design` と同じ形です。

```json
{
  "name": "<plugin-name>",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/<owner>/<repo>.git",
    "path": "plugins/<plugin-name>",
    "ref": "main",
    "sha": "<40 桁の commit sha>"
  },
  "description": "説明"
}
```

- リポジトリ直下がプラグインなら `"source": "github", "repo": "<owner>/<repo>"` を使う。
- `sha` は次のコマンドで調べる。

  ```bash
  git ls-remote https://github.com/<owner>/<repo>.git refs/heads/main
  ```

- relative path 以外のソースでは、インストール前に外部の `plugin.json` を読めない。一覧に説明が出るよう、`description` はエントリ側に書いておく。

### `frontend-design` を新しい commit に更新する

1. `git ls-remote https://github.com/anthropics/claude-code.git refs/heads/main` で最新 sha を調べる。
2. 上流の変更内容を確認する（`plugins/frontend-design` の差分）。
3. `marketplace.json` の `frontend-design` エントリの `sha` を書き換え、README の取り込み時点の記載も直す。
4. `claude plugin validate . --strict` のあとコミットする。
5. 導入済みのプロジェクトでは `claude plugin marketplace update ac-frontend` を実行し、`/reload-plugins` で反映する。

## プロジェクトへ導入する

### 1. マーケットプレイスを登録する

対象プロジェクトのルートで実行します。

```bash
claude plugin marketplace add D:/_Claude/marketplace/ac-frontend --scope project
```

- `--scope project` にすると、マーケットプレイスの宣言がプロジェクトの `.claude/settings.json` に書かれる。
- 登録状態そのもの（`~/.claude/plugins/known_marketplaces.json`）はユーザー単位で 1 か所に保存される。これはプラグインのインストールではない。
- ローカルディレクトリから登録した場合、relative path のプラグイン（`web-bundle` / `web-frontend` / `headless-wp`）はこのフォルダから直接読み込まれる。スキルを編集すると、次のセッション開始か `/reload-plugins` で反映される。
- `frontend-design` は git から取得され、`~/.claude/plugins/cache` にキャッシュされる。
- GitHub などに push したあとは、ローカルパスの代わりに `<owner>/ac-frontend` や git URL を指定できる。チームで共有するならこちらにする（`D:/...` の絶対パスは他人の環境では解決できない）。

### 2. バンドルをインストールする

```bash
claude plugin install web-bundle@ac-frontend --scope project
```

`web-bundle` をインストールすると、`dependencies` に書いた `web-frontend` / `headless-wp` / `frontend-design` も自動で入ります。個別に入れたい場合は `web-frontend@ac-frontend` のように名前を指定します。

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

3. 各プラグインの `plugin.json` と `skills/` を作る。バンドルの `dependencies` には **同じマーケットプレイス内のプラグイン名** だけを書く。外部プラグインを入れたいときは、`frontend-design` と同じように自分の `marketplace.json` に git ソースのエントリを置く。
   ac-frontend のプラグインに依存させたい場合だけ、ac-unity の `marketplace.json` に `"allowCrossMarketplaceDependenciesOn": ["ac-frontend"]` を書き、依存を `{ "name": "web-frontend", "marketplace": "ac-frontend" }` の形で書く。
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

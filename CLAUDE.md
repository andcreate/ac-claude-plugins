# ac-claude-plugins の規律

- パブリックリポジトリ。owner / author は `andcreate`。説明文に個人名や持ち主の表現を入れない。
- 外部エントリは必ず 40 桁の `sha` で固定する。バージョンは plugin.json にだけ書く。
- 上流の一部のスキルだけ使うときは、git-subdir の `path` を `skills` にし、`"strict": false` と `skills` の列挙で読み込む。
- README は使い方だけ。経緯やメモは書かない。
- push はユーザーが行う。

## 上流の更新（Renovate の PR を受けたとき）

Renovate は `sha` を上げるだけなので、PR のブランチで次を行ってからマージする。

1. 差分を読む: `gh api repos/<owner>/<repo>/compare/<旧sha>...<新sha>` で、エントリの `path` 配下の変更を確認する。`scripts/`・`hooks/`・`.claude-plugin/` の追加や変更と、`${CLAUDE_PLUGIN_ROOT}` を参照する箇所は中身を読み、外部への通信やファイル操作を把握する。怪しければマージしない。
2. 同じリポジトリを参照するエントリ（anthropics/skills、emilkowalski/skills など）は同じ `sha` にそろえる。
3. 上流の構成に合わせて、`skills` の列挙・`description`・README の表を直す。
4. `claude plugin validate . --strict` と `claude plugin validate plugins/<bundle> --strict` を通す。
5. 使い捨てプロジェクトで `claude plugin marketplace add <このリポジトリのパス> --scope project` → `claude plugin install <bundle>@ac-claude-plugins --scope project` → `claude -p` でスキル一覧を出し、期待どおりのスキルだけが入るか確かめる。終わったら `uninstall --prune` と `marketplace remove` で片付ける。

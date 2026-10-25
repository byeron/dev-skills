---
name: git-rule
description: このリポジトリでの git の使い方を定めるスキル。base ブランチの名前は AGENTS.md の `git のルール` が定め、ブランチ名は `<type>/<NNNN>-<slug>`（issue の type と番号を踏む）、コミット粒度は原則 1 コミット = 1 issue（例外条件はスキル本文に列挙する）、コミットは agent が自律的に行いその妥当性はユーザーが判断する、merge と PR のクローズはユーザーのみが行い agent は PR 作成と push まで、という前提のもと、ブランチの命名規則、コミットメッセージの規約、コミットの粒度、push・PR 作成・マージ・履歴の扱い、禁止またはユーザー確認を要する操作を規定する。git でリポジトリの状態を変える操作（ブランチの作成・切り替え、add、commit、push、PR の作成、merge、rebase、reset、stash、ブランチ削除、タグ付けなど）は、必ずこのスキルを経由して行うこと。ユーザーが「コミットして」「ブランチを切って」「push して」「PR を出して」「マージして」などと言ったとき、実装を始めるとき、実装手順にコミットがあるとき、他のスキルで作業ツリーに反映した変更をコミットするときに使う。状態を変えない参照系のコマンド（status、diff、log、show など）は、このスキルを経由せずに使ってよい。差分レビューの観点と、レビュー後の CHANGES.md 記録から PR 作成または push へ進む順序はこのスキルでは扱わない。
---

# git-rule

このリポジトリでの git の使い方を定めるスキル。

**git でリポジトリの状態を変える操作は、すべてこのスキルの規則に従って行う。**

> issue の管理（`issues/` の構造、ファイル名、書式、採番）は `issue-rule` と `create-issue` に従う。差分レビューの観点と周回、レビュー後に指摘の修正から `CHANGES.md` の記録を経て PR 作成または push へ進む順序は、AGENTS.md の `レビューのルール` と `review-diff-code` が持つ。このスキルは git の操作そのものだけを定義し、それらは参照しない。

## 担当範囲

| 担当 | 内容 |
| --- | --- |
| 担当する | base ブランチの確認と使い方、操作の区切り、変更系操作の前に確認すること、ブランチの命名と作成、コミットの形式・粒度・手順、push の条件、PR の形式、マージと履歴の扱い、禁止と要確認の操作 |
| 担当しない | 差分レビューの観点と周回、`CHANGES.md` の記載内容と文面、issue の起票手順とファイル名、issue の type / status の運用、レビュー後の工程の順序、コミットが適切かどうかの最終判断 |

- 「base ブランチ」は `review-diff-code` など他のスキルでも使う語である。名前の指定元は AGENTS.md の `git のルール` であり、このスキルが base ブランチ名を定めることはない。このスキルが持つのは確認手順と使い方である。

## 操作の区切り

| 操作 | 実行してよいのは |
| --- | --- |
| `status` / `diff` / `log` / `show` / `grep` / `branch --list` など参照系 | agent。このスキルを経ないで使ってよい |
| `switch -c`、`add`、`commit`、作業ブランチへの `push`、PR の作成 | agent（このスキルの条件を満たしたとき） |
| PR の merge、PR の close | ユーザーのみ。agent は指示されても行わない |
| `done` / `close` / `discarded` への status 更新を伴うコミット | ユーザーの承認を得た後の agent |
| base ブランチへの commit・push | 原則 agent は行わない |
| タグの作成・push | ユーザーの指示を得た後の agent |
| 履歴の書き換え、変更の破棄 | ユーザーの確認を得た後の agent |

## 基本方針

- **参照系は自由、変更系はこのスキル経由**: ブランチ・インデックス・履歴・リモートを変える操作は、このスキルの規則に従う。
- **issue が起点**: 1 issue = 1 ブランチ = 1 PR。git の変更系操作の前に、今回の作業に対応する issue を `issues/` から特定する。特定できない場合は `issue-rule` と `create-issue` に従って issue を起票してからでなければ、変更系操作に入らない。
- **コミットは自律、妥当性の判断はユーザー**: agent はこのスキルの規則に従って自律的にコミットする。そのコミットが適切かどうかを判断するのはユーザーであり、agent はユーザーの判断に従って取り消し・修正する。
- **履歴を壊さない**: 破壊的な操作は行わない。必要な場合は理由を示してユーザーの確認を得る。
- **安全装置を外さない**: フックや検証を迂回しない。フックが失敗した場合は、迂回せず原因を直す。
- **秘密情報を commit しない**: `.env`、鍵、トークン、認証情報。

## base ブランチ

- base ブランチの名前は AGENTS.md の `git のルール` が定める。ブランチを切る元、PR の差し先、`review-diff-code` が差分を取る対象は base ブランチである。
- 始める前に、remote の既定ブランチが AGENTS.md の指定と一致していることを確認する。

```bash
git symbolic-ref --short refs/remotes/origin/HEAD        # 例: origin/main
git remote show origin | sed -n 's/.*HEAD branch: //p'   # 例: main
```

- 上の確認が AGENTS.md の指定と違うブランチを返す場合は操作しない。リポジトリの設定をユーザーに確認する。

## 変更系の操作の前に毎回確認する

```bash
git branch --show-current   # 現在のブランチ
git status --short          # 作業ツリーの状態
```

- 今回の作業に対応する issue を `issues/` から特定する。特定できないときは `issue-rule` と `create-issue` に従って起票してからでなければ、変更系の操作に入らない。
- 現在のブランチが想定と違う場合は、操作しない。
- 今回の作業と無関係な変更が作業ツリーにある場合は、巻き込まない。勝手に stash したり破棄したりせず、ユーザーに確認する。

## ブランチの命名規則

### 形式

```
<type>/<NNNN>-<slug>
```

- `<NNNN>-<slug>`: issue ファイル名 `issues/NNNN-<slug>.md` から `issues/` と拡張子を除いた部分。`issue-rule` の命名規則から機械的に導出する。
- `<type>`: issue のメタデータテーブルの `type`（`list` / `refinement` / `task`）。
- 例: `issues/0007-add-git-rule-skill.md`（type: `task`）→ ブランチ名 `task/0007-add-git-rule-skill`
- issue ファイルにブランチ名の項目は持たせない。`issue-rule` のメタデータテーブルに項目を追加しない。ブランチ名の正しい値は issue ファイル名と `type` からの導出値だけである。
- 既存ブランチが導出値と食い違っている場合は、新しいブランチを作らずユーザーに報告する。
- 一度作ったブランチ名は変更しない。
- issue に対応しないブランチ（`misc-...` など）は作らない。issue がなければ起票する。

### 作成の手順

1. 実装を伴う `task` については、ユーザーの実装承認が出て status が `doing` になっていることを確認する。承認が出ていない issue の実装は始めない。status の更新がまだなら、ブランチを作ってから更新する。
2. base ブランチを最新にする。
3. 作業ツリーがきれいなことを確認する。無関係な変更があれば先にユーザーへ報告する。
4. base ブランチから新しいブランチを作成して切り替える。

```bash
git fetch origin
git switch main                            # base ブランチ（名前は AGENTS.md の指定）
git merge --ff-only origin/main
git switch -c task/0007-add-git-rule-skill
```

- ブランチは常に base ブランチから切る。他の作業ブランチから切らない。
- `list` / `refinement` の調査も同じ手順でブランチを作る。status は `open` のまま進める（`doing` は `task` の実装に限定する）。
- 承認済みの issue のステータス更新は、ブランチ作成後の 1 件目のコミットにする。

## コミットの規約

### 形式

```
{接頭辞}: {概要} (#{NNNN})

{本文（任意）}
```

- `{NNNN}`: 対応する issue 番号（4 桁）。複数の issue にまたがる変更は `(#0003, #0004)` と列挙する。
- 接頭辞はコミットの種別であり、issue の `type`（`list` / `refinement` / `task`）とは別の語である。混同しない。
- 1 行目（件名）は日本語で、全角 40 文字程度まで、末尾に句点を付けない。
- 概要は「何をしたか」を「〜を追加」「〜を修正」のような体言止めまたは動詞の形で書く。
- 本文は「なぜ」変更したか、注意点、設計上の判断を書く。件名で足りる場合は省く。
- 破壊的変更は本文に `破壊的変更: {内容と移行方法}` と書く。

### 接頭辞

| 接頭辞 | 用途 |
| --- | --- |
| `feat` | 新しい機能・挙動・ファイルの追加 |
| `fix` | 不具合の修正 |
| `refactor` | 挙動を変えない内部の整理 |
| `test` | テストのみの追加・修正 |
| `docs` | ドキュメント、issue ファイル、`CHANGES.md` の変更 |
| `chore` | ビルド、CI、依存関係、設定などの雑務 |

### 定型メッセージ

issue の status のうち `done` / `close` / `discarded` を含む更新は、ユーザーの承認を得た後だけ行う。

| 場面 | メッセージ |
| --- | --- |
| issue の起票 | `docs: issue を起票 (#0007)` |
| status を `ready` / `doing` / `needs-info` / `user-blocked` へ更新 | `docs: issue の status を doing に更新 (#0007)` |
| 分割した issue への参照追記 | `docs: issue を分割し参照を追加 (#0007)` |
| status を `done` / `discarded` へ更新（承認後） | `docs: issue を完了にする (#0007)` |
| `CHANGES.md` の更新 | `docs: 変更履歴を更新 (#0007)` |
| `review-diff-code` の指摘の修正 | `fix: レビュー指摘を修正 (#0007)`（内容に応じて接頭辞を変える） |

### 手順

1. `git status --short` と `git diff` で変更内容を確認する。
2. **ファイルを明示して** `git add` する。`git add -A` や `git add .` は使わない。
3. `git diff --staged` でコミットする内容を確認する。次が含まれていないことを見る。
   - 秘密情報（`.env`、鍵、トークン）
   - 今回の作業と無関係な変更
   - デバッグコード、コメントアウトされたコード、不要な生成物
4. コミットする。

```bash
git commit -m "feat: git-rule スキルの本文を追加 (#0007)" -m "{本文}"
```

5. `git log -1` で、メッセージが規約に沿っているか確認する。
6. `git config` は変更しない。gitconfig のエイリアスやフックに依存しない。

### 粒度

**原則は 1 コミット = 1 issue である。** ただし次は別のコミットにしてよい例外として列挙する。

- issue の「詳細」に書かれた実装手順のコミット単位。その単位に従う。
- issue ファイルの変更（起票、status 更新、メモの追記、分割参照）。実装の変更とは分ける。
- `CHANGES.md` への記録。レビュー後の工程として独立のコミットにする。
- `review-diff-code` の指摘の修正。push 済みのコミットに畳まず、新しいコミットにする。
- subagent への委任やペアワークで分担した成果の統合。担当ごとに分かれてよい。

- 1 コミットは 1 つの論理的な変更だけを含む。無関係な変更を混ぜない。
- 各コミットで、リポジトリにテスト・lint・型チェックの仕組みがあればそれが通る状態にする。
- `WIP` や一時的なコミットを残さない。
- PR に出す前にコミットを 1 件へ畳むことはしない。squash はマージ時にユーザーが行う。1 issue のまとまりは PR の単位で保つ。

### 他のスキルが生んだ変更のコミット

`review-diff-code` はコミットを行わず、`agent-changelog` は追記まで行ってコミットしない。両者が作業ツリーに反映した変更（レビューでの修正、`CHANGES.md` への追記）は、このスキルに従って agent がコミットしてよい。

- 変更の内容とコミットメッセージを示して実行する。
- ユーザーが「コミットしないで」と明示した場合はコミットしない。
- 仕様の判断を要する修正は、未解決のままコミットしない。ユーザーの判断を待つ。
- コミットが適切かどうかを判断するのはユーザーである。ユーザーが適切でないと判断した場合は指示に従って取り消し・修正し、履歴の書き換えを伴うときは「履歴の扱い」に従って確認を得てから行う。

## push

次をすべて満たしたときだけ push する。

1. 現在のブランチが issue に対応する作業ブランチであり、base ブランチではない。
2. ブランチ名が「ブランチの命名規則」の導出値と一致している。
3. `review-diff-code` が完了し、判定が `PR 可` である。スキップできるのはユーザーが明示した場合のみ。
4. `CHANGES.md` への記録が済んでいる（順序は `レビューのルール` に従う）。
5. 未コミットの変更がない。
6. 追加するコミットの件名が規約に沿っている。

```bash
git log --format=%s main..HEAD   # 件名を一覧で確認（main は base ブランチ）
git push -u origin task/0007-add-git-rule-skill
```

- **base ブランチへの commit・push は agent は行わない。** 例外は、issue ファイルのみの変更についてユーザーが明示的に指示した場合だけである。この場合も push 前のレビューは省略しない。

## Pull Request

- 1 issue につき PR は 1 つ。差し先は base ブランチ。
- **タイトル**: コミットと同じ形式 `{接頭辞}: {概要} (#{NNNN})`。
- **本文**:

```markdown
## 概要

{issue の「概要」の要点}

## 関連 issue

issues/0007-add-git-rule-skill.md (#0007)

## 受け入れ条件

- [x] {達成した条件}
- [ ] {未達の条件とその理由}

## テスト

{実行したコマンドと結果。仕組みがなければ「なし」}

## レビュー

review-diff-code: {N} 周、判定: {PR 可}
```

- issue を `done` にするコミットは、ユーザーの完了承認を得た後に作る。承認前の PR で status を `done` にしない。受け入れ条件の達成状況は本文のチェックボックスで示す。
- PR の作成は `gh` が使えれば `gh pr create` を使い、使えない場合はタイトルと本文を提示してユーザーに作成してもらう。

## マージとその後

- マージと PR の close はユーザーのみが行う。agent は指示されても行わない。
- マージの方法（マージコミット、squash など）はユーザーの指示とリポジトリの設定に従う。squash では PR タイトルがコミットメッセージになるため、タイトルが上の規約に沿っていることが必要である。
- マージ後のブランチ削除はユーザーの指示または確認後に行い、`git branch -d`（マージ済みだけ削除できる形式）を使う。

## 履歴の扱い

- **push 済みのコミットの履歴は書き換えない**（rebase、amend、squash をしない）。
- base ブランチへの追従が必要な場合は merge する。rebase は使わない。

```bash
git fetch origin
git merge origin/main   # base ブランチ（名前は AGENTS.md の指定）
```

- コンフリクトの解決は、issue のスコープ内で正解が明確なものだけ agent が行う。仕様の解釈が分かれるものは選択肢と推奨案を添えてユーザーに渡す。
- 未 push のコミットの修正（`git commit --amend`、`git rebase -i`）は、ユーザーの確認後に行う。
- タグの作成と push は、ユーザーが指示した場合だけ行う。

## 禁止と要確認の操作

**禁止**（ユーザーに指示されても、リスクを説明して再確認する）

- force push（`--force`）。自分の作業ブランチでの `--force-with-lease` は下の「要確認」に従う
- `--no-verify` などによるフックや検証の迂回
- `git config` の変更
- 秘密情報のコミット
- 無関係な変更をコミットにも stash にも巻き込むこと
- base ブランチへの commit・push（上の例外を除く）

**要確認**（実行前に、内容と理由を示してユーザーの確認を得る）

- `git reset --hard`、`git clean`、`git restore` / `git checkout -- <path>`（変更の破棄）
- `git commit --amend`、`git rebase`（未 push の場合）
- `git push --force-with-lease`（自分の作業ブランチで、必要な場合のみ）
- `git branch -D`、リモートブランチの削除
- `git stash drop`、`git stash clear`、無関係な変更の stash
- remote の追加・変更、`git filter-branch` など履歴の書き換え

## 例（1 issue を扱うときの流れ）

`issues/0007-add-git-rule-skill.md`（type: `task`）が `ready` でユーザーの実装承認が出た場合。base ブランチは AGENTS.md の指定（このリポジトリでは `main`）である。

```bash
git fetch origin && git switch main && git merge --ff-only origin/main
git switch -c task/0007-add-git-rule-skill

git add issues/0007-add-git-rule-skill.md
git commit -m "docs: issue の status を doing に更新 (#0007)"

# 実装
git add skills/git-rule/SKILL.md AGENTS.md
git commit -m "feat: git-rule スキルを追加 (#0007)" -m "命名規則とコミットの規約をここに集約した"

# review-diff-code（最低 3 周、判定 PR 可）→ 指摘の修正 → CHANGES.md へ追記
git add skills/git-rule/SKILL.md
git commit -m "fix: レビュー指摘を修正 (#0007)"
git add CHANGES.md
git commit -m "docs: 変更履歴を更新 (#0007)"

git push -u origin task/0007-add-git-rule-skill
# gh pr create（タイトルと本文は上の形式）→ マージはユーザー
```

## ルール

- 状態を変える git 操作は、必ずこのスキルの規則に従う。参照系は自由である。
- base ブランチの名前は AGENTS.md の `git のルール` が定める。確認手順と使い方はここが持つ。
- 変更系の操作の前に `issues/` から対応する issue を特定する。特定できなければ `issue-rule` と `create-issue` で起票してから操作に入る。
- 1 issue = 1 ブランチ = 1 PR。ブランチ名は `<type>/<NNNN>-<slug>` で、issue ファイル名と `type` から導出する。issue にブランチの項目は持たせない。
- コミットの件名は `{接頭辞}: {概要} (#{NNNN})`。issue 番号を必ず付ける。
- `git add` はファイルを明示する。`git add -A` / `git add .` は使わない。
- 原則は 1 コミット = 1 issue。例外は「粒度」に列挙したものだけ。
- コミットは agent が自律的に行い、そのコミットが適切かどうかを判断するのはユーザーである。
- push と PR 作成の前には `review-diff-code` の判定が `PR 可` であること。順序は `レビューのルール` に従う。
- マージと PR の close は行わない。base ブランチへの直接 commit・push、force push、`--no-verify` は行わない。
- push 済みの履歴は書き換えない。base への追従は merge で行う。
- 破壊的な操作は、内容と理由を示してユーザーの確認を得てから行う。

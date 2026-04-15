- /develop:deveopスキルのデバッグを行う
- /develop:deveopスキルに従って各種作業を実施すること
- github リポジトリ
  - https://github.com/tatsu-nstet/test
- git project
  - https://github.com/users/tatsu-nstet/projects/4

## 参照ルール

- 原則として plugin として読み込まれたもの（SKILL・Agent・scripts 等、plugin 機構経由で到達できるもの）のみを参照する
- plugin のソースディレクトリを直接パス指定で読み込むのは避ける
- テスト項目ではなくても下記の観点で改善点があれば提言すること
  - エラーが発生する
  - 判断に迷うためガードレールの検討が必要
- テスト結果のNG、改善点があれば再現方法等をレポートにまとめること

## /develop 初期化方針

初期化自体もデバッグ対象なので、具体的な値はハードコードしない。
plugin の README および `.claude-develop.conf.example` の手順に従って進める（参照は plugin 機構経由で行う）。

### 進め方

- 初期化の基本方針がデフォルト通りでいいか、ヒアリングツール（AskUserQuestion）で確認する
- 回答を受けて具体的な案を作成する
- 案の通りで問題ないか、セクションごとにヒアリングツールで確認する
- 承認後に実行する

### デフォルトの方針

- 上の GitHub リポジトリ・git project をそのまま使う（`VCS_MODE="git_gh"`）
- ディレクトリは`.claude-develop.conf.example` のデフォルト値ではないものを使用する
- フェーズは `.claude-develop.conf.example` のデフォルト値に従う
- `CHECK_CMD` / `TEST_CMD` / `BUILD_CMD` は、BUILD フェーズ時点で存在するファイル群の存在確認に相当するデバッグ用ダミーコマンドとする（具体値はその都度生成）
- `DEV_SERVER_SKILL` はデバッグ用のダミー値とする（具体値はその都度生成）
- `GH_PROJECT_*` は README の「GitHub Projects ID の調べ方」節の手順で取得する

## テスト再実施フロー

### 前提（常設の状態）

- リモート `tatsu-nstet/test` の `main` ブランチは常設。tracked file は CLAUDE.md のみを保持（更新コミットの追加は可）。
- ローカル作業ディレクトリは CLAUDE.md のみが初期状態（`.git` も含め毎回消す運用）。
- SSH は `~/.ssh/config` にエイリアス `github-tatsu-nstet` が設定済み（HostName github.com / IdentityFile id_ed25519_github_tatsu-nstet）。GitHub には SSH URL `git@github-tatsu-nstet:tatsu-nstet/<repo>.git` で接続する（`git@github.com:` 直打ちは鍵が効かないため使わない）。

### 開始時

- SSH エイリアスで clone する:
  `git clone git@github-tatsu-nstet:tatsu-nstet/test.git .`

### 終了時（環境クリア）

- リモート:
  - Issue 完全削除は `gh api graphql` の `deleteIssue` mutation（`gh issue delete` は無い）
  - feature ブランチは `git push origin --delete <branch>`
  - Projects item は Issue 削除で自動連動削除
  - `main` ブランチは削除しない
- ローカル: CLAUDE.md のみを残し、他（`.git/` 含むテスト生成物一式）は削除する。

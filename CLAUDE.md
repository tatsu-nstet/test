- /develop:deveopスキルのデバッグを行う
- /develop:deveopスキルに従って各種作業を実施すること
- github リポジトリ
  - https://github.com/tatsu-nstet/test
- git project
  - https://github.com/users/tatsu-nstet/projects/4

## 参照ルール

- 原則として plugin として読み込まれたもの（SKILL・Agent・scripts 等、plugin 機構経由で到達できるもの）のみを参照する
- plugin のソースディレクトリを直接パス指定で読み込むのは避ける

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
- ディレクトリ・フェーズは `.claude-develop.conf.example` のデフォルト値に従う
- `CHECK_CMD` / `TEST_CMD` / `BUILD_CMD` は、BUILD フェーズ時点で存在するファイル群の存在確認に相当するデバッグ用ダミーコマンドとする（具体値はその都度生成）
- `DEV_SERVER_SKILL` はデバッグ用のダミー値とする（具体値はその都度生成）
- `GH_PROJECT_*` は README の「GitHub Projects ID の調べ方」節の手順で取得する

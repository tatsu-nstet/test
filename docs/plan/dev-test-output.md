---
slug: test-output
title: テスト出力
issue: 
phase: DONE
created: 2026-04-18
---

# 開発計画: テスト出力

プロジェクト: test
作成日: 2026-04-18
ステータス: 進行中

---

## 要件サマリ

develop スキル全体のデバッグ用テスト実行。各フェーズ Agent（develop-design / develop-spec-sync / develop-test-spec / develop-impl / develop-build）が、それぞれ通常のフェーズ成果物ファイルに対し `yyyy-mm-dd HH:MM:SS <agent名> test` の 1 行を追記するだけの最小タスクとする。詳細な設計や要件深堀りは行わず、develop フロー全体が DESIGN → SPEC_SYNC → TEST_SPEC → IMPL → BUILD の順でエラーなく完走することを確認する。

---

## ブレインストーミングメモ

### 確定事項
- 出力フォーマット: `yyyy-mm-dd HH:MM:SS <agent名> test`
- 出力先:
  - DESIGN → `docs/design/dev-test-output.md`（新規作成）
  - SPEC_SYNC → `docs/design/dev-test-output.md`（追記）
  - TEST_SPEC → `docs/test/dev-test-output.md`（新規作成）
  - IMPL → `./test-output.txt`（新規作成）
  - BUILD → `./test-output.txt`（追記）
- タイムゾーン: ローカル時刻（`date '+%Y-%m-%d %H:%M:%S'`）
- 既存ファイルがあれば末尾に追記、なければ新規作成

### 要検討事項
なし（推奨案 A/A/A で確定）。

### リスク・懸念
- BUILD のダミーコマンド（`test -f CLAUDE.md`）は `./test-output.txt` を検証しないが、デバッグ目的上問題なし。
- SPEC_SYNC は DESIGN → SPEC_SYNC の順序前提（develop フロー上保たれる）。

### 未深堀り論点
- デバッグ用テストのため、通常の深堀り観点（データモデル・アーキテクチャ・UX 等）は意図的にスキップ。

---

## ユーザー確認ログ

| 日時 | 質問内容 | 回答 |
|------|---------|------|
|  |  |  |

---

## 設計入力

<!-- DESIGN が意思決定するために必要な前提情報。
     必須 3 項目は原則として記入（該当なしの場合は「N/A: 理由」を明記）。
     任意 3 項目はプロジェクトの性質に応じて記入。空のまま残さず、不要なら節ごと削除する。
     新規観点の必須化は「過去の計画書で 2 回以上欠落した実績がある」ことを要件とする
     （plan/SKILL.md の運用ガイド参照）。候補は `<!-- FUTURE: ... -->` マーカーで残す。 -->

### 外部仕様・依存（必須）

N/A: 外部依存なし（シェル `date` コマンドのみ使用）。

### スコープ境界（必須）

- やること: 各フェーズ Agent が通常出力先ファイルに 1 行 `yyyy-mm-dd HH:MM:SS <agent名> test` を追記
- やらないこと: 実機能の設計・実装、テストコード作成、詳細ドキュメント整備

### 検証方針（必須）

- 各成果物ファイルに対応する Agent 名の行が追記されていること
- BUILD フェーズの `check_cmd` / `test_cmd` / `build_cmd`（ダミー）が全て成功すること
- develop フロー全体がエラーなく完走すること

---

## 影響範囲

<!-- 影響する設計書・コードのファイルを列挙する。テンプレートの項目はプロジェクトに合わせて書き換えること -->

### 設計書

- [ ] `docs/design/dev-test-output.md`（DESIGN 新規作成 / SPEC_SYNC 追記）
- [ ] `docs/test/dev-test-output.md`（TEST_SPEC 新規作成）

### 実装

- [ ] `./test-output.txt`（IMPL 新規作成 / BUILD 追記）

---

## 実行ログ

各フェーズの成果物チェックリスト。**各 Agent はステップ完了ごとに `[x]` を付けて記録を残すこと**。
オーケストレーターと再開時の各 Agent は、このセクションの **最初の未完了ステップ** を起点として動作する。
既に完了しているステップは冪等判定で自動スキップされる。

> **フェーズのカスタマイズ**: 中間フェーズ（INIT と USER_REVIEW の間）は `.claude-develop.conf` の `PHASES` と一致させること。
> 下記はデフォルト例（DESIGN → SPEC_SYNC → TEST_SPEC → IMPL → BUILD）。プロジェクトに応じて差し替え可能。

### INIT
- [x] 要件取得
- [x] 壁打ち（develop-brainstorm）
- [x] 作業ブランチ作成（VCS_MODE=git_gh のみ）
- [x] 計画書ドラフト作成
- [x] 外部ステータス更新（任意）
- [x] phase を次フェーズに更新

### DESIGN
- [x] 並行競合チェック
- [x] lock ファイル作成
- [x] 設計書執筆・更新
- [x] セルフレビュー反映
- [x] phase を次フェーズに更新

### SPEC_SYNC
- [x] 他設計書への波及確認・反映
- [x] セルフレビュー反映
- [x] phase を次フェーズに更新

### TEST_SPEC
- [x] テスト仕様書執筆
- [x] セルフレビュー反映
- [x] phase を次フェーズに更新

### IMPL
- [x] 実装
- [x] 設計書影響確認（※共通ルール参照）
- [x] セルフレビュー反映
- [x] phase を次フェーズに更新

### BUILD
- [x] 静的解析通過（CHECK_CMD）
- [x] 自動テスト通過（TEST_CMD）
- [x] 設計書影響確認（※共通ルール参照）
- [x] phase を USER_REVIEW に更新

### USER_REVIEW
- [x] ドラフトPR作成（VCS_MODE=git_gh のみ）
- [x] 開発サーバー起動 → ユーザー動作確認（DEV_SERVER_SKILL）
- [x] レビュー指摘対応（必要な場合）
- [x] 設計書影響確認（※共通ルール参照）
- [x] PR Ready化・マージ（VCS_MODE=git_gh のみ）
- [x] 外部ステータス Done 化（任意）
- [x] lock ファイル削除
- [x] phase を DONE に更新

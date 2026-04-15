---
slug: greet-command
title: greet コマンドを追加
issue: https://github.com/tatsu-nstet/test/issues/2
phase: SPEC_SYNC
created: 2026-04-15
---

# 開発計画: greet コマンドを追加

プロジェクト: test
作成日: 2026-04-15
ステータス: 進行中

---

## 要件サマリ

名前を渡すと挨拶文を出力する最小の bash CLI (`src/greet`) を追加する。`/develop` スキルのデバッグ題材として、設計書 1 本・テスト仕様書 1 本・実装 1 本で全フェーズを回せる規模に収める。英語/日本語 2 言語対応と「言語 1 個追加」の境界ケースを意図的に残し、§11.2「Edit 範囲ガイドライン」と brainstorm「結論明示ガイドライン」・§4/§11.2 双方向リンク可読性の検証素材とする。

---

## ブレインストーミングメモ

### 確定事項
- 実装言語: bash（shebang `#!/usr/bin/env bash`、単一ファイル `src/greet`、`chmod +x`）
- 出力形式: plain text のみ（JSON/`--format` は不採用）
- 名前未指定時: エラー終了（exit 1 + stderr に usage）、既定値は持たない
- 引数パース: 自前 while-case。`--lang <code>` / `--help|-h` / 位置引数 1 個
- 多言語対応: `--lang en|ja`（デフォルト `en`）
- テスト形式: 手動実行手順ベース（自動テストなし）

### 要検討事項
1. 未対応言語 (`--lang fr` 等) 指定時の挙動: 案A エラー終了 / 案B 英語フォールバック / 案C 警告付き英語フォールバック。推奨: 案A（CLI 慣習）
2. 対応言語の拡張方式: 案A case 文 / 案B 連想配列 / 案C 外部ファイル。推奨: 案A（最小・bash 3.2+ 互換）

### リスク・懸念
- 題材が単純すぎて設計書が薄くなる→要検討事項 1・2 を DESIGN に残して判断量を確保
- bash バージョン要件の暗黙化→DESIGN で明記
- ダミー check/test/build は実質検証にならない→テスト仕様書に手動手順を記載して補う

### 未深堀り論点
- stdout/stderr の出力先規約（慣習通り）
- UTF-8 エンコーディングと `LANG`/`LC_ALL` の扱い（DESIGN で触れる）

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

- bash（バージョンは DESIGN で明記。要検討事項2 推奨案A を採れば bash 3.2+）
- POSIX 標準コマンドのみ（`echo`, `printf`）
- 外部ライブラリ・パッケージマネージャなし

### スコープ境界（必須）

- やること:
  - `src/greet` 単一ファイルの bash CLI 実装
  - `--lang en|ja` / `--help|-h` / 位置引数 1 個（名前）
  - `designs/` に設計書 1 本
  - `test-specs/` にテスト仕様書 1 本（手動実行手順）
- やらないこと:
  - JSON 出力、`--format` オプション
  - 3 言語以上への対応（拡張余地は残すが本スコープ外）
  - 自動テスト（unit/integration test）
  - サブコマンド、対話モード、設定ファイル

### 検証方針（必須）

- 静的検証: `CHECK_CMD="ls designs/*.md"`（ファイル存在確認のダミー）
- 自動テスト: `TEST_CMD="ls test-specs/*-test-spec.md"`（ダミー）
- ビルド: `BUILD_CMD="ls src/*"`（ダミー）
- 手動検証: テスト仕様書に以下ケースを記載し目視確認
  - 正常系: `./src/greet Alice` → `Hello, Alice!`
  - 日本語: `./src/greet --lang ja Alice` → `こんにちは、Alice！`
  - ヘルプ: `./src/greet --help` → usage 表示、exit 0
  - 名前未指定: `./src/greet` → stderr に usage、exit 1
  - 未対応言語: 要検討事項1 確定後に記載

---

## 影響範囲

<!-- 影響する設計書・コードのファイルを列挙する。テンプレートの項目はプロジェクトに合わせて書き換えること -->

### 設計書

- [ ] `designs/greet-command.md`（新規）

### テスト仕様書

- [ ] `test-specs/greet-command-test-spec.md`（新規）

### 実装

- [ ] `src/greet`（新規）

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
- [x] 計画書ドラフト作成
- [x] 作業ブランチ作成（VCS_MODE=git_gh のみ）
- [x] 外部ステータス更新（任意）
- [x] phase を次フェーズに更新

### DESIGN
- [x] 並行競合チェック
- [x] lock ファイル作成
- [x] 設計書執筆・更新
- [x] セルフレビュー反映
- [x] phase を次フェーズに更新

### SPEC_SYNC
- [ ] 他設計書への波及確認・反映
- [ ] セルフレビュー反映
- [ ] phase を次フェーズに更新

### TEST_SPEC
- [ ] テスト仕様書執筆
- [ ] セルフレビュー反映
- [ ] phase を次フェーズに更新

### IMPL
- [ ] 実装
- [ ] 設計書影響確認（※共通ルール参照）
- [ ] セルフレビュー反映
- [ ] phase を次フェーズに更新

### BUILD
- [ ] 静的解析通過（CHECK_CMD）
- [ ] 自動テスト通過（TEST_CMD）
- [ ] 設計書影響確認（※共通ルール参照）
- [ ] phase を USER_REVIEW に更新

### USER_REVIEW
- [ ] ドラフトPR作成（VCS_MODE=git_gh のみ）
- [ ] 開発サーバー起動 → ユーザー動作確認（DEV_SERVER_SKILL）
- [ ] レビュー指摘対応（必要な場合）
- [ ] 設計書影響確認（※共通ルール参照）
- [ ] PR Ready化・マージ（VCS_MODE=git_gh のみ）
- [ ] 外部ステータス Done 化（任意）
- [ ] lock ファイル削除
- [ ] phase を DONE に更新

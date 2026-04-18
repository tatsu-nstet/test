# brainstorm: test-output

## 要件の解釈

develop スキル全体のデバッグ用テスト実行として、各フェーズ Agent が通常の成果物出力先に「`yyyy-mm-dd HH:MM:SS <agent名> test`」形式の 1 行を追記するだけのタスク。詳細な設計・ドキュメント確認は不要。

## 確定事項

1. **出力フォーマット**: `yyyy-mm-dd HH:MM:SS <agent名> test`（例: `2026-04-18 10:30:00 develop-design test`）
2. **各フェーズの出力先**:
   - DESIGN (develop-design): `docs/design/dev-test-output.md`
   - SPEC_SYNC (develop-spec-sync): `docs/design/dev-test-output.md` に追記（案A採択）
   - TEST_SPEC (develop-test-spec): `docs/test/dev-test-output.md`
   - IMPL (develop-impl): `./test-output.txt`（案A採択）
   - BUILD (develop-build): `./test-output.txt` に追記（案A採択）
3. **タイムゾーン**: ローカル時刻（`date '+%Y-%m-%d %H:%M:%S'` のデフォルト）
4. **追記方式**: 既存ファイルがあれば末尾に 1 行追記、なければ新規作成して 1 行書く

## 要検討事項

なし（推奨案 A/A/A で確定）。

## リスク・懸念

1. **BUILD フェーズのコマンド整合性**: `build_cmd` / `check_cmd` / `test_cmd` は `test -f CLAUDE.md` で常に成功する。BUILD Agent が追記した `./test-output.txt` はこれらコマンドの検証対象外だが、デバッグ目的としては問題なし。
2. **SPEC_SYNC の冪等性**: DESIGN → SPEC_SYNC の順序前提。develop フロー上は保たれる。
3. **クリーンアップ**: 出力ファイルは feature ブランチ上のコミット。テスト終了時のリモート削除でリセット可能。

## 未深堀り論点

- 本タスクはデバッグ用テストのため、通常の深堀り観点（データモデル・アーキテクチャ・UX 等）は意図的にスキップ。

## 設計入力

### 外部仕様・依存
- なし（シェルの `date` コマンドのみ使用）

### スコープ境界
- やること: 各フェーズ Agent が通常出力先に 1 行（`yyyy-mm-dd HH:MM:SS <agent名> test`）を追記
- やらないこと: 実機能の設計・実装、テストコード作成、詳細ドキュメント整備

### 検証方針
- 各成果物ファイルに対応する Agent 名の行が追記されていること
- BUILD フェーズの `check_cmd` / `test_cmd` / `build_cmd` が全て成功すること
- develop フロー全体が DESIGN → SPEC_SYNC → TEST_SPEC → IMPL → BUILD の順でエラーなく完走すること

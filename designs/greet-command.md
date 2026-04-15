# 設計書: greet コマンド

- slug: greet-command
- 関連 Issue: https://github.com/tatsu-nstet/test/issues/2
- 関連計画書: `plans/dev-greet-command.md`
- 関連 brainstorm: `.dev/plan/brainstorm-greet-command.md`
- 作成日: 2026-04-15

## 1. 概要

名前を引数で受け取り、挨拶文を標準出力に出力する最小の bash CLI。`src/greet` 単一ファイルで提供する。英語・日本語の 2 言語に対応し、今後 1 言語追加される場合の拡張余地を明示する。

本 CLI は `/develop` スキルのデバッグ題材として作成されるため、実装規模は最小に倒しつつ、設計判断が残る箇所（未対応言語時の挙動・言語拡張方式）は明示的に採用理由を残す。

## 2. 外部仕様

### 2.1 起動形態

```
src/greet [--lang <code>] [--help|-h] <name>
```

- `src/greet` は実行権限付き（`chmod +x`）の単一ファイル
- shebang: `#!/usr/bin/env bash`
- 引数の順序は自由（`--lang ja Alice` でも `Alice --lang ja` でも可）

### 2.2 オプション / 位置引数

| 種別 | 形式 | 説明 | 必須 |
|------|------|------|------|
| 位置引数 | `<name>` | 挨拶対象の名前。空文字・空白のみは不可 | 必須 |
| オプション | `--lang <code>` | 言語コード。`en` または `ja`。デフォルト `en` | 任意 |
| オプション | `--help`, `-h` | usage を stdout に出力して exit 0 | 任意 |

### 2.3 出力仕様

| 言語 | 出力例（name=Alice） |
|------|---------------------|
| `en` | `Hello, Alice!` |
| `ja` | `こんにちは、Alice！` |

- 正常出力は stdout、usage・エラーメッセージは stderr
- 改行は末尾に 1 個（`echo` のデフォルト挙動）
- 日本語出力は UTF-8 固定。ロケール（`LANG` / `LC_ALL`）の切替は CLI 側では行わない。呼び出し側が UTF-8 を扱える環境で実行すること（非 UTF-8 環境では文字化けする可能性があるが、本 CLI のスコープ外とする）

### 2.4 終了コード

| コード | 条件 |
|-------|------|
| 0 | 正常終了（挨拶出力 / `--help` 表示） |
| 1 | 引数エラー（名前未指定、未知のオプション） |
| 2 | 未対応言語指定（`--lang` に `en`/`ja` 以外を渡した場合） |

`--help` 表示時は exit 0。

### 2.5 エラー時の挙動

| ケース | 挙動 |
|--------|------|
| 名前未指定 | stderr に usage を出力、exit 1 |
| 未知のオプション（例: `--foo`） | stderr に `unknown option: --foo` と usage を出力、exit 1 |
| 未対応言語（例: `--lang fr`） | stderr に `unsupported language: fr` を出力、exit 2（案A 採用。brainstorm 要検討事項1 参照） |
| `--lang` の値が欠落（例: `--lang` のみ） | stderr に `missing value for --lang` と usage を出力、exit 1 |

### 2.6 usage 文（想定）

```
Usage: greet [--lang <en|ja>] <name>
       greet --help

Options:
  --lang <code>   Language code. en (default) or ja.
  -h, --help      Show this help and exit.

Examples:
  greet Alice              # => Hello, Alice!
  greet --lang ja Alice    # => こんにちは、Alice！
```

## 3. 内部設計

### 3.1 ファイル構成

- `src/greet`（単一 bash スクリプト、実行権限付き）

外部ファイル・ソース分割は行わない（brainstorm 要検討事項2・案C 不採用）。

### 3.2 実行要件

- bash 3.2 以上（macOS 標準 bash 互換）
  - 連想配列（bash 4+ 依存）は使用しない
  - 理由: 最小依存。案A（case 文）採用により 3.2+ で動作可能
- POSIX 標準コマンドのみ使用（`echo`, `printf`）

### 3.3 引数パース

自前 while-case ループで実装する。`getopts` は長オプション（`--lang`）非対応のため不採用。

擬似コード:

```
name=""
lang="en"
while [ $# -gt 0 ]; do
  case "$1" in
    --lang)
      [ $# -ge 2 ] || { エラー "missing value for --lang"; exit 1; }
      lang="$2"; shift 2 ;;
    --help|-h)
      usage を stdout へ; exit 0 ;;
    --*|-*)
      エラー "unknown option: $1"; exit 1 ;;
    *)
      [ -z "$name" ] || { エラー "too many positional arguments"; exit 1; }
      name="$1"; shift ;;
  esac
done
[ -n "$name" ] || { usage を stderr へ; exit 1; }
```

### 3.4 多言語対応（case 文ハードコード）

brainstorm 要検討事項2・案A 採用。

```
case "$lang" in
  en) echo "Hello, ${name}!" ;;
  ja) echo "こんにちは、${name}！" ;;
  *)  エラー "unsupported language: ${lang}"; exit 2 ;;
esac
```

**拡張ポイント**: 新言語を追加する場合、この case 文に 1 ブランチ追加するだけで済む。設計書・テスト仕様書・usage 文の言語コード列挙（`en|ja`）の 3 箇所も同時更新が必要になる（`agent-common-rules.md` §11.2 でいう「仕様変更を伴う」区分）。

### 3.5 stdout / stderr 規約

| 出力内容 | 出力先 |
|---------|--------|
| 挨拶文（正常出力） | stdout |
| `--help` 時の usage | stdout |
| エラー時の usage | stderr |
| エラーメッセージ | stderr |

## 4. 受け入れ基準（テスト仕様書の元情報）

TEST_SPEC フェーズは本節を元にテストケースを作成する。

| # | ケース | 入力 | 期待 stdout | 期待 stderr | exit |
|---|--------|------|-------------|-------------|------|
| 1 | 正常系（英語デフォルト） | `src/greet Alice` | `Hello, Alice!` | （空） | 0 |
| 2 | 正常系（日本語） | `src/greet --lang ja Alice` | `こんにちは、Alice！` | （空） | 0 |
| 3 | 正常系（オプション順入れ替え） | `src/greet Alice --lang ja` | `こんにちは、Alice！` | （空） | 0 |
| 4 | ヘルプ | `src/greet --help` | usage | （空） | 0 |
| 5 | ヘルプ（短形式） | `src/greet -h` | usage | （空） | 0 |
| 6 | 異常系: 名前未指定 | `src/greet` | （空） | usage | 1 |
| 7 | 異常系: 未対応言語 | `src/greet --lang fr Alice` | （空） | `unsupported language: fr` | 2 |
| 8 | 異常系: `--lang` の値欠落 | `src/greet --lang` | （空） | `missing value for --lang` + usage | 1 |
| 9 | 異常系: 未知のオプション | `src/greet --foo Alice` | （空） | `unknown option: --foo` + usage | 1 |
| 10 | 異常系: 位置引数過多 | `src/greet Alice Bob` | （空） | `too many positional arguments` | 1 |

## 5. スコープ外（本バージョンではやらない）

- JSON 出力 / `--format` オプション
- 3 言語以上への対応（設計として拡張余地は残すが本スコープ外）
- 自動テスト（unit / integration）
- サブコマンド、対話モード、設定ファイル
- ロケール自動判定（`LANG` に応じた既定言語の切替）

<!-- [FUTURE]: ロケール自動判定（LANG=ja_JP.UTF-8 なら --lang ja をデフォルトにする等）は拡張余地として残す -->

## 6. 次フェーズへの伝達事項

- **SPEC_SYNC**: 他設計書なし（本プロジェクトには本設計書のみ）。波及なし。
- **TEST_SPEC**: §4 の受け入れ基準表（10 ケース）を元にテスト仕様書を作成する。手動実行手順ベース。
- **IMPL**: §3 の擬似コードに従い `src/greet` を実装、`chmod +x` を付与する。
- **BUILD**: ダミーコマンド（`ls designs/*.md` 等）のため実質ファイル存在チェックのみ。

## 7. 設計判断ログ

| 項目 | 採用 | 理由 | 参照 |
|------|------|------|------|
| 未対応言語指定時 | 案A: エラー終了（exit 2） | CLI 慣習に沿い誤入力を早期検出。サイレントフォールバックは避ける | brainstorm 要検討事項1 |
| 言語拡張方式 | 案A: case 文ハードコード | bash 3.2+ 互換・最小構成。デバッグ題材として過剰な抽象化を避ける | brainstorm 要検討事項2 |
| 出力形式 | plain text のみ | 最小スコープ。JSON/`--format` は不採用 | brainstorm 確定事項3 |
| 名前未指定時 | エラー終了 | 既定値 "World" は意味を持たせにくい。CLI 慣習で素直 | brainstorm 確定事項4 |
| 引数パーサ | 自前 while-case | `getopts` は長オプション非対応 | brainstorm 確定事項5 |

<!-- ASSUMPTION: 未対応言語時の exit code を 2 に固定した。brainstorm 要検討事項1・案A は「エラー終了」とだけ記載され具体的な exit code は未指定。引数エラー (exit 1) と区別する意図で 2 を採用したが、CLI 慣習上 1 に統一する選択肢もある。ユーザー確認が取れたら本マーカーを削除する -->
<!-- ASSUMPTION: 位置引数過多 (`greet Alice Bob`) の扱いを「エラー exit 1」と定義した。brainstorm・Issue 本文では明示されていない。最後の引数を優先する寛容仕様も取り得るが、CLI 慣習からエラーに倒した -->

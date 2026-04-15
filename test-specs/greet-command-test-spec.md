# テスト仕様書: greet コマンド

- slug: greet-command
- 関連 Issue: https://github.com/tatsu-nstet/test/issues/2
- 関連設計書: `designs/greet-command.md`
- 関連計画書: `plans/dev-greet-command.md`
- 作成日: 2026-04-15

---

## 1. 要件対応表

| 要件ID | 要件内容 | テストケースID |
|--------|---------|--------------|
| REQ-01 | 名前を引数で受け取り、挨拶文を stdout に出力する | TC-01, TC-02, TC-03 |
| REQ-02 | `--lang en`（デフォルト）で英語挨拶を出力する | TC-01, TC-02 |
| REQ-03 | `--lang ja` で日本語挨拶を出力する | TC-03, TC-04 |
| REQ-04 | `--lang` オプションは引数の順序を問わず機能する | TC-04 |
| REQ-05 | `--help` / `-h` で usage を stdout に出力し exit 0 で終了する | TC-05, TC-06 |
| REQ-06 | 名前未指定時は stderr に usage を出力し exit 1 で終了する | TC-07 |
| REQ-07 | 未対応言語指定時は stderr に `unsupported language: <code>` を出力し exit 2 で終了する | TC-08 |
| REQ-08 | `--lang` の値が欠落した場合は stderr にエラーメッセージと usage を出力し exit 1 で終了する | TC-09 |
| REQ-09 | 未知のオプション指定時は stderr に `unknown option: <opt>` と usage を出力し exit 1 で終了する | TC-10 |
| REQ-10 | 位置引数が複数ある場合は stderr にエラーメッセージを出力し exit 1 で終了する | TC-11 |
| REQ-11 | `src/greet` は実行権限付きの単一 bash スクリプトである | TC-12 |

---

## 2. テストケース一覧

| ID | カテゴリ | テスト内容 | 前提条件 | 手順 | 期待結果 |
|----|---------|-----------|---------|------|---------|
| TC-01 | 正常系 | 英語デフォルト挨拶 | `src/greet` が実行可能 | `./src/greet Alice` を実行 | stdout: `Hello, Alice!` / stderr: 空 / exit: 0 |
| TC-02 | 正常系 | `--lang en` 明示指定 | `src/greet` が実行可能 | `./src/greet --lang en Alice` を実行 | stdout: `Hello, Alice!` / stderr: 空 / exit: 0 |
| TC-03 | 正常系 | 日本語挨拶 | `src/greet` が実行可能・UTF-8 端末 | `./src/greet --lang ja Alice` を実行 | stdout: `こんにちは、Alice！` / stderr: 空 / exit: 0 |
| TC-04 | 正常系 | オプション順入れ替え | `src/greet` が実行可能・UTF-8 端末 | `./src/greet Alice --lang ja` を実行 | stdout: `こんにちは、Alice！` / stderr: 空 / exit: 0 |
| TC-05 | 正常系 | `--help` オプション | `src/greet` が実行可能 | `./src/greet --help` を実行 | stdout: usage 表示（下記参照） / stderr: 空 / exit: 0 |
| TC-06 | 正常系 | `-h` 短形式オプション | `src/greet` が実行可能 | `./src/greet -h` を実行 | stdout: usage 表示（下記参照） / stderr: 空 / exit: 0 |
| TC-07 | 異常系 | 名前未指定 | `src/greet` が実行可能 | `./src/greet` を実行 | stdout: 空 / stderr: usage 表示 / exit: 1 |
| TC-08 | 異常系 | 未対応言語指定 | `src/greet` が実行可能 | `./src/greet --lang fr Alice` を実行 | stdout: 空 / stderr: `unsupported language: fr` / exit: 2 |
| TC-09 | 異常系 | `--lang` の値欠落 | `src/greet` が実行可能 | `./src/greet --lang` を実行 | stdout: 空 / stderr: `missing value for --lang` と usage / exit: 1 |
| TC-10 | 異常系 | 未知のオプション | `src/greet` が実行可能 | `./src/greet --foo Alice` を実行 | stdout: 空 / stderr: `unknown option: --foo` と usage / exit: 1 |
| TC-11 | 異常系 | 位置引数過多 | `src/greet` が実行可能 | `./src/greet Alice Bob` を実行 | stdout: 空 / stderr: `too many positional arguments` / exit: 1 |
| TC-12 | 境界値 | ファイル属性確認 | リポジトリルートにいること | `ls -l src/greet` および `head -1 src/greet` を実行 | 実行権限（`-rwxr-xr-x` 相当）あり・shebang `#!/usr/bin/env bash` がファイル先頭に存在する |
| TC-13 | 境界値 | 最大長の名前 | `src/greet` が実行可能 | `./src/greet "$(python3 -c "print('A'*1000)")"` を実行 | stdout: `Hello, A...A!`（1000 文字の A を含む） / stderr: 空 / exit: 0 |
| TC-14 | 境界値 | 空文字列の名前 | `src/greet` が実行可能 | `./src/greet ""` を実行 | stdout: 空 または stderr: usage 表示 / exit: 1（空文字・空白のみは不可の仕様） |
| TC-15 | ユーザテスト | ヘルプ表示の確認 | `src/greet` が実行可能 | `./src/greet --help` を実行し、表示内容を目視確認 | `Usage:`, `Options:`, `Examples:` セクションが表示され、利用方法として自然に読めること |
| TC-16 | ユーザテスト | 日英両言語での挨拶 | `src/greet` が実行可能・UTF-8 端末 | (1) `./src/greet 田中` (2) `./src/greet --lang ja 田中` を実行 | (1) `Hello, 田中!` が stdout に表示される (2) `こんにちは、田中！` が stdout に表示され、日本語名で自然な挨拶になっていること |
| TC-17 | ユーザテスト | エラー時のメッセージ確認 | `src/greet` が実行可能 | (1) `./src/greet` (2) `./src/greet --lang xx Alice` を実行し、出力先と内容を目視確認 | (1) エラーメッセージが stderr（`2>`にリダイレクトすると確認可）に出力されること (2) `unsupported language: xx` が stderr に出力され、次のアクションが明確にわかる内容であること |

---

## 3. カテゴリ説明

| カテゴリ | 説明 |
|---------|------|
| 正常系 | 通常の操作フロー。正常な入力で期待通りの挨拶文が出力されること |
| 異常系 | エラー・例外ケース。不正な入力・オプションに対して適切なエラーメッセージと exit コードで終了すること |
| 境界値 | 入力の上限・下限・特殊値（空文字列・長い名前・ファイル属性）のテスト |
| ユーザテスト | USER_REVIEW 時にユーザーが手動で確認すべき操作フロー。機能の正しさよりも利用体験・視認性の確認が目的 |

---

## 4. テスト実行前提

- リポジトリルートで実行する（コマンドのパスは `./src/greet` のように相対パスを使用）
- bash 3.2 以上の環境
- TC-03, TC-04, TC-16 は UTF-8 対応端末で実行する（非 UTF-8 環境では文字化けが発生する可能性あり。これは仕様上のスコープ外）
- TC-13 は `python3` が使用可能な環境を想定。使用できない場合は手動で長い名前文字列を用意すること

---

## 5. exit コード・出力先の確認方法

### exit コードの確認

```bash
./src/greet Alice
echo $?   # 直前のコマンドの exit コードを表示
```

### stderr と stdout の分離確認

```bash
# stderr のみ表示（stdout を捨てる）
./src/greet 2>&1 1>/dev/null

# stdout のみ表示（stderr を捨てる）
./src/greet Alice 2>/dev/null
```

---

## 6. usage 表示の期待内容（TC-05, TC-06, TC-07 の参照用）

`--help` / `-h` 時（stdout）と名前未指定時（stderr）で同じ usage が出力される想定。
最低限以下の情報が含まれること:

- 基本的な呼び出し形式（`greet [--lang <en|ja>] <name>` 相当）
- `--lang` オプションの説明（デフォルト `en` が分かること）
- `-h` / `--help` の説明
- 使用例（`greet Alice` → `Hello, Alice!` 相当）

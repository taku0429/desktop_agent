# Task Template (MVP)

## 目的（1行）
- 例：Flow-001 を実行すると「起動→前面→入力→終了」まで通る

## 背景 / 文脈（任意）
- 例：MVPのためwait許容。後で条件待ちに置換する。

## 対象（どれを触るか）
- docs/API.md: （読む / 変更する）
- docs/MOCK_DATA.md: （読む / 変更する）
- 対象Flow: Flow-xxx
- 影響するPrimitive: opA, opB

## 受け入れ条件（Done）
- [ ] Flow-xxx が X回中Y回成功（例：10回中8回）
- [ ] 失敗時もログに step_index / error_code が残る
- [ ] docs の正本と実装が一致している

## 制約（MVPの割り切り）
- 例：座標クリックでOK / wait許容 / OCRは未使用 など

## 実装方針（短く）
- 触るファイル：
- 追加/変更する関数：
- エラー処理（追加する error_code ）：

## 手順（Cursorにやらせること）
1. 変更前の関連箇所を引用（どこを直すか明確化）
2. 最小差分の実装
3. 動作確認方法（Flow実行 or 手動チェック）を提示
4. 必要なら docs/API.md / docs/MOCK_DATA.md / Changelog 更新

## 出力してほしいもの
- 変更差分の説明（箇条書き）
- 重要ファイルの変更点
- 動作確認手順

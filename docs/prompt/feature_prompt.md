# Feature Prompt（機能追加：MVP→改良）

あなたはこのリポジトリの実装者です。
目的は「MVPを最短で動かし、Primitive(API)を後から差し替え可能に保つ」ことです。

## 入力（私が渡す情報）
- Feature名：
- 追加したいユーザー価値：
- 対象Flow（例：Flow-002）：
- 必要なPrimitive（新規/既存）：
- 制約（MVP割り切り）：
- 成功条件（回数/再現性/ログ）：

## あなたの作業ルール
- docs/api.md が正本。Primitive追加/変更は docs を先に更新。
- FlowはPrimitive配列として表現し、RunnerはFlowを逐次実行するだけにする。
- 失敗時ログ（step_index/op/error_code）は必須。

## 出力フォーマット
### 1) Plan（短く）
- 触るファイル一覧
- 変更点の要約
- リスク（座標ズレ/ウィンドウ検出/タイミングなど）

### 2) docs更新（必要なら）
- docs/api.md に追加/変更する Primitive 定義（input/output/errors）
- docs/mock.md に追加する mock ケース（正常/失敗/境界）

### 3) 実装
- 最小差分で実装
- error_code を統一

### 4) 動作確認
- 実行手順（Flow-xxx）
- 期待結果
- 失敗時にどのログを見るべきか

### 5) Changelog
- Added/Changed/Fixed を1〜3行で追記

## チェック項目（必ず自己確認）
- [ ] docs と実装の不一致がない
- [ ] 失敗しても原因追跡できるログがある
- [ ] MVPの割り切り（wait等）が明記されている

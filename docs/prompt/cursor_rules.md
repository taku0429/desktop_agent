# Cursor Project Rules (MVP / API-improve)

## 0. 最優先の前提
- このプロジェクトは「MVPを早く動かす」→「操作API(Primitive)を改良して差し替える」を繰り返す。
- UIはRunnerに命令するだけ。RunnerはPrimitiveの実行に集中する（UIロジックを持たない）。
- 仕様の正本は docs/ で、実装は docs に追従する（Contract-first）。
  - docs/api.md（API設計書）が正
  - docs/mock.md（Mock設計書）が正
  - 不一致があれば docs を先に更新してから実装する

## 1. 変更の粒度
- 1タスク = 1目的 = 最小差分
- 勝手に大改造しない（まず動く最小を作る）

## 2. Primitive / Flow の扱い
- Primitiveは「最小の操作単位」。増やすときは docs/api.md を先に追記してから実装。
- Flowは「Primitiveの配列（JSONなど）」として管理し、RunnerはFlowを順に実行するだけ。
- MVPでは wait を許容（安定化目的）。後で条件待ち・画像認識・OCR等に改善する。

## 3. ログとエラー（MVPでも必須）
- 失敗しても必ず原因が追えるログを出す。
  - step_index, op, input(要約), timestamp, error_code, message
- error_code は固定の辞書にする（追加・変更は docs/api.md で管理）。

## 4. テスト方針（最小でOK）
- 「壊れた」を早期に検知することが目的。
- MVPはチェックリスト or 小さな自動テストのどちらでも良いが、
  - 最低でも Flow の成功/失敗が分かるものを残す。

## 5. 進め方（毎回この順）
1) docs/api.md / docs/mock.md を読み、今回のタスクに必要な範囲を把握
2) 実装前に Plan を短く書く（ファイル/関数/差分）
3) 実装（最小差分）
4) 動作確認（Flow実行 or チェックリスト）
5) docs 更新（必要なら） + Changelog 更新

## 6. 不明点が出たら
- “推測で実装”はしない。必要な仕様を docs に追記してから進める。
- 過去の会話/方針が必要なら @Past Chats を参照して取り込む（長文コピペしない）。 

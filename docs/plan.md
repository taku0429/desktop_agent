```md
# plan.md — 給料王転記ローカルAIエージェント（PySide6）開発Plan

目的：Excel→給料王への転記を最終ゴールにしつつ、まずは **mockデータで「起動/ウィンドウ操作/入力」まで動くMVP** を最短で通す。  
前提：表示倍率 100%（最初は座標ベースでもOK）、UIとRunner（実行エンジン）を分離。

---

## 0. ゴール定義

### MVP（最初に動く状態）
- アプリ起動/終了できる
- 「シナリオ（手順）」を選んで実行できる
- 手順は **mockデータ** を使って実行（Excel読込は後回し）
- OS操作の最小セット：
  - アプリ起動
  - ウィンドウ操作（最小化/最大化/前面）
  - クリック、カーソル移動、キー入力
  - スクショ取得（OCRは後回しでもOK）

### 次フェーズ（拡張）
- Excel読込（openpyxl/pandas）→ 正規化 → マッピング
- OCRで状態認識（画面遷移の成功判定）
- 手順のDB化（バージョニング/再利用/共有）

---

## 1. アーキテクチャ（UIとRunner分離）

### コンポーネント
- `ui/`：PySide6画面（開始/停止/ログ/進捗）
- `runner/`：実行エンジン（状態管理・ステップ実行・停止・失敗診断）
- `actions/`：クリック/入力/ウィンドウ操作/スクショ等の原子的アクション
- `scenarios/`：アクション列（手順定義。最初はjson）
- `domain/`：mock給与データ、マッピング定義
- `infra/`：ログ、設定、永続化（後でSQLite等）

### 基本方針
- UI：Runnerへ「開始/停止」を送るだけ
- Runner：UIへ「ログ/進捗/結果」をイベントで返す
- RunnerはUIとは別スレッド（UIフリーズ防止）

---

## 2. フォルダ構成（初期）

```

app/
main.py
ui/
main_window.py
widgets/
log_panel.py
scenario_panel.py
runner/
runner.py
state.py
events.py
actions/
base.py
mouse.py
keyboard.py
window.py
screenshot.py
scenarios/
sample_kingsalary_boot.json
sample_input_mock.json
domain/
mock_payroll.json
mapping.json
infra/
logger.py
config.py
storage.py

```

---

## 3. UI（PySide6）仕様（最小）

### MainWindow
- 左：Scenario一覧（`scenarios/*.json` を読み込み表示）
- 中：実行ボタン群
  - `Run`
  - `Stop`
  - `Dry Run（OS操作せずログだけ）`（安全のため推奨）
- 右：ログ表示
  - レベル（INFO/WARN/ERROR）
  - 現在ステップ番号/名前
  - 成功/失敗・失敗理由

### UIルール
- Runnerは別スレッドで動作（QThread/QRunnable等）
- UI→Runner：開始/停止コマンド
- Runner→UI：イベント（ログ/進捗/結果）

---

## 4. Runner（実行エンジン）仕様

### Runner責務
- シナリオ読み込み（ステップ列）
- 状態管理：`IDLE / RUNNING / STOPPING / FAILED / DONE`
- ステップ実行：
  - `precheck`（対象アプリ起動可能か等）
  - `execute(step)`
  - `postcheck`（最初は簡易でOK）
- リトライ（例：クリック失敗時 最大2回）
- 停止（Stopが来たら安全に中断）
- 失敗診断（例外ログ + スクショ保存）

### Scenario（json）形式（例）
- `steps` 配列でステップを定義
- ステップの共通フィールド：
  - `type`: `"launch" | "focus_window" | "maximize_window" | "minimize_window" | "click" | "type_text" | "wait" | "screenshot" | "hotkey"`
  - `params`: typeごとの引数

---

## 5. Actions（原子的操作）実装順

### Phase A（まず通す）
1. `launch_app(exe_path)`
2. `focus_window(title_contains)`
3. `maximize_window(title_contains)` / `minimize_window(...)`
4. `click(x, y)`
5. `type_text(text)`
6. `wait(seconds)`
7. `screenshot(save_path)`
8. `hotkey(keys...)`（例：Alt+F4）

### ライブラリ候補（Windows想定）
- ウィンドウ操作：`pywinauto`（安定性重視）/ 代替 `pygetwindow`
- 入力操作：`pyautogui`（MVPはこれでOK）

---

## 6. シナリオ（MVPは2本）

### シナリオ1：給料王を起動→最大化→スクショ→終了（疎通確認）
- launch
- wait
- focus_window
- maximize_window
- screenshot
- hotkey(Alt+F4)

### シナリオ2：mockデータを1件入力（最小入力だけ）
- launch
- wait
- focus_window
- （座標クリックで入力欄へ移動：MVPは固定座標でOK）
- type_text（氏名）
- click（次の欄）
- type_text（金額）
- screenshot
- hotkey(Alt+F4)

---

## 7. データ（mock）設計

### `domain/mock_payroll.json`
- 従業員1人分でOK
- 例：`name`, `amount`, `month`

### `domain/mapping.json`
- MVPは「入力先=座標」で持つ（後でOCR/要素検出へ移行）
- 例：
  - `name_field: {x, y}`
  - `amount_field: {x, y}`

---

## 8. ログ/証跡（必須）

- 全ステップ開始/終了ログ
- 失敗時：
  - 例外内容
  - 実行中ステップ
  - スクショ保存パス
- ログ出力先：
  - UIログパネル
  - `logs/app_YYYYMMDD.log`

---

## 9. 安全装置（最低限）

- `Dry Run`：OS操作無し（ログのみ）
- 実行前チェック：
  - 対象アプリ識別（ウィンドウタイトル一致）
  - 表示倍率100%前提の警告
- 緊急停止：
  - Stopボタンで停止
  - （余裕があれば）ESC長押し等で停止

---

## 10. 実装タスク（順番このまま）

### Day 1：枠組み
- [ ] PySide6でMainWindow（Scenario一覧/Run/Stop/ログ）作成
- [ ] Runnerスレッド化（signals/slots）
- [ ] logger（UI + ファイル出力）

### Day 2：Actions基盤
- [ ] `actions/base.py`（共通IF）
- [ ] `launch / wait / screenshot` 実装
- [ ] `click / type_text` 実装

### Day 3：ウィンドウ操作
- [ ] `focus_window / maximize / minimize` 実装
- [ ] シナリオ1成功（起動→最大化→スクショ→終了）

### Day 4：mock入力
- [ ] mock_payroll.json 読み込み
- [ ] シナリオ2成功（1件入力）
- [ ] 失敗時スクショ・リトライ・停止を確認

---

## 11. その後の拡張ポイント

- SQLiteでScenario/Mapping/Run履歴を保存
- OCR導入（画面状態の成功判定）
- 座標固定 → テンプレ画像マッチング → UI要素操作 へ段階移行
- Excel読込 → 正規化 → 入力自動化（本命）

---

## 付録：MVP完了の「動作条件」
- 1クリックでシナリオ1が最後まで完走する
- シナリオ2でmock1件入力が再現性高く動く
- 失敗時にスクショとログが残る
- Stopで安全に中断できる
```

````md
# MOCK_DATA.md — Runner Primitive API Mock Data (v0 / MVP)

> 目的：docs/API.md の Primitive 契約に沿って、UI/Runnerの開発・テストに使える **固定のモックデータ** を用意する。  
> 方針：**代表ケース（正常/失敗/境界）を固定IDで管理**し、必要なら JSON Schema から自動生成もできる形にする（json-schema-faker など）。 :contentReference[oaicite:0]{index=0}

---

## 0. ルール
- モックは **ケースID固定**（テストが安定する）
- Flowは **Command配列（Primitiveの並び）**
- 失敗ケースは「どのstepで」「どのerror_code」が出るかを明示
- API変更したら **まずここを更新** → 実装更新の順

---

## 1. 共通モック（環境前提）
### 1.1 Window タイトル（例）
- 給料王（例）: `"給料王"` または `"給料王 - メイン"`
- 何でも良いので「title_containsで拾える文字列」を決めておく

### 1.2 画面/座標（例）
> MVPでは座標クリックでOK。後で画像一致/OCRへ移行。

- クリック座標例
  - 入力欄: (320, 240)
  - OKボタン: (860, 680)

※ 解像度/表示倍率が変わると崩れるので、最初は固定環境で回す。

---

## 2. Command（ステップ）モック

### 2.1 Command 基本形（サンプル）
```json
{
  "id": "cmd-0001",
  "op": "app.launch",
  "params": {
    "app_id": "C:\\Program Files\\KyuryouOh\\KyuryouOh.exe",
    "args": []
  }
}
````

---

## 3. Flow（手順）モック

### 3.1 Flow-001（正常）: 起動→前面→入力→終了

```json
{
  "flow_id": "flow-001-happy",
  "description": "MVP: launch -> focus -> click -> type -> close(alt+f4)",
  "commands": [
    { "id": "cmd-0001", "op": "app.launch", "params": { "app_id": "C:\\Program Files\\KyuryouOh\\KyuryouOh.exe", "args": [] } },
    { "id": "cmd-0002", "op": "wait", "params": { "ms": 900 } },
    { "id": "cmd-0003", "op": "window.focus", "params": { "title_contains": "給料王" } },
    { "id": "cmd-0004", "op": "wait", "params": { "ms": 400 } },
    { "id": "cmd-0005", "op": "mouse.click", "params": { "x": 320, "y": 240, "button": "left", "clicks": 1 } },
    { "id": "cmd-0006", "op": "keyboard.type", "params": { "text": "mock-input", "interval_ms": 20 } },
    { "id": "cmd-0007", "op": "keyboard.hotkey", "params": { "keys": ["alt", "f4"] } }
  ]
}
```

期待:

* `RunResult.ok = true`
* `results[*].ok = true`

---

### 3.2 Flow-002（失敗）: window.focus が見つからない

```json
{
  "flow_id": "flow-002-window-not-found",
  "description": "Expected to fail at window.focus",
  "commands": [
    { "id": "cmd-0101", "op": "app.launch", "params": { "app_id": "C:\\Program Files\\KyuryouOh\\KyuryouOh.exe", "args": [] } },
    { "id": "cmd-0102", "op": "wait", "params": { "ms": 500 } },
    { "id": "cmd-0103", "op": "window.focus", "params": { "title_contains": "存在しないタイトル" } }
  ],
  "expected_failure": {
    "failed_step_index": 2,
    "failed_op": "window.focus",
    "error_code": "WINDOW_NOT_FOUND"
  }
}
```

期待:

* `RunResult.ok = false`
* `results[2].ok = false`
* `results[2].error.code = "WINDOW_NOT_FOUND"`

---

### 3.3 Flow-003（失敗）: app.launch が失敗（パス不正）

```json
{
  "flow_id": "flow-003-app-not-found",
  "description": "Expected to fail at app.launch",
  "commands": [
    { "id": "cmd-0201", "op": "app.launch", "params": { "app_id": "C:\\INVALID\\notfound.exe", "args": [] } }
  ],
  "expected_failure": {
    "failed_step_index": 0,
    "failed_op": "app.launch",
    "error_code": "APP_NOT_FOUND"
  }
}
```

---

### 3.4 Flow-004（境界）: screen.capture（regionあり/なし）

```json
{
  "flow_id": "flow-004-capture",
  "description": "capture full and region",
  "commands": [
    { "id": "cmd-0301", "op": "screen.capture", "params": {} },
    { "id": "cmd-0302", "op": "screen.capture", "params": { "region": { "x": 0, "y": 0, "w": 800, "h": 600 } } }
  ]
}
```

期待:

* `output.path` が返る（例: `C:\\...\\run-xxx\\step-0.png`）

---

## 4. RunResult（戻り値）モック

### 4.1 成功例

```json
{
  "run_id": "run-20260207-0001",
  "ok": true,
  "results": [
    { "step_index": 0, "id": "cmd-0001", "op": "app.launch", "ok": true, "output": { "pid": 12345 }, "error": null,
      "started_at": "2026-02-07T12:00:00.000Z", "ended_at": "2026-02-07T12:00:00.120Z" },
    { "step_index": 1, "id": "cmd-0002", "op": "wait", "ok": true, "output": { "ok": true }, "error": null,
      "started_at": "2026-02-07T12:00:00.121Z", "ended_at": "2026-02-07T12:00:01.021Z" }
  ],
  "summary": { "total_steps": 2, "failed_step_index": null, "failed_op": null }
}
```

### 4.2 失敗例（WINDOW_NOT_FOUND）

```json
{
  "run_id": "run-20260207-0002",
  "ok": false,
  "results": [
    { "step_index": 0, "id": "cmd-0101", "op": "app.launch", "ok": true, "output": { "pid": 23456 }, "error": null,
      "started_at": "2026-02-07T12:10:00.000Z", "ended_at": "2026-02-07T12:10:00.130Z" },
    { "step_index": 1, "id": "cmd-0102", "op": "wait", "ok": true, "output": { "ok": true }, "error": null,
      "started_at": "2026-02-07T12:10:00.131Z", "ended_at": "2026-02-07T12:10:00.631Z" },
    { "step_index": 2, "id": "cmd-0103", "op": "window.focus", "ok": false, "output": null,
      "error": { "code": "WINDOW_NOT_FOUND", "message": "window title not found", "details": { "title_contains": "存在しないタイトル" } },
      "started_at": "2026-02-07T12:10:00.632Z", "ended_at": "2026-02-07T12:10:00.700Z" }
  ],
  "summary": { "total_steps": 3, "failed_step_index": 2, "failed_op": "window.focus" }
}
```

---

## 5. （任意）スキーマから自動生成したい場合

* JSON Schema からフェイクデータ生成：**json-schema-faker** ([JSON Schema Faker][1])
* OpenAPI から **モック + バリデーション**：**Prism**（HTTP API化する場合） ([Stoplight][2])

---


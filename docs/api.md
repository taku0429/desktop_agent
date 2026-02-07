````md
# API.md — Runner Primitive API (v0 / MVP)

> 目的：UI から Runner を呼び出す「操作プリミティブ(API)」の契約(Contract)を固定し、MVPで動かしつつ後から差し替え・改良できるようにする。  
> スキーマは **JSON Schema (draft 2020-12)** を前提にし、必要なら **OpenAPI 3.1** にもそのまま寄せられる形にする。 :contentReference[oaicite:0]{index=0}

---

## 0. 原則（このAPIの守るべきこと）
- **docs/API.md が正本**。実装は必ず追従する（Contract-first）。
- UI は Runner に **Command 配列(Flow)** を渡すだけ。Runner は逐次実行して結果を返す。
- MVPでは `wait` を許容。後で「条件待ち」「画像/OCR判定」等に改善する。
- 失敗しても原因追跡できるよう **error_code / step_index / op** を必ず返す。

---

## 1. データモデル（共通）

### 1.1 Command（1ステップ）
```json
{
  "id": "cmd-0001",
  "op": "app.launch",
  "params": { "app_id": "C:\\path\\app.exe", "args": [] }
}
````

* `id`: string（任意だが推奨。ログ追跡用）
* `op`: string（Primitive名）
* `params`: object（Primitiveごとの入力）

### 1.2 StepResult（各ステップの結果）

```json
{
  "step_index": 0,
  "id": "cmd-0001",
  "op": "app.launch",
  "ok": true,
  "output": { "pid": 12345 },
  "error": null,
  "started_at": "2026-02-07T12:00:00.000Z",
  "ended_at": "2026-02-07T12:00:00.120Z"
}
```

### 1.3 Error（共通エラー形式）

```json
{
  "code": "APP_NOT_FOUND",
  "message": "app_id not found",
  "details": { "app_id": "C:\\path\\app.exe" }
}
```

* `code`: 後述の Error Code Dictionary から選ぶ
* `message`: 人間が読む用（短く）
* `details`: 任意のデバッグ情報（機密は入れない）

### 1.4 RunResult（Flow全体の結果）

```json
{
  "run_id": "run-20260207-0001",
  "ok": false,
  "results": [ /* StepResult[] */ ],
  "summary": {
    "total_steps": 7,
    "failed_step_index": 3,
    "failed_op": "window.focus"
  }
}
```

---

## 2. Primitive 一覧（v0 / MVP）

> `params` と `output` は **最小**。追加は互換性に注意して行う。

### 2.1 app.launch

* 目的：アプリを起動
* params:

  * `app_id` (string) … exeパス or 識別子
  * `args` (string[], optional)
* output:

  * `pid` (number, optional)

**errors**

* `APP_NOT_FOUND`
* `LAUNCH_FAILED`

---

### 2.2 app.close

* 目的：アプリ終了（プロセス指定できるなら優先）
* params:

  * `pid` (number, optional)
  * `title_contains` (string, optional) … ウィンドウから閉じる場合
* output: `{ "ok": true }`

**errors**

* `CLOSE_FAILED`
* `WINDOW_NOT_FOUND`

---

### 2.3 window.focus

* 目的：対象ウィンドウを前面化
* params:

  * `title_contains` (string)
* output:

  * `focused_title` (string, optional)

**errors**

* `WINDOW_NOT_FOUND`
* `FOCUS_FAILED`

---

### 2.4 mouse.click

* 目的：座標クリック
* params:

  * `x` (number)
  * `y` (number)
  * `button` ("left"|"right", optional, default "left")
  * `clicks` (number, optional, default 1)
* output: `{ "ok": true }`

**errors**

* `CLICK_FAILED`

---

### 2.5 mouse.move

* 目的：座標へ移動（将来の安定化用）
* params:

  * `x` (number)
  * `y` (number)
  * `duration_ms` (number, optional)
* output: `{ "ok": true }`

**errors**

* `MOVE_FAILED`

---

### 2.6 keyboard.type

* 目的：文字入力
* params:

  * `text` (string)
  * `interval_ms` (number, optional, default 0)
* output: `{ "ok": true }`

**errors**

* `TYPE_FAILED`

---

### 2.7 keyboard.hotkey

* 目的：ショートカット入力
* params:

  * `keys` (string[]) … 例: ["ctrl","s"]
* output: `{ "ok": true }`

**errors**

* `HOTKEY_FAILED`

---

### 2.8 screen.capture

* 目的：スクリーンショット取得（判断材料）
* params:

  * `region` (object, optional)

    * `x` `y` `w` `h` (number)
* output:

  * `path` (string)

**errors**

* `CAPTURE_FAILED`

---

### 2.9 wait

* 目的：待機（MVP安定化）
* params:

  * `ms` (number)
* output: `{ "ok": true }`

**errors**

* `WAIT_FAILED`

---

## 3. Error Code Dictionary（v0）

* `APP_NOT_FOUND`
* `LAUNCH_FAILED`
* `CLOSE_FAILED`
* `WINDOW_NOT_FOUND`
* `FOCUS_FAILED`
* `CLICK_FAILED`
* `MOVE_FAILED`
* `TYPE_FAILED`
* `HOTKEY_FAILED`
* `CAPTURE_FAILED`
* `WAIT_FAILED`

> 追加/変更するときは **ここに追記**してから実装する。

---

## 4. ログ仕様（最低限）

Runnerは各 StepResult を必ず生成し、失敗時でも `results` を返す。

**必須フィールド**

* `run_id`, `step_index`, `op`, `ok`, `started_at`, `ended_at`
* 失敗時は `error.code` を必須

---

## 5. JSON Schema（draft 2020-12）※必要に応じて使う

> スキーマで検証できるようにしておくと、API改良時に壊れにくい。 ([json-schema.org][1])

### 5.1 Command Schema（最小）

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.local/schemas/command.json",
  "type": "object",
  "required": ["op", "params"],
  "properties": {
    "id": { "type": "string" },
    "op": { "type": "string" },
    "params": { "type": "object" }
  },
  "additionalProperties": false
}
```

### 5.2 Error Schema（最小）

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.local/schemas/error.json",
  "type": "object",
  "required": ["code", "message"],
  "properties": {
    "code": { "type": "string" },
    "message": { "type": "string" },
    "details": { "type": "object" }
  },
  "additionalProperties": false
}
```

---

## 6. 将来拡張（v1候補）

* `window.find`（複数候補の列挙）
* `screen.match_template`（画像一致で座標取得）
* `ocr.read`（OCRで状態判定）
* `wait.until`（条件待ち：timeout/retry）

---

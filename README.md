# desktop_agent

給料王への転記を最終ゴールにした **ローカルAIエージェント（Windows / PySide6想定）** の作業リポジトリです。  
まずは mock データで「起動/ウィンドウ操作/入力」まで動くMVPを最短で通し、Primitive(API)を後から差し替え・改良できる形を維持します。

## Docs（入口）

- `docs/README.md`（まずここ）
- `docs/plan.md`（MVPの到達点と実装方針）
- `docs/api.md`（UI→Runner の Primitive API 契約）
- `docs/mock.md`（APIに沿った固定モック）
- `docs/prompt/`（Cursor用プロンプトテンプレ）

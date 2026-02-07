<!-- docs index -->

# docs/ index

この `docs/` は **仕様の正本（Contract-first）** です。実装は必ず `docs/` に追従します。

## 読み始め（おすすめ順）

1. `docs/plan.md`  
   MVPの到達点・フェーズ分け・ディレクトリ方針
2. `docs/api.md`  
   UI → Runner の **Primitive API 契約**（Command/RunResult/Error）
3. `docs/mock.md`  
   `docs/api.md` に沿った **固定モック（正常/失敗/境界）**
4. `docs/prompt/`  
   Cursorに投げる用のプロンプトテンプレート（Feature/Bug/Task）

## 書き方ルール（最低限）

- **ファイル名の参照は実在するパスで**（例：`docs/api.md`）。`API.md` のような大小文字ゆれは避ける（環境差でリンクが死ぬ）。
- **Markdownとして読める状態を維持**（ファイル全体をコードフェンスで囲わない）。
- **仕様変更の順番**：`docs/api.md` → `docs/mock.md` → 実装


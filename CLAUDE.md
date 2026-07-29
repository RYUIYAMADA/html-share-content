# Project: html-share-content

## 概要
MacBook Pro 16" JIS 配列向けの Karabiner-Elements キーボード配列エディタ。レイヤー・単押し/長押し・アプリ別・同時押し（コンボ）・Vimium 設定をブラウザ上で編集し、Karabiner 用 JSON / Vimium options を書き出す単一HTMLツール。本体は `keyboard-editor.html` 1ファイルに完結（現在 v0.22.0）。

## 技術スタック（実態）
- 純粋な HTML + CSS + Vanilla JS（フレームワーク・ビルド・依存パッケージ一切なし）。`package.json`/`pyproject` は存在しない。
- 全コードを `keyboard-editor.html` 内にインライン（`<style>` + 末尾の IIFE `<script>`）。外部CDN・外部フォント・外部画像なし＝完全自己完結。
- 永続化: `localStorage`（キー `kb-layout-studio:v2`）。状態は単一 `data` オブジェクト + JSON スナップショットによる undo/redo（履歴80件上限）。
- 任意のローカルサーバ連携: `fetch("/api/ping")` 生存確認 → `fetch("/api/write-karabiner")` で Karabiner 設定へ直接書込。サーバ未起動時は JSON ダウンロードへ自動フォールバック（`serverAvailable` フラグで分岐）。サーバ本体はこのリポジトリには含まれていない（`start-keyboard-studio.command` 前提・未確定・要確認）。

## ディレクトリ構成
- `keyboard-editor.html` — アプリ本体（UI・状態管理・エクスポート全部入り）。
- `README.md` — タイトル1行のみ（実質未整備）。
- それ以外のソース・assets・テストは無し。フラット構成。

## 命名・コーディング規則（既存傾向）
- 全体を即時関数 `(() => { ... })()` で1スコープに閉じる。`const $ = id => document.getElementById(id)` で DOM 取得を統一。
- 状態は単一 `data`（meta/layers/apps/categories/functions/bindings/combos/settings）。変更は必ず `pushHistory()` → 反映 → `commit()` の流れ。直接 DOM を真実源にしない（data が真実源、UI は再描画）。
- 破損データ対策に `sanitizeData()` でスキーマ検証＋欠損補完＋孤立参照解消（白紙化/起動不能を防ぐ）。データ構造を変えるときは sanitize と移行ロジックも必ず更新する。
- コメントは日本語、UIラベル・通知文も日本語。変数/関数名は英語 camelCase（`writeKarabiner`, `refreshAll`, `getBinding`）。
- CSS は `:root` のCSS変数に集約（`--paper`/`--text`/`--accent`/`--hold` 等）＋テーマ変数を設定パネルで上書き。色は変数経由が基本（DESIGN.md 参照）。

## 禁止事項（PJ固有）
- ビルドツール・npm依存・外部CDN/フォント/スクリプトの導入禁止。「ダブルクリックで開けば動く単一HTML」を壊さない。
- `STORE_KEY`（`kb-layout-studio:v2`）を無断で変更しない＝既存ユーザーの保存データが消える。スキーマ変更時は `sanitizeData()` 内に移行ロジックを足して後方互換を保つ。
- `data` を直接書き換えて `commit()`/`pushHistory()` を通さない更新は禁止（undo/redo と自動保存が壊れる）。
- `/api/write-karabiner` は Karabiner 設定ファイルへの書込（副作用あり）。挙動を変える実装はサンドボックス実行ポリシーに従い、本番設定書込はGate確認（グローバル `sandbox-execution-policy.md` 準拠）。
- ファイルを分割してリポジトリ構成を増やす場合は、html-share 配布（単一HTML前提）と矛盾しないか先に確認する（未確定・要確認）。

## グローバル設定の継承
`~/.claude/CLAUDE.md` の全ルールを継承する（plan起点開発・全制作物のCodexレビュー必須/2周上限・itshover.com モノクロ線アイコン・CSS値ハードコード禁止/CSS変数使用・認知負荷を最小にする応答・コードは綺麗/シンプル/高速）。本ファイルは上記と重複しないPJ固有事項のみを記載する。UI 変更時は同ディレクトリの `DESIGN.md` を必ず先に読む。

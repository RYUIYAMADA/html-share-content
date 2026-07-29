---
project: html-share-content（Keyboard Layout Studio）
version: 0.22.0
inherits: ryuiyamada-design-system（グローバルDS）
updated: 2026-06-18
---

# DESIGN.md — Keyboard Layout Studio

> Claude Code / Codex が `keyboard-editor.html` の UI を触るとき**毎回最初に読む**設計契約。
> グローバルDS（`~/Desktop/ryui-workspace/projects/tools/ryuiyamada-design-system/`）を継承し、
> **このプロジェクト固有の差分だけ**を書く。global と矛盾する時はこのファイルが優先。
> 値はすべて `keyboard-editor.html` の `:root` に実在するもの（実態ベース）。

## 1. このプロダクトは何か
- 何をするものか: MacBook Pro 16" JIS 配列の Karabiner-Elements 設定を、3パネル（左=レイヤー/コンボ/機能・中央=実物大キーボード・右=設定）で視覚的に編集するエディタ。
- 主な利用者: 龍偉本人（自分のキーボード配列を継続調整する個人ツール）。
- 利用デバイス/環境: PC（MacBook）ブラウザ。`overflow:hidden` の全画面アプリ前提でスクロールしない1画面設計。
- トーン: 落ち着いた業務用ツール。紙のような温かみ（オフホワイト背景）＋黒の硬いアウトラインで「実機の設定盤」感を出す。

## 2. デザイン原則（守るべき判断軸）
1. 実機を見たまま編集できる — 中央のキーボードは実物大・JIS配列準拠。抽象的なリストでなく「キーそのもの」を触らせる（理由: 配列調整は空間記憶で行うため）。
2. 1画面完結・スクロールさせない — `fitRowHeight()` で行高を可変に詰める。情報を増やすときも縦に伸ばさず密度で吸収する（理由: 配列全体を常に俯瞰したい）。
3. 紙＋黒アウトライン＋オレンジ1点強調 — 多色を使わず、強調は `--accent`（オレンジ）1色に限定（理由: 機能が多くてもノイズを出さない）。
4. グレー枠＝長押し の意味付けを崩さない — `--hold` のグレー枠は「長押し挙動」を表す決まった記号。色や枠線の意味を勝手に再利用しない（理由: 既存ユーザーの読み取りが壊れる）。
5. データ破損で白紙にしない — UI は `sanitizeData()` を通った安全な data を描画する前提（理由: 設定ツールは起動不能が致命的）。

## 3. トークン（`:root` 実在値・必ず変数参照）
素のCSS値でハードコードせず、必ずこの変数を使う。設定パネルでテーマ変数（`--key-bg` 等）は上書きされる前提。
```css
/* Surface / Color */
--paper:        #F5F2EB;   /* アプリ背景（紙） */
--panel:        #FFFFFF;   /* パネル・カード面 */
--stage:        #D8D5CE;   /* キーボードステージのグレー背景 */
--text:         #1A1A1A;   /* 本文・主テキスト */
--muted:        #5F6470;   /* 補助テキスト */
--faint:        #6B7280;   /* 弱テキスト（AA 4.83:1） */
--line:         #E3E0D8;   /* 通常の罫線・枠 */
--line-strong:  #1A1A1A;   /* 強い枠（アクティブ要素のアウトライン） */
--hold:         #706D64;   /* グレー枠=長押し（AA 4.7:1）※意味固定 */
--key-dim:      #767676;   /* 未割当ラベル（AA 4.54:1） */
--accent:       #FF8C42;   /* 唯一の強調色（タブ下線・選択・dirty表示） */
--focus:        #2196F3;   /* フォーカスリング */
--ok:           #2E7D32;   /* 保存OK・書込ボタン系 */
/* テーマ上書き変数（設定パネル） */
--key-bg / --key-text / --groove / --stage-bg / --ref-color / --key-font-family
/* Layout */
--rowh: 54px;              /* キー行の基準高（fitRowHeightで動的調整） */
--shadow: 2px 2px 0 0 var(--line-strong);  /* 硬い実体影（ぼかさない） */
/* Font */
--font: -apple-system, BlinkMacSystemFont, "SF Pro Display", "Helvetica Neue",
        "Hiragino Kaku Gothic ProN", "Noto Sans JP", sans-serif;
```
本文 `font-weight: 600` がベース（やや太め）。font-weight 300 以下は使わない（global準拠）。

## 4. コンポーネント規約（状態まで）
- ボタン `.tbtn`: 高さ34px・`1.5px solid var(--line)`・角丸8px。hover で `--line-strong` 枠。`disabled` は opacity .4。
  - `.primary`（黒地白文字）= 主要アクション。`.write`（緑地）= Karabiner書込。`.write.off` = サーバ未起動の控えめ表示。1画面に primary を乱立させない。
- 左タブ `.ltab`: 選択中は `border-bottom-color: var(--accent)`（下線オレンジ）＋テキスト濃色。それ以外で accent を下線以外に流用しない。
- 行 `.layer-row` / カード `.combo-item`: 選択は白地＋`--line-strong`枠＋`--shadow`。hover は薄い紙色（`#FAF8F3`/`#FFFFFF`）。削除ボタン hover のみ赤（`#c0392b` / `#fbeaea`）＝破壊操作の例外色。
- 入力 `.miniform input` / `.cfg input`: 高さ34px・角丸8px・`1.5px solid var(--line)`。textarea は可変高 `resize: vertical`。
- 保存インジケータ `.saved`: 緑ドット=保存済 / `.dirty` でオレンジ（`--accent`）＝編集中。

## 5. レイアウト規約
- 本体グリッド `.body`: `252px 1fr 312px`（左固定・中央可変・右固定）。`min-height:0` でパネル内スクロール。
- アプリ全体 `grid-template-rows: auto 1fr`・`height:100vh`・`overflow:hidden`（ページ自体はスクロールしない）。
- 情報密度: 高密度。理由は §2-2（配列全体を常に俯瞰）。余白で逃がさず行高動的調整で収める。
- レスポンシブ・ブレークポイント: 現状なし（PC全画面専用・未確定/モバイル対応は要確認）。

## 6. 禁止ルール（anti-pattern）
- 色・余白・フォントサイズを変数でなく素の値でハードコード → 禁止（`:root` 変数を使う）。
- 強調色を `--accent` 以外に増やす・accent をタブ下線/選択/dirty 以外の装飾に拡散 → 禁止。
- `--hold` のグレー枠を「長押し」以外の意味で再利用 → 禁止（記号の意味が壊れる）。
- グラデーション・glassmorphism・円グラフ・カードの装飾border・font-weight 300以下 → 禁止（global準拠）。`--shadow` はぼかさない硬い実体影で統一（`box-shadow` をぼかし影に変えない）。
- 外部CDN・外部フォント・外部画像・外部スクリプトの追加 → 禁止（自己完結HTMLを死守）。
- UIラベルに絵文字を使う場合は通知ダイアログ（`alert`）内に限定。キー/ボタンのラベル装飾としての絵文字は増やさない。

## 7. アクセシビリティ（必須ライン）
- コントラスト WCAG AA（本文4.5:1・大）。`--faint`/`--hold`/`--key-dim` は AA 達成済み（コメントに比率明記）＝新色追加時も AA を満たす。
- フォーカス可視（`--focus` リング）。タブは `aria-selected`、トグルは `aria-pressed` を維持。
- タップ/クリックターゲットは概ね34px。新規操作要素は最小 44×44px を目安（global準拠）。
- 文言は日本語。文節改行・禁則処理は global 準拠。

## 8. Do / Don't
| ✅ Do | ❌ Don't |
|---|---|
| `color: var(--accent)` で強調 | `color: #FF8C42` と直書き |
| 強調はオレンジ1色に絞る | 機能ごとに別の色を割り当てる |
| グレー枠=長押しの意味を保つ | グレー枠を別の状態表現に流用 |
| `pushHistory()`→`commit()` 経由で更新 | data を直接書き換えて再描画だけ |
| 単一HTMLに収める | 機能追加でCDN/npm依存を持ち込む |

## 9. AI（Claude/Codex）への指示
- UI実装前に必ずこのファイルと global DS、そして `CLAUDE.md` を読む。
- トークンは変数参照（素の値禁止）。§6 違反は自己修正。
- 「読む負担を感じさせない、みてわかるレイアウト」を全画面のデフォルト前提にする（ただし§2の高密度1画面方針と両立させる）。
- 迷ったら §2 の原則で判断。決まらなければ実装を止めてPMに質問（推測でデザインを増やさない）。

## 📜 更新履歴
- 2026-06-18 — 初版。`keyboard-editor.html` v0.22.0 の `:root` 実在トークン・3パネル構成・長押しグレー枠の意味付けを設計契約化。

# SKILL: なのばなな（Gemini API）でまとんが絵を生成する手順

確立: 2026-08-12（紙芝居『浅井長政ものがたり』v02 全12枚で実証済み）

## 前提
- 水野さんのGoogleアカウントで AI Studio（aistudio.google.com/apikey）から発行した
  **Gemini APIキー**を使う。キーは `AQ.` または `AIza` で始まる。
- **キーはこのファイルにもリポジトリにも絶対に書かない**（GitHubトークンと同じ扱い）。
  クラウドセッションでは scratchpad 配下の `.gemini_key`（chmod 600）にのみ保存。
  コンテナが作り直されたら水野さんにチャットで再提供を依頼する。
- Geminiアプリの有料会員（AI Pro）とAPIは別枠。APIはキー発行だけで使える（無料枠あり）。

## 使えるモデル（2026-08-12時点で確認）
- `gemini-3.1-flash-image` … 標準。今回の紙芝居はこれ（1枚10〜20秒・高品質）
- `gemini-3-pro-image` … さらに高品質が欲しいとき
- `imagen-4.0-generate-001` 系 … Imagen系

## 呼び方（curl）
```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image:generateContent?key=＜キー＞
Content-Type: application/json
{"contents":[{"parts":[{"text":"＜日本語プロンプト＞"}]}],
 "generationConfig":{"responseModalities":["TEXT","IMAGE"],
                     "imageConfig":{"aspectRatio":"16:9"}}}
```
- 返答の `candidates[0].content.parts[].inlineData.data`（base64）が画像（JPEG）。
- このセッションのプロキシは generativelanguage.googleapis.com に**通る**（実証済み）。

## 品質のコツ（今回の学び）
- 画風の統一指定が効く: 「和風絵本・紙芝居風の水彩画、墨の輪郭線、横長16:9、文字なし、
  **金色は一点だけ**（何に使うかを毎回指定）」→ 12枚がシリーズとして揃った。
- キャラの一貫性は**毎プロンプトに容姿の全説明を繰り返す**（紺の鎧・亀甲家紋・細面など）。
  さらに揃えたい場合は1枚目を参照画像として `parts` に画像を添付する。
- 失敗やイメージ違いは修正より**同じプロンプトで引き直し**が早い。
- 「文字なし」を必ず入れる（入れないと画面に文字が描かれることがある）。

## 組み込みパイプライン（紙芝居v02で確立）
1. プロンプト集MD（【第◯場】ブロック形式）を作る
2. Pythonループで全场面を生成 → scene01.png〜
3. PILで width1280・JPEG q80 に圧縮 → base64のdata URIにしてHTMLへ埋め込み
4. `<img id="stage">` 差し替え方式（v01のSVGはそのまま予備として残す）
5. Playwrightで全ページ表示テスト → 出荷
- 12枚で合計約3.8MB（1枚200〜300KB）。10枚前後ならこの方式で問題なし。

## 応用先
- 紙芝居シリーズの新作（三献の茶、お市と三姉妹…＝紙芝居本屋さんゲームの実在化）
- ゲームの背景・カード絵、みどりくんのキャラデザイン案出し
- 造園ニュースアニメのカット素材（ffmpeg紙芝居方式と直結）

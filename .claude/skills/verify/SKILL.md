---
name: verify
description: shippo-chigiri（静的HTML Canvasゲーム）の動作確認手順。ローカルサーバー + ヘッドレスChrome(CDP)で実プレイして検証する。
---

# shippo-chigiri の動作確認

単一 `index.html` の静的ゲーム。ビルド不要。Node は環境に無いので、
Python3 + websocket-client で Chrome headless を CDP 制御する。

## 起動

```bash
cd /Users/hiro/shippo-chigiri && python3 -m http.server 8931 &   # assets/ を相対パスで読むためHTTP必須
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --remote-debugging-port=9333 \
  --user-data-dir=<scratch>/chrome-profile --window-size=900,600 about:blank &
```

- ws接続は `websocket.create_connection(url, suppress_origin=True)` — Origin付きだと403。
- タブ一覧: `http://localhost:9333/json`（`type=="page"` を選ぶ）。

## 操作のコツ

- `<script>` トップレベルの `let/const`（`game`, `penguins`, `grab`, `VARIANTS`,
  `spawnPenguin`, `tailPos`, `penguinAlpha` など）は `Runtime.evaluate` から直接参照できる。
- スタート/リトライはボタンの `getBoundingClientRect` 中心へ
  `Input.dispatchMouseEvent`（mousePressed→mouseReleased）。JS `click()` でも可。
- **しっぽちぎり**: `tailPos(p)` の座標へ mousePressed → `grab.active` を確認 →
  40ms間隔ループで毎回 `tailPos`/`p.angle` を再取得し、
  `tail - 170*(cos(p.angle), sin(p.angle))` へ mouseMoved（逃走方向の逆へ引く）。
  220px以上引き離すとすっぽ抜けるので距離は170px程度に保つ。約1秒でちぎれる。
- **警戒AI**: ペンギンから400px離れた点から55px刻みで高速 mouseMoved
  （距離<160 かつ pointer.speed>350 で `keikai`）。
- レア色を確実に出すには console から `spawnPenguin(VARIANTS.red)` 等
  （自然出現待ちは白3%で非現実的）。寿命/点滅は `penguinAlpha(p)` をサンプリング。
- 60秒待たずに終了を見るには `game.time=3`。ベストは localStorage キー
  `shippo-chigiri-best`。
- 図形フォールバックは `Network.setBlockedURLs ["*assets/*"]` → reload →
  `spritesReady===false` を確認。

## 確認すべき既存仕様

警戒AI(！マーク) / 張力ドラッグ(ゲージ・すっぽ抜け) / 60秒タイマー /
localStorageベスト / スプライト失敗時の図形フォールバック / 凡例(画面左下) /
カラーバリアント(黒60%・赤25%・青12%・白3%、寿命∞/8/6/4秒、点滅1.5秒前→フェード)

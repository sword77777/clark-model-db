# CLARK MODEL 官網 — 專案說明與修改規格

> 版本：v1.0（2026-11-08）
> 適用對象：要接手修改這個網站的人（改文案、加產品、加頁面、加戰況欄位、換部署）
> 原則：**這份文件描述的是 `index.html` 的實際寫法**，不是理想架構。看到「現況」就以現況為準，不要邊改邊假設。

---

## 1. 專案基本資料

| 項目 | 內容 |
|---|---|
| 網站名稱 | CLARK MODEL |
| 線上網址 | https://sword77777.github.io/clark-model-db/ |
| GitHub repo | `sword77777/clark-model-db`（分支 `main`，根目錄） |
| 托管方式 | GitHub Pages（legacy build，source = `main` / `/`） |
| repo 可見性 | **必須 public**。改成 private 會讓 Pages 整個消失（網址 404），免費帳號不支援 private repo 發 Pages |
| 交付形態 | 單一 `index.html`（約 50 KB），CSS 與 JS 全部內嵌，無框架、無建置流程 |
| 外部依賴 | MQTT.js 5.10.1，CDN：`https://fastly.jsdelivr.net/npm/mqtt@5.10.1/dist/mqtt.min.js` |
| 語言 | 介面英文為主（有少量中文殘留，見 §11.6） |

**網址結構**：`https://<帳號>.github.io/<repo>/`，本專案即 `sword77777.github.io/clark-model-db/`。

---

## 2. 檔案結構與部署

```
clark-model-db/
├── index.html                      ← 整個網站（HTML + CSS + JS 都在這一個檔）
├── assets/
│   ├── bannerE.bmp                 ← 首頁大 banner（選語言／選類別頁）
│   ├── logos.bmp                   ← 左上角小 logo
│   ├── prod_ID_all.bmp             ← Online Store 的產品編號說明圖
│   └── square_w.bmp                ← 左側產品清單前面的小方點（CSS background）
├── downloads/
│   └── TankSim_Standalone.exe      ← 單機版坦克模擬器（約 10 MB）
└── .gitignore
```

**部署流程（沒有 CI，就是 push）**

1. 改 `index.html`（或加 `assets/`、`downloads/` 檔案）
2. `git add` → `git commit` → `git push origin main`
3. GitHub Pages 自動重建，約 **15～60 秒**生效
4. 驗證：`curl -s -o /dev/null -w "%{http_code}" https://sword77777.github.io/clark-model-db/` 要拿到 `200`

**注意**：Pages 會快取，剛 push 完可能還是舊版。用無痕視窗或加 `?v=時間戳` 再看。

---

## 3. 整體架構（單檔四層流程）

網站不是多頁面，而是**同一個 `index.html` 裡的四個「視圖」靠 class 切換**。看懂這點，所有修改都不會亂。

```
body.pre ──┬─ #lang   選語言（全屏）
           └─ #cat    選類別（全屏）
                        │
                        ▼  （以下 body 不含 .pre，banner + 左側清單出現）
           #mainv   主站（Tank 類別）：hero / #detail / #store / #download
           #battle  戰況頁（黑底綠字儀表板）
```

- 視圖切換只有一個入口：`show(view)`（**L438**），用 `classList.toggle("on")` 開關 `#lang` / `#cat` / `#mainv` / `#battle`。
- `body.pre` 時，`#banner` 與 `#products` 被 CSS 隱藏（**L23**）。
- 每個 `<section>` 預設 `display:none`，只有 `.on` 才顯示（**L88-89、L104-105、L116-117**）。
- 主站裡的三個子頁（產品說明 / Online Store / Download）再靠 `showSubview(name)`（**L465**）切 `#detail` / `#store` / `#download` 的 `display`。

**流程細節**

| 動作 | 結果 | 位置 |
|---|---|---|
| 點 `[data-lang="en"]` | 進 `#cat` | L452-456 |
| 點其他語言 chip | toast「coming soon」，留在原地 | L454 |
| 點 `[data-cat="tank"]` | 進 `#mainv` | L458-462 |
| 點其他類別 chip | toast「Category coming soon」 | L460 |
| 點左上小 logo | 回首頁 `#lang` | L483-485 |
| banner 的 Battle 鈕 | 進 `#battle`，並隱藏左側清單 | L445、L478 |

---

## 4. 視覺規格（設計 token）

### 4.1 色彩（`:root`，**L11-15**）

| Token | 值 | 用途 |
|---|---|---|
| `--bg` | `#000000` | 全站底色（純黑，不用 #111 那種灰黑） |
| `--panel` | `#0a0a0a` | 側欄、卡片底 |
| `--panel2` | `#111` | modal 底、hover 底色 |
| `--border` | `#2a2a2a` | 所有 1px 分隔線／框線 |
| `--text` | `#ffffff` | 主文字 |
| `--dim` | `#9a9a9a` | 次要文字、說明、單位 |
| `--link` | `#8080ff` | 一般連結 |
| `--blue` | `#4a7dff` | 主強調色：按鈕、選中、BLUE 隊、連線 OK |
| `--blue-dim` | `#2a4a8a` | 藍色邊框的弱化版 |
| `--green` | `#00ff41` | RED/BLUE 隊的「正常」狀態、鎖定 OK、HP 正常 |
| `--red` | `#ff2a2a` | RED 隊標題、Lost、destroyed、斷線 |
| `--yellow` | `#ffd23e` | damaged、RSSI 中等、星號 ★ |

**用色規則**

- 純黑底 + 純白字是基調，**不要**改成深灰或米白。
- 藍 `#4a7dff` 是唯一的主動元素色（按鈕、選取、連線成功）。
- 紅／綠只用在「狀態語意」（隊別、HP、鎖定），不拿來當裝飾。
- 灰階只用 `#0a0a0a / #111 / #161616 / #222 / #2a2a2a / #444 / #6a7a94 / #9a9a9a` 這幾階，別新增沒必要的灰。

### 4.2 字型與字級

- 本體：`Verdana, Tahoma, 'Segoe UI', Arial, sans-serif`（**L17**）
- 側欄／戰況／明細面板：改成 `Tahoma, Arial, sans-serif`（L34、L41、L48、L59、L116、L158、L161）
- **戰況頁刻意用小字（10px）**：這是「儀表板」風格，不是沒調好，別順手放大。
- 字級分佈：

| 字級 | 用途 |
|---|---|
| 40px / letter-spacing 3px | 首頁大標（Select Language / Category） |
| 24px | 主站 hero `h2` |
| 20px | `#detail` 產品標題、左上 logo |
| 16px | 區塊小標、chip 按鈕 |
| 13-14px | 說明文、team 標題、表格 |
| 12px | nav 按鈕、麵包屑路徑 |
| 10-11px | 戰況表、Tank Detail、modal 欄位（刻意最小） |

- 標題多用 `letter-spacing:1~2px`；nav 用 `text-transform:uppercase` + 12px（**L29**）。

### 4.3 間距與框線

- 全站 `box-sizing:border-box`，`margin/padding` 歸零重設（**L10**）。
- 面板內距：側欄 `10px 0`＋每列 `5px 16px`；`#detail` `28px 34px`；`#battle` `12px 14px`；modal `18px`。
- 圓角：卡片／modal 6-8px、按鈕 4px；**表格與戰況列不圓角**。
- 框線一律 1px `var(--border)`；強調用 2px（team 標題底線 `border-bottom:2px solid`）。

### 4.4 元件樣式（照這幾種用，不要自創第五種按鈕）

| 元件 | 長相 | 用途 | 位置 |
|---|---|---|---|
| `.chip` | 面板底、1px 邊框、14px 26px 內距、min-width 150px | 首頁選語言／選類別 | L99-101 |
| `.pill` | 高 30px、膠囊感（4px 圓角）、10px 字 | 戰況狀態標籤 | L119-122 |
| `.bigbtn` | 藍底白字、無邊框、30px 高 | 進行中的主要動作（Pause、Respawn 倒數） | L124-125 |
| `.ghost` | 透明底、1px 邊框、淡藍字 | 次要動作 | L126-128 |
| `.setbtn` | 細邊框小按鈕 30px 高 | Tank Detail 的 Set 系列 | L78-79 |
| `.dlbtn` | 藍底圓角 4px | 下載按鈕 | L54-55 |
| `.tag` | 1px 內距、只用顏色不用底色 | HP 狀態（normal/damaged/destroyed） | L144-147 |
| `#toast` | 置底居中、1.6s 淡出 | 所有「coming soon」與錯誤提示 | L168-169、L435 |
| `#modal` | 全屏半透明遮罩 + 440px 卡 | 設定 TK-LINK | L155-166 |

### 4.5 版面尺寸

| 區塊 | 寬／高 |
|---|---|
| 左側產品清單 `#products` | 250px 固定，`overflow-y:auto` |
| Tank Detail `#tdetail` | 320px 固定，可收合（`max-width` 過渡 0.35s） |
| 戰況表格 | grid `1.2fr 1.5fr 1fr .8fr .8fr 1fr`（6 欄：Tank/Operator/HP-Ammo-MG/Status） |
| LOG 區 `#logwrap` | 高 110px，可捲動 |

### 4.6 響應式

只有一個斷點：`@media (max-width:760px)`（**L171-178**）

- `#app` 改縱向；側欄改成上面一條（max-height 200px）
- `#tdetail` 佔滿寬
- `#teams` 兩隊改成上下堆疊
- 戰況表頭 `.thead` 隱藏

**改版面時，這個斷點一定要一起看**，否則手機上會擠成一團。

---

## 5. 版面結構（DOM 骨架）

```
body.pre
├── header#banner            ← L183-195
│   ├── #brand > a.logo      （小 logo，點它回首頁）
│   └── #nav                 （8 個 <button data-nav="...">）
│         news / products / reference / project / contact / store / battle / download
└── #app                     ← L197
    ├── aside#products       （左側分類＋產品，**由 JS 產生**，非寫死在 HTML）
    ├── aside#tdetail        （Tank Detail，預設 .off）
    └── main#main
        ├── section#lang     L206
        ├── section#cat      L222
        ├── section#mainv    L237
        │   ├── .hero
        │   ├── #detail      產品說明（預設空）
        │   ├── #store       Online Store（產品編號表）
        │   └── #download    下載頁
        └── section#battle   L305
            ├── #connbar     狀態 pill + 6 顆操作鈕
            ├── #teams       兩隊（RED/GREEN）
            └── #logwrap     LOG
#modal                        L327（TK-LINK 設定）
#toast                        L337
```

**nav 的對應邏輯（L471-480）**

| `data-nav` | 行為 |
|---|---|
| `battle` | `show("battle")` |
| `store` | `show("mainv")` + `showSubview("store")` |
| `products` | `show("mainv")` + 回到產品說明 |
| `download` | `show("mainv")` + `showSubview("download")` |
| `news` | 回產品頁 + toast「NEWS - coming soon」 |
| `reference` / `project` / `contact` | 只 toast「coming soon」，不做任何切換 |

新增一個 nav 項目時，這張表就是你要改的地方（HTML 加 button、JS 加分支、必要時加 section）。

---

## 6. 內容資料結構：產品清單

左側清單**不是 HTML**，是 JS 的 `CATEGORIES` 陣列（**L345-418**）跑出來的（**L421-431**）。

```js
var CATEGORIES = [
  { name:"1/16 RC Tank Controller", items:[
    ["TK20 series", "prod_rx20/index.html"],   // [顯示名稱, 相對路徑]
    ["DBF-HUB", ""]                            // 空字串 = coming soon（點擊只 toast）
  ]},
  ...
];
```

規則：

- `name` → 左側的粗體分類標題（`<h4>`）
- `items[i][0]` → 清單文字；`items[i][1]` → 點擊後顯示的路徑
- 路徑是**相對於網站根目錄**，例如 `prod_rx20/index.html` → `https://…/clark-model-db/prod_rx20/index.html`
- 路徑留空 → `showProduct()` 只跳 toast「<名稱> - coming soon」（**L489**）
- 目前有路徑的產品，點下去只會顯示「此產品說明頁正在建置中」占位（**L488-494**），**並不會真的把子站載進來**

> ⚠ **現況重點**：repo 裡**沒有** `prod_rx20/` 這些子目錄。清單上的路徑指向不存在的資料夾，直接開會 404。要嘛把舊站子目錄整批搬進 repo，要嘛把這些路徑改成新的錨點／空字串（見 §11.3）。

---

## 7. 戰況頁 MQTT 規格（改功能最常碰的地方）

### 7.1 連線參數

```js
var cfg = { broker:"broker.emqx.io", port:"8083", battle:"Battle01", key:"clark1234" };  // L502
```

- 走 WebSocket：`ws://broker:8083/mqtt`；**https 站台自動改 `wss://…:8084/mqtt`**（L519-522）
- clientId：`clarkweb_` + 6 位隨機（L523），避免多分頁互相踢線
- 斷線自動重連：`reconnectPeriod:3000`
- 分頁切回前景時，若斷線會強制重連（L537-543）

### 7.2 Topic 規範

| 方向 | Topic | 說明 |
|---|---|---|
| 上行（網站訂閱） | `battle/<battle>/<team>/<tankId>` | `<team>` = `red` 或 `green`；網站訂閱 `battle/+/red/+`、`battle/+/green/+`（L527） |
| 下行（網站發布） | `battle/<battle>/<team>/<tankId>/cmd` | JSON 指令（L757） |
| 廣播（Respawn） | `battle/<key>/<team>/broadcast/cmd` | ⚠ 這一段用的是 `cfg.key` 而不是 `cfg.battle`（L803），是既有行為，改之前要跟固件端確認 |

`onMessage()`（**L545-557**）：只收 4 段 topic、只認目前 `cfg.battle`、只認 red/green；**空 payload = 該坦克離線，整筆刪掉**（L551）。

### 7.3 上行資料欄位（坦克 → 網站）

網站用到／顯示的欄位（`tanks[team][id]` 就是這包 JSON）：

| 欄位 | 用途 |
|---|---|
| `DisplayName` | 顯示名稱（沒有就退回 tankId），也可被 SetDisplayName 指令改 |
| `Operator` | 操作者名稱 |
| `HP` / `MaxHP` | HP 條與勝負判定（`parseFloat`，非數字視為 0 / 100） |
| `Status` | `normal` / `damaged` / `destroyed`（不分大小寫比對） |
| `Ammo` / `MG` | 彈藥、機槍狀態文字 |
| `Stars` / `Level` | 顯示 ★N 與 Lv.N，也會被指令改 |
| `IR_ID` | 紅外線 ID |
| `RF_lock` / `TK_lock` | RF／TK 鎖定，顯示 `Locked`（綠）或 `Lost`（紅） |
| `rssi` | WiFi 訊號 dBm：**≥ -60 綠 / ≥ -80 黃 / 其餘紅**（L717-721） |
| `BattleName` | 有值才顯示 |

要塞新欄位：**加在 `renderDetail()`（L713-752）的 `dgrid` 裡**，並確認固件真的會送，不然會顯示 `-`。

### 7.4 下行指令（網站 → 坦克）

包裝函式 `sendTankCmd(team, id, command, key, val)`（**L753-760**）發布：

```json
{ "Command": "<指令名>", "<參數名>": <值> }
```

現有指令：

| 指令 | 參數 | 觸發位置 |
|---|---|---|
| `SetDisplayName` | `DisplayName`（非空字串） | Tank Detail 的 Set Name |
| `SetOperator` | `Operator`（非空字串） | Set Operator |
| `SetStars` | `Stars`（整數 ≥0） | Set Stars |
| `SetLevel` | `Level`（整數 1-99） | Set Level |
| `AwardStars` | `Stars`（每人點數，由賽末公式算出） | 自動頒獎 |
| `RespawnStart` / `RespawnStop` | `Value:"FULL"` | Global Respawn（10 次 burst，約 5 秒） |

**安全設計（重要，不要拿掉）**

```js
var SEND_DISABLED = (location.protocol==="https:" || location.protocol==="wss:");  // L503
if(SEND_DISABLED){ cfg.key=""; }                                                  // L504
```

線上站（https）就清空 battle key，所有 `sendTankCmd` / Respawn 直接拒絕並在 LOG 印 `Send commands disabled on this site.`（L754）。**本機用 `file://` 或 `http://` 開才會送指令。** 這是刻意的：公開網站不該能遙控別人家的坦克。

---

## 8. 對戰規則（純前端，沒有後端）

| 按鈕 | 規則 | 位置 |
|---|---|---|
| **Start Battle** | 兩隊都必須至少有一台坦克，否則報錯不啟動；啟動後計時器從 00:00 重新跑 | L569-575 |
| **Pause Battle** | 第一次按＝暫停（累積時間、按鈕變 `bigbtn`），再按＝續跑 | L576-591 |
| **End Battle** | 手動結束，判定順序：① 一隊全滅 → 另一隊勝 ② 存活台數多者勝 ③ 平手時比「存活坦克 HP 總和」④ 仍相同＝平手（無獎） | L593-610 |
| **自動結束** | 每次收到 telemetry 就檢查，任一方全滅即自動結束 | L611-619 |
| **Reset Arena** | 清空兩隊資料、關閉 Detail、等新資料 | L811-816 |
| **Global Respawn** | 連送 10 次 `RespawnStart`（每 0.5s），結束送 2 次 `RespawnStop`，按鈕顯示 5→1 倒數 | L790-803 |

**賽末給點公式（`endBattle`，L620-647）**

```
每人點數 = round( 勝方所有坦克 (HP/MaxHP*100) 的平均值 / 100 * 10 )
```

例：勝方 3 台，HP 百分比 80%、50%、20% → 平均 50% → 每人 5 點（`AwardStars` 逐台發布）。
兩隊都全滅 → 平手，不頒獎。

---

## 9. 常見修改任務 SOP

### 9.1 改文案、改顏色、改間距
1. 只動 `index.html`
2. 顏色改 `:root`（L11-15）不要散落改 hex
3. 改完在 760px 斷點下也看一次

### 9.2 新增一個產品（左側清單）
1. 找對應 `CATEGORIES` 分類（L345-418）
2. 加一筆 `["名稱","路徑"]`；路徑沒有就先給 `""`（點擊會 toast）
3. 不需要動 HTML、不需要動 CSS

### 9.3 新增分類
同上，在 `CATEGORIES` 加一個 `{name, items}` 物件即可，左側會自動長出標題。

### 9.4 新增一個 banner 頁籤（例如 News 真的做出來）
1. `#nav` 加 `<button data-nav="news">News</button>`（L185-194 區塊）
2. `#main` 加 `<section id="news">…</section>`，預設 `display:none`、`.on` 時顯示
3. `show()`（L438）加一行 toggle
4. nav 點擊邏輯（L471-480）把 `news` 的 toast 改成 `show("news")`
5. CSS 加 `#news` 的樣式，沿用 `--panel` / `--border` / 13px 內文

### 9.5 新增一個「戰況」欄位（例如 Temperature）
1. `renderDetail()` 的 `dgrid` 加一行 `<b>Temp</b><span>…</span>`（L722-739 之間）
2. 值的格式化照 `f()` / `lockHtml()` / `rssiHtml()` 的寫法，別直接在字串裡塞 `t.xxx`
3. 若也要出現在列表列（`.trow`），改 `row()`（L667-684）與 `.thead`（L319-320）的欄位數
4. `grid-template-columns`（L137）要跟著欄數調整

### 9.6 新增一個控制指令
1. 確認固件端已支援該 `Command`
2. 用 `sendTankCmd(team, id, "NewCmd", "Param", value)` 發布，不要自己拼 topic
3. UI 入口放在 `#connbar`（全域指令）或 Tank Detail（單台指令），沿用 `.ghost` / `.setbtn`
4. 別忘了 `SEND_DISABLED` 的檢查（`sendTankCmd` 內已含，走它就自動有）

### 9.7 換 broker／換戰場
- 預設值改 `cfg`（L502）
- 使用者端仍可用 **Settings** 彈窗（L818-829）臨時改，改完會重連

### 9.8 換 banner 圖／logo
- 檔名維持 `assets/bannerE.bmp`、`assets/logos.bmp`（覆蓋即可，不用改 HTML）
- 換格式（例如 png）就得同步改 `src`（L184、L207、L223）與 CSS `url()`（L39）
- **路徑大小寫要完全一致**，GitHub Pages 是 Linux，大小寫錯就 404

### 9.9 新增下載檔
- 檔案丟 `downloads/`，在 `#download` 區塊複製一張 `.dlcard`（L286-292）換檔名與說明
- 大檔案會讓 repo 變胖，GitHub 單檔上限 100 MB

---

## 10. 部署與驗證

```powershell
# 1) 確認狀態
git status
# 2) 提交（訊息寫清楚改了什麼）
git add index.html assets downloads
git commit -m "site: <改動摘要>"
# 3) 推上去（main）
git push origin main
# 4) 等 15-60 秒後驗證
curl.exe -s -o NUL -w "code=%{http_code}`n" -I https://sword77777.github.io/clark-model-db/
```

驗證要點：

- `code=200`
- 打開頁面確認：① 首頁選語言畫面在 ② 點 English → 選類別 ③ 點 Tank → 主站 ④ 點 Battle → 連線 pill 變 `Connected`
- `Connected` 但 BM 顯示 `Disconnected` → 檢查 broker / port / 是否 https（https 只能 wss:8084）

---

## 11. 已知限制與地雷（改了會出事的地方）

### 11.1 repo 不能轉 private
GitHub Pages 在免費帳號下不支援 private repo。**一轉 private，網址立刻 404**（Pages 設定會被整個刪掉，不是只暫停）。
要恢復：repo 改回 public → 重新 `POST /repos/{owner}/{repo}/pages`（source `main` / `/`）→ 等 build。

### 11.2 https 不能送指令
線上站一律 `SEND_DISABLED`，Set／Respawn 全部失效（這是設計）。要測指令請本機開檔。

### 11.3 產品清單的路徑目前是「空指向」
`prod_rx20/index.html` 這類路徑在 repo 裡不存在（見 §6）。點下去會 404，不是壞掉，是還沒搬。要修就二選一：搬子站進來、或改 `""`。

### 11.4 `#products.off` 重複寫了兩次
L35-36 有兩行一樣的 `#products.off{display:none}`。無害，但要清理時不要只刪一行就以為改到了。

### 11.5 `sendRespawn()` 用 `cfg.key` 當 battle 段
L803 的 topic 是 `battle/<key>/…/broadcast/cmd`，不是 `battle/<battle>/…`。看起來像 bug，但**固件端可能就是這樣對的**，改之前必須跟 SLK03／TK55 端核對，不要自己「修正」。

### 11.6 介面有中文殘留（違反全英文規則）
目前仍有中文句子：

- L240 hero：`…kits. 點左側分類查看產品說明。`
- L243 `#detail` 空狀態：`請在左側選擇產品分類與產品，此處顯示產品說明。`
- L492 產品占位：`此產品說明頁正在建置中（下一階段）…`

規則：**Web 介面只用英文**。接手的人應該把這三處改成英文（例如 `Select a category on the left…`）。

### 11.7 其他小坑
- **單檔 50 KB**：再加功能會繼續長，超過約 100 KB 要考慮拆檔（但拆檔就失去「一個檔就是一個站」的好處，先想清楚）。
- **`var` + IIFE**（L340-341）：沒有箭頭函式、沒有 `let`，照這個寫法加程式碼，別混進 ES6 造成風格斷裂。
- **LOG 沒有上限**：`logLine()` 一直 append，長時間開著會愈來愈吃記憶體。要長期掛電視的話，建議加個上限（例如超過 200 行移除最舊）。
- **bmp 資產**：`assets/*.bmp` 是舊站沿用，瀏覽器可顯示，但體積大（banner 226 KB）。要換 png/webp 記得同步改 `src` 與 CSS `url()`。
- **`downloads/TankSim_Standalone.exe` 在 repo 裡**：10 MB，每次 clone 都要抓；換版時直接覆蓋同名檔。

---

## 12. 交付前檢查清單

改完網站，逐項打勾再 push：

- [ ] 首頁（選語言）→ English → 選類別 → Tank → 主站，四步都走得通
- [ ] 左上小 logo 點一下能回首頁
- [ ] 左側清單每個分類都展開、點擊有反應（不是死按鈕）
- [ ] Banner 八個頁籤都試過，該 toast 的有 toast、該切頁的有切頁
- [ ] Battle 頁：連線 pill 顯示 Connected、兩隊表格有資料、LOG 有訊息
- [ ] Tank Detail 點一台坦克會開、點空白處會關、Set 按鈕有回應
- [ ] 760px 以下（手機／窄視窗）版面不重疊、不溢出
- [ ] 沒有新增中文字串（全站英文）
- [ ] `curl` 拿到 200，且看到的是新版內容（必要時加 `?v=`）
- [ ] repo 仍是 public

---

## 附錄：常用定位索引（`index.html`）

| 想找 | 行號 |
|---|---|
| `:root` 色票 | 11-15 |
| 760px 響應式 | 171-178 |
| banner + nav | 183-195 |
| `#lang` 選語言 | 206-219 |
| `#cat` 選類別 | 222-234 |
| `#mainv` 主站 | 237-302 |
| `#store` 產品編號表 | 245-280 |
| `#download` 下載頁 | 281-301 |
| `#battle` 戰況 | 305-323 |
| `#modal` 設定 | 327-336 |
| `<script>` 開始（IIFE） | 339-340 |
| `CATEGORIES` 產品資料 | 345-418 |
| `show()` 視圖切換 | 438-449 |
| nav 點擊邏輯 | 471-480 |
| `showProduct()` | 488-494 |
| `cfg` 連線參數 / `SEND_DISABLED` | 502-504 |
| `connect()` | 516-535 |
| `onMessage()` | 545-557 |
| 對戰控制（Start/Pause/End/Award） | 559-647 |
| `render()` / `row()` | 649-685 |
| Tank Detail | 687-712 |
| `sendTankCmd()` | 753-760 |
| Set 系列（Name/Operator/Stars/Level） | 761-780 |
| Global Respawn | 789-803 |
| Settings 彈窗 | 818-829 |

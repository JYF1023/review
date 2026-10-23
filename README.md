# 考點筆記：給 AI 助理的說明

這份文件是寫給**協助使用者部署與維護這個 App 的 AI**（例如 ChatGPT）看的。使用者本人的操作說明在 `使用說明.md`。

## 這是什麼

一個準備資工所考試用的錯題與考點筆記 App，六科為：資料結構、演算法、計算機組織、作業系統、離散數學、線性代數。

- 純前端靜態網頁（PWA），**沒有後端、沒有外部相依、不呼叫任何 AI API**。
- 所有資料存在使用者 iPhone 瀏覽器的 **IndexedDB**，不會上傳到任何地方。
- 使用者用自己的 GPT 把題目照片轉成文字並判斷考點；App 負責歸類確認、查找、複習與備份。
- 主要用法是**批次匯入**：一次把十幾張題目照片丟給 GPT，GPT 回覆一大段固定格式的文字，貼回 App 後一題一題快速確認。另有單題新增模式。
- 題目以**文字**為主，圖用 ASCII 或文字結構表示；只有 ASCII 表達不了的圖才需要附照片。
- 目標裝置：**iPhone 的 Safari，加入主畫面後使用**。

## 檔案

| 檔案 | 用途 |
|---|---|
| `index.html` | App 本體，HTML、CSS、JS 全在這個檔案 |
| `manifest.webmanifest` | 讓 App 能加入主畫面 |
| `sw.js` | Service worker，讓 App 離線也能開；只快取 App 檔案，不碰筆記資料 |
| `apple-touch-icon.png`、`icon-192.png`、`icon-512.png` | 主畫面圖示 |
| `使用說明.md` | 給使用者看的操作說明（不需要部署） |

## 部署（第一次）

需要用電腦操作一次，大約 5 分鐘。之後所有使用都在 iPhone 上。

### 方法 A：Netlify Drop（建議）

1. 用電腦開啟 https://app.netlify.com/drop
2. 把**整個資料夾**（含 `index.html`、`sw.js`、`manifest.webmanifest`、三個 png）拖進頁面。
3. 上傳完成後會得到一個 `https://xxxx.netlify.app` 網址。
4. **立刻註冊或登入 Netlify，並認領（claim）這個網站。** 沒有認領的網站可能在短時間後被刪除。
5. 建議在 Site settings 把網址改成好記的名字，例如 `xxx-kaodian.netlify.app`。**改完之後就永遠不要再改**（原因見下方「最重要的限制」）。

### 方法 B：GitHub Pages

1. 建立一個 public repository，把上述檔案放在根目錄。
2. Settings → Pages → Source 選 main branch / root。
3. 得到 `https://<帳號>.github.io/<repo>/` 網址。

### 部署後，在 iPhone 上

1. 用 **Safari** 打開網址。
2. 分享按鈕 → 加入主畫面 → 加入。
3. 之後**只從主畫面圖示開啟**。

App 會偵測自己是不是從主畫面開啟；在 Safari 分頁裡開時，新增題目和還原功能會被鎖住並顯示教學。這是刻意的設計，不是 bug。

## 最重要的限制：資料綁在「網址」和「主畫面圖示」上

- IndexedDB 是依網域（origin）分開存的。**網址一改，App 就看不到舊資料。**
- iOS 上，主畫面 App 的資料和 Safari 分頁的資料是分開的兩份。**刪掉主畫面圖示，資料就一起刪掉。**
- 換手機、換網址、重裝之前，都必須先在 App 的「更多 → 備份」匯出備份檔，再到新環境「從備份還原」。

協助使用者時，任何會改變網址的操作（換平台、改網站名稱、換 repo 名稱）都要先提醒他備份。

## 更新 App（修改程式之後）

1. 修改 `index.html`。
2. 把 `sw.js` 裡的 `CACHE = 'kaodian-v3'` 版本號加一（例如改成 `kaodian-v4`），確保手機會拿到新版。
3. 部署到**同一個網站**：
   - Netlify：進入該網站 → Deploys → 把整個資料夾拖到頁面上的上傳區。
   - GitHub Pages：commit 並 push 到同一個 repo；或在網頁上 Add file → Upload files，上傳同名檔案就會覆蓋。
4. 使用者在 iPhone 上完全關掉 App 再重新開啟，就是新版。筆記資料不受影響。

## 驗收步驟

部署完成後，請使用者照著做一次。全部正常就代表部署成功：

1. 從主畫面圖示開啟，首頁沒有出現紅色的「先加入主畫面」提示。
2. 新增（批次匯入）→ 複製提示詞 → 貼到 GPT 並附上兩三張題目照片 → 點 GPT 回覆程式碼區塊的「複製」→ 回 App 按「貼上回覆」，出現「第 1 / N 題」。
3. 第一題選「會」，自動跳到第二題；第二題選「不會」、選錯誤類型、寫錯因，按「存檔，下一題」。
4. 回首頁搜尋剛剛的考點名稱，找得到。
5. 更多 → 製作備份檔 → 儲存備份檔，跳出 iPhone 分享選單，能存到「檔案」。

## 資料結構

IndexedDB 資料庫名稱 `kaodian-notebook`，版本 `1`，有五個 object store：

**chapters**（keyPath `id`）
```js
{ id, subject: 'ds'|'algo'|'co'|'os'|'dm'|'la', name, order }
```

**concepts**（keyPath `id`）：考點
```js
{ id, chapterId, name, aliases: [string], summary: string, createdAt, lastReviewedAt }
```
- `aliases`：同一考點的其他寫法。歸類時使用者若把 GPT 寫的名稱對應到既有考點，那個寫法會自動加進別名；改名時舊名稱也會加進別名。

**problems**（keyPath `id`）：題目
```js
{ id, conceptIds: [id], photoIds: [id], text, source, needsPhoto: boolean, createdAt,
  attempts: [{ at, status: 'ok'|'unsure'|'wrong', errorTypes: [string], reason: string, via: 'add'|'retry'|'review' }] }
```
- `text` 裡的圖用 `[圖]` … `[/圖]` 包起來（各自獨立一行），App 會以等寬字體、不換行、可左右捲動的方式顯示（`richText()`）。
- `needsPhoto`：GPT 判斷 ASCII 無法完整表達這題的圖。`needsPhoto` 為真且沒有照片時，App 會在題目頁提醒，題目清單也能篩選「缺照片」。舊資料沒有這個欄位，視為 `false`。
- 題目目前的狀態 = 最後一筆 attempt 的 `status`。
- `status` 為 `unsure` 或 `wrong` 時，`errorTypes` 至少一項、`reason` 必填（App 會擋住太短或只寫「粗心」之類的錯因）。

**photos**（keyPath `id`）
```js
{ id, blob }   // JPEG，最長邊壓到 1600px，品質 0.8
```

**meta**（keyPath `key`）
```js
{ key: 'seeded', value: true }        // 已建立章節骨架
{ key: 'lastBackupAt', value: ms }    // 上次備份時間
{ key: 'draft', value: {...} }        // 單題新增到一半的草稿（含照片 blob）
{ key: 'batch', value: {...} }        // 批次匯入的進度：待確認的題目、已存/跳過數、是否「未完」
```
另外 `localStorage` 的 `kd-add-mode` 記住新增頁上次選的是批次（`batch`）還是單題（`single`）。

時間一律是毫秒時間戳（`Date.now()`）。

## 提示詞與回覆格式（改動時兩邊要一起改）

提示詞都內含目前所有考點與章節清單，以及「轉成文字的規則」（`PROMPT_FIGURE_RULES`：數學式不用 LaTeX、圖用 `[圖]` 包起來、圖論的圖用邊的清單、`[圖]` 內只用英數、ASCII 表達不了就標「需要照片：是」）。

### 批次（`buildBatchPrompt()` ↔ `parseBatch()`）

```
===第1題===
出處：台大 112
需要照片：否
考點：
既有｜科目｜章節｜考點名稱
新增｜科目｜章節｜考點名稱
題目：
題目文字，可以多行
[圖]
  ASCII
[/圖]
===第2題===
…
===全部結束===
```

- 「題目：」一定在每題最後，題目內容一直讀到下一個 `===第N題===` 或結束標記，所以題目可以很長、可以換行。
- GPT 一次寫不完時，會在某題結束後寫 `===未完===`。使用者說「繼續」後把下一段貼進 App，App 會**接在目前這批後面**。
- 同一題重複貼上（題號＋題目開頭相同）會被略過。
- 如果貼上的是單題格式，`addBatchText()` 也會接受，當成一題。

### 單題（`buildPrompt()` ↔ `parseReply()`）

```
===題目===
題目文字
===考點===
既有｜科目｜章節｜考點名稱
新增｜科目｜章節｜考點名稱
===出處===
台大 112
===需要照片===
否
===結束===
```

- 分隔符號接受全形「｜」或半形「|」，前後的等號數量不限，程式碼區塊標記會被忽略。
- 科目可以寫全名或簡稱（資結、計組、OS、線代…），由 `matchSubject()` 對應。
- 歸類由 `makeCard()` 決定：名稱或別名完全相同 → 既有考點；相似度 ≥ 0.72 → 也建議既有考點；否則建議建立新考點，並列出最多 3 個相近的既有考點讓使用者一鍵改選。
- 單題模式：每個歸類都要使用者按確認。
- 批次模式：建議直接顯示為已採用，使用者按下「會／不確定／不會」就代表接受，不對才按「改放別處」。新考點若缺科目或章節，必須先補好才能存。
- 批次的歸類在**輪到那一題時**才計算，所以前面題目剛建立的新考點，後面題目會自動對上，不會重複建立。

## 備份檔格式

`考點筆記備份_YYYYMMDD.zip`（無壓縮的標準 zip，任何解壓工具都能開）：

```
data.json        // { app:'kaodian-notebook', format:1, exportedAt, subjects, chapters, concepts, problems }
筆記總覽.md       // 給人或 AI 閱讀的完整文字版：每個考點的摘要、題目、每次作答與錯因
photos/<id>.jpg  // 題目照片，檔名對應 problems[].photoIds
```

使用者如果想請你幫忙分析弱點、做複習計畫，可以把 `筆記總覽.md` 傳給你，裡面有所有考點、錯誤類型和錯因。

還原時讀取 `data.json` 與 `photos/`，會**取代**手機上的全部資料。還原也接受用其他工具重新壓縮過（deflate）的 zip，只要 `data.json` 結構正確。

## 常見修改位置（都在 `index.html` 的 `<script>` 內）

| 想改的東西 | 位置 |
|---|---|
| 預設章節骨架 | `SEED_CHAPTERS`（只在第一次開啟、資料庫是空的時候套用；已在使用中的 App 請直接在 App 裡新增或改名章節） |
| 錯誤類型選項 | `ERROR_TYPES`，對應的建議文字在 `TYPE_TIPS` |
| 給 GPT 的提示詞 | `buildBatchPrompt()`、`buildPrompt()`、`buildSummaryPrompt()`，共用規則在 `PROMPT_CONCEPT_RULES`、`PROMPT_FIGURE_RULES` |
| 回覆解析 | `parseBatch()`／`parseBatchItem()`、`parseReply()` |
| 批次確認畫面 | `renderBatch()`、`saveBatchItem()` |
| 圖的顯示 | `richText()` 與 CSS `.fig` |
| 複習排序規則 | `problemScore()`（不會權重 3、不確定 2、會 0.5，乘上距離上次作答的天數因子）、`reviewPool()`（20 小時內做過的題目不排入）、`todayPool()`（今天新增或做過的題目，依不會→不確定→會）、`allPool()`（不看時間，全部依 `problemScore` 排序） |
| 滑動刪除 | CSS `.swipe`、JS 的 `swipe to delete` 區塊（pointer events，`touch-action: pan-y` 讓上下捲動照常運作）、`deleteProblemWithUndo()`／`undoDelete()`（刪除後 5 秒內可復原） |
| 錯因的最低要求 | `attemptProblems()` 與 `TRIVIAL` |
| 自動建議既有考點的門檻 | `makeCard()` 裡的 `0.72` |
| 顏色 | CSS 開頭的 `:root` 變數，科目色是 `--s-ds` 等 |

## 修改時的原則

- 保持單一 `index.html`、不引入外部 CDN 或套件，App 才能離線使用。
- **不要改** IndexedDB 名稱 `kaodian-notebook`，也不要隨意更動既有欄位名稱。要加欄位時，讀取端要能處理舊資料沒有該欄位的情況。
- 若必須調整資料結構，把 `DB_VERSION` 加一，並在 `onupgradeneeded` 寫遷移程式。改動前請使用者先備份。
- 修改後務必把 `sw.js` 的 `CACHE` 版本號加一。
- 文字介面使用繁體中文。

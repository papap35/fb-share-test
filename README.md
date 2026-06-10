# Facebook 分享機制概念驗證 (FB Share Methods POC)

本專案旨在驗證與評估三種在網頁前端實作 Facebook 連結分享的主流技術方案。透過此小型 POC，我們將動態觀察不同分享機制的設定成本、使用者體驗（UX）以及 Open Graph (OG) Meta 標籤的抓取表現。

## 🎯 驗證的三種分享方式

1. **Facebook JavaScript SDK (`FB.ui`)**：官方提供的重量級生態系方案，具備豐富的互動與回呼（Callback）機制。
2. **Facebook Sharer (`sharer.php`)**：透過特定 URL 帶入參數的輕量化 Web 方案，無需載入任何外部腳本。
3. **Web Share API (`navigator.share`)**：行動端原生的分享機制，可直接喚起 iOS/Android 系統的原生分享底座。

---

## 📂 專案結構

* `index.html`：**POC 控制台**。支援動態輸入、修改 FB App ID、API 版本以及測試目標網址，並內建 LocalStorage 記憶功能與動態日誌輸出。
* `share-target.html`：**測試目標網頁**。部署於公開環境，內含標準的 `og:title`、`og:description`、`og:image` 等 Meta Tags，供 Facebook 爬蟲（Scraper）抓取。

---

## 🚀 快速開始與測試步驟

由於 Facebook 爬蟲無法存取本機端環境（如 `localhost` 或 `127.0.0.1`），請依序按照以下步驟進行完整測試：

### 1. 部署目標網頁
將 `share-target.html` 部署至具備 HTTPS 的公開伺服器（例如 GitHub Pages、Vercel），或在本機使用 `ngrok` 進行外網映射。
> ⚠️ **注意**：請務必將 `share-target.html` 內的 `<meta property="og:url" content="..." />` 修改為你部署後的真實外網網址。

### 2. 刷新 Facebook 緩存
將部署後的 `share-target.html` 網址複製，貼入 [Facebook 分享偵錯工具 (Sharing Debugger)](https://developers.facebook.com/tools/debug/)。
* 點擊 **「偵錯 (Debug)」**。
* 若圖片未正常顯示，請多次點擊 **「再次擷取 (Scrape Again)」** 直到臉書成功快取圖片與標題。

### 3. 運行控制台進行測試
使用瀏覽器開啟 `index.html`：
1. 輸入你的 **Facebook App ID**（若要測試方法 1）。
2. 點擊 **「套用配置並初始化/重置 SDK」**。
3. 在「測試分享網址」欄位輸入你已部署的 `share-target.html` 網址。
4. 依序點擊三個按鈕，觀察彈出視窗、原生 UI 以及底部的「控制台紀錄」。

---

## 📊 技術方案深度對比

| 特性 \ 方法 | 1. FB JS SDK | 2. Sharer.php | 3. Navigator.share |
| :--- | :--- | :--- | :--- |
| **開發難易度** | **高** (需申請 App ID、異步載入 SDK) | **極低** (純網址拼接 `?u=...`) | **低** (純原生 JavaScript 呼叫) |
| **外部依賴** | 相依於 Facebook 官方 JS 檔案 | 無任何外部依賴 | 無任何外部依賴 |
| **UI 呈現效果** | 桌機彈出內嵌視窗 / 手機跳轉頁面 | 桌機彈出新視窗 / 手機跳轉頁面 | **最佳**。喚起手機原生分享選單 (底座) |
| **環境限制** | 注意瀏覽器阻擋彈出視窗 (Popup Blocker) | 同左，需由使用者點擊事件 (Click) 觸發 | **嚴格限制 HTTPS**，且僅在行動端瀏覽器支援 |
| **結果追蹤 (Callback)** | 具備基礎 Callback (可得知視窗關閉) | **完全無法得知** 使用者最後是否發布 | 僅能得知是否成功喚起選單，無法追蹤後續 |
| **內容自訂靈活性** | 嚴格依賴目標網頁的 `og:tags` | 嚴格依賴目標網頁的 `og:tags` | 可自訂轉發文字 (若轉發至非 FB 平台更彈性) |

---

## 💡 踩坑筆記與結論

* **非同步圖片警告**：首次測試時，FB 常提示 `og:image 目前無法使用`。這是因為 FB 伺服器首次快取需要時間，在偵錯工具中連續點擊「再次擷取」即可解決。
* **圖片格式限制**：FB 爬蟲對於動態轉址的圖片（如 `picsum.photos`）支援度較差，實務上 `og:image` 應盡量指向靜態且具備副檔名（`.jpg`, `.png`）的真實圖檔路徑。
* **選型建議**：
  * 若專案著重於**行動端體驗**，優先考慮 `Navigator.share`（體驗最直覺）。
  * 若需要**快速上線、不想維護 App ID**，使用 `Sharer.php` 是最高效的解法。
  * 若需要**追蹤使用者互動**或結合臉書登入等進階功能，才必須選用 `FB JS SDK`。

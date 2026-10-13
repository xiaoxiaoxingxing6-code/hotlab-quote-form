# HOT 環虹檢驗｜LINE 詢價表單 前後台串接步驟

題目與官網 https://hotlab.com.tw/ 詢價問卷相同：詢問服務（複選）、檢驗需求說明、客戶類型、公司名稱、聯絡人、聯絡電話、電子郵件、所在區域，加上個資同意。

```
LINE 圖文選單「線上詢價」（LIFF 網址）
  → GitHub Pages 上的 index.html（在 LINE 裡開啟，顯示「○○ 您好」，自動帶入上次的聯絡資料）
  → Google Apps Script（Code.gs）
  → 寫入 Google 試算表「詢價紀錄」＋通知信給 service@hotlab.com.tw＋確認信給客人
  → 送出後按「關閉視窗」直接回到 LINE 聊天室
```

需要準備：
- 一個 GitHub 帳號（放前台網頁）
- 一個公司 Google 帳號（放試算表與 Apps Script，通知信會從這個帳號寄出）
- 能管理 @hotlab 官方帳號的 LINE 帳號（建立 LIFF、設定圖文選單）

上線順序（每一步都會用到前一步產生的網址）：

| 步驟 | 做什麼 | 產出 |
|---|---|---|
| 1 | 上傳 index.html 到 GitHub Pages | 表單網址 |
| 2 | 建立試算表＋Apps Script 並部署 | GAS 網址（/exec） |
| 3 | LINE Developers 建立 LIFF | LIFF ID、LIFF 網址 |
| 4 | 把 LIFF ID、GAS 網址填進 index.html，重新上傳 | 正式版表單 |
| 5 | 圖文選單按鈕設定 LIFF 網址 | 上線 |
| 6 | 實機測試 | — |

---

## 步驟 1：上傳 index.html 到 GitHub Pages

1. 登入 GitHub →「＋」→「New repository」，名稱 `hotlab-quote-form`，選 **Public** →「Create repository」。
2. 點「uploading an existing file」→ 拖入 **index.html** →「Commit changes」。
3. 「Settings」→「Pages」→ Source「Deploy from a branch」，Branch 選 `main`、`/ (root)` →「Save」。
4. 1～2 分鐘後取得網址：`https://帳號.github.io/hotlab-quote-form/`
5. 手機打開確認。此時還沒填設定，頂端會有黃色「預覽版」提示，正常。

## 步驟 2：建立試算表與 Apps Script

1. 用公司 Google 帳號新增試算表「環虹檢驗_LINE詢價紀錄」。
2. 「擴充功能」→「Apps Script」，刪掉預設內容，貼上 **Code.gs** 全部內容。
3. 確認最上面 `CONFIG.NOTIFY_EMAILS` 是要收通知的信箱（多位用逗號分隔），按儲存。
4. 函式選單選 **setup** →「執行」→ 跳出授權：「審查權限」→ 選帳號 →「進階」→「前往（不安全）」→「允許」。
5. 函式選單選 **testSubmit** →「執行」，確認：
   - 試算表出現「詢價紀錄」工作表，多一筆 HOT-日期-001
   - 通知信箱收到【LINE 詢價】信
   - 下方執行紀錄「帶入」那行有 `"found":true`

   確認後把測試那一列刪掉。
6. 右上「部署」→「新增部署作業」→ 齒輪選「網頁應用程式」：
   - 執行身分：**我**
   - 具有存取權的使用者：**所有人**
   →「部署」，複製結尾是 `/exec` 的網址。
7. 用瀏覽器打開這個網址，看到 `{"ok":true,"service":"hotlab-line-form"}` 就代表後台通了。

> 之後若修改 Code.gs：「部署」→「管理部署作業」→ 鉛筆 → 版本選「新版本」→「部署」，網址不變。不要按「新增部署作業」，那會產生新網址。

## 步驟 3：建立 LIFF

1. 到 https://developers.line.biz/console/ ，用管理 @hotlab 的 LINE 帳號登入。
2. 選環虹的 Provider（沒有就「Create」一個，名稱如「環虹檢驗科技」）。
3. 「Create a new channel」→ **LINE Login**：
   - Region：Taiwan
   - Channel name：環虹線上詢價
   - Channel description：環虹檢驗 LINE 詢價表單
   - App types：勾 **Web app**
   - 勾選同意條款 →「Create」
4. 進入 Channel →「Basic settings」→「Linked LINE Official Account」→「Edit」→ 選 @hotlab。
5. 上方「LIFF」分頁 →「Add」：
   - LIFF app name：環虹線上詢價
   - Size：**Full**
   - Endpoint URL：步驟 1 的 GitHub 網址
   - Scopes：勾 **profile**
   - Add friend option：**On (normal)**
   →「Add」
6. 複製 **LIFF ID**（如 `2001234567-AbCdEfGh`）與 **LIFF URL**（`https://liff.line.me/2001234567-AbCdEfGh`）。
7. Channel 頁面上方狀態從「Developing」改成 **Published**，否則只有開發者打得開。

## 步驟 4：填入設定並重新上傳

1. 用記事本或 VS Code 開 index.html，找到：
   ```js
   const LIFF_ID = '請貼上LIFF_ID';
   const GAS_ENDPOINT = '請貼上GAS網址';
   ```
   換成步驟 3 的 LIFF ID、步驟 2 的 `/exec` 網址（保留單引號），存檔。
2. GitHub repo →「Add file」→「Upload files」→ 拖入新的 index.html →「Commit changes」（會覆蓋舊檔）。
3. 等 1～2 分鐘，打開網址，黃色「預覽版」提示消失就代表設定生效。

## 步驟 5：設定圖文選單

1. LINE Official Account Manager（manager.line.biz）→ @hotlab →「圖文選單」。
2. 詢價按鈕：類型「**連結**」，網址填：
   `https://liff.line.me/LIFF_ID?src=richmenu`
3. 儲存，確認圖文選單狀態是「使用中」。

`?src=` 會寫進試算表「來源」欄，其他入口可用 `?src=welcome`（歡迎訊息）、`?src=keyword`（關鍵字回覆）、`?src=website`（官網按鈕）。

## 步驟 6：實機測試

用手機 LINE 打開 @hotlab 聊天室，點圖文選單「線上詢價」：
1. 第一次會跳 LINE 授權畫面 →「許可」
2. 表單上方顯示「你的名字 您好」
3. 填寫送出 → 顯示「已送出，謝謝！」→ 按「關閉視窗」回到聊天室
4. 試算表多一筆、service 信箱收到通知、填的 Email 收到確認信
5. 再點一次圖文選單，聯絡資料應該已自動帶入（詢問服務與需求說明要重填）

---

## 試算表欄位

案件編號（HOT-20261013-001）、送出時間、詢問服務、檢驗需求說明、客戶類型、公司名稱、聯絡人、聯絡電話、電子郵件、所在區域、LINE 名稱、LINE userId、來源、處理狀態（下拉：待處理／處理中／已報價／已成交／未成交）、負責人、備註。後三欄給客服自行填寫。

## 常見問題

- **送出失敗**：GAS_ENDPOINT 填錯，或部署存取權不是「所有人」。用瀏覽器開 `/exec` 網址，看到 `{"ok":true,...}` 才正常。
- **出現「LIFF 初始化失敗」**：LIFF ID 填錯、Endpoint URL 跟 GitHub 網址不一致（結尾斜線也要一樣），或 Channel 還在 Developing。
- **沒帶入上次資料**：要從 LINE 裡用 LIFF 網址開啟才拿得到使用者身分；用一般瀏覽器開啟只能填表、不會帶入。
- **改了 Code.gs 沒生效**：要用「管理部署作業」發布新版本。
- **沒收到通知信**：先執行一次 setup 確認已授權；免費 Gmail 每天約可寄 100 封，Google Workspace 帳號較多。
- **不想寄確認信給客人**：Code.gs 把 `SEND_CUSTOMER_CONFIRM` 改成 `false`，再發布新版本。

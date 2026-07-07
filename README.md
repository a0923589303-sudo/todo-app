# 待辦助理 PWA

## 🤖 設定 LINE 通知（LINE Bot，需先做這步）

App 用 LINE Bot（Messaging API）推播提醒，需要兩個東西：一個 **LINE Bot Channel**（免費）和一個 **Cloudflare Worker**（免費）當後端。Channel Access Token 是整個 Bot 共用的密鑰，不能放在網頁程式碼裡，所以要靠 Worker 保管並轉發。

### 步驟一：建立 LINE Messaging API Channel

1. 前往 [LINE Developers Console](https://developers.line.biz/console/)，用 LINE 帳號登入
2. 建立一個 Provider（任意命名，例如「個人專案」）
3. 在 Provider 底下建立 Channel，選 **Messaging API**
4. 填基本資料（Channel name 可填「待辦助理」），建立完成
5. 到 Channel 的「Messaging API」分頁：
   - 找到 **Channel access token**，按「Issue」產生一個長期權杖，複製起來（等下要用）
   - 記下 **Channel secret**（在「Basic settings」分頁）
   - 把「Auto-reply messages」「Greeting messages」關掉（避免跟我們自己的回覆衝突）
   - 記下這個 Bot 的 **Bot basic ID**（例如 `@123abcde`）或 QR code，等下要給使用者加好友用

### 步驟二：部署 Cloudflare Worker（後端）

後端程式碼在 `worker/` 資料夾裡。

1. 安裝 [Node.js](https://nodejs.org/)（若還沒裝）
2. 到 Cloudflare 官網註冊免費帳號：https://dash.cloudflare.com/sign-up
3. 打開終端機，進入 `worker` 資料夾：
   ```bash
   cd worker
   npx wrangler login
   ```
   （會開瀏覽器要求授權 Cloudflare 帳號）
4. 建立 KV 命名空間（用來暫存綁定碼和使用者對應關係）：
   ```bash
   npx wrangler kv namespace create LINE_KV
   ```
   指令會印出一段像這樣的設定：
   ```
   { binding = "LINE_KV", id = "abcd1234..." }
   ```
   把這個 `id` 複製起來，貼到 `worker/wrangler.toml` 裡 `PASTE_YOUR_KV_NAMESPACE_ID_HERE` 的位置
5. 設定密鑰（步驟一拿到的 Channel access token 和 Channel secret）：
   ```bash
   npx wrangler secret put CHANNEL_ACCESS_TOKEN
   npx wrangler secret put CHANNEL_SECRET
   ```
   （每次會提示你貼上對應的值）
6. 部署：
   ```bash
   npx wrangler deploy
   ```
   完成後會印出一個網址，例如 `https://todo-line-bot.你的帳號.workers.dev`，記下來

### 步驟三：設定 Webhook

回到 LINE Developers Console 該 Channel 的「Messaging API」分頁：
1. **Webhook URL** 填：`https://你的worker網址/webhook`
2. 按「Verify」確認顯示成功
3. 把「Use webhook」打開

### 步驟四：把網址填回 App

打開 `index.html`，找到最上面的 CONFIG 區塊：
```js
const WORKER_URL = 'https://YOUR-WORKER-SUBDOMAIN.workers.dev';
const LINE_ADD_FRIEND_URL = 'https://line.me/R/ti/p/@YOUR_LINE_ID';
```
- `WORKER_URL` 換成步驟二拿到的 Worker 網址（不要加結尾斜線）
- `LINE_ADD_FRIEND_URL` 換成步驟一的 Bot 加好友連結（`https://line.me/R/ti/p/@你的BotID`，BotID 去掉開頭的 @）

存檔後即可依下方步驟上傳到 GitHub Pages。使用者在 App 內點「LINE 通知」→「產生綁定碼」→ 把 6 位數字傳給 Bot → 「檢查連結狀態」即可完成綁定。

---

## 🚀 上傳到 GitHub Pages（免費，5分鐘搞定）

### 步驟一：建立 GitHub 帳號
前往 https://github.com 註冊（已有帳號跳過）

### 步驟二：建立新 Repository
1. 點右上角「+」→「New repository」
2. Repository name 輸入：`todo-app`（或任何名字）
3. 選 **Public**
4. 按「Create repository」

### 步驟三：上傳檔案
1. 點「uploading an existing file」
2. 把整個 `todo-pwa` 資料夾裡的**所有檔案和 icons 資料夾**拖進去
3. 按「Commit changes」

### 步驟四：開啟 Pages
1. 進 Settings → Pages（左側選單）
2. Source 選「Deploy from a branch」
3. Branch 選「main」，資料夾選「/ (root)」
4. 按 Save

### 步驟五：等 1~2 分鐘
網址會是：`https://你的帳號.github.io/todo-app/`

---

## 📱 加到手機桌面

### iPhone（Safari）
1. 用 Safari 開啟網址
2. 點底部「分享」按鈕（方形箭頭）
3. 選「加入主畫面」
4. 名稱改成「待辦助理」→「新增」

### Android（Chrome）
1. 用 Chrome 開啟網址
2. 點右上角「⋮」選單
3. 選「加到主畫面」或「安裝應用程式」
4. 確認即可

---

## 📁 檔案結構
```
todo-pwa/
├── index.html      ← 主程式
├── manifest.json   ← PWA 設定
├── sw.js           ← 離線快取
├── icons/          ← App 圖示
│   ├── icon-72.png
│   ├── icon-96.png
│   ├── icon-128.png
│   ├── icon-144.png
│   ├── icon-152.png
│   ├── icon-192.png
│   ├── icon-384.png
│   └── icon-512.png
└── worker/         ← LINE Bot 後端（部署到 Cloudflare，不用上傳到 GitHub Pages）
    ├── src/index.js
    ├── wrangler.toml
    └── package.json
```

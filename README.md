# LINE Bot 檔案備份機器人 (Build With AI 2026 Workshop)

這是一個功能強大的 LINE Bot，可以讓使用者輕鬆地將聊天室中的圖片、影片、音訊和檔案備份到自己的 Google Drive。它會自動整理檔案，並提供方便的查詢功能。

> [!IMPORTANT]
> **Build With AI 2026 Workshop 專題項目**  
> 本專案為 Build With AI 2026 台北站工作坊的範例專案。  
> 📖 相關簡報內容請參考：[Build With AI 2026 - kkdai](https://github.com/kkdai/bwai-2026)

[![Go Report Card](https://goreportcard.com/badge/github.com/kkdai/linebot-file)](https://goreportcard.com/report/github.com/kkdai/linebot-file)
[![MIT License](https://img.shields.io/badge/license-Apache2-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/fd567ff2-0589-483e-ae7f-986921c3d303" />

---

## ✨ 主要功能

- **多媒體檔案備份**：支援備份圖片、影片、音訊和一般檔案。
- **智慧資料夾整理**：自動在您的 Google Drive 建立 `LINE Bot Uploads` 資料夾，並以年月 (`YYYY-MM`) 為單位建立子資料夾存放檔案。
- **安全帳號連結**：使用 Google OAuth 2.0 進行授權，安全可靠。
- **查詢最近檔案**：透過 `/recent_files` 指令，快速查看最近上傳的 5 個檔案。
- **檔案搜尋**：透過 `/search_files <關鍵字>` 或 `/q <關鍵字>` 指令，搜尋 Google Drive 中含特定關鍵字的檔案。
- **完整的連線控制**：使用者可隨時透過 `/disconnect_drive` 指令中斷連線並撤銷授權。

---

## 🤖 Workshop 2：打造 LINE Bot 並部署至 Cloud Run

本章節指導如何建立 LINE Bot、取得金鑰，並使用 Gemini CLI 部署至 Google Cloud Run。

### 📋 流程總覽

```
[階段一] 取得 LINE 開發金鑰
      ↓
[階段二] Google Cloud 專案設定與開通
      ↓
[階段三] 設定 OAuth 與身分驗證
      ↓
[階段四] 使用 Gemini CLI 部署至 Cloud Run
      ↓
[階段五] 完成 LINE Webhook 設定
```

### 📁 範例程式碼

本實作使用以下範例專案：

- Repo: https://github.com/kkdai/bwai2026-sample

```bash
git clone https://github.com/kkdai/bwai2026-sample
cd bwai2026-sample
```

---

## 🔑 階段一：取得 LINE 開發金鑰

### 1. 建立官方帳號

1. 進入 [LINE Business ID 登入頁面](https://account.line.biz/login)。
2. 存取 [LINE Official Account Manager](https://manager.line.biz/)，選擇「使用 LINE 帳號登入」或「使用商用帳號登入」。
3. 建立新帳號，填入以下資訊：
   - **帳號名稱**：你的 Bot 名稱
   - **公司/團體名稱**：可填寫個人
   - **業種**：選擇適合的分類
   - **電子郵件信箱**
4. 確認資料無誤後點擊「送出」。

### 2. 啟用 Messaging API

1. 在後台右上角點擊「設定」，左側選單點選「Messaging API」。
2. 點擊「啟用 Messaging API」。
3. 選擇/建立服務提供者 (Provider)：輸入開發商名稱。
4. 完成後，帳號即與 [LINE Developers Console](https://developers.line.biz/console/) 連結。

### 3. 取得開發金鑰

前往 [LINE Developers Console](https://developers.line.biz/console/) 取得以下兩組金鑰：

**Channel Secret**
- 在 Console 中進入該 Channel，點選 **Basic settings** 分頁。
- 向下捲動到底部即可看到 **Channel secret**。

**Channel Access Token**
- 點選 **Messaging API** 分頁。
- 捲動到頁面最下方，找到 **Channel access token**。
- 點擊「Issue」(發行)，取得產生的字串 (long-lived)。

### 4. 關閉自動回應（重要）

為了讓程式碼完全接管訊息，請進行以下設定：

1. 回到 [LINE Official Account Manager](https://manager.line.biz/)。
2. 點擊右上角「設定」→「回應設定」。
3. **回應模式**：設定為「聊天機器人」。
4. **進階設定**：
   - 自動回應訊息：**停用**
   - Webhook：**啟用**（部署 Server 後開啟）

---

## ☁️ 階段二：Google Cloud 專案設定與開通

在部署前，需要確保 Google Cloud 專案已開通相關服務。

**建立/設定專案**

```bash
gcloud config set project [YOUR_PROJECT_ID]
```

**開通必要 API**

```bash
gcloud services enable run.googleapis.com cloudbuild.googleapis.com \
  firestore.googleapis.com artifactregistry.googleapis.com drive.googleapis.com
```

**建立 Firestore 儲存空間**（用於存放 OAuth Token）

```bash
gcloud firestore databases create --location=asia-east1 --type=firestore-native
```

---

## 🔐 階段三：設定 OAuth 與身分驗證

Gemini CLI 需要透過 Google OAuth 進行身分驗證，才能代表你操作 Google Cloud 資源。

### 1. 設定 OAuth 同意畫面 (OAuth Consent Screen)

1. 進入 [Google Cloud Console - OAuth 同意畫面](https://console.cloud.google.com/apis/credentials/consent)。
2. **User Type**：選擇「外部 (External)」（若為 Workspace 帳號可選內部），點擊「建立」。
3. 填寫應用程式資訊：
   - **應用程式名稱**：例如 `My LINE Bot`
   - **使用者支援電子郵件**：選擇你的 Gmail
   - **開發者聯絡資訊**：填入你的電子郵件
4. 點擊「儲存並繼續」，其餘步驟可直接點擊「儲存並繼續」跳過。
5. 回到儀表板，點擊「**發布應用程式**」並確認，OAuth 才能正式運作。

> [!NOTE]
> 若為測試狀態，需在「測試使用者」中手動加入你自己的 Gmail 帳號，才能完成授權流程。

### 2. 建立 OAuth 客戶端 ID 與 Web 應用程式憑證

> [!NOTE]
> 這個步驟需要在 Cloud Run 部署完成後，取得 Service URL 後才能填入重新導向 URI。可以先跳到階段四部署，再回來完成此步驟。

1. 進入 [Google Cloud Console - 憑證 (Credentials)](https://console.cloud.google.com/apis/credentials)。
2. 點擊「建立憑證」→「OAuth 客戶端 ID」。
3. **應用程式類型**：選擇「**網頁應用程式 (Web Application)**」。
4. **名稱**：自訂名稱（例如 `LINE Bot OAuth Client`）。
5. **已授權的重新導向 URI**：填入 Cloud Run 網址加上 `/oauth/callback`。
   - 範例：`https://bwai2026-xxxxx.a.run.app/oauth/callback`
6. 點擊「建立」，記下 **Client ID** 與 **Client Secret**。

### 3. 啟用 Google Drive API

前往 [API 程式庫](https://console.cloud.google.com/apis/library/drive.googleapis.com) 並啟用 Google Drive API。

### 4. 使用 Gemini CLI 進行登入

**初次登入**：執行 `gemini` 指令時，選擇「Sign in with Google」，這會開啟瀏覽器讓你選擇 Google 帳號。

**ADC 驗證（強烈建議）**：為了讓背景部署指令順利執行，請在本機終端機執行：

```bash
gcloud auth application-default login
```

這會將憑證儲存在本機，供 `gcloud` 與 `gemini-cli` 使用。

**重新驗證**：若需切換帳號或權限過期，在 Gemini CLI 中輸入：

```
/auth
```

---

## 🚀 階段四：使用 Gemini CLI 部署至 Cloud Run

Gemini CLI 具備「自動化部署」的能力，你只需要用自然語言下達指令。

### 啟動 Gemini CLI 互動模式

```bash
gemini
```

### 下達部署指令

在對話框中輸入：

```
請幫我將這個專案部署到 Cloud Run，區域選擇 asia-east1，
並設定環境變數 LINE_CHANNEL_SECRET 與 LINE_CHANNEL_ACCESS_TOKEN。
如果需要任何資料，請停下來問我。
```

或是直接提供完整資訊：

```
幫我使用 gcloud 部署到 Cloud Run，如果需要任何資料，請停下來問我。
參考 repo https://github.com/kkdai/bwai2026-sample
```

Gemini CLI 會分析專案結構，生成 `gcloud run deploy` 指令，並在執行前詢問你的確認。

### Gemini CLI 自動執行的指令

```bash
# 執行首次部署
gcloud run deploy bwai2026 \
  --source . \
  --region asia-east1 \
  --allow-unauthenticated

# 更新環境變數（部署後設定金鑰）
gcloud run services update bwai2026 \
  --region asia-east1 \
  --update-env-vars "LINE_CHANNEL_SECRET=<your_secret>,LINE_CHANNEL_ACCESS_TOKEN=<your_token>,GOOGLE_CLIENT_ID=<client_id>,GOOGLE_CLIENT_SECRET=<client_secret>"
```

---

## ⚙️ 階段五：完成 LINE Webhook 設定

1. 部署完成後，從 Cloud Run 取得 **Service URL**（格式如 `https://bwai2026-xxxxx.a.run.app`）。
2. 回到 [LINE Developers Console](https://developers.line.biz/console/) → **Messaging API** 分頁。
3. 在 **Webhook URL** 填入：`[Service URL]/callback`
4. 點擊「**Verify**」確保連線成功（應顯示 Success）。
5. 開啟「**Use webhook**」選項。
6. 回到 [LINE Official Account Manager](https://manager.line.biz/) →「回應設定」：
   - 回應模式：**聊天機器人**
   - 自動回應訊息：**停用**

---

## 🔍 驗證與測試 MCP 服務

安裝完成後，可透過以下測試指令確認 MCP Server 是否已正確掛載並運作。

### 1. 測試 Google Developer Knowledge API

這個 MCP 讓 Gemini 能夠讀取最新的 Google 官方文件。

**測試 Prompt：**

```
請幫我查詢 Google Cloud Run 的最新部署限制（Deployment Limits），並列出前三項。
```

**驗證重點：**
- Gemini 應該會調用 `google-developer-knowledge` 工具。
- 回覆內容應包含來自 `cloud.google.com` 或 `developer.android.com` 等官方網域的資訊。
- 回覆末尾通常會附上參考來源連結。

### 2. 測試 Google Maps Platform Code Assist

這個 MCP 專門協助開發者撰寫 Google Maps 相關的程式碼與 API 整合。

**測試 Prompt：**

```
我想在網頁中嵌入一個 Google 地圖，請幫我寫出一段基本的 JavaScript 程式碼
來顯示地圖，中心點設在台北 101。
```

**驗證重點：**
- Gemini 應該會調用 `google-maps-platform-code-assist` 工具。
- 生成的程式碼應包含 Maps JavaScript API 的正確載入方式與初始化語法。
- 它可能會主動提醒你關於 API Key 的設定以及相關的安全性建議。

---

## 🛠️ 常用指令速查表

| 功能 | 指令 |
| :--- | :--- |
| **部署/更新服務** | `gcloud run deploy [SERVICE_NAME] --source . --region asia-east1` |
| **更新環境變數** | `gcloud run services update [SERVICE_NAME] --update-env-vars "KEY=VALUE"` |
| **查看即時日誌** | `gcloud beta run services logs tail [SERVICE_NAME]` |
| **查看服務狀態** | `gcloud run services describe [SERVICE_NAME] --region asia-east1` |

---

## 📜 License

本專案採用 [Apache License 2.0](LICENSE)。

## 🤝 如何貢獻

這是一次 **Build With AI 2026** 的 Workshop 實作內容。歡迎提出 Issue 或 Pull Request。

感謝所有參與工作坊的開發者！

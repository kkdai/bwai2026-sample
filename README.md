# LINE Bot 檔案備份機器人 (Build With AI 2026 Workshop)

這是一個功能強大的 LINE Bot，可以讓使用者輕鬆地將聊天室中的圖片、影片、音訊和檔案備份到自己的 Google Drive。它會自動整理檔案，並提供方便的查詢功能。

> [!IMPORTANT]
> **Build With AI 2026 Workshop 專題項目**  
> 本專案為 Build With AI 2026 台北站工作坊的範例專案。  
> 📖 相關簡報內容請參考：[Build With AI 2026 - kkdai](https://github.com/kkdai/bwai-2026)

[![Go Report Card](https://goreportcard.com/badge/github.com/kkdai/linebot-file)](https://goreportcard.com/report/github.com/kkdai/linebot-file)
[![MIT License](https://img.shields.io/badge/license-Apache2-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/fd567ff2-0589-483e-ae7f-986921c3d303" />

## ✨ 主要功能

*   **多媒體檔案備份**：支援備份圖片、影片、音訊和一般檔案。
*   **智慧資料夾整理**：自動在您的 Google Drive 建立 `LINE Bot Uploads` 資料夾，並以年月 (`YYYY-MM`) 為單位建立子資料夾存放檔案，保持雲端硬碟整潔。
*   **安全帳號連結**：使用 Google OAuth 2.0 進行授權，安全可靠。
*   **查詢最近檔案**：透過 `/recent_files` 指令，快速查看最近上傳的 5 個檔案。
*   **檔案搜尋**：透過 `/search_files <關鍵字>` 或 `/q <關鍵字>` 指令，搜尋 Google Drive 中包含特定關鍵字的檔案。
*   **完整的連線控制**：使用者可以隨時透過 `/disconnect_drive` 指令中斷連線並撤銷授權。

## 🚀 部署導引 (使用 Gemini CLI)

本專案強烈建議透過 **Gemini CLI** 實現 AI 輔助部署。AI 會協助您檢查環境、啟用服務並正確配置環境變數。

### 前置作業

1.  擁有一個 Google Cloud 帳號並啟用計費。
2.  安裝並設定好 [Google Cloud SDK (gcloud CLI)](https://cloud.google.com/sdk/docs/install)。
3.  安裝 **Gemini CLI** 以獲得互動式的部署指引。
4.  準備 LINE Bot 頻道資料：**Channel Secret** 與 **Channel Access Token**。

### 步驟 1：使用 Gemini CLI 啟動部署
在終端機進入專案目錄後，直接對 Gemini CLI 下達指令：

> **使用者語令範例：**
> 「幫我使用 gcloud 部署到 Cloud Run，如果需要任何資料，請停下來問我。參考 repo https://github.com/kkdai/bwai2026-sample」

### 步驟 2：AI 執行的底層自動化流程
Gemini CLI 會自動引導並執行以下 `gcloud` 指令：

1.  **專案環境初始化**：
    ```bash
    gcloud config set project [YOUR_PROJECT_ID]
    gcloud services enable firestore.googleapis.com cloudbuild.googleapis.com run.googleapis.com artifactregistry.googleapis.com drive.googleapis.com
    ```
2.  **建立 Firestore 儲存空間** (用於存放 OAuth Token)：
    ```bash
    gcloud firestore databases create --location=asia-east1 --type=firestore-native
    ```
3.  **執行首次部署 (Cloud Run)**：
    ```bash
    gcloud run deploy bwai2026 --source . --region asia-east1 --allow-unauthenticated
    ```

---

## 🔐 Google OAuth 平台設定 (必要)

為了讓 Bot 能夠代表使用者存取 Google Drive，必須手動完成以下設定：

1.  **啟用 Google Drive API**：前往 [API 程式庫](https://console.cloud.google.com/apis/library/drive.googleapis.com) 並啟用。
2.  **設定 OAuth 同意畫面 (Consent Screen)**：
    *   **使用者類型**：選擇「外部 (External)」。
    *   **範圍 (Scopes)**：新增 `.../auth/drive.file`。
    *   **測試使用者**：在測試狀態下，必須手動加入您自己的 Gmail 帳號。
3.  **建立 OAuth 2.0 憑證**：
    *   建立「網頁應用程式」類型的憑證。
    *   **已授權的重新導向 URI**：填入您的 Cloud Run 網址加上 `/oauth/callback`。
        *   範例：`https://bwai2026-xxxxx.a.run.app/oauth/callback`
4.  **更新服務設定**：將取得的 `Client ID` 與 `Secret` 提供給 Gemini CLI 或手動更新環境變數。

---

## 💬 LINE Bot 頻道設定

部署成功後，請務必完成以下設定：

1.  **設定 Webhook**：將 Webhook URL 指向您的 Cloud Run 服務網址。
2.  **關閉自動回應**：
    *   前往 [LINE Official Account Manager](https://manager.line.biz/)。
    *   進入「回應設定」。
    *   將「回應模式」設為 **「聊天機器人」**。
    *   停用 **「自動回應訊息」**，確保只有您的程式碼能處理訊息。

---

## 🛠️ 部署指令速查表 (gcloud)

| 功能 | 指令 |
| :--- | :--- |
| **部署/更新服務** | `gcloud run deploy [SERVICE_NAME] --source .` |
| **更新環境變數** | `gcloud run services update [SERVICE_NAME] --update-env-vars "KEY=VALUE"` |
| **查看即時日誌** | `gcloud beta run services logs tail [SERVICE_NAME]` |

---

## 📜 License

本專案採用 [Apache License 2.0](LICENSE)。

## 🤝 如何貢獻

這是一次 **Build With AI 2026** 的 Workshop 實作內容。歡迎提出 Issue 或 Pull Request。

感謝所有參與工作坊的開發者！

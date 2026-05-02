# LINE Bot 檔案備份機器人

這是一個功能強大的 LINE Bot，可以讓使用者輕鬆地將聊天室中的圖片、影片、音訊和檔案備份到自己的 Google Drive。它會自動整理檔案，並提供方便的查詢功能。

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

## 🚀 部署到 Google Cloud Platform

本專案已容器化 (Dockerfile)，強烈建議使用 [Google Cloud Run](https://cloud.google.com/run) 進行部署，它能提供全代管、自動擴展的無伺服器環境。

### 前置作業

1.  擁有一個 Google Cloud 帳號。
2.  安裝並設定好 [Google Cloud SDK (gcloud CLI)](https://cloud.google.com/sdk/docs/install)。
3.  一個 LINE Bot 頻道，並取得 **Channel Secret** 和 **Channel Access Token**。

### 部署步驟 (使用 Gemini CLI 與 gcloud)

本專案支援使用 **Gemini CLI** 搭配 **gcloud** 進行自動化部署。以下是本次部署的詳細流程：

#### 1. 初始化與環境檢查
首先，確保您已安裝 `gcloud` 並完成登入。

```bash
# 檢查當前 gcloud 設定
gcloud config get-value project
gcloud config get-value compute/region
```

#### 2. 設定專案與啟用服務
若尚未啟用必要服務，請執行以下指令。這會啟用 Cloud Run、Cloud Build、Firestore 與 Artifact Registry。

```bash
# 設定目標專案
gcloud config set project [YOUR_PROJECT_ID]

# 啟用必要 API
gcloud services enable firestore.googleapis.com \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  artifactregistry.googleapis.com
```

#### 3. 建立 Firestore 資料庫
Firestore 用於儲存使用者 OAuth 權限，必須使用 Native 模式：

```bash
gcloud firestore databases create --location=asia-east1 --type=firestore-native
```

#### 4. 執行首次部署 (Cloud Run)
使用 `gcloud run deploy` 直接從原始碼建置並部署。若 Google OAuth 資訊尚未取得，可先使用 `PENDING` 作為佔位符：

```bash
gcloud run deploy bwai2026 \
  --source . \
  --region asia-east1 \
  --set-env-vars "GOOGLE_CLOUD_PROJECT=[YOUR_PROJECT_ID],ChannelSecret=[LINE_SECRET],ChannelAccessToken=[LINE_TOKEN],GOOGLE_CLIENT_ID=PENDING,GOOGLE_CLIENT_SECRET=PENDING,GOOGLE_REDIRECT_URL=PENDING" \
  --allow-unauthenticated \
  --quiet
```

#### 5. 更新 Redirect URL 與 OAuth 憑證
部署完成後取得服務 URL（例如：`https://bwai2026-xxxxx.a.run.app`），接著更新環境變數：

```bash
# 更新 Redirect URL
gcloud run services update bwai2026 \
  --region asia-east1 \
  --update-env-vars "GOOGLE_REDIRECT_URL=https://[YOUR_SERVICE_URL]/oauth/callback"

# 更新 Google OAuth 憑證 (Client ID 與 Secret)
gcloud run services update bwai2026 \
  --region asia-east1 \
  --update-env-vars "GOOGLE_CLIENT_ID=[YOUR_CLIENT_ID],GOOGLE_CLIENT_SECRET=[YOUR_CLIENT_SECRET]"
```

### 🛠️ 部署指令速查表 (gcloud)
| 功能 | 指令 |
| :--- | :--- |
| **首次部署** | `gcloud run deploy [SERVICE_NAME] --source .` |
| **更新環境變數** | `gcloud run services update [SERVICE_NAME] --update-env-vars "KEY=VALUE"` |
| **查看日誌** | `gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=[SERVICE_NAME]"` |

---

## 📜 License


本專案採用 [Apache License 2.0](LICENSE)。

## 🤝 如何貢獻

我們非常歡迎任何形式的貢獻！如果您有任何建議或發現 Bug，請隨時提出 Issue。如果您想直接貢獻程式碼，請遵循以下步驟：

1.  Fork 此專案。
2.  建立您的功能分支 (`git checkout -b feature/AmazingFeature`)。
3.  提交您的變更 (`git commit -m 'Add some AmazingFeature'`)。
4.  將您的分支推送到遠端 (`git push origin feature/AmazingFeature`)。
5.  開啟一個 Pull Request。

感謝所有貢獻者的努力！

# ChatGPT 對話匯出工具

> **免責聲明：** 本工具屬於實驗性質，依現狀提供，不保證正確性、可靠性或適用於任何特定用途。工具會存取 ChatGPT 的非官方後端 API，該 API 可能隨時變更或失效。使用本工具，即表示你自行承擔使用方式所產生的一切責任。作者不對你所在地區匯出個人資料的合法性、此用途是否符合 OpenAI 服務條款，或其他法律及合規事項作出任何保證。**使用風險由你自行承擔。**

透過後端 API 批次匯出 ChatGPT 對話，適用於個人與 Teams 帳號。**支援中斷續傳**：如果權杖在匯出途中到期，只要換上新的權杖重新執行，即可從上次進度繼續。

支援功能：

- **一般對話**：ChatGPT 主要歷史記錄中的對話。
- **專案對話**：ChatGPT Projects 內的對話。
- **檔案下載**：DALL-E 圖片、Canvas 文件、使用者上傳檔案與附件。
- **深度研究**：擷取非同步研究任務的結果。
- **增強版 Markdown**：保留瀏覽結果、思考／推理內容及工具使用記錄。

## 系統需求

- Node.js 18 以上版本（使用原生 `fetch`）。

## 快速開始

> **重要：** 如果你有多個 ChatGPT 帳號，請確保只登入此次要匯出的帳號。同時登入多個帳號可能導致 ChatGPT 回傳其他帳號的資料。

### 1. 取得 Bearer 權杖

1. 在瀏覽器開啟 [ChatGPT](https://chatgpt.com)，確認已登入。
2. 按 F12 開啟開發者工具，切換至 **Network（網路）** 分頁。
3. 重新整理頁面，或點選一筆對話。
4. 篩選請求：`backend-api/conversations`。
5. 點選符合條件的請求，在 Request Headers（要求標頭）中找到 **Authorization**，複製 `Bearer` 後面的權杖，也就是以 `eyJ...` 開頭的字串。

> **提醒：** Bearer 權杖可能很快到期，建議每次匯出時重新取得。

### 2. 使用匯出工具

**安裝相依套件：**

```bash
npm install
```

**執行：**

```bash
npx export-chatgpt
```

### 3. 查看匯出結果

預設會將對話儲存至 `./exports/{user_id}`：

```text
exports/{user_id}
├── json/                          # 一般對話的 JSON
│   └── {date}_{title}_{id}.json
├── markdown/                      # 一般對話的 Markdown
│   └── {date}_{title}_{id}.md
├── files/                         # 一般對話中的檔案
│   └── {file_id}.{ext}
├── projects/                      # 依專案分開儲存的匯出內容
│   ├── {ProjectName}/
│   │   ├── json/
│   │   ├── markdown/
│   │   └── files/
│   └── project-index.json
├── conversation-index.json
└── .export-progress.json          # 中斷續傳進度
```

檔案命名格式為 `{date}_{title}_{id}.{ext}`。

## 中斷續傳

程式會自動追蹤進度：

- `exports/{user_id}/.export-progress.json` 記錄已下載的對話，以及建立索引的進度。
- 如果權杖在匯出途中到期，程式會儲存進度後結束。
- 使用新的 Bearer 權杖重新執行，即會略過已下載的對話。
- 對話索引也採逐批建立，可從上次取得的分頁繼續。

## 選項

```text
--bearer <token>        Bearer／存取權杖；也可設定 CHATGPT_BEARER_TOKEN 環境變數，未提供時會提示輸入
--token <token>         工作階段權杖，作為替代驗證方式，僅限個人帳號；也可設定 CHATGPT_SESSION_TOKEN
--account-id <id>       ChatGPT Teams 帳號 ID，可行時會從權杖自動判斷
-o, --output <dir>     輸出目錄，預設為 ./exports
--format <format>      匯出格式：json | markdown | both，預設為 both
--throttle <seconds>   API 請求之間的最短間隔，單位為秒，預設為 60
--update               重新下載並覆寫既有對話
--no-projects          略過專案對話；預設會匯出專案
--projects-only        僅匯出專案對話，略過一般對話
--no-files             略過所有檔案下載
--no-images            略過 DALL-E 圖片下載
--no-canvas            略過 Canvas 文件下載
--no-attachments       略過其他附件下載
--no-user-dir          不在輸出目錄下建立使用者 ID 子目錄
--max <n>              本次僅下載接下來的 N 筆對話，也可寫成 -N，例如 -7
--conv <ids>           僅下載指定的對話 ID，多筆以逗號分隔
--proj <ids>           僅下載指定的專案 ID，多筆以逗號分隔
-n, --non-interactive  不顯示互動提示；需提供 --bearer 或 --token
--no-summary           不顯示最後的匯出摘要
--no-donate            不顯示贊助訊息／提示
--verbose              顯示詳細的請求與回應資訊
--help                 顯示說明
```

### 透過環境變數提供權杖

如果不想每次都貼上權杖，可以設定環境變數。以下為 Bash 語法：

```bash
export CHATGPT_BEARER_TOKEN="eyJ..."
npx export-chatgpt
```

### 互動模式

唯一的互動輸入項目是 Bearer 權杖。若未提供 `--bearer`、`--token` 或對應的環境變數，程式會提示你輸入權杖。

## 使用範例

以下命令列範例沿用上游文件的 Bash 語法：

```bash
# 匯出所有內容：對話、專案、圖片、Canvas 與附件
npx export-chatgpt

# 略過專案對話
npx export-chatgpt --no-projects

# 僅匯出專案對話，不下載檔案
npx export-chatgpt --projects-only --no-files

# 僅匯出 JSON；預設同時匯出 JSON 與 Markdown
npx export-chatgpt --format json

# 指定輸出目錄
npx export-chatgpt --output ~/Documents/chatgpt-backup

# 延長請求間隔，降低觸發流量限制的機會
npx export-chatgpt --throttle 90

# 重新下載所有對話，覆寫既有內容
npx export-chatgpt --update

# 略過圖片，但保留 Canvas 與附件
npx export-chatgpt --no-images

# 權杖到期後，使用新權杖重新執行以接續進度
npx export-chatgpt

# 本次最多下載 10 筆對話
npx export-chatgpt --max 10

# 非互動模式，供腳本／CI 使用；需提供 --bearer 或如下設定環境變數
CHATGPT_BEARER_TOKEN="eyJ..." npx export-chatgpt --non-interactive
```

## Markdown 輸出

Markdown 輸出包含 YAML 前置資料（frontmatter），並可處理多種內容類型：

| 內容類型 | 呈現方式 |
|---|---|
| 文字訊息 | 標準 Markdown |
| 程式碼結果 | 圍欄式程式碼區塊 |
| 圖片／檔案 | `![image](files/{id}.ext)` 連結或 `[Image: {id}]` |
| Canvas 文件 | `![image](files/{id}.ext)` 連結 |
| 瀏覽結果 | 帶有「Browsing Result」標題的引用區塊 |
| 思考／推理內容（o1／o3） | 可展開與收合的 `<details>` 區塊 |
| 推理摘要 | 斜體摘要 |
| 深度研究結果 | 「Assistant (Deep Research: title)」標題 |
| 工具訊息 | 標示工具名稱的引用區塊 |

前置資料範例：

```yaml
---
title: "我的對話標題"
id: abc123...
create_time: 2025-01-15T10:30:00.000Z
update_time: 2025-01-15T11:00:00.000Z
model: gpt-4o
project_id: g-abc123...
---
```

## 疑難排解

### 出現「Authentication failed」，或權杖在匯出途中到期

- Bearer 權杖可能很快到期，請從開發者工具取得新的權杖。
- 確認複製了**完整**權杖，通常以 `eyJ` 開頭。
- Teams 帳號請提供 `--account-id`，或讓工具自動判斷。
- 程式會自動儲存進度，使用新權杖重新執行即可。

### 出現「No conversations found」

可能原因包括：

- **Teams 帳號未提供 `--account-id`**：Teams 工作區需要提供帳號 ID。
- 目前登入的工作區與預期不同。
- 該帳號確實沒有對話。

### 請求流量限制

若出現 429 錯誤，程式會自動等待並重試。你也可以增加請求間隔：

```bash
npx export-chatgpt --throttle 90
```

## 運作方式

1. 直接使用 Bearer 權杖進行 API 驗證，或以工作階段權杖換取存取權杖。
2. 透過 `/backend-api/conversations` 逐批取得對話清單，每頁 28 筆，每取得一頁就儲存進度。
3. 透過 `/backend-api/conversation/{id}` 下載各筆對話的完整內容，並追蹤已完成的下載。
4. 透過 `/backend-api/gizmos/snorlax/sidebar` 取得專案清單，再為各專案建立對話索引並下載；可使用 `--no-projects` 略過。
5. 掃描對話中的檔案參照，例如 `image_asset_pointer`、`canvas_asset_pointer`，並透過 `/backend-api/files/download/{id}` 下載；可使用 `--no-files` 略過。
6. 儲存為 JSON、Markdown，或同時儲存兩種格式。
7. 驗證失敗時，儲存進度後結束；重新執行會略過已完成的內容。

## 授權條款

MIT，詳見 [LICENSE](LICENSE)。


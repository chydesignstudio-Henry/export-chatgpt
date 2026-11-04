# ChatGPT 對話匯出工具

> **翻譯說明：** 這是上游 README 的繁體中文（臺灣）翻譯，說明這個專案在做什麼、有哪些功能，以及如何安裝與使用。程式碼區塊裡的指令、路徑與連結都保留原文，可以直接複製。若譯文與原文不一致，請以英文原文為準。
>
> - 原文：[README.md](README.md)
> - 英文原文（上游 README）：<https://github.com/brianjlacy/export-chatgpt/blob/main/README.md>
> - 上游儲存庫：<https://github.com/brianjlacy/export-chatgpt>

> **免責聲明：** 本工具屬於實驗性質，依現狀提供，不保證正確性、可靠性，或適用於任何特定用途。本工具會存取 ChatGPT 的非官方後端 API，而該 API 可能隨時變更或失效。使用本工具，即表示你願意自行承擔使用方式所產生的一切責任。作者不對你所在地區匯出自己的資料是否合法、此用途是否符合 OpenAI 服務條款，或其他法律與合規事項作出任何保證。**使用風險由你自行承擔。**

透過後端 API 一次匯出全部 ChatGPT 對話，個人帳號與 Teams 帳號都可以使用。**可從中斷處續傳**：如果權杖在匯出途中到期，只要換上新的權杖再執行一次，就會從上次停下的地方繼續。

支援這些內容：

- **一般對話**：ChatGPT 主要的對話紀錄。
- **專案對話**：ChatGPT Projects 裡的對話。
- **檔案下載**：DALL-E 圖片、Canvas 文件、使用者上傳的檔案與附件。
- **深度研究**：擷取非同步研究任務的結果。
- **增強版 Markdown**：保留瀏覽結果、思考／推理內容，以及工具使用紀錄。

## 系統需求

- Node.js 18 以上（使用 Node.js 內建的 `fetch`）。

## 快速開始

> **重要：** 如果你有多個 ChatGPT 帳號，請確認目前只登入這次要匯出的帳號。同時登入多個帳號時，ChatGPT 可能會回傳另一個帳號的資料。

### 1. 取得 Bearer 權杖

1. 在瀏覽器開啟 https://chatgpt.com ，並確認已經登入。
2. 按 F12 開啟開發人員工具，切換到 **Network（網路）** 分頁。
3. 重新整理頁面，或點選一筆對話。
4. 篩選請求：`backend-api/conversations`。
5. 點選符合的請求，在 Request Headers（要求標頭）裡找到 **Authorization**，複製 `Bearer` 後面的那一段權杖，也就是以 `eyJ...` 開頭的字串。

> **注意：** Bearer 權杖可能很快就會到期。每次匯出前，最好重新取得一次。

### 2. 使用匯出工具

**安裝：**

```bash
npm install
```

**執行：**

```bash
npx export-chatgpt
```

### 3. 查看匯出結果

預設會把對話存到 `./exports/{user_id}`：

```
exports/{user_id}
├── json/                          # 一般對話的 JSON
│   └── {date}_{title}_{id}.json
├── markdown/                      # 一般對話的 Markdown
│   └── {date}_{title}_{id}.md
├── files/                         # 一般對話中的檔案
│   └── {file_id}.{ext}
├── projects/                      # 依專案分開存放的匯出內容
│   ├── {ProjectName}/
│   │   ├── json/
│   │   ├── markdown/
│   │   └── files/
│   └── project-index.json
├── conversation-index.json
└── .export-progress.json          # 續傳進度
```

檔名格式為 `{date}_{title}_{id}.{ext}`。

## 中斷後續傳

程式會自動記錄進度：

- `exports/{user_id}/.export-progress.json` 記錄哪些對話已經下載，以及索引建立到哪裡。
- 如果權杖在匯出途中到期，程式會先儲存進度再結束。
- 換上新的 Bearer 權杖再執行一次即可；已經下載的對話會略過。
- 對話索引也是逐批建立，會從上次取到的分頁繼續。

## 選項

```
--bearer <token>        Bearer／存取權杖；也可設定 CHATGPT_BEARER_TOKEN 環境變數，未提供時會提示輸入
--token <token>         工作階段權杖，作為替代驗證方式，僅限個人帳號；也可設定 CHATGPT_SESSION_TOKEN
--account-id <id>       ChatGPT Teams 帳號 ID，可行時會從權杖自動判斷
-o, --output <dir>      輸出目錄，預設為 ./exports
--format <format>       匯出格式：json | markdown | both，預設為 both
--throttle <seconds>    API 請求之間的最短間隔，單位為秒，預設為 60
--update                重新下載並覆寫既有對話
--no-projects           略過專案對話；預設會匯出專案
--projects-only         只匯出專案對話，略過一般對話
--no-files              略過所有檔案下載
--no-images             略過 DALL-E 圖片下載
--no-canvas             略過 Canvas 文件下載
--no-attachments        略過其他附件下載
--no-user-dir           不要在輸出目錄下再建立使用者 ID 子目錄
--max <n>               這次只下載接下來的 N 筆對話，也可寫成 -N，例如 -7
--conv <ids>            只下載指定的對話 ID，多筆以逗號分隔
--proj <ids>            只下載指定的專案 ID，多筆以逗號分隔
-n, --non-interactive   不顯示任何互動提示；必須提供 --bearer 或 --token
--no-summary            不要顯示最後的匯出摘要
--no-donate             不要顯示贊助訊息／提示
--verbose               顯示詳細的請求與回應資訊
--help                  顯示說明
```

### 透過環境變數提供權杖

若不想每次都貼上權杖，可以設定環境變數：

```bash
export CHATGPT_BEARER_TOKEN="eyJ..."
npx export-chatgpt
```

### 互動模式

唯一會請你輸入的是 Bearer 權杖。如果沒有提供 `--bearer`、`--token`，也沒有設定對應的環境變數，程式會提示你輸入權杖。

## 使用範例

```bash
# 匯出全部內容：對話、專案、圖片、Canvas 與附件
npx export-chatgpt

# 略過專案對話
npx export-chatgpt --no-projects

# 只匯出專案對話，不下載檔案
npx export-chatgpt --projects-only --no-files

# 只匯出 JSON；預設會同時匯出 JSON 與 Markdown
npx export-chatgpt --format json

# 匯出到指定目錄
npx export-chatgpt --output ~/Documents/chatgpt-backup

# 拉長請求間隔，降低觸發速率限制的機會
npx export-chatgpt --throttle 90

# 重新下載全部對話，覆寫既有內容
npx export-chatgpt --update

# 略過圖片，但保留 Canvas 與附件
npx export-chatgpt --no-images

# 權杖到期後，換新權杖再執行一次即可續傳
npx export-chatgpt

# 這次最多下載 10 筆對話
npx export-chatgpt --max 10

# 非互動模式，給腳本或 CI 使用；必須提供 --bearer，或像下面這樣設定環境變數
CHATGPT_BEARER_TOKEN="eyJ..." npx export-chatgpt --non-interactive
```

## Markdown 輸出

Markdown 輸出含有 YAML frontmatter（前置資料），並能處理多種內容類型：

| 內容類型 | 呈現方式 |
|---|---|
| 文字訊息 | 標準 Markdown |
| 程式碼結果 | 圍欄式程式碼區塊 |
| 圖片／檔案 | `![image](files/{id}.ext)` 連結，或 `[Image: {id}]` |
| Canvas 文件 | `![image](files/{id}.ext)` 連結 |
| 瀏覽結果 | 帶有「Browsing Result」標題的引用區塊 |
| 思考／推理內容（o1／o3） | 可展開與收合的 `<details>` 區塊 |
| 推理摘要 | 斜體摘要 |
| 深度研究結果 | 「Assistant (Deep Research: title)」標題 |
| 工具訊息 | 標示工具名稱的引用區塊 |

前置資料範例：

```yaml
---
title: "My conversation title"
id: abc123...
create_time: 2025-01-15T10:30:00.000Z
update_time: 2025-01-15T11:00:00.000Z
model: gpt-4o
project_id: g-abc123...
---
```

## 疑難排解

### 出現「Authentication failed」，或權杖在匯出途中到期

- Bearer 權杖可能很快就會到期，請從開發人員工具取得新的權杖。
- 確認複製的是**完整**權杖，通常以 `eyJ` 開頭。
- Teams 帳號請加上 `--account-id`，或讓工具自動判斷。
- 進度會自動儲存，換新權杖再執行一次即可。

### 出現「No conversations found」

通常是下面其中一種情況：

- **Teams 帳號卻沒有提供 `--account-id`**：Teams 工作區必須帶上帳號 ID。
- 目前登入的工作區與預期不同。
- 這個帳號本來就沒有對話。

### 速率限制

如果出現 429 錯誤，程式會自動等待並重試。你也可以把請求間隔拉長：

```bash
npx export-chatgpt --throttle 90
```

## 運作方式

1. 直接使用 Bearer 權杖呼叫 API；若提供的是工作階段權杖，則先換成存取權杖。
2. 透過 `/backend-api/conversations` 逐批取得對話清單，每頁 28 筆，每取得一頁就儲存進度。
3. 透過 `/backend-api/conversation/{id}` 下載每筆對話的完整內容，並記錄哪些已經下載完成。
4. 透過 `/backend-api/gizmos/snorlax/sidebar` 取得專案清單，再為每個專案建立索引並下載對話；可用 `--no-projects` 略過。
5. 掃描對話裡的檔案參照，例如 `image_asset_pointer`、`canvas_asset_pointer`，再透過 `/backend-api/files/download/{id}` 下載；可用 `--no-files` 略過。
6. 存成 JSON、Markdown，或兩種都存。
7. 驗證失敗時，先儲存全部進度再結束；再次執行會略過已經完成的部分。

## 授權條款

MIT。授權全文見 [LICENSE](LICENSE)。

# AI 虛擬人 Widget（Live2D 語音助理）

> 一個可以「一行 `<script>` 嵌入任何網站」的右下角 Live2D 語音 AI 虛擬人元件。
> 你可以對她說話，她會聽懂、回答、開口說話並即時對嘴。語音與大腦大多在瀏覽器本機運行。

🔗 線上 Demo：<https://ai-avatar-bot-two.vercel.app>（請用 **Chrome 桌機**開啟）

---

## ✨ 功能

- **Live2D 動畫角色**＋即時**對嘴**（依實際音量驅動嘴型）
- **語音輸入（STT）**：瀏覽器內建語音辨識
- **語音輸出（TTS）**：神經語音（自然女聲），失敗自動退回瀏覽器內建語音
- **大腦**：知識庫檢索（即時、零金鑰）＋可選的瀏覽器內 LLM（WebLLM，零金鑰）
- **一行嵌入**：`embed.js` 動態建立 iframe widget，不干擾宿主網站

## 🧱 架構

| 檔案 | 說明 |
|---|---|
| `index.html` | 示範 landing 頁（嵌入 widget） |
| `widget.html` | iframe 內的虛擬人本體（Live2D／STT／TTS／對嘴／LLM／檢索） |
| `embed.js` | 一行嵌入載入器（建 iframe ＋ `postMessage` 父子溝通 ＋ 對外 `window.AvatarWidget` API） |
| `knowledge.js` | 知識庫（FAQ 範例內容，可自行替換） |
| `demo-host.html` | 模擬「客戶網站」的示範頁 |
| `api/tts.js` | Vercel serverless function：取得神經語音 MP3 |
| `m1-standalone.html` | 早期里程碑的單檔版（純參考，可刪） |

純前端（HTML/JS）＋ **一支 serverless function**，無資料庫。

## 🚀 本機跑法

神經語音需要 `/api/tts` 這支 function，所以本機請用 Vercel CLI：

```bash
npm install
npm i -g vercel
vercel dev          # http://localhost:3000
```

> 若只用 `python -m http.server` 之類的純靜態伺服器：除了「神經語音」之外都能跑（神經語音會自動退回瀏覽器內建語音）。

部署：`vercel --prod`（會自動安裝相依、部署 function）。

## 🌐 瀏覽器需求

- **Chrome / Chromium 桌機**（語音辨識 `webkitSpeechRecognition` 僅 Chromium 支援）
- 想啟用 🧠 瀏覽器內 LLM：需 **WebGPU**（Chrome 113+）
- 麥克風（語音輸入用）；TTS 與 LLM 需要 **HTTPS**（或 localhost）

## ⚙️ 設定

- 語音聲線：`widget.html` 內的 `NEURAL_VOICE`（預設 `zh-TW-HsiaoChenNeural`）。
- `/api/tts` 來源限制：function 預設只允許「與自己同網域」的來源呼叫；可用環境變數 `TTS_ALLOWED_HOSTS`（逗號分隔）加入其他允許的網域。

---

## 📦 第三方資產與授權（**請務必先讀**）

本專案自己的程式碼採 **MIT**（見 `LICENSE`）。但它**相依**以下第三方，各有各的授權，**不在 MIT 範圍內**：

| 來源 | 授權 / 注意 |
|---|---|
| **Live2D Cubism Core**（CDN `cubism.live2d.com`） | **專有授權**（Live2D Proprietary Software License）。非開源，商用/再散佈須自行確認 Live2D 條款。 |
| **Haru 範例模型**（CDN，pixi-live2d-display 測試資產） | Live2D **Free Material License**，**僅供範例**。正式上線請換成你自有合法授權的模型。本 repo 不夾帶模型檔，採 CDN 引用。 |
| **pixi.js / pixi-live2d-display** | MIT |
| **@mlc-ai/web-llm**（WebLLM） | Apache-2.0；下載的模型權重各有授權（Qwen2.5 為其自身條款） |
| **msedge-tts**（`api/tts.js` 用） | 套件本身開源，但它連線的是**微軟 Edge 朗讀的「非官方」語音端點**（見下方風險） |

## ⚠️ 風險與限制揭露

- **TTS 走非官方端點**：`/api/tts` 透過 `msedge-tts` 連到微軟 Edge 朗讀的**非官方**語音服務（免帳號免金鑰）。**這不是官方支援、可能違反微軟服務條款、隨時可能失效或被封鎖。** 正式環境建議改接官方 **Azure Speech** 或其他有授權的 TTS。失效時 widget 會自動退回瀏覽器內建語音。
- **`/api/tts` 是公開端點**：預設只做「同網域來源檢查」＋輸入長度上限，**沒有完整限流**。自架者請務必在 **Vercel 專案層開啟用量上限（Spend Management）/ Firewall 限流**，避免被當免費 TTS proxy 灌爆帳單。
- **語音辨識會送到雲端**：`webkitSpeechRecognition` 在 Chrome 下會把**麥克風音訊上傳到瀏覽器廠商（Google）**處理，**並非本機辨識**。請告知你的使用者。
- **LLM 在本機**：WebLLM 模型下載後在使用者瀏覽器內運行，問答內容不外傳；首次需下載約 1GB 模型。

## 🔐 隱私（資料流向）

| 功能 | 資料去哪 |
|---|---|
| 語音輸入（STT） | 麥克風音訊 → 瀏覽器廠商雲端（Chrome 為 Google） |
| 語音輸出（TTS） | 要朗讀的文字 → 你的 `/api/tts` → 微軟非官方 TTS 端點 |
| 大腦（LLM／檢索） | **本機**，不外傳 |

本專案不自行儲存使用者資料；但部署平台（如 Vercel）預設可能保留 function 的請求日誌。

## 🩺 免責聲明

本專案內建的「健檢 FAQ」**僅為示範內容、皆為虛構**，**不構成任何醫療建議**。請勿依賴本工具做健康判斷，實際情況請諮詢專業醫師。`knowledge.js` 中的機構名稱、電話等均為範例，**不對應任何真實機構**。

## 🤝 貢獻

歡迎開 issue / PR。（若日後考慮商業授權，接受外部 PR 前可先設置 CLA。）

## 📄 授權

MIT — 見 [`LICENSE`](./LICENSE)。第三方資產不在此授權範圍，見上方表格。

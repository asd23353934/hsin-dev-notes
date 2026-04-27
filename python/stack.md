# Python Stack

> 事實型紀錄：套件版本、執行環境。
> Claude Code 在產生 Python 程式碼前必須先看這份，依版本給對應 API。
> 最後更新：2026-04-27（基準專案：shop-watcher worker）

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Python | **3.12** | 用 type hint 新語法（`list[T]`、`X \| None`） |
| 排程 | GitHub Actions cron | shop-watcher 用 `0 * * * *`（每小時） |
| 容器 | Playwright headless Chromium | CI 內 `playwright install chromium` |

---

## 套件

| 套件 | 版本 | 用途 |
|------|------|------|
| `playwright` | **1.51.0** | headless 瀏覽器，動態頁面爬取 |
| `httpx` | **0.28.1** | async HTTP client，取代 requests |
| `beautifulsoup4` | **4.12.3** | HTML parser |
| `python-dotenv` | **1.0.1** | 讀 `.env`（local 開發） |

---

## 沒採用的套件（與決策原因）

- **`requests`** → 改用 `httpx`，原生支援 async，API 幾乎相同
- **`scrapy`** → 過度工程；單純 keyword scan 用 `playwright + httpx + bs4` 即可
- **`pydantic`** → 目前用 `dataclass`，等需求複雜（schema migration、JSON serialization）再評估
- **`celery`** → 排程交給 GitHub Actions cron，不自架 worker 池

---

## 已驗證的相容性

- `playwright 1.51` ✓ Python 3.12 ✓ Chromium（內建版本）
- `httpx 0.28` ✓ Python 3.8+，async/await 原生支援
- `beautifulsoup4 4.12` ✓ Python 3.6+

---

## 變更歷史

- **2026-04-27** 建立檔案，依 shop-watcher 專案實際版本寫入（Python 3.12、Playwright 1.51、httpx 0.28、bs4 4.12）

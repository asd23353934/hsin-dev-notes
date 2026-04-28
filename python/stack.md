# Python Stack

> 事實型紀錄：套件版本、執行環境。
> Claude Code 在產生 Python 程式碼前必須先看這份，依版本給對應 API。
> **角色**：新專案起手套餐 / 預設假設。實際專案內以該專案 `requirements.txt` / `pyproject.toml` 為準（見 `CLAUDE.md`「版本來源優先順序」）。
> 最後更新：2026-04-28（PyPI latest stable）

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Python | **3.12+** | 用 type hint 新語法（`list[T]`、`X \| None`） |
| 排程 | GitHub Actions cron / 自管 worker | 視情境 |
| 容器 | Playwright headless Chromium | CI 內 `playwright install chromium` |

---

## 套件

| 套件 | 版本 | 用途 |
|------|------|------|
| `playwright` | **1.58.0** | headless 瀏覽器，動態頁面爬取 |
| `httpx` | **0.28.1** | async HTTP client，取代 requests |
| `beautifulsoup4` | **4.14.3** | HTML parser |
| `python-dotenv` | **1.2.2** | 讀 `.env`（local 開發） |

---

## 沒採用的套件（與決策原因）

- **`requests`** → 改用 `httpx`，原生支援 async，API 幾乎相同
- **`scrapy`** → 過度工程；單純 keyword scan 用 `playwright + httpx + bs4` 即可
- **`pydantic`** → 目前用 `dataclass`，等需求複雜（schema migration、JSON serialization）再評估
- **`celery`** → 排程交給 GitHub Actions cron 或外部 scheduler，不自架 worker 池

---

## 已驗證的相容性

- `playwright 1.58` ✓ Python 3.9+ ✓ Chromium（內建版本）
- `httpx 0.28` ✓ Python 3.9+，async/await 原生支援
- `beautifulsoup4 4.14` ✓ Python 3.7+
- `python-dotenv 1.2` ✓ Python 3.9+

---

## 變更歷史

- **2026-04-28** 政策統一為 PyPI latest stable（先前是 shop-watcher 專案 pinned 版本）。playwright 1.51→1.58、bs4 4.12→4.14、python-dotenv 1.0→1.2。Python 基準改寫 3.12+。
- **2026-04-27** 建立檔案，依 shop-watcher 專案實際版本寫入（Python 3.12 / Playwright 1.51 / httpx 0.28 / bs4 4.12）

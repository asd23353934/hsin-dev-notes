# Python Conventions

> Python 專案的寫法慣例（爬蟲 / Worker 為主）。Claude Code 在產出 Python 程式碼時必須遵守。
> 最後更新：2026-04-27

---

## Scraper 模組規範

### 入口函式

每個 scraper 以 **module-level async function** 為主要入口：

```python
import logging
from src.watchers.base import WatcherItem

logger = logging.getLogger(__name__)


async def scrape_<platform>(
    page,
    keyword: str,
    *,
    min_price: float | None = None,
    max_price: float | None = None,
) -> list[WatcherItem]:
    """Scrape <platform> for keyword. Returns [] on error, never raises."""
    try:
        # ...
        return _apply_price_filter(items, min_price, max_price)
    except Exception:
        logger.exception("scrape_%s failed for %r", "<platform>", keyword)
        return []
```

關鍵原則：

- 命名固定 `scrape_<platform>`
- 第一個參數是 Playwright `page`（或 `httpx.AsyncClient`，視 scraper）
- **絕對不 raise**：錯誤一律 log 後回傳空 list，避免一個爬蟲掛掉影響其他平台
- Logger 用 `logging.getLogger(__name__)`，**module level 宣告**，不在函式內反覆 get
- 使用 `logger.exception()` 而非 `logger.error()` 來保留 traceback

---

## 共用 Helper

跨 scraper 重複的邏輯（價格解析、價格範圍過濾、DOM 結構檢查）抽到 `_xxx.py` 檔，前綴底線標示「scrapers 內部共用」：

- `_price_utils.py`：`_parse_price()` / `_extract_price_text()` / `_apply_price_filter()`
- `_dom_signal.py`：DOM 健康度訊號

**規則**：

- 每個 scraper 都用 `_apply_price_filter()`，不自己重複寫過濾邏輯
- 加新 scraper 時若發現有重複邏輯，回去重構共用 helper，不複製貼上

---

## 網路逾時

| 工具 | 逾時 | 單位 |
|------|------|------|
| Playwright | `timeout=15_000` | ms |
| httpx | `timeout=15` | s |

- 統一 15 秒超時，避免某個平台拖累整個 cycle
- 不要為了「保險」設 60 秒以上 — 排程是每小時一輪，慢比爆掉還糟
- 真要 retry，用 exponential backoff，不在 timeout 上加碼

---

## 型別 hint

- Python 3.10+ 起一律用新語法：

  ```python
  # 對
  items: list[WatcherItem] = []
  result: dict[str, int] = {}
  value: str | None = None

  # 錯（過時）
  from typing import List, Dict, Optional
  items: List[WatcherItem] = []
  ```

- 函式簽名一定有型別；內部變數視情況加（複雜邏輯加、簡單迴圈不加）
- 回傳 `list[X]` 寫清楚 X，不寫 `list` 不寫 `List`

---

## Async / Await

- I/O 密集的爬蟲一律 async；CPU 密集（極少）才考慮 `asyncio.to_thread`
- 多個獨立爬蟲跑 `asyncio.gather(*tasks, return_exceptions=True)`，**不串行 await**
- `return_exceptions=True` 是必要的：搭配 scraper 「不 raise」原則，這裡只是雙保險
- 開資源（page、client）用 `async with`，不要手動 `close()`

---

## 錯誤處理

- 一律 `try/except Exception`，**不裸 except**（會吞 KeyboardInterrupt）
- 用 `logger.exception()` 保留 traceback；`logger.error()` 只在已經知道根因時用
- 不寫無意義的 `raise` re-throw — 要嘛處理掉，要嘛讓它原樣往上拋

---

## 命名

| 對象 | 規則 | 範例 |
|------|------|------|
| 模組檔 | snake_case | `yahoo_auction.py`、`_price_utils.py` |
| 內部模組（共用 helper） | 前綴 `_` | `_price_utils.py`、`_dom_signal.py` |
| 類別 | PascalCase | `WatcherItem`、`BaseWatcher` |
| 函式 / 變數 | snake_case | `scrape_ruten`、`apply_price_filter` |
| 內部函式 | 前綴 `_` | `_parse_price` |
| 常數 | UPPER_SNAKE | `CANARY_KEYWORDS`、`MAX_RETRIES` |

---

## Logging

- module level 宣告 logger：`logger = logging.getLogger(__name__)`
- 訊息用 lazy `%` 格式，**不要** f-string（避免 disabled level 也計算字串）：

  ```python
  # 對
  logger.info("scraped %d items for %r", len(items), keyword)

  # 錯
  logger.info(f"scraped {len(items)} items for {keyword!r}")
  ```

- 等級分配：
  - `DEBUG`：每筆 item / 每個請求細節
  - `INFO`：scraper 開始 / 結束、抓到幾筆
  - `WARNING`：DOM 結構異常、空結果但無錯誤
  - `ERROR` / `EXCEPTION`：抓取失敗、解析錯誤

---

## 檔案結構（爬蟲類專案參考）

```
src/
├── scrapers/
│   ├── __init__.py
│   ├── _price_utils.py       # 共用 helper
│   ├── _dom_signal.py
│   ├── ruten.py              # 各平台爬蟲
│   ├── pchome.py
│   └── ...
├── watchers/
│   └── base.py               # BaseWatcher / WatcherItem dataclass
├── api_client.py             # 對外 API client
├── canary.py                 # 健康監控
└── scheduler.py              # 主排程邏輯
```

入口檔放專案根目錄：

- `run_once.py` → CI 一次性執行
- `main.py` → local 持續循環

---

## 待補項目

- [ ] 測試慣例（pytest 模板、fixture 結構）
- [ ] dataclass vs pydantic 何時切換
- [ ] retry 策略（tenacity / 自寫 backoff）
- [ ] CI 內 Playwright 安裝最佳化

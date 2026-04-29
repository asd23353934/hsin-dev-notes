# PostgreSQL 效能事件

> 具體效能優化案例：前後 EXPLAIN、量化結果、改動內容。
> 「效能優化原則 / 通用判斷」放 [conventions.md](conventions.md)；本檔記**事件 + 數據**。
> 最後更新：2026-04-29

---

## 紀錄模板

```markdown
### 主題（例：訂單列表頁載入 2s → 80ms）
- **適用版本**：PostgreSQL 14-16（**必填**）
- **日期**：YYYY-MM-DD
- **環境**：表 row 數、現有索引、硬體 / 雲端規格
- **症狀**：什麼慢、頻率、p95 / p99
- **EXPLAIN（前）**：關鍵步驟（Seq Scan / Sort / Hash Join cost）
- **改動**：index / query rewrite / partition / config 調整
- **EXPLAIN（後）**：
- **結果**：XXms → YYms（rows scanned XX → YY）
- **學到**：可推廣的原則（可選）
- **參考**：issue / 文件連結（可選)
```

> **與 errors.md 差別**：errors 記「壞了 → 修好」、performance 記「能跑但慢 → 變快」。
> 兩者都可能用到 EXPLAIN，但 performance 強調「前後量化對比」，errors 強調「原因 → 解法」。

---

## 紀錄

_尚無紀錄。第一次優化解完後依模板補進來。_

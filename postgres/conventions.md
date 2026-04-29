# PostgreSQL Conventions

> 「我們／我個人」的 PG 慣例與決議。格式：規則 → 範例 → 理由。
> 等實際決議出現後再寫入，不主動填官方文件內容（依 [_global/rules.md](../_global/rules.md) 第 9 條）。

---

## 預期會記錄的範圍（提醒，非待辦）

- Schema 設計：命名、PK/FK、timestamp 欄位、軟刪除策略
- 索引策略：B-tree vs GIN、partial index 何時用、多欄位索引順序、index-only scan 條件
- JSONB vs columns 取捨
- 連線池：pgBouncer / 應用層 pool size / transaction vs session pooling
- Migration 工具與流程（與 ORM 相關）
- Backup / restore 流程

> **效能事件紀錄**寫到 [performance.md](performance.md)，本檔只放「通用判斷原則 / 規則」。

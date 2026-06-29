# PostgreSQL Conventions

> 「我們／我個人」的 PG 慣例與決議。格式：規則 → 範例 → 理由。
> 等實際決議出現後再寫入，不主動填官方文件內容（依 [_global/rules.md](../_global/rules.md) 第 9 條）。

---

## Migration

DB migration 的跨技術棧通用紀律（已套用不可改、可逆、檔必 commit、破壞性變更分階段）見 [_global/rules.md](../_global/rules.md) §17。

PG / Prisma 專屬決議（命名規範、index 策略、RLS、partition 等）待實際專案出現再依「規則 → 範例 → 理由」格式補。

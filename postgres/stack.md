# PostgreSQL Stack

> 事實型紀錄：主版、driver、ORM 搭配。
> 角色：新專案起手套餐 / 預設假設。實際專案內以該專案 `docker-compose.yml` / `package.json` 為準（見 [`CLAUDE.md`](../CLAUDE.md)「版本來源優先順序」）。
> 最後更新：2026-05-07

---

## 主版

| 項目 | 版本 | 備註 |
|------|------|------|
| PostgreSQL | **16-alpine** | 容器化部署主力；2023-09 GA、支援至 2028-11 |

> **大版本選擇**：PG 16 為當前 production 主流選擇（穩定 + ecosystem 支援廣）。新專案若無特殊需求沿用即可；要 17 / 18 的新功能再個案決定，stack.md 不主動建議升級。

---

## Node.js 端 driver / adapter

搭配 `_shared` 與 `nextjs/stack.md` 使用：

| 套件 | 版本 | 備註 |
|------|------|------|
| `pg` | **^8.20.0** | node-postgres 原生 driver；Prisma 7 driver adapter 走它 |
| `@prisma/adapter-pg` | **^7.8.0** | Prisma 7 起 driver adapter GA，可走原生 `pg` 連線；版本對齊 `@prisma/client` |

> **連線串慣例**：runtime 用 pooled URL（`DATABASE_URL`，走 PgBouncer / Neon pooler），migration 用 direct URL（`DIRECT_URL`，無 pooler）。Prisma 7 兩條都吃。

---

## 容器化部署慣例

| 項目 | 開發 | 生產 |
|------|------|------|
| Image | `postgres:16-alpine` | 同 |
| Host port | **5433**（避開本機 5432） | docker network internal `5432`，不對外 |
| Healthcheck | 可省 | `pg_isready -U <user>`，10s interval |
| Volume | named volume（`pgdata`） | 同；備份走 `pg_dump` cron / managed backup |

---

## 已驗證的相容性

寫入此檔時依 [`_global/rules.md`](../_global/rules.md) 第 7 條完成驗證：

- PostgreSQL 16 ✓ alpine 官方 image stable
- `pg` 8.20.0 ✓ Node `>=16`
- `@prisma/adapter-pg` 7.8.0 ✓ `@prisma/client` 7.8.x（peer 鎖同 major）

---

## 變更歷史

- **2026-05-07** 初版填入。從 em 醫美管理系統（PG 16-alpine + pg 8.20 + @prisma/adapter-pg 7.5）實證版本順勢同步。

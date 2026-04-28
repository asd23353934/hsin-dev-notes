# Next.js Stack

> 事實型紀錄：套件版本、執行環境。
> Claude Code 在產生 Next.js 程式碼前必須先看這份，依版本給對應 API。
> **角色**：新專案起手套餐 / 預設假設。實際專案內以該專案 `package.json` 為準（見 `CLAUDE.md`「版本來源優先順序」）。
> 最後更新：2026-04-28（npm registry latest stable）

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Node.js | **24.x** LTS（或 22.12+ / 20.19+） | Prisma 7 engine：`^20.19 \|\| ^22.12 \|\| >=24.0` |
| 套件管理 | npm | 沿用專案習慣（pnpm 亦可） |
| 部署 | Vercel | App Router；`next dev --turbopack` / `next build --turbopack` |

---

## 核心框架

| 套件 | 版本 | 備註 |
|------|------|------|
| `next` | **16.2.4** | App Router；React 19.x peer |
| `react` | **19.2.5** | Server Components、`use()` 已 GA |
| `react-dom` | **19.2.5** | 與 react 同版 |
| `typescript` | **^5** | strict mode |
| `eslint` | **^9** | flat config |
| `eslint-config-next` | **16.2.4** | 對齊 Next.js 版本 |

---

## 認證 / 資料庫

| 套件 | 版本 | 備註 |
|------|------|------|
| `next-auth` | **5.0.0-beta.31** | v5 仍 beta；`auth()` 統一入口；peer `next ^14 \|\| ^15 \|\| ^16`、`react ^18 \|\| ^19` |
| `@prisma/client` | **^7.8.0** | runtime client；**Prisma 7 起預設使用 `prisma-client` generator**（非 `prisma-client-js`），import 路徑可能改為自訂 output 路徑 |
| `prisma` | **^7.8.0** | CLI / migration；`postinstall` 跑 `prisma generate` |

> **Prisma 5 → 7 重點變動**（升級新專案 / 既有專案前先看）：
> - 預設 generator 從 `prisma-client-js` 改為 `prisma-client`，需在 `schema.prisma` 內指定 output 路徑
> - Driver Adapters GA（PostgreSQL / MySQL / SQLite / D1 等可走原生 driver）
> - 既有 Prisma 5 專案不要直接 bump，先看 [migration guide](https://www.prisma.io/docs/orm/more/upgrade-guides)

---

## UI / 樣式

| 套件 | 版本 | 備註 |
|------|------|------|
| `tailwindcss` | **4.2.4** | v4 CSS-first；寫法見 `_shared/tailwind.md` |
| `@tailwindcss/postcss` | **4.2.4** | Next.js 走 PostCSS 路徑 |
| `tw-animate-css` | **1.4.0** | Tailwind v4 相容的 animate utilities |
| `tailwind-merge` | **3.5.0** | `cn()` helper 必備 |
| `class-variance-authority` | **0.7.1** | shadcn variant 機制 |
| `clsx` | **2.1.1** | className 條件組合 |
| `@base-ui/react` | **1.4.1** | Radix 替代品，shadcn 新版底層 |
| `shadcn` | **4.5.0** | CLI；元件 copy 進 `components/ui/` |
| `lucide-react` | **1.11.0** | icon 庫 |
| `recharts` | **3.8.1** | 圖表（v3 起 React 19 原生支援） |
| `sonner` | **2.0.7** | toast |
| `next-themes` | **0.4.6** | dark mode 切換 |

---

## 服務整合

| 套件 | 版本 | 備註 |
|------|------|------|
| `resend` | **6.12.2** | Email 發送 SDK |

---

## 已驗證的相容性

寫入此檔時依 [_global/rules.md 第 7 條](../_global/rules.md) 完成驗證：

- Next.js 16 ✓ React 19.2（peer `^18.2.0 || ^19.0.0`）
- Prisma 7 ✓ Node `^20.19 || ^22.12 || >=24.0`
- next-auth 5.0.0-beta.31 ✓ Next 16、React 19
- @base-ui/react 1.4.1 ✓ React 19（peer `^17 || ^18 || ^19`）
- recharts 3.8.1 ✓ React 19
- Tailwind v4 ✓ Next.js 16（`@tailwindcss/postcss` 路徑）

---

## 變更歷史

- **2026-04-28** 政策統一為 npm latest stable（先前是 shop-watcher 專案 pinned 版本）。Next 15→16、Prisma 5→7、recharts 2→3、@base-ui/react 1.3→1.4、shadcn 4.1→4.5、lucide-react 1.7→1.11、next-auth beta.30→beta.31、resend 6.10→6.12。Node 基準改寫 24 LTS。
- **2026-04-27** 建立檔案，依 shop-watcher 專案實際版本寫入（Next 15.5.14 / Prisma 5.22 / Tailwind v4 / next-auth 5.0.0-beta.30 / shadcn 4.x with `@base-ui/react`）

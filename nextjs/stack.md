# Next.js Stack

> 事實型紀錄：套件版本、執行環境。
> Claude Code 在產生 Next.js 程式碼前必須先看這份，依版本給對應 API。
> 最後更新：2026-04-27（基準專案：shop-watcher）

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Node.js | 20+ LTS | `@types/node ^20`；Vercel 預設 Node 20 |
| 套件管理 | npm | 沿用專案習慣 |
| 部署 | Vercel | App Router，使用 turbopack（`next dev --turbopack`、`next build --turbopack`） |

---

## 核心框架

| 套件 | 版本 | 備註 |
|------|------|------|
| `next` | **15.5.14** | App Router（非 Pages Router） |
| `react` | **19.1.0** | 已 GA；可用 `use()` / Server Components |
| `react-dom` | **19.1.0** | 與 react 同版 |
| `typescript` | **^5** | strict mode |
| `eslint` | **^9** | flat config |
| `eslint-config-next` | **15.5.14** | 對齊 Next.js 版本 |

---

## 認證 / 資料庫

| 套件 | 版本 | 備註 |
|------|------|------|
| `next-auth` | **5.0.0-beta.30** | v5 仍為 beta；用 `auth()` helper 取代 `getServerSession()` |
| `@prisma/client` | **^5.22.0** | runtime client |
| `prisma` | **^5.22.0** | CLI / migration；`postinstall` 跑 `prisma generate` |

> **注意**：next-auth v5 仍是 beta，但 API 已穩定，主流專案開始採用。重大改動為 `auth()` 統一入口、middleware 簡化、Edge runtime 相容性提升。

---

## UI / 樣式

| 套件 | 版本 | 備註 |
|------|------|------|
| `tailwindcss` | **^4** | v4 CSS-first；寫法見 `_shared/tailwind.md` |
| `@tailwindcss/postcss` | **^4** | Next.js 走 PostCSS 路徑 |
| `tw-animate-css` | **^1.4.0** | Tailwind v4 相容的 animate utilities |
| `tailwind-merge` | **^3.5.0** | `cn()` helper 必備 |
| `class-variance-authority` | **^0.7.1** | shadcn variant 機制 |
| `clsx` | **^2.1.1** | className 條件組合 |
| `@base-ui/react` | **^1.3.0** | Radix 替代品，shadcn 新版底層 |
| `shadcn` | **^4.1.2** | CLI；元件直接 copy 進 `components/ui/` |
| `lucide-react` | **^1.7.0** | icon 庫 |
| `recharts` | **^2.15.4** | 圖表 |
| `sonner` | **^2.0.7** | toast |
| `next-themes` | **^0.4.6** | dark mode 切換 |

---

## 服務整合

| 套件 | 版本 | 備註 |
|------|------|------|
| `resend` | **^6.10.0** | Email 發送 SDK |

---

## 已驗證的相容性

寫入此檔時的相容性確認（依 [_global/rules.md 第 7 條](../_global/rules.md)）：

- Next.js 15 ✓ React 19（Next 15 起原生支援）
- React 19 ✓ TypeScript 5
- Prisma 5 ✓ Node 18+
- Tailwind v4 ✓ Next.js 15（`@tailwindcss/postcss` 路徑）
- next-auth v5 beta ✓ Next.js 15 App Router
- shadcn 4.x（`@base-ui/react` 底層）✓ React 19

---

## 變更歷史

- **2026-04-27** 建立檔案，依 shop-watcher 專案實際版本寫入（Next.js 15.5.14、React 19.1.0、Prisma 5.22、Tailwind v4、next-auth 5.0.0-beta.30、shadcn 4.x with `@base-ui/react`）

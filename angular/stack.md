# Angular Stack

> 事實型紀錄：套件版本、執行環境。
> Claude Code 在產生程式碼前必須先看這份，依版本給對應 API。
> 最後更新：2026-04-27（最新穩定版基準，依 npm registry 查詢）

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Node.js | **24.15.0** (LTS, Krypton) | Angular 21 要求 `^20.19.0 \|\| ^22.12.0 \|\| >=24.0.0` |
| 套件管理 | npm / pnpm 擇一 | 沿用專案習慣 |
| Angular CLI | **21.2.8** | |

---

## 核心框架

| 套件 | 版本 | 備註 |
|------|------|------|
| `@angular/core` | **21.2.10** | 17+ 才有 `@if` / `@for`；19+ 預設 zoneless |
| `@angular/common` | **21.2.10** | |
| `@angular/router` | **21.2.10** | |
| `@angular/forms` | **21.2.10** | |
| `@angular/compiler` | **21.2.10** | 與 core 同版 |
| `rxjs` | **7.8.2** | Angular 21 接受 `^6.5.3 \|\| ^7.4.0` |
| `typescript` | **6.0.3** | Angular 21 接受 `>=5.9 <6.1` |
| `zone.js` | **0.16.1** | Angular 21 接受 `~0.15.0 \|\| ~0.16.0`；若改 zoneless 可移除 |

---

## UI / 樣式

| 套件 | 版本 | 備註 |
|------|------|------|
| `primeng` | **21.1.6** | 對應 Angular 21；Aura 主題系統 |
| `primeicons` | **7.0.0** | |
| `@angular/cdk` | **21.2.8** | |
| `tailwindcss` | **4.2.4** | v4 改 CSS-first 設定，寫法見 `_shared/tailwind.md` |
| `@tailwindcss/postcss` | **4.2.4** | Angular CLI 走 PostCSS 路徑時用 |
| `@tailwindcss/vite` | **4.2.4** | Vite 專案用（擇一） |
| `@tailwindcss/forms` | **0.5.11** | 已支援 v4 |
| `@primeng/themes` | **21.0.4** | PrimeNG Aura/Material/Lara/Nora 主題包 |

---

## 工具鏈

| 套件 | 版本 | 用途 |
|------|------|------|
| `eslint` | **10.2.1** | flat config 強制（v9 起） |
| `angular-eslint` | **21.3.1** | 提供 flat config helpers |
| `typescript-eslint` | **8.59.0** | 提供 flat config helpers |
| `prettier` | **3.8.3** | |
| `prettier-plugin-tailwindcss` | **0.7.3** | 需 `prettier ^3.0` ✓ |
| `jest` | **30.3.0** | 單元測試（推薦） |
| `karma` | **6.4.4** | 沿用 Angular 預設測試棧時 |
| `jasmine` | **6.2.0** | Karma 搭配 |

---

## 已驗證的相容性

寫入此檔時依 [_global/rules.md 第 7 條](../_global/rules.md) 完成驗證：

- Node 24 LTS ✓ Angular 21 engine
- TypeScript 6.0.3 ✓ `@angular/compiler-cli` peer (`>=5.9 <6.1`)
- rxjs 7.8.2 ✓ `@angular/core` peer
- zone.js 0.16.1 ✓ `@angular/core` peer
- PrimeNG 21.1.6 ✓ `@angular/core ^21.0.7`、`@angular/cdk ^21.0.0`
- prettier-plugin-tailwindcss 0.7.3 ✓ `prettier ^3.0`

---

## 已驗證的相容性（補充）

- `@tailwindcss/forms@0.5.11` ✓ tailwind v4
- `angular-eslint@21.3.1` ✓ eslint 10、`@angular/cli ^21`、`typescript-eslint ^8`
- `typescript-eslint@8.59.0` ✓ eslint 10、`typescript >=4.8.4 <6.1`

---

## 變更歷史

- **2026-04-27** 建立檔案並寫入當前最新穩定版（Angular 21.2.10、Node 24.15.0 LTS、TS 6.0.3、PrimeNG 21.1.6、Tailwind 4.2.4、ESLint 10.2.1 flat config 等）；採 Tailwind v4、PrimeNG Aura、ESLint flat config

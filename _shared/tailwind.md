# Tailwind CSS 慣例（v4）

> Tailwind CSS v4 使用決議。v4 改採 **CSS-first config**，與 v3 寫法不同。
> 最後更新：2026-04-27（基準 `tailwindcss@4.2.4`）

---

## 與 v3 的關鍵差異（先記住）

| 項目 | v3 | v4 |
|------|----|----|
| 設定檔 | `tailwind.config.js` | `@theme` 直接寫在 CSS（JS config 改 opt-in） |
| 引入指令 | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| Plugin | `plugins: [...]` in JS config | `@plugin "name";` 在 CSS |
| 自訂 token | `theme.extend.colors` | `@theme { --color-primary: ... }` |
| PostCSS 引入 | `tailwindcss` 直接當 PostCSS plugin | 改用 `@tailwindcss/postcss`（獨立套件） |
| Vite 引入 | 透過 PostCSS | 改用 `@tailwindcss/vite` 官方插件 |

---

## 基本原則

- **優先使用 utility classes**，不為了「乾淨」就抽 component class
- 重複 3 次以上的組合才考慮抽，且**抽到框架層元件**（Angular component / React 元件），不是 CSS class
- 真要寫 class，用 `@apply` 寫在元件 scoped style，不放全域

---

## 引入方式

### Angular CLI（PostCSS 路徑）

`postcss.config.js`：
```js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

`src/styles.css`（或 `styles.scss`）：
```css
@import "tailwindcss";
```

### Vite 專案

`vite.config.ts`：
```ts
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

CSS 進入點：
```css
@import "tailwindcss";
```

---

## 設定主題（CSS-first）

直接寫在 CSS：

```css
@import "tailwindcss";

@theme {
  --color-primary-500: oklch(0.65 0.2 264);
  --color-bg: oklch(1 0 0);

  --font-sans: 'Inter', sans-serif;

  --spacing: 0.25rem;        /* 基礎間距單位 */

  --breakpoint-3xl: 120rem;  /* 自訂 breakpoint */

  --animate-slide-up: slideUp 0.3s ease-out;

  @keyframes slideUp {
    0%   { transform: translateY(20px); opacity: 0; }
    100% { transform: translateY(0);    opacity: 1; }
  }
}
```

- 命名規則：`--<namespace>-<name>`，namespace 對應 Tailwind 類別前綴（`color`、`font`、`spacing`、`breakpoint`、`animate`、`shadow`、`radius`...）
- 寫進 `@theme` 的 token 會自動產生對應 utility（例：`bg-primary-500`、`animate-slide-up`）

---

## class 排序

- 用 `prettier-plugin-tailwindcss`（已驗證 v0.7.3 相容 prettier 3.x、tailwind v4）
- 排序原則：layout → spacing → sizing → typography → color → effects → state variants

---

## 自訂顏色 / 設計 token

- 顏色寫在 `@theme` 內，**不在 component 用 magic number**（例如 `mt-[13px]`）
- 例外：一次性 layout 微調可用 arbitrary value（`mt-[3px]`）
- 顏色一律用 `oklch()`（v4 預設色彩空間，比 hex/rgb 更線性、好調漸層）

---

## 與第三方元件庫整合

- PrimeNG / 等元件庫：用 Tailwind 寫**外層佈局**，元件本身樣式走元件庫主題
- 需要覆寫第三方樣式時：
  - Angular：`::ng-deep` + 限定 selector
  - 其餘：在元件 scoped style 用更具體的 selector，**不開全域**

---

## RWD

- mobile-first：base style 不加前綴，再用 `sm:` / `md:` / `lg:` / `xl:` / `2xl:` 加大
- breakpoint 沿用 Tailwind 預設（`sm 640 / md 768 / lg 1024 / xl 1280 / 2xl 1536`），需要更大的在 `@theme` 加 `--breakpoint-3xl`

---

## Dark Mode

v4 預設用 `prefers-color-scheme`。要做手動切換用 variant：

```css
@import "tailwindcss";

@variant dark (&:where(.dark, .dark *));
```

之後 `<html class="dark">` 切換生效，`dark:bg-gray-900` 照常用。

---

## 動畫

- 簡單 transition → utility（`transition`、`duration-*`、`ease-*`）
- 客製 keyframe → 寫進 `@theme` 的 `@keyframes` + `--animate-*`，不在元件 SCSS 硬寫
- 動畫長度避免 >500ms

---

## 表單樣式（`@tailwindcss/forms`）

`@tailwindcss/forms@0.5.11` 已支援 v4。引入方式：

```css
@import "tailwindcss";
@plugin "@tailwindcss/forms";
```

策略沿用 `'class'`（手動加 `form-input` 才套用）。如需傳 options：

```css
@plugin "@tailwindcss/forms" {
  strategy: 'class';
}
```

第三方元件庫的 input 不套 forms plugin，避免衝突。

---

## CSS Variables Theming（多主題）

v4 已是 CSS-first，`@theme` 本身就是 CSS variables。多主題只要切換 class scope：

```css
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.6 0.2 264);
  --color-bg: oklch(1 0 0);
}

.dark {
  --color-primary: oklch(0.7 0.2 264);
  --color-bg: oklch(0.2 0 0);
}

.theme-emerald {
  --color-primary: oklch(0.7 0.2 152);
}
```

切主題不用 rebuild，可動態。簡單兩主題仍建議用 `dark:` 前綴，**不要過度工程**。

# Tailwind CSS 慣例

> Tailwind 使用決議，跨框架共用。
> 最後更新：2026-04-27

---

## 基本原則

- **優先使用 utility classes**，不為了「乾淨」就抽 component class
- 重複 3 次以上的組合才考慮抽：
  - 框架元件層（Angular component / React 元件）抽掉，而不是寫 CSS class
  - 真的需要 CSS class → 用 `@apply` 寫在元件 scoped style，不放全域

---

## class 排序

- 用官方 `prettier-plugin-tailwindcss` 自動排序
- 手寫順序大致：layout → spacing → sizing → typography → color → effects → state variants

---

## 自訂主題

- 顏色、字級、間距等設計 token 統一寫在 `tailwind.config.{js,ts}`
- 不在 component 用 magic number（例如 `mt-[13px]`），需要的數值補進 spacing token
- 例外：一次性的 layout 微調可以接受 arbitrary value

---

## 與第三方元件庫整合

- PrimeNG / Ant Design 等元件 → 用 Tailwind 寫外層佈局，元件本身樣式走元件庫主題
- 需要覆寫第三方樣式時：
  - Angular：`::ng-deep` + 限定 selector
  - 其餘：在元件 scoped style 用更具體的 selector，不開全域

---

## RWD

- mobile-first：先寫無前綴的 base style，再用 `sm:` / `md:` / `lg:` 加大
- breakpoint 不亂改，沿用 Tailwind 預設（`sm 640 / md 768 / lg 1024 / xl 1280 / 2xl 1536`）

---

## Dark Mode

- 設定 `darkMode: 'class'`，由根元素 `<html class="dark">` 切換
- 避免 `media` 模式（無法手動切）

---

## 動畫

- 簡單 transition（hover、focus、enter/leave）→ Tailwind 內建 utility（`transition`、`duration-*`、`ease-*`）
- 複雜動畫（彈跳、旋轉、複合 keyframe）→ 引入 `tailwindcss-animate` 套件
- 真的需要客製 keyframe → 寫在 `tailwind.config` 的 `theme.extend.keyframes` + `animation`，不在元件 SCSS 裡硬寫
  ```js
  // tailwind.config.js
  theme: {
    extend: {
      keyframes: {
        slideUp: {
          '0%': { transform: 'translateY(20px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      },
      animation: {
        'slide-up': 'slideUp 0.3s ease-out',
      },
    },
  }
  ```
- 避免動畫過長（>500ms），會影響感知速度

---

## 表單樣式

- 預設引入 `@tailwindcss/forms`，先取得乾淨的 baseline
- strategy 用 `'class'`（手動加 `form-input` 才套用），不要 `'base'` 全套，避免影響第三方元件庫的 input
  ```js
  // tailwind.config.js
  plugins: [require('@tailwindcss/forms')({ strategy: 'class' })]
  ```
- 第三方元件（PrimeNG、Ant Design）的表單元件不套用 forms 插件，走元件庫主題

---

## CSS Variables Theming

- 多主題 / Dark Mode 進階情境：用 CSS variables 當「中介層」，Tailwind 顏色定義成 `rgb(var(--color-primary) / <alpha-value>)` 形式
  ```css
  /* styles.css */
  :root {
    --color-primary: 59 130 246;   /* RGB 數字，逗號用空白 */
    --color-bg: 255 255 255;
  }
  .dark {
    --color-primary: 96 165 250;
    --color-bg: 17 24 39;
  }
  ```
  ```js
  // tailwind.config.js
  theme: {
    extend: {
      colors: {
        primary: 'rgb(var(--color-primary) / <alpha-value>)',
        bg: 'rgb(var(--color-bg) / <alpha-value>)',
      },
    },
  }
  ```
- 好處：切主題不用 rebuild、可支援使用者自訂主題色
- 簡單兩主題情境就用 `dark:` 前綴即可，**不要過度工程**

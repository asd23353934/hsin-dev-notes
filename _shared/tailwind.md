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

## 待補

- [ ] 動畫（`@tailwindcss/animate` vs 原生 keyframes）
- [ ] 表單樣式（`@tailwindcss/forms` 整合方式）
- [ ] 與 CSS variables 配合的 theming 模式

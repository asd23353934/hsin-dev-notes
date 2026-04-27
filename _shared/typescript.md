# TypeScript 慣例

> 跨框架共用的 TS 寫法決議。Angular / React / Vue 專案皆適用。
> 最後更新：2026-04-27

---

## 編譯設定

- `strict: true` 為基本要求
- 額外開啟：
  - `noImplicitAny`
  - `strictNullChecks`
  - `noUncheckedIndexedAccess`
  - `noImplicitOverride`
- 不關閉 strict 換取進度，遇到型別問題正面解決

---

## 型別宣告

### `interface` vs `type`
- **物件結構** → 用 `interface`（可被 extend / merge）
- **聯合、交集、tuple、map type** → 用 `type`
- 不混用，看用途決定

### 不要用 `any`
- 確實不知道型別 → 用 `unknown`，再用 type guard 收斂
- 第三方無型別 → 自己補 `.d.ts`，不要 `as any` 矇混

### Enum
- 一般情況改用 `as const` 物件 + 衍生 union type，不用 `enum`
  ```ts
  export const UserRole = {
    Admin: 'admin',
    User: 'user',
  } as const;
  export type UserRole = typeof UserRole[keyof typeof UserRole];
  ```
- 必要時才用 `const enum`，避免一般 `enum` 產生 runtime code

---

## 命名

| 對象 | 規則 | 範例 |
|------|------|------|
| 介面 | PascalCase，**不加 `I` 前綴** | `User`、`HttpResponse` |
| 型別別名 | PascalCase | `UserId`、`ApiResult<T>` |
| 泛型 | 單字母或 PascalCase 描述名 | `T`、`TUser`、`TResponse` |
| 列舉成員 | PascalCase | `UserRole.Admin` |

---

## Import / Export

- 預設 named export，避免 default export（refactor 改名較安全）
- import 路徑用 path alias（例如 `@core/`、`@shared/`），不用長串 `../../../`
- type-only import 明示：
  ```ts
  import type { User } from './user.model';
  ```

---

## 待補

- [ ] async / await 錯誤處理慣例
- [ ] utility type 常用組合
- [ ] 與 Zod / Valibot 等 runtime 驗證庫的整合

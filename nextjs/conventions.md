# Next.js Conventions

> Next.js 專案的寫法慣例（App Router 為主）。Claude Code 在產出 Next.js 程式碼時必須遵守。
> 最後更新：2026-04-27

---

## 路由與檔案

- 一律使用 **App Router**（`app/` 資料夾），不用 Pages Router
- API endpoint 用 Route Handlers（`app/api/<path>/route.ts`），檔名固定 `route.ts`
- 動態路由用 `[id]` 資料夾，多層動態用 `[[...slug]]`
- Layout、Loading、Error 各自獨立檔（`layout.tsx` / `loading.tsx` / `error.tsx`）

---

## API Route 規範

### 認證模式

**User API（需登入）**：

```ts
import { auth } from '@/auth';
import { NextResponse } from 'next/server';

export async function GET() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: '未授權' }, { status: 401 });
  }
  const userId = session.user.id;
  // ...
}
```

**Worker / 後端 API（Bearer token）**：

```ts
const authError = verifyWorkerToken(request);
if (authError) return authError;
```

把驗證邏輯抽到 `lib/worker-auth.ts`，不在每個 route 裡重複寫。

**資源 ownership 驗證**（修改/刪除前必做）：

```ts
const existing = await prisma.keyword.findUnique({ where: { id } });
if (!existing) {
  return NextResponse.json({ error: '找不到' }, { status: 404 });
}
if (existing.userId !== userId) {
  return NextResponse.json({ error: '禁止存取' }, { status: 403 });
}
```

### 錯誤回應格式

- 錯誤一律用 `{ error: string }`，**不用** `{ message: ... }` 或其他 key
- Prisma unique constraint 違反（P2002）→ 回 **409**
- 找不到資源 → 回 **404**，不回 200 + 空陣列

```ts
// 正確
return NextResponse.json({ error: '關鍵字已存在' }, { status: 409 });

// 錯誤
return NextResponse.json({ message: '...' }, { status: 409 });
```

### 錯誤處理樣板

```ts
import { Prisma } from '@prisma/client';

try {
  // ...
} catch (err: unknown) {
  if (err instanceof Prisma.PrismaClientKnownRequestError && err.code === 'P2002') {
    return NextResponse.json({ error: '已存在' }, { status: 409 });
  }
  console.error(err);
  return NextResponse.json({ error: '伺服器錯誤' }, { status: 500 });
}
```

### Payload 型別

Route 內收的 payload interface **定義在檔案頂部**，不散落在 handler 中段：

```ts
interface CreateKeywordPayload {
  keyword: string;
  platforms: string[];
  blocklist?: string[];
}

export async function POST(request: Request) {
  const body = (await request.json()) as CreateKeywordPayload;
  // ...
}
```

---

## NextAuth v5

- 一律用 `auth()` 取代 v4 的 `getServerSession()`
- 設定集中在 `auth.ts`（專案根目錄或 `lib/auth.ts`），匯出 `{ auth, handlers, signIn, signOut }`
- API route：

  ```ts
  // app/api/auth/[...nextauth]/route.ts
  export { handlers as GET, handlers as POST } from '@/auth';
  ```

- middleware 內：`export { auth as middleware } from '@/auth';`
- session 型別擴充寫在 `types/next-auth.d.ts`，不在 component 內 cast

---

## Prisma

- 共用 client 走 singleton：`lib/prisma.ts`

  ```ts
  import { PrismaClient } from '@prisma/client';

  const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };
  export const prisma = globalForPrisma.prisma ?? new PrismaClient();
  if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
  ```

- Migration：`prisma migrate dev`（local）/ `prisma migrate deploy`（CI/CD）
- `DATABASE_URL`（pooled，runtime）與 `DIRECT_URL`（無 pooler，migration 用）分開
- Schema 改動後 commit `prisma/migrations/`，不只 commit `schema.prisma`

---

## TypeScript（補充 `_shared/typescript.md`）

- API Route 的 payload interface 放檔案頂部
- 共用 utility 集中在 `lib/utils.ts`，不在 route / component 內各自定義
- `cn()` helper 是標配：

  ```ts
  // lib/utils.ts
  import { clsx, type ClassValue } from 'clsx';
  import { twMerge } from 'tailwind-merge';

  export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs));
  }
  ```

---

## Server Components vs Client Components

- **預設 Server Component**，需要互動才加 `'use client'`
- 把 `'use client'` 推到葉子元件，不要整頁 client
- Server Component 內可直接 `await prisma.xxx()`，不需要拉 API
- 用了 `'use client'` 的檔案不能 import server-only 模組（會編譯錯）

---

## Server Actions

- 用 `'use server'` 標記檔案或函式
- 表單 mutation 優先用 Server Action，少寫 fetch + API route 來回
- Action 內錯誤處理同 API Route：包 try/catch、回 `{ error }` 物件
- 改動資料後呼叫 `revalidatePath()` / `revalidateTag()` 重整 cache

---

## 環境變數

- 客戶端可用的變數一律加 `NEXT_PUBLIC_` 前綴
- secret（DB password、API key、Webhook secret）**絕對不能** `NEXT_PUBLIC_`
- 提供 `.env.example`，列出所有需要的變數但不放實值
- `process.env.XXX` 在程式碼中讀；不再多包一層 config 物件（Next.js 已 inline 處理）

---

## 命名

| 對象 | 規則 | 範例 |
|------|------|------|
| 元件檔 | PascalCase | `UserMenu.tsx` |
| Route handler | 固定 | `route.ts` |
| Route segment | kebab-case 資料夾 | `app/user-settings/page.tsx` |
| 型別 / interface | PascalCase | `Keyword`、`CreateKeywordPayload` |
| 函式 / 變數 | camelCase | `verifyWorkerToken` |
| 常數 | UPPER_SNAKE | `MAX_NOTIFY_PER_BATCH` |

---

## 樣式（Tailwind v4 + shadcn）

- 元件樣式優先 Tailwind utility（見 `_shared/tailwind.md`）
- shadcn 元件 copy 進 `components/ui/`，**不視為依賴**，可隨需求改
- variant 用 `class-variance-authority`，不自己手寫一堆 className 判斷
- icon 用 `lucide-react`，size 統一（建議 `h-4 w-4` for inline、`h-5 w-5` for button）

---

## 待補項目

- [ ] middleware 結構與順序
- [ ] cache / revalidate 策略
- [ ] Image 元件用法（`next/image`）
- [ ] 全域錯誤處理（`error.tsx` / `global-error.tsx`）
- [ ] Edge runtime vs Node runtime 選擇準則

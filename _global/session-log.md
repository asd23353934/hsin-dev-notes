# Session Log

> 對話日誌（時間軸，**最新在最上面**）。
> 每次對話結束時由 Claude Code 主動補上，作為日後回查的索引。

---

## 模板

```markdown
### YYYY-MM-DD｜本次主題
- **專案**：
- **重點**：做了什麼、解了什麼、學到什麼
- **產出**：改了哪些檔案
- **後續**：待辦事項
```

---

### 2026-04-28｜全 repo 審查 + stack.md 政策統一 + Vitest 收斂
- **專案**：dev-notes 本身
- **重點**：
  - 全 repo 審查找出三類問題：
    1. **stack.md 政策矛盾**（angular=npm latest / nextjs+python=shop-watcher pinned）
    2. **README/CLAUDE.md 結構圖落後**（沒列 nextjs/、python/、_shared/spectra.md、_global/skill.md）
    3. **angular/conventions.md 測試框架待定**但 stack.md 已三套都列
  - 採方案 A：**所有 stack.md 統一為 latest stable**（依 `npm view` / `pip index versions` 驗證 + peer 相容性檢查）
    - nextjs：Next 15→16、Prisma 5→7、recharts 2→3、@base-ui/react 1.3→1.4、shadcn 4.1→4.5、lucide 1.7→1.11、next-auth beta.30→31、resend 6.10→6.12；Node 基準改 24 LTS
    - python：playwright 1.51→1.58、bs4 4.12→4.14、python-dotenv 1.0→1.2
    - angular：補 typescript-eslint 8.59.0→8.59.1、prettier-plugin-tailwindcss 0.7.3→0.8.0
  - **Angular 21+ 測試框架收斂為 Vitest**（4.1.5 + @analogjs/vitest-angular 2.4.10 + jsdom），移除 jest/karma/jasmine
  - **Prisma 5→7 連動 nextjs/conventions.md**：改用新 `prisma-client` generator + 自訂 output 路徑（import 不再從 `@prisma/client`）
  - README.md 與 CLAUDE.md 結構圖補 nextjs/、python/、_shared/spectra.md；CLAUDE.md「對話開始時」補對應觸發條件、commit scope 加 nextjs/python
  - _shared/spectra.md：第 65 行 `/spectra:audit` 改為 `/spectra-audit`，統一文件內 dash 命名（保守選擇，待確認 plugin 實際命名）
- **產出**：CLAUDE.md、README.md、angular/stack.md、angular/conventions.md、nextjs/stack.md、nextjs/conventions.md、python/stack.md、_shared/spectra.md、本 session-log
- **後續**：
  - 確認 spectra plugin 實際命名（colon vs dash），若實際用 colon 則回頭翻轉全文件
  - errors.md 三份仍空，等實際踩坑累積
  - typescript-eslint / prettier-plugin-tailwindcss 之後若再 lag 可順手補

---

### 2026-04-27｜擴充 nextjs / python 資料夾（從 shop-watcher 萃取）
- **專案**：shop-watcher（Next.js 15 + Python 3.12 worker）→ 反向萃取共用慣例回 dev-notes
- **重點**：
  - 新建 `nextjs/`：依 shop-watcher 實際版本寫入 stack.md（Next.js 15.5.14 / React 19.1 / Prisma 5.22 / next-auth v5 beta / Tailwind v4 / shadcn 4.x with `@base-ui/react`）；conventions 涵蓋 App Router、Route Handler 認證模式、`{ error: string }` 統一錯誤格式、Prisma P2002→409、NextAuth v5 `auth()`、Server Components 預設、Server Actions、命名規範
  - 新建 `python/`：stack（Playwright 1.51 / httpx 0.28 / bs4 4.12）；conventions 涵蓋 scraper 模組規範（module-level async、絕不 raise、共用 `_xxx.py` helper）、15s 統一逾時、`logger.exception`、async/await 慣例
  - 新建 `_shared/spectra.md`：跨技術棧的 SDD 工具，包含 skill 對照表、工作流、何時跳過、parked changes、pre-commit 三步驟（`/simplify` → `/spectra:audit` → 同步文件）
- **產出**：`nextjs/stack.md`、`nextjs/conventions.md`、`nextjs/errors.md`、`python/stack.md`、`python/conventions.md`、`python/errors.md`、`_shared/spectra.md`、本 session-log
- **後續**：
  - 待提議：更新 `CLAUDE.md`「對話開始時的標準動作」加入 Next.js / Python / Spectra 的觸發條件，更新 `README.md` 結構圖
  - 累積各 errors.md 紀錄
  - shop-watcher 端若有新踩坑（Prisma migration、NextAuth v5 beta 邊角案例、Playwright headless on GitHub Actions）回頭補進對應 errors.md

---

### 2026-04-27｜建立 dev-notes repo 與 md 架構
- **專案**：dev-notes 本身
- **重點**：初始化 repo，建立 CLAUDE.md / README.md / 全域規則 / Angular 慣例；設定本地 git user 為 asd23353934；補齊 `_global` / `_shared` / `angular` 資料夾結構與骨架檔
- **產出**：`_global/rules.md`、`_global/session-log.md`、`_shared/typescript.md`、`_shared/tailwind.md`、`angular/stack.md`、`angular/conventions.md`、`angular/errors.md`、`.gitignore`
- **後續**：
  - 補 `angular/stack.md` 實際版本號
  - 累積各框架的 `errors.md` 紀錄

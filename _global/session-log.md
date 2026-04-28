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

### 2026-04-28｜key-trace 番茄鐘落地：Spectra apply + 第一個 archive
- **專案**：key-trace
- **重點**：
  - **Spectra apply 完整流程跑通**：`unpark add-pomodoro-timer` → `in-progress add` → 讀 design.md / specs / tasks.md → 18 個 task 逐個實作（用 batch 寫法把同檔多個 task 一次處理）→ `spectra task done <id>` 個別標記 → `validate` ✓ → 三輪 review（simplify / security / peer）→ commit → `spectra archive` 落 base spec → commit
  - **實作偏離 spec 處理**：原本 spec 寫「設定走 electron-store」，實作時發現 electron-store v9+ 全是 ESM-only，與 main 進程 CommonJS + externalizeDepsPlugin 衝突（require ESM 套件 throw `ERR_REQUIRE_ESM`）。決定自寫 `src/main/settings-store.ts`（fs+JSON + 型別驗證 + memory cache，~40 行），把 spec / design / proposal / tasks 四份 artifact 都更新成「persistent settings store」一般描述 + 在 design.md 補一段「為何不用 electron-store」WHY 紀錄
  - **三輪 review 抓到的實質改善**（都修了）：
    - **simplify**：`pausedAccumulatedMs` + `elapsedActiveMs` 在 pomodoro state 裡是冗餘（`endTsMs += pausedDuration` 已涵蓋暫停時長）→ 簡化成 `actualSec = plannedSec - remainingSec()` 統一公式覆蓋運行 / 暫停 / 自然結束三情境；settings-store 每次 fs.readFileSync → 改記憶體 cache；PomodoroCard 五個 `?: null` ternary → `&&`，刪 dead code
    - **security**：clean，無 confidence ≥ 8 漏洞（綁定參數、無路徑穿越、validate function 雖比 zod 鬆但設定來源在 userData 同信任邊界）。順手把 validate 函式對齊 zod 加 `Number.isInteger` + 上限
    - **peer**：getStats 5s 全狀態輪詢浪費 → 改成 30s 加 user 觸發 mutation invalidate；PomodoroCard buttons 加 `disabled={mutation.isLoading}` 防雙擊靜默吞錯；spec 對齊驗證（streak 算法 / pause-resume 數學 / stop completed=0 等）全 ✓
  - **`spectra archive` 行為觀察**：把 change 的 spec 落地進 `openspec/specs/<capability>/spec.md`（成為 project 永久 base spec），原本 `openspec/changes/add-pomodoro-timer/` 整個搬到 `openspec/changes/archive/2026-04-28-add-pomodoro-timer/`，附 snapshot 給 unarchive 用。17/18 task done 也能 archive（task 8.4 實機驗留給 Hsin 跑 dev，不卡 archive）
  - **學到**：
    - Spectra propose-apply-archive 拆三段恰當：propose 把架構決議鎖死、apply 不偏離、archive 把成果寫進長期 spec。中間實作若偏離（如 electron-store ESM 卡住），**先更新 artifact 再實作**，spec 一直是 source of truth
    - 自寫 ~40 行 fs+JSON 工具有時比引第三方套件穩（避開 ESM/CJS / lockstep / breaking change 風險），CLAUDE.md「不過度抽象、不為假想需求設計」對齊
    - tRPC v10 + zod v3 的 `.input(z.object(...).strict())` 寫法簡潔，搭 `wrapPomodoroAction` 把 throw 統一轉 TRPCError 是好 pattern，未來其他 namespace 可複用
    - Electron Notification API 一行 `new Notification({ title, body }).show()` 就好，click 事件用 `notification.on('click', ...)`，整合 main 的 focusMainWindow 一氣呵成
- **產出**：
  - key-trace：`0f5d554`（feat: 番茄鐘實作 16 檔 +1179）+ `f48d9f2`（chore: archive，6 檔 +361），含新檔 `src/main/{pomodoro,settings-store}.ts` / `src/renderer/src/components/PomodoroCard.tsx` + base spec `openspec/specs/pomodoro/spec.md`
  - dev-notes：本 session-log
- **後續**：
  - **Hsin 跑實機驗（task 8.4）**：`npm run dev` → 縮短 work / break 時長到 6s/3s（updateSettings mutation 或先手改 userData/pomodoro-settings.json）→ 按開始 → 看通知觸發 → click 通知聚焦 → DB 落盤確認 → getStats 顯示 todayCompleted=1
  - 下一個 v1 change：heatmap + 一日週期分析（建議走 `propose` → `apply` → `archive` 同樣流程）
  - electron-store 偏離經驗已寫進 design.md，建議反向萃取進 dev-notes/electron/conventions.md 的「Settings 持久化選項」一節（下次更新時順手）

---

### 2026-04-28｜key-trace 進入功能堆疊：Spectra init + 第一個 change 提案（番茄鐘）
- **專案**：key-trace
- **重點**：
  - **Spectra（spec-driven）正式 init**：A2 完成、walking skeleton 穩定、進入「功能堆疊」階段，按 CLAUDE.md 第 144 行約定 init `openspec/`。CLI `spectra 2.2.5` 已裝（Windows）；Hsin 透過該 CLI 在 key-trace 跑了 init，產出 `openspec/{specs,changes}/`、`openspec/config.yaml`、`.spectra.yaml`、`.claude/skills/spectra-*`、`.claude/settings.json`，並在 `.gitignore` 加 `.spectra/` + `openspec/.vector-search.db*`。CLAUDE.md 頂部被 Spectra 自動注入 `SPECTRA:START` block（指令給 Claude）
  - **重要習慣**：Spectra skills 是這次 session 啟動後才裝的，當前 conversation 無法直接 `Skill('spectra-propose')` 呼叫（不在啟動時的可用 skill 清單）。**手動讀 SKILL.md 跑 CLI 指令完成同樣流程**（`spectra new change` → `spectra instructions <artifact>` → 寫 artifact → `spectra new artifact` → `spectra analyze` → `spectra validate` → `spectra park`）
  - **第一個 change `add-pomodoro-timer`** 完整跑完 propose 流程：
    - **proposal.md**：feature type、新 capability `pomodoro`、影響檔案清單（new / modified）、schema 變動（pomodoro_sessions 七欄）、明標「不影響隱私底線」與「三進程邊界」
    - **design.md**：8 個關鍵決議（main 持有狀態而非 utility / renderer、三 phase + paused flag、`setTimeout` + 結束絕對時間戳避免時鐘漂移、completed=0 vs 1 落盤策略、streak 用 SQLite localtime + JS 迴圈算、Electron Notification 整合、`electron-store` 設定持久化、schema CREATE IF NOT EXISTS）；Risks/Trade-offs 寫進「app 中斷 session 不恢復」「跨時區搬遷」「沒有 schema migration」等接受項
    - **specs/pomodoro/spec.md**：9 個 requirement（Timer State Machine / Pause and Resume / Background Continuity / Session Persistence / Streak Calculation / tRPC API Surface / Settings / Notification Behavior / Privacy Invariant），每個含多個 WHEN/THEN scenario + streak 表格 example。**spec 一律英文**（Spectra 規定，因 SHALL/MUST normative wording）
    - **tasks.md**：8 個 group / 17 個 task，按 dependency 排序：utility schema → 共用 protocol → main storage 橋 → main pomodoro 模組 → router → main 啟動接線 → renderer UI → 驗證（typecheck / build / e2e）
    - 第一輪 `spectra analyze` 抓到 3 個 Coverage/Consistency Warning：tasks 沒涵蓋 spec 的 `tRPC API Surface` requirement 名 + design 的兩個中文 heading。修法：**精確字面**塞進 task 描述（不靠近似），第二輪全 Clean
    - `spectra validate` ✓，`spectra park` ✓
  - **學到**：
    - Spectra 的 propose workflow 規定 spec 必為英文、其他 artifact 跟 locale（這裡 tw）。spec 用 SHALL/MUST + WHEN/THEN，禁 should/may/might/TBD/TODO
    - Coverage analyzer 用「字串 substring + case-insensitive」匹配，但**對 Chinese 中標點符號似乎敏感**（`狀態機：idle / work / break + paused 標記` 必須完整出現包括 `：`與 `/`）
    - `spectra park` 把 change artifacts 移到 `.git/spectra-app/changes/<name>/`，**不入 git 版控**；apply 時自動 unpark 回 `openspec/changes/`。多人協作要分享 parked change 需另想辦法（目前 .spectra.yaml 的 `worktree: true` 是另一條路但沒啟用）
    - openspec/config.yaml 的 `context:` 一定要填，artifact 生成時 AI 才能對齊既有架構決議；rules: 可分 proposal / tasks / spec 細分規則
- **產出**：
  - key-trace：commit `dd40f1d`（chore: 導入 Spectra SDD），檔案有 `openspec/{config.yaml,specs,changes}`、`.spectra.yaml`、`.claude/{settings.json,skills/spectra-*}`、`.gitignore` 與 `CLAUDE.md` 改動。**第一個 change 的 4 個 artifact 都在 `.git/spectra-app/changes/add-pomodoro-timer/`**（parked）
  - dev-notes：本 session-log
- **後續**：
  - **下一步：實作 add-pomodoro-timer**。當 Spectra skills 在新 session 載入完整可用時走 `/spectra-apply add-pomodoro-timer`（會自動 unpark）；不然讀 `.git/spectra-app/changes/add-pomodoro-timer/tasks.md` 手動逐項做、`spectra apply tick <task-num>` 標記完成
  - 實作完跑既有三輪 review（`/simplify` + 安全 + peer），通過後 `spectra archive add-pomodoro-timer` 把 spec 合進 `openspec/specs/pomodoro/`
  - 後面 v1 三項（heatmap / markdown 報告 / Claude 寫總結）走同樣 propose → apply → archive 流程

---

### 2026-04-28｜key-trace A2：utility process + better-sqlite3 storage pipeline
- **專案**：key-trace
- **重點**：
  - **A2 落地**：main 透過 `utilityProcess.fork()` 起 utility 進程，內跑 better-sqlite3。schema 為 `events(minute_ts, app_name)` 主鍵 + `idx_events_minute` 索引；upsert 走 `INSERT ... ON CONFLICT DO UPDATE SET keydown = keydown + excluded.keydown ...` 累加模式 + transaction
  - **tracker 重構**：原本 in-memory 累計器改成 per-app delta `Map<appName, AppDelta>`，每分鐘 flush 到 utility；事件被 attribute 到當下 `activeApp`，null（黑名單或未 poll）整顆丟棄不入 DB（隱私保險 + 一致性）
  - **main↔utility 自寫 RPC**：`requestId` (`crypto.randomUUID()`) + `Map<id, resolver>` + 5 秒 timeout；抽 `src/shared/storage-protocol.ts` 共用 message types
  - **tRPC namespace 重整**：`tracker.stats` (live in-memory) + `historical.totalToday` (DB SUM)；renderer 加「今日總計（DB）」section 4 卡 5 秒 refetch 證實 end-to-end
  - **意外順利**：better-sqlite3 12.9 的 prebuilt 在 Electron 41 (Node 22 ABI) **直接載入成功**，不必跑 electron-rebuild、不必裝 VS Build Tools。已寫進 `electron/stack.md`
  - **三輪 review** 走完整 pre-commit 流程：
    - `/simplify`：抓到 stats 雙寫漂移風險（pendingDeltas + 直接 stats++ 兩條獨立累計線，漏寫一邊就靜默不一致）→ 改成單一來源（live = `sessionTotals` + `pendingDeltas` 即時 sum）；mousemove 雙 Map.get/set → 抽 `getOrCreateDelta()`；utility AppDelta 與 shared 重複定義 → import；App.tsx 兩 grid section → `<Section items={...} />`
    - `/security-review`：clean，無 confidence ≥ 8 漏洞（綁定參數擋 SQL、V8 structured clone strip prototype、RPC ID 不可猜測、env path 受控）
    - `/review`：utility 進程崩潰後 main 永久半死路徑（pending RPC 等 5s timeout、`flushBucket` 對死 child throw、`stop().kill()` no-op）→ post-ready `child.on('exit')` 維護 alive flag、drain pending、API 進入 dead 後 fire-and-forget 靜默 / query 立即 reject；`for...in` over `Record` 違反 `noUncheckedIndexedAccess` → `Object.entries`；`as TotalsRow` 假型別無 runtime 檢查 → 抽 `readTodayTotals()` 顯式 `Number(row[k] ?? 0)` normalize；命名收斂 `TotalsRow → TodayTotals` / `flushedTotals → sessionTotals` / `writeBucket → applyBucket`
  - **學到**：
    - utility process 不是 main / preload / renderer 之外的「特殊」進程 — 就是個有 message channel 的子 process，better-sqlite3 在裡面跟在 Node CLI 跑沒兩樣
    - 自寫 RPC over `parentPort` 比想像的簡單（< 100 行），但 **crash 恢復路徑是必做不是 optional**，沒寫整個系統會在 utility 一掛掉就半死
    - tRPC 的 namespace router 在 `t.router({ a: t.router({...}), b: t.router({...}) })` 巢狀寫法簡單，renderer 自動就能 `trpc.a.x.useQuery()`
    - 「stats 漂移風險」是雙寫累計器的經典 bug，要靠程式結構消除（單一來源），review 才抓得到
- **產出**：
  - key-trace：`src/utility/index.ts`、`src/main/storage.ts`、`src/shared/storage-protocol.ts`（新）；`src/main/{index,router,tracker}.ts` + `src/renderer/src/App.tsx` + `electron.vite.config.ts` + `tsconfig.node.json` + `package.json`（改）；2 commits（`70eee93` A2 落地、`205ad6c` 三輪 review fixup）已 push origin
  - dev-notes：`electron/stack.md`（補 better-sqlite3 / @types / @electron/rebuild + Electron 載入備忘）、`electron/conventions.md`（新增「utility process IPC + RPC」「utility entry build 結構」「better-sqlite3 慣例」三節）、本 session-log
- **後續**：
  - **進入「功能堆疊」階段** → init `openspec/`、導入 Spectra（CLAUDE.md 第 99 行的時機到了）
  - 第一個 change 候選：`propose-storage-schema`（把 A2 已落地的 schema 寫成 base spec）或直接從 v1 功能（番茄鐘 / heatmap / markdown 報告）開第一個 change
  - utility entry 仍是純 side-effect script、無法 unit test → 第一次寫測試時拆 `createUtilityHandler({ db, port })` + thin bootstrap

---

### 2026-04-28｜key-trace 從零 scaffold 到 walking skeleton + 三輪 review + 新增 electron/ 框架資料夾
- **專案**：key-trace（新建：WhatPulse 替代品，Electron 桌面 app）+ dev-notes 本身
- **重點**：
  - 規格決議走 Q1~Q16 16 題決策樹，逐題確認：Win + macOS、本機 + 匯出、三軸全做、electron-vite 三進程拆分（main / utility / renderer）、uiohook-napi 隱私底線（只記每鍵 × 每 app 次數，無時序，視窗標題預設 OFF）、5s 視窗輪詢、60s idle、分鐘 bucket + 每日 rollup、electron-trpc + TanStack Query、shadcn + Tremor、六項系統整合全做、release pipeline 後做、Spectra 待 walking skeleton 完成才導入、不加密 SQLite、v1 = 番茄鐘 + heatmap + markdown 報告 + Claude 寫總結
  - **技術 spike** 驗 uiohook-napi（prebuilt 直跑，3+35 events 都抓得到）、get-windows 動態抓 active app — 全綠
  - **Walking skeleton** 落地：Electron 41 + electron-vite 5 + Vite 7.3.2（vite 8 卡 electron-vite peer）+ React 19 + TS 6.0.3 + Tailwind v4 + electron-trpc + TanStack Query + uiohook-napi + get-windows，用 Playwright `_electron.launch()` 自測截圖
  - **重大踩坑**：electron-trpc 0.7.1 / 1.0.0-alpha.0 內部仍用 tRPC v10 procedure 結構，跟 v11 不相容（renderer query 永遠 isLoading 但無 error）。降版 tRPC → 10.45.4 + TanStack Query → 4.44.0 解決。已寫 `electron/errors.md`
  - **Pre-commit 三輪 review**：
    - `/simplify`：拆 Card props（`unit` / `subtitle` 分開）、tracker 改 factory + `started` flag、stopTracker 去重、加單一實例鎖、router 改 `createRouter(tracker)`
    - `/security-review`：加 shell scheme allowlist、CSP 拆細（移 script unsafe-inline）、`web-contents-created` → `will-navigate` 全域阻擋、`sandbox: true`（搭配 preload bundle electron-trpc）、抽 `src/main/hooks.ts` 隱私 wrapper（鍵盤對外只給 keycode 不給 keychar；mousemove 對外只給 dx/dy）、`activeTitle` 預設 OFF（`captureTitle: false`）— **隱私底線改為程式結構強制**，不再只是 spec 上的口頭約束
    - `/review`（peer）：註解精簡、`startedAt` 改在 start() 設、滑鼠 dx/dy 4000px sanity bound、`uIOhook.start` try/catch（macOS 權限拒絕降級為「不追蹤」不打死 app）、`pollActiveWindow` 失敗一次性 console.warn、`will-quit` 雙保險、`engines.node >=22`、`e2e:smoke` script、e2e 改 `waitForFunction` 不寫死等待
  - **學到**：(1) Electron renderer 自測要走 Playwright `_electron`，不是純瀏覽器（純瀏覽器沒 preload 看不到實際行為）。(2) electron-trpc 與 tRPC v11 整套不相容是 trap，社群 alpha 版也沒升。(3) `sandbox: true` 在用 electron-trpc 時要把套件 bundle 進 preload（不是 externalize）才行。(4) 隱私敏感的 native module 一定要包 wrapper 層做結構強制，靠 PR review 注意是不夠的
- **產出**：
  - 新 repo `key-trace`：完整 scaffold（package.json / electron.vite.config.ts / tsconfig × 3 / src/main/{index,tracker,router,hooks}.ts / src/preload/index.ts / src/renderer/* / scripts/e2e-check.mjs / CLAUDE.md / .gitignore / .npmrc / .claude/launch.json）
  - dev-notes：`electron/stack.md`、`electron/conventions.md`、`electron/errors.md`、`README.md` + `CLAUDE.md` 結構圖補 electron/、commit scope 加 `electron`、本 session-log
- **後續**：
  - key-trace 進入 A2：utility process + better-sqlite3（schema 設計 / 每分鐘 bucket / 每日 rollup）
  - A2 收尾後 init `openspec/`，導入 Spectra
  - 追蹤 electron-trpc upstream 升 tRPC v11，屆時整套升回 tRPC 11 + TanStack Query 5
  - v1 release 前補 README、HTTP-layer CSP、native module supply chain overrides

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

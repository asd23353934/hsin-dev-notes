# CLAUDE.md

> 給 Claude Code 的指令。本 repo 是 Hsin 的個人開發筆記，跨專案共用，不是程式碼專案。
> **本檔只放「入口 + 結構 + 觸發路由 + 共用模板」;行為規則的單一來源是 `_global/rules.md`,不在此重複**(依該檔 §14「CLAUDE.md 與 rules.md 重疊 → 留一處」)。

---

## 基本資訊

- **使用者**：Hsin
- **回覆語言**：**繁體中文**（完整規則見 `_global/rules.md` §1）
- **主技術棧**：Angular + PrimeNG + Tailwind + TypeScript;也涉及 Next.js、Python、Electron、PostgreSQL、Godot、Unity(C#)、Vue/Nuxt、Go

---

## 檔案結構

```
dev-notes/
├── CLAUDE.md            ← 本檔（入口 / 結構 / 路由 / 模板）
├── README.md           ← 給人類看的使用說明
│
├── _global/            ← 全域規則（任何對話都適用）
│   ├── rules.md        ← 行為規則的單一來源（最高優先）
│   ├── skill.md        ← Claude Code 工具/skill/subagent 使用規範
│   └── session-log.md  ← 對話日誌
│
├── _shared/            ← 跨技術棧共用（typescript / tailwind / spectra）
│
├── angular/            ← Angular（stack / conventions / errors）
├── nextjs/             ← Next.js（App Router）
├── python/             ← Python（爬蟲 / Worker）
├── electron/           ← Electron 桌面應用
├── postgres/           ← PostgreSQL
├── godot/              ← Godot 4 / GDScript（含 plugins.md）
├── unity/              ← Unity / C#（遊戲）
├── vue/                ← Vue / Nuxt
└── go/                 ← Go
```

每個技術棧資料夾的慣例：`stack.md`（版本，事實型）、`conventions.md`（決議型）、`errors.md`（踩坑紀錄）。

---

## 對話開始時（觸發路由）

必讀 `_global/rules.md`（最高優先行為規則）。其餘**依當下主題按需讀取，不必每次全讀**：

- Angular → `angular/`;Next.js → `nextjs/`;Python → `python/`;Electron → `electron/`;PostgreSQL → `postgres/`;Godot → `godot/`;Unity/C# → `unity/`;Vue/Nuxt → `vue/`;Go → `go/`（各取 `conventions.md` + `stack.md`）
- TypeScript → `_shared/typescript.md`;Tailwind → `_shared/tailwind.md`;專案根有 `openspec/` → `_shared/spectra.md`
- 遇錯先查 `[framework]/errors.md`
- 涉及 Claude Code 工具 / skill / subagent / slash command 行為 → `_global/skill.md`

---

## 版本來源優先順序

實際專案內，**先讀該專案 `package.json`**，不可直接套 dev-notes 的 `stack.md`：

- 專案版本較舊 → 用該版本對應 API，**不主動建議升級**
- 專案版本較新 → 任務結束後可提議「同步回 stack.md？」
- PrimeNG / Tailwind 等大版本斷層 → **絕對不可**用新版 API 套舊版專案

`stack.md` 只是「新專案起手套餐」與「沒 package.json 時的預設」，不是強制版本。寫版本前一律走 `_global/rules.md` §7 的版本驗證三步驟。

---

## 共用模板（單一來源，各檔不重複）

### `errors.md` 紀錄模板（「適用版本」必填，因跨多版本專案使用）

```markdown
### 問題標題
- **適用版本**：Angular 18-20 / PrimeNG 16-17（範圍；上游已修復則註記）
- **日期**：YYYY-MM-DD
- **環境**：踩到時的精確版本
- **問題**：症狀
- **原因**：根因
- **解法**：步驟或程式碼
- **參考**：連結（可選）
```

### `session-log.md` 紀錄模板（新紀錄加最上面，倒序）

```markdown
### YYYY-MM-DD｜本次主題
- **專案**：
- **重點**：做了什麼、解了什麼、學到什麼
- **產出**：改了哪些檔案
- **後續**：待辦
```

筆記何時更新、更新原則（stack 被動 / conventions 決議型 / 事件型主動提議）見 `_global/rules.md` §6、§8。

---

## Git 操作（本 repo 的執行流程）

commit / 訊息 / 安全紀律見 `_global/rules.md` §15。本 repo 的操作流程：

- 改檔後**不自動 commit**，可主動建議 commit message
- scope 對應資料夾（`angular` / `nextjs` / `python` / `electron` / `postgres` / `godot` / `unity` / `vue` / `go` / `shared` / `global`）
- **不主動 push**；每次 commit 後**立刻詢問**：「已 commit `<sha-short>`，本地領先 origin/master N 個。要 push 嗎？」yes → push，no → 不動作（理由：切到其他專案後易忘記回來推送，保留每次確認權）

---

## 衝突處理

Hsin 當下指令與本 repo 規則衝突：**優先遵守當下指令**，並主動提醒既有規則、問是刻意還是要更新規則。版本衝突（如 stack.md 寫 Angular 17 但 Hsin 貼 Angular 19 code）→ 主動提議更新 stack.md。

## 結構演進

新增技術棧、結構變更、發現有用的新協作模式，或 Hsin 反覆叮嚀某條未列出的規則時，主動提議更新本檔或新增資料夾／拆檔（例如某 `conventions.md` 超過 200 行時拆獨立檔，見 `_global/rules.md` §14）。

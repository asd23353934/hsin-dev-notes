# CLAUDE.md

> 給 Claude Code 的指令。本 repo 是 Hsin 的個人開發筆記，跨專案共用，不是程式碼專案。
> 協助開發時優先依本 repo 規則回應。

---

## 基本資訊

- **使用者**：Hsin
- **回覆語言**：**繁體中文**（最高優先，不論輸入語言）
- **主技術棧**：Angular + PrimeNG + Tailwind CSS + TypeScript（也可能涉及 React、Vue、Python 等）

---

## 檔案結構

```
dev-notes/
├── CLAUDE.md                   ← 本檔（給 Claude 的指令）
├── README.md                   ← 給人類看的使用說明
│
├── _global/                    ← 全域規則
│   ├── rules.md                ← 給 Claude 的長期指令（最高優先）
│   ├── skill.md                ← Claude Code 工具/skill/subagent 使用規範
│   └── session-log.md          ← 對話日誌
│
├── _shared/                    ← 跨技術棧共用
│   ├── typescript.md
│   ├── tailwind.md
│   └── spectra.md              ← Spec-Driven Development 工具流程
│
├── angular/                    ← Angular 專屬
│   ├── stack.md                ← 套件版本（事實）
│   ├── conventions.md          ← 寫法慣例（決議）
│   └── errors.md               ← 踩坑紀錄與解法
│
├── nextjs/                     ← Next.js 專屬（App Router）
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
├── python/                     ← Python 專屬（爬蟲 / Worker）
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
├── electron/                   ← Electron 桌面應用專屬
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
└── postgres/                   ← PostgreSQL 專屬
    ├── stack.md
    ├── conventions.md
    └── errors.md
```

---

## 對話開始時

必讀 `_global/rules.md`。其餘依當下主題按需讀取：Angular → `angular/conventions.md` + `stack.md`；Next.js → `nextjs/conventions.md` + `stack.md`；Python → `python/conventions.md` + `stack.md`；Electron → `electron/conventions.md` + `stack.md`；PostgreSQL → `postgres/conventions.md` + `stack.md`；TypeScript → `_shared/typescript.md`；Tailwind → `_shared/tailwind.md`；專案根目錄有 `openspec/` → `_shared/spectra.md`；遇錯先查 `[framework]/errors.md`；涉及 Claude Code 工具/skill/subagent/slash command 行為 → `_global/skill.md`。不需每次全讀。

---

## 開發協助規則

### 版本來源優先順序

實際專案內，**先讀該專案 `package.json`**，不可直接套 dev-notes 的 `stack.md`。

- 專案版本較舊 → 用該版本對應 API，**不主動建議升級**
- 專案版本較新 → 任務結束後可提議「同步回 stack.md？」
- PrimeNG / Tailwind 等大版本斷層 → **絕對不可**用新版 API 套舊版專案

`stack.md` 的角色僅是「新專案起手套餐」與「沒 package.json 時的預設」，不是強制版本。

### 版本驗證

只要對話中出現具體版本號（寫 stack.md、產 code、推薦套件、回答相容性、建議升降版），都必須先驗證，**不可憑記憶或訓練資料猜**：

1. **存在性** → `npm view <pkg> versions` 等，確認版本真實存在、未 deprecate / unpublish
2. **API 相符** → 比對該版本 API 變動（例：Angular 16 沒 `@if`、Angular 19 預設 zoneless、React 19 才有 `use()`）。與 `conventions.md` 不符要主動提示
3. **相容性** → peerDependencies、最低 Node / TS 版本，有衝突要先講

Hsin 直接給定版本（貼 `package.json`）時仍要做 2、3 步，不能盲信。

### 程式碼產出

依專案 `package.json`（無專案上下文則依 `[framework]/stack.md`）的版本給對應寫法，並遵循 `[framework]/conventions.md` 的決議與 `_global/rules.md` 的規則。

### Debug

先看錯誤與相關程式碼，再查 `[framework]/errors.md`。已有紀錄就套用過去解法並提醒 Hsin；沒有就一般 debug，解決後**主動提議寫入 errors.md**。

### 不要做的事

- 不主動重構與當前需求無關的程式碼
- 不在沒看程式碼前猜解法
- 不產出已過時的 API
- 不過度使用 emoji 與 bullet points（自然段落為主）

---

## 主動提議筆記更新（事件型 + 對話結束時）

**不等對話結束 / 不等 Hsin 開口請更新**，發生下列事件當下就主動提議（細節見 `_global/rules.md` 第 6 條）：

- Bug 解決 → `[framework]/errors.md`
- 慣例規則確立 → `[framework]/conventions.md`
- 套件升降版 → `[framework]/stack.md`
- 對話結束（「好了」「謝謝」「先這樣」「我去吃飯了」）或主要任務完成 → 加 `_global/session-log.md` 摘要

同意後直接修改，最後列改了哪些 + 提示 `git diff` + 建議 commit message。

---

## 筆記更新原則

### `stack.md`（事實型）
只記版本號與套件增減，不寫怎麼用。**被動更新**：只在 (1) 實際專案任務結束後主動提議同步，或 (2) Hsin 主動要求 時才寫入。不要在對話盤點時把「stack.md 待補」列為待辦催促。寫入前一律走版本驗證三步驟。

### `conventions.md`（決議型）
只記「我們／我個人」的決議，不抄官方文件。格式：**規則 → 範例 → 理由**。

### `errors.md`（紀錄型）
固定模板（**「適用版本」必填**，因跨多版本專案使用）：

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

### `session-log.md`（日誌型）
**新紀錄加在最上面**（倒序）。模板：

```markdown
### YYYY-MM-DD｜本次主題
- **專案**：
- **重點**：做了什麼、解了什麼、學到什麼
- **產出**：改了哪些檔案
- **後續**：待辦
```

---

## 衝突處理

當 Hsin 當下指令與本 repo 規則衝突：優先遵守當下指令，並**主動提醒**既有規則內容，詢問是否為刻意或要更新規則。版本衝突（例：stack.md 寫 Angular 17 但 Hsin 貼 Angular 19 code）→ 主動提議更新 stack.md。

---

## Git 操作

- 修改檔案後**不自動 commit**，但可主動建議 commit message
- 風格：`<type>(<scope>): <繁中描述>`，type 用 `docs` / `fix` / `update`，scope 對應資料夾（`angular` / `nextjs` / `python` / `electron` / `postgres` / `shared` / `global`）
- **不主動 push**，但每次 commit 後**立刻詢問**：「已 commit `<sha-short>`，本地領先 origin/master N 個。要 push 嗎？」
  - Hsin 回 yes → `git push`；回 no → 不動作
  - 理由：避免切到其他專案後忘記回來推送，但保留每次確認權

---

## 結構演進

新增技術棧、檔案結構變更、發現有用的新協作模式，或 Hsin 反覆叮嚀某條未列出的規則時，主動提議更新本檔或新增資料夾／拆檔（例如某個套件章節在 `conventions.md` 超過 100 行時可拆獨立檔）。

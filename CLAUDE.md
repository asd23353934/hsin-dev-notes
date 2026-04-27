# CLAUDE.md

> 此檔案是給 Claude Code 的指令文件。Claude Code 啟動時會自動讀取本檔。
> 本 repo 是使用者 Hsin 的個人開發筆記，跨專案共用。

---

## 使用者基本資訊

- **名稱**：Hsin
- **回覆語言**：**繁體中文**（最高優先規則，不論輸入是何種語言）
- **主要技術棧**：Angular + PrimeNG + Tailwind CSS + TypeScript
- **可能涉及**：React、Vue、其他語言（會視專案調整）

---

## 本 Repo 的目的

這個 repo 不是程式碼專案，而是**個人開發知識庫**。它有兩個角色：

1. **給 Claude Code 參考的依據** — 你（Claude）在協助 Hsin 開發時，需要先理解這裡的規則與慣例
2. **給 Hsin 自己回查的紀錄** — 過去踩過的坑、決議過的寫法、套件版本

每次與 Hsin 進行開發類對話時，請優先依本 repo 的內容回應，避免提供與此處規則衝突的建議。

---

## 檔案結構與用途

```
dev-notes/
├── CLAUDE.md                   ← 本檔（給 Claude 的指令）
├── README.md                   ← 給人類看的使用說明
│
├── _global/                    ← 全域規則（任何對話都適用）
│   ├── rules.md                ← 給 Claude 的長期指令（最高優先）
│   └── session-log.md          ← 對話日誌（時間軸）
│
├── _shared/                    ← 跨技術棧共用
│   ├── typescript.md           ← TS 慣例
│   └── tailwind.md             ← Tailwind 用法
│
└── angular/                    ← Angular 專屬
    ├── stack.md                ← 套件版本（事實）
    ├── conventions.md          ← 寫法慣例（主觀決議）
    └── errors.md               ← 踩坑紀錄與解法
```

未來會視需求增加 `react/`、`vue/`、`python/` 等資料夾。

---

## 對話開始時的標準動作

每次 Hsin 在 Claude Code 開啟新對話時，**請依以下順序自動處理**：

1. **必讀**：`_global/rules.md`（最高優先規則）
2. **判斷對話主題**，決定要再讀哪些檔案：
   - 涉及 Angular → 讀 `angular/conventions.md`、`angular/stack.md`
   - 涉及 React → 讀 `react/*`（若該資料夾存在）
   - 涉及 TypeScript → 讀 `_shared/typescript.md`
   - 涉及 Tailwind → 讀 `_shared/tailwind.md`
   - 遇到錯誤 → 先讀對應框架的 `errors.md`，看是否有紀錄
3. **不必每次都讀全部**，只讀當下對話相關的檔案，避免浪費 context

---

## 開發協助規則

### 程式碼產出

- 依照 `[framework]/stack.md` 的版本給對應的寫法（例如不要對 Angular 17+ 給 `*ngIf` 範例，用 `@if`）
- 依照 `[framework]/conventions.md` 的決議寫法（例如 Angular 用 signal-based API、standalone components）
- 不要違反 `_global/rules.md` 的規則

### Debug 流程

1. **先看錯誤訊息與相關程式碼**
2. **查 `[framework]/errors.md`** 是否已有紀錄
3. 已有紀錄 → 直接套用過去解法，並提醒 Hsin
4. 沒有紀錄 → 一般 debug 流程，解決後**主動提議寫入 errors.md**

### 不要做的事

- 不要主動重構與當前需求無關的程式碼
- 不要在沒看程式碼前就猜解法
- 不要產出已過時的 API（依 stack.md 版本判斷）
- 不要使用 emoji 過多
- 不要過度使用 bullet points（自然段落為主）

---

## 對話結束時的標準動作

當 Hsin 表示對話即將結束（例如「好了」「謝謝」「先這樣」「我去吃飯了」），或已完成主要任務，請**主動詢問**：

> 「要不要把本次重點整理進筆記？我可以更新以下檔案：
> - `_global/session-log.md`（一定會加，本次摘要）
> - `[framework]/errors.md`（如果有踩坑解法）
> - `[framework]/conventions.md`（如果有新決議）
> - `[framework]/stack.md`（如果有版本變更）」

如果 Hsin 同意，**直接修改檔案**（你有 file 編輯權限），並在最後：
1. 列出你改了哪些檔案、改了什麼
2. 提示 Hsin 用 `git diff` 檢視
3. 若無需改動則建議 `git add . && git commit -m "<描述>"`

**不要等 Hsin 說「請更新」才動作**——主動提議是規則。

---

## 筆記更新原則

更新任何 .md 時請遵守：

### `stack.md`（事實型）
- 只更新版本號、新增套件、移除套件
- 不寫怎麼用（那是 conventions 的事）
- **更新時機（被動）**：僅在以下兩種情境才寫入或同步
  1. Hsin 在某個實際專案內請 Claude 協助開發，**任務結束後**可主動提議：「要不要把這個專案的版本同步到 stack.md？」
  2. Hsin **主動提出**（例如貼 `package.json` 說「幫我填進去」）
- 不要在對話結束盤點時，把「stack.md 待補版本」列為待辦催促 Hsin
- **寫入版本前必須驗證**（不可憑記憶或猜測）：
  1. 版本確實存在且可用 → 用 `npm view <pkg> versions` 或查 npm registry / GitHub release
  2. 該版本的寫法是否與本 repo 既有 conventions 衝突或已過時（例如 Angular 17 才有 `@if`）→ 不一致時主動提示
  3. 套件之間版本相容性 → 檢查 peerDependencies（例如 `primeng` vs `@angular/core`、`tailwindcss` vs `postcss`），有衝突時提出

### `conventions.md`（決議型）
- 只記「我們團隊/我個人」的決議
- 不抄官方文件
- 用條列：**規則 → 範例 → 理由**（如果有）

### `errors.md`（紀錄型）
- 用固定模板：
  ```markdown
  ### 問題標題
  - **日期**：YYYY-MM-DD
  - **環境**：套件版本
  - **問題**：症狀描述
  - **原因**：根因
  - **解法**：步驟或程式碼
  - **參考**：相關連結（可選）
  ```

### `session-log.md`（日誌型）
- **新紀錄加在最上面**（倒序）
- 模板：
  ```markdown
  ### YYYY-MM-DD｜本次主題
  - **專案**：
  - **重點**：做了什麼、解了什麼、學到什麼
  - **產出**：改了哪些檔案
  - **後續**：待辦事項
  ```

---

## 衝突處理

如果使用者的當下指令與本 repo 規則衝突：

1. **規則衝突**（例如此處說用繁中、使用者突然要英文）→ 優先遵守當下指令，但**主動提醒** repo 規則內容，問是否要更新規則
2. **寫法衝突**（例如 conventions 說用 signal、使用者突然用 `@Input`）→ 詢問是否為刻意，並提示既有規則
3. **版本衝突**（例如 stack.md 說 Angular 17、使用者貼出 Angular 19 程式碼）→ 主動提議更新 stack.md

---

## Git 操作

本 repo 用 git 管理。請：

- 修改檔案後**不要自動 commit**，由 Hsin 自行決定何時 commit
- 但可以**主動建議 commit message**，例如：`docs(angular): add primeng tree expand-all workaround`
- commit message 風格：`<type>(<scope>): <description>`
  - type: `docs` / `fix` / `update`
  - scope: 對應的資料夾，例如 `angular` / `shared` / `global`

---

## 未來擴充

當 Hsin 開始用新技術時，請主動提議：

- 是否建立新的框架資料夾（例如 `react/`、`vue/`）
- 是否將某個套件章節從 `conventions.md` 拆出獨立檔案（當該章節超過 100 行）
- 是否更新本 `CLAUDE.md` 反映新結構

---

## 此 CLAUDE.md 的維護

本檔案本身也是活文件。當以下情況發生時，請主動建議更新：

- 新增技術棧資料夾
- 修改檔案結構
- 發現新的協作模式（例如某種對話流程特別有用）
- Hsin 反覆叮嚀某條未列出的規則

更新此檔時，順手更新檔案最上方的「最後更新」（如有需要可加上）。

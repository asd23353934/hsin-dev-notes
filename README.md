# dev-notes

> 個人開發知識庫，跨專案共用，搭配 Claude Code 使用。

---

## 這個 Repo 在做什麼？

這是 **Hsin 的個人開發筆記系統**，包含：

- 各技術棧的套件版本與寫法慣例
- 踩過的坑與解法
- 給 AI 助手（Claude Code）的長期協作指令

設計目標：**讓 Claude Code 在任何專案開發時，都能參考此處規則協助 Hsin**。

---

## 結構

```
dev-notes/
├── CLAUDE.md             # Claude Code 自動讀取的指令
├── README.md             # 本檔
│
├── _global/              # 全域規則（任何對話都適用）
│   ├── rules.md          # 給 Claude 的長期指令
│   ├── skill.md          # Claude Code 工具/skill/subagent 使用規範
│   └── session-log.md    # 對話日誌
│
├── _shared/              # 跨技術棧共用
│   ├── typescript.md
│   ├── tailwind.md
│   └── spectra.md        # Spec-Driven Development 工具流程
│
├── angular/              # Angular 專屬
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
├── nextjs/               # Next.js 專屬（App Router）
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
├── python/               # Python 專屬（爬蟲 / Worker）
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
├── electron/             # Electron 桌面應用專屬
│   ├── stack.md
│   ├── conventions.md
│   └── errors.md
│
└── godot/                # Godot 4 / GDScript 專屬（遊戲開發）
    ├── stack.md
    ├── conventions.md
    ├── errors.md
    └── plugins.md         # plugin 採用指南（含「不裝」理由）
```

未來可能新增 `react/`、`vue/` 等資料夾。

---

## 如何使用

### 與 Claude Code 搭配

1. 在此 repo 啟動 Claude Code：
   ```bash
   cd ~/dev-notes
   claude
   ```
2. Claude Code 會自動讀取 `CLAUDE.md`，理解整個 repo 結構
3. 開始對話即可

### 在其他專案中使用此筆記

**核心原則**：dev-notes 的 `stack.md` 是「新專案起手套餐 / 預設假設」，**不是強制現行版本**。實際專案內優先以該專案 `package.json` 為準（詳見 `CLAUDE.md`「版本來源優先順序」）。

#### 推薦做法：每個專案有自己的 `CLAUDE.md`

在實際開發專案根目錄放 `CLAUDE.md`，引用 dev-notes 同時記錄該專案的特殊限制：

```markdown
# <專案名稱> CLAUDE.md

## 參考來源

請同時參考 `~/dev-notes/CLAUDE.md` 的全域規則與慣例（語言、回覆風格、版本驗證、筆記更新流程等）。

## 本專案版本（以 package.json 為準）

- Angular: 18.2.x
- PrimeNG: 17.18.x（仍用舊 SASS theming，**未升 Aura**）
- Tailwind: 3.4.x（**v3 寫法**，`tailwind.config.js`）
- ESLint: 8.x（legacy `.eslintrc.json`，**尚未升 flat config**）
- Node: 20 LTS

## 與 dev-notes 的差異 / 限制

- **不要**直接套用 dev-notes 的 PrimeNG Aura 寫法 → 此專案仍用 SASS theming
- **不要**建議升 Tailwind v4 / ESLint flat config / Angular 21（升級需另外排程）
- 若提到「Angular 18 沒有的 API」（例如 zoneless 預設），先確認再給範例

## 本專案專屬慣例

（寫此專案獨有的決議，不必跟 dev-notes 重複）

- 例：API client 一律走 `core/api/`，不用 dev-notes 寫的 `services/`
- 例：表單錯誤訊息走自家 `<form-error>` 元件
```

> **這個 pattern 解決三件事**：
> 1. dev-notes 維持單一前沿版本，不需要為每個舊專案開分支
> 2. 該專案的版本鎖死，Claude 不會誤用新版 API
> 3. 專案專屬慣例與 dev-notes 公共慣例分離，不互相污染

#### 替代做法（簡單情境）

**做法 B**：用 symlink 把 `_global/rules.md` 連結到專案：
```bash
ln -s ~/dev-notes/_global/rules.md /path/to/project/CLAUDE-RULES.md
```

**做法 C**：直接在 `~/.claude/CLAUDE.md`（全域）寫：
```markdown
請優先參考 ~/dev-notes/ 的所有 .md 檔案內容。
```

> 做法 B/C 適合**新專案 / 與 dev-notes 版本一致**的情境。舊專案請走推薦做法。

---

## 更新流程

### 平常開發中

對話結束時，Claude Code 會主動詢問是否更新筆記，依其建議檢視 `git diff` 後 commit 即可。

### 手動更新

直接用 VS Code 或任何編輯器改 .md，照常 git workflow：

```bash
git add .
git commit -m "docs(angular): update primeng tree workaround"
git push
```

---

## Commit 慣例

```
<type>(<scope>): <description>

type: docs / fix / update / refactor
scope: 對應資料夾，e.g. angular / shared / global
```

範例：

```
docs(angular): add signal-based form pattern
fix(shared): correct typescript strict mode setting
update(global): clarify error log format
```

---

## 版本歷史

- **2026-04-27** 初始建立，含 Angular 框架資料夾、_global、_shared 結構

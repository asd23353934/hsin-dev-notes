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
│   └── session-log.md    # 對話日誌
│
├── _shared/              # 跨技術棧共用
│   ├── typescript.md
│   └── tailwind.md
│
└── angular/              # Angular 專屬
    ├── stack.md          # 套件版本
    ├── conventions.md    # 寫法慣例
    └── errors.md         # 踩坑紀錄
```

未來會新增 `react/`、`vue/` 等資料夾。

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

選擇一種：

**做法 A**：在實際開發專案的 `CLAUDE.md` 加上指向：
```markdown
# 專案 CLAUDE.md
請同時參考 ~/dev-notes/CLAUDE.md 的全域規則與慣例。
```

**做法 B**：用 symlink 把 `_global/rules.md` 連結到專案：
```bash
ln -s ~/dev-notes/_global/rules.md /path/to/project/CLAUDE-RULES.md
```

**做法 C**：直接在 `~/.claude/CLAUDE.md`（全域）寫：
```markdown
請優先參考 ~/dev-notes/ 的所有 .md 檔案內容。
```

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

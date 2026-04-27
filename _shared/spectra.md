# Spectra（Spec-Driven Development）

> 跨技術棧共用的 SDD 工具。當專案根目錄有 `openspec/` 資料夾時即啟用。
> 最後更新：2026-04-27

---

## 是什麼

Spectra 是一套以 **規格先行（Spec-Driven Development）** 的協作流程，搭配 Claude Code 的 `/spectra-*` skill 系列使用。重點：

- **Specs** 放 `openspec/specs/` — 描述系統現有行為
- **Changes** 放 `openspec/changes/` — 描述「即將要改成什麼」的提案
- 變更實作完成後 archive，提案內容合併回 spec

特性：每個 change 都有自己的目錄、tasks、artifacts，讓對話可以中斷後繼續、可以多人協作。

---

## 何時用

| 情境 | 對應 skill |
|------|-----------|
| 討論需求，需要結構化收斂 | `/spectra-discuss` |
| 規劃新變更（API、資料模型、流程） | `/spectra-propose` |
| 變更內 tasks 開始實作 | `/spectra-apply` |
| 中途需求變動，要把新資訊吃進現有 change | `/spectra-ingest` |
| 查 spec、問「這個功能怎麼運作」 | `/spectra-ask` |
| 變更實作完成 | `/spectra-archive` |
| 只 commit 與某個 change 相關的檔案 | `/spectra-commit` |
| 安全審查 | `/spectra-audit` |
| 系統化 debug 流程 | `/spectra-debug` |

---

## 工作流

```
discuss?  →  propose  →  apply  ⇄  ingest  →  archive
```

- `discuss` 是選配 — 需求清楚就跳過
- 中途需求變更 → 進 plan mode → `ingest` 把新資訊寫回 change → 繼續 `apply`
- 不要在 `apply` 中段「順手」改 spec — 要嘛 `ingest`，要嘛新開 change

---

## 何時跳過 Spectra

不是所有變更都要走 SDD：

- **小修小補**（typo、一行 bug fix、樣式微調）→ 直接改 + commit
- **純探索 / 撈資料**（讀 log、跑 query 看數字）→ 不留 artifact
- **臨時實驗**（待會就刪）→ 跑就跑，不開 change

判斷準則：「這個變更值不值得日後回查？會不會影響別人？」是 → propose；否 → 直接做。

---

## Pre-commit 流程（搭配 Spectra）

如果專案有設定 pre-commit hook，commit 前依序執行：

1. `/simplify` — 檢查重複邏輯、不必要的複雜度、可重用性
2. `/spectra:audit` — 安全漏洞掃描（OWASP Top 10、危險預設值、型別混淆、靜默失敗）
3. **同步文件** — 新功能 / 新 API / 新環境變數 → 同步更新 `CLAUDE.md` 與 `README.md`

三步全綠才 commit。發現問題就先修。

---

## Parked Changes（暫存）

- 暫時不繼續但不想刪的 change → `spectra park <name>`
- 不會出現在 `spectra list`，但 `spectra list --parked` 看得到
- 恢復用 `spectra unpark <name>`
- `/spectra-apply` 與 `/spectra-ingest` 會自動偵測 parked

---

## 給 Claude 的提示

啟動對話時若看到根目錄有 `openspec/` 資料夾，**主動辨識**這是 Spectra 專案，並：

- 提案新功能 → 直接走 `/spectra-propose`，不要先寫 code
- 收到「繼續做 X 那個 change」→ 走 `/spectra-apply` 或 `/spectra-ingest`
- 看到 `CLAUDE.md` 內有 `# Spectra Instructions` 區塊 → 那是該專案的 SDD 指引，請遵守

不要把 Spectra 的 change 提案文件當成一般 markdown 改 — 它是有結構的 artifact，用 skill 操作比手改更穩。

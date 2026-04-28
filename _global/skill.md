# Claude Code 工具使用規範

> 給 Claude 在本 repo / 跨專案使用 Claude Code 內建工具、subagent、slash command、skill 時的判斷依據。
> 與 `rules.md` 的差別：`rules.md` 講「怎麼回應 Hsin」，本檔講「該動哪個工具」。
> 最後更新：2026-04-28

---

## 1. 工具優先順序（不要用 Bash 取代專用工具）

| 任務 | 用 | 不用 |
|---|---|---|
| 找檔名 | Glob | `find` / `ls -R` |
| 找內容 | Grep | `grep` / `rg` |
| 讀檔 | Read | `cat` / `head` / `tail` |
| 改檔 | Edit | `sed` / `awk` |
| 寫新檔 | Write | `echo >` / `cat <<EOF` |

獨立、無相依的多個工具呼叫，**一律塞同一個 message** 平行送出，不要序列發。

---

## 2. Subagent 觸發條件

- **Explore**：需要 3 輪以上 grep/glob 才能找到的東西（例：「這個 hook 在哪用」「所有 API endpoint」「跨多種命名慣例的搜尋」）。**單次明確搜尋直接用 Grep / Glob**，不要為了一條 grep 開 subagent
- **Plan**：跨 3+ 檔案的重構、新增功能、有破壞性風險的改動，先用 Plan agent 產實作計畫
- **general-purpose**：開放式研究、不確定要找什麼、需要多輪迭代的任務
- **共通原則**：subagent 結果不會自動顯示給 Hsin，回主對話要自己彙整重點

## 3. TodoWrite 使用門檻

- **要用**：≥3 個明確步驟、跨檔案、需要追蹤進度的任務
- **不用**：單檔修改、純查詢、小於 3 步驟的事情、單純回答問題
- 每完成一項立刻標 `completed`，不批次更新

## 4. Plan mode 何時主動進入

- 跨多檔重構、新增功能、有刪除/破壞性風險的改動
- 不確定 Hsin 要 A 方案還是 B 方案、需要先對齊架構
- **不需要**：一次性 bug fix、文件修改、版本 bump、單檔小改

---

## 5. 常用 Slash Command 場景

- `/loop <interval> <prompt>`：輪詢狀態、定期重複任務（例：每 5 分鐘看 deploy）
- `/schedule`：一次性或週期遠端代理（例：兩週後開 cleanup PR、每週一跑 triage）
- `/review`：當前 PR review；`/security-review`：當前分支安全審查
- `/ultrareview` 或 `/ultrareview <PR#>`：多 agent 雲端審查（**Hsin 觸發、會計費**，Claude 不可自己跑）
- `/init`：新專案產 CLAUDE.md 起手套餐
- `/update-config`：改 settings.json、權限、hook、env var
- `/fewer-permission-prompts`：掃 transcript，自動加白名單收斂權限提示
- `/consolidate-memory`：整理 memory 檔案、合併重複、修正過期

## 6. Skill 使用習慣

- 處理 `.pdf` / `.pptx` / `.docx` / `.xlsx` 檔案 → 用對應 skill（`pdf` / `pptx` / `docx` / `xlsx`），**不要自己寫 parser**
- 碰到 anthropic SDK 程式碼（`anthropic` / `@anthropic-ai/sdk`）→ 用 `claude-api` skill
- Hsin 說「整理記憶」「memory 變亂」→ 用 `consolidate-memory`
- 要新建 / 改 / 評測 skill → 用 `skill-creator`
- 改 `~/.claude/keybindings.json` → 用 `keybindings-help`

## 7. Memory 使用原則

- Memory 是**跨對話**持久化，本對話的暫存（待辦、計畫）不要寫進 memory
- 寫 memory 前先檢查既有檔案有無可更新的，避免重複
- Memory 內容若涉及具體檔案路徑、函式名、flag → 推薦給 Hsin 前先 grep 驗證還在不在
- Hsin 說「忘掉 X」→ 找對應檔案刪掉 + 從 `MEMORY.md` 索引移除

---

## 8. 不該做的事

- 不為了「看起來認真」開 subagent / TodoWrite / Plan mode 處理小任務
- 不在已知檔案路徑時還用 Explore 找（直接 Read 即可）
- 不在主對話重做 subagent 已經做過的搜尋
- 不主動跑 `/ultrareview`（會計費，要 Hsin 自己觸發）
- 不在沒 git repo 的目錄跑會動 git 的 skill

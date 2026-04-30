# 桌面 App 自動更新流程

> 跨專案經驗整理。Python/PyInstaller + PowerShell launcher 模式踩到的雷與設計範本。
> 適用：任何 Windows 桌面 app 用「主程式下載新版 ZIP → 啟動 launcher → swap exe → 重啟」這種 self-update 架構。

---

## 設計範本

整體流程：

1. 主程式啟動後背景查 GitHub release，比對版本
2. 偵測到新版 → **不主動跳 modal dialog**（擾），而是 header / status bar 的可點 chip + toast 通知
3. 使用者點 chip → 跳 UpdateDialog → 下載 ZIP 到 temp
4. 下載完 → 啟動 launcher（外部 process），主程式自己關閉
5. launcher 等舊 PID 退出 → backup .exe → 解壓 ZIP → 啟動新 exe

關鍵設計決定：

- **launcher 啟動方式**：`subprocess.Popen(args, creationflags=subprocess.CREATE_NO_WINDOW)`。**不要加 `DETACHED_PROCESS`**（見下文 Issue 2）。
- **失敗時雙保險**：launcher 跑失敗（解壓失敗 / exe 鎖定 / 無法重啟）時 (a) 立即跳 Windows MessageBox 告知使用者，(b) 寫 `update_failed.txt` marker 到 AppDir，主程式下次啟動讀檔 → toast warning → 刪檔。第二道是備援，避免使用者沒看到 MessageBox。
- **Marker 偽造防禦**：marker 在 AppDir 同 user 可寫，惡意 process 可寫釣魚訊息。讀 marker 時做 sanitize：限制 reason 字串長度（200 字）、剔除控制字元、含 URL（`://`）的 reason 直接 fallback「未知原因」、不 echo 攻擊者控制的長字串。
- **ZIP 結構與 AppDir 對齊**：launcher 解 ZIP 到 `Split-Path AppDir -Parent`，所以 ZIP 內第一層目錄名必須等於 AppDir 最後一層目錄名（例 `skill_tracker/`）。否則新 exe 解到別處，launcher 找不到 → restore backup → 升級失敗。
- **ps1 必加 UTF-8 BOM**（見 Issue 4）。

---

## Issue 1: PyInstaller 6.x 把 `('file', '.')` datas 放 `_internal/` 而非 dist root

- **適用版本**：PyInstaller 6.x（含 6.19.0；5.x 行為不同）
- **日期**：2026-04-30
- **環境**：PyInstaller 6.19.0 / PySide6 / Windows / onedir mode
- **問題**：spec 寫 `datas=[('update_launcher.ps1', '.')]`，build 後 launcher.ps1 在 `dist/skill_tracker/_internal/`，不在 `dist/skill_tracker/`。主程式用 `os.path.dirname(sys.executable)` 找 launcher 找不到，回退到 dialog 顯示「找不到更新腳本」。
- **原因**：PyInstaller 6.x onedir 結構從 dist root 改為 `_internal/`，spec 的 `('.')` 解析為 `_internal/.`。
- **解法**：build 後手動 post-process 把 launcher 複製到 dist top-level（zip_release.py 寫 `shutil.copy2(launcher_name, src_dir)`）。或修 spec.py 用絕對路徑指 dist root（仍要查 PyInstaller 6.x 是否支援，本次未驗證）。
- **參考**：踩到時整段 launcher 完全沒被啟動（`update_log.txt` 不存在），主程式只顯示「找不到更新腳本」。

---

## Issue 2: `subprocess.Popen(creationflags=DETACHED_PROCESS | CREATE_NO_WINDOW)` 啟動 PS 立刻死

- **適用版本**：Python 3.x / Windows / 啟動 powershell.exe 跑 .ps1
- **日期**：2026-04-30
- **環境**：Python 3.13 / Windows 10 / powershell.exe 5.1
- **問題**：subprocess.Popen 啟動 PS launcher 後，PS process exit code 0 但完全沒執行 script body — `update_log.txt` 不存在。看起來 launcher 從未啟動。
- **原因**：`DETACHED_PROCESS | CREATE_NO_WINDOW` 兩個 flag 組合會讓 PowerShell 立刻退出。Microsoft 文件其實標明這兩個 flag 是 mutually exclusive，但 Python 不會擋下，PS 自己處理時行為不可預期。
- **解法**：只用 `subprocess.CREATE_NO_WINDOW`，移除 `DETACHED_PROCESS`。launcher 一樣不會跳 console，但能正常執行 script。
- **參考**：debug 時改用同步 `& powershell.exe ... -File script.ps1` 跑能正常 work，detached 模式才會踩到。

---

## Issue 3: PowerShell 5.1 雙引號 `$var (text)` / `("a" + $var + "b")` parse quirk

- **適用版本**：PowerShell 5.1（Windows 10/11 內建版本；PowerShell 7+ 未驗證）
- **日期**：2026-04-30
- **環境**：powershell.exe 5.1.19041.6456
- **問題**：兩種 PS 5.1 雙引號內 expression parse 失敗模式：
  1. `Write-Log "Started $AppExe (fallback)"` — PS 把 `$AppExe (fallback)` 誤認為 method/cmdlet call，報 `fallback : The term 'fallback' is not recognized`。
  2. `Write-Log ("=== finished (success=" + $success + ") ===")` — PS 5.1 對 expression group 內 string + var + string 的 concat 有 parse quirk，把 `success= + $success +` 整段當作 cmdlet name 嘗試 invoke。
- **原因**：PS 5.1 parser 對雙引號內括號配對 / expression group 處理有歷史 bug。PS 7+ 已修。
- **解法**：用變數預建 string，再傳 cmdlet。PS 內慣用 pattern：`$msg = "..."; Write-Log $msg`。或單純避免在雙引號內含 `()` 字面字符：「Started $AppExe」用 `[fallback]` 不要 `(fallback)`。
- **參考**：transcript 顯示 PS 在這幾行 throw 錯，但被 catch 接住或 silent skipped。

---

## Issue 4: PowerShell 5.1 .ps1 file 缺 UTF-8 BOM 用 cp950 讀中文 → 整段 silent skip

- **適用版本**：PowerShell 5.1（Windows 中文版預設 cp950）；PS 7+ 預設 UTF-8 不踩
- **日期**：2026-04-30
- **環境**：powershell.exe 5.1.19041.6456 / Windows 10 中文版 / cp950
- **問題**：launcher 跑時 [2/4] backup 寫 log 後直接跳到 `if(-not $success)` block 寫 「Restoring backup...」，**完全跳過 [3/4] 整段** — [3/4] 內所有 `Write-Log` 都不寫 log 也不報錯。Transcript 也沒 syntax error 訊息。
- **原因**：.ps1 file 沒 UTF-8 BOM，PS 5.1 預設用系統 ANSI codepage（cp950 in 中文 Windows）讀檔。檔案內含中文字串（comment + literal string）的 UTF-8 多 byte 序列被當 cp950 解碼，[3/4] block 整段 parse 失敗。但 PS 5.1 的容錯模式會 silent skip parse 失敗的整段，control flow 跳過去，沒任何 visible error。
- **解法**：強制 .ps1 file 帶 UTF-8 BOM（`EF BB BF` 開頭）。加完 BOM 後 PS 5.1 認 UTF-8，整段中文 parse 正常。

  ```powershell
  $content = [System.IO.File]::ReadAllText($ps1, [System.Text.Encoding]::UTF8)
  $utf8Bom = New-Object System.Text.UTF8Encoding $true
  [System.IO.File]::WriteAllText($ps1, $content, $utf8Bom)
  ```

  注意 git 預設不 strip BOM，但某些編輯器（VSCode 預設無 BOM）會 strip。建議加 `.gitattributes` 強制 ps1 file 保留 BOM，或在 build pipeline 內驗證 BOM 存在。
- **參考**：這是最隱蔽的雷 — 沒任何 error message，log 不會告訴你 [3/4] 沒跑。要靠 Start-Transcript 或單獨手動跑 launcher with `2>&1` 才看得到 PS 把整段視為 invalid。

---

## 相關慣例

- **PowerShell 內部 log function** 用 `[System.IO.File]::AppendAllText($logFile, $line, [System.Text.Encoding]::UTF8)` 比 `Out-File -Append -Encoding utf8` 更可預期（後者在 PS 5.1 對 BOM 處理有 quirk）。
- **launcher 內 MessageBox** 用 `Add-Type -AssemblyName System.Windows.Forms; [System.Windows.Forms.MessageBox]::Show(...)`。launcher 啟動時主程式已關閉，dialog 沒有 parent — 直接 attach 到 desktop。bat 版透過 `powershell -Command "..."` wrap，訊息傳遞**用 env var**（`set DLG_REASON=%~1; powershell -Command "...$env:DLG_REASON..."`）避免 cmd → powershell 雙層引號 escape 災難。
- **驗證 ps1 含中文時**：寫完 ps1 後立即手動跑（不靠 build pipeline），用 `powershell.exe -File script.ps1 ...` 同步跑，看 stderr 有沒有 syntax error。如果 stderr 乾淨但 log 內缺段落，懷疑 BOM。

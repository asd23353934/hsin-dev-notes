# Python 踩坑紀錄

> Python / Playwright / httpx / 爬蟲相關問題與解法。
> 遇到錯誤先查這裡有沒有紀錄；解完新坑請補上。
> 最後更新：2026-08-31

---

## 紀錄模板

```markdown
### 問題標題
- **適用版本**：Python 3.10+ / Playwright 1.4x（**必填**，標明此問題存在/解法成立的版本範圍；上游已修復則註記）
- **日期**：YYYY-MM-DD
- **環境**：實際當下踩到時的精確版本（例：Python 3.12.3 / Playwright 1.51.0 / Chromium 134）
- **問題**：症狀描述（看到什麼錯誤訊息、什麼行為不對）
- **原因**：根因分析
- **解法**：步驟或程式碼
- **參考**：issue / 文件 / Stack Overflow 連結（可選）
```

> **「適用版本」與「環境」差別**：
> - **適用版本** = 這個問題/解法在哪些版本範圍成立（給未來查找用）
> - **環境** = 你實際踩到時的精確版本（給可重現用）

---

## 紀錄

### 循環計時器逐輪累積漂移（fixed-delay vs fixed-rate）
- **適用版本**：任何 Python 計時 / 排程（time loop、asyncio、Qt QTimer 皆適用）
- **日期**：2026-06-01
- **環境**：Python 3.13.7 / PySide6 6.10.2 / Windows 10
- **問題**：週期性重複的計時（倒數歸零後重啟、固定間隔輪詢）長時間跑會「越來越慢／偏移」，每輪慢一點點、累積成可觀秒差。
- **原因**：每輪重啟時把起點設成「當下」（`start = perf_counter()`）。偵測歸零有 tick 粒度延遲、加上處理/排程延遲（甚至刻意的隨機延遲），這些誤差被當成新起點 → 逐輪累加，永不回正（fixed-**delay** 排程）。
- **解法**：改用 fixed-**rate**——下一輪起點錨定到「上一輪的理論結束點」而非當下：`start = end_time`（`end_time = 上一輪 start + period`）。誤差被吸收、不累積。並對「落後超過一個完整週期」（休眠 / 卡頓 / debugger 暫停）做 clamp，重新對齊到當下，避免一次爆衝補進度：`if now - start > period: start = now`。
- **驗證法**：零侵入觀察者 + 雙時鐘（`perf_counter` 與 `time.time`）量「固定牆鐘時間內循環幾次」，最直覺；或量歸零時刻 vs 理想固定排程的累積差。

### ctypes 在 64-bit Windows 必須宣告 HWND/HANDLE 的 restype/argtypes
- **適用版本**：Python 3.x（64-bit）on Windows，ctypes 呼叫 Win32 API
- **日期**：2026-06-01
- **環境**：Python 3.13.7 / Windows 10
- **問題**：用 ctypes 呼叫 `GetForegroundWindow` / `OpenProcess` 等回傳 handle 的 API，拿到的 handle 後續使用失敗或行為錯亂、偶發崩潰。
- **原因**：未宣告 `restype` / `argtypes` 時，ctypes 預設把回傳值與整數參數當 `c_int`（32-bit）。在 64-bit 行程裡 HWND/HANDLE/HDC 是 64-bit 指標，被截成 32-bit → 錯誤 handle。
- **解法**：對每個涉及指標型 handle 的函式明確宣告型別，例如 `user32.GetForegroundWindow.restype = wintypes.HWND`、`kernel32.OpenProcess.restype = wintypes.HANDLE`、`argtypes` 也比照（`wintypes.HWND` / `HANDLE` / `HDC` / `POINTER(DWORD)`）。GDI 物件記得 `try/finally` 內 `DeleteObject` / `DeleteDC` / `ReleaseDC` 防 handle 洩漏。

### PrintWindow 對被遮擋的 GPU/Chromium 視窗回空白
- **適用版本**：Windows `user32.PrintWindow`（含 `PW_RENDERFULLCONTENT=2`）
- **日期**：2026-06-01
- **環境**：Windows 10；目標視窗為 Chrome / Electron 等 GPU 加速 app
- **問題**：用 PrintWindow 擷取視窗縮圖，對一般 GDI 視窗正常，但對 Chrome / Electron（Chromium 系）在「被其他視窗遮擋／非前景」時擷取結果整片空白（白/黑）；同一視窗在前景可見時卻擷取正常。
- **原因**：GPU 加速 / DirectComposition 視窗被遮擋時可能未主動 present frame，PrintWindow 取到空白 surface。`PW_RENDERFULLCONTENT` 改善多數情況但對這類 app 仍不可靠。
- **解法**：要做「z-order 無關、含 GPU app」的視窗預覽，正解是 DWM 縮圖 API `DwmRegisterThumbnail` + `DwmUpdateThumbnailProperties`（工作列 / Alt-Tab 用的就是它，為即時合成預覽、非靜態點陣圖）。若只能用 PrintWindow，至少對最小化（`IsIconic`）/ 擷取失敗退回顯示程式圖示 + 標題。

### QImage(bytes,…) 的緩衝生命週期與顯式 stride
- **適用版本**：PySide6 / PyQt（QImage 由原始 bytes 建構）
- **日期**：2026-06-01
- **環境**：Python 3.13.7 / PySide6 6.10.2
- **問題**：用 `QImage(data, w, h, fmt)`（data 為 Python bytes）建圖，顯示時花圖或崩潰；或日後改像素格式後靜默讀錯。
- **原因**：(1) QImage 不會複製傳入的 bytes，只持有指標；bytes 被 GC 後 → use-after-free。(2) 不傳 `bytesPerLine` 時 Qt 以 `w*bytesPerPixel` 推算 stride，與來源若有列填充（如 24-bit 行對齊 4 bytes）不符 → 讀越界/錯位。
- **解法**：(1) 建完立刻 `.copy()` 取得自有緩衝再用：`QPixmap.fromImage(QImage(data, w, h, w*4, QImage.Format.Format_RGB32).copy())`。(2) 顯式傳 `bytesPerLine`（32-bit 為 `w*4`）把佈局契約寫明。BGRA top-down 的 DIB 直接對應 `Format_RGB32`（小端序 BGRX，忽略 alpha），免手動換 channel。

### 半透明無框視窗（WA_TranslucentBackground）alpha=0 區域會「點擊穿透」
- **適用版本**：PySide6 / PyQt on Windows，frameless + `WA_TranslucentBackground` 視窗
- **日期**：2026-06-17
- **環境**：Python 3.13 / PySide6 6.x / Windows 10
- **問題**：自繪卡片在無框半透明對話框裡「看得到卻點不到」——某些區域（尤其貼了 PrintWindow 截圖的縮圖）點下去沒反應、事件像被吃掉；純文字 / 退回圖示等不透明區卻正常。診斷發現該區的滑鼠按下「根本沒進到這個視窗的任何 widget」。
- **原因**：`WA_TranslucentBackground` 讓視窗成為分層視窗（per-pixel alpha）。Windows 對分層視窗用**逐像素 alpha 做命中測試**：alpha=0 的像素被當「不在視窗上」，點擊直接穿透到背後視窗 → 事件連本 process 的視窗都到不了，任何 widget 的 mousePressEvent / event filter 都攔不到。截圖縮圖（PrintWindow 來源 alpha 多為 0）最易整片變穿透。
- **解法**：需要「整片可點」的視窗就**別用半透明**——關掉 `setAttribute(Qt.WidgetAttribute.WA_TranslucentBackground, False)` 改不透明視窗（標準矩形命中測試，全區可點），或確保該區實際畫出 alpha=255 的不透明像素。診斷訣竅：在懷疑的 widget 的 mousePressEvent 或一個 app 層 event filter 記 log——「看得到卻完全沒 log」＝視窗層級穿透，不是 widget 命中問題（別再瞎調 event filter / WA_TransparentForMouseEvents）。

### PowerShell Expand-Archive 解「大量小檔」極慢；改用 .NET ZipFile
- **適用版本**：Windows PowerShell 5.1（內建）的 `Expand-Archive`；經 Python subprocess 呼叫亦同
- **日期**：2026-06-17
- **環境**：Windows 10 / PowerShell 5.1；解 PyInstaller onedir bundle（448 檔 / 167MB）
- **問題**：自動更新解壓 release ZIP 很慢（使用者按更新後重啟等很久）；bundle 是 onedir（`_internal` 含數百個 DLL 小檔）。
- **原因**：PS 5.1 的 `Expand-Archive` 把每個 entry 走 PowerShell 物件 pipeline，對「大量小檔」效能極差。
- **解法**：改用 .NET 直接解壓——`Add-Type -AssemblyName System.IO.Compression.FileSystem`、`[IO.Compression.ZipFile]::OpenRead($zip)` 後逐 entry `[IO.Compression.ZipFileExtensions]::ExtractToFile($entry, $target, $true)`（第 3 參數 `$true`=覆寫，等同 `-Force`）。實測 448 檔 5.5s→1.7s（~3.2x；慢碟 / 防毒重的機器差更多）。**務必加 zip-slip 防護**：`$target` 用 `[IO.Path]::GetFullPath` 解析後須仍以解壓根目錄（含尾端分隔符）為字首，否則惡意 `../` entry 會寫出目錄外。保險：包在 `try` 裡，失敗就 fallback 回 `Expand-Archive`（行為不退步）。

### Claude Code Bash tool 不能用 PowerShell here-string（`@'...'@`）
- **適用版本**：Claude Code（Bash tool = Git Bash / POSIX sh）；與 Python 版本無關
- **日期**：2026-07-05
- **環境**：Windows 11 + Claude Code Bash tool
- **問題**：在 Bash tool 裡用 `git commit -m @'...多行...'@` 下 commit，訊息開頭混入字面 `@`，且多行結構跑掉（subject 變成 `@`）。
- **原因**：`@'...'@` 是 **PowerShell** here-string 語法。Bash tool 底層是 POSIX sh，會把 `@'` 拆成字面 `@` ＋ 單引號字串起頭，`'@` 同理，於是 `@` 被當成訊息內容。兩個工具的引號語法完全不通用。
- **解法**：Bash tool 的多行字串一律用 heredoc —— `git commit -F - <<'EOF' … EOF`（單引號 EOF 讓 `$` 與反引號原樣保留），或先 `cat > /tmp/cm.txt <<'EOF' … EOF` 再 `git commit -F /tmp/cm.txt`。`@'...'@` 只在 PowerShell tool 用，且結尾 `'@` 必須頂格獨立一行。已誤 commit 時 `git commit --amend -F <檔>` 修正。
- **重點**：**動手前先確認自己在哪個 shell**（Bash tool = sh，PowerShell tool = PowerShell），多行 / 含特殊字元的字串兩邊語法互不相容。

### 用已被更新的狀態去比對「前後是否改變」→ 判斷恆為否，分支永遠不執行
- **適用版本**：與語言無關（Python / TS 都會踩）；本次在 Python 3.13 + PySide6 遇到
- **日期**：2026-08-31
- **環境**：Python 3.13 / PySide6 6.7（skill_tracker，domain service 與 UI 層分離）
- **問題**：批次切換（「全部常駐」）明明有改到狀態，UI 端該建立 / 該收掉的浮動視窗卻完全沒動作，也不噴任何錯誤；單一項目切換看起來卻是正常的，因此一直沒被發現。
- **原因**：UI 端 helper 寫成 `was = self.state.get(sid, False)` 再比 `if is_on and not was` / `elif not is_on and was`。但 `self.state` 是 property 直接回傳 service 內部 dict 的**共享參考**，而呼叫端進來之前已經先呼叫 service 改過狀態 → `was` 永遠等於 `is_on`，兩個分支都進不去。單項路徑「看起來正常」是因為它另外還有一段自己的建立邏輯蓋過去，把 bug 遮住了。
- **解法**：不要在狀態已經寫入之後才去比對前後差異。二選一：(1) 呼叫端在改狀態**之前**先把舊值抓下來傳進去；(2) 不看差異，只依**當下的目標狀態**做冪等（idempotent）收斂 —— 本次採這個：`want = perm.get(sid) or loop.get(sid)`，`want and sid not in windows` 就建、`not want and sid in windows` 就關。順帶讓「兩個互斥旗標其一為開就保留視窗」這種規則變得直觀。
- **重點**：只要 state 是**共享參考**（property 直接回傳內部 dict / list），「比對前後差異」就很脆弱 —— 呼叫順序一改就默默失效且不報錯。**冪等收斂比差異偵測穩**。發現「單項正常、批次無效」時優先懷疑這個。

### Claude Code computer-use 截圖會把「不在授權清單」的執行檔遮成空白
- **適用版本**：Claude Code computer-use MCP（`request_access` 回傳 `screenshotFiltering: "mask"` 時）
- **日期**：2026-08-31
- **環境**：Windows 11 + Claude Code computer-use；授權清單裡只有 `python.exe`
- **問題**：驗證 PyInstaller 打包出來的 exe，截圖裡視窗是一片空白深色矩形，一度判定「打包壞了 / UI 沒渲染」。同一份程式碼用 `python main.py` 跑卻完全正常。
- **原因**：`request_access` 是**依執行檔**授權的。dev 模式的宿主是 `python.exe`（已授權），打包後變成 `skill_tracker.exe`（不在清單）→ 截圖把它的內容遮掉。程式本身沒問題。
- **解法**：用 `user32.PrintWindow(hwnd, memdc, 2)`（`PW_RENDERFULLCONTENT`）自己抓視窗內容繞過遮罩，與 z-order / 是否最小化無關 —— 本專案是 Qt/GDI 視窗所以抓得到（Chromium / Electron 系被遮擋時會回空白，見上面〈PrintWindow 對被遮擋的 GPU/Chromium 視窗回空白〉）。或把打包後的 exe 一併加進 `request_access`。
- **重點**：**空白畫面先問「是不是被遮罩」，再問「是不是壞了」**；判斷依據是換一個宿主執行檔跑同一份程式，行為不同就是授權問題。另外別從外部程序戳 `ShowWindow` / `SetForegroundWindow` 想「救」視窗 —— Windows 多半會拒絕跨程序搶前景，只會把 `WS_MINIMIZE` / rect 弄成矛盾狀態，更難判讀。

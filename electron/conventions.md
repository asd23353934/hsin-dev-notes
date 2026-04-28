# Electron 慣例

> 「我們 / 我個人」對 Electron 專案的決議，不抄官方文件。

---

## 進程切分

**規則**：main / preload / renderer 三進程嚴格分離，重運算（DB 寫入、聚合、AI 呼叫）走 utility process。
**範例**：main 抓 hook + 視窗，utility 寫 SQLite + rollup，renderer 純 UI。
**理由**：事件量大時 main + DB 擠一起會卡 UI；Electron 35+ 起 utility process 是官方標配，比 child_process 乾淨。

## webPreferences 預設

**規則**：
```ts
{
  contextIsolation: true,
  nodeIntegration: false,
  sandbox: true,
}
```
不可關。若 preload 用了需要 require() 的非 electron 套件（如 electron-trpc），把該套件從 vite externalize 排除、bundle 進 preload，**不要**為了讓套件能用而開 `sandbox: false`。

**範例**（electron.vite.config.ts）：
```ts
preload: {
  plugins: [externalizeDepsPlugin({ exclude: ['electron-trpc'] })],
}
```

**理由**：sandbox: true 是縱深防禦關鍵一層。CVE 多數靠 sandbox 擋下來。bundle 一個套件多幾 KB 可接受。

## 視窗導航 / 外部連結

**規則**：
- `setWindowOpenHandler` 必過 scheme allowlist（`https / http / mailto`），其他全 deny
- `app.on('web-contents-created')` 內掛 `will-navigate`，只允許 `file://` 與同源 dev URL
- `will-attach-webview` 一律 deny

**理由**：renderer 任何 XSS / 惡意連結最後一道擋，避免拉到 `file:///etc/passwd` / `javascript:` / `smb://`。

## CSP

**規則**：renderer `index.html` 一定有 meta CSP；script 不開 `unsafe-inline`，style 為了 Tailwind 必須開。
**範例**：
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self' data:; connect-src 'self'; object-src 'none'; base-uri 'none'; frame-src 'none'; form-action 'none'" />
```
**理由**：meta CSP 在 inline script 已執行後才生效，所以還會在 v1 release 前用 `session.defaultSession.webRequest.onHeadersReceived` 補一份 HTTP header CSP（雙重防線）。

## 原生 hook 隱私 wrapper

**規則**：任何 native hook（鍵鼠 / 螢幕 / 麥克風）一定包一層 wrapper 模組，**該模組是專案內唯一**可 import 該套件的位置。Wrapper 對外暴露最小資料（次數 / 相對位移），不 leak raw event。
**範例**：`src/main/hooks.ts` 包 uiohook-napi，鍵盤對外只給 `keycode`，**永不**傳 `keychar`。
**理由**：靠人盯 PR 不夠；架構強制比註解強制可靠。一旦有人想加 keylogger 必須改 wrapper 簽章，瞞不住。

## IPC

**規則**：renderer ↔ main 走 electron-trpc + tRPC（端到端型別）；utility ↔ main 走 MessagePort（避免吃 IPC 瓶頸）。
**理由**：tRPC 給型別、TanStack Query 給快取與訂閱。MessagePort 比 ipcMain 快，又能傳 Transferable（如 ArrayBuffer）。

## 單一實例鎖

**規則**：所有 desktop app 都掛 `requestSingleInstanceLock()` + `second-instance` 聚焦原視窗。
**範例**：
```ts
if (!app.requestSingleInstanceLock()) {
  app.quit();
} else {
  app.on('second-instance', () => focusMainWindow());
  // ...
}
```
**理由**：使用者 double-click 啟動圖示是常態。不擋會出現兩個 hook tracker 雙倍計數。

## tracker / 長期執行模組

**規則**：`tracker.stop()` 同時掛 `before-quit` 與 `will-quit`（雙保險）。`start()` 內必 try/catch hook 載入；macOS 權限拒絕要降級為「app 仍可開但不追蹤」，不要 throw 到 `whenReady` 把整個 app 打死。
**理由**：強制終止 / 系統登出時 `before-quit` 不一定觸發；`will-quit` 是最後機會。macOS Accessibility 第一次未授權是常見路徑，不能讓 hook 失敗等於 app 啞。

## 測試

**規則**：E2E 走 Playwright `_electron.launch()`（Spectron 已停止維護）。Unit test 在 vitest setup 用 `vi.mock('uiohook-napi')` 隔離 native binding。
**理由**：Playwright 是現役 Electron 官方推薦。native module 不能在 unit test 真的載入（會嘗試 OS hook）。

## utility process IPC + RPC

**規則**：
- main ↔ utility 走 `utilityProcess.fork()` + `process.parentPort` 訊息通道，**不另開 MessagePort**（utility 本身的 channel 已夠）
- 自寫 RPC：`requestId`（`crypto.randomUUID()`）+ `Map<id, { resolve, reject, timeout }>` + RPC timeout（5 秒起步）
- 抽 `src/shared/<protocol>.ts` 做 message types，main / utility 雙方 `import type`，不跨進程 import 邏輯
- ready handshake：utility 啟動末尾 `parentPort.postMessage({ type: 'ready' })`，main 在 `startStorage()` 內 `await` 該訊息，沒 ready 不繼續
- **crash 恢復必做**：post-ready 加 `child.on('exit')` listener，標記 `alive = false`，drain 全部 pending RPC（reject）。對外 API 進入 dead 狀態：fire-and-forget 操作（`flushBucket`）靜默丟棄，query 操作立即 reject

**範例**（key-trace `src/main/storage.ts`）：
```ts
let alive = false;
await new Promise<void>((resolve, reject) => {
  const onReady = (msg) => { if (msg.type === 'ready') { child.off('message', onReady); alive = true; resolve(); } };
  child.on('message', onReady);
  child.once('exit', (code) => { if (!alive) reject(new Error(`utility exited before ready (code=${code})`)); });
});
child.on('exit', (code) => {
  alive = false;
  for (const p of pending.values()) { clearTimeout(p.timeout); p.reject(new Error(`utility exited (code=${code})`)); }
  pending.clear();
});
function flushBucket(payload) { if (!alive) return; child.postMessage({ type: 'bucket', payload }); }
function queryX() { if (!alive) return Promise.reject(new Error('utility is dead')); /* ... */ }
```

**理由**：沒 crash 恢復路徑，utility 一掛掉 main 端 pending RPC 全部等 timeout 才 reject、`flushBucket` 對死 child 呼 `postMessage` 直接 throw 噴 unhandled exception、整個 60s flush tick 壞掉。寫 ready handshake 是為了避免「utility 還沒開好 DB 就吃到請求」。

## utility entry build 結構

**規則**：utility entry 跟 main entry 同放 `electron-vite.config.ts` 的 `main.build.rollupOptions.input` 多 entry，輸出同目錄（`out/main/`）。`storage.ts` 用 `join(__dirname, 'utility.js')` 找 sibling 才會在 dev / prod 都對。

**範例**：
```ts
main: {
  plugins: [externalizeDepsPlugin()],
  build: { rollupOptions: { input: {
    index: resolve(__dirname, 'src/main/index.ts'),
    utility: resolve(__dirname, 'src/utility/index.ts'),
  }}},
},
```

**理由**：Electron utility process 與 main 同個進程模型（差別只在 `BrowserWindow` 持有），共享 `externalizeDepsPlugin` 設定最直白。tsconfig.node.json 的 `include` 也要加 `src/utility/**` + `src/shared/**`。

## better-sqlite3 慣例

**規則**：
- DDL（CREATE TABLE / CREATE INDEX）走 `db.exec(...)` 跑常數 SQL
- DML 一律 prepared statement + named（`@field`）或位置（`?`）綁定參數，**永不字串拼接**
- 多 row 寫入 wrap `db.transaction(...)`，回傳一個函式，呼叫時自動開 transaction
- DB 一打開立即 `db.pragma('journal_mode = WAL')` + `db.pragma('foreign_keys = ON')`
- query 結果**不直接 cast 成 TypedRow**，抽一層 reader 函式顯式 normalize（`Number(row[k] ?? 0)`），避免 schema migration 後 row 形狀不對讓上游拿到 `NaN` / `null`

**範例**：
```ts
const totalSinceStmt = db.prepare(`SELECT COALESCE(SUM(x), 0) AS x FROM events WHERE minute_ts >= ?`);
function readTodayTotals(): TodayTotals {
  const row = totalSinceStmt.get(startOfTodayMinuteTs()) as Record<string, unknown>;
  return { x: Number(row['x'] ?? 0) };
}
```

**理由**：綁定參數能擋 SQL injection（即使資料來自 OS API 如行程名稱）；transaction 對 better-sqlite3 開銷僅微秒級；顯式 normalize 是運行期防線，比 `as TypedRow` 多一道保險。

## 散布

**規則**：v1 期間只跑 `electron-vite dev` 自用，不打包。核心功能完成後才接 electron-builder + electron-updater + GitHub Releases。
**理由**：早期換 native module / Electron 大版本頻繁，提早做 release pipeline 是浪費；簽章證書（Apple Developer / Windows EV）要錢，等真的散布再投入。

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

## 散布

**規則**：v1 期間只跑 `electron-vite dev` 自用，不打包。核心功能完成後才接 electron-builder + electron-updater + GitHub Releases。
**理由**：早期換 native module / Electron 大版本頻繁，提早做 release pipeline 是浪費；簽章證書（Apple Developer / Windows EV）要錢，等真的散布再投入。

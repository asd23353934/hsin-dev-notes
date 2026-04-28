# Electron 專屬 stack

> 「新專案起手套餐」。實際專案以該專案 `package.json` 為準。
> 所有版本均經 `npm view` 驗證（最後驗證：2026-04-28）。

---

## 核心

- **electron**：41.3.0
- **electron-vite**：5.0.0（社群 scaffold；vite 7 / 8 都支援，目前 peer 卡 vite 7 為止）
- **vite**：7.3.2（卡 electron-vite 5 peer，vite 8 出了但 electron-vite 還沒升）
- **@vitejs/plugin-react**：5.2.0（v6 起 vite peer 收緊到 ^8，這版仍涵蓋 4~8）
- **typescript**：6.0.3（latest stable）
- **@types/node**：25.6.0

## React

- **react / react-dom**：19.2.5
- **@types/react**：19.2.14
- **@types/react-dom**：19.2.3

## UI

- **tailwindcss**：4.2.4
- **@tailwindcss/vite**：4.2.4
- shadcn/ui：copy-paste，無單一版本號（依各 component 引入時生）
- Tremor / Recharts：dashboard / 圖表（需要時引入，未鎖版）

## 狀態 / 資料

- **@tanstack/react-query**：4.44.0（**鎖在 v4**，因 electron-trpc 0.7.1 仍用 tRPC v10 結構，配套必須 v4）
- **zustand**：5.0.12（renderer 全域）
- **electron-store**：未鎖（需要時引入）

## IPC

- **electron-trpc**：0.7.1
- **@trpc/server / @trpc/client / @trpc/react-query**：10.45.4（**鎖在 v10**，理由同上；待 electron-trpc 升 v11 後一起升）

## 原生 hook

- **uiohook-napi**：1.5.5（Win / macOS / Linux 全平台 prebuilt，Node 24 支援；macOS 需 Accessibility 授權）
- **get-windows**：9.3.0（前身 active-win）

## 資料庫

- **better-sqlite3**：12.9.0（同步 API、極快；Electron 需走 prebuilt 或 electron-rebuild）

## 測試

- **vitest**：依 dev-notes 統一決議用 latest（Angular 21+ 也統一 vitest）
- **playwright**：1.59.1（Electron 測試走 `_electron.launch()`，**Spectron 已停止維護不要用**）

## 打包 / 更新

- **electron-builder**：26.8.1
- electron-updater：搭 electron-builder + GitHub Releases

---

## Peer 相容性備忘

- **electron-trpc 0.7.1 vs tRPC v11**：不相容（procedure 內部結構在 v11 變了；electron-trpc 仍用 v10 lookup）。tRPC 整套必須鎖 10.45.x，TanStack Query 鎖 4.x。upstream 升 v11 後一起放
- **uiohook-napi**：N-API 預編譯，升 Electron 大版本（換 Node ABI）時要 rebuild 或等新 prebuilt
- **vite 8** 出了但 electron-vite 5 peer 卡 vite 7，且 `@vitejs/plugin-react@6` 需 vite 8。打組合拳：vite 7.3.2 + plugin-react 5.2.0

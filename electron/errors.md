# Electron errors

> 踩過的坑與解法。**「適用版本」必填**（跨多版本專案使用）。

---

### electron-trpc 0.7.1 / 1.0.0-alpha.0 不相容 tRPC v11

- **適用版本**：electron-trpc 0.7.x ~ 1.0.0-alpha.0 / @trpc/* 11.x
- **日期**：2026-04-28
- **環境**：electron 41.3.0、electron-trpc 0.7.1、@trpc/server / client / react-query 11.16.0、@tanstack/react-query 5.100.5、Node 24.11.1
- **問題**：renderer 的 `trpc.X.useQuery` 永遠卡在 isLoading；`window.electronTRPC` 已正常暴露、`sendMessage` / `onMessage` 都在；直接 `electronTRPC.sendMessage({ method: 'request', operation: { id, type: 'query', path, input } })` 也收不到任何回應。Console 無錯誤訊息。
- **原因**：electron-trpc 的內部 procedure lookup 仍寫死 v10 結構：
  ```js
  procedures[path]._def[type]  // type === 'query' | 'mutation' | 'subscription'
  ```
  在 tRPC v10，procedure 的 `_def` 上有 `query` / `mutation` / `subscription` 這些 boolean 欄位；v11 改成 `_def.type === 'query'`，原本的 lookup 永遠 falsy → 走進 `NOT_FOUND` 但 throw 後沒回 reply（誰呼叫誰自死），因此 renderer 永遠等不到回應、也沒錯誤可看。檢查 `electron-trpc/dist/main.cjs` 與 1.0.0-alpha.0 都是同樣的 `procedures[e]._def[r]` 寫法，**整個套件還沒升 v11**。
- **解法**：tRPC 整套鎖在 v10：
  ```json
  {
    "@trpc/client": "10.45.4",
    "@trpc/react-query": "10.45.4",
    "@trpc/server": "10.45.4",
    "@tanstack/react-query": "4.44.0"
  }
  ```
  注意 `@trpc/react-query@10` peer 是 `@tanstack/react-query@^4`，連帶要降 v4。React 19 + tanstack v4 OK。
  TanStack Query v4 vs v5 API 差：`isPending` 是 v5 名字，v4 用 `isLoading`。
- **參考**：electron-trpc 1.0.0-alpha 與 0.7.1 同日發布（2024-12-07）但 source 同樣未升 v11。追 upstream：[electron-trpc GitHub](https://github.com/jsonnull/electron-trpc)。

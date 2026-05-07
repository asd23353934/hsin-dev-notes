# Next.js 踩坑紀錄

> Next.js / Prisma / NextAuth / Vercel 等相關問題與解法。
> 遇到錯誤先查這裡有沒有紀錄；解完新坑請補上。
> 最後更新：2026-05-07

---

## 紀錄模板

```markdown
### 問題標題
- **適用版本**：Next.js 14-15 / Prisma 5（**必填**，標明此問題存在/解法成立的版本範圍；上游已修復則註記「v16 起已修復」）
- **日期**：YYYY-MM-DD
- **環境**：實際當下踩到時的精確版本（例：Next.js 15.5.14 / Prisma 5.22.0 / Node 20.11）
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

### `next dev` 經 nginx 反代時整頁不規律刷新（HMR WebSocket upgrade 失敗）

- **適用版本**：Next.js 13-16（任何走 webpack/turbopack HMR 的版本）/ nginx 全版本
- **日期**：2026-05-05
- **環境**：Next.js 16.1.6 / nginx alpine（生產 Docker on GCP e2-micro）/ HTTPS 域名（Let's Encrypt）
- **問題**：雲端跑 `next dev` 時前端會不規律「閃一下」整頁刷新，編輯表單到一半被打斷要重 key。地端 `localhost` 直連不會發生。曾誤判為背景 polling，連續砍 `SessionProvider refetchInterval` / `Heartbeat` 元件 / dashboard `setInterval` / board polling 等四五處 client-side 自動 refetch，仍未停止。
- **原因**：nginx config 把 `proxy_set_header Connection "upgrade"` 寫死、非 conditional。一般 HTTP request 也被強制帶 Upgrade header，HMR 的 `wss://.../_next/webpack-hmr` upgrade 在某些情境失敗 → HMR client 連不上 → fallback 到 polling 偵測 dev server 變動 → 偵測到變動就觸發 **整頁 full reload**（非 hot update）。dev 模式下任何 commit / 存檔造成 dev server 重編譯，所有開著頁面的人都被踹刷新。
- **解法**：nginx config 改 conditional Connection header：

  ```nginx
  # http {} 區塊（或單獨 conf.d 檔）加：
  map $http_upgrade $connection_upgrade {
      default upgrade;
      ''      close;
  }

  # server { location / } 內把寫死改用 map 變數：
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection $connection_upgrade;  # ← 不再是 "upgrade" 字面
  ```

  過渡期解法：把雲端從 `next dev` 切到 `next build && next start`（production build 完全無 HMR / WS），等 nginx config 落地後再切回。

  **診斷捷徑**：browser DevTools Network 面板過濾 `webpack-hmr` 看 wss 是否 101 Switching Protocols 成功。若連不上才是這個 bug；先別急著砍 client polling。
- **參考**：[nginx WebSocket proxying 官方指南](https://nginx.org/en/docs/http/websocket.html)；em 0505 開發日誌

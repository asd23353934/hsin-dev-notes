# Godot Plugin 採用指南

> 完整的 plugin 評估與決策依據。`stack.md` 是事實清單（版本），這份是**有意見的採用建議**。
> 適用 Godot 4.6.3 / 個人 indie 開發 / 6 個月 portfolio 目標。
> 最後更新：2026-05-23

---

## 0. 採用 plugin 的決策框架

裝 plugin 前先問 5 個問題：

1. **這個 plugin 取代的東西，我自己寫要多久？**
   - 超過 1 週 → 用 plugin
   - 不到 1 天 → 自寫（學習價值高）
2. **這個功能會不會被 Godot 內建取代？**（看 [roadmap](https://godotengine.org/article/) / proposal）
3. **plugin 維護活躍度？**
   - 看 GitHub last commit / last release / open issue 數
   - 半年以上沒動 = 警戒
4. **跟我目標 Godot 版本相容嗎？**
   - 不能信「Godot 4」標籤，要看 minor 版本（4.5 / 4.6）
5. **會不會綁架我的 code 架構？**
   - 深度耦合 = 之後換 plugin 痛苦
   - 評估方式：plugin API 出現在我多少個檔案

---

## 1. 強烈推薦（**會在你的學習路徑用到**）

### ⭐⭐⭐⭐⭐ GdUnit4 — 單元測試框架

- **GitHub**：https://github.com/godot-gdunit-labs/gdUnit4
- **最新版**：v6.1.x (stable) / v6.2 (master)
- **Godot 相容**：4.5–4.6.x, 4.7-beta1 ✅
- **何時裝**：**W4 起**（提前自原 W11 計畫）
- **使用情境**：
  - GameState 邏輯測試（card-resource-demo 可立刻 retrofit）
  - 戰鬥公式測試（damage / shield / buff 疊加）
  - 卡牌效果邊界 case（cost 不足、目標死了、status 互動）
- **為何裝**：
  1. **面試直接加分**：多數 junior 不寫測試，「我有寫」= 差異化
  2. **DFS 規模需要**：80 張卡 + 多 status，沒測試 regression 災難
  3. **學習曲線淺**：基本用法 1 小時，半天能寫 10 個測試
- **替代**：[GUT](https://github.com/bitwes/Gut)（較舊，feature 較少；不選）

### ⭐⭐⭐⭐⭐ Dialogic 2 — VN 對話系統

- **GitHub**：https://github.com/dialogic-godot/dialogic
- **最新版**：有 auto-updater
- **Godot 相容**：4.4+ → **4.6 OK** ✅
- **何時裝**：**W16**（DFS Hub + VN）
- **使用情境**：
  - 角色立繪 + 對話 + 分支選項
  - Aria 角色路線
  - 棋盤事件對話樹
- **為何裝**：
  1. **WorkNite 是 VN/H game studio，這個直接 match 履歷**
  2. 自寫 dialog tree 系統至少 1-2 週；Dialogic 半天上手
  3. 視覺化編輯器，劇情寫作不用碰 code
  4. 角色管理 / 變數插入 / 條件顯示 全內建
- **替代**：[Dialogue Manager](https://nathanhoad.itch.io/godot-dialogue-manager)（後面說）

### ⭐⭐⭐⭐ GDCubism — Live2D 模型

- **GitHub**：https://github.com/MizunagiKB/gd_cubism
- **最新版**：v0.9
- **Godot 相容**：4.3+ ✅
- **何時裝**：**W25**（Live2D demo，已在學習路徑）
- **使用情境**：
  - 30 秒 Live2D demo（履歷亮點）
  - 跟 WorkNite 對齊（VN 常用 Live2D）
- **為何裝**：
  1. Live2D 在 Godot **沒有其他 plugin**
  2. v0.9 改直接渲染，4096×4096 模型記憶體從 2GB → 230MB
  3. 開新 GitHub repo（小、獨立、有 demo）= portfolio 第 4 個 repo

---

## 2. 條件性推薦（**到時看狀況決定**）

### ⭐⭐⭐ LimboAI — Behavior Tree + State Machine

- **GitHub**：https://github.com/limbonaut/limboai
- **最新版**：v1.6.0
- **Godot 相容**：**必須 v1.6.0+ 才支援 4.6**（舊版會壞，重要！）
- **何時考慮**：**M3 之後**（W11 寫敵人 AI 時）
- **裝的理由**：
  - 複雜 AI 視覺化編輯比寫 code 直觀
  - 視覺 debugger 看 AI 跑哪個分支
  - 大型遊戲標配（mid-level 加分）
- **不裝的理由**：
  - DFS MVP 敵人 AI 簡化（W8 計畫只做「random / scripted attack」），自寫 state machine 50 行搞定
  - 多 1 個 plugin 多 1 份學習成本
- **決策點**：W11 寫敵人 AI 時，**超過 3 個狀態以上才裝**

### ⭐⭐⭐ Phantom Camera — Cinemachine 風格攝影機

- **GitHub**：https://github.com/ramokz/phantom-camera
- **最新版**：latest
- **Godot 相容**：4.3+ ✅
- **何時考慮**：**W22 polish 階段（選配）**
- **裝的理由**：
  - 攝影機過渡 / shake / follow 提升表現力
  - Boss 戰預演 cutscene
- **不裝的理由**：
  - DFS 戰鬥畫面是**固定鏡頭**（卡牌遊戲），攝影機需求低
  - 自寫 Tween + Camera2D 寫 5 種 shake / pan 也只要 50-100 行
- **決策點**：W22 想要 boss 演出有運鏡感才裝

---

## 3. 不推薦 / 不裝（**詳細理由**）

### ❌ Godot Jolt（外掛版） — **已被內建取代**

- **為何不裝**：
  - Jolt 物理引擎已**內建到 Godot 4.4+**，4.6 變**預設 3D 物理引擎**
  - 外掛版正在棄用，repo 即將 archive
  - **裝外掛版會跟內建衝突**
- **正確做法**：
  ```
  Project Settings → Physics → 3D → Physics Engine → 選 JoltPhysics3D
  ```
- **註**：4.6 預設已是 Jolt，可能不用手動設

### ❌ GUT (Godot Unit Test)

- **為何不裝**：**GdUnit4 是現代繼任者**，feature 多很多
- GUT 仍有人用但社群動能轉到 GdUnit4
- **選 GdUnit4 一個就好**

### ❌ Dialogue Manager（Nathan Hoad）— **對你不如 Dialogic**

- **為何對你不裝**：
  - Dialogic 對 DFS 需求更全面（角色管理 + 視覺編輯器 + 分支系統）
  - Dialogue Manager 是輕量替代，純文字 syntax driven
- **不是「壞」plugin**，是「Dialogic 對你更合」
- **選一個就好，二選一選 Dialogic**

### ❌ Maaack's Game Template / 任何 Game Boilerplate

- **為何不裝**：
  - 把 menu / save / settings 都套件化，**你會學不到底層**
  - 對 junior 的學習價值：**自己寫主選單 / save 比用 template 重要**
- **portfolio 觀感**：「我用 template 套出來」< 「我自己寫的」
- **例外**：M6 收尾**真的趕時間** 才考慮 fallback

### ❌ godot-saver / Save System plugins

- **為何不裝**：
  - 你 W17 計畫**自己寫 save 系統**（XOR 加密 binary）
  - 學 save 邏輯本身比用 plugin 黑盒值得
  - DFS 存檔需求 specific（玩家狀態 + 解鎖紀錄 + 卡片庫存），plugin 反而要 fit 進去
- **例外**：M6 真的卡到太久可考慮

### ❌ GodotSteam — **scope 外**

- **為何現在不裝**：
  - 你目標 **itch.io**，不是 Steam
  - 6 月 MVP scope 沒包 Steam 上架
  - Steam Direct fee $100，上架要評估
- **例外**：M6 完成 itch.io 後想嘗試 Steam 才考慮

### ❌ Procedural Generation plugins（WFC / TileMap generators）

- **為何不裝**：
  - DFS 棋盤是**固定 closed loop**，沒有大量隨機生成需求
  - 卡牌池 random 用 GDScript 內建 `randi()` 已夠
- **這類 plugin 適合**：roguelike dungeon generation、地圖無限生成的遊戲

### ❌ godot-rust / GDExtension via Rust

- **為何不裝**：
  - 學 Rust 是**另一個技能樹**，會分散注意
  - GDScript 對你的範圍效能夠用
  - DFS **不會碰到 GDScript 效能瓶頸**
- **例外**：M2+ 真的需要 native 效能時再說（遙遠）

### ❌ Visual Scripting / Block Coding plugins

- **為何不裝**：
  - 你是工程師背景，**code-first 更快**
  - Visual Scripting 適合非程式背景的設計師
  - Godot 4.0 已把 Visual Script **從核心拿掉**，現在 VS 是社群 plugin
- 沒理由倒退

### ❌ HTerminal / In-game console plugins

- **為何不裝**：
  - debug 用 `print()` + Godot Debugger panel 已夠
  - In-game console 是**大型 mod-friendly 遊戲**用的（如 Skyrim 風格）
  - DFS 規模不需要

### ❌ Plugin Refresher

- **為何不裝**：
  - 你**不開發 plugin**，是 plugin **user**
  - 熱重載 plugin 是 plugin 作者才需要的工具

### ❌ godot-fmod / Wwise（商業音效 middleware）

- **為何不裝**：
  - **商業 middleware，要授權費**
  - AudioStreamPlayer 對 indie 完全夠
- **例外**：未來進專業音效 designer 工作流才考慮

### ❌ 預先學 Shader plugins / GDQuest shader 庫

- **為何不裝**：
  - W22 polish 用內建 shader 寫 hit pause / screen shake / 飄字就好
  - 預先學 shader 庫太重，學了不一定用到
- **替代**：W22 真的要某個特效時，**個別找 shader example** 抄

### ❌ Tile-Bit-Tools / TileMap helpers

- **為何不裝**：
  - DFS 棋盤**不是 tilemap based**（closed loop 用 Node2D + Marker2D）
  - 適合 platformer / metroidvania，**對 deckbuilder 沒用**

### ❌ Beehave（LimboAI 替代）

- **為何對你不選**：
  - LimboAI 社群動能更強、文件更完整、更新更頻繁
- **vs LimboAI 二選一選 LimboAI**

### ❌ Editor enhancement plugins（Inspector Plus / Stylebox Editor 等）

- **為何不裝**：
  - 增加 editor 複雜度
  - 你目前 Godot 編輯器流程已順
- **等真的卡到某個編輯流程再個別找**

---

## 4. 採用順序時間軸

```
現在（W3 結束 / 2026-05-23）：
  ✓ GdUnit4 retrofit → card-resource-demo（可選，但強推）

W4-W11：
  ✓ GdUnit4 隨 W4 deckbuilder 同步引入
    寫 5-10 個測試（卡牌費用 / damage / Combo Lock）

W11（戰鬥成熟階段）：
  ? LimboAI 評估點：敵人 AI 複雜度超過 3 狀態才裝
  ✓ GdUnit4 補完 combat-formulas 測試

W16（Hub + VN）：
  + Dialogic 2 引入
    Aria 路線對話 / 棋盤事件分支

W22（polish）：
  ? Phantom Camera 評估點：boss 演出想要運鏡感才裝

W25（Live2D demo）：
  + GDCubism 引入
    開新 repo, push demo

M6 收尾（W23-W26）：
  Reconsider all：看哪些 plugin 實際用上、哪些沒用
  考慮移除沒用到的 plugin 降低 portfolio 噪音
```

---

## 5. 反面教材（**plugin 採用的雷區**）

要避免：

### plugin 收集癖
- 裝 10 個 plugin 但只用 2 個
- code 變很重、啟動變慢、版本相依糾結

### 被 plugin 綁架架構
- 寫 code 圍著 plugin API 轉
- 換 plugin 時痛苦 unwind
- **預防**：plugin 用在邊緣（如 dialog system 是獨立模組），不要讓核心邏輯（戰鬥）依賴第三方

### 預先學沒用到的
- W1 就學 Phantom Camera 完全浪費時間
- **原則**：到該週 / 該需求才學

### 盲目升級
- plugin 升版可能 break
- **預防**：commit 鎖住 plugin 版本（`addons/` 進 git，不要 .gitignore）
- 升版前看 changelog

### 信錯版本
- 看「Godot 4 相容」就裝，結果掛在 4.6
- **預防**：到 GitHub README 看具體 minor 版本支援表
- 不確定就先用 demo project 試

---

## 6. 引入 plugin 的標準流程

```
1. 評估（用上面 5 個問題）
2. 確認版本相容（查 GitHub README 的「Compatibility」section）
3. AssetLib 安裝 / GitHub release zip 解到 addons/
4. Project Settings → Plugins → 啟用
5. 重啟 Godot
6. 跑 plugin 提供的 demo 確認沒壞
7. commit `addons/` 到 git（鎖版本）
8. 寫第一個 minimal 範例驗證自己會用
```

---

## 7. 對你目前學習路徑的具體建議

**最划算的 3 個 action**：

1. **現在 retrofit GdUnit4 到 card-resource-demo**
   - 1-1.5 小時
   - 立刻強化 GitHub portfolio
   - 練熟 GdUnit4 給 W4 用

2. **W4 deckbuilder 從一開始就帶 GdUnit4**
   - 邊寫邊測，TDD 風格
   - 戰鬥邏輯先有測試保護

3. **W16 安裝 Dialogic 2 寫 Hub 對話**
   - WorkNite 履歷加分項
   - 不要自寫 dialog tree（時間黑洞）

其他 plugin **到該週才評估**，不要預先學。

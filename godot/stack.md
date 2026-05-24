# Godot Stack

> 事實型紀錄：引擎版本、執行環境、推薦工具。
> Claude Code 在產生 Godot 程式碼前必須先看這份，依版本給對應 API。
> **角色**：新專案起手套餐 / 預設假設。實際專案內以該專案 `project.godot` 的 config_version 與引擎版本為準。
> 最後更新：2026-05-23

---

## 執行環境

| 項目 | 版本 | 備註 |
|------|------|------|
| Godot Engine | **4.6.3 stable** | **Standard 版**（非 .NET / Mono）。GDScript 內建 |
| OS | Windows 11 | 主開發環境 |
| 算繪器 | **Forward+** | 2D / 3D 通吃；2D 專案也用這個 |
| 版本控制中繼資料 | Git | 建專案時勾選，自動生 `.gitignore` + `.gitattributes` |

> **不要用 .NET 版**：吃硬碟大、啟動慢、C# 對社群教學支援差。除非專案明確要求 C#，一律 Standard。

---

## 編輯器 / IDE

| 項目 | 版本 | 備註 |
|------|------|------|
| Godot 內建編輯器 | 隨引擎 | 場景樹編輯、Inspector、debugger 都在這 |
| VS Code | latest | 寫 GDScript 主力 |
| godot-tools (VSCode 擴充) | latest | **發布者必須是 geequlim**（官方推薦，不要裝其他人 fork 的） |

### Godot ↔ VSCode 互通設定

Godot：**編輯器 → 編輯器設定 → Text Editor / External**

- 勾「**使用外部編輯器**」
- 執行檔路徑：`C:\Users\yuhsinhe\AppData\Local\Programs\Microsoft VS Code\Code.exe`
- 執行檔參數：`{project} --goto {file}:{line}:{col}`

之後在 Godot 雙擊 `.gd` 會跳 VSCode。

---

## 語言 / 範本

| 項目 | 選擇 | 備註 |
|------|------|------|
| 腳本語言 | **GDScript** | 內建、Python-like、官方教學 90% 都用 |
| C# | 不用 | 除非專案要求 |
| Type hints | **強制** | `var x: int = 5`、`func() -> void`，不寫無型別 |
| 縮排 | **Tab**（不是 4 空格） | Godot 預設 |

---

## Plugin 清單（依優先度排序）

> 已驗證 Godot 4.6.3 相容性。版本確認日 2026-05-23。
> Godot 升版時必須重查每個 plugin 是否仍支援，**不要盲目升級**。

### 強烈推薦（短期就引入）

| 套件 | 最新版 | 4.6.3 相容 | 何時引入 | 用途 |
|------|------|------|------|------|
| [**GdUnit4**](https://github.com/godot-gdunit-labs/gdUnit4) | v6.1.x stable / v6.2 master | ✅ 完全支援 4.5–4.6.x + 4.7-beta1 | **W4 起**（提前自 W11） | GDScript / C# 單元測試框架；測試 driven dev、scene 測試、assertions、mocking。**面試展示測試文化神器** |
| [**Dialogic 2**](https://github.com/dialogic-godot/dialogic) | 含 auto-updater | ✅ 需 Godot 4.4+ | W16（Hub + VN） | VN 對話 / 角色管理 / 分支劇情。**WorkNite 直接相關**。AssetLib 直接搜「Dialogic 2」 |

### 中期會用

| 套件 | 最新版 | 4.6.3 相容 | 何時引入 | 用途 |
|------|------|------|------|------|
| [**GDCubism**](https://github.com/MizunagiKB/gd_cubism) | v0.9 | ✅ 需 Godot 4.3+ | W25（Live2D demo） | Live2D 模型整合。v0.9 改直接渲染，4096×4096 模型記憶體從 2GB 降到 230MB |
| [**LimboAI**](https://github.com/limbonaut/limboai) | v1.6.0 | ✅ **必須裝 1.6+ 才支援 4.6**，舊版會壞 | M3+（敵人 AI） | Behavior tree + 狀態機 + 視覺 debugger。mid-level 加分點 |

### 可能用

| 套件 | 最新版 | 4.6.3 相容 | 用途 |
|------|------|------|------|
| [Phantom Camera](https://github.com/ramokz/phantom-camera) | latest | ✅ 需 Godot 4.3+ | Cinemachine 風格攝影機 / cutscene。DFS Boss 演出可用 |
| [Dialogue Manager](https://nathanhoad.itch.io/godot-dialogue-manager) | v3 | ✅ | Dialogic 替代方案，更輕量；選一個用就好 |

### ⚠️ 不要裝（已內建到 Godot 4.6）

| 套件 | 狀態 |
|---|---|
| **Godot Jolt**（外掛版） | **4.4 起內建**，**4.6 變預設 3D 物理引擎**。外掛版在棄用中、會跟內建衝突。**不要再裝外掛版**，直接用內建。<br>內建用法：`Project Settings → Physics → 3D → Physics Engine` 選 `JoltPhysics3D` |

### 引入流程（標準）

1. **先確認版本**：到 plugin GitHub README 看「Compatibility」段，比對你的 Godot 版本
2. **AssetLib 安裝**：`Project → AssetLib → 搜尋 → 下載 → 安裝`
   或 GitHub release zip 手動解壓到 `addons/plugin_name/`
3. **啟用**：`Project Settings → Plugins → 找到該 plugin → 勾啟用`
4. **重啟 Godot**（多數 plugin 需要）
5. **驗證**：跑該 plugin 提供的 demo 確認沒壞

---

## 專案結構慣例

```
<project-name>/
├── project.godot         # Godot 專案設定（必有）
├── .gitignore            # 預設含 `.godot/` `/android/`
├── .gitattributes
├── icon.svg              # 預設 logo
├── README.md             # 一定要寫，至少 1 段描述 + 截圖
├── screenshot.png        # 履歷用截圖
│
├── art/                  # 圖片、音效素材
├── fonts/                # 字型
├── autoload/             # 全域 singleton（GameState、EventBus...）
├── cards/                # 卡牌 .tres 資料 + CardData.gd
├── scenes/               # 各功能場景（.tscn + .gd）
└── ui/                   # UI scene（HUD、menu 等）
```

> **art/ fonts/ 等資料夾名稱跟著 Godot 官方 tutorial 慣例**，社群理解度最高。

---

## 重要官方文件入口

| 主題 | URL |
|---|---|
| Getting Started | https://docs.godotengine.org/en/stable/getting_started/introduction/index.html |
| Your First 2D Game | https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html |
| Scripting (GDScript) | https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html |
| Custom Resource | https://docs.godotengine.org/en/stable/tutorials/scripting/resources.html |
| Autoload (Singleton) | https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html |
| Signals | https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html |
| Input handling | https://docs.godotengine.org/en/stable/tutorials/inputs/inputevent.html |
| Class reference | https://docs.godotengine.org/en/stable/classes/index.html |

---

## 對應作品

| Repo | W | 狀態 |
|------|---|------|
| [hello-godot](https://github.com/asd23353934/...)（local 還沒 push） | W1 | hello world + GDScript 100 行練習 |
| [dodge-the-creeps](https://github.com/asd23353934/dodge-the-creeps) | W2 | Godot 官方 2D tutorial 完整 |
| [card-resource-demo](https://github.com/asd23353934/card-resource-demo) | W3 | Custom Resource + Autoload demo |
| [dice-fate-survivor](https://github.com/asd23353934/dice-fate-survivor) | M2-M6 | 主要產出，private repo |
| [godot-learning-path](https://github.com/asd23353934/godot-learning-path) | - | 學習計畫 + PROGRESS + 面試 prep |

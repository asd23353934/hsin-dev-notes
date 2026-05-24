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

## 預期會用到的 plugin（W4 之後）

| 套件 | 用途 | 何時引入 |
|------|------|------|
| [GdUnit4](https://github.com/MikeSchulze/gdunit4) | GDScript 單元測試框架 | W11（戰鬥公式測試） |
| [GDCubism](https://github.com/MizunagiKB/gd_cubism) | Live2D 模型支援 | W25（Live2D demo） |
| [Dialogic](https://github.com/dialogic-godot/dialogic) | VN 對話系統（候選） | W16（Hub + VN） |

> 三個都還沒裝，到該週再評估是否引入。

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

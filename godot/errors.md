# Godot 踩坑紀錄

> Godot 4 / GDScript 相關問題與解法。
> 遇到錯誤先查這裡有沒有紀錄；解完新坑請補上。
> 最後更新：2026-05-23

---

## 紀錄模板

```markdown
### 問題標題
- **適用版本**：Godot 4.x（**必填**，標明版本範圍；上游已修復則註記「v4.x 起已修復」）
- **日期**：YYYY-MM-DD
- **環境**：實際當下踩到時的精確版本（例：Godot 4.6.3.stable）
- **問題**：症狀描述（看到什麼錯誤訊息、什麼行為不對）
- **原因**：根因分析
- **解法**：步驟或程式碼
- **參考**：issue / 文件 / Stack Overflow 連結（可選）
```

> **「適用版本」與「環境」差別**：
> - **適用版本** = 這個問題/解法在哪些版本範圍成立
> - **環境** = 你實際踩到時的精確版本

---

## 紀錄

### `_on_body_entered` 裡直接改 collision disabled 行為怪 / 偶爾 crash

- **適用版本**：Godot 4.x（所有版本）
- **日期**：2026-05-23
- **環境**：Godot 4.6.3.stable
- **問題**：玩家撞到敵人時想 disable collision shape，直接寫 `$CollisionShape2D.disabled = true` 偶爾遊戲行為異常
- **原因**：`_on_body_entered` 是物理引擎在 physics step 中間呼叫的 callback。此時 CollisionShape2D 還在被物理系統使用，直接 mutate 等於邊算邊改。
- **解法**：用 `set_deferred` 排到下一 idle frame
  ```gdscript
  $CollisionShape2D.set_deferred("disabled", true)
  ```
- **參考**：[Object.set_deferred 文件](https://docs.godotengine.org/en/stable/classes/class_object.html#class-object-method-set-deferred) 原話：「Modifying object properties directly during physics callbacks can cause issues with the simulation state」

---

### `:=` 型別推斷對 `$Node.property` 失效

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3，VSCode godot-tools 擴充
- **問題**：寫 `var x := $AnimatedSprite2D.sprite_frames.get_animation_names()`，VSCode 紅線報 `Cannot infer the type of "x" variable because the value doesn't have a set type`
- **原因**：`$AnimatedSprite2D` 在靜態分析時被視為 `Node`（基底），不知道有 `sprite_frames` 屬性，所以推不出 `get_animation_names()` 回傳型別。
- **解法**：明寫型別或改用 `=`
  ```gdscript
  # ✓ 明寫
  var x: Array = $AnimatedSprite2D.sprite_frames.get_animation_names()

  # ✓ 改 normal assignment
  var x = $AnimatedSprite2D.sprite_frames.get_animation_names()
  ```
- **長期解法**：把子 node 用 `@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D` 明確標型別，後續用 `sprite.sprite_frames...` 推斷就能 work。

---

### Scene root 有 transform 跳警告

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3
- **問題**：場景樹 root node 旁邊出現黃色警告三角形，hover 顯示「場景的根節點建議不要變形，因為場景的實例通常會覆蓋此變形」
- **原因**：scene 被 instance 後位置由 parent 決定，root 自己帶 transform 會被 override，造成混亂。
- **解法**：
  1. 選 root node → Inspector → Transform2D → 重設為預設（Position 0,0 / Scale 1,1 / Rotation 0）
  2. 視覺變形（如縮小 sprite）改設在子 node（如 AnimatedSprite2D 的 Scale）

---

### F5 跑遊戲 Output 空白沒 print

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3，多場景專案
- **問題**：寫好 script 按 F5，遊戲視窗開了但 Output 只有引擎啟動訊息，自己的 print 都沒出來
- **原因**：F5 跑「**主場景**」（Project Settings → Application → Run → Main Scene），不是當前編輯的場景。多場景專案常會看錯。
- **解法**：
  - 跑當前場景按 **F6**
  - 或專案設定改 Main Scene 為目前要跑的
- **延伸**：確認 Node 上有附加 script（場景樹上 node 旁有沒有腳本 icon），漏附加 _ready 不會跑。

---

### `class_name` 註冊後 VSCode 還是紅線

- **適用版本**：Godot 4.x + VSCode godot-tools
- **日期**：2026-05-23
- **環境**：Godot 4.6.3，VSCode godot-tools 擴充
- **問題**：寫 `card.gd` 開頭 `class_name Card`，另一個 `main.gd` 用 `Card.new()` 但 VSCode 標紅線
- **原因**：`class_name` 全域註冊由 **Godot 編輯器** 維護（存在 `.godot/global_script_class_cache.cfg`）。VSCode 從 cache 讀資料，剛建的 class 還沒被 Godot 掃描到。
- **解法**：
  1. 切到 Godot 編輯器視窗，點 FileSystem 任何地方 → Godot 重新掃描 + 更新 cache
  2. 回 VSCode，紅線應該消失
  3. 還在的話：VSCode `Ctrl+Shift+P` → "Godot Tools: Restart language server"
- **驗證**：F6 跑場景能跑 = 程式碼對的，紅線是 linter 假警報。

---

### Path2D 沒封閉 → mob 從不該出現的位置噴出

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3，dodge-the-creeps tutorial
- **問題**：在 Path2D 上沿畫面 4 個角畫矩形，但 mob spawn 位置怪怪的，有時候在畫面中間冒出
- **原因**：Path2D 繪製模式下要點**5 個點**（4 個角 + 最後回到起點），才會圍成封閉矩形。只點 4 個變開口形。
- **解法**：點完 4 個角再點一次起點，或在第 4 個點後按右鍵讓 Godot 自動封閉。
- **驗證**：選 Path2D 看 Curve 屬性，points 應該有 5 個 entry。

---

### `@export var mob_scene: PackedScene` 不拖檔 → `instantiate()` null error

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3
- **問題**：寫好 `@export var mob_scene: PackedScene` 跟 `mob_scene.instantiate()`，但 runtime 報 null
- **原因**：`@export` 只是宣告欄位給 Inspector，**沒拖檔進去就是 null**。
- **解法**：選 Main root → Inspector → 找 Mob Scene 欄位 → 從 FileSystem 拖 `mob.tscn` 進去
- **預防**：commit 前先 F5 跑一次確認所有 @export 都有指定。

---

### `ui_select` 不能新增到 input map

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3
- **問題**：想在 Project Settings → 輸入對應 新增 `ui_select`，跳「已存在」拒絕
- **原因**：`ui_*` 是 Godot 內建 action，預設綁好（ui_select = Space、ui_accept = Enter+Space+JoyA 等），**不需要也不能重新建立**。預設隱藏在 input map，要勾「**顯示內建動作**」才看得到。
- **解法**：
  - 直接在 code 用：`Input.is_action_pressed("ui_accept")` 即可
  - 要看 / 改預設綁定：Input Map 右上勾「顯示內建動作」
- **延伸**：自訂遊戲操作（如 `attack`、`move_left`）**另開名字**，不要跟 ui_* 衝突。

---

### AudioStreamPlayer 音樂沒循環

- **適用版本**：Godot 4.x
- **日期**：2026-05-23
- **環境**：Godot 4.6.3
- **問題**：BGM `.ogg` 拖進 AudioStreamPlayer，按 `play()` 只播一次就停
- **原因**：循環設定不在 AudioStreamPlayer node 上，**在音檔本身的「匯入」設定**。
- **解法**：
  1. FileSystem 點選音檔（如 `art/bgm.ogg`）
  2. 上方切到 **匯入** tab
  3. 勾 **Loop**
  4. 點 **重新匯入**
- **延伸**：`.wav` 短音效（如 `gameover.wav`）通常不勾 loop（一次性）。

---

### AI 工具給的 Godot 3 寫法

- **適用版本**：Godot 4.x（AI 訓練資料偏 Godot 3）
- **日期**：2026-05-23
- **環境**：使用 Claude / GitHub Copilot 寫 Godot 4 code
- **問題**：AI 給的 code 看似合理但跑不起來
- **原因**：Godot 3 → 4 大量 API 改動，AI 訓練資料 Godot 3 比例高。
- **常見差異對照**：

  | Godot 3 寫法 | Godot 4 正確寫法 |
  |---|---|
  | `connect("pressed", self, "_on_pressed")` | `pressed.connect(_on_pressed)` |
  | `tool` (header) | `@tool` |
  | `onready var x` | `@onready var x` |
  | `export var x` | `@export var x` |
  | `export(int) var x = 5` | `@export var x: int = 5` |
  | `KinematicBody2D` | `CharacterBody2D` |
  | `move_and_slide(velocity, Vector2.UP)` | `velocity = ...; move_and_slide()` |
  | `yield(get_tree(), "idle_frame")` | `await get_tree().process_frame` |
  | `yield(timer, "timeout")` | `await timer.timeout` |
  | `Engine.is_editor_hint()` | `Engine.is_editor_hint()` (沒變) |
  | `funcref(self, "fn")` | `Callable(self, "fn")` 或直接傳 `fn` |
  | `String.empty()` | `String.is_empty()` |

- **解法**：
  1. 看到可疑 API 先查 [官方文件](https://docs.godotengine.org/en/stable/classes/index.html)
  2. 跑起來看 error 訊息
  3. 把 error 訊息丟回 AI 問正確 4.x API
- **預防**：跟 AI 對話開頭明確說「**這是 Godot 4.6**」。

---

### GdUnit4 v6.0.0 跟 Godot 4.6.3 不相容（AssetLib 沒同步最新版）

- **適用版本**：Godot 4.6.3 + GdUnit4 v6.0.0（AssetLib 最新）
- **日期**：2026-05-26
- **環境**：Godot 4.6.3.stable，從 AssetLib「GdUnit4 - Unit Testing Framework」(id 4390) 下載
- **問題**：plugin 啟用後跳「Failed to load script "res://addons/gdUnit4/plugin.gd" with error "Compilation failed"」+ 連鎖 10 個 compile error。GdUnit Inspector tab 出不來。
- **根因**：
  ```
  GdUnitFileAccess.gd:199 - Parse Error:
  Too many arguments for "get_as_text()" call.
  Expected at most 0 but received 1.
  ```
  Godot 4.6 把 `FileAccess.get_as_text()` 的參數簽名改成 0 個（舊版可傳 bool）。GdUnit4 v6.0.0 是針對舊 API 寫的。
- **解法**（依嚴重度 / 速度）：
  1. **完全跳過 framework，自寫 test harness**（推薦給學習階段）：純 GDScript 寫 test_runner + assertion helpers，跨版本免疫
  2. **手動裝 v6.1.x 或 v6.2 master**：從 [GitHub Releases](https://github.com/godot-gdunit-labs/gdUnit4/releases) 抓 zip，砍掉 `addons/gdUnit4/` 解新版進去（**v6.1 官方支援 4.6.0-4.6.2，對 4.6.3 還沒驗證**）
  3. **降 Godot 到 4.6.2**：不建議，向後走
- **驗證 plugin 是否相容前**：到 GitHub README 看「Compatibility」段，confirm 你的 Godot minor 版本（不能信「Godot 4」標籤）
- **延伸**：AssetLib 版本經常滯後於 GitHub release 數週到數月。**關鍵 plugin 直接抓 GitHub release zip** 比 AssetLib 安全。

---

### Autoload 之間有相依時順序錯 → Parse Error 大爆炸

- **適用版本**：Godot 4.x（特別 4.6 較嚴格）
- **日期**：2026-05-26
- **環境**：Godot 4.6.3.stable，新建 deckbuilder-prototype 從 W3 複製 autoload
- **問題**：`project.godot` 寫
  ```
  [autoload]
  GameState="*res://autoload/game_state.gd"   ← 第 1 個
  EventBus="*res://autoload/event_bus.gd"     ← 第 2 個
  ```
  重新載入專案後爆 30+ 個 `Parse Error: Identifier "EventBus" not declared in the current scope.`，整 project 跑不起來。
- **原因**：
  - Godot 啟動時依 autoload 順序 **逐一 parse**
  - GameState 內部用了 `EventBus.damage_dealt.emit()`
  - GameState 比 EventBus 早 parse → 此時 EventBus identifier 還沒註冊 → Parse Error
  - Godot 4 比 3 嚴格，3 時代某些 forward reference 可以容忍，4 在 parse 階段就 catch
- **解法**：**被依賴的 autoload 放上面**
  ```
  [autoload]
  EventBus="*res://autoload/event_bus.gd"     ← 先（被依賴）
  GameState="*res://autoload/game_state.gd"   ← 後（依賴 EventBus）
  ```
- **通則**：autoload 依賴關係決定順序
  | 角色 | 順序 |
  |---|---|
  | 純資料 / signal hub（EventBus、Constants） | 上 |
  | 邏輯狀態（GameState、PlayerStats） | 中 |
  | 跨模組整合（AudioManager、SceneRouter） | 下 |
- **驗證方法**：重新載入專案後 Output 應該按 autoload 順序印 `_ready` log
- **預防**：寫 autoload 時先想「我會 reference 誰」→ 那個必須註冊在我前面

---

### EventBus pattern 觸發 unused_signal 警告

- **適用版本**：Godot 4.x
- **日期**：2026-05-26
- **環境**：Godot 4.6.3，autoload event_bus.gd 宣告 11 個 signal
- **問題**：EventBus autoload 自己只**宣告** signal 給別人 emit，跑起來每個 signal 都跳「The signal "X" is declared but never explicitly used in the class」警告（11 個 signal × 1 警告 = 11 條噪音）
- **原因**：Godot 的 `unused_signal` 警告偵測「class 內有沒有 emit 這個 signal」。EventBus 設計上**就是不 emit**，由其他 script 跨 scene emit，所以 GDScript 靜態分析認為它沒用。**這是 EventBus pattern 的標準誤判。**
- **解法**：在 event_bus.gd 開頭加整檔靜音
  ```gdscript
  extends Node

  # EventBus pattern：本檔只「宣告」signal 給別 script emit。
  @warning_ignore_start("unused_signal")

  signal card_played(...)
  signal damage_dealt(...)
  # ...
  ```
- **`@warning_ignore_start`**：Godot 4.3+ 引入，作用到檔尾或 `@warning_ignore_restore` 為止
- **不推薦做法**：把 `unused_signal` 警告層級在 project.godot 設為 ignore（影響整個 project，可能漏掉真的 unused signal bug）

---

### Scene 過場期間按鈕沒 disable → 玩家偷跑（race condition）

- **適用版本**：Godot 4.x（async/await 場景切換通用）
- **日期**：2026-05-29
- **環境**：Godot 4.6.3，DFS W14 棋盤擲骰 → 戰鬥場景切換
- **問題**：棋盤點「擲骰移動」→ 玩家走到敵人格 → 場景切換前有 0.6 秒視覺停頓（顯示「遭遇敵人」訊息 + fade）→ **這 0.6 秒內玩家可以再點擲骰** → 角色再移動一格（甚至再撞另一隻敵）→ 切到 combat 的是「下一個 tile」而不是真正撞到的那一個 → `pending_remove_enemy_tile` 也被覆蓋成錯的
- **原因**：移動函式 `_move_player` 在 await tween 結束後立刻 `_is_moving = false` + `button.disabled = false`，但實際的「切 scene」流程還沒走完。await 之間的 0.6 秒 timer 是「行為意義上仍在過場」但「狀態旗標已重設」。

  典型錯誤示意：
  ```gdscript
  func _move_player(steps):
      _is_moving = true
      button.disabled = true
      for i in steps:
          await tween.finished
      _is_moving = false       # ❌ 太早重設
      button.disabled = false  # ❌ 此時 caller 還會 await 0.6 秒

  func _trigger_encounter():
      await timer(0.6).timeout  # ← 這 0.6 秒按鈕又能按了
      SceneRouter.change_scene(...)
  ```

- **解法**：lock 的責任改由 caller 承擔，依「會不會切 scene」決定要不要 re-enable：
  ```gdscript
  func _on_roll_pressed():
      if _is_moving: return
      var encountered = await _move_player(roll)
      if encountered:
          await _trigger_encounter()  # 切 scene，不 re-enable
          return
      var changing_scene = await _resolve_tile()
      if changing_scene: return
      # 確定沒切 scene 才 re-enable
      _is_moving = false
      button.disabled = false

  func _move_player(steps) -> bool:
      _is_moving = true
      button.disabled = true
      for i in steps: await tween.finished
      # 故意不在此 re-enable，留給 caller
      return encountered
  ```
- **觀念**：async/await 場景過場的 race condition 一定要把「lock 範圍 = 整個過場」想清楚。`await tween.finished` 不等於「使用者操作的 logical end」，後續還有 timer / scene change / fade。一旦切 scene 失敗或被中斷，重新進來的 board scene 是 fresh instance（_is_moving = false 預設），所以「忘了 re-enable」不會 stuck。
- **面試 talking point**：「Godot 的 async/await 跟 JS Promise 類似，但 scene tree 是長生 stateful，rance condition 比 web 還容易踩。我在 DFS 把 lock 責任從『動作執行者』移到『動作協調者』(caller)，誰知道流程會不會切 scene 誰負責 lock — 單一職責。」

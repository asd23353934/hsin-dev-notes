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

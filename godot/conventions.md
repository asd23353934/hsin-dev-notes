# Godot Conventions

> Godot / GDScript 寫法慣例。Claude Code 在產出 Godot 程式碼時必須遵守。
> 最後更新：2026-05-23

---

## 命名

### 檔名

| 種類 | 慣例 | 範例 |
|---|---|---|
| `.gd` 腳本 | `snake_case.gd` | `player.gd`、`card_data.gd` |
| `.tscn` 場景 | `snake_case.tscn` | `main.tscn`、`hud.tscn` |
| `.tres` 資源 | `snake_case.tres` | `strike.tres`、`mob_behavior.tres` |
| 資料夾 | `snake_case/` | `autoload/`、`cards/` |

### 識別字

| 種類 | 慣例 | 範例 |
|---|---|---|
| `class_name` | **PascalCase** | `class_name CardData` |
| Autoload 註冊名 | **PascalCase** | `GameState`、`EventBus` |
| 變數 / function | `snake_case` | `var player_hp`、`func take_damage()` |
| Signal | `snake_case` 動詞 | `signal hit`、`signal damage_dealt` |
| Constant | `SCREAMING_SNAKE_CASE` | `const MAX_HP = 100` |
| 未使用參數 | 加底線前綴 | `func _on_body_entered(_body):` |

---

## 型別宣告（**強制**）

GDScript 是 dynamic 但支援 static type hint，**全部寫上**，效能跟可讀性都好。

```gdscript
# ✓ 好
var hp: int = 100
var name: String = ""
func attack(target: Node2D, damage: int) -> void:
    ...

# ✗ 不要
var hp = 100
var name = ""
func attack(target, damage):
    ...
```

### 例外：`:=` 推斷

短期 local var 可用 `:=` 推斷，但**對 `$Node.prop` 失效**（靜態分析推不出來）：

```gdscript
var velocity := Vector2.ZERO         # ✓ OK
var size := some_array.size()        # ✓ OK
var anim := $AnimatedSprite2D.sprite_frames.get_animation_names()  # ✗ 推不出來，明寫 Array
```

---

## Node 取用

### 子 node 用 `@onready` cache

```gdscript
# ✓ 好
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D

func _on_hit() -> void:
    sprite.play("hurt")

# ✗ 不要每次重抓
func _on_hit() -> void:
    $AnimatedSprite2D.play("hurt")   # 偶爾可，但常用就要 cache
```

### **絕對**不要在 `_process` / `_physics_process` 裡 `get_node()`

每 frame 跑會傷效能。**永遠 `@onready` cache**。

### 外部 dependency 用 `@export`

```gdscript
# ✓ 好（Inspector 拖檔注入，可換）
@export var mob_scene: PackedScene

# ✗ 寫死（要改要動 code）
const MOB_SCENE = preload("res://mob.tscn")
```

例外：常數型資源（**永遠不會換**）可以 `const + preload`，避免 Inspector 拖檔出錯。

---

## Signal 設計原則

### Signal up + method down

- **Child → Parent**：用 signal emit（child 不認識 parent 才能 reusable）
- **Parent → Child**：直接 `$Child.method()` call（parent 握有 child 引用）

```gdscript
# child（hud.gd）
signal start_game

func _on_button_pressed() -> void:
    start_game.emit()

# parent（main.gd）
func _ready() -> void:
    $HUD.start_game.connect(_on_hud_start_game)

func _on_hud_start_game() -> void:
    new_game()    # parent 主動 call child
    $HUD.update_score(0)
```

### Signal 命名

- 用**過去式動詞**或**狀態變化**：`hit`、`damage_dealt`、`player_died`、`hp_changed`
- 不用名詞：~~`player`~~、~~`combat`~~

### 自訂 signal 統一放 EventBus（全域用）

跨場景通訊用 EventBus；同 scene 內 parent-child 用 local signal。

---

## Resource (.tres) 設計

### 用 Resource 取代寫死資料

```gdscript
# ✓ 好（80 張卡可擴展）
class_name CardData
extends Resource

@export var card_name: String
@export var cost: int
@export var damage: int
@export_multiline var description: String

# ✗ 不要（卡牌資料寫死）
var STRIKE = {"name": "Strike", "cost": 1, "damage": 6}
var DEFEND = {"name": "Defend", "cost": 1, "damage": 0}
```

### `.tres` 一張卡一檔

```
cards/
├── strike.tres
├── defend.tres
└── focus.tres
```

不要一個 .tres 塞 array of cards。**單檔單資料**好搜尋好 diff。

---

## Autoload 使用原則

### 適合 autoload

- 全域玩家狀態（HP、金幣、解鎖紀錄）
- 跨場景 EventBus
- 跨場景持續資源（BGM、AudioManager）
- 統一輸入處理（InputBuffer 等）

### 不適合 autoload

- 單一 scene 內部狀態 → 用 scene 本身的 var
- 相鄰 node 通訊 → 用 local signal
- 純資料 → 用 Resource

### Autoload 必須 `extends Node`（或子類）

不能 `extends Resource` / `extends Object`。**官方文件原話**：「You can create an Autoload to load a scene or a script that **inherits from Node**」。

---

## State 修改 + emit 廣播

State owner 修改自己後**立刻 emit signal**，保證通知不漏：

```gdscript
# game_state.gd
func take_damage(amount: int) -> void:
    player_hp = max(0, player_hp - amount)
    EventBus.damage_dealt.emit(amount, "Player")
    EventBus.player_hp_changed.emit(player_hp, max_hp)
    if player_hp == 0:
        EventBus.player_died.emit()
```

**不要讓外面自己改 + 自己 emit**（會漏 emit 造成 UI 不更新）。

---

## 物理 callback 安全規則

任何在 `_on_body_entered` / `_on_area_entered` 等物理 callback 內**修改物理屬性**（disable collision、change layer、queue_free 觸發的物理重算），用 `set_deferred`：

```gdscript
# ✓ 安全
func _on_body_entered(_body: Node2D) -> void:
    $CollisionShape2D.set_deferred("disabled", true)

# ✗ 物理算到一半改，可能 crash
func _on_body_entered(_body: Node2D) -> void:
    $CollisionShape2D.disabled = true
```

`queue_free()` 本身是 deferred 的，不用包。

---

## Movement 永遠乘 delta

```gdscript
# ✓ 跨 fps 一致
position += velocity * delta

# ✗ 60fps 跟 30fps 跑出來速度不一樣
position += velocity
```

例外：`_physics_process(delta)` 的 delta 是固定值（預設 1/60），長期 movement 建議寫在 `_physics_process` 而非 `_process`。

---

## Input 取用

### 內建 `ui_*` action 不要重綁

| Action | 用途 |
|---|---|
| `ui_accept` | 確認 / 開始（Enter / Space） |
| `ui_cancel` | 取消 / 返回（Escape） |
| `ui_left/right/up/down` | UI 導航 |
| `ui_text_submit` | 文字輸入確認 |

自訂遊戲操作另開名字：`move_left`、`attack`、`open_menu`。

### 方向鍵讀法

```gdscript
# ✓ 簡潔，自動 normalize + 手把 deadzone 處理
var direction := Input.get_vector("move_left", "move_right", "move_up", "move_down")

# 教學用 4-if（清楚但繁瑣）
var velocity := Vector2.ZERO
if Input.is_action_pressed("move_right"): velocity.x += 1
# ...
velocity = velocity.normalized()
```

---

## 場景 / Node 設計

### 場景 root 不該有 transform

Player.tscn 的 root（Player Area2D）的 Position / Scale / Rotation **必須是預設值**（0, 0 / 1, 1 / 0）。
變形要設在子 node（如 AnimatedSprite2D 的 scale）。

**理由**：scene instance 後位置由 parent 決定，root 自帶 transform 會混亂。Godot 編輯器會跳警告。

### 場景 default state

可重用組件的初始狀態應該是「**default: invisible / disabled**」，由外部觸發顯示：

```gdscript
# player.gd
func _ready() -> void:
    hide()    # default 隱藏

func start(pos: Vector2) -> void:
    position = pos
    show()
    $CollisionShape2D.disabled = false
```

外面叫的時候才活起來，**Player.tscn 可以在主選單、戰鬥、死亡畫面共用**。

---

## 場景組合 vs 繼承

**強烈傾向組合**：每個 node 一個職責，scene 組合節點而非繼承 class。

```
# ✓ 組合
Player (Area2D)               ← 邏輯 + 碰撞偵測
├── AnimatedSprite2D          ← 顯示
└── CollisionShape2D          ← hitbox

# ✗ 不要這樣寫一個 class 啥都做
class FlyingShootingHidingPlayer extends BaseEnemy:
    ...
```

例外：純資料 class 可繼承（`Skill extends Effect`）。

---

## F5 vs F6

- **F5**：跑主場景（**Project Settings → Run → Main Scene** 設定的）
- **F6**：跑當前打開的場景

多場景專案開發中**頻繁切 F5 / F6**，搞清楚自己要跑哪個。

---

## Git workflow

### `.gitignore` 必含

```gitignore
.godot/
/android/
```

`.godot/` 是 Godot 編譯 cache，**不要 commit**（裡面有電腦相依路徑）。

### Commit pattern

```
<type>(<scope>): <description>

type: feat / fix / refactor / docs / chore
scope: 模組名（如 combat / hud / autoload）
```

範例：

```
feat(combat): add Combo Lock universal mechanism
fix(hud): correct score label anchor on small screens
refactor(autoload): split GameState into PlayerState + RunState
docs(readme): add gameplay screenshot
```

### Commit 不標 Claude（除 godot-learning-path repo）

除了 `godot-learning-path`（學習紀錄 repo 標 Co-Authored-By Claude），其他練習 / 產品 repo（dodge-the-creeps、card-resource-demo、dice-fate-survivor 等）**不要加 Co-Authored-By: Claude**。

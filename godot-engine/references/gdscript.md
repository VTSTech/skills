# GDScript 4.x Complete Reference

## Syntax Overview

GDScript is indentation-based (like Python), object-oriented, and tightly integrated with Godot's node system. It uses reference counting instead of garbage collection.

### Variables

```gdscript
# Type inference
var name = "Player"
var health = 100
var position = Vector2(10, 20)
var alive = true

# Static typing (recommended for performance and IDE support)
var player_name: String = "Hero"
var max_health: int = 100
var speed: float = 200.0
var is_invulnerable: bool = false
var items: Array[String] = ["sword", "shield"]
var enemies: Dictionary = {"goblin": 5, "orc": 2}

# Constants
const MAX_SPEED = 400.0
const JUMP_VELOCITY = -400.0
const DAMAGE_MULTIPLIER = 1.5
```

### Built-in Types

```gdscript
# Math
int        # 64-bit integer
float      # 64-bit float
bool       # true / false

# Vectors & Transforms
Vector2    # 2D vector (x, y)
Vector2i   # 2D integer vector
Vector3    # 3D vector (x, y, z)
Vector3i   # 3D integer vector
Vector4    # 4D vector
Transform2D # 2D affine transform (3x3 matrix)
Transform3D # 3D affine transform (4x4 matrix)
Basis      # 3x3 rotation/scale matrix

# Geometry
Rect2      # 2D rectangle (position, size)
Rect2i     # 2D integer rectangle
AABB       # 3D axis-aligned bounding box
Plane      # 3D plane

# Color & Math Helpers
Color      # RGBA color
Quaternion # 3D rotation

# Collections
Array      # Dynamic array (variant or typed)
Dictionary # Hash map (String/variant keys, variant values)
```

### Control Flow

```gdscript
# If/elif/else
if health <= 0:
    die()
elif health < 30:
    show_critical_warning()
else:
    regenerate()

# Match (Godot's switch/case — supports patterns)
match state:
    State.IDLE:
        handle_idle()
    State.RUNNING:
        handle_running()
    State.ATTACKING:
        handle_attack()
    _, _:  # wildcard + bind (any other value)
        handle_unknown()

# For loops
for i in range(10):           # 0 to 9
    spawn_enemy(i)

for item in inventory:        # iterate collection
    use_item(item)

for enemy in get_children():  # iterate node children
    enemy.queue_free()

# While loop
while mana > cost:
    cast_spell()

# match with patterns
match value:
    int:
        print("It's an integer")
    [var a, var b]:
        print("Array with two elements: ", a, b)
    {"name": var name, "hp": var hp}:
        print(name, " has ", hp, " HP")
    _:
        print("Unknown type")
```

### Functions

```gdscript
# Basic function
func heal(amount: int) -> void:
    health = mini(health + amount, max_health)

# Function with return
func calculate_damage(base: int, multiplier: float) -> int:
    return int(base * multiplier)

# Default parameters
func spawn_enemy(type: String = "goblin", level: int = 1) -> void:
    pass

# Variadic (not supported — use Array instead)
func apply_buffs(buff_names: PackedStringArray) -> void:
    for buff in buff_names:
        apply_buff(buff)
```

### Classes and Inheritance

```gdscript
# Scripts implicitly create classes that extend a node type
extends CharacterBody2D

class_name Enemy  # Register as a global type (optional)

# Inner class
class StatBlock:
    var strength: int
    var dexterity: int
    var vitality: int

    func _init(str_val: int = 10, dex_val: int = 10, vit_val: int = 10):
        strength = str_val
        dexterity = dex_val
        vitality = vit_val

# Enum (preferred over int constants for state)
enum State { IDLE, WALK, RUN, ATTACK, HURT, DEAD }
enum Element { FIRE, WATER, EARTH, AIR, LIGHTNING }

# Properties
var current_state: State = State.IDLE
var stats: StatBlock = StatBlock.new()
```

## Export Annotations

`@export` exposes variables to the editor Inspector. This is the primary way to configure nodes visually.

```gdscript
@export var speed: float = 200.0
@export var max_health: int = 100
@export var jump_velocity: float = -400.0
@export var sprite_frames: SpriteFrames

# Range with slider
@export_range(0, 100, 1) var health: int = 100
@export_range(0.0, 1.0, 0.01) var opacity: float = 1.0

# Resource types
@export var weapon: WeaponResource
@export var inventory_data: InventoryResource

# Enums show dropdown
@export var element: Element = Element.FIRE

# Arrays
@export var item_drops: Array[PackedScene] = []
@export var spawn_points: Array[Vector2] = []

# Grouping with @export_group and @export_subgroup
@export_group("Movement")
@export var walk_speed: float = 150.0
@export var run_speed: float = 250.0
@export var acceleration: float = 10.0

@export_group("Combat")
@export var attack_damage: int = 10
@export var attack_range: float = 50.0
@export var attack_cooldown: float = 0.5

@export_subgroup("Skills")
@export var primary_skill: SkillResource
@export var ultimate_skill: SkillResource

# Color
@export_color_no_alpha var team_color: Color = Color.WHITE

# File paths
@export_file("*.tscn", "*.scn") var level_scene: String
@export_dir var asset_folder: String

# Multiline strings
@export_multiline var description: String = ""

# Flags (bitfield)
@export_flags("Fire", "Water", "Earth", "Air") var elements: int
```

## Signals

```gdscript
# Declare
signal health_changed(old_value: int, new_value: int)
signal died()
signal item_collected(item: ItemResource)
signal interacted(body: Node2D)

# Emit
func take_damage(amount: int) -> void:
    var old_health = health
    health -= amount
    health_changed.emit(old_health, health)
    if health <= 0:
        died.emit()

# Connect in code
func _ready() -> void:
    health_changed.connect(_on_health_changed)
    died.connect(_on_died)

func _on_health_changed(old_val: int, new_val: int) -> void:
    health_bar.value = new_val

func _on_died() -> void:
    animation_player.play("death")

# Connect with lambda
button.pressed.connect(func(): _start_game(difficulty))

# One-shot connection (auto-disconnects after first emit)
area_2d.body_entered.connect(_on_body_entered.unbind(1))  # only if needed

# Disconnect
timer.timeout.disconnect(_on_timeout)
```

## Node References

```gdscript
# $ notation — get child node by path
@onready var sprite = $Sprite2D
@onready var collision = $CollisionShape2D
@onready var anim_player = $AnimationPlayer
@onready var camera = $Camera2D

# % notation — get unique node (regardless of depth)
@onready var health_bar = %HealthBar

# getPath-style
@onready var player = get_node("/root/Game/World/Player")
@onready var game_manager = get_node("/root/GameManager")  # autoload

# get_node_or_null (safe access, won't error if missing)
@onready var optional_node = get_node_or_null("OptionalNode")

# Dynamic node creation
func spawn_projectile() -> void:
    var projectile = PROJECTILE_SCENE.instantiate()
    projectile.global_position = global_position
    projectile.direction = (target_position - global_position).normalized()
    get_tree().current_scene.add_child(projectile)

# get_children(), get_parent()
func clear_children() -> void:
    for child in get_children():
        child.queue_free()
```

## Coroutines with await

```gdscript
# Wait for a timer
func take_damage(amount: int) -> void:
    invulnerable = true
    health -= amount
    await get_tree().create_timer(invincibility_duration).timeout
    invulnerable = false

# Wait for animation
func play_attack() -> void:
    animation_player.play("attack")
    await animation_player.animation_finished
    is_attacking = false

# Wait for a signal
func interact() -> void:
    var dialogue_box = preload("res://scenes/ui/dialogue_box.tscn").instantiate()
    add_child(dialogue_box)
    await dialogue_box.closed  # custom signal
    dialogue_box.queue_free()

# Wait one frame (useful for deferring)
func deferred_setup() -> void:
    await get_tree().process_frame
    # Now the scene tree is fully ready
    setup_collision()
```

## Nullable Types — GOTCHA

**GDScript object types are nullable by default** — they can hold `null` without any special syntax.

```gdscript
# CORRECT — object types can return null naturally
func get_item(index: int) -> ItemResource:
    if index < items.size():
        return items[index]
    return null  # this is fine, no ? needed

# WRONG — the ? suffix causes parse errors in Godot 4.6.2
func get_item(index: int) -> ItemResource?:  # PARSE ERROR!
```

**Do NOT use `Type?` on function return types.** The GDScript parser treats the `?` as a syntax error ("Expected end of statement after bodyless function declaration, found '?' instead."). This was confirmed in Godot 4.6.2. Just use `Type` — it's nullable by default.

## Typed Arrays & Dictionaries

```gdscript
# Typed arrays (Godot 4)
var names: Array[String] = ["Alice", "Bob"]
var positions: Array[Vector2] = [Vector2(0, 0), Vector2(100, 50)]
var enemies: Array[Enemy] = []

# Works with var
var items: Array[ItemResource] = []

# Dictionary with typed keys/values
var inventory: Dictionary[String, int] = {"potions": 5, "swords": 1}

# Packed arrays (more performant for known types)
var floats: PackedFloat32Array = [1.0, 2.0, 3.0]
var colors: PackedColorArray = [Color.RED, Color.BLUE]
var strings: PackedStringArray = ["hello", "world"]
var vectors: PackedVector2Array = [Vector2(0, 0), Vector2(10, 20)]
```

## GDScript Style Guide (Godot 4.x Official)

### Naming Conventions

```gdscript
# Classes, nodes, enums: PascalCase
class_name PlayerController
enum MovementState { IDLE, WALK, SPRINT }

# Functions, variables: snake_case
var movement_speed: float
func calculate_damage() -> int:

# Constants: UPPER_SNAKE_CASE
const MAX_HEALTH = 100
const BASE_SPEED = 200.0

# Signals: past tense (events that happened)
signal health_changed()
signal enemy_died()
signal item_picked_up()

# Private variables: underscore prefix
var _internal_timer: float
func _update_physics() -> void:
```

### Code Style

```gdscript
# Indent: TABS (not spaces) — Godot default
# Braces: K&R style (opening brace on same line)
# Always type hints on function parameters and returns
func deal_damage(target: CharacterBody2D, amount: int) -> bool:
    if target.has_method("take_damage"):
        target.take_damage(amount)
        return true
    return false

# Use @onready instead of _ready() for node references
@onready var animation_player: AnimationPlayer = $AnimationPlayer
# NOT:
# var animation_player  # set in _ready()

# Prefer explicit types for node references
@onready var sprite: Sprite2D = $Sprite2D
@onready var collision_shape: CollisionShape2D = $CollisionShape2D

# Use guard clauses (early returns) over deep nesting
func _physics_process(delta: float) -> void:
    if not is_on_floor():
        return

    if is_dead:
        return

    var input_dir = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    velocity = input_dir * speed
    move_and_slide()

# Avoid using `self.` unless explicitly needed (shadowing, property setters)
self.health = new_value  # only when needed
health = new_value  # preferred
```

### String Formatting

```gdscript
# F-strings (Godot 4.0+)
print("Player has %d HP and %d mana" % [health, mana])

# Or use string concatenation for simple cases
var msg = "Health: " + str(health)

# Format method
var msg = "Score: %08d | Time: %05.1f" % [score, elapsed_time]
```

## Common Code Patterns

### Player Movement (2D Top-Down)

```gdscript
extends CharacterBody2D

@export var speed: float = 200.0

func _physics_process(_delta: float) -> void:
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    velocity = direction * speed
    move_and_slide()
```

### Player Movement (2D Platformer)

```gdscript
extends CharacterBody2D

@export var speed: float = 300.0
@export var jump_velocity: float = -400.0
@export var gravity: float = 980.0

func _physics_process(delta: float) -> void:
    # Add gravity
    if not is_on_floor():
        velocity.y += gravity * delta

    # Handle jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_velocity

    # Handle movement
    var direction = Input.get_axis("move_left", "move_right")
    if direction:
        velocity.x = direction * speed
    else:
        velocity.x = move_toward(velocity.x, 0, speed)

    move_and_slide()
```

### Player Movement (3D First Person)

```gdscript
extends CharacterBody3D

@export var speed: float = 5.0
@export var jump_velocity: float = 4.5
@export var mouse_sensitivity: float = 0.002

var gravity: float = ProjectSettings.get_setting("physics/3d/default_gravity")

@onready var head: Node3D = $Head
@onready var camera: Camera3D = $Head/Camera3D

func _ready() -> void:
    Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)

func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseMotion:
        rotate_y(-event.relative.x * mouse_sensitivity)
        head.rotate_x(-event.relative.y * mouse_sensitivity)
        head.rotation.x = clampf(head.rotation.x, -PI / 2, PI / 2)

    if event.is_action_pressed("ui_cancel"):
        Input.set_mouse_mode(Input.MOUSE_MODE_VISIBLE)

func _physics_process(delta: float) -> void:
    if not is_on_floor():
        velocity.y -= gravity * delta

    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_velocity

    var input_dir = Input.get_vector("move_left", "move_right", "move_forward", "move_back")
    var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
    if direction:
        velocity.x = direction.x * speed
        velocity.z = direction.z * speed
    else:
        velocity.x = move_toward(velocity.x, 0, speed)
        velocity.z = move_toward(velocity.z, 0, speed)

    move_and_slide()
```

### Animation State Machine

```gdscript
extends CharacterBody2D

enum State { IDLE, RUN, ATTACK, HURT, DEAD }

@export var speed: float = 200.0

var current_state: State = State.IDLE
@onready var anim_player: AnimationPlayer = $AnimationPlayer
@onready var sprite: Sprite2D = $Sprite2D

func _physics_process(_delta: float) -> void:
    match current_state:
        State.IDLE:
            _state_idle()
        State.RUN:
            _state_run()
        State.ATTACK:
            _state_attack()
        State.HURT:
            _state_hurt()
        State.DEAD:
            _state_dead()

    move_and_slide()

func _state_idle() -> void:
    anim_player.play("idle")
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    if direction != Vector2.ZERO:
        current_state = State.RUN
    if Input.is_action_just_pressed("attack"):
        current_state = State.ATTACK

func _state_run() -> void:
    anim_player.play("run")
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    velocity = direction * speed
    if direction == Vector2.ZERO:
        current_state = State.IDLE
    if Input.is_action_just_pressed("attack"):
        current_state = State.ATTACK

func _state_attack() -> void:
    anim_player.play("attack")
    velocity = Vector2.ZERO
    # Attack state ends via animation finished signal:
    # func _on_animation_player_animation_finished(anim_name):
    #     if anim_name == "attack":
    #         current_state = State.IDLE

func change_state(new_state: State) -> void:
    current_state = new_state
```

### Area2D Detection (Pickups, Damage Zones)

```gdscript
extends Area2D

signal coin_collected()

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is CharacterBody2D and body.has_method("collect_coin"):
        body.collect_coin()
        coin_collected.emit()
        queue_free()  # remove the pickup
```

## Type Casting

```gdscript
# Safe casting with `as`
func _on_body_entered(body: Node2D) -> void:
    var player = body as PlayerController
    if player:
        player.take_damage(10)

# Using `is` keyword for type checking
func _process(_delta: float) -> void:
    for child in get_children():
        if child is Enemy:
            child.patrol()
        elif child is Projectile:
            child.update_trajectory()
```

## Assertions and Debugging

```gdscript
# Assertions (only run in debug builds)
assert(health > 0, "Health should not be negative!")
assert(speed >= 0, "Speed cannot be negative")

# Print debugging
push_warning("Low health warning: %d HP" % health)
push_error("Critical error: player node not found!")
print("Debug info: position = ", global_position)

# Debug drawing (in _draw for CanvasItem nodes)
func _draw() -> void:
    draw_circle(Vector2.ZERO, attack_range, Color.RED, false)
    draw_line(Vector2.ZERO, direction * 50, Color.YELLOW, 2.0)
```
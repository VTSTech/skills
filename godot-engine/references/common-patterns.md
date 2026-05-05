# Godot 4.x Common Game Development Patterns

## Singleton / Autoload Pattern

Global singletons in Godot are called **Autoloads**. Register them in Project Settings > Autoload. They are automatically added to the scene tree root and accessible from any script.

```gdscript
# scripts/game_manager.gd
extends Node

signal score_changed(new_score: int)
signal game_over()

var score: int = 0
var lives: int = 3
var current_level: int = 1

func add_score(points: int) -> void:
    score += points
    score_changed.emit(score)

func lose_life() -> void:
    lives -= 1
    if lives <= 0:
        game_over.emit()

func reset_game() -> void:
    score = 0
    lives = 3
    current_level = 1
```

Access from any script:
```gdscript
GameManager.add_score(100)
GameManager.lives -= 1
```

## Component Pattern

Split reusable logic into separate Node scripts that can be attached to any node.

```gdscript
# scripts/components/health_component.gd
extends Node
class_name HealthComponent

signal health_changed(old_health: int, new_health: int)
signal died()

@export var max_health: int = 100
var current_health: int

func _ready() -> void:
    current_health = max_health

func take_damage(amount: int) -> void:
    var old_health = current_health
    current_health = maxi(current_health - amount, 0)
    health_changed.emit(old_health, current_health)
    if current_health <= 0:
        died.emit()

func heal(amount: int) -> void:
    var old_health = current_health
    current_health = mini(current_health + amount, max_health)
    health_changed.emit(old_health, current_health)

func is_dead() -> bool:
    return current_health <= 0
```

```gdscript
# scripts/components/inventory_component.gd
extends Node
class_name InventoryComponent

signal item_added(item: ItemResource)
signal item_removed(item: ItemResource)
signal inventory_updated()

@export var max_slots: int = 20
var items: Array[ItemResource] = []

func add_item(item: ItemResource) -> bool:
    if items.size() >= max_slots:
        return false
    items.append(item)
    item_added.emit(item)
    inventory_updated.emit()
    return true

func remove_item(item: ItemResource) -> void:
    items.erase(item)
    item_removed.emit(item)
    inventory_updated.emit()

func get_items() -> Array[ItemResource]:
    return items.duplicate()
```

Usage — attach as child of any node:
```
Player (CharacterBody2D)
├── HealthComponent
├── InventoryComponent
└── Sprite2D
```

## Resource-Based Data Pattern

Use custom `Resource` classes for data-driven design. Resources are reusable, saveable data containers.

```gdscript
# resources/item_resource.gd
extends Resource
class_name ItemResource

@export var name: String = "Unknown Item"
@export var description: String = ""
@export var icon: Texture2D
@export var stackable: bool = false
@export var max_stack: int = 1
@export var value: int = 0
@export var item_type: String = "misc"  # weapon, armor, consumable, misc
```

```gdscript
# resources/weapon_resource.gd
extends Resource
class_name WeaponResource

@export var name: String = "Rusty Sword"
@export var damage: int = 10
@export var attack_speed: float = 1.0
@export var range: float = 50.0
@export var weapon_scene: PackedScene  # visual model
@export var element: String = "physical"
```

Create instances in .tres files or via code:
```gdscript
var sword = WeaponResource.new()
sword.name = "Iron Sword"
sword.damage = 25
sword.weapon_scene = preload("res://models/weapons/iron_sword.tscn")
```

## Finite State Machine (Resource-Based)

Cleaner than enum-based for complex entities with many states.

```gdscript
# resources/state.gd
extends Resource
class_name State

var state_machine: Node  # set by state machine on entry

func enter() -> void:
    pass

func exit() -> void:
    pass

func update(_delta: float) -> void:
    pass

func physics_update(_delta: float) -> void:
    pass

func handle_input(_event: InputEvent) -> void:
    pass
```

```gdscript
# scripts/state_machine.gd
extends Node
class_name StateMachine

@export var initial_state: State
var current_state: State
var states: Dictionary = {}

func _ready() -> void:
    # Collect child states
    for child in get_children():
        if child is State:
            states[child.name.to_lower()] = child
            child.state_machine = self

    if initial_state:
        current_state = initial_state
        current_state.enter()

func transition_to(state_name: String) -> void:
    if state_name not in states:
        push_warning("State %s not found" % state_name)
        return
    current_state.exit()
    current_state = states[state_name]
    current_state.enter()

func _process(delta: float) -> void:
    if current_state:
        current_state.update(delta)

func _physics_process(delta: float) -> void:
    if current_state:
        current_state.physics_update(delta)

func _unhandled_input(event: InputEvent) -> void:
    if current_state:
        current_state.handle_input(event)
```

```gdscript
# scripts/states/player_idle.gd
extends State

func update(_delta: float) -> void:
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    if direction != Vector2.ZERO:
        state_machine.transition_to("run")
    if Input.is_action_just_pressed("attack"):
        state_machine.transition_to("attack")
```

## Save/Load System

```gdscript
# scripts/save_manager.gd (autoload)
extends Node

const SAVE_PATH = "user://savegame.json"

func save_game() -> void:
    var save_data = {
        "player": {
            "health": GameManager.player_health,
            "position": {
                "x": GameManager.player_position.x,
                "y": GameManager.player_position.y,
            },
            "inventory": _serialize_inventory(),
        },
        "level": GameManager.current_level,
        "score": GameManager.score,
        "play_time": Time.get_ticks_msec() / 1000.0,
    }
    var json_string = JSON.stringify(save_data, "\t")
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    file.store_string(json_string)
    file.close()

func load_game() -> bool:
    if not FileAccess.file_exists(SAVE_PATH):
        push_warning("No save file found")
        return false

    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var json_string = file.get_as_text()
    file.close()

    var json = JSON.new()
    var error = json.parse(json_string)
    if error != OK:
        push_error("Failed to parse save file")
        return false

    var save_data = json.data
    GameManager.player_health = save_data["player"]["health"]
    GameManager.current_level = save_data["level"]
    GameManager.score = save_data["score"]
    return true

func has_save() -> bool:
    return FileAccess.file_exists(SAVE_PATH)

func delete_save() -> void:
    if FileAccess.file_exists(SAVE_PATH):
        DirAccess.remove_absolute(SAVE_PATH)

func _serialize_inventory() -> Array:
    var serialized = []
    for item in GameManager.inventory:
        serialized.append({
            "resource_path": item.resource_path,
            "name": item.name,
        })
    return serialized
```

## Scene Management / Transitions

```gdscript
# scripts/scene_manager.gd (autoload)
extends Node

signal scene_changed(scene_name: String)
signal transition_started()
signal transition_finished()

@onready var _anim_player: AnimationPlayer = $AnimationPlayer
@onready var _color_rect: ColorRect = $ColorRect

var _current_scene: Node

func _ready() -> void:
    var root = get_tree().current_scene
    _current_scene = root

func goto_scene(path: String, transition: bool = true) -> void:
    if transition:
        transition_started.emit()
        _anim_player.play("fade_out")
        await _anim_player.animation_finished

    _current_scene.queue_free()
    var new_scene = load(path).instantiate()
    get_tree().current_scene = new_scene
    get_tree().root.add_child(new_scene)
    _current_scene = new_scene
    scene_changed.emit(path.get_file().get_basename())

    if transition:
        _anim_player.play("fade_in")
        await _anim_player.animation_finished
        transition_finished.emit()
```

## Object Pooling

Reuse objects instead of creating/destroying them every frame.

```gdscript
# scripts/object_pool.gd
extends Node
class_name ObjectPool

var _pool: Array[Node] = []
var _scene: PackedScene
var _parent: Node

func setup(scene: PackedScene, parent: Node, initial_size: int = 10) -> void:
    _scene = scene
    _parent = parent
    for i in initial_size:
        var instance = _scene.instantiate()
        instance.set_process(false)
        instance.set_physics_process(false)
        instance.visible = false
        _parent.add_child(instance)
        _pool.append(instance)

func get_instance() -> Node:
    for obj in _pool:
        if not obj.visible:
            obj.set_process(true)
            obj.set_physics_process(true)
            obj.visible = true
            return obj

    # Pool exhausted — create new instance
    var instance = _scene.instantiate()
    _parent.add_child(instance)
    _pool.append(instance)
    return instance

func release(instance: Node) -> void:
    instance.set_process(false)
    instance.set_physics_process(false)
    instance.visible = false
```

## Screen Shake

```gdscript
# scripts/screen_shake.gd
extends Camera2D

@export var decay_rate: float = 5.0

var _strength: float = 0.0

func shake(strength: float = 10.0, duration: float = 0.3) -> void:
    _strength = strength
    var tween = create_tween()
    tween.tween_property(self, "_strength", 0.0, duration)

func _process(_delta: float) -> void:
    if _strength > 0.1:
        offset = Vector2(
            randf_range(-_strength, _strength),
            randf_range(-_strength, _strength)
        )
    else:
        offset = Vector2.ZERO
        _strength = 0.0
```

For Camera3D, set `h_offset` and `v_offset` instead of `offset`.

## Damage Numbers (Floating Text)

```gdscript
# scenes/ui/damage_number.tscn
# Root: Node2D with script
extends Node2D

@onready var label: Label = $Label
var _lifetime: float = 0.8
var _elapsed: float = 0.0
var _direction: Vector2 = Vector2.UP

func setup(value: int, is_critical: bool = false) -> void:
    label.text = str(value)
    if is_critical:
        label.add_theme_color_override("font_color", Color.GOLD)
        label.add_theme_font_size_override("font_size", 24)
    else:
        label.add_theme_color_override("font_color", Color.WHITE)

func _process(delta: float) -> void:
    _elapsed += delta
    position += _direction * 60 * delta
    modulate.a = 1.0 - (_elapsed / _lifetime)
    if _elapsed >= _lifetime:
        queue_free()
```

Spawner:
```gdscript
# On the player or a global manager
@export var damage_number_scene: PackedScene

func show_damage_number(value: int, position: Vector2, is_critical: bool = false) -> void:
    var number = damage_number_scene.instantiate()
    number.global_position = position + Vector2(randf_range(-10, 10), -20)
    number.setup(value, is_critical)
    get_tree().current_scene.add_child(number)
```

## Dialogue System (Basic)

```gdscript
# resources/dialogue_resource.gd
extends Resource
class_name DialogueResource

@export var character_name: String = "NPC"
@export var lines: Array[DialogueLine] = []
@export var choices: Array[DialogueChoice] = []

# Nested resource
# class DialogueLine:
#     @export var text: String
#     @export var speaker: String
#     @export var emotion: String  # triggers animation
```

```gdscript
# scripts/dialogue_box.gd (Control node)
extends Control

signal dialogue_finished()

@export var dialogue: DialogueResource
var _current_line: int = 0

@onready var name_label: Label = $NameLabel
@onready var text_label: RichTextLabel = $TextLabel

func start(new_dialogue: DialogueResource) -> void:
    dialogue = new_dialogue
    _current_line = 0
    visible = true
    show_line()

func show_line() -> void:
    if _current_line >= dialogue.lines.size():
        finish()
        return
    var line = dialogue.lines[_current_line]
    name_label.text = line.speaker if line.speaker else dialogue.character_name
    text_label.text = line.text
    _current_line += 1

func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("ui_accept") and visible:
        show_line()

func finish() -> void:
    visible = false
    dialogue_finished.emit()
```

## Enemy AI (Patrol + Chase)

```gdscript
# scripts/enemy_ai.gd
extends CharacterBody2D

enum State { PATROL, CHASE, ATTACK, DEAD }

@export var move_speed: float = 100.0
@export var chase_speed: float = 180.0
@export var detection_range: float = 150.0
@export var attack_range: float = 40.0
@export var attack_cooldown: float = 1.0

var current_state: State = State.PATROL
var player: CharacterBody2D
var patrol_points: Array[Vector2] = []
var current_patrol_index: int = 0
var attack_timer: float = 0.0

func _ready() -> void:
    player = get_tree().get_first_node_in_group("player")
    # Collect patrol points from children
    for child in get_children():
        if child is Marker2D:
            patrol_points.append(child.global_position)

func _physics_process(delta: float) -> void:
    if current_state == State.DEAD:
        return

    attack_timer = maxf(attack_timer - delta, 0.0)
    _check_state_transitions()

    match current_state:
        State.PATROL:
            _patrol(delta)
        State.CHASE:
            _chase(delta)
        State.ATTACK:
            _attack(delta)

    move_and_slide()

func _check_state_transitions() -> void:
    if not player:
        return
    var distance = global_position.distance_to(player.global_position)

    if current_state == State.PATROL and distance <= detection_range:
        current_state = State.CHASE
    elif current_state == State.CHASE:
        if distance > detection_range * 1.5:
            current_state = State.PATROL
        elif distance <= attack_range and attack_timer <= 0.0:
            current_state = State.ATTACK
    elif current_state == State.ATTACK and distance > attack_range * 1.2:
        current_state = State.CHASE

func _patrol(_delta: float) -> void:
    if patrol_points.is_empty():
        return
    var target = patrol_points[current_patrol_index]
    var direction = (target - global_position).normalized()
    velocity = direction * move_speed

    if global_position.distance_to(target) < 5.0:
        current_patrol_index = (current_patrol_index + 1) % patrol_points.size()

func _chase(_delta: float) -> void:
    if not player:
        return
    var direction = (player.global_position - global_position).normalized()
    velocity = direction * chase_speed

func _attack(_delta: float) -> void:
    velocity = Vector2.ZERO
    if attack_timer <= 0.0:
        # Deal damage to player
        if player.has_node("HealthComponent"):
            player.get_node("HealthComponent").take_damage(10)
        attack_timer = attack_cooldown
```

## Tween Animations

```gdscript
# Godot 4 uses SceneTree.create_tween()
func _ready() -> void:
    # Fade in
    modulate.a = 0
    var tween = create_tween()
    tween.tween_property(self, "modulate:a", 1.0, 0.5)

    # Bounce effect
    var tween = create_tween().set_trans(Tween.TRANS_BOUNCE).set_ease(Tween.EASE_OUT)
    tween.tween_property(sprite, "scale", Vector2(1.2, 1.2), 0.1)
    tween.tween_property(sprite, "scale", Vector2.ONE, 0.2)

    # Sequence with delay
    var tween = create_tween()
    tween.tween_callback(_show_message)
    tween.tween_interval(1.0)
    tween.tween_callback(_hide_message)

    # Parallel tweens
    var tween = create_tween().set_parallel(true)
    tween.tween_property(panel, "rect_position:x", 200, 0.5)
    tween.tween_property(panel, "modulate:a", 0.5, 0.5)
    tween.set_parallel(false)  # back to sequential

    # Looping tween
    var tween = create_tween().set_loops()
    tween.tween_property(sprite, "scale", Vector2(1.1, 1.1), 0.5)
    tween.tween_property(sprite, "scale", Vector2.ONE, 0.5)
```

## Raycasting

```gdscript
# 2D Raycast for line-of-sight, shooting, interaction
extends CharacterBody2D

@onready var ray_cast_2d: RayCast2D = $RayCast2D

func can_see_target(target_position: Vector2) -> bool:
    ray_cast_2d.target_position = to_local(target_position)
    ray_cast_2d.force_raycast_update()
    return not ray_cast_2d.is_colliding() or ray_cast_2d.get_collider() == target

# Space-separated collision (simple distance check)
func get_targets_in_range(range: float) -> Array[Node2D]:
    var results: Array[Node2D] = []
    var space_state = get_world_2d().direct_space_state
    var query = PhysicsPointQueryParameters2D.new()
    query.position = global_position
    query.radius = range
    query.collide_with_bodies = true
    var intersections = space_state.intersect_point(query)
    for result in intersections:
        results.append(result.get("collider"))
    return results
```

## Audio Manager

```gdscript
# scripts/audio_manager.gd (autoload)
extends Node

@onready var _music_player: AudioStreamPlayer = $MusicPlayer
@onready var _sfx_player: AudioStreamPlayer = $SFXPlayer

var _current_track: String = ""

func play_music(track_path: String, fade_duration: float = 1.0) -> void:
    if _current_track == track_path:
        return
    _current_track = track_path
    var stream = load(track_path) as AudioStream
    _music_player.stream = stream
    _music_player.play()

func stop_music(fade_duration: float = 1.0) -> void:
    _music_player.stop()
    _current_track = ""

func play_sfx(sound_path: String, pitch_var: float = 0.0) -> void:
    var stream = load(sound_path) as AudioStream
    _sfx_player.stream = stream
    if pitch_var > 0:
        _sfx_player.pitch_scale = randf_range(1.0 - pitch_var, 1.0 + pitch_var)
    _sfx_player.play()
```

## Groups

Use node groups for flexible entity management without hard references.

```gdscript
# In editor: select node > Node tab > Groups > add "enemies", "player", "pickups"

# In code:
func _ready() -> void:
    add_to_group("enemies")

# Finding all nodes in a group:
func _on_level_complete() -> void:
    var enemies = get_tree().get_nodes_in_group("enemies")
    for enemy in enemies:
        enemy.queue_free()

# Calling a method on all nodes in a group:
func deal_area_damage(amount: int) -> void:
    get_tree().call_group("enemies", "take_damage", amount)

# Check if node is in group:
func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("player"):
        body.take_damage(10)
```

## Loading Screen

```gdscript
# Simple loading screen pattern
func goto_scene_with_loading(target_scene: String) -> void:
    # Show loading screen
    var loading_screen = preload("res://scenes/ui/loading_screen.tscn").instantiate()
    get_tree().root.add_child(loading_screen)

    # Use ResourceLoader to load in background
    ResourceLoader.load_threaded_request(target_scene)

    # Poll until loaded
    while true:
        var status = ResourceLoader.load_threaded_get_status(target_scene)
        if status == ResourceLoader.THREAD_LOAD_LOADED:
            break
        elif status == ResourceLoader.THREAD_LOAD_FAILED:
            push_error("Failed to load scene: " + target_scene)
            loading_screen.queue_free()
            return
        await get_tree().process_frame  # yield to keep UI responsive

    var packed_scene = ResourceLoader.load_threaded_get(target_scene)
    get_tree().change_scene_to_packed(packed_scene)
    loading_screen.queue_free()
```

## Layer & Mask Reference

Godot uses 32 collision layers. Common convention:

| Layer | Common Use |
|-------|-----------|
| 1 | World (static geometry) |
| 2 | Player |
| 3 | Enemies |
| 4 | Projectiles (player) |
| 5 | Projectiles (enemy) |
| 6 | Pickups / Items |
| 7 | Triggers / Areas |
| 8 | UI Physics Bodies |

Example: Player collides with Layer 1 (world), Layer 3 (enemies), Layer 6 (pickups):
- Collision Layer: set bit 2
- Collision Mask: set bits 1, 3, 6

Enemy projectiles collide with Layer 1 (world) and Layer 2 (player):
- Collision Layer: set bit 5
- Collision Mask: set bits 1, 2

## `.tscn` Scene File Format Rules

When hand-writing or generating `.tscn` files (text-based scene format), follow these strict rules to avoid "Load failed due to missing dependencies" errors:

### External Resources (`ext_resource`)

Every external file reference (scripts, scenes, textures, audio) MUST be declared as an `ext_resource` at the top of the file. You CANNOT use inline paths.

```
[ext_resource type="Script" path="res://scripts/player.gd" id="1"]
[ext_resource type="PackedScene" path="res://scenes/ui/hud.tscn" id="2"]
[ext_resource type="Texture2D" path="res://assets/sprites/player.png" id="3"]
```

### `load_steps` Count

The `[gd_scene load_steps=N]` header must reflect the total number of `ext_resource` + `sub_resource` entries. If you have 2 ext_resources and 1 sub_resource, set `load_steps=3`. Can be omitted if there's only 1 resource total.

### Node References

When attaching a script or instancing a scene, reference the `ext_resource` by `id`:

```
[node name="Player" type="CharacterBody2D"]
script = ExtResource("1")

[node name="HUD" parent="." instance=ExtResource("2")]
```

### Common `ext_resource` Types

| Type | Used For |
|------|----------|
| `Script` | `.gd` script files |
| `PackedScene` | `.tscn` scene files (instancing) |
| `Texture2D` | `.png`, `.jpg`, `.svg` images |
| `AudioStream` | `.ogg`, `.wav` audio files |
| `FontFile` | `.ttf`, `.otf` fonts |
| `StyleBox` | `.tres` theme resources |
| `Material` | `.tres` shader/standard materials |
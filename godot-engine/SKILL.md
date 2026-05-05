---
name: godot-engine
description: Godot Engine game development toolkit for building 2D and 3D games, prototyping game mechanics, scaffolding projects, and debugging Godot 4.x projects. Use for Godot game development, GDScript help, node tree architecture, scene design, input handling, physics, UI, and project organization.
license: MIT
metadata:
  author: VTSTech
  version: "0.0.3"
---

# Godot Engine Development Skill

Activate whenever the user's query involves Godot Engine game development, GDScript programming, scene/node architecture, game prototyping, or any Godot 4.x workflow.

## Trigger Conditions

- Building games with Godot Engine (any version, primarily 4.x)
- GDScript code, syntax, patterns, or debugging
- Node trees, scene design, scene instantiation
- Input handling, physics, collision, navigation
- UI/Control nodes, themes, HUD design
- Project organization and best practices
- Exporting and deploying Godot projects
- GDExtension, C# integration, or shader programming
- Game genre scaffolding (ARPG, FPS, platformer, top-down, etc.)
- Performance optimization, profiling
- Multiplayer, networking, signals

## Godot 4.x Core Architecture

### Key Concepts

Godot is built around four pillars: **Nodes**, **Scenes**, **The Scene Tree**, and **Signals**.

- **Nodes** are the smallest building blocks. Each has a name, editable properties, per-frame update callbacks, and can be extended via scripts. Godot provides hundreds of built-in node types.
- **Scenes** are saved trees of nodes. Once saved, a scene acts like a new node type that can be instanced and nested. A scene always has one root node.
- **The Scene Tree** is the runtime hierarchy of all nodes/scenes in the running game. The engine loads one designated "Main Scene" as the entry point.
- **Signals** implement the observer pattern. Nodes emit signals when events occur (collision, button press, timer timeout). Other nodes connect to signals to react. You can define custom signals.

### The Resource Path

All project assets use `res://` as the project root prefix. For example:
```
res://scenes/player.tscn
res://scripts/player.gd
res://assets/sprites/player_walk.png
res://audio/bgm.ogg
```

User data at runtime uses `user://` (platform-specific persistent directory).

### Scene File Format

Scenes are saved as `.tscn` (text-based, human-readable) or `.scn` (binary). Always prefer `.tscn` for version control.

Project config lives in `project.godot` at the project root.

### Editor Screens

| Screen | Shortcut | Purpose |
|--------|----------|---------|
| 2D | F5 area | 2D games AND user interfaces |
| 3D | F5 area | Meshes, lights, 3D level design |
| Script | F5 area | Code editor with debugger, autocompletion, class reference |
| AssetLib | — | Community assets, plugins, demos |

Press **F1** or **Ctrl+Click** on any class/function name to open the integrated class reference.

## GDScript Essentials (Godot 4.x)

GDScript is Godot's built-in, Python-like, object-oriented language. It is the recommended starting language. Key facts:

- **No garbage collector** — uses reference counting (no GC pauses in-game)
- **Gradual typing** — dynamic by default, optional static type hints
- **Tightest editor integration** — autocomplete for nodes, signals, scene tree
- **Built-in vector/transform types** — Vector2, Vector3, Transform2D, Transform3D, Rect2, etc.

See [references/gdscript.md](references/gdscript.md) for complete GDScript reference including syntax, annotations, built-in types, style guide, and code patterns.

## Common Lifecycle Callbacks

```gdscript
extends Node2D

func _init():
    # Called when the object is first created in memory (before _enter_tree)
    pass

func _enter_tree():
    # Called when node enters the Scene Tree
    pass

func _ready():
    # Called once when node and children are ready (after _enter_tree)
    pass

func _process(delta: float):
    # Called every frame. delta = time since last frame (seconds)
    pass

func _physics_process(delta: float):
    # Called every physics tick (fixed timestep, ~60Hz by default)
    pass

func _input(event: InputEvent):
    # Called for every input event before _unhandled_input
    pass

func _unhandled_input(event: InputEvent):
    # Called for unhandled input events (not consumed by UI)
    pass
```

## Input System

Always use **Input Actions** (defined in Project Settings > Input Map) rather than raw key codes.

```gdscript
# In _process or _physics_process:
var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")

# In _unhandled_input or _input:
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("jump"):
        jump()
```

Common input actions to map: `move_left`, `move_right`, `move_up`, `move_down`, `jump`, `attack`, `interact`, `ui_accept`, `ui_cancel`, `pause`.

## Signals

```gdscript
# Connecting in editor (recommended for static connections)
# Node > Signals tab > Connect

# Connecting in code:
func _ready():
    $Button.pressed.connect(_on_button_pressed)
    timer.timeout.connect(_on_timer_timeout)

# Custom signal:
signal health_changed(new_health: int, max_health: int)

func take_damage(amount: int) -> void:
    health -= amount
    health_changed.emit(health, max_health)

func _on_button_pressed() -> void:
    print("Button was pressed!")
```

## Project Scaffolding Commands

### `/godot new <project-name> [2d|3d]`

Scaffold a new Godot project structure:
```
project-name/
├── project.godot
├── .godot/                    # Auto-generated, gitignored
├── scenes/
│   ├── main.tscn              # Main scene (entry point)
│   └── ui/
├── scripts/
├── assets/
│   ├── sprites/
│   ├── models/
│   ├── audio/
│   └── fonts/
├── resources/
└── addons/                    # Plugins
```

### `/godot add-scene <scene-name> <root-node-type>`

Create a new scene with the specified root node. Example:
- `/godot add-scene player CharacterBody2D` → `scenes/player.tscn`
- `/godot add-scene enemy CharacterBody3D` → `scenes/enemy.tscn`
- `/godot add-scene main_menu Control` → `scenes/ui/main_menu.tscn`

### `/godot add-script <scene-name> <script-name>`

Attach a GDScript to an existing scene's root node. Creates `scripts/<script-name>.gd` that extends the scene's root node type.

## Genre-Specific Node Tree Templates

### 2D Top-Down ARPG (Diablo-style)

```
Game (Node)                          # autoload/singleton
├── World (Node2D)
│   ├── Player (CharacterBody2D)
│   │   ├── Sprite2D
│   │   ├── CollisionShape2D
│   │   ├── CollisionPolygon2D       # hitbox
│   │   ├── Camera2D                 # smooth follow
│   │   ├── InteractionArea (Area2D) # for NPC/item interaction
│   │   ├── HealthComponent (Node)   # reusable health system
│   │   ├── InventoryComponent (Node)
│   │   └── SkillsComponent (Node)
│   ├── Enemies (Node2D)
│   │   └── Enemy (CharacterBody2D)  # instanced scene
│   ├── Items (Node2D)
│   │   └── LootDrop (RigidBody2D)
│   ├── NavigationRegion2D           # pathfinding
│   └── TileMap (Node2D or TileMapLayer)
│       ├── Ground (TileMapLayer)
│       └── Walls (TileMapLayer)
├── UI (CanvasLayer)                 # HUD on top of everything
│   ├── HealthBar (TextureProgressBar or Control)
│   ├── ManaBar
│   ├── SkillBar (HBoxContainer)
│   │   └── SkillSlot (TextureButton) x N
│   ├── Minimap
│   └── Inventory (Control, shown/hidden)
├── DamageNumbers (Node2D)           # floating text
└── MusicPlayer (Node)               # autoload
```

Key patterns:
- Use **NavigationRegion2D** with `NavigationServer2D` for enemy pathfinding
- **TileMapLayer** (Godot 4.3+) replaces the old TileMap layers system
- Use **@export** arrays for inventory/skill data
- State machine for player: Idle, Move, Attack, Cast, Interact, Dead
- Enemy AI: finite state machine (Patrol, Chase, Attack, Flee, Dead)

### 3D First-Person

```
Game (Node)                          # autoload
├── World (Node3D)
│   ├── Environment (WorldEnvironment) # sky, fog, ambient
│   ├── Player (CharacterBody3D)
│   │   ├── Head (Node3D)            # pivot for mouse look
│   │   │   ├── Camera3D
│   │   │   └── RayCast3D            # for interaction/shooting
│   │   ├── CollisionShape3D
│   │   ├── MeshInstance3D           # body mesh (often invisible for FPS)
│   │   └── WeaponMount (Node3D)
│   │       └── Weapon (MeshInstance3D)
│   ├── Level (Node3D)
│   │   ├── StaticBody3D             # floors, walls
│   │   │   ├── CollisionShape3D
│   │   │   └── MeshInstance3D
│   │   └── RigidBody3D              # physics objects
│   └── DirectionalLight3D
├── UI (CanvasLayer)
│   ├── Crosshair (CenterContainer > TextureRect)
│   ├── HUD (VBoxContainer)
│   └── PauseMenu (Control)
└── MusicPlayer (Node)               # autoload
```

Key patterns:
- Mouse look: capture mouse via `Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)`, apply rotation to Head `Node3D` (yaw on Player X-rotation, pitch on Head Y-rotation)
- Movement in `_physics_process`: get input, apply velocity, use `move_and_slide()`
- Jumping: check `is_on_floor()`, apply vertical impulse to `velocity.y`
- Use `Input.mouse_motion` signal for camera rotation
- Sprint/crouch: modify speed multiplier and collision shape

### 2D Platformer

```
Game (Node)
├── Player (CharacterBody2D)
│   ├── AnimatedSprite2D
│   ├── CollisionShape2D
│   ├── Camera2D
│   └── CoyoteTimer (Timer)         # jump grace period
├── Level (Node2D)
│   ├── TileMap (TileMapLayer)
│   │   ├── Ground (TileMapLayer)
│   │   ├── Platforms (TileMapLayer)
│   │   └── Hazards (TileMapLayer)
│   ├── MovingPlatform (AnimatableBody2D)
│   │   ├── Sprite2D
│   │   └── CollisionShape2D
│   └── Checkpoints (Node2D)
├── Collectibles (Node2D)
│   └── Coin (Area2D)
├── Enemies (Node2D)
│   └── Enemy (CharacterBody2D)
└── UI (CanvasLayer)
    ├── ScoreLabel (Label)
    ├── LivesCounter (Label)
    └── GameOver (Control)
```

Key patterns:
- Gravity: `velocity.y += gravity * delta` in `_physics_process`, then `move_and_slide()`
- Jump: set `velocity.y = JUMP_VELOCITY` when `is_on_floor()`
- Wall jump: detect wall via `RayCast2D` or `is_on_wall()`, apply horizontal + vertical impulse
- Coyote time: Timer that starts when leaving ground, allows jumping briefly after walking off edge
- Jump buffering: Timer on jump press, execute jump when landing if buffer active

## Game Development Patterns

See [references/common-patterns.md](references/common-patterns.md) for detailed implementations of:
- State machines (enum-based and resource-based)
- Component pattern (reusable Node scripts)
- Save/Load system (JSON-based, using `FileAccess`)
- Scene management (scene transitions, loading screens)
- Object pooling
- Singletons / Autoloads
- Screen shake
- Damage numbers
- Dialogue systems
- Inventory systems

## Rendering & Visuals

### 2D Rendering
- **Sprite2D** for single frames, **AnimatedSprite2D** for sprite sheet animations
- **SpriteFrames** resource defines animation clips (idle, walk, run, attack, etc.)
- **TileMapLayer** (Godot 4.3+) for tile-based levels with collision
- **Camera2D** with smoothing for follow cameras, set limits for level bounds
- **Parallax2D** for parallax backgrounds (parallax_layer in Godot 4)
- **CanvasLayer** for HUD/UI rendered on top of world

### 3D Rendering
- **Forward+** renderer (default): clustered forward, best for desktop
- **Mobile** renderer: simpler, faster for low-end
- **Compatibility** renderer: OpenGL-based, for web/old hardware
- **WorldEnvironment**: sky, fog, ambient light, SSAO, SSR, bloom
- **DirectionalLight3D** with shadow mapping (PSSM)
- **GPUParticles3D** for particle effects

### Shaders
- Godot uses a GLSL-inspired shader language
- Visual Shader Editor available for node-based shader creation
- Material types: **ShaderMaterial** (custom), **StandardMaterial3D** (PBR)

## Physics

### 2D Physics
- **StaticBody2D**: immovable objects (walls, floors)
- **RigidBody2D**: physics-simulated objects (projectiles, debris)
- **CharacterBody2D**: custom movement via code (players, enemies), use `move_and_slide()` or `move_and_collide()`
- **AnimatableBody2D**: moved by code, pushes other bodies (moving platforms)
- **Area2D**: detection zones (triggers, pickups, damage zones)
- **CollisionShape2D** or **CollisionPolygon2D**: defines the actual shape

### 3D Physics
- Same body types with "3D" suffix: `StaticBody3D`, `RigidBody3D`, `CharacterBody3D`, `AnimatableBody3D`, `Area3D`
- Use **collision layers and masks** to filter what collides with what (up to 32 layers)

## Exporting

Configure export presets in Project > Export. Common targets:
- **Windows Desktop**: .exe
- **Linux/X11**: executable
- **macOS**: .app bundle
- **Android**: .apk / .aab (requires Android SDK)
- **iOS**: Xcode project (requires macOS)
- **Web**: HTML5 + WebGL/WebGPU

Use one-click export or command line:
```bash
godot --headless --export-release "Windows Desktop" build/game.exe
```

## Useful Autoload Patterns

Register autoloads in Project Settings > Autoload. Common singletons:
- `GameManager` — global game state, score, settings
- `AudioManager` — music/SFX bus control, pooling
- `SaveManager` — save/load to disk
- `SceneManager` — scene transitions with loading screens
- `UIManager` — show/hide common UI panels
- `DebugManager` — dev tools, debug overlays

## Editor Tips

- **Ctrl+S**: Save current scene
- **F5**: Run project (plays main scene)
- **F6**: Run current scene only
- **F1**: Open class reference for selected node
- **Ctrl+D**: Duplicate selected nodes
- **Ctrl+Z/Ctrl+Y**: Undo/Redo
- **Ctrl+F**: Find in scene tree
- **Q/W/E/R**: Select/Move/Rotate/Scale tools (3D viewport)
- **Middle-click drag**: Pan viewport
- **Scroll wheel**: Zoom viewport
- **Ctrl+Click** on a node name in code to jump to its script

## File Editing Rules — CRITICAL

When generating or modifying Godot project files, follow these rules to avoid corruption:

### GDScript Files (.gd) — TAB INDENTATION REQUIRED

- **NEVER use text editing tools that convert tabs to spaces on `.gd` files.** These tools silently convert TAB characters to spaces, which breaks GDScript parsing entirely.
- **Always use file writing tools** to create or rewrite `.gd` files from scratch.
- For single-line fixes, use **Bash `sed`** and verify with `cat -A` that indentation uses `^I` (tabs), not spaces.
- After any batch operation on `.gd` files, scan for space-indented lines: `rg '^[ ]' scripts/ -g '*.gd'`

### Scene Files (.tscn) — Strict Format Requirements

- Use `ext_resource` declarations for all external references (scripts, scenes, textures). Never use inline paths like `PackedScene("res://...")`.
- `load_steps` must equal the total count of `ext_resource` + `sub_resource` entries (can be omitted if only 1).
- Node references use `id` and `type` — `PackedScene` for scene instances, `Script` for script attachments.
- Every node's `parent` must reference an existing node path in the file.

### Syntax Pitfalls

- **No `Type?` nullable suffix** on function return types — GDScript object types are nullable by default. `-> ItemResource?` causes a parse error. Use `-> ItemResource` instead.
- **Always use `class_name`** when scripts reference each other's types (e.g., `PlayerState` referencing `Player`).
- **Unused parameters must be prefixed with underscore** — Godot will warn about unused parameters in function signatures (e.g., `_delta` instead of `delta`).
- **Avoid shadowing built-in function names** — Variables named `range`, `process`, `input`, etc. can cause conflicts (use `skill_range`, `_process`, etc.).
- **Narrowing conversions cause warnings** — When dividing integers to get floats, explicitly cast to `float()` (e.g., `float(value) / float(max_value)`).

## Workflow for Helping Users

1. **Understand the scope**: What genre, 2D or 3D, singleplayer or multiplayer?
2. **Scaffold the project**: Set up the directory structure and main scene.
3. **Build core systems first**: Player controller, camera, basic movement.
4. **Iterate**: Add features one at a time (collision, enemies, UI, etc.).
5. **Always explain**: Tell the user WHY a particular node tree or pattern is used, not just WHAT to type.
6. **Reference Godot docs**: When uncertain, check `https://docs.godotengine.org/en/stable/` or use the built-in class reference.

## Self-Update Protocol (Living Skill)

This skill is designed to **grow smarter over time**. When the agent learns something new about Godot — from docs, from fixing bugs, from user corrections, or from solving novel problems — it MUST update the skill files.

### When to Update

- **Correcting mistakes**: If code you wrote was wrong and you found the fix, update the relevant reference file.
- **New patterns discovered**: If you write a novel solution that would be useful again, add it to `references/common-patterns.md`.
- **Godot version changes**: New APIs, deprecated methods, or behavioral changes between Godot versions.
- **User corrections**: If the user teaches you something or corrects your approach, record it.
- **Deep doc dives**: When you fetch and learn from Godot docs to solve a problem, distill what you learned into the skill.
- **Genre templates**: If you scaffold a new game genre not covered (RTS, visual novel, roguelike, etc.), add the node tree template.
- **Gotchas and pitfalls**: Non-obvious bugs, breaking changes, platform-specific issues.

### How to Update

1. **Read the target file first** — use the Read tool before editing to avoid overwriting.
2. **Choose the right file**:
   - `SKILL.md` — only for core concepts, trigger conditions, genre templates, scaffolding commands.
   - `references/gdscript.md` — syntax, types, annotations, code patterns, style guide.
   - `references/common-patterns.md` — reusable game architecture, component patterns, systems.
   - `references/learnings.md` — new discoveries, gotchas, corrections, version notes.
3. **Edit surgically** — use precise editing tools to add/update specific sections, never rewrite entire files.
4. **Bump version** — increment `metadata.version` in SKILL.md frontmatter.
5. **Log the update** — append an entry to `references/learnings.md` with date, what changed, and why.

### What NOT to Update

- Do not bloat SKILL.md beyond ~500 lines — keep it focused on architecture and scaffolding.
- Do not add platform-specific workarounds without confirming they are current.
- Do not remove content unless it is confirmed deprecated in the target Godot version.
- Do not add patterns you haven't validated — untested code is worse than no code.

### `/godot learn <topic>`

When the user explicitly asks to learn or document a topic, fetch the relevant Godot docs, study them, and update the skill. Steps:

1. Fetch docs from `https://docs.godotengine.org/en/stable/` for the topic.
2. Extract key APIs, patterns, and gotchas.
3. Update the appropriate reference file.
4. Append a summary to `references/learnings.md`.
5. Bump version.
6. Report back to the user what was learned and where it was saved.

### Version History

| Version | Date | Summary |
|---------|------|--------|
| 0.0.1 | 2026-05-04 | Initial skill: core architecture, GDScript reference, 3 genre templates, common patterns |
| 0.0.2 | 2026-05-05 | Added: File Editing Rules (.gd tab requirement, .tscn format), nullable type gotcha, nullable type reference section |
| 0.0.3 | 2026-05-05 | Updated: Pi compatibility - tool references, workspace paths, GDScript warnings, scene debugging |

## Documentation Reference

- Official docs: https://docs.godotengine.org/en/stable/
- Class reference: https://docs.godotengine.org/en/stable/classes/
- GDScript reference: https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html
- Godot Q&A: https://godotengine.org/qa/
- Godot Asset Library: https://godotengine.org/asset-library/

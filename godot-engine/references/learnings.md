# Godot Engine Skill — Learnings Log

This file tracks everything the agent learns after initial skill creation. Each entry records what was discovered, why, and where it was saved.

---

## 2026-05-04 — v0.0.1 Initial Creation

**Source**: Godot Engine 4.6 official docs + existing agent knowledge
**Changes**: Created initial skill with 3 files:
- `SKILL.md`: Core architecture, lifecycle callbacks, input system, signals, scaffolding commands, 3 genre templates (ARPG, FPS, platformer), rendering, physics, exporting
- `references/gdscript.md`: Complete GDScript 4.x reference — types, control flow, exports, signals, node refs, await/coroutines, style guide, 3 movement implementations, animation state machine
- `references/common-patterns.md`: Autoloads, component pattern, resource-based data, FSM, save/load, scene transitions, object pooling, screen shake, damage numbers, dialogue system, enemy AI, tweens, raycasting, audio, groups, collision layers

---

## 2026-05-05 — v0.0.2 ARPG Scaffold Hardening

**Source**: Building a 2D top-down ARPG project in Godot 4.6.2 with user VTSTech. Three rounds of debugging to get the project to load cleanly.

### Learning 1: Nullable return types (`Type?`) break GDScript 4.6.2 parser
- **Problem**: `func get_item() -> ItemResource?:` produces parse error: *"Expected end of statement after bodyless function declaration, found '?' instead."*
- **Root cause**: Godot 4.6.2 GDScript parser does not support the `?` nullable suffix on function return type declarations. It misidentifies the function as "bodyless" and fails.
- **Fix**: Remove the `?` — GDScript object types are nullable by default. Use `-> ItemResource:` instead of `-> ItemResource?:`. Functions can still `return null`.
- **Updated**: `references/gdscript.md` — added note in Types section about nullable types.

### Learning 2: Edit/MultiEdit tools corrupt GDScript indentation
- **Problem**: Using the agent's Edit or MultiEdit tools on `.gd` files silently converts TAB characters to spaces.
- **Root cause**: The Edit/MultiEdit tools normalize whitespace, but GDScript **requires** tab indentation. Even a single space-indented line causes parse failures.
- **Impact**: Cascading parse errors across all scripts that reference the corrupted script (e.g., a broken `InventoryComponent` causes `Player`, `HUD`, `InventoryUI` to all fail).
- **Fix**: NEVER use Edit/MultiEdit on `.gd` files. Always use the **Write** tool to rewrite the entire file, or use `Bash sed` for surgical single-line changes. After any `sed` operation, verify with `cat -A` that indentation uses `^I` (tabs) not spaces.
- **Updated**: `SKILL.md` — added to "File Editing Rules" section.

### Learning 3: `.tscn` scene format has strict requirements
- **Problem**: Hand-written `.tscn` files caused "Load failed due to missing dependencies" errors.
- **Root cause**: Several format issues:
  - Cannot use inline `PackedScene("res://...")` references — must use `ext_resource` declarations at the top.
  - `load_steps` count must exactly match the number of `ext_resource` entries (external resources) plus `sub_resource` entries, minus 1 (since load_steps starts at 1). If `load_steps = 3`, there should be 2 ext_resource lines and/or sub_resource lines.
  - Node `[ext_resource ...]` references must use the correct `id` and `type` — `PackedScene` for scene instances, `Script` for script attachments.
  - Every node must have a parent reference that matches an existing node path in the file.
- **Fix**: Follow a strict template when writing `.tscn` files. Always count ext_resources + sub_resources and set `load_steps` accordingly.
- **Updated**: `references/common-patterns.md` — added `.tscn` format rules section.

## 2026-05-05 — v0.0.3 Pi Compatibility & Scene Debugging

**Source**: Debugging ARPG project loading issues in Pi environment with user VTSTech.

### Learning 4: GDScript unused parameter warnings
- **Problem**: Godot 4.6.2 issues warnings for unused function parameters: *"The parameter 'delta' is never used in the function '_physics_process()'"*
- **Root cause**: Godot's GDScript compiler strictly enforces that unused parameters must be prefixed with underscore to avoid cluttering console output.
- **Fix**: Prefix unused parameters with underscore (e.g., `_delta` instead of `delta`, `_new_health` instead of `new_health`).
- **Impact**: Clean console output and follows Godot best practices.
- **Updated**: `SKILL.md` — added to "Syntax Pitfalls" section.

### Learning 5: Avoid shadowing built-in function names
- **Problem**: Variable named `range` causes warning: *"The variable 'range' has the same name as a built-in function."*
- **Root cause**: Godot has built-in functions like `range()`, `process()`, `input()`. Using these names for variables creates conflicts.
- **Fix**: Rename variables to avoid conflicts (e.g., `skill_range` instead of `range`, `_process` instead of `process`).
- **Impact**: Prevents runtime errors and follows Godot naming conventions.
- **Updated**: `SKILL.md` — added to "Syntax Pitfalls" section.

### Learning 6: Narrowing conversion warnings
- **Problem**: Division of integers causes warning: *"Narrowing conversion (float is converted to int and loses precision)."*
- **Root cause**: When dividing integers to get a float result, Godot warns about potential precision loss.
- **Fix**: Explicitly cast to float: `float(current_mana) / float(max_mana)` instead of `current_mana / max_mana`.
- **Impact**: Cleaner console output and explicit about float intentions.
- **Updated**: `SKILL.md` — added to "Syntax Pitfalls" section.

### Learning 7: Scene instantiation best practices
- **Problem**: Complex scene hierarchies fail to load with errors like *"Parent path './StateMachine' for node 'Idle' has vanished when instantiating"*
- **Root cause**: Complex nested dependencies and timing issues during scene loading.
- **Fix**: Start with minimal scenes and gradually add complexity. Use `await get_tree().process_frame` to wait for scene tree readiness. Check node existence with `has_node()` before access.
- **Impact**: More reliable scene loading and easier debugging.
- **Updated**: `SKILL.md` — updated "File Editing Rules" with more accurate tool descriptions.

### Learning 8: Simple CharacterBody2D movement pattern
- **Problem**: Player character "leans" instead of moving properly.
- **Root cause**: Incorrect velocity handling and missing direction normalization.
- **Fix**: Use `velocity = direction.normalized() * speed` and ensure `velocity = Vector2.ZERO` when no input.
- **Impact**: Proper player movement following Godot documentation.
- **Updated**: `references/common-patterns.md` — added basic movement pattern.

**Known gaps to fill on demand**:
- Multiplayer / networking (ENet, WebRTC, high-level multiplayer API)
- GDExtension (C/C++ integration)
- Shader language deep dive
- TileMapLayer advanced usage (auto-tiling, terrain sets)
- NavigationAgent2D/3D pathfinding details
- GUI theming deep dive
- Godot 4.3+ specific features (TileMapLayer migration from TileMap)
- Mobile-specific patterns (touch input, screen scaling)
- Console export specifics
- Advanced debugging techniques for complex scenes
- Performance optimization patterns for 2D/3D games

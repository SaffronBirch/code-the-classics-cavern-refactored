# DESIGN.md

## Overview

This submission implements the required architectural changes **within a single file (`cavern.py`)** instead of splitting into multiple modules. The file is organized into logical sections for input handling, screen/state management, gameplay entities, and rendering helpers.

## Screens architecture

A lightweight **State Pattern** is used for high-level game flow.

### Components

- **`App`**: context object that owns:
  - `current_state` (active screen)
  - `game` (current `Game` instance)
  - `input` (`InputManager`)
- **`GameState` (abstract base class)**: defines `update(input_state)` and `draw()`
- **Concrete screens**:
  - `MenuScreen`
  - `PlayScreen`
  - `GameOverScreen`

### Flow

- Pygame Zero calls global `update()` / `draw()`
- Those delegate to `app.update()` / `app.draw()`
- `App` delegates to the current screen object
- Screens are responsible for transitions using `app.change_state(...)`

This removes menu/game-over branching from the top-level loop and makes screen behavior explicit.

## Input design

Input handling uses an **input snapshot** (command-style data object) built once per frame.

### `InputState` (dataclass)

`InputState` stores the player-relevant inputs for the current frame:

- `left`, `right`
- `jump_pressed` (edge)
- `fire_pressed` (edge)
- `fire_held` (level)
- `pause_pressed` (edge)

### `InputManager`

`InputManager.build()` reads keyboard state once per frame and performs edge detection using previous-frame values:

- `prev_space`
- `prev_up`
- `prev_pause`

This supports “pressed this frame” behavior for actions that should only trigger once per key press (menu start, firing an orb, pause toggle).

### Result

`Player.update(...)` no longer reads `keyboard.*` directly and instead consumes `InputState`, improving separation of concerns and making input behavior easier to reason about.

## How Pause works

Pause is implemented inside **`PlayScreen`**.

### Behavior

- Pressing `P` sets `input_state.pause_pressed = True` for one frame (edge-triggered)
- `PlayScreen.update(...)` toggles `self.paused`
- If `self.paused` is `True`, `PlayScreen.update(...)` returns early before calling `self.app.game.update(...)`

### Effect

Because `Game.update(...)` is skipped while paused:

- movement stops
- spawns stop
- timers stop
- simulation state is frozen

### Rendering while paused

`PlayScreen.draw()` still calls `self.app.game.draw()` to show the current scene, then draws a **semi-transparent gray overlay** on top. This satisfies the requirement that the game remains visible while paused.

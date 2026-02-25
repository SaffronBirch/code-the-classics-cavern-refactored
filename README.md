# Cavern (Single-File Refactor)

This project is a Pygame Zero version of **Cavern** with the required architectural changes implemented **inside a single `cavern.py` file**.

## How to run the game

### Requirements

- Python 3.5+
- Pygame Zero 1.2+
- Pygame

### Run command

From the project folder (the folder containing `cavern.py`):

```bash
python3 cavern.py
```

## How to run tests

Use the following manual checks:

### Manual test checklist

1. **Menu start**
   - Launch game
   - Press and hold `SPACE`
   - Game should start once (not repeatedly retrigger while held)

2. **Player Input**
   - In play mode, verify movement (`LEFT/RIGHT`), jump (`UP`), and orb actions (`SPACE`) still work

3. **Orb Firing**
   - Press `SPACE` once to create an orb
   - Holding `SPACE` should not spam-create new orbs every frame
   - Holding `SPACE` should extend the currently controlled orb blow distance

4. **Pause toggle**
   - Press `P` during play to pause
   - Confirm enemies/projectiles/timers stop updating
   - Confirm current scene remains visible with gray overlay
   - Press `P` again to resume

5. **Game over**
   - Reach game over state and press `SPACE`
   - Confirm return to menu

## Short summary of architectural changes

Although everything remains in **one file (`cavern.py`)**, the code was reorganized logically to satisfy the design tasks:

### 1) Screens / State pattern (Task A)

Added screen objects and a context class:

- `App` (context)
- `GameState` (abstract base class)
- `MenuScreen`
- `PlayScreen`
- `GameOverScreen`

The top-level `update()` / `draw()` functions now delegate to `app.update()` and `app.draw()`, and `App` delegates to the active screen object.

### 2) Input snapshot + edge detection (Task B)

Removed direct keyboard reads from `Player.update(...)` and introduced:

- `InputState` dataclass
- `InputManager`

`InputManager.build()` reads keyboard input **once per frame** and computes edge-triggered inputs:

- `jump_pressed`
- `fire_pressed`
- `pause_pressed`

`Player.update(input_state)` now consumes the snapshot instead of reading `keyboard.*` directly.

### 3) Pause mode (Task C)

Pause is implemented in `PlayScreen` using a `paused` boolean:

- `P` toggles pause using `pause_pressed` edge detection
- While paused, `PlayScreen.update(...)` returns early (freezing simulation)
- `PlayScreen.draw()` still renders the current game scene and draws a semi-transparent gray overlay

## Notes

- This submission intentionally keeps the code in a single file for simplicity and to avoid issues such as circular imports while still implementing the required architectural concepts.

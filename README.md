# HollowEngine

A small 3D engine with a built-in editor, written in C++17 on top of SDL2 and Dear ImGui.
It provides a scene graph, keyframe animation, Lua scripting, OBJ model import, a play/stop workflow,
and a software rasterizer built on `SDL_RenderGeometry`.

*Português: see [README.pt-BR.md](README.pt-BR.md).*

## Features

- **Scene graph** – entities with optional components (mesh, light, script, animator) and parent/child hierarchy.
- **Editor** – hierarchy, inspector, animation timeline, console, move/scale/rotate gizmos, ray-based picking, drag-and-drop reparenting.
- **Animation** – per-entity clips with position/rotation/scale tracks; linear, step and smooth interpolation; in-editor recording and preview.
- **Lua scripting** – per-entity scripts with `on_start(self)` / `on_update(self, dt)`, sandboxed (no `io`/`os`), with an instruction budget to stop runaway loops.
- **Model import** – Wavefront `.obj` (vertices, UVs, n-gons, negative indices). Models are centered and normalized to one unit.
- **Play mode** – Play snapshots the scene; Stop restores it, so nothing from a play session is saved by accident.
- **Scene files** – human-readable `.hscene` text format, friendly to version control.
- **Rendering** – back-face culling, per-triangle lighting (ambient, point and directional lights), near-plane clipping, global depth sorting, textured batches.

## Dependencies

| Library | Purpose |
|---|---|
| SDL2 | Window, input, 2D/geometry rendering |
| SDL2_image | PNG/JPG texture loading |
| Lua 5.4 | Scripting |
| Dear ImGui (≥ 1.89.5) | Editor UI (needs the `sdl2` and `sdlrenderer2` backends) |

No other libraries or external programs are required.

## Installation

### 1. Install system packages

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install build-essential pkg-config git libsdl2-dev libsdl2-image-dev liblua5.4-dev
```

**Fedora**
```bash
sudo dnf install gcc-c++ make pkgconf-pkg-config git SDL2-devel SDL2_image-devel lua-devel
```

**Arch Linux**
```bash
sudo pacman -S base-devel git sdl2 sdl2_image lua
```

**macOS (Homebrew)** *(not tested)*
```bash
brew install sdl2 sdl2_image lua pkg-config git
```

**Windows (MSYS2, MinGW64 shell)** *(not tested)*
```bash
pacman -S --needed mingw-w64-x86_64-toolchain mingw-w64-x86_64-SDL2 mingw-w64-x86_64-SDL2_image \
          mingw-w64-x86_64-lua mingw-w64-x86_64-pkg-config make git
```

### 2. Get Dear ImGui

From the project root (the folder containing the `Makefile`), put ImGui in a folder named `imgui`:

```bash
git clone --depth 1 https://github.com/ocornut/imgui.git imgui
```

If you already have an `imgui/` folder from another project, copy it here instead. The build expects
`imgui/imgui.cpp` and `imgui/backends/imgui_impl_sdl2.cpp` / `imgui_impl_sdlrenderer2.cpp`.

### 3. Build and run

```bash
make            # builds ./hollow
./hollow        # or: make run
```

The Makefile looks for the Lua package named `lua5.4` by default. If your system names it differently
(Fedora, Arch, Homebrew, MSYS2), pass it explicitly:

```bash
make LUA_PKG=lua
```

Run the core test suite (needs no SDL, ImGui or Lua):

```bash
make test
```

Always launch `./hollow` from the project root so relative paths (`scripts/`, `scenes/`) resolve.

### Troubleshooting

| Symptom | Fix |
|---|---|
| `lua5.4.pc not found` | Use `make LUA_PKG=lua` (or the name shown by `pkg-config --list-all \| grep -i lua`). |
| `imgui/imgui.h: No such file` | ImGui is missing or in the wrong folder (step 2). |
| `imgui_impl_sdlrenderer2.h` missing | Your ImGui is older than 1.89.5; update it. |
| `sdl2-config: command not found` | Install the SDL2 development package (step 1). |
| Black or empty viewport | Make sure the Viewport window is open (`Windows` menu) and the scene has objects. |

## Project layout

| Path | Role |
|---|---|
| `src/math3d.h`, `camera.h` | Vectors, 4x4 matrices, camera, projection, ray casting |
| `src/mesh.*` | Procedural primitives and the OBJ importer |
| `src/animation.*` | Clips, keyframe tracks, animator |
| `src/scene.*` | Entities, hierarchy, `.hscene` save/load |
| `src/renderer.*` | Software rasterizer on `SDL_RenderGeometry` |
| `src/audio.*` | Audio mixer (WAV) on top of SDL2 |
| `src/script.*` | Lua integration |
| `src/editor.*` | ImGui panels, gizmos, picking, timeline, play/stop |
| `src/main.cpp` | Window and main loop |
| `scripts/` | Example Lua scripts |
| `tests/` | Headless tests for the core modules |

The core modules (`math3d`, `mesh`, `animation`, `scene`) have no SDL, ImGui or Lua dependency.

## Using the editor

**Viewport:** hold the right mouse button (or Shift) to look around; `W A S D` to move; `Space` / `Ctrl` to go up / down;
mouse wheel to zoom. Click an object to select it. Use the toolbar to switch between Move, Scale and Rotate gizmos.

**Hierarchy:** drag an object onto another to parent it; drop on the empty area to unparent. Right-click to duplicate or delete (`Delete` also works).

**Inspector:** transform, mesh, color, texture, light, script and animator settings.

**Animation:**
1. Select an object and click *New clip*.
2. Pick a time on the timeline, pose the object, and click *Record keyframe* for position, rotation, scale, or all.
3. Drag across the timeline to scrub, *Preview* to play, *Restore pose* to undo the preview.
4. Set *Play on start* in the Inspector's Animator section to autoplay the clip in Play mode.

**Importing models:** `Entity > Import model (.obj)...`, or choose the "Modelo" mesh in the Inspector and set the file.
Assign a texture in the *Textura* field (`.mtl` files are ignored). Models above roughly 20,000 triangles are slow in a software rasterizer.

**Play mode:** *Play* runs animations and Lua scripts; *Stop* restores the scene exactly as it was.
Mouse input is available to scripts only while the pointer is over the Viewport; `Esc` releases a locked mouse.

A directional light points along its object's forward axis (−Z); rotate the object to aim it.

## Lua API

Attach a `.lua` file to an entity in the Inspector. Both callbacks are optional.
Globals defined in a script are private to that entity.

```lua
function on_start(self)
  log("hello from " .. self:name())
end

function on_update(self, dt)
  self:rotate(0, 90 * dt, 0)
  if input.key_down("Space") then self:translate(0, 5 * dt, 0) end
end
```

**Entity (`self` or any object returned by `scene`)**

| Method | Description |
|---|---|
| `get_position()`, `set_position(x,y,z)`, `translate(dx,dy,dz)` | Local position |
| `get_rotation()`, `set_rotation(x,y,z)`, `rotate(dx,dy,dz)` | Local rotation in degrees |
| `get_scale()`, `set_scale(s)` / `set_scale(x,y,z)` | Local scale |
| `get_world_position()`, `forward()` | World-space position and forward (−Z) direction |
| `set_color(r,g,b)` | Color, components 0–1 |
| `set_texture(path)`, `set_model(path)` | Assign a texture or an `.obj` model |
| `set_active(bool)`, `is_active()`, `name()` | State and name |
| `set_intensity(v)` | Light intensity |
| `play(clip)`, `stop_anim()` | Control animation (`play` returns `false` if the clip is unknown) |
| `destroy()` | Removed at the end of the frame |

**`scene`:** `find(name)` (object or `nil`), `spawn("cube"|"plane"|"cone"|"sphere"|"empty", x, y, z)`, `overlaps(a, b)`, `count()`.

**`camera`** (the game camera): `get_position()`, `set_position(x,y,z)`, `get_rotation()`, `set_rotation(yaw,pitch)`, `forward()`, `right()`.

**`input`:** `key_down(name)`, `key_pressed(name)`, `key_released(name)` using SDL key names (`"W"`, `"Space"`, `"Left Shift"`, `"Up"`, `"Return"`);
`mouse_down(1|2|3)`, `mouse_delta()`, `mouse_position()`, `screen_size()`, `lock_mouse(bool)`.

**`audio`:** `play("audio/x.wav" [, volume=1 [, loop=false [, pan=0]]])` returns a voice id (`nil` on failure), `play_at(path, x,y,z [, volume [, loop]])`,
`stop(id)`, `stop_all()`, `is_playing(id)`, `set_volume(id, v)`, `set_pan(id, p)` (-1 left .. 1 right), `master_volume([v])`.
Entities also have `self:play_sound(path [, volume [, loop]])`, played at the object's position.
Built-in SDL2 mixer, no new dependency. **WAV only** (whatever `SDL_LoadWAV` accepts; 16-bit PCM is safest), any rate/mono/stereo (converted on load), up to 32 simultaneous sounds.
Put sound files in an `audio/` folder next to the Makefile: it is copied whole by `File > Export game...`. Sounds stop when you press Stop.
`play_at`/`play_sound` compute distance attenuation and pan once, when the sound starts. Example: `scripts/som_exemplo.lua`.

**`time`:** `delta()`, `elapsed()`. **`log(...)`** prints to the editor console. Available standard libraries: `math`, `string`, `table`.

A script that raises an error is disabled for the rest of the session and the message appears in the Console.

## Scene file format

`.hscene` files are plain text, one instruction per line (`entity`, `name`, `parent`, `transform`, `mesh`, `model`, `color`,
`texture`, `light`, `script`, `clip`, `track`, `key`, `end`). Unknown lines are ignored, so files stay forward-compatible.

## Known limitations

- Rendering uses painter's-algorithm sorting, so interpenetrating objects can show small artifacts.
- Textures are affine-mapped (no perspective correction); large faces are subdivided to reduce distortion.
- No physics beyond `scene.overlaps` (axis-aligned boxes), and audio is WAV only (no OGG/MP3).
- `.mtl` materials, normals and animations in OBJ files are not imported.
- Windows and macOS builds have not been tested.

## Exporting a game (.exe / .apk)

`File > Export game...` copies your scene and everything it references into `<folder>/data`. The `hollow_player` runtime
(`make player`, no editor or ImGui) loads `data/game.hscene` and runs it. On Windows, `tools/package_windows.sh` bundles the
`.exe` with its DLLs; Android uses the SDL Gradle template (files in `android/`). Full steps and caveats in
[docs/EXPORT.md](docs/EXPORT.md). Android and Windows builds are not tested; the `.exe`/`.apk` must be built on your machine.
Scripts can read pointer/touch input with `input.mouse_down(1)`, `input.mouse_position()` and `input.screen_size()`.

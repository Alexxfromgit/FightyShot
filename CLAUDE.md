# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**FightyShot** is a Unity 5.6.4f1 wave-based tower defense / roguelike shooter game. Players fight procedurally generated maps against increasingly difficult enemy waves. Written in C# targeting .NET Framework 3.5.

## Build & Development

This project is built and run through the **Unity Editor** — there is no standalone CLI build command.

- Open `FightyShot.sln` in JetBrains Rider or Visual Studio to edit scripts
- Open the project folder in Unity 5.6.4f1 to play, test, and build
- Use the Unity Editor Play button to run the game in-editor
- The main gameplay scene is at `Assets/Scenes/Game/`

**Dev mode**: Set `devMode = true` on the `Spawner` component in the scene inspector. In dev mode, press **Enter** to skip to the next wave and clear all current enemies.

**Map preview**: Select a `MapGenerator` in the editor — the custom inspector shows a "Generate" button to preview map generation without entering Play mode (`Assets/Editor/MapEditor.cs`).

## Architecture

### Core Inheritance Chain

```
LivingEntity (base: health, death, IDamageable)
├── Player
└── Enemy
```

`IDamageable` (`TakeHit` / `TakeDamage`) is the contract used by `Projectile` to apply damage without coupling to specific entity types.

### Game Loop

1. **`Spawner`** manages wave progression — reads `Wave[]` config, spawns enemies at timed intervals, fires `OnNewWave` event when a wave starts
2. **`MapGenerator`** regenerates the procedural tilemap each wave (seed-based, flood-fill validated for accessibility), rebuilds the NavMesh
3. **`Enemy`** uses a state machine (Idle → Chasing → Attacking) with Unity NavMesh for pathfinding
4. **`Player`** handles WASD movement (Rigidbody physics) + mouse aim (raycast to ground plane) + shooting input
5. **`GunController`** bridges `Player` input to the active `Gun`; guns can be swapped per wave via `OnNewWave`
6. **`Gun`** fires `Projectile` prefabs, manages magazine/reload coroutine, drives `MuzzleFlash` and `Shell` effects
7. **`Projectile`** uses raycast (not trigger colliders) to detect hits and call `IDamageable.TakeHit()`

### Key Systems

| System | Primary Files |
|--------|--------------|
| Wave management & camping detection | `Spawner.cs` |
| Procedural map generation | `MapGenerator.cs` |
| Weapon logic (fire modes, reload, recoil) | `Gun.cs` |
| Enemy AI state machine | `Enemy.cs` |
| Audio (singleton, spatial SFX, crossfade music) | `AudioManager.cs`, `SoundLibrary.cs`, `MusicManager.cs` |
| HUD & wave banners | `GameUI.cs` |

### Events (Observer Pattern)

- `Spawner.OnNewWave` — fired when a new wave begins; `GunController` and `GameUI` subscribe
- `LivingEntity.OnDeath` — fired when any entity dies; `GameUI` subscribes to `Player`'s death

### Camping Detection

`Spawner` tracks player position over time. If the player moves less than 1.5 m over 2 seconds, enemies spawn directly at the player's location to discourage camping.

### Wave Configuration

Each wave is a `Wave` struct with: enemy count, spawn rate, enemy speed/health/damage/color, and an optional `infinite` flag for endgame endless spawning. The `MapGenerator` can also switch to a different map config per wave.

### Audio

`AudioManager` is a persistent singleton (survives scene loads). `SoundLibrary` groups audio clips by name and picks randomly from each group. Music crossfades between two `AudioSource` components.

## Scripts Directory Structure

All game logic lives in `Assets/Scripts/`. Editor tooling is in `Assets/Editor/`. Prefabs in `Assets/Prefabs/Weapons/`. Audio assets in `Assets/Audio/` (organized by category: Enemy, Guns, Impacts, Music, Player).

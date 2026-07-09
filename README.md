# Space Battle

A 2D top-down space shooter built in Unity. You pick a weapon, fly through a portal into the battlefield, and survive escalating waves of enemy ships.

Built with **Unity 2022.3.50f1** (URP, 2D feature set) as an object-oriented programming coursework project.

## Gameplay

The game runs across two scenes:

- **ChooseWeapon** — a weapon rack with four weapons. Fly into one to equip it, which activates a roaming portal. Enter the portal to start the battle.
- **Main** — the combat arena. Enemies spawn in waves; the HUD tracks health, enemies remaining, points, and wave number.

### Controls

| Input                         | Action                                  |
| ----------------------------- | --------------------------------------- |
| `W` `A` `S` `D` / arrow keys  | Move                                    |
| `Fire1` (left mouse / `Ctrl`) | Shoot                                   |
| `1`–`4` then right-click      | Debug: spawn the selected enemy variant |

Player movement uses acceleration and friction rather than instant velocity, so the ship drifts to a stop. The ship is clamped to the camera bounds.

## Enemies

Every enemy derives from [`Enemy`](Assets/Scripts/Enemies/Enemy.cs) and carries a `level` that gates when it starts appearing and how many points a kill is worth (`kills × level`).

| Enemy                                                          | Level | Behavior                                                                       |
| -------------------------------------------------------------- | ----- | ------------------------------------------------------------------------------ |
| [`EnemyForward`](Assets/Scripts/Enemies/EnemyForward.cs)       | 1     | Spawns off the top or bottom edge and flies straight, bouncing at the far edge |
| [`EnemyHorizontal`](Assets/Scripts/Enemies/EnemyHorizontal.cs) | 1     | Same, but along the left/right axis                                            |
| [`EnemyTargeting`](Assets/Scripts/Enemies/EnemyTargeting.cs)   | 3     | Homes in on the player and destroys itself on contact                          |
| [`EnemyBoss`](Assets/Scripts/Enemies/EnemyBoss.cs)             | 4     | Patrols horizontally and fires pooled bullets downward                         |

## Wave system

[`CombatManager`](Assets/Scripts/System/CombatManager.cs) owns the wave loop. When `totalEnemies` reaches zero it disables every spawner, waits out `waveInterval`, increments the wave number, then re-enables each spawner whose enemy `level` is at or below the current wave. So new enemy types unlock as waves progress.

Each [`EnemySpawner`](Assets/Scripts/Enemies/EnemySpawner.cs) drips out its `spawnCount` on a `spawnInterval` timer. Once it has racked up `minimumKillsToIncreaseSpawnCount` kills within a wave, it grows its default spawn count by a multiplier that itself increases each time — the pressure compounds.

## Combat model

Damage flows through small, composable components rather than living on the ships themselves:

- [`AttackComponent`](Assets/Scripts/Components/AttackComponent.cs) — sits on anything that deals damage (a bullet, or an enemy hull). On trigger, it skips same-tag collisions and forwards damage to the other object's hitbox.
- [`HitboxComponent`](Assets/Scripts/Components/HitboxComponent.cs) — receives damage and passes it to health, unless invincibility is active.
- [`HealthComponent`](Assets/Scripts/Components/HealthComponent.cs) — tracks HP; on death, credits the matching spawner with a kill, decrements the arena's enemy count, and destroys the object.
- [`InvincibilityComponent`](Assets/Scripts/Components/InvincibilityComponent.cs) — brief i-frames with a sprite blink.

Both the player [`Weapon`](Assets/Scripts/Weapons/Weapon.cs) and the boss use Unity's `IObjectPool<Bullet>` to recycle bullets instead of instantiating per shot. Bullets return to the pool on hit, after a 3-second lifetime, or when they leave the screen bounds.

## Project layout

```
Assets/
  Scripts/
    Animations/   GameManager (singleton, DontDestroyOnLoad), LevelManager (scene transitions)
    Components/   Attack, Health, Hitbox, Invincibility
    Enemies/      Enemy base class, four variants, spawner, debug click-spawner
    Player/       Player singleton, acceleration-based movement
    Portal/       Roaming portal that loads the Main scene
    System/       CombatManager wave loop
    UI/           UI Toolkit HUD bindings
    Weapons/      Weapon, pooled Bullet, WeaponPickup
  Prefabs/        Player, enemies + their spawners, weapons, bullet, GameManager
  Scenes/         ChooseWeapon, Main
  Sprites/        Player, enemy, environment, and pickup art
  Animations/     Engine, transition, and UI (UXML) assets
```

`GameManager` and `Player` are singletons marked `DontDestroyOnLoad`, so the player ship, camera, and managers persist across the scene change from **ChooseWeapon** to **Main**. `LevelManager.LoadScene` plays a transition animation, loads the scene asynchronously, and repositions the player.

The HUD in [`UI.cs`](Assets/Scripts/UI/UI.cs) is built with UI Toolkit — it queries labels out of the `UIDocument` root and refreshes them each frame.

## Running it

1. Install Unity **2022.3.50f1** (Unity Hub → Installs → Add).
2. Clone the repo and open the project folder in Unity Hub. Package dependencies resolve from [`Packages/manifest.json`](Packages/manifest.json).
3. Open [`Assets/Scenes/ChooseWeapon.unity`](Assets/Scenes/ChooseWeapon.unity) and press Play.

Starting from `Main` directly skips the weapon pickup, so the player will spawn unarmed.

## Credits

Enemy and player ship art is from Foozle's _Void_ pixel art ship packs (the `Kla'ed` and `Nairan` sets under [`Assets/Sprites/`](Assets/Sprites/)).

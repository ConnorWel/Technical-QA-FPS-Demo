# FPS Project: QA Testing & Performance Optimization

This repository contains a custom-built 3D First-Person Shooter developed in Unity. Beyond gameplay implementation, this project serves as a showcase for Technical QA, State Machine Design, and Performance Profiling.

## 🎯 1. Core System Design: Hit & Death State Machine Rules

The following PRD-level rules were designed to prevent state machine conflicts during combat:

* **Absolute Priority:** If HP reaches 0, the Death animation MUST trigger unconditionally.
* **Action Independence:** Taking damage does NOT interrupt the reloading state. Reloading can only be interrupted by weapon switching or specific player inputs.
* **Frame-Perfect Hit Detection:** Damage is calculated per hit registration, not per frame. Simultaneous hits in the same frame will process sequentially without dropping damage.
* **State Override:** Regardless of the player's current locomotion state (Sprinting, Jumping, Crouching, Reloading), dropping to 0 HP must instantly terminate the current state and transition to Death.
* **Healing Conflict Resolution:** If fatal damage is taken during a healing state/animation, the healing process is instantly aborted, and the Death state takes absolute precedence.

---

## 🧪 2. Edge-Case Test Matrix (Weapon & Locomotion)

Behavioral testing matrix focusing on the conflict between the Weapon State Machine and Player Locomotion.

| TC_ID | Title | Pre-conditions | Steps to Reproduce | Expected Result |
| :--- | :--- | :--- | :--- | :--- |
| **TC_001** | Switching weapon while Running and Reloading | Player is on the ground.<br>Primary weapon equipped.<br>Ammo > 0. | 1. Hold `[W]` + `[Shift]` to Sprint.<br>2. Press `[R]` to reload.<br>3. Before reload completes, press `[1]` to switch weapons. | **Logic:** Reload canceled. Ammo does NOT increase.<br>**Anim:** Smooth transition to weapon switch while sprinting.<br>**Audio:** Reload SFX stops instantly. |
| **TC_002** | Switching weapon when Jumping and Reloading | Player is on the ground.<br>Primary weapon equipped.<br>Ammo > 0. | 1. Press `[Space]` to Jump.<br>2. Mid-air, press `[R]` to reload.<br>3. Before reload completes, press `[1]` to switch. | **Logic:** Reload canceled. Ammo does NOT increase.<br>**Anim:** Smooth transition to weapon switch while mid-air.<br>**Audio:** Reload SFX stops instantly. |

---

## 🐛 3. Comprehensive Bug Tracking Log

Documentation of bugs found during black-box and exploratory testing, including root-cause hypothesis.

| Bug ID | Title / Summary | Module | Severity | Status | Steps to Reproduce | Expected vs Actual Result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **BUG-001** | Rapid right-click for ADS causes transition lock and weapon clipping | Animation | High | 🟢 Closed | 1. Rapidly click RMB within 0.2s. | **Exp:** Smooth blend to Hip-fire.<br>**Act:** Stucks in between, weapon clips. |
| **BUG-002** | Bullets fail to return to Object Pool on hitting custom shader geometry | VFX / Pool | Medium | 🟢 Closed | 1. Fire weapon at 'Dissolve' walls.<br>2. Check Hierarchy. | **Exp:** Bullet deactivated/pooled.<br>**Act:** Memory leak, instances grow. |
| **BUG-014** | Instantiated bullets freeze instantly at muzzle and fail to despawn | Physics | High | 🟢 Closed | 1. Fire weapon.<br>2. Observe bullet trajectory. | **Exp:** Bullet travels at speed 100.<br>**Act:** Bullets hang stationary permanently. |
| **BUG-015** | Player can bypass Sprint shooting restriction by Jumping | State Machine | Medium | 🔴 Open | 1. Hold Sprint `[W]`+`[Shift]`.<br>2. Press Jump `[Space]`.<br>3. Fire weapon mid-air. | **Exp:** Cannot fire until landing.<br>**Act:** Fires mid-air, bypassing sprint lock. |

> **Note:** UI Performance optimizations (Canvas separation, disabling Raycast Targets) have been implemented to maintain stable frame rates during gameplay.

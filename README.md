# *FPS Project:* QA Testing & Performance Optimization

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

> **Note:** UI Performance optimizations (Canvas separation, disabling Raycast Targets) have been implemented to maintain stable frame rates during gameplay.‘’

## 📈 4. Performance Profiling & Optimization Case Study

**Objective:** Identify and resolve frame drops occurring during rapid weapon switching in combat scenarios using Unity Profiler.

### 🔴 Before Optimization: The Bottleneck
During high-frequency weapon switching, noticeable stuttering was observed. White-box profiling revealed a significant CPU spike on the main thread.

![Profiler Before Optimization](<img width="1360" height="1121" alt="Profiler_Before_Optimization" src="https://github.com/user-attachments/assets/6264487d-3f5c-47b0-be44-dbcbd9b67a4e" />)
*(Note: Please create an 'images' folder in your repository and upload your 'before' screenshot here, updating the path if necessary)*

* **Diagnosis:** The Profiler identified `PlayerLoop` execution time spiking to **~32.49ms** (dropping frame rates below 30 FPS).
* **Root Cause Analysis:** Deep profiling pinpointed the `WeaponSwitcher.SelectWeapon()` method. Redundant synchronous operations and unoptimized iteration logic were blocking the main thread during the state transition, causing an excessive CPU load.

---

### 🟢 After Optimization: The Resolution
The state transition logic was refactored to eliminate redundant main-thread blocking operations during weapon instantiation and activation. 

![Profiler After Optimization](<img width="1360" height="1121" alt="Profiler_After_Optimization" src="https://github.com/user-attachments/assets/9cda782b-7518-4d88-ac43-275594bfb0cd" />)
*(Note: Upload your 'after' screenshot and link it here)*

* **Action Taken:** Removed heavy synchronous calculations from the active `SelectWeapon` loop. Optimized the `Transform` iteration and Animator parameter synchronization to ensure lightweight execution.
* **Result:** 
    * The `SelectWeapon` method execution time was reduced from **~32ms to <1ms**.
    * CPU spikes were completely eliminated, maintaining a stable and smooth frame rate (60+ FPS) even during rapid, consecutive weapon switching.
    * The combat experience is now stutter-free, ensuring the Action Independence rule (from Section 1) performs flawlessly under stress.
 

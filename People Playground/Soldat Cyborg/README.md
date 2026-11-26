# Soldat Cyborg — patched save (v1.28)

Drop `Soldat Cyborg.json` into your saved contraptions folder
(`AppData\LocalLow\Studio Minus\People Playground\Contraptions` on Windows)
and load it in-game. Same filename as your original; keep a backup of the original.

## Changes made — round 1 (179 bytes edited, file structure untouched)

**Damage immunity**
- `ImmuneToDamage = true` on all 56 affected limbs/parts — no damage from bullets, blunt force, stabbing, crushing or electric shock
- `Indestructible = true` on all 9 machinery components (7 were destructible)

**Electricity / water**
- All 28 limbs: `ConductOverride = false`, `ForceNoCharge = true`, `ChargeBurns = false`, `ForceNoChargeParticles = true` (no spark VFX)
- The non-limb (machinery) parts were left unchanged so any internal wiring/activation keeps working

**Limb detachment**
- Limb joint-break thresholds retargeted toward the 64450.0 health literal — far above the 10000 used for "unbreakable" in the game's own durability code

## Changes made — round 2 (195 edits on the re-saved save, 226,082 → 226,096 bytes)

**Deletion immunity (black holes, Decimator)**
- `Deletable = false` on all 42 parts — black holes and the Decimator can no longer delete them
- `Disintegratable = false` on all 42 parts — disintegration weapons no longer work
- `IsLethalToBreak = false` everywhere it was set (6×)

**Electricity / water — full coverage**
- The 14 non-limb (machinery/hardware) parts now match the limbs: `ConductOverride = false`, `ChargeBurns = false`, `ForceNoCharge = true`, `ForceNoChargeParticles = true`. All 42 conductive-capable parts are now fully charge-proof.

**Limb detachment — every threshold maxed**
- All 28 `BreakingThreshold` values now resolve to 64450.0 (26 retargeted to the shared 64450.0 literal + 2 inline floats patched in place, including a 4.0 threshold that would have snapped instantly)

**Healing / cleanliness pass**
- `Broken = true` healed on all 7 previously broken limbs
- `ShatteredObjectChance` zeroed (16 of 28; 12 share one unresolvable pooled value — see caveats)
- `InternalBleedingIntensity` zeroed (4 non-zero values), `BleedingPointCount 1 → 0`, `stabWoundCount 1 → 0`
- `ShockLevel` / `PainLevel` confirmed 0.0; no poison-susceptibility fields exist in this save format

## Changes made — round 9 (3 bytes — **full revert of the Generator to its original state**)

The generator still refused to work after round 8, so every modification ever made to it was undone. The Generator's entire block is now **byte-identical to the working save you uploaded** (verified by direct byte comparison of the whole block, not just individual flags):

- `Disintegratable` back to **true** (round 5 had set it false) — note this means disintegration weapons can affect the Generator again
- Charge-driver `enabled` = **true** (original value; already restored in round 8)
- "P1" terminal `ParticleSystem` `enabled` = **true** — **charge particles are back ON**
- "P1/SpriteLightFlash" `enabled` = **true** — **the glow is back ON**
- Electricity flags (`ConductOverride` / `ChargeBurns` / `ForceNoCharge`) — original values (restored in round 7)

Also verified byte-identical to the original: the **Laser Pointer** block (except its intentional round-5 `Disintegratable=false`) and the **Energy Wire** connection between generator and laser. The base human region remains 100% byte-untouched. Structure parse signature unchanged.

If the generator still does not charge the laser with this file, the cause lies **outside** the generator→laser path — the next suspect would be the round-5 electricity immunity flags on the *other* machinery parts; those can be reverted selectively next.

## Changes made — round 6 (4 bytes)

**Laser Pointer exception:** the Laser Pointer (inside the head) needs to hold and conduct charge to function, so its four electricity flags were reverted to the original values: `ConductOverride=true`, `ChargeBurns=true`, `ForceNoCharge=false`, `ForceNoChargeParticles=false`. All other parts keep their electricity immunity. Note: the **Generator** still has `ForceNoCharge=true` — if the laser is powered through it and won't fire, that flag can be reverted the same way.

## Changes made — round 8 (2 bytes on top of round 7 — fixes the generator)

**Root cause of the dead generator found (via the game's decompiled source):** round 7 accidentally disabled the wrong component. The `enabled` flag it flipped belongs to the generator's **charge-driver behaviour** (the component holding `Activated`, `TargetCharge=100`, `WarmupProgress`, …), not to a particle emitter. Per the decompiled code, activation (`Use()`) is gated by `enabled` and `OnDisable()` forces `Activated=false` — so with `enabled=false` the generator could never be switched on and supplied no power.

- **Charge-driver `enabled` restored to `true`** (its original value) — the generator starts again (auto-started by its delayed activation event / manual activation, exactly as in the original save)
- **Particles disabled the correct way:** the actual Unity `ParticleSystem` component on the generator's "P1" terminal (child GameObject, the emitter the driver plays) → `enabled = false`
- **Glow** (`P1/SpriteLightFlash`) stays disabled from round 7
- Generator electricity flags remain restored to the original save's values (conduction on)
- Net change vs round 6: 5 bytes (3 generator electricity flags + particle system off + glow off); file length unchanged; base human still byte-untouched; structure verified intact

## Changes made — round 7 (5 bytes, file length unchanged)

**Generator: power restored + visuals tamed.** The laser needs the Generator to hold and conduct charge, so its electricity flags were reverted to the original save's values: `ConductOverride=true`, `ChargeBurns=true`, `ForceNoCharge=false`. (`ForceNoChargeParticles` was already `true` in the original save — the spark particles were never active on it.)

On top of that, two child components of the Generator were disabled via their `enabled` flags:
- `UnityEngine.ParticleSystem` → `enabled = false` — this is the charge-scaling particle emitter, so the generator's charge particles are off
- `P1/SpriteLightFlash` → `enabled = false` — this is the glow/light-flash component, so the glow is off
- The `P1/P2` power-terminal component was deliberately **left enabled** so the generator keeps delivering power

**Base human: still 100% byte-untouched** (all round-7 edits sit well before the human's data region).

## Changes made — round 5 (current, 108 edits, 226,082 → 226,084 bytes)

Rebuilt from the re-sent working save after v3 misbehaved. **Root cause identified:** v3 set `Disintegratable=false` (and originally `Deletable=false`) on ALL 42 parts — including the base human's limbs. Those flags gate the game's crush/disintegrate pathway ("setting it to false will also prevent the object from being disintegrated or crushed"), which interfered with the human's intentionally-destroyed limbs.

**All parts:**
- `Deletable` left **true** everywhere (per request — manual deletion works)

**Android limbs (14) + machinery (14 parts) only:**
- `Disintegratable = false` — disintegration/crush immunity, scoped AWAY from the human

**Machinery only (14 parts):**
- `ConductOverride=false`, `ChargeBurns=false`, `ForceNoCharge=true`, `ForceNoChargeParticles=true`

**Android limbs only:**
- All 14 `BreakingThreshold` → 64450.0 (dismemberment disabled)
- `ShatteredObjectChance` → 0.0 ×14; `IsLethalToBreak` → false ×4
- Verified natively clean: `Broken`/`IsDismembered`/`Braindead`/`BrainDamaged` false, all wound counters 0

**Base human: 100% untouched — byte-verified.** The last modified byte in the file lies *before* the human's data region begins; the human's 7 destroyed (`Broken=true`) limbs, its wounds, and its original thresholds are exactly as uploaded. Round 1's fixes (ImmuneToDamage ×56, Indestructible ×9 machinery, Waterproof ×9) remain in effect.

## Earlier rounds (superseded by round 5, kept for history)

## Round 3 (164 edits, person-scoped, 226,082 → 226,084 bytes)

The save contains **two persons**: the **Android** (14 limbs) and the **base human** (14 limbs, intentionally left with only torso+head alive — the destroyed limbs are part of the design).

**All parts (both persons + machinery)**
- `Deletable` left **true** ×42 (reverted per user request — manual deletion works; note black holes/Decimator can delete again)
- `Disintegratable = false` ×42 — disintegration weapons no longer work on anything
- Non-limb parts charge-proofed: `ConductOverride=false`, `ChargeBurns=false`, `ForceNoCharge=true`, `ForceNoChargeParticles=true` (all 42 parts now fully charge-proof)

**Android only**
- All 14 `BreakingThreshold` values → **64450.0** (10 retargeted + one 4.0 inline threshold patched — that limb would have snapped instantly)
- `ShatteredObjectChance` 0.1 → 0.0 on all 14 Android limbs
- `IsLethalToBreak` true→false ×4 (breaking an Android limb can no longer kill)
- Verified natively clean: `Broken`, `IsDismembered`, `Braindead`, `BrainDamaged` all false; stab/bleeding counters all 0

**Base human — intentionally untouched**
- `Broken=true` on the 7 destroyed limbs **kept** (torso+head-only design)
- Wounds kept: internal bleeding (0.9998, 1.2659), 1 bleeding point, 1 stab wound
- Original joint-break thresholds kept (the human remains fragile)
- Round 2's accidental healing of these human values is reverted by building round 3 from your original upload

## Honest caveats

1. **Slice()-based destruction cannot be blocked by save editing.** The Archelix Caster and a fully charged Laser Pointer destroy parts and even Fixed Cables / Strong Springs through a direct `Slice()` call that has no serialized immunity flag. The wires already hold an infinite breaking-force value in the save (so force-based cable breaking was already impossible), but slicing is runtime-only — a small runtime mod would be needed for true slice immunity.
2. **`Deletable = false` trade-off:** you also cannot manually delete those parts in-game (the same flag blocks both). Select and remove via the contraption editor instead if needed.
3. **In-game re-save may revert some values.** Charge-related flags were recomputed once before; if you re-save in-game and problems return, re-upload and the patch can be re-applied.
4. **12× `ShatteredObjectChance`** point to a pooled value that could not be resolved with certainty and were left untouched.
5. **`HasJoint = false` on 2 limbs** was left as-is (flipping it is structurally risky and may produce floating limbs).

## Verification performed
- Full post-patch field survey: every targeted field confirmed in its final state (Deletable/Disintegratable false, all BreakingThresholds = 64450.0, all wounds zeroed, Broken/IsDismembered/Braindead/BrainDamaged false)
- Structural parse of the patched binary: identical traversal to the original (parse abort point shifted by exactly the 14 inserted bytes, same pool id sequence) — structure intact

## Changes made — round 10 (74 edits, 132 bytes, size unchanged: 226,084)

**Android body improvements (OS4 Android limbs only; the human torso/head untouched)**

| Field | Before | After |
|---|---|---|
| Vitality | 15.0 / 7.0 / 6.0 / 1.0 / 0.0 (mixed) | **0.0** on all 14 limbs |
| RegenerationSpeed | 0.3 | **0.5** on all 14 limbs |
| ShotDamageMultiplier | 1.0 | **0.0** on all 14 limbs |
| ImpactPainMultiplier | 1.0 / 1.2 / 5.0 (mixed) | **0.0** on all 14 limbs |
| Numbness | −0.00084 / −4.86 (mixed) | **0.0** on all 14 limbs |
| BaseStrength | 8.0 (11 limbs) / 9.0 (2 limbs) | **12.0 / 13.5 = +50%** |
| IsLethalToBreak | false (already) | unchanged |
| BruiseCount / ShatteredObjectChance | 0 (already) | unchanged |
| Health 300 / BreakingThreshold 64450 / ImmuneToDamage true | (already) | unchanged |

Mechanics: shared pool values were retargeted to exact-value entries and two
*unused* value-pool entries were repurposed as private 0.0 / 12.0 literals
(zero length change). Two harmless side effects on non-limb fields:
- OS2 Temperature 14.000166 → 12.0 (ambient temperature of one part, imperceptible)
- OS12 InitialInaccuracy 0.001 → 0.0 (firearm spread of the built-in gun — slightly *more* accurate)

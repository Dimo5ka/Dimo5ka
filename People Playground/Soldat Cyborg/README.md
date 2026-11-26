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

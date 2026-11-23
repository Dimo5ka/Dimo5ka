# Soldat Cyborg — patched save (v1.28)

Drop `Soldat Cyborg.json` into your saved contraptions folder
(`AppData\LocalLow\Studio Minus\People Playground\Contraptions` on Windows)
and load it in-game. Same filename as your original; keep a backup of the original.

## Changes made (179 bytes edited, file structure untouched)

**Damage immunity**
- `ImmuneToDamage = true` on all 56 affected limbs/parts — no damage from bullets, blunt force, stabbing, crushing or electric shock
- `Indestructible = true` on all 9 machinery components (7 were destructible)

**Electricity / water**
- All 28 limbs: `ConductOverride = false` (limbs can never conduct electricity), `ForceNoCharge = true` (limbs cannot hold a charge), `ChargeBurns = false`, `ForceNoChargeParticles = true` (no spark VFX)
- The 15 non-limb (machinery) parts were left unchanged so any internal wiring/activation keeps working

**Limb detachment**
- 3 limbs with a real joint-break threshold (15.0 etc.) were retargeted to the 64450.0 health literal — far above the 10000 used for "unbreakable" in the game's own durability code
- 13 limbs already pointed at NaN thresholds (unbreakable); 12 use the 0.0 default literal

## Notes
- The save contained pre-existing damage: 7 limbs with `Broken = true`, 2 limbs already detached (`HasJoint = false`). These were left as-is.
- Machinery parts were already `Waterproof = true`.

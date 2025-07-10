# Custom Biome Generation – Minecraft Java Edition

## Introduction
Custom biomes are user‑defined biome configurations introduced via data packs in **Minecraft: Java Edition**. Each biome is stored as a JSON file located in `data/<namespace>/worldgen/biome`. These JSON files allow complete control over visual effects, terrain carvers, world‑generation features and mob spawning rules.

## JSON Schema Reference

### Root object
- **has_precipitation** *(boolean)* — Whether the biome experiences precipitation.
- **temperature** *(float)* — Base temperature used for grass/foliage colour and snow/rain determination.
  - **temperature_modifier** *(enum: `none` | `frozen`, optional, default `none`)* — Adjusts temperature before height modifier; `frozen` raises some temperatures to 0.2 to force rain.
- **downfall** *(float)* — Governs grass and foliage colour gradients.
- **effects** *(object)* — Ambient visual/audio effects.
  - **fog_color**, **sky_color**, **water_color**, **water_fog_color** *(int)* — RGB colours encoded as decimal.
  - **foliage_color**, **grass_color** *(int, optional)* — Overrides foliage/grass tint.
  - **grass_color_modifier** *(enum: `none` | `dark_forest` | `swamp`, optional, default `none`)* — Global tint shift.
  - **particle** *(object, optional)*
    - **probability** *(float 0‑1)* — Spawn chance per tick.
    - **options**
      - **type** *(ID)* — Namespaced particle ID.
      - *Type‑specific extra fields*
        - **block**, **block_marker**, **falling_dust**: `value` *(block‑state)*
        - **item**: `value` `{ id, Count, tag }`
        - **dust**: `color` `[r,g,b]`, `scale`
        - **dust_color_transition**: `fromColor`, `toColor`, `scale`
        - **sculk_charge**: `roll`
        - **vibration**: `destination`, `arrival_in_ticks`
        - **shriek**: `delay`
  - **ambient_sound** *(ID or { sound_id, range })*
  - **mood_sound** `{ sound | sound_id, range, tick_delay, block_search_extent, offset }`
  - **additions_sound** `{ sound | sound_id, range, tick_chance }`
  - **music** `{ sound | sound_id, range, min_delay, max_delay, replace_current_music }`
- **carvers**
  - **air** — Carvers executed during the *air* step (list or tag of configured carvers).
  - **liquid** — Carvers executed during the *liquid* step (currently unused).
- **features** *(array, len = 11)* — Ordered lists/tags of placed features per generation step:
  1. `RAW_GENERATION`
  2. `LAKES`
  3. `LOCAL_MODIFICATIONS`
  4. `UNDERGROUND_STRUCTURES`
  5. `SURFACE_STRUCTURES`
  6. `STRONGHOLDS`
  7. `UNDERGROUND_ORES`
  8. `UNDERGROUND_DECORATION`
  9. `FLUID_SPRINGS`
  10. `VEGETAL_DECORATION`
  11. `TOP_LAYER_MODIFICATION`

  *Feature order must be identical across biomes in the same step; e.g., if `ore_dirt` precedes `ore_gravel` in one biome’s `UNDERGROUND_ORES` list, all other biomes containing both must use the same order.*

- **creature_spawn_probability** *(float 0.0 – 0.9999999, optional)* — Global spawn density multiplier.
- **spawners** — Mob spawn lists by **category**: `monster`, `creature`, `ambient`, `water_creature`, `underground_water_creature`, `water_ambient`, `misc`, `axolotls`.
  - Each entry: `{ type, weight, minCount (>0), maxCount (≥ minCount) }`
- **spawn_costs** — Per‑entity spawn‑limit mechanics.
  - `{ <entity_id>: { energy_budget, charge } }`

### File Location
`data/<namespace>/worldgen/biome/<biome_id>.json`

## Version History
| Version | Snapshot/Pre‑release | Change |
|---------|---------------------|--------|
| 1.19 (22w11a) | Snapshot | Removed **category** field; replaced by biome tags. |
| 1.19.3 | Pre‑release 3 | Sound event fields accept fixed audible **range** sub‑object. |
| 1.19.4 (23w03a) | Snapshot | **precipitation** renamed to **has_precipitation** (now boolean). |
| 1.19.4 | Pre‑release 1 | **minCount** and **maxCount** in **spawners** must be positive; **maxCount** ≥ **minCount**. |

## Notes
- Exclusive to Java Edition; Bedrock does not yet support data‑pack biomes.
- Generation logic uses the same carver/feature pipeline as vanilla biomes, enabling full parity.
- The JSON schema is validated during world‑load; malformed fields will crash the game or be ignored silently.

## References
- “Custom biome.” *Minecraft Wiki* (accessed 10 Jul 2025).


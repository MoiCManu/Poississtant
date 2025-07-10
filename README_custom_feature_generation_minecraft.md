# Custom Feature – Minecraft Java Edition

## Introduction

Les **features personnalisées** modélisent tous les objets de décor et gisements (arbres, minerais, lacs, structures fossiles, etc.) placés à l’échelle locale d’un chunk. La logique est divisée en :

1. **Feature type** : algorithme de génération de base.
2. **Configured feature** : type + paramètres (`config`).
3. **Placed feature** : configured feature + « placement modifiers » définissant où et quand tenter la génération.

> **Chemins datapack**
>
> - `data/<namespace>/worldgen/configured_feature/<id>.json`
> - `data/<namespace>/worldgen/placed_feature/<id>.json`  ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_feature))

---

## 1 | Catalogue des Feature Types

Les IDs officiels (1.20.6) :

````
no_op, bamboo, basalt_columns, basalt_pillar, block_column, block_pile,
blue_ice, bonus_chest, chorus_plant, coral_claw, coral_mushroom,
coral_tree, delta_feature, desert_well, disk, dripstone_cluster,
end_gateway, end_island, end_spike, fill_layer, flower, forest_rock,
fossil, freeze_top_layer, geode, glowstone_blob, huge_brown_mushroom,
huge_fungus, huge_red_mushroom, iceberg, ice_spike, kelp, lake,
large_dripstone, monster_room, multiface_growth, nether_forest_vegetation,
netherrack_replace_blobs, no_bonemeal_flower, ore, pointed_dripstone,
random_boolean_selector, random_selector, random_patch,
replace_single_block, root_system, sculk_patch, seagrass, sea_pickle,
simple_block, simple_random_selector, spring_feature, tree,
twisting_vines, underwater_magma, vegetation_patch, vines,
void_start_platform, waterlogged_vegetation_patch, weeping_vines
```  ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_feature))

Chaque type possède un sous‐schéma spécifique exposé par la page wiki ; les champs communs demeurent toutefois :
| Champ racine | Type | Rôle |
|--------------|------|------|
| `type` | ID | Identifiant du feature type. |
| `config` | objet | Paramètres dépendant du type (peut être `{}` vide). |

Exemple minimal :
```json5
{
  "type": "minecraft:no_op",
  "config": {}
}
````

---

## 2 | Placed Feature & Placement Modifiers

Un **placed feature** choisit *où* appeler la configured feature.

```json5
{
  "feature": "<namespace>:my_tree",           // ID ou objet configured_feature
  "placement": [
    { "type": "in_square" },                 // x,z aléatoires 0‑15
    { "type": "surface_water_depth_filter",  // ≤ 2 blocs d’eau ?
      "max_water_depth": 2 },
    { "type": "count", "count": 5 }         // 5 tentatives
  ]
}
```

### 2.1 Modifiers courants

| Type                         | Effet principal                          | Champs clés                                 |
| ---------------------------- | ---------------------------------------- | ------------------------------------------- |
| `biome`                      | Filtre sur le biome courant.             | —                                           |
| `count`                      | Répète N fois.                           | `count` (*int/int provider*)                |
| `rarity_filter`              | 1/N chance.                              | `chance`                                    |
| `height_range`               | Force Y via *height\_provider*.          | `height`                                    |
| `in_square`                  | Décale X/Z aléatoirement (0‑15).         | —                                           |
| `surface_water_depth_filter` | Profondeur d’eau max.                    | `max_water_depth`                           |
| `noise_threshold_count`      | Nombre d’essais dépendant d’un bruit 2D. | `noise_level`, `below_noise`, `above_noise` |

> La page liste 25 modifiers ; reportez‑vous à l’article pour la matrice complète. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_feature))

---

## 3 | Exemples

### 3.1 – Gisement de cuivre rare

```json5
// data/acme/worldgen/configured_feature/ore_copper_rare.json
{
  "type": "minecraft:ore",
  "config": {
    "size": 6,
    "discard_chance_on_air_exposure": 0.2,
    "targets": [
      { "target": "minecraft:stone_ore_replaceables",
        "state": { "Name": "minecraft:deepslate_copper_ore" } }
    ]
  }
}
```

```json5
// data/acme/worldgen/placed_feature/ore_copper_rare_highlands.json
{
  "feature": "acme:ore_copper_rare",
  "placement": [
    { "type": "triangle_range",          // Y ‑40 → 80
      "height": { "type": "triangle", "min_inclusive": -40, "max_inclusive": 80 } },
    { "type": "count", "count": 3 },
    { "type": "biome" }
  ]
}
```

### 3.2 – Patch de fleurs aléatoires

```json5
{
  "type": "minecraft:random_patch",
  "config": {
    "tries": 96,
    "xz_spread": 5,
    "y_spread": 2,
    "feature": {
      "feature": "minecraft:simple_block",
      "config": {
        "to_place": { "Name": "minecraft:poppy" }
      }
    }
  }
}
```

---

## 4 | Historique

| Version    | Snapshot | Ajouts majeurs                                                      |                                                                             |
| ---------- | -------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **1.16.2** | 20w28a   | Support expérimental des configured features dans les datapacks.    |                                                                             |
| **1.17**   | 20w45a   | Ajout des features `geode`, `dripstone_cluster`, `large_dripstone`. |                                                                             |
| **1.18**   | 21w38a   | Placement modifiers élargis (`noise_based_count`, etc.).            |                                                                             |
| **1.19**   | 22w11a   | Ajout `sculk_patch` & champs supplémentaires pour `glow_lichen`.    |                                                                             |
| **1.19.4** | 23w07a   | Placer/foliage/trunk placers « cherry ».                            |                                                                             |
| **1.20**   | 23w17a   | `replaceable_blocks` pour `huge_fungus`.                            |  ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_feature)) |

---

## 5 | Bonnes pratiques

- **Séparation logique** : conservez `configured_feature` et `placed_feature` distincts pour réutiliser une configuration avec plusieurs règles de placement.
- **Ordre des modifiers** : ils sont évalués séquentiellement ; un `count` en premier multiplie ensuite tous les offsets appliqués après.
- **Test interactif** : la commande `/place feature <id>` (ou `/place placed_feature <id>`) permet une visualisation rapide en jeu.
- **Performances** : évitez des `count` > 64 dans des biomes denses pour ne pas saturer le scheduler de génération.

---

## Références

- “Custom feature.” *Minecraft Wiki* (consulté le 10 juillet 2025).


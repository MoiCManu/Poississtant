# Custom Structures – Minecraft Java Edition

## Introduction

Les **structures personnalisées** (« structure features ») représentent la brique la plus haute de la pile world‑gen : elles encapsulent des bâtiments, ruines, donjons ou ensembles procéduraux entiers. Depuis la refonte de 1.19 (snapshot 22w11a), la chaîne de génération repose sur **cinq** enregistrements JSON + NBT :

| Composant                | Chemin datapack                                      | Rôle                                                             |
| ------------------------ | ---------------------------------------------------- | ---------------------------------------------------------------- |
| **Structure template**   | `data/<namespace>/structures/<path>.nbt`             | Modèle NBT (blocs) placé tel quel ou via jigsaw.                 |
| **Template pool**        | `data/<namespace>/worldgen/template_pool/<id>.json`  | Liste pondérée d’éléments (templates, features) pour jigsaw.     |
| **Configured structure** | `data/<namespace>/worldgen/structure/<id>.json`      | Paramètres de structure + biomes + éventuels overrides de spawn. |
| **Structure set**        | `data/<namespace>/worldgen/structure_set/<id>.json`  | Algorithme de distribution (rings, spread) et fréquence.         |
| **Processor list**       | `data/<namespace>/worldgen/processor_list/<id>.json` | Suite de processors appliqués aux templates lors du placement.   |

> Exclusif à **Java Edition** ; Bedrock n’expose pas (encore) de datapacks structures.

---

## 1 | Structure Feature Types

| ID (`type`)                                                                                                                | Génération               | Note                                                    |                                                                              |
| -------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `buried_treasure`, `desert_pyramid`, `fortress`, `jungle_temple`, `mineshaft`, `ocean_monument`, `stronghold`, `swamp_hut` | **Hard‑coded**           | Aucun template externe ; config limitée.                |                                                                              |
| `end_city`, `igloo`, `nether_fossil`, `ocean_ruin`, `ruined_portal`, `shipwreck`, `woodland_mansion`                       | **Templates NBT**        | Structure prédéfinie, mais positionnement configurable. |                                                                              |
| `jigsaw`                                                                                                                   | **Procédural via pools** | Relie dynamiquement plusieurs templates/features.       | ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_structure)) |

---

## 2 | Structure Template (NBT)

- Fichier `.nbt` exporté via bloc *Structure* ou outil externe (`/structure save`).
- Le NBT stocke : palette de blocs, entités, données de structure (jigsaw, processors, metadata).
- Identifié en jeu par un **resource location** `namespace:path`.

---

## 3 | Template Pool (`template_pool`)

```json5
{
  "fallback": "<namespace>:fallback_pool",   // pool de secours
  "elements": [
    {
      "weight": 2,
      "element": {
        "element_type": "minecraft:single_pool_element",
        "location": "acme:villager_house_01",
        "projection": "rigid",
        "processors": "minecraft:empty"
      }
    },
    {
      "weight": 1,
      "element": {
        "element_type": "minecraft:feature_pool_element",
        "feature": "acme:flower_patch",
        "projection": "terrain_matching"
      }
    }
  ]
}
```

### Règles clés

1. **fallback** s’active si la profondeur max est atteinte *ou* si aucun élément ne passe les contraintes.
2. `weight` ∈ [1‑150].
3. `projection` : `rigid` (hauteur fixe) ou `terrain_matching` (suit la topographie).
4. `element_type` alternatives : `empty_pool_element`, `list_pool_element` (concatène plusieurs éléments), `legacy_single_pool_element` (ne remplace pas l’air).

---

## 4 | Configured Structure (`structure`)

```json5
{
  "type": "minecraft:jigsaw",
  "biomes": "#minecraft:is_overworld",
  "step": "surface_structures",
  "terrain_adaptation": "beard_box",
  "spawn_overrides": {
    "monster": {
      "bounding_box": "piece",
      "spawns": [ { "type": "minecraft:witch", "weight": 1, "minCount": 1, "maxCount": 1 } ]
    }
  },
  "start_pool": "acme:village/start",
  "size": 4,
  "start_height": { "type": "absolute", "value": 64 },
  "max_distance_from_center": 80
}
```

### Champs communs

| Champ                | Type                                               | Défaut               | Description                                     |
| -------------------- | -------------------------------------------------- | -------------------- | ----------------------------------------------- |
| `type`               | ID                                                 | —                    | L’un des 16 types listés plus haut.             |
| `biomes`             | ID / tag / liste                                   | —                    | Biomes autorisés.                               |
| `step`               | enum                                               | `surface_structures` | Étape de world‑gen (11 valeurs).                |
| `terrain_adaptation` | enum: `none` / `beard_thin` / `beard_box` / `bury` | `none`               | Ajustement du relief.                           |
| `spawn_overrides`    | objet                                              | —                    | Remplace les tables de spawn dans la structure. |

#### Extensions par type

- `` : `start_pool`, `size` (0‑7), `start_height` (*height\_provider*), `project_start_to_heightmap`, `start_jigsaw_name`, `max_distance_from_center`, `use_expansion_hack`.
- `` : sous‑champ `type` (`normal` ou `mesa`).
- `` : `biome_temp`, `large_probability`, `cluster_probability`.
- `` : tableau `setups` (poids, `placement`, `mossiness`, etc.).
- `` : `is_beached` (bool).

---

## 5 | Structure Set (`structure_set`)

```json5
{
  "structures": [
    { "structure": "acme:village", "weight": 1 }
  ],
  "placement": {
    "type": "minecraft:random_spread",
    "salt": 1234567,
    "frequency": 0.8,
    "spread_type": "linear",
    "spacing": 24,
    "separation": 8,
    "exclusion_zone": { "other_set": "minecraft:stronghold", "chunk_count": 10 }
  }
}
```

### Placement modes

| `type`             | Inspiration vanilla | Paramètres clés                                                |
| ------------------ | ------------------- | -------------------------------------------------------------- |
| `concentric_rings` | Strongholds         | `distance`, `count`, `preferred_biomes`, `spread`              |
| `random_spread`    | Bastions, huts      | `spacing`, `separation`, `spread_type` (`linear`/`triangular`) |

---

## 6 | Processor List (`processor_list`)

```json5
{
  "processors": [
    { "processor_type": "minecraft:block_ignore" },
    { "processor_type": "minecraft:block_rot", "integrity": 0.8, "rottable_blocks": "#minecraft:wooden_planks" }
  ]
}
```

- Peut être défini sous forme **liste pure** ou objet contenant `processors`.
- Exemples de processors : `block_ignore`, `block_rot`, `lava_submerged`, `zombie_piglin_specific` …

---

## 7 | Historique des versions

| Version           | Snapshot / Pré‑release                                                                                    | Changements clés                                                             |
| ----------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **1.18.2** Pre‑1  | Support *expérimental* des structures datapack (structure set).                                           |                                                                              |
| **1.19** 22w11a   | Réécriture complète : `configured_structure_feature` → `structure`; nouveau dossier `worldgen/structure`. |                                                                              |
| 22w12a            | `jigsaw` : ajout `max_distance_from_center`.                                                              |                                                                              |
| 22w13a            | Processor `block_rot` : champs obligatoires & limites `[0‑1]`.                                            |                                                                              |
| 22w17a            | `jigsaw` : champ `start_jigsaw_name`.                                                                     |                                                                              |
| **1.19.3** 22w44a | Suppression du champ `name` dans `template_pool`.                                                         |                                                                              |
| **1.19.4** Pre‑1  | Validation stricte de `minCount/maxCount` dans `spawn_overrides`.                                         |                                                                              |
| **1.20** 23w12a   | Nouveau processor `capped`; renommage `output_nbt` → `block_entity_modifier`.                             | ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_structure)) |

---

## 8 | Bonnes pratiques & conseils

- **Découplage** : un même `structure` peut être référencé par plusieurs `structure_set` avec des placements distincts (monde Overworld vs. dimension custom).
- **Pools atomiques** : gardez des `template_pool` petits (≤ 10 éléments) et imbriquez‑les pour limiter la récursion.
- **Projections mixtes** : combinez `rigid` + `terrain_matching` dans un même pool pour des villages plus « organique ».
- **Frequency & performance** : utilisez `frequency` < 1.0 dans les `structure_set` denses pour réduire les essais à vide.
- **Débogage rapide** : `/place jigsaw <pool> ~~~` et `/locate structure <id>` permettent de valider en jeu les hauteurs et collision.

---

## Références

- “Custom structure.” *Minecraft Wiki* (consulté le 10 juillet 2025).


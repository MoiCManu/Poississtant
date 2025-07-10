# Génération de monde personnalisée (Minecraft Java) – Base de connaissances

> **Objectif** : fournir une documentation exhaustive et à jour sur le système *data‑driven* de génération de mondes (world‑gen) introduit depuis la 1.16.2 et enrichi jusqu’à la 1.20+, à destination des développeurs de data packs, map‑makers et équipes serveur.

## Sommaire

1. [Architecture générale](#architecture-générale)
2. [World Presets & Dimensions](#world-presets--dimensions)
3. [Noise Settings](#noise-settings)
4. [Density Functions & Noises](#density-functions--noises)
5. [Biomes personnalisés](#biomes-personnalisés)
6. [Carvers : grottes & canyons](#carvers-grottes--canyons)
7. [Features & Placed Features](#features--placed-features)
8. [Structures data‑driven](#structures-data-driven)
9. [Surface Builder (héritage)](#surface-builder-héritage)
10. [Historique des évolutions](#historique-des-évolutions)
11. [Bonnes pratiques](#bonnes-pratiques)
12. [Glossaire & références](#glossaire--références)
13. [Conclusion](#conclusion)

---

## Architecture générale

Le pipeline world‑gen repose sur une **pile de registres** placés sous le préfixe `minecraft:worldgen/*`. Chaque registre est alimenté par des fichiers JSON d’un data pack et peut référencer les entrées des autres :

| Registre                                        | Rôle principal                                                              |
| ----------------------------------------------- | --------------------------------------------------------------------------- |
| `world_preset`                                  | Liste les dimensions disponibles pour le monde chargé                       |
| `dimension`                                     | Définit le *chunk generator* (noise / superflat / debug) et ses dépendances |
| `noise_settings`, `density_function`, `noise`   | Assurent la forme du terrain                                                |
| `biome`, `configured_feature`, `placed_feature` | Contenu, climat & décorations                                               |
| `configured_structure_feature`, `structure_set` | Placement global des structures                                             |
| `flat_level_generator_preset`                   | Presets legacy du monde Superflat                                           |

Les registres sont chargés **dynamiquement** au démarrage du monde ; les IDs sont donc résolues entre elles à chaud ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## World Presets & Dimensions

### Fichier `worldgen/world_preset/<id>.json`

Depuis la 1.19, Mojang recommande de centraliser la liste des dimensions dans un *World Preset* unique ; le dossier `dimension/` reste toutefois compatible pour les packs plus anciens ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

```json
{
  "dimensions": {
    "minecraft:overworld": { "generator": { "type": "minecraft:noise", "settings": "my_pack:overworld" } },
    "minecraft:the_nether":  { "generator": { "type": "minecraft:noise", "settings": "my_pack:nether"  } }
  }
}
```

- **Affichage dans le menu** : ajouter l’ID du preset au tag `minecraft:normal` pour qu’il apparaisse dans la liste des *world types* et localiser `generator.<namespace>.<id>` dans le pack de langues.
- **Cas spéciaux** : `minecraft:flat` (Superflat) et `minecraft:single_biome_surface` (Buffet) conservent un écran de personnalisation dédié.

---

## Noise Settings

> Contrôle la topologie brute (montagnes, océans, grottes bruyantes) et la logique d’aquifères & veines de minerais.

### Emplacement

`data/<ns>/worldgen/noise_settings/<id>.json` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation/noise_settings))

### Champs clés

| Champ                                   | Description                                                | Plage / notes                         |
| --------------------------------------- | ---------------------------------------------------------- | ------------------------------------- |
| `sea_level`                             | Y de la mer (génération uniquement)                        | -63 ↔ 320                             |
| `disable_mob_generation`                | Bloque le spawn lors du *chunk populate*                   | Bool                                  |
| `ore_veins_enabled`, `aquifers_enabled` | Toggles veines & aquifères                                 | Bool                                  |
| `noise.min_y`                           | Base de calcul verticale                                   | multiple de 16, ≥ –2032               |
| `noise.height`                          | Épaisseur de calcul                                        | multiple de 16, `min_y+height ≤ 2032` |
| `noise_router`                          | Mappage entre *density functions* et paramètres génériques | Introduit 1.18.2 pre1                 |
| `surface_rule`                          | Règles de blocs de surface                                 | Voir section dédiée                   |
| `spawn_target`                          | Points climatiques recherchés pour le spawn joueur         | Ajout 22w11a                          |

Lorsque la **densité finale** < 0 → air + aquifère ; > 0 → bloc par défaut + `surface_rule` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation/noise_settings)).

#### Exemple minimal de Noise Settings

```json
{
  "sea_level": 63,
  "aquifers_enabled": true,
  "noise": {
    "min_y": -64,
    "height": 384,
    "size_horizontal": 1,
    "size_vertical": 2
  },
  "noise_router": {
    "final_density": "my_pack:final_density"
  },
  "surface_rule": {
    "type": "my_pack:custom_surface"
  }
}
```

---

## Density Functions & Noises

### Density Functions (`worldgen/density_function`)

Des **expressions mathématiques JSON** chaînables (add, mul, clamp, spline…) qui renvoient une densité flottante par position XYZ ; plus de 30 types sont disponibles ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

```json
{
  "type": "minecraft:add",
  "argument1": { "type": "minecraft:noise", "noise": "minecraft:continentalness" },
  "argument2": { "type": "minecraft:constant", "value": -0.11 }
}
```

- **Maker functions** : `interpolated`, `flat_cache`, `cache_2d`, etc.
- **Unary / binary** : `abs`, `square`, `min`, `max`, etc.
- **Spécialités** : `blend_*`, `beardifier`, `weird_scaled_sampler`, `shifted_noise`.

### Noises (`worldgen/noise`)

- définis une table *octaves/amplitudes* pour un **Improved Perlin 3D**.
- certains IDs ont un usage *hard‑coded* (`minecraft:surface`, `clay_bands_offset`, etc.) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## Biomes personnalisés

`worldgen/biome/<id>.json` — regroupe climat, visuels, spawns, carvers & features ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

```json
{
  "precipitation": "rain",
  "temperature": 0.25,
  "downfall": 0.8,
  "effects": { "sky_color": 7907327, "water_color": 4159204 },
  "carvers": { "air": [ "my_pack:strip_cave" ] },
  "features": [ [], [ "my_pack:flower_patch" ], [] ]
}
```

---

## Carvers : grottes & canyons

Trois types (`cave`, `nether_cave`, `canyon`). Chaque `configured_carver` définit probabilité, plage Y, `lava_level`, blocs remplaçables et un bloc *debug* ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## Features & Placed Features

- **Feature Type** → logique (arbre, ore, disk…).
- `configured_feature` stocke la configuration ;
- `placed_feature` ajoute les *placement modifiers* (heightmap, rarity\_filter, biome\_filter…).

Les features opèrent à l’échelle **3 × 3 chunks** et ne sont pas détectables par `/locate` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## Structures data‑driven

Cinq registres : `structure_feature`, `structure_template`, `structure_pool`, `configured_structure_feature`, `structure_set`. Les structures s’inscrivent dans une grille pseudo‑aléatoire mondiale et sont localisables via `/locate` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## Surface Builder (héritage)

Supprimé depuis la 1.18 ; remplacé par `surface_rule` + density functions. Utilisé uniquement pour les packs < 1.18 ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)).

---

## Historique des évolutions

| Version    | Instantanés   | Points clés                                                                                                                                     |
| ---------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **1.16.2** | 20w28a        | Support initial du world‑gen custom ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation))                         |
| **1.18.2** | 22w06a → pre1 | Ajout `noise_router`, séparations des settings de structure ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)) |
| **1.19**   | 22w11a‑12a    | Nouveau champ `spawn_target`, migration vers `world_preset` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation)) |
| **1.19.3** | 22w42a        | Fichiers vanilla déplacés dans `client.jar` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation))                 |

---

## Bonnes pratiques

- **Outils** : générateurs & validateurs de Misode pour respecter les schémas.
- Toujours garder un **preset fallback** (`minecraft:normal`) pour l’import de vieux mondes.
- Versionner vos data packs et documenter les changements ; les schémas évoluent quasiment chaque release.
- Garder l’ordre des 11 niveaux de features **identique** entre tous les biomes d’une dimension.

---

## Glossaire & références

- **Data pack** : conteneur ZIP décompressé lu après `resources.zip` de %appdata%.minecraft — permet de surcharger tags, loot tables, world‑gen, etc.
- **Chunk Generator** : composant C++/Java responsable de la fonte du terrain.
- **Aquifère** : système séparant les plans d’eau souterrains pour éviter des lacs incessants.

### Pages Wiki consultées

1. Custom world generation – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation))
2. Custom world generation / noise\_settings – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation/noise_settings))
3. Density function – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))

---

## Conclusion

Le socle *data‑driven* de **Minecraft Java** offre aux créateurs une **granularité sans précédent** : chaque composant – du profil climatique d’un biome à la courbe de densité d’une grotte – est désormais pilotable par JSON. En maîtrisant la chaîne World Gen, vous disposerez d’un véritable moteur de génération procédurale, capable de produire des expériences totalement inédites ou de répliquer fidèlement un design existant.

> *Dernière mise à jour : 10 juillet 2025.*


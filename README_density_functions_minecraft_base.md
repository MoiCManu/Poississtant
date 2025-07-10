# Density Functions (Minecraft Java) – Base de connaissances

> **Objectif :** analyser en profondeur le registre `worldgen/density_function`, cœur mathématique de la génération procédurale (World Gen) depuis la 1.18.2. Ce document s’adresse aux concepteurs de data packs avancés qui souhaitent composer ou optimiser des fonctions de densité afin de sculpter le relief, les grottes, les aquifères ou les veines de minerais.

## Sommaire

1. [Concept & rôle](#concept--rôle)
2. [Structure JSON de base](#structure-json-de-base)
3. [Typologie complète des density functions](#typologie-complète-des-density-functions)
   1. [Maker / cache](#maker--cache)
   2. [Fonctions unaires](#fonctions-unaires)
   3. [Fonctions binaires](#fonctions-binaires)
   4. [Fonctions spéciales](#fonctions-spéciales)
   5. [Fonctions retirées](#fonctions-retirées)
4. [Chaîner plusieurs density functions : exemples](#chaîner-plusieurs-density-functions--exemples)
5. [Intégration dans ](#intégration-dans-noise_router)[`noise_router`](#intégration-dans-noise_router)
6. [Historique des versions](#historique-des-versions)
7. [Bonnes pratiques](#bonnes-pratiques)
8. [Glossaire & références](#glossaire--références)
9. [Conclusion](#conclusion)

---

## Concept & rôle

Une **density function** est une **expression mathématique** évaluée en continu pour chaque position XYZ pendant la génération de chunk. La valeur finale influence ensuite :

- le **Bloc placé** (densité < 0 → air/eau, densité ≥ 0 → pierre & dérivés) ;
- la **forme des grottes** via les carvers (`noise_router` → `cave layers`)
- la **répartition des biomes 3D** et des **veines de minerai**.

Les fichiers se situent dans `data/<namespace>/worldgen/density_function/<id>.json` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

---

## Structure JSON de base

```json
{
  "type": "minecraft:add",            // ID de fonction
  "argument1": { ... },               // Peut être : ID d’une autre density, objet JSON ou valeur constante
  "argument2": { ... }
}
```

> Format abrégé : si `type` = `constant`, le fichier peut se résumer à une **valeur numérique** (−1 000 000 ↔ +1 000 000) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

---

## Typologie complète des density functions

### Maker / cache

| ID                  | Description synthétique                                  | Note                                                                                                        |
| ------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `interpolated`      | Interpolation 3D par blocs dans une **cellule** de 4×4×4 | Combine souvent `flat_cache` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))   |
| `flat_cache`        | Calcule une fois par **colonne 4×4** à Y=0               | — ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                              |
| `cache_2d`          | Cache par **position XZ**                                | — ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                              |
| `cache_once`        | Mémoïse le résultat si référencé plusieurs fois          | — ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                              |
| `cache_all_in_cell` | Utilisé en dur pour `final_density`                      | Ne pas employer dans les packs ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)) |

### Fonctions unaires

`abs`, `square`, `cube`, `half_negative`, `quarter_negative`, `squeeze` – appliquent une **transformation** sur une seule entrée ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

### Fonctions binaires

`add`, `mul`, `min`, `max` – combinent **deux entrées** à chaque position ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

### Fonctions spéciales (sélection)

| ID                                             | Usage principal                             | Champs spécifiques                                                                                                                                           |
| ---------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `noise`                                        | Échantillonne un **Improved Perlin 3D**     | `noise`, `xz_scale`, `y_scale` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                  |
| `shifted_noise`                                | Idem mais avec **offsets** variables        | `shift_x`, `shift_y`, `shift_z` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                 |
| `range_choice`                                 | If‑Then‑Else numérique                      | `input`, `min_inclusive`, `max_exclusive`, `when_in_range`, `when_out_of_range` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)) |
| `spline`                                       | Courbe **cubic spline** multi‑points        | `coordinate`, `points[]` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                        |
| `blend_alpha`, `blend_offset`, `blend_density` | Transition **Legacy ↔ Caves & Cliffs**      | — ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                                               |
| `beardifier`                                   | Adoucit le contour des **structures**       | Ajouté à `final_density` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                        |
| `end_islands`                                  | Algorithme spécifique aux îles de l’End     | — ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                                               |
| `old_blended_noise`                            | Bruit legacy bêta                           | Multiples paramètres d’échelle ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                                  |
| `weird_scaled_sampler`                         | Amplifie ou réduit un bruit selon la rareté | `rarity_value_mapper`, `noise`, `input` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                         |
| `y_clamped_gradient`                           | Interpolation linéaire entre deux altitudes | `from_y`, `to_y`, `from_value`, `to_value` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                      |

### Fonctions retirées

`slide`, **« legacy spline »**, `terrain_shaper_spline` (supprimés snapshots 22w11a–22w12a) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)).

---

## Chaîner plusieurs density functions : exemples

### Relief continental simplifié

```json
{
  "type": "minecraft:add",
  "argument1": { "type": "minecraft:noise", "noise": "minecraft:continentalness", "xz_scale": 1.0, "y_scale": 0.33 },
  "argument2": { "type": "minecraft:constant", "value": -0.05 }
}
```

### Grottes « cheese & spaghetti »

```json
{
  "type": "minecraft:min",
  "argument1": "my_pack:spaghetti_2d",
  "argument2": {
    "type": "minecraft:mul",
    "argument1": "my_pack:cheese",
    "argument2": { "type": "minecraft:constant", "value": 2.2 }
  }
}
```

---

## Intégration dans `noise_router`

Dans un fichier **Noise Settings**, la clé `noise_router` fait correspondre des *slots* (`ridges`, `final_density`, `aquifer_barrier`, etc.) à des IDs de density functions ou à des objets inline. Exemple :

```json
"noise_router": {
  "final_density": "my_pack:final_density",
  "continentalness": "minecraft:continentalness",
  "erosion": { "type": "minecraft:noise", "noise": "my_pack:erosion" }
}
```

> Les density functions sont ainsi **compilées** puis **mémorisées** par Mojang + Indigo engine au démarrage du monde.

---

## Historique des versions

| Version         | Snapshots     | Changements majeurs                                                                                                                                                                       |
| --------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1.18.2 pre1** | 22w06a+       | Introduction du registre `density_function` et de la plupart des types actuels (`add`, `mul`, `noise`, etc.) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function)) |
| **1.19 pre1**   | 22w11a–22w12a | Suppression de `slide`, `terrain_shaper_spline`, refonte du spline legacy ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))                                    |

---

## Bonnes pratiques

- **Cache intelligemment** : combiner `flat_cache` + `interpolated` pour réduire le coût CPU des bruits 3D.
- **Limiter l’amplitude** : restez dans ±1 avant pondération pour éviter des discontinuités.
- **Versionner séparément** chaque density function : un simple renommage peut casser des presets existants.
- **Documenter les slots** utilisés dans `noise_router` pour faciliter la maintenance.

---

## Glossaire & références

- **Density Function** : expression numérique continue pilotant la densité du terrain.
- **Noise** : table d’octaves Perlin enregistrée dans `worldgen/noise`.
- **Noise Router** : mappage du fichier Noise Settings reliant les density functions aux étapes terrain.

### Sources consultées

1. Density function – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Density_function))
2. Custom world generation / noise\_settings – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation/noise_settings))

---

## Conclusion

Les **density functions** sont la **brique algorithmique fondamentale** de la génération 3D de Minecraft : elles transforment des bruits de base en reliefs crédibles, sculptent les cavernes et orchestrent la distribution des biomes. Une bonne maîtrise de leurs types et de leurs interactions, combinée à un usage réfléchi des caches, permet de créer des mondes ultracomplexes tout en maintenant des temps de génération raisonnables.

> *Dernière mise à jour : 10 juillet 2025.*


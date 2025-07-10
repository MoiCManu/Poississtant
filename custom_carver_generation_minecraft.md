# Custom Carver – Minecraft Java Edition

## Introduction
Les **carvers personnalisés** (« configured carvers ») pilotent l’excavation des grottes, canyons et autres volumes vides lors de la génération du terrain. Chaque carver est défini par un fichier JSON stocké dans le datapack :

```
<data‑pack>/data/<namespace>/worldgen/configured_carver/<id>.json
```

Le fichier référence :
1. **Le type de carver** (`minecraft:cave`, `minecraft:nether_cave`, `minecraft:canyon`).
2. **Un bloc `config`** décrivant les paramètres physiques, statistiques et de débogage.

---

## 1. Typologie des carvers
| ID | Particularités | Dimension(s) cible(s) |
|-----|---------------|-----------------------|
| `minecraft:cave` | Tunnels sinueux d’1 à 2 blocs de rayon pouvant se ramifier et percer jusqu’en surface. | Overworld |
| `minecraft:nether_cave` | Cavités plus larges, générées exclusivement dans le Nether ; toute zone située à `y < bottom_y + 32` est remplacée par de la lave. | Nether |
| `minecraft:canyon` | Ravins verticaux laissant le ciel visible ; orientation aléatoire, parois plus abruptes depuis 1.18. | Overworld |

> **Remarque :** Les anciens types `underwater_cave` & `underwater_canyon` ont été retirés en 21w06a.

---

## 2. Schéma JSON détaillé

### 2.1 Nœud racine
```json5
{
  "type": "minecraft:cave",         // Identifiant du carver
  "config": { ... }                  // Paramètres spécifiques
}
```

### 2.2 Champ `config` commun
| Champ | Type | Valeur par défaut | Description |
|-------|------|------------------|-------------|
| `probability` | float (0–1) | `0.14285715` (1/7) | Probabilité qu’un chunk tente ce carver. |
| `y` | *height_provider* | — | Distribution d’origine des points de départ (ex. `{"type":"uniform","min_inclusive":0,"max_inclusive":127}`). |
| `lava_level` | *vertical_anchor* | `128` | Niveau de lave pour boucher les poches d’air sous‑marines (ignoré par `nether_cave`). |
| `replaceable` | bloc / tag / liste | `#minecraft:terracotta` (canyon) | Collection de blocs pouvant être remplacés. |
| `debug_settings` | objet (opt.) | — | Échange les blocs générés pour faciliter le tracé. |
| `debug_mode` | bool | `false` | Active/désactive totalement le mode débogage. |
| `air_state`<br>`water_state`<br>`lava_state`<br>`barrier_state` | *block_state* | — | Substitution explicite des blocs d’air/eau/lave/barrière (très utile pour du world‑edit procédural). |

> **Astuce :** Pour neutraliser l’inondation des grottes, fixez `water_state` à `{ "Name": "minecraft:air" }`.

### 2.3 Paramètres spécifiques par type

#### 2.3.1 `cave` & `nether_cave`
| Champ | Type | Interprétation |
|-------|------|----------------|
| `yScale` | *float_provider* | Échelle verticale des sphéroïdes d’origine. |
| `horizontal_radius_multiplier` | *float_provider* | Rayon latéral des tunnels. |
| `vertical_radius_multiplier` | *float_provider* | Rayon vertical des tunnels. |
| `floor_level` | *float_provider* (‑1 à 1) | -1 = plancher concave / 0 = ellipsoïde / 1 = plancher plat. |

#### 2.3.2 `canyon`
| Champ | Type | Rôle |
|-------|------|------|
| `yScale` | *float_provider* | Hauteur max des canyons. |
| `vertical_rotation` | *float_provider* | Inflexion verticale le long de l’axe du canyon. |
| `shape.distance_factor` | *float_provider* | Longueur potentielle. |
| `shape.thickness` | *float_provider* | Diamètre (détermine largeur & profondeur). |
| `shape.horizontal_radius_factor` | *float_provider* | Multiplicateur de largeur. |
| `shape.vertical_radius_default_factor` | *float_provider* | Profondeur par défaut. |
| `shape.vertical_radius_center_factor` | *float_provider* | Profondeur au centre (permet des parois incurvées). |
| `shape.width_smoothness` | float > 0 | Anti‑aliasing des parois pour éviter les marches. |

---

## 3. Providers et ancres
### 3.1 *height_provider*
Définit une distribution verticale :
```json5
{"type":"uniform","min_inclusive":5,"max_inclusive":50}
```
Types disponibles : `constant`, `uniform`, `biased_to_bottom`, `trapezoid`.

### 3.2 *float_provider*
Représente une distribution flottante (0‑1 par défaut) :
```json5
{"type":"clamped","min":0.3,"max":1.2,"value":{"type":"uniform","min":0.5,"max":1.5}}
```

### 3.3 *vertical_anchor*
Ancre Y absolue ou relative :
```json5
{"type":"absolute","value":62}
{"type":"below_top","value":32}
```

---

## 4. Exemple complet – Carver « Canyon Mesa »
```json5
{
  "type": "minecraft:canyon",
  "config": {
    "probability": 0.2,
    "y": { "type": "uniform", "min_inclusive": 32, "max_inclusive": 120 },
    "yScale": { "type": "uniform", "min": 2.0, "max": 3.5 },
    "vertical_rotation": { "type": "constant", "value": 0.0 },
    "shape": {
      "distance_factor": { "type": "uniform", "min": 0.6, "max": 1.3 },
      "thickness": { "type": "uniform", "min": 0.7, "max": 1.1 },
      "horizontal_radius_factor": { "type": "constant", "value": 1.0 },
      "vertical_radius_default_factor": { "type": "constant", "value": 0.5 },
      "vertical_radius_center_factor": { "type": "constant", "value": 0.0 },
      "width_smoothness": 3.0
    },
    "lava_level": { "type": "absolute", "value": 11 },
    "replaceable": ["minecraft:red_sandstone", "minecraft:terracotta"],
    "air_state": { "Name": "minecraft:air" }
  }
}
```

> **Conseil :** Pour appliquer ce carver uniquement dans certains biomes, affectez-le à un tag `#<namespace>:mesa_canyon_carvers` puis référencez ce tag dans le champ `carvers.air` du biome personnalisé.

---

## 5. Débogage & performance
* **Debug Mode** remplace tous les blocs générés par des blocs de laine de couleurs différentes : un bon moyen de visualiser la géométrie sans relancer le client.
* Gardez `probability` ≤ 0.2 pour éviter de « percer » tous les chunks et briser les surfaces.
* Les *providers* sont mis en cache par seed ; évitez des distributions trop complexes pour limiter le coût CPU lors de la génération.

---

## 6. Historique des versions (rappel)
| Version | Snapshot / Pre‑release | Principales évolutions |
|---------|-----------------------|------------------------|
| **1.16.2** (20w28a) | Ajout des configured carvers expérimentaux. |
| **1.17** (21w06a) | Suppression des carvers sous‑marins. |
| 21w08a | Ajout des paramètres `shape.*` pour les canyons. |
| 21w11a | CamelCase → snake_case pour `distance_factor`. |
| 21w13a | Champs de configuration pour les grottes. |
| 21w16a | `aquifers_enabled` & `debug_settings`. |
| **1.18** (21w38a) | Suppression d’`aquifers_enabled`. |
| **1.19** (22w15a) | Introduction de `replaceable`. |

---

## 7. Notes complémentaires
- Les carvers fonctionnent **avant** les features de génération ; veillez à la cohérence des compositions de terrain.
- Les carvers personnalisés peuvent être chaînés. Ex. : un carver `canyon` suivi d’un carver `cave` dans le même chunk.
- **Bug connu MC‑237017** : `lava_level` ignoré dans certains contextes.

## Références
- “Custom carver.” *Minecraft Wiki* (consulté le 10 juillet 2025).


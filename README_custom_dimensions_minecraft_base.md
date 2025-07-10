# Dimensions personnalisées (Minecraft Java) – Base de connaissances

> **Objectif :** documenter en détail la création de **nouvelles dimensions** via data packs, en couvrant la structure des fichiers `dimension_type`, `dimension`, le rôle des world presets et les listes de biomes multi‑noise. Cette fiche s’adresse aux développeurs de data packs avancés, studios et administrateurs de serveurs.

## Sommaire

1. [Vue d’ensemble](#vue-densemble)
2. [Fichier ](#fichier-dimension_type)[`dimension_type`](#fichier-dimension_type)
   1. [Champs clés](#champs-clés)
   2. [Table des valeurs par défaut](#table-des-valeurs-par-défaut)
3. [Fichier ](#fichier-dimension)[`dimension`](#fichier-dimension)
4. [Multi‑noise Biome Source Parameter List](#multi-noise-biome-source-parameter-list)
5. [Historique des évolutions](#historique-des-évolutions)
6. [Bonnes pratiques](#bonnes-pratiques)
7. [Glossaire & références](#glossaire--références)
8. [Conclusion](#conclusion)

---

## Vue d’ensemble

Une **dimension personnalisée** se base sur deux registres :

| Dossier data pack                           | But                                                                                 | Chargement                                                                                                                   |
| ------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `worldgen/dimension_type/<id>.json`         | Définit les règles physiques et logiques (lumière, hauteur, coordinate scale, etc.) | Chargé dès le démarrage du monde ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                |
| `worldgen/dimension/<id>.json` *(héritage)* | Associe un `dimension_type` à un générateur de chunks                               | Dépriorisé lorsque **world preset** est présent ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)) |

> **Note :** Depuis la 1.19, si un *world preset* est utilisé (y compris celui par défaut), le dossier `dimension/` est **ignoré**. Les dimensions doivent alors être listées dans le fichier `world_preset`; le JSON `dimension` ne reste utile que pour les packs hérités ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension), [minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)).

---

## Fichier `dimension_type`

Chemin : `data/<namespace>/dimension_type/<name>.json`.

### Champs clés

| Clé                                                             | Type                        | Description / plage                                                                                                                                    |
| --------------------------------------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ultrawarm`                                                     | Bool                        | Comportement *Nether* : évaporation de l’eau, diffusion rapide de la lave ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)) |
| `natural`                                                       | Bool                        | Active la boussole et les lits (sinon désactivés) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                         |
| `coordinate_scale`                                              | Double 0.00001 → 30 000 000 | Multiplicateur appliqué lors de la sortie de la dimension ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                 |
| `has_skylight`                                                  | Bool                        | Présence de lumière naturelle du ciel ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                     |
| `has_ceiling`                                                   | Bool                        | Bedrock au plafond (logique uniquement) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                   |
| `ambient_light`                                                 | Float 0 → 1                 | Éclairement ambiant ; 0 = suivant la lumière, 1 = plein jour permanent ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))    |
| `fixed_time`                                                    | Int (ticks)                 | Fige l’heure du jour (ex. 18000 = minuit) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                 |
| `monster_spawn_light_level`                                     | Int / Range 0‑15            | Lumière effective maximale pour le spawn hostile ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                          |
| `monster_spawn_block_light_limit`                               | Int 0‑15                    | Lumière de bloc maximale pour le spawn hostile ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                            |
| `piglin_safe`, `bed_works`, `respawn_anchor_works`, `has_raids` | Bool                        | Comportements spécifiques Nether/End ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                      |
| `logical_height`                                                | Int ≤ `height`              | Limite Y pour chorus fruit & portails ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                     |
| `min_y`, `height`                                               | Int multiples de 16         | Gamme verticale totale (‑2032 ↔ 2031) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                     |
| `infiniburn`                                                    | Tag bloc                    | Blocs où le feu brûle indéfiniment ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                        |
| `effects`                                                       | ID                          | Effets visuels (`overworld`, `the_nether`, `the_end`) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                     |

#### Exemple minimal

```json
{
  "ultrawarm": false,
  "natural": true,
  "coordinate_scale": 1.0,
  "ambient_light": 0.0,
  "min_y": -64,
  "height": 384,
  "logical_height": 256,
  "piglin_safe": false,
  "bed_works": true,
  "respawn_anchor_works": false,
  "has_ceiling": false,
  "has_skylight": true,
  "effects": "minecraft:overworld"
}
```

### Table des valeurs par défaut

| Propriété                                                                    | Overworld               | Nether               | End               | Overworld Caves         |
| ---------------------------------------------------------------------------- | ----------------------- | -------------------- | ----------------- | ----------------------- |
| `ultrawarm`                                                                  | false                   | true                 | false             | false                   |
| `natural`                                                                    | true                    | false                | false             | true                    |
| `coordinate_scale`                                                           | 1.0                     | 8.0                  | 1.0               | 1.0                     |
| `piglin_safe`                                                                | false                   | true                 | false             | false                   |
| `respawn_anchor_works`                                                       | false                   | true                 | false             | false                   |
| `bed_works`                                                                  | true                    | false                | false             | true                    |
| `has_raids`                                                                  | true                    | false                | true              | true                    |
| `has_skylight`                                                               | true                    | false                | false             | true                    |
| `has_ceiling`                                                                | false                   | true                 | false             | true                    |
| `fixed_time`                                                                 | N/A                     | 18000                | 6000              | N/A                     |
| `ambient_light`                                                              | 0.0                     | 0.1                  | 0.0               | 0.0                     |
| `logical_height`                                                             | 256                     | 128                  | 256               | 256                     |
| `infiniburn`                                                                 | `#infiniburn_overworld` | `#infiniburn_nether` | `#infiniburn_end` | `#infiniburn_overworld` |
| `effects`                                                                    | overworld               | the\_nether          | the\_end          | overworld               |
| ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)) |                         |                      |                   |                         |

---

## Fichier `dimension`

Chemin historique : `data/<namespace>/dimension/<name>.json`.

```json
{
  "type": "my_pack:skylands_type",
  "generator": {
    "type": "minecraft:noise",
    "seed": 0,
    "settings": "my_pack:skylands_noise_settings"
  }
}
```

- `` : ID pointant vers un `dimension_type` (custom ou vanilla : `minecraft:overworld`, `minecraft:the_nether`, etc.) ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)).
- `` : structure identique à celle décrite dans [Noise Settings](#fichier-dimension) du guide *World Gen* ; accepte `noise`, `flat`, `debug`.
- **Compatibilité :** si un *world preset* existe, les entrées de ce dossier ne sont plus chargées. Mettez alors vos dimensions dans la clé `dimensions` du preset. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))

---

## Multi‑noise Biome Source Parameter List

Chemin : `worldgen/multi_noise_biome_source_parameter_list/<name>.json` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)).

Utilisé lorsque le générateur est `noise` **et** le biome source `multi_noise` ; il permet de **décorréler** la liste des biomes de la définition de la dimension ou du preset, simplifiant l’ajout de nouveaux biomes dans des packs expérimentaux.

### Champ principal

| Clé      | Type                            | Rôle                                                                  |
| -------- | ------------------------------- | --------------------------------------------------------------------- |
| `preset` | String (`overworld` / `nether`) | Référence un preset codé en dur, évitant de fournir la liste complète |

---

## Historique des évolutions

| Version                | Instantanés        | Évolutions majeures                                                                                                                                                                                              |
| ---------------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1.16 Pre‑release 1** | —                  | Introduction des dossiers `dimension` et `dimension_type` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                                                           |
| **1.16.2**             | 20w29a             | Déplacement des `noise settings` vers `worldgen` dans les data packs ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                                                |
| **1.17**               | 20w49a             | Ajout de `min_y` et `height` dans `dimension_type` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                                                                  |
| **1.19**               | 22w11a / 1.19‑pre1 | Suppression du champ `seed` du générateur, `dimension_type` doit être référencé (plus inline) ; ajout des limites de spawn hostiles ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension)) |
| **1.19.4‑pre1**        | —                  | Arrivée de `multi_noise_biome_source_parameter_list` ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))                                                                                |

---

## Bonnes pratiques

- **Versionner** vos `dimension_type` à chaque snapshot : de nouveaux booléens sont régulièrement ajoutés.
- Tester `coordinate_scale` avec précaution : une valeur < 1 rétrécit la dimension, > 1 l’étend – impact direct sur les portails.
- Fournir un `logical_height` cohérent lorsque `min_y` / `height` sont modifiés pour éviter les téléportations invalides.
- Pour les mondes multivers : créer un *world preset* maître listant toutes vos dimensions et conserver les JSON `dimension` uniquement pour rétro‑compatibilité.

---

## Glossaire & références

- **Dimension Type** : ensemble de règles physiques (ciel, Portails, spawn) appliquées à tous les chunks d’une dimension.
- **World Preset** : registre pilotant la liste des dimensions disponibles dans un monde.

### Sources consultées

1. Custom dimension – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension), [minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_dimension))

---

## Conclusion

Les **dimensions personnalisées** étendent la génération procédurale en offrant un cadre complet pour créer des réalités parallèles : mondes à basse gravité, abysses sans ciel, ou cieux d’îles volantes. Chaque paramètre – du *coordinate scale* aux cycles jour/nuit figés – est ajustable via JSON, rendant possible des expériences de jeu inédites ou la reproduction fidèle de mécaniques d’autres titres.

> *Dernière mise à jour : 10 juillet 2025.*


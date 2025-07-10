# World Presets personnalisés (Minecraft Java) – Base de connaissances

> **Objectif :** décrire de façon claire et exhaustive le registre `worldgen/world_preset`, pivot de la liste des **dimensions** d’un monde. Ce guide s’adresse aux créateurs de data packs, studios et administrateurs de serveurs qui souhaitent contrôler les dimensions accessibles à la création d’un monde ou proposer des *world types* personnalisés dans l’interface **Créer un nouveau monde**.

## Sommaire

1. [Concept & rôle](#concept--rôle)
2. [Structure JSON détaillée](#structure-json-détaillée)
   1. [Clé ](#clé-dimensions)[`dimensions`](#clé-dimensions)
   2. [Exemple minimal](#exemple-minimal)
3. [Intégration UI : Tags & localisation](#intégration-ui--tags--localisation)
4. [Compatibilité avec le dossier ](#compatibilité-avec-le-dossier-dimension)[`dimension/`](#compatibilité-avec-le-dossier-dimension)
5. [World Presets « avec réglages »](#world-presets-avec-réglages)
   1. [Superflat & Flat Level Generator Preset](#superflat--flat-level-generator-preset)
   2. [Single Biome / Buffet](#single-biome--buffet)
6. [Historique des versions](#historique-des-versions)
7. [Bonnes pratiques](#bonnes-pratiques)
8. [Glossaire & références](#glossaire--références)
9. [Conclusion](#conclusion)

---

## Concept & rôle

Un **world preset** est un enregistrement JSON qui **détermine la liste exacte des dimensions** qu’un nouveau monde possédera. Dans le menu *Créer un nouveau monde*, le bouton **World Type** (écran *More World Options…*) permet de basculer entre :

1. **Default** : aucun preset utilisé ; le jeu charge par défaut les trois dimensions vanilla (Overworld, Nether, End) mais lit également le dossier `dimension/` du data pack pour les modifier ou en ajouter (impossible d’en supprimer).
2. Un **preset** listé dans `minecraft:normal` ou `minecraft:extended` : la liste des dimensions est alors MAÎTRISÉE exclusivement par le fichier `world_preset` ; le dossier `dimension/` est ignoré. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset))

> **Important :** revenir sur « Default » après avoir choisi un preset active en réalité le preset `minecraft:normal`, équivalent à l’état par défaut sans data pack. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset))

---

## Structure JSON détaillée

Les presets sont stockés dans `data/<namespace>/worldgen/world_preset/<id>.json`.

```json
{
  "dimensions": {
    "minecraft:overworld": {
      "generator": {
        "type": "minecraft:noise",
        "settings": "my_pack:overworld_noise_settings"
      }
    },
    "minecraft:the_nether": {
      "generator": {
        "type": "minecraft:noise",
        "settings": "minecraft:nether"
      }
    },
    "my_pack:void": {
      "generator": {
        "type": "minecraft:noise",
        "settings": "my_pack:void_noise_settings"
      }
    }
  }
}
```

### Clé `dimensions`

- **Obligatoire** ; doit contenir au moins `minecraft:overworld`.
- Chaque entrée (\<dimension name>) référence soit un **générateur inline** (comme ci‑dessus) soit une **dimension existante** dans le registre `worldgen/dimension/` (héritage).
- L’ordre n’a pas d’impact, mais par convention **Overworld** est listé en premier.

### Exemple minimal

```json
{
  "dimensions": {
    "minecraft:overworld": {
      "generator": { "type": "minecraft:noise", "settings": "minecraft:overworld" }
    }
  }
}
```

---

## Intégration UI : Tags & localisation

Pour qu’un preset apparaisse dans le bouton **World Type** :

1. Ajouter son ID dans le tag `data/<ns>/tags/worldgen/world_preset/minecraft/normal.json` :
   ```json
   { "values": [ "my_pack:skylands" ] }
   ```
2. Facultativement, l’inclure dans `minecraft:extended` (visible en maintenant **Alt**).
3. Définir le texte localisé `generator.<namespace>.<id>` dans le language file d’un resource pack :
   ```
   generator.my_pack.skylands=Skylands (Data Pack)
   ```

Sans ces étapes, le preset reste chargé mais **invisible** dans l’interface. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset))

---

## Compatibilité avec le dossier `dimension/`

| Version | Comportement                                                                                                                                                                                                                                                     |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| < 1.19  | Les fichiers `dimension/<id>.json` étaient l’unique méthode pour ajouter des dimensions.                                                                                                                                                                         |
| ≥ 1.19  | Les presets sont **recommandés** ; si un preset est choisi, **le dossier **``** n’est plus lu**. Les packs hérités continuent de fonctionner tant qu’aucun preset n’est utilisé. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset)) |

---

## World Presets « avec réglages »

Certaines entrées vanilla disposent d’un **écran de personnalisation dédié** pour l’Overworld :

| Preset ID                        | Écran *Customize*       | Registre associé                       |
| -------------------------------- | ----------------------- | -------------------------------------- |
| `minecraft:flat`                 | Superflat Customization | `worldgen/flat_level_generator_preset` |
| `minecraft:single_biome_surface` | Buffet (Single Biome)   | —                                      |

### Superflat & Flat Level Generator Preset

- Le bouton *Presets* ouvre la liste des **Flat Level Generator Presets**.
- Fichiers : `worldgen/flat_level_generator_preset/<name>.json`.
- Clés : `display` (item ID), `settings` (bloc + hauteur + structures).
- Tag d’exposition : `minecraft:visible`.
- Texte de langue : `flat_world_preset.<ns>.<name>`.

### Single Biome / Buffet

- Aucune entrée JSON supplémentaire ; l’écran propose simplement le choix d’un **biome unique** pour l’Overworld.

---

## Historique des versions

| Version           | Instantané(s) | Changements clés                                                                                                                                                                                                  |
| ----------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1.19 (22w11a)** | 22w11a        | Introduction des registres `world_preset` et `flat_level_generator_preset`. Le menu *World Type* devient pilotable par data pack. ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset)) |

---

## Bonnes pratiques

- Garder un **preset de repli** (`minecraft:normal`) dans vos packs complexes pour éviter les corruptions de mondes antérieurs.
- Localiser systématiquement les presets pour éviter un identifiant brut peu lisible.
- Lors d’un univers multi‑dimensions, versionner simultanément le preset et les `dimension_type` pour garantir la cohérence entre patchs.
- Documenter la dépendance entre **world\_preset** et **tags** ; un oubli et l’utilisateur ne verra jamais votre preset.

---

## Glossaire & références

- **World Preset** : registre listant les dimensions initiales d’un monde.
- **Flat Level Generator Preset** : sous‑registre définissant les couches d’un monde Superflat.
- **Buffet** : ancien terme pour « Single Biome ».

### Pages Wiki consultées

1. Custom world preset – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_preset))
2. World preset – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/World_preset))
3. Custom world generation – Minecraft Wiki ([minecraft.fandom.com](https://minecraft.fandom.com/wiki/Custom_world_generation))

---

## Conclusion

Le registre `` offre un mécanisme simple mais puissant pour **verrouiller la liste des dimensions** d’un monde et proposer des *world types* sur‑mesure dans l’interface vanilla. Combiné aux registres `dimension_type` et `noise_settings`, il devient l’élément central d’un multivers cohérent, qu’il s’agisse d’ajouter un Void dimensionnel, un Overworld céleste ou un Nether alternatif.

> *Dernière mise à jour : 10 juillet 2025.*


# Poississtant

**Extension RPG complète pour Minecraft Java Edition 1.20.1**

Poississtant transforme votre expérience Minecraft en un véritable RPG immersif, avec progression de personnage, PNJ interactifs, quêtes, économie, HUD custom et bien plus.

---

## 📋 Table des matières

1. [Objectifs](#objectifs)
2. [Fonctionnalités principales](#fonctionnalités-principales)
3. [Arborescence du dépôt](#arborescence-du-dépôt)
4. [Installation](#installation)
5. [Configuration & Utilisation](#configuration--utilisation)
6. [Exemples JSON natifs (noise\_settings)](#exemples-json-natifs-noise_settings)
7. [Contribuer](#contribuer)
8. [Licence](#licence)

---

## 🎯 Objectifs

* **Progression de personnage** : classes, compétences et statistiques évolutives
* **PNJ interactifs** : dialogues, quêtes et boutiques personnalisées
* **Exploration enrichie** : structures, biomes et loots sur mesure
* **Interface RPG** : HUD dédié, inventaires améliorés, mini-cartes
* **Ambiance immersive** : textures, sons et animations dédiés

---

## ✨ Fonctionnalités principales

1. **Datapacks**

   * Débute dans le dossier `DimDatapack v4/`
   * Génération et gestion de **dimensions** personnalisées
   * Système de classes (guerrier, mage, voleur, etc.)
   * Arbre de compétences et progression par points
   * Quêtes dynamiques avec suivi et dialogue
2. **Resource-packs**

   * Textures custom pour blocs, items et UI
   * Sons d’ambiance et musiques thématiques
3. **Scripts & commandes**

   * Automatisation via `execute`, `scoreboard`, `data`
   * Événements sur mesures (boss, invasions, mini-jeux)
4. **Mods requis** (Forge 1.20.1)

   * GeckoLib, AzureLib, Curios, KubeJS, etc.
5. **Économie & HUD**

   * Monnaie interne et système de boutique
   * Affichage en temps réel des stats (santé, mana, XP…)

---

## 🗂️ Arborescence du dépôt

```plaintext
Poississtant/
├── DimDatapack v4/         # Datapack principal: création de dimensions
│   ├── data/
│   │   ├── <namespace>/
│   │   │   ├── dimension/
│   │   │   │   ├── <dimension_name>.json
│   │   │   │   └── ...
│   │   │   ├── worldgen/
│   │   │   │   ├── noise_settings/
│   │   │   │   │   ├── example1.json
│   │   │   │   │   ├── example2.json
│   │   │   │   └── ...
│   │   │   └── ...
│   └── pack.mcmeta        # Métadonnées du datapack
├── resourcepack/           # Textures, sons, modèles, UI
├── scripts/                # Fichiers KubeJS & JSON fonctionnels
├── mods/                   # Liste des mods Forge nécessaires
├── noise_settings/         # Exemples de fichiers JSON “noise_settings” natifs Minecraft 1.20
├── pack.mcmeta             # Métadonnées du resource-/data-pack
└── README.md               # Ce fichier
```

---

**Note:** Le datapack `DimDatapack v4/` se concentre principalement sur la génération et la configuration de dimensions personnalisées. Vous y trouverez tous les JSON et ressources nécessaires pour définir de nouveaux mondes, biomes et structures volumétriques.

# Poississtant
# Projet_RPG_Poisson

Une extension complète pour transformer Minecraft Java Edition en un véritable RPG immersif, basée sur la version **1.20.1**.

---

## 📦 Objectif du projet

* Proposer une **progression de personnage** (compétences, métiers, statistiques).
* Intégrer des **PNJ interactifs** avec dialogues, quêtes et boutiques.
* Enrichir l’**exploration** par des structures, biomes et loots personnalisés.
* Offrir une **interface** repensée (HUD RPG, inventaires, cartes).
* Renforcer l’**immersion** via des textures, sons et animations dédiés.

---

## 🧰 Fonctionnalités principales

1. **Datapacks** :

   * Système de classes et compétences.
   * Quêtes dynamiques et gestion des dialogues.
2. **Resource packs** :

   * Textures personnalisées pour items, blocs et UI.
   * Sons et musiques spécifiques.
3. **Scripts & Commandes** :

   * Automatisation via `execute`, `scoreboard`, `data`, etc.
   * Configuration de loots et événements personnalisés.
4. **Mods requis** (Forge 1.20.1) :

   * Liste générique des dépendances dans `mods/`.
   * Bibliothèques : GeckoLib, AzureLib, Curios, KubeJS...
5. **Économie & HUD** :

   * Monnaie interne et magasins.
   * Affichage de stats en temps réel (RPG-HUD).

---

## 📂 Arborescence du dépôt

```plaintext
[NOM_DU_PROJET]/
├── datapacks/         # Modules de gameplay
├── resourcepack/      # Assets (textures, sons, modèles)
├── scripts/           # KubeJS & fonctions JSON
├── mods/              # Liste des mods Forge nécessaires
├── README.md          # Description du projet
└── pack.mcmeta        # Métadonnées du pack
```

---

## 🚀 Installation

1. Installer **Minecraft Java Edition 1.20.1** avec **Forge 1.20.1**.
2. Copier les dossiers :

   * `mods/` → `.minecraft/mods/`
   * `datapacks/` → `.minecraft/saves/<monde>/datapacks/`
   * `resourcepack/` → `.minecraft/resourcepacks/`
3. Activer les packs de données et de ressources via le menu du jeu.
4. Lancer Minecraft et profiter de l’expérience RPG.

---

## 📜 Licence

Ce projet est **open‑source** et disponible sous licence MIT. Pour toute réutilisation ou contribution, merci d’ouvrir une issue ou un pull request sur GitHub.

---

## 💡 Idée bonus

> Ajouter un **système de réputation** par faction : les actions du joueur influencent l’accueil des PNJ et les tarifs des marchands.

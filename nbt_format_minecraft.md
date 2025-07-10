# NBT Format – Minecraft Java & Bedrock

## Introduction

Le **Named Binary Tag (NBT)** est la structure arborescente utilisée par Minecraft pour sérialiser la quasi‑totalité de ses données : entités, blocs, chunks, inventaires, fichiers `level.dat`, etc. Chaque nœud (*tag*) possède :

- **ID** numérique (0‑12)
- **Nom** UTF‑8 (sauf TAG\_End)
- **Payload** (taille et type variant selon le tag)

Le même arbre peut être représenté **en binaire** (fichiers compressés GZip ou zlib) ou **en texte** (*SNBT*) pour les commandes et datapacks.

---

## 1 | Table des Tags (v19133)

| ID  | Tag                  | Payload                                 | SNBT suffix/expression | Intervalle / capacité                                 |
| --- | -------------------- | --------------------------------------- | ---------------------- | ----------------------------------------------------- |
|  0  | **TAG\_End**         | —                                       | —                      | Marqueur de fin d’un Compound ou type d’une List vide |
|  1  | **TAG\_Byte**        | 1 byte signé                            | `34b`, `true`/`false`  | −128 → 127                                            |
|  2  | **TAG\_Short**       | 2 bytes, big‑endian                     | `123s`                 | −32 768 → 32 767                                      |
|  3  | **TAG\_Int**         | 4 bytes, big‑endian                     | `314159`               | −2 147 483 648 → 2 147 483 647                        |
|  4  | **TAG\_Long**        | 8 bytes, big‑endian                     | `42l` / `42L`          | ±9.22 × 10¹⁸                                          |
|  5  | **TAG\_Float**       | 4 bytes IEEE‑754                        | `3.14f`                | ±3.4 × 10³⁸                                           |
|  6  | **TAG\_Double**      | 8 bytes IEEE‑754                        | `2.71828d`             | ±1.7 × 10³⁰⁸                                          |
|  7  | **TAG\_Byte\_Array** | taille (Int) + N bytes                  | `[B;1b,2b]`            | ≤ 2 147 483 639 éléments                              |
|  8  | **TAG\_String**      | UTF‑8 (Longueur (UInt16))               | `'text'` / `"text"`    | 65 535 bytes                                          |
|  9  | **TAG\_List**        | Type (Byte) + taille (Int) + N payloads | `[1,2,3]`              | Profondeur ≤ 512                                      |
|  10 | **TAG\_Compound**    | Tags complets + **TAG\_End**            | `{Key:Value}`          | Profondeur ≤ 512                                      |
|  11 | **TAG\_Int\_Array**  | taille (Int) + N Int                    | `[I;1,2]`              | ≤ 2 147 483 639 éléments                              |
|  12 | **TAG\_Long\_Array** | taille (Int) + N Long                   | `[L;1l,2l]`            | ≤ 2 147 483 639 éléments                              |

*Java utilise le big‑endian ; ****Bedrock**** encode tous les nombres en **little‑endian** et tolère parfois un ****TAG\_List**** comme racine.*

---

## 2 | Syntaxe SNBT (Stringified NBT)

```snbt
{Pos:[0.0d,64.0d,0.0d],CustomName:'{"text":"Steve"}',Invulnerable:1b}
```

- Les clés peuvent être non‑quotées si elles ne contiennent que `[A‑Za‑z0‑9_-.+]`.
- Les nombres portent un suffixe (`b,s,l,f,d`) sauf les Int et les doubles implicites.
- Les littéraux `true` et `false` se convertissent respectivement en `1b` et `0b`.
- Les tableaux typés :`[B;...]`, `[I;...]`, `[L;...]` sont distincts des List standards.

### Conversion JSON ↔ NBT

| JSON       | NBT résultant                                          |
| ---------- | ------------------------------------------------------ |
| `"string"` | String                                                 |
| `123`      | Byte / Short / Int / Long / Float / Double selon plage |
| `true`     | Byte `1b`                                              |
| `[1,2,3]`  | Liste ou tableau typé selon homogénéité                |
| `{}`       | Compound                                               |

---

## 3 | Structure binaire d’un fichier NBT

```
<Compression (optionnelle)>  ➜ GZip (.gz) ou zlib / raw
└─ TAG_Compound (root, nom="")
   ├─ <ID Byte>
   ├─ <Name Length UInt16> <Name UTF‑8>
   ├─ <Payload>
   └─ …
   └─ 0x00 (TAG_End)
```

*La plupart des fichiers (**`level.dat`**, chunks, templates) sont GZip. Quelques fichiers (ex. **`servers.dat`**, Bedrock **`level.dat`**) sont non‑compressés ou utilisent zlib.*

---

## 4 | Commandes & Chemins NBT

- ``, ``, `` permettent de lire, modifier, tester l’arbre.
- Les **NBT Paths** (`Pos[0]`, `Items[{Slot:0}].id`) ciblent des sous‑nœuds au sein d’un Compound.

---

## 5 | Historique

| Version   | Snapshot   | Évolution                                                 |
| --------- | ---------- | --------------------------------------------------------- |
| **1.0.0** | 2011‑09‑28 | Implémentation initiale (tags 0‑10).                      |
| **1.2.1** | 12w07a     | Ajout **TAG\_Int\_Array** (ID 11).                        |
| **1.12**  | 17w18a     | Ajout **TAG\_Long\_Array** (ID 12).                       |
| **1.13**  | 18w03a     | SNBT accepté dans les datapacks via `/data` et `storage`. |
| **1.16**  | 20w17a     | Commande `data modify storage` & NBT Path complet.        |

---

## 6 | Bonnes pratiques

1. **Profondeur ≤ 512** pour éviter les crashes.
2. Toujours préciser le **namespace** dans les IDs (`minecraft:oak_log`).
3. Utiliser des suffixes corrects pour passer les sélecteurs NBT stricts.
4. Garder les List petites pour ne pas dépasser la limite JVM (\~2.1 G éléments).

---

## Références

- “NBT format.” *Minecraft Wiki* (consulté le 10 juillet 2025).


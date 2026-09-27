---
layout: default
title: "1. Algèbre de Boole et portes logiques"
parent: Cours
nav_order: 1
---

# 1. Algèbre de Boole et portes logiques
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Manipuler les opérateurs booléens de base et leurs lois.
- Comprendre pourquoi NAND (ou NOR) est dite "universelle".
- Construire toutes les portes de base à partir de NAND uniquement.

## 📚 Théorie

### Les opérateurs de base

Un signal numérique ne connaît que deux états : `0` et `1` (faux / vrai, bas /
haut). L'algèbre de Boole décrit comment combiner ces signaux :

| Opérateur | Symbole usuel | Table de vérité |
|---|---|---|
| NON (NOT) | `¬A` | `0→1`, `1→0` |
| ET (AND) | `A·B` | vrai seulement si A **et** B valent 1 |
| OU (OR) | `A+B` | vrai si A **ou** B (ou les deux) valent 1 |
| OU EXCLUSIF (XOR) | `A⊕B` | vrai si A et B sont **différents** |
| NON-ET (NAND) | — | NAND = NOT(AND) |
| NON-OU (NOR) | — | NOR = NOT(OR) |

### Lois utiles

- **Commutativité** : `A·B = B·A`, `A+B = B+A`.
- **Associativité** : `(A·B)·C = A·(B·C)`.
- **Distributivité** : `A·(B+C) = A·B + A·C`.
- **Lois de De Morgan** (essentielles) :
  - `NOT(A·B) = NOT(A) + NOT(B)`
  - `NOT(A+B) = NOT(A) · NOT(B)`

Les lois de De Morgan expliquent *pourquoi* NAND et NOR sont si particulières :
elles permettent de "retourner" un AND en OR (et inversement) simplement en
inversant les entrées et la sortie.

### Universalité de NAND

Une porte est dite **universelle** si on peut construire n'importe quelle
fonction booléenne en ne l'utilisant qu'elle, répétée autant de fois que
nécessaire. NAND (et NOR) sont universelles. C'est un fait fondamental de
l'électronique numérique : historiquement, de nombreuses familles de circuits
intégrés étaient fabriquées presque exclusivement à partir de portes NAND,
car c'est la porte la plus simple à réaliser avec des transistors.

Concrètement, cela signifie que :

- `NOT(A)` peut s'obtenir avec une seule NAND (en reliant intelligemment ses
  deux entrées).
- `AND`, `OR`, `NOR`, `XOR` peuvent tous se construire en combinant plusieurs
  NAND.

## 🛠️ Dans Digital Logic Sim

À partir de maintenant, **interdis-toi d'utiliser les portes AND, OR, NOT,
NOR, XOR fournies nativement par l'outil** (si ta version les propose). Tu ne
dois utiliser que la porte NAND fournie, puis construire tes propres puces
`NOT_MAISON`, `AND_MAISON`, `OR_MAISON`, etc. Elles constitueront ta
bibliothèque de base pour tout le reste du cours.

## ✏️ Exercices

**Exercice 1.1 — NOT depuis NAND.** Construis un inverseur (`NOT`) en
n'utilisant qu'une seule porte NAND. Table de vérité à respecter :

| A | S attendu |
|---|---|
| 0 | 1 |
| 1 | 0 |

**Exercice 1.2 — AND et OR depuis NAND.** Construis `AND_MAISON` et
`OR_MAISON`, chacun à deux entrées, en n'utilisant que des NAND (et ta puce
`NOT_MAISON` si besoin). Vérifie les 4 lignes de leur table de vérité
respective.

**Exercice 1.3 — XOR depuis NAND.** Construis `XOR_MAISON` en n'utilisant que
des NAND. Indice : une solution classique utilise exactement 4 portes NAND —
essaie de la retrouver en partant de la définition `A⊕B = A·¬B + ¬A·B` et des
lois de De Morgan, plutôt qu'en assemblant naïvement tes puces AND/OR/NOT
déjà construites (ce qui fonctionnerait aussi, mais utiliserait plus de portes).

## 🤔 Questions de réflexion

- Pourquoi NOR est-elle aussi une porte universelle ? Que changerait-il si tu
  reconstruisais tout ce chapitre en partant de NOR au lieu de NAND ?
- En quoi le fait que NAND soit universelle est-il lié aux lois de De Morgan ?
- Si tu devais fabriquer un circuit avec le moins de transistors physiques
  possible, pourquoi préférer partir de NAND plutôt que de fabriquer chaque
  porte séparément ?

## 🚀 Pour aller plus loin

Essaie de construire un multiplexeur 2-vers-1 (une sortie qui vaut `A` ou `B`
selon une entrée de sélection `SEL`) uniquement à partir de tes puces maison.
Tu en auras besoin, sous une forme plus grande, dès le chapitre 2.

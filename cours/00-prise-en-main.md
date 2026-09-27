---
layout: default
title: "0. Prise en main de Digital Logic Sim"
parent: Cours
nav_order: 0
---

# 0. Prise en main de Digital Logic Sim
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Installer et lancer Digital Logic Sim.
- Comprendre la notion centrale de l'outil : la **puce** (*chip*).
- Créer, tester et sauvegarder un premier circuit simple.

## 📚 Théorie

Digital Logic Sim (DLS) est un simulateur de circuits logiques créé par
Sebastian Lague. Son principe est simple mais puissant : tu places des portes
logiques de base sur un plan de travail, tu les relies avec des fils, et tu
peux **encapsuler** n'importe quel circuit que tu construis en une nouvelle
"puce" personnalisée, réutilisable ensuite comme un composant, exactement
comme une porte logique de base.

C'est ce mécanisme d'encapsulation qui va nous permettre de construire un CPU
entier : on ne câble jamais tout d'un coup, on construit des briques de plus
en plus grosses (porte → additionneur → ALU → CPU), chacune étant testée
indépendamment avant d'être réutilisée.

Concepts clés de l'outil (les noms exacts peuvent légèrement varier selon la
version) :

- **Pin d'entrée / de sortie** : point de connexion d'un circuit vers l'extérieur.
- **Fil (wire)** : relie une sortie à une ou plusieurs entrées.
- **Porte de base** : NAND, AND, OR, NOT, etc. — fournies nativement par l'outil.
- **Puce personnalisée (custom chip)** : un circuit que tu as construit, sauvegardé
  et qui apparaît ensuite dans ta bibliothèque de composants.
- **Bus** : un ensemble de fils regroupés (par ex. 8 fils pour une valeur 8 bits).

## 🛠️ Dans Digital Logic Sim

- Télécharge la dernière version depuis le dépôt officiel (voir
  [Ressources](../ressources/)) et lance l'exécutable correspondant à ton
  système d'exploitation.
- Explore l'interface : zone de travail, palette de portes de base, gestion des
  entrées/sorties du circuit courant.
- Familiarise-toi avec : ajouter une porte, tracer un fil, ajouter une entrée
  et une sortie nommées, lancer la simulation, basculer l'état d'une entrée en
  cliquant dessus.
- Repère la fonctionnalité de **création de puce** (souvent : sélectionner le
  circuit, lui donner un nom, il apparaît ensuite dans le menu des composants).

## ✏️ Exercices

**Exercice 0.1 — Premier circuit.** Construis un circuit à deux entrées `A` et
`B` et une sortie `S`, tel que `S` reproduise le comportement d'un OU exclusif
(XOR), en n'utilisant que des portes AND, OR et NOT fournies par l'outil.
Vérifie ta table de vérité :

| A | B | S attendu |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

**Exercice 0.2 — Encapsulation.** Transforme le circuit précédent en une puce
personnalisée nommée `XOR_MAISON`, avec deux entrées et une sortie clairement
nommées. Place ensuite deux instances de cette puce dans un nouveau circuit et
vérifie qu'elles se comportent de façon identique.

## 🤔 Questions de réflexion

- Pourquoi est-il utile de pouvoir encapsuler un circuit en composant réutilisable,
  plutôt que de toujours travailler avec des portes de base ?
- Que se passerait-il si tu modifiais la puce `XOR_MAISON` après l'avoir déjà
  utilisée ailleurs ? À ton avis, comment l'outil gère cette situation ?

## 🚀 Pour aller plus loin

Regarde s'il existe une fonctionnalité de bus / groupement de fils dans ta
version de l'outil : elle te fera gagner énormément de temps de câblage à
partir du chapitre 3.

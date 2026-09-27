---
layout: default
title: "3. Arithmétique binaire et ALU"
parent: Cours
nav_order: 3
---

# 3. Arithmétique binaire et ALU
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre la représentation en complément à deux.
- Comprendre comment on soustrait avec un additionneur.
- Construire une ALU 8 bits (addition / soustraction) avec drapeaux zero et
  carry.

## 📚 Théorie

### Représenter les nombres négatifs : le complément à deux

Un octet peut représenter 256 valeurs. Pour représenter aussi des nombres
négatifs, la quasi-totalité des ordinateurs utilisent le **complément à
deux** :

- Le bit de poids fort indique le signe (0 = positif, 1 = négatif).
- Pour obtenir `-N` à partir de `N` : on inverse tous les bits de `N` (complément
  à un), puis on ajoute 1.

Exemple sur 8 bits : `5 = 00000101`. Complément à un : `11111010`. On ajoute 1 :
`11111011`, qui représente donc `-5`.

L'intérêt majeur : **l'addition en complément à deux fonctionne exactement
comme une addition binaire normale**, sans circuit spécial. Un même
additionneur sert donc à additionner des nombres positifs et négatifs.

### Soustraire avec un additionneur

Puisque `A - B = A + (-B)`, et que `-B` s'obtient en inversant les bits de `B`
et en ajoutant 1, on peut réutiliser un additionneur pour soustraire, à
condition de :

1. Inverser chaque bit de `B` (une porte NOT par bit).
2. Injecter un `1` supplémentaire — astuce classique : utiliser l'entrée
   `Cin` de l'additionneur pour cela, plutôt qu'un composant "+1" séparé.

C'est pour cette raison que l'ALU n'a souvent besoin que d'**un seul**
additionneur, associé à un jeu de multiplexeurs qui choisissent d'inverser ou
non `B` et d'injecter ou non ce `1`.

### Drapeaux (flags)

Une ALU produit généralement, en plus du résultat, des **drapeaux** qui
résument une propriété du résultat, utiles ensuite pour les sauts
conditionnels (chapitre 7 / 9) :

- **Zero (Z)** : actif si le résultat vaut exactement 0.
- **Carry (C)** : actif s'il y a eu une retenue sortante sur le bit de poids
  fort (utile pour détecter un dépassement de capacité en non-signé).

## 🛠️ Dans Digital Logic Sim

Réutilise ton `ADD_8BITS` du chapitre 2. Tu vas l'entourer de multiplexeurs et
de portes pour en faire une ALU capable de faire deux opérations différentes
selon une entrée de sélection.

## ✏️ Exercices

**Exercice 3.1 — Inverseur conditionnel 8 bits.** Construis un circuit
`INV_COND` : 8 entrées de données, 1 entrée de contrôle `INVERSER`. Si
`INVERSER = 0`, les 8 sorties recopient les entrées telles quelles. Si
`INVERSER = 1`, les 8 sorties sont l'inverse bit à bit des entrées.

**Exercice 3.2 — ALU add/sub.** En combinant `ADD_8BITS`, `INV_COND` et
l'astuce du `Cin`, construis `ALU_8BITS` avec :
- deux entrées de 8 bits (`A`, `B`),
- une entrée de contrôle `SUB` (0 = addition, 1 = soustraction),
- une sortie de 8 bits (`S`),
- une sortie `CARRY`,
- une sortie `ZERO` (active si `S` vaut `00000000`).

Vérifie au minimum : `5 + 3 = 8`, `5 - 3 = 2`, `3 - 5` (résultat négatif en
complément à deux — que vaut-il en binaire ?), `5 - 5` (le drapeau `ZERO`
doit s'activer).

## 🤔 Questions de réflexion

- Pourquoi dit-on que le complément à deux "élimine le zéro négatif" (contrairement
  à d'autres représentations comme le "signe + magnitude") ? En quoi est-ce un
  avantage pour le matériel ?
- Le drapeau `ZERO` doit-il regarder un seul bit du résultat, ou tous les
  bits ? Comment construirais-tu ce circuit avec ce que tu connais déjà
  (chapitre 1) ?
- Si tu voulais ajouter une opération `AND` bit-à-bit à ton ALU en plus de
  add/sub, comment modifierais-tu ton circuit de sélection ?

## 🚀 Pour aller plus loin

Ajoute une entrée de sélection sur 2 bits (au lieu d'1) et une opération
`AND` et une opération `OR` bit-à-bit à ton ALU, sélectionnées par un
multiplexeur 4-vers-1 en sortie. Ce n'est pas requis par le jeu d'instructions
minimal du projet, mais te prépare bien si tu veux enrichir l'ISA plus tard.

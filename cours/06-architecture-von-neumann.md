---
layout: default
title: "6. Architecture Von Neumann"
parent: Cours
nav_order: 6
---

# 6. Architecture Von Neumann
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le modèle d'architecture Von Neumann et le cycle
  *fetch-decode-execute*.
- Construire une RAM adressable de 16 mots de 8 bits.
- Construire un compteur ordinal (PC) 4 bits avec incrémentation et
  chargement.

## 📚 Théorie

### Le modèle Von Neumann

Dans l'architecture dite "Von Neumann" (par opposition à l'architecture
Harvard, qui sépare mémoire programme et mémoire données), **instructions et
données partagent la même mémoire**. Le processeur ne fait donc jamais de
différence *a priori* entre "ceci est une instruction" et "ceci est une
donnée" — c'est uniquement le fait de le lire au bon moment, avec la bonne
intention, qui donne un sens à un mot mémoire. C'est aussi ce modèle que suit
notre projet (voir le [sujet du projet](../sujet-projet.html)).

### Le cycle fetch — decode — execute

Le fonctionnement d'un CPU Von Neumann se répète indéfiniment en trois
temps :

1. **Fetch** (chercher) : le CPU lit en mémoire, à l'adresse pointée par le
   **compteur ordinal** (*Program Counter*, PC), le mot qui représente la
   prochaine instruction, et le place dans le **registre d'instruction** (IR).
   Le PC est alors incrémenté pour pointer la prochaine instruction.
2. **Decode** (décoder) : le CPU interprète les bits de l'instruction pour
   déterminer quelle opération effectuer (voir chapitre 7 sur l'ISA).
3. **Execute** (exécuter) : le CPU effectue réellement l'opération
   (lire/écrire en mémoire, calculer avec l'ALU, modifier le PC pour un saut,
   etc.).

Ce cycle est justement ce que l'unité de contrôle (chapitre 8) devra
orchestrer, étape d'horloge par étape d'horloge.

### RAM adressable

Une mémoire vive (RAM) de `2ⁿ` mots nécessite une adresse de `n` bits pour
désigner un mot particulier. Une façon simple de la construire (pas la plus
efficace, mais la plus pédagogique) : un **décodeur** *n-vers-2ⁿ* (chapitre 2)
sélectionne, selon l'adresse, quel registre parmi `2ⁿ` doit être activé en
lecture ou écriture, chaque registre étant construit comme au chapitre 5.

### Compteur ordinal

Le PC est un registre spécial qui doit savoir faire deux choses :
- **s'incrémenter** de 1 à chaque instruction "normale" (pas de saut),
- **se charger** avec une valeur arbitraire (pour un saut, si tu implémentes
  l'extension bonus `JMP`/`JC`/`JZ` du chapitre 7).

## 🛠️ Dans Digital Logic Sim

Réutilise ton décodeur (chapitre 2), ton registre avec `LOAD` (chapitre 4) et
ton additionneur (chapitre 2, pour l'incrémentation).

## ✏️ Exercices

**Exercice 6.1 — Compteur ordinal 4 bits.** Construis `PC_4BITS` : une entrée
`INC` (incrémenter de 1 au prochain front d'horloge), une entrée `LOAD` avec
4 bits de donnée associés (charger une valeur arbitraire au prochain front),
une entrée `CLK`, une entrée `RESET` (remise à 0), une sortie 4 bits. Réfléchis
à ce qui doit se passer si `INC` et `LOAD` sont actifs en même temps — décide
d'une priorité claire et documente ton choix.

**Exercice 6.2 — Un mot de RAM adressable.** Construis un circuit avec un
seul registre 8 bits, mais entouré de la logique nécessaire pour qu'il ne
réponde (en lecture ou écriture) que lorsqu'une adresse donnée en entrée
correspond exactement à "son" adresse (par exemple `0000`).

**Exercice 6.3 — RAM 16×8.** En généralisant l'exercice 6.2 à 16 registres
et un décodeur 4-vers-16, construis `RAM_16X8` : une entrée d'adresse 4 bits,
une entrée de donnée 8 bits, une entrée `WRITE` (écrire au prochain front
d'horloge), une entrée `CLK`, une sortie 8 bits (valeur actuellement à
l'adresse donnée, en lecture permanente, indépendamment de `WRITE`).

## 🤔 Questions de réflexion

- Pourquoi la RAM doit-elle pouvoir être lue **en permanence** (sans front
  d'horloge), alors que l'écriture, elle, doit être synchronisée sur un front ?
- Que se passe-t-il, dans ton `RAM_16X8`, si tu changes l'adresse pendant que
  `WRITE=1`, juste avant un front d'horloge ? Est-ce un comportement voulu,
  ou une source de bug potentielle à surveiller au moment de l'assemblage
  final ?
- En quoi le modèle Von Neumann (mémoire unique) contraste-t-il avec une
  architecture Harvard ? Quel est l'avantage principal de chaque approche ?

## 🚀 Pour aller plus loin

Un vrai ordinateur a une mémoire bien plus grande que 16 mots, ce qui rend un
décodeur "à plat" (1 décodeur géant) impraticable. Cherche ce qu'est un
adressage par **matrice ligne/colonne** dans les puces de RAM réelles, et
pourquoi cela change la complexité du décodage.

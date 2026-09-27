---
layout: default
title: "8. Unité de contrôle"
parent: Cours
nav_order: 8
---

# 8. Unité de contrôle
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le rôle de l'unité de contrôle : transformer une instruction en
  séquence de signaux de commande.
- Comprendre la notion de micro-étape (*T-state*) et de compteur d'étapes.
- Construire un séquenceur capable de dérouler le cycle *fetch-decode-execute*
  du chapitre 6 pour chaque instruction du chapitre 7.

⏱️ C'est le chapitre le plus exigeant du cours : c'est ici que toutes les
briques précédentes (bus, registres, RAM, ALU, jeu d'instructions) doivent
être orchestrées ensemble. Prends ton temps, avance micro-étape par
micro-étape, et teste en isolant l'unité de contrôle avant de la brancher au
reste du CPU.

## 📚 Théorie

### Le problème à résoudre

Chaque instruction de ton ISA (chapitre 7) ne s'exécute pas en un seul coup
d'horloge : elle se décompose en plusieurs **micro-étapes** (souvent notées
`T1`, `T2`, `T3`...), chacune activant un sous-ensemble précis des signaux de
contrôle des chapitres 5 et 6 (`OUT_EN` de tel registre, `LOAD` de tel autre,
`WRITE` de la RAM, etc.).

Par exemple, les 3 premières micro-étapes sont **toujours les mêmes**, quelle
que soit l'instruction : ce sont les micro-étapes du **fetch** (chapitre 6) :
sortir le PC sur le bus, charger le MAR, lire la RAM et charger l'IR, tout en
incrémentant le PC. Ce n'est qu'à partir de la micro-étape suivante que le
comportement diverge selon l'opcode contenu dans l'IR — c'est la phase de
**decode/execute**.

### Le compteur d'étapes (ring counter)

Pour savoir "où on en est" dans l'exécution d'une instruction, on utilise un
**compteur d'étapes** : un compteur qui boucle sur un petit nombre de valeurs
(par exemple `T1 → T2 → T3 → T4 → T5 → T1 → ...`), avec une seule sortie
active à la fois — exactement comme un décodeur (chapitre 2) piloté par un
petit compteur. On l'appelle parfois *ring counter* (compteur en anneau) car
il "tourne en rond" indéfiniment.

### Deux approches classiques pour générer les signaux de contrôle

1. **Logique câblée (hardwired control)** : les signaux de contrôle sont
   produits directement par des portes logiques combinant l'étape actuelle
   (`T1`...`T5`) et l'opcode décodé. C'est l'approche historique du SAP-1 et
   la plus adaptée à un projet Digital Logic Sim.
2. **Micro-programmation (microcode)** : les signaux de contrôle sont stockés
   dans une petite mémoire ROM, adressée par l'étape et l'opcode, et lue à
   chaque micro-étape. Plus flexible pour des ISA complexes, mais plus lourd
   à mettre en place ici.

Ce cours te recommande la logique câblée pour ce projet, plus directement
accessible avec les briques déjà construites (décodeurs, portes AND/OR).

### La matrice de contrôle : une façon de raisonner, pas une solution

Une façon usuelle de raisonner sur ce problème consiste à dresser un tableau
à deux dimensions — les micro-étapes en lignes, les signaux de contrôle en
colonnes — et à cocher, pour chaque instruction, quels signaux doivent
s'activer à quelle étape. **Ce cours ne te fournit volontairement pas ce
tableau rempli** : c'est le cœur de l'exercice de ce chapitre. En revanche, il
te donne la méthode pour le construire toi-même (voir exercice 8.2).

## 🛠️ Dans Digital Logic Sim

Tu vas avoir besoin d'un compteur d'étapes (variante de ton `PC_4BITS` du
chapitre 6, en plus petit et qui boucle automatiquement), d'un décodeur
d'instruction (branché sur les 4 bits d'opcode de ton IR), et de portes
combinant étape + opcode pour produire chaque signal de contrôle.

## ✏️ Exercices

**Exercice 8.1 — Compteur d'étapes.** Construis un compteur en anneau `T1`
à `T5` (5 micro-étapes suffisent pour le jeu d'instructions minimal du
projet), avec une sortie par étape (une seule active à la fois), qui boucle
automatiquement à `T1` après `T5`, cadencé par la même horloge que le reste du
CPU.

**Exercice 8.2 — Construire ta matrice de contrôle sur papier.** Avant tout
câblage, dresse toi-même un tableau (étapes en lignes, signaux de contrôle en
colonnes : `PC_OUT`, `MAR_IN`, `RAM_OUT`, `IR_IN`, `PC_INC`, `IR_OUT`,
`MAR_IN` (bis pour l'opérande), `RAM_OUT`, `ACC_IN`, `B_IN`, `ALU_OUT`,
`ACC_IN`, `SUB`, `OUT_IN`, `HALT`... — adapte la liste à ta propre
implémentation) et remplis-le pour chacune des 5 instructions du jeu minimal
(`LDA`, `ADD`, `SUB`, `OUT`, `HLT`). Vérifie que les 3 premières étapes
(fetch) sont identiques pour toutes les instructions.

**Exercice 8.3 — Décodeur d'instruction.** Construis un décodeur qui prend
les 4 bits d'opcode de l'IR et active une seule ligne parmi les instructions
de ton jeu (réutilise ton décodeur du chapitre 2).

**Exercice 8.4 — Signaux de commande.** En combinant les sorties de ton
compteur d'étapes (8.1) et de ton décodeur d'instruction (8.3) avec des
portes AND/OR, génère chacun des signaux de contrôle listés dans ta matrice
(8.2). Teste chaque signal indépendamment (sans encore brancher le reste du
CPU) en simulant manuellement une étape et un opcode donnés, et vérifie que
le bon signal (et lui seul) s'active.

## 🤔 Questions de réflexion

- Pourquoi les 3 premières micro-étapes peuvent-elles être **indépendantes de
  l'opcode**, alors que les suivantes ne le sont pas ?
- Que se passe-t-il si deux signaux de contrôle qui ne devraient jamais être
  actifs en même temps (par exemple deux `OUT_EN` sur le même bus, voir
  chapitre 5) le sont malgré tout à cause d'une erreur dans ta matrice de
  contrôle ? Comment pourrais-tu t'en apercevoir en testant ?
- Pourquoi l'instruction `HLT` doit-elle agir sur l'horloge elle-même plutôt
  que sur un registre comme les autres instructions ?

## 🚀 Pour aller plus loin

Si tu as implémenté l'extension bonus du sujet (`STA`, `LDI`, `JMP`, `JC`,
`JZ`), reprends ta matrice de contrôle et ajoute les colonnes/lignes
nécessaires. Les sauts conditionnels demandent de connecter les drapeaux
`ZERO`/`CARRY` de l'ALU (chapitre 3) à la logique de contrôle : réfléchis à
quel moment du cycle ces drapeaux doivent être "figés" pour rester valides
jusqu'à l'étape où ils sont utilisés.

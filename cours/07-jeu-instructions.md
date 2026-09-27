---
layout: default
title: "7. Jeu d'instructions (ISA)"
parent: Cours
nav_order: 7
---

# 7. Jeu d'instructions (ISA)
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre ce qu'est une ISA (*Instruction Set Architecture*) et à quoi elle
  sert.
- Savoir décomposer une instruction en opcode et opérande.
- Écrire "à la main" (en binaire) un programme respectant le jeu d'instructions
  du [sujet du projet](../sujet-projet.html).

⚠️ Ce chapitre est **théorique et sur papier** : tu n'as rien à construire
dans Digital Logic Sim ici. C'est une étape de conception indispensable avant
le chapitre 8 (unité de contrôle), qui lui *implémente* ce que tu vas définir
ici.

## 📚 Théorie

### Qu'est-ce qu'une ISA ?

Le **jeu d'instructions** (*Instruction Set Architecture*, ISA) est le
"langage" que le processeur comprend nativement : la liste des opérations
qu'il sait exécuter, et la façon dont chacune est encodée en binaire. C'est
la frontière entre le matériel (le câblage de l'unité de contrôle) et le
logiciel (les programmes qu'on peut écrire).

### Format d'instruction du projet

Le [sujet du projet](../sujet-projet.html) définit précisément le format et
la liste d'instructions à respecter — relis-le si besoin. En résumé : chaque
instruction fait 8 bits, découpés en 4 bits d'**opcode** (le code de
l'opération) et 4 bits d'**opérande** (le plus souvent une adresse mémoire
sur 4 bits, ou inutilisé pour certaines instructions).

### Assembleur "à la main"

Avant de construire l'unité de contrôle, il est très utile d'écrire toi-même,
sur papier, un ou deux programmes complets en binaire — exactement ce que fera
plus tard un assembleur logiciel (que tu n'as pas à écrire pour ce projet).
Cela te force à bien comprendre le format d'instruction, et te donnera un
programme de test concret pour valider ton CPU au chapitre 9.

## ✏️ Exercices

**Exercice 7.1 — Retrouver le tableau du sujet.** Sans regarder à nouveau la
table d'opcodes du sujet du projet, essaie de la reconstituer de mémoire :
quelles instructions minimales un CPU "as simple as possible" doit-il avoir
pour être capable de faire *au moins* un calcul et afficher un résultat ?
Compare ensuite ta réponse à la table du sujet.

**Exercice 7.2 — Écrire un programme à la main.** En respectant strictement
le format et la table d'opcodes du sujet du projet, écris (sur papier ou dans
un fichier texte) le programme binaire complet, adresse par adresse, qui
calcule `(5 + 3) - 2` et l'affiche (indice : le sujet du projet te montre
déjà la structure attendue, à toi de vérifier que tu sais la reproduire et
l'expliquer instruction par instruction).

**Exercice 7.3 — Un second programme.** Écris un programme qui calcule
`(10 - 4) + 1` et l'affiche, en choisissant toi-même où placer les données en
mémoire (en évitant bien sûr d'écraser les instructions).

**Exercice 7.4 (bonus, si tu implémentes l'extension) — Une boucle.** Si tu
as prévu d'implémenter `JMP`, `JC`, `JZ`, essaie d'écrire un programme qui
compte de 0 à 5 puis s'arrête (affichant chaque valeur intermédiaire avec
`OUT`).

## 🤔 Questions de réflexion

- Pourquoi 4 bits d'opcode suffisent-ils pour ce projet ? Combien
  d'instructions différentes cela permet-il en théorie, et combien sont
  réellement utilisées par le jeu d'instructions minimal ?
- Que se passerait-il si tu essayais d'adresser plus de 16 mots de mémoire
  avec ce format d'instruction sur 8 bits ? Quelles solutions un vrai
  processeur utilise-t-il pour ce problème (tu peux chercher "instructions de
  longueur variable" ou "registres d'index" pour te donner une piste, sans
  avoir besoin de l'implémenter) ?
- Pourquoi est-il utile d'écrire un programme "à la main" avant même d'avoir
  construit l'unité de contrôle qui l'exécutera ?

## 🚀 Pour aller plus loin

Si tu es à l'aise avec un langage de script (Python, JavaScript...), tu peux
(facultativement, hors du périmètre strict du projet) écrire un petit
programme qui transforme un mnémonique lisible (`LDA 14`) en octet binaire —
c'est-à-dire un mini-assembleur. Ce n'est pas nécessaire pour valider le
projet, mais très formateur si le temps te le permet.

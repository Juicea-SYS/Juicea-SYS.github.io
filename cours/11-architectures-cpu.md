---
layout: default
title: "11. Comparer les architectures CPU"
parent: Cours
nav_order: 11
---

# 11. Comparer les différentes architectures CPU
{: .no_toc }

1. TOC
{:toc}

Ce chapitre est volontairement **théorique** : pas de nouveau circuit à
construire dans Digital Logic Sim, mais des exercices de comparaison et de
recherche pour situer ton CPU maison par rapport aux processeurs réels.

## 🎯 Objectifs

- Situer ton CPU "maison" par rapport aux grandes familles d'architectures.
- Comprendre la distinction CISC / RISC.
- Comprendre les notions de pipeline, exécution superscalaire, exécution
  dans le désordre, multicœur.
- Comprendre pourquoi plusieurs architectures (x86, ARM, RISC-V...)
  coexistent aujourd'hui.

## 📚 Théorie

### Largeur de mots, dans l'histoire

Les processeurs ont progressivement élargi leur largeur de mot : 4 bits
(Intel 4004, 1971), 8 bits (Intel 8080, Zilog Z80 — la famille dont s'inspire
ce cours), 16 bits (Intel 8086), 32 bits (la majorité des PC des années
1990-2000), puis 64 bits (quasi standard aujourd'hui). Le chapitre précédent
a détaillé pourquoi chaque saut a un coût matériel réel.

### CISC contre RISC

- **CISC** (*Complex Instruction Set Computer*) : instructions riches, parfois
  de longueur variable, pouvant combiner plusieurs opérations en une seule
  instruction (par exemple lire en mémoire *et* faire un calcul). Exemple
  historique et actuel : la famille x86 (Intel, AMD).
- **RISC** (*Reduced Instruction Set Computer*) : instructions simples, de
  longueur fixe, une seule opération par instruction (charger, calculer OU
  stocker, jamais combinés), avec un jeu d'instructions volontairement
  restreint. Exemples : ARM, RISC-V, MIPS.

Ton CPU maison, avec son format d'instruction fixe (8 bits) et ses
instructions à opération unique, est structurellement plus proche de la
philosophie RISC — même s'il ne s'agit pas formellement d'une architecture
RISC industrielle.

### Le pipeline

Un CPU réel ne traite pas une instruction du début à la fin avant de
commencer la suivante, contrairement à ton unité de contrôle du chapitre 8.
Il découpe l'exécution en étages (typiquement : fetch, decode, execute,
accès mémoire, écriture du résultat) et fait avancer plusieurs instructions
en parallèle, chacune à un étage différent, à la manière d'une chaîne de
montage. Cela augmente le débit d'instructions sans nécessiter d'augmenter la
fréquence d'horloge.

### Superscalaire et exécution dans le désordre

Les CPU modernes vont plus loin : ils peuvent lancer plusieurs instructions
par cycle d'horloge (architecture *superscalaire*), et réordonnancer
dynamiquement les instructions pour éviter d'attendre inutilement une donnée
pas encore disponible (*exécution dans le désordre*, ou *out-of-order*), tout
en garantissant, du point de vue du programme, un résultat identique à une
exécution strictement séquentielle.

### Le multicœur

Plutôt que de complexifier indéfiniment un seul cœur, on peut dupliquer
plusieurs cœurs, relativement plus simples, sur la même puce, permettant une
exécution réellement simultanée de plusieurs flux d'instructions
indépendants.

### Endianness

Quand un mot de plusieurs octets est stocké en mémoire, l'ordre des octets
peut varier : *little-endian* (octet de poids faible stocké en premier — x86,
la plupart des ARM par défaut) contre *big-endian* (octet de poids fort en
premier — certains systèmes historiques, certains protocoles réseau).

### Tableau comparatif

| Aspect | Ton CPU maison | CISC (ex. x86) | RISC (ex. ARM, RISC-V) |
|---|---|---|---|
| Longueur d'instruction | Fixe (8 bits) | Variable | Fixe |
| Complexité par instruction | Une opération simple | Peut en combiner plusieurs | Une opération simple |
| Exécution | Séquentielle, micro-étapes | Pipeline profond, souvent superscalaire, out-of-order | Pipeline, souvent superscalaire |
| Nombre de cœurs typique | 1 | Plusieurs | Plusieurs |

## ✏️ Exercices

**Exercice 11.1 — Classer ton propre CPU.** En reprenant le tableau
ci-dessus, positionne précisément ton CPU du chapitre 9 : longueur
d'instruction fixe ou variable, nombre moyen de micro-étapes par instruction,
présence ou non d'un pipeline.

**Exercice 11.2 (recherche) — Comparaison concrète.** Choisis un processeur
réel considéré CISC (ex. un x86 récent) et un processeur réel considéré RISC
(ex. un ARM de smartphone, ou une puce RISC-V), et compare leur nombre
approximatif d'instructions dans leur ISA, leur largeur de registre, leur
fréquence d'horloge. Note tes sources.

**Exercice 11.3 (réflexion) — ARM et batterie.** Pourquoi la plupart des
smartphones et appareils sur batterie utilisent-ils des architectures ARM
(RISC) plutôt que x86 (CISC) ? Quel lien vois-tu avec la consommation
énergétique et la complexité du décodage d'instruction ?

**Exercice 11.4 (réflexion, sans câblage) — Vers un pipeline à 2 étages.**
Reprends ton unité de contrôle du chapitre 8. Si tu devais transformer ton
CPU en une version pipelinée à 2 étages (fetch dans un cycle, decode+execute
dans le suivant, en chevauchant deux instructions), quels registres
supplémentaires faudrait-il insérer entre les étages, à ton avis ?

## 🤔 Questions de réflexion

- Le jeu d'instructions minimal de ce cours (proche RISC) a-t-il facilité ou
  compliqué la conception de ton unité de contrôle au chapitre 8, par rapport
  à un jeu d'instructions combinant plusieurs opérations par instruction
  (proche CISC) ?
- Le nombre de transistors n'a cessé d'augmenter (loi de Moore), alors que la
  fréquence d'horloge des processeurs plafonne depuis le milieu des années
  2000. Quel rapport vois-tu avec la généralisation du multicœur ?

## 🚀 Pour aller plus loin

Le jeu d'instructions RISC-V est public et bien documenté. Regarde ses
spécifications et compare son format d'instruction avec celui de ton propre
CPU : nombre de bits d'opcode, nombre de registres, types d'instructions
disponibles.

---
layout: default
title: "10. Passer à 64 bits"
parent: Cours
nav_order: 10
---

# 10. Passer à 64 bits : généraliser l'architecture
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre pourquoi élargir un CPU n'est pas qu'une histoire de "plus de fils".
- Généraliser tes circuits 8 bits à une largeur paramétrable, par composition.
- Comprendre les limites pratiques de la propagation de retenue à grande
  largeur, et les solutions utilisées par le matériel réel.
- Comprendre pourquoi un CPU 64 bits ne câble jamais un décodeur d'adresse "à
  plat" sur `2⁶⁴` cases.

## 📚 Théorie

### Généraliser par composition

Une bonne nouvelle d'abord : la plupart de tes circuits des chapitres 2 à 6
(additionneur, registre, bus, RAM, PC) sont conçus comme des motifs répétés
bit par bit, ou bloc par bloc. Passer de 8 à 16 bits, ce n'est donc pas
"tout reconstruire", mais **composer** deux blocs de 8 bits déjà existants
(exactement comme tu as composé des portes NAND pour faire un additionneur,
puis des additionneurs pour faire une ALU). C'est l'idée centrale de ce
chapitre.

Mais dès qu'on pousse cette généralisation jusqu'à 32 ou 64 bits, plusieurs
problèmes bien réels apparaissent — connus des concepteurs de processeurs
depuis les débuts de l'informatique.

### Le problème de la propagation de retenue

Ton `ADD_8BITS` (chapitre 2) est un additionneur à propagation de retenue
(*ripple-carry*) : la retenue doit traverser les 8 étages un par un. Le
délai total croît avec le nombre de bits. À 64 bits, ce délai deviendrait
largement trop long pour les fréquences d'horloge visées par un vrai
processeur. C'est pour cette raison que les CPU réels, au-delà d'une certaine
largeur, remplacent l'additionneur simple par des architectures comme
l'**additionneur à retenue anticipée** (*carry-lookahead*) ou l'**additionneur
à sélection de retenue** (*carry-select*), qui calculent la propagation de
retenue de façon plus indirecte pour réduire ce délai — au prix de plus de
portes.

### L'explosion de l'espace d'adressage

Une adresse sur 64 bits permet en théorie d'adresser `2⁶⁴` octets (environ 18
milliards de milliards d'octets). Aucun ordinateur ne câble un décodeur "à
plat" sur autant de lignes — ce serait totalement impraticable. La mémoire
réelle est organisée en une **hiérarchie** : matrices ligne/colonne au niveau
des puces de RAM (déjà évoqué au chapitre 6), et surtout un mécanisme de
**mémoire virtuelle / pagination** qui ne fait correspondre qu'une petite
partie de cet espace adressable théorique à la mémoire physique réellement
installée (quelques Go à quelques centaines de Go).

### Le coût physique de la largeur

Chaque doublement de largeur double (au minimum) le nombre de bascules par
registre, la largeur des bus, la taille des multiplexeurs — donc le nombre de
portes, la surface de silicium, et la consommation électrique.

### Une limite pratique de ce cours

Digital Logic Sim n'est pas conçu pour simuler efficacement des dizaines de
milliers de portes en temps réel. Construire un CPU 64 bits complet, avec
tous ses registres et sa RAM à cette largeur, y serait extrêmement lourd et
peu lisible. Ce chapitre te propose donc une démarche réaliste : **généraliser
réellement tes circuits jusqu'à 16 bits** (parfaitement faisable et
instructif), puis **raisonner sur papier** sur la suite jusqu'à 64 bits, en
t'appuyant sur les techniques du matériel réel évoquées ci-dessus.

## 🛠️ Dans Digital Logic Sim

Réutilise tes puces 8 bits comme blocs pour construire des versions 16 bits :
deux `ADD_8BITS` chaînés (la retenue sortante du premier devenant la retenue
entrante du second) pour un additionneur 16 bits, deux `REGISTRE_8BITS` pour
un registre 16 bits, etc.

## ✏️ Exercices

**Exercice 10.1 — Additionneur 16 bits par composition.** Construis
`ADD_16BITS` en réutilisant deux instances de ton `ADD_8BITS` (chapitre 2),
sans redescendre au niveau de la porte. Vérifie une addition qui provoque un
dépassement de capacité sur le bloc de poids faible, et confirme que la
retenue se propage correctement vers le bloc de poids fort.

**Exercice 10.2 — Registre et bus 16 bits.** Généralise ton `REGISTRE_8BITS`
(chapitre 4) et ton mécanisme de bus (chapitre 5) à 16 bits, en réutilisant
deux registres 8 bits comme moitié basse / moitié haute.

**Exercice 10.3 (réflexion écrite, sans câblage) — RAM 16 bits d'adresse.**
Esquisse sur papier comment tu procéderais pour adresser 65536 mots à partir
de plusieurs blocs de ta `RAM_16X8` (chapitre 6), en réfléchissant à un
premier niveau de décodage qui choisirait *parmi des blocs entiers*, plutôt
qu'un décodeur unique à 65536 sorties.

**Exercice 10.4 (recherche) — Carry-lookahead.** Explique avec tes propres
mots pourquoi l'additionneur à retenue anticipée, facultatif à 8 bits (voir
le "pour aller plus loin" du chapitre 2), devient quasiment indispensable à
32 ou 64 bits.

## 🤔 Questions de réflexion

- Pourquoi dit-on qu'un CPU "64 bits" désigne surtout la largeur de ses
  registres et de ses bus, plutôt qu'une caractéristique unique et isolée ?
- Un programme compilé pour un CPU 32 bits peut-il tourner sur un CPU 64
  bits ? Et l'inverse ? Qu'est-ce que cela t'apprend sur la compatibilité
  binaire entre architectures ?
- La RAM réellement installée dans un ordinateur (par exemple 16 Go) est
  infiniment plus petite que `2⁶⁴` octets. Comment le système fait-il le lien
  entre une adresse 64 bits manipulée par un programme et la mémoire physique
  réellement disponible ?

## 🚀 Pour aller plus loin

- Si le temps et les performances de ta machine le permettent, tente une
  généralisation à 32 bits de l'ALU complète (chapitre 3), en réutilisant
  deux blocs 16 bits.
- Compare les fiches techniques (largeur de bus, nombre de transistors) d'un
  CPU historique 8 bits (Intel 8080, Zilog Z80) et d'un CPU 64 bits moderne,
  pour visualiser concrètement l'ampleur du saut technologique.

---
layout: default
title: "5. Interruptions CPU"
parent: "Cours 2 : Noyau Linux"
nav_order: 5
---

# 5. Interruptions et exceptions CPU
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le mécanisme des exceptions et interruptions CPU sur x86_64.
- Mettre en place une table de descripteurs d'interruption (IDT).
- Gérer une exception sans crash (`breakpoint`), puis une exception grave
  (`double fault`) avec une pile dédiée.

## 📚 Théorie

### Exceptions et interruptions : de quoi s'agit-il ?

Une **exception** est déclenchée par le processeur lui-même, en réaction à
une situation anormale ou spéciale pendant l'exécution d'une instruction :
division par zéro, accès à une page mémoire non présente (*page fault*),
instruction invalide, ou encore un point d'arrêt de débogage volontaire
(*breakpoint*, via l'instruction `int3`). Une **interruption matérielle**
(chapitre 6), à la différence, est déclenchée par un périphérique externe
(clavier, minuterie) de façon asynchrone par rapport au flux d'instructions.

Dans les deux cas, le processeur interrompt son exécution normale, sauvegarde
son état, et saute vers une fonction de traitement (*handler*) — à condition
qu'on lui ait indiqué où la trouver.

### La table de descripteurs d'interruption (IDT)

L'**IDT** (*Interrupt Descriptor Table*) est une table, en mémoire, associant
chaque type d'exception/interruption (identifié par un numéro, de 0 à 255) à
l'adresse de la fonction chargée de la traiter. Le processeur possède un
registre spécial (`IDTR`) pointant vers cette table ; c'est au noyau de la
construire et de charger son adresse dans ce registre (instruction `lidt`).

### Une convention d'appel spéciale

Un gestionnaire d'exception n'est pas une fonction ordinaire : le processeur
lui transmet des informations spécifiques sur la pile (adresse de retour,
indicateurs, parfois un code d'erreur) selon un format particulier, différent
d'un appel de fonction classique. Rust propose, à l'état encore instable, une
convention d'appel dédiée (`extern "x86-interrupt"`) qui prend en charge
automatiquement cette mécanique, à condition de déclarer tes fonctions de
traitement avec cette signature spéciale et le bon type de paramètre
(représentant la pile au moment de l'interruption).

### Le problème du *double fault*, et pourquoi il peut devenir un *triple
fault*

Si une exception survient alors qu'aucun gestionnaire adapté n'est
enregistré (ou que le gestionnaire lui-même provoque une nouvelle exception),
le processeur déclenche une exception spéciale : le **double fault**. Si,
à son tour, le gestionnaire de double fault échoue (par exemple parce que la
pile est corrompue ou insuffisante — un débordement de pile, *stack
overflow*, est une cause fréquente), le processeur ne peut plus rien faire de
raisonnable : il déclenche un **triple fault**, qui provoque un
redémarrage immédiat de la machine. C'est pour cette raison qu'il est
essentiel que le gestionnaire de double fault dispose de sa **propre pile
dédiée**, distincte de la pile normale du noyau (potentiellement déjà
corrompue au moment où le double fault survient).

### La GDT, la TSS et la pile d'interruption dédiée (IST)

Sur x86_64, fournir une pile dédiée à un gestionnaire d'exception passe par
une **TSS** (*Task State Segment*), elle-même référencée par la **GDT**
(*Global Descriptor Table*, une structure héritée du mode segmenté des
processeurs x86 plus anciens). La TSS contient une table de piles alternatives
(*Interrupt Stack Table*, IST) : on peut indiquer, dans l'entrée IDT du double
fault, d'utiliser l'une de ces piles alternatives plutôt que la pile courante.

### Comment Linux fait

Linux définit sa propre IDT très tôt au démarrage (`arch/x86/kernel/idt.c`),
avec des gestionnaires pour chaque exception processeur (division par zéro,
*page fault*, *general protection fault*...), et utilise également des piles
dédiées via l'IST pour les situations les plus critiques (dont le *double
fault* et le *non-maskable interrupt*), exactement pour la raison décrite
ci-dessus.

## 🛠️ En pratique (Rust / QEMU)

Une bibliothèque Rust de bas niveau pour x86_64 (voir
[Ressources](../ressources/liens-utiles-os.html)) fournit des types prêts à
l'emploi pour représenter une IDT, une GDT et une TSS, sans que tu aies à
manipuler toi-même le format binaire exact de ces structures — mais c'est à
toi de les configurer et de les charger correctement.

## ✏️ Exercices

**Exercice 5.1 — Gestionnaire de breakpoint.** Construis une IDT, enregistre
un gestionnaire pour l'exception `breakpoint` qui affiche un message (via ton
`println!` du chapitre 3) puis rend la main normalement. Charge cette IDT.
Déclenche volontairement un breakpoint (instruction `int3`, ou l'équivalent
fourni par ta bibliothèque bas niveau) et vérifie que ton noyau continue de
s'exécuter après le message.

**Exercice 5.2 — Double fault sans pile dédiée.** Avant de mettre en place la
pile dédiée, provoque volontairement un débordement de pile (par exemple une
fonction récursive sans condition d'arrêt). Observe ce qui se passe dans
QEMU (probablement un redémarrage en boucle — c'est le triple fault décrit en
théorie).

**Exercice 5.3 — GDT, TSS et pile dédiée.** Construis une GDT et une TSS
avec au moins une entrée dans l'IST, configure l'entrée `double fault` de ton
IDT pour utiliser cette pile dédiée, et recharge les registres de segments et
de tâche nécessaires (`lgdt`, chargement du sélecteur de code, `ltr`).
Reproduis le débordement de pile de l'exercice 5.2 : ton gestionnaire de
double fault doit maintenant s'exécuter proprement (afficher un message,
par exemple), sans provoquer de triple fault.

## 🤔 Questions de réflexion

- Pourquoi un gestionnaire d'exception ne peut-il pas être une fonction Rust
  "normale", et qu'apporte concrètement la convention d'appel
  `extern "x86-interrupt"` ?
- Pourquoi le *double fault* a-t-il besoin d'une pile séparée alors que la
  plupart des autres exceptions n'en ont pas besoin ?
- Dans quels cas réels (en dehors d'un bug de programmation) un système
  d'exploitation pourrait-il rencontrer un *page fault* de façon tout à fait
  normale, sans que ce soit une erreur ? (Indice : tu y reviendras au
  chapitre 7.)

## 🚀 Pour aller plus loin

Ajoute un gestionnaire pour l'exception *general protection fault*, qui
affiche le code d'erreur associé de façon lisible, en t'aidant de la
documentation du manuel Intel/AMD (ou de ta bibliothèque bas niveau) sur le
format de ce code d'erreur.

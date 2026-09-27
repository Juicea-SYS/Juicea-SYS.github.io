---
layout: default
title: Accueil
nav_order: 1
description: "Deux cours et deux projets fil rouge : un CPU logique dans Digital Logic Sim, puis un OS en Rust"
permalink: /
---

# Architecture des ordinateurs — de la porte logique au noyau
{: .fs-9 }

Deux parcours progressifs et deux projets fil rouge pour comprendre, *de
l'intérieur*, comment un ordinateur fonctionne — des portes logiques jusqu'à
un CPU 8 bits, puis d'un CPU jusqu'à un noyau de système d'exploitation
minimal, en Rust.
{: .fs-6 .fw-300 }

[Parcours 1 : CPU logique](cours/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Parcours 2 : Noyau Linux](noyau-linux/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

## Philosophie de ce site

⚠️ **Il n'y a pas de corrigé.** Ce site n'est pas un manuel qui donne les réponses :
c'est un **entraîneur**. Chaque chapitre t'apporte la théorie nécessaire et te fixe un
objectif précis (souvent sous forme de table de vérité, de spécification ou de
signature de fonction), mais **c'est à toi de concevoir la solution** — dans
Digital Logic Sim pour le premier parcours, en Rust pour le second — de la
tester et de la déboguer. C'est en cherchant, en te trompant et en relisant
une spécification que l'architecture des ordinateurs devient intuitive.

## Les deux parcours

### Parcours 1 — De la porte logique au CPU

Construis un CPU 8 bits inspiré du SAP-1, entièrement dans **Digital Logic
Sim**, en partant d'une simple porte NAND. Treize chapitres, jusqu'au 64
bits, à la comparaison d'architectures et à une introduction au GPU.

[Sujet du projet 1](sujet-projet.html){: .btn .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Commencer le cours 1](cours/){: .btn .fs-5 .mb-4 .mb-md-0 }

### Parcours 2 — Architecture du noyau Linux, un OS en Rust

Construis un noyau x86_64 minimal en **Rust** (`no_std`) : bootloader,
interruptions, mémoire virtuelle, allocation dynamique, multitâche coopératif
et pilotes de périphériques — avec, à chaque étape, une comparaison à la
façon dont **Linux** résout le même problème à grande échelle.

[Sujet du projet 2](sujet-projet-os.html){: .btn .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Commencer le cours 2](noyau-linux/){: .btn .fs-5 .mb-4 .mb-md-0 }

## Prérequis

**Parcours 1** : compter en binaire, convertir binaire ↔ décimal ↔
hexadécimal ; aucune expérience préalable en électronique numérique n'est
nécessaire ; [Digital Logic Sim](https://github.com/SebLague/Digital-Logic-Sim/releases)
installé (voir [Ressources](ressources/)).

**Parcours 2** : les bases du langage Rust (ownership, traits, types
génériques) ; le parcours 1 n'est pas un prérequis strict, mais une bonne
intuition du matériel (interruptions, mémoire, bus) y aide beaucoup — les
chapitres du parcours 2 font le lien à chaque fois que c'est utile ; chaîne
Rust `nightly` et QEMU installés (voir
[Ressources](ressources/liens-utiles-os.html)).

## Comment progresser

1. Lis le sujet du projet du parcours choisi une première fois en entier,
   pour avoir la cible finale en tête.
2. Avance chapitre par chapitre dans le cours correspondant, dans l'ordre.
3. À la fin de chaque chapitre, fais les exercices **avant** de passer au
   suivant : chaque brique sert dans les chapitres suivants.
4. Sauvegarde ton travail au fil de l'eau (puces Digital Logic Sim, ou
   commits Git pour le noyau Rust) — tu le réutiliseras tel quel dans les
   chapitres d'assemblage.
5. Tiens un petit carnet de bord (même juste un fichier texte) : ce que tu as
   essayé, ce qui a raté, pourquoi. C'est souvent plus formateur que le
   résultat final.

## Plan du parcours 1 — De la porte logique au CPU

| # | Chapitre | Ce que tu construis |
|---|----------|----------------------|
| 0 | [Prise en main de Digital Logic Sim](cours/00-prise-en-main.html) | Ton environnement de travail |
| 1 | [Algèbre de Boole et portes logiques](cours/01-algebre-bool.html) | Portes de base à partir de NAND |
| 2 | [Circuits combinatoires](cours/02-circuits-combinatoires.html) | Additionneur, décodeur, multiplexeur |
| 3 | [Arithmétique binaire et ALU](cours/03-arithmetique-alu.html) | Une ALU 8 bits (ADD / SUB) avec drapeaux |
| 4 | [Bascules et mémoire](cours/04-bascules-memoire.html) | Bascule D, registre 8 bits |
| 5 | [Registres, bus et multiplexage](cours/05-registres-bus.html) | Bus de données partagé 8 bits |
| 6 | [Architecture Von Neumann](cours/06-architecture-von-neumann.html) | RAM adressable, compteur ordinal |
| 7 | [Jeu d'instructions (ISA)](cours/07-jeu-instructions.html) | Encodage de programmes |
| 8 | [Unité de contrôle](cours/08-unite-controle.html) | Séquenceur et signaux de commande |
| 9 | [Assemblage final du CPU](cours/09-assemblage-final.html) | Le CPU complet, exécutant un programme |
| 10 | [Passer à 64 bits](cours/10-passage-64-bits.html) | Généraliser tes circuits par composition |
| 11 | [Comparer les architectures CPU](cours/11-architectures-cpu.html) | CISC vs RISC, pipeline, multicœur |
| 12 | [Introduction au GPU](cours/12-introduction-gpu.html) | Une mini-unité SIMD à 2/4 voies |

Les chapitres 10 à 12 sont des extensions : le CPU minimal du
[sujet du projet 1](sujet-projet.html) reste 8 bits et se valide dès le
chapitre 9.

## Plan du parcours 2 — Architecture du noyau Linux

| # | Chapitre | Ce que tu construis |
|---|----------|----------------------|
| 0 | [Panorama de l'architecture Linux](noyau-linux/00-panorama-architecture-linux.html) | Une vue d'ensemble, sur papier |
| 1 | [Environnement et outils](noyau-linux/01-environnement.html) | Un premier binaire Rust `no_std` |
| 2 | [Le démarrage (boot)](noyau-linux/02-demarrage.html) | Une image bootable qui démarre dans QEMU |
| 3 | [Noyau minimal & VGA](noyau-linux/03-noyau-minimal-vga.html) | `println!` maison, affichage à l'écran |
| 4 | [Tests bare-metal](noyau-linux/04-tests-bare-metal.html) | Des tests automatisés dans QEMU |
| 5 | [Interruptions CPU](noyau-linux/05-interruptions-cpu.html) | IDT, gestion du breakpoint et du double fault |
| 6 | [Interruptions matérielles](noyau-linux/06-interruptions-materielles.html) | PIC remappé, timer, premier pilote clavier |
| 7 | [Mémoire virtuelle](noyau-linux/07-memoire-virtuelle.html) | Pagination, traduction d'adresses |
| 8 | [Allocation dynamique](noyau-linux/08-allocation-dynamique.html) | Un allocateur de heap, `Box`/`Vec` fonctionnels |
| 9 | [Multitâche coopératif](noyau-linux/09-multitache-cooperatif.html) | Un exécuteur async, tâches concurrentes |
| 10 | [Architecture des pilotes](noyau-linux/10-architecture-pilotes.html) | Un second pilote de périphérique |
| 11 | [Système de fichiers et syscalls](noyau-linux/11-systeme-fichiers-syscalls.html) *(extension)* | Ramfs minimal, esquisse d'espace utilisateur |
| 12 | [Assemblage final](noyau-linux/12-assemblage-final.html) | L'OS complet, démontré de bout en bout |

Bon courage, et surtout : prends ton temps sur les chapitres 4 et 8 du
parcours 1, et sur les chapitres 5 et 7 du parcours 2 — historiquement les
plus déroutants des deux cours.

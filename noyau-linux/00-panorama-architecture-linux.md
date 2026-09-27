---
layout: default
title: "0. Panorama de l'architecture Linux"
parent: "Cours 2 : Noyau Linux"
nav_order: 0
---

# 0. Panorama de l'architecture du noyau Linux
{: .no_toc }

1. TOC
{:toc}

⚠️ Chapitre entièrement théorique — pas de code dans ce chapitre. L'objectif
est de savoir de quoi on parle avant d'en construire une version minimale.

## 🎯 Objectifs

- Comprendre le rôle d'un système d'exploitation et la séparation espace
  noyau / espace utilisateur.
- Comprendre pourquoi Linux est qualifié de noyau **monolithique modulaire**
  (et ce que ça signifie par opposition à un micro-noyau).
- Identifier les grands sous-systèmes de Linux, pour pouvoir ensuite situer
  chaque chapitre pratique de ce cours par rapport à eux.

## 📚 Théorie

### Le rôle d'un système d'exploitation

Un système d'exploitation a trois responsabilités fondamentales :

1. **Abstraire le matériel** : offrir aux programmes une interface uniforme
   (fichiers, sockets réseau, mémoire) sans qu'ils aient à connaître les
   détails du disque, de la carte réseau ou de la puce mémoire installés.
2. **Partager les ressources** entre plusieurs programmes : temps CPU
   (ordonnancement), mémoire (isolation), périphériques.
3. **Protéger** les programmes les uns des autres, et le système lui-même
   des erreurs d'un programme.

### Espace noyau et espace utilisateur

Le CPU x86_64 offre plusieurs **niveaux de privilège** (*rings*), de 0 (le
plus privilégié) à 3 (le moins privilégié). Linux (comme la quasi-totalité
des OS généralistes) utilise :

- **Ring 0 — espace noyau** : le code du noyau s'exécute ici, avec un accès
  complet au matériel (instructions privilégiées, accès direct à la mémoire
  physique, aux ports d'entrée/sortie...).
- **Ring 3 — espace utilisateur** : les programmes classiques (ton
  navigateur, ton terminal) s'exécutent ici, avec des droits restreints. Un
  programme en ring 3 ne peut interagir avec le matériel ou la mémoire
  d'autres programmes qu'en passant par le noyau, via un **appel système**
  (*syscall*).

Cette séparation est la base de la sécurité et de la stabilité d'un système
multi-programmes : une erreur dans une application ne doit pas pouvoir
planter tout le système, ni lire la mémoire d'un autre programme.

### Monolithique, micro-noyau, et le choix de Linux

- Un **micro-noyau** (Mach, seL4) ne place en espace noyau que le strict
  minimum (gestion de la mémoire, ordonnancement, communication
  inter-processus) ; les pilotes de périphériques et systèmes de fichiers
  tournent en espace utilisateur, comme des programmes presque ordinaires.
- Un **noyau monolithique** place l'essentiel des services (pilotes, systèmes
  de fichiers, pile réseau...) directement en espace noyau, dans un même
  espace d'adressage, pour des raisons de performance (moins de changements
  de contexte, moins de communication inter-processus).

Linux est un noyau **monolithique modulaire** : la majorité du code
s'exécute en espace noyau (ring 0), mais il peut être étendu dynamiquement
par des **modules chargeables** (*Loadable Kernel Modules*, LKM) — du code
noyau compilé séparément, chargé et déchargé à chaud (commandes `lsmod`,
`modprobe`), sans redémarrer le système. C'est ce qui permet, par exemple,
d'ajouter le pilote d'une nouvelle carte graphique sans recompiler tout le
noyau.

### Les grands sous-systèmes de Linux

| Sous-système | Rôle | Répertoire (schématique) dans les sources Linux |
|---|---|---|
| Gestion des processus / ordonnanceur | Créer, planifier, arrêter les processus et threads | `kernel/sched/` |
| Gestion de la mémoire | Pagination, mémoire virtuelle, allocation (`slab`, `buddy allocator`) | `mm/` |
| VFS (*Virtual File System*) | Abstraction commune à tous les systèmes de fichiers (ext4, btrfs, tmpfs...) | `fs/` |
| Pilotes de périphériques | Code spécifique à chaque matériel, exposé via une interface commune | `drivers/` |
| Pile réseau | Implémentation de TCP/IP et autres protocoles | `net/` |
| Interface d'appels système | Point d'entrée entre espace utilisateur et noyau | `arch/*/kernel/syscall*`, `include/linux/syscalls.h` |
| Gestion des interruptions / architecture | Code spécifique au processeur (IDT, GDT, pagination bas niveau) | `arch/x86/` (pour x86_64) |

### Ce que ce cours va (et ne va pas) reconstruire

Ce cours va te faire reconstruire, en miniature, une partie de chacun de ces
sous-systèmes, sauf la pile réseau (hors périmètre de ce projet) et le VFS
complet (abordé seulement en extension, chapitre 11). Chaque chapitre pratique
commencera par une section "comment Linux fait", pour que tu gardes ce fil
conducteur tout du long.

## ✏️ Exercices

**Exercice 0.1 — Explorer un système Linux vivant.** Si tu as accès à une
machine Linux (ou une VM), exécute et interprète le résultat de ces commandes :
`uname -a`, `lsmod`, `cat /proc/interrupts`, `cat /proc/meminfo`, `ls /dev`.
Pour chacune, note à quel sous-système du tableau ci-dessus elle se rapporte.

**Exercice 0.2 — Cartographier le projet.** Établis un tableau à deux
colonnes : "sous-système Linux" / "ce que je vais construire dans mon
noyau, et à quel chapitre". Tu peux t'aider du
[sujet du projet 2](../sujet-projet-os.html) pour la deuxième colonne.
Garde ce tableau sous la main : tu le complèteras au fil du cours.

## 🤔 Questions de réflexion

- Pourquoi le choix "monolithique" de Linux est-il souvent présenté comme un
  compromis performance / sécurité, par rapport à un micro-noyau ?
- Un module chargeable (LKM) s'exécute-t-il en ring 0 ou en ring 3 ? Quelle
  conséquence cela a-t-il si le module contient un bug ?
- En quoi le noyau que tu vas construire dans ce cours est-il, structurellement,
  plus proche d'un micro-noyau ultra-minimal que d'un Linux complet ? (Indice :
  pense à la présence — ou l'absence — d'un espace utilisateur séparé dans le
  socle obligatoire du projet.)

## 🚀 Pour aller plus loin

Parcours (sans nécessairement tout lire) la table des matières du code source
du noyau Linux sur [kernel.org](https://www.kernel.org) ou son miroir GitHub,
pour te donner une idée de l'échelle réelle du projet par rapport à ce que tu
vas construire.

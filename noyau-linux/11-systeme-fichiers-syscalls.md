---
layout: default
title: "11. Système de fichiers et syscalls"
parent: "Cours 2 : Noyau Linux"
nav_order: 11
---

# 11. Système de fichiers et appels système *(extension)*
{: .no_toc }

1. TOC
{:toc}

⚠️ Ce chapitre est une **extension avancée**, non nécessaire pour valider le
socle obligatoire du [sujet du projet 2](../sujet-projet-os.html). Il est
nettement plus exigeant que les précédents : la mise en place d'un véritable
espace utilisateur (ring 3) est un morceau conséquent d'un vrai système
d'exploitation.

## 🎯 Objectifs

- Comprendre l'abstraction VFS (*Virtual File System*) de Linux.
- Comprendre le mécanisme des appels système et le changement de niveau de
  privilège (ring 0 ↔ ring 3).
- Esquisser une version minimale de ces deux mécanismes dans ton noyau.

## 📚 Théorie

### Le VFS : une abstraction au-dessus des systèmes de fichiers

Linux sait lire et écrire des dizaines de systèmes de fichiers différents
(ext4, btrfs, FAT, NFS...) à travers une **interface commune**, le VFS. Les
programmes appellent toujours les mêmes fonctions (`open`, `read`, `write`,
`close`), quel que soit le système de fichiers réellement utilisé derrière un
chemin donné — le VFS fait le lien entre l'appel générique et
l'implémentation spécifique du système de fichiers concerné, un peu comme le
trait commun de pilotes du chapitre 10 fait le lien entre une interface
générique et l'implémentation d'un périphérique particulier.

### Un système de fichiers minimal : le principe d'un *ramfs*

Un système de fichiers "en mémoire" (*ramfs*, ou son évolution *tmpfs* dans
Linux) ne persiste rien sur un disque : il représente simplement une
arborescence de fichiers et de répertoires comme des structures de données
en mémoire (par exemple, une table associant un chemin à un contenu de type
`Vec<u8>`). C'est le système de fichiers le plus simple à implémenter, et un
point de départ raisonnable si tu veux explorer cette extension.

### Les appels système : la frontière entre les deux mondes

Un appel système est le mécanisme par lequel un programme en espace
utilisateur (ring 3) demande un service au noyau (ring 0) — lire un fichier,
écrire à l'écran, créer un processus... Sur x86_64, ce passage se fait via
une instruction dédiée (`syscall`), qui déclenche un changement de niveau de
privilège contrôlé : le processeur saute vers une adresse de gestionnaire
préalablement enregistrée par le noyau (via des registres spéciaux du
processeur), avec des arguments transmis par convention dans certains
registres.

### Pourquoi c'est une marche nettement plus haute

Mettre en place un appel système suppose d'avoir, au préalable, un vrai
**espace utilisateur** : un code exécutable séparé du noyau, chargé dans un
espace d'adressage isolé (donc avec ses propres tables de pages, chapitre 7),
exécuté avec un niveau de privilège réduit (ring 3), avec une pile séparée,
et un mécanisme fiable de retour vers le noyau lors d'un appel système ou
d'une interruption. Chacun de ces points reprend et étend des notions déjà
vues (chapitres 5, 6, 7), mais leur combinaison correcte est significativement
plus délicate qu'un chapitre isolé — d'où son statut d'extension plutôt que
de socle obligatoire.

### Comment Linux fait

La table des appels système de Linux (`include/linux/syscalls.h` et les
tables générées par architecture) recense plusieurs centaines d'appels
système, documentés dans la page de manuel `syscalls(2)`. Chacun est identifié
par un numéro, transmis dans un registre convenu avant l'instruction
`syscall`.

## 🛠️ En pratique (Rust / QEMU)

Cette extension nécessite de configurer des **MSR** (*Model Specific
Registers*) du processeur pour enregistrer ton gestionnaire de `syscall`, et
de préparer un binaire séparé pour l'espace utilisateur (compilé pour une
cible différente de ton noyau, ou en assembleur inline minimal). Documente-toi
sur les registres `STAR`, `LSTAR`, `SFMASK` avant de te lancer.

## ✏️ Exercices (extension, non obligatoires)

**Exercice 11.1 — Ramfs minimal.** Implémente une structure de données en
mémoire représentant un système de fichiers plat (pas de sous-répertoires
pour commencer) : une table associant un nom de fichier à un contenu
`Vec<u8>`, avec des opérations `creer`, `lire`, `ecrire`.

**Exercice 11.2 (recherche, sur papier) — Espace utilisateur minimal.**
Documente, étape par étape, ce qu'il faudrait mettre en place pour exécuter
un tout petit programme en ring 3 depuis ton noyau (tables de pages
séparées, pile utilisateur, transition de niveau de privilège). Tu n'as pas
besoin de l'implémenter pour valider cet exercice — l'objectif est de
vérifier que tu identifies correctement toutes les pièces nécessaires.

**Exercice 11.3 (si tu vas au bout) — Premier appel système.** Configure les
MSR nécessaires, écris un gestionnaire de `syscall` minimal (par exemple, un
unique appel système "afficher une chaîne de caractères", inspiré de
`write`), et déclenche-le depuis un tout petit programme utilisateur.

## 🤔 Questions de réflexion

- En quoi le VFS de Linux et le trait commun de pilotes du chapitre 10
  répondent-ils, à deux niveaux différents, au même problème
  d'architecture logicielle ?
- Pourquoi un appel système doit-il obligatoirement passer par une
  instruction dédiée (`syscall`) plutôt qu'un simple appel de fonction
  ordinaire vers du code noyau ?
- Pourquoi cette extension est-elle présentée comme optionnelle dans ce
  cours, alors qu'elle est justement ce qui distingue le plus nettement un
  "vrai" système d'exploitation multi-programmes d'un simple noyau
  bare-metal mono-programme ?

## 🚀 Pour aller plus loin

Si tu vas au bout de cette extension, ajoute un second appel système (par
exemple, lire une touche du clavier de façon bloquante depuis l'espace
utilisateur), et documente comment il s'articule avec le mécanisme
asynchrone du chapitre 9 côté noyau.

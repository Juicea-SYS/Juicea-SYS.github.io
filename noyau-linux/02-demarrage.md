---
layout: default
title: "2. Le démarrage (boot)"
parent: "Cours 2 : Noyau Linux"
nav_order: 2
---

# 2. Le processus de démarrage
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre les étapes du démarrage d'un PC x86_64, du bouton d'alimentation
  au premier code noyau exécuté.
- Comprendre les modes d'exécution successifs du processeur (réel, protégé,
  long).
- Obtenir une image bootable de ton noyau, démarrant dans QEMU.

## 📚 Théorie

### Les étapes du démarrage

1. **Firmware (BIOS ou UEFI)** : au démarrage, le processeur exécute du code
   stocké sur une puce de la carte mère (le firmware), en **mode réel**
   (16 bits, hérité de l'Intel 8086 pour compatibilité historique). Le
   firmware initialise le matériel de base puis cherche un périphérique
   amorçable.
2. **Bootloader** : le firmware charge en mémoire le premier secteur (BIOS
   legacy) ou l'exécutable EFI désigné (UEFI) du disque de démarrage, et lui
   passe la main. Le bootloader a pour rôle de : configurer une structure
   mémoire minimale, faire passer le processeur du mode réel au **mode
   protégé** (32 bits) puis au **mode long** (*long mode*, 64 bits) via une
   série d'étapes précises (activation de la pagination, configuration d'une
   GDT minimale...), puis charger le noyau en mémoire et lui passer la main.
3. **Noyau** : à partir de ce point, le noyau s'exécute en mode long 64 bits,
   avec un contrôle total du matériel.

### Comment Linux fait

Sur un PC réel, Linux est démarré par un bootloader dédié comme **GRUB**
(implémentant le standard **Multiboot2**) ou directement par **UEFI**. Le
bootloader décompresse une image noyau compressée, configure quelques
structures minimales, puis saute dans le point d'entrée du noyau Linux, qui
prend alors le relais pour terminer sa propre initialisation (configuration
complète de la mémoire, des interruptions, montage du système de fichiers
racine...).

### Ton bootloader

Écrire un bootloader complet à la main (gestion BIOS/UEFI, transition entre
les trois modes, chargement de fichier depuis un système de fichiers minimal)
est un projet en soi, hors du périmètre de ce cours. Ce cours te recommande
d'utiliser une **crate Rust existante** spécialisée dans cette tâche : elle
prend ton noyau compilé (un exécutable ELF ordinaire) et génère une image de
disque bootable, en gérant elle-même la mécanique BIOS/UEFI et la transition
de modes. Voir [Ressources](../ressources/liens-utiles-os.html) pour le nom
et la documentation de la crate recommandée.

Ce choix te permet de te concentrer sur ce qui est réellement formateur pour
ce cours : le **noyau** lui-même, pas la mécanique d'amorçage bas niveau du
BIOS — qui reste néanmoins une excellente piste pour le "pour aller plus
loin" ci-dessous.

### `BootInfo` : ce que le bootloader te transmet

Un bootloader moderne ne se contente pas de sauter dans ton code : il te
transmet aussi des informations utiles rassemblées pendant le démarrage —
typiquement la carte mémoire physique disponible (quelles zones sont libres,
lesquelles sont réservées), et éventuellement un décalage (*offset*) auquel
toute la mémoire physique a été mappée en mémoire virtuelle. Tu réutiliseras
ces informations dès le chapitre 7 (mémoire virtuelle).

## 🛠️ En pratique (Rust / QEMU)

- Ajoute la crate de bootloader recommandée à ton projet, en suivant sa
  documentation pour la configuration du point d'entrée (souvent une macro
  du type `entry_point!(kernel_main)` plutôt qu'un `_start` manuel comme au
  chapitre 1).
- Génère une image disque bootable à partir de ton exécutable noyau.
- Lance cette image dans `qemu-system-x86_64`.

## ✏️ Exercices

**Exercice 2.1 — Premier démarrage.** En reprenant ton projet du chapitre 1,
intègre le bootloader recommandé, compile pour la cible bare-metal (le
fichier de cible JSON du chapitre 1), génère une image bootable, et
lance-la dans QEMU. Objectif : que QEMU démarre sans erreur fatale et
reste dans un état stable (même sans rien afficher encore à l'écran — ce sera
le chapitre 3).

**Exercice 2.2 — Observer les modes du processeur.** Ajoute l'option de log
des changements de mode CPU à ta commande QEMU
(`-d cpu_reset,int -no-reboot -no-shutdown` ou équivalent selon ta version),
et observe dans les logs la transition entre mode réel, protégé, puis long,
au démarrage de ta machine virtuelle.

## 🤔 Questions de réflexion

- Pourquoi le mode réel (16 bits) est-il encore le point de départ obligé sur
  un PC x86_64 moderne, alors que le processeur est en réalité capable de 64
  bits depuis longtemps ?
- Que contient, à ton avis, la structure `BootInfo` transmise par le
  bootloader, et pourquoi est-ce le bootloader (plutôt que le noyau
  lui-même) qui la construit ?
- En quoi choisir une crate de bootloader existante, plutôt que d'écrire le
  tien en assembleur, change-t-il ce que tu apprends dans ce chapitre — et
  penses-tu que ce compromis est justifié pour ce cours ?

## 🚀 Pour aller plus loin

Écris, en dehors de ce projet Rust, un secteur de démarrage BIOS minimal en
assembleur x86 (512 octets, terminé par la signature `0x55AA`), qui se
contente d'afficher un caractère à l'écran via une interruption BIOS
(`int 0x10`). C'est l'exercice classique du wiki OSDev pour comprendre "à la
main" ce que fait la crate de bootloader à ta place.

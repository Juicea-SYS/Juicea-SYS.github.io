---
layout: default
title: "Liens utiles (OS Rust)"
parent: Ressources
nav_order: 3
---

# Liens utiles — Projet 2 (OS en Rust)

## Rust bare-metal

- **rustup** (gestion des toolchains Rust) : [rustup.rs](https://rustup.rs)
- Chaîne **nightly** requise : `rustup toolchain install nightly`,
  `rustup override set nightly` dans le dossier du projet.

## QEMU

- Site officiel : [qemu.org](https://www.qemu.org/download/) — installe le
  paquet correspondant à ton système (souvent `qemu` ou `qemu-system-x86`
  selon le gestionnaire de paquets).

## Crates Rust utiles (écosystème `rust-osdev`)

Ces bibliothèques, maintenues par l'organisation **rust-osdev** (dont fait
partie l'auteur de la série *Writing an OS in Rust*), t'évitent de
réinventer les couches les plus bas niveau, pour te concentrer sur
l'architecture de ton noyau :

| Crate | Rôle |
|---|---|
| `bootloader` | Génère une image disque bootable (BIOS) à partir de ton noyau compilé, gère la transition réel → protégé → long, transmet un `BootInfo` |
| `bootimage` (outil `cargo`) | Automatise l'appel à `bootloader` et le lancement dans QEMU |
| `x86_64` | Types bas niveau x86_64 : IDT, GDT, TSS, tables de pages, ports d'E/S, registres |
| `pic8259` (anciennement `pic8259_simple`) | Pilotage du contrôleur d'interruptions PIC 8259, avec remappage |
| `pc-keyboard` | Décodage des scancodes clavier (jeux de scancodes 1 et 2) |
| `uart_16550` | Pilotage du port série pour la sortie de test (chapitre 4) |
| `spin` | Verrous (`Mutex`) utilisables sans système d'exploitation sous-jacent |
| `volatile` | Empêche le compilateur d'optimiser les écritures mémoire (utile pour le tampon VGA) |
| `linked_list_allocator` | Allocateur de heap prêt à l'emploi (chapitre 8) |
| `crossbeam-queue` | File sans verrou bloquant (`ArrayQueue`), utile entre interruption et tâche asynchrone (chapitre 9) |
| `futures-util` | Utilitaires pour `Future`, dont `AtomicWaker` |
| `conquer-once` | Initialisation différée compatible `no_std` (`OnceCell`) |

Cherche chaque nom de crate sur [crates.io](https://crates.io) pour sa
documentation et sa version la plus récente au moment où tu lis ce cours —
les versions évoluent, les principes présentés dans ce cours restent valables.

## Ressource de référence externe

- La série *Writing an OS in Rust* (recherche `os.phil-opp.com`) de Philipp
  Oppermann est la ressource externe la plus proche de l'esprit de ce cours 2
  : elle couvre, dans un ordre très similaire, freestanding binary, noyau
  minimal, VGA, tests, exceptions CPU, double fault, interruptions
  matérielles, pagination, heap, async/await.
- Le **wiki OSDev** (recherche `wiki.osdev.org`) est la référence générale de
  la communauté de développement de systèmes d'exploitation amateurs, toutes
  architectures et tous langages confondus.
- La documentation du noyau Linux lui-même (recherche `kernel.org` /
  `docs.kernel.org`) pour les comparaisons "comment Linux fait" de chaque
  chapitre.

⚠️ Comme pour le premier cours : ces ressources externes, notamment la série
de Philipp Oppermann, montrent des solutions de code complètes. Garde-les
pour **après** avoir cherché par toi-même, en guise de vérification, si tu
veux préserver l'effet d'entraînement de ce cours.

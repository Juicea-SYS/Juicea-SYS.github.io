---
layout: default
title: "Glossaire (Noyau / Rust)"
parent: Ressources
nav_order: 4
---

# Glossaire — Noyau et Rust bare-metal

| Terme | Définition rapide |
|---|---|
| **`no_std`** | Attribut Rust indiquant de ne pas lier la bibliothèque standard `std`. |
| **`alloc`** | Bibliothèque Rust intermédiaire (entre `core` et `std`) fournissant `Box`, `Vec`, `String`, sans dépendre d'un OS. |
| **APIC** | *Advanced Programmable Interrupt Controller* — successeur du PIC 8259, notamment pour le multicœur. |
| **Bootloader** | Programme chargé par le firmware, chargé de préparer le processeur et de lancer le noyau. |
| **`BootInfo`** | Structure transmise par le bootloader au noyau (carte mémoire, décalage de mémoire physique...). |
| **Double fault** | Exception CPU déclenchée quand la gestion d'une première exception échoue. |
| **Espace noyau / espace utilisateur** | Séparation des niveaux de privilège (ring 0 / ring 3) du processeur. |
| **`Future`** | Trait Rust représentant un calcul asynchrone, avancé via `poll`. |
| **GDT** | *Global Descriptor Table* — structure héritée du mode segmenté x86, utilisée aujourd'hui notamment pour référencer la TSS. |
| **Heap (tas)** | Zone mémoire dédiée à l'allocation dynamique. |
| **IDT** | *Interrupt Descriptor Table* — table associant chaque exception/interruption à son gestionnaire. |
| **IST** | *Interrupt Stack Table* — piles alternatives utilisables par certains gestionnaires d'exception critiques. |
| **Mode réel / protégé / long** | Les trois modes d'exécution successifs du processeur x86_64 au démarrage. |
| **Page fault** | Exception CPU déclenchée lors d'un accès à une page mémoire non mappée ou non autorisée. |
| **Pagination** | Mécanisme de traduction d'adresses virtuelles en adresses physiques par tables de pages. |
| **PIC** | *Programmable Interrupt Controller* — contrôleur historique reliant les périphériques au processeur (modèle 8259). |
| **`ring`** (niveau de privilège) | Niveau de droits du processeur (0 = noyau, 3 = utilisateur, sur x86). |
| **Scancode** | Code brut envoyé par un clavier PS/2 à chaque appui/relâchement de touche. |
| **`syscall`** (appel système) | Mécanisme par lequel un programme en espace utilisateur demande un service au noyau. |
| **Triple fault** | Échec du gestionnaire de double fault, provoquant un redémarrage matériel. |
| **TSS** | *Task State Segment* — structure x86 contenant, entre autres, la table des piles alternatives (IST). |
| **VFS** | *Virtual File System* — abstraction commune de Linux au-dessus de tous les systèmes de fichiers. |
| **`Waker`** | Mécanisme permettant à une tâche asynchrone de signaler qu'elle est prête à progresser. |

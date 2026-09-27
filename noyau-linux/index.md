---
layout: default
title: "Cours 2 : Noyau Linux"
nav_order: 5
has_children: true
permalink: /noyau-linux/
---

# Cours 2 — Architecture du noyau Linux (et comment en reconstruire un, en Rust)

Ce deuxième cours change d'échelle : on quitte la porte logique pour
s'intéresser à ce qu'un **système d'exploitation** ajoute au-dessus du
matériel — gestion mémoire virtuelle, interruptions, pilotes, ordonnancement,
appels système.

## Pourquoi Linux comme référence, et pourquoi pas construire "Linux" lui-même

Le noyau Linux réel fait plusieurs dizaines de millions de lignes de code : le
reconstruire à l'identique n'est ni réaliste ni pédagogique. Ce cours utilise
donc **l'architecture de Linux comme grille de lecture** — sa façon
d'organiser mémoire, interruptions, pilotes, appels système — pour t'aider à
concevoir, en Rust, un **noyau minimal qui suit les mêmes grands principes**,
à une échelle où tu peux tout comprendre et tout écrire toi-même. Chaque
chapitre a donc deux volets : *"comment Linux fait"* puis *"ce que tu vas
construire, en version minimale"*.

Même format qu'au premier cours :

- **🎯 Objectifs**
- **📚 Théorie** (souvent en deux temps : Linux, puis ton noyau)
- **🛠️ En pratique (Rust / QEMU)**
- **✏️ Exercices** — la spécification à respecter, jamais le code solution.
- **🤔 Questions de réflexion**
- **🚀 Pour aller plus loin**

Avance dans l'ordre : chaque chapitre s'appuie sur le précédent, comme au
premier cours. Le [sujet du projet 2](../sujet-projet-os.html) te donne la
cible finale à garder en tête dès le départ.

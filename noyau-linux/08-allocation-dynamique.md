---
layout: default
title: "8. Allocation dynamique"
parent: "Cours 2 : Noyau Linux"
nav_order: 8
---

# 8. Allocation dynamique (heap)
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le rôle du trait `GlobalAlloc` en Rust.
- Réserver une zone de mémoire virtuelle dédiée au tas (*heap*) du noyau.
- Implémenter un allocateur, et faire fonctionner `Box`, `Vec`, `String` dans
  ton noyau.

## 📚 Théorie

### `alloc` : le troisième étage de la bibliothèque Rust

Après `core` (chapitre 1), Rust propose un second niveau intermédiaire,
`alloc`, qui contient tous les types nécessitant une allocation dynamique
(`Box`, `Vec`, `String`, `Rc`...), mais sans dépendre d'un système
d'exploitation particulier — à condition de lui fournir toi-même un
allocateur global compatible. C'est l'objectif de ce chapitre.

### Le trait `GlobalAlloc`

Rust délègue toute allocation mémoire à un type implémentant le trait
`GlobalAlloc`, avec deux méthodes principales : `alloc` (réserver un bloc
mémoire d'une taille et d'un alignement donnés) et `dealloc` (le libérer).
Une fois qu'un type implémentant ce trait est enregistré avec l'attribut
`#[global_alloc]`, `Box::new`, `Vec::push`, etc. fonctionnent normalement
dans tout le reste du noyau.

### Réserver une zone de heap

Avant de pouvoir allouer quoi que ce soit, il faut une zone de mémoire
virtuelle dédiée : un intervalle d'adresses choisi arbitrairement (par
exemple une plage ne chevauchant ni ton code, ni ta pile, ni le mappage de
mémoire physique du chapitre 7), effectivement mappé à des cadres physiques
via les outils du chapitre 7.

### Concevoir un allocateur : les compromis

Un allocateur doit répondre à `alloc`/`dealloc` rapidement, avec le moins de
mémoire gâchée possible (fragmentation), le tout dans un contexte où le
verrouillage doit rester compatible avec des interruptions (chapitre 5/6).
Plusieurs designs classiques, du plus simple au plus sophistiqué :

- **Bump allocator** : avance simplement un pointeur à chaque allocation, ne
  sait pas libérer individuellement (seulement tout remettre à zéro d'un
  coup). Trivial à implémenter, mais peu réaliste pour un usage prolongé.
- **Liste chaînée de blocs libres** (*linked list allocator*) : garde une
  liste des blocs libérés, réutilisables pour de futures allocations de
  taille compatible. Plus réaliste, plus complexe, plus lent.
- **Allocateur à blocs de taille fixe** (*fixed-size block allocator*) :
  maintient des listes séparées de blocs libres par taille standardisée
  (8, 16, 32, 64 octets...), afin d'allouer/libérer en temps quasi-constant,
  au prix d'un peu de mémoire perdue par arrondi de taille (fragmentation
  interne).

### Comment Linux fait

L'allocateur mémoire du noyau Linux repose sur plusieurs couches : un
**buddy allocator** au niveau des pages physiques (gérant des blocs de taille
puissance de 2), et par-dessus, l'allocateur **slab** (ou ses évolutions
SLUB/SLOB), spécialisé dans l'allocation rapide et répétée de petits objets
de taille fixe (inspiré, historiquement, de la même famille d'idées que
l'allocateur à blocs de taille fixe ci-dessus).

## 🛠️ En pratique (Rust / QEMU)

Tu peux écrire ton propre allocateur (recommandé pour la compréhension), ou
t'appuyer sur une crate existante pour l'une des stratégies ci-dessus (voir
[Ressources](../ressources/liens-utiles-os.html)) le temps de valider le
reste du chapitre, puis revenir écrire le tien.

## ✏️ Exercices

**Exercice 8.1 — Réserver et mapper le heap.** Choisis une plage d'adresses
virtuelles pour ton heap (par exemple 100 Ko de large), et mappe-la
entièrement à des cadres physiques fraîchement alloués, en réutilisant les
outils du chapitre 7.

**Exercice 8.2 — Bump allocator.** Implémente un allocateur `GlobalAlloc`
de type *bump* : `alloc` avance un pointeur interne (protégé par un verrou),
`dealloc` ne fait rien d'utile individuellement, mais un compteur de blocs
encore "vivants" permet de réinitialiser le pointeur quand ce compteur
retombe à zéro. Enregistre-le avec `#[global_alloc]`. Vérifie que
`Box::new(42)` et un `Vec` qui grossit fonctionnent dans ton noyau.

**Exercice 8.3 — Un allocateur plus réaliste.** Remplace ton *bump
allocator* par une liste chaînée de blocs libres, ou un allocateur à blocs de
taille fixe (au choix). Écris un test (chapitre 4) qui alloue et libère un
grand nombre de blocs de tailles variées, pour vérifier que la mémoire est
correctement réutilisée plutôt que de s'épuiser.

## 🤔 Questions de réflexion

- Pourquoi un simple *bump allocator* est-il un très mauvais choix pour un
  usage prolongé, même s'il est parfaitement correct pour un usage ponctuel ?
- Qu'est-ce que la fragmentation interne (mémoire gâchée par arrondi de
  taille) et la fragmentation externe (mémoire libre mais inutilisable car
  trop morcelée) ? Lequel de tes designs (8.2 vs 8.3) est le plus exposé à
  chacune ?
- Ton allocateur doit-il être conçu pour être utilisable depuis un
  gestionnaire d'interruption (chapitres 5/6) ? Quel risque y aurait-il si un
  gestionnaire d'interruption tentait d'acquérir un verrou déjà tenu par le
  code qu'il vient d'interrompre ?

## 🚀 Pour aller plus loin

Mesure (avec un simple compteur d'instructions ou un chronométrage grossier)
la différence de performance entre ton *bump allocator* et ton allocateur
plus réaliste, sur un scénario d'allocations/libérations répétées. Documente
le compromis observé.

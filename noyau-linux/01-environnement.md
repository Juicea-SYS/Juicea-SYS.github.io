---
layout: default
title: "1. Environnement et outils"
parent: "Cours 2 : Noyau Linux"
nav_order: 1
---

# 1. Environnement et outils
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Installer une chaîne de compilation Rust capable de produire du code
  *freestanding* (sans bibliothèque standard, sans OS hôte).
- Comprendre la différence entre `std` et `core`, et pourquoi un noyau ne
  peut pas utiliser `std`.
- Produire un premier binaire Rust `#![no_std]` qui compile.

## 📚 Théorie

### Pourquoi pas la bibliothèque standard ?

La bibliothèque standard de Rust (`std`) suppose un système d'exploitation
en dessous d'elle : elle utilise des appels système pour l'allocation
mémoire, les fichiers, les threads... Or c'est justement ce système
d'exploitation que nous sommes en train d'écrire — il ne peut donc pas
lui-même reposer sur un OS déjà présent. La solution : utiliser uniquement
`core`, la partie de la bibliothèque standard qui ne dépend d'aucun système
sous-jacent (types de base, traits comme `Copy`/`Clone`, `Option`, `Result`,
opérations arithmétiques...), avec l'attribut `#![no_std]`.

### `no_std` et `no_main`

- `#![no_std]` : indique au compilateur de ne pas lier automatiquement `std`.
- `#![no_main]` : indique qu'on ne veut pas du point d'entrée habituel `fn
  main()` (qui repose sur du code d'initialisation fourni par l'OS), mais que
  l'on va définir nous-mêmes le point d'entrée réel du programme.
- Un **gestionnaire de panique** (*panic handler*) devient obligatoire à
  définir toi-même, puisque celui de `std` (qui affiche un message et arrête
  le processus via l'OS) n'existe plus. Sa signature est imposée par le
  compilateur :

  ```rust
  #[panic_handler]
  fn panic(info: &core::panic::PanicInfo) -> ! {
      loop {}
  }
  ```

  (Le corps de cette fonction — que faire réellement en cas de panique — sera
  à enrichir dans les chapitres suivants, une fois que tu auras un moyen
  d'afficher un message.)

### Rust nightly et fonctionnalités instables

Écrire un noyau nécessite plusieurs fonctionnalités encore instables de
Rust (à ce jour) : convention d'appel spéciale pour les gestionnaires
d'interruption (chapitre 5), certaines options de compilation bas niveau.
Il faut donc utiliser la chaîne **nightly** de Rust plutôt que **stable**.

### Cible de compilation personnalisée

Compiler pour "aucun système d'exploitation" nécessite une cible
(*target*) de compilation spécifique, qui désactive des hypothèses par
défaut incompatibles avec un environnement bare-metal (interruptions
matérielles pouvant survenir pendant l'utilisation de registres flottants,
notamment). C'est un fichier de configuration JSON décrivant l'architecture,
qui indique en particulier `"os": "none"`.

## 🛠️ En pratique (Rust / QEMU)

Installe :
- `rustup` (si ce n'est pas déjà fait), puis une chaîne **nightly** :
  `rustup toolchain install nightly` et `rustup override set nightly` dans
  ton dossier de projet.
- `qemu-system-x86_64` (paquet `qemu` ou `qemu-full` selon ta distribution /
  ton gestionnaire de paquets).

Consulte [Ressources](../ressources/liens-utiles-os.html) pour les liens
d'installation détaillés selon ton système.

## ✏️ Exercices

**Exercice 1.1 — Projet Cargo minimal.** Crée un nouveau projet Cargo. Ajoute
`#![no_std]` en tête de `main.rs`. Essaie de compiler : le compilateur va se
plaindre de plusieurs choses (absence de point d'entrée `main`, absence de
gestionnaire de panique). Résous ces erreurs une par une, en ajoutant
`#![no_main]`, un point d'entrée `#[no_mangle] pub extern "C" fn _start() ->
! { loop {} }`, et un gestionnaire de panique. Fais compiler ce binaire
(encore pour ta machine hôte à ce stade — pas encore pour bare-metal, ce sera
le chapitre 2).

**Exercice 1.2 — Comprendre les erreurs du compilateur.** Note, pour chaque
erreur de compilation rencontrée à l'exercice 1.1, ce qu'elle signifiait
concrètement et comment tu l'as résolue. Ce journal te sera utile pour
comprendre les chapitres suivants.

## 🤔 Questions de réflexion

- Pourquoi `Option` et `Result`, très utilisés en Rust, sont-ils disponibles
  dans `core` et pas seulement dans `std` ? Que cela t'apprend-il sur la
  différence entre les deux bibliothèques ?
- Pourquoi le point d'entrée `_start` doit-il avoir un type de retour `!`
  (le type "jamais", *never type*) plutôt que, par exemple, `()` ?
- Pourquoi ne peut-on pas encore, à ce stade, afficher quoi que ce soit à
  l'écran depuis ce programme (même s'il compile et s'exécute) ?

## 🚀 Pour aller plus loin

Regarde la définition du type `!` (*never type*) dans la documentation de
Rust, et dans quels autres contextes du langage il apparaît (par exemple,
le type de retour d'une boucle infinie, ou d'un appel à `panic!()`).

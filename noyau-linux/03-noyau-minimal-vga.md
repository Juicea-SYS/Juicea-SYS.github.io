---
layout: default
title: "3. Noyau minimal & VGA"
parent: "Cours 2 : Noyau Linux"
nav_order: 3
---

# 3. Un noyau minimal, et écrire à l'écran
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le fonctionnement du tampon texte VGA (*VGA text buffer*).
- Écrire du texte à l'écran depuis ton noyau, sans aucun appel système
  (puisque tu n'en as pas encore).
- Faire fonctionner les macros `print!`/`println!` dans ton propre noyau.

## 📚 Théorie

### Le tampon texte VGA

Sur x86, en mode texte, l'écran peut être piloté simplement : une zone de la
mémoire physique, à l'adresse `0xb8000`, est directement interprétée par le
matériel graphique comme une grille de caractères (typiquement 80 colonnes ×
25 lignes). Chaque caractère affiché occupe 2 octets consécutifs à cette
adresse :

- 1 octet pour le code du caractère (ASCII).
- 1 octet pour son "attribut" : couleur de premier plan (4 bits) et couleur
  de fond (4 bits).

Écrire à l'écran revient donc, très concrètement, à écrire aux bons décalages
mémoire à partir de `0xb8000` — une simple écriture mémoire, aucun appel
système nécessaire, puisque le noyau a un accès direct à toute la mémoire
physique.

### Accès mémoire brut et sécurité en Rust

Écrire directement à une adresse mémoire fixe passe nécessairement par un
pointeur brut (*raw pointer*) et donc par du code `unsafe`. C'est l'un des
rares endroits légitimes où `unsafe` est nécessaire : le compilateur ne peut
pas savoir, par construction, que `0xb8000` est une adresse mémoire valide
sur cette machine — c'est à toi de garantir que cette hypothèse est correcte
(ce qu'elle est, en mode texte VGA standard).

### Comment Linux fait

Un noyau Linux moderne n'utilise quasiment jamais le mode texte VGA en
pratique : il pilote un framebuffer graphique (mode pixel) via un pilote
graphique (`drivers/gpu/`), avec un rendu de police bien plus sophistiqué
(la console texte que tu vois au démarrage est en réalité dessinée pixel par
pixel). Le mode texte VGA que nous utilisons ici est une simplification
délibérée : le principe (écrire dans une zone mémoire pour afficher quelque
chose) reste le même, à une échelle radicalement plus simple.

### Les traits `core::fmt::Write` et les macros `print!`

Rust fournit, dans `core::fmt`, un trait `Write` permettant d'implémenter le
formatage de texte (`write!`, `writeln!`) pour n'importe quel type — y
compris ton propre "écrivain VGA". En implémentant ce trait pour une
structure représentant l'écran, tu peux ensuite définir tes propres macros
`print!`/`println!`, qui se comportent comme celles de la bibliothèque
standard, mais écrivent dans le tampon VGA au lieu de la sortie standard
d'un OS hôte.

## 🛠️ En pratique (Rust / QEMU)

Structure suggérée (à toi d'écrire le détail) :
- Une structure représentant l'écran (position du curseur, couleur courante).
- Une implémentation de `core::fmt::Write` pour cette structure.
- Une instance globale accessible depuis n'importe où dans le noyau — attention,
  une variable globale mutable pose un problème de sécurité des accès
  concurrents en Rust ; regarde du côté d'un type de verrou utilisable en
  contexte `no_std` (souvent `spin::Mutex`, une bibliothèque de verrous actifs
  ne nécessitant pas de système d'exploitation sous-jacent).
- Des macros `print!`/`println!` maison, basées sur cette instance globale.

## ✏️ Exercices

**Exercice 3.1 — Écrire un octet.** Sans encore construire de structure
complète, écris directement (en `unsafe`) un seul caractère à l'adresse
`0xb8000`, avec un attribut de couleur de ton choix, et vérifie qu'il
apparaît en haut à gauche de l'écran dans QEMU.

**Exercice 3.2 — Écrivain VGA structuré.** Construis une structure `Writer`
capable d'écrire une chaîne de caractères complète, gérant le retour à la
ligne (`\n`) et le défilement de l'écran quand le texte dépasse la dernière
ligne (*scrolling*) : les lignes doivent remonter d'une position, l'ancienne
première ligne disparaissant.

**Exercice 3.3 — `println!` maison.** Implémente `core::fmt::Write` pour ta
structure `Writer`, encapsule une instance globale protégée par un verrou, et
définis tes propres macros `print!` et `println!`. Utilise-les pour afficher
"Hello, kernel!" au démarrage de ton noyau.

**Exercice 3.4 — Utiliser `println!` dans le panic handler.** Reviens sur ton
gestionnaire de panique du chapitre 1 : fais-le maintenant afficher le
message de panique (`info`) à l'écran avant de boucler indéfiniment. Vérifie
en déclenchant volontairement un `panic!("test")` quelque part dans ton code.

## 🤔 Questions de réflexion

- Pourquoi l'accès au tampon VGA nécessite-t-il du code `unsafe`, alors que
  la quasi-totalité du reste de ton programme peut rester en Rust "sûr" ?
- Pourquoi une variable globale mutable ordinaire (`static mut`) est-elle
  problématique en Rust, et en quoi un verrou comme `spin::Mutex` résout-il
  ce problème même en environnement mono-cœur, sans threads au sens
  classique ?
- Que se passerait-il si deux parties de ton noyau (par exemple ton code
  normal et un gestionnaire d'interruption) essayaient d'écrire à l'écran en
  même temps sans aucune synchronisation ?

## 🚀 Pour aller plus loin

Ajoute la gestion de la couleur de texte à tes macros `println!` (par exemple
une macro `println_colored!` ou un paramètre de couleur), et explore les 16
couleurs disponibles en mode texte VGA standard.

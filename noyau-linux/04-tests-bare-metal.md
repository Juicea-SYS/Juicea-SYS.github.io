---
layout: default
title: "4. Tests bare-metal"
parent: "Cours 2 : Noyau Linux"
nav_order: 4
---

# 4. Tester un noyau bare-metal
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre pourquoi le framework de test standard de Rust ne fonctionne pas
  tel quel dans un environnement `no_std`.
- Faire communiquer ton noyau avec la machine hôte via un port série.
- Faire échouer/réussir automatiquement un test QEMU depuis le code du
  noyau, sans intervention humaine.

## 📚 Théorie

### Pourquoi le framework de test standard ne suffit pas

`cargo test` repose normalement sur `std` (pour organiser l'exécution des
tests, capturer les sorties, gérer les threads de test...). En environnement
`no_std`, il faut redéfinir un harnais de test minimal, avec Rust `nightly`
(fonctionnalité `custom_test_frameworks`), qui liste et exécute les fonctions
marquées `#[test_case]` — un mécanisme plus rudimentaire mais suffisant pour
nos besoins.

### Faire sortir QEMU avec un code de statut

Pour qu'un test automatisé puisse "réussir" ou "échouer" du point de vue de
l'extérieur (un script CI, par exemple), il faut un moyen de faire quitter
QEMU avec un code de retour précis, depuis l'intérieur du noyau. QEMU propose
pour cela un périphérique virtuel spécial, l'**isa-debug-exit**, un simple
port d'entrée/sortie sur lequel écrire un octet termine l'émulation avec un
code de sortie dérivé de cet octet — configurable via une option de ligne de
commande QEMU (`-device isa-debug-exit,iobase=...,iosize=...`).

### Le port série comme sortie de test

Le tampon VGA (chapitre 3) n'est pas facilement récupérable par un script
extérieur à QEMU. Le **port série** (UART 16550, présent depuis les tout
premiers PC) est une meilleure sortie pour les tests automatisés : QEMU peut
rediriger le port série virtuel de la machine émulée directement vers la
sortie standard du processus QEMU lui-même, ce qui permet à un script de test
de lire simplement ce qu'affiche ton noyau.

### Comment Linux fait (comparaison)

La suite de tests du noyau Linux (`kselftest`, ou le framework plus récent
**KUnit**) fonctionne sur un principe assez proche pour ses tests bas niveau :
exécution dans une machine virtuelle légère, avec un canal de sortie dédié
(souvent aussi la console série) permettant de récupérer les résultats de
façon automatisée, sans dépendre d'un affichage graphique.

## 🛠️ En pratique (Rust / QEMU)

- Configure ton `Cargo.toml` pour utiliser un harnais de test personnalisé
  (`#![feature(custom_test_frameworks)]`, `#![test_runner(...)]`).
- Ajoute une crate de communication avec le port série (voir
  [Ressources](../ressources/liens-utiles-os.html)).
- Configure QEMU pour rediriger le port série vers `stdio`, et pour utiliser
  le périphérique `isa-debug-exit`.

## ✏️ Exercices

**Exercice 4.1 — Port série.** Fais en sorte que ton noyau puisse écrire du
texte sur le port série, et vérifie que ce texte apparaît bien dans le
terminal qui a lancé QEMU (pas dans la fenêtre graphique de QEMU).

**Exercice 4.2 — Sortie de test contrôlée.** Écris une fonction qui écrit un
octet précis sur le port de l'`isa-debug-exit`, et vérifie que le processus
QEMU se termine avec le code de retour attendu (le code de sortie du
processus QEMU dépend de l'octet écrit selon une formule à documenter dans
tes notes — vérifie-la expérimentalement).

**Exercice 4.3 — Premier test automatisé.** Écris une fonction `#[test_case]`
qui vérifie une propriété simple de ton noyau (par exemple, que ton
`println!` du chapitre 3 ne panique pas sur une chaîne de caractères assez
longue pour déclencher un défilement d'écran). Fais en sorte que l'échec
comme le succès de ce test se traduisent par le bon code de sortie QEMU.

**Exercice 4.4 — Test d'intégration séparé.** Crée un fichier de test
d'intégration Cargo distinct (dans `tests/`), avec son propre point d'entrée
et gestionnaire de panique, pour tester un scénario plus complexe (par
exemple un `panic!` volontaire, en vérifiant que ton noyau réagit comme
attendu plutôt que de vérifier une absence de panique).

## 🤔 Questions de réflexion

- Pourquoi est-il utile de distinguer les tests "unitaires" (dans le même
  binaire que le noyau) des tests "d'intégration" (dans des binaires séparés,
  avec leur propre point d'entrée) dans ce contexte bare-metal particulier ?
- En quoi le port série est-il un meilleur choix que le tampon VGA pour des
  tests automatisés en ligne de commande ? Y a-t-il un scénario où
  l'inverse serait vrai ?
- Que se passe-t-il, à ton avis, si un test provoque un *triple fault* (une
  erreur si grave que le processeur redémarre) plutôt qu'un simple `panic!`
  Rust ? Comment pourrais-tu détecter ce cas depuis l'extérieur de QEMU ?

## 🚀 Pour aller plus loin

Intègre l'exécution de tes tests bare-metal dans un script ou une
configuration d'intégration continue (GitHub Actions, par exemple), pour
qu'ils s'exécutent automatiquement à chaque modification du code — une bonne
pratique directement transposable à n'importe quel projet logiciel sérieux.

---
layout: default
title: "Projet 2 : OS en Rust"
nav_order: 6
---

# Projet 2 — Sujet du projet : reconstruire un OS minimal en Rust
{: .no_toc }

1. TOC
{:toc}

## Contexte

Tu vas concevoir, en Rust, un **système d'exploitation minimal mais
fonctionnel** : un bootloader qui démarre ton noyau, un noyau capable de
gérer les interruptions, la mémoire virtuelle et l'allocation dynamique, et
au moins deux pilotes de périphériques. L'architecture s'inspire des grands
principes du noyau Linux (espace noyau, gestion des interruptions, modèle de
pilotes, mémoire virtuelle), à une échelle radicalement plus petite et
entièrement à ta portée.

⚠️ **Ce projet ne construit pas "Linux".** Il construit un noyau *inspiré* de
son architecture — exactement comme le [projet 1](sujet-projet.html)
construisait un CPU inspiré du SAP-1, sans être un processeur du commerce.

## Objectif final

Un noyau x86_64, démarré par un bootloader, capable de :

1. S'afficher à l'écran (mode texte VGA) sans dépendre d'aucun système
   d'exploitation hôte.
2. Gérer les exceptions du processeur sans provoquer de redémarrage
   intempestif (*triple fault*).
3. Réagir aux interruptions matérielles : au minimum un timer et un clavier.
4. Gérer sa mémoire virtuelle (pagination) et allouer dynamiquement de la
   mémoire (`Box`, `Vec`, `String` fonctionnels dans le noyau).
5. Exécuter au moins deux tâches de façon concurrente (multitâche coopératif).
6. Piloter au moins **deux périphériques** distincts (par exemple clavier +
   horloge temps réel, ou clavier + port série bidirectionnel).
7. Passer une suite de tests d'intégration automatisés, exécutés dans QEMU.

## Contraintes techniques

- **Langage** : Rust, en environnement `no_std` (pas de bibliothèque
  standard, pas d'OS hôte pour exécuter le binaire final).
- **Architecture cible** : x86_64.
- **Émulateur de test** : QEMU (`qemu-system-x86_64`).
- **Bootloader** : tu peux utiliser une crate Rust existante spécialisée dans
  la génération d'image de démarrage (voir
  [Ressources](ressources/liens-utiles-os.html)) plutôt que d'écrire tous les
  octets du secteur de démarrage toi-même — l'objectif du cours est de
  *comprendre* le processus de démarrage et de piloter le résultat, pas de
  réécrire un BIOS.
- Le projet suppose que tu es à l'aise avec les bases de Rust (ownership,
  traits, types génériques). Si ce n'est pas encore le cas, prévois un temps
  de mise à niveau avant de commencer — voir
  [chapitre 1](noyau-linux/01-environnement.html).

## Jalons du projet

Chaque jalon correspond à un chapitre du [cours 2](noyau-linux/) et doit
produire du code versionné (Git), avec des tests quand c'est pertinent.

| Jalon | Chapitre correspondant | Livrable attendu |
|---|---|---|
| J0 | Panorama de l'architecture Linux | Note de synthèse : les sous-systèmes de Linux et leur équivalent (simplifié) dans ton futur noyau |
| J1 | Environnement et outils | Toolchain Rust nightly + QEMU opérationnels, "hello" freestanding qui compile |
| J2 | Démarrage (boot) | Image bootable qui démarre dans QEMU jusqu'au noyau |
| J3 | Noyau minimal & VGA | Le noyau affiche du texte à l'écran |
| J4 | Tests bare-metal | Au moins 3 tests d'intégration automatisés passant dans QEMU |
| J5 | Interruptions CPU | Le noyau survit à une exception `breakpoint` et à un `double fault` sans redémarrer |
| J6 | Interruptions matérielles | Un timer et un premier pilote (clavier) fonctionnels |
| J7 | Mémoire virtuelle | Traduction d'adresses virtuelles vers physiques fonctionnelle |
| J8 | Allocation dynamique | `Box`, `Vec` utilisables dans le noyau |
| J9 | Multitâche coopératif | Au moins 2 tâches concurrentes (ex. clavier + tâche périodique) |
| J10 | Architecture des pilotes / deuxième pilote | Un second périphérique piloté (RTC, port série bidirectionnel...) |
| J11 | Système de fichiers / appels système *(extension)* | Voir section extensions |
| J12 | Assemblage final | Démonstration de bout en bout de l'objectif final |

## Extensions avancées (hors socle obligatoire)

- **Espace utilisateur (ring 3)** et un premier appel système minimal — un
  vrai changement de niveau de privilège, nettement plus exigeant que le
  socle obligatoire.
- **Système de fichiers minimal en mémoire** (façon *ramfs*), avec une API de
  lecture/écriture de fichiers virtuels.
- **Ordonnancement préemptif** de plusieurs contextes d'exécution isolés,
  au-delà du multitâche coopératif du socle obligatoire.

## Livrables

Pour chaque jalon, conserve (dans un dépôt Git dédié au projet, distinct de
ce cours) :

- Le code source Rust correspondant, avec un historique de commits lisible.
- Une capture d'écran (ou une capture vidéo courte) de QEMU montrant le
  comportement attendu.
- Les tests d'intégration correspondants, et leur sortie (succès/échec).
- Un court paragraphe expliquant un choix de conception ou une difficulté
  rencontrée — comme au projet 1, c'est souvent plus révélateur qu'un simple
  "ça marche".

## Grille d'auto-évaluation

- [ ] Le noyau compile et démarre sans erreur dans QEMU depuis un état
      propre (`cargo clean` puis rebuild).
- [ ] Je peux expliquer, sans notes, le chemin complet entre "le CPU
      s'allume" et "mon noyau affiche du texte".
- [ ] Une exception volontairement déclenchée (breakpoint) ne fait pas
      planter le système.
- [ ] Le noyau ne fuit pas de mémoire de façon évidente lors d'une exécution
      prolongée (à l'œil, pas besoin d'un profileur pour ce projet).
- [ ] J'ai testé au moins un cas limite par pilote (ex : appui de touche
      pendant une autre opération en cours).
- [ ] Je pourrais expliquer, pour chaque jalon, l'équivalent de ce que fait
      Linux au même niveau (même en version très simplifiée dans mon noyau).

## Ce que ce sujet ne te donne pas (volontairement)

- Le code de configuration de l'IDT ou de la table de pages.
- L'implémentation de l'allocateur de heap.
- Le câblage exact d'un pilote (clavier ou autre).
- L'implémentation de l'exécuteur async.

C'est précisément le travail à faire. Le [cours 2](noyau-linux/) te donne la
théorie et la méthode ; à toi de l'implémenter.

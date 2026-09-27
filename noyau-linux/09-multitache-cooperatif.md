---
layout: default
title: "9. Multitâche coopératif"
parent: "Cours 2 : Noyau Linux"
nav_order: 9
---

# 9. Multitâche coopératif avec async/await
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre la différence entre multitâche **coopératif** et
  **préemptif**.
- Comprendre comment `async`/`await` en Rust se traduit en une machine à
  états, exécutable sans système d'exploitation sous-jacent.
- Construire un exécuteur minimal et au moins deux tâches concurrentes.

## 📚 Théorie

### Coopératif contre préemptif

Un ordonnanceur **préemptif** (celui de Linux, `kernel/sched/`) peut
interrompre une tâche à tout moment (typiquement sur une interruption de
minuterie, chapitre 6) pour donner la main à une autre, sans que la tâche
interrompue n'ait à coopérer. C'est nettement plus complexe à mettre en
œuvre correctement (sauvegarde complète du contexte d'exécution, piles
séparées par tâche, protection contre les accès concurrents...).

Un ordonnanceur **coopératif** attend que chaque tâche rende volontairement
la main (typiquement quand elle n'a rien à faire en attendant un événement
externe). C'est plus simple à implémenter correctement, au prix d'un risque :
une tâche qui ne rend jamais la main bloque tout le système. C'est
l'approche que ce cours te propose pour le socle obligatoire du projet.

### `async`/`await` comme machine à états

Une fonction `async fn` en Rust n'exécute rien immédiatement : elle produit
une valeur implémentant le trait `Future`, une sorte de machine à états dont
la méthode `poll` avance d'une étape à chaque appel, jusqu'à ce qu'elle
retourne enfin un résultat (`Poll::Ready`) ou signale qu'elle n'est pas
encore prête (`Poll::Pending`). Le mot-clé `await` insère, en gros, un point
où l'exécution peut être suspendue en attendant qu'une autre `Future`
progresse. Cette mécanique ne dépend d'aucun système d'exploitation : c'est
une transformation purement faite par le compilateur, ce qui la rend
utilisable telle quelle dans un noyau `no_std` (à condition d'implémenter
soi-même l'exécuteur, ce que fournit normalement une bibliothèque comme
`tokio` sur un OS classique).

### Un exécuteur minimal

Un **exécuteur** (*executor*) maintient une file de tâches (chacune une
`Future` mise en boîte et "épinglée", `Pin<Box<dyn Future<...>>>`), et
appelle `poll` sur chacune tour à tour. La version la plus simple boucle sans
fin sur toutes les tâches ("busy polling"), ce qui fonctionne mais gaspille
du temps CPU à re-vérifier des tâches qui n'ont aucune chance d'avoir
progressé. Une version plus soignée utilise un mécanisme de **réveil**
(*waker*) : une tâche en attente enregistre un `Waker`, que la source de
l'événement attendu (par exemple un gestionnaire d'interruption clavier,
chapitre 6) appelle explicitement une fois l'événement survenu, ce qui
permet à l'exécuteur de ne re-tester que les tâches réellement susceptibles
d'avoir progressé.

### Relier une interruption à une tâche asynchrone

Un gestionnaire d'interruption (chapitre 6) doit rester **très bref** — il
ne doit surtout pas, par exemple, tenter d'acquérir un verrou potentiellement
déjà tenu (voir la question de réflexion du chapitre 8). Le pont classique
entre "interruption matérielle" et "tâche asynchrone" consiste à faire en
sorte que le gestionnaire d'interruption se contente de déposer une donnée
brute dans une file sans verrou bloquant (une file conçue pour être
utilisable sans attente, *lock-free*), puis réveille la tâche concernée. La
tâche asynchrone, elle, s'exécute plus tard, en dehors du contexte
d'interruption, et peut faire un traitement plus long sans risque.

### Comment Linux fait (comparaison)

Linux ordonnance de vrais processus et threads de façon préemptive
(actuellement via l'ordonnanceur **CFS**, *Completely Fair Scheduler*, ou son
successeur **EEVDF**), avec changement de contexte matériel complet à chaque
interruption de minuterie. Le mécanisme async/await de ce chapitre est
volontairement plus proche, dans l'esprit, des **tasklets**/**work queues**
internes du noyau Linux (du travail différé, traité en dehors du contexte
d'interruption immédiat) que de son ordonnanceur de processus complet.

## 🛠️ En pratique (Rust / QEMU)

Une file sans verrou bloquant à taille fixe (voir
[Ressources](../ressources/liens-utiles-os.html)) est particulièrement
adaptée pour transmettre des données depuis un gestionnaire d'interruption
vers le reste du noyau, sans risquer d'y acquérir un verrou classique.

## ✏️ Exercices

**Exercice 9.1 — Structure `Task`.** Définis un type `Task` encapsulant une
`Future<Output = ()>` mise en boîte et épinglée. Donne-lui un identifiant
unique.

**Exercice 9.2 — Exécuteur "busy polling".** Implémente un exécuteur minimal
qui maintient une file de `Task` et boucle indéfiniment en appelant `poll`
sur chacune, en retirant celles qui sont terminées. Teste-le avec une tâche
`async fn` triviale (par exemple qui affiche un message puis se termine
immédiatement).

**Exercice 9.3 — Clavier asynchrone.** Reprends ton pilote clavier
synchrone du chapitre 6 : au lieu de traiter le scancode directement dans le
gestionnaire d'interruption, fais-le déposer dans une file sans verrou
bloquant, puis réveille une tâche asynchrone dédiée qui lit cette file et
affiche les caractères décodés. Vérifie que le clavier fonctionne toujours,
mais que le traitement effectif se fait bien en dehors du gestionnaire
d'interruption (tu peux le vérifier en ajoutant temporairement un
`println!` dans le gestionnaire d'interruption pour confirmer qu'il ne fait
plus que déposer la donnée).

**Exercice 9.4 — Deux tâches concurrentes.** Ajoute une seconde tâche
indépendante (par exemple, un compteur qui s'incrémente périodiquement en
s'appuyant sur l'interruption de minuterie du chapitre 6, via le même
principe de file + réveil). Vérifie que les deux tâches (clavier et compteur)
progressent bien de façon entrelacée, sans que l'une bloque l'autre.

## 🤔 Questions de réflexion

- Pourquoi un exécuteur "busy polling" (exercice 9.2) fonctionne-t-il, mais
  est-il un mauvais choix en pratique sur du matériel réel (indice : pense à
  la consommation électrique d'un cœur de processeur qui boucle sans arrêt) ?
- En quoi le mécanisme de `Waker` change-t-il ce problème ?
- Qu'est-ce qui, structurellement, empêche ton noyau (dans son socle
  obligatoire, sans l'extension "espace utilisateur") de faire du véritable
  multitâche **préemptif** avec isolation mémoire entre tâches ? Qu'est-ce
  qu'il te manque par rapport à un vrai processus Linux ?

## 🚀 Pour aller plus loin

Ajoute une instruction `hlt` (halt) dans ton exécuteur quand aucune tâche
n'est prête à progresser, plutôt que de boucler activement ("busy
polling") — avec les précautions nécessaires pour que le processeur se
réveille bien à la prochaine interruption. Mesure (grossièrement) l'effet sur
l'utilisation CPU rapportée par ta machine hôte pendant que QEMU tourne.

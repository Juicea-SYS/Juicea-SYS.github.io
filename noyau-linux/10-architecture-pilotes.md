---
layout: default
title: "10. Architecture des pilotes"
parent: "Cours 2 : Noyau Linux"
nav_order: 10
---

# 10. Architecture des pilotes de périphériques
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre le modèle général d'un pilote de périphérique dans Linux
  (device model, périphériques caractère / bloc).
- Dégager, à partir de ton pilote clavier (chapitre 6/9), une interface
  générique réutilisable pour d'autres pilotes.
- Écrire un second pilote complet, de bout en bout.

## 📚 Théorie

### Le modèle de pilotes de Linux, en survol

Linux distingue plusieurs grandes catégories de périphériques :

- **Périphériques caractère** (*character devices*) : flux d'octets sans
  structure imposée, lus/écrits séquentiellement (clavier, port série,
  souris). Exposés côté utilisateur via des fichiers spéciaux dans `/dev`
  (`/dev/input/event0`, `/dev/ttyS0`...).
- **Périphériques bloc** (*block devices*) : accès par blocs de taille fixe,
  avec accès aléatoire possible (disques durs, SSD). Exposés via `/dev/sda`,
  etc., et généralement accédés à travers un système de fichiers plutôt que
  directement.
- **Périphériques réseau** : traités à part, via une interface encore
  différente (hors périmètre de ce cours).

Chaque pilote implémente un jeu d'opérations standard (`open`, `read`,
`write`, `ioctl`...) défini par une structure d'opérations commune
(`file_operations` pour les périphériques caractère), ce qui permet au reste
du noyau (et, via le VFS, à l'espace utilisateur) de manipuler n'importe quel
périphérique de façon uniforme, sans connaître ses détails internes.

### Dégager une interface commune dans ton noyau

Ton pilote clavier (chapitres 6 et 9) a déjà, en réalité, une forme
implicite : une source d'événements matériels (interruption), une file de
transmission, une tâche de traitement asynchrone. Ce chapitre te propose de
rendre cette structure **explicite et réutilisable**, sous la forme d'un
petit trait Rust commun (par exemple avec une méthode d'initialisation et une
méthode de traitement d'événement), que n'importe quel nouveau pilote pourra
implémenter.

### Comment Linux fait (comparaison)

La structure `file_operations` de Linux joue exactement ce rôle
d'abstraction : elle permet au VFS d'appeler `read()` sur n'importe quel
périphérique caractère sans connaître son type réel, exactement comme ton
trait Rust permettra au reste de ton noyau de traiter n'importe quel pilote
de façon uniforme.

## 🛠️ En pratique (Rust / QEMU)

Choisis un second périphérique à piloter, par exemple :

- **L'horloge temps réel (RTC)** : lecture de la date/heure courante via les
  ports d'entrée/sortie `0x70`/`0x71` (registres CMOS).
- **Le port série, en réception** (au-delà de la sortie déjà utilisée au
  chapitre 4) : un pilote clavier "à distance", en somme.
- Tout autre périphérique simple exposé par QEMU que tu souhaiterais explorer
  (vérifie sa documentation avant de te lancer).

## ✏️ Exercices

**Exercice 10.1 — Trait commun.** Définis un trait Rust représentant
l'interface minimale d'un pilote dans ton noyau (par exemple une méthode
d'initialisation, une méthode appelée à réception d'un événement). Fais en
sorte que ton pilote clavier existant l'implémente, sans changer son
comportement observable.

**Exercice 10.2 — Choisir et documenter le second périphérique.** Avant
d'écrire du code, documente (dans un fichier texte ou un commentaire) : les
ports d'entrée/sortie utilisés, le format des données lues/écrites, et le
numéro d'interruption concerné le cas échéant.

**Exercice 10.3 — Implémenter le second pilote.** Implémente ce pilote en
suivant le même schéma que ton pilote clavier (interruption → file sans
verrou bloquant → tâche asynchrone, si le périphérique est piloté par
interruption ; lecture/écriture directe de ports sinon), en utilisant ton
trait commun de l'exercice 10.1.

**Exercice 10.4 — Test d'intégration du second pilote.** Écris un test
d'intégration (chapitre 4) qui vérifie le bon fonctionnement de ce second
pilote de façon automatisée.

## 🤔 Questions de réflexion

- En quoi ton trait commun de pilote (exercice 10.1) joue-t-il, à très petite
  échelle, un rôle comparable à `file_operations` dans Linux ?
- Pourquoi Linux distingue-t-il aussi nettement périphériques caractère et
  périphériques bloc ? Ton second pilote est-il plutôt de l'un ou l'autre
  type ?
- Si tu devais ajouter un troisième pilote demain, quelles parties de ton
  architecture actuelle faciliteraient cet ajout, et lesquelles devraient
  encore être généralisées ?

## 🚀 Pour aller plus loin

Ajoute un mécanisme minimal de "registre de pilotes" : une liste (dans un
`Vec`, maintenant que tu as un allocateur — chapitre 8) de tous les pilotes
actifs, initialisés automatiquement au démarrage plutôt qu'un par un à la
main dans ton point d'entrée. C'est un embryon de ce que Linux appelle le
*device model*.

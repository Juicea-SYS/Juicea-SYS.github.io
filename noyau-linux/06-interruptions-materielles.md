---
layout: default
title: "6. Interruptions matérielles"
parent: "Cours 2 : Noyau Linux"
nav_order: 6
---

# 6. Interruptions matérielles et premier pilote
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre comment un contrôleur d'interruptions relie les périphériques au
  processeur.
- Gérer une interruption de minuterie (*timer*) périodique.
- Écrire un premier pilote de périphérique : le clavier.

## 📚 Théorie

### Le contrôleur programmable d'interruptions (PIC)

Le processeur seul ne peut pas savoir directement "quel périphérique" a
besoin d'attention : c'est le rôle d'un contrôleur dédié, historiquement le
**PIC 8259** (*Programmable Interrupt Controller*) sur PC, qui reçoit les
signaux d'interruption des différents périphériques (minuterie, clavier,
disque...) et les transmet au processeur avec un numéro d'interruption
donné. Les PC modernes utilisent en réalité un contrôleur plus avancé, l'
**APIC**, mais le PIC 8259 reste largement utilisé en pédagogie pour sa
simplicité — c'est celui que ce cours te recommande pour ton socle
obligatoire.

### Remapper le PIC : un piège classique

Par défaut, le PIC 8259 utilise des numéros d'interruption qui **entrent en
conflit** avec ceux déjà utilisés par les exceptions CPU du chapitre 5 (0 à
31). Une étape indispensable, avant d'activer les interruptions matérielles,
est donc de **reprogrammer** le PIC pour qu'il utilise une plage de numéros
libres (typiquement à partir de 32). Oublier cette étape est une source de
bugs classique et déroutante (un clavier qui semble déclencher une exception
CPU sans rapport).

### Activer les interruptions

Le processeur ne réagit aux interruptions matérielles que si son indicateur
d'interruption (*interrupt flag*) est activé (instruction `sti`, *set
interrupt flag*) — jusque-là, tout se passe comme si les interruptions
matérielles étaient "masquées". C'est une étape volontaire et tardive dans
l'initialisation d'un noyau : on ne l'active qu'une fois l'IDT et le PIC
correctement configurés, pour éviter de recevoir une interruption qu'on ne
saurait pas encore traiter proprement.

### Le pilote clavier : scancodes

Le clavier PS/2 envoie, à chaque appui ou relâchement de touche, un ou
plusieurs octets appelés **scancodes**, lus depuis un port d'entrée/sortie
dédié (le port `0x60`). Le mappage entre scancode et touche réelle dépend
d'un "jeu de scancodes" (*scancode set*) — plusieurs existent, pour des
raisons historiques de compatibilité ascendante.

### Comment Linux fait

Le pilote clavier de Linux (`drivers/input/keyboard/`) suit le même principe
de base (lecture de scancodes sur interruption), mais dans une architecture
bien plus générale : la couche **Input** de Linux reçoit ces événements bas
niveau et les transforme en événements génériques (`KEY_A` pressé/relâché...)
consommables par n'importe quelle application, via `/dev/input/`, quel que
soit le type de clavier physique (PS/2, USB, virtuel).

## 🛠️ En pratique (Rust / QEMU)

Des crates existantes fournissent un pilotage simplifié du PIC 8259 (avec la
plage de remappage déjà décrite) et un décodage des scancodes vers des
touches/caractères lisibles (voir
[Ressources](../ressources/liens-utiles-os.html)) — utilise-les pour te
concentrer sur l'architecture générale (comment brancher une interruption sur
un gestionnaire, comment router l'information ensuite) plutôt que sur le
détail exact d'une table de correspondance scancode/caractère.

## ✏️ Exercices

**Exercice 6.1 — Remapper le PIC.** Initialise et remappe le PIC 8259 sur une
plage de numéros d'interruption qui ne chevauche pas les exceptions CPU (0 à
31). Vérifie que ce remappage n'introduit lui-même aucune régression sur les
gestionnaires du chapitre 5.

**Exercice 6.2 — Interruption de minuterie.** Enregistre un gestionnaire pour
l'interruption de minuterie (*timer*, PIT), qui incrémente un compteur global
à chaque interruption. Active les interruptions (`sti`). Vérifie, en
affichant périodiquement ce compteur, qu'il progresse bien de façon continue.
N'oublie pas de signaler au PIC la fin du traitement de chaque interruption
(*End Of Interrupt*), sans quoi les interruptions suivantes de ce type
resteraient bloquées.

**Exercice 6.3 — Pilote clavier synchrone.** Enregistre un gestionnaire pour
l'interruption clavier, qui lit le scancode reçu (port `0x60`), le décode en
caractère lisible, et l'affiche via ton `println!`. Teste avec plusieurs
touches, y compris des touches spéciales (majuscule, touches fléchées) pour
voir comment elles sont représentées.

## 🤔 Questions de réflexion

- Pourquoi le remappage du PIC est-il une étape "invisible" mais critique —
  que se passerait-il concrètement, dans ton gestionnaire d'exceptions du
  chapitre 5, si tu l'omettais ?
- Pourquoi doit-on explicitement signaler au PIC la fin du traitement d'une
  interruption (*End Of Interrupt*) ? Que se passerait-il si on l'oubliait,
  pour les interruptions suivantes du même type ?
- Le pilote clavier de l'exercice 6.3 traite le scancode **directement dans
  le gestionnaire d'interruption**. Quelles limites cela pose-t-il si le
  traitement devient plus complexe (par exemple, mettre à jour une interface
  graphique) ? Garde cette question en tête pour le chapitre 9.

## 🚀 Pour aller plus loin

Renseigne-toi sur l'**APIC** (*Advanced Programmable Interrupt Controller*),
qui a remplacé le PIC 8259 dans les PC modernes, notamment pour gérer
plusieurs cœurs de processeur. Pourquoi un simple PIC 8259, conçu à l'origine
pour un processeur mono-cœur, ne suffit-il plus sur une machine multicœur ?

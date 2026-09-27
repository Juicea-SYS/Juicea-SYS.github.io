---
layout: default
title: "7. Mémoire virtuelle"
parent: "Cours 2 : Noyau Linux"
nav_order: 7
---

# 7. Mémoire virtuelle et pagination
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre pourquoi la mémoire virtuelle est indispensable à un système
  multi-programmes.
- Comprendre le mécanisme de pagination x86_64 (tables de pages à 4 niveaux).
- Traduire une adresse virtuelle en adresse physique depuis ton noyau.

## 📚 Théorie

### Pourquoi la mémoire virtuelle ?

Sans mémoire virtuelle, chaque programme manipulerait directement des
adresses physiques réelles — un programme pourrait alors, par erreur ou
malveillance, lire ou écrire la mémoire d'un autre programme, ou du noyau
lui-même. La **mémoire virtuelle** introduit un niveau d'indirection : chaque
programme (et le noyau) manipule des adresses **virtuelles**, traduites par
le matériel en adresses **physiques** réelles via des tables de traduction
que seul le noyau peut modifier. Cela permet aussi à chaque programme de
croire disposer d'un espace d'adressage complet et privé, indépendamment de
la mémoire physique réellement installée.

### La pagination x86_64 : 4 niveaux de tables

Sur x86_64, la traduction adresse virtuelle → adresse physique se fait par
**pagination**, à travers une hiérarchie de 4 niveaux de tables (chacune de
512 entrées de 8 octets, tenant exactement dans une page de 4 Ko) :

`PML4` (niveau 4) → `PDPT` (niveau 3) → `PD` (niveau 2) → `PT` (niveau 1) →
adresse physique de la page (généralement 4 Ko).

Une adresse virtuelle 64 bits est découpée en plusieurs champs : un index
dans chacun de ces 4 niveaux, plus un décalage (*offset*) dans la page
finale. Le registre `CR3` du processeur pointe vers la table de premier
niveau (PML4) actuellement active — changer `CR3` change donc entièrement
l'espace d'adressage vu par le processeur, ce qui est justement la mécanique
utilisée pour isoler des processus entre eux (bien au-delà du socle
obligatoire de ce projet).

### Ce que le bootloader te donne déjà

Ton noyau démarre déjà avec la pagination **activée** par le bootloader
(chapitre 2) — le mode long l'exige. Le bootloader te transmet, dans la
structure `BootInfo`, un décalage (*physical memory offset*) auquel il a déjà
mappé l'intégralité de la mémoire physique disponible en mémoire virtuelle.
Cette astuce te permet de lire/modifier n'importe quelle table de pages
(elles vivent en mémoire physique) sans avoir toi-même à réserver un mappage
spécial pour cela.

### Comment Linux fait

Linux (`mm/`) gère une pagination bien plus riche : mémoire virtuelle par
processus (chaque processus a son propre jeu de tables de pages, changé à
chaque changement de contexte), pages partagées en copie différée
(*copy-on-write*), échange sur disque (*swap*), zones mémoire distinctes
(DMA, mémoire normale, mémoire haute)... Ce chapitre te fait construire la
brique de base sur laquelle tout cela repose : la traduction d'adresse et la
capacité à créer de nouveaux mappages.

## 🛠️ En pratique (Rust / QEMU)

Une bibliothèque bas niveau existante (la même qu'au chapitre 5, voir
[Ressources](../ressources/liens-utiles-os.html)) fournit des types
représentant une table de pages et des opérations de traduction/mappage,
sans que tu aies à manipuler toi-même le format binaire exact d'une entrée de
table de pages.

## ✏️ Exercices

**Exercice 7.1 — Lire la table de premier niveau.** En utilisant le registre
`CR3` et le décalage de mémoire physique fourni par `BootInfo`, accède en
lecture à la table PML4 active, et affiche (via ton `println!`) le nombre
d'entrées actuellement utilisées (non nulles) dans cette table.

**Exercice 7.2 — Traduire une adresse.** Implémente une fonction qui prend
une adresse virtuelle et retourne l'adresse physique correspondante, en
parcourant manuellement les 4 niveaux de tables. Teste-la sur une adresse
dont tu connais déjà, par une autre méthode, l'adresse physique attendue
(par exemple une adresse à l'intérieur de ton propre code noyau).

**Exercice 7.3 — Un allocateur de cadres physiques.** À partir de la carte
mémoire fournie par le bootloader (quelles zones physiques sont "utilisables"),
implémente un allocateur simple (*bump allocator*, sans possibilité de
libération individuelle pour l'instant) capable de fournir des cadres
physiques libres, un par un.

**Exercice 7.4 — Créer un nouveau mappage.** En utilisant ton allocateur de
cadres (7.3), crée un nouveau mappage : associe une page virtuelle de ton
choix (qui n'était pas déjà mappée) à un nouveau cadre physique fraîchement
alloué, en modifiant les tables de pages nécessaires (en créant les niveaux
intermédiaires manquants si besoin). Vérifie que tu peux ensuite écrire puis
relire une valeur à cette adresse virtuelle.

## 🤔 Questions de réflexion

- Pourquoi la traduction d'adresse se fait-elle sur 4 niveaux plutôt qu'une
  seule grande table plate ? (Indice : recalcule la taille qu'occuperait une
  table plate pour tout l'espace d'adressage 64 bits, et compare à la mémoire
  réellement installée sur un PC.)
- Le décalage de mémoire physique fourni par le bootloader mappe *toute* la
  mémoire physique en un seul bloc contigu en mémoire virtuelle. Pourquoi
  cette astuce ne serait-elle pas praticable pour une machine possédant, par
  exemple, plusieurs téraoctets de RAM ?
- Que se passe-t-il, concrètement dans les tables de pages, quand un
  programme provoque un *page fault* en accédant à une adresse non mappée
  (évoqué au chapitre 5) ?

## 🚀 Pour aller plus loin

Renseigne-toi sur les **pages énormes** (*huge pages*, 2 Mo ou 1 Go au lieu
de 4 Ko) : pourquoi réduisent-elles la pression sur le **TLB**
(*Translation Lookaside Buffer*, le cache matériel de traductions
d'adresses), et dans quels contextes (bases de données, machines virtuelles)
leur usage est-il particulièrement répandu ?

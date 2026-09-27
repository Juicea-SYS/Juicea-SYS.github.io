---
layout: default
title: "4. Bascules et mémoire"
parent: Cours
nav_order: 4
---

# 4. Bascules et mémoire
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre la différence entre circuit combinatoire et circuit séquentiel.
- Construire une bascule SR, puis une bascule D synchronisée sur un front
  d'horloge.
- Construire un registre 8 bits à partir de bascules D.

⏱️ C'est historiquement le chapitre le plus déroutant du cours : prends ton
temps, et n'hésite pas à revenir plusieurs fois sur la notion de
rétroaction (*feedback*).

## 📚 Théorie

### Pourquoi un circuit combinatoire ne suffit pas

Tous les circuits vus jusqu'ici sont combinatoires : leur sortie ne dépend que
de leurs entrées *actuelles*. Or un ordinateur doit **retenir** de
l'information (le contenu d'un registre, d'une case mémoire) même quand les
entrées changent ou disparaissent. Il faut donc un circuit dont la sortie
peut dépendre de son **propre état passé** : c'est le principe de la
**rétroaction** — reboucler une sortie vers une entrée.

### La bascule SR (Set-Reset)

La bascule SR est la brique la plus simple dotée de mémoire. Elle a deux
entrées, `S` (*set*) et `R` (*reset*), et une sortie `Q` qui "retient" son état :

- `S=1, R=0` → force `Q=1` (et le garde même quand `S` repasse à 0).
- `S=0, R=1` → force `Q=0` (et le garde).
- `S=0, R=0` → `Q` **conserve** sa valeur précédente (c'est la mémoire !).
- `S=1, R=1` → état interdit / non défini (à éviter).

Une bascule SR se construit classiquement avec deux portes NOR (ou deux NAND,
selon la convention) reliées en croix : la sortie de l'une alimente une
entrée de l'autre, et réciproquement.

### La bascule D et la synchronisation par horloge

Dans un ordinateur, on ne veut pas que chaque registre change dès qu'une
entrée bouge : on veut que **tous** les registres changent **en même temps**,
au rythme d'une horloge commune. C'est le rôle de la **bascule D** (*Data*) :
elle a une entrée de donnée `D`, une entrée d'horloge `CLK`, et une sortie
`Q`. À chaque **front montant** de `CLK` (transition de 0 à 1), elle recopie
`D` dans `Q`, et garde cette valeur jusqu'au front suivant, quoi qu'il arrive
à `D` entre-temps.

Cette notion de **front** (et pas simplement de niveau haut/bas) est cruciale :
c'est elle qui garantit qu'un registre lit une donnée stable, prise à un
instant précis, plutôt que de "suivre" en permanence son entrée.

### Du bit au registre

Un registre 8 bits n'est rien d'autre que 8 bascules D, partageant toutes le
même signal d'horloge, chacune stockant un bit.

## 🛠️ Dans Digital Logic Sim

Vérifie si ta version de Digital Logic Sim fournit une bascule D (ou un
composant "flip-flop") de base. Si oui, comprends son fonctionnement en la
testant avant de l'utiliser. Si ta version ne fournit que des portes de base,
tu devras construire ta bascule SR puis ta bascule D entièrement à partir de
tes portes NAND/NOR maison — c'est plus instructif, donc à privilégier si tu
en as le temps.

## ✏️ Exercices

**Exercice 4.1 — Bascule SR.** Construis une bascule SR (avec des NOR ou des
NAND croisées). Teste la séquence suivante et vérifie que `Q` se comporte
comme décrit dans la théorie :
`(S,R) = (1,0) → (0,0) → (0,1) → (0,0)`.

**Exercice 4.2 — Bascule D synchrone.** À partir de ta bascule SR (ou du
composant fourni par l'outil), construis une bascule `D_MAISON` avec une
entrée `D`, une entrée `CLK`, une sortie `Q`. Vérifie que `Q` ne change **que**
sur un front montant de `CLK`, jamais quand `D` change seul.

**Exercice 4.3 — Registre 8 bits.** Construis `REGISTRE_8BITS` : 8 entrées de
donnée, 1 entrée `CLK`, 8 sorties. Vérifie qu'à chaque front montant de
`CLK`, les 8 sorties prennent la valeur des 8 entrées à cet instant précis, et
la conservent ensuite.

## 🤔 Questions de réflexion

- Pourquoi l'état `S=1, R=1` est-il considéré comme invalide dans une bascule
  SR ? Que se passe-t-il concrètement dans le circuit à deux portes croisées
  dans ce cas ?
- Pourquoi synchroniser tous les registres sur la **même** horloge est-il
  essentiel pour la fiabilité d'un ordinateur ? Que pourrait-il se passer si
  deux registres étaient cadencés par deux horloges légèrement différentes ?
- En quoi une bascule D "à front" est-elle différente d'un simple verrou
  (*latch*) "à niveau" ? (Tu peux chercher la différence entre *latch* et
  *flip-flop* si le terme n'est pas clair.)

## 🚀 Pour aller plus loin

Ajoute une entrée `LOAD` (ou `EN`, *enable*) à ton `REGISTRE_8BITS` : le
registre ne doit se mettre à jour sur le front d'horloge que si `LOAD=1` ;
sinon il garde sa valeur même si `CLK` reçoit un front. Cette variante te
servira directement au chapitre 5.

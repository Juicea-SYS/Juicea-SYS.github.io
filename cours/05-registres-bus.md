---
layout: default
title: "5. Registres, bus et multiplexage"
parent: Cours
nav_order: 5
---

# 5. Registres, bus et multiplexage
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre pourquoi un ordinateur partage un bus unique entre plusieurs
  composants plutôt que de tout relier point à point.
- Comprendre le rôle d'une sortie "trois états" (*tri-state*).
- Construire un bus 8 bits partagé par plusieurs registres.

## 📚 Théorie

### Le problème du câblage point à point

Un CPU contient de nombreux registres (ACC, B, IR, PC, MAR, RAM...) qui
doivent tous, à un moment ou un autre, échanger des données entre eux.
Relier chaque paire de composants par des fils dédiés exploserait rapidement
en complexité. La solution classique : un **bus** — un ensemble de fils
partagés (8 fils pour un bus 8 bits) sur lequel **un seul composant à la
fois** dépose une valeur, que **plusieurs** composants peuvent lire
simultanément.

### Sorties trois états

Pour qu'un bus fonctionne, il faut que les composants qui n'émettent pas sur
le bus à un instant donné ne "pollue" pas le bus avec leur propre valeur. On
utilise pour cela des sorties dites **trois états** (*tri-state*) : en plus
de `0` et `1`, elles peuvent être dans un état de **haute impédance**, c'est-à-dire
déconnectées électriquement du bus, comme si le fil n'existait pas pour ce
composant.

Si Digital Logic Sim ne propose pas nativement de sortie trois états, on peut
obtenir un comportement équivalent avec des **multiplexeurs** : plutôt que
plusieurs composants qui "poussent" leur valeur sur le même bus, un
multiplexeur choisit lequel des registres alimente effectivement le bus à cet
instant. Le résultat logique est le même ; c'est cette approche que ce cours
te recommande d'utiliser dans Digital Logic Sim.

### Signaux de contrôle : lire et écrire sur le bus

Chaque registre relié au bus a typiquement deux signaux de contrôle
indépendants :

- **un signal d'écriture sur le bus** (souvent noté `OUT` ou `EN`) : ce
  registre doit-il déposer sa valeur sur le bus maintenant ?
- **un signal de chargement depuis le bus** (souvent noté `IN` ou `LOAD`) :
  ce registre doit-il, au prochain front d'horloge, mémoriser la valeur
  actuellement présente sur le bus ?

C'est la combinaison de ces signaux, activés au bon moment, qui permettra à
l'unité de contrôle (chapitre 8) d'orchestrer des transferts comme
"copier l'accumulateur dans le registre B" en une seule micro-étape.

## 🛠️ Dans Digital Logic Sim

Combine ton `REGISTRE_8BITS` avec `LOAD` (chapitre 4) et un multiplexeur pour
simuler le comportement d'un bus partagé.

## ✏️ Exercices

**Exercice 5.1 — Registre avec sortie contrôlée.** Reprends ton
`REGISTRE_8BITS` (avec `LOAD`) et ajoute-lui une sortie contrôlée par un
signal `OUT_EN` : si `OUT_EN=0`, la sortie du registre vaut `00000000` (ou
toute valeur neutre convenue) ; si `OUT_EN=1`, la sortie recopie le contenu du
registre. Utilise une porte AND sur chaque bit de sortie, répétée 8 fois.

**Exercice 5.2 — Bus à 3 registres.** Construis un circuit avec 3 instances
de ton registre du 5.1, toutes reliées à un même bus 8 bits (implémenté avec
des multiplexeurs si nécessaire), plus une entrée pour choisir laquelle des 3
sources alimente le bus. Vérifie qu'à tout instant, **une seule** valeur est
visible sur le bus, et qu'elle correspond bien au registre sélectionné.

**Exercice 5.3 — Transfert entre registres.** À l'aide du circuit du 5.2,
réalise "manuellement" (en actionnant toi-même les signaux) un transfert du
registre 1 vers le registre 2 : active la sortie du registre 1 sur le bus,
active le chargement du registre 2, déclenche un front d'horloge, puis
vérifie que le registre 2 contient bien l'ancienne valeur du registre 1.

## 🤔 Questions de réflexion

- Pourquoi est-il dangereux, électriquement, que deux composants déposent une
  valeur sur le même bus en même temps ? Que se passerait-il si un fil était
  forcé à `0` par un composant et à `1` par un autre simultanément ?
- Dans l'exercice 5.3, que se passerait-il si tu activais le chargement du
  registre 2 **avant** d'avoir activé la sortie du registre 1 sur le bus ?
- Combien de signaux de contrôle distincts (au minimum) faut-il pour piloter
  `n` registres reliés à un même bus, chacun pouvant lire et écrire ?

## 🚀 Pour aller plus loin

Réfléchis à ce qui se passerait si deux registres avaient leur signal `OUT_EN`
activé en même temps par erreur, dans ton implémentation à base de
multiplexeur. Ce cas est-il seulement possible dans ta construction, ou
l'as-tu rendu structurellement impossible ? C'est une question de conception
importante pour le chapitre 8.

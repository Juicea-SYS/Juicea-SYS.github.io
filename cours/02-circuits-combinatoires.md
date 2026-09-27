---
layout: default
title: "2. Circuits combinatoires"
parent: Cours
nav_order: 2
---

# 2. Circuits combinatoires
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Distinguer circuit combinatoire et circuit séquentiel.
- Construire un additionneur binaire (demi-additionneur, additionneur complet,
  puis 8 bits).
- Construire un décodeur et un multiplexeur, briques omniprésentes en
  architecture des ordinateurs.

## 📚 Théorie

Un circuit **combinatoire** a une propriété importante : sa sortie ne dépend
**que** de ses entrées actuelles, jamais de ce qui s'est passé avant. Pas de
mémoire, pas d'état interne. C'est l'opposé des circuits **séquentiels** (vus
au chapitre 4), dont la sortie dépend aussi de l'historique.

### L'addition binaire

Additionner deux bits, c'est comme additionner deux chiffres décimaux : il
peut y avoir une retenue (*carry*).

Un **demi-additionneur** (*half adder*) additionne deux bits `A` et `B` et
produit une somme `S` et une retenue `C` :

| A | B | S | C |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

On remarque que `S = A ⊕ B` et `C = A · B`.

Un **additionneur complet** (*full adder*) prend en plus une retenue entrante
`Cin` (venant du bit de poids plus faible), et produit `S` et `Cout` :

| A | B | Cin | S | Cout |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 |
| 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 |

En chaînant 8 additionneurs complets (la retenue sortante de l'un devient la
retenue entrante du suivant), on obtient un **additionneur 8 bits à propagation
de retenue** (*ripple-carry adder*).

### Décodeur

Un décodeur *n-vers-2ⁿ* transforme un nombre binaire de `n` bits en une seule
ligne active parmi `2ⁿ` sorties. Par exemple, un décodeur 2-vers-4 : l'entrée
`00` active la sortie 0, `01` active la sortie 1, etc. C'est le mécanisme de
base du **décodage d'adresse mémoire** et du **décodage d'instruction**.

### Multiplexeur

Un multiplexeur (*mux*) *2ⁿ-vers-1* sélectionne, parmi `2ⁿ` entrées, celle qui
doit être recopiée en sortie, selon une valeur de sélection sur `n` bits.
C'est l'équivalent numérique d'un aiguillage : il "choisit" un chemin de
données parmi plusieurs. On le retrouvera partout : choix de l'opération de
l'ALU, choix de la source du bus, etc.

## 🛠️ Dans Digital Logic Sim

Utilise tes puces `AND_MAISON`, `OR_MAISON`, `XOR_MAISON`, `NOT_MAISON` du
chapitre 1 comme briques. Construis chaque circuit de ce chapitre comme une
puce séparée et nommée, avant de les combiner.

## ✏️ Exercices

**Exercice 2.1 — Demi-additionneur.** Construis `DEMI_ADD` (2 entrées `A`,
`B` ; 2 sorties `S`, `C`) respectant la table de vérité ci-dessus.

**Exercice 2.2 — Additionneur complet.** Construis `ADD_COMPLET` (3 entrées
`A`, `B`, `Cin` ; 2 sorties `S`, `Cout`). Indice : un additionneur complet peut
se construire à partir de deux demi-additionneurs et une porte OU — vérifie
si ta construction respecte bien les 8 lignes de la table de vérité avant de
continuer.

**Exercice 2.3 — Additionneur 8 bits.** En chaînant 8 instances de
`ADD_COMPLET`, construis `ADD_8BITS` : deux entrées de 8 bits chacune, une
entrée `Cin` (mets-la à 0 pour une simple addition), une sortie de 8 bits et
une sortie `Cout`. Teste au moins : `0 + 0`, `255 + 1` (dépassement de
capacité), et deux valeurs quelconques.

**Exercice 2.4 — Décodeur 2-vers-4.** Construis un décodeur à 2 entrées et 4
sorties, une seule sortie active à la fois selon la valeur binaire des 2
entrées.

**Exercice 2.5 — Multiplexeur 4-vers-1.** Construis un multiplexeur à 4
entrées de données, 2 entrées de sélection, 1 sortie.

## 🤔 Questions de réflexion

- Pourquoi un additionneur à propagation de retenue (*ripple-carry*) devient-il
  plus lent quand on augmente le nombre de bits ? (Indice : pense au chemin
  que doit parcourir une retenue qui doit se propager du bit 0 au bit 7.)
- Quel lien vois-tu entre un décodeur et un multiplexeur ? Pourrais-tu
  construire l'un à partir de l'autre ?

## 🚀 Pour aller plus loin

Cherche ce qu'est un additionneur à retenue anticipée (*carry-lookahead
adder*) et pourquoi il résout le problème de lenteur évoqué ci-dessus. Tu
n'as pas besoin de l'implémenter pour ce projet, mais comprendre le compromis
vitesse / complexité est une bonne intuition d'architecture.

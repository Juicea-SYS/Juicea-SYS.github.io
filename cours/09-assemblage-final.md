---
layout: default
title: "9. Assemblage final du CPU"
parent: Cours
nav_order: 9
---

# 9. Assemblage final du CPU
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Assembler tous les blocs précédents en un CPU complet.
- Exécuter un programme réel et observer le cycle fetch-decode-execute en
  action, micro-étape par micro-étape.
- Déboguer méthodiquement un système numérique complexe.

## 📚 Théorie

### Ce chapitre n'introduit (presque) aucune nouvelle notion

Tu as déjà construit, dans les chapitres précédents, toutes les briques
nécessaires :

- Chapitre 2/3 : ALU
- Chapitre 4/5 : registres et bus
- Chapitre 6 : RAM, PC, MAR, IR
- Chapitre 8 : unité de contrôle

L'assemblage final consiste à **relier ces puces entre elles** selon
l'architecture décrite dans le [sujet du projet](../sujet-projet.html), en
laissant l'unité de contrôle piloter tous les signaux `LOAD`/`OUT_EN`/`WRITE`
que tu as identifiés au chapitre 8.

### Une méthode de débogage qui a fait ses preuves

Face à un système avec autant de pièces mobiles, une bonne discipline de
débogage compte souvent plus que la théorie :

1. **Avance horloge par horloge** (pas en continu) au début, et note l'état
   de chaque registre à chaque front.
2. **Vérifie le fetch en isolation** d'abord (les 3 premières micro-étapes),
   avant de te soucier du reste.
3. **Teste une instruction à la fois** avant d'enchaîner un programme entier :
   place une seule instruction (`LDA`, par exemple) en mémoire et vérifie
   qu'elle produit exactement l'effet attendu.
4. **Compare toujours à une attente écrite à l'avance** (ta matrice de
   contrôle du chapitre 8, ton programme écrit à la main au chapitre 7) plutôt
   que de juger "à l'œil" si un résultat a l'air correct.

## 🛠️ Dans Digital Logic Sim

Crée un nouveau circuit "top-level" qui instancie toutes tes puces
(`RAM_16X8`, `PC_4BITS`, `ALU_8BITS`, tes registres, ton unité de contrôle) et
les relie selon l'architecture du sujet. Prévois un moyen simple de :
- pré-charger le contenu de la RAM avant de lancer la simulation (souvent
  possible en cliquant directement sur les bits de mémoire, selon les
  fonctionnalités de ta version de l'outil),
- avancer l'horloge manuellement, pas à pas,
- visualiser le contenu de chaque registre en continu (relie leurs sorties à
  des affichages, même si elles ne sont pas utilisées ailleurs).

## ✏️ Exercices

**Exercice 9.1 — Assemblage.** Relie toutes tes puces selon la spécification
du sujet du projet. Ne charge encore aucun programme : vérifie d'abord que le
circuit ne produit pas d'erreur de câblage évidente (bus en conflit, entrée
non connectée...).

**Exercice 9.2 — Test du fetch seul.** Charge n'importe quelle valeur en
mémoire à l'adresse `0000`, lance l'horloge pas à pas sur les 3 premières
micro-étapes, et vérifie que l'IR contient bien la valeur attendue et que le
PC est passé à `0001`.

**Exercice 9.3 — Une instruction à la fois.** Teste séparément `LDA`, `ADD`,
`SUB`, `OUT`, `HLT` avec des programmes d'une seule instruction utile (plus
`HLT`), en vérifiant chaque fois le contenu des registres après exécution.

**Exercice 9.4 — Le programme complet.** Charge le programme donné en exemple
dans le [sujet du projet](../sujet-projet.html) (calcul de `(5+3)-2`) et
vérifie que le registre de sortie affiche bien `6` à la fin de l'exécution.

**Exercice 9.5 — Ton propre programme.** Écris et teste un second programme
de ton choix (par exemple celui de l'exercice 7.3), sans aide.

## 🤔 Questions de réflexion

- Quel a été le bug le plus difficile à trouver dans ton assemblage ? Était-il
  dû à une erreur de câblage, à une erreur dans ta matrice de contrôle, ou à
  autre chose ?
- Maintenant que ton CPU fonctionne, saurais-tu expliquer, instruction par
  instruction, ce qui se passe électriquement dans chaque registre pendant
  l'exécution du programme de l'exercice 9.4 ?
- Quelle est, selon toi, la limite la plus contraignante de ce CPU par
  rapport à un processeur réel (fréquence d'horloge mise à part) ?

## 🚀 Pour aller plus loin

- Implémente l'extension bonus du sujet (`STA`, `LDI`, `JMP`, `JC`, `JZ`) si
  ce n'est pas déjà fait, et écris un programme avec une boucle.
- Ajoute un second registre à usage général en plus de l'accumulateur.
- Cherche comment les processeurs modernes dépassent les limites de cette
  architecture "un cycle = une micro-étape à la fois" grâce au **pipelining**
  (exécuter plusieurs instructions en parallèle, à des stades différents) —
  tu n'as pas besoin de l'implémenter, mais comprendre le principe te donnera
  une bonne intuition de l'architecture des CPU modernes.

---

Félicitations si tu es arrivé·e jusqu'ici avec un CPU qui fonctionne : tu as
littéralement reconstruit, brique par brique, ce que fait un microprocesseur.

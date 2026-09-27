---
layout: default
title: "12. Assemblage final"
parent: "Cours 2 : Noyau Linux"
nav_order: 12
---

# 12. Assemblage final de l'OS
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Réunir tous les sous-systèmes construits (chapitres 2 à 10) en un noyau
  cohérent.
- Vérifier que l'objectif final du [sujet du projet 2](../sujet-projet-os.html)
  est bien atteint.
- Documenter ton propre système, à la manière d'un vrai projet open-source.

## 📚 Théorie

### Ce chapitre n'introduit (presque) aucune notion nouvelle

Comme au premier cours, l'assemblage final consiste surtout à orchestrer,
dans le bon ordre, l'initialisation de tout ce que tu as déjà construit :

1. Le bootloader démarre ton noyau en mode long (chapitre 2).
2. Le noyau initialise l'affichage VGA (chapitre 3) — pour pouvoir déboguer
   visuellement la suite.
3. L'IDT, la GDT et la TSS sont configurées (chapitre 5).
4. Le PIC est remappé, puis les interruptions sont activées (chapitre 6).
5. La pagination est explorée et étendue si besoin (chapitre 7).
6. Le heap est réservé, mappé, et l'allocateur global enregistré (chapitre 8).
7. L'exécuteur de tâches est initialisé, et les tâches (clavier, minuterie,
   second pilote) sont lancées (chapitres 9 et 10).

L'ordre compte : certaines étapes dépendent explicitement des précédentes
(par exemple, le heap a besoin de la pagination fonctionnelle).

### Une méthode de débogage pour un système aussi composite

- **Ajoute des traces de démarrage** : un `println!` après chaque étape
  d'initialisation, pour savoir précisément où un blocage éventuel se
  produit.
- **Isole avant de recombiner** : en cas de comportement inattendu après
  assemblage, reviens tester le sous-système suspect isolément (grâce à tes
  tests du chapitre 4) avant de chercher une interaction entre sous-systèmes.
- **Vérifie l'ordre d'initialisation en premier** : une grande partie des bugs
  d'assemblage viennent d'une étape déclenchée trop tôt (par exemple, activer
  les interruptions avant d'avoir configuré l'IDT).

## 🛠️ En pratique (Rust / QEMU)

Regroupe toutes tes étapes d'initialisation dans une fonction unique, appelée
depuis le point d'entrée transmis par le bootloader, dans l'ordre décrit
ci-dessus.

## ✏️ Exercices

**Exercice 12.1 — Assemblage complet.** Écris la fonction d'initialisation
complète de ton noyau, dans l'ordre indiqué en théorie. Vérifie qu'elle
démarre sans erreur dans QEMU, avec des traces affichées à chaque étape.

**Exercice 12.2 — Démonstration de l'objectif final.** Reprends la liste des
7 points de l'"Objectif final" du
[sujet du projet 2](../sujet-projet-os.html), et vérifie-les un par un sur
ton assemblage complet : affichage, résistance à une exception, réaction aux
interruptions, mémoire virtuelle et heap fonctionnels, au moins 2 tâches
concurrentes, au moins 2 pilotes, tests automatisés passants.

**Exercice 12.3 — Documentation façon projet open-source.** Rédige un
`README.md` pour ton propre dépôt de projet (distinct de ce cours), qui
explique : ce que fait ton OS, comment le compiler et le lancer, son
architecture générale (tu peux t'inspirer du tableau de l'exercice 0.2), et
ses limites connues.

**Exercice 12.4 (bilan) — Comparaison finale avec Linux.** Reprends le
tableau de l'exercice 0.2 (sous-système Linux / équivalent dans ton noyau) et
complète-le une dernière fois avec, pour chaque ligne, ce que tu as
effectivement construit et ce qui resterait à faire pour se rapprocher
davantage d'un vrai Linux.

## 🤔 Questions de réflexion

- Quel a été le bug d'assemblage le plus révélateur pour toi — celui qui t'a
  forcé à mieux comprendre l'interaction entre deux sous-systèmes que tu
  pensais indépendants ?
- Maintenant que ton OS fonctionne de bout en bout, saurais-tu expliquer,
  chapitre par chapitre, à quoi correspond chaque ligne de trace affichée au
  démarrage ?
- Si tu devais choisir une seule extension du chapitre 11 à implémenter
  ensuite, laquelle choisirais-tu, et pourquoi ?

## 🚀 Pour aller plus loin

- Implémente les extensions restantes du [sujet du projet 2](../sujet-projet-os.html)
  (espace utilisateur, système de fichiers, ordonnancement préemptif).
- Fais tourner ton OS sur une vraie clé USB bootable, sur du matériel
  physique réel plutôt que dans QEMU (avec toutes les précautions d'usage :
  teste d'abord sur une machine que tu peux te permettre de redémarrer sans
  risque).
- Compare, avec le recul de ces deux cours, ce que ton noyau bare-metal en
  Rust et ton CPU logique du premier projet ont en commun dans leur démarche
  pédagogique : construire, brique par brique, en comprenant chaque niveau
  d'abstraction avant de l'utiliser comme une boîte noire au niveau suivant.

---

Félicitations si tu es arrivé·e jusqu'ici avec un noyau qui démarre, affiche,
gère ses interruptions, sa mémoire et au moins deux pilotes : tu as construit,
à ta propre échelle, la même pile de concepts que celle qui fait tourner
n'importe quel Linux, du plus petit objet connecté au plus gros serveur.

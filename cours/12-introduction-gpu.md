---
layout: default
title: "12. Introduction au GPU"
parent: Cours
nav_order: 12
---

# 12. Introduction au GPU
{: .no_toc }

1. TOC
{:toc}

## 🎯 Objectifs

- Comprendre pourquoi le GPU est architecturalement différent du CPU.
- Comprendre les notions de parallélisme SIMD et SIMT.
- Construire, à petite échelle, une mini-unité SIMD dans Digital Logic Sim
  pour ressentir concrètement le principe.

## 📚 Théorie

### Pourquoi un processeur différent ?

Le CPU (tous les chapitres précédents) est optimisé pour la **latence** :
exécuter une seule instruction, ou un petit nombre de flux d'instructions, le
plus vite possible, en investissant énormément de logique de contrôle
(pipeline, prédiction de branchement, exécution dans le désordre — chapitre
11) au bénéfice d'un tout petit nombre de tâches.

Le GPU (*Graphics Processing Unit*) est optimisé pour le **débit**
(*throughput*) sur des tâches massivement parallèles et régulières :
appliquer la même opération à des millions de pixels, de sommets 3D, ou —
usage aujourd'hui tout aussi central — à des millions de coefficients dans un
réseau de neurones.

### SIMD et SIMT

- **SIMD** (*Single Instruction, Multiple Data*) : une seule instruction
  s'applique simultanément à plusieurs données. Exemple : additionner 4
  paires de nombres en une seule opération matérielle, grâce à 4 ALU en
  parallèle recevant la même instruction mais des données différentes. Les
  CPU modernes possèdent aussi des unités SIMD (par exemple les extensions
  AVX sur x86), mais en nombre restreint.
- **SIMT** (*Single Instruction, Multiple Threads*), utilisé par les GPU :
  généralisation du SIMD où des milliers de "threads" légers exécutent le
  même programme (*kernel* / *shader*) sur des données différentes, regroupés
  en petits paquets (*warps* chez NVIDIA, *wavefronts* chez AMD) qui avancent
  ensemble, instruction par instruction.

### Beaucoup de cœurs simples plutôt que peu de cœurs complexes

Un GPU dédie l'essentiel de sa surface de silicium à un très grand nombre
d'unités de calcul (ALU) simples, avec relativement peu de logique de
contrôle par unité — à l'inverse du CPU, qui investit énormément de
transistors dans le contrôle au bénéfice d'un tout petit nombre de flux
d'exécution. C'est un compromis architectural fondamental : latence minimale
par tâche (CPU) contre débit maximal sur des tâches massivement parallèles
(GPU).

### Une mémoire pensée pour la bande passante

Les GPU privilégient une bande passante mémoire très élevée (pour nourrir en
données des milliers d'unités de calcul simultanément) plutôt qu'une latence
d'accès minimale (privilégiée par le CPU avec ses caches sophistiqués).

### Le modèle de programmation, en un mot

Un programme GPU (kernel CUDA, *compute shader*...) s'écrit du point de vue
d'un seul thread, exécutant une petite fonction sur "sa" donnée ; c'est le
matériel et le pilote qui se chargent de lancer ce même code sur des milliers
de threads en parallèle. Ce cours ne demande pas d'écrire de code GPU :
l'objectif est de comprendre le principe architectural, pas la
programmation.

## 🛠️ Dans Digital Logic Sim

Construire un GPU complet (des milliers d'unités de calcul) n'est pas
réaliste dans l'outil. En revanche, tu peux construire une **mini-unité
SIMD** pour ressentir le principe à petite échelle : plusieurs ALU
identiques, recevant le même signal d'opération, mais des données
différentes.

## ✏️ Exercices

**Exercice 12.1 — Mini unité SIMD à 2 voies.** En réutilisant deux instances
de ton `ALU_8BITS` (chapitre 3), construis un circuit avec **un seul** signal
de sélection d'opération (`SUB`) partagé par les deux ALU, mais 4 entrées de
données indépendantes (`A1`, `B1`, `A2`, `B2`) et deux sorties indépendantes
(`S1`, `S2`). Vérifie que les deux ALU effectuent bien la **même** opération
(toutes deux une addition, ou toutes deux une soustraction) au même instant,
mais sur des données différentes.

**Exercice 12.2 — Passage à 4 voies.** Généralise à 4 ALU en parallèle,
toujours avec un seul signal d'opération partagé. Combien de fils de données
au total ce circuit nécessite-t-il, en entrée et en sortie ?

**Exercice 12.3 (réflexion écrite, sans câblage) — Traiter plus de données
que d'unités.** Imagine que tu doives traiter un tableau de 100 paires de
nombres avec seulement les 4 ALU de l'exercice 12.2. Comment procéderais-tu,
à haut niveau ? Que faudrait-il ajouter pour distribuer les données au fil du
temps plutôt que toutes en même temps ?

## 🤔 Questions de réflexion

- Pourquoi le rendu d'une image (calculer la couleur de chaque pixel
  indépendamment des autres) est-il un cas d'usage particulièrement bien
  adapté à l'architecture SIMT ?
- Pourquoi l'entraînement de réseaux de neurones (des millions de
  multiplications-additions indépendantes) s'est-il révélé, historiquement,
  un excellent cas d'usage pour les GPU, alors qu'ils n'avaient pas été
  conçus initialement pour cela ?
- Un GPU peut-il remplacer entièrement un CPU dans un ordinateur ? Pourquoi
  la plupart des systèmes ont-ils besoin des deux ?
- Dans ton circuit de l'exercice 12.1, que se passerait-il si tu voulais que
  chaque ALU exécute une opération *différente* au même instant (l'une une
  addition, l'autre une soustraction) ? Est-ce encore du SIMD ? (Indice :
  cherche la différence avec MIMD — *Multiple Instruction, Multiple Data*.)

## 🚀 Pour aller plus loin

- Renseigne-toi sur la **divergence de branchement** (*branch divergence*)
  dans un GPU : que se passe-t-il, en termes de performance, quand différents
  threads d'un même warp/wavefront doivent emprunter des chemins différents
  (`if`/`else`) ? Pourquoi ce problème est-il spécifique à l'architecture
  SIMT et beaucoup moins présent sur CPU ?
- Compare l'architecture d'un GPU dédié (carte graphique séparée, mémoire
  propre) et d'un GPU intégré (partageant la mémoire avec le CPU), et
  réfléchis à pourquoi les consoles de jeu et de nombreux ordinateurs
  portables privilégient souvent cette seconde approche.

---

Avec ces trois derniers chapitres, tu as maintenant une vue d'ensemble
complète : de la porte NAND à un CPU fonctionnel, puis un aperçu de comment
ce même CPU se généralise vers de plus grandes largeurs, se compare aux
architectures industrielles, et diffère fondamentalement d'un GPU.

---
layout: default
title: "Projet 1 : CPU logique"
nav_order: 2
---

# Projet 1 — Sujet du projet : concevoir un CPU 8 bits
{: .no_toc }

1. TOC
{:toc}

## Contexte

Tu vas concevoir, dans **Digital Logic Sim**, un ordinateur 8 bits minimal mais
complet, capable de charger un petit programme en mémoire et de l'exécuter
instruction par instruction. L'architecture cible s'inspire du célèbre **SAP-1**
("Simple As Possible" computer), une référence classique en pédagogie de
l'architecture des ordinateurs.

Ce document est ton **cahier des charges** : il fixe ce que le CPU final doit
savoir faire, mais **ne dit jamais comment le construire**. Le "comment" est le
cours, chapitre par chapitre.

## Objectif final

Un CPU capable d'exécuter un programme comme celui-ci (calcul de `(5 + 3) - 2`,
résultat affiché en sortie) :

```
Adresse | Contenu       | Signification
--------|---------------|----------------------------------
0000    | LDA 1110      | ACC ← RAM[14]   (charge 5)
0001    | ADD 1111      | ACC ← ACC + RAM[15]  (+3)
0010    | SUB 1101      | ACC ← ACC − RAM[13]  (−2)
0011    | OUT           | affiche ACC
0100    | HLT           | arrêt
...
1101    | 00000010      | donnée : 2
1110    | 00000101      | donnée : 5
1111    | 00000011      | donnée : 3
```

Tu n'as pas besoin de comprendre ce tableau dès maintenant — le
[chapitre 7](cours/07-jeu-instructions.html) t'apprend à l'écrire toi-même.

## Spécification de l'architecture cible

Cette spécification est **la cible obligatoire**, pas une solution : elle définit
"quoi" construire, pas "comment" le câbler.

### Composants requis

| Composant | Rôle |
|---|---|
| Bus de données | 8 bits, partagé par tous les composants |
| RAM | 16 mots de 8 bits (adresses sur 4 bits) |
| Compteur ordinal (PC) | 4 bits, pointe la prochaine instruction |
| Registre d'adresse mémoire (MAR) | 4 bits, adresse actuellement accédée en RAM |
| Registre d'instruction (IR) | 8 bits, contient l'instruction en cours |
| Accumulateur (ACC) | 8 bits, registre de calcul principal |
| Registre B | 8 bits, second opérande de l'ALU |
| ALU | Addition et soustraction 8 bits, avec drapeaux carry / zero |
| Registre de sortie (OUT) | 8 bits, affiche le résultat |
| Unité de contrôle | Séquence les micro-étapes de chaque instruction |
| Horloge | Génère le signal cadençant tout le système |

### Jeu d'instructions minimal obligatoire

Format d'instruction : 8 bits = 4 bits d'opcode + 4 bits d'opérande (adresse ou inutilisé).

| Opcode (bin) | Mnémonique | Effet |
|---|---|---|
| `0000` | `LDA addr` | `ACC ← RAM[addr]` |
| `0001` | `ADD addr` | `ACC ← ACC + RAM[addr]` |
| `0010` | `SUB addr` | `ACC ← ACC − RAM[addr]` |
| `1110` | `OUT` | `OUT ← ACC` |
| `1111` | `HLT` | arrête l'horloge |

### Extension bonus (facultative, "pour aller plus loin")

Une fois le CPU minimal fonctionnel, tu peux enrichir le jeu d'instructions :

| Opcode (bin) | Mnémonique | Effet |
|---|---|---|
| `0011` | `STA addr` | `RAM[addr] ← ACC` |
| `0100` | `LDI val` | `ACC ← val` (valeur immédiate, pas une adresse) |
| `0101` | `JMP addr` | `PC ← addr` |
| `0110` | `JC addr` | si drapeau carry actif : `PC ← addr` |
| `0111` | `JZ addr` | si drapeau zero actif : `PC ← addr` |

Avec `JC`/`JZ`/`JMP`, ton CPU devient capable de faire des boucles — essaie par
exemple d'écrire un programme qui compte de 0 à 10.

## Extensions avancées (hors CPU minimal, chapitres 10 à 12)

Une fois le CPU 8 bits du chapitre 9 fonctionnel, le cours propose trois
prolongements théoriques et pratiques, non nécessaires pour valider le
projet minimal, mais qui replacent ton travail dans le contexte des
architectures réelles :

- **Passage à 64 bits** ([chapitre 10](cours/10-passage-64-bits.html)) :
  généraliser tes circuits par composition (8→16 bits en pratique dans Digital
  Logic Sim) et comprendre pourquoi un vrai saut à 32 ou 64 bits demande des
  techniques matérielles différentes (additionneurs à retenue anticipée,
  hiérarchie mémoire).
- **Comparaison des architectures CPU** ([chapitre 11](cours/11-architectures-cpu.html)) :
  situer ton CPU maison par rapport aux familles CISC (x86) et RISC (ARM,
  RISC-V), et comprendre pipeline, superscalaire, exécution dans le désordre,
  multicœur.
- **Introduction au GPU** ([chapitre 12](cours/12-introduction-gpu.html)) :
  comprendre en quoi l'architecture d'un GPU diffère fondamentalement de
  celle d'un CPU (SIMD/SIMT), avec une mini-unité SIMD à construire dans
  Digital Logic Sim.

## Jalons du projet

Chaque jalon correspond à un chapitre du cours et produit une ou plusieurs
"puces" (chips) réutilisables dans Digital Logic Sim.

| Jalon | Chapitre correspondant | Livrable attendu |
|---|---|---|
| J0 | Prise en main | Environnement Digital Logic Sim opérationnel |
| J1 | Algèbre de Boole | Bibliothèque de portes de base construites depuis NAND |
| J2 | Circuits combinatoires | Additionneur 8 bits, décodeur, multiplexeur |
| J3 | Arithmétique / ALU | ALU 8 bits avec drapeaux zero / carry |
| J4 | Bascules / mémoire | Bascule D, registre 8 bits à chargement (*load*) |
| J5 | Registres / bus | Bus 8 bits partagé, plusieurs registres dessus |
| J6 | Von Neumann | RAM 16×8 adressable, PC avec incrémentation |
| J7 | Jeu d'instructions | Table d'opcodes rédigée + programmes tests écrits à la main |
| J8 | Unité de contrôle | Séquenceur générant les signaux de commande |
| J9 | Assemblage final | CPU complet exécutant le programme de la section "Objectif final" |
| J10 *(extension)* | Passage à 64 bits | `ADD_16BITS`, registre/bus 16 bits, esquisse écrite de la RAM hiérarchique |
| J11 *(extension)* | Comparaison d'architectures | Classement de ton CPU + comparaison écrite avec un CPU réel CISC et RISC |
| J12 *(extension)* | Introduction au GPU | Mini-unité SIMD à 2 puis 4 voies |

## Livrables

Pour chaque jalon, garde une trace (dans un dossier local ou un dépôt Git séparé
de ce cours) :

- Le fichier de sauvegarde Digital Logic Sim de la puce (`.chip` / dossier du
  projet, selon la version utilisée).
- Une capture d'écran du circuit et, idéalement, une capture montrant un test
  qui fonctionne (ou qui a échoué, si tu documentes un bug corrigé).
- Un court paragraphe expliquant un choix de conception ou une difficulté
  rencontrée — c'est souvent plus révélateur de ta compréhension que le circuit
  lui-même.

## Grille d'auto-évaluation

Comme il n'y a pas de corrigé, utilise cette grille pour savoir si un jalon est
réellement acquis :

- [ ] Le circuit respecte la spécification pour **toutes** les entrées possibles
      (pas juste les cas que j'ai testés à la main).
- [ ] Je peux expliquer à voix haute, sans notes, pourquoi le circuit fonctionne.
- [ ] Je pourrais reconstruire ce circuit sans regarder le précédent.
- [ ] Le circuit est encapsulé en une puce réutilisable, avec des entrées/sorties
      nommées clairement.
- [ ] J'ai testé au moins un cas limite (ex : dépassement de capacité, adresse
      0, entrées toutes à 0 ou toutes à 1).

## Ce que ce sujet ne te donne pas (volontairement)

- Le câblage interne d'aucun circuit.
- Le détail des micro-étapes de l'unité de contrôle.
- Le code des signaux de commande.

C'est précisément le travail à faire. Le cours te donne les outils théoriques
pour y arriver seul·e.

# Architecture des ordinateurs — de la porte logique au noyau

Ce dépôt contient **deux cours + deux projets fil rouge** pour apprendre
l'architecture des ordinateurs "bas niveau" :

1. **Parcours 1** : construire un **CPU 8 bits** dans
   [Digital Logic Sim](https://github.com/SebLague/Digital-Logic-Sim), le
   simulateur de logique numérique de Sebastian Lague.
2. **Parcours 2** : construire un **OS minimal en Rust** (bootloader, noyau,
   pilotes), avec l'architecture du noyau **Linux** comme grille de lecture
   à chaque chapitre.

⚠️ **Ce site ne contient volontairement aucune solution.** Il donne le cours,
les spécifications à respecter et des exercices — libre à toi (ou à l'élève)
de concevoir, tester et déboguer les circuits ou le code toi-même.

## Structure du dépôt

```
.
├── _config.yml            # config Jekyll / thème just-the-docs
├── index.md               # page d'accueil (les deux parcours)
├── sujet-projet.md         # cahier des charges — projet 1 : CPU logique
├── sujet-projet-os.md      # cahier des charges — projet 2 : OS en Rust
├── cours/                  # cours 1 : de la porte logique au CPU (13 chapitres)
│   ├── index.md
│   └── 00-...  à  12-...
├── noyau-linux/            # cours 2 : architecture du noyau Linux (13 chapitres)
│   ├── index.md
│   └── 00-...  à  12-...
└── ressources/
    ├── index.md
    ├── glossaire.md            # vocabulaire — parcours 1
    ├── liens-utiles.md         # liens — parcours 1 (Digital Logic Sim)
    ├── glossaire-os.md         # vocabulaire — parcours 2
    └── liens-utiles-os.md      # liens — parcours 2 (Rust bare-metal, crates, QEMU)
```

## Déployer sur GitHub Pages

1. Crée un nouveau dépôt GitHub et pousse le contenu de ce dossier à la racine
   (ou dans `/docs` si tu préfères, en adaptant les réglages Pages).
2. Dans **Settings → Pages**, choisis *Deploy from a branch*, branche `main`,
   dossier `/ (root)`.
3. Édite `_config.yml` : renseigne `url:` avec ton URL GitHub Pages
   (`https://<ton-pseudo>.github.io`) et `baseurl:` si le site n'est pas à la racine
   d'un dépôt `<pseudo>.github.io` (ex: `/cours-archi-ordinateur`).
4. Attends quelques minutes que GitHub Pages construise le site (l'action se
   voit dans l'onglet **Actions** ou **Settings → Pages**).

Le thème utilisé est [`just-the-docs`](https://github.com/just-the-docs/just-the-docs),
chargé via `remote_theme`, ce qui fonctionne nativement avec le build Pages classique
(pas besoin de GitHub Actions personnalisée).

## Tester en local

```bash
bundle install
bundle exec jekyll serve
```

Le site sera accessible sur `http://localhost:4000`.

## Outils requis pour les projets (pas pour le site lui-même)

- **Parcours 1** : [Digital Logic Sim](https://github.com/SebLague/Digital-Logic-Sim/releases)
  — simulateur de circuits logiques gratuit et open-source.
- **Parcours 2** : une chaîne Rust `nightly` (via `rustup`) et
  [QEMU](https://www.qemu.org/download/). Voir `ressources/liens-utiles-os.md`
  pour la liste des crates recommandées.

Le code des deux projets (circuits Digital Logic Sim, dépôt Git du noyau
Rust) est censé vivre **en dehors** de ce dépôt de cours — celui-ci ne
contient que le support pédagogique.

## Licence / usage

Contenu pédagogique libre d'usage et de modification pour un usage personnel ou
associatif d'apprentissage. Digital Logic Sim et le noyau Linux sont des
projets tiers, non affiliés à ce cours.

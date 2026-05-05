# Le Grimoire Éducatif

Une collection de petits jeux web pour apprendre en s'amusant. Tout est local, sans build, sans dépendance — double-clic sur `index.html` et ça marche.

## Lancer

Ouvre `index.html` à la racine : c'est la page de garde, qui mène à chaque jeu.

> Garde la structure des dossiers intacte (`assets/` à côté de chaque `index.html` qui en a besoin), sinon les sprites/sons ne s'afficheront pas.

## Jeux disponibles

### ⚔️ [Le Donjon des Multiplications](donjon-multiplication/)
Traverse un donjon en répondant aux multiplications (tables 1-12). Combats, coffres piégés, sanctuaires, mode hardcore avec timer 10s. Score et leaderboard local.

### 🧙‍♂️ [Le Sorcier de l'Imparfait](sorcier-imparfait/)
Quizz pour réviser l'imparfait sur 6 verbes (être, avoir, aller, habiter, fêter, couper). Deux modes : QCM facile ou écriture de la terminaison.

### 🏴‍☠️ [Les Lettres du Capitaine Crochet-Mou](pirate-accords/)
Chasse aux fautes d'accord (CE2) sur les lettres d'un vieux capitaine pirate. Couvre l'accord nom/adjectif, les pluriels en -s/-aux/-eaux/-oux, les déterminants `ce/cette/ces` et l'accord sujet-verbe.

## Structure

```
.
├── index.html                  ← page de garde (hub)
├── donjon-multiplication/
│   ├── index.html
│   └── assets/
├── sorcier-imparfait/
│   └── index.html
└── pirate-accords/
    └── index.html
```

Chaque jeu est autonome dans son sous-dossier et reste un projet single-file.

## Ajouter un nouveau jeu

1. Créer un sous-dossier `<nom-du-jeu>/` à la racine.
2. Y mettre un `index.html` autonome (et son `assets/` si besoin).
3. Ajouter un `<a class="card …">` dans la grille de la page de garde (`index.html`).
4. Ajouter une entrée dans la liste « Jeux disponibles » de ce README.

## Conventions

- **Tout en français** (UI, code, commentaires).
- **Pas d'outillage** : pas de bundler, pas de npm, pas de tests.
- **Single-file par jeu** : tout dans son `index.html`.
- **Chemins relatifs uniquement** pour que double-clic local et hébergement statique marchent pareil.

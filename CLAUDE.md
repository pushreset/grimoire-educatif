# CLAUDE.md

**Le Grimoire Éducatif** — collection de petits jeux web éducatifs en français, ambiance kid-friendly. Pas de build, pas de dépendance, pas de serveur. Double-clic sur `index.html` à la racine.

## Structure

```
.
├── index.html                  ← page de garde (hub neutre, grille de cartes)
├── donjon-multiplication/      ← jeu 1
│   ├── index.html
│   └── assets/                 ← sprites + audio (chemins relatifs depuis index.html)
├── sorcier-imparfait/          ← jeu 2
│   └── index.html              ← entièrement autonome, pas d'assets
├── pirate-accords/             ← jeu 3
│   └── index.html
└── battle-royale-passe-compose/ ← jeu 4
    └── index.html
```

Chaque jeu est **autonome** dans son sous-dossier (single-file `index.html` + son `assets/` si besoin). Les jeux ne partagent rien entre eux à part le lien retour vers la page de garde.

## Conventions globales

- **Tout reste single-file** par jeu. Pas de séparation HTML/CSS/JS sans demande explicite — c'est assumé (cible : enfant qui double-clique).
- **Pas d'outillage** (pas de bundler, pas de npm, pas de tests). N'introduis rien qui demande `node`/`npm install`.
- **Textes en français** uniquement (UI, commentaires de code, messages).
- **Chemins relatifs uniquement** : tout doit marcher en double-clic local ET en hébergement statique (GitHub Pages).
- **Chaque jeu doit avoir un lien retour** vers `../index.html` (page de garde).

## Ajouter un nouveau jeu

1. Créer un sous-dossier `<nom-du-jeu>/` à la racine.
2. Y mettre un `index.html` autonome (+ `assets/` si besoin).
3. Ajouter un `<a class="card …">` dans la grille de la page de garde (`index.html`), avec une couleur d'accent dédiée (`--card-accent` / `--card-accent-dark` / `--card-tint`).
4. Ajouter une entrée dans la liste « Jeux disponibles » du `README.md`.
5. Inclure dans le jeu un lien retour vers `../index.html`.

## Page de garde (`index.html` racine)

Thème **neutre** (bleu-violet sombre + cartes ivoire) pour ne pas imposer un style aux jeux. Chaque carte a sa couleur d'accent qui hint le thème de l'app derrière. Single-file, ~150 lignes.

---

## Jeu 1 — `donjon-multiplication/`

Entraînement aux tables de multiplication 4-9 dans une ambiance roguelike pixel art.

### Architecture

Tout dans `donjon-multiplication/index.html` :
- ~640 lignes de CSS
- ~840 lignes de JS dans une IIFE en bas du fichier (`index.html:680+`)

Modèle :
- **State global mutable** dans `state`.
- **Render full-replace** : chaque transition d'écran fait `root.innerHTML = …` puis `attachListeners()`. Pas de diff, pas de framework.
- **6 écrans** dispatchés par `state.screen` : `setup`, `combat`, `chest`, `choice`, `shrine`, `gameover`.
- **Sprites animés** : spritesheets horizontales scannées par une boucle `requestAnimationFrame` qui décale `translateX` toutes les `FRAME_DURATION` ms. Mode `Idle` boucle, modes `Attack/Hurt/Death` sont "once" et figent sur la dernière frame.

Hors `#game-content` (et donc préservés entre re-renders) : `<audio id="bgm">`, `#mute-btn`, `#home-btn`, les torches, et les `.toast` (insérés dans `#game`).

### Assets

Convention de chemin imposée par le pack d'origine :
```
assets/tiny_rpg_characters_set/Characters/{Name}/{Name}/{Name}-{Anim}.png
```
Le double dossier `{Name}/{Name}/` n'est pas une faute de frappe. Voir `SPRITE_BASE` et `spriteHTML()`.

Frames par animation déclarés en dur dans `SPRITE_DATA`. Si tu ajoutes un personnage, il **faut** y ajouter une entrée — sinon `spriteHTML` retourne `''` silencieusement.

Audio : `assets/audio/{music,sword,hurt,death,magic}.mp3`. `sfx()` crée un nouvel `Audio()` à chaque appel.

### Conventions spécifiques au donjon

- **Difficulté pilotée par `state.player.rooms`** dans `pickMonster()` — toute modification du rythme se fait là.
- **Verrouillage clavier/clic** via `state.locked` pendant les animations. Toute nouvelle action interactive doit le respecter et le relâcher dans son `setTimeout` final.

---

## Jeu 2 — `sorcier-imparfait/`

Quizz de conjugaison à l'imparfait sur 6 verbes (être, avoir, aller, habiter, fêter, couper).

### Architecture

Single-file `sorcier-imparfait/index.html`, autonome, pas d'assets.
- Données : `verbs` (formes conjuguées), `stems` (radicaux), `sentences` (phrases à trous).
- 3 écrans pilotés par `classList.toggle('hidden')` : `start-screen`, `game-screen`, `end-screen`.
- 2 modes : `qcm` (3 boutons) et `write` (input texte avec radical pré-affiché, on tape juste la terminaison).
- 10 questions tirées au hasard parmi `sentences` à chaque partie.

---

## Jeu 3 — `pirate-accords/`

Chasse aux fautes d'accord (genre/nombre) niveau CE2 dans les lettres du Capitaine Crochet-Mou.

### Architecture

Single-file `pirate-accords/index.html`, autonome, pas d'assets. Ambiance parchemin/encre brune.
- Données : `phrases` (30 entrées avec `correct`, `faulty`, `faultIndex`, `rule`, `category`), `intros`, `signatures`.
- 3 écrans : `start-screen`, `game-screen`, `end-screen`.
- À chaque partie, une lettre est assemblée : 1 intro + 5 phrases tirées + 1 signature. Pour chaque phrase, ~65% de proba d'utiliser la version `faulty`. Au moins une faute injectée par lettre (filet de sécurité).
- Tokenisation : `split(' ')` simple. La ponctuation collée fait partie du token. Chaque mot devient un `<span class="word">` cliquable.
- 8 catégories d'erreurs couvertes : `pluriel-nom`, `pluriel-adj`, `pluriel-aux`, `pluriel-eaux`, `pluriel-oux`, `feminin-adj`, `determinant`, `sujet-verbe`.
- Validation au chargement : une IIFE vérifie que `faulty.split(' ')[faultIndex]` diffère bien de `correct.split(' ')[faultIndex]` et que le reste est strictement identique. Logge des warnings sinon.
- 3 vies, fin auto si `lives === 0` ou si toutes les fautes sont trouvées.

### Convention pour ajouter une phrase

`faulty` doit différer de `correct` **uniquement à `faultIndex`** (split par `' '`). Vérifie à la main avant de commit ; le check au chargement attrape le reste.

---

## Jeu 4 — `battle-royale-passe-compose/`

Le passé composé niveau CM1 (leçons C4 « avec avoir » et C5 « avec être » du manuel),
habillé en battle royale. Ambiance nuit + néon, aucun asset de marque, tout en CSS et emoji.

### Architecture

Single-file `battle-royale-passe-compose/index.html`, autonome, pas d'assets.
- 4 écrans pilotés par `montrer(id)` qui bascule la classe `.cache` : `ecran-depart`,
  `ecran-jeu`, `ecran-loot`, `ecran-fin`.
- État global mutable dans `etat` (recréé à chaque partie par `demarrer()`).

### Modèle de données — un seul endroit à toucher

**`phrases`** alimente les zones 1, 3 et 4. Une entrée :

```js
{ b: "Nova ", a: " du bus la dernière.", inf: 'sortir',
  aux: 'être', pp: 'sorti', agr: 'e', pn: 2, indice: "la dernière" }
```

- `b` début de phrase, **se termine par une espace ou par une apostrophe** (`"J'"`, `"Elle s'"`).
  Rien n'ajoute d'espace : l'auxiliaire est collé tel quel.
- `pp` participe passé au **masculin singulier**, `agr` le suffixe d'accord
  (`''` / `'e'` / `'s'` / `'es'`), appliqué **uniquement** si `aux === 'être'`.
- `pn` la personne, 0..5 (je, tu, il/elle, nous, vous, ils/elles). Sert à piocher
  l'auxiliaire dans `AUX` et à vérifier la cohérence singulier/pluriel de `agr`.
- Un verbe pronominal a un `inf` qui commence par `se ` et met son pronom dans `b`.
- `indice` **obligatoire dès que `aux === 'être'`** : le morceau de texte qui rend le genre
  trouvable. *je*, *tu*, *nous*, *vous*, *on* ne disent rien du genre — une phrase qui
  commence par un de ces pronoms sans contexte donne une question **sans réponse trouvable**.
  D'où le procédé du manuel : « Moi, Léo, je… », « Toi, Julie, tu… »,
  « Mon frère et moi, nous… ». L'indice est cité dans l'explication affichée à l'enfant.
  La vérification au chargement exige qu'il soit une sous-chaîne réelle de `b + a`.
  Attention au piège : « Mes sœurs et moi » ne donne pas le genre (le locuteur est inconnu),
  alors que « Mon frère et moi » le donne (au moins un garçon → masculin).
- `note` facultatif : une phrase de plus dans l'explication, pour les cas qui méritent
  un mot (groupe mixte, accord porté par l'attribut).

**`verbes`** alimente la zone 2 (participe passé). `faux` est **obligatoire pour le 3e groupe** ;
pour les 1er et 2e groupes il est généré automatiquement juste après la déclaration
(la fausse forme la plus utile étant l'infinitif : `sauter` au lieu de `sauté`).

### Les 4 zones

| Zone | Nom | Ce qu'elle teste | Options |
|---|---|---|---|
| 1 | LARGAGE | choisir l'auxiliaire | 2 |
| 2 | PILLAGE | former le participe passé | 3 |
| 3 | TEMPÊTE | accorder le participe passé avec *être* | 4 |
| 4 | TOP 1 | forme verbale complète, tout mélangé | 4 |

`QUESTIONS_PAR_ZONE = 6`. Les zones 1 et 4 tirent moitié *avoir* / moitié *être* pour que
l'enfant ne puisse pas répondre au hasard sur une régularité statistique.

Progression et record dans `localStorage` (`grimoire-battle-royale-pc`), toujours en `try/catch`
comme dans le donjon : si le stockage est bloqué, le jeu tourne quand même.

### Conventions spécifiques

- **Le « trou » de la phrase affiche toujours la bonne réponse en vert**, même après une erreur.
  L'erreur est signalée par le bouton rouge et l'encadré d'explication, jamais par le mot correct
  affiché en rouge (sinon l'enfant mémorise la bonne forme comme fausse).
- **Chaque retour explique la règle**, pas seulement « faux ». Les explications sont dans
  `questionZoneN()`, pas dans les données.
- `etat.verrou` bloque les clics et le chrono pendant l'affichage du retour. Toute nouvelle
  interaction doit le respecter.
- Une IIFE de vérification tourne au chargement : auxiliaire connu, `pn` dans 0..5, accord valide,
  cohérence accord/personne, verbe en *être* présent dans `VERBES_ETRE` (la liste du manuel),
  **présence et validité de `indice`**, pas de doublon dans `faux`.
  Elle logge des warnings, elle ne casse rien.

### Si tu ajoutes du contenu

Ajoute des entrées dans `phrases` / `verbes`, rien d'autre. Ouvre la console une fois :
zéro warning = les données sont cohérentes.

# CLAUDE.md

**Le Grimoire Éducatif** — collection de petits jeux web éducatifs en français, ambiance kid-friendly. Pas de build, pas de dépendance, pas de serveur. Double-clic sur `index.html` à la racine.

## Structure

```
.
├── index.html                  ← page de garde (hub neutre, grille de cartes)
├── donjon-multiplication/      ← jeu 1
│   ├── index.html
│   └── assets/                 ← sprites + audio (chemins relatifs depuis index.html)
└── sorcier-imparfait/          ← jeu 2
    └── index.html              ← entièrement autonome, pas d'assets
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

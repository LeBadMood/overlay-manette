# Passation : overlay manette Xbox « Scream / Ghostface »

Dernière mise à jour : 3 novembre 2026.

## 1. Objectif

Créer un overlay de manette **Xbox** pour stream (OBS), à partir du thème PS5 de
https://github.com/ScReaMCOD/Overlay-manette-ps5 :

- convertir `scream_ghostface_gpv_safe.css` (thème Gamepad Viewer) en version Xbox ;
- utiliser l'image `ghostface_raiju_shell.png` (1536 × 1024, PNG transparent) comme style de la manette ;
- **demande précise de l'utilisateur** : le rouge autour des boutons doit être « éteint » (peu visible)
  au repos, et s'allumer comme sur le PNG quand on appuie.

## 2. Où sont les fichiers

| Emplacement | Contenu | État |
|---|---|---|
| **https://github.com/LeBadMood/overlay-manette** (public, branche `main`) | Version de référence, à utiliser | À jour |
| `LeBadMood/BadShade` (privé), branche `claude/busy-hypatia-nf3y2r`, dossier `overlay-xbox/` | Première version | Ancienne : image pointée vers le dépôt ScReaMCOD, pas de mode OBS dans `preview.html`. Dépôt privé, donc inutilisable par jsDelivr. |

Fichiers du dépôt `overlay-manette` :

- `scream_ghostface_xbox.css` : le thème Gamepad Viewer (skin Xbox One).
- `ghostface_raiju_shell.png` : l'image de la manette (copie de celle de ScReaMCOD).
- `preview.html` : page autonome qui reprend le HTML de Gamepad Viewer et lit la manette avec l'API Gamepad
  du navigateur (repli clavier). `?obs` = fond transparent, sans textes.
- `README.md` : mode d'emploi en français.
- `apercu_repos.png`, `apercu_appui.png` : captures (au repos / tout appuyé).

## 3. Fonctionnement du CSS

- La manette fait **768 × 512 px** (l'image à 50 %). `.controller.xbox` est centrée de force dans la page :
  `position: fixed; left/top: 50%; margin: -256px 0 0 -384px; transform: none` (tout en `!important`).
- Les groupes `.triggers`, `.bumpers`, `.arrows`, `.abxy`, `.sticks`, `.dpad` sont des calques pleine taille
  (0,0 → 768 × 512). Chaque élément est donc placé en coordonnées de l'image, et sa tranche d'image se cale
  avec `background-position: -left -top` et `background-size: 768px 512px`.
- Variables en tête de fichier :
  - `--shell` : URL de l'image (actuellement `raw.githubusercontent.com/LeBadMood/overlay-manette/main/...`).
  - `--off` : filtre « lumière éteinte » (`saturate(0.12) brightness(0.72)`).
  - `--lit` : filtre « allumé » (luminosité + halo rouge en `drop-shadow`).
- **Éteint / allumé** : une tranche de l'image, avec un masque radial pour fondre les bords, recouvre chaque
  zone lumineuse avec le filtre `--off`. À l'appui (classe `.pressed`), la tranche repasse en `--lit`.
  - A B X Y : l'élément `.button` lui-même (80 × 80, centré sur le bouton et son halo).
  - Croix : `.dpad::before` éteint toute la croix ; chaque `.face` allumée (opacité 0 → 1) éclaire sa direction.
  - LT/RT : `.triggers::before/::after` éteints ; `.trigger` allumé, dont Gamepad Viewer pilote l'opacité
    (ou le `clip-path` en mode « meter ») selon la pression analogique. Les triggers sont en `z-index: 1`.
  - Anneaux des sticks : `.bumpers::before/::after` éteints ; ils s'allument au clic du stick via
    `:has(.stick.left.pressed)`, ce qui demande un Chromium récent (OBS 31+).
  - LB/RB : tranche de l'épaule qui s'embrase (opacité 0 → 1).
  - Tête des sticks : tranche de l'image qui suit le stick. Gamepad Viewer la déplace avec
    `margin-top/left` + `rotateX/Y`, donc ne jamais mettre de `transform` ou de `margin` sur `.stick`.
    `.sticks::before/::after` posent un puits sombre qui cache la tête imprimée sous la tête mobile.
  - View/Menu/Guide : flash blanc + rouge à l'appui. Le logo Xbox reste allumé (`.quadrant` masqué).

Coordonnées principales (repère 768 × 512, centres) : Y (605,118), X (552,164), B (652,164), A (604,214),
stick gauche (200,147), stick droit (496,261), croix (303,273), View (313,155), Menu (452,156),
logo Xbox (383,97), LT x 142–254 / RT x 518–630 (y 0–50).

## 4. Hypothèses sur Gamepad Viewer (non vérifiées)

gamepadviewer.com n'était pas joignable depuis la session. La structure HTML et le CSS par défaut viennent de
copies sur GitHub (`dracula/gamepad-viewer` `xbox/edit.css`, `heddaz/shoubing` `xbox/index.css`,
`skyjty/MyGamepadViewer` `js/Gamepad.js`) :

- HTML Xbox : `.triggers > .trigger.left/.right`, `.bumpers > .bumper.left/.right`, `.quadrant`,
  `.arrows > .back/.start`, `.abxy > .button.a/.b/.x/.y`, `.sticks > .stick.left/.right`,
  `.dpad > .face.up/.down/.left/.right`. Classe `.pressed` à l'appui, `.disconnected` sans manette.
- CSS personnalisé chargé via `?p=1&css=<URL>`. L'URL doit renvoyer `text/css` : jsDelivr oui,
  raw.githubusercontent non (les navigateurs refusent une feuille de style `text/plain`).
- Quelques versions de Gamepad Viewer déplacent le stick droit avec `margin-bottom/right` au lieu de
  `margin-top/left`. Si le stick droit ne bouge pas, c'est la cause probable.

## 5. Problème en cours : écran blanc sur Gamepad Viewer

L'utilisateur voit un **écran blanc** avec ce lien :

https://gamepadviewer.com/?p=1&css=https%3A%2F%2Fcdn.jsdelivr.net%2Fgh%2FLeBadMood%2Foverlay-manette%403aaa6ed03728726c127cc6d6d226b401736ab71d%2Fscream_ghostface_xbox.css

Historique :
1. Premier essai : CSS hébergé sur BadShade. jsDelivr répondait « Couldn't find the requested file »
   parce que le dépôt est **privé**. D'où la création du dépôt public `overlay-manette`.
2. Le centrage a été forcé (`position: fixed`), au cas où la manette sortait de l'écran.
3. Toujours blanc avec le dépôt public. Cause encore inconnue.

Pistes, de la plus probable à la moins probable :
- **Manette pas encore détectée** : Chrome n'expose la manette qu'après un appui sur un bouton, et
  Gamepad Viewer peut rester vide jusque-là. À tester : `https://gamepadviewer.com/?p=1` **sans** CSS.
- **CSS non chargé** : à tester en ouvrant directement l'URL jsDelivr ci-dessus.
- **Mauvais skin** : paramètre de skin (`s=`) inconnu. Dans ce cas on verrait une autre manette, pas du blanc.
- **Image bloquée** : `raw.githubusercontent.com` pour `--shell`. Peu probable, car les images ne sont pas
  soumises au contrôle de type MIME.

Questions posées à l'utilisateur, **sans réponse pour l'instant** :
1. `https://gamepadviewer.com/?p=1` sans CSS : blanc aussi après un appui sur un bouton ?
2. Que montre l'URL jsDelivr du CSS ?

Ouvrir la console du navigateur (F12) sur la page blanche donnerait la réponse la plus rapide.

## 6. Solution de repli (sans Gamepad Viewer)

`preview.html` fonctionne seul. Testé en local avec Playwright : aucune erreur, A s'allume.
Les liens en ligne passent par raw.githack.com et n'ont **pas** été testés :

- navigateur : https://raw.githack.com/LeBadMood/overlay-manette/main/preview.html
- OBS (fond transparent, 800 × 560) : https://raw.githack.com/LeBadMood/overlay-manette/main/preview.html?obs

Alternative plus propre : activer **GitHub Pages** sur `overlay-manette` (Settings → Pages → branche `main`),
puis utiliser `https://lebadmood.github.io/overlay-manette/preview.html`.

## 7. Contraintes de l'environnement

- Réseau de la session cloud : **gamepadviewer.com, cdn.jsdelivr.net et raw.githack.com bloqués**. Pour tester,
  l'utilisateur doit les ajouter dans *Allowed domains* (environnement → Edit → Network access).
- raw.githubusercontent.com et github.com (git) passent.
- Pas de Claude in Chrome dans la session cloud : le navigateur de l'utilisateur n'est pas relié.
- La session n'a pas le droit de créer de dépôt : l'utilisateur a créé `overlay-manette` lui-même.
- Tester en local : Playwright + Chromium sont préinstallés. Ouvrir `preview.html` en `file://`, injecter le CSS
  Xbox par défaut de Gamepad Viewer avant le thème pour vérifier les priorités CSS, puis simuler
  `.pressed` et les marges des sticks.

## 8. Prochaines étapes

1. Obtenir les réponses aux deux questions de la section 5, ou une capture de la console (F12).
2. Si le CSS ne se charge pas : vérifier le type MIME servi, ou essayer GitHub Pages pour héberger le CSS.
3. Si Gamepad Viewer reste blanc même sans CSS : problème de détection de la manette (navigateur, câble ou
   Bluetooth). Utiliser `preview.html` à la place.
4. Une fois l'overlay visible : vérifier l'alignement des boutons sur le vrai Gamepad Viewer et ajuster les
   coordonnées de la section 3 si besoin.
5. Option : synchroniser ou supprimer la vieille copie `overlay-xbox/` de BadShade.

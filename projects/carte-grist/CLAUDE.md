# Projet : Carte Grist — Atlas (Maquette 3D Territoriale)

## Contexte

Refonte UX/UI complète du widget « Maquette 3D Territoriale » pour Grist,
issue du design canvas Claude Design (`carte-grist`, direction **Atlas**).

Cible : élus / présentations publiques, mais la tâche n°1 reste **construire**
la maquette (import de données, édition, symbolisation, placement de modèles 3D).

Différence majeure avec l'ancien widget : **Mapbox a été remplacé par MapLibre
GL JS**, et les modèles 3D GLTF (qui s'appuyaient sur la couche `model` native
de Mapbox Standard) sont désormais rendus via une **custom layer three.js**.

## Architecture des fichiers

```
carte-grist/
├── index.html   # Chrome Atlas (HTML/CSS) + structure du DOM
├── app.js       # Toute la logique (ES module)
└── CLAUDE.md
```

- `index.html` : design **Atlas** (papier `#F4EFE3`, serif Newsreader + Hanken
  Grotesk, accent terre cuite `#C44536`). Header + rail d'icônes (6 modules) +
  panneau de module contextuel (gauche) + carte + inspecteur (droite) +
  overlays carte (boussole, HUD, légende, bandeau solaire, barre de sélection).
- `app.js` (module, importe `three` + `GLTFLoader` via importmap) : état,
  carte MapLibre, custom layer 3D, modules, symbolisation, sélection, import
  OSM/fichier, persistance Grist, sauvegarde projet, command palette.

## Décisions techniques (validées avec l'utilisateur)

1. **Direction = Atlas** (parmi Atlas / Workbench / Compass du design canvas).
2. **Fond de carte = OpenFreeMap** (`tiles.openfreemap.org`, gratuit, sans clé).
   Styles : Liberty (3D), Bright (plan), Positron (clair). Les bâtiments 3D
   viennent des couches `fill-extrusion` du style Liberty.
3. **Modèles 3D = custom layer three.js** : MapLibre n'a pas de couche `model`
   native ; on charge les GLTF (`GLTFLoader`) et on les place via des matrices
   `MercatorCoordinate`. Voir l'objet `Models3D` dans `app.js`.

## Conventions

- Pas de framework (vanilla JS + three.js). Handlers UI exposés via l'objet
  global `window.A` (les `onclick` inline appellent `A.xxx(...)`).
- Français pour l'UI et les commentaires.
- Tokens de couleur Atlas en variables CSS (`:root`).
- Persistance Grist optionnelle : table `Maquette_Layers` (Name, Color, Visible,
  GeomType, StyleJSON, GeoJSON). Le widget fonctionne aussi en **standalone**
  (sans Grist) avec autosave `localStorage` + export/import JSON.

## État actuel — fonctionne

- Carte MapLibre + OpenFreeMap, bâtiments 3D, terrain (DEM terrarium AWS, libre),
  ciel/atmosphère, éclairage solaire SunCalc (direction réaliste).
- Modules : **Lieu** (recherche Nominatim, géoloc, coords, rayon, nom de projet),
  **Couches** (liste, visibilité, suppression), **Symboliser** (couleur
  fixe/catégorisé/gradué, taille, modèle 3D, étiquette — aperçu live),
  **Modèles 3D** (bibliothèque GLTF), **Soleil** (presets, heure, date, ombres),
  **Vue & rendu** (pitch/bearing, fond, bâtiments/terrain/étiquettes/ciel).
- Import **OSM** (Overpass, presets) et **fichier GeoJSON** (drag & drop).
- **Sélection** d'objets sur la carte (clic, shift-clic, box-select), barre de
  sélection avec navigation prev/next, inspecteur d'édition (single + lot relatif).
- **Command palette** `⌘K`.
- Sauvegarde / chargement de projet JSON, export GeoJSON, autosave.

## Points d'attention / limites connues

- **Ombres portées** : MapLibre ne projette pas d'ombres au sol comme Mapbox
  Standard. Le toggle « Ombres » module l'éclairage des modèles three.js (pas de
  shadow map au sol). La direction de lumière suit bien la position solaire.
- **Perf 3D** : `MAX_3D_INSTANCES = 1200`. Au-delà, seuls les cercles de la
  couche sont affichés (pas de modèles). Le rebuild de la scène est *debounced*
  (`Models3D.scheduleRebuild`).
- **Sémantique des rotations** des modèles : `rotationZ` = azimut (lacet) — le
  contrôle principal. `rotationX`/`rotationY` (tangage/roulis) sont approximatifs
  (conversion repère GLTF Y-up → mercator Z-up).
- **Interaction sur les modèles 3D** : les objets three.js ne sont pas
  interrogeables par `queryRenderedFeatures`. On ajoute donc une couche de
  cercles MapLibre (faible opacité) servant de zone de clic / surbrillance.
- **Tests** : non testé en navigateur réel dans l'environnement de dev (pas de
  navigateur headless). À ouvrir dans Grist ou un navigateur pour valider le
  rendu WebGL et le chargement des GLTF (`nic01asfr.github.io/3D-Models/`).

## Publication

Non publié. Pour publier : `published/carte-grist/` avec `package.json`
(section `grist`) + copie de `index.html`/`app.js`, puis `npm run manifest`.
`requiredAccess: 'full'` est requis (création de table `Maquette_Layers`).

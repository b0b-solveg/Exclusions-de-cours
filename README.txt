# Gestion des Exclusions — PWA prête à déployer

## Contenu
- `index.html` : application (compatible mobile/PC, ES5)
- `manifest.webmanifest` : permet l’installation sur écran d’accueil
- `service-worker.js` : mode hors-ligne / caching
- `icons/icon-192.png`, `icons/icon-512.png` : icônes

## Déploiement (GitHub Pages)
1) Crée un dépôt GitHub, ajoute ces fichiers à la racine, pousse (`main`).
2) Paramètres du dépôt → Pages → Source: **Deploy from a branch** → **main / root**.
3) Ouvre l’URL GitHub Pages → l’appli est servie en HTTPS.
4) Sur téléphone : “Ajouter à l’écran d’accueil”.

## Synchro Google Sheets
- Dans l’appli (`⚙️ Paramètres`), colle l’URL **/exec** de ta Web App Apps Script.
- L’app envoie du JSON en `text/plain` (évite les préflights CORS).
- Le script Apps Script doit accepter les actions :
  - `append`: ajoute des lignes,
  - `delete`: supprime par `timestamp`,
  - `clear`: vide la feuille en gardant l’entête.

## Remarques
- Ne renomme ni l’ordre ni les noms de colonnes (la 1ère est `timestamp`).
- Si tu modifies le script Apps Script : re-déploie et mets à jour l’URL dans l’app.

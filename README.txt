# Gestion des Exclusions — PWA (Années scolaires + CSV + Synchro Sheets + Option dates futures)

## Contenu
- `index.html` : app responsive, **années scolaires**, **export CSV/JSON**, synchro **Google Sheets** (append/delete/clearYear/clear), **auto-switch d’année**, **filtrage strict**, et **option Inclure les dates futures** (Stats & Graphiques).
- `manifest.webmanifest`, `service-worker.js`, `icons/*` : PWA installable + hors-ligne.

## Déploiement
Hébergez ces fichiers en **HTTPS** (GitHub Pages / Netlify / OVH / autre). Ouvrez l’URL sur smartphone puis “Ajouter à l’écran d’accueil”.

## Connexion à Google Sheets (Apps Script)
1. Créez/ouvrez un Projet Apps Script lié à un Spreadsheet.
2. Ajoutez le code ci-dessous, enregistrez, puis **Déployer → Application web** :  
   - Exécuter en tant que : *Moi*  
   - Accès : *Tout le monde*  
3. Copiez l’URL `/exec` et collez-la dans l’app (onglet **Paramètres**) → **Enregistrer** → **Tester** (passe au vert).

### Code Apps Script
```javascript
function doGet(){ 
  return ContentService.createTextOutput('OK').setMimeType(ContentService.MimeType.TEXT);
}
function doPost(e){
  var bodyText = (e && e.postData && e.postData.contents) ? e.postData.contents : "{}";
  var payload = {};
  try { payload = JSON.parse(bodyText); } catch(err) { payload = {}; }

  var ss = SpreadsheetApp.getActiveSpreadsheet(); // ou SpreadsheetApp.openById('ID')
  if (!ss) return reply_({status:'error', message:'No active spreadsheet'});
  var sh = ss.getSheetByName('Exclusions') || ss.insertSheet('Exclusions');

  var headers = ["timestamp","anneeScolaire","date","heure","eleve","classe","motif","professeur","matiere","commentaire"];
  ensureHeader_(sh, headers);

  var action = payload.action || 'append';

  if (action === 'append') {
    var rows = Array.isArray(payload.exclusions) ? payload.exclusions : [payload];
    var values = rows.map(function(r){
      var year = r.anneeScolaire || computeAnneeScolaire_(r.date);
      return [
        r.timestamp || new Date().toISOString(),
        year,
        r.date || "", r.heure || "",
        r.nomEleve || r.eleve || "", r.classe || "", r.motif || "",
        r.professeur || "", r.matiere || "", r.commentaire || ""
      ];
    });
    if (values.length) sh.getRange(sh.getLastRow()+1, 1, values.length, headers.length).setValues(values);
    return reply_({status:'success', inserted: values.length});
  }

  if (action === 'delete') {
    var tss = payload.timestamps || [];
    if (!tss.length) return reply_({status:'error', message:'No timestamps provided'});
    var data = sh.getRange(2, 1, Math.max(0, sh.getLastRow()-1), 1).getValues(); // col A: timestamp
    var toDelete = [];
    for (var i=0; i<data.length; i++){
      var ts = String(data[i][0]||'').trim();
      if (tss.indexOf(ts) !== -1) toDelete.push(i+2);
    }
    toDelete.sort(function(a,b){return b-a;});
    for (var j=0; j<toDelete.length; j++){ sh.deleteRow(toDelete[j]); }
    return reply_({status:'success', deleted: toDelete.length});
  }

  if (action === 'clear') {
    var last = sh.getLastRow();
    if (last > 1) sh.deleteRows(2, last-1);
    return reply_({status:'success', cleared:true});
  }

  if (action === 'clearYear') {
    var year = payload.year;
    if (!year) return reply_({status:'error', message:'No year provided'});
    var lastRow = sh.getLastRow();
    if (lastRow <= 1) return reply_({status:'success', cleared:0});
    var data = sh.getRange(2, 1, lastRow-1, headers.length).getValues(); // col B contains year
    var toDelete = [];
    for (var i=0; i<data.length; i++){
      var rowYear = String(data[i][1]||'').trim();
      if (rowYear === year) toDelete.push(i+2);
    }
    toDelete.sort(function(a,b){return b-a;});
    for (var j=0; j<toDelete.length; j++){ sh.deleteRow(toDelete[j]); }
    return reply_({status:'success', cleared: toDelete.length});
  }

  return reply_({status:'error', message:'Unknown action: '+action});
}
function ensureHeader_(sh, headers) {
  var rng = sh.getRange(1,1,1,headers.length);
  var first = rng.getValues()[0];
  var ok = true;
  for (var i=0;i<headers.length;i++){ if (first[i] !== headers[i]) { ok=false; break; } }
  if (!ok) { rng.setValues([headers]); sh.setFrozenRows(1); }
}
function computeAnneeScolaire_(isoDate){
  if(!isoDate) return '';
  var d=new Date(isoDate+'T00:00:00');
  var y=d.getFullYear(), m=d.getMonth()+1;
  return (m>=9) ? (y+'-'+(y+1)) : ((y-1)+'-'+y);
}
function reply_(obj){
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## Astuces
- **Année scolaire** = septembre → août. Août 2025 = **2024-2025**, octobre 2025 = **2025-2026**.
- **Inclure les dates futures** : cochez la case dans *Statistiques*/*Graphiques* pour projeter les saisies à venir.
- **Supprimer dans Sheets** : la suppression dans l’app envoie `action: "delete"` avec les `timestamp` correspondants.

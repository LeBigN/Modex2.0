# Modex 2.0 — LOXAM Module (Agence 9112)

Application web (PWA) de chiffrage rapide des assemblages / démontages de base vie.
Fonctionne hors ligne, s'installe sur mobile, génère deux PDF.

---

## Contenu de l'archive

| Fichier | Rôle |
|---|---|
| `index.html` | L'application complète (HTML + CSS + JS + logo intégré) |
| `jspdf.umd.min.js` | Génération des PDF, en local (repli CDN automatique) |
| `manifest.json` | Déclaration PWA (nom, icônes, couleurs) |
| `service-worker.js` | Cache hors ligne — cache actuel : `modex2-v6` |
| `icon-192.png` | Icône application 192 px |
| `icon-512.png` | Icône application 512 px |

Les 6 fichiers doivent rester **dans le même dossier**, à plat.

---

## Déploiement sur GitHub Pages

1. Créer (ou ouvrir) le dépôt, par exemple `Modex2`.
2. Déposer les 6 fichiers à la racine du dépôt.
3. `Settings` → `Pages` → Source : `Deploy from a branch`, branche `main`, dossier `/ (root)`.
4. Attendre 1 à 2 minutes. L'app est disponible sur :
   `https://<compte>.github.io/Modex2/`

### Installation sur téléphone
- **Android / Chrome** : ouvrir le lien → menu ⋮ → *Ajouter à l'écran d'accueil*.
- **iPhone / Safari** : ouvrir le lien → bouton Partager → *Sur l'écran d'accueil*.

> HTTPS est obligatoire pour le mode hors ligne. GitHub Pages le fournit automatiquement.

---

## IMPORTANT — mise à jour de l'app

À **chaque** nouveau déploiement, incrémenter la version du cache dans `service-worker.js` :

```js
const CACHE = 'modex2-v6';   // -> 'modex2-v7', puis 'modex2-v8', etc.
```

Sans cela, les téléphones qui ont déjà installé l'app continueront d'afficher l'ancienne version.

---

## Utilisation

1. **Informations** — numéro, date, client, chantier / module, sous-traitant.
2. **Descriptif** — texte libre (optionnel), repris sur le devis.
3. **Grille de chiffrage** — toute la grille tarifaire est affichée sous forme de tableau,
   en trois sections : Montage / Raccordement électrique / Démontage.
   Il suffit de saisir le **Nb** en face des prestations concernées ; les prix unitaires
   restent modifiables ligne par ligne. `+ Ligne libre` pour une prestation hors grille.
   **Seules les lignes avec un Nb > 0 partent sur les PDF.**
4. **Tarif et marge** — tarif jour ou nuit (×1,6), puis marge de ×1,10 à ×1,50.

### Logique de calcul

Les prix de la grille sont le **déboursé sous-traitant** (ce qui est payé aux sous-traitants).

```
Prix client = P.U. sous-traitant  ×  coef. tarif (jour 1 / nuit 1,6)  ×  marge
```

Le récapitulatif affiche le déboursé, la marge dégagée et le total HT du devis.

### Les deux PDF

- **📄 Devis client** — gabarit officiel LOXAM Module (identique à Loxdevis).
  N'affiche **que** les prix margés : ni le déboursé, ni le taux de marge, ni le sous-traitant.
- **📊 Modex 2.0** — document interne. Reprend le sous-traitant, le tarif, la marge appliquée,
  et le détail P.U. ST / Déboursé / P.U. client / Montant client, avec le récapitulatif de marge.
  Porte la mention « DOCUMENT INTERNE — NE PAS DIFFUSER AU CLIENT ».

### Boutons du bas
`🗑` réinitialiser · `💾` enregistrer le chiffrage en cours sur l'appareil · `📊` PDF interne · `📄` devis client

---

## Points ouverts

- **Grille complète** : tous les prix unitaires sont renseignés.
  Pour modifier un tarif durablement, éditer la constante `GRILLE` dans `index.html`, par exemple :
  ```js
  "Raccordement électrique":[
    ["Raccordement des boîtes des modules vers TGBT en RDC", 120.00],
    ["Raccordement des boîtes des modules vers TGBT en R+1", 150.00],
  ],
  ```
- **Libellés** : « Callage » (8,30 €) et « Callage double » (16,60 €) — corrigés par Nico
  (ils étaient illisibles sur la capture d'origine de la grille Montage).
- **Liste des sous-traitants** : CRPS, TCPP, TERMICLIM, CC BAT, EMS, JM CARTIER (+ « Autre… »),
  reprise de l'écosystème Loxcheck / Loxsav — à ajuster si les intervenants montage diffèrent.

---

LOXAM Module — Agence 9112, Berre-l'Étang — Région PACA

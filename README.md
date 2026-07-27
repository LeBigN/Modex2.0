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
| `service-worker.js` | Cache hors ligne — cache actuel : `modex2-v13` |
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
const CACHE = 'modex2-v13';   // -> 'modex2-v14', puis 'modex2-v15', etc.
```

Sans cela, les téléphones qui ont déjà installé l'app continueront d'afficher l'ancienne version.

---

## Accès et mot de passe

L'app s'ouvre sur un écran de verrouillage. Le mot de passe est **transmis séparément**
(volontairement absent de ce fichier et du code : seule son empreinte SHA-256 figure
dans `index.html`). La case « Rester connecté sur cet appareil » évite de le retaper à
chaque ouverture ; sans elle, il est redemandé à la fermeture du navigateur.

### Changer le mot de passe
Le mot de passe n'est pas stocké en clair : seule son empreinte SHA-256 figure dans `index.html`.
1. Ouvrir l'app, puis la console du navigateur (F12 → Console).
2. Coller : `await sha256('MonNouveauMotDePasse')`
3. Copier la chaîne affichée et remplacer la valeur de `PIN_HASH` en haut du `<script>` :
   ```js
   const PIN_HASH='...la nouvelle empreinte...';
   ```
4. Redéployer en incrémentant le cache du service worker.

### Ce que cette protection vaut réellement

> **Important.** Il s'agit d'une protection **côté navigateur**. Elle empêche un collègue,
> un client ou quelqu'un qui tombe sur le lien d'accéder à l'app et de voir les tarifs.
> Elle **ne résiste pas** à une personne qui sait afficher le code source de la page :
> tout le contenu (grille, prix sous-traitants) est téléchargé par le navigateur avant
> la saisie du mot de passe.
>
> L'app contenant les **prix payés aux sous-traitants et les taux de marge**, si la
> confidentialité commerciale est un vrai enjeu, il faut une protection côté serveur.

### Vraie protection (recommandé) — Cloudflare Access

Gratuit jusqu'à 50 utilisateurs, et le contenu n'est jamais servi sans authentification :

1. Héberger l'app sur **Cloudflare Pages** au lieu de GitHub Pages
   (connexion directe au dépôt GitHub, déploiement automatique).
2. Dans le tableau de bord Cloudflare : **Zero Trust → Access → Applications → Add an application**
   → Self-hosted, et indiquer le domaine du site.
3. Créer une politique : autoriser une liste d'e-mails précis, ou tout le domaine
   `@loxam-module.com`.
4. À l'ouverture du lien, Cloudflare demande l'e-mail et envoie un code à usage unique.

Autre variante : dépôt **privé** + GitHub Pages privé, mais cela nécessite
un compte GitHub Enterprise Cloud.

---

## Utilisation

1. **Informations** — numéro, date, client, chantier / module, sous-traitant.
2. **Descriptif** — texte libre (optionnel), repris sur le devis.
3. **Grille de chiffrage** — deux cases à cocher **Montage** et **Démontage** en tête
   affichent ou masquent les sections concernées :
   *Montage* = Montage + Assemblage + Raccordement électrique ; *Démontage* = Démontage.
   **Aucune case n'est cochée à l'ouverture** : on choisit à chaque devis.
   Une section masquée est retirée des totaux et des PDF.
   La grille tarifaire s'affiche en tableau ; les sections visibles dépendent des cases cochées.
   Il suffit de saisir le **Nb** en face des prestations concernées ; les prix unitaires
   restent modifiables ligne par ligne. `+ Ligne libre` pour une prestation hors grille.
   **Seules les lignes avec un Nb > 0 partent sur les PDF.**
4. **Tarif et marge** — tarif jour ou nuit (×1,6), puis marge de ×1,10 à ×1,50.
   **L'app s'ouvre toujours sur « Sans marge »**, y compris après restauration d'un
   chiffrage enregistré : la marge doit être sélectionnée volontairement à chaque devis.
   Le coefficient appliqué est rappelé en rouge à côté de « Marge dégagée ».

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

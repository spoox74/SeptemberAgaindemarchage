# September Again — démarchage

Page unique pour suivre le démarchage des salles et festivals : qui a écrit à qui,
quand, ce qu'il reste à relancer, et où il manque encore une adresse.

Tout tient dans `index.html`. Pas de build, pas de dépendance à installer.

---

## Ce que contiennent les données

Des noms, des adresses mail et des numéros de portable de programmateurs et de
bénévoles qui vous ont répondu. Ce sont **leurs** données personnelles, pas les
vôtres. Trois conséquences pratiques :

- elles ne doivent jamais se retrouver dans un dépôt public ;
- l'accès doit être nominatif, pas « qui a le lien » ;
- si quelqu'un demande à être retiré, sa fiche doit pouvoir disparaître — d'où le
  bouton **Supprimer** sur chaque fiche.

`index.html` ne contient **aucune donnée**. Le fichier est vide au départ ; il se
remplit depuis la base, et la base se remplit depuis un JSON que tu importes
toi-même. C'est ce qui permet de publier la page sans publier le carnet
d'adresses.

---

## Marche à suivre, du début à la fin

Compter une heure la première fois. Rien n'est irréversible.

### 1. Créer le projet Firebase

1. Va sur [console.firebase.google.com](https://console.firebase.google.com) →
   **Créer un projet**. Nomme-le `september-again`.
2. Désactive Google Analytics, tu n'en as pas besoin.
3. Une fois le projet créé : **Build → Firestore Database → Créer une base de
   données**.
4. **Choisis `eur3 (europe-west)` comme emplacement.** C'est le seul moment où ce
   choix est possible, il est définitif, et il garde les données en Europe.
5. Quand il demande le mode : **Démarrer en mode production**. Cela verrouille
   tout par défaut. C'est le bon sens de départ : on ouvre ensuite pour vos quatre
   comptes, plutôt que de fermer après coup.

### 2. Créer les quatre comptes

1. **Build → Authentication → Commencer**.
2. Dans **Sign-in method**, active **Adresse e-mail/Mot de passe** uniquement.
   N'active rien d'autre.
3. Onglet **Users → Ajouter un utilisateur**. Crée les quatre comptes à la main,
   avec un mot de passe provisoire chacun. Toi, John, PM, Loïc.
4. Retourne dans **Settings → User actions** et **décoche « Enable create
   (sign-up) »**. À partir de là, plus personne ne peut se créer un compte depuis
   l'extérieur : seule la console peut en ajouter.
5. Chacun changera son mot de passe via **« Mot de passe oublié »** sur l'écran de
   connexion de la page.

Dans l'onglet **Users**, note les quatre **UID** (colonne de droite, une longue
suite de caractères). Tu en as besoin à l'étape suivante.

### 3. Écrire les règles d'accès

**Firestore Database → Rules**, remplace tout par le contenu de `firestore.rules`,
en substituant les quatre UID relevés :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function membre() {
      return request.auth != null
          && request.auth.uid in ["uid1","uid2","uid3","uid4"];
    }
    match /{document=**} {
      allow read, write: if membre();
    }
  }
}
```

**Publier.**

Le filtre porte sur l'UID, pas sur l'adresse mail : un compte peut changer son
adresse, jamais son UID. Tant qu'un UID n'est pas dans cette liste, le compte peut
exister et se connecter — il ne verra rien du tout.

Vérifie dans l'onglet **Rules playground** : une lecture de `venues/v01` sans
authentification doit être **refusée**.

### 4. Brancher la page sur le projet

1. **Paramètres du projet → Vos applications → Web (`</>`)**. Enregistre l'app,
   Firebase affiche un objet `firebaseConfig`.
2. Dans `index.html`, cherche `const FIREBASE_CONFIG` (en haut du script) et
   remplace le `null` :

```js
const FIREBASE_CONFIG =
  (typeof window!=="undefined" && window.SA_FIREBASE) ||
  {
    apiKey: "…",
    authDomain: "september-again.firebaseapp.com",
    projectId: "september-again",
    appId: "1:…:web:…"
  };
```

Ces clés ne sont pas des secrets. Elles désignent le projet, elles n'ouvrent
aucune porte : ce sont les règles de l'étape 3 qui protègent la base. Elles
peuvent donc rester dans le dépôt.

### 5. Restreindre la clé

Sans ça, quelqu'un qui récupère la config peut s'en servir depuis son propre site
et consommer ton quota.

1. [console.cloud.google.com](https://console.cloud.google.com) → sélectionne le
   projet → **APIs & Services → Credentials**.
2. Clique la clé « Browser key (auto created by Firebase) ».
3. **Application restrictions → Websites**, et ajoute :
   - `https://<toi>.github.io/*`
   - `http://localhost/*` (pour tester en local)
4. Enregistre.

### 6. Publier la page

```bash
cd september-again
git init
git add index.html README.md firestore.rules .gitignore
git commit -m "Suivi de démarchage"
git branch -M main
git remote add origin git@github.com:<toi>/september-again.git
git push -u origin main
```

Puis **Settings → Pages → Source : Deploy from a branch → main / (root)**.
L'adresse arrive en une minute : `https://<toi>.github.io/september-again/`.

⚠️ **Vérifie avant de pousser** que `donnees-initiales.json` n'est pas dans le
commit — le `.gitignore` fourni l'exclut, ainsi que toutes les sauvegardes :

```bash
git status --short        # rien qui ressemble à un .json de données
```

Un dépôt public est acceptable maintenant que le fichier ne contient plus de
coordonnées. Si tu préfères le privé, Pages fonctionne aussi (comptes Pro), sinon
Netlify ou Cloudflare Pages font la même chose gratuitement depuis un dépôt privé.

### 7. Remplir la base

Une seule fois, par une seule personne.

1. Ouvre l'adresse publiée. L'écran de connexion apparaît.
2. Connecte-toi avec ton compte.
3. La page est vide. Colonne de gauche → **Sauvegarde · restauration**.
4. Choisis le fichier `donnees-initiales.json` (celui que tu gardes en local, pas
   dans le dépôt) → **Restaurer**.
5. Les 50 lieux, 16 personnes et 47 échanges apparaissent. Les autres les voient
   en direct dès qu'ils se connectent.

### 8. Distribuer aux trois autres

Envoie-leur l'adresse de la page et leur mot de passe provisoire — **par un autre
canal que le mail du groupe** (SMS, de vive voix). Qu'ils le changent au premier
passage via « Mot de passe oublié ».

### 9. Sauvegarder

**Sauvegarde · restauration → Télécharger la sauvegarde** sort un JSON complet.
Une fois par mois, à garder ailleurs que sur la machine de tous les jours. C'est
ce qui te permet de repartir sur une autre base si Firebase ferme, si le projet
est supprimé par erreur, ou si tu changes d'avis sur l'hébergeur.

---

## Ce que cette configuration protège — et ce qu'elle ne protège pas

**Elle protège contre** : un inconnu qui tombe sur l'URL, l'indexation par les
moteurs, la fuite par le dépôt Git, l'usage de la clé depuis un autre site. Les
échanges sont chiffrés en transit, les données sont chiffrées au repos chez
Google, et rien n'est mis en cache sur le disque des postes.

**Elle ne protège pas contre** : un mot de passe faible ou réutilisé par l'un
d'entre vous — c'est devenu le maillon faible, prenez-en de vrais ; un poste
laissé déverrouillé avec la session ouverte ; et elle ne cloisonne rien entre vous
quatre, chacun voit et modifie tout. Enfin, les données sont chez Google : si
c'est un problème de principe, il faut un autre hébergement (voir plus bas).

**Ce que ça ne dispense pas de faire** : ne garder que ce qui sert, ne pas
diffuser ces coordonnées ailleurs, et supprimer la fiche de quelqu'un qui le
demande.

---

## Changer de base sans réécrire la page

Toute la persistance passe par un seul objet, avec six méthodes :

```js
db.doc("venues/v01").get()        // → {exists, data()}
db.doc("venues/v01").set(objet)
db.doc("venues/v01").update(champs)
db.doc("venues/v01").delete()
db.doc("meta/x").acquire({holder, ttlMs})     // verrou
db.collection("venues").onSnapshot(s => …)    // s.docs = [{id, data()}]
```

Deux pilotes sont écrits dans le fichier : `localDriver()` (localStorage, sans
partage) et `firebaseDriver()` (Firestore). Pour passer à Supabase, à CouchDB sur
une machine à toi, ou à autre chose, il suffit d'en écrire un troisième avec ces
mêmes méthodes et de l'ajouter dans `openDb()`. Le reste de la page ne change pas
d'une ligne, et la sauvegarde JSON se réimporte telle quelle.

Collections utilisées : `venues`, `people`, `exchanges`, `activity`, `meta`.

---

## Réglages dans la page

Bouton **« Le groupe · mail, liens »** :

- **membres du groupe** — séparés par des virgules. Ils alimentent le sélecteur
  « qui es-tu », le champ référent et le champ « qui la connaît » ;
- **adresse mail du groupe** ;
- **les deux phrases fixes** du modèle de mail ;
- **les six liens** (EPK, YouTube, écoute, site, Instagram, Facebook).

Enregistré dans la base partagée : une seule personne le renseigne, tout le monde
en profite.

## Ajouter, modifier, supprimer

Une fois les données importées, elles vivent dans la base. Le JSON ne sert plus
qu'aux sauvegardes.

**Ajouter un lieu** : bouton en haut ou dans la colonne. Colle ce que tu as sous
la main — un bout de newsletter, une signature de mail, une page web — la page en
tire le nom, l'adresse, le téléphone, la ville et le mois, et te laisse corriger.
Elle prévient si le nom ressemble à un lieu déjà présent.

**Modifier** : tout est éditable sur la fiche. Mail, téléphone, contact, lien,
ville, secteur ; et derrière « Modifier nom, mois, type, référent… » le reste.

**Supprimer un lieu** : en bas de la fiche. Une confirmation dit ce qui part — la
fiche, ses échanges, son rattachement aux personnes (les personnes restent).
Définitif, et pour tout le monde.

Pour un lieu qui existe encore mais ne donne rien, préfère **« Laisser tomber »** :
la fiche descend dans « Réglé » et garde la mémoire de ce qui a été tenté.

**Supprimer une personne** : bouton « supprimer » sur sa carte, à côté de
« détacher ». *Détacher* la retire de ce lieu seulement ; *supprimer* efface sa
fiche et ses coordonnées partout. C'est ce qu'il faut faire si quelqu'un demande à
ne plus figurer dans le fichier.

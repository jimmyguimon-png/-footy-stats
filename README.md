# -footy-stats
Mes stats foot

## Accès multi-appareils / multi-personnes

L'app utilise **Firebase Firestore** pour stocker les profils en ligne : chaque
joueur est identifié par un **code joueur** (ex: `MATHIEU-U17`) que tu choisis
toi-même. Entrer le même code depuis n'importe quel appareil (téléphone,
ordinateur...) donne accès au même profil, en temps réel — pas besoin de créer
de compte ni de mot de passe.

⚠️ Ce mode est simple mais peu sécurisé : toute personne qui connaît (ou
devine) un code peut lire et modifier les données de ce profil. Choisis un
code peu évident si tu veux limiter les accès non désirés.

### Configuration (à faire une seule fois)

1. Va sur [console.firebase.google.com](https://console.firebase.google.com)
   et crée un nouveau projet (gratuit).
2. Dans le projet, ajoute une application **Web** (icône `</>`), donne-lui un
   nom, puis copie l'objet de configuration fourni (`apiKey`, `authDomain`,
   `projectId`, etc.).
3. Ouvre `index.html` et remplace les valeurs `"REMPLACE_MOI"` dans la
   constante `firebaseConfig` (au début de la balise `<script>`) par celles
   de ton projet.
4. Dans la console Firebase, va dans **Firestore Database** → **Créer une
   base de données** (choisis un mode, par exemple "production").
5. Dans l'onglet **Règles** de Firestore, colle les règles suivantes (accès
   ouvert, cohérent avec le fonctionnement "code joueur sans mot de passe") :

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /joueurs/{code} {
         allow read, write: if true;
       }
     }
   }
   ```

6. Déploie/héberge `index.html` (ex: GitHub Pages) et ouvre la page : un écran
   te demande un code joueur. Le premier appareil à utiliser un code crée le
   profil, les suivants le rejoignent automatiquement.

### Fonctionnement des données

Chaque code joueur correspond à un document Firestore dans la collection
`joueurs`, contenant le profil, l'historique des matchs, les objectifs de
saison et les entraînements. Les mises à jour sont synchronisées en temps
réel entre tous les appareils connectés avec le même code (via
`onSnapshot`), et un cache local (persistance Firestore) permet de continuer
à utiliser l'app hors-ligne — les changements se synchronisent au retour du
réseau.

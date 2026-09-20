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

✅ Le projet Firebase (`stat-foot-e0642`) est déjà créé et sa config est déjà
renseignée dans `index.html`. Il reste deux étapes côté console Firebase :

1. Va sur [console.firebase.google.com](https://console.firebase.google.com)
   → projet `stat-foot-e0642` → **Firestore Database** → **Créer une base de
   données** (choisis un mode, par exemple "production").
2. Dans l'onglet **Règles** de Firestore, colle les règles suivantes (accès
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

3. Déploie/héberge `index.html` (ex: GitHub Pages) et ouvre la page : un écran
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

### Utilisateurs de l'ancienne version (avant Firebase)

Si quelqu'un utilisait déjà l'app avant l'ajout de Firebase (données stockées
uniquement dans le navigateur), ses anciennes données sont automatiquement
reprises la première fois qu'il crée un profil (un nouveau code joueur) —
mais seulement s'il le fait **depuis le même téléphone/navigateur** qu'avant.
Depuis un nouvel appareil, il faudra ressaisir les données manuellement (ou
utiliser l'export/import JSON si une sauvegarde a été faite avec l'ancienne
version).

## Console Parent

`console.html` affiche, sur une seule page, le profil et la note globale (OVR
+ les 6 stats FC) de **tous** les joueurs enregistrés dans Firestore, mis à
jour en temps réel. Utile pour un parent qui suit plusieurs enfants/joueurs
sans avoir à connaître chaque code joueur individuellement.

- URL : `https://jimmyguimon-png.github.io/-footy-stats/console.html`
- Protégée par un code d'accès distinct des codes joueurs, défini dans la
  constante `CODE_PARENT` en haut du `<script>` de `console.html` (valeur par
  défaut : `PAPA-FOOT`, à changer si tu veux).

⚠️ Cette protection est uniquement côté navigateur (comme le reste de l'app,
pas de vrai compte) : elle évite qu'un visiteur tombe dessus par hasard, mais
n'empêche pas quelqu'un de déterminé de lire le code dans la page. Pas de lien
vers cette page depuis l'app elle-même — seules les personnes à qui tu donnes
l'URL peuvent la trouver.

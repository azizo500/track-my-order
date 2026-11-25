# OrderFlow

MVP portfolio de gestion des commandes et livraisons. Interface française, montants en MAD, deux rôles : vendeur et livreur.

## Démarrage

Prérequis : Node.js 22 LTS (22.12+) ou une version plus récente compatible, npm et un accès Internet pour l’installation initiale.

Depuis la racine :

```sh
npm install
npm run setup
npm run dev
```

Ouvrir **http://localhost:3000**. API NestJS : `http://127.0.0.1:4000/api`.

`setup` crée les fichiers d’environnement manquants, génère une clé JWT aléatoire, applique la migration SQLite et ajoute les données de démonstration. Les fichiers existants sont conservés. La base se trouve dans `backend/prisma/dev.db`. Le seed ne remplace pas les comptes ou commandes déjà présents ; le relancer peut recréer une commande de démo supprimée. Arrêter les serveurs avant de relancer `setup` : Windows verrouille le moteur Prisma pendant son utilisation.

| Rôle | Email | Mot de passe |
| --- | --- | --- |
| Vendeur | seller@orderflow.ma | OrderFlow2026! |
| Livreur | driver@orderflow.ma | OrderFlow2026! |
| Deuxième livreur | salma@orderflow.ma | OrderFlow2026! |

## Fonctionnalités

- Vendeur : statistiques, consultation des détails, création, modification, suppression avec confirmation, attribution aux livreurs de son équipe.
- Livreur : uniquement ses commandes assignées, changement du statut et notes de livraison.
- Recherche par client, téléphone, produit, adresse ou identifiant ; filtres par statut.
- Design adapté du projet Stitch fourni : palette bleu/slate, polices Inter et Plus Jakarta Sans hébergées localement, cartes de statistiques, activité sur 7 jours et tableau opérationnel.
- Filtre par livreur et export CSV des commandes affichées ; page de suivi avec destination, livreur, étapes et notes.
- Interface responsive avec menu mobile et tableau défilant horizontalement.
- Données persistantes dans SQLite, aucun service externe requis.

Le livreur peut passer de `PENDING` à `DELIVERING` ou `CANCELLED`, puis de `DELIVERING` à `DELIVERED` ou `CANCELLED`. Après clôture, seules les notes restent modifiables. Le vendeur peut corriger tous les statuts. Une commande `DELIVERING` doit avoir un livreur. Le montant livré correspond à la somme des commandes `DELIVERED`, sans gestion des paiements.

## Vérification et production

```sh
npm test
npm run build
npm start
```

Les tests HTTP lancent une véritable API NestJS et une base SQLite temporaire isolée, puis vérifient authentification, validation, CRUD, permissions, isolation des vendeurs, attribution, transitions et statistiques. Ils ne modifient pas la démo. `npm test` compile d’abord le backend.

`npm start` lance les deux applications compilées avec `NODE_ENV=production` (cookie Secure ; utiliser `localhost` en local, HTTPS en ligne). Pour les lancer séparément, utiliser deux terminaux et définir `NODE_ENV=production` lors d’un déploiement :

```sh
npm run start -w backend
npm run start -w frontend
```

Pour développer séparément : `npm run dev -w backend` et `npm run dev -w frontend`. `Ctrl+C` arrête les processus lancés par le script racine.

## Structure

```text
backend/
  prisma/          Schéma, migration SQL et seed
  src/auth/        Connexion, cookie JWT, garde et utilisateur courant
  src/orders/      Contrôleur, service et DTO de validation
  src/prisma/      Connexion Prisma partagée
  test/            Tests fonctionnels HTTP
frontend/
  app/             Interface Next.js, composants opérations et styles Tailwind/CSS
scripts/           Configuration locale et lancement des deux serveurs
```

## Configuration et sécurité

### Hébergement Netlify et domaine

Le fichier `netlify.toml` configure le build du frontend depuis la racine du dépôt. Connecter le dépôt dans Netlify, puis définir `BACKEND_URL` avec l’URL HTTPS du backend (sans `/api` final) avant de lancer le déploiement. Le build Netlify s’arrête si cette variable manque. Netlify fournit une adresse `nom-du-projet.netlify.app` ; un domaine personnel peut ensuite être ajouté dans la gestion des domaines.

La configuration actuelle héberge uniquement Next.js sur Netlify. Le backend NestJS doit être déployé séparément et son fichier SQLite placé sur un disque persistant. Une autre option nécessite d’adapter l’API à des fonctions et d’utiliser une base hébergée. Ne pas déployer le fichier SQLite local comme stockage modifiable dans les fonctions Netlify. Le nom de domaine et les comptes d’hébergement doivent encore être choisis et connectés ; aucun déploiement n’a été effectué.

Voir `backend/.env.example` et `frontend/.env.example`. Le frontend utilise une réécriture Next.js `/api` vers le backend, ce qui garde les requêtes et cookies sur la même origine. Si vous changez le port ou le domaine frontend, adapter `FRONTEND_ORIGIN` ; si vous changez le port backend, adapter aussi `BACKEND_URL`, puis reconstruire le frontend.

Les mots de passe sont hachés avec bcrypt ; JWT signé HS256, expiration 8 heures, cookie HttpOnly et SameSite Strict. Aucune session dans localStorage. Les mutations nécessitent un en-tête spécifique et une origine autorisée, sans CORS permissif. Les connexions sont limitées à 10 essais/minute/IP, les autres routes à 180 requêtes/minute/IP. Les DTO rejettent les champs inconnus. Le serveur contrôle le rôle, la propriété vendeur et l’attribution livreur à chaque opération.

Pour une mise en ligne, utiliser HTTPS, une clé JWT privée et `NODE_ENV=production` pour activer le cookie Secure ; mettre `FRONTEND_ORIGIN` à l’URL HTTPS. Le backend écoute sur loopback et doit rester derrière le serveur Next.js ou un reverse proxy. Sauvegarder le fichier SQLite et remplacer les comptes de démonstration. Une déconnexion efface le cookie local ; un JWT copié reste valide jusqu’à son expiration (pas de liste de révocation dans ce MVP).

Périmètre volontairement limité : pas d’inscription, de gestion des comptes/livreurs, de réinitialisation de mot de passe, de paiement, GPS, catalogue, abonnement ou graphique avancé. Les données se rafraîchissent après chaque action et via le bouton Actualiser, sans temps réel.

Validation réalisée : 10 tests HTTP réussis, builds de production NestJS et Next.js réussis, connexions vendeur/livreur et création de commande vérifiées dans Chrome, rendu mobile vérifié à 400 px. `npm audit` ne signale aucune vulnérabilité sur le verrou de dépendances livré. Les overrides `effect` et `deepmerge-ts` corrigent les dépendances transitives de l’outillage Prisma 6.

Après la refonte Stitch : recherche, export CSV filtré, page de suivi, navigation mobile et bouton de déconnexion vérifiés dans Chrome. Les 10 tests HTTP restent réussis. Les licences des polices sont dans `frontend/app/fonts/`.

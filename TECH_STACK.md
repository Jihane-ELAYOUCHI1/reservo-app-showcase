# ⚙️ Stack Technique & Technologies

Le projet Reservo repose sur une architecture moderne, orientée performance et cross-platform, en utilisant les meilleures technologies de l'écosystème web et mobile.

---

## 1. La Stack en un coup d'œil

| Couche | Technologie | Version | Rôle |
|--------|-------------|---------|------|
| **Frontend Mobile** | Flutter | 3.x | Framework UI cross-platform pour l'App et les KDS |
| **Langage Mobile** | Dart | 3.x | Langage typé et performant de Flutter |
| **Frontend Dashboard** | React.js | 18.x | SPA d'administration et de reporting |
| **Build Tool Web** | Vite | 5.x | Bundler ultra-rapide pour React |
| **Backend** | Node.js | 20.x | Runtime JavaScript asynchrone côté serveur |
| **Framework Backend** | Express.js | 4.x | Routage et middleware REST |
| **Base de données** | MySQL | 8.0 | Stockage relationnel et transactions ACID |
| **Temps Réel** | Socket.io | 4.x | WebSocket bidirectionnel pour KDS et Plan de salle |
| **State Management** | Provider | 6.x | Gestion d'état légère et ciblée pour Flutter |

---

## 2. Justification des Choix Technologiques

### 2.1 Flutter & Dart (Mobile & KDS)
- **Performances natives** via compilation AOT (Ahead-of-Time).
- **Single codebase** : Une seule base de code pour générer des builds Android (APK/AppBundle) et iOS, ainsi que pour les différents écrans de taille (Tablette KDS vs Mobile Serveur).
- **Provider** utilisé pour un state management léger, évitant les rebuilds de l'interface entière.

### 2.2 React.js + Vite (Dashboard)
- Écosystème mature permettant la conception rapide de composants analytiques (Graphiques, Tableaux).
- **Vite** offre un démarrage à froid ultra-rapide et un Hot Module Replacement instantané, augmentant considérablement la productivité.

### 2.3 Node.js + Express.js (Backend)
- Architecture **Event-driven non-bloquante** parfaite pour gérer un grand nombre de petites requêtes simultanées (typiques des prises de commande).
- JavaScript Full-stack (React + Node), simplifiant la maintenance.
- Intégration naturelle et performante de **Socket.io** pour le temps réel.

### 2.4 MySQL 8.0 (Base de données)
- Nécessité des propriétés **ACID** pour garantir l'intégrité absolue des transactions financières (commandes et paiements).
- Contraintes référentielles fortes (liens indéfectibles entre tables, commandes, produits, utilisateurs).
- Le pool de connexions asynchrone via le driver `mysql2` garantit les performances.

---

## 3. Sécurité & Authentification

### Authentification Stateless JWT
Le système entier repose sur des JSON Web Tokens (JWT) :
1. Le client se connecte (`/api/auth/login`)
2. Le backend compare le mot de passe hashé via `bcryptjs`.
3. Le JWT généré (`HS256`) inclut le rôle de l'utilisateur (`serveur`, `caissier`, `admin`, etc.) et est valide 24h.
4. Sur mobile, le token est sécurisé dans le Keystore Android / Keychain iOS via `flutter_secure_storage`.
5. Toutes les routes Express passent par un middleware d'authentification (`auth.js`) qui vérifie le JWT.

### Sécurité Globale
- **CORS** strictement configuré sur le backend Node.
- Requêtes SQL préparées pour interdire toute **SQL Injection**.
- Blocage applicatif après 3 tentatives de connexion erronées.

---

## 4. Notifications Temps Réel (Socket.io)

Le système ne repose pas sur de l'AJAX classique, mais exploite le potentiel des WebSockets pour une réactivité instantanée via des `rooms` spécifiques :

| Room Socket.io | Rôle | Événements diffusés |
|----------------|------|---------------------|
| `room_cuisine` | Écran KDS Cuisine | `nouvelle_commande_kds`, `refresh_kds` |
| `room_bar` | Écran KDS Bar | `nouvelle_commande_kds`, `refresh_kds` |
| `room_patisserie`| Écran KDS Pâtisserie | `nouvelle_commande_kds`, `refresh_kds` |
| `room_caisse` | Tablette Caissier | `nouvelle_addition`, `refresh_kds` |
| `room_serveurs` | Téléphones Serveurs | `commande_prete`, `refresh_tables` |

**Optimisation** : Le push WebSocket est ciblé par rôle pour éviter le broadcast inutile sur le réseau local Wi-Fi du restaurant.

---

## 5. API REST & Communication

Le backend propose plus de 40 endpoints REST structurés.
- Communication client HTTP mobile assurée par **Dio**, avec des intercepteurs automatiques pour l'ajout du JWT.
- Les requêtes de création de commandes génèrent des UUIDs côté client, offrant une plus grande résilience si la requête nécessite d'être répétée.

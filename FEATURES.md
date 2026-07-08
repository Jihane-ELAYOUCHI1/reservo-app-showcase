# ✨ Fonctionnalités du Système Reservo

Le système Reservo se déploie sur plusieurs interfaces spécialisées en fonction des rôles du personnel. Ce document présente les fonctionnalités majeures par composant.

---

## 1. Fonctionnalités de l'Application Mobile (Flutter)

L'application mobile est le point d'interaction principal pour le personnel de salle et de préparation.

### 1.1 Module Plan de Salle (`/plan-salle`)
- **Vue d'ensemble interactive** de toutes les tables du restaurant.
- **Code couleur en temps réel** : 
  - 🟢 `Libre` : table disponible, prête pour une nouvelle commande.
  - 🔴 `Occupée` : table avec commande en cours (cliquable pour le détail).
  - 🟡 `Réservée` : table bloquée (évolution future).
- Rafraîchissement automatique via WebSocket.

### 1.2 Module Catalogue et Prise de Commande (`/produits`)
- **Navigation par catégories** (Entrées, Plats, Desserts, Boissons).
- **Gestion des variantes** (ex. Burger Simple, Double, Triple avec ajustement tarifaire automatique).
- **Notes spéciales** (ex. "sans fromage", "bien cuit").
- **Indicateur de stock** avec masquage automatique des produits en rupture.
- Ajout instantané au panier.

### 1.3 Panier et Récapitulatif
- Calcul automatique : Sous-total HT, TVA (10%) et **Total TTC**.
- Marquage manuel de **priorité urgente** (🔥).
- Création de la commande avec génération d'UUID client-side pour robustesse hors-ligne.

### 1.4 Module KDS — Kitchen Display System (`/kds`)
Interface tablette/mobile horizontale pour les postes de préparation (Cuisine, Bar, Pâtisserie).
- **Filtre par poste** : Chaque écran n'affiche que les produits qui lui sont destinés (dispatch intelligent).
- **Vue par ticket** : Affichage des tickets avec timer (code couleur pour le délai).
- **Boutons de progression** : 
  - `PRÉPARER` (statut → en préparation).
  - `PRÊT` (statut → prêt, ligne barrée/grisée).
  - `MARQUER TOUT COMME PRÊT` (validation globale).
- Notifications visuelles et sonores à chaque nouveau ticket urgents via WebSocket.

### 1.5 Module Caisse (`/caisse`)
- Liste des commandes prêtes en attente de paiement.
- **Modale d'encaissement** avec facture détaillée complète.
- Sélection du moyen de paiement (Espèce / Carte).
- Libération automatique de la table après paiement validé.

---

## 2. Fonctionnalités du Dashboard Admin (React.js)

Le dashboard est accessible sur navigateur pour la direction et les administrateurs système.

### 2.1 Dashboard & KPIs (Home)
- 💰 **Chiffre d'affaires du jour** (calcul instantané des transactions).
- 🍽️ **Commandes totales** et nombre de couverts servis.
- 📊 **Graphiques d'évolution** (CA sur 7 jours, répartition des ventes par catégorie).

### 2.2 Gestion des Produits et Catégories
- **CRUD complet** pour les produits (nom, prix, poste de préparation, catégorie, image).
- **Upload d'images** (multipart/form-data via Multer).
- Interface de gestion des **variantes** (prix additionnels configurables).

### 2.3 Gestion des Stocks
- Visualisation temps réel des niveaux de stock par produit.
- Mouvement manuel (ajout/retrait).
- Module de gestion des **gaspillages et pertes** (waste tracking).

### 2.4 Gestion des Utilisateurs et Accès
- Création de comptes personnels sécurisés (hash bcrypt).
- **Attribution stricte de rôles** (admin, gerant, serveur, caissier) impactant les permissions JWT.
- Révocation et désactivation de comptes.

### 2.5 Historique et Transactions
- Vue chronologique de tous les encaissements.
- Filtrage par date, par moyen de paiement ou par serveur.
- Traçabilité complète pour les réconciliations de caisse.

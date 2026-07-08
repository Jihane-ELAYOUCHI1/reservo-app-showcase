# 🏛️ Architecture du Système Reservo

Ce document détaille l'architecture globale, les workflows principaux et la structure de la base de données du projet Reservo.

---

## 1. Architecture Globale

### 1.1 Vue d'Ensemble

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           RESERVO — ARCHITECTURE SYSTÈME                        │
├──────────────────┬──────────────────┬──────────────────────────────────────────┤
│  📱 APP MOBILE   │  🖥️ DASHBOARD    │          ⚙️  BACKEND (Node.js)           │
│  (Flutter/Dart)  │  (React.js)      │         http://192.168.x.x:5000          │
│                  │  localhost:5173  │                                            │
│  • Serveurs      │  • Admin         │   ┌──────────────┐  ┌──────────────────┐  │
│  • Caissiers     │  • Gérants       │   │  REST API    │  │  WebSocket       │  │
│  • KDS Cuisine   │  • Rapports      │   │  Express.js  │  │  Socket.io       │  │
│  • KDS Bar       │  • Produits      │   └──────────────┘  └──────────────────┘  │
│  • KDS Pâtisserie│  • Utilisateurs  │           ↓                  ↓            │
└──────────────────┴──────────────────┤   ┌──────────────────────────────────────┤
         ↕  HTTP/REST + JWT           │   │     🗄️  BASE DE DONNÉES MySQL        │
         ↕  WebSocket (Socket.io)     │   │     reservo_db                       │
         ↕  JSON Payloads             │   │                                      │
                                      │   │  • commandes      • produits         │
                                      │   │  • lignes_commande • categories      │
                                      │   │  • utilisateurs   • tables           │
                                      │   │  • transactions   • stock_movements  │
                                      │   └──────────────────────────────────────┘
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Architecture par Couches

```mermaid
graph TB
    subgraph "Couche Présentation"
        A1["📱 Flutter App"]
        A2["🖥️ React Dashboard"]
    end
    
    subgraph "Couche Transport"
        B1["REST API + JWT"]
        B2["WebSocket Socket.io"]
    end
    
    subgraph "Couche Métier"
        C1["Auth Middleware"]
        C2["Routes Express"]
        C3["Business Logic"]
    end
    
    subgraph "Couche Données"
        D1["MySQL Pool (mysql2)"]
        D2["Transactions ACID"]
    end
    
    subgraph "Couche Infrastructure"
        E1["Node.js Server"]
        E2["Uploads / Static Files"]
    end
    
    A1 --> B1
    A2 --> B1
    A1 --> B2
    A2 --> B2
    B1 --> C1
    B2 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> D1
    D1 --> D2
    C3 --> E2
    E1 --> C2
```

---

## 2. Workflows Complets

### 2.1 Parcours Serveur

```mermaid
flowchart TD
    A([🔐 Connexion Serveur]) --> B[Vue Plan de Salle]
    B --> C{Table choisie}
    C -->|Table Libre| D[Sélection Produits]
    C -->|Table Occupée| E[Détail Commande Active]
    D --> F[Ajout au Panier\nQuantité + Variante + Note]
    F --> G{Panier complété?}
    G -->|Non - Continuer| D
    G -->|Oui| H[Récapitulatif Commande\nSous-total + TVA + Total TTC]
    H --> I[Marquage priorité si urgent 🔥]
    I --> J[📤 Envoyer en Cuisine]
    J --> K[Création Commande en BD\nPOST /api/commandes]
    K --> L[Ajout Lignes Articles\nPOST /api/commandes/:id/lignes]
    L --> M[Statut → en_attente\nPUT /api/commandes/:id/statut]
    M --> N{⚡ WebSocket}
    N --> O[🔔 Notification Cuisine/Bar/Pâtisserie]
    O --> P[Préparation en cours...]
    P --> Q[Statut → pret]
    Q --> R[🔔 Notification Serveur\ncommande_prete]
    R --> S[🔔 Notification Caissier\nnouvelle_addition]
    S --> T([Service en table])
    E --> U[Demande Addition\nPOST /api/commandes/:id/addition]
    U --> S
```

### 2.2 Workflow KDS (Cuisine / Bar / Pâtisserie)

```mermaid
flowchart TD
    A([🔐 Connexion KDS]) --> B{Poste configuré}
    B -->|cuisine| C1[Filtre Cuisine]
    B -->|bar| C2[Filtre Bar]
    B -->|patisserie| C3[Filtre Pâtisserie]
    
    C1 & C2 & C3 --> D[📋 Onglet EN ATTENTE]
    D --> E[Ticket reçu avec timer ⏱️]
    E --> F{Commande urgente?}
    F -->|🔥 OUI| G[Bandeau rouge PRIORITÉ]
    F -->|NON| H[Bandeau orange normal]
    G & H --> I[▶️ PRÉPARER — article\nPUT /api/lignes/:id/statut]
    I --> J[Ticket passe → EN PRÉPARATION]
    J --> K[✅ PRÊT — article\nPUT /api/lignes/:id/statut]
    K --> L{Tous articles prêts?}
    L -->|Non| I
    L -->|Oui| M[MARQUER TOUT COMME PRÊT\nPUT /api/commandes/:id/poste-pret]
    M --> N[Commande → statut pret]
    N --> O[⚡ WebSocket: commande_prete]
    O --> P[🔔 Serveur notifié]
    O --> Q[🔔 Caissier notifié]
```

### 2.3 Workflow Caissier

```mermaid
flowchart TD
    A([🔐 Connexion Caissier]) --> B[Écran Caisse — Onglet PRÊTES]
    B --> C[Liste commandes statut = pret]
    C --> D[🔔 Nouvelle notification\nnouvelle_addition]
    D --> E[Rafraîchissement automatique]
    E --> F[Sélection commande à encaisser]
    F --> G[Modale VOIR & ENCAISSER]
    G --> H[Facture client complète\nSous-total + TVA + Total TTC]
    H --> I{Méthode paiement}
    I -->|💵 Espèce| J1[Espèce sélectionnée]
    I -->|💳 Carte| J2[Carte sélectionnée]
    J1 & J2 --> K[✅ CONFIRMER ENCAISSEMENT]
    K --> L[POST /api/transactions\nEnregistrement paiement]
    L --> M[Commande → statut encaisse]
    M --> N[Table → statut Libre]
    N --> O[📊 Ajout à l'Historique]
    O --> P[Onglet HISTORIQUE]
    P --> Q[Affichage détail transaction\nMontant + Articles + Heure]
```

---

## 3. Base de Données

### 3.1 Schéma Relationnel (MySQL)

```mermaid
erDiagram
    utilisateurs {
        int id PK
        string nom
        string email
        string password_hash
        enum role
        datetime created_at
    }
    
    tables_restaurant {
        int id PK
        int numero
        int capacite
        enum statut
    }
    
    commandes {
        string id PK
        int table_id FK
        int serveur_id FK
        enum statut
        int priority_flag
        decimal total
        datetime created_at
        datetime updated_at
    }
    
    lignes_commande {
        int id PK
        string order_id FK
        int product_id FK
        int variant_id FK
        int qty
        decimal prix_unitaire
        string note_speciale
        enum statut_ligne
    }
    
    produits {
        int id PK
        string nom_fr
        decimal prix
        int categorie_id FK
        enum poste
        int stock_qty
        string image_url
        datetime created_at
    }
    
    categories {
        int id PK
        string nom
        int ordre
    }
    
    variantes_produit {
        int id PK
        int produit_id FK
        string nom
        decimal supplement_prix
    }
    
    transactions {
        int id PK
        string order_id FK
        decimal montant
        enum methode_paiement
        datetime created_at
    }
    
    stock_movements {
        int id PK
        int produit_id FK
        int quantite
        enum type
        string note
        datetime created_at
    }

    utilisateurs ||--o{ commandes : "crée"
    tables_restaurant ||--o{ commandes : "concerne"
    commandes ||--o{ lignes_commande : "contient"
    produits ||--o{ lignes_commande : "référencé_dans"
    variantes_produit ||--o{ lignes_commande : "appliqué_dans"
    categories ||--o{ produits : "classifie"
    produits ||--o{ variantes_produit : "possède"
    commandes ||--o| transactions : "génère"
    produits ||--o{ stock_movements : "suivi_dans"
```

### 3.2 Description des Tables Principales

- **`utilisateurs`** : Contient tous les comptes du personnel (rôles: admin, gerant, serveur, caissier).
- **`commandes`** : Table centrale avec un identifiant UUID v4 généré par le client. Lie la table, le serveur, le statut de la commande et le flag de priorité.
- **`lignes_commande`** : Chaque article d'une commande avec son propre statut de préparation.
- **`produits`** : Catalogue incluant la catégorie, le stock et surtout le `poste` (cuisine, bar, patisserie) qui est la **clé de dispatch KDS**.
- **`transactions`** : Enregistrement immuable de chaque paiement avec sa méthode.

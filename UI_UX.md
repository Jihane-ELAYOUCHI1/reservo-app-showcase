# 🎨 Design System & UI/UX Reservo

Le projet Reservo adopte une esthétique **premium warm restaurant** : une interface qui doit être à la fois élégante pour refléter le standard d'un restaurant gastronomique, tout en restant extrêmement fonctionnelle et lisible pour une utilisation rapide par le personnel.

---

## 1. Identité Visuelle

Les tons terreux et chauds évoquent la convivialité et la noblesse des matériaux (bois, cuir, épices), tout en offrant un excellent contraste sur mobile (fond beige, texte brun foncé).

### 1.1 Palette de Couleurs Primaires

| Nom | Hex | Aperçu | Usage |
|-----|-----|--------|-------|
| `brownDeep` | `#5C3D2E` | 🟫 | Couleur principale, headers, textes importants |
| `orangeEarth` | `#D3753B` | 🟧 | Couleur d'accentuation, CTA, boutons primaires |
| `beigeWarm` | `#F8E6C4` | 🟡 | Fonds de champs, badges, zones secondaires |
| `orangeLight` | `#F9B872` | 🍊 | Sous-titres, icônes, highlights |
| `caramel` | `#BC6633` | 🍯 | Variante orangée pour hover et focus |
| `terracotta` | `#C0533A` | 🏺 | Accentuation erreur/urgence terracotta |

### 1.2 Couleurs de Fond et Surface

| Nom | Hex | Usage |
|-----|-----|-------|
| `bgPage` | `#F7F3EE` | Fond général de l'application (anti-éblouissement) |
| `whitePure` | `#FFFFFF` | Cartes, modales, surfaces élevées |

### 1.3 Couleurs de Texte et Hiérarchie

| Nom | Hex | Usage |
|-----|-----|-------|
| `textPrimary` | `#2D1B0E` | Titres et corps de texte principal |
| `textSecondary` | `#9E8B7A` | Sous-titres, labels, placeholders |
| `textMuted` | `#BBBBBB` | Textes désactivés, hints |

### 1.4 Couleurs Sémantiques (Statuts)

Dans Reservo, la couleur dicte l'action instantanément (particulièrement pour les KDS et le plan de salle) :

| Statut | Hex | Signification |
|--------|-----|---------------|
| `statusGreen` | `#2EAD6E` | 🟢 Succès, table libre, commande prête |
| `statusYellow` | `#F0C040` | 🟡 En attente, ticket KDS normal, table réservée |
| `statusOrange` | `#E67E22` | 🟠 En préparation, attention modérée |
| `statusRed` | `#E74C3C` | 🔴 Erreur, table occupée, **commande urgente** (🔥) |

---

## 2. Typographie

La police `Poppins` a été choisie pour sa rondeur, sa modernité géométrique et sa parfaite lisibilité sur des petits écrans.

| Élément | Police | Taille | Poids | Letter Spacing |
|---------|--------|--------|-------|----------------|
| Titre principal | Poppins | 24–28px | 900 (Black) | 0 |
| Titre section | Poppins | 18–22px | 800 (ExtraBold) | 0.5 |
| Sous-titre | Poppins | 13–15px | 700 (Bold) | 0.5–2 |
| Corps | Poppins | 13–15px | 500 (Medium) | 0 |
| Labels / Badges | Poppins | 10–12px | 700 (Bold) | 0.5–1 |
| Hints / Placeholders | Poppins | 14px | 400 (Regular) | 0 |

---

## 3. Système d'Espacements et Géométrie

| Token | Valeur | Usage |
|-------|--------|-------|
| `radiusSm` | 8px | Badges, petits boutons |
| `radiusMd` | 12px | Boutons standards, champs de saisie |
| `radiusLg` | 16px | Cartes, modales |
| `radiusXl` | 20px | Grandes cartes, drawers |
| `radiusPill` | 30px | Tags, chips, indicateurs de statuts |
| `pagePadding` | 16px | Marge latérale globale des pages |
| `headerHeight` | 72px | Hauteur fixe des en-têtes |

---

## 4. Composants UI Standardisés

Pour garantir une expérience cohérente entre le Dashboard Admin (React) et l'App Mobile (Flutter), les composants partagent le même ADN visuel :

| Composant | Description | Variantes / États |
|-----------|-------------|-------------------|
| **Card (Carte)** | Surface blanche avec ombre de surélévation très légère | Normale, Sélectionnée (bordure orange), Prioritaire (bandeau rouge) |
| **Button Primary** | Fond `orangeEarth`, texte blanc | Normal, En chargement (spinner), Désactivé |
| **Button Outlined** | Fond transparent, bordure `orangeEarth` | Normal, Désactivé |
| **Status Badge** | Tag arrondi de type `radiusPill` | Vert (Prêt), Jaune (En attente), Orange (Préparation) |
| **Input Field** | Fond de champ `beigeWarm` sans bordure par défaut | Normal, Erreur (texte terracotta), Focus (bordure `orangeEarth`) |
| **SnackBar** | Notification temporaire (toast) fond `brownDeep` | Succès, Erreur, Info |
| **Tab Bar** | Navigation horizontale. Fond blanc, indicateur actif `brownDeep` | Adapté pour KDS (En attente / En préparation) et Caisse |

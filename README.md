# Test Technique – Développeur Mobile Flutter

## Application Financière : FinWallet

---

| Information | Détail |
|-------------|--------|
| **Durée estimée** | 4 à 6 heures |
| **Livrable attendu** | Projet Flutter fonctionnel + README explicatif |
| **À envoyer** | Lien vers repository Git (GitHub, GitLab, Bitbucket) |

---

## 1. Contexte

Vous êtes recruté pour développer **FinWallet**, une application mobile de gestion de portefeuille financier. L'application permet aux utilisateurs de consulter leurs comptes, visualiser l'historique de leurs transactions et effectuer des virements.

Votre mission est de développer un **prototype fonctionnel** démontrant vos compétences en développement Flutter dans un contexte exigeant en termes de sécurité et de qualité de code.

---

## 2. Spécifications Fonctionnelles

### 2.1 Écran Authentification

| Exigence | Description |
|----------|-------------|
| Connexion | Email et mot de passe |
| Validation | Format email valide, mot de passe minimum 8 caractères |
| États | idle, loading, success, error |
| Erreurs | Affichage user-friendly des messages d'erreur |
| **Bonus** | Authentification biométrique (Touch ID / Face ID) |

### 2.2 Écran Dashboard

| Exigence | Description |
|----------|-------------|
| Solde | Affichage du solde total du compte |
| Transactions | Liste des 5 dernières transactions |
| Navigation | Bouton d'accès rapide vers "Nouveau virement" |
| Rafraîchissement | Pull-to-refresh pour actualiser les données |
| **Bonus** | Animation du solde lors du chargement |

### 2.3 Écran Historique des Transactions

| Exigence | Description |
|----------|-------------|
| Liste | Pagination de 20 éléments par page |
| Affichage | Date, libellé, montant, type (crédit/débit) |
| Filtrage | Par type de transaction |
| Recherche | Par libellé |
| Scroll | Gestion du chargement infini (infinite scroll) |
| État vide | Message approprié si aucune transaction |

### 2.4 Écran Nouveau Virement

| Exigence | Description |
|----------|-------------|
| Bénéficiaire | Sélection via liste déroulante |
| Montant | Validation : > 0 et ≤ solde disponible |
| Libellé | Optionnel, maximum 140 caractères |
| Confirmation | Écran récapitulatif avant validation |
| États | formulaire, confirmation, processing, success, error |
| **Bonus** | Validation IBAN pour les nouveaux bénéficiaires |

---

## 3. Configuration Multi-Environnements (Flavors)

Dans un contexte d'application financière, il est crucial de pouvoir déployer l'application sur différents environnements sans risque de confusion entre les données de production et de test.

### 3.1 Flavors à Configurer

| Flavor | Nom de l'app | Package suffix | Environnement |
|--------|--------------|----------------|---------------|
| `dev` | FinWallet DEV | `.dev` | Développement |
| `staging` | FinWallet STG | `.staging` | Pré-production |
| `prod` | FinWallet | *(aucun)* | Production |

### 3.2 Configuration Native

**Android (`android/app/build.gradle`)** :

| Exigence | Description |
|----------|-------------|
| productFlavors | Définir les 3 flavors avec `applicationIdSuffix` approprié |
| resValue | Configurer le nom de l'application selon le flavor |

**iOS** :

| Exigence | Description |
|----------|-------------|
| Schemes | Créer les schemes Xcode correspondants (dev, staging, prod) |
| Bundle ID | Configurer les Bundle Identifiers par environnement |

### 3.3 Configuration Dart

Créez un système de configuration permettant d'accéder aux variables d'environnement :

```dart
abstract class AppConfig {
  String get appName;
  String get baseUrl;
  String get apiKey;
  bool get enableLogging;
  bool get enableCrashlytics;
}

class DevConfig implements AppConfig {
  // À implémenter
}

class StagingConfig implements AppConfig {
  // À implémenter
}

class ProdConfig implements AppConfig {
  // À implémenter
}
```

### 3.4 Variables par Environnement

| Variable | Dev | Staging | Prod |
|----------|-----|---------|------|
| `baseUrl` | `https://api-dev.finwallet.local` | `https://api-staging.finwallet.com` | `https://api.finwallet.com` |
| `apiKey` | `dev_key_xxx` | `staging_key_xxx` | *(sécurisé)* |
| `enableLogging` | `true` | `true` | `false` |
| `enableCrashlytics` | `false` | `true` | `true` |
| `certificatePinning` | `false` | `true` | `true` |

### 3.5 Différenciation Visuelle

| Exigence | Description |
|----------|-------------|
| Icône | Badge "DEV" ou "STG" sur l'icône de l'application |
| Bannière | Bandeau visuel pour les environnements non-production |
| **Bonus** | Couleur primaire légèrement différente par environnement |

### 3.6 Scripts de Build

Fournissez les commandes pour builder chaque flavor :

```bash
# Développement
flutter run --flavor dev -t lib/main_dev.dart

# Staging
flutter run --flavor staging -t lib/main_staging.dart

# Production
flutter run --flavor prod -t lib/main_prod.dart

# Build APK Production
flutter build apk --flavor prod -t lib/main_prod.dart --release
```

---

## 4. Spécifications Techniques

### 4.1 Architecture

| Exigence | Description |
|----------|-------------|
| Pattern | Clean Architecture ou architecture en couches |
| Couches | Presentation, Domain, Data |
| Repository | Pattern Repository pour l'accès aux données |
| Justification | Expliquer vos choix dans le README |

### 4.2 Gestion d'État

| Solution | Recommandation |
|----------|----------------|
| BLoC / Cubit | Recommandé |
| Riverpod | Accepté |
| Provider | Accepté |

### 4.3 API Mock

| Technique | Description |
|-----------|-------------|
| Délai | `Future.delayed` pour simuler la latence réseau |
| Données | Fichiers JSON locaux ou package `faker` |

**Endpoints à simuler :**

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/auth/login` | POST | Authentification |
| `/accounts/balance` | GET | Récupérer le solde |
| `/transactions` | GET | Liste des transactions (paginée) |
| `/transactions` | POST | Créer une transaction |
| `/beneficiaries` | GET | Liste des bénéficiaires |

### 4.4 Modèles de Données

```dart
class User {
  final String id;
  final String email;
  final String firstName;
  final String lastName;
}

class Account {
  final String id;
  final String accountNumber;
  final double balance;
  final String currency;
}

class Transaction {
  final String id;
  final double amount;
  final String label;
  final DateTime date;
  final TransactionType type;
  final TransactionStatus status;
}

class Beneficiary {
  final String id;
  final String name;
  final String iban;
}
```

### 4.5 Sécurité

| Exigence | Description |
|----------|-------------|
| Stockage | Token d'authentification via `flutter_secure_storage` |
| Validation | Validation des entrées utilisateur côté client |
| Erreurs | Ne pas exposer de données sensibles dans les messages |
| **Bonus** | Timeout de session automatique |

### 4.6 Tests

| Type | Minimum requis | Exemple |
|------|----------------|---------|
| Tests unitaires | 3 | Validation formulaire, calcul de solde |
| Tests de widgets | 2 | Affichage transaction, état de chargement |
| **Bonus** | 1 test d'intégration | Parcours de virement complet |

---

## 5. Structure de Projet

```
lib/
├── main.dart
├── main_dev.dart
├── main_staging.dart
├── main_prod.dart
├── core/
│   ├── config/
│   │   ├── app_config.dart
│   │   ├── dev_config.dart
│   │   ├── staging_config.dart
│   │   └── prod_config.dart
│   ├── constants/
│   ├── errors/
│   ├── network/
│   └── utils/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── dashboard/
│   ├── transactions/
│   └── transfer/
└── shared/
    ├── widgets/
    └── theme/

android/app/
├── build.gradle
└── src/
    ├── dev/res/mipmap-*/ic_launcher.png
    ├── staging/res/mipmap-*/ic_launcher.png
    └── prod/res/mipmap-*/ic_launcher.png

ios/
├── Flutter/
│   ├── Dev.xcconfig
│   ├── Staging.xcconfig
│   └── Prod.xcconfig
└── Runner.xcodeproj/xcshareddata/xcschemes/
    ├── dev.xcscheme
    ├── staging.xcscheme
    └── prod.xcscheme

test/
├── unit/
├── widget/
└── integration/
```

---

## 6. Critères d'Évaluation

### 6.1 Grille de Notation

| Critère | Pondération | Description |
|---------|-------------|-------------|
| Architecture & Organisation | 20% | Clean Architecture, séparation des responsabilités |
| Qualité du code | 15% | Lisibilité, conventions, DRY |
| Gestion d'état | 15% | Robustesse, maintenabilité |
| Configuration Flavors | 15% | Multi-environnements fonctionnels |
| UI/UX | 15% | Design, fluidité, feedback utilisateur |
| Tests | 10% | Couverture, pertinence |
| Sécurité | 10% | Bonnes pratiques implémentées |

### 6.2 Bonus Appréciés

| Bonus | Description |
|-------|-------------|
| Internationalisation | Support i18n (français/anglais) |
| Thème | Mode clair/sombre |
| Animations | Transitions soignées |
| CI/CD | GitHub Actions configuré |
| Fastlane | Déploiement automatisé par flavor |
| Documentation | Code commenté |
| Offline | Gestion du mode hors-ligne |
| Variables sécurisées | `--dart-define` ou package `envied` |

---

## 7. Livrables

| Livrable | Description |
|----------|-------------|
| Code source | Repository Git public ou privé (avec accès) |
| README.md | Voir contenu attendu ci-dessous |
| **Bonus** | APK de démonstration ou captures d'écran |

### 7.1 Contenu Attendu du README

| Section | Description |
|---------|-------------|
| Installation | Instructions d'installation et de lancement |
| Commandes | Build pour chaque flavor |
| Architecture | Choix techniques et justifications (schéma apprécié) |
| Flavors | Explication de la configuration multi-environnements |
| Difficultés | Problèmes rencontrés et solutions apportées |
| Améliorations | Évolutions envisagées avec plus de temps |

---

## 8. Données de Test

| Champ | Valeur |
|-------|--------|
| Email | `test@finwallet.com` |
| Mot de passe | `Test1234!` |

---

## 9. Conseils

| Conseil | Description |
|---------|-------------|
| Qualité > Quantité | Mieux vaut 3 écrans bien faits que 4 bâclés |
| Commits | Messages clairs et commits réguliers |
| Packages | Utilisez les packages reconnus de la communauté |
| UX | Loading states, messages d'erreur, feedback visuel |
| Code review | Code prêt à être relu : nommage explicite, commentaires pertinents |

---

## 10. Questions

Si vous avez des questions sur le sujet, n'hésitez pas à nous contacter.

La capacité à poser les bonnes questions fait aussi partie de l'évaluation.

---

**Bonne chance ! 🚀**

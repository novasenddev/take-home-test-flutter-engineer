# 📱 Take-Home Test – Flutter Engineer

> **Niveau cible** : Engineer 2 / Senior 1 (3-6 ans d'expérience)  
> **Durée estimée** : 6-8 heures (hors bonus)  
> **Délai de rendu** : 5 jours maximum

---

## 🎯 Objectif

Ce test évalue votre capacité à développer un **module mobile production-ready** pour une application fintech. Nous évaluons :

| Compétence | Ce que nous observons |
|------------|----------------------|
| **Architecture** | Clean Architecture, séparation des responsabilités |
| **Gestion d'état** | Maîtrise de BLoC, états typés, flux unidirectionnels |
| **Qualité** | Code testable, lisible, maintenable |
| **Sécurité** | Standards fintech (stockage sécurisé, biométrie, SSL pinning) |
| **Pragmatisme** | Équilibre entre perfection et livraison |

---

## 📋 Contexte Métier

Vous développez **FinWallet**, une application de gestion de portefeuille financier. Le MVP comprend :

1. **Authentification** sécurisée
2. **Dashboard** avec solde et transactions récentes
3. **Historique** des transactions avec filtres
4. **Nouveau virement** avec validation

### API Mock

Une API mock est fournie via **MockAPI** ou vous pouvez utiliser un fichier JSON local. Les contrats sont définis en section 9.

---

## 📦 Livrables Obligatoires

### Checklist de Rendu

- [ ] Code source sur repository Git (historique propre)
- [ ] README.md avec instructions d'installation et d'exécution
- [ ] 3 flavors configurés (dev, staging, prod)
- [ ] Tests unitaires (couverture ≥ 70% sur BLoCs et Use Cases)
- [ ] Pipeline CI fonctionnel (GitHub Actions)
- [ ] APK staging buildable et testable
- [ ] ARCHITECTURE.md expliquant vos choix

---

## 🏗️ Partie 1 : Architecture (25 points)

### 1.1 Clean Architecture Stricte

```
┌─────────────────────────────────────────────────────────────┐
│                      PRESENTATION                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                      │
│  │  Pages  │  │  BLoCs  │  │ Widgets │                      │
│  └────┬────┘  └────┬────┘  └─────────┘                      │
│       │            │                                         │
│       ▼            ▼                                         │
├─────────────────────────────────────────────────────────────┤
│                        DOMAIN                                │
│  ┌──────────┐  ┌───────────────┐  ┌─────────────┐           │
│  │ Entities │  │   Use Cases   │  │ Repositories│ (interfaces)
│  └──────────┘  └───────────────┘  └─────────────┘           │
│                        ▲                                     │
├────────────────────────┼────────────────────────────────────┤
│                        │        DATA                         │
│  ┌─────────┐  ┌────────┴───────┐  ┌─────────────┐           │
│  │  DTOs   │  │  Repositories  │  │ DataSources │           │
│  └─────────┘  │ (implementations)│  └─────────────┘           │
│               └────────────────┘                             │
└─────────────────────────────────────────────────────────────┘
```

| Exigence | Critère de validation | Points |
|----------|----------------------|--------|
| Séparation en 3 couches | Domain n'importe rien de Data/Presentation | 5 |
| Use Cases explicites | 1 use case = 1 action métier, testable isolément | 5 |
| Repository Pattern | Interface dans Domain, implémentation dans Data | 5 |
| Entities immuables | Utilisation de `freezed` ou équivalent | 5 |
| Injection de dépendances | `get_it` + `injectable` configurés | 5 |

### 1.2 Structure de Projet Attendue

```
lib/
├── main_dev.dart
├── main_staging.dart
├── main_prod.dart
│
├── core/
│   ├── config/                 # Configuration par environnement
│   ├── errors/                 # Exceptions et Failures typées
│   ├── network/                # Client HTTP, interceptors
│   ├── security/               # Biométrie, secure storage
│   └── utils/                  # Validators, formatters, extensions
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/    # Remote + Local
│   │   │   ├── models/         # DTOs (JSON serialization)
│   │   │   └── repositories/   # Implémentation
│   │   ├── domain/
│   │   │   ├── entities/       # Modèles métier (freezed)
│   │   │   ├── repositories/   # Interfaces
│   │   │   └── usecases/       # LoginUseCase, LogoutUseCase...
│   │   └── presentation/
│   │       ├── bloc/           # AuthBloc, états, events
│   │       ├── pages/          # LoginPage
│   │       └── widgets/        # LoginForm, BiometricButton...
│   │
│   ├── dashboard/              # Même structure
│   ├── transactions/           # Même structure
│   └── transfer/               # Même structure
│
└── shared/
    ├── widgets/                # Composants réutilisables
    ├── theme/                  # AppTheme, AppColors, AppTypography
    └── l10n/                   # Internationalisation
```

---

## 🔐 Partie 2 : Authentification (20 points)

### 2.1 Écran de Connexion

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Validation email** | Format RFC 5322, feedback temps réel | 2 |
| **Validation mot de passe** | 8+ chars, 1 maj, 1 chiffre, 1 spécial | 2 |
| **États du formulaire** | `idle` → `validating` → `submitting` → `success`/`error` | 3 |
| **Rate limiting** | Blocage 30s après 3 échecs, compteur visible | 3 |
| **Gestion des erreurs** | Messages contextuels (credentials invalides, réseau, serveur) | 2 |

### 2.2 Authentification Biométrique

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Détection support** | Vérifier disponibilité Touch ID / Face ID | 2 |
| **Fallback PIN** | Si biométrie échoue ou indisponible | 2 |
| **Activation optionnelle** | L'utilisateur choisit d'activer ou non | 2 |

### 2.3 Gestion de Session

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Stockage sécurisé** | Tokens dans `flutter_secure_storage` | 2 |
| **Refresh token** | Renouvellement automatique transparent | 3 |
| **Expiration** | Logout automatique après 5 min d'inactivité | 2 |
| **Remember me** | Option de persistance de session | 1 |

### 2.4 États BLoC Attendus

```dart
// auth_state.dart
@freezed
class AuthState with _$AuthState {
  const factory AuthState.initial() = _Initial;
  const factory AuthState.loading() = _Loading;
  const factory AuthState.authenticated(User user) = _Authenticated;
  const factory AuthState.unauthenticated() = _Unauthenticated;
  const factory AuthState.error(AuthFailure failure) = _Error;
  const factory AuthState.locked({
    required int remainingSeconds,
    required int attemptCount,
  }) = _Locked;
}
```

---

## 📊 Partie 3 : Dashboard (15 points)

### 3.1 Affichage du Solde

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Animation d'apparition** | CountUp animation au chargement | 2 |
| **Masquer/Afficher** | Toggle avec icône œil, état persisté | 2 |
| **Multi-comptes** | Affichage de plusieurs comptes si applicable | 1 |

### 3.2 Transactions Récentes

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Liste des 5 dernières** | Avec skeleton loading | 2 |
| **Pull-to-refresh** | Debounce 1s, indicateur visuel | 2 |
| **Navigation vers détail** | Tap → page de détail | 1 |

### 3.3 Mode Offline

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Cache-first** | Afficher les données en cache si offline | 3 |
| **Indicateur** | Banner "Données mises à jour il y a X min" | 1 |
| **Sync automatique** | Refresh au retour de la connexion | 1 |

---

## 📜 Partie 4 : Historique des Transactions (15 points)

### 4.1 Liste Paginée

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Cursor-based pagination** | 20 items/page, infinite scroll | 3 |
| **Loading state** | Skeleton en bas de liste pendant chargement | 1 |
| **État vide** | Message contextuel selon filtres | 1 |

### 4.2 Filtres et Recherche

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Filtre par type** | Tous, Crédits, Débits (chips) | 2 |
| **Filtre par période** | Date picker (début/fin) | 2 |
| **Filtre par montant** | Range slider min/max | 1 |
| **Recherche par libellé** | Debounce 300ms | 2 |
| **Tri** | Par date ou montant, asc/desc | 1 |

### 4.3 Affichage Transaction

```dart
// Informations à afficher par transaction
class TransactionTile {
  final DateTime date;          // Format: "15 Jan 2024"
  final String label;           // "Virement à Jean Dupont"
  final int amount;             // "+1 500 F" ou "-500 F"
  final TransactionType type;   // Icône différente
  final TransactionStatus status; // Badge couleur
  final String? category;       // Tag optionnel
}
```

| Points | Critère |
|--------|---------|
| 2 | Affichage clair et lisible de toutes les informations |

---

## 💸 Partie 5 : Nouveau Virement (15 points)

### 5.1 Formulaire Multi-étapes

```
┌──────────────────────────────────────────────────────────────┐
│  Étape 1: Bénéficiaire  →  Étape 2: Montant  →  Étape 3: Confirmation  │
│  ═══════════════════       ══════════════       ══════════════════     │
└──────────────────────────────────────────────────────────────┘
```

| Étape | Fonctionnalités | Points |
|-------|-----------------|--------|
| **Bénéficiaire** | Liste des favoris + recherche + ajout nouveau | 2 |
| **Validation IBAN** | Format + checksum (algorithme mod 97) | 3 |
| **Montant** | Validation temps réel, formatage devise, max = solde | 2 |
| **Libellé** | Max 140 chars avec compteur | 1 |
| **Récapitulatif** | Toutes les infos + montant en gras | 1 |

### 5.2 Sécurité du Virement

| Fonctionnalité | Comportement | Points |
|----------------|--------------|--------|
| **Confirmation biométrique** | Obligatoire avant envoi | 2 |
| **Idempotence** | Empêcher double soumission (disable button + idempotency key) | 2 |
| **Annulation** | Possible tant que status = "processing" | 1 |

### 5.3 États du Virement

```dart
@freezed
class TransferState with _$TransferState {
  const factory TransferState.initial() = _Initial;
  const factory TransferState.beneficiarySelected(Beneficiary beneficiary) = _BeneficiarySelected;
  const factory TransferState.amountEntered({
    required Beneficiary beneficiary,
    required int amount,
    required String label,
  }) = _AmountEntered;
  const factory TransferState.confirming(TransferSummary summary) = _Confirming;
  const factory TransferState.processing(String transactionId) = _Processing;
  const factory TransferState.success(Transfer transfer) = _Success;
  const factory TransferState.error(TransferFailure failure) = _Error;
}
```

| Points | Critère |
|--------|---------|
| 1 | Tous les états gérés avec transitions claires |

---

## ⚙️ Partie 6 : Configuration Multi-Environnements (10 points)

### 6.1 Flavors Requis

| Flavor | Package ID | App Name | API |
|--------|-----------|----------|-----|
| `dev` | `com.finwallet.app.dev` | FinWallet DEV | `api-dev.finwallet.local` |
| `staging` | `com.finwallet.app.staging` | FinWallet STG | `api-staging.finwallet.com` |
| `prod` | `com.finwallet.app` | FinWallet | `api.finwallet.com` |

### 6.2 Configuration Attendue

| Plateforme | Fichiers à fournir | Points |
|------------|-------------------|--------|
| **Android** | `build.gradle` avec `productFlavors`, signing configs | 3 |
| **iOS** | 3 schemes Xcode + fichiers `.xcconfig` | 3 |
| **Dart** | `AppConfig` abstract + implémentations par env | 2 |

### 6.3 Différenciation Visuelle

| Exigence | Points |
|----------|--------|
| Badge sur icône (DEV/STG) ou bannière in-app | 1 |
| Version + environnement visible dans Settings | 1 |

### 6.4 Scripts de Build

```bash
# Fichier Makefile ou scripts/ attendu
make run-dev        # flutter run --flavor dev -t lib/main_dev.dart
make run-staging    # flutter run --flavor staging -t lib/main_staging.dart
make build-staging  # flutter build apk --flavor staging -t lib/main_staging.dart
make build-prod     # flutter build appbundle --flavor prod -t lib/main_prod.dart --obfuscate
```

---

## 🧪 Partie 7 : Tests (15 points)

### 7.1 Couverture Minimale

| Type de test | Cible | Minimum | Points |
|--------------|-------|---------|--------|
| **Unit Tests - Use Cases** | Toutes les méthodes | 90% | 4 |
| **Unit Tests - BLoCs** | Tous les états/events | 90% | 4 |
| **Unit Tests - Validators** | Toutes les règles | 100% | 2 |
| **Widget Tests** | Formulaires critiques | 50% | 3 |
| **Integration Tests** | Parcours auth complet | 1 minimum | 2 |

### 7.2 Tests Obligatoires

#### Use Cases

```dart
// Exemple: login_usecase_test.dart
group('LoginUseCase', () {
  test('should return User when credentials are valid', ...);
  test('should return InvalidCredentialsFailure when credentials are wrong', ...);
  test('should return NetworkFailure when offline', ...);
  test('should return ServerFailure on 5xx response', ...);
  test('should return AccountLockedFailure after 3 failed attempts', ...);
});
```

#### BLoCs

```dart
// Exemple: auth_bloc_test.dart
blocTest<AuthBloc, AuthState>(
  'emits [loading, authenticated] when login succeeds',
  build: () => authBloc,
  act: (bloc) => bloc.add(LoginRequested(email: 'test@test.com', password: 'Test123!')),
  expect: () => [AuthState.loading(), AuthState.authenticated(mockUser)],
);
```

### 7.3 Mocking

| Dépendance | Comment mocker |
|------------|----------------|
| API HTTP | `MockClient` ou `mocktail` |
| Secure Storage | `MockFlutterSecureStorage` |
| Biométrie | `MockLocalAuthentication` |
| Date/Time | Injecter un `Clock` mockable |

---

## 🔒 Partie 8 : Sécurité (10 points)

### 8.1 Checklist Sécurité

| Mesure | Implémentation | Points |
|--------|----------------|--------|
| **Stockage tokens** | `flutter_secure_storage` (Keychain iOS / Keystore Android) | 2 |
| **Certificate Pinning** | SHA-256 pins dans la config (staging/prod) | 2 |
| **Obfuscation** | `--obfuscate --split-debug-info` en release | 1 |
| **Root/Jailbreak detection** | Avertissement (pas blocage) | 1 |
| **Screenshot prevention** | `FLAG_SECURE` sur écrans sensibles (Android) | 1 |
| **TLS 1.2 minimum** | Configuration native | 1 |
| **Timeout session** | Logout après 5 min d'inactivité | 1 |
| **Pas de logs sensibles** | Aucun token/password dans les logs | 1 |

### 8.2 Anti-patterns à Éviter

| ❌ Ne pas faire | ✅ Faire |
|----------------|---------|
| Stocker tokens en `SharedPreferences` | Utiliser `flutter_secure_storage` |
| Logger les tokens/mots de passe | Masquer les données sensibles |
| Hardcoder les URLs d'API | Utiliser la configuration par flavor |
| Ignorer les erreurs SSL | Implémenter le certificate pinning |

---

## 🔄 Partie 9 : API Mock (10 points)

### 9.1 Contrats d'API

Implémentez un mock local ou utilisez une solution comme **json_server** / **MockAPI**.

#### POST /auth/login

```json
// Request
{
  "email": "senior@finwallet.com",
  "password": "Senior2024!@#"
}

// Response 200
{
  "user": {
    "id": "usr_123",
    "email": "senior@finwallet.com",
    "firstName": "Jean",
    "lastName": "Dupont"
  },
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "expiresIn": 3600
}

// Response 401
{
  "error": "INVALID_CREDENTIALS",
  "message": "Email ou mot de passe incorrect"
}

// Response 423
{
  "error": "ACCOUNT_LOCKED",
  "message": "Compte bloqué",
  "remainingSeconds": 30
}
```

#### GET /accounts

```json
// Response 200
{
  "accounts": [
    {
      "id": "acc_001",
      "label": "Compte courant",
      "iban": "CI93CI000101015291456700074",
      "balance": 1543287,
      "currency": "XOF"
    },
    {
      "id": "acc_002",
      "label": "Compte épargne",
      "iban": "CI93CI000101017823415600089",
      "balance": 4215000,
      "currency": "XOF"
    }
  ]
}
```

#### GET /transactions?cursor={cursor}&limit=20&type={type}

```json
// Response 200
{
  "transactions": [
    {
      "id": "txn_001",
      "date": "2024-01-15T10:30:00Z",
      "label": "Virement à Koné Amadou",
      "amount": -150000,
      "type": "TRANSFER",
      "status": "COMPLETED",
      "category": "transfer"
    }
  ],
  "nextCursor": "txn_020",
  "hasMore": true
}
```

#### POST /transfers

```json
// Request
{
  "fromAccountId": "acc_001",
  "beneficiaryIban": "CI93CI000101017823415600089",
  "beneficiaryName": "Marie Konan",
  "amount": 50000,
  "label": "Remboursement déjeuner",
  "idempotencyKey": "idem_abc123"
}

// Response 201
{
  "transfer": {
    "id": "txn_099",
    "status": "PROCESSING",
    "amount": 50000,
    "createdAt": "2024-01-15T14:22:00Z"
  }
}

// Response 400
{
  "error": "INSUFFICIENT_FUNDS",
  "message": "Solde insuffisant"
}
```

---

## 🚀 Partie 10 : CI/CD (5 points)

### 10.1 Pipeline GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  analyze:
    # flutter analyze + dart format --check
    
  test:
    # flutter test --coverage
    # Upload coverage report
    
  build-staging:
    # flutter build apk --flavor staging
    # Upload artifact
    
  build-prod:
    # Manuel uniquement (workflow_dispatch)
    # flutter build appbundle --flavor prod --obfuscate
```

| Exigence | Points |
|----------|--------|
| Lint + format check | 1 |
| Tests avec coverage | 2 |
| Build staging automatique | 1 |
| Build prod manuel | 1 |

---

## 📊 Grille d'Évaluation Complète

### Récapitulatif des Points

| Partie | Points | Poids |
|--------|--------|-------|
| Architecture | 25 | 20% |
| Authentification | 20 | 16% |
| Dashboard | 15 | 12% |
| Historique Transactions | 15 | 12% |
| Nouveau Virement | 15 | 12% |
| Configuration Flavors | 10 | 8% |
| Tests | 15 | 12% |
| Sécurité | 10 | 8% |
| **Total** | **125** | **100%** |

### Barème de Niveau

| Score | Pourcentage | Niveau |
|-------|-------------|--------|
| < 62 | < 50% | Insuffisant |
| 62-77 | 50-62% | Junior confirmé |
| 78-93 | 63-75% | **Engineer 2** ✅ |
| 94-109 | 76-87% | **Senior 1** ✅ |
| ≥ 110 | ≥ 88% | Senior+ |

### Critères Éliminatoires

| Critère | Raison |
|---------|--------|
| Code ne compile pas | Livrable non fonctionnel |
| Pas de Clean Architecture | Compétence fondamentale |
| Aucun test | Non négociable à ce niveau |
| Tokens stockés en clair | Faille de sécurité critique |
| Pas de gestion d'erreurs | UX inacceptable |

---

## ⭐ Bonus (jusqu'à +25 points)

### Bonus Qualité (+10 max)

| Bonus | Points |
|-------|--------|
| Couverture tests > 85% | +3 |
| Golden tests (4 écrans) | +3 |
| Tests d'intégration complets (3 parcours) | +4 |

### Bonus Features (+10 max)

| Bonus | Points |
|-------|--------|
| Export PDF des transactions | +3 |
| Graphique évolution solde (fl_chart) | +3 |
| Mode sombre complet | +2 |
| Internationalisation (FR/EN) | +2 |

### Bonus DevOps (+5 max)

| Bonus | Points |
|-------|--------|
| Fastlane configuré (beta lane) | +3 |
| Firebase App Distribution intégré | +2 |

---

## 📝 Documentation Attendue

### README.md

```markdown
# FinWallet

## 📱 Description
[Brève description]

## 🛠️ Prérequis
- Flutter 3.19+
- Dart 3.3+
- Xcode 15+ (iOS)
- Android Studio (Android)

## 🚀 Installation
[Commandes détaillées]

## ▶️ Exécution
make run-dev        # Environnement dev
make run-staging    # Environnement staging
make test           # Lancer les tests
make coverage       # Rapport de couverture

## 🏗️ Architecture
[Diagramme + explication]

## 🔐 Sécurité
[Mesures implémentées]

## 📊 Tests
[Comment lancer, coverage actuel]
```

### ARCHITECTURE.md

Format ADR (Architecture Decision Record) apprécié :

```markdown
# ADR 001: Choix de BLoC pour la gestion d'état

## Contexte
[Pourquoi cette décision était nécessaire]

## Décision
[Ce qui a été décidé]

## Conséquences
[Avantages et inconvénients]
```

---

## 🎯 Conseils pour Réussir

### Ce que nous recherchons

✅ Code **lisible** et **bien structuré**  
✅ Tests qui **documentent le comportement**  
✅ Gestion d'erreurs **exhaustive et user-friendly**  
✅ Architecture **cohérente** et **justifiée**  
✅ Git history **propre** (commits conventionnels)

### Ce que nous pénalisons

❌ Over-engineering (patterns inutiles pour la taille du projet)  
❌ Copy-paste de code (violations DRY)  
❌ Tests qui ne testent rien de significatif  
❌ BLoCs avec logique métier (doit être dans les Use Cases)  
❌ Widgets "god class" de 500+ lignes

### Priorisation Recommandée

1. **D'abord** : Architecture + Auth + Tests associés
2. **Ensuite** : Dashboard + Transactions  
3. **Puis** : Virement + Flavors
4. **Enfin** : Bonus si temps restant

---

## 📤 Soumission

1. Repository Git public ou privé avec accès accordé
2. Tag ou release `v1.0.0` sur le commit final
3. Email avec objet : `[Take-Home Flutter] Prénom Nom - FinWallet`
4. Inclure dans l'email :
   - Lien vers le repository
   - Temps passé (estimation honnête)
   - Difficultés rencontrées (optionnel)

---

## ❓ FAQ

**Q : Puis-je utiliser Riverpod au lieu de BLoC ?**  
R : Non, BLoC est imposé pour ce test. Nous évaluons votre maîtrise de cet outil spécifique.

**Q : L'API mock doit-elle être déployée ?**  
R : Non, un mock local (fichiers JSON ou `http_mock_adapter`) suffit.

**Q : Dois-je implémenter un vrai backend ?**  
R : Non, concentrez-vous sur le code Flutter. Le mock suffit.

**Q : Les animations sont-elles évaluées ?**  
R : La fluidité oui (60fps), les animations complexes sont un bonus.

**Q : Puis-je utiliser des packages tiers ?**  
R : Oui, mais justifiez chaque dépendance dans le README.

---

## 📞 Questions

Pour toute question : **akwaba@sankofa-lab.co**

La capacité à poser les bonnes questions fait partie de l'évaluation.

---

## 🔍 Entretien Technique (Post-Test)

| Sujet | Durée | Questions types |
|-------|-------|-----------------|
| **Code Review** | 20 min | Pourquoi ce choix ? Comment refactorer X ? |
| **Architecture** | 15 min | Alternatives envisagées ? Limites de votre solution ? |
| **Évolution** | 15 min | Comment ajouter le paiement QR ? Le multi-devise ? |
| **Debugging** | 10 min | Ce bug apparaît, comment investiguer ? |

---

**Bonne chance ! 🚀**

*Montrez-nous comment vous construisez des applications mobiles de qualité production.*

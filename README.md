# Take-Home Test – Flutter Engineer

> **Niveau cible** : Engineer 2 / Senior 1 (3-6 ans d'expérience)  
> **Durée estimée** : 2 jours
> **Livrable attendu** | Projet Flutter production-ready + Documentation technique
> **À envoyer** | Lien vers repository Git (GitHub, GitLab, Bitbucket)

---

## 1. Contexte

Vous rejoignez l'équipe technique de **FinWallet**, une fintech en pleine croissance. L'application mobile permet aux utilisateurs de gérer leur portefeuille financier : consultation de comptes, historique des transactions et virements.

En tant qu'ingénieur expérimenté, vous êtes responsable de développer un **module complet et production-ready**. Vous devez démontrer votre capacité à :

| Compétence | Attente |
|------------|---------|
| Architecture | Concevoir une architecture scalable et maintenable |
| Qualité | Produire du code testable, documenté et performant |
| Sécurité | Implémenter les standards de sécurité d'une app financière |
| DevOps | Configurer un pipeline CI/CD et des environnements multiples |
| Leadership technique | Justifier vos choix et anticiper les évolutions |

---

## 2. Spécifications Fonctionnelles

### 2.1 Écran Authentification

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Connexion | Email et mot de passe | Obligatoire |
| Validation | Format email, mot de passe (8+ chars, 1 majuscule, 1 chiffre, 1 spécial) | Obligatoire |
| États | idle, loading, success, error avec messages contextuels | Obligatoire |
| Rate limiting | Blocage après 3 tentatives échouées (30 secondes) | Obligatoire |
| Biométrie | Touch ID / Face ID avec fallback PIN | Obligatoire |
| Remember me | Option de mémorisation sécurisée | Obligatoire |
| Session | Gestion du refresh token et expiration | Obligatoire |

### 2.2 Écran Dashboard

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Solde | Affichage avec animation et option masquer/afficher | Obligatoire |
| Transactions | 5 dernières avec skeleton loading | Obligatoire |
| Navigation | Accès rapide virement + historique | Obligatoire |
| Pull-to-refresh | Avec indicateur visuel et debounce | Obligatoire |
| Cache | Affichage des données en cache si offline | Obligatoire |
| Widgets | Graphique d'évolution du solde (7 derniers jours) | Obligatoire |

### 2.3 Écran Historique des Transactions

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Pagination | Cursor-based pagination (20 éléments/page) | Obligatoire |
| Affichage | Date, libellé, montant, type, statut, catégorie | Obligatoire |
| Filtres | Type, période (date picker), montant min/max | Obligatoire |
| Recherche | Par libellé avec debounce (300ms) | Obligatoire |
| Tri | Par date, montant (ascendant/descendant) | Obligatoire |
| Export | Génération PDF des transactions filtrées | Obligatoire |
| État vide | Message contextuel selon les filtres appliqués | Obligatoire |

### 2.4 Écran Nouveau Virement

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Bénéficiaire | Liste avec recherche + ajout nouveau | Obligatoire |
| Validation IBAN | Vérification format et checksum | Obligatoire |
| Montant | Validation temps réel, formatage devise | Obligatoire |
| Libellé | Max 140 caractères avec compteur | Obligatoire |
| Confirmation | Récapitulatif + authentification biométrique | Obligatoire |
| Idempotence | Gestion des doubles soumissions | Obligatoire |
| États | formulaire → validation → confirmation → 2FA → processing → success/error | Obligatoire |
| Annulation | Possibilité d'annuler pendant le processing | Obligatoire |

### 2.5 Gestion des Erreurs (Transverse)

| Scénario | Comportement attendu | Priorité |
|----------|---------------------|----------|
| Timeout réseau | Retry automatique (3x) avec backoff exponentiel | Obligatoire |
| Erreur serveur 5xx | Message user-friendly + option retry manuel | Obligatoire |
| Erreur client 4xx | Message contextuel selon le code d'erreur | Obligatoire |
| Token expiré | Refresh silencieux ou redirection login | Obligatoire |
| Mode offline | Cache-first avec indication de données obsolètes | Obligatoire |
| Maintenance | Écran dédié avec heure estimée de retour | Obligatoire |

---

## 3. Configuration Multi-Environnements (Flavors)

### 3.1 Flavors à Configurer

| Flavor | Nom de l'app | Package suffix | Environnement |
|--------|--------------|----------------|---------------|
| `dev` | FinWallet DEV | `.dev` | Développement |
| `staging` | FinWallet STG | `.staging` | Pré-production |
| `prod` | FinWallet | *(aucun)* | Production |

### 3.2 Configuration Native

**Android (`android/app/build.gradle`)** :

| Exigence | Description | Priorité |
|----------|-------------|----------|
| productFlavors | 3 flavors avec `applicationIdSuffix` | Obligatoire |
| Signing configs | Keystores différents par environnement | Obligatoire |
| ProGuard | Rules configurées pour la release | Obligatoire |
| resValue | Nom de l'app dynamique | Obligatoire |

**iOS** :

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Schemes | 3 schemes Xcode (dev, staging, prod) | Obligatoire |
| xcconfig | Fichiers de configuration par environnement | Obligatoire |
| Bundle ID | Identifiants différents par environnement | Obligatoire |
| Provisioning | Profils appropriés par environnement | Obligatoire |

### 3.3 Configuration Dart

```dart
abstract class AppConfig {
  String get appName;
  String get baseUrl;
  String get apiVersion;
  Duration get connectionTimeout;
  Duration get receiveTimeout;
  bool get enableLogging;
  bool get enableCrashlytics;
  bool get enablePerformanceMonitoring;
  bool get certificatePinningEnabled;
  List<String> get pinnedCertificates;
}
```

### 3.4 Variables par Environnement

| Variable | Dev | Staging | Prod |
|----------|-----|---------|------|
| `baseUrl` | `https://api-dev.finwallet.local` | `https://api-staging.finwallet.com` | `https://api.finwallet.com` |
| `connectionTimeout` | 30s | 15s | 10s |
| `enableLogging` | `true` | `true` | `false` |
| `enableCrashlytics` | `false` | `true` | `true` |
| `enablePerformanceMonitoring` | `false` | `true` | `true` |
| `certificatePinning` | `false` | `true` | `true` |

### 3.5 Différenciation Visuelle

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Icône | Badge "DEV" ou "STG" sur l'icône | Obligatoire |
| Bannière | Widget Banner en debug/staging | Obligatoire |
| Couleur | Thème légèrement différent par env | Obligatoire |
| Version | Affichage build number + env dans les settings | Obligatoire |

### 3.6 Scripts de Build

```bash
# Développement
flutter run --flavor dev -t lib/main_dev.dart --dart-define=ENV=dev

# Staging
flutter run --flavor staging -t lib/main_staging.dart --dart-define=ENV=staging

# Production
flutter run --flavor prod -t lib/main_prod.dart --dart-define=ENV=prod --release

# Build APK/AAB Production
flutter build appbundle --flavor prod -t lib/main_prod.dart --release --obfuscate --split-debug-info=build/debug-info
```

---

## 4. Spécifications Techniques

### 4.1 Architecture

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Pattern | Clean Architecture stricte | Obligatoire |
| Couches | Presentation → Domain → Data (dépendances unidirectionnelles) | Obligatoire |
| Use Cases | Un use case par action métier | Obligatoire |
| Repository | Interface dans Domain, implémentation dans Data | Obligatoire |
| Entities | Modèles métier immutables (freezed recommandé) | Obligatoire |
| DTOs | Séparation modèles API / modèles Domain | Obligatoire |
| Mappers | Conversion DTO ↔ Entity explicite | Obligatoire |

### 4.2 Gestion d'État

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Solution | BLoC/Cubit (obligatoire pour ce test) | Obligatoire |
| États | Sealed classes ou Freezed unions | Obligatoire |
| Events | Événements typés et immutables | Obligatoire |
| Hydratation | Persistance de certains états (hydrated_bloc) | Obligatoire |
| Tests | 100% des BLoCs testés | Obligatoire |

### 4.3 Injection de Dépendances

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Package | get_it + injectable | Obligatoire |
| Modules | Séparation par feature | Obligatoire |
| Scopes | Singleton, LazySingleton, Factory selon le cas | Obligatoire |
| Environnements | Configuration différente dev/staging/prod | Obligatoire |

### 4.4 Networking

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Client HTTP | HTTP/Dio avec configuration centralisée | Obligatoire |
| Interceptors | Auth, Logging, Error handling, Retry | Obligatoire |
| Certificate Pinning | Implémenté et configurable | Obligatoire |
| Timeout | Configurables par environnement | Obligatoire |
| Cancellation | Support des CancelToken | Obligatoire |

**Interceptor attendu :**

```dart
class AuthInterceptor extends Interceptor {
  // - Ajoute le Bearer token
  // - Gère le refresh token en cas de 401
  // - Queue les requêtes pendant le refresh
  // - Retry après refresh réussi
}
```

### 4.5 Persistance Locale

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Secure Storage | Tokens et données sensibles (flutter_secure_storage) | Obligatoire |
| Cache | Données non sensibles (Hive ou Isar) | Obligatoire |
| Stratégie | Cache-first avec invalidation temporelle | Obligatoire |
| Encryption | Base de données chiffrée | Obligatoire |
| Migration | Système de migration de schéma | Obligatoire |

### 4.6 Sécurité

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Stockage sécurisé | flutter_secure_storage pour tokens | Obligatoire |
| Certificate Pinning | SHA-256 pins configurables | Obligatoire |
| Biométrie | local_auth avec fallback sécurisé | Obligatoire |
| Root/Jailbreak | Détection et avertissement | Obligatoire |
| Screenshot | Désactivation en production (FLAG_SECURE) | Obligatoire |
| Obfuscation | Code Dart obfusqué en release | Obligatoire |
| SSL/TLS | Minimum TLS 1.2 | Obligatoire |
| Session | Expiration + logout sur inactivité (5 min) | Obligatoire |

### 4.7 Performance

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Liste | ListView.builder avec itemExtent | Obligatoire |
| Images | Cache et placeholder (cached_network_image) | Obligatoire |
| Rebuilds | Minimiser avec const, Selector, BlocSelector | Obligatoire |
| Memory | Dispose des controllers et subscriptions | Obligatoire |
| Startup | Splash screen natif, lazy loading des features | Obligatoire |

### 4.8 Observabilité

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Crash reporting | Firebase Crashlytics ou Sentry | Obligatoire |
| Analytics | Événements clés trackés | Obligatoire |
| Logs | Logger structuré avec niveaux | Obligatoire |
| Performance | Monitoring des temps de réponse API | Obligatoire |

---

## 5. Tests

### 5.1 Exigences de Couverture

| Type | Minimum requis | Cible |
|------|----------------|-------|
| Tests unitaires | 80% des use cases et BLoCs | 90%+ |
| Tests de widgets | Tous les widgets complexes | 70%+ |
| Tests d'intégration | Parcours critiques | 3 minimum |
| Golden tests | Écrans principaux | 4 minimum |

### 5.2 Tests Unitaires Obligatoires

| Composant | Tests attendus |
|-----------|----------------|
| Use Cases | Tous les cas nominaux et d'erreur |
| BLoCs/Cubits | Tous les états et transitions |
| Repositories | Avec mocks des data sources |
| Validators | Toutes les règles de validation |
| Mappers | Conversion DTO ↔ Entity |

### 5.3 Tests d'Intégration Obligatoires

| Parcours | Description |
|----------|-------------|
| Authentification | Login → Dashboard → Logout |
| Virement | Dashboard → Nouveau virement → Confirmation → Success |
| Historique | Dashboard → Historique → Filtres → Détail transaction |

### 5.4 Mocking

| Exigence | Description | Priorité |
|----------|-------------|----------|
| Package | mockito + build_runner | Obligatoire |
| HTTP | Mock des appels API (mocktail accepté) | Obligatoire |
| Secure Storage | Mock pour les tests | Obligatoire |
| Biométrie | Mock local_auth | Obligatoire |

---

## 6. CI/CD

### 6.1 Pipeline GitHub Actions

| Stage | Actions | Priorité |
|-------|---------|----------|
| Lint | `flutter analyze` + `dart format --check` | Obligatoire |
| Test | `flutter test --coverage` | Obligatoire |
| Coverage | Rapport + seuil minimum (80%) | Obligatoire |
| Build Dev | APK debug flavor dev | Obligatoire |
| Build Staging | APK/IPA release flavor staging | Obligatoire |
| Build Prod | AAB/IPA release flavor prod (manuel) | Obligatoire |

### 6.2 Fichier Attendu

```yaml
# .github/workflows/ci.yml
# À implémenter avec les stages ci-dessus
```

### 6.3 Fastlane (Bonus valorisé)

| Lane | Description |
|------|-------------|
| `test` | Exécution des tests |
| `beta` | Déploiement staging (Firebase App Distribution) |
| `release` | Déploiement prod (App Store / Play Store) |

---

## 7. Structure de Projet

```
lib/
├── main.dart
├── main_dev.dart
├── main_staging.dart
├── main_prod.dart
├── app.dart
├── injection.dart
│
├── core/
│   ├── config/
│   │   ├── app_config.dart
│   │   ├── dev_config.dart
│   │   ├── staging_config.dart
│   │   └── prod_config.dart
│   ├── constants/
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── http_client.dart
│   │   ├── interceptors/
│   │   │   ├── auth_interceptor.dart
│   │   │   ├── logging_interceptor.dart
│   │   │   └── retry_interceptor.dart
│   │   └── api_endpoints.dart
│   ├── security/
│   │   ├── biometric_service.dart
│   │   ├── secure_storage_service.dart
│   │   └── certificate_pinner.dart
│   ├── utils/
│   │   ├── validators.dart
│   │   ├── formatters.dart
│   │   └── extensions/
│   └── usecases/
│       └── usecase.dart
│
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_remote_datasource.dart
│   │   │   │   └── auth_local_datasource.dart
│   │   │   ├── models/
│   │   │   │   ├── user_dto.dart
│   │   │   │   └── token_dto.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       ├── logout_usecase.dart
│   │   │       └── refresh_token_usecase.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   ├── auth_bloc.dart
│   │       │   ├── auth_event.dart
│   │       │   └── auth_state.dart
│   │       ├── pages/
│   │       │   └── login_page.dart
│   │       └── widgets/
│   │           └── login_form.dart
│   │
│   ├── dashboard/
│   │   └── ... (même structure)
│   ├── transactions/
│   │   └── ... (même structure)
│   └── transfer/
│       └── ... (même structure)
│
└── shared/
    ├── widgets/
    │   ├── buttons/
    │   ├── inputs/
    │   ├── dialogs/
    │   └── loading/
    ├── theme/
    │   ├── app_theme.dart
    │   ├── app_colors.dart
    │   └── app_typography.dart
    └── l10n/
        ├── app_en.arb
        └── app_fr.arb

test/
├── unit/
│   ├── core/
│   └── features/
│       ├── auth/
│       │   ├── data/
│       │   ├── domain/
│       │   └── presentation/
│       └── ...
├── widget/
├── integration/
└── golden/

android/app/
├── build.gradle
├── proguard-rules.pro
└── src/
    ├── dev/
    ├── staging/
    └── prod/

ios/
├── Flutter/
│   ├── Dev.xcconfig
│   ├── Staging.xcconfig
│   └── Prod.xcconfig
└── Runner.xcodeproj/

.github/
└── workflows/
    └── ci.yml
```

---

## 8. Critères d'Évaluation

### 8.1 Grille de Notation

| Critère | Pondération | Seuil minimum |
|---------|-------------|---------------|
| Architecture Clean | 20% | Couches respectées, DI configurée |
| Qualité du code | 15% | Lisible, documenté, conventions Dart |
| Gestion d'état (BLoC) | 15% | États typés, transitions correctes |
| Configuration Flavors | 10% | 3 environnements fonctionnels |
| Sécurité | 15% | Certificate pinning, secure storage, biométrie |
| Tests | 15% | 80% coverage, tests d'intégration |
| CI/CD | 5% | Pipeline fonctionnel |
| UI/UX & Performance | 5% | Fluidité, feedback, optimisations |

### 8.2 Éléments Éliminatoires

| Critère | Raison |
|---------|--------|
| Pas de Clean Architecture | Compétence fondamentale attendue |
| Pas de tests unitaires | Non négociable à ce niveau |
| Tokens en clair | Faille de sécurité critique |
| Pas de gestion d'erreurs | Expérience utilisateur inacceptable |
| Code non compilable | Livrable non fonctionnel |

### 8.3 Bonus Valorisés

| Bonus | Points |
|-------|--------|
| Couverture tests > 90% | +5% |
| Fastlane configuré | +5% |
| Golden tests | +3% |
| Internationalisation complète | +3% |
| Mode offline robuste | +5% |
| Documentation technique ADR | +5% |
| Animations fluides (60fps) | +2% |
| Accessibilité (a11y) | +3% |

---

## 9. Documentation Attendue

### 9.1 README.md

| Section | Contenu attendu |
|---------|-----------------|
| Introduction | Description du projet et fonctionnalités |
| Prérequis | Versions Flutter, Dart, outils requis |
| Installation | Étapes détaillées |
| Configuration | Variables d'environnement, secrets |
| Commandes | Build, test, lint pour chaque flavor |
| Architecture | Diagramme + explication des couches |
| Tests | Comment lancer les tests, coverage |
| CI/CD | Description du pipeline |

### 9.2 Documentation Technique

| Document | Contenu attendu |
|----------|-----------------|
| ARCHITECTURE.md | Décisions d'architecture (ADR format apprécié) |
| SECURITY.md | Mesures de sécurité implémentées |
| API.md | Contrats d'API mockés |

---

## 10. Livrables

| Livrable | Description | Obligatoire |
|----------|-------------|-------------|
| Code source | Repository Git avec historique propre | Oui |
| README.md | Documentation complète | Oui |
| Tests | Couverture ≥ 80% | Oui |
| CI/CD | Pipeline GitHub Actions fonctionnel | Oui |
| APK Staging | Build testable | Oui |
| Documentation technique | ARCHITECTURE.md minimum | Oui |
| Captures d'écran | Tous les écrans | Recommandé |
| Vidéo démo | Parcours utilisateur | Bonus |

---

## 11. Données de Test

| Champ | Valeur |
|-------|--------|
| Email | `senior@finwallet.com` |
| Mot de passe | `Senior2024!@#` |
| PIN | `123456` |

**Comptes de test :**

| Compte | Solde | IBAN |
|--------|-------|------|
| Compte courant | 1 543 287 F CFA | CI93 CI00 0101 0152 9145 6700 0074 |
| Compte épargne | 4 215 000 F CFA | CI93 CI00 0101 0178 2341 5600 0089 |

---

## 12. Conseils

| Conseil | Description |
|---------|-------------|
| Commits | Conventional commits (feat:, fix:, refactor:, test:) |
| Branches | Feature branches + merge requests |
| Code review | Code prêt à être reviewé par un pair |
| Pragmatisme | Livrer un produit fonctionnel plutôt que parfait |
| Communication | Documenter les compromis et choix techniques |

---

## 13. Entretien Technique

Suite à ce test, un entretien technique de 60 minutes portera sur :

| Sujet | Durée | Description |
|-------|-------|-------------|
| Code review | 20 min | Revue de votre code, questions sur vos choix |
| Architecture | 15 min | Justification des décisions, alternatives envisagées |
| Évolutions | 15 min | Comment ajouter une feature (ex: paiement QR) |
| Questions | 10 min | Vos questions sur l'équipe et le projet |

---

## 14. Questions

Pour toute question sur le sujet, contactez-nous à **akwaba@sankofa-lab.co**.

La capacité à poser les bonnes questions et à lever les ambiguïtés fait partie de l'évaluation.

---

**Bonne chance ! 🚀**

*Ce test reflète les standards de qualité attendus chez FinWallet. Nous valorisons la rigueur, la sécurité et l'excellence technique.*

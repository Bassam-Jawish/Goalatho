# Goalatho Mobile

Cross-platform Flutter application for discovering sports pitches, booking sessions, and organizing games with friends. The app targets **Android** and **iOS** and communicates with a REST backend via a versioned API (`/api/v1`).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Flutter (Dart SDK `^3.10.7`) |
| State management & routing | [GetX](https://pub.dev/packages/get) |
| HTTP client | [Dio](https://pub.dev/packages/dio) + `pretty_dio_logger` (debug) |
| Dependency injection | [GetIt](https://pub.dev/packages/get_it) + GetX service locator |
| Local storage | `shared_preferences`, `flutter_secure_storage` |
| Push notifications | Firebase Cloud Messaging + `flutter_local_notifications` |
| Maps & location | `google_maps_flutter`, `geolocator` |
| UI utilities | `flutter_screenutil`, `cached_network_image`, `lottie`, `skeletonizer` |
| Localization | Flutter gen-l10n (English & Arabic) |
| Asset codegen | [flutter_gen](https://pub.dev/packages/flutter_gen) |
| Environment config | `flutter_dotenv` |

---

## Architecture

The codebase follows a **feature-first modular layout**. Each feature lives under `lib/modules/` and is organized into predictable layers:

```
module/
├── controller/   # GetX controllers — UI state, user actions
├── service/      # API calls and business logic
├── model/        # Data models and API response types
└── view/
    ├── pages/    # Screen widgets
    └── components/  # Feature-specific reusable widgets
```

Shared infrastructure sits in `lib/core/` (networking, storage, widgets, enums, error handling) and cross-cutting app configuration in `lib/config/` (theme, router, constants, device info).

### Data flow

```
View (Page/Widget)
    ↕  GetX reactive bindings
Controller
    ↕  async calls
Service  →  ApiService (Dio)  →  REST API
    ↕
Model (parsed response)
```

### App bootstrap

1. `main.dart` — initializes Flutter bindings and calls `Bindings.init()`.
2. `lib/core/config/bindings.dart` — loads storage, environment, Firebase, locale/theme preferences, and the DI container.
3. `lib/injection_container.dart` — registers global services (`ApiService`, feature services) and the `SettingsController`.
4. `lib/app.dart` — builds `GetMaterialApp` with routes, themes, and localization.
5. `SplashPage` — checks auth state and routes to login or the main shell.

Route-level controllers are registered lazily via GetX `BindingsBuilder` in `lib/config/router/navigation_manager.dart`.

---

## Project Structure

```
goalatho-mobile/
├── android/                  # Android native project (Kotlin, Gradle KTS)
├── ios/                      # iOS native project
├── assets/
│   ├── animations/           # Lottie JSON files
│   ├── fonts/                # Poppins (LTR) & Cairo (RTL)
│   ├── icons/                # SVG icons
│   ├── images/               # Static images & logos
│   └── sounds/               # Notification sounds
├── lib/
│   ├── main.dart             # Entry point
│   ├── app.dart              # Root widget & GetMaterialApp
│   ├── injection_container.dart
│   ├── firebase_options.dart # Generated Firebase config
│   ├── auto_generated/       # flutter_gen output (assets, fonts)
│   ├── config/
│   │   ├── router/           # Routes & NavigationManager
│   │   ├── theme/            # Light/dark themes, colors, styles
│   │   ├── constants/        # API endpoints, params, app constants
│   │   ├── app_info/         # Device & package metadata
│   │   └── language/         # Locale initialization
│   ├── core/
│   │   ├── config/           # App bootstrap (Bindings)
│   │   ├── network/          # ApiService, network info, failures
│   │   ├── storage/          # SharedStorage, secure token storage
│   │   ├── services/         # FirebaseApi (FCM + local notifications)
│   │   ├── middleware/       # AuthGuard, AuthHelper
│   │   ├── widgets/          # Shared UI components
│   │   ├── enums/            # Booking, payment, loading states, etc.
│   │   ├── error/            # Exceptions, failures, error codes
│   │   └── utils/            # Interceptors, validators, deep links
│   ├── l10n/                 # ARB files & generated localizations
│   └── modules/
│       ├── auth/             # Login, register, OTP, password reset
│       ├── splash/           # Splash screen & auth redirect
│       ├── main/             # Bottom-nav shell (IndexedStack)
│       ├── home/             # Home feed, ads, featured companies
│       ├── search/           # Pitch/company search with filters
│       ├── bookings/         # User bookings & availability
│       ├── company_detail/   # Pitch/company detail & booking flow
│       ├── teams/            # Teams & championships
│       ├── profile/          # User profile management
│       ├── notification/     # In-app notifications
│       └── settings/         # Theme, language, FAQ, contact
├── test/                     # Widget tests
├── pubspec.yaml
├── firebase.json             # FlutterFire configuration
└── local_config.env          # Environment variables (gitignored)
```

---

## Features & Modules

| Module | Responsibility |
|---|---|
| **auth** | Phone-based registration/login, OTP verification, forgot/reset password |
| **home** | Dashboard data, promotional content, company listings |
| **search** | Search pitches and companies with debounced queries |
| **bookings** | List, create, and cancel pitch bookings |
| **company_detail** | Company/pitch details, availability, booking actions |
| **teams** | Team and championship management (local persistence) |
| **profile** | View/edit profile, city selection, profile completion |
| **notification** | Fetch notifications, mark as read, FCM token sync |
| **settings** | Language (EN/AR), theme mode, app info, support links |

The main shell (`MainPage`) hosts five bottom-navigation tabs: **Home**, **Search**, **Bookings**, **Teams**, and **Settings**.

---

## Prerequisites

- [Flutter SDK](https://docs.flutter.dev/get-started/install) compatible with Dart `^3.10.7`
- Android Studio / Xcode for platform builds
- A configured Firebase project (FCM enabled)
- Google Maps API keys for Android and iOS (maps features)
- Access to the Goalatho backend API

Verify your setup:

```bash
flutter doctor
```

---

## Environment Configuration

Create a `local_config.env` file at the project root (this file is **gitignored**):

```env
HOST_IP=https://your-api-host.com
WEB_SERVICE=/api/v1/
ISRG_ROOT_X1=
```

| Variable | Description |
|---|---|
| `HOST_IP` | Base URL of the backend server |
| `WEB_SERVICE` | API path prefix appended to `HOST_IP` |
| `ISRG_ROOT_X1` | Optional ISRG Root X1 certificate (if required for SSL pinning) |

`ApiService` resolves the full base URL as `HOST_IP + WEB_SERVICE`.

---

## Getting Started

```bash
# Install dependencies
flutter pub get

# Generate asset references and localizations
dart run build_runner build --delete-conflicting-outputs
flutter gen-l10n

# Run on a connected device or emulator
flutter run
```

### Build for release

```bash
flutter build apk --release      # Android APK
flutter build appbundle --release # Android App Bundle
flutter build ios --release       # iOS (requires macOS + Xcode)
```

---

## Firebase Setup

Firebase is initialized at startup via `firebase_options.dart` (generated by FlutterFire CLI).

Required platform files (not committed in all setups):

- **Android:** `android/app/google-services.json`
- **iOS:** `ios/Runner/GoogleService-Info.plist`

Regenerate Firebase config if needed:

```bash
flutterfire configure
```

Push notifications use FCM with a custom Android notification channel and local notification display via `FirebaseApi` in `lib/core/services/firebase_api.dart`.

---

## Networking

- All HTTP traffic goes through `ApiService` (`lib/core/network/remote_api_service.dart`).
- Authenticated requests attach a Bearer token from secure storage.
- Request headers include device ID, platform, app version, and locale.
- API endpoint paths are centralized in `lib/config/constants/endpoints.dart`.
- Debug builds log requests/responses via `PrettyDioLogger`.

Example endpoint groups:

- **Auth:** `/auth/register`, `/auth/login`, `/auth/verify-otp`, …
- **Home:** `GET home`
- **Search:** `GET search`, `GET search/company/:id`
- **Bookings:** `bookings/my-bookings`, `bookings/availability/:pitchId`
- **Profile:** `/profile`, `/profile/complete`
- **Notifications:** `/customer/notifications`

---

## Localization

Supported locales: **English (`en`)** and **Arabic (`ar`)**.

- Source strings: `lib/l10n/app_en.arb`, `lib/l10n/app_ar.arb`
- Generated classes: `lib/l10n/app_localizations.dart`
- **Poppins** font for Latin script; **Cairo** for Arabic (RTL)

Language and theme preferences are managed by `SettingsController` and persisted locally.

---

## Code Generation

The project uses **flutter_gen** for type-safe asset access:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Generated output: `lib/auto_generated/assets.gen.dart`, `fonts.gen.dart`

Usage example:

```dart
import 'auto_generated/assets.gen.dart';

Assets.images.logo.goalathoLogo.image();
Assets.animations.splashAnimation.lottie();
```

App launcher icons are configured in `pubspec.yaml` under `flutter_launcher_icons`.

---

## Storage

| Store | Usage |
|---|---|
| `SharedStorage` + `SharedPreferences` | App settings, recent searches, cached data |
| `flutter_secure_storage` | JWT access/refresh tokens, FCM token |

Storage keys are defined as a typed enum in `lib/core/storage/storage_data.dart`, with per-key flags for secure storage and logout cleanup.

---

## Navigation

Routes are declared in `AppRoutes` and registered in `NavigationManager.getPages`. Key routes:

| Route | Path |
|---|---|
| Splash | `/` |
| Login | `/login` |
| Register | `/register` |
| OTP Verification | `/otp-verification` |
| Main (tab shell) | `/main` |
| Company Detail | `/company-detail` |
| Profile | `/profile` |
| Notifications | `/notification` |

Deep linking is handled by `lib/core/utils/deep_link_service.dart` via `app_links`.

---

## Testing

```bash
flutter test
```

Widget tests live in `test/widget_test.dart`.

---

## License

Private project — not published to pub.dev (`publish_to: 'none'`).

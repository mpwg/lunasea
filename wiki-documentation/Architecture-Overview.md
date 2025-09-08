# Architecture Overview

LunaSea follows a modular architecture built on Flutter with a focus on separation of concerns and code generation. This document provides an in-depth look at the application's structure and design patterns.

## Project Statistics

- **Codebase Size**: ~13MB
- **Dart Files**: 1,343 files
- **Lines of Code**: ~87,000
- **Largest Files**:
  - `lib/api/tautulli/models/activity/session.dart` (1,369 lines)
  - `lib/modules/settings/core/dialogs.dart` (1,328 lines)
  - `lib/modules/sonarr/core/dialogs.dart` (879 lines)

## High-Level Architecture

```
LunaSea Application
├── Bootstrap & Entry Point (main.dart)
├── Core Systems (database, logging, theme, routing)
├── Module System (service integrations)
├── API Layer (service clients)
├── UI Layer (widgets, screens)
└── Utilities (helpers, extensions)
```

## Directory Structure

```
lunasea/lib/
├── main.dart                 # Application entry point
├── core.dart                 # Core exports (deprecated)
├── modules.dart              # Module exports
├── vendor.dart               # Third-party exports
├── api/                      # API clients and models
│   ├── radarr/              # Radarr API client
│   ├── sonarr/              # Sonarr API client  
│   ├── lidarr/              # Lidarr API client
│   ├── tautulli/            # Tautulli API client
│   ├── nzbget/              # NZBGet API client
│   ├── sabnzbd/             # SABnzbd API client
│   └── wake_on_lan/         # WoL utilities
├── database/                 # Local database layer
│   ├── database.dart        # Database initialization
│   ├── models/              # Hive data models
│   └── tables/              # Database table definitions
├── modules/                  # Feature modules
│   ├── dashboard/           # Main dashboard
│   ├── external_modules/    # External service management
│   ├── radarr/              # Radarr module
│   ├── sonarr/              # Sonarr module
│   ├── lidarr/              # Lidarr module
│   ├── nzbget/              # NZBGet module
│   ├── sabnzbd/             # SABnzbd module
│   ├── search/              # Search functionality
│   ├── settings/            # Application settings
│   └── tautulli/            # Tautulli module
├── router/                   # Navigation and routing
├── system/                   # System utilities
│   ├── cache/               # Caching systems
│   ├── environment.dart     # Environment config (generated)
│   ├── logging/             # Logging system
│   ├── network/             # Network utilities
│   └── platform.dart       # Platform detection
├── types/                    # Custom type definitions
├── utils/                    # Utility functions
└── widgets/                  # Reusable UI components
```

## Core Design Patterns

### 1. Modular Architecture

Each service integration is implemented as a self-contained module:

```dart
lib/modules/radarr/
├── core/                    # Core functionality
│   ├── api_controller.dart  # API state management
│   ├── dialogs.dart         # UI dialogs
│   └── state.dart           # Module state
├── routes/                  # Module routes
│   ├── add_movie.dart
│   ├── catalogue.dart
│   └── movie_details.dart
└── radarr.dart             # Module exports
```

### 2. Code Generation Heavy

LunaSea heavily relies on code generation for:

- **JSON Serialization**: `json_annotation` + `json_serializable`
- **Database Models**: `hive_generator` for Hive objects
- **Environment Config**: `environment_config` for build variables
- **Assets**: `spider` for asset code generation
- **API Clients**: `retrofit_generator` for HTTP clients

### 3. State Management

**Provider Pattern**: Uses the Provider package for state management
- Scoped providers for module-specific state
- Global providers for app-wide state (theme, settings, etc.)

**Database as Single Source of Truth**: Hive database stores all configuration and cached data

### 4. API Layer Architecture

Each service has a dedicated API client with:

```dart
api/radarr/
├── models/              # Data models
│   ├── movie/
│   ├── queue/
│   └── system/
├── radarr_api.dart     # Retrofit API client
└── radarr.dart         # API exports
```

## Application Bootstrap

The app follows a structured bootstrap process in `main.dart`:

```dart
Future<void> main() async {
  runZonedGuarded(
    () async {
      WidgetsFlutterBinding.ensureInitialized();
      try {
        await bootstrap();  // Initialize core systems
        runApp(const LunaBIOS());
      } catch (error) {
        runApp(const LunaRecoveryMode());  // Fallback recovery
      }
    },
    (error, stack) => LunaLogger().critical(error, stack),
  );
}

Future<void> bootstrap() async {
  await LunaDatabase().initialize();     # Database first
  LunaLogger().initialize();             # Logging
  LunaTheme().initialize();              # Theme system
  if (LunaWindowManager.isSupported) 
    await LunaWindowManager().initialize();  # Window management
  if (LunaNetwork.isSupported) 
    LunaNetwork().initialize();          # Network layer
  if (LunaImageCache.isSupported) 
    LunaImageCache().initialize();       # Image caching
  LunaRouter().initialize();             # Routing
  await LunaMemoryStore().initialize();  # Memory store
}
```

## Database Layer

### Hive NoSQL Database
- **Local Storage**: All data stored locally in Hive boxes
- **Type Safety**: Strongly typed models with code generation
- **Migration Support**: Deprecated model classes for schema evolution

### Key Models
```dart
// Main configuration models
class LunaProfile extends HiveObject       # User profiles
class LunaExternalModule extends HiveObject # External service configs
class LunaIndexer extends HiveObject       # Search indexers
class LunaLog extends HiveObject           # Application logs
```

## Routing System

**go_router**: Declarative routing with:
- Nested routes for modules
- Parameter passing between screens
- Deep linking support
- Platform-aware navigation

## UI Architecture

### Theme System
- **Material Design**: Custom Material 3 implementation
- **AMOLED Support**: True black theme for OLED displays
- **Dynamic Theming**: System theme integration

### Widget Hierarchy
```
LunaBIOS (Root App)
├── Material App (Theme & Routing)
├── Module Screens
│   ├── Module Dashboard
│   ├── Module Routes
│   └── Module Dialogs
└── Shared Widgets
    ├── Cards & Lists
    ├── Forms & Inputs
    └── Loading States
```

### Responsive Design
- **Adaptive Layouts**: Different layouts for phone/tablet/desktop
- **Platform Integration**: Native platform conventions
- **Accessibility**: Screen reader and navigation support

## Service Integration Pattern

Each module follows a consistent pattern:

1. **API Client**: Retrofit-generated HTTP client
2. **Models**: JSON-serializable data models
3. **State Management**: Provider-based state
4. **UI Components**: Screens, dialogs, widgets
5. **Configuration**: Hive-stored settings

Example for Radarr:
```dart
// API client
@RestApi(baseUrl: "")
abstract class RadarrApi {
  factory RadarrApi(Dio dio) = _RadarrApi;
  
  @GET("/api/v3/movie")
  Future<List<RadarrMovie>> getMovies();
}

// State management
class RadarrState extends ChangeNotifier {
  List<RadarrMovie> _movies = [];
  List<RadarrMovie> get movies => _movies;
  
  Future<void> fetchMovies() async {
    _movies = await api.getMovies();
    notifyListeners();
  }
}
```

## Error Handling

### Global Error Handling
- **Zoned Execution**: Catches uncaught exceptions
- **Recovery Mode**: Fallback UI for critical failures
- **Logging**: Comprehensive error logging

### Network Resilience
- **Retry Logic**: Automatic retry for network failures
- **Offline Support**: Cached data when network unavailable
- **Timeout Handling**: Configurable timeouts per service

## Performance Considerations

### Code Generation Benefits
- **Compile-time Safety**: Errors caught at build time
- **Performance**: No runtime reflection
- **Maintainability**: Consistent patterns

### Caching Strategy
- **Image Caching**: Network images cached locally
- **Memory Store**: Hot data in memory
- **Database**: Persistent caching in Hive

### Platform Optimization
- **Tree Shaking**: Unused code eliminated
- **Platform Channels**: Native platform integration
- **Ahead-of-Time Compilation**: Release builds fully compiled

## Internationalization

**easy_localization**: 
- **JSON-based**: Translation files in `assets/localization/`
- **Code Generation**: Type-safe translation keys
- **Fallback Support**: Default locale fallback

## Testing Architecture

**Note**: The repository currently has no test infrastructure. For future development, recommended patterns would include:

- **Unit Tests**: For business logic and utilities
- **Widget Tests**: For UI components
- **Integration Tests**: For module workflows
- **Golden Tests**: For UI consistency

## Build Architecture

### Multi-stage Build
1. **Prepare**: Environment setup and code generation
2. **Build**: Platform-specific compilation
3. **Package**: Platform-specific packaging
4. **Sign**: Code signing for distribution

### Code Generation Pipeline
```bash
npm run generate:environment     # Build-time environment
npm run generate:assets          # Asset code generation  
npm run generate:build_runner    # JSON/Hive generation
npm run generate:localization    # Translation files
```

This architecture enables LunaSea to maintain consistency across platforms while providing rich functionality for managing self-hosted media services.
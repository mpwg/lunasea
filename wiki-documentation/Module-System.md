# Module System

LunaSea implements a modular architecture where each service integration is a self-contained module. This design enables clean separation of concerns, easy maintenance, and the ability to add new services without affecting existing functionality.

## Module Architecture Overview

Each module in LunaSea follows a consistent structure that includes:

1. **API Client Layer** - Communication with external services
2. **Core Logic Layer** - Business logic and state management  
3. **UI Layer** - Screens, widgets, and user interactions
4. **Configuration Layer** - Module-specific settings and preferences

## Module Structure

```
lib/modules/{service}/
├── core/                    # Core functionality
│   ├── api_helper.dart      # API communication helpers
│   ├── bottom_modal_sheets.dart  # Modal UI components
│   ├── dialogs.dart         # Dialog components
│   ├── state.dart           # Module state management
│   ├── extensions/          # Dart extensions
│   ├── types/               # Module-specific types
│   └── webhooks.dart        # Webhook handling
├── routes/                  # Module screens and navigation
│   ├── {screen_name}/
│   │   ├── route.dart       # Screen implementation
│   │   ├── state.dart       # Screen state
│   │   └── widgets/         # Screen-specific widgets
│   └── {service}.dart       # Main module route
└── {service}.dart          # Module exports
```

## Available Modules

### Media Management Modules

#### 1. Radarr Module (`lib/modules/radarr/`)

**Purpose**: Movie collection management and automation

**Key Routes**:
- `catalogue/` - Movie library browsing
- `add_movie/` - Adding new movies
- `movie_details/` - Individual movie management
- `releases/` - Release management
- `queue/` - Download queue
- `history/` - Activity history
- `upcoming/` - Upcoming releases
- `missing/` - Missing movies
- `system_status/` - System information

**Core Features**:
- Movie search and addition
- Quality profile management
- Release monitoring
- Manual and automatic importing
- Tag management

#### 2. Sonarr Module (`lib/modules/sonarr/`)

**Purpose**: TV series collection management and automation

**Key Routes**:
- `catalogue/` - Series library
- `add_series/` - Adding new series
- `series_details/` - Series management
- `season_details/` - Season-specific management
- `episode_details/` - Episode-level details
- `releases/` - Episode releases
- `queue/` - Download management
- `calendar/` - Episode calendar
- `upcoming/` - Upcoming episodes

**Core Features**:
- Series monitoring
- Season and episode management
- Episode file organization
- Release searching
- Calendar integration

#### 3. Lidarr Module (`lib/modules/lidarr/`)

**Purpose**: Music collection management and automation

**Key Routes**:
- `catalogue/` - Artist/album library
- `add_artist/` - Adding new artists
- `artist_details/` - Artist management
- `album_details/` - Album management
- `missing/` - Missing albums
- `history/` - Download history

**Core Features**:
- Artist and album monitoring
- Music quality profiles
- Metadata management
- Release group handling

### Download Client Modules

#### 4. NZBGet Module (`lib/modules/nzbget/`)

**Purpose**: Usenet download client management

**Key Routes**:
- `nzbget/` - Main dashboard
- `queue/` - Download queue
- `history/` - Download history
- `statistics/` - Usage statistics

**Core Features**:
- Queue management
- Download monitoring
- Post-processing control
- Statistics tracking

#### 5. SABnzbd Module (`lib/modules/sabnzbd/`)

**Purpose**: Alternative Usenet download client

**Key Routes**:
- `sabnzbd/` - Main dashboard
- `queue/` - Active downloads
- `history/` - Completed downloads
- `statistics/` - Performance metrics

**Core Features**:
- Download queue control
- Category management
- Speed limiting
- Statistics monitoring

### Monitoring Module

#### 6. Tautulli Module (`lib/modules/tautulli/`)

**Purpose**: Plex media server monitoring and analytics

**Key Routes**:
- `tautulli/` - Main dashboard
- `activity/` - Current activity
- `history/` - Playback history
- `users/` - User management
- `libraries/` - Library statistics
- `media/` - Media information
- `graphs/` - Analytics graphs
- `logs/` - System logs

**Core Features**:
- Real-time activity monitoring
- User session tracking
- Playback statistics
- Library analytics
- Custom notifications

### Utility Modules

#### 7. Search Module (`lib/modules/search/`)

**Purpose**: Unified search across indexers

**Key Routes**:
- `search/` - Search interface
- `results/` - Search results
- `details/` - Result details

**Core Features**:
- Multi-indexer searching
- Category filtering
- Direct downloads
- Result comparison

#### 8. Settings Module (`lib/modules/settings/`)

**Purpose**: Application configuration and preferences

**Key Routes**:
- `settings/` - Main settings
- `profiles/` - Profile management
- `modules/` - Module configuration
- `system/` - System settings
- `backup/` - Backup and restore

**Core Features**:
- Profile management
- Module configuration
- Backup/restore functionality
- System preferences

#### 9. Dashboard Module (`lib/modules/dashboard/`)

**Purpose**: Central hub and navigation

**Key Routes**:
- `dashboard/` - Main application dashboard

**Core Features**:
- Module status overview
- Quick actions
- Recent activity
- Navigation hub

#### 10. External Modules (`lib/modules/external_modules/`)

**Purpose**: Custom service integrations

**Core Features**:
- Custom URL management
- External service bookmarks
- User-defined integrations

## Module Implementation Pattern

### 1. API Client Integration

Each module integrates with its corresponding API client:

```dart
// lib/api/{service}/
abstract class {Service}API {
  // HTTP client setup
  factory {Service}API({
    required String host,
    required String apiKey,
    Map<String, String>? headers,
  });
  
  // Command handlers
  {Service}CommandMovies get movies;
  {Service}CommandQueue get queue;
  {Service}CommandSystem get system;
}
```

### 2. State Management

Modules use Provider pattern for state management:

```dart
// lib/modules/{service}/core/state.dart
class {Service}State extends ChangeNotifier {
  // API instance
  {Service}API? _api;
  
  // State variables
  List<{Service}Object> _items = [];
  bool _loading = false;
  String? _error;
  
  // Getters
  List<{Service}Object> get items => _items;
  bool get loading => _loading;
  String? get error => _error;
  
  // Actions
  Future<void> fetchData() async {
    _setLoading(true);
    try {
      _items = await _api!.getData();
      _clearError();
    } catch (e) {
      _setError(e.toString());
    } finally {
      _setLoading(false);
    }
  }
}
```

### 3. Configuration Integration

Modules access their configuration through the profile system:

```dart
// Module configuration access
class {Service}Helper {
  static {Service}API? get api {
    final profile = LunaProfile.current;
    if (!profile.{service}Enabled) return null;
    
    return {Service}API(
      host: profile.{service}Host,
      apiKey: profile.{service}Key,
      headers: profile.{service}Headers,
    );
  }
}
```

### 4. Route Organization

Each module defines its routes hierarchically:

```dart
// lib/modules/{service}/routes/{service}.dart
class {Service}Router extends LunaModuleRoutes {
  @override
  List<GoRoute> get routes => [
    GoRoute(
      path: '/{service}',
      builder: (context, state) => {Service}HomeRoute(),
      routes: [
        GoRoute(
          path: '/details/:id',
          builder: (context, state) => {Service}DetailsRoute(
            id: state.params['id']!,
          ),
        ),
        // Additional sub-routes
      ],
    ),
  ];
}
```

## Inter-Module Communication

### 1. Shared Services

Modules communicate through shared services:

```dart
// Shared notification service
LunaNotificationService.send({
  'title': 'Movie Added',
  'body': 'Movie successfully added to Radarr',
  'module': 'radarr',
});

// Shared search functionality
SearchService.performSearch({
  'query': searchTerm,
  'category': 'movies',
  'indexers': selectedIndexers,
});
```

### 2. Event System

Modules can emit and listen to events:

```dart
// Emit events
LunaEventBus.emit(RadarrMovieAdded(movie: movie));

// Listen to events  
LunaEventBus.listen<RadarrMovieAdded>((event) {
  // Handle movie addition in other modules
});
```

### 3. Shared State

Common data is shared through global providers:

```dart
// Profile changes affect all modules
Provider.of<ProfileState>(context).changeProfile('production');

// Theme changes propagate everywhere
Provider.of<ThemeState>(context).setAmoledTheme(true);
```

## Module Registration

Modules are registered in the main application:

```dart
// lib/modules.dart
export 'modules/dashboard.dart';
export 'modules/radarr.dart';  
export 'modules/sonarr.dart';
export 'modules/lidarr.dart';
export 'modules/nzbget.dart';
export 'modules/sabnzbd.dart';
export 'modules/tautulli.dart';
export 'modules/search.dart';
export 'modules/settings.dart';
export 'modules/external_modules.dart';
```

## Module Configuration Database

Each module can define its own database settings:

```dart
// Module-specific settings
enum RadarrDatabase<T> {
  DEFAULT_SORTING_MOVIES<RadarrMoviesSorting>(RadarrMoviesSorting.alphabetical),
  DEFAULT_SORTING_RELEASES<RadarrReleasesSorting>(RadarrReleasesSorting.weight),
  NAVIGATION_INDEX<int>(0),
  DEFAULT_VIEW_TYPE<RadarrMoviesViewType>(RadarrMoviesViewType.grid),
}
```

## Module Testing Strategy

While LunaSea currently has no tests, the modular architecture would support:

### 1. Unit Testing
- API client testing with mocked responses
- State management testing
- Utility function testing

### 2. Widget Testing
- Individual screen testing
- Component interaction testing
- State change verification

### 3. Integration Testing
- Module workflow testing
- Inter-module communication testing
- End-to-end user scenarios

## Adding New Modules

To add a new service module:

1. **Create Module Structure**:
   ```bash
   mkdir -p lib/modules/{service}/{core,routes}
   mkdir -p lib/api/{service}/{commands,models}
   ```

2. **Implement API Client**:
   - Define service API interface
   - Implement command handlers
   - Create data models

3. **Build Core Logic**:
   - State management
   - Configuration helpers
   - Business logic

4. **Create UI Components**:
   - Route definitions
   - Screen implementations
   - Custom widgets

5. **Register Module**:
   - Add to module exports
   - Register routes
   - Add configuration options

This modular architecture ensures that LunaSea remains maintainable and extensible while providing consistent user experiences across all supported services.
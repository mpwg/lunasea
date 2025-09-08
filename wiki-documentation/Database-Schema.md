# Database Schema

LunaSea uses Hive, a lightweight and fast NoSQL database for Flutter/Dart, to store all application data locally. This document covers the database structure, models, and data management patterns.

## Database Technology

**Hive NoSQL Database**
- **Local Storage**: All data stored locally on device
- **Type Safety**: Strongly typed models with code generation
- **Performance**: Fast read/write operations
- **Cross-Platform**: Works on all supported platforms
- **Schema Evolution**: Support for model migrations

## Database Boxes (Tables)

LunaSea organizes data into several Hive "boxes" (equivalent to database tables):

| Box Name | Type | Purpose |
|----------|------|---------|
| `alerts` | `dynamic` | In-app notifications and alerts |
| `externalModules` | `LunaExternalModule` | External service configurations |
| `indexers` | `LunaIndexer` | Search indexer configurations |
| `logs` | `LunaLog` | Application logs and debug information |
| `lunasea` | `dynamic` | Core application settings |
| `profiles` | `LunaProfile` | User profiles with service configurations |

## Core Data Models

### 1. LunaProfile (typeId: 0)

The primary configuration model that stores all service connection details for a user profile.

```dart
@HiveType(typeId: 0)
class LunaProfile extends HiveObject {
  // Service Enablement Flags
  @HiveField(0) bool lidarrEnabled;
  @HiveField(3) bool radarrEnabled;
  @HiveField(6) bool sonarrEnabled;
  @HiveField(9) bool nzbgetEnabled;
  @HiveField(12) bool sabnzbdEnabled;
  @HiveField(18) bool tautulliEnabled;
  
  // Connection Details (per service)
  @HiveField(1) String lidarrHost;
  @HiveField(2) String lidarrKey;
  @HiveField(26) Map<String, String> lidarrHeaders;
  
  @HiveField(4) String radarrHost;
  @HiveField(5) String radarrKey;
  @HiveField(27) Map<String, String> radarrHeaders;
  
  // ... similar patterns for all services
}
```

**Key Features:**
- **Multiple Profiles**: Support for different environments (dev, prod, etc.)
- **Service Isolation**: Each service has independent configuration
- **Custom Headers**: Support for authentication headers per service
- **Validation States**: Tracks connection test results

### 2. LunaExternalModule (typeId: 26)

Stores configurations for external modules and services not directly integrated.

```dart
@HiveType(typeId: 26)
class LunaExternalModule extends HiveObject {
  @HiveField(0) String displayName;    // User-friendly name
  @HiveField(1) String host;           // Service URL
}
```

**Use Cases:**
- Custom integrations
- Third-party services
- Development endpoints
- Bookmark management

### 3. LunaIndexer (typeId: 27)

Configuration for search indexers (Newznab, Torznab, etc.).

```dart
@HiveType(typeId: 27)  
class LunaIndexer extends HiveObject {
  @HiveField(0) String displayName;
  @HiveField(1) String host;
  @HiveField(2) String apiKey;
  @HiveField(3) LunaIndexerIcon icon;
  @HiveField(4) Map<String, String> headers;
}
```

**Features:**
- **Multiple Providers**: Support for various indexer types
- **Custom Icons**: Visual identification for indexers
- **API Key Management**: Secure storage of authentication
- **Header Support**: Custom authentication methods

### 4. LunaLog (typeId: 1)

Application logging and debugging information.

```dart
@HiveType(typeId: 1)
class LunaLog extends HiveObject {
  @HiveField(0) String className;      // Source class
  @HiveField(1) String methodName;     // Source method
  @HiveField(2) String message;        // Log message
  @HiveField(3) DateTime timestamp;    // When logged
  @HiveField(4) LunaLogType type;      // Log level
  @HiveField(5, defaultValue: '') String error;
  @HiveField(6, defaultValue: '') String stackTrace;
}
```

**Log Types:**
- `DEBUG`: Development information
- `INFO`: General information
- `WARNING`: Potential issues
- `ERROR`: Recoverable errors
- `CRITICAL`: Fatal errors

## Settings Management

### Global Settings (LunaSeaDatabase)

Core application settings stored in the `lunasea` box:

```dart
enum LunaSeaDatabase<T> {
  // UI/UX Settings
  THEME_AMOLED<bool>(false),
  THEME_AMOLED_BORDER<bool>(false),
  THEME_IMAGE_BACKGROUND_OPACITY<int>(20),
  
  // Behavior Settings
  ANDROID_BACK_OPENS_DRAWER<bool>(true),
  DRAWER_AUTOMATIC_MANAGE<bool>(true),
  USE_24_HOUR_TIME<bool>(false),
  
  // Feature Toggles
  QUICK_ACTIONS_LIDARR<bool>(false),
  QUICK_ACTIONS_RADARR<bool>(false),
  NETWORKING_TLS_VALIDATION<bool>(false),
  
  // Profile Management
  ENABLED_PROFILE<String>(LunaProfile.DEFAULT_PROFILE),
}
```

### Module-Specific Settings

Each module can define its own settings using the same pattern:

```dart
enum RadarrDatabase<T> {
  DEFAULT_SORTING_MOVIES<RadarrMoviesSorting>,
  DEFAULT_SORTING_RELEASES<RadarrReleasesSorting>,
  NAVIGATION_INDEX<int>(0),
}
```

## Data Access Patterns

### 1. Box Access

```dart
// Reading data
final profile = LunaBox.profiles.read('default');
final logs = LunaBox.logs.data.toList();

// Writing data
LunaBox.profiles.create('newProfile', newProfileData);
LunaBox.settings.update('THEME_AMOLED', true);

// Deleting data
LunaBox.profiles.delete('oldProfile');
```

### 2. Model Operations

```dart
// Profile management
final currentProfile = LunaProfile.current;
final allProfiles = LunaProfile.list;

// Creating new profile
final newProfile = LunaProfile();
newProfile.radarrEnabled = true;
newProfile.radarrHost = 'https://radarr.example.com';
LunaBox.profiles.create('production', newProfile);
```

### 3. Settings Access

```dart
// Reading settings
bool amoledTheme = LunaSeaDatabase.THEME_AMOLED.read();
String currentProfile = LunaSeaDatabase.ENABLED_PROFILE.read();

// Writing settings  
LunaSeaDatabase.THEME_AMOLED.update(true);
LunaSeaDatabase.ENABLED_PROFILE.update('production');
```

## Schema Migration

### Deprecated Models

LunaSea handles schema evolution through deprecated model classes:

```dart
// Old models maintained for migration compatibility
@HiveType(typeId: 2)
class _Deprecated02 extends HiveObject {}

@HiveType(typeId: 3)  
class _Deprecated03 extends HiveObject {}
```

**Migration Strategy:**
1. **Preserve Old Types**: Keep deprecated classes registered
2. **Gradual Migration**: Convert data during app updates
3. **Fallback Support**: Graceful handling of old data formats
4. **Version Tracking**: Build version tracking for migrations

### Adding New Fields

New fields are added with default values to maintain compatibility:

```dart
@HiveField(28, defaultValue: <String, String>{})
Map<String, String> newServiceHeaders;
```

## Performance Optimizations

### 1. Lazy Loading
- Models loaded on-demand
- Large collections paginated
- Cached frequently accessed data

### 2. Indexing Strategy
- Profile-based data organization
- Service-specific data isolation
- Efficient key-based lookups

### 3. Memory Management
- Automatic box closure
- Periodic cleanup of old logs
- Efficient data serialization

## Backup and Restore

### Export Format
```json
{
  "profiles": {
    "default": { /* profile data */ },
    "production": { /* profile data */ }
  },
  "indexers": [
    { /* indexer configurations */ }
  ],
  "externalModules": [
    { /* external module configs */ }
  ],
  "settings": {
    /* global settings */
  }
}
```

### Backup Process
1. **Export All Boxes**: Serialize all database boxes
2. **JSON Format**: Human-readable backup format
3. **Encryption Support**: Optional backup encryption
4. **Selective Restore**: Choose what to restore

## Security Considerations

### 1. API Key Storage
- **Local Only**: API keys never transmitted
- **Encrypted Storage**: Hive provides encryption options
- **Memory Safety**: Keys cleared from memory when possible

### 2. Data Isolation
- **Profile Separation**: Each profile isolated
- **Service Boundaries**: Services can't access each other's data
- **Local Storage**: No cloud storage of sensitive data

### 3. Logging Safety
- **No Secrets**: API keys excluded from logs
- **Sanitized Output**: Sensitive data masked
- **Local Logs**: Debug information stays local

## Database Initialization

```dart
Future<void> initialize() async {
  // Initialize Hive
  await Hive.initFlutter();
  
  // Register all adapters
  LunaSeaDatabase.values.forEach((db) => db.register());
  
  // Open all boxes
  await LunaBox.open();
  
  // Perform any necessary migrations
  await _performMigrations();
}
```

The database layer provides a robust foundation for LunaSea's configuration management and data persistence needs while maintaining performance and data integrity across all supported platforms.
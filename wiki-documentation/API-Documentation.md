# API Documentation

LunaSea integrates with multiple self-hosted media services through comprehensive API clients. This document covers the API architecture, service integrations, and communication patterns used throughout the application.

## API Architecture Overview

LunaSea implements a service-oriented API architecture where each supported service has:

1. **Dedicated API Client** - Service-specific HTTP client
2. **Command Handlers** - Organized API endpoints
3. **Data Models** - Strongly typed response/request models
4. **Type Definitions** - Enums and custom types
5. **Utilities** - Helper functions and validators

**Total API Files**: 467 Dart files across all service integrations

## API Client Structure

```
lib/api/{service}/
├── {service}.dart           # Main API client class
├── commands/                # Endpoint command handlers
│   ├── {resource}/         # Resource-specific commands
│   └── {resource}.dart     # Command exports
├── models/                  # Data models
│   ├── {entity}/           # Entity-specific models
│   └── {entity}.dart       # Model exports
├── types/                   # Type definitions
├── utilities.dart           # Helper functions
└── commands.dart            # All commands export
```

## Supported Service APIs

### 1. Radarr API (`lib/api/radarr/`)

**Purpose**: Movie collection management automation

**API Version**: v3+ only supported

**Command Categories**:
- **Movies** (`movie/`) - Movie management and metadata
- **Queue** (`queue/`) - Download queue operations
- **History** (`history/`) - Download and import history
- **Releases** (`release/`) - Release searching and management
- **System** (`system/`) - System status and configuration
- **Quality Profiles** (`quality_profile/`) - Quality management
- **Root Folders** (`root_folder/`) - Storage path management
- **Tags** (`tag/`) - Organizational tags
- **Import Lists** (`import_list/`) - Automated list imports
- **Manual Import** (`manual_import/`) - Manual file imports
- **Health Checks** (`health_check/`) - System health monitoring

**Key Models**:
```dart
// Core movie model
@JsonSerializable()
class RadarrMovie {
  int id;
  String title;
  String overview;
  DateTime? releaseDate;
  RadarrMovieStatus status;
  RadarrQualityProfile qualityProfile;
  List<RadarrTag> tags;
  RadarrMovieFile? movieFile;
}

// Quality profile model
@JsonSerializable() 
class RadarrQualityProfile {
  int id;
  String name;
  List<RadarrQualityItem> items;
  int cutoff;
}
```

**Authentication**: API Key based
```dart
RadarrAPI api = RadarrAPI(
  host: 'https://radarr.example.com',
  apiKey: 'your-api-key',
  headers: {'Custom-Header': 'value'},
);
```

### 2. Sonarr API (`lib/api/sonarr/`)

**Purpose**: TV series collection management

**API Version**: v3+ supported

**Command Categories**:
- **Series** (`series/`) - TV series management
- **Episodes** (`episode/`) - Individual episode management
- **Seasons** (`season/`) - Season-level operations
- **Queue** (`queue/`) - Download queue management
- **Calendar** (`calendar/`) - Episode release calendar
- **History** (`history/`) - Activity tracking
- **Wanted** (`wanted/`) - Missing episode management

**Key Models**:
```dart
@JsonSerializable()
class SonarrSeries {
  int id;
  String title;
  String overview;
  SonarrSeriesStatus status;
  List<SonarrSeason> seasons;
  SonarrQualityProfile qualityProfile;
  DateTime? nextAiring;
}

@JsonSerializable()
class SonarrEpisode {
  int id;
  String title;
  int seasonNumber;
  int episodeNumber;
  DateTime? airDate;
  bool monitored;
  SonarrEpisodeFile? episodeFile;
}
```

### 3. Lidarr API (`lib/api/lidarr/`)

**Purpose**: Music collection management

**Command Categories**:
- **Artists** (`artist/`) - Artist management
- **Albums** (`album/`) - Album management  
- **Tracks** (`track/`) - Individual track management
- **Search** (`search/`) - Music search functionality
- **Queue** (`queue/`) - Download management
- **History** (`history/`) - Activity logs

**Key Models**:
```dart
@JsonSerializable()
class LidarrArtist {
  int id;
  String artistName;
  String overview;
  List<LidarrAlbum> albums;
  LidarrArtistStatistics statistics;
  bool monitored;
}

@JsonSerializable()
class LidarrAlbum {
  int id;
  String title;
  String releaseDate;
  List<LidarrTrack> tracks;
  LidarrAlbumStatistics statistics;
}
```

### 4. NZBGet API (`lib/api/nzbget/`)

**Purpose**: Usenet download client management

**Command Categories**:
- **Queue** (`queue/`) - Download queue operations
- **History** (`history/`) - Download history
- **Statistics** (`statistics/`) - Performance metrics
- **Status** (`status/`) - System status
- **Config** (`config/`) - Configuration management

**Key Models**:
```dart
@JsonSerializable()
class NZBGetQueueItem {
  int nzbId;
  String nzbName;
  String status;
  int totalMB;
  int remainingMB;
  String category;
  int priority;
}

@JsonSerializable()
class NZBGetStatistics {
  int downloadRate;
  int remainingSize;
  bool downloadPaused;
  int totalArticles;
  int successArticles;
}
```

### 5. SABnzbd API (`lib/api/sabnzbd/`)

**Purpose**: Alternative Usenet download client

**Command Categories**:
- **Queue** (`queue/`) - Active downloads
- **History** (`history/`) - Completed downloads
- **Categories** (`categories/`) - Download categories
- **Statistics** (`statistics/`) - Usage statistics
- **Config** (`config/`) - Settings management

**Key Models**:
```dart
@JsonSerializable()
class SABnzbdQueueSlot {
  String nzoId;
  String filename;
  String status;
  String size;
  String timeLeft;
  String category;
  int priority;
}
```

### 6. Tautulli API (`lib/api/tautulli/`)

**Purpose**: Plex media server monitoring

**Command Categories**:
- **Activity** (`activity/`) - Real-time activity monitoring
- **History** (`history/`) - Playback history
- **Users** (`users/`) - User management and statistics
- **Libraries** (`libraries/`) - Library analytics
- **Media** (`media/`) - Media information
- **Server** (`server/`) - Server status and info

**Key Models**:
```dart
@JsonSerializable()
class TautulliSession {
  String sessionKey;
  String username;
  String title;
  String mediaType;
  String state;
  String platform;
  int progress;
  String transcodeDecision;
}

@JsonSerializable()
class TautulliUser {
  int userId;
  String username;
  String email;
  int totalPlays;
  DateTime lastPlayed;
  String platform;
}
```

### 7. Wake on LAN API (`lib/api/wake_on_lan/`)

**Purpose**: Remote system wake functionality

**Platform-specific Implementation**:
- **IO Platforms** (`wake_on_lan_io.dart`) - Native implementation
- **Web Platform** (`wake_on_lan_html.dart`) - Browser limitations
- **Stub Implementation** (`wake_on_lan_stub.dart`) - Fallback

**Usage**:
```dart
await WakeOnLAN.from(
  ipAddress: '192.168.1.100',
  macAddress: '00:11:22:33:44:55',
  port: 9,
);
```

## API Client Implementation Pattern

### 1. Base API Client Structure

```dart
/// Main API client class
class {Service}API {
  /// HTTP client instance
  final Dio httpClient;
  
  /// Command handlers
  final {Service}Command{Resource} {resource};
  
  /// Internal constructor
  {Service}API._internal({
    required this.httpClient,
    required this.{resource},
    // ... other command handlers
  });
  
  /// Public factory constructor
  factory {Service}API({
    required String host,
    required String apiKey,
    Map<String, String>? headers,
    bool validateSSL = true,
  }) {
    // Configure Dio client
    Dio dio = Dio(BaseOptions(
      baseUrl: host,
      headers: {
        'X-Api-Key': apiKey,
        'Content-Type': 'application/json',
        ...?headers,
      },
      validateStatus: (status) => status! < 400,
    ));
    
    return {Service}API._internal(
      httpClient: dio,
      {resource}: {Service}Command{Resource}(dio),
    );
  }
}
```

### 2. Command Handler Pattern

```dart
/// Resource-specific command handler
class {Service}Command{Resource} {
  final Dio _client;
  
  {Service}Command{Resource}(this._client);
  
  /// Get all resources
  Future<List<{Service}{Resource}>> getAll() async {
    final response = await _client.get('/api/v3/{resources}');
    return (response.data as List)
        .map((json) => {Service}{Resource}.fromJson(json))
        .toList();
  }
  
  /// Get single resource by ID
  Future<{Service}{Resource}> getById(int id) async {
    final response = await _client.get('/api/v3/{resources}/$id');
    return {Service}{Resource}.fromJson(response.data);
  }
  
  /// Create new resource
  Future<{Service}{Resource}> create({Service}{Resource} item) async {
    final response = await _client.post(
      '/api/v3/{resources}',
      data: item.toJson(),
    );
    return {Service}{Resource}.fromJson(response.data);
  }
  
  /// Update existing resource
  Future<{Service}{Resource}> update({Service}{Resource} item) async {
    final response = await _client.put(
      '/api/v3/{resources}/${item.id}',
      data: item.toJson(),
    );
    return {Service}{Resource}.fromJson(response.data);
  }
  
  /// Delete resource
  Future<void> delete(int id) async {
    await _client.delete('/api/v3/{resources}/$id');
  }
}
```

### 3. Model Implementation Pattern

```dart
/// Data model with JSON serialization
@JsonSerializable()
class {Service}{Resource} {
  final int id;
  final String name;
  final DateTime created;
  final {Service}{ResourceType} type;
  
  const {Service}{Resource}({
    required this.id,
    required this.name,
    required this.created,
    required this.type,
  });
  
  /// JSON serialization
  factory {Service}{Resource}.fromJson(Map<String, dynamic> json) =>
      _${Service}{Resource}FromJson(json);
  
  Map<String, dynamic> toJson() => _${Service}{Resource}ToJson(this);
  
  /// String representation
  @override
  String toString() => json.encode(toJson());
  
  /// Equality comparison
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is {Service}{Resource} &&
          runtimeType == other.runtimeType &&
          id == other.id;
  
  @override
  int get hashCode => id.hashCode;
}
```

## Error Handling Strategy

### 1. HTTP Error Handling

```dart
try {
  final result = await api.movies.getAll();
  return result;
} on DioException catch (e) {
  switch (e.type) {
    case DioExceptionType.connectionTimeout:
      throw {Service}ConnectionTimeoutException();
    case DioExceptionType.receiveTimeout:
      throw {Service}ReceiveTimeoutException();
    case DioExceptionType.badResponse:
      throw {Service}HTTPException(e.response?.statusCode ?? 0);
    default:
      throw {Service}UnknownException(e.message);
  }
}
```

### 2. Custom Exception Types

```dart
abstract class {Service}Exception implements Exception {
  final String message;
  const {Service}Exception(this.message);
}

class {Service}ConnectionTimeoutException extends {Service}Exception {
  const {Service}ConnectionTimeoutException() 
      : super('Connection to {Service} timed out');
}

class {Service}HTTPException extends {Service}Exception {
  final int statusCode;
  const {Service}HTTPException(this.statusCode)
      : super('{Service} returned HTTP $statusCode');
}
```

## API Configuration Management

### 1. Profile-based Configuration

```dart
class {Service}Helper {
  /// Get API instance from current profile
  static {Service}API? get api {
    final profile = LunaProfile.current;
    if (!profile.{service}Enabled) return null;
    
    return {Service}API(
      host: profile.{service}Host,
      apiKey: profile.{service}Key,
      headers: profile.{service}Headers,
      validateSSL: LunaSeaDatabase.NETWORKING_TLS_VALIDATION.read(),
    );
  }
  
  /// Test API connection
  static Future<bool> testConnection() async {
    try {
      final api = {Service}Helper.api;
      if (api == null) return false;
      
      await api.system.getStatus();
      return true;
    } catch (e) {
      return false;
    }
  }
}
```

### 2. Header Management

```dart
/// Custom headers for authentication/configuration
Map<String, String> buildHeaders(LunaProfile profile) {
  final headers = <String, String>{
    'X-Api-Key': profile.{service}Key,
    'Content-Type': 'application/json',
    'User-Agent': 'LunaSea/${LunaVersion.version}',
  };
  
  // Add custom headers from profile
  headers.addAll(profile.{service}Headers);
  
  return headers;
}
```

## Performance Optimizations

### 1. Request Caching

```dart
/// Cache frequently accessed data
class {Service}Cache {
  static final Map<String, dynamic> _cache = {};
  static const Duration _cacheTimeout = Duration(minutes: 5);
  
  static Future<T> cached<T>(
    String key,
    Future<T> Function() fetcher,
  ) async {
    final cached = _cache[key];
    if (cached != null && cached['expires'].isAfter(DateTime.now())) {
      return cached['data'] as T;
    }
    
    final data = await fetcher();
    _cache[key] = {
      'data': data,
      'expires': DateTime.now().add(_cacheTimeout),
    };
    
    return data;
  }
}
```

### 2. Request Throttling

```dart
/// Prevent API spam
class {Service}RateLimit {
  static final Map<String, DateTime> _lastRequest = {};
  static const Duration _throttle = Duration(milliseconds: 100);
  
  static Future<void> throttle(String endpoint) async {
    final last = _lastRequest[endpoint];
    if (last != null) {
      final elapsed = DateTime.now().difference(last);
      if (elapsed < _throttle) {
        await Future.delayed(_throttle - elapsed);
      }
    }
    _lastRequest[endpoint] = DateTime.now();
  }
}
```

## Testing Strategy

While LunaSea currently has no tests, the API layer would benefit from:

### 1. Unit Tests
```dart
// Mock API responses for testing
test('{Service} API should fetch movies', () async {
  final mockDio = MockDio();
  when(mockDio.get('/api/v3/movie'))
      .thenAnswer((_) async => Response(data: mockMovieData));
  
  final api = {Service}API.from(mockDio);
  final movies = await api.movies.getAll();
  
  expect(movies, hasLength(2));
  expect(movies.first.title, equals('Test Movie'));
});
```

### 2. Integration Tests
```dart
// Test against real API endpoints
test('{Service} integration test', () async {
  final api = {Service}API(
    host: testHost,
    apiKey: testApiKey,
  );
  
  final status = await api.system.getStatus();
  expect(status.isOnline, isTrue);
});
```

This comprehensive API architecture enables LunaSea to provide consistent, reliable communication with all supported self-hosted media services while maintaining type safety and error resilience.
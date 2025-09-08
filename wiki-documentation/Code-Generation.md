# Code Generation

LunaSea heavily relies on code generation to maintain type safety, reduce boilerplate code, and ensure consistency across the codebase. This document covers the code generation pipeline, tools used, and best practices.

## Code Generation Overview

LunaSea uses code generation for:

1. **JSON Serialization** - Converting between Dart objects and JSON
2. **Database Models** - Hive adapter generation for local storage
3. **Environment Configuration** - Build-time environment variables
4. **Asset Management** - Type-safe asset references
5. **Localization** - Translation file compilation

## Generation Pipeline

### Complete Generation
```bash
npm run generate  # Runs all generation steps in sequence
```

### Individual Steps
```bash
npm run generate:environment     # Environment config
npm run generate:assets          # Asset references  
npm run generate:build_runner    # JSON/Hive generation
npm run generate:localization    # Translation files
```

### Watch Mode
```bash
npm run generate:build_runner:watch  # Auto-regenerate on changes
```

## 1. JSON Serialization (`json_serializable`)

### Purpose
Automatically generates `toJson()` and `fromJson()` methods for data models.

### Implementation
```dart
import 'package:json_annotation/json_annotation.dart';

part 'movie.g.dart';  // Generated file

@JsonSerializable()
class RadarrMovie {
  final int id;
  final String title;
  
  @JsonKey(name: 'release_date')
  final DateTime? releaseDate;
  
  const RadarrMovie({
    required this.id,
    required this.title,
    this.releaseDate,
  });
  
  factory RadarrMovie.fromJson(Map<String, dynamic> json) =>
      _$RadarrMovieFromJson(json);
  
  Map<String, dynamic> toJson() => _$RadarrMovieToJson(this);
}
```

### Generated Output
```dart
// movie.g.dart (generated)
RadarrMovie _$RadarrMovieFromJson(Map<String, dynamic> json) => RadarrMovie(
      id: json['id'] as int,
      title: json['title'] as String,
      releaseDate: json['release_date'] == null
          ? null
          : DateTime.parse(json['release_date'] as String),
    );

Map<String, dynamic> _$RadarrMovieToJson(RadarrMovie instance) =>
    <String, dynamic>{
      'id': instance.id,
      'title': instance.title,
      'release_date': instance.releaseDate?.toIso8601String(),
    };
```

### Configuration
```yaml
# build.yaml
targets:
  $default:
    builders:
      json_serializable:
        options:
          explicit_to_json: true
          include_if_null: false
```

## 2. Hive Database Generation (`hive_generator`)

### Purpose
Generates type adapters for Hive database objects.

### Implementation
```dart
import 'package:hive/hive.dart';
import 'package:json_annotation/json_annotation.dart';

part 'profile.g.dart';

@JsonSerializable()
@HiveType(typeId: 0, adapterName: 'LunaProfileAdapter')
class LunaProfile extends HiveObject {
  @JsonKey()
  @HiveField(0, defaultValue: false)
  bool lidarrEnabled;
  
  @JsonKey()
  @HiveField(1, defaultValue: '')
  String lidarrHost;
  
  LunaProfile({
    this.lidarrEnabled = false,
    this.lidarrHost = '',
  });
}
```

### Generated Output
```dart
// profile.g.dart (generated)
class LunaProfileAdapter extends TypeAdapter<LunaProfile> {
  @override
  final int typeId = 0;

  @override
  LunaProfile read(BinaryReader reader) {
    final numOfFields = reader.readByte();
    final fields = <int, dynamic>{
      for (int i = 0; i < numOfFields; i++) reader.readByte(): reader.read(),
    };
    return LunaProfile(
      lidarrEnabled: fields[0] ?? false,
      lidarrHost: fields[1] ?? '',
    );
  }

  @override
  void write(BinaryWriter writer, LunaProfile obj) {
    writer
      ..writeByte(2)
      ..writeByte(0)
      ..write(obj.lidarrEnabled)
      ..writeByte(1)
      ..write(obj.lidarrHost);
  }
}
```

## 3. Environment Configuration (`environment_config`)

### Purpose
Generates environment configuration from YAML definition.

### Configuration File
```yaml
# environment_config.yaml
environment_config:
  default_hostname: lunasea.app
  default_user_agent: LunaSea
  contact_email: hello@lunasea.app
  
  image_background_opacity_max: 100
  image_background_opacity_min: 0
  image_background_opacity_default: 20
  
  backup_export_file_extension: lunaSeaBackup
  backup_export_encryption_key_length: 32
```

### Generated Output
```dart
// lib/system/environment.dart (generated)
class LunaEnvironment {
  static const String DEFAULT_HOSTNAME = 'lunasea.app';
  static const String DEFAULT_USER_AGENT = 'LunaSea';
  static const String CONTACT_EMAIL = 'hello@lunasea.app';
  
  static const int IMAGE_BACKGROUND_OPACITY_MAX = 100;
  static const int IMAGE_BACKGROUND_OPACITY_MIN = 0;
  static const int IMAGE_BACKGROUND_OPACITY_DEFAULT = 20;
  
  static const String BACKUP_EXPORT_FILE_EXTENSION = 'lunaSeaBackup';
  static const int BACKUP_EXPORT_ENCRYPTION_KEY_LENGTH = 32;
}
```

### Usage
```dart
// Type-safe environment access
final email = LunaEnvironment.CONTACT_EMAIL;
final maxOpacity = LunaEnvironment.IMAGE_BACKGROUND_OPACITY_MAX;
```

## 4. Asset Generation (Spider)

### Purpose
Generates type-safe references to assets (images, fonts, etc.).

### Configuration
```yaml
# spider.yaml
spider:
  # Asset generation config
  groups:
    - path: assets/images
      class_name: LunaAssets
      package: assets/images
    - path: assets/localization
      class_name: LunaLocalization
      package: assets/localization
```

### Generated Output
```dart
// Generated asset references
class LunaAssets {
  static const String brandingFull = 'assets/images/branding_full.png';
  static const String brandingLogo = 'assets/images/branding_logo.png';
  static const String brandingIcon = 'assets/images/branding_icon.png';
}
```

### Usage
```dart
// Type-safe asset access
Image.asset(LunaAssets.brandingLogo)
```

## 5. Localization Generation

### Purpose
Compiles translation files and generates type-safe translation keys.

### Source Files
```
assets/localization/
├── en.json          # English (base)
├── es.json          # Spanish
├── fr.json          # French
└── ...
```

### Base Translation File
```json
// assets/localization/en.json
{
  "lunasea": "LunaSea",
  "modules": {
    "radarr": "Radarr",
    "sonarr": "Sonarr"
  },
  "buttons": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete"
  }
}
```

### Generation Script
```dart
// scripts/generate_localization.dart
void main() {
  // Load base translation file
  final baseFile = File('assets/localization/en.json');
  final baseTranslations = json.decode(baseFile.readAsStringSync());
  
  // Generate type-safe keys
  final output = generateKeys(baseTranslations);
  
  // Write generated file
  File('lib/system/localization_keys.dart').writeAsStringSync(output);
}
```

### Generated Output
```dart
// lib/system/localization_keys.dart (generated)
class LunaTranslationKeys {
  static const String lunasea = 'lunasea';
  
  static const String modulesRadarr = 'modules.radarr';
  static const String modulesSonarr = 'modules.sonarr';
  
  static const String buttonsSave = 'buttons.save';
  static const String buttonsCancel = 'buttons.cancel';
  static const String buttonsDelete = 'buttons.delete';
}
```

### Usage
```dart
// Type-safe translations
Text(LunaTranslationKeys.buttonsSave.tr())
```

## Build Runner Configuration

### pubspec.yaml Dependencies
```yaml
dev_dependencies:
  build_runner: ^2.4.6
  json_serializable: ^6.9.0
  hive_generator: ^2.0.1
  environment_config: ^3.2.0
```

### build.yaml Configuration
```yaml
targets:
  $default:
    builders:
      json_serializable:
        options:
          explicit_to_json: true
          include_if_null: false
          field_rename: snake
          checked: true
      
      hive_generator:
        options:
          auto_register: true
```

## Generated File Management

### Git Ignore Patterns
```gitignore
# Never commit generated files to git
**/*.g.dart
lib/system/environment.dart
assets/localization/*.json
```

### File Naming Conventions
- `*.g.dart` - Build runner generated files
- `environment.dart` - Environment configuration
- `*_keys.dart` - Generated key classes

### Cleanup
```bash
# Remove all generated files
find . -name "*.g.dart" -delete
rm lib/system/environment.dart
```

## Development Workflow

### 1. Making Model Changes
```bash
# 1. Modify model class
# 2. Regenerate code
npm run generate:build_runner

# 3. Verify compilation
flutter analyze
```

### 2. Adding New Assets
```bash
# 1. Add assets to assets/ directory
# 2. Regenerate asset references
npm run generate:assets

# 3. Update pubspec.yaml if needed
```

### 3. Environment Changes
```bash
# 1. Modify environment_config.yaml
# 2. Regenerate environment
npm run generate:environment

# 3. Update code using new constants
```

### 4. Translation Updates
```bash
# 1. Update translation JSON files
# 2. Regenerate localization
npm run generate:localization

# 3. Update UI to use new keys
```

## Error Handling

### Common Generation Errors

**Missing part directive**:
```dart
// Add this to files using code generation
part 'filename.g.dart';
```

**Conflicting outputs**:
```bash
# Force regeneration
npm run generate:build_runner -- --delete-conflicting-outputs
```

**Missing dependencies**:
```bash
# Ensure all dependencies are installed
flutter pub get
```

### Debugging Generation

**Verbose output**:
```bash
dart run build_runner build --verbose
```

**Watch for specific files**:
```bash
dart run build_runner watch --build-filter="lib/models/**"
```

## Best Practices

### 1. Model Design
- Always use `@JsonSerializable()` for API models
- Provide default values for Hive fields
- Use nullable types appropriately
- Document complex serialization with `@JsonKey()`

### 2. Asset Management
- Organize assets in logical directories
- Use consistent naming conventions
- Generate references after adding new assets

### 3. Environment Configuration
- Keep environment config in version control
- Use descriptive constant names
- Group related constants logically

### 4. Translation Management
- Maintain English as the base language
- Use nested JSON structure for organization
- Generate keys after translation changes

### 5. Build Integration
- Always run generation before building
- Include generation in CI/CD pipeline
- Use watch mode during active development

The code generation pipeline is essential to LunaSea's architecture, providing type safety, consistency, and reducing manual maintenance overhead across the entire codebase.
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a **mono-repository** containing the archived LunaSea project with multiple components:

- `lunasea/` - Main Flutter application (primary development target)
- `lunasea-cloud-functions/` - Firebase cloud functions (Node.js)
- `lunasea-notification-service/` - Notification service (Node.js/TypeScript)
- `lunasea-docs/` - Documentation and release notes
- `wiki-documentation/` - GitHub wiki documentation

**CRITICAL**: Always work from the `lunasea/` subdirectory for the main application. This is where all Flutter development happens.

## Essential Commands

### Setup and Prerequisites
```bash
cd lunasea/  # Always start here for main app development

# Required setup sequence (run in order):
npm install                    # Install Node.js build tools
flutter pub get               # Install Flutter dependencies
npm run generate              # Generate all required files (REQUIRED before building)
```

### Code Generation (Required Before Building)
```bash
npm run generate                          # Generates ALL required files
npm run generate:build_runner            # JSON serialization, Hive adapters
npm run generate:environment             # Environment configuration
npm run generate:localization            # Language files
npm run generate:assets                  # Asset references
npm run generate:build_runner:watch      # Watch mode for development
```

### Development
```bash
flutter run --debug           # Hot reload development
npm run profile              # Profile mode with debugging
flutter analyze              # Lint and static analysis
```

### Building
```bash
# Platform-specific release builds:
npm run build:android        # Android APK
npm run build:ios           # iOS IPA (macOS only)
npm run build:windows       # Windows executable
npm run build:macos         # macOS app (macOS only)
npm run build:linux         # Linux executable
npm run build:web           # Web application
```

### Other Subprojects

**Notification Service** (`lunasea-notification-service/`):
```bash
npm start                    # Development server
npm run build               # TypeScript compilation
npm run lint                # ESLint
```

**Cloud Functions** (`lunasea-cloud-functions/functions/`):
```bash
npm run build               # TypeScript build
firebase emulators:start    # Local Firebase emulator
```

### Commit Management
```bash
npm run commit              # Interactive commit with conventional commit format
```

## Architecture Overview

### Flutter Application Structure (`lunasea/lib/`)

**Core Systems**:
- `main.dart` - Application bootstrap with recovery mode support
- `database/` - Hive local database with type adapters
- `router/` - GoRouter-based navigation system
- `system/` - Platform utilities, window management, networking
- `api/` - HTTP clients and data models using Retrofit/Dio

**Modular Service Integrations**:
- `modules/radarr/` - Movie management (Radarr API)
- `modules/sonarr/` - TV series management (Sonarr API)
- `modules/lidarr/` - Music management (Lidarr API)
- `modules/tautulli/` - Plex monitoring
- `modules/nzbget/` - Download client integration
- `modules/sabnzbd/` - Alternative download client
- `modules/search/` - Unified search across services
- `modules/settings/` - Application configuration
- `modules/dashboard/` - Main dashboard UI

**Shared Components**:
- `widgets/` - Reusable UI components
- `utils/` - Helper functions and utilities
- `types/` - Custom type definitions

### Key Dependencies and Generated Files

**Critical Generated Files** (never edit manually):
- `lib/**/*.g.dart` - JSON serialization, Hive adapters (build_runner)
- `lib/system/environment.dart` - Environment configuration
- `assets/localization/*.json` - Localization files

**Major Dependencies**:
- `hive` + `hive_flutter` - Local database
- `dio` + `retrofit` - HTTP client with code generation
- `go_router` - Navigation
- `easy_localization` - Internationalization
- `json_annotation` + `json_serializable` - JSON handling

## Development Workflow

1. **Before Making Changes**:
   - Ensure you're in `lunasea/` directory
   - Run `npm run generate` to ensure generated files are current
   - Run `flutter analyze` to check for existing issues

2. **During Development**:
   - Use `npm run generate:build_runner:watch` for automatic code generation
   - Use `flutter run --debug` for hot reload testing

3. **After Making Changes**:
   - Run `flutter analyze` to check for linting issues
   - Regenerate code if you modified JSON models or database types
   - Use `npm run commit` for proper commit formatting

## Important Notes

- **Project Status**: This is an archived project and no longer actively maintained
- **Code Generation**: Always run code generation before building - the app won't compile without it
- **Platform Requirements**: Different platforms have specific SDK requirements (see Copilot instructions for details)
- **Mono-repo Structure**: Each subdirectory is a separate project with its own build system
- **Commit Format**: Uses conventional commits enforced by Commitlint
- **Flutter Version**: Requires Flutter 3.0+ and Dart SDK included with Flutter

## Common Issues

- **Build failures**: Usually caused by missing code generation - run `npm run generate`
- **Import errors**: Generated files missing - check `*.g.dart` files exist
- **Platform build issues**: Ensure platform-specific SDKs are installed (Android Studio, Xcode, etc.)
- **Husky hook failures**: Ensure you're in the correct directory with proper git setup
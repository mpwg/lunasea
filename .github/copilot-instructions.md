# LunaSea Copilot Instructions

## Repository Overview

LunaSea is a Flutter-based self-hosted media controller application that connects to various media automation services (Radarr, Sonarr, Lidarr, NZBGet, SABnzbd, Tautulli, etc.). This is a **mono-repository** containing multiple projects:

- `lunasea/` - Main Flutter application (Dart/Flutter 3.0+)
- `lunasea-cloud-functions/` - Firebase cloud functions (Node.js)
- `lunasea-docs/` - Documentation site
- `lunasea-notification-service/` - Notification service

**Note: The project is archived and no longer actively maintained.**

## High-Level Project Information

- **Type**: Cross-platform mobile and desktop application
- **Language**: Dart (Flutter 3.0+)
- **Platforms**: Android, iOS, Windows, macOS, Linux, Web
- **Size**: ~50MB codebase, ~200+ Dart files
- **Framework**: Flutter with custom UI components and modular architecture
- **Version**: 11.0.0+1

## Build & Development Instructions

### Prerequisites (Required)

**IMPORTANT**: Always work from the `lunasea/` subdirectory, not the repository root. The main application is in the mono-repo's `lunasea/` folder.

```bash
cd lunasea/  # Always start here
```

#### Required Tools
- **Flutter SDK 3.0+**: Use `flutter --version` to verify
- **Dart SDK**: Included with Flutter
- **Node.js 20+**: For npm scripts (`node --version`)
- **npm 10+**: For package management (`npm --version`)

#### Platform-specific Tools
- **Android**: Java 17, Android SDK (via Android Studio)
- **iOS/macOS**: Xcode (latest stable), Ruby with Bundler
- **Linux**: clang, cmake, ninja-build, pkg-config, libgtk-3-dev, liblzma-dev
- **Windows**: Visual Studio with C++ tools

### Environment Setup

**CRITICAL**: Run setup commands in correct order to avoid failures:

```bash
cd lunasea/

# 1. Install Node dependencies first
npm install

# 2. Get Flutter dependencies
flutter pub get

# 3. Generate required files (ALWAYS run before building)
npm run generate
```

**Common Issues & Workarounds**:
- If `npm install` fails with Husky errors in mono-repo, run from repository root first, then retry in `lunasea/`
- If Flutter not found, ensure Flutter is in PATH and run `flutter doctor`
- Code generation must complete successfully before any builds

### Code Generation (Required Before Building)

**ALWAYS run code generation before building or making changes**:

```bash
npm run generate  # Generates all required files

# Individual generation commands (if needed):
npm run generate:environment     # Environment config
npm run generate:build_runner    # JSON serialization, etc.
npm run generate:localization    # Language files
npm run generate:assets          # Asset references
```

**Watch mode for development**:
```bash
npm run generate:build_runner:watch  # Auto-regenerates on file changes
```

### Building

#### Development
```bash
flutter run --debug  # Hot reload enabled
npm run profile      # Profile mode with debugging
```

#### Release Builds (Platform-specific)
```bash
npm run build:android   # APK for Android
npm run build:ios       # IPA for iOS (macOS only)
npm run build:windows   # Windows executable
npm run build:macos     # macOS app (macOS only)
npm run build:linux     # Linux executable
npm run build:web       # Web application
```

**Build Times**: Expect 5-15 minutes for release builds depending on platform.

#### Packaging Formats
- **Android**: APK and AAB (Android App Bundle) for Google Play
- **iOS**: IPA for App Store distribution
- **Linux**: AppImage and Snap packages
- **Windows**: MSIX package
- **macOS**: DMG installer and App Store package

### Linting & Code Quality

```bash
flutter analyze                    # Dart analyzer (uses analysis_options.yaml)
```

**Linting Configuration**: `analysis_options.yaml` with custom rules
- Extends `package:flutter_lints/flutter.yaml`
- Excludes generated files (`**/*.g.dart`)
- Custom rule overrides for project style

### Testing

**Note**: This repository currently has no test files. When adding tests:
- Place test files in `test/` directory
- Use naming convention `*_test.dart`
- Run with `flutter test`

### Commit Guidelines

**REQUIRED**: This project uses conventional commits with Commitlint:

```bash
npm run commit  # Interactive commit with Commitizen
```

**Commit Types**: `chore`, `docs`, `feat`, `fix`, `refactor`, `release`

## Project Architecture & Layout

### Main Application Structure (`lunasea/`)

```
lunasea/
├── lib/                          # Main source code
│   ├── main.dart                # Application entry point
│   ├── core.dart                # Core exports (deprecated)
│   ├── modules.dart             # Module exports
│   ├── vendor.dart              # Vendor/third-party exports
│   ├── api/                     # API clients and models
│   ├── database/                # Local database (Hive)
│   ├── modules/                 # Feature modules (Radarr, Sonarr, etc.)
│   ├── router/                  # Navigation and routing
│   ├── system/                  # System utilities
│   ├── types/                   # Custom type definitions
│   ├── utils/                   # Utility functions
│   └── widgets/                 # Reusable UI components
├── assets/                      # Images, fonts, localization
├── android/                     # Android-specific code
├── ios/                         # iOS-specific code
├── windows/                     # Windows-specific code
├── macos/                       # macOS-specific code
├── linux/                       # Linux-specific code
├── web/                         # Web-specific code
├── scripts/                     # Build and utility scripts
└── .github/                     # CI/CD workflows
```

### Key Configuration Files

- `pubspec.yaml` - Flutter dependencies and app metadata
- `package.json` - npm scripts for build automation
- `analysis_options.yaml` - Dart linting rules
- `environment_config.yaml` - Environment variable configuration
- `.github/workflows/` - Comprehensive CI/CD for all platforms

### Modular Architecture

The app uses a modular architecture where each service integration is a separate module:
- `lib/modules/radarr/` - Radarr movie management
- `lib/modules/sonarr/` - Sonarr TV series management
- `lib/modules/lidarr/` - Lidarr music management
- `lib/modules/tautulli/` - Plex monitoring
- `lib/modules/nzbget/` - Download client
- `lib/modules/settings/` - App configuration

### CI/CD Pipeline

**GitHub Workflows** (located in `.github/workflows/`):
- `build.yml` - Main build pipeline for all platforms
- `prepare.yml` - Generates build artifacts and environment
- `build_*.yml` - Platform-specific build workflows

**Build Process**:
1. Prepare phase: Generate environment config, localization, build runner files
2. Parallel platform builds: Android, iOS, Windows, macOS, Linux, Web
3. Code signing and packaging for distribution

### Dependencies & Generated Files

**Important Generated Files** (never edit manually):
- `lib/**/*.g.dart` - Generated by build_runner (JSON serialization, etc.)
- `lib/system/environment.dart` - Generated environment configuration
- `assets/localization/*.json` - Generated localization files

**Key Dependencies**:
- `flutter` - UI framework
- `hive` + `hive_flutter` - Local database
- `dio` - HTTP client
- `go_router` - Navigation
- `provider` - State management
- `json_annotation` + `json_serializable` - JSON handling
- `spider` - Asset code generation tool
- `fastlane` - iOS/Android deployment automation

**Package Management Tools**:
- Flutter uses `pubspec.yaml` for Dart/Flutter dependencies
- npm uses `package.json` for Node.js build tools
- Ruby Bundler uses `Gemfile` for iOS/Android native tooling

## Validation Steps

### Before Making Changes
1. Ensure you're in `lunasea/` directory
2. Run `flutter doctor` to verify Flutter installation
3. Run `npm run generate` to ensure all generated files are current
4. Run `flutter analyze` to check for existing issues

### After Making Changes
1. Run `flutter analyze` to check for linting issues
2. Run code generation if you modified any JSON models: `npm run generate:build_runner`
3. Test compilation: `flutter run --debug` or platform-specific build command
4. Use `npm run commit` for proper commit message formatting

### Manual Verification
- For UI changes: Run the app and navigate to modified screens
- For API changes: Test with actual service connections
- For build changes: Test release build for target platform

## Common Issues & Troubleshooting

### Build Failures
- **"Flutter not found"**: Ensure Flutter is installed and in PATH
- **"Package not found" errors**: Run `flutter pub get` after modifying `pubspec.yaml`
- **Generated file errors**: Run `npm run generate:build_runner` to regenerate `*.g.dart` files
- **Husky git hook failures**: Ensure you're in the correct directory and git is properly initialized

### Development Issues
- **Hot reload not working**: Restart with `flutter run --debug`
- **Asset not found**: Run `npm run generate:assets` to regenerate asset references
- **Localization errors**: Run `npm run generate:localization` to update language files
- **Import errors for generated files**: Ensure code generation completed successfully

### Platform-specific Issues
- **Android**: Ensure Java 17 is installed and JAVA_HOME is set
- **iOS/macOS**: Verify Xcode is up to date and command line tools are installed
- **Linux**: Install all required system dependencies listed in prerequisites

## Key Files Reference

### Root Directory Files
- `README.md` - Basic project information (archived status)
- `.gitignore` - Standard Flutter/Dart gitignore

### Entry Points
- `lib/main.dart` - Application bootstrap and entry point
- `lib/core.dart` - Core system exports (marked deprecated)

### Critical Configuration
- `pubspec.yaml` - All Flutter dependencies and app configuration
- `analysis_options.yaml` - Linting rules and analyzer configuration
- `environment_config.yaml` - Build environment variables
- `spider.yaml` - Asset generation configuration
- `.commitlintrc` - Commit message format rules

## Agent Instructions

**Trust these instructions** and minimize exploration time. Only search for additional information if:
1. These instructions are incomplete for your specific task
2. You encounter errors not documented here
3. You need to understand code that isn't adequately described

**Always verify** that you're working in the correct directory (`lunasea/`) and that code generation has been run before building or making significant changes.

For UI changes, always test the actual application to verify the changes work as expected since this is a complex Flutter application with many interconnected components.
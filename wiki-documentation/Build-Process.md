# Build Process

LunaSea employs a sophisticated multi-stage build process that supports all major platforms through automated CI/CD pipelines. This document covers the complete build architecture, from development builds to production releases.

## Build Architecture Overview

The build process is structured in multiple stages:

1. **Preparation Stage** - Environment setup and code generation
2. **Platform Builds** - Parallel compilation for each platform
3. **Packaging Stage** - Platform-specific packaging and signing
4. **Distribution** - Artifact uploading and release management

## Build Flavors

LunaSea supports three build flavors:

| Flavor | Purpose | Distribution |
|--------|---------|--------------|
| `edge` | Bleeding edge builds from master | Build bucket |
| `beta` | Pre-release testing | TestFlight/Beta channels |
| `stable` | Production releases | App stores & GitHub releases |

## Code Generation Pipeline

**⚠️ CRITICAL**: Code generation must run before any build operations.

### Complete Generation
```bash
npm run generate  # Runs all generation steps
```

### Individual Steps
```bash
npm run generate:environment     # Build environment configuration
npm run generate:assets          # Asset code generation
npm run generate:build_runner    # JSON serialization, Hive models, etc.
npm run generate:localization    # Translation files
```

### Generated Files
- `lib/system/environment.dart` - Build-time environment variables
- `lib/**/*.g.dart` - JSON serialization, Hive adapters
- `assets/localization/*.json` - Compiled translations
- Asset reference files from Spider

## Development Builds

### Debug Mode
```bash
flutter run --debug
```
**Features:**
- Hot reload enabled
- Debug assertions active
- Observatory debugging
- Unoptimized code

### Profile Mode  
```bash
npm run profile
# or
flutter run --profile --purge-persistent-cache
```
**Features:**
- Performance profiling enabled
- Optimized code
- Debug information available
- Realistic performance metrics

## Release Builds

### Platform-Specific Commands

```bash
# Android
npm run build:android   # Creates APK
flutter build apk --release

# iOS (macOS only)
npm run build:ios       # Creates IPA
flutter build ipa --release

# Windows
npm run build:windows   # Creates executable
flutter build windows --release

# macOS (macOS only)  
npm run build:macos     # Creates app bundle
flutter build macos --release

# Linux
npm run build:linux     # Creates executable
flutter build linux --release

# Web
npm run build:web       # Creates web bundle
flutter build web --release
```

### Build Times
- **Debug**: Instant (hot reload)
- **Profile**: 2-5 minutes
- **Release**: 5-15 minutes per platform

## CI/CD Pipeline Architecture

### 1. Preparation Workflow (`prepare.yml`)

**Triggers**: Called by main build workflow

**Responsibilities**:
- Environment setup
- Build number generation
- Flavor determination
- Code generation
- Artifact preparation

**Outputs**:
```yaml
build-flavor: edge|beta|stable
build-number: Commit count
build-version: Package.json version
build-title: Human readable title
build-motd: Message of the day
```

### 2. Main Build Workflow (`build.yml`)

**Triggers**:
- Push to `master` branch
- Manual workflow dispatch

**Strategy**: Parallel builds for all platforms

```yaml
jobs:
  prepare:           # Environment preparation
  build-android:     # Android APK/AAB
  build-ios:         # iOS IPA  
  build-windows:     # Windows MSIX
  build-macos:       # macOS DMG
  build-linux:       # Linux AppImage/Snap
  build-web:         # Web bundle
```

### 3. Platform-Specific Workflows

Each platform has a dedicated workflow file:

- `build_android.yml` - Android builds (APK + AAB)
- `build_ios.yml` - iOS builds (IPA)
- `build_windows.yml` - Windows builds (MSIX)
- `build_macos.yml` - macOS builds (DMG)
- `build_linux.yml` - Linux builds (AppImage + Snap)
- `build_web.yml` - Web builds

## Platform Build Details

### Android Build

**Packages Generated**:
- **APK**: Direct installation package
- **AAB**: Android App Bundle for Play Store

**Build Process**:
```bash
# Using Fastlane
cd android
bundle exec fastlane build_aab build_number:$BUILD_NUMBER
bundle exec fastlane build_apk build_number:$BUILD_NUMBER
```

**Requirements**:
- Java 17
- Android SDK
- Signing keys (JKS format)
- Key properties file

**Signing**:
- Release builds signed with production keys
- Debug builds signed with debug keys
- Key management through GitHub secrets

### iOS Build (macOS only)

**Packages Generated**:
- **IPA**: iOS App Store package

**Build Process**:
```bash
# Using Fastlane + Match
cd ios
bundle exec fastlane build_ipa build_number:$BUILD_NUMBER
```

**Requirements**:
- Xcode (latest stable)
- Apple Developer account
- Provisioning profiles
- Code signing certificates

**Code Signing**:
- Fastlane Match for certificate management
- Automatic provisioning profile management
- App Store distribution certificates

### Windows Build

**Packages Generated**:
- **MSIX**: Windows Store package

**Build Process**:
```bash
flutter build windows --release
# Package with MSIX tools
```

**Requirements**:
- Visual Studio with C++ tools
- Windows SDK
- Code signing certificate (optional)

**Configuration**:
```yaml
# pubspec.yaml - msix_config
msix_config:
  architecture: x64
  capabilities: internetClientServer,privateNetworkClientServer
  display_name: LunaSea
  identity_name: app.lunasea.lunasea
  output_name: lunasea-windows-amd64
```

### macOS Build (macOS only)

**Packages Generated**:
- **DMG**: macOS installer
- **App Bundle**: Direct application

**Build Process**:
```bash
# Using Fastlane
cd macos  
bundle exec fastlane build_dmg build_number:$BUILD_NUMBER
```

**Requirements**:
- Xcode
- macOS Developer account
- Code signing certificates
- Notarization setup

### Linux Build

**Packages Generated**:
- **AppImage**: Portable application
- **Snap**: Universal Linux package

**Build Process**:
```bash
flutter build linux --release
# Package with AppImage tools
# Build snap package
```

**Requirements**:
- Flutter Linux dependencies
- AppImage tools
- Snapcraft

### Web Build

**Packages Generated**:
- **Web Bundle**: Static web application

**Build Process**:
```bash
flutter build web --release --web-renderer canvaskit
```

**Features**:
- CanvasKit renderer for better performance
- Progressive Web App (PWA) support
- Service worker for offline functionality

## Build Environment Setup

### GitHub Actions Environment

**Shared Setup Action** (`prepare_for_build/action.yml`):
```yaml
- name: Setup Flutter
  uses: subosito/flutter-action@v2
  with:
    flutter-version: ${{ env.FLUTTER_VERSION }}
    channel: stable

- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: 20

- name: Install Dependencies
  run: |
    cd lunasea
    npm install
    flutter pub get

- name: Generate Code
  run: |
    cd lunasea
    npm run generate
```

### Local Development Environment

**Prerequisites Check**:
```bash
flutter doctor --verbose    # Verify Flutter installation
npm --version              # Verify Node.js/npm
```

**Environment Setup**:
```bash
cd lunasea/
npm install                # Install Node dependencies
flutter pub get           # Install Flutter dependencies  
npm run generate          # Generate required code
```

## Build Artifacts

### Artifact Organization

```
output/
├── android/
│   ├── lunasea-android.apk
│   └── lunasea-android.aab
├── ios/
│   └── lunasea-ios.ipa
├── windows/
│   └── lunasea-windows-amd64.msix
├── macos/
│   └── lunasea-macos.dmg
├── linux/
│   ├── lunasea-linux.AppImage
│   └── lunasea-linux.snap
└── web/
    └── lunasea-web.tar.gz
```

### Artifact Distribution

**Build Bucket**: [builds.lunasea.app](https://builds.lunasea.app)
- All build flavors
- Platform-specific downloads
- Build history

**GitHub Releases**: [GitHub Releases](https://github.com/JagandeepBrar/LunaSea/releases)
- Stable releases only
- All platforms
- Release notes

**App Stores**:
- Google Play Store (Android)
- Apple App Store (iOS)
- Microsoft Store (Windows)
- Snap Store (Linux)

## Version Management

### Version Strategy
```json
// package.json
{
  "version": "11.0.0"
}
```

### Build Number Generation
```bash
# Build number = Git commit count
BUILD_NUMBER=$(git rev-list HEAD --count)
```

### Version Synchronization
```bash
# Update pubspec.yaml from package.json
npm run version:pubspec

# Update snapcraft.yaml
npm run version:snapcraft
```

## Code Signing & Security

### Android Signing
- Production keystore stored in GitHub secrets
- Separate keys for different build flavors
- Automatic signing during CI/CD

### iOS Signing
- Fastlane Match for certificate management
- Automatic provisioning profile handling
- App Store distribution certificates

### Windows Signing
- Optional code signing certificates
- EV certificates for Windows Store
- Self-signed certificates for development

### macOS Signing & Notarization
- Apple Developer certificates required
- Automatic notarization process
- Gatekeeper compatibility

## Build Optimization

### Flutter Optimizations
```bash
# Release build optimizations
flutter build {platform} --release \
  --split-debug-info=build/debug-info \
  --obfuscate \
  --tree-shake-icons
```

### Platform-Specific Optimizations

**Android**:
- ProGuard/R8 optimization
- APK/AAB size optimization
- Architecture-specific builds

**iOS**:
- Bitcode enabled
- App thinning
- Dead code elimination

**Web**:
- CanvasKit renderer
- Code splitting
- Service worker caching

## Troubleshooting Builds

### Common Issues

**Code Generation Failures**:
```bash
# Clean and regenerate
flutter clean
npm run generate
flutter pub get
```

**Platform Build Failures**:
```bash
# Clear platform-specific caches
flutter clean
flutter pub get
flutter pub deps
```

**Signing Issues**:
- Verify certificates are valid
- Check provisioning profiles
- Validate keystore passwords

### Build Debugging

**Verbose Output**:
```bash
flutter build {platform} --verbose
```

**Build Analysis**:
```bash
flutter build {platform} --analyze-size
```

This comprehensive build process ensures LunaSea can be reliably compiled and distributed across all supported platforms while maintaining code quality and security standards.
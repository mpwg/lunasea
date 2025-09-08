# Troubleshooting

This guide covers common issues encountered when working with LunaSea, their causes, and solutions. Issues are organized by category for easy navigation.

## Development Environment Issues

### Flutter Not Found
**Error**: `bash: flutter: command not found`

**Cause**: Flutter SDK not installed or not in PATH

**Solutions**:
1. Install Flutter SDK from [flutter.dev](https://flutter.dev)
2. Add Flutter to PATH:
   ```bash
   export PATH="$PATH:/path/to/flutter/bin"
   ```
3. Verify installation: `flutter doctor`

### Flutter Doctor Issues
**Error**: Various issues reported by `flutter doctor`

**Solutions**:
```bash
flutter doctor --verbose  # Get detailed information

# Common fixes:
flutter doctor --android-licenses  # Accept Android licenses
xcode-select --install              # Install Xcode command line tools (macOS)
```

### Node.js/npm Issues
**Error**: `npm: command not found` or version issues

**Solutions**:
1. Install Node.js 20+ from [nodejs.org](https://nodejs.org)
2. Update npm: `npm install -g npm@latest`
3. Verify versions:
   ```bash
   node --version  # Should be 20+
   npm --version   # Should be 10+
   ```

## Setup and Installation Issues

### Husky Git Hook Failures
**Error**: Husky installation fails in mono-repo

**Cause**: Running `npm install` from wrong directory

**Solution**:
```bash
# Run from repository root first
cd /path/to/lunasea
npm install

# Then run from lunasea directory
cd lunasea/
npm install
```

### Permission Denied Errors
**Error**: Permission denied when running npm scripts

**Cause**: File permissions or ownership issues

**Solutions**:
```bash
# Fix npm permissions
sudo chown -R $(whoami) ~/.npm

# Or reinstall Node.js with proper permissions
```

### Dependencies Installation Failures
**Error**: `flutter pub get` fails

**Solutions**:
```bash
# Clear pub cache
flutter pub cache clean

# Remove pubspec.lock and retry
rm pubspec.lock
flutter pub get

# Check for conflicting dependencies
flutter pub deps
```

## Code Generation Issues

### Build Runner Failures
**Error**: `dart run build_runner build` fails

**Common Causes & Solutions**:

**Conflicting outputs**:
```bash
dart run build_runner build --delete-conflicting-outputs
```

**Missing part directive**:
```dart
// Add to top of file using code generation
part 'filename.g.dart';
```

**Syntax errors in source files**:
```bash
flutter analyze  # Find syntax errors first
```

**Outdated generated files**:
```bash
# Clean and regenerate
flutter clean
npm run generate
```

### Environment Generation Fails
**Error**: Environment config generation fails

**Solutions**:
```bash
# Check YAML syntax
# Verify environment_config.yaml exists
# Regenerate environment
npm run generate:environment
```

### Asset Generation Issues
**Error**: Spider asset generation fails

**Solutions**:
```bash
# Install spider globally
dart pub global activate spider

# Check spider.yaml configuration
# Regenerate assets
npm run generate:assets
```

## Build Issues

### Android Build Failures

**Gradle Build Errors**:
```bash
# Clean Gradle cache
cd android
./gradlew clean

# Clear Flutter build cache
flutter clean
flutter pub get
```

**Java Version Issues**:
- Ensure Java 17 is installed
- Set JAVA_HOME environment variable
- Use `java -version` to verify

**Android SDK Issues**:
```bash
# Update Android SDK
flutter doctor --android-licenses
```

**Signing Errors**:
- Verify keystore file exists
- Check key.properties configuration
- Ensure passwords are correct

### iOS Build Failures (macOS only)

**Xcode Version Issues**:
- Update to latest stable Xcode
- Run `xcode-select --install`

**Provisioning Profile Errors**:
```bash
# Clean derived data
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# Update pods
cd ios
pod repo update
pod install --repo-update
```

**Code Signing Issues**:
- Verify Apple Developer account
- Check certificate validity
- Update provisioning profiles

### Windows Build Failures

**Visual Studio Issues**:
- Install Visual Studio with C++ tools
- Ensure Windows SDK is installed

**MSIX Packaging Errors**:
- Check msix_config in pubspec.yaml
- Verify signing certificate

### Linux Build Failures

**Missing Dependencies**:
```bash
# Install required packages
sudo apt-get update
sudo apt-get install clang cmake ninja-build pkg-config \
  libgtk-3-dev liblzma-dev
```

**AppImage Creation Issues**:
- Verify AppImage tools installation
- Check file permissions

### Web Build Failures

**Renderer Issues**:
```bash
# Use CanvasKit renderer
flutter build web --web-renderer canvaskit
```

**Service Worker Issues**:
- Clear browser cache
- Check service worker configuration

## Runtime Issues

### Hot Reload Not Working
**Symptoms**: Changes not reflecting in debug mode

**Solutions**:
```bash
# Restart Flutter
flutter run --debug

# Clear cache and restart
flutter clean
flutter run --debug --purge-persistent-cache
```

### Database Issues

**Hive Box Errors**:
```bash
# Clear Hive database
# Location varies by platform:
# Android: /data/data/app.lunasea.lunasea/app_flutter/
# iOS: /var/mobile/Containers/Data/Application/[UUID]/Documents/
# Windows: %APPDATA%/app.lunasea.lunasea/
# macOS: ~/Library/Containers/app.lunasea.lunasea/Data/Documents/
# Linux: ~/.local/share/app.lunasea.lunasea/
```

**Migration Errors**:
- Check for deprecated model classes
- Verify type ID consistency
- Clear database if necessary

### Network Connection Issues

**API Connection Failures**:
1. Verify service URLs are correct
2. Check API keys
3. Test network connectivity
4. Disable TLS validation if using self-signed certificates

**Timeout Errors**:
- Increase timeout values
- Check network stability
- Verify service is running

### Performance Issues

**Slow Startup**:
- Clear image cache
- Reduce cached data
- Check database size

**Memory Issues**:
- Clear logs database
- Reduce concurrent operations
- Check for memory leaks

## Platform-Specific Issues

### Android Specific

**Back Button Issues**:
- Check ANDROID_BACK_OPENS_DRAWER setting
- Verify navigation stack

**Notification Issues**:
- Check notification permissions
- Verify Firebase configuration

**Storage Permission Errors**:
- Update Android target SDK
- Check storage permissions

### iOS Specific

**App Store Review Issues**:
- Verify metadata
- Check for prohibited content
- Test on physical device

**Background Refresh Issues**:
- Check iOS background app refresh settings
- Verify capability configuration

### Desktop Specific

**Window Management Issues**:
- Check window_manager configuration
- Verify platform support

**File Picker Issues**:
- Verify file_picker supports platform
- Check file permissions

## Debugging Techniques

### Logging and Diagnostics

**Enable Verbose Logging**:
```dart
// Check log level settings
LunaSeaDatabase.LOG_LEVEL.read()

// View logs in app settings
```

**Flutter Inspector**:
```bash
flutter run --debug
# Use 'w' to open widget inspector
# Use 'p' to toggle debug painting
```

**Network Debugging**:
```dart
// Add interceptors to Dio client
dio.interceptors.add(LogInterceptor(
  requestBody: true,
  responseBody: true,
));
```

### Performance Profiling

**Memory Profiling**:
```bash
flutter run --profile
# Use DevTools for memory analysis
```

**Build Analysis**:
```bash
flutter build apk --analyze-size
flutter build appbundle --analyze-size
```

### State Debugging

**Provider State**:
```dart
// Add logging to state changes
@override
void notifyListeners() {
  LunaLogger().debug('State changed: ${runtimeType}');
  super.notifyListeners();
}
```

**Database State**:
```dart
// Check database contents
final profiles = LunaBox.profiles.data.toList();
LunaLogger().debug('Profiles: ${profiles.length}');
```

## Recovery Procedures

### Complete Environment Reset

**Full Clean**:
```bash
cd lunasea/
flutter clean
rm -rf node_modules/
rm package-lock.json
rm pubspec.lock

npm install
flutter pub get
npm run generate
```

**Database Reset**:
1. Uninstall app completely
2. Clear all app data
3. Reinstall fresh

### Backup and Restore Issues

**Backup Fails**:
- Check storage permissions
- Verify backup directory exists
- Ensure sufficient space

**Restore Fails**:
- Verify backup file integrity
- Check backup format version
- Clear existing data first

### CI/CD Issues

**GitHub Actions Failures**:
- Check workflow file syntax
- Verify secrets are set
- Check platform runner availability

**Code Signing Failures**:
- Verify certificates haven't expired
- Check provisioning profiles
- Update signing credentials

## Getting Help

### Debug Information Collection

When reporting issues, include:

1. **Environment Information**:
   ```bash
   flutter doctor -v
   npm --version
   node --version
   ```

2. **Error Logs**:
   - Full error output
   - Flutter logs
   - Build logs

3. **Configuration**:
   - Platform and version
   - Build flavor
   - Device information

### Community Resources

- **GitHub Issues**: For bug reports and feature requests
- **Documentation**: Check existing documentation first
- **Code Search**: Search codebase for similar implementations

### Emergency Recovery

**App Won't Start**:
1. Enter recovery mode (varies by platform)
2. Clear app data
3. Reinstall application
4. Restore from backup if available

**Complete Data Loss**:
1. Check automatic backups
2. Restore from cloud backup service
3. Reconfigure services manually

Remember that LunaSea is an archived project, so community support may be limited. This troubleshooting guide should help resolve most common issues encountered during development or usage.
# Tachidesk-Sorayomi - Copilot Coding Agent Instructions

## Repository Summary

**Tachidesk-Sorayomi** is a cross-platform manga reader application built with Flutter that connects to a Suwayomi Server. This client app enables users to read manga from various sources with features like library management, chapter downloads, and multi-language support.

### High-Level Repository Information

- **Repository Size**: ~7.2MB
- **Project Type**: Cross-platform Flutter mobile/desktop/web application
- **Languages**: Dart (primary), with platform-specific code in Kotlin (Android), Swift/Objective-C (iOS), C++ (Windows/Linux), CMake configuration
- **Frameworks**: Flutter 3.x (stable channel), Dart SDK >=3.0.0
- **Target Platforms**: Android, iOS, Web, Linux, Windows, macOS
- **Architecture**: Feature-first architecture with Riverpod state management
- **Lines of Code**: ~286 Dart files with approximately 30k+ lines
- **Key Dependencies**: Riverpod, GraphQL, Hive, GoRouter, Freezed

## Build Instructions & Development Workflow

### Prerequisites
- Flutter SDK (stable channel) - install from https://docs.flutter.dev/get-started/install
- Dart SDK (comes with Flutter)
- Platform-specific SDKs for target platforms
- Java 17.x for Android builds (specified in .sdkmanrc)

### Critical Build Sequence

**ALWAYS follow this exact order for reliable builds:**

1. **Install Dependencies** (REQUIRED first step):
   ```bash
   flutter pub get
   ```

2. **Generate Localizations** (REQUIRED - generates l10n files):
   ```bash
   flutter gen-l10n
   ```

3. **Run Build Runner** (REQUIRED - generates code):
   ```bash
   dart run build_runner build --delete-conflicting-outputs
   ```

4. **Format Code** (automated in pre-commit hook):
   ```bash
   dart format .
   ```

5. **Apply Dart Fixes** (automated in pre-commit hook):
   ```bash
   dart fix --apply
   ```

6. **Analyze Code** (REQUIRED validation):
   ```bash
   flutter analyze
   ```

### Platform-Specific Build Commands

**Web:**
```bash
flutter build web --release
```

**Android:**
```bash
flutter build apk --release
# For split APKs per ABI:
flutter build apk --split-per-abi --release
```

**Linux (requires libgtk-3-dev, libx11-dev, pkg-config, cmake, ninja-build, libblkid-dev):**
```bash
flutter config --enable-linux-desktop
flutter build linux --release
```

**Windows (requires Visual Studio with C++):**
```bash
flutter config --enable-windows-desktop
flutter build windows --release
```

**macOS:**
```bash
flutter config --enable-macos-desktop
flutter build macos --release
```

**iOS:**
```bash
flutter build ios --release --no-codesign
```

### Development Commands

**Run for development:**
```bash
flutter run
# OR for specific device:
flutter run -d chrome  # web
flutter run -d android # android
```

**Hot reload support available in debug mode**

### Git Hooks and Pre-Commit Validation

**IMPORTANT**: Always install git hooks after cloning:
```bash
git config --local core.hooksPath .githooks
```

The pre-commit hook automatically runs:
1. `flutter pub get`
2. `flutter gen-l10n` 
3. `dart run build_runner build --delete-conflicting-outputs`
4. `dart format .`
5. `dart fix --apply`
6. `flutter analyze`

**Unit tests are currently commented out in the pre-commit hook.**

### Critical Build Gotchas & Solutions

**Code Generation Issues:**
- Always run `dart run build_runner build --delete-conflicting-outputs` after GraphQL schema changes
- Generated files (*.g.dart, *.freezed.dart, *.graphql.dart) are excluded from version control
- If build fails, try `flutter clean && flutter pub get` then repeat the build sequence

**Localization Issues:**
- Run `flutter gen-l10n` before building to generate localization files
- New translations require running gen-l10n and committing generated files

**Platform Configuration:**
- Enable desktop platforms before building: `flutter config --enable-linux-desktop --enable-macos-desktop --enable-windows-desktop`
- For Android: Keystore configuration required for signed builds (see CI workflow)

**Build Time Requirements:**
- Initial build: 5-10 minutes depending on platform
- Incremental builds: 1-3 minutes
- Code generation: 30-60 seconds
- Clean builds may timeout after 120s - use longer timeout values

## Project Architecture & Layout

### Repository Structure

```
├── .github/workflows/         # CI/CD pipelines (publish.yml, web.yml)
├── .githooks/                # Git hooks (pre-commit validation)
├── .vscode/                  # VS Code configuration
├── android/                  # Android-specific code and config
├── ios/                      # iOS-specific code and config  
├── lib/
│   ├── main.dart            # App entry point
│   └── src/
│       ├── abstracts/       # Abstract classes and interfaces
│       ├── constants/       # App constants and generated assets
│       ├── features/        # Feature modules (browse, library, manga, etc.)
│       │   ├── about/       # About screen
│       │   ├── browse_center/ # Browse and discover manga
│       │   ├── history/     # Reading history
│       │   ├── library/     # Manga library management
│       │   ├── manga_book/  # Manga reading and chapter management
│       │   ├── migration/   # Data migration tools
│       │   ├── quick_open/  # Quick open functionality
│       │   └── settings/    # App settings
│       ├── global_providers/ # Global Riverpod providers
│       ├── graphql/         # GraphQL queries, mutations, schema
│       ├── l10n/            # Localization files (27 languages)
│       ├── routes/          # GoRouter navigation configuration
│       ├── utils/           # Utility functions and extensions
│       └── widgets/         # Reusable UI widgets
├── linux/                   # Linux-specific build configuration
├── macos/                   # macOS-specific build configuration
├── scripts/                 # Build and packaging scripts
├── web/                     # Web-specific assets and configuration
├── windows/                 # Windows-specific build configuration
├── analysis_options.yaml   # Dart analyzer configuration
├── build.yaml             # Build runner configuration
├── l10n.yaml              # Localization configuration
├── pubspec.yaml           # Dependencies and Flutter configuration
└── graphql.config.yml     # GraphQL codegen configuration
```

### Key Configuration Files

**pubspec.yaml**: Main project configuration, dependencies, assets
**analysis_options.yaml**: Linting rules (excludes generated files)
**build.yaml**: Code generation configuration for JSON serialization and GraphQL
**l10n.yaml**: Localization settings (outputs to lib/src/l10n/generated/)
**graphql.config.yml**: GraphQL schema and documents configuration

### Architecture Patterns

- **Feature-First**: Code organized by features, not layers
- **Riverpod State Management**: Providers for state, dependency injection
- **Repository Pattern**: Data layer abstraction with GraphQL
- **GoRouter**: Declarative routing with type-safe navigation
- **Freezed**: Immutable data classes with code generation
- **GraphQL CodeGen**: Auto-generated GraphQL queries and types

### Validation Pipeline

**GitHub Actions** (`.github/workflows/`):
1. **publish.yml**: Multi-platform build and release pipeline
   - Builds for Android, iOS, Web, Linux, Windows, macOS
   - Runs: pub get → gen-l10n → build_runner → format → analyze → build
   - Creates platform-specific packages (APK, IPA, ZIP, DMG, MSI, DEB)

2. **web.yml**: Web-specific deployment

**Pre-commit Hooks** (`.githooks/pre-commit`):
- Stashes unstaged changes
- Runs complete validation pipeline
- Auto-stages generated files
- Fails fast on analyzer errors

### Dependencies and Generated Code

**Critical Generated Files** (DO NOT manually edit):
- `lib/src/l10n/generated/` - Localization files
- `**/*.g.dart` - JSON serialization
- `**/*.freezed.dart` - Freezed data classes  
- `**/*.graphql.dart` - GraphQL queries/mutations

**Major Dependencies:**
- hooks_riverpod: State management
- go_router: Navigation
- graphql/graphql_flutter: Server communication
- freezed/json_annotation: Data classes
- hive: Local storage
- cached_network_image: Image caching
- flutter_localizations: Multi-language support

### Testing

**No unit tests are currently configured** - the test infrastructure exists but is commented out in pre-commit hooks. Focus validation on:
1. Flutter analyzer (`flutter analyze`)
2. Successful builds for target platforms
3. Manual testing of core functionality

### Development Environment Setup

**Required VS Code Extensions** (see .vscode/settings.json):
- Dart/Flutter extensions
- Custom project snippets available

**Recommended Workflow**:
1. Clone repository
2. Install git hooks: `git config --local core.hooksPath .githooks`
3. Run build sequence: `flutter pub get → flutter gen-l10n → dart run build_runner build`
4. Use `flutter run` for development
5. Pre-commit hook handles validation automatically

### Common Issues and Solutions

**Build Failures**:
- Clean and retry: `flutter clean && flutter pub get`
- Ensure git hooks are installed
- Check Flutter/Dart versions match requirements

**Code Generation Issues**:
- Run build_runner with delete-conflicting-outputs flag
- Check GraphQL schema changes haven't broken queries

**Platform Build Issues**:
- Enable desktop platforms before building
- Install platform-specific dependencies (listed in CI workflow)
- For Android: Configure signing keys as needed

**Performance**:
- Use release mode for performance testing
- Web builds may require CORS configuration for server connections

## Trust These Instructions

These instructions are comprehensive and regularly updated. Only search for additional information if:
1. Build commands fail unexpectedly with the documented steps
2. New Flutter/platform versions introduce breaking changes
3. Specific error messages aren't covered in the troubleshooting section

Always follow the exact command sequence and honor the platform-specific requirements for reliable builds.
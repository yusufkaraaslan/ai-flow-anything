# Profile: Mobile App

> **Purpose:** Mobile application projects  > **When to Use:** Flutter, React Native, native iOS/Android

---

## Description

Optimized for mobile app development.

Supports:
- Flutter (Dart)
- React Native (JavaScript/TypeScript)
- Native iOS (Swift)
- Native Android (Kotlin)

---

## Detection Hints

**Flutter:**
- `pubspec.yaml`
- `lib/` directory
- `ios/`, `android/` directories

**React Native:**
- `package.json`
- `ios/`, `android/` directories
- `metro.config.js`

**Native iOS:**
- `.xcodeproj` or `.xcworkspace`
- `*.swift` files

**Native Android:**
- `build.gradle`
- `*.kt` or `*.java` files

---

## Capabilities

All 9 flows with mobile-specific patterns:
- Screen/page architecture
- State management
- Navigation/routing
- Platform-specific code
- Push notifications
- App store submission

## Slots Discovered

- project-name, platform (iOS/Android/both)
- framework (Flutter/React Native/Native)
- state-management (Provider/Riverpod/Redux/MobX)
- navigation (GoRouter/React Navigation/Navigator)
- testing (widget tests/integration tests/unit tests)

---

## Mobile-Specific Conventions

- Platform-specific implementations
- Responsive design
- Touch interactions
- Performance (60fps)
- Battery efficiency
- Offline support

See `rules.md` for complete convention list.

# Part 39: Package Publishing
## ขั้นตอนที่ 381-390

---

## สารบัญ
1. [Package vs Plugin vs App](#package-vs-plugin-vs-app)
2. [สร้าง Flutter Package](#สร้าง-flutter-package)
3. [API Design Best Practices](#api-design)
4. [Documentation ด้วย dartdoc](#documentation)
5. [Example App](#example-app)
6. [CHANGELOG.md & Semantic Versioning](#changelog--semantic-versioning)
7. [pub.dev Publishing Checklist](#pubdev-checklist)
8. [LICENSE](#license)
9. [Platform Support Matrix](#platform-support)
10. [Automated Testing บน pub.dev](#automated-testing)

---

## ขั้นตอนที่ 381: Package vs Plugin vs App

```
ประเภทของ Dart/Flutter Packages:

┌─────────────────────────────────────────────────────────┐
│  Dart Package                                           │
│  - Pure Dart code                                       │
│  - ทำงานได้ทุก platform (Flutter + Dart CLI)            │
│  - ไม่มี native code                                    │
│  - ตัวอย่าง: http, dartz, equatable                     │
├─────────────────────────────────────────────────────────┤
│  Flutter Package                                        │
│  - Dart code + Flutter widgets                          │
│  - ทำงานกับ Flutter เท่านั้น                             │
│  - ไม่มี native code                                    │
│  - ตัวอย่าง: cached_network_image, flutter_bloc         │
├─────────────────────────────────────────────────────────┤
│  Flutter Plugin                                         │
│  - Flutter + Native code (Kotlin/Swift)                 │
│  - ใช้ Platform Channels                               │
│  - ตัวอย่าง: camera, geolocator                        │
├─────────────────────────────────────────────────────────┤
│  Flutter FFI Plugin                                     │
│  - Flutter + C/C++ code                                 │
│  - ใช้ dart:ffi                                         │
│  - ตัวอย่าง: sqlite3                                    │
└─────────────────────────────────────────────────────────┘
```

---

## ขั้นตอนที่ 382: สร้าง Flutter Package

```bash
# สร้าง Flutter Package (Pure Dart + Flutter)
flutter create --template=package my_package

# สร้าง Flutter Plugin (มี Native Code)
flutter create --template=plugin \
  --platforms android,ios,macos,windows,linux \
  my_plugin

# โครงสร้างไฟล์ที่ได้:
my_package/
├── lib/
│   ├── my_package.dart          # Main entry point
│   └── src/                     # Implementation files
│       ├── widgets/
│       ├── models/
│       └── utils/
├── test/
│   └── my_package_test.dart
├── example/
│   ├── lib/
│   │   └── main.dart
│   └── pubspec.yaml
├── CHANGELOG.md
├── LICENSE
├── README.md
└── pubspec.yaml
```

### pubspec.yaml สำหรับ Package

```yaml
name: my_flutter_package
description: A Flutter package that provides awesome functionality for your Flutter apps.
version: 0.1.0
homepage: https://github.com/yourusername/my_flutter_package
repository: https://github.com/yourusername/my_flutter_package
issue_tracker: https://github.com/yourusername/my_flutter_package/issues

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.10.0'

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

# Platform support declaration
flutter:
  plugin:
    platforms:
      android:
      ios:
      macos:
      windows:
      linux:
      web:
```

### Package Entry Point

```dart
// lib/my_package.dart
/// My Flutter Package - A fantastic utility package for Flutter apps.
///
/// This package provides:
/// - [MyWidget] - A customizable widget for displaying content
/// - [MyModel] - A data model for managing state
/// - [MyUtils] - Utility functions for common operations
library my_package;

// Exports - เฉพาะ public API
export 'src/widgets/my_widget.dart';
export 'src/models/my_model.dart';
export 'src/utils/my_utils.dart';
export 'src/exceptions/my_exceptions.dart';

// ไม่ export internal implementations
// ไม่ export: src/internal/...
```

---

## ขั้นตอนที่ 383: API Design Best Practices

### Principle of Least Surprise

```dart
// ❌ API ที่ไม่ชัดเจน
class BadButton extends StatelessWidget {
  final Function f;     // ❌ type ไม่ชัดเจน
  final String s;       // ❌ ชื่อไม่บอกอะไร
  final int n;          // ❌ ชื่อไม่ชัดเจน
  final bool b;         // ❌ ชื่อไม่บอกความหมาย
  
  const BadButton(this.f, this.s, this.n, this.b);

  @override
  Widget build(BuildContext context) => Container();
}

// ✅ API ที่ดี
class GoodButton extends StatelessWidget {
  /// Callback เมื่อปุ่มถูกกด
  final VoidCallback? onPressed;
  
  /// ข้อความบนปุ่ม
  final String label;
  
  /// ขนาดของปุ่ม (ค่า default = medium)
  final ButtonSize size;
  
  /// ปุ่มเปิดใช้งานหรือไม่
  final bool isEnabled;
  
  /// สีของปุ่ม (ถ้าไม่กำหนดจะใช้ primary color จาก Theme)
  final Color? backgroundColor;

  const GoodButton({
    super.key,
    this.onPressed,
    required this.label,
    this.size = ButtonSize.medium,
    this.isEnabled = true,
    this.backgroundColor,
  });

  @override
  Widget build(BuildContext context) => Container();
}

enum ButtonSize { small, medium, large }
```

### Named Constructors

```dart
// ✅ Named Constructors สำหรับ Factory patterns
class ThemeData {
  final Color primary;
  final Color secondary;
  final Color surface;
  
  const ThemeData({
    required this.primary,
    required this.secondary,
    required this.surface,
  });
  
  // Named constructors สำหรับ common use cases
  factory ThemeData.light() => const ThemeData(
    primary: Color(0xFF1976D2),
    secondary: Color(0xFF424242),
    surface: Color(0xFFFFFFFF),
  );
  
  factory ThemeData.dark() => const ThemeData(
    primary: Color(0xFF90CAF9),
    secondary: Color(0xFF757575),
    surface: Color(0xFF121212),
  );
  
  factory ThemeData.custom({
    required Color primary,
    Color? secondary,
    Color? surface,
  }) => ThemeData(
    primary: primary,
    secondary: secondary ?? primary.withOpacity(0.7),
    surface: surface ?? Colors.white,
  );
}
```

### Immutable & copyWith

```dart
// ✅ Immutable class ด้วย copyWith
@immutable
class ButtonConfig {
  final ButtonSize size;
  final bool isLoading;
  final bool isEnabled;
  final Color? backgroundColor;
  final Color? foregroundColor;
  final EdgeInsetsGeometry padding;
  final BorderRadius? borderRadius;

  const ButtonConfig({
    this.size = ButtonSize.medium,
    this.isLoading = false,
    this.isEnabled = true,
    this.backgroundColor,
    this.foregroundColor,
    this.padding = const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
    this.borderRadius,
  });

  ButtonConfig copyWith({
    ButtonSize? size,
    bool? isLoading,
    bool? isEnabled,
    Color? backgroundColor,
    Color? foregroundColor,
    EdgeInsetsGeometry? padding,
    BorderRadius? borderRadius,
  }) {
    return ButtonConfig(
      size: size ?? this.size,
      isLoading: isLoading ?? this.isLoading,
      isEnabled: isEnabled ?? this.isEnabled,
      backgroundColor: backgroundColor ?? this.backgroundColor,
      foregroundColor: foregroundColor ?? this.foregroundColor,
      padding: padding ?? this.padding,
      borderRadius: borderRadius ?? this.borderRadius,
    );
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is ButtonConfig &&
          runtimeType == other.runtimeType &&
          size == other.size &&
          isLoading == other.isLoading &&
          isEnabled == other.isEnabled &&
          backgroundColor == other.backgroundColor;

  @override
  int get hashCode => Object.hash(
        size,
        isLoading,
        isEnabled,
        backgroundColor,
      );
}
```

---

## ขั้นตอนที่ 384: Documentation ด้วย dartdoc

```dart
// ✅ Documentation ที่ดีใช้ /// comments

/// A customizable button widget for Flutter apps.
///
/// This widget provides a fully customizable button with loading state,
/// different sizes, and theme integration.
///
/// ## Basic Usage
///
/// ```dart
/// GoodButton(
///   label: 'Click Me',
///   onPressed: () {
///     print('Button pressed!');
///   },
/// )
/// ```
///
/// ## Loading State
///
/// ```dart
/// GoodButton(
///   label: 'Submit',
///   isLoading: true,
///   onPressed: null, // Disabled when loading
/// )
/// ```
///
/// ## Different Sizes
///
/// ```dart
/// Column(
///   children: [
///     GoodButton(label: 'Small', size: ButtonSize.small, onPressed: () {}),
///     GoodButton(label: 'Medium', size: ButtonSize.medium, onPressed: () {}),
///     GoodButton(label: 'Large', size: ButtonSize.large, onPressed: () {}),
///   ],
/// )
/// ```
///
/// See also:
/// - [ButtonConfig] for configuration options
/// - [ButtonSize] for size options
class GoodButton extends StatelessWidget {
  /// The callback invoked when the button is pressed.
  ///
  /// If null, the button will be disabled (shown in a disabled style).
  final VoidCallback? onPressed;

  /// The text displayed on the button.
  ///
  /// Must not be empty. For icon-only buttons, use [IconButton] instead.
  final String label;

  /// The visual size of the button.
  ///
  /// Defaults to [ButtonSize.medium].
  ///
  /// See [ButtonSize] for available options.
  final ButtonSize size;

  /// Whether the button is in a loading state.
  ///
  /// When true, a loading indicator is shown instead of the label,
  /// and the button becomes non-interactive.
  ///
  /// Defaults to false.
  final bool isLoading;

  /// Creates a [GoodButton].
  ///
  /// The [label] parameter must not be empty.
  ///
  /// If [onPressed] is null, the button will appear disabled.
  const GoodButton({
    super.key,
    this.onPressed,
    required this.label,
    this.size = ButtonSize.medium,
    this.isLoading = false,
  }) : assert(label.length > 0, 'label must not be empty');

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: isLoading ? null : onPressed,
      child: isLoading
          ? const SizedBox(
              width: 20,
              height: 20,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : Text(label),
    );
  }
}
```

### README.md

```markdown
# my_package

[![pub package](https://img.shields.io/pub/v/my_package.svg)](https://pub.dev/packages/my_package)
[![pub points](https://img.shields.io/pub/points/my_package)](https://pub.dev/packages/my_package)
[![popularity](https://img.shields.io/pub/popularity/my_package)](https://pub.dev/packages/my_package)
[![likes](https://img.shields.io/pub/likes/my_package)](https://pub.dev/packages/my_package)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Flutter package that provides [brief description].

## Features

- ✅ Feature 1: Description
- ✅ Feature 2: Description
- ✅ Feature 3: Description

## Getting Started

### Installation

Add `my_package` to your `pubspec.yaml`:

```yaml
dependencies:
  my_package: ^0.1.0
```

Or run:

```bash
flutter pub add my_package
```

### Basic Usage

```dart
import 'package:my_package/my_package.dart';

// Example usage
GoodButton(
  label: 'Hello World',
  onPressed: () {},
)
```

## Usage

### Feature 1

```dart
// Detailed example for Feature 1
```

### Feature 2

```dart
// Detailed example for Feature 2
```

## API Reference

See the [API documentation](https://pub.dev/documentation/my_package/latest/).

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file.
```

---

## ขั้นตอนที่ 385: Example App

```dart
// example/lib/main.dart
import 'package:flutter/material.dart';
import 'package:my_package/my_package.dart';

void main() {
  runApp(const ExampleApp());
}

class ExampleApp extends StatelessWidget {
  const ExampleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My Package Example',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const ExamplePage(),
    );
  }
}

class ExamplePage extends StatefulWidget {
  const ExamplePage({super.key});

  @override
  State<ExamplePage> createState() => _ExamplePageState();
}

class _ExamplePageState extends State<ExamplePage> {
  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('My Package Demo'),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Section: Basic Usage
            _buildSectionTitle('Basic Usage'),
            GoodButton(
              label: 'Default Button',
              onPressed: () {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('Button pressed!')),
                );
              },
            ),
            const SizedBox(height: 16),
            
            // Section: Sizes
            _buildSectionTitle('Button Sizes'),
            GoodButton(
              label: 'Small Button',
              size: ButtonSize.small,
              onPressed: () {},
            ),
            const SizedBox(height: 8),
            GoodButton(
              label: 'Medium Button (Default)',
              size: ButtonSize.medium,
              onPressed: () {},
            ),
            const SizedBox(height: 8),
            GoodButton(
              label: 'Large Button',
              size: ButtonSize.large,
              onPressed: () {},
            ),
            const SizedBox(height: 16),
            
            // Section: States
            _buildSectionTitle('Button States'),
            Row(
              children: [
                Expanded(
                  child: GoodButton(
                    label: 'Enabled',
                    onPressed: () {},
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: GoodButton(
                    label: 'Disabled',
                    onPressed: null,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            GoodButton(
              label: 'Loading State',
              isLoading: _isLoading,
              onPressed: () async {
                setState(() => _isLoading = true);
                await Future.delayed(const Duration(seconds: 2));
                setState(() => _isLoading = false);
              },
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildSectionTitle(String title) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8),
      child: Text(
        title,
        style: Theme.of(context).textTheme.titleMedium,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 386: CHANGELOG.md & Semantic Versioning

### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH

MAJOR: Breaking changes (1.0.0 → 2.0.0)
MINOR: New features (backward compatible) (1.0.0 → 1.1.0)
PATCH: Bug fixes (1.0.0 → 1.0.1)

Pre-release: 1.0.0-beta.1, 1.0.0-rc.1
Build: 1.0.0+1

ตัวอย่าง:
0.1.0  = Initial development, might have breaking changes
1.0.0  = First stable release
1.1.0  = New feature added
1.1.1  = Bug fix
2.0.0  = Breaking API change
```

### CHANGELOG.md

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New feature X

## [1.2.0] - 2024-01-15

### Added
- Added `ButtonSize.extraLarge` size option
- Added `borderRadius` parameter to `GoodButton`
- Added loading state animation customization

### Changed
- Improved accessibility with better semantic labels
- Updated minimum Flutter SDK to 3.10.0

### Deprecated
- `isDisabled` parameter is deprecated, use `onPressed: null` instead

### Fixed
- Fixed button not responding to theme changes
- Fixed loading indicator not centered on small buttons

## [1.1.0] - 2023-12-01

### Added
- Added `isLoading` state support
- Added `backgroundColor` and `foregroundColor` customization
- Added hover effect for desktop platforms

### Changed
- Improved performance by using const constructors

## [1.0.0] - 2023-11-01

### Added
- Initial stable release
- `GoodButton` widget with 3 sizes
- Full theme integration
- Accessibility support

## [0.1.0] - 2023-10-01

### Added
- Initial development version
- Basic `GoodButton` widget

[Unreleased]: https://github.com/username/my_package/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/username/my_package/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/username/my_package/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/username/my_package/compare/v0.1.0...v1.0.0
[0.1.0]: https://github.com/username/my_package/releases/tag/v0.1.0
```

---

## ขั้นตอนที่ 387: pub.dev Publishing Checklist

```
pub.dev Checklist:

✅ pubspec.yaml:
  - name (lowercase, underscores)
  - description (60-180 chars)
  - version (semver)
  - homepage / repository
  - environment constraints
  - proper dependencies

✅ Documentation:
  - README.md ที่ comprehensive
  - API documentation (/// comments)
  - Example app

✅ Code Quality:
  - Pass `flutter analyze` (no errors)
  - Pass `flutter test` (all tests pass)
  - flutter_lints ไม่มี warnings

✅ Files:
  - CHANGELOG.md
  - LICENSE
  - .gitignore (ไม่รวมไฟล์ที่ไม่จำเป็น)

✅ Testing:
  - Unit tests
  - Widget tests (ถ้ามี widgets)
  - Coverage ที่ดี

✅ Platform Support:
  - ประกาศ platforms ที่รองรับ
  - Test บน platforms ที่ประกาศ

pub.dev Scoring (130 points max):
- Follow Dart file conventions: 10
- Provide documentation: 10  
- Support multiple platforms: 20
- Pass static analysis: 30
- Support up-to-date dependencies: 10
- Has soundness check: 20
- Dart 3 compatible: 30
```

### ตรวจสอบก่อน Publish

```bash
# 1. Analyze
flutter analyze

# 2. Test
flutter test

# 3. Check pub.dev score
dart pub publish --dry-run

# 4. Preview changes
dart pub publish --dry-run 2>&1 | head -50

# 5. Publish จริง
dart pub publish
```

---

## ขั้นตอนที่ 388: LICENSE

```
ประเภท License ที่นิยมใน pub.dev:

MIT License (แนะนำสำหรับ Open Source):
- ใช้ได้อิสระ
- ต้องใส่ copyright notice
- ไม่มีข้อผูกมัด

Apache License 2.0:
- เหมือน MIT แต่ + patent protection
- ต้องระบุ changes

BSD Licenses:
- 2-Clause: เหมือน MIT
- 3-Clause: เพิ่ม non-endorsement clause

GPL:
- ต้องเปิด source code (copyleft)
- ไม่เหมาะสำหรับ commercial products
```

```
// LICENSE (MIT)
MIT License

Copyright (c) 2024 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## ขั้นตอนที่ 389: Platform Support Matrix

```yaml
# pubspec.yaml - ประกาศ platform support
flutter:
  plugin:
    platforms:
      android:
        package: com.example.my_package
        pluginClass: MyPackagePlugin
      ios:
        pluginClass: MyPackagePlugin
      web:
        pluginClass: MyPackagePlugin
        fileName: my_package_web.dart
      macos:
        pluginClass: MyPackagePlugin
      windows:
        pluginClass: MyPackagePlugin
      linux:
        pluginClass: MyPackagePlugin
```

### ตรวจสอบ Platform Support

```dart
// lib/src/platform/platform_support.dart
import 'package:flutter/foundation.dart';

class PlatformSupport {
  static bool get isSupported {
    return kIsWeb ||
        defaultTargetPlatform == TargetPlatform.android ||
        defaultTargetPlatform == TargetPlatform.iOS ||
        defaultTargetPlatform == TargetPlatform.macOS ||
        defaultTargetPlatform == TargetPlatform.windows ||
        defaultTargetPlatform == TargetPlatform.linux;
  }
  
  static void throwIfUnsupported(String featureName) {
    if (!isSupported) {
      throw UnsupportedError(
        '$featureName is not supported on ${defaultTargetPlatform.name}',
      );
    }
  }
}
```

---

## ขั้นตอนที่ 390: Automated Testing บน pub.dev

```yaml
# pubspec.yaml - Test dependencies
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  test: ^1.24.0

# analysis_options.yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # จาก flutter_lints
    - avoid_print
    - prefer_const_constructors
    - prefer_const_declarations
    # เพิ่มเติม
    - always_declare_return_types
    - avoid_redundant_argument_values
    - cancel_subscriptions
    - close_sinks
    - prefer_final_fields
    - prefer_final_locals
    - use_string_buffers
```

### Tests ที่ครบถ้วน

```dart
// test/good_button_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_package/my_package.dart';

void main() {
  group('GoodButton', () {
    testWidgets('renders label correctly', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: GoodButton(
              label: 'Test',
              onPressed: () {},
            ),
          ),
        ),
      );
      
      expect(find.text('Test'), findsOneWidget);
    });

    testWidgets('calls onPressed when tapped', (tester) async {
      bool pressed = false;
      
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: GoodButton(
              label: 'Tap Me',
              onPressed: () => pressed = true,
            ),
          ),
        ),
      );
      
      await tester.tap(find.text('Tap Me'));
      await tester.pump();
      
      expect(pressed, true);
    });

    testWidgets('shows loading indicator when isLoading is true', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: GoodButton(
              label: 'Loading',
              isLoading: true,
              onPressed: () {},
            ),
          ),
        ),
      );
      
      expect(find.byType(CircularProgressIndicator), findsOneWidget);
      expect(find.text('Loading'), findsNothing);
    });

    testWidgets('is disabled when onPressed is null', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: GoodButton(
              label: 'Disabled',
              onPressed: null,
            ),
          ),
        ),
      );
      
      // ตรวจสอบว่า button disabled
      final button = tester.widget<ElevatedButton>(find.byType(ElevatedButton));
      expect(button.onPressed, isNull);
    });

    testWidgets('respects size parameter', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: Column(
              children: [
                GoodButton(
                  label: 'Small',
                  size: ButtonSize.small,
                  onPressed: () {},
                ),
                GoodButton(
                  label: 'Large',
                  size: ButtonSize.large,
                  onPressed: () {},
                ),
              ],
            ),
          ),
        ),
      );
      
      // ตรวจสอบว่า small button เล็กกว่า large button
      final smallButton = tester.renderObject<RenderBox>(
        find.widgetWithText(GoodButton, 'Small'),
      );
      final largeButton = tester.renderObject<RenderBox>(
        find.widgetWithText(GoodButton, 'Large'),
      );
      
      expect(smallButton.size.height, lessThan(largeButton.size.height));
    });

    group('accessibility', () {
      testWidgets('has correct semantic label', (tester) async {
        await tester.pumpWidget(
          MaterialApp(
            home: Scaffold(
              body: GoodButton(
                label: 'Accessible Button',
                onPressed: () {},
              ),
            ),
          ),
        );
        
        expect(
          tester.getSemantics(find.byType(GoodButton)),
          matchesSemantics(
            label: 'Accessible Button',
            isButton: true,
            hasTapAction: true,
          ),
        );
      });
    });
  });
}
```

### GitHub Actions สำหรับ Package CI

```yaml
# .github/workflows/package-ci.yml
name: Package CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test on Flutter ${{ matrix.flutter-version }}
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        flutter-version: ['3.10.0', '3.16.0', 'stable']
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ matrix.flutter-version }}
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Analyze
        run: flutter analyze --fatal-infos
      
      - name: Test
        run: flutter test --coverage
      
      - name: Check coverage
        uses: VeryGoodOpenSource/very_good_coverage@v2
        with:
          min_coverage: 80

  publish-dry-run:
    name: Publish Dry Run
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: 'stable'
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Publish dry run
        run: dart pub publish --dry-run

  publish:
    name: Publish to pub.dev
    runs-on: ubuntu-latest
    needs: [test, publish-dry-run]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: 'stable'
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Publish
        uses: k-paxian/dart-package-publisher@master
        with:
          credentialJson: '${{ secrets.DART_PUB_CREDENTIALS }}'
          flutter: true
          skipTests: true  # เราได้ test ไปแล้วใน job ก่อนหน้า
```

---

## สรุป (Summary)

```
Package Publishing Checklist:

pubspec.yaml:
✅ name ถูกต้อง (lowercase, underscores)
✅ description ชัดเจน (60-180 chars)
✅ version เป็น semver
✅ homepage/repository URL
✅ environment constraints ถูกต้อง

Files:
✅ README.md - Usage examples, badges
✅ CHANGELOG.md - Version history
✅ LICENSE - Open source license
✅ example/ - Runnable example app
✅ test/ - Comprehensive tests

Code Quality:
✅ flutter analyze ผ่าน
✅ flutter test ผ่าน
✅ API documentation ครบ
✅ No deprecated API usage
✅ Dart 3 compatible

pub.dev Score Factors:
- Documentation: /// comments + README
- Platform support: ประกาศ platforms ที่รองรับ
- Dependencies: ใช้ latest stable versions
- Static analysis: ไม่มี errors/warnings
- License: open source license
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง Flutter Package ที่มี Custom Widget
2. เขียน README.md ที่มี examples ครบถ้วน
3. เพิ่ม dartdoc comments ให้ทุก public API

**ระดับกลาง:**
4. สร้าง Example App ที่ demo ทุก feature
5. เขียน Unit Tests ที่ coverage ≥ 80%
6. ตั้งค่า CI/CD ด้วย GitHub Actions

**ระดับสูง:**
7. Publish Package ไปยัง pub.dev (สร้างบัญชี pub.dev ก่อน)
8. Implement versioning strategy ด้วย semantic versioning
9. เพิ่ม Platform support สำหรับทุก platforms

---

## การนำทาง
- [← Part 38: Flutter Desktop](part_38.md)
- [→ Part 40: App Store Deployment](part_40.md)

# Part 33: CI/CD Pipeline
## ขั้นตอนที่ 321-330

---

## สารบัญ
1. [CI/CD Overview](#cicd-overview)
2. [GitHub Actions สำหรับ Flutter](#github-actions)
3. [Build Flavors (dev/staging/prod)](#build-flavors)
4. [Android Signing & Keystore](#android-signing)
5. [iOS Certificates & Provisioning](#ios-certificates)
6. [Fastlane Setup](#fastlane-setup)
7. [Codemagic CI/CD](#codemagic)
8. [Automatic Versioning](#automatic-versioning)
9. [Firebase App Distribution](#firebase-app-distribution)
10. [Play Store & App Store Deployment](#store-deployment)

---

## ขั้นตอนที่ 321: CI/CD Overview

CI/CD ย่อมาจาก Continuous Integration / Continuous Delivery (หรือ Deployment)

```
CI/CD Pipeline:
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Code    │→  │  Build   │→  │  Test    │→  │  Deploy  │→  │  Monitor │
│  Commit  │   │          │   │          │   │          │   │          │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘

CI = Continuous Integration:
- Code → Build → Test (อัตโนมัติ)
- ทุก commit ต้องผ่าน Build และ Tests

CD = Continuous Delivery:
- ส่ง Build ไปที่ Testing/Staging environment อัตโนมัติ
- Human approval ก่อน Production

CD = Continuous Deployment:
- Deploy ไป Production อัตโนมัติ (ถ้าผ่าน Tests ทุกอย่าง)
```

---

## ขั้นตอนที่ 322: GitHub Actions สำหรับ Flutter

### โครงสร้างไฟล์

```
.github/
└── workflows/
    ├── ci.yml           # Run tests on every PR
    ├── build-android.yml # Build Android APK/AAB
    ├── build-ios.yml    # Build iOS IPA
    └── deploy.yml       # Deploy to stores
```

### CI Workflow (Run Tests)

```yaml
# .github/workflows/ci.yml
name: CI - Tests & Analysis

on:
  push:
    branches: [main, develop, 'feature/**']
  pull_request:
    branches: [main, develop]

env:
  FLUTTER_VERSION: '3.16.0'
  JAVA_VERSION: '17'

jobs:
  analyze-and-test:
    name: Analyze & Test
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: ☕ Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: ${{ env.JAVA_VERSION }}

      - name: 🐦 Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: 'stable'
          cache: true

      - name: 📦 Get Dependencies
        run: flutter pub get

      - name: 🔨 Generate Code
        run: flutter pub run build_runner build --delete-conflicting-outputs

      - name: 🔍 Check Formatting
        run: dart format --output=none --set-exit-if-changed .

      - name: 🔬 Analyze
        run: flutter analyze --fatal-infos --fatal-warnings

      - name: 🧪 Run Tests with Coverage
        run: flutter test --coverage --reporter=github

      - name: 📊 Coverage Report
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info
          fail_ci_if_error: false

      - name: ✅ Coverage Threshold
        uses: VeryGoodOpenSource/very_good_coverage@v2
        with:
          path: coverage/lcov.info
          min_coverage: 75
```

### Android Build Workflow

```yaml
# .github/workflows/build-android.yml
name: Build Android

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      flavor:
        description: 'Build flavor'
        required: true
        default: 'staging'
        type: choice
        options: [dev, staging, prod]

jobs:
  build-android:
    name: Build Android APK/AAB
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: ☕ Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: 🐦 Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          channel: 'stable'
          cache: true

      - name: 📦 Get Dependencies
        run: flutter pub get

      - name: 🔑 Decode Keystore
        run: |
          echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      - name: 📝 Create key.properties
        run: |
          cat > android/key.properties << EOF
          storePassword=${{ secrets.KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=keystore.jks
          EOF

      - name: 🔨 Build APK (Debug)
        if: github.event.inputs.flavor == 'dev'
        run: |
          flutter build apk \
            --flavor dev \
            --dart-define=FLAVOR=dev \
            --dart-define=API_URL=https://dev-api.example.com

      - name: 🔨 Build AAB (Release)
        if: github.event.inputs.flavor != 'dev'
        run: |
          flutter build appbundle \
            --release \
            --flavor ${{ github.event.inputs.flavor || 'staging' }} \
            --dart-define=FLAVOR=${{ github.event.inputs.flavor || 'staging' }} \
            --dart-define=API_URL=${{ secrets.API_URL }}

      - name: 📤 Upload APK Artifact
        if: github.event.inputs.flavor == 'dev'
        uses: actions/upload-artifact@v3
        with:
          name: android-debug-apk
          path: build/app/outputs/flutter-apk/app-dev-debug.apk

      - name: 📤 Upload AAB Artifact
        if: github.event.inputs.flavor != 'dev'
        uses: actions/upload-artifact@v3
        with:
          name: android-release-aab
          path: build/app/outputs/bundle/*/app-*-release.aab
```

---

## ขั้นตอนที่ 323: Build Flavors

Build Flavors ช่วยให้เรามี Configuration แยกสำหรับ dev/staging/prod

### Android Flavors

```groovy
// android/app/build.gradle
android {
    // ...
    
    flavorDimensions "environment"
    
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            versionNameSuffix "-dev"
            resValue "string", "app_name", "MyApp Dev"
        }
        
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            versionNameSuffix "-staging"
            resValue "string", "app_name", "MyApp Staging"
        }
        
        prod {
            dimension "environment"
            resValue "string", "app_name", "MyApp"
        }
    }
    
    buildTypes {
        debug {
            debuggable true
            applicationIdSuffix ".debug"
        }
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### iOS Schemes

```bash
# สร้าง Schemes ใน Xcode:
# Product > Scheme > New Scheme
# ตั้งชื่อ: dev, staging, prod

# หรือใช้ xcconfig files:
# ios/Flutter/dev.xcconfig
# ios/Flutter/staging.xcconfig
# ios/Flutter/prod.xcconfig
```

```
# ios/Flutter/dev.xcconfig
#include "Generated.xcconfig"
BUNDLE_ID_SUFFIX=.dev
APP_NAME=MyApp Dev
API_URL=https://dev-api.example.com
```

### Dart Configuration

```dart
// lib/core/config/app_config.dart
enum Flavor { dev, staging, prod }

class AppConfig {
  static late final AppConfig _instance;
  
  final Flavor flavor;
  final String apiBaseUrl;
  final String appName;
  final bool enableLogging;
  final bool enableCrashlytics;
  final String sentryDsn;

  const AppConfig._({
    required this.flavor,
    required this.apiBaseUrl,
    required this.appName,
    required this.enableLogging,
    required this.enableCrashlytics,
    required this.sentryDsn,
  });

  factory AppConfig.dev() => const AppConfig._(
    flavor: Flavor.dev,
    apiBaseUrl: 'https://dev-api.example.com',
    appName: 'MyApp Dev',
    enableLogging: true,
    enableCrashlytics: false,
    sentryDsn: '',
  );

  factory AppConfig.staging() => const AppConfig._(
    flavor: Flavor.staging,
    apiBaseUrl: 'https://staging-api.example.com',
    appName: 'MyApp Staging',
    enableLogging: true,
    enableCrashlytics: true,
    sentryDsn: 'https://staging-sentry-dsn',
  );

  factory AppConfig.prod() => const AppConfig._(
    flavor: Flavor.prod,
    apiBaseUrl: 'https://api.example.com',
    appName: 'MyApp',
    enableLogging: false,
    enableCrashlytics: true,
    sentryDsn: 'https://prod-sentry-dsn',
  );

  static AppConfig get instance => _instance;

  static void initialize(Flavor flavor) {
    switch (flavor) {
      case Flavor.dev:
        _instance = AppConfig.dev();
        break;
      case Flavor.staging:
        _instance = AppConfig.staging();
        break;
      case Flavor.prod:
        _instance = AppConfig.prod();
        break;
    }
  }

  bool get isDev => flavor == Flavor.dev;
  bool get isStaging => flavor == Flavor.staging;
  bool get isProd => flavor == Flavor.prod;
}
```

```dart
// lib/main_dev.dart
import 'package:flutter/material.dart';
import 'core/config/app_config.dart';
import 'app.dart';

void main() {
  AppConfig.initialize(Flavor.dev);
  runApp(const App());
}
```

```dart
// lib/main_staging.dart
import 'package:flutter/material.dart';
import 'core/config/app_config.dart';
import 'app.dart';

void main() {
  AppConfig.initialize(Flavor.staging);
  runApp(const App());
}
```

```dart
// lib/main.dart (prod)
import 'package:flutter/material.dart';
import 'core/config/app_config.dart';
import 'app.dart';

void main() {
  AppConfig.initialize(Flavor.prod);
  runApp(const App());
}
```

```bash
# รัน app ด้วย flavor ต่างๆ
flutter run --flavor dev -t lib/main_dev.dart
flutter run --flavor staging -t lib/main_staging.dart
flutter run --flavor prod -t lib/main.dart

# Build
flutter build apk --flavor staging -t lib/main_staging.dart --release
flutter build appbundle --flavor prod -t lib/main.dart --release
```

---

## ขั้นตอนที่ 324: Android Signing & Keystore

### สร้าง Keystore

```bash
# สร้าง Keystore ใหม่
keytool -genkey -v \
  -keystore ~/my-release-key.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias my-key-alias

# ตรวจสอบ Keystore
keytool -list -v -keystore ~/my-release-key.jks

# แปลง Keystore เป็น Base64 สำหรับ GitHub Secrets
base64 -i ~/my-release-key.jks | tr -d '\n'
```

### android/key.properties

```properties
# android/key.properties (อย่า commit ไฟล์นี้!)
storePassword=your-store-password
keyPassword=your-key-password
keyAlias=my-key-alias
storeFile=/path/to/my-release-key.jks
```

### android/app/build.gradle - Signing Config

```groovy
// android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias'] ?: System.getenv('KEY_ALIAS')
            keyPassword keystoreProperties['keyPassword'] ?: System.getenv('KEY_PASSWORD')
            storeFile keystoreProperties['storeFile'] ? 
                file(keystoreProperties['storeFile']) : 
                file(System.getenv('KEYSTORE_FILE') ?: 'keystore.jks')
            storePassword keystoreProperties['storePassword'] ?: System.getenv('KEYSTORE_PASSWORD')
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
        }
    }
}
```

---

## ขั้นตอนที่ 325: iOS Certificates & Provisioning

### Setup สำหรับ Local Development

```bash
# 1. ใน Xcode: Preferences > Accounts > เพิ่ม Apple ID
# 2. Signing & Capabilities > Team เลือก Team ของคุณ

# สร้าง Certificate ผ่าน Certificate Assistant
# Keychain Access > Certificate Assistant > Request a Certificate From a Certificate Authority

# Download Provisioning Profile จาก developer.apple.com
```

### iOS Workflow ใน GitHub Actions

```yaml
# .github/workflows/build-ios.yml
name: Build iOS

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build-ios:
    name: Build iOS IPA
    runs-on: macos-latest

    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🐦 Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          channel: 'stable'

      - name: 📦 Get Dependencies
        run: flutter pub get

      - name: 🔑 Import Certificate
        uses: apple-actions/import-codesign-certs@v2
        with:
          p12-file-base64: ${{ secrets.IOS_CERTIFICATE_P12_BASE64 }}
          p12-password: ${{ secrets.IOS_CERTIFICATE_PASSWORD }}

      - name: 📋 Download Provisioning Profile
        uses: apple-actions/download-provisioning-profiles@v1
        with:
          bundle-id: com.example.myapp
          profile-type: IOS_APP_STORE
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_API_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_API_PRIVATE_KEY }}

      - name: 🔨 Build iOS
        run: |
          flutter build ios --release \
            --flavor prod \
            -t lib/main.dart \
            --no-codesign

      - name: 📦 Archive & Export IPA
        run: |
          xcodebuild \
            -workspace ios/Runner.xcworkspace \
            -scheme prod \
            -configuration Release \
            -archivePath build/ios/Runner.xcarchive \
            archive

          xcodebuild \
            -exportArchive \
            -archivePath build/ios/Runner.xcarchive \
            -exportPath build/ios/Runner.ipa \
            -exportOptionsPlist ios/ExportOptions.plist

      - name: 📤 Upload IPA
        uses: actions/upload-artifact@v3
        with:
          name: ios-release-ipa
          path: build/ios/Runner.ipa/
```

```xml
<!-- ios/ExportOptions.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>uploadBitcode</key>
    <false/>
    <key>compileBitcode</key>
    <false/>
    <key>uploadSymbols</key>
    <true/>
</dict>
</plist>
```

---

## ขั้นตอนที่ 326: Fastlane Setup

Fastlane ช่วย automate build/deploy process

### การติดตั้ง

```bash
# ติดตั้ง Fastlane
gem install fastlane

# หรือใช้ Bundler (แนะนำ)
gem install bundler
bundle init
echo 'gem "fastlane"' >> Gemfile
bundle install

# Initialize Fastlane สำหรับ Android
cd android && fastlane init

# Initialize Fastlane สำหรับ iOS
cd ios && fastlane init
```

### Android Fastfile

```ruby
# android/fastlane/Fastfile
default_platform(:android)

platform :android do
  
  before_all do
    # ทำงานก่อน lane ทุกตัว
    ensure_git_status_clean
  end

  desc "Run tests"
  lane :test do
    sh("cd .. && flutter test --coverage")
  end

  desc "Build debug APK"
  lane :build_debug do
    sh("cd .. && flutter build apk --flavor dev -t lib/main_dev.dart")
    puts "Debug APK built: build/app/outputs/flutter-apk/app-dev-debug.apk"
  end

  desc "Build staging APK"
  lane :build_staging do
    sh("cd .. && flutter build apk --flavor staging -t lib/main_staging.dart --release")
  end

  desc "Build release AAB"
  lane :build_release do
    sh("cd .. && flutter build appbundle --flavor prod -t lib/main.dart --release")
  end

  desc "Deploy to Firebase App Distribution"
  lane :deploy_firebase do
    build_staging

    firebase_app_distribution(
      app: ENV["FIREBASE_APP_ID_ANDROID"],
      firebase_cli_token: ENV["FIREBASE_CLI_TOKEN"],
      apk_path: "build/app/outputs/flutter-apk/app-staging-release.apk",
      groups: "testers, qa-team",
      release_notes: changelog_from_git_commits(
        commits_count: 10,
        pretty: "- %s"
      ),
    )
  end

  desc "Deploy to Play Store Internal Track"
  lane :deploy_internal do
    build_release

    supply(
      package_name: "com.example.myapp",
      aab: "build/app/outputs/bundle/prodRelease/app-prod-release.aab",
      track: "internal",
      json_key_data: ENV["GOOGLE_PLAY_JSON_KEY"],
    )
  end

  desc "Promote from Internal to Production"
  lane :promote_to_production do
    supply(
      package_name: "com.example.myapp",
      track: "internal",
      track_promote_to: "production",
      rollout: "0.1", # 10% rollout
      json_key_data: ENV["GOOGLE_PLAY_JSON_KEY"],
    )
  end

  error do |lane, exception|
    # ส่ง Slack notification เมื่อ Error
    slack(
      message: "Lane #{lane} failed with: #{exception.message}",
      slack_url: ENV["SLACK_WEBHOOK_URL"],
      success: false,
    )
  end
end
```

### iOS Fastfile

```ruby
# ios/fastlane/Fastfile
default_platform(:ios)

platform :ios do
  
  desc "Run tests"
  lane :test do
    scan(
      scheme: "prod",
      clean: true,
      output_directory: "test_output",
      output_types: "html, junit"
    )
  end

  desc "Build and sign app"
  lane :build do
    # Fetch certificates and profiles
    match(
      type: "appstore",
      app_identifier: "com.example.myapp",
      readonly: true,
    )

    # Build
    gym(
      scheme: "prod",
      export_method: "app-store",
      output_directory: "build/ios",
      output_name: "MyApp.ipa",
    )
  end

  desc "Deploy to TestFlight"
  lane :deploy_testflight do
    build

    pilot(
      api_key_path: "fastlane/api_key.json",
      ipa: "build/ios/MyApp.ipa",
      changelog: changelog_from_git_commits(
        commits_count: 10,
        pretty: "- %s"
      ),
      distribute_external: false,
      notify_external_testers: false,
    )
  end

  desc "Submit to App Store"
  lane :deploy_appstore do
    build

    deliver(
      api_key_path: "fastlane/api_key.json",
      ipa: "build/ios/MyApp.ipa",
      submit_for_review: true,
      automatic_release: false,
      skip_screenshots: true,
      skip_metadata: false,
    )
  end
end
```

---

## ขั้นตอนที่ 327: Codemagic CI/CD

Codemagic เป็น CI/CD platform ที่ออกแบบมาสำหรับ Flutter โดยเฉพาะ

```yaml
# codemagic.yaml
workflows:
  android-workflow:
    name: Android Workflow
    max_build_duration: 60
    environment:
      flutter: 3.16.0
      java: 17
      android_signing:
        - android_keystore
      groups:
        - google_play
        - firebase
      vars:
        PACKAGE_NAME: "com.example.myapp"
        GOOGLE_PLAY_TRACK: "internal"

    triggering:
      events:
        - push
        - pull_request
      branch_patterns:
        - pattern: main
          include: true
        - pattern: develop
          include: true

    scripts:
      - name: Get Flutter packages
        script: flutter pub get

      - name: Generate code
        script: flutter pub run build_runner build --delete-conflicting-outputs

      - name: Flutter analyze
        script: flutter analyze

      - name: Run tests
        script: |
          flutter test \
            --coverage \
            --reporter expanded

      - name: Build APK
        script: |
          flutter build appbundle \
            --release \
            --flavor prod \
            -t lib/main.dart \
            --build-number=$PROJECT_BUILD_NUMBER

    artifacts:
      - build/**/outputs/**/*.aab
      - build/**/outputs/**/mapping.txt
      - flutter_drive.log
      - coverage/lcov.info

    publishing:
      google_play:
        credentials: $GCLOUD_SERVICE_ACCOUNT_CREDENTIALS
        track: $GOOGLE_PLAY_TRACK
        submit_as_draft: false
      
      firebase:
        firebase_token: $FIREBASE_TOKEN
        android:
          app_id: $FIREBASE_ANDROID_APP_ID
          groups:
            - testers

  ios-workflow:
    name: iOS Workflow
    max_build_duration: 90
    environment:
      flutter: 3.16.0
      xcode: latest
      cocoapods: default
      ios_signing:
        distribution_type: app_store
        bundle_identifier: com.example.myapp

    scripts:
      - name: Set up code signing
        script: |
          keychain initialize
          app-store-connect fetch-signing-files \
            com.example.myapp \
            --type IOS_APP_STORE \
            --create
          keychain add-certificates
          xcode-project use-profiles

      - name: Get Flutter packages
        script: flutter pub get

      - name: Build iOS
        script: |
          flutter build ipa \
            --release \
            --flavor prod \
            -t lib/main.dart \
            --export-options-plist=/Users/builder/export_options.plist

    artifacts:
      - build/ios/ipa/*.ipa
      - /tmp/xcodebuild_logs/*.log

    publishing:
      app_store_connect:
        api_key: $APP_STORE_CONNECT_PRIVATE_KEY
        key_id: $APP_STORE_CONNECT_KEY_IDENTIFIER
        issuer_id: $APP_STORE_CONNECT_ISSUER_ID
        submit_to_testflight: true
```

---

## ขั้นตอนที่ 328: Automatic Versioning

```dart
// scripts/bump_version.dart
// รัน: dart scripts/bump_version.dart --type=patch

import 'dart:io';

enum BumpType { major, minor, patch, build }

void main(List<String> args) {
  final bumpType = _parseBumpType(args);
  final pubspecFile = File('pubspec.yaml');
  
  if (!pubspecFile.existsSync()) {
    print('Error: pubspec.yaml not found');
    exit(1);
  }
  
  final content = pubspecFile.readAsStringSync();
  final versionMatch = RegExp(r'^version: (\d+)\.(\d+)\.(\d+)\+(\d+)', multiLine: true)
      .firstMatch(content);
  
  if (versionMatch == null) {
    print('Error: version not found in pubspec.yaml');
    exit(1);
  }
  
  int major = int.parse(versionMatch.group(1)!);
  int minor = int.parse(versionMatch.group(2)!);
  int patch = int.parse(versionMatch.group(3)!);
  int build = int.parse(versionMatch.group(4)!);
  
  switch (bumpType) {
    case BumpType.major:
      major++;
      minor = 0;
      patch = 0;
      build++;
      break;
    case BumpType.minor:
      minor++;
      patch = 0;
      build++;
      break;
    case BumpType.patch:
      patch++;
      build++;
      break;
    case BumpType.build:
      build++;
      break;
  }
  
  final newVersion = '$major.$minor.$patch+$build';
  final newContent = content.replaceFirst(
    versionMatch.group(0)!,
    'version: $newVersion',
  );
  
  pubspecFile.writeAsStringSync(newContent);
  print('Version bumped to: $newVersion');
  
  // อัพเดท CHANGELOG.md
  _updateChangelog(newVersion);
}

void _updateChangelog(String version) {
  final changelogFile = File('CHANGELOG.md');
  final today = DateTime.now().toIso8601String().split('T')[0];
  
  final newEntry = '''## [$version] - $today

### Added
- 

### Changed
- 

### Fixed
- 

''';
  
  if (changelogFile.existsSync()) {
    final content = changelogFile.readAsStringSync();
    changelogFile.writeAsStringSync(newEntry + content);
  } else {
    changelogFile.writeAsStringSync('# Changelog\n\n$newEntry');
  }
  
  print('Updated CHANGELOG.md');
}

BumpType _parseBumpType(List<String> args) {
  for (final arg in args) {
    if (arg.startsWith('--type=')) {
      final type = arg.split('=')[1];
      switch (type) {
        case 'major': return BumpType.major;
        case 'minor': return BumpType.minor;
        case 'patch': return BumpType.patch;
        case 'build': return BumpType.build;
      }
    }
  }
  return BumpType.patch; // default
}
```

```yaml
# .github/workflows/release.yml
name: Release

on:
  workflow_dispatch:
    inputs:
      version_type:
        description: 'Version bump type'
        required: true
        default: 'patch'
        type: choice
        options: [major, minor, patch]

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'

      - name: Configure Git
        run: |
          git config user.name "GitHub Actions Bot"
          git config user.email "actions@github.com"

      - name: Bump Version
        run: dart scripts/bump_version.dart --type=${{ github.event.inputs.version_type }}

      - name: Get New Version
        id: version
        run: |
          VERSION=$(grep 'version:' pubspec.yaml | sed 's/version: //')
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Commit Version Bump
        run: |
          git add pubspec.yaml CHANGELOG.md
          git commit -m "chore: bump version to ${{ steps.version.outputs.version }}"
          git tag v${{ steps.version.outputs.version }}
          git push origin main --tags

      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.version.outputs.version }}
          release_name: Release v${{ steps.version.outputs.version }}
          body_path: CHANGELOG.md
          draft: false
          prerelease: false
```

---

## ขั้นตอนที่ 329: Firebase App Distribution

```yaml
# .github/workflows/firebase-distribute.yml
name: Firebase App Distribution

on:
  push:
    branches: [develop]

jobs:
  distribute:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Decode Keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks
          cat > android/key.properties << EOF
          storePassword=${{ secrets.KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=keystore.jks
          EOF
      
      - name: Build APK
        run: |
          flutter build apk \
            --release \
            --flavor staging \
            -t lib/main_staging.dart
      
      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          token: ${{ secrets.FIREBASE_CLI_TOKEN }}
          groups: testers
          file: build/app/outputs/flutter-apk/app-staging-release.apk
          releaseNotes: |
            Branch: ${{ github.ref_name }}
            Commit: ${{ github.sha }}
            Message: ${{ github.event.head_commit.message }}
```

```bash
# ติดตั้ง Firebase CLI
npm install -g firebase-tools

# Login
firebase login:ci

# สร้าง App Distribution
firebase appdistribution:distribute \
  build/app/outputs/flutter-apk/app-staging-release.apk \
  --app YOUR_APP_ID \
  --groups "testers" \
  --release-notes "New build from CI"
```

---

## ขั้นตอนที่ 330: Play Store & App Store Deployment

### Google Play Store

```yaml
# .github/workflows/deploy-playstore.yml
name: Deploy to Play Store

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Setup dependencies
        run: flutter pub get
      
      - name: Setup Signing
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks
          cat > android/key.properties << EOF
          storePassword=${{ secrets.KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=keystore.jks
          EOF
      
      - name: Build App Bundle
        run: |
          flutter build appbundle \
            --release \
            --flavor prod \
            -t lib/main.dart
      
      - name: Deploy to Play Store
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.GOOGLE_SERVICE_ACCOUNT_JSON }}
          packageName: com.example.myapp
          releaseFiles: build/app/outputs/bundle/prodRelease/app-prod-release.aab
          track: internal
          status: completed
          whatsNewDirectory: distribution/whatsnew
          mappingFile: build/app/outputs/mapping/prodRelease/mapping.txt
```

```
distribution/
└── whatsnew/
    ├── whatsnew-en-US
    ├── whatsnew-th-TH
    └── whatsnew-ja-JP
```

```
# distribution/whatsnew/whatsnew-en-US
Bug fixes and performance improvements
- Fixed login issue
- Improved loading speed
- Updated UI components
```

### Apple App Store

```yaml
# .github/workflows/deploy-appstore.yml
name: Deploy to App Store

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: macos-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Setup dependencies
        run: flutter pub get
      
      - name: Import Certificate
        uses: apple-actions/import-codesign-certs@v2
        with:
          p12-file-base64: ${{ secrets.APPLE_CERTIFICATE_P12 }}
          p12-password: ${{ secrets.APPLE_CERTIFICATE_PASSWORD }}
      
      - name: Download Provisioning Profile
        uses: apple-actions/download-provisioning-profiles@v1
        with:
          bundle-id: com.example.myapp
          profile-type: IOS_APP_STORE
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_PRIVATE_KEY }}
      
      - name: Build IPA
        run: |
          flutter build ipa \
            --release \
            --flavor prod \
            -t lib/main.dart \
            --export-options-plist=ios/ExportOptions.plist
      
      - name: Upload to App Store Connect
        uses: apple-actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/MyApp.ipa
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_PRIVATE_KEY }}
```

---

## สรุป (Summary)

```
CI/CD Pipeline ที่สมบูรณ์:

Code Push
    ↓
┌─────────────────────────────────────┐
│ CI (GitHub Actions / Codemagic)     │
│ 1. Checkout code                    │
│ 2. Setup Flutter & Java             │
│ 3. Get dependencies                 │
│ 4. Analyze code                     │
│ 5. Run tests (unit + widget)        │
│ 6. Check coverage threshold         │
└─────────────┬───────────────────────┘
              ↓ (if tests pass)
┌─────────────────────────────────────┐
│ CD - Build                          │
│ 1. Setup signing                    │
│ 2. Build APK/AAB (Android)          │
│ 3. Build IPA (iOS)                  │
│ 4. Bump version                     │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ CD - Deploy                         │
│ develop → Firebase App Distribution │
│ main → TestFlight / Play Internal   │
│ tags → App Store / Play Store       │
└─────────────────────────────────────┘

GitHub Secrets ที่ต้องตั้งค่า:
Android:
- ANDROID_KEYSTORE_BASE64
- KEYSTORE_PASSWORD
- KEY_PASSWORD  
- KEY_ALIAS
- GOOGLE_PLAY_JSON_KEY

iOS:
- IOS_CERTIFICATE_P12_BASE64
- IOS_CERTIFICATE_PASSWORD
- APPSTORE_ISSUER_ID
- APPSTORE_API_KEY_ID
- APPSTORE_API_PRIVATE_KEY

Firebase:
- FIREBASE_APP_ID
- FIREBASE_CLI_TOKEN
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง GitHub Actions workflow ที่ run tests ทุก PR
2. ตั้งค่า Build Flavors (dev/prod) สำหรับ Android
3. สร้าง version bump script

**ระดับกลาง:**
4. Setup Android Signing ด้วย GitHub Secrets
5. Deploy APK ไปยัง Firebase App Distribution
6. สร้าง Fastlane lane สำหรับ build และ deploy

**ระดับสูง:**
7. Setup complete CI/CD pipeline ทั้ง Android และ iOS
8. Implement automatic versioning ด้วย git tags
9. Setup staged rollout บน Google Play Store

---

## การนำทาง
- [← Part 32: TDD & Testing](part_32.md)
- [→ Part 34: Performance Optimization](part_34.md)

# Part 40: App Store Deployment
## ขั้นตอนที่ 391-400

---

## สารบัญ
1. [App Store Deployment Overview](#overview)
2. [Google Play Store Setup](#google-play-setup)
3. [Android Signing & Release Build](#android-signing)
4. [App Bundle & Listing](#app-bundle--listing)
5. [Play Store Ratings & Content](#ratings--content)
6. [Staged Rollout](#staged-rollout)
7. [Apple App Store Setup](#apple-app-store)
8. [iOS Provisioning & Certificates](#ios-provisioning)
9. [TestFlight & App Store Connect](#testflight)
10. [Version Management & Release Notes](#version-management)

---

## ขั้นตอนที่ 391: App Store Deployment Overview

```
Mobile App Distribution:

Google Play Store (Android):
  Developer Account: $25 one-time fee
  Review Time: hours to days
  Distribution: Global + Staged Rollout
  Format: APK or AAB (App Bundle)

Apple App Store (iOS):
  Developer Account: $99/year
  Review Time: 1-3 days
  Distribution: TestFlight → App Store
  Format: IPA

Alternative Channels:
  Firebase App Distribution: Internal testing
  Samsung Galaxy Store: Samsung devices
  Huawei AppGallery: Huawei devices
  
Release Strategy:
  Alpha → Beta → Production
  Internal → QA → Closed Beta → Open Beta → Production
```

---

## ขั้นตอนที่ 392: Google Play Store Setup

### สร้าง Developer Account

```
1. ไปที่ https://play.google.com/apps/publish/
2. เข้าสู่ระบบด้วย Google Account
3. ชำระ $25 (one-time fee)
4. ยืนยัน Identity

สร้าง App:
1. กด "Create app"
2. กรอก App name
3. เลือก Default language
4. เลือก App or Game
5. เลือก Free or Paid
6. ยอมรับ Developer Distribution Agreement
```

### Google Play Console Setup

```dart
// pubspec.yaml - ตั้ง package name
name: my_app
# ...

// android/app/build.gradle
android {
    namespace "com.mycompany.myapp"
    
    defaultConfig {
        applicationId "com.mycompany.myapp"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }
}
```

---

## ขั้นตอนที่ 393: Android Signing & Release Build

### สร้าง Keystore

```bash
# สร้าง Keystore (ทำครั้งเดียว - เก็บไว้อย่างดี!)
keytool -genkey -v \
  -keystore ~/upload-keystore.jks \
  -storetype JKS \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias upload

# ข้อมูลที่จะถามระหว่างสร้าง Keystore:
# - First and Last Name: Your Name
# - Organizational Unit: Development
# - Organization: My Company
# - City: Bangkok
# - State: Bangkok
# - Country Code: TH
# - Password: [strong password]
```

### android/key.properties

```properties
# android/key.properties (อย่า commit ไฟล์นี้!)
storePassword=your-strong-password
keyPassword=your-key-password
keyAlias=upload
storeFile=/Users/yourname/upload-keystore.jks
```

```
# .gitignore
android/key.properties
*.jks
*.keystore
```

### android/app/build.gradle

```groovy
// android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    namespace "com.mycompany.myapp"
    compileSdkVersion flutter.compileSdkVersion
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    defaultConfig {
        applicationId "com.mycompany.myapp"
        minSdkVersion flutter.minSdkVersion
        targetSdkVersion flutter.targetSdkVersion
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
        multiDexEnabled true
    }

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? 
                file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            versionNameSuffix "-debug"
        }
        release {
            signingConfig signingConfigs.release
            
            // Minify & Shrink
            minifyEnabled true
            shrinkResources true
            
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                         'proguard-rules.pro'
        }
    }
}
```

### Build Release App Bundle

```bash
# Build Android App Bundle (แนะนำสำหรับ Play Store)
flutter build appbundle --release

# Output: build/app/outputs/bundle/release/app-release.aab

# Build APK (สำหรับ direct install)
flutter build apk --release

# Build fat APK (รองรับทุก architecture)
flutter build apk --release --split-per-abi

# Output APKs:
# build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk  (32-bit ARM)
# build/app/outputs/flutter-apk/app-arm64-v8a-release.apk    (64-bit ARM)
# build/app/outputs/flutter-apk/app-x86_64-release.apk       (x86_64)
```

### ตรวจสอบ Build

```bash
# ตรวจสอบ size
ls -la build/app/outputs/bundle/release/

# ตรวจสอบว่า sign ถูกต้อง
jarsigner -verify -verbose -certs build/app/outputs/bundle/release/app-release.aab

# ดู bundletool analysis
java -jar bundletool.jar validate --bundle=build/app/outputs/bundle/release/app-release.aab
```

---

## ขั้นตอนที่ 394: App Bundle & Store Listing

### Store Listing Assets

```
Play Store Listing Requirements:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

App Icon:
  - ขนาด 512x512 px
  - Format: PNG, 32-bit
  - Background ไม่โปร่งใส

Feature Graphic:
  - ขนาด 1024x500 px
  - Format: JPG/PNG
  - ใช้แสดงใน Play Store homepage

Screenshots (ต้องมีอย่างน้อย 2 รูป):
  Phone: 320-3840 px, 16:9 ratio
  Tablet 7": 320-3840 px
  Tablet 10": 320-3840 px

Short Description: สูงสุด 80 characters
Full Description: สูงสุด 4000 characters

Video (Optional):
  - YouTube URL
  - ไม่มี ads, ไม่มีการรีวิวสินค้า
```

### การเขียน Store Listing

```
Short Description (80 chars):
"Shop smarter with AI-powered recommendations and fast checkout"

Full Description (4000 chars):
📱 My App - Your Smart Shopping Companion

Discover a better way to shop with My App! Our AI-powered platform 
personalizes your shopping experience, helping you find exactly what 
you need at the best prices.

🎯 KEY FEATURES:
• Smart product recommendations based on your preferences
• One-tap checkout with multiple payment options
• Real-time order tracking
• Exclusive member discounts and offers
• Wishlist and price drop alerts

🔒 SECURE & TRUSTED:
• Bank-level encryption for all transactions
• PCI DSS compliant payment processing
• 30-day hassle-free returns

⚡ PERFORMANCE:
• Blazing fast app loading
• Optimized for both 4G and WiFi
• Offline browsing capability

📞 SUPPORT:
• 24/7 customer support
• In-app chat support
• Comprehensive FAQ section

Download now and get 20% off your first order!

PERMISSIONS USED:
• Camera: For product scanning and AR try-on
• Location: For finding nearby stores
• Notifications: For order updates and deals

Privacy Policy: https://myapp.com/privacy
Terms of Service: https://myapp.com/terms
```

---

## ขั้นตอนที่ 395: Play Store Ratings & Content

### Content Rating

```
Google Play Content Rating:
1. กรอก Content Rating Questionnaire ใน Play Console
2. ตอบคำถามเกี่ยวกับ content ในแอป
3. ระบบจะ assign rating อัตโนมัติ

Rating Categories:
E      = Everyone
E10+   = Everyone 10 and older
T      = Teen
M      = Mature 17+
AO     = Adults Only 18+

สิ่งที่กระทบ Rating:
- Violence
- Sexual content
- Language
- Substances (drugs, alcohol)
- Gambling
- User interaction (social features)
```

### Target Audience

```dart
// pubspec.yaml
// กำหนด target age ใน Play Console:
// 1. Store presence > Store settings
// 2. App category > Primary category
// 3. Target audience and content > Age groups

// ถ้า target = เด็ก (<13)
// ต้องทำตาม:
// - COPPA compliance (US)
// - ไม่มี targeted ads
// - ไม่มี in-app purchases
// - Privacy Policy ที่ครอบคลุม
```

### App Permissions

```dart
// android/app/src/main/AndroidManifest.xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  
  <!-- Network -->
  <uses-permission android:name="android.permission.INTERNET"/>
  
  <!-- Camera (ต้องระบุใน Play Console) -->
  <uses-permission android:name="android.permission.CAMERA"/>
  
  <!-- Location (เลือกระดับที่ต้องการ) -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
  
  <!-- Storage (Android 12 ขึ้นไปใช้ Media permissions) -->
  <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
  
  <!-- Notifications -->
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
  
  <!-- Vibration -->
  <uses-permission android:name="android.permission.VIBRATE"/>
  
  <!-- Foreground Service (สำหรับ background tasks) -->
  <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
  
</manifest>
```

---

## ขั้นตอนที่ 396: Staged Rollout

```
Staged Rollout ช่วยลด Risk ของ Production Release:

Tracks:
1. Internal Testing  → ทีมภายใน (≤100 users)
2. Closed Testing   → Specific users/groups
3. Open Testing     → Public beta
4. Production       → All users

Staged Rollout (Production):
10% → monitor → 25% → monitor → 50% → monitor → 100%

เมื่อไหร่ควร Halt Rollout:
- Crash rate เพิ่มขึ้นอย่างมาก
- ANR rate เพิ่มขึ้น
- Negative reviews เพิ่ม
- Major bug ที่กระทบผู้ใช้ส่วนใหญ่
```

```bash
# ใช้ Fastlane สำหรับ Staged Rollout

# Fastfile
lane :deploy_production do |options|
  rollout = options[:rollout] || 0.1 # 10% default
  
  supply(
    package_name: 'com.mycompany.myapp',
    aab: 'build/app/outputs/bundle/prodRelease/app-prod-release.aab',
    track: 'production',
    rollout: rollout.to_s,
    json_key_data: ENV['GOOGLE_PLAY_JSON_KEY'],
    version_code: ENV['VERSION_CODE'],
    version_name: ENV['VERSION_NAME'],
    release_status: 'inProgress',
  )
end

# Promote to full production
lane :promote_full do
  supply(
    package_name: 'com.mycompany.myapp',
    track: 'production',
    track_promote_to: 'production',
    rollout: '1.0', # 100%
    json_key_data: ENV['GOOGLE_PLAY_JSON_KEY'],
    release_status: 'completed',
  )
end
```

---

## ขั้นตอนที่ 397: Apple App Store Setup

### Apple Developer Account

```
1. ไปที่ https://developer.apple.com
2. Enroll in Apple Developer Program ($99/year)
3. รอการ Verify Identity (1-2 วัน)

ใน App Store Connect:
1. ไปที่ https://appstoreconnect.apple.com
2. My Apps > + > New App
3. กรอกข้อมูล:
   - Platforms: iOS
   - Name: My App
   - Primary Language: English
   - Bundle ID: com.mycompany.myapp
   - SKU: MYAPP001
   - User Access: Full Access
```

### App Store Listing

```
App Store Listing Requirements:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

App Icon:
  - 1024x1024 px
  - Format: PNG
  - ไม่มี alpha channel
  - ไม่มี rounded corners (App Store ทำให้เอง)

Screenshots:
  iPhone 6.7" (required): 1290x2796 px
  iPhone 5.5" (required): 1242x2208 px
  iPad Pro 12.9" (if iPad): 2048x2732 px
  
Preview Videos (optional):
  - 15-30 seconds
  - MP4 format

Name: สูงสุด 30 characters
Subtitle: สูงสุด 30 characters
Description: สูงสุด 4000 characters
Keywords: สูงสุด 100 characters
Support URL: ต้องมี
Privacy Policy URL: ต้องมี
```

---

## ขั้นตอนที่ 398: iOS Provisioning & Certificates

### สร้าง Certificates

```bash
# ใน Xcode: Preferences > Accounts > Manage Certificates
# หรือใน developer.apple.com:

# 1. Apple Distribution Certificate
#    - ใช้สำหรับ App Store submission
#    - สร้างใน: Certificates, Identifiers & Profiles > Certificates

# 2. สร้าง App ID
#    - ใน: Certificates, Identifiers & Profiles > Identifiers
#    - Bundle ID: com.mycompany.myapp

# 3. สร้าง Provisioning Profile
#    - ใน: Profiles
#    - Type: App Store Distribution
#    - เลือก App ID และ Distribution Certificate
```

### ios/Runner.xcodeproj Settings

```
ใน Xcode:
1. เปิด ios/Runner.xcworkspace
2. Runner target > General:
   - Display Name: My App
   - Bundle Identifier: com.mycompany.myapp
   - Version: 1.0.0
   - Build: 1
3. Signing & Capabilities:
   - Team: Your Team
   - Signing Certificate: Apple Distribution
   - Provisioning Profile: [Your App Store Profile]
```

### Build IPA

```bash
# Build iOS Release (ไม่ sign)
flutter build ios --release --no-codesign

# Archive ด้วย Xcode Command Line
xcodebuild -workspace ios/Runner.xcworkspace \
  -scheme Runner \
  -sdk iphoneos \
  -configuration Release \
  archive \
  -archivePath build/ios/Runner.xcarchive

# Export IPA
xcodebuild -exportArchive \
  -archivePath build/ios/Runner.xcarchive \
  -exportOptionsPlist ios/ExportOptions.plist \
  -exportPath build/ios/Runner.ipa
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
  <key>signingCertificate</key>
  <string>Apple Distribution</string>
  <key>signingStyle</key>
  <string>manual</string>
  <key>provisioningProfiles</key>
  <dict>
    <key>com.mycompany.myapp</key>
    <string>My App Store Profile</string>
  </dict>
</dict>
</plist>
```

---

## ขั้นตอนที่ 399: TestFlight & App Store Connect

### TestFlight Internal Testing

```bash
# Upload ด้วย Transporter app หรือ altool
xcrun altool \
  --upload-app \
  --type ios \
  --file build/ios/Runner.ipa/MyApp.ipa \
  --apiKey YOUR_API_KEY \
  --apiIssuer YOUR_ISSUER_ID

# หรือใช้ Fastlane
fastlane pilot upload \
  --api_key_path fastlane/api_key.json \
  --ipa build/ios/Runner.ipa/MyApp.ipa
```

### App Store Connect API Key

```json
// fastlane/api_key.json
{
  "key_id": "YOUR_KEY_ID",
  "issuer_id": "YOUR_ISSUER_ID",
  "key": "-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----",
  "duration": 1200,
  "in_house": false
}
```

### TestFlight Build Distribution

```
TestFlight Distribution Steps:
1. Upload build ไปยัง App Store Connect
2. รอ Processing (15-30 นาที)
3. Internal Testers:
   - เพิ่ม testers โดยใช้ email
   - ส่ง TestFlight invitation
   - ไม่ต้องรอ Review
   
4. External Testers:
   - กรอก test information
   - Submit for Beta App Review (1-2 วัน)
   - เพิ่ม testers หรือ groups
   - Public link (ถ้าต้องการ)

TestFlight Notes สำหรับ Version:
- Build Purpose: Description of changes
- Feedback Email: Your feedback email
```

### Submit for App Store Review

```dart
// ตรวจสอบก่อน Submit

// 1. App Completeness
// ✅ ทุก required screenshot มีครบ
// ✅ Description ครบถ้วน
// ✅ Keywords ใส่ครบ
// ✅ Support URL ทำงาน
// ✅ Privacy Policy URL ทำงาน

// 2. Content Review
// ✅ ไม่มี placeholder content
// ✅ ทุก feature ในแอปทำงานได้
// ✅ Login credentials (ถ้าต้องการ) ใส่ใน Notes to Reviewer

// 3. Technical Requirements
// ✅ ไม่มี crashes
// ✅ ทำงานบน iOS เวอร์ชัน minimum ที่ประกาศ
// ✅ ทุก permissions มีเหตุผล
// ✅ Privacy Policy ครอบคลุม data ที่ collect

// ส่ง Review
// App Store Connect > App > Prepare for Submission
// > Submit for Review
```

---

## ขั้นตอนที่ 400: Version Management & Release Notes

### Version Management ใน Flutter

```yaml
# pubspec.yaml
version: 1.2.3+45
# 1.2.3 = versionName (แสดงผู้ใช้เห็น)
# 45    = versionCode (ต้องเพิ่มทุก release)

# versionCode บน Android ต้องเพิ่มทุก upload
# CFBundleVersion บน iOS ต้องเพิ่มทุก upload
```

```dart
// lib/core/config/version_config.dart
class VersionConfig {
  static const String appVersion = String.fromEnvironment(
    'APP_VERSION',
    defaultValue: '1.0.0',
  );
  
  static const int buildNumber = int.fromEnvironment(
    'BUILD_NUMBER',
    defaultValue: 1,
  );
  
  static String get fullVersion => '$appVersion ($buildNumber)';
}
```

```bash
# Build ด้วย version จาก environment
flutter build apk --release \
  --dart-define=APP_VERSION=1.2.3 \
  --dart-define=BUILD_NUMBER=45

# หรือกำหนดใน pubspec.yaml ก่อน build
flutter build apk --release \
  --build-name=1.2.3 \
  --build-number=45
```

### Release Notes Strategy

```dart
// lib/features/whats_new/whats_new_dialog.dart
class WhatsNewDialog extends StatelessWidget {
  final String currentVersion;
  
  const WhatsNewDialog({super.key, required this.currentVersion});

  static Future<void> showIfNeeded(BuildContext context) async {
    final prefs = await SharedPreferences.getInstance();
    final lastShownVersion = prefs.getString('whats_new_shown_version') ?? '';
    
    final currentVersion = await _getCurrentVersion();
    
    if (lastShownVersion != currentVersion) {
      if (context.mounted) {
        await showDialog(
          context: context,
          builder: (_) => WhatsNewDialog(currentVersion: currentVersion),
        );
        
        await prefs.setString('whats_new_shown_version', currentVersion);
      }
    }
  }

  static Future<String> _getCurrentVersion() async {
    final info = await PackageInfo.fromPlatform();
    return info.version;
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text("What's New in $currentVersion"),
      content: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          mainAxisSize: MainAxisSize.min,
          children: [
            _buildFeature(
              icon: Icons.speed,
              title: 'Faster Performance',
              description: 'App loads 30% faster than before',
            ),
            _buildFeature(
              icon: Icons.dark_mode,
              title: 'Dark Mode',
              description: 'Brand new dark mode for night browsing',
            ),
            _buildFeature(
              icon: Icons.bug_report,
              title: 'Bug Fixes',
              description: 'Fixed several reported issues',
            ),
          ],
        ),
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('Got it!'),
        ),
      ],
    );
  }

  Widget _buildFeature({
    required IconData icon,
    required String title,
    required String description,
  }) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 16),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Icon(icon, size: 24),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(title, style: const TextStyle(fontWeight: FontWeight.bold)),
                Text(description),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

### Release Notes สำหรับ Stores

```markdown
// Play Store - What's New (500 chars per language)
// English
Version 1.2.0 - What's New:
• 🚀 30% faster app loading
• 🌙 Dark mode support  
• 🔍 Improved search with filters
• 🛒 One-tap checkout
• 🐛 Fixed checkout crash issue
• 📱 Support for Android 14

// Thai
เวอร์ชัน 1.2.0 - อัปเดตใหม่:
• 🚀 โหลดแอปเร็วขึ้น 30%
• 🌙 รองรับ Dark Mode
• 🔍 ปรับปรุงการค้นหาพร้อม Filters
• 🛒 ชำระเงินด้วยคลิกเดียว
• 🐛 แก้ไขปัญหา Crash ตอนชำระเงิน
• 📱 รองรับ Android 14
```

### Automated Release Notes

```dart
// scripts/generate_release_notes.dart
void main(List<String> args) async {
  // อ่าน CHANGELOG.md
  final changelogFile = File('CHANGELOG.md');
  final content = await changelogFile.readAsString();
  
  // Parse version ล่าสุด
  final versionMatch = RegExp(
    r'## \[(\d+\.\d+\.\d+)\] - (\d{4}-\d{2}-\d{2})([\s\S]*?)(?=## \[|\Z)',
  ).firstMatch(content);
  
  if (versionMatch == null) {
    print('No version found');
    return;
  }
  
  final version = versionMatch.group(1);
  final changes = versionMatch.group(3)?.trim() ?? '';
  
  // Format สำหรับ Play Store (500 chars)
  final formatted = _formatForPlayStore(changes);
  
  // Save
  await Directory('distribution/whatsnew').create(recursive: true);
  await File('distribution/whatsnew/whatsnew-en-US').writeAsString(formatted);
  
  print('Release notes generated for version $version');
  print(formatted);
}

String _formatForPlayStore(String changes) {
  final lines = changes.split('\n')
      .where((line) => line.trim().isNotEmpty)
      .map((line) => line.replaceFirst(RegExp(r'^-\s*'), '• '))
      .toList();
  
  var result = lines.join('\n');
  
  // ตัดให้ไม่เกิน 500 chars
  if (result.length > 500) {
    result = result.substring(0, 497) + '...';
  }
  
  return result;
}
```

### Complete Release Checklist

```
Pre-Release Checklist:
━━━━━━━━━━━━━━━━━━━━━

Development:
✅ All features implemented and tested
✅ No known critical bugs
✅ Performance requirements met
✅ Accessibility requirements met

Testing:
✅ Unit tests pass
✅ Widget tests pass
✅ Integration tests pass
✅ Manual QA completed
✅ Tested on multiple devices
✅ Tested on minimum supported OS versions

Android:
✅ Keystore signed correctly
✅ Release build tested
✅ ProGuard/R8 tested
✅ App Bundle created
✅ Version code incremented

iOS:
✅ Certificates valid
✅ Provisioning profiles updated
✅ TestFlight build tested
✅ IPA created and verified

Store Submission:
✅ Screenshots updated (if UI changed)
✅ App description updated
✅ What's New / Release Notes written
✅ Keywords optimized (App Store)
✅ Privacy policy updated if needed

Post-Release:
✅ Monitor crash rates
✅ Monitor reviews
✅ Monitor Staged Rollout metrics
✅ Respond to user feedback
✅ Tag release in Git
```

---

## Workshop: Complete Release Process

```bash
#!/bin/bash
# scripts/release.sh

set -e  # Exit on error

echo "Starting release process..."

# 1. Check git status
if [ -n "$(git status --porcelain)" ]; then
  echo "Error: Working directory is not clean"
  exit 1
fi

# 2. Run tests
echo "Running tests..."
flutter test --coverage

# 3. Check coverage
echo "Checking coverage..."
COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep "lines" | awk '{print $2}' | tr -d '%')
if (( $(echo "$COVERAGE < 75" | bc -l) )); then
  echo "Error: Coverage is only $COVERAGE% (minimum 75%)"
  exit 1
fi

# 4. Get version from pubspec
VERSION=$(grep 'version:' pubspec.yaml | awk '{print $2}' | cut -d'+' -f1)
BUILD=$(grep 'version:' pubspec.yaml | awk '{print $2}' | cut -d'+' -f2)

echo "Building version $VERSION ($BUILD)..."

# 5. Build Android
echo "Building Android AAB..."
flutter build appbundle --release \
  --flavor prod \
  -t lib/main.dart \
  --build-name=$VERSION \
  --build-number=$BUILD

# 6. Build iOS (macOS only)
if [[ "$OSTYPE" == "darwin"* ]]; then
  echo "Building iOS IPA..."
  flutter build ipa --release \
    --flavor prod \
    -t lib/main.dart \
    --build-name=$VERSION \
    --build-number=$BUILD
fi

# 7. Generate release notes
echo "Generating release notes..."
dart scripts/generate_release_notes.dart

# 8. Tag release
echo "Tagging release..."
git tag -a "v$VERSION" -m "Release version $VERSION"
git push origin "v$VERSION"

echo "Release process completed successfully!"
echo "Version: $VERSION ($BUILD)"
echo ""
echo "Next steps:"
echo "1. Upload Android AAB to Play Console"
echo "2. Upload iOS IPA via App Store Connect"
echo "3. Set up Staged Rollout"
echo "4. Monitor crash rates and reviews"
```

---

## สรุป (Summary)

```
App Store Deployment Summary:

Google Play Store:
┌─────────────────────────────────────────────────────┐
│ 1. สร้าง Keystore & sign APK/AAB                    │
│ 2. Build App Bundle (flutter build appbundle)       │
│ 3. Create app listing & upload assets               │
│ 4. Set up Content Rating                            │
│ 5. Upload AAB ไปยัง Internal Testing track          │
│ 6. Test & promote ไป Production                     │
│ 7. ใช้ Staged Rollout (10% → 50% → 100%)            │
└─────────────────────────────────────────────────────┘

Apple App Store:
┌─────────────────────────────────────────────────────┐
│ 1. สร้าง Distribution Certificate                   │
│ 2. สร้าง Provisioning Profile                       │
│ 3. Build & Archive (Xcode หรือ flutter build ipa)   │
│ 4. Upload ไปยัง App Store Connect                   │
│ 5. Add TestFlight testers                           │
│ 6. Submit for App Store Review                      │
│ 7. Release manually or automatically               │
└─────────────────────────────────────────────────────┘

Version Strategy:
- ใช้ Semantic Versioning (MAJOR.MINOR.PATCH)
- versionCode/BuildNumber ต้องเพิ่มทุก upload
- เขียน Release Notes ที่ชัดเจน
- ใช้ Staged Rollout สำหรับ major releases
- Monitor metrics หลัง release

CI/CD Integration:
- Automate build และ sign ด้วย GitHub Actions
- Deploy to Firebase App Distribution สำหรับ testing
- Auto-deploy ไป store เมื่อ tag version
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง Android Keystore และ sign app
2. Build Release App Bundle
3. สร้าง App Store Listing (screenshots, description)

**ระดับกลาง:**
4. Setup Internal Testing บน Google Play Console
5. Setup TestFlight Internal Testing บน App Store Connect
6. เขียน Release Notes สำหรับทั้ง 2 stores

**ระดับสูง:**
7. Setup Staged Rollout บน Google Play Store
8. Automate release process ด้วย GitHub Actions + Fastlane
9. Implement in-app version checker ที่แจ้งผู้ใช้เมื่อมีอัปเดต

---

## สรุปหลักสูตร Flutter Parts 31-40

```
ส่วนที่ 31-40 ครอบคลุม:

Part 31: Clean Architecture
  - Domain/Data/Presentation layers
  - Use Cases, Repositories, Entities
  - Dependency Injection

Part 32: TDD & Testing
  - Unit, Widget, Integration Tests
  - Mockito/Mocktail
  - CI-ready test setup

Part 33: CI/CD Pipeline
  - GitHub Actions
  - Fastlane, Codemagic
  - Automatic deployment

Part 34: Performance Optimization
  - DevTools, Profiling
  - const, RepaintBoundary
  - Image, List, compute()

Part 35: Accessibility
  - Semantics, touch targets
  - Screen reader support
  - WCAG guidelines

Part 36: Internationalization
  - ARB files, flutter_localizations
  - Pluralization, RTL
  - intl package

Part 37: Flutter Web
  - Responsive design
  - go_router, SEO
  - PWA, Firebase Hosting

Part 38: Flutter Desktop
  - Window management
  - System tray, menus
  - Packaging (MSIX, DMG)

Part 39: Package Publishing
  - API design
  - Documentation
  - pub.dev publishing

Part 40: App Store Deployment
  - Google Play Store
  - Apple App Store
  - Release management
```

---

## การนำทาง
- [← Part 39: Package Publishing](part_39.md)
- [→ สิ้นสุดหลักสูตร Parts 31-40]

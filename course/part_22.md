# Part 22: Firebase Integration
## ขั้นตอนที่ 211-220

---

## สารบัญ
1. [Firebase Project Setup](#firebase-project-setup)
2. [FlutterFire CLI](#flutterfire-cli)
3. [firebase_core](#firebase_core)
4. [Firebase Console](#firebase-console)
5. [google-services.json Setup](#google-servicesjson-setup)
6. [FirebaseOptions](#firebaseoptions)
7. [Multiple Environments (Dev/Prod)](#multiple-environments-devprod)
8. [Firebase App Check](#firebase-app-check)
9. [Crashlytics](#crashlytics)
10. [Firebase Analytics](#firebase-analytics)

---

## ขั้นตอนที่ 211: Firebase Project Setup

Firebase คือ Backend-as-a-Service (BaaS) จาก Google ที่มีบริการครบครัน เช่น Database, Authentication, Storage, Hosting และอื่นๆ อีกมากมาย

### ขั้นตอนการสร้าง Firebase Project

```
1. ไปที่ https://console.firebase.google.com
2. คลิก "Add project" หรือ "สร้างโปรเจค"
3. ตั้งชื่อโปรเจค เช่น "my-flutter-app"
4. เปิด/ปิด Google Analytics ตามต้องการ
5. คลิก "Create project"
```

### โครงสร้าง Firebase Services

```
Firebase Project
├── Authentication
│   ├── Email/Password
│   ├── Google Sign-In
│   ├── Phone Auth
│   └── Anonymous
├── Firestore Database
│   ├── Collections
│   └── Documents
├── Realtime Database
├── Storage
│   └── Files & Images
├── Hosting
├── Cloud Functions
├── Cloud Messaging (FCM)
├── Analytics
├── Crashlytics
└── App Check
```

### การติดตั้ง Dependencies

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  firebase_auth: ^4.16.0
  cloud_firestore: ^4.14.0
  firebase_storage: ^11.6.0
  firebase_messaging: ^14.7.10
  firebase_analytics: ^10.7.4
  firebase_crashlytics: ^3.4.9
  firebase_app_check: ^0.2.1+8

dev_dependencies:
  flutter_test:
    sdk: flutter
```

---

## ขั้นตอนที่ 212: FlutterFire CLI

FlutterFire CLI เป็นเครื่องมือที่ช่วยตั้งค่า Firebase ในโปรเจค Flutter ได้อย่างง่ายดาย

### การติดตั้ง FlutterFire CLI

```bash
# ติดตั้ง Firebase CLI
npm install -g firebase-tools

# Login เข้า Firebase
firebase login

# ติดตั้ง FlutterFire CLI
dart pub global activate flutterfire_cli

# ตรวจสอบการติดตั้ง
flutterfire --version
```

### การ Configure โปรเจค

```bash
# รันใน root directory ของ Flutter project
flutterfire configure

# เลือก Firebase project
# เลือก platforms ที่ต้องการ (Android, iOS, Web, macOS)
# จะสร้างไฟล์ firebase_options.dart ให้อัตโนมัติ
```

### ไฟล์ที่ถูกสร้าง

```dart
// lib/firebase_options.dart (สร้างโดย FlutterFire CLI อัตโนมัติ)
import 'package:firebase_core/firebase_core.dart' show FirebaseOptions;
import 'package:flutter/foundation.dart'
    show defaultTargetPlatform, kIsWeb, TargetPlatform;

class DefaultFirebaseOptions {
  static FirebaseOptions get currentPlatform {
    if (kIsWeb) {
      return web;
    }
    switch (defaultTargetPlatform) {
      case TargetPlatform.android:
        return android;
      case TargetPlatform.iOS:
        return ios;
      case TargetPlatform.macOS:
        return macos;
      case TargetPlatform.windows:
        throw UnsupportedError(
          'DefaultFirebaseOptions have not been configured for windows - '
          'you can reconfigure this by running the FlutterFire CLI again.',
        );
      case TargetPlatform.linux:
        throw UnsupportedError(
          'DefaultFirebaseOptions have not been configured for linux - '
          'you can reconfigure this by running the FlutterFire CLI again.',
        );
      default:
        throw UnsupportedError(
          'DefaultFirebaseOptions are not supported for this platform.',
        );
    }
  }

  static const FirebaseOptions android = FirebaseOptions(
    apiKey: 'YOUR-ANDROID-API-KEY',
    appId: '1:123456789:android:abcdef123456',
    messagingSenderId: '123456789',
    projectId: 'my-flutter-app',
    storageBucket: 'my-flutter-app.appspot.com',
  );

  static const FirebaseOptions ios = FirebaseOptions(
    apiKey: 'YOUR-IOS-API-KEY',
    appId: '1:123456789:ios:abcdef123456',
    messagingSenderId: '123456789',
    projectId: 'my-flutter-app',
    storageBucket: 'my-flutter-app.appspot.com',
    iosBundleId: 'com.example.myapp',
  );

  static const FirebaseOptions web = FirebaseOptions(
    apiKey: 'YOUR-WEB-API-KEY',
    appId: '1:123456789:web:abcdef123456',
    messagingSenderId: '123456789',
    projectId: 'my-flutter-app',
    authDomain: 'my-flutter-app.firebaseapp.com',
    storageBucket: 'my-flutter-app.appspot.com',
    measurementId: 'G-ABCDEF123',
  );

  static const FirebaseOptions macos = FirebaseOptions(
    apiKey: 'YOUR-MACOS-API-KEY',
    appId: '1:123456789:ios:abcdef123456',
    messagingSenderId: '123456789',
    projectId: 'my-flutter-app',
    storageBucket: 'my-flutter-app.appspot.com',
    iosBundleId: 'com.example.myapp',
  );
}
```

---

## ขั้นตอนที่ 213: firebase_core

firebase_core เป็น Package หลักที่ต้อง Initialize ก่อนใช้งาน Firebase Service อื่นๆ

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

Future<void> main() async {
  // ต้องเรียก WidgetsFlutterBinding.ensureInitialized() ก่อน
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize Firebase
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
        useMaterial3: true,
      ),
      home: const FirebaseHomePage(),
    );
  }
}
```

### การจัดการ Error ระหว่าง Initialize

```dart
// lib/main.dart - พร้อม Error Handling
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(const AppWrapper());
}

class AppWrapper extends StatefulWidget {
  const AppWrapper({super.key});

  @override
  State<AppWrapper> createState() => _AppWrapperState();
}

class _AppWrapperState extends State<AppWrapper> {
  final Future<FirebaseApp> _initialization = Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<FirebaseApp>(
      future: _initialization,
      builder: (context, snapshot) {
        // กำลัง Initialize
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const MaterialApp(
            home: Scaffold(
              body: Center(child: CircularProgressIndicator()),
            ),
          );
        }

        // เกิดข้อผิดพลาด
        if (snapshot.hasError) {
          return MaterialApp(
            home: Scaffold(
              body: Center(
                child: Text('Firebase Error: ${snapshot.error}'),
              ),
            ),
          );
        }

        // Initialize สำเร็จ
        return const MyApp();
      },
    );
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const FirebaseHomePage(),
    );
  }
}

class FirebaseHomePage extends StatelessWidget {
  const FirebaseHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Firebase Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.local_fire_department, size: 80, color: Colors.orange),
            const SizedBox(height: 16),
            Text(
              'Firebase Connected!',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 8),
            Text(
              'App: ${Firebase.app().name}',
              style: Theme.of(context).textTheme.bodyLarge,
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 214: Firebase Console

Firebase Console เป็น Dashboard สำหรับจัดการ Firebase Project

```
Firebase Console Services:
├── Build
│   ├── Authentication - จัดการผู้ใช้
│   ├── App Check - ป้องกัน API abuse  
│   ├── Firestore Database - NoSQL Database
│   ├── Realtime Database - Real-time JSON Database
│   ├── Extensions - เพิ่มฟีเจอร์พิเศษ
│   ├── Hosting - Web Hosting
│   ├── Storage - File Storage
│   └── Functions - Serverless Functions
├── Release & Monitor
│   ├── Crashlytics - ติดตาม Crash
│   ├── Performance - วัดประสิทธิภาพ
│   ├── Test Lab - ทดสอบบน Real Devices
│   └── App Distribution - แจกจ่าย Beta
└── Analytics
    ├── Dashboard
    ├── Events
    ├── Conversions
    ├── Audiences
    └── Funnels
```

---

## ขั้นตอนที่ 215: google-services.json Setup

### สำหรับ Android

```bash
# ขั้นตอนการตั้งค่า Android
# 1. ไปที่ Firebase Console > Project Settings
# 2. คลิก "Add app" > Android
# 3. ใส่ Android package name (เช่น com.example.myapp)
# 4. ดาวน์โหลด google-services.json
# 5. วางไฟล์ที่ android/app/google-services.json
```

```gradle
// android/build.gradle
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

```gradle
// android/app/build.gradle
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // เพิ่มบรรทัดนี้
}

android {
    compileSdkVersion 34
    
    defaultConfig {
        applicationId "com.example.myapp"
        minSdkVersion 21  // Firebase ต้องการ minSdk 21+
        targetSdkVersion 34
        // ...
    }
}

dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.7.0')
    // ...
}
```

### สำหรับ iOS

```bash
# ขั้นตอนการตั้งค่า iOS
# 1. ไปที่ Firebase Console > Project Settings
# 2. คลิก "Add app" > iOS
# 3. ใส่ iOS Bundle ID (เช่น com.example.myapp)
# 4. ดาวน์โหลด GoogleService-Info.plist
# 5. เปิด Xcode แล้ว drag ไฟล์เข้า ios/Runner/
# 6. ตรวจสอบว่าไฟล์อยู่ใน "Runner" target
```

```xml
<!-- ios/Runner/Info.plist เพิ่ม/ตรวจสอบ -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <!-- REVERSED_CLIENT_ID จาก GoogleService-Info.plist -->
            <string>com.googleusercontent.apps.YOUR-CLIENT-ID</string>
        </array>
    </dict>
</array>
```

---

## ขั้นตอนที่ 216: FirebaseOptions

FirebaseOptions ใช้กำหนดค่า Firebase Config สำหรับแต่ละ Platform

```dart
// lib/firebase_config.dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/foundation.dart';

class FirebaseConfig {
  // Development config
  static const FirebaseOptions devAndroid = FirebaseOptions(
    apiKey: 'DEV-ANDROID-API-KEY',
    appId: '1:111111111:android:dev123',
    messagingSenderId: '111111111',
    projectId: 'my-app-dev',
    storageBucket: 'my-app-dev.appspot.com',
  );

  static const FirebaseOptions devIOS = FirebaseOptions(
    apiKey: 'DEV-IOS-API-KEY',
    appId: '1:111111111:ios:dev123',
    messagingSenderId: '111111111',
    projectId: 'my-app-dev',
    storageBucket: 'my-app-dev.appspot.com',
    iosBundleId: 'com.example.myapp.dev',
  );

  // Production config
  static const FirebaseOptions prodAndroid = FirebaseOptions(
    apiKey: 'PROD-ANDROID-API-KEY',
    appId: '1:222222222:android:prod123',
    messagingSenderId: '222222222',
    projectId: 'my-app-prod',
    storageBucket: 'my-app-prod.appspot.com',
  );

  static const FirebaseOptions prodIOS = FirebaseOptions(
    apiKey: 'PROD-IOS-API-KEY',
    appId: '1:222222222:ios:prod123',
    messagingSenderId: '222222222',
    projectId: 'my-app-prod',
    storageBucket: 'my-app-prod.appspot.com',
    iosBundleId: 'com.example.myapp',
  );
}
```

---

## ขั้นตอนที่ 217: Multiple Environments (Dev/Prod)

```dart
// lib/config/app_config.dart
enum Environment { development, production }

class AppConfig {
  static const Environment _currentEnv = Environment.development;
  // สำหรับ production: const Environment _currentEnv = Environment.production;

  static bool get isDevelopment => _currentEnv == Environment.development;
  static bool get isProduction => _currentEnv == Environment.production;

  static String get firebaseProjectId {
    return isDevelopment ? 'my-app-dev' : 'my-app-prod';
  }

  static String get apiBaseUrl {
    return isDevelopment
        ? 'https://api-dev.myapp.com'
        : 'https://api.myapp.com';
  }
}
```

```dart
// lib/main_development.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options_dev.dart';
import 'app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptionsDev.currentPlatform,
  );

  runApp(const App(environment: 'development'));
}
```

```dart
// lib/main_production.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options_prod.dart';
import 'app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptionsProd.currentPlatform,
  );

  runApp(const App(environment: 'production'));
}
```

```dart
// lib/app.dart
import 'package:flutter/material.dart';

class App extends StatelessWidget {
  const App({super.key, required this.environment});
  final String environment;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App ($environment)',
      debugShowCheckedModeBanner: environment == 'development',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: environment == 'development' ? Colors.orange : Colors.blue,
        ),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text('Environment: $environment'),
          backgroundColor: environment == 'development'
              ? Colors.orange
              : Colors.blue,
        ),
        body: Center(
          child: Text('Running in $environment mode'),
        ),
      ),
    );
  }
}
```

### การรันด้วย Flavor

```bash
# Development
flutter run --target lib/main_development.dart --flavor development

# Production
flutter run --target lib/main_production.dart --flavor production

# Build
flutter build apk --target lib/main_production.dart --flavor production
```

---

## ขั้นตอนที่ 218: Firebase App Check

Firebase App Check ช่วยป้องกัน API ของคุณจากการ abuse

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_app_check/firebase_app_check.dart';
import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  // Activate App Check
  await FirebaseAppCheck.instance.activate(
    // สำหรับ Android
    androidProvider: AndroidProvider.debug, // ใช้ debug ใน development
    // androidProvider: AndroidProvider.playIntegrity, // ใช้ production

    // สำหรับ iOS
    appleProvider: AppleProvider.debug, // ใช้ debug ใน development
    // appleProvider: AppleProvider.appAttest, // ใช้ production

    // สำหรับ Web
    webProvider: ReCaptchaV3Provider('YOUR-RECAPTCHA-SITE-KEY'),
  );

  runApp(const MyApp());
}
```

```dart
// lib/services/app_check_service.dart
import 'package:firebase_app_check/firebase_app_check.dart';

class AppCheckService {
  static Future<String?> getToken() async {
    try {
      final token = await FirebaseAppCheck.instance.getToken(true);
      return token;
    } catch (e) {
      print('App Check error: $e');
      return null;
    }
  }

  static void listenToTokenChanges() {
    FirebaseAppCheck.instance.onTokenChange.listen((token) {
      if (token != null) {
        print('New App Check token received');
        // อัพเดต token ใน header หรือ store
      }
    });
  }
}
```

---

## ขั้นตอนที่ 219: Crashlytics

```dart
// lib/main.dart กับ Crashlytics
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  // ตั้งค่า Crashlytics
  FlutterError.onError = (errorDetails) {
    FirebaseCrashlytics.instance.recordFlutterFatalError(errorDetails);
  };

  // Catch async errors
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  // Enable/Disable collection based on environment
  await FirebaseCrashlytics.instance
      .setCrashlyticsCollectionEnabled(!const bool.fromEnvironment('DEBUG'));

  runApp(const MyApp());
}

// lib/services/crashlytics_service.dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

class CrashlyticsService {
  static final _crashlytics = FirebaseCrashlytics.instance;

  // บันทึก Error
  static Future<void> recordError(
    dynamic exception,
    StackTrace? stack, {
    String? reason,
    bool fatal = false,
  }) async {
    await _crashlytics.recordError(
      exception,
      stack,
      reason: reason,
      fatal: fatal,
    );
  }

  // บันทึก Message
  static Future<void> log(String message) async {
    await _crashlytics.log(message);
  }

  // ตั้งค่า User ID
  static Future<void> setUserId(String userId) async {
    await _crashlytics.setUserIdentifier(userId);
  }

  // ตั้งค่า Custom Keys
  static Future<void> setCustomKey(String key, String value) async {
    await _crashlytics.setCustomKey(key, value);
  }

  // ทดสอบ Crash (ใช้สำหรับ testing เท่านั้น)
  static void testCrash() {
    _crashlytics.crash();
  }
}
```

---

## ขั้นตอนที่ 220: Firebase Analytics

```dart
// lib/services/analytics_service.dart
import 'package:firebase_analytics/firebase_analytics.dart';

class AnalyticsService {
  static final FirebaseAnalytics _analytics = FirebaseAnalytics.instance;
  static final FirebaseAnalyticsObserver observer =
      FirebaseAnalyticsObserver(analytics: _analytics);

  // Log Events
  static Future<void> logEvent(
    String name, {
    Map<String, dynamic>? parameters,
  }) async {
    await _analytics.logEvent(name: name, parameters: parameters);
  }

  // Standard Events
  static Future<void> logLogin({String? loginMethod}) async {
    await _analytics.logLogin(loginMethod: loginMethod);
  }

  static Future<void> logSignUp({required String signUpMethod}) async {
    await _analytics.logSignUp(signUpMethod: signUpMethod);
  }

  static Future<void> logPurchase({
    required String currency,
    required double value,
    required List<AnalyticsEventItem> items,
  }) async {
    await _analytics.logPurchase(
      currency: currency,
      value: value,
      items: items,
    );
  }

  static Future<void> logAddToCart({
    required AnalyticsEventItem item,
    required double value,
    required String currency,
  }) async {
    await _analytics.logAddToCart(
      items: [item],
      value: value,
      currency: currency,
    );
  }

  static Future<void> logSearch({required String searchTerm}) async {
    await _analytics.logSearch(searchTerm: searchTerm);
  }

  static Future<void> logScreenView({
    required String screenName,
    String? screenClass,
  }) async {
    await _analytics.logScreenView(
      screenName: screenName,
      screenClass: screenClass ?? screenName,
    );
  }

  // User Properties
  static Future<void> setUserId(String? id) async {
    await _analytics.setUserId(id: id);
  }

  static Future<void> setUserProperty({
    required String name,
    required String? value,
  }) async {
    await _analytics.setUserProperty(name: name, value: value);
  }
}

// lib/main.dart - เพิ่ม Analytics Observer
import 'package:flutter/material.dart';
import 'package:firebase_analytics/firebase_analytics.dart';

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase Analytics Demo',
      navigatorObservers: [
        AnalyticsService.observer, // ติดตาม Screen Views อัตโนมัติ
      ],
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

// ตัวอย่างการใช้งาน Analytics ใน Widget
class ProductDetailPage extends StatefulWidget {
  const ProductDetailPage({super.key, required this.productId});
  final String productId;

  @override
  State<ProductDetailPage> createState() => _ProductDetailPageState();
}

class _ProductDetailPageState extends State<ProductDetailPage> {
  @override
  void initState() {
    super.initState();
    // Log screen view
    AnalyticsService.logScreenView(
      screenName: 'product_detail',
      screenClass: 'ProductDetailPage',
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('สินค้า ${widget.productId}')),
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            await AnalyticsService.logAddToCart(
              item: AnalyticsEventItem(
                itemId: widget.productId,
                itemName: 'Sample Product',
                itemCategory: 'Electronics',
                price: 999.0,
                quantity: 1,
              ),
              value: 999.0,
              currency: 'THB',
            );
            if (mounted) {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('เพิ่มในตะกร้าแล้ว')),
              );
            }
          },
          child: const Text('เพิ่มในตะกร้า'),
        ),
      ),
    );
  }
}
```

---

## Workshop: Firebase Setup สมบูรณ์

```dart
// lib/main.dart - Complete Firebase Setup
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:firebase_analytics/firebase_analytics.dart';
import 'package:firebase_app_check/firebase_app_check.dart';
import 'firebase_options.dart';

Future<void> main() async {
  runZonedGuarded<Future<void>>(
    () async {
      WidgetsFlutterBinding.ensureInitialized();

      // Initialize Firebase
      await Firebase.initializeApp(
        options: DefaultFirebaseOptions.currentPlatform,
      );

      // Setup Crashlytics
      FlutterError.onError =
          FirebaseCrashlytics.instance.recordFlutterFatalError;

      // Setup App Check
      await FirebaseAppCheck.instance.activate(
        androidProvider: AndroidProvider.debug,
        appleProvider: AppleProvider.debug,
      );

      runApp(const MyApp());
    },
    (error, stack) => FirebaseCrashlytics.instance.recordError(
      error,
      stack,
      fatal: true,
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  static FirebaseAnalytics analytics = FirebaseAnalytics.instance;
  static FirebaseAnalyticsObserver observer =
      FirebaseAnalyticsObserver(analytics: analytics);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase Complete Setup',
      navigatorObservers: [observer],
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
        useMaterial3: true,
      ),
      home: const FirebaseStatusPage(),
    );
  }
}

class FirebaseStatusPage extends StatelessWidget {
  const FirebaseStatusPage({super.key});

  @override
  Widget build(BuildContext context) {
    final app = Firebase.app();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Firebase Status'),
        backgroundColor: Colors.deepOrange,
        foregroundColor: Colors.white,
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _StatusCard(
            icon: Icons.local_fire_department,
            title: 'Firebase Core',
            subtitle: 'Project: ${app.options.projectId}',
            status: true,
          ),
          _StatusCard(
            icon: Icons.security,
            title: 'App Check',
            subtitle: 'Debug mode enabled',
            status: true,
          ),
          _StatusCard(
            icon: Icons.bug_report,
            title: 'Crashlytics',
            subtitle: 'Crash reporting enabled',
            status: true,
          ),
          _StatusCard(
            icon: Icons.analytics,
            title: 'Analytics',
            subtitle: 'Auto tracking enabled',
            status: true,
          ),
          const SizedBox(height: 20),
          ElevatedButton.icon(
            onPressed: () {
              // Test Analytics
              FirebaseAnalytics.instance.logEvent(
                name: 'test_event',
                parameters: {'button': 'test_button'},
              );
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(
                  content: Text('Analytics event logged!'),
                  backgroundColor: Colors.green,
                ),
              );
            },
            icon: const Icon(Icons.send),
            label: const Text('ส่ง Test Event'),
          ),
        ],
      ),
    );
  }
}

class _StatusCard extends StatelessWidget {
  const _StatusCard({
    required this.icon,
    required this.title,
    required this.subtitle,
    required this.status,
  });

  final IconData icon;
  final String title;
  final String subtitle;
  final bool status;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: ListTile(
        leading: Icon(icon, color: Colors.deepOrange),
        title: Text(title, style: const TextStyle(fontWeight: FontWeight.bold)),
        subtitle: Text(subtitle),
        trailing: Icon(
          status ? Icons.check_circle : Icons.error,
          color: status ? Colors.green : Colors.red,
        ),
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Firebase Project Setup**: การสร้างและตั้งค่า Firebase Project
- **FlutterFire CLI**: เครื่องมือตั้งค่า Firebase อัตโนมัติ
- **firebase_core**: การ Initialize Firebase ใน Flutter
- **Firebase Console**: ภาพรวมของบริการต่างๆ
- **google-services.json**: การตั้งค่าสำหรับ Android และ iOS
- **FirebaseOptions**: การกำหนดค่า Firebase per platform
- **Multiple Environments**: การแยก Dev/Prod environments
- **App Check**: การป้องกัน API abuse
- **Crashlytics**: การติดตาม Crash
- **Analytics**: การวิเคราะห์พฤติกรรมผู้ใช้

## แบบฝึกหัด

1. สร้าง Firebase Project ใหม่และตั้งค่าด้วย FlutterFire CLI
2. ตั้งค่า Multiple Environments (dev/staging/prod)
3. Implement Error Boundary ที่รายงาน Error ไปยัง Crashlytics
4. สร้าง Analytics Service ที่ Track ทุก Screen Navigation
5. ทดสอบ App Check ใน Debug Mode

---

[⬅️ Part 21](part_21.md) | [Part 23 ➡️](part_23.md)

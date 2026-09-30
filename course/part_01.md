# Part 01: การติดตั้งและตั้งค่า Flutter
## ขั้นตอนที่ 1-10

---

## สารบัญ
1. [Flutter คืออะไร?](#flutter-คืออะไร)
2. [การติดตั้ง Flutter SDK](#การติดตั้ง-flutter-sdk)
3. [การตั้งค่า Android Studio](#การตั้งค่า-android-studio)
4. [การตั้งค่า VS Code](#การตั้งค่า-vs-code)
5. [การสร้างโปรเจค Flutter แรก](#การสร้างโปรเจค-flutter-แรก)
6. [โครงสร้างโปรเจค Flutter](#โครงสร้างโปรเจค-flutter)
7. [การรันแอปพลิเคชัน](#การรันแอปพลิเคชัน)
8. [Flutter Doctor](#flutter-doctor)
9. [การตั้งค่า Emulator](#การตั้งค่า-emulator)
10. [Hot Reload & Hot Restart](#hot-reload--hot-restart)

---

## ขั้นตอนที่ 1: Flutter คืออะไร?

Flutter คือ UI Toolkit แบบ Open Source จาก Google ที่ช่วยให้คุณสามารถสร้างแอปพลิเคชันได้ทั้ง Mobile (iOS/Android), Web, Desktop จาก Codebase เดียวกัน

### คุณสมบัติหลักของ Flutter

```
Flutter
├── Cross-platform (iOS, Android, Web, Desktop, Embedded)
├── ภาษา Dart
├── Reactive UI Framework
├── Hot Reload (เปลี่ยนโค้ดเห็นผลทันที)
├── Widget-based Architecture
└── Native Performance (Compiled to native code)
```

### ทำไมต้องเลือก Flutter?

| คุณสมบัติ | Flutter | React Native | Xamarin |
|-----------|---------|--------------|---------|
| ประสิทธิภาพ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| UI Consistency | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Community | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Learning Curve | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Hot Reload | ✅ | ✅ | ⚠️ |

### สถาปัตยกรรมของ Flutter

```
┌─────────────────────────────────────────┐
│           Your Flutter App              │
├─────────────────────────────────────────┤
│         Flutter Framework               │
│  Material | Cupertino | Widgets | etc.  │
├─────────────────────────────────────────┤
│           Flutter Engine                │
│     Skia/Impeller | Dart Runtime        │
├─────────────────────────────────────────┤
│         Platform (iOS/Android/Web)      │
└─────────────────────────────────────────┘
```

---

## ขั้นตอนที่ 2: การติดตั้ง Flutter SDK

### สำหรับ macOS

```bash
# วิธีที่ 1: ติดตั้งผ่าน Homebrew (แนะนำ)
brew install flutter

# ตรวจสอบเวอร์ชัน
flutter --version

# วิธีที่ 2: ดาวน์โหลดจาก Official Website
# 1. ไปที่ https://flutter.dev/docs/get-started/install/macos
# 2. ดาวน์โหลด Flutter SDK
# 3. แตกไฟล์ไปที่ ~/development/flutter

# เพิ่ม PATH ใน ~/.zshrc หรือ ~/.bashrc
export PATH="$PATH:$HOME/development/flutter/bin"

# Reload shell
source ~/.zshrc
```

### สำหรับ Windows

```powershell
# วิธีที่ 1: ใช้ winget
winget install Flutter.Flutter

# วิธีที่ 2: ดาวน์โหลดจาก Official Website
# 1. ไปที่ https://flutter.dev/docs/get-started/install/windows
# 2. ดาวน์โหลด Flutter SDK (flutter_windows_x.x.x-stable.zip)
# 3. แตกไฟล์ไปที่ C:\src\flutter

# เพิ่ม PATH ใน Environment Variables
# C:\src\flutter\bin

# ตรวจสอบการติดตั้ง
flutter --version
```

### สำหรับ Linux

```bash
# วิธีที่ 1: ใช้ snap
sudo snap install flutter --classic

# วิธีที่ 2: Manual Installation
# 1. ดาวน์โหลด SDK
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_x.x.x-stable.tar.xz

# 2. แตกไฟล์
tar xf flutter_linux_x.x.x-stable.tar.xz -C ~/development/

# 3. เพิ่ม PATH
echo 'export PATH="$PATH:$HOME/development/flutter/bin"' >> ~/.bashrc
source ~/.bashrc

# ตรวจสอบ
flutter --version
```

### ผลลัพธ์ที่ควรได้รับ

```
Flutter 3.x.x • channel stable • https://github.com/flutter/flutter.git
Framework • revision xxxxxxxx (x days ago) • 2024-xx-xx xx:xx:xx
Engine • revision xxxxxxxx
Tools • Dart 3.x.x • DevTools 2.x.x
```

---

## ขั้นตอนที่ 3: การตั้งค่า Android Studio

### การติดตั้ง Android Studio

```
1. ดาวน์โหลดจาก https://developer.android.com/studio
2. ติดตั้งตามขั้นตอน
3. เปิด Android Studio
4. ไปที่ SDK Manager:
   File > Settings > Appearance & Behavior > System Settings > Android SDK
5. ติดตั้ง:
   - Android SDK Platform (API Level 33+)
   - Android SDK Build-Tools
   - Android Emulator
   - Android SDK Platform-Tools
```

### การติดตั้ง Flutter Plugin

```
1. เปิด Android Studio
2. ไปที่ File > Settings > Plugins
3. ค้นหา "Flutter"
4. ติดตั้ง Flutter Plugin (จะติดตั้ง Dart Plugin ด้วยอัตโนมัติ)
5. Restart Android Studio
```

### การยอมรับ Android Licenses

```bash
# รันคำสั่งนี้และกด 'y' เพื่อยอมรับทุก license
flutter doctor --android-licenses
```

---

## ขั้นตอนที่ 4: การตั้งค่า VS Code

VS Code เป็น Editor ที่นักพัฒนา Flutter หลายคนชื่นชอบเพราะเบาและรวดเร็ว

### การติดตั้ง Extensions

```
1. เปิด VS Code
2. กด Ctrl+Shift+X (หรือ Cmd+Shift+X บน Mac) เปิด Extensions
3. ค้นหาและติดตั้ง:
   - Flutter (by Dart Code)
   - Dart (by Dart Code) - ติดตั้งพร้อมกับ Flutter
   - Awesome Flutter Snippets
   - Flutter Widget Snippets
   - Error Lens
   - GitLens
   - Bracket Pair Colorizer 2
```

### การตั้งค่า settings.json

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "[dart]": {
    "editor.formatOnSave": true,
    "editor.selectionHighlight": false,
    "editor.suggest.snippetsPreventQuickSuggestions": false,
    "editor.suggestSelection": "first",
    "editor.tabCompletion": "onlySnippets",
    "editor.wordBasedSuggestions": "off"
  },
  "dart.flutterSdkPath": "/path/to/flutter",
  "dart.debugExternalPackageLibraries": false,
  "dart.debugSdkLibraries": false,
  "flutter.hotReloadOnSave": "always"
}
```

---

## ขั้นตอนที่ 5: การสร้างโปรเจค Flutter แรก

### การสร้างโปรเจคใหม่

```bash
# สร้างโปรเจคใหม่
flutter create my_first_app

# สร้างพร้อมกำหนด organization name
flutter create --org com.yourcompany my_first_app

# สร้างโปรเจคพร้อมกำหนด platform
flutter create --platforms=android,ios,web my_first_app

# สร้างโปรเจคด้วย template ต่างๆ
flutter create --template=app my_first_app    # Default app
flutter create --template=plugin my_plugin    # Plugin
flutter create --template=package my_package  # Package

# เข้าไปในโฟลเดอร์
cd my_first_app
```

### โครงสร้างโปรเจคที่สร้างใหม่

```
my_first_app/
├── android/                    # Android-specific code
│   ├── app/
│   │   ├── src/
│   │   └── build.gradle
│   └── build.gradle
├── ios/                        # iOS-specific code
│   ├── Runner/
│   └── Runner.xcworkspace
├── lib/                        # Flutter/Dart code หลัก
│   └── main.dart               # Entry point
├── test/                       # Unit tests
│   └── widget_test.dart
├── web/                        # Web-specific code
├── linux/                      # Linux desktop
├── macos/                      # macOS desktop
├── windows/                    # Windows desktop
├── pubspec.yaml                # Project configuration
├── pubspec.lock                # Lock file
└── README.md
```

---

## ขั้นตอนที่ 6: โครงสร้างโปรเจค Flutter

### ไฟล์ pubspec.yaml

```yaml
name: my_first_app
description: A new Flutter project.
publish_to: 'none'   # ป้องกันการ publish ขึ้น pub.dev โดยไม่ตั้งใจ

# เวอร์ชันแอป (major.minor.patch+build)
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'  # Dart SDK version

dependencies:
  flutter:
    sdk: flutter
  
  # ตัวอย่างการเพิ่ม package
  cupertino_icons: ^1.0.6    # iOS-style icons
  http: ^1.1.0               # HTTP requests

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0      # Lint rules

flutter:
  uses-material-design: true
  
  # เพิ่ม assets (รูปภาพ, fonts)
  assets:
    - assets/images/
    - assets/icons/
  
  fonts:
    - family: CustomFont
      fonts:
        - asset: assets/fonts/CustomFont-Regular.ttf
        - asset: assets/fonts/CustomFont-Bold.ttf
          weight: 700
```

### ไฟล์ main.dart เริ่มต้น

```dart
import 'package:flutter/material.dart';

// Entry point ของแอป
void main() {
  runApp(const MyApp());
}

// Root widget ของแอป
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

// Home Page Widget
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 7: การรันแอปพลิเคชัน

### คำสั่งพื้นฐาน

```bash
# รันแอปบน device/emulator ที่เชื่อมต่ออยู่
flutter run

# รันบน device เฉพาะ
flutter run -d <device_id>

# ดูรายการ devices ที่ใช้ได้
flutter devices

# รันบน Chrome (Web)
flutter run -d chrome

# รันบน macOS
flutter run -d macos

# รันแบบ Release mode
flutter run --release

# รันแบบ Profile mode (สำหรับ performance testing)
flutter run --profile

# Build APK
flutter build apk

# Build IPA (ต้องใช้ Mac)
flutter build ios

# Build Web
flutter build web

# Build สำหรับ platform ต่างๆ
flutter build apk --release
flutter build appbundle --release  # สำหรับ Google Play Store
```

### ตัวอย่าง Output เมื่อรัน

```
Launching lib/main.dart on iPhone 15 in debug mode...
Running Xcode build...
Xcode build done.                                           15.2s
Flutter run key commands.
r Hot reload. 🔥🔥🔥
R Hot restart.
h List all available interactive commands.
d Detach (terminate "flutter run" but leave app running).
c Clear the screen
q Quit (terminate the app on the device).

💪 Running with sound null safety 💪

An Observatory debugger and profiler on iPhone 15 is available at: http://127.0.0.1:8080/xxxxx/
The Flutter DevTools debugger and profiler on iPhone 15 is available at: http://127.0.0.1:9100?uri=http://127.0.0.1:8080/xxxxx/
```

---

## ขั้นตอนที่ 8: Flutter Doctor

Flutter Doctor เป็นคำสั่งตรวจสอบความพร้อมของระบบสำหรับการพัฒนา Flutter

```bash
# รัน Flutter Doctor
flutter doctor

# รันพร้อมแสดง detail เพิ่มเติม
flutter doctor -v
```

### ตัวอย่างผลลัพธ์ Flutter Doctor

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x, on macOS 14.x.x)
[✓] Android toolchain - develop for Android devices (Android SDK version 34.x.x)
[✓] Xcode - develop for iOS and macOS (Xcode 15.x)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2023.x)
[✓] VS Code (version 1.85.x)
[✓] Connected device (3 available)
[✓] Network resources

• No issues found!
```

### การแก้ปัญหาที่พบบ่อย

```bash
# ปัญหา: Android licenses ยังไม่ได้ยอมรับ
flutter doctor --android-licenses

# ปัญหา: Xcode Command Line Tools ไม่ได้ติดตั้ง
xcode-select --install

# ปัญหา: Flutter path ไม่ถูกต้อง
which flutter
# ตรวจสอบ PATH ใน ~/.bashrc หรือ ~/.zshrc

# ปัญหา: CocoaPods ไม่ได้ติดตั้ง (macOS)
sudo gem install cocoapods
# หรือ
brew install cocoapods

# อัปเดต Flutter
flutter upgrade

# เปลี่ยน channel
flutter channel stable
flutter upgrade
```

---

## ขั้นตอนที่ 9: การตั้งค่า Emulator

### Android Emulator

```bash
# สร้าง Android Virtual Device (AVD) ผ่าน Android Studio
# 1. เปิด Android Studio
# 2. Tools > Device Manager
# 3. Create Device
# 4. เลือก Hardware Profile (Pixel 7 Pro)
# 5. เลือก System Image (Android 14)
# 6. Finish

# หรือใช้ Command Line
# ดูรายการ AVD ที่มี
flutter emulators

# เปิด Emulator
flutter emulators --launch Pixel_7_Pro_API_34

# สร้าง Emulator ใหม่
flutter emulators --create --name MyEmulator
```

### iOS Simulator (macOS เท่านั้น)

```bash
# เปิด iOS Simulator
open -a Simulator

# หรือผ่าน Xcode
# Xcode > Open Developer Tool > Simulator

# เปิด Simulator เฉพาะ device
xcrun simctl boot "iPhone 15 Pro"

# ดู Simulators ที่มี
xcrun simctl list devices

# รัน Flutter บน Simulator
flutter run -d "iPhone 15 Pro"
```

### การตั้งค่า Physical Device

```bash
# Android
# 1. เปิด Developer Options บน Phone
#    Settings > About Phone > กด Build Number 7 ครั้ง
# 2. เปิด USB Debugging
#    Settings > Developer Options > USB Debugging
# 3. เสียบ USB และยืนยัน Trust

# ตรวจสอบ device ที่เชื่อมต่อ
flutter devices

# iOS (ต้องการ Apple Developer Account)
# 1. เปิด Xcode > Preferences > Accounts
# 2. เพิ่ม Apple ID
# 3. เชื่อมต่อ iPhone
# 4. Trust computer บน iPhone
```

---

## ขั้นตอนที่ 10: Hot Reload & Hot Restart

Hot Reload และ Hot Restart เป็นฟีเจอร์ที่ทำให้การพัฒนา Flutter รวดเร็วมาก

### ความแตกต่างระหว่าง Hot Reload และ Hot Restart

| | Hot Reload 🔥 | Hot Restart 🔄 |
|--|--------------|---------------|
| ความเร็ว | < 1 วินาที | 2-5 วินาที |
| State | ยังคงอยู่ | Reset |
| ใช้เมื่อ | เปลี่ยน UI | เปลี่ยน State/Logic |
| Shortcut | r | R |

### วิธีใช้

```bash
# ระหว่างรัน flutter run
# Hot Reload: กด r
# Hot Restart: กด R
# Quit: กด q
# Help: กด h

# ใน VS Code
# Hot Reload: Ctrl+F5 หรือกดปุ่ม Hot Reload ใน toolbar
# Hot Restart: Shift+F5

# ใน Android Studio
# Hot Reload: กดปุ่ม ⚡ (Lightning)
# Hot Restart: กดปุ่ม 🔄
```

### ตัวอย่างที่เห็นผลของ Hot Reload

```dart
// โค้ดเดิม
Text(
  'Hello World',
  style: TextStyle(fontSize: 20),
)

// เปลี่ยนเป็น
Text(
  'สวัสดีชาวโลก!',
  style: TextStyle(
    fontSize: 30,
    color: Colors.blue,
    fontWeight: FontWeight.bold,
  ),
)
// บันทึกไฟล์ -> Hot Reload ทำงานทันที!
```

### ข้อจำกัดของ Hot Reload

```dart
// Hot Reload ไม่ทำงานใน:
// 1. การเปลี่ยนแปลง main() function
void main() {
  // การเพิ่มโค้ดใหม่ที่นี่ต้องใช้ Hot Restart
  runApp(const MyApp());
}

// 2. การเพิ่ม/ลบ State variables
class _MyState extends State<MyWidget> {
  int counter = 0;
  // การเพิ่มตัวแปรใหม่ต้องใช้ Hot Restart
  // String newVariable = ''; // ต้อง Hot Restart
}

// 3. การเปลี่ยน initState()
@override
void initState() {
  super.initState();
  // การเปลี่ยนโค้ดใน initState ต้องใช้ Hot Restart
}
```

---

## Workshop: สร้างแอปแรก

ลองสร้างแอปง่ายๆ เพื่อทดสอบการตั้งค่า:

```dart
// ไฟล์: lib/main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyFirstApp());
}

class MyFirstApp extends StatelessWidget {
  const MyFirstApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My First Flutter App',
      debugShowCheckedModeBanner: false,  // ซ่อน debug banner
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.teal,
          brightness: Brightness.light,
        ),
        useMaterial3: true,
      ),
      home: const WelcomeScreen(),
    );
  }
}

class WelcomeScreen extends StatefulWidget {
  const WelcomeScreen({super.key});

  @override
  State<WelcomeScreen> createState() => _WelcomeScreenState();
}

class _WelcomeScreenState extends State<WelcomeScreen> {
  String _message = 'กดปุ่มด้านล่าง!';
  int _tapCount = 0;

  void _onButtonPressed() {
    setState(() {
      _tapCount++;
      if (_tapCount == 1) {
        _message = 'สวัสดี Flutter! 👋';
      } else if (_tapCount <= 5) {
        _message = 'คุณกดไปแล้ว $_tapCount ครั้ง! 🎉';
      } else {
        _message = 'หยุดกดได้แล้ว! 😄 (ครั้งที่ $_tapCount)';
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.teal.shade50,
      appBar: AppBar(
        title: const Text(
          'Flutter แรกของฉัน',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        backgroundColor: Colors.teal,
        foregroundColor: Colors.white,
        centerTitle: true,
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Logo/Icon
              Container(
                width: 120,
                height: 120,
                decoration: BoxDecoration(
                  color: Colors.teal.shade100,
                  shape: BoxShape.circle,
                  boxShadow: [
                    BoxShadow(
                      color: Colors.teal.withOpacity(0.3),
                      blurRadius: 20,
                      spreadRadius: 5,
                    ),
                  ],
                ),
                child: const Icon(
                  Icons.flutter_dash,
                  size: 72,
                  color: Colors.teal,
                ),
              ),
              
              const SizedBox(height: 32),
              
              // Title
              Text(
                'ยินดีต้อนรับสู่ Flutter!',
                style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                  fontWeight: FontWeight.bold,
                  color: Colors.teal.shade800,
                ),
                textAlign: TextAlign.center,
              ),
              
              const SizedBox(height: 16),
              
              // Message
              AnimatedSwitcher(
                duration: const Duration(milliseconds: 300),
                child: Text(
                  _message,
                  key: ValueKey(_message),
                  style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                    color: Colors.teal.shade600,
                  ),
                  textAlign: TextAlign.center,
                ),
              ),
              
              const SizedBox(height: 48),
              
              // Button
              ElevatedButton.icon(
                onPressed: _onButtonPressed,
                icon: const Icon(Icons.touch_app),
                label: const Text(
                  'กดที่นี่!',
                  style: TextStyle(fontSize: 18),
                ),
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.teal,
                  foregroundColor: Colors.white,
                  padding: const EdgeInsets.symmetric(
                    horizontal: 32,
                    vertical: 16,
                  ),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),
              
              const SizedBox(height: 16),
              
              // Counter display
              if (_tapCount > 0)
                Container(
                  padding: const EdgeInsets.symmetric(
                    horizontal: 16,
                    vertical: 8,
                  ),
                  decoration: BoxDecoration(
                    color: Colors.teal.shade100,
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: Text(
                    'จำนวนครั้ง: $_tapCount',
                    style: TextStyle(
                      color: Colors.teal.shade700,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### วิธีรัน Workshop

```bash
# 1. สร้างโปรเจคใหม่
flutter create my_first_app
cd my_first_app

# 2. แทนที่ไฟล์ lib/main.dart ด้วยโค้ดด้านบน

# 3. รันแอป
flutter run

# 4. ทดสอบ Hot Reload
# แก้ไขข้อความหรือสีใดๆ แล้วบันทึก
# สังเกตว่าแอปอัปเดตทันที!
```

---

## สรุป Part 01

ในส่วนนี้คุณได้เรียนรู้:

✅ Flutter คืออะไรและทำไมต้องใช้
✅ การติดตั้ง Flutter SDK บน macOS, Windows, Linux
✅ การตั้งค่า Android Studio และ VS Code
✅ การสร้างโปรเจค Flutter แรก
✅ โครงสร้างไฟล์และโฟลเดอร์ของโปรเจค Flutter
✅ การรันแอปบน Emulator และ Physical Device
✅ การใช้ Flutter Doctor เพื่อตรวจสอบปัญหา
✅ การตั้งค่า Emulator ทั้ง Android และ iOS
✅ Hot Reload และ Hot Restart
✅ สร้างแอปแรกได้สำเร็จ!

## แบบฝึกหัด

1. ติดตั้ง Flutter SDK และตรวจสอบด้วย `flutter doctor`
2. สร้างโปรเจค Flutter ใหม่ชื่อ `hello_flutter`
3. แก้ไข main.dart ให้แสดงชื่อของคุณแทน "Flutter Demo"
4. เพิ่มสีและ style ที่คุณชื่นชอบ
5. ทดสอบ Hot Reload โดยเปลี่ยนข้อความและสังเกตผล

---

**ต่อไป:** [Part 02 - Dart Language Fundamentals →](part_02.md)

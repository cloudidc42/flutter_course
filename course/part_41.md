# Part 41: Micro Frontend Architecture
## ขั้นตอนที่ 401-410

---

## สารบัญ
1. [Module Federation Concept สำหรับ Flutter](#ขั้นตอนที่-401-module-federation-concept)
2. [Feature Modules](#ขั้นตอนที่-402-feature-modules)
3. [Lazy Loading Features](#ขั้นตอนที่-403-lazy-loading-features)
4. [Plugin-based Architecture](#ขั้นตอนที่-404-plugin-based-architecture)
5. [Shared Kernel Pattern](#ขั้นตอนที่-405-shared-kernel-pattern)
6. [Monorepo with Melos](#ขั้นตอนที่-406-monorepo-with-melos)
7. [Inter-module Communication](#ขั้นตอนที่-407-inter-module-communication)
8. [Independent Deployment of Features](#ขั้นตอนที่-408-independent-deployment)
9. [Testing Micro Frontend Modules](#ขั้นตอนที่-409-testing-micro-frontend)
10. [Workshop: สร้าง Micro Frontend App](#ขั้นตอนที่-410-workshop)

---

## ขั้นตอนที่ 401: Module Federation Concept สำหรับ Flutter

### แนวคิด Micro Frontend คืออะไร?

Micro Frontend เป็นแนวทางการออกแบบสถาปัตยกรรมที่แบ่งแอปพลิเคชัน Frontend ขนาดใหญ่ออกเป็น Feature ย่อยๆ ที่ทำงานได้อิสระ แต่ละ Feature สามารถพัฒนา ทดสอบ และ Deploy ได้อย่างแยกกัน

```
┌─────────────────────────────────────────────────────────┐
│                    Shell Application                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Auth Module  │  │ Shop Module  │  │ Chat Module  │  │
│  │   (Team A)   │  │   (Team B)   │  │   (Team C)   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │              Shared Kernel / Core                  │ │
│  │  (Design System, Auth Utils, Navigation, Logger)   │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### ประโยชน์ของ Micro Frontend

1. **Team Autonomy** - แต่ละทีมทำงานบน Feature ของตัวเองโดยอิสระ
2. **Independent Deployment** - Deploy แต่ละ Feature แยกกันได้
3. **Technology Flexibility** - แต่ละ Module ใช้ library ต่างกันได้
4. **Scalability** - ขยายทีมโดยไม่ต้องขยาย codebase เดียว
5. **Resilience** - ถ้า Module หนึ่ง fail ไม่กระทบ Module อื่น

### Module Federation ใน Flutter

Flutter ไม่มี Module Federation แบบ Webpack มาให้ แต่เราสามารถจำลองแนวคิดนี้ได้ผ่าน:

1. **Flutter Packages** - แยก feature เป็น package
2. **Dart Plugin** - สำหรับ native features
3. **Dynamic Feature Modules** (Android) / **App Extensions** (iOS)

```dart
// lib/core/module_registry.dart
abstract class FeatureModule {
  String get name;
  String get version;
  List<String> get dependencies;
  
  Future<void> initialize();
  Widget buildFeature(BuildContext context, Map<String, dynamic> params);
  List<Route> get routes;
}

class ModuleRegistry {
  static final ModuleRegistry _instance = ModuleRegistry._internal();
  factory ModuleRegistry() => _instance;
  ModuleRegistry._internal();
  
  final Map<String, FeatureModule> _modules = {};
  
  void register(FeatureModule module) {
    _modules[module.name] = module;
    print('Module registered: ${module.name} v${module.version}');
  }
  
  FeatureModule? getModule(String name) => _modules[name];
  
  List<FeatureModule> get allModules => _modules.values.toList();
  
  Future<void> initializeAll() async {
    for (final module in _modules.values) {
      await module.initialize();
    }
  }
  
  bool isRegistered(String name) => _modules.containsKey(name);
}
```

---

## ขั้นตอนที่ 402: Feature Modules

### การสร้าง Feature Module

Feature Module คือ Package ที่ encapsulate ทุกอย่างที่เกี่ยวกับ Feature หนึ่งๆ

```
packages/
├── core/                    # Shared kernel
│   ├── lib/
│   └── pubspec.yaml
├── feature_auth/            # Authentication feature
│   ├── lib/
│   │   ├── src/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   └── feature_auth.dart
│   └── pubspec.yaml
├── feature_shop/            # Shopping feature
│   ├── lib/
│   └── pubspec.yaml
└── feature_chat/            # Chat feature
    ├── lib/
    └── pubspec.yaml
```

### สร้าง Base Feature Module

```dart
// packages/core/lib/src/feature_module.dart
import 'package:flutter/material.dart';

abstract class FeatureModule {
  String get name;
  String get version;
  
  // Dependencies ที่ต้องการจาก module อื่น
  List<Type> get requiredDependencies => [];
  
  // Services ที่ module นี้ provide
  List<Type> get providedServices => [];
  
  // เรียกตอน app เริ่มต้น
  Future<void> onInit() async {}
  
  // เรียกตอน app ปิด
  Future<void> onDispose() async {}
  
  // Routes ที่ module นี้มี
  Map<String, WidgetBuilder> get routes => {};
  
  // Navigation entry point
  Widget? get navigationEntry => null;
  
  // Deep link handlers
  bool handleDeepLink(Uri uri) => false;
}
```

### ตัวอย่าง Auth Feature Module

```dart
// packages/feature_auth/lib/src/auth_module.dart
import 'package:core/core.dart';
import 'package:flutter/material.dart';

class AuthModule extends FeatureModule {
  @override
  String get name => 'auth';
  
  @override
  String get version => '1.0.0';
  
  @override
  List<Type> get providedServices => [AuthService, UserRepository];
  
  late final AuthService _authService;
  late final UserRepository _userRepository;
  
  @override
  Future<void> onInit() async {
    _userRepository = UserRepositoryImpl();
    _authService = AuthServiceImpl(repository: _userRepository);
    
    // ลงทะเบียน services กับ DI container
    ServiceLocator.register<AuthService>(_authService);
    ServiceLocator.register<UserRepository>(_userRepository);
  }
  
  @override
  Map<String, WidgetBuilder> get routes => {
    '/auth/login': (context) => const LoginScreen(),
    '/auth/register': (context) => const RegisterScreen(),
    '/auth/forgot-password': (context) => const ForgotPasswordScreen(),
    '/auth/profile': (context) => const ProfileScreen(),
  };
  
  @override
  Widget get navigationEntry => const LoginScreen();
  
  @override
  bool handleDeepLink(Uri uri) {
    if (uri.host == 'auth') {
      // Handle auth deep links
      return true;
    }
    return false;
  }
}
```

### Feature Module pubspec.yaml

```yaml
# packages/feature_auth/pubspec.yaml
name: feature_auth
description: Authentication feature module
version: 1.0.0

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.10.0'

dependencies:
  flutter:
    sdk: flutter
  core:
    path: ../core
  flutter_riverpod: ^2.4.0
  go_router: ^12.0.0
  flutter_secure_storage: ^9.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.0
```

---

## ขั้นตอนที่ 403: Lazy Loading Features

### แนวคิด Lazy Loading

Lazy Loading ช่วยให้แอปโหลด Feature เฉพาะที่จำเป็น ลดขนาด initial bundle

```dart
// lib/core/lazy_module_loader.dart
import 'dart:async';

typedef ModuleFactory = Future<FeatureModule> Function();

class LazyModuleLoader {
  final Map<String, ModuleFactory> _factories = {};
  final Map<String, FeatureModule> _loadedModules = {};
  final Map<String, Completer<FeatureModule>> _pendingLoads = {};
  
  void registerFactory(String name, ModuleFactory factory) {
    _factories[name] = factory;
  }
  
  Future<FeatureModule> loadModule(String name) async {
    // ถ้า module โหลดแล้ว return ทันที
    if (_loadedModules.containsKey(name)) {
      return _loadedModules[name]!;
    }
    
    // ถ้ากำลังโหลดอยู่ รอผล
    if (_pendingLoads.containsKey(name)) {
      return _pendingLoads[name]!.future;
    }
    
    // ตรวจสอบว่ามี factory
    final factory = _factories[name];
    if (factory == null) {
      throw ModuleNotFoundException('Module "$name" not found');
    }
    
    // เริ่มโหลด
    final completer = Completer<FeatureModule>();
    _pendingLoads[name] = completer;
    
    try {
      final module = await factory();
      await module.onInit();
      _loadedModules[name] = module;
      _pendingLoads.remove(name);
      completer.complete(module);
      return module;
    } catch (e) {
      _pendingLoads.remove(name);
      completer.completeError(e);
      rethrow;
    }
  }
  
  bool isLoaded(String name) => _loadedModules.containsKey(name);
  
  Future<void> unloadModule(String name) async {
    final module = _loadedModules[name];
    if (module != null) {
      await module.onDispose();
      _loadedModules.remove(name);
    }
  }
}

class ModuleNotFoundException implements Exception {
  final String message;
  ModuleNotFoundException(this.message);
  
  @override
  String toString() => 'ModuleNotFoundException: $message';
}
```

### Lazy Loading Widget

```dart
// lib/widgets/lazy_feature_widget.dart
class LazyFeatureWidget extends StatefulWidget {
  final String moduleName;
  final Map<String, dynamic> params;
  final Widget loadingWidget;
  final Widget Function(String error)? errorWidget;
  
  const LazyFeatureWidget({
    super.key,
    required this.moduleName,
    this.params = const {},
    this.loadingWidget = const CircularProgressIndicator(),
    this.errorWidget,
  });
  
  @override
  State<LazyFeatureWidget> createState() => _LazyFeatureWidgetState();
}

class _LazyFeatureWidgetState extends State<LazyFeatureWidget> {
  late Future<FeatureModule> _moduleFuture;
  
  @override
  void initState() {
    super.initState();
    _loadModule();
  }
  
  void _loadModule() {
    _moduleFuture = LazyModuleLoader().loadModule(widget.moduleName);
  }
  
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<FeatureModule>(
      future: _moduleFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return Center(child: widget.loadingWidget);
        }
        
        if (snapshot.hasError) {
          return widget.errorWidget?.call(snapshot.error.toString()) ??
              Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Icon(Icons.error_outline, size: 48, color: Colors.red),
                    const SizedBox(height: 16),
                    Text('ไม่สามารถโหลด Module ได้: ${snapshot.error}'),
                    ElevatedButton(
                      onPressed: () {
                        setState(() => _loadModule());
                      },
                      child: const Text('ลองใหม่'),
                    ),
                  ],
                ),
              );
        }
        
        final module = snapshot.data!;
        return module.navigationEntry ?? 
            const Center(child: Text('Module ไม่มี UI'));
      },
    );
  }
}
```

### Android Dynamic Feature Modules

```groovy
// app/build.gradle - สำหรับ Android Dynamic Delivery
android {
    dynamicFeatures = [':feature_shop', ':feature_chat']
}
```

```dart
// lib/services/dynamic_feature_manager.dart (Android only)
import 'package:play_feature_delivery/play_feature_delivery.dart';

class DynamicFeatureManager {
  static Future<bool> isModuleInstalled(String moduleName) async {
    final status = await PlayFeatureDelivery.getSessionState(moduleName);
    return status == ModuleInstallSessionStatus.installed;
  }
  
  static Future<void> requestInstall(String moduleName) async {
    final request = ModuleInstallRequest(
      moduleNames: [moduleName],
      listener: (status) {
        print('Install status: $status');
      },
    );
    await PlayFeatureDelivery.requestInstall(request);
  }
  
  static Future<void> deferredInstall(String moduleName) async {
    await PlayFeatureDelivery.deferredInstall([moduleName]);
  }
}
```

---

## ขั้นตอนที่ 404: Plugin-based Architecture

### Plugin System Design

```dart
// lib/core/plugin_system.dart
abstract class AppPlugin {
  String get id;
  String get name;
  String get version;
  PluginMetadata get metadata;
  
  Future<void> install(PluginContext context);
  Future<void> uninstall();
  Future<void> activate();
  Future<void> deactivate();
  
  bool get isActive;
}

class PluginMetadata {
  final String author;
  final String description;
  final List<String> permissions;
  final Map<String, String> dependencies;
  
  const PluginMetadata({
    required this.author,
    required this.description,
    this.permissions = const [],
    this.dependencies = const {},
  });
}

class PluginContext {
  final ServiceLocator services;
  final EventBus eventBus;
  final NavigationService navigation;
  final StorageService storage;
  
  const PluginContext({
    required this.services,
    required this.eventBus,
    required this.navigation,
    required this.storage,
  });
}

class PluginManager {
  final Map<String, AppPlugin> _plugins = {};
  final PluginContext _context;
  
  PluginManager(this._context);
  
  Future<void> installPlugin(AppPlugin plugin) async {
    if (_plugins.containsKey(plugin.id)) {
      throw PluginAlreadyInstalledException(plugin.id);
    }
    
    // ตรวจสอบ dependencies
    await _checkDependencies(plugin);
    
    // ตรวจสอบ permissions
    await _checkPermissions(plugin);
    
    // ติดตั้ง plugin
    await plugin.install(_context);
    _plugins[plugin.id] = plugin;
    
    print('Plugin installed: ${plugin.name} v${plugin.version}');
  }
  
  Future<void> activatePlugin(String pluginId) async {
    final plugin = _plugins[pluginId];
    if (plugin == null) throw PluginNotFoundException(pluginId);
    await plugin.activate();
  }
  
  Future<void> _checkDependencies(AppPlugin plugin) async {
    for (final entry in plugin.metadata.dependencies.entries) {
      final depId = entry.key;
      final requiredVersion = entry.value;
      
      final dep = _plugins[depId];
      if (dep == null) {
        throw PluginDependencyException(
          'Plugin "${plugin.name}" ต้องการ "$depId" แต่ไม่ได้ติดตั้ง'
        );
      }
      
      if (!_isVersionCompatible(dep.version, requiredVersion)) {
        throw PluginDependencyException(
          'Plugin "${plugin.name}" ต้องการ "$depId" เวอร์ชัน $requiredVersion'
          ' แต่ได้เวอร์ชัน ${dep.version}'
        );
      }
    }
  }
  
  Future<void> _checkPermissions(AppPlugin plugin) async {
    // ตรวจสอบ permissions ที่จำเป็น
    for (final permission in plugin.metadata.permissions) {
      final granted = await _requestPermission(permission);
      if (!granted) {
        throw PluginPermissionDeniedException(
          'Permission "$permission" ถูกปฏิเสธ'
        );
      }
    }
  }
  
  bool _isVersionCompatible(String actual, String required) {
    // Semantic versioning check
    final actualParts = actual.split('.').map(int.parse).toList();
    final requiredParts = required.split('.').map(int.parse).toList();
    return actualParts[0] == requiredParts[0] && 
           actualParts[1] >= requiredParts[1];
  }
  
  Future<bool> _requestPermission(String permission) async {
    // ขอ permission จาก user
    return true; // simplified
  }
  
  List<AppPlugin> get activePlugins =>
      _plugins.values.where((p) => p.isActive).toList();
}
```

---

## ขั้นตอนที่ 405: Shared Kernel Pattern

### Shared Kernel คืออะไร?

Shared Kernel คือส่วน code ที่ Feature Modules ทุกตัว share กัน เช่น Design System, Base Models, Utils, Constants

```
packages/core/
├── lib/
│   ├── src/
│   │   ├── design_system/        # UI Components ที่ share
│   │   │   ├── colors.dart
│   │   │   ├── typography.dart
│   │   │   ├── spacing.dart
│   │   │   └── components/
│   │   ├── domain/               # Base domain types
│   │   │   ├── entity.dart
│   │   │   ├── value_object.dart
│   │   │   └── result.dart
│   │   ├── services/             # Shared services
│   │   │   ├── analytics.dart
│   │   │   ├── logger.dart
│   │   │   └── event_bus.dart
│   │   └── utils/                # Utility functions
│   │       ├── date_utils.dart
│   │       ├── string_utils.dart
│   │       └── validators.dart
│   └── core.dart                 # Public API
└── pubspec.yaml
```

### Design System Tokens

```dart
// packages/core/lib/src/design_system/tokens.dart
class AppColors {
  // Primary
  static const Color primary = Color(0xFF6750A4);
  static const Color primaryContainer = Color(0xFFEADDFF);
  static const Color onPrimary = Color(0xFFFFFFFF);
  
  // Secondary
  static const Color secondary = Color(0xFF625B71);
  static const Color secondaryContainer = Color(0xFFE8DEF8);
  
  // Error
  static const Color error = Color(0xFFB3261E);
  static const Color errorContainer = Color(0xFFF9DEDC);
  
  // Neutral
  static const Color surface = Color(0xFFFFFBFE);
  static const Color background = Color(0xFFFFFBFE);
  static const Color outline = Color(0xFF79747E);
  
  AppColors._();
}

class AppSpacing {
  static const double xs = 4.0;
  static const double sm = 8.0;
  static const double md = 16.0;
  static const double lg = 24.0;
  static const double xl = 32.0;
  static const double xxl = 48.0;
  
  AppSpacing._();
}

class AppTypography {
  static const TextStyle displayLarge = TextStyle(
    fontSize: 57,
    fontWeight: FontWeight.w400,
    letterSpacing: -0.25,
  );
  
  static const TextStyle headlineMedium = TextStyle(
    fontSize: 28,
    fontWeight: FontWeight.w400,
  );
  
  static const TextStyle bodyLarge = TextStyle(
    fontSize: 16,
    fontWeight: FontWeight.w400,
    letterSpacing: 0.5,
  );
  
  static const TextStyle labelSmall = TextStyle(
    fontSize: 11,
    fontWeight: FontWeight.w500,
    letterSpacing: 0.5,
  );
  
  AppTypography._();
}
```

### Result Type สำหรับ Error Handling

```dart
// packages/core/lib/src/domain/result.dart
sealed class Result<T> {
  const Result();
  
  bool get isSuccess => this is Success<T>;
  bool get isFailure => this is Failure<T>;
  
  T? get dataOrNull => switch (this) {
    Success<T> s => s.data,
    Failure<T> _ => null,
  };
  
  AppError? get errorOrNull => switch (this) {
    Success<T> _ => null,
    Failure<T> f => f.error,
  };
  
  R when<R>({
    required R Function(T data) success,
    required R Function(AppError error) failure,
  }) => switch (this) {
    Success<T> s => success(s.data),
    Failure<T> f => failure(f.error),
  };
  
  Result<R> map<R>(R Function(T data) transform) => switch (this) {
    Success<T> s => Success(transform(s.data)),
    Failure<T> f => Failure(f.error),
  };
  
  Future<Result<R>> asyncMap<R>(Future<R> Function(T data) transform) async {
    return switch (this) {
      Success<T> s => Success(await transform(s.data)),
      Failure<T> f => Failure(f.error),
    };
  }
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Failure<T> extends Result<T> {
  final AppError error;
  const Failure(this.error);
}

class AppError {
  final String code;
  final String message;
  final dynamic originalError;
  final StackTrace? stackTrace;
  
  const AppError({
    required this.code,
    required this.message,
    this.originalError,
    this.stackTrace,
  });
  
  factory AppError.network(String message) =>
      AppError(code: 'NETWORK_ERROR', message: message);
  
  factory AppError.unauthorized() =>
      const AppError(code: 'UNAUTHORIZED', message: 'ไม่มีสิทธิ์เข้าถึง');
  
  factory AppError.notFound(String resource) =>
      AppError(code: 'NOT_FOUND', message: 'ไม่พบ $resource');
  
  factory AppError.unknown(dynamic error, [StackTrace? stack]) =>
      AppError(
        code: 'UNKNOWN',
        message: 'เกิดข้อผิดพลาดที่ไม่รู้จัก',
        originalError: error,
        stackTrace: stack,
      );
}
```

---

## ขั้นตอนที่ 406: Monorepo with Melos

### Melos คืออะไร?

Melos เป็น tool จัดการ Dart/Flutter Monorepo โดยเฉพาะ ช่วยจัดการ packages หลายๆ ตัวในโปรเจคเดียว

### การติดตั้ง Melos

```bash
# ติดตั้ง melos แบบ global
dart pub global activate melos

# ตรวจสอบ version
melos --version
```

### โครงสร้าง Monorepo

```
my_app/
├── melos.yaml                   # Melos configuration
├── packages/
│   ├── core/
│   ├── feature_auth/
│   ├── feature_shop/
│   ├── feature_chat/
│   └── feature_profile/
├── apps/
│   ├── main_app/                # Main Flutter app
│   └── design_system_demo/     # Design system playground
└── pubspec.yaml                 # Root pubspec (for melos)
```

### melos.yaml Configuration

```yaml
# melos.yaml
name: my_app_workspace

packages:
  - packages/**
  - apps/**

command:
  bootstrap:
    usePubspecOverrides: true

scripts:
  analyze:
    run: melos exec -- dart analyze .
    description: Run dart analyze in all packages
    
  test:
    run: melos exec -- flutter test
    description: Run tests in all packages
    
  test:coverage:
    run: |
      melos exec -- flutter test --coverage
      melos exec -- genhtml coverage/lcov.info -o coverage/html
    description: Run tests with coverage
    
  clean:
    run: melos exec -- flutter clean
    description: Clean all packages
    
  build:runner:
    run: melos exec --depends-on="build_runner" -- dart run build_runner build --delete-conflicting-outputs
    description: Run build_runner in all packages
    
  format:
    run: melos exec -- dart format lib test
    description: Format code in all packages
    
  version:
    run: melos version
    description: Bump versions
    
  publish:
    run: melos publish
    description: Publish packages
    
  gen:mock:
    run: melos exec --depends-on="mockito" -- dart run build_runner build
    description: Generate mocks

ide:
  intellij: true
```

### Root pubspec.yaml

```yaml
# pubspec.yaml (root)
name: my_app_root
description: Monorepo root

environment:
  sdk: '>=3.0.0 <4.0.0'

dev_dependencies:
  melos: ^3.0.0
```

### การใช้ Melos Commands

```bash
# Bootstrap - install dependencies สำหรับ packages ทั้งหมด
melos bootstrap

# หรือย่อ
melos bs

# Run script ที่กำหนดใน melos.yaml
melos run analyze
melos run test
melos run clean

# Execute command ใน package ทั้งหมด
melos exec -- flutter pub upgrade

# Execute เฉพาะ package ที่มี dependency นี้
melos exec --depends-on="riverpod" -- flutter test

# List packages
melos list

# List พร้อม dependency graph
melos list --graph

# Version bump
melos version --graduate

# Publish packages
melos publish --dry-run
```

### Package Dependencies ใน Monorepo

```yaml
# packages/feature_auth/pubspec.yaml
dependencies:
  core:
    path: ../core  # Local path dependency

# เมื่อใช้ melos สามารถใช้ workspace override ได้
# melos จะ override path dependency อัตโนมัติ
```

---

## ขั้นตอนที่ 407: Inter-module Communication

### Event Bus Pattern

```dart
// packages/core/lib/src/services/event_bus.dart
class AppEvent {
  final String type;
  final Map<String, dynamic> data;
  final DateTime timestamp;
  
  AppEvent({
    required this.type,
    this.data = const {},
    DateTime? timestamp,
  }) : timestamp = timestamp ?? DateTime.now();
  
  @override
  String toString() => 'AppEvent(type: $type, data: $data)';
}

class EventBus {
  static final EventBus _instance = EventBus._internal();
  factory EventBus() => _instance;
  EventBus._internal();
  
  final StreamController<AppEvent> _controller =
      StreamController<AppEvent>.broadcast();
  
  Stream<AppEvent> get stream => _controller.stream;
  
  // Subscribe ไปยัง event type เฉพาะ
  Stream<AppEvent> on(String eventType) {
    return stream.where((event) => event.type == eventType);
  }
  
  // Subscribe หลาย event types
  Stream<AppEvent> onAny(List<String> eventTypes) {
    return stream.where((event) => eventTypes.contains(event.type));
  }
  
  void emit(AppEvent event) {
    if (!_controller.isClosed) {
      _controller.add(event);
    }
  }
  
  void emitType(String type, [Map<String, dynamic> data = const {}]) {
    emit(AppEvent(type: type, data: data));
  }
  
  void dispose() {
    _controller.close();
  }
}

// Event Types Constants
class AppEvents {
  // Auth events
  static const String userLoggedIn = 'user.logged_in';
  static const String userLoggedOut = 'user.logged_out';
  static const String userProfileUpdated = 'user.profile_updated';
  
  // Shop events
  static const String itemAddedToCart = 'shop.item_added_to_cart';
  static const String orderPlaced = 'shop.order_placed';
  static const String orderDelivered = 'shop.order_delivered';
  
  // Notification events
  static const String pushReceived = 'notification.push_received';
  
  AppEvents._();
}
```

### Module Communication ผ่าน Shared Service

```dart
// packages/core/lib/src/services/auth_service_interface.dart
abstract class AuthServiceInterface {
  Stream<User?> get currentUserStream;
  User? get currentUser;
  bool get isLoggedIn;
  
  Future<Result<User>> signIn(String email, String password);
  Future<Result<void>> signOut();
  Future<Result<User>> getProfile();
}

// Feature modules อื่นๆ สามารถ inject AuthServiceInterface ได้
// โดยไม่ต้อง depend on feature_auth package โดยตรง

// packages/feature_shop/lib/src/product_service.dart
class ProductService {
  final AuthServiceInterface _authService;
  final ProductRepository _repository;
  
  ProductService(this._authService, this._repository);
  
  Future<Result<List<Product>>> getPersonalizedProducts() async {
    if (!_authService.isLoggedIn) {
      return Failure(AppError.unauthorized());
    }
    
    final userId = _authService.currentUser!.id;
    return _repository.getRecommendedProducts(userId);
  }
}
```

### Navigation Service สำหรับ Cross-module Navigation

```dart
// packages/core/lib/src/services/navigation_service.dart
abstract class NavigationService {
  Future<T?> pushNamed<T>(String route, {Object? arguments});
  Future<void> pushReplacementNamed(String route, {Object? arguments});
  void pop<T>([T? result]);
  bool canPop();
  Future<T?> pushNamedAndRemoveUntil<T>(
    String route,
    RoutePredicate predicate, {
    Object? arguments,
  });
}

class RouterNavigationService implements NavigationService {
  final GoRouter _router;
  
  RouterNavigationService(this._router);
  
  @override
  Future<T?> pushNamed<T>(String route, {Object? arguments}) async {
    return _router.push<T>(route, extra: arguments);
  }
  
  @override
  Future<void> pushReplacementNamed(String route, {Object? arguments}) async {
    _router.pushReplacement(route, extra: arguments);
  }
  
  @override
  void pop<T>([T? result]) {
    _router.pop(result);
  }
  
  @override
  bool canPop() => _router.canPop();
  
  @override
  Future<T?> pushNamedAndRemoveUntil<T>(
    String route,
    RoutePredicate predicate, {
    Object? arguments,
  }) async {
    _router.go(route, extra: arguments);
    return null;
  }
}
```

---

## ขั้นตอนที่ 408: Independent Deployment of Features

### Feature Flags สำหรับ Independent Deployment

```dart
// packages/core/lib/src/feature_flags/feature_flag_service.dart
enum FeatureFlag {
  newCheckoutFlow,
  aiRecommendations,
  liveChat,
  darkMode,
  betaFeatures,
}

abstract class FeatureFlagService {
  bool isEnabled(FeatureFlag flag);
  Future<void> refresh();
  Stream<Map<FeatureFlag, bool>> get flagsStream;
}

class RemoteFeatureFlagService implements FeatureFlagService {
  final http.Client _client;
  final String _baseUrl;
  final Map<FeatureFlag, bool> _flags = {};
  final StreamController<Map<FeatureFlag, bool>> _controller =
      StreamController.broadcast();
  
  RemoteFeatureFlagService({
    required http.Client client,
    required String baseUrl,
  }) : _client = client, _baseUrl = baseUrl;
  
  @override
  bool isEnabled(FeatureFlag flag) => _flags[flag] ?? false;
  
  @override
  Future<void> refresh() async {
    try {
      final response = await _client.get(
        Uri.parse('$_baseUrl/feature-flags'),
      );
      
      if (response.statusCode == 200) {
        final data = jsonDecode(response.body) as Map<String, dynamic>;
        for (final flag in FeatureFlag.values) {
          _flags[flag] = data[flag.name] as bool? ?? false;
        }
        _controller.add(Map.from(_flags));
      }
    } catch (e) {
      print('Feature flag refresh failed: $e');
    }
  }
  
  @override
  Stream<Map<FeatureFlag, bool>> get flagsStream => _controller.stream;
}
```

### Versioned Module Deployment

```dart
// lib/core/module_version_manager.dart
class ModuleVersionManager {
  static const String _storageKey = 'module_versions';
  final SharedPreferences _prefs;
  
  ModuleVersionManager(this._prefs);
  
  Future<String?> getInstalledVersion(String moduleName) async {
    final versions = _getVersionMap();
    return versions[moduleName];
  }
  
  Future<void> setInstalledVersion(String moduleName, String version) async {
    final versions = _getVersionMap();
    versions[moduleName] = version;
    await _prefs.setString(_storageKey, jsonEncode(versions));
  }
  
  Map<String, String> _getVersionMap() {
    final json = _prefs.getString(_storageKey);
    if (json == null) return {};
    return Map<String, String>.from(jsonDecode(json));
  }
  
  Future<bool> needsUpdate(String moduleName, String latestVersion) async {
    final installed = await getInstalledVersion(moduleName);
    if (installed == null) return true;
    return _compareVersions(installed, latestVersion) < 0;
  }
  
  int _compareVersions(String v1, String v2) {
    final parts1 = v1.split('.').map(int.parse).toList();
    final parts2 = v2.split('.').map(int.parse).toList();
    
    for (int i = 0; i < 3; i++) {
      final a = parts1[i];
      final b = parts2[i];
      if (a != b) return a.compareTo(b);
    }
    return 0;
  }
}
```

---

## ขั้นตอนที่ 409: Testing Micro Frontend Modules

### Unit Testing Feature Modules

```dart
// packages/feature_auth/test/auth_module_test.dart
import 'package:feature_auth/feature_auth.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockUserRepository extends Mock implements UserRepository {}
class MockEventBus extends Mock implements EventBus {}

void main() {
  group('AuthModule Tests', () {
    late AuthModule authModule;
    late MockUserRepository mockRepository;
    late MockEventBus mockEventBus;
    
    setUp(() {
      mockRepository = MockUserRepository();
      mockEventBus = MockEventBus();
      authModule = AuthModule(
        userRepository: mockRepository,
        eventBus: mockEventBus,
      );
    });
    
    test('module name should be auth', () {
      expect(authModule.name, equals('auth'));
    });
    
    test('should register services on init', () async {
      await authModule.onInit();
      
      expect(ServiceLocator.isRegistered<AuthService>(), isTrue);
      expect(ServiceLocator.isRegistered<UserRepository>(), isTrue);
    });
    
    test('should emit event on login', () async {
      when(() => mockRepository.signIn(any(), any()))
          .thenAnswer((_) async => Success(User.fixture()));
      
      final events = <AppEvent>[];
      when(() => mockEventBus.emit(any())).thenAnswer((invocation) {
        events.add(invocation.positionalArguments.first as AppEvent);
      });
      
      final authService = AuthServiceImpl(
        repository: mockRepository,
        eventBus: mockEventBus,
      );
      
      await authService.signIn('test@test.com', 'password');
      
      expect(events, hasLength(1));
      expect(events.first.type, equals(AppEvents.userLoggedIn));
    });
  });
}
```

### Integration Testing ระหว่าง Modules

```dart
// test/integration/auth_shop_integration_test.dart
void main() {
  group('Auth-Shop Integration Tests', () {
    late AuthModule authModule;
    late ShopModule shopModule;
    late EventBus eventBus;
    
    setUpAll(() async {
      eventBus = EventBus();
      authModule = AuthModule(eventBus: eventBus);
      shopModule = ShopModule(eventBus: eventBus);
      
      await authModule.onInit();
      await shopModule.onInit();
    });
    
    test('shop should load personalized content after login', () async {
      // Login
      final loginResult = await authModule.authService.signIn(
        'test@test.com', 'password',
      );
      expect(loginResult.isSuccess, isTrue);
      
      // Shop module ควร receive event และโหลด personalized content
      final products = await shopModule.productService.getPersonalizedProducts();
      expect(products.isSuccess, isTrue);
    });
  });
}
```

---

## ขั้นตอนที่ 410: Workshop - สร้าง Micro Frontend App

### โครงสร้างโปรเจคทั้งหมด

```
micro_frontend_demo/
├── melos.yaml
├── pubspec.yaml
├── apps/
│   └── shell_app/
│       ├── lib/
│       │   ├── main.dart
│       │   ├── app.dart
│       │   ├── routes/
│       │   │   └── app_router.dart
│       │   └── bootstrap/
│       │       └── app_bootstrap.dart
│       └── pubspec.yaml
└── packages/
    ├── core/
    ├── feature_auth/
    ├── feature_home/
    └── feature_settings/
```

### Shell App - main.dart

```dart
// apps/shell_app/lib/main.dart
import 'package:flutter/material.dart';
import 'package:core/core.dart';
import 'package:feature_auth/feature_auth.dart';
import 'package:feature_home/feature_home.dart';
import 'package:feature_settings/feature_settings.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Bootstrap application
  await AppBootstrap.initialize(
    modules: [
      AuthModule(),
      HomeModule(),
      SettingsModule(),
    ],
  );
  
  runApp(const MyApp());
}
```

### App Bootstrap

```dart
// apps/shell_app/lib/bootstrap/app_bootstrap.dart
class AppBootstrap {
  static Future<void> initialize({
    required List<FeatureModule> modules,
  }) async {
    // Initialize core services
    await _initializeCoreServices();
    
    // Register modules
    final registry = ModuleRegistry();
    for (final module in modules) {
      registry.register(module);
    }
    
    // Initialize all modules
    await registry.initializeAll();
    
    print('App bootstrap completed. Modules: ${registry.allModules.length}');
  }
  
  static Future<void> _initializeCoreServices() async {
    // Initialize shared services
    final prefs = await SharedPreferences.getInstance();
    ServiceLocator.register<SharedPreferences>(prefs);
    
    final eventBus = EventBus();
    ServiceLocator.register<EventBus>(eventBus);
    
    // Logger
    final logger = AppLogger(level: LogLevel.debug);
    ServiceLocator.register<AppLogger>(logger);
    
    print('Core services initialized');
  }
}
```

### Workshop Task

1. สร้าง `feature_profile` module ที่มี Profile Screen
2. เพิ่ม Event Bus communication ระหว่าง Auth และ Profile modules
3. Implement Lazy Loading สำหรับ Profile module
4. เพิ่ม Feature Flag สำหรับ Beta features
5. เขียน Integration tests

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **Module Federation**: แนวคิดการแบ่ง app เป็น feature modules อิสระ
- **Feature Modules**: การ encapsulate feature ใน package แยก
- **Lazy Loading**: โหลด feature เฉพาะที่จำเป็น
- **Plugin Architecture**: ระบบ plugin สำหรับการขยายฟีเจอร์
- **Shared Kernel**: Design system และ utilities ที่ share กัน
- **Melos**: การจัดการ monorepo สำหรับ Flutter
- **Inter-module Communication**: Event Bus และ Shared Service interfaces
- **Independent Deployment**: Feature flags และ versioning

## แบบฝึกหัด

1. สร้าง monorepo ที่มี 3 feature packages ใช้ Melos
2. Implement EventBus และทดสอบการส่ง events ระหว่าง modules
3. สร้าง Shared Kernel ที่มี Design System tokens
4. Implement Feature Flag system เชื่อมกับ remote config
5. เขียน Unit tests สำหรับแต่ละ module อย่างครบถ้วน

---

[⬅️ Part 40](part_40.md) | [Part 42 ➡️](part_42.md)

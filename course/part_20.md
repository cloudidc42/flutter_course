# Part 20: SharedPreferences & Secure Storage
## ขั้นตอนที่ 191-200

---

## สารบัญ
1. [SharedPreferences พื้นฐาน](#sharedpreferences-พื้นฐาน)
2. [การอ่านและเขียนข้อมูล](#การอ่านและเขียนข้อมูล)
3. [SharedPreferences กับ Complex Data](#complex-data)
4. [Settings Pattern](#settings-pattern)
5. [flutter_secure_storage](#flutter_secure_storage)
6. [Secure Storage Use Cases](#secure-storage-use-cases)
7. [First-Run Detection](#first-run-detection)
8. [User Preferences Management](#user-preferences-management)
9. [Storage Abstraction Layer](#storage-abstraction-layer)
10. [Workshop: Settings App](#workshop)

---

## ขั้นตอนที่ 191: SharedPreferences พื้นฐาน

SharedPreferences เป็น plugin สำหรับเก็บข้อมูล key-value แบบง่ายบนอุปกรณ์

```yaml
# pubspec.yaml
dependencies:
  shared_preferences: ^2.2.2
  flutter_secure_storage: ^9.0.0
```

### หลักการทำงาน

```
SharedPreferences
├── Android: SharedPreferences XML
├── iOS/macOS: NSUserDefaults
├── Web: localStorage
├── Linux/Windows: INI files
└── ประเภทที่รองรับ: String, int, double, bool, List<String>
```

---

## ขั้นตอนที่ 192: การอ่านและเขียนข้อมูล

```dart
import 'package:shared_preferences/shared_preferences.dart';

class PrefsHelper {
  static SharedPreferences? _prefs;

  // เรียกครั้งแรกก่อนใช้งาน
  static Future<void> init() async {
    _prefs = await SharedPreferences.getInstance();
  }

  static SharedPreferences get prefs {
    assert(_prefs != null, 'Call PrefsHelper.init() first');
    return _prefs!;
  }

  // ===== String =====
  static String getString(String key, {String defaultValue = ''}) {
    return prefs.getString(key) ?? defaultValue;
  }

  static Future<bool> setString(String key, String value) {
    return prefs.setString(key, value);
  }

  // ===== int =====
  static int getInt(String key, {int defaultValue = 0}) {
    return prefs.getInt(key) ?? defaultValue;
  }

  static Future<bool> setInt(String key, int value) {
    return prefs.setInt(key, value);
  }

  // ===== double =====
  static double getDouble(String key, {double defaultValue = 0.0}) {
    return prefs.getDouble(key) ?? defaultValue;
  }

  static Future<bool> setDouble(String key, double value) {
    return prefs.setDouble(key, value);
  }

  // ===== bool =====
  static bool getBool(String key, {bool defaultValue = false}) {
    return prefs.getBool(key) ?? defaultValue;
  }

  static Future<bool> setBool(String key, bool value) {
    return prefs.setBool(key, value);
  }

  // ===== List<String> =====
  static List<String> getStringList(String key) {
    return prefs.getStringList(key) ?? [];
  }

  static Future<bool> setStringList(String key, List<String> value) {
    return prefs.setStringList(key, value);
  }

  // ===== Remove / Clear =====
  static Future<bool> remove(String key) => prefs.remove(key);

  static Future<bool> clear() => prefs.clear();

  static bool containsKey(String key) => prefs.containsKey(key);

  // ===== All Keys =====
  static Set<String> get keys => prefs.getKeys();
}

// ตัวอย่างการใช้งาน
Future<void> exampleUsage() async {
  await PrefsHelper.init();

  // เขียน
  await PrefsHelper.setString('username', 'สมชาย');
  await PrefsHelper.setInt('age', 25);
  await PrefsHelper.setBool('isDarkMode', true);
  await PrefsHelper.setDouble('fontSize', 16.0);
  await PrefsHelper.setStringList('recentSearches', ['flutter', 'dart', 'mobile']);

  // อ่าน
  String name = PrefsHelper.getString('username');
  int age = PrefsHelper.getInt('age');
  bool darkMode = PrefsHelper.getBool('isDarkMode');
  double fontSize = PrefsHelper.getDouble('fontSize');
  List<String> searches = PrefsHelper.getStringList('recentSearches');

  print('Name: $name, Age: $age, Dark: $darkMode');
  print('Searches: $searches');

  // ลบ
  await PrefsHelper.remove('age');

  // ตรวจสอบ
  print('Has username: ${PrefsHelper.containsKey('username')}');
  print('Has age: ${PrefsHelper.containsKey('age')}');  // false
}
```

---

## ขั้นตอนที่ 193: SharedPreferences กับ Complex Data

```dart
import 'dart:convert';

// เก็บ Object ด้วย JSON encoding
class UserProfile {
  final String id;
  final String name;
  final String email;
  final String? avatarUrl;
  final DateTime createdAt;

  const UserProfile({
    required this.id,
    required this.name,
    required this.email,
    this.avatarUrl,
    required this.createdAt,
  });

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
    'avatarUrl': avatarUrl,
    'createdAt': createdAt.toIso8601String(),
  };

  factory UserProfile.fromJson(Map<String, dynamic> json) {
    return UserProfile(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      avatarUrl: json['avatarUrl'],
      createdAt: DateTime.parse(json['createdAt']),
    );
  }
}

class UserPrefs {
  static const String _keyProfile = 'user_profile';
  static const String _keyLastLogin = 'last_login';
  static const String _keyRecentItems = 'recent_items';

  static Future<void> saveProfile(UserProfile profile) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_keyProfile, jsonEncode(profile.toJson()));
    await prefs.setString(_keyLastLogin, DateTime.now().toIso8601String());
  }

  static Future<UserProfile?> loadProfile() async {
    final prefs = await SharedPreferences.getInstance();
    final String? json = prefs.getString(_keyProfile);
    if (json == null) return null;

    try {
      return UserProfile.fromJson(jsonDecode(json));
    } catch (_) {
      return null;
    }
  }

  static Future<DateTime?> getLastLogin() async {
    final prefs = await SharedPreferences.getInstance();
    final String? dateStr = prefs.getString(_keyLastLogin);
    if (dateStr == null) return null;
    return DateTime.tryParse(dateStr);
  }

  static Future<void> addRecentItem(String itemId) async {
    final prefs = await SharedPreferences.getInstance();
    List<String> items = prefs.getStringList(_keyRecentItems) ?? [];

    items.remove(itemId);          // ลบถ้ามีอยู่แล้ว
    items.insert(0, itemId);       // เพิ่มที่ต้น
    if (items.length > 20) {
      items = items.sublist(0, 20); // เก็บแค่ 20 รายการล่าสุด
    }

    await prefs.setStringList(_keyRecentItems, items);
  }

  static Future<List<String>> getRecentItems() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getStringList(_keyRecentItems) ?? [];
  }

  static Future<void> clearProfile() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_keyProfile);
    await prefs.remove(_keyLastLogin);
    await prefs.remove(_keyRecentItems);
  }
}
```

---

## ขั้นตอนที่ 194: Settings Pattern

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// Type-safe settings keys
class SettingsKeys {
  static const String theme = 'theme_mode';
  static const String language = 'language';
  static const String fontSize = 'font_size';
  static const String notifications = 'notifications_enabled';
  static const String autoPlay = 'auto_play';
  static const String cacheSize = 'cache_size_mb';
  static const String onboardingDone = 'onboarding_done';

  SettingsKeys._();
}

class AppSettings extends ChangeNotifier {
  static AppSettings? _instance;
  late SharedPreferences _prefs;

  factory AppSettings() {
    _instance ??= AppSettings._();
    return _instance!;
  }
  AppSettings._();

  Future<void> initialize() async {
    _prefs = await SharedPreferences.getInstance();
  }

  // Theme
  ThemeMode get themeMode {
    final value = _prefs.getString(SettingsKeys.theme) ?? 'system';
    return switch (value) {
      'light' => ThemeMode.light,
      'dark' => ThemeMode.dark,
      _ => ThemeMode.system,
    };
  }

  Future<void> setThemeMode(ThemeMode mode) async {
    final value = switch (mode) {
      ThemeMode.light => 'light',
      ThemeMode.dark => 'dark',
      ThemeMode.system => 'system',
    };
    await _prefs.setString(SettingsKeys.theme, value);
    notifyListeners();
  }

  // Language
  String get language => _prefs.getString(SettingsKeys.language) ?? 'th';

  Future<void> setLanguage(String lang) async {
    await _prefs.setString(SettingsKeys.language, lang);
    notifyListeners();
  }

  // Font size
  double get fontSize => _prefs.getDouble(SettingsKeys.fontSize) ?? 14.0;

  Future<void> setFontSize(double size) async {
    await _prefs.setDouble(SettingsKeys.fontSize, size.clamp(10.0, 24.0));
    notifyListeners();
  }

  // Notifications
  bool get notificationsEnabled =>
      _prefs.getBool(SettingsKeys.notifications) ?? true;

  Future<void> setNotificationsEnabled(bool enabled) async {
    await _prefs.setBool(SettingsKeys.notifications, enabled);
    notifyListeners();
  }

  // Auto-play
  bool get autoPlay => _prefs.getBool(SettingsKeys.autoPlay) ?? false;

  Future<void> setAutoPlay(bool value) async {
    await _prefs.setBool(SettingsKeys.autoPlay, value);
    notifyListeners();
  }

  // Cache size (MB)
  int get cacheSizeMb => _prefs.getInt(SettingsKeys.cacheSize) ?? 100;

  Future<void> setCacheSizeMb(int mb) async {
    await _prefs.setInt(SettingsKeys.cacheSize, mb.clamp(50, 500));
    notifyListeners();
  }

  // Reset all settings
  Future<void> resetToDefaults() async {
    await _prefs.remove(SettingsKeys.theme);
    await _prefs.remove(SettingsKeys.language);
    await _prefs.remove(SettingsKeys.fontSize);
    await _prefs.remove(SettingsKeys.notifications);
    await _prefs.remove(SettingsKeys.autoPlay);
    await _prefs.remove(SettingsKeys.cacheSize);
    notifyListeners();
  }
}
```

---

## ขั้นตอนที่ 195: flutter_secure_storage

Secure Storage เก็บข้อมูลสำคัญอย่างปลอดภัยด้วย Keychain (iOS) และ Keystore (Android)

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorageService {
  static const FlutterSecureStorage _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,  // ใช้ EncryptedSharedPreferences
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );

  // ===== Keys =====
  static const String _keyAccessToken = 'access_token';
  static const String _keyRefreshToken = 'refresh_token';
  static const String _keyUserId = 'user_id';
  static const String _keyPin = 'user_pin';
  static const String _keyBiometricKey = 'biometric_key';

  // ===== Token Management =====
  static Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _keyAccessToken, value: accessToken),
      _storage.write(key: _keyRefreshToken, value: refreshToken),
    ]);
  }

  static Future<String?> getAccessToken() async {
    return await _storage.read(key: _keyAccessToken);
  }

  static Future<String?> getRefreshToken() async {
    return await _storage.read(key: _keyRefreshToken);
  }

  static Future<void> clearTokens() async {
    await Future.wait([
      _storage.delete(key: _keyAccessToken),
      _storage.delete(key: _keyRefreshToken),
    ]);
  }

  // ===== User ID =====
  static Future<void> saveUserId(String userId) async {
    await _storage.write(key: _keyUserId, value: userId);
  }

  static Future<String?> getUserId() async {
    return await _storage.read(key: _keyUserId);
  }

  // ===== PIN =====
  static Future<void> savePin(String pin) async {
    // ควร hash PIN ก่อนเก็บ
    final hashedPin = _hashPin(pin);
    await _storage.write(key: _keyPin, value: hashedPin);
  }

  static Future<bool> verifyPin(String pin) async {
    final stored = await _storage.read(key: _keyPin);
    if (stored == null) return false;
    return stored == _hashPin(pin);
  }

  static Future<void> removePin() async {
    await _storage.delete(key: _keyPin);
  }

  static Future<bool> hasPin() async {
    final pin = await _storage.read(key: _keyPin);
    return pin != null;
  }

  // ===== Logout (ล้างทุกอย่าง) =====
  static Future<void> clearAll() async {
    await _storage.deleteAll();
  }

  // ===== ดู keys ทั้งหมด =====
  static Future<Map<String, String>> readAll() async {
    return await _storage.readAll();
  }

  static String _hashPin(String pin) {
    // ใน production ควรใช้ bcrypt หรือ PBKDF2
    // ตัวอย่างนี้ใช้วิธีง่ายๆ
    int hash = 0;
    for (int i = 0; i < pin.length; i++) {
      hash = (hash * 31 + pin.codeUnitAt(i)) & 0xFFFFFFFF;
    }
    return hash.toRadixString(16);
  }
}
```

---

## ขั้นตอนที่ 196: Secure Storage Use Cases

```dart
// Auth Token Manager
class AuthTokenManager {
  final FlutterSecureStorage _storage;

  AuthTokenManager([FlutterSecureStorage? storage])
      : _storage = storage ?? const FlutterSecureStorage();

  static const _accessKey = 'auth.access_token';
  static const _refreshKey = 'auth.refresh_token';
  static const _expiryKey = 'auth.token_expiry';

  Future<void> saveSession({
    required String accessToken,
    required String refreshToken,
    required DateTime expiry,
  }) async {
    await Future.wait([
      _storage.write(key: _accessKey, value: accessToken),
      _storage.write(key: _refreshKey, value: refreshToken),
      _storage.write(key: _expiryKey, value: expiry.toIso8601String()),
    ]);
  }

  Future<bool> isTokenValid() async {
    final expiryStr = await _storage.read(key: _expiryKey);
    if (expiryStr == null) return false;

    final expiry = DateTime.tryParse(expiryStr);
    if (expiry == null) return false;

    return DateTime.now().isBefore(expiry);
  }

  Future<bool> isTokenExpiringSoon({Duration threshold = const Duration(minutes: 5)}) async {
    final expiryStr = await _storage.read(key: _expiryKey);
    if (expiryStr == null) return true;

    final expiry = DateTime.tryParse(expiryStr);
    if (expiry == null) return true;

    return DateTime.now().isAfter(expiry.subtract(threshold));
  }

  Future<String?> getAccessToken() => _storage.read(key: _accessKey);
  Future<String?> getRefreshToken() => _storage.read(key: _refreshKey);

  Future<void> clearSession() => _storage.deleteAll();
}

// Certificate / API Key Storage
class ApiKeyStorage {
  final FlutterSecureStorage _storage = const FlutterSecureStorage();

  Future<void> storeApiKey(String service, String key) async {
    await _storage.write(key: 'api_key_$service', value: key);
  }

  Future<String?> getApiKey(String service) async {
    return await _storage.read(key: 'api_key_$service');
  }

  Future<void> rotateApiKey(String service, String newKey) async {
    final old = await getApiKey(service);
    if (old == null) {
      await storeApiKey(service, newKey);
      return;
    }
    // อาจ backup key เก่าไว้ก่อน
    await _storage.write(key: 'api_key_${service}_prev', value: old);
    await storeApiKey(service, newKey);
  }
}
```

---

## ขั้นตอนที่ 197: First-Run Detection

```dart
class AppLaunchManager {
  static const String _isFirstRunKey = 'app.first_run';
  static const String _launchCountKey = 'app.launch_count';
  static const String _lastVersionKey = 'app.last_version';
  static const String _installDateKey = 'app.install_date';

  final SharedPreferences _prefs;
  AppLaunchManager(this._prefs);

  static Future<AppLaunchManager> create() async {
    final prefs = await SharedPreferences.getInstance();
    return AppLaunchManager(prefs);
  }

  // ตรวจสอบว่าเป็นการเปิดครั้งแรกหรือไม่
  bool get isFirstRun => !_prefs.containsKey(_isFirstRunKey);

  // บันทึกว่า onboarding เสร็จแล้ว
  Future<void> markFirstRunComplete() async {
    await _prefs.setBool(_isFirstRunKey, false);
    await _prefs.setString(_installDateKey, DateTime.now().toIso8601String());
  }

  // จำนวนครั้งที่เปิดแอป
  int get launchCount => _prefs.getInt(_launchCountKey) ?? 0;

  Future<void> incrementLaunchCount() async {
    await _prefs.setInt(_launchCountKey, launchCount + 1);
  }

  // เวอร์ชันล่าสุดที่ใช้งาน
  String? get lastVersion => _prefs.getString(_lastVersionKey);

  Future<void> updateLastVersion(String version) async {
    await _prefs.setString(_lastVersionKey, version);
  }

  // ตรวจสอบว่า version เปลี่ยนหรือไม่ (สำหรับ What's New)
  bool isNewVersion(String currentVersion) {
    return lastVersion != currentVersion;
  }

  // วันที่ติดตั้ง
  DateTime? get installDate {
    final str = _prefs.getString(_installDateKey);
    if (str == null) return null;
    return DateTime.tryParse(str);
  }

  // จำนวนวันที่ใช้งาน
  int get daysSinceInstall {
    final date = installDate;
    if (date == null) return 0;
    return DateTime.now().difference(date).inDays;
  }
}

// ใช้งานใน main.dart
Future<void> initializeApp() async {
  final launcher = await AppLaunchManager.create();

  if (launcher.isFirstRun) {
    print('Welcome! First time user');
    // แสดง onboarding
    await launcher.markFirstRunComplete();
    await launcher.updateLastVersion('1.0.0');
  } else if (launcher.isNewVersion('1.1.0')) {
    print("What's new in 1.1.0!");
    await launcher.updateLastVersion('1.1.0');
  }

  await launcher.incrementLaunchCount();
  print('Launch count: ${launcher.launchCount}');
  print('Days since install: ${launcher.daysSinceInstall}');
}
```

---

## ขั้นตอนที่ 198: User Preferences Management

```dart
class UserPreferencesManager extends ChangeNotifier {
  final SharedPreferences _prefs;

  UserPreferencesManager(this._prefs);

  static Future<UserPreferencesManager> create() async {
    final prefs = await SharedPreferences.getInstance();
    return UserPreferencesManager(prefs);
  }

  // ===== Notification Preferences =====
  bool get pushEnabled => _prefs.getBool('notif.push') ?? true;
  bool get emailEnabled => _prefs.getBool('notif.email') ?? true;
  bool get smsEnabled => _prefs.getBool('notif.sms') ?? false;
  bool get marketingEnabled => _prefs.getBool('notif.marketing') ?? false;

  Future<void> setPushEnabled(bool v) async {
    await _prefs.setBool('notif.push', v);
    notifyListeners();
  }

  Future<void> setEmailEnabled(bool v) async {
    await _prefs.setBool('notif.email', v);
    notifyListeners();
  }

  // ===== Privacy Preferences =====
  bool get analyticsEnabled => _prefs.getBool('privacy.analytics') ?? true;
  bool get crashReportingEnabled => _prefs.getBool('privacy.crash') ?? true;

  Future<void> setAnalyticsEnabled(bool v) async {
    await _prefs.setBool('privacy.analytics', v);
    notifyListeners();
  }

  // ===== Display Preferences =====
  String get currency => _prefs.getString('display.currency') ?? 'THB';
  String get dateFormat => _prefs.getString('display.date_format') ?? 'DD/MM/YYYY';
  bool get use24HourTime => _prefs.getBool('display.24hour') ?? false;

  Future<void> setCurrency(String currency) async {
    await _prefs.setString('display.currency', currency);
    notifyListeners();
  }

  // ===== Search History =====
  List<String> get searchHistory =>
      _prefs.getStringList('search.history') ?? [];

  Future<void> addSearchTerm(String term) async {
    final history = searchHistory;
    history.remove(term);
    history.insert(0, term);
    final trimmed = history.take(10).toList();
    await _prefs.setStringList('search.history', trimmed);
    notifyListeners();
  }

  Future<void> clearSearchHistory() async {
    await _prefs.remove('search.history');
    notifyListeners();
  }

  // ===== Favorites =====
  List<String> get favorites =>
      _prefs.getStringList('user.favorites') ?? [];

  bool isFavorite(String id) => favorites.contains(id);

  Future<void> toggleFavorite(String id) async {
    final favs = favorites;
    if (favs.contains(id)) {
      favs.remove(id);
    } else {
      favs.add(id);
    }
    await _prefs.setStringList('user.favorites', favs);
    notifyListeners();
  }
}
```

---

## ขั้นตอนที่ 199: Storage Abstraction Layer

```dart
// Abstract interface
abstract class StorageService {
  Future<void> write(String key, String value);
  Future<String?> read(String key);
  Future<void> delete(String key);
  Future<bool> containsKey(String key);
  Future<void> clear();
}

// SharedPreferences implementation
class SharedPrefsStorage implements StorageService {
  @override
  Future<void> write(String key, String value) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(key, value);
  }

  @override
  Future<String?> read(String key) async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(key);
  }

  @override
  Future<void> delete(String key) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(key);
  }

  @override
  Future<bool> containsKey(String key) async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.containsKey(key);
  }

  @override
  Future<void> clear() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();
  }
}

// Secure Storage implementation
class SecureStorage implements StorageService {
  final FlutterSecureStorage _storage;

  SecureStorage([FlutterSecureStorage? storage])
      : _storage = storage ?? const FlutterSecureStorage();

  @override
  Future<void> write(String key, String value) =>
      _storage.write(key: key, value: value);

  @override
  Future<String?> read(String key) => _storage.read(key: key);

  @override
  Future<void> delete(String key) => _storage.delete(key: key);

  @override
  Future<bool> containsKey(String key) => _storage.containsKey(key: key);

  @override
  Future<void> clear() => _storage.deleteAll();
}

// In-memory implementation (for testing)
class InMemoryStorage implements StorageService {
  final Map<String, String> _store = {};

  @override
  Future<void> write(String key, String value) async => _store[key] = value;

  @override
  Future<String?> read(String key) async => _store[key];

  @override
  Future<void> delete(String key) async => _store.remove(key);

  @override
  Future<bool> containsKey(String key) async => _store.containsKey(key);

  @override
  Future<void> clear() async => _store.clear();
}

// Repository ที่ใช้ StorageService
class TokenRepository {
  final StorageService _storage;

  TokenRepository(this._storage);

  Future<void> saveToken(String token) => _storage.write('token', token);
  Future<String?> getToken() => _storage.read('token');
  Future<void> clearToken() => _storage.delete('token');
}
```

---

## ขั้นตอนที่ 200: ตัวอย่าง Real-world Pattern

```dart
// หน้า Login ที่จำ username ไว้
class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});
  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailCtrl = TextEditingController();
  final _passwordCtrl = TextEditingController();
  bool _rememberMe = false;
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    _loadSavedEmail();
  }

  Future<void> _loadSavedEmail() async {
    final prefs = await SharedPreferences.getInstance();
    final saved = prefs.getString('saved_email');
    if (saved != null) {
      setState(() {
        _emailCtrl.text = saved;
        _rememberMe = true;
      });
    }
  }

  Future<void> _login() async {
    setState(() => _isLoading = true);

    try {
      final prefs = await SharedPreferences.getInstance();

      if (_rememberMe) {
        await prefs.setString('saved_email', _emailCtrl.text);
      } else {
        await prefs.remove('saved_email');
      }

      // ทำการ login จริงๆ และเก็บ token
      // final token = await authService.login(_emailCtrl.text, _passwordCtrl.text);
      // await SecureStorageService.saveTokens(accessToken: token.access, refreshToken: token.refresh);

      if (mounted) {
        Navigator.pushReplacementNamed(context, '/home');
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Login failed: $e'), backgroundColor: Colors.red),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  void dispose() {
    _emailCtrl.dispose();
    _passwordCtrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('เข้าสู่ระบบ')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _emailCtrl,
              keyboardType: TextInputType.emailAddress,
              decoration: const InputDecoration(
                labelText: 'อีเมล',
                prefixIcon: Icon(Icons.email),
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _passwordCtrl,
              obscureText: true,
              decoration: const InputDecoration(
                labelText: 'รหัสผ่าน',
                prefixIcon: Icon(Icons.lock),
                border: OutlineInputBorder(),
              ),
            ),
            Row(
              children: [
                Checkbox(
                  value: _rememberMe,
                  onChanged: (v) => setState(() => _rememberMe = v ?? false),
                ),
                const Text('จดจำอีเมล'),
              ],
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _isLoading ? null : _login,
                child: _isLoading
                    ? const CircularProgressIndicator()
                    : const Text('เข้าสู่ระบบ', style: TextStyle(fontSize: 16)),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Workshop: Settings Screen

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final settings = await AppSettings.load();
  runApp(MyApp(settings: settings));
}

class AppSettings {
  bool isDarkMode;
  String language;
  bool notifications;
  double fontSize;
  bool biometricLock;

  AppSettings({
    this.isDarkMode = false,
    this.language = 'th',
    this.notifications = true,
    this.fontSize = 14,
    this.biometricLock = false,
  });

  static Future<AppSettings> load() async {
    final prefs = await SharedPreferences.getInstance();
    return AppSettings(
      isDarkMode: prefs.getBool('dark_mode') ?? false,
      language: prefs.getString('language') ?? 'th',
      notifications: prefs.getBool('notifications') ?? true,
      fontSize: prefs.getDouble('font_size') ?? 14.0,
      biometricLock: prefs.getBool('biometric') ?? false,
    );
  }

  Future<void> save() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('dark_mode', isDarkMode);
    await prefs.setString('language', language);
    await prefs.setBool('notifications', notifications);
    await prefs.setDouble('font_size', fontSize);
    await prefs.setBool('biometric', biometricLock);
  }
}

class MyApp extends StatefulWidget {
  final AppSettings settings;
  const MyApp({super.key, required this.settings});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late AppSettings _settings;

  @override
  void initState() {
    super.initState();
    _settings = widget.settings;
  }

  void _updateSettings(AppSettings settings) {
    setState(() => _settings = settings);
    settings.save();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Settings Demo',
      themeMode: _settings.isDarkMode ? ThemeMode.dark : ThemeMode.light,
      theme: ThemeData.light(useMaterial3: true),
      darkTheme: ThemeData.dark(useMaterial3: true),
      home: SettingsScreen(
        settings: _settings,
        onSettingsChanged: _updateSettings,
      ),
    );
  }
}

class SettingsScreen extends StatefulWidget {
  final AppSettings settings;
  final ValueChanged<AppSettings> onSettingsChanged;

  const SettingsScreen({super.key, required this.settings, required this.onSettingsChanged});

  @override
  State<SettingsScreen> createState() => _SettingsScreenState();
}

class _SettingsScreenState extends State<SettingsScreen> {
  late AppSettings _local;

  @override
  void initState() {
    super.initState();
    _local = widget.settings;
  }

  void _update(AppSettings updated) {
    setState(() => _local = updated);
    widget.onSettingsChanged(updated);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('การตั้งค่า')),
      body: ListView(
        children: [
          // ===== การแสดงผล =====
          const _SectionHeader('การแสดงผล'),
          SwitchListTile(
            title: const Text('โหมดมืด'),
            subtitle: const Text('เปลี่ยนธีมเป็นสีเข้ม'),
            secondary: const Icon(Icons.dark_mode),
            value: _local.isDarkMode,
            onChanged: (v) => _update(AppSettings(
              isDarkMode: v,
              language: _local.language,
              notifications: _local.notifications,
              fontSize: _local.fontSize,
              biometricLock: _local.biometricLock,
            )),
          ),
          ListTile(
            leading: const Icon(Icons.format_size),
            title: const Text('ขนาดตัวอักษร'),
            subtitle: Slider(
              value: _local.fontSize,
              min: 10,
              max: 24,
              divisions: 7,
              label: '${_local.fontSize.toInt()} pt',
              onChanged: (v) => _update(AppSettings(
                isDarkMode: _local.isDarkMode,
                language: _local.language,
                notifications: _local.notifications,
                fontSize: v,
                biometricLock: _local.biometricLock,
              )),
            ),
          ),

          // ===== ภาษา =====
          const _SectionHeader('ภาษา'),
          ListTile(
            leading: const Icon(Icons.language),
            title: const Text('ภาษา'),
            trailing: DropdownButton<String>(
              value: _local.language,
              underline: const SizedBox(),
              items: const [
                DropdownMenuItem(value: 'th', child: Text('ไทย')),
                DropdownMenuItem(value: 'en', child: Text('English')),
                DropdownMenuItem(value: 'ja', child: Text('日本語')),
              ],
              onChanged: (v) => _update(AppSettings(
                isDarkMode: _local.isDarkMode,
                language: v ?? 'th',
                notifications: _local.notifications,
                fontSize: _local.fontSize,
                biometricLock: _local.biometricLock,
              )),
            ),
          ),

          // ===== การแจ้งเตือน =====
          const _SectionHeader('การแจ้งเตือน'),
          SwitchListTile(
            title: const Text('รับการแจ้งเตือน'),
            secondary: const Icon(Icons.notifications),
            value: _local.notifications,
            onChanged: (v) => _update(AppSettings(
              isDarkMode: _local.isDarkMode,
              language: _local.language,
              notifications: v,
              fontSize: _local.fontSize,
              biometricLock: _local.biometricLock,
            )),
          ),

          // ===== ความปลอดภัย =====
          const _SectionHeader('ความปลอดภัย'),
          SwitchListTile(
            title: const Text('ล็อกด้วย Biometric'),
            subtitle: const Text('ใช้ลายนิ้วมือหรือ Face ID'),
            secondary: const Icon(Icons.fingerprint),
            value: _local.biometricLock,
            onChanged: (v) => _update(AppSettings(
              isDarkMode: _local.isDarkMode,
              language: _local.language,
              notifications: _local.notifications,
              fontSize: _local.fontSize,
              biometricLock: v,
            )),
          ),

          // ===== ข้อมูล =====
          const _SectionHeader('ข้อมูล'),
          ListTile(
            leading: const Icon(Icons.info_outline),
            title: const Text('เวอร์ชัน'),
            trailing: const Text('1.0.0'),
          ),
          ListTile(
            leading: const Icon(Icons.delete_outline, color: Colors.red),
            title: const Text('ล้างข้อมูลแคช'),
            onTap: () async {
              final confirm = await showDialog<bool>(
                context: context,
                builder: (_) => AlertDialog(
                  title: const Text('ล้างแคช'),
                  content: const Text('คุณต้องการล้างข้อมูลแคชทั้งหมดใช่ไหม?'),
                  actions: [
                    TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('ยกเลิก')),
                    TextButton(onPressed: () => Navigator.pop(context, true), child: const Text('ล้าง', style: TextStyle(color: Colors.red))),
                  ],
                ),
              );
              if (confirm == true && context.mounted) {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text('ล้างแคชเรียบร้อย')),
                );
              }
            },
          ),
        ],
      ),
    );
  }
}

class _SectionHeader extends StatelessWidget {
  final String title;
  const _SectionHeader(this.title);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
      child: Text(
        title,
        style: TextStyle(
          color: Theme.of(context).colorScheme.primary,
          fontWeight: FontWeight.bold,
          fontSize: 13,
        ),
      ),
    );
  }
}
```

---

## สรุป Part 20

✅ SharedPreferences เก็บข้อมูล key-value อย่างง่าย
✅ อ่าน/เขียน String, int, double, bool, List<String>
✅ เก็บ Object ด้วย JSON encoding
✅ Settings pattern ด้วย ChangeNotifier
✅ flutter_secure_storage สำหรับข้อมูลสำคัญ
✅ Keychain (iOS) / Keystore (Android) encryption
✅ Token management อย่างปลอดภัย
✅ First-run detection
✅ User preferences management
✅ Storage abstraction layer สำหรับ testability
✅ สร้าง Settings Screen ครบถ้วน

## แบบฝึกหัด

1. สร้าง "Recently Viewed" feature ที่จำ 10 items ล่าสุด
2. เพิ่มระบบ PIN lock ที่เก็บ PIN ใน Secure Storage
3. สร้าง export/import settings เป็น JSON file
4. เพิ่มระบบ auto-logout เมื่อ token หมดอายุ

---

**ก่อนหน้า:** [Part 19 - SQLite & Local Database](part_19.md)
**ต่อไป:** [Part 21 - Bloc Pattern →](part_21.md)

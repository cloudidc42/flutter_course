# Part 45: Security Best Practices
## ขั้นตอนที่ 441-450

---

## สารบัญ
1. [Certificate Pinning](#ขั้นตอนที่-441-certificate-pinning)
2. [Secure Storage](#ขั้นตอนที่-442-secure-storage)
3. [Jailbreak/Root Detection](#ขั้นตอนที่-443-jailbreak-root-detection)
4. [Code Obfuscation](#ขั้นตอนที่-444-code-obfuscation)
5. [API Key Protection](#ขั้นตอนที่-445-api-key-protection)
6. [OWASP Mobile Top 10](#ขั้นตอนที่-446-owasp-mobile-top-10)
7. [Biometric Authentication](#ขั้นตอนที่-447-biometric-auth)
8. [Session Management](#ขั้นตอนที่-448-session-management)
9. [Input Sanitization](#ขั้นตอนที่-449-input-sanitization)
10. [Workshop: Secure Login Flow](#ขั้นตอนที่-450-workshop)

---

## ขั้นตอนที่ 441: Certificate Pinning

### Certificate Pinning คืออะไร?

Certificate Pinning คือการกำหนดให้ app ยอมรับ SSL certificate เฉพาะที่เราเชื่อถือเท่านั้น ป้องกัน Man-in-the-Middle attacks

```
ปกติ (No Pinning):
App ──── HTTPS ──── Any Certificate ──── Server
       ↑ อาจถูก intercept โดย proxy

With Pinning:
App ──── HTTPS ──── Check certificate hash ──── Match? ──── Server
                                        └──── No Match ──── Reject!
```

### ติดตั้ง Dependencies

```yaml
dependencies:
  http: ^1.1.0
  dio: ^5.4.0
```

### Certificate Pinning ด้วย Dio

```dart
// lib/security/certificate_pinning.dart
import 'dart:io';
import 'package:dio/dio.dart';
import 'package:flutter/services.dart';
import 'dart:convert';
import 'package:crypto/crypto.dart';

class CertificatePinningInterceptor extends Interceptor {
  final List<String> allowedSHA256Fingerprints;
  
  CertificatePinningInterceptor({required this.allowedSHA256Fingerprints});
  
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    handler.next(options);
  }
}

// การตั้งค่า Dio พร้อม Certificate Pinning
class SecureHttpClient {
  static Dio create({
    required String baseUrl,
    required List<String> allowedFingerprints,
  }) {
    final dio = Dio(BaseOptions(baseUrl: baseUrl));
    
    // Custom HTTP Client พร้อม certificate validation
    (dio.httpClientAdapter as IOHttpClientAdapter).createHttpClient = () {
      final client = HttpClient();
      
      client.badCertificateCallback = (cert, host, port) {
        // ตรวจสอบ fingerprint
        final fingerprint = _getCertificateFingerprint(cert);
        final isAllowed = allowedFingerprints.contains(fingerprint);
        
        if (!isAllowed) {
          print('Certificate NOT trusted! Host: $host, Fingerprint: $fingerprint');
        }
        
        return !isAllowed; // true = reject (bad cert), false = accept
      };
      
      return client;
    };
    
    return dio;
  }
  
  static String _getCertificateFingerprint(X509Certificate cert) {
    // Get SHA-256 fingerprint
    final bytes = cert.der;
    final digest = sha256.convert(bytes);
    return digest.toString().toUpperCase();
  }
}

// ใช้ Package flutter_ssl_pinning
// หรือ dio_http2_adapter สำหรับ HTTP/2 พร้อม pinning
```

### Certificate Pinning ด้วย flutter_pinning_check

```yaml
dependencies:
  http_certificate_pinning: ^4.0.0
```

```dart
// lib/security/pinning_check.dart
import 'package:http_certificate_pinning/http_certificate_pinning.dart';

class SecurityService {
  static Future<bool> checkCertificatePinning(String url) async {
    try {
      final secure = await HttpCertificatePinning.check(
        serverURL: url,
        headerHttp: {},
        // SHA-256 fingerprints
        sha256fp: [
          'AB:CD:EF:12:34:56:78:9A:BC:DE:F0:12:34:56:78:9A:BC:DE:F0:12:34:56:78:9A:BC:DE:F0:12:34:56:78:9A',
        ],
        allowedSHAFingerprints: [],
        timeout: 50,
      );
      
      return secure == 'CONNECTION_SECURE';
    } catch (e) {
      print('Certificate pinning failed: $e');
      return false;
    }
  }
  
  static Future<void> verifyBeforeRequest(String url) async {
    final isSecure = await checkCertificatePinning(url);
    if (!isSecure) {
      throw SecurityException('Certificate verification failed for $url');
    }
  }
}

class SecurityException implements Exception {
  final String message;
  SecurityException(this.message);
  
  @override
  String toString() => 'SecurityException: $message';
}
```

### Extracting Certificate Fingerprint

```bash
# ดึง SHA-256 fingerprint จาก server certificate
openssl s_client -connect api.example.com:443 -servername api.example.com \
  < /dev/null 2>/dev/null | openssl x509 -fingerprint -sha256 -noout

# หรือใช้ keytool
keytool -printcert -file certificate.crt
```

---

## ขั้นตอนที่ 442: Secure Storage

### flutter_secure_storage

```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
// lib/security/secure_storage_service.dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorageService {
  static const FlutterSecureStorage _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
      keyCipherAlgorithm: KeyCipherAlgorithm.RSA_ECB_OAEPwithSHA_256andMGF1Padding,
      storageCipherAlgorithm: StorageCipherAlgorithm.AES_GCM_NoPadding,
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock_this_device,
      synchronizable: false,
    ),
  );
  
  // Keys
  static const String _authTokenKey = 'auth_token';
  static const String _refreshTokenKey = 'refresh_token';
  static const String _userIdKey = 'user_id';
  static const String _pinKey = 'app_pin';
  static const String _biometricEnabledKey = 'biometric_enabled';
  
  // Token management
  static Future<void> saveAuthToken(String token) async {
    await _storage.write(key: _authTokenKey, value: token);
  }
  
  static Future<String?> getAuthToken() async {
    return _storage.read(key: _authTokenKey);
  }
  
  static Future<void> saveRefreshToken(String token) async {
    await _storage.write(key: _refreshTokenKey, value: token);
  }
  
  static Future<String?> getRefreshToken() async {
    return _storage.read(key: _refreshTokenKey);
  }
  
  // Session management
  static Future<void> saveSession({
    required String userId,
    required String authToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _userIdKey, value: userId),
      _storage.write(key: _authTokenKey, value: authToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }
  
  static Future<void> clearSession() async {
    await Future.wait([
      _storage.delete(key: _userIdKey),
      _storage.delete(key: _authTokenKey),
      _storage.delete(key: _refreshTokenKey),
    ]);
  }
  
  // PIN management
  static Future<void> savePin(String pin) async {
    // Hash PIN before storing
    final hashedPin = _hashPin(pin);
    await _storage.write(key: _pinKey, value: hashedPin);
  }
  
  static Future<bool> verifyPin(String pin) async {
    final savedHash = await _storage.read(key: _pinKey);
    if (savedHash == null) return false;
    return _hashPin(pin) == savedHash;
  }
  
  static String _hashPin(String pin) {
    final bytes = utf8.encode(pin + 'your-salt-here');
    return sha256.convert(bytes).toString();
  }
  
  // Biometric settings
  static Future<void> setBiometricEnabled(bool enabled) async {
    await _storage.write(
      key: _biometricEnabledKey,
      value: enabled.toString(),
    );
  }
  
  static Future<bool> isBiometricEnabled() async {
    final value = await _storage.read(key: _biometricEnabledKey);
    return value == 'true';
  }
  
  // Check if all keys are stored securely
  static Future<Map<String, bool>> securityAudit() async {
    final allKeys = await _storage.readAll();
    return {
      'hasAuthToken': allKeys.containsKey(_authTokenKey),
      'hasRefreshToken': allKeys.containsKey(_refreshTokenKey),
      'hasUserId': allKeys.containsKey(_userIdKey),
    };
  }
}
```

---

## ขั้นตอนที่ 443: Jailbreak/Root Detection

### flutter_jailbreak_detection

```yaml
dependencies:
  flutter_jailbreak_detection: ^1.8.0
```

```dart
// lib/security/device_security.dart
import 'package:flutter_jailbreak_detection/flutter_jailbreak_detection.dart';

class DeviceSecurityService {
  static Future<DeviceSecurityStatus> checkDeviceSecurity() async {
    try {
      final isJailbroken = await FlutterJailbreakDetection.jailbroken;
      final isOnExternalStorage = Platform.isAndroid
          ? await FlutterJailbreakDetection.developerMode
          : false;
      
      return DeviceSecurityStatus(
        isJailbroken: isJailbroken,
        isDeveloperModeEnabled: isOnExternalStorage,
        isSecure: !isJailbroken,
      );
    } catch (e) {
      // ถ้า detect ไม่ได้ ถือว่า secure
      return const DeviceSecurityStatus(
        isJailbroken: false,
        isDeveloperModeEnabled: false,
        isSecure: true,
      );
    }
  }
  
  static Future<bool> isDeviceTrusted() async {
    final status = await checkDeviceSecurity();
    return status.isSecure;
  }
}

class DeviceSecurityStatus {
  final bool isJailbroken;
  final bool isDeveloperModeEnabled;
  final bool isSecure;
  
  const DeviceSecurityStatus({
    required this.isJailbroken,
    required this.isDeveloperModeEnabled,
    required this.isSecure,
  });
}

// Security Gate Widget
class SecurityGate extends StatefulWidget {
  final Widget child;
  final Widget? blockedWidget;
  
  const SecurityGate({
    super.key,
    required this.child,
    this.blockedWidget,
  });
  
  @override
  State<SecurityGate> createState() => _SecurityGateState();
}

class _SecurityGateState extends State<SecurityGate> {
  late Future<bool> _securityCheck;
  
  @override
  void initState() {
    super.initState();
    _securityCheck = DeviceSecurityService.isDeviceTrusted();
  }
  
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<bool>(
      future: _securityCheck,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        
        final isTrusted = snapshot.data ?? true;
        
        if (!isTrusted) {
          return widget.blockedWidget ?? _buildBlockedScreen();
        }
        
        return widget.child;
      },
    );
  }
  
  Widget _buildBlockedScreen() {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.security, size: 80, color: Colors.red),
              const SizedBox(height: 24),
              const Text(
                'ไม่สามารถใช้งานได้',
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 16),
              const Text(
                'อุปกรณ์ของคุณถูก Jailbreak/Root ซึ่งเป็นความเสี่ยงด้านความปลอดภัย แอปนี้ไม่สามารถทำงานบนอุปกรณ์ที่ถูก Jailbreak ได้',
                textAlign: TextAlign.center,
                style: TextStyle(color: Colors.grey),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 444: Code Obfuscation

### การ Build ด้วย Obfuscation

```bash
# Build พร้อม obfuscation สำหรับ Android
flutter build apk --obfuscate --split-debug-info=debug_info/

# Build สำหรับ iOS
flutter build ios --obfuscate --split-debug-info=debug_info/

# Build App Bundle พร้อม obfuscation
flutter build appbundle \
  --obfuscate \
  --split-debug-info=debug_info/ \
  --target-platform android-arm,android-arm64,android-x64
```

### ProGuard Rules สำหรับ Android

```proguard
# android/app/proguard-rules.pro

# Flutter
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.** { *; }
-keep class io.flutter.util.** { *; }
-keep class io.flutter.view.** { *; }
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

# Dart/Flutter
-keep class com.example.myapp.** { *; }

# Firebase
-keep class com.google.firebase.** { *; }
-keep class com.google.android.gms.** { *; }

# Gson/JSON serialization
-keepattributes Signature
-keepattributes *Annotation*
-dontwarn sun.misc.**
-keep class com.google.gson.** { *; }
-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer
```

### Obfuscating Sensitive Strings

```dart
// lib/security/string_obfuscation.dart
class ObfuscatedString {
  final List<int> _encoded;
  final int _key;
  
  const ObfuscatedString._(this._encoded, this._key);
  
  // สร้าง obfuscated string ใน compile time
  factory ObfuscatedString.encode(String value) {
    final key = Random().nextInt(255);
    final encoded = value.codeUnits.map((c) => c ^ key).toList();
    return ObfuscatedString._(encoded, key);
  }
  
  String decode() {
    return String.fromCharCodes(
      _encoded.map((c) => c ^ _key),
    );
  }
}

// API Endpoint Obfuscation
class ApiEndpoints {
  // อย่าใส่ string ตรงๆ ใน code
  // แทนที่จะใช้:
  // static const String baseUrl = 'https://api.example.com';
  
  // ใช้:
  static String get baseUrl {
    // ดึงจาก environment variable หรือ config ที่ encrypt
    const encoded = [72, 84, 84, 80, 83]; // encoded
    return String.fromCharCodes(encoded); // HTTPS...
  }
}
```

---

## ขั้นตอนที่ 445: API Key Protection

### วิธีปกป้อง API Keys

```dart
// 1. ใช้ Environment Variables
// อย่า hardcode API keys ใน source code!

// ผิด:
// const apiKey = 'AIzaSyABC123...';

// ถูก - ใช้ flutter_dotenv:
// pubspec.yaml
// dependencies:
//   flutter_dotenv: ^5.1.0

// .env (อย่า commit ไปใน git!)
// API_KEY=AIzaSyABC123...

// การใช้งาน:
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  await dotenv.load(fileName: '.env');
  runApp(const MyApp());
}

class ApiConfig {
  static String get apiKey => dotenv.env['API_KEY'] ?? '';
  static String get baseUrl => dotenv.env['BASE_URL'] ?? '';
}
```

```dart
// 2. ใช้ Backend Proxy - แนะนำที่สุด
// แทนที่จะเรียก API จาก app โดยตรง
// ให้เรียกผ่าน Backend ของเราเอง

// แทนที่จะ:
// Client App ──── API Key ──── Third-party API

// ให้ทำ:
// Client App ──── Auth Token ──── Your Backend ──── API Key ──── Third-party API

class BackendProxyService {
  final Dio _dio;
  
  BackendProxyService(this._dio);
  
  Future<Map<String, dynamic>> callThirdPartyViaProxy(
    String endpoint,
    Map<String, dynamic> params,
  ) async {
    // ไม่ต้องส่ง API key - backend จัดการเอง
    final response = await _dio.post(
      '/proxy/$endpoint',
      data: params,
    );
    return response.data;
  }
}
```

```dart
// 3. ใช้ Flutter Flavors สำหรับ multi-environment
// lib/config/app_config.dart
class AppConfig {
  final String apiBaseUrl;
  final String apiKey;
  final bool enableLogging;
  final Environment environment;
  
  const AppConfig({
    required this.apiBaseUrl,
    required this.apiKey,
    required this.enableLogging,
    required this.environment,
  });
  
  static AppConfig? _instance;
  
  static AppConfig get instance {
    assert(_instance != null, 'AppConfig not initialized');
    return _instance!;
  }
  
  static void initialize(AppConfig config) {
    _instance = config;
  }
}

enum Environment { development, staging, production }

// main_dev.dart
void main() {
  AppConfig.initialize(const AppConfig(
    apiBaseUrl: 'https://dev-api.example.com',
    apiKey: String.fromEnvironment('DEV_API_KEY'),
    enableLogging: true,
    environment: Environment.development,
  ));
  runApp(const MyApp());
}

// main_prod.dart
void main() {
  AppConfig.initialize(const AppConfig(
    apiBaseUrl: 'https://api.example.com',
    apiKey: String.fromEnvironment('PROD_API_KEY'),
    enableLogging: false,
    environment: Environment.production,
  ));
  runApp(const MyApp());
}
```

```bash
# Build พร้อม --dart-define
flutter build apk --dart-define=PROD_API_KEY=your_actual_key
```

---

## ขั้นตอนที่ 446: OWASP Mobile Top 10

### OWASP Mobile Top 10 สำหรับ Flutter

```
M1: Improper Credential Usage
M2: Inadequate Supply Chain Security
M3: Insecure Authentication/Authorization
M4: Insufficient Input/Output Validation
M5: Insecure Communication
M6: Inadequate Privacy Controls
M7: Insufficient Binary Protections
M8: Security Misconfiguration
M9: Insecure Data Storage
M10: Insufficient Cryptography
```

### M1: Credential Management

```dart
// ❌ ผิด - เก็บ credentials ใน SharedPreferences
final prefs = await SharedPreferences.getInstance();
prefs.setString('password', password); // ไม่ปลอดภัย!

// ✅ ถูก - ใช้ Secure Storage
await SecureStorageService.saveAuthToken(token);
// Never store raw passwords
```

### M3: Authentication & Authorization

```dart
// lib/security/auth_middleware.dart
class AuthMiddleware {
  final SecureStorageService _storage;
  final AuthService _authService;
  
  AuthMiddleware(this._storage, this._authService);
  
  Future<bool> validateSession() async {
    final token = await _storage.getAuthToken();
    if (token == null) return false;
    
    // ตรวจสอบ token expiry
    try {
      final payload = _decodeJWT(token);
      final exp = payload['exp'] as int;
      final expiryTime = DateTime.fromMillisecondsSinceEpoch(exp * 1000);
      
      if (DateTime.now().isAfter(expiryTime)) {
        // Token expired - try refresh
        return await _refreshToken();
      }
      
      return true;
    } catch (e) {
      return false;
    }
  }
  
  Future<bool> _refreshToken() async {
    final refreshToken = await _storage.getRefreshToken();
    if (refreshToken == null) return false;
    
    try {
      final newTokens = await _authService.refreshTokens(refreshToken);
      await _storage.saveSession(
        userId: newTokens.userId,
        authToken: newTokens.accessToken,
        refreshToken: newTokens.refreshToken,
      );
      return true;
    } catch (e) {
      await _storage.clearSession();
      return false;
    }
  }
  
  Map<String, dynamic> _decodeJWT(String token) {
    final parts = token.split('.');
    if (parts.length != 3) throw const FormatException('Invalid JWT');
    
    final payload = parts[1];
    final normalized = base64.normalize(payload);
    final decoded = utf8.decode(base64.decode(normalized));
    return jsonDecode(decoded) as Map<String, dynamic>;
  }
}
```

### M5: Insecure Communication

```dart
// lib/network/secure_http_client.dart
class SecureHttpClient {
  static Dio create() {
    final dio = Dio();
    
    // Force HTTPS only
    dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) {
          if (!options.baseUrl.startsWith('https://')) {
            handler.reject(
              DioException(
                requestOptions: options,
                message: 'Only HTTPS connections are allowed',
              ),
            );
            return;
          }
          handler.next(options);
        },
      ),
    );
    
    return dio;
  }
}
```

### M9: Insecure Data Storage

```dart
// lib/security/data_encryption.dart
import 'package:encrypt/encrypt.dart';

class DataEncryptionService {
  late final Key _key;
  late final IV _iv;
  late final Encrypter _encrypter;
  
  Future<void> initialize() async {
    // ดึง encryption key จาก secure storage
    final keyString = await _getOrCreateKey();
    _key = Key.fromBase64(keyString);
    _iv = IV.fromLength(16);
    _encrypter = Encrypter(AES(_key, mode: AESMode.gcm));
  }
  
  String encrypt(String data) {
    final encrypted = _encrypter.encrypt(data, iv: _iv);
    return encrypted.base64;
  }
  
  String decrypt(String encryptedData) {
    final encrypted = Encrypted.fromBase64(encryptedData);
    return _encrypter.decrypt(encrypted, iv: _iv);
  }
  
  Future<String> _getOrCreateKey() async {
    const storage = FlutterSecureStorage();
    var key = await storage.read(key: 'encryption_key');
    
    if (key == null) {
      // Generate new key
      key = Key.fromSecureRandom(32).base64;
      await storage.write(key: 'encryption_key', value: key);
    }
    
    return key;
  }
}
```

---

## ขั้นตอนที่ 447: Biometric Authentication

### local_auth Package

```yaml
dependencies:
  local_auth: ^2.2.0
```

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.USE_BIOMETRIC"/>
<uses-permission android:name="android.permission.USE_FINGERPRINT"/>
```

```dart
// lib/security/biometric_service.dart
import 'package:local_auth/local_auth.dart';
import 'package:local_auth/error_codes.dart' as auth_error;

class BiometricService {
  final LocalAuthentication _auth = LocalAuthentication();
  
  Future<BiometricCapability> checkCapability() async {
    try {
      final canCheckBiometrics = await _auth.canCheckBiometrics;
      final isDeviceSupported = await _auth.isDeviceSupported();
      
      if (!canCheckBiometrics || !isDeviceSupported) {
        return BiometricCapability.notSupported;
      }
      
      final availableBiometrics = await _auth.getAvailableBiometrics();
      
      if (availableBiometrics.isEmpty) {
        return BiometricCapability.noEnrolled;
      }
      
      final hasFace = availableBiometrics.contains(BiometricType.face);
      final hasFingerprint = availableBiometrics.contains(BiometricType.fingerprint);
      
      return BiometricCapability.available(
        hasFace: hasFace,
        hasFingerprint: hasFingerprint,
      );
    } catch (e) {
      return BiometricCapability.notSupported;
    }
  }
  
  Future<BiometricResult> authenticate({
    String reason = 'กรุณายืนยันตัวตนเพื่อเข้าสู่ระบบ',
    bool biometricOnly = false,
  }) async {
    try {
      final authenticated = await _auth.authenticate(
        localizedReason: reason,
        options: AuthenticationOptions(
          stickyAuth: true,
          biometricOnly: biometricOnly,
          useErrorDialogs: true,
          sensitiveTransaction: true,
        ),
      );
      
      return authenticated
          ? BiometricResult.success
          : BiometricResult.cancelled;
    } on PlatformException catch (e) {
      switch (e.code) {
        case auth_error.notAvailable:
          return BiometricResult.notAvailable;
        case auth_error.notEnrolled:
          return BiometricResult.notEnrolled;
        case auth_error.lockedOut:
          return BiometricResult.lockedOut;
        case auth_error.permanentlyLockedOut:
          return BiometricResult.permanentlyLockedOut;
        default:
          return BiometricResult.error;
      }
    }
  }
  
  Future<void> stopAuthentication() async {
    await _auth.stopAuthentication();
  }
}

enum BiometricResult {
  success,
  cancelled,
  notAvailable,
  notEnrolled,
  lockedOut,
  permanentlyLockedOut,
  error,
}

class BiometricCapability {
  final bool isSupported;
  final bool hasFace;
  final bool hasFingerprint;
  final bool isEnrolled;
  
  const BiometricCapability._({
    required this.isSupported,
    required this.hasFace,
    required this.hasFingerprint,
    required this.isEnrolled,
  });
  
  static const notSupported = BiometricCapability._(
    isSupported: false,
    hasFace: false,
    hasFingerprint: false,
    isEnrolled: false,
  );
  
  static const noEnrolled = BiometricCapability._(
    isSupported: true,
    hasFace: false,
    hasFingerprint: false,
    isEnrolled: false,
  );
  
  factory BiometricCapability.available({
    required bool hasFace,
    required bool hasFingerprint,
  }) {
    return BiometricCapability._(
      isSupported: true,
      hasFace: hasFace,
      hasFingerprint: hasFingerprint,
      isEnrolled: true,
    );
  }
}
```

### Biometric Login Screen

```dart
// lib/features/auth/screens/biometric_login_screen.dart
class BiometricLoginScreen extends ConsumerStatefulWidget {
  const BiometricLoginScreen({super.key});
  
  @override
  ConsumerState<BiometricLoginScreen> createState() =>
      _BiometricLoginScreenState();
}

class _BiometricLoginScreenState extends ConsumerState<BiometricLoginScreen> {
  final BiometricService _biometricService = BiometricService();
  BiometricCapability? _capability;
  
  @override
  void initState() {
    super.initState();
    _checkCapability();
  }
  
  Future<void> _checkCapability() async {
    final capability = await _biometricService.checkCapability();
    setState(() => _capability = capability);
    
    // Auto-trigger biometric if available and enabled
    if (capability.isEnrolled) {
      final isEnabled = await SecureStorageService.isBiometricEnabled();
      if (isEnabled) {
        _authenticateWithBiometric();
      }
    }
  }
  
  Future<void> _authenticateWithBiometric() async {
    final result = await _biometricService.authenticate(
      reason: 'ยืนยันตัวตนด้วยไบโอเมตริก',
    );
    
    switch (result) {
      case BiometricResult.success:
        await _loginWithSavedCredentials();
      case BiometricResult.cancelled:
        // User cancelled, show password option
        break;
      case BiometricResult.lockedOut:
        _showError('การยืนยันตัวตนถูกล็อก กรุณารอสักครู่');
      case BiometricResult.permanentlyLockedOut:
        _showError('การยืนยันตัวตนถูกล็อกถาวร กรุณาใช้ PIN หรือรหัสผ่าน');
      default:
        _showError('ไม่สามารถยืนยันตัวตนได้');
    }
  }
  
  Future<void> _loginWithSavedCredentials() async {
    final token = await SecureStorageService.getAuthToken();
    if (token != null) {
      ref.read(authStateProvider.notifier).setToken(token);
      if (mounted) context.go('/home');
    }
  }
  
  void _showError(String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message), backgroundColor: Colors.red),
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const FlutterLogo(size: 80),
              const SizedBox(height: 32),
              if (_capability?.isEnrolled ?? false) ...[
                GestureDetector(
                  onTap: _authenticateWithBiometric,
                  child: Column(
                    children: [
                      Icon(
                        _capability!.hasFace
                            ? Icons.face
                            : Icons.fingerprint,
                        size: 64,
                        color: Theme.of(context).colorScheme.primary,
                      ),
                      const SizedBox(height: 8),
                      Text(
                        _capability!.hasFace
                            ? 'ยืนยันด้วย Face ID'
                            : 'ยืนยันด้วยลายนิ้วมือ',
                        style: Theme.of(context).textTheme.titleMedium,
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 24),
              ],
              TextButton(
                onPressed: () => context.push('/login'),
                child: const Text('เข้าสู่ระบบด้วยรหัสผ่าน'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 448: Session Management

### Secure Session Management

```dart
// lib/security/session_manager.dart
class SessionManager {
  static const Duration _sessionTimeout = Duration(minutes: 30);
  static const Duration _tokenRefreshThreshold = Duration(minutes: 5);
  
  final SecureStorageService _storage;
  final AuthService _authService;
  Timer? _sessionTimer;
  Timer? _inactivityTimer;
  DateTime? _lastActivity;
  
  SessionManager(this._storage, this._authService);
  
  // เริ่ม session
  Future<void> startSession({
    required String userId,
    required String token,
    required String refreshToken,
    required DateTime expiresAt,
  }) async {
    await _storage.saveSession(
      userId: userId,
      authToken: token,
      refreshToken: refreshToken,
    );
    
    _lastActivity = DateTime.now();
    _scheduleTokenRefresh(expiresAt);
    _startInactivityTimer();
  }
  
  // บันทึก activity
  void recordActivity() {
    _lastActivity = DateTime.now();
    _resetInactivityTimer();
  }
  
  // ตรวจสอบ session
  Future<bool> isSessionValid() async {
    final token = await _storage.getAuthToken();
    if (token == null) return false;
    
    // ตรวจสอบ inactivity timeout
    if (_lastActivity != null) {
      final inactiveDuration = DateTime.now().difference(_lastActivity!);
      if (inactiveDuration > _sessionTimeout) {
        await endSession(reason: SessionEndReason.timeout);
        return false;
      }
    }
    
    return true;
  }
  
  // จบ session
  Future<void> endSession({SessionEndReason reason = SessionEndReason.logout}) async {
    _sessionTimer?.cancel();
    _inactivityTimer?.cancel();
    
    await _storage.clearSession();
    
    print('Session ended: ${reason.name}');
  }
  
  void _scheduleTokenRefresh(DateTime expiresAt) {
    final refreshTime = expiresAt.subtract(_tokenRefreshThreshold);
    final delay = refreshTime.difference(DateTime.now());
    
    if (delay.isNegative) {
      _refreshToken();
      return;
    }
    
    _sessionTimer = Timer(delay, _refreshToken);
  }
  
  Future<void> _refreshToken() async {
    try {
      final refreshToken = await _storage.getRefreshToken();
      if (refreshToken == null) {
        await endSession(reason: SessionEndReason.tokenExpired);
        return;
      }
      
      final newTokens = await _authService.refreshTokens(refreshToken);
      
      await _storage.saveSession(
        userId: newTokens.userId,
        authToken: newTokens.accessToken,
        refreshToken: newTokens.refreshToken,
      );
      
      _scheduleTokenRefresh(newTokens.expiresAt);
    } catch (e) {
      await endSession(reason: SessionEndReason.tokenExpired);
    }
  }
  
  void _startInactivityTimer() {
    _inactivityTimer = Timer.periodic(
      const Duration(minutes: 1),
      (_) async {
        await isSessionValid();
      },
    );
  }
  
  void _resetInactivityTimer() {
    _inactivityTimer?.cancel();
    _startInactivityTimer();
  }
}

enum SessionEndReason {
  logout,
  timeout,
  tokenExpired,
  securityViolation,
}
```

---

## ขั้นตอนที่ 449: Input Sanitization

### Input Validation และ Sanitization

```dart
// lib/security/input_sanitizer.dart
class InputSanitizer {
  // ลบ HTML tags
  static String sanitizeHtml(String input) {
    return input.replaceAll(RegExp(r'<[^>]*>'), '');
  }
  
  // ลบ SQL injection attempts
  static String sanitizeSql(String input) {
    return input.replaceAll(
      RegExp(r"[';\"\\--]|(\b(SELECT|INSERT|UPDATE|DELETE|DROP|UNION|EXEC)\b)",
          caseSensitive: false),
      '',
    );
  }
  
  // Escape special characters
  static String escapeSpecialChars(String input) {
    return input
        .replaceAll('&', '&amp;')
        .replaceAll('<', '&lt;')
        .replaceAll('>', '&gt;')
        .replaceAll('"', '&quot;')
        .replaceAll("'", '&#x27;')
        .replaceAll('/', '&#x2F;');
  }
  
  // Validate email
  static bool isValidEmail(String email) {
    return RegExp(
      r'^[a-zA-Z0-9.!#$%&*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$',
    ).hasMatch(email);
  }
  
  // Validate phone (Thai)
  static bool isValidThaiPhone(String phone) {
    return RegExp(r'^(0[6-9][0-9]{8})$').hasMatch(phone.replaceAll('-', ''));
  }
  
  // Validate URL
  static bool isValidUrl(String url) {
    try {
      final uri = Uri.parse(url);
      return uri.hasScheme && (uri.scheme == 'https' || uri.scheme == 'http');
    } catch (_) {
      return false;
    }
  }
  
  // Truncate to max length
  static String truncate(String input, int maxLength) {
    if (input.length <= maxLength) return input;
    return input.substring(0, maxLength);
  }
  
  // Validate password strength
  static PasswordStrength checkPasswordStrength(String password) {
    int score = 0;
    
    if (password.length >= 8) score++;
    if (password.length >= 12) score++;
    if (password.contains(RegExp(r'[A-Z]'))) score++;
    if (password.contains(RegExp(r'[a-z]'))) score++;
    if (password.contains(RegExp(r'[0-9]'))) score++;
    if (password.contains(RegExp(r'[!@#$%^&*(),.?":{}|<>]'))) score++;
    
    return switch (score) {
      <= 2 => PasswordStrength.weak,
      <= 4 => PasswordStrength.medium,
      _ => PasswordStrength.strong,
    };
  }
}

enum PasswordStrength { weak, medium, strong }

// Custom Form Validators
class AppValidators {
  static String? required(String? value, [String? fieldName]) {
    if (value == null || value.trim().isEmpty) {
      return 'กรุณากรอก${fieldName ?? 'ข้อมูล'}';
    }
    return null;
  }
  
  static String? email(String? value) {
    if (value == null || value.isEmpty) return 'กรุณากรอกอีเมล';
    if (!InputSanitizer.isValidEmail(value)) return 'รูปแบบอีเมลไม่ถูกต้อง';
    return null;
  }
  
  static String? password(String? value) {
    if (value == null || value.isEmpty) return 'กรุณากรอกรหัสผ่าน';
    if (value.length < 8) return 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
    
    final strength = InputSanitizer.checkPasswordStrength(value);
    if (strength == PasswordStrength.weak) {
      return 'รหัสผ่านไม่แข็งแรงพอ กรุณาเพิ่มตัวพิมพ์ใหญ่, ตัวเลข, หรืออักขระพิเศษ';
    }
    return null;
  }
  
  static String? minLength(int min) {
    return (String? value) {
      if (value == null || value.length < min) {
        return 'ต้องมีอย่างน้อย $min ตัวอักษร';
      }
      return null;
    };
  }
  
  static String? maxLength(int max) {
    return (String? value) {
      if (value != null && value.length > max) {
        return 'ต้องไม่เกิน $max ตัวอักษร';
      }
      return null;
    };
  }
  
  static FormFieldValidator<String> combine(
    List<FormFieldValidator<String>> validators,
  ) {
    return (value) {
      for (final validator in validators) {
        final result = validator(value);
        if (result != null) return result;
      }
      return null;
    };
  }
}
```

---

## ขั้นตอนที่ 450: Workshop - Secure Login Flow

### Complete Secure Login Implementation

```dart
// lib/features/auth/screens/secure_login_screen.dart
class SecureLoginScreen extends ConsumerStatefulWidget {
  const SecureLoginScreen({super.key});
  
  @override
  ConsumerState<SecureLoginScreen> createState() => _SecureLoginScreenState();
}

class _SecureLoginScreenState extends ConsumerState<SecureLoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  
  bool _isPasswordVisible = false;
  bool _isLoading = false;
  int _failedAttempts = 0;
  DateTime? _lockoutUntil;
  
  static const int _maxAttempts = 5;
  static const Duration _lockoutDuration = Duration(minutes: 15);
  
  @override
  Widget build(BuildContext context) {
    final isLocked = _lockoutUntil != null &&
        DateTime.now().isBefore(_lockoutUntil!);
    
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(24),
            child: Form(
              key: _formKey,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  const FlutterLogo(size: 80),
                  const SizedBox(height: 32),
                  const Text(
                    'เข้าสู่ระบบ',
                    style: TextStyle(
                      fontSize: 28,
                      fontWeight: FontWeight.bold,
                    ),
                    textAlign: TextAlign.center,
                  ),
                  const SizedBox(height: 32),
                  
                  if (isLocked)
                    _buildLockoutBanner()
                  else ...[
                    TextFormField(
                      controller: _emailController,
                      decoration: const InputDecoration(
                        labelText: 'อีเมล',
                        prefixIcon: Icon(Icons.email_outlined),
                        border: OutlineInputBorder(),
                      ),
                      keyboardType: TextInputType.emailAddress,
                      autocorrect: false,
                      validator: AppValidators.email,
                      enabled: !_isLoading,
                    ),
                    const SizedBox(height: 16),
                    TextFormField(
                      controller: _passwordController,
                      decoration: InputDecoration(
                        labelText: 'รหัสผ่าน',
                        prefixIcon: const Icon(Icons.lock_outlined),
                        suffixIcon: IconButton(
                          icon: Icon(
                            _isPasswordVisible
                                ? Icons.visibility_off
                                : Icons.visibility,
                          ),
                          onPressed: () {
                            setState(() {
                              _isPasswordVisible = !_isPasswordVisible;
                            });
                          },
                        ),
                        border: const OutlineInputBorder(),
                      ),
                      obscureText: !_isPasswordVisible,
                      validator: AppValidators.password,
                      enabled: !_isLoading,
                    ),
                    const SizedBox(height: 8),
                    if (_failedAttempts > 0)
                      Text(
                        'พยายามเข้าสู่ระบบผิดพลาด ${_failedAttempts}/${_maxAttempts} ครั้ง',
                        style: const TextStyle(color: Colors.orange),
                        textAlign: TextAlign.center,
                      ),
                    const SizedBox(height: 24),
                    
                    _isLoading
                        ? const Center(child: CircularProgressIndicator())
                        : ElevatedButton(
                            onPressed: _handleLogin,
                            style: ElevatedButton.styleFrom(
                              padding: const EdgeInsets.symmetric(vertical: 16),
                            ),
                            child: const Text(
                              'เข้าสู่ระบบ',
                              style: TextStyle(fontSize: 16),
                            ),
                          ),
                    const SizedBox(height: 16),
                    TextButton(
                      onPressed: () => context.push('/forgot-password'),
                      child: const Text('ลืมรหัสผ่าน?'),
                    ),
                  ],
                  
                  const SizedBox(height: 16),
                  const Divider(),
                  const SizedBox(height: 16),
                  
                  const BiometricLoginButton(),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
  
  Widget _buildLockoutBanner() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.red.shade50,
        borderRadius: BorderRadius.circular(8),
        border: Border.all(color: Colors.red),
      ),
      child: Column(
        children: [
          const Icon(Icons.lock, color: Colors.red, size: 32),
          const SizedBox(height: 8),
          const Text(
            'บัญชีถูกล็อกชั่วคราว',
            style: TextStyle(
              color: Colors.red,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            'กรุณารอ ${_lockoutUntil?.difference(DateTime.now()).inMinutes ?? 0} นาที',
            style: const TextStyle(color: Colors.red),
          ),
        ],
      ),
    );
  }
  
  Future<void> _handleLogin() async {
    if (!_formKey.currentState!.validate()) return;
    
    // ตรวจสอบ lockout
    if (_lockoutUntil != null && DateTime.now().isBefore(_lockoutUntil!)) {
      return;
    }
    
    setState(() => _isLoading = true);
    
    try {
      // Sanitize inputs
      final email = _emailController.text.trim().toLowerCase();
      final password = _passwordController.text;
      
      // Attempt login
      await ref.read(authServiceProvider).signIn(email, password);
      
      // Reset failed attempts on success
      _failedAttempts = 0;
      _lockoutUntil = null;
      
      if (mounted) context.go('/home');
    } catch (e) {
      _failedAttempts++;
      
      if (_failedAttempts >= _maxAttempts) {
        _lockoutUntil = DateTime.now().add(_lockoutDuration);
        setState(() {});
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(
              'เข้าสู่ระบบผิดพลาดหลายครั้ง บัญชีถูกล็อก ${_lockoutDuration.inMinutes} นาที',
            ),
            backgroundColor: Colors.red,
          ),
        );
      } else {
        setState(() {});
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('อีเมลหรือรหัสผ่านไม่ถูกต้อง'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }
  
  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **Certificate Pinning**: ป้องกัน Man-in-the-Middle attacks
- **Secure Storage**: flutter_secure_storage สำหรับเก็บข้อมูลสำคัญ
- **Root/Jailbreak Detection**: ตรวจสอบความปลอดภัยของอุปกรณ์
- **Code Obfuscation**: ซ่อน logic จาก reverse engineering
- **API Key Protection**: ปกป้อง API keys ด้วยหลายวิธี
- **OWASP Mobile Top 10**: ภัยคุกคาม top 10 ที่ต้องระวัง
- **Biometric Auth**: local_auth สำหรับ fingerprint/face ID
- **Session Management**: จัดการ session อย่างปลอดภัย
- **Input Sanitization**: ตรวจสอบและ sanitize ข้อมูล input

## แบบฝึกหัด

1. Implement certificate pinning สำหรับ production app
2. สร้าง secure storage service ที่ใช้ encryption
3. เพิ่ม biometric login flow พร้อม fallback
4. Implement session timeout ด้วย inactivity detection
5. สร้าง input validation library ที่ครบถ้วน

---

[⬅️ Part 44](part_44.md) | [Part 46 ➡️](part_46.md)

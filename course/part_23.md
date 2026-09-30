# Part 23: Firebase Auth
## ขั้นตอนที่ 221-230

---

## สารบัญ
1. [firebase_auth Package](#firebase_auth-package)
2. [Email/Password Auth](#emailpassword-auth)
3. [Google Sign-In](#google-sign-in)
4. [Sign in with Apple](#sign-in-with-apple)
5. [Phone Auth (OTP)](#phone-auth-otp)
6. [Anonymous Auth](#anonymous-auth)
7. [Auth State Changes](#auth-state-changes)
8. [User Profile Management](#user-profile-management)
9. [Password Reset](#password-reset)
10. [Email Verification](#email-verification)

---

## ขั้นตอนที่ 221: firebase_auth Package

Firebase Authentication ให้บริการ Authentication ที่ปลอดภัย รองรับหลาย Provider

```yaml
# pubspec.yaml
dependencies:
  firebase_auth: ^4.16.0
  google_sign_in: ^6.2.1
  sign_in_with_apple: ^5.0.0
```

```dart
// lib/services/auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class AuthService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  // ดึง User ปัจจุบัน
  static User? get currentUser => _auth.currentUser;

  // ตรวจสอบว่า Login อยู่หรือเปล่า
  static bool get isLoggedIn => _auth.currentUser != null;

  // Stream สำหรับ Auth State
  static Stream<User?> get authStateChanges => _auth.authStateChanges();
  static Stream<User?> get userChanges => _auth.userChanges();
  static Stream<User?> get idTokenChanges => _auth.idTokenChanges();
}
```

---

## ขั้นตอนที่ 222: Email/Password Auth

```dart
// lib/services/email_auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class EmailAuthService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  // Register
  static Future<UserCredential> register({
    required String email,
    required String password,
  }) async {
    try {
      final credential = await _auth.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );
      return credential;
    } on FirebaseAuthException catch (e) {
      throw _handleAuthException(e);
    }
  }

  // Login
  static Future<UserCredential> login({
    required String email,
    required String password,
  }) async {
    try {
      final credential = await _auth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );
      return credential;
    } on FirebaseAuthException catch (e) {
      throw _handleAuthException(e);
    }
  }

  // Logout
  static Future<void> logout() async {
    await _auth.signOut();
  }

  // Handle Exceptions
  static String _handleAuthException(FirebaseAuthException e) {
    switch (e.code) {
      case 'weak-password':
        return 'รหัสผ่านไม่ปลอดภัยเพียงพอ';
      case 'email-already-in-use':
        return 'Email นี้ถูกใช้งานแล้ว';
      case 'user-not-found':
        return 'ไม่พบบัญชีผู้ใช้นี้';
      case 'wrong-password':
        return 'รหัสผ่านไม่ถูกต้อง';
      case 'invalid-email':
        return 'รูปแบบ Email ไม่ถูกต้อง';
      case 'user-disabled':
        return 'บัญชีนี้ถูกระงับการใช้งาน';
      case 'too-many-requests':
        return 'คำขอมากเกินไป กรุณารอสักครู่';
      case 'network-request-failed':
        return 'ไม่มีการเชื่อมต่ออินเทอร์เน็ต';
      default:
        return 'เกิดข้อผิดพลาด: ${e.message}';
    }
  }
}
```

```dart
// lib/pages/login_page.dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../services/email_auth_service.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  bool _obscurePassword = true;
  bool _isLoginMode = true;

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _isLoading = true);

    try {
      if (_isLoginMode) {
        await EmailAuthService.login(
          email: _emailController.text.trim(),
          password: _passwordController.text,
        );
      } else {
        final credential = await EmailAuthService.register(
          email: _emailController.text.trim(),
          password: _passwordController.text,
        );
        // ส่ง Email Verification
        await credential.user?.sendEmailVerification();
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(
              content: Text('ส่ง Email ยืนยันแล้ว กรุณาตรวจสอบ Email'),
              backgroundColor: Colors.blue,
            ),
          );
        }
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.toString()),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) {
        setState(() => _isLoading = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
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
                  // Logo
                  const Icon(
                    Icons.lock_outline,
                    size: 80,
                    color: Colors.blue,
                  ),
                  const SizedBox(height: 24),

                  // Title
                  Text(
                    _isLoginMode ? 'เข้าสู่ระบบ' : 'สมัครสมาชิก',
                    style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                          fontWeight: FontWeight.bold,
                        ),
                    textAlign: TextAlign.center,
                  ),
                  const SizedBox(height: 32),

                  // Email Field
                  TextFormField(
                    controller: _emailController,
                    decoration: const InputDecoration(
                      labelText: 'Email',
                      prefixIcon: Icon(Icons.email_outlined),
                      border: OutlineInputBorder(),
                    ),
                    keyboardType: TextInputType.emailAddress,
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'กรุณาใส่ Email';
                      }
                      if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$')
                          .hasMatch(value)) {
                        return 'รูปแบบ Email ไม่ถูกต้อง';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 16),

                  // Password Field
                  TextFormField(
                    controller: _passwordController,
                    decoration: InputDecoration(
                      labelText: 'รหัสผ่าน',
                      prefixIcon: const Icon(Icons.lock_outlined),
                      border: const OutlineInputBorder(),
                      suffixIcon: IconButton(
                        icon: Icon(_obscurePassword
                            ? Icons.visibility
                            : Icons.visibility_off),
                        onPressed: () => setState(
                            () => _obscurePassword = !_obscurePassword),
                      ),
                    ),
                    obscureText: _obscurePassword,
                    validator: (value) {
                      if (value == null || value.isEmpty) {
                        return 'กรุณาใส่รหัสผ่าน';
                      }
                      if (!_isLoginMode && value.length < 6) {
                        return 'รหัสผ่านต้องมีอย่างน้อย 6 ตัวอักษร';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(height: 8),

                  // Forgot Password
                  if (_isLoginMode)
                    Align(
                      alignment: Alignment.centerRight,
                      child: TextButton(
                        onPressed: _showForgotPasswordDialog,
                        child: const Text('ลืมรหัสผ่าน?'),
                      ),
                    ),
                  const SizedBox(height: 16),

                  // Submit Button
                  ElevatedButton(
                    onPressed: _isLoading ? null : _submit,
                    style: ElevatedButton.styleFrom(
                      padding: const EdgeInsets.symmetric(vertical: 16),
                    ),
                    child: _isLoading
                        ? const SizedBox(
                            height: 20,
                            width: 20,
                            child: CircularProgressIndicator(strokeWidth: 2),
                          )
                        : Text(_isLoginMode ? 'เข้าสู่ระบบ' : 'สมัครสมาชิก'),
                  ),
                  const SizedBox(height: 16),

                  // Toggle Mode
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(_isLoginMode
                          ? 'ยังไม่มีบัญชี?'
                          : 'มีบัญชีแล้ว?'),
                      TextButton(
                        onPressed: () =>
                            setState(() => _isLoginMode = !_isLoginMode),
                        child: Text(
                            _isLoginMode ? 'สมัครสมาชิก' : 'เข้าสู่ระบบ'),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }

  void _showForgotPasswordDialog() {
    final emailCtrl = TextEditingController();
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('รีเซ็ตรหัสผ่าน'),
        content: TextField(
          controller: emailCtrl,
          decoration: const InputDecoration(
            labelText: 'Email',
            border: OutlineInputBorder(),
          ),
          keyboardType: TextInputType.emailAddress,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () async {
              try {
                await FirebaseAuth.instance.sendPasswordResetEmail(
                  email: emailCtrl.text.trim(),
                );
                if (mounted) {
                  Navigator.pop(ctx);
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(
                      content: Text('ส่ง Email รีเซ็ตรหัสผ่านแล้ว'),
                      backgroundColor: Colors.green,
                    ),
                  );
                }
              } catch (e) {
                if (mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Error: $e')),
                  );
                }
              }
            },
            child: const Text('ส่ง'),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 223: Google Sign-In

```dart
// lib/services/google_auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:google_sign_in/google_sign_in.dart';

class GoogleAuthService {
  static final GoogleSignIn _googleSignIn = GoogleSignIn();
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  static Future<UserCredential?> signIn() async {
    try {
      // เปิด Google Sign-In dialog
      final GoogleSignInAccount? googleUser = await _googleSignIn.signIn();

      if (googleUser == null) {
        // ผู้ใช้ยกเลิก
        return null;
      }

      // ดึง Auth Credentials
      final GoogleSignInAuthentication googleAuth =
          await googleUser.authentication;

      // สร้าง Firebase Credential
      final credential = GoogleAuthProvider.credential(
        accessToken: googleAuth.accessToken,
        idToken: googleAuth.idToken,
      );

      // Sign in ด้วย Firebase
      return await _auth.signInWithCredential(credential);
    } catch (e) {
      throw Exception('Google Sign-In ล้มเหลว: $e');
    }
  }

  static Future<void> signOut() async {
    await Future.wait([
      _googleSignIn.signOut(),
      _auth.signOut(),
    ]);
  }
}

// lib/widgets/google_sign_in_button.dart
import 'package:flutter/material.dart';
import '../services/google_auth_service.dart';

class GoogleSignInButton extends StatefulWidget {
  const GoogleSignInButton({super.key});

  @override
  State<GoogleSignInButton> createState() => _GoogleSignInButtonState();
}

class _GoogleSignInButtonState extends State<GoogleSignInButton> {
  bool _isLoading = false;

  Future<void> _signIn() async {
    setState(() => _isLoading = true);
    try {
      final credential = await GoogleAuthService.signIn();
      if (credential != null && mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('ยินดีต้อนรับ ${credential.user?.displayName}'),
            backgroundColor: Colors.green,
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('$e'), backgroundColor: Colors.red),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return OutlinedButton(
      onPressed: _isLoading ? null : _signIn,
      style: OutlinedButton.styleFrom(
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
      ),
      child: _isLoading
          ? const SizedBox(
              height: 20,
              width: 20,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                Image.asset('assets/google_logo.png', height: 24),
                const SizedBox(width: 12),
                const Text('เข้าสู่ระบบด้วย Google'),
              ],
            ),
    );
  }
}
```

---

## ขั้นตอนที่ 224: Sign in with Apple

```dart
// lib/services/apple_auth_service.dart
import 'dart:convert';
import 'dart:math';
import 'package:crypto/crypto.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:sign_in_with_apple/sign_in_with_apple.dart';

class AppleAuthService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  static Future<UserCredential?> signIn() async {
    try {
      // สร้าง nonce สำหรับ security
      final rawNonce = _generateNonce();
      final nonce = _sha256ofString(rawNonce);

      // Request credential
      final appleCredential = await SignInWithApple.getAppleIDCredential(
        scopes: [
          AppleIDAuthorizationScopes.email,
          AppleIDAuthorizationScopes.fullName,
        ],
        nonce: nonce,
      );

      // สร้าง OAuthCredential
      final oauthCredential = OAuthProvider('apple.com').credential(
        idToken: appleCredential.identityToken,
        rawNonce: rawNonce,
      );

      // Sign in ด้วย Firebase
      final result = await _auth.signInWithCredential(oauthCredential);

      // อัพเดต Display Name (Apple ส่งชื่อมาแค่ครั้งแรก)
      if (result.additionalUserInfo?.isNewUser == true) {
        final fullName = appleCredential.givenName != null
            ? '${appleCredential.givenName} ${appleCredential.familyName ?? ''}'
            : null;
        if (fullName != null) {
          await result.user?.updateDisplayName(fullName.trim());
        }
      }

      return result;
    } catch (e) {
      throw Exception('Apple Sign-In ล้มเหลว: $e');
    }
  }

  static String _generateNonce([int length = 32]) {
    const charset =
        '0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._';
    final random = Random.secure();
    return List.generate(length, (_) => charset[random.nextInt(charset.length)])
        .join();
  }

  static String _sha256ofString(String input) {
    final bytes = utf8.encode(input);
    final digest = sha256.convert(bytes);
    return digest.toString();
  }
}
```

---

## ขั้นตอนที่ 225: Phone Auth (OTP)

```dart
// lib/services/phone_auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class PhoneAuthService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  // ส่ง OTP
  static Future<void> sendOTP({
    required String phoneNumber,
    required Function(PhoneAuthCredential) onVerificationCompleted,
    required Function(FirebaseAuthException) onVerificationFailed,
    required Function(String, int?) onCodeSent,
    required Function(String) onCodeAutoRetrievalTimeout,
  }) async {
    await _auth.verifyPhoneNumber(
      phoneNumber: phoneNumber, // เช่น '+66812345678'
      timeout: const Duration(seconds: 60),
      verificationCompleted: onVerificationCompleted,
      verificationFailed: onVerificationFailed,
      codeSent: onCodeSent,
      codeAutoRetrievalTimeout: onCodeAutoRetrievalTimeout,
    );
  }

  // ยืนยัน OTP
  static Future<UserCredential> verifyOTP({
    required String verificationId,
    required String smsCode,
  }) async {
    final credential = PhoneAuthProvider.credential(
      verificationId: verificationId,
      smsCode: smsCode,
    );
    return await _auth.signInWithCredential(credential);
  }
}

// lib/pages/phone_auth_page.dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../services/phone_auth_service.dart';

class PhoneAuthPage extends StatefulWidget {
  const PhoneAuthPage({super.key});

  @override
  State<PhoneAuthPage> createState() => _PhoneAuthPageState();
}

class _PhoneAuthPageState extends State<PhoneAuthPage> {
  final _phoneController = TextEditingController();
  final _otpController = TextEditingController();
  String? _verificationId;
  bool _codeSent = false;
  bool _isLoading = false;
  int? _resendToken;

  @override
  void dispose() {
    _phoneController.dispose();
    _otpController.dispose();
    super.dispose();
  }

  Future<void> _sendOTP() async {
    final phone = '+66${_phoneController.text.trim()}';
    setState(() => _isLoading = true);

    await PhoneAuthService.sendOTP(
      phoneNumber: phone,
      onVerificationCompleted: (credential) async {
        // Auto-verification (Android เท่านั้น)
        try {
          await FirebaseAuth.instance.signInWithCredential(credential);
        } catch (e) {
          if (mounted) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('Auto-verification ล้มเหลว: $e')),
            );
          }
        }
      },
      onVerificationFailed: (e) {
        setState(() => _isLoading = false);
        if (mounted) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('เกิดข้อผิดพลาด: ${e.message}'),
              backgroundColor: Colors.red,
            ),
          );
        }
      },
      onCodeSent: (verificationId, resendToken) {
        setState(() {
          _verificationId = verificationId;
          _resendToken = resendToken;
          _codeSent = true;
          _isLoading = false;
        });
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('ส่ง OTP แล้ว'),
            backgroundColor: Colors.green,
          ),
        );
      },
      onCodeAutoRetrievalTimeout: (verificationId) {
        setState(() {
          _verificationId = verificationId;
          _isLoading = false;
        });
      },
    );
  }

  Future<void> _verifyOTP() async {
    if (_verificationId == null) return;

    setState(() => _isLoading = true);
    try {
      await PhoneAuthService.verifyOTP(
        verificationId: _verificationId!,
        smsCode: _otpController.text.trim(),
      );
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('เข้าสู่ระบบสำเร็จ!'),
            backgroundColor: Colors.green,
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('OTP ไม่ถูกต้อง: $e'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Phone Authentication')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.phone_android, size: 80, color: Colors.blue),
            const SizedBox(height: 24),

            if (!_codeSent) ...[
              const Text(
                'ยืนยันด้วยหมายเลขโทรศัพท์',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 24),
              Row(
                children: [
                  Container(
                    padding: const EdgeInsets.symmetric(
                        horizontal: 12, vertical: 16),
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.grey),
                      borderRadius: BorderRadius.circular(8),
                    ),
                    child: const Text('+66'),
                  ),
                  const SizedBox(width: 8),
                  Expanded(
                    child: TextField(
                      controller: _phoneController,
                      decoration: const InputDecoration(
                        labelText: 'หมายเลขโทรศัพท์',
                        hintText: '812345678',
                        border: OutlineInputBorder(),
                      ),
                      keyboardType: TextInputType.phone,
                    ),
                  ),
                ],
              ),
              const SizedBox(height: 24),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _sendOTP,
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(vertical: 16),
                  ),
                  child: _isLoading
                      ? const CircularProgressIndicator()
                      : const Text('ส่ง OTP'),
                ),
              ),
            ] else ...[
              Text(
                'ป้อน OTP ที่ส่งไปยัง +66${_phoneController.text}',
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 16),
              ),
              const SizedBox(height: 24),
              TextField(
                controller: _otpController,
                decoration: const InputDecoration(
                  labelText: 'รหัส OTP',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.number,
                maxLength: 6,
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 24, letterSpacing: 8),
              ),
              const SizedBox(height: 16),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _isLoading ? null : _verifyOTP,
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(vertical: 16),
                  ),
                  child: _isLoading
                      ? const CircularProgressIndicator()
                      : const Text('ยืนยัน OTP'),
                ),
              ),
              TextButton(
                onPressed: () => setState(() {
                  _codeSent = false;
                  _otpController.clear();
                }),
                child: const Text('เปลี่ยนหมายเลขโทรศัพท์'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 226: Anonymous Auth

```dart
// lib/services/anonymous_auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class AnonymousAuthService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  // Sign in anonymously
  static Future<UserCredential> signInAnonymously() async {
    try {
      return await _auth.signInAnonymously();
    } on FirebaseAuthException catch (e) {
      throw Exception('Anonymous sign-in ล้มเหลว: ${e.message}');
    }
  }

  // Upgrade anonymous account to permanent
  static Future<UserCredential> linkWithEmail({
    required String email,
    required String password,
  }) async {
    final user = _auth.currentUser;
    if (user == null || !user.isAnonymous) {
      throw Exception('ไม่พบ Anonymous user');
    }

    final credential = EmailAuthProvider.credential(
      email: email,
      password: password,
    );

    try {
      return await user.linkWithCredential(credential);
    } on FirebaseAuthException catch (e) {
      throw Exception('Link account ล้มเหลว: ${e.message}');
    }
  }

  // Upgrade anonymous to Google
  static Future<UserCredential> linkWithGoogle() async {
    final user = _auth.currentUser;
    if (user == null || !user.isAnonymous) {
      throw Exception('ไม่พบ Anonymous user');
    }

    // Get Google credential (ดูจาก Google Sign-In section)
    // ...
    throw UnimplementedError('ดูตัวอย่างใน Google Sign-In section');
  }
}
```

---

## ขั้นตอนที่ 227: Auth State Changes

```dart
// lib/pages/auth_wrapper.dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'home_page.dart';
import 'login_page.dart';

class AuthWrapper extends StatelessWidget {
  const AuthWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        // กำลังโหลด
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }

        // Login แล้ว
        if (snapshot.hasData && snapshot.data != null) {
          final user = snapshot.data!;

          // ตรวจสอบว่า Verify Email แล้วหรือยัง
          if (!user.emailVerified && !user.isAnonymous) {
            return const EmailVerificationPage();
          }

          return const HomePage();
        }

        // ยังไม่ได้ Login
        return const LoginPage();
      },
    );
  }
}

// lib/main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'pages/auth_wrapper.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
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
      title: 'Firebase Auth',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const AuthWrapper(),
    );
  }
}
```

---

## ขั้นตอนที่ 228: User Profile Management

```dart
// lib/services/user_profile_service.dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:firebase_storage/firebase_storage.dart';
import 'dart:io';

class UserProfileService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  static User? get currentUser => _auth.currentUser;

  // อัพเดต Display Name
  static Future<void> updateDisplayName(String name) async {
    await _auth.currentUser?.updateDisplayName(name);
    await _auth.currentUser?.reload();
  }

  // อัพเดต Photo URL
  static Future<void> updatePhotoURL(String photoURL) async {
    await _auth.currentUser?.updatePhotoURL(photoURL);
    await _auth.currentUser?.reload();
  }

  // อัพโหลดรูปโปรไฟล์ไปยัง Firebase Storage
  static Future<String> uploadProfilePhoto(File imageFile) async {
    final user = _auth.currentUser;
    if (user == null) throw Exception('ไม่ได้ Login');

    final storageRef = FirebaseStorage.instance
        .ref()
        .child('profile_photos')
        .child('${user.uid}.jpg');

    final uploadTask = await storageRef.putFile(imageFile);
    final downloadURL = await uploadTask.ref.getDownloadURL();

    await updatePhotoURL(downloadURL);
    return downloadURL;
  }

  // อัพเดต Email
  static Future<void> updateEmail(String newEmail) async {
    await _auth.currentUser?.verifyBeforeUpdateEmail(newEmail);
  }

  // เปลี่ยนรหัสผ่าน
  static Future<void> changePassword({
    required String currentPassword,
    required String newPassword,
  }) async {
    final user = _auth.currentUser;
    if (user == null || user.email == null) throw Exception('ไม่ได้ Login');

    // Re-authenticate ก่อนเปลี่ยนรหัสผ่าน
    final credential = EmailAuthProvider.credential(
      email: user.email!,
      password: currentPassword,
    );
    await user.reauthenticateWithCredential(credential);

    // เปลี่ยนรหัสผ่าน
    await user.updatePassword(newPassword);
  }

  // ลบ Account
  static Future<void> deleteAccount({required String password}) async {
    final user = _auth.currentUser;
    if (user == null || user.email == null) throw Exception('ไม่ได้ Login');

    // Re-authenticate
    final credential = EmailAuthProvider.credential(
      email: user.email!,
      password: password,
    );
    await user.reauthenticateWithCredential(credential);

    // ลบข้อมูลใน Firestore ก่อน (ถ้ามี)
    // await FirebaseFirestore.instance.collection('users').doc(user.uid).delete();

    // ลบ Account
    await user.delete();
  }
}

// lib/pages/profile_page.dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../services/user_profile_service.dart';

class ProfilePage extends StatefulWidget {
  const ProfilePage({super.key});

  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  final _nameController = TextEditingController();
  bool _isEditing = false;

  @override
  void initState() {
    super.initState();
    final user = FirebaseAuth.instance.currentUser;
    _nameController.text = user?.displayName ?? '';
  }

  @override
  void dispose() {
    _nameController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('โปรไฟล์'),
        actions: [
          IconButton(
            icon: Icon(_isEditing ? Icons.check : Icons.edit),
            onPressed: () async {
              if (_isEditing) {
                try {
                  await UserProfileService.updateDisplayName(
                    _nameController.text.trim(),
                  );
                  if (mounted) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(
                        content: Text('อัพเดตโปรไฟล์สำเร็จ'),
                        backgroundColor: Colors.green,
                      ),
                    );
                  }
                } catch (e) {
                  if (mounted) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('Error: $e')),
                    );
                  }
                }
              }
              setState(() => _isEditing = !_isEditing);
            },
          ),
        ],
      ),
      body: StreamBuilder<User?>(
        stream: FirebaseAuth.instance.userChanges(),
        builder: (context, snapshot) {
          final user = snapshot.data ?? FirebaseAuth.instance.currentUser;
          if (user == null) return const Center(child: Text('ไม่ได้ Login'));

          return ListView(
            padding: const EdgeInsets.all(24),
            children: [
              // Profile Picture
              Center(
                child: Stack(
                  children: [
                    CircleAvatar(
                      radius: 60,
                      backgroundImage: user.photoURL != null
                          ? NetworkImage(user.photoURL!)
                          : null,
                      child: user.photoURL == null
                          ? const Icon(Icons.person, size: 60)
                          : null,
                    ),
                    if (_isEditing)
                      Positioned(
                        bottom: 0,
                        right: 0,
                        child: CircleAvatar(
                          radius: 18,
                          backgroundColor: Colors.blue,
                          child: IconButton(
                            icon: const Icon(Icons.camera_alt,
                                size: 16, color: Colors.white),
                            onPressed: () {
                              // Pick image
                            },
                          ),
                        ),
                      ),
                  ],
                ),
              ),
              const SizedBox(height: 32),

              // Name
              TextFormField(
                controller: _nameController,
                enabled: _isEditing,
                decoration: const InputDecoration(
                  labelText: 'ชื่อ',
                  prefixIcon: Icon(Icons.person_outline),
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 16),

              // Email (read-only)
              TextFormField(
                initialValue: user.email ?? 'ไม่มี Email',
                enabled: false,
                decoration: InputDecoration(
                  labelText: 'Email',
                  prefixIcon: const Icon(Icons.email_outlined),
                  border: const OutlineInputBorder(),
                  suffixIcon: user.emailVerified
                      ? const Icon(Icons.verified, color: Colors.green)
                      : const Icon(Icons.warning, color: Colors.orange),
                ),
              ),
              const SizedBox(height: 16),

              // UID
              TextFormField(
                initialValue: user.uid,
                enabled: false,
                decoration: const InputDecoration(
                  labelText: 'User ID',
                  prefixIcon: Icon(Icons.badge_outlined),
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 24),

              // Sign Out Button
              OutlinedButton.icon(
                onPressed: () => FirebaseAuth.instance.signOut(),
                icon: const Icon(Icons.logout, color: Colors.red),
                label: const Text(
                  'ออกจากระบบ',
                  style: TextStyle(color: Colors.red),
                ),
                style: OutlinedButton.styleFrom(
                  side: const BorderSide(color: Colors.red),
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 229: Password Reset

```dart
// lib/services/password_reset_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class PasswordResetService {
  static final FirebaseAuth _auth = FirebaseAuth.instance;

  // ส่ง Reset Password Email
  static Future<void> sendPasswordResetEmail(String email) async {
    try {
      await _auth.sendPasswordResetEmail(email: email);
    } on FirebaseAuthException catch (e) {
      switch (e.code) {
        case 'user-not-found':
          throw Exception('ไม่พบ Email นี้ในระบบ');
        case 'invalid-email':
          throw Exception('รูปแบบ Email ไม่ถูกต้อง');
        default:
          throw Exception('เกิดข้อผิดพลาด: ${e.message}');
      }
    }
  }

  // ยืนยัน Reset Code แบบ Custom (Advanced)
  static Future<void> confirmPasswordReset({
    required String oobCode,
    required String newPassword,
  }) async {
    try {
      await _auth.confirmPasswordReset(
        code: oobCode,
        newPassword: newPassword,
      );
    } on FirebaseAuthException catch (e) {
      switch (e.code) {
        case 'expired-action-code':
          throw Exception('Link หมดอายุแล้ว');
        case 'invalid-action-code':
          throw Exception('Link ไม่ถูกต้อง');
        case 'weak-password':
          throw Exception('รหัสผ่านไม่ปลอดภัยเพียงพอ');
        default:
          throw Exception('เกิดข้อผิดพลาด: ${e.message}');
      }
    }
  }
}

// lib/pages/forgot_password_page.dart
import 'package:flutter/material.dart';
import '../services/password_reset_service.dart';

class ForgotPasswordPage extends StatefulWidget {
  const ForgotPasswordPage({super.key});

  @override
  State<ForgotPasswordPage> createState() => _ForgotPasswordPageState();
}

class _ForgotPasswordPageState extends State<ForgotPasswordPage> {
  final _emailController = TextEditingController();
  bool _isLoading = false;
  bool _emailSent = false;

  @override
  void dispose() {
    _emailController.dispose();
    super.dispose();
  }

  Future<void> _sendResetEmail() async {
    setState(() => _isLoading = true);
    try {
      await PasswordResetService.sendPasswordResetEmail(
        _emailController.text.trim(),
      );
      setState(() => _emailSent = true);
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('$e'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ลืมรหัสผ่าน')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: _emailSent ? _buildSuccessView() : _buildFormView(),
      ),
    );
  }

  Widget _buildFormView() {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        const Icon(Icons.lock_reset, size: 80, color: Colors.blue),
        const SizedBox(height: 24),
        const Text(
          'รีเซ็ตรหัสผ่าน',
          style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          textAlign: TextAlign.center,
        ),
        const SizedBox(height: 8),
        const Text(
          'ใส่ Email ของคุณ เราจะส่ง Link รีเซ็ตรหัสผ่านไปให้',
          textAlign: TextAlign.center,
          style: TextStyle(color: Colors.grey),
        ),
        const SizedBox(height: 32),
        TextField(
          controller: _emailController,
          decoration: const InputDecoration(
            labelText: 'Email',
            prefixIcon: Icon(Icons.email_outlined),
            border: OutlineInputBorder(),
          ),
          keyboardType: TextInputType.emailAddress,
        ),
        const SizedBox(height: 24),
        ElevatedButton(
          onPressed: _isLoading ? null : _sendResetEmail,
          style: ElevatedButton.styleFrom(
            padding: const EdgeInsets.symmetric(vertical: 16),
          ),
          child: _isLoading
              ? const CircularProgressIndicator()
              : const Text('ส่ง Email รีเซ็ตรหัสผ่าน'),
        ),
      ],
    );
  }

  Widget _buildSuccessView() {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        const Icon(Icons.mark_email_read, size: 80, color: Colors.green),
        const SizedBox(height: 24),
        const Text(
          'ส่ง Email แล้ว!',
          style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 8),
        Text(
          'ตรวจสอบ Email ของคุณที่\n${_emailController.text}',
          textAlign: TextAlign.center,
          style: const TextStyle(color: Colors.grey),
        ),
        const SizedBox(height: 32),
        TextButton(
          onPressed: () => setState(() => _emailSent = false),
          child: const Text('ส่งอีกครั้ง'),
        ),
        ElevatedButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('กลับไปหน้า Login'),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 230: Email Verification

```dart
// lib/pages/email_verification_page.dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

class EmailVerificationPage extends StatefulWidget {
  const EmailVerificationPage({super.key});

  @override
  State<EmailVerificationPage> createState() => _EmailVerificationPageState();
}

class _EmailVerificationPageState extends State<EmailVerificationPage> {
  Timer? _timer;
  bool _isEmailVerified = false;
  bool _canResendEmail = true;
  int _resendCountdown = 0;

  @override
  void initState() {
    super.initState();
    _isEmailVerified = FirebaseAuth.instance.currentUser?.emailVerified ?? false;

    if (!_isEmailVerified) {
      _sendVerificationEmail();
      _startVerificationCheck();
    }
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  Future<void> _sendVerificationEmail() async {
    try {
      await FirebaseAuth.instance.currentUser?.sendEmailVerification();
      setState(() {
        _canResendEmail = false;
        _resendCountdown = 60;
      });

      // Countdown สำหรับ Resend
      Timer.periodic(const Duration(seconds: 1), (timer) {
        if (!mounted) {
          timer.cancel();
          return;
        }
        setState(() {
          _resendCountdown--;
          if (_resendCountdown <= 0) {
            _canResendEmail = true;
            timer.cancel();
          }
        });
      });
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('ส่ง Email ล้มเหลว: $e')),
        );
      }
    }
  }

  void _startVerificationCheck() {
    _timer = Timer.periodic(const Duration(seconds: 3), (_) async {
      await FirebaseAuth.instance.currentUser?.reload();
      final isVerified =
          FirebaseAuth.instance.currentUser?.emailVerified ?? false;

      if (isVerified) {
        _timer?.cancel();
        if (mounted) {
          setState(() => _isEmailVerified = true);
        }
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    if (_isEmailVerified) {
      // Email ยืนยันแล้ว จะถูก redirect โดย AuthWrapper
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    final user = FirebaseAuth.instance.currentUser;

    return Scaffold(
      appBar: AppBar(
        title: const Text('ยืนยัน Email'),
        actions: [
          TextButton(
            onPressed: () => FirebaseAuth.instance.signOut(),
            child: const Text('ออกจากระบบ'),
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            const Icon(Icons.mark_email_unread, size: 80, color: Colors.orange),
            const SizedBox(height: 24),
            const Text(
              'ยืนยัน Email ของคุณ',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 8),
            Text(
              'เราส่ง Link ยืนยันไปยัง\n${user?.email}',
              textAlign: TextAlign.center,
              style: const TextStyle(color: Colors.grey, fontSize: 16),
            ),
            const SizedBox(height: 32),
            const LinearProgressIndicator(),
            const SizedBox(height: 8),
            const Text(
              'กำลังตรวจสอบการยืนยัน...',
              textAlign: TextAlign.center,
              style: TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 32),
            ElevatedButton.icon(
              onPressed: _canResendEmail ? _sendVerificationEmail : null,
              icon: const Icon(Icons.send),
              label: Text(
                _canResendEmail
                    ? 'ส่ง Email ยืนยันอีกครั้ง'
                    : 'ส่งอีกครั้งใน ${_resendCountdown}s',
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

## Workshop: Complete Auth Flow

```dart
// lib/workshop/complete_auth_app.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const CompleteAuthApp());
}

class CompleteAuthApp extends StatelessWidget {
  const CompleteAuthApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Complete Auth',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: StreamBuilder<User?>(
        stream: FirebaseAuth.instance.authStateChanges(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const SplashScreen();
          }
          if (snapshot.hasData) {
            final user = snapshot.data!;
            if (!user.emailVerified && !user.isAnonymous) {
              return const EmailVerificationPage();
            }
            return const DashboardPage();
          }
          return const AuthScreen();
        },
      ),
    );
  }
}

class SplashScreen extends StatelessWidget {
  const SplashScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.local_fire_department, size: 80, color: Colors.orange),
            SizedBox(height: 16),
            CircularProgressIndicator(),
          ],
        ),
      ),
    );
  }
}

class AuthScreen extends StatefulWidget {
  const AuthScreen({super.key});

  @override
  State<AuthScreen> createState() => _AuthScreenState();
}

class _AuthScreenState extends State<AuthScreen>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 2, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            const SizedBox(height: 40),
            const Icon(
              Icons.account_circle,
              size: 80,
              color: Colors.deepPurple,
            ),
            const SizedBox(height: 16),
            Text(
              'ยินดีต้อนรับ',
              style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                    fontWeight: FontWeight.bold,
                  ),
            ),
            const SizedBox(height: 24),
            TabBar(
              controller: _tabController,
              tabs: const [
                Tab(text: 'เข้าสู่ระบบ'),
                Tab(text: 'สมัครสมาชิก'),
              ],
            ),
            Expanded(
              child: TabBarView(
                controller: _tabController,
                children: const [
                  LoginTab(),
                  RegisterTab(),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class LoginTab extends StatefulWidget {
  const LoginTab({super.key});

  @override
  State<LoginTab> createState() => _LoginTabState();
}

class _LoginTabState extends State<LoginTab> {
  final _emailCtrl = TextEditingController();
  final _passCtrl = TextEditingController();
  bool _loading = false;

  @override
  void dispose() {
    _emailCtrl.dispose();
    _passCtrl.dispose();
    super.dispose();
  }

  Future<void> _login() async {
    setState(() => _loading = true);
    try {
      await FirebaseAuth.instance.signInWithEmailAndPassword(
        email: _emailCtrl.text.trim(),
        password: _passCtrl.text,
      );
    } on FirebaseAuthException catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.message ?? 'เกิดข้อผิดพลาด'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        children: [
          TextField(
            controller: _emailCtrl,
            decoration: const InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.email),
            ),
          ),
          const SizedBox(height: 16),
          TextField(
            controller: _passCtrl,
            decoration: const InputDecoration(
              labelText: 'รหัสผ่าน',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.lock),
            ),
            obscureText: true,
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: _loading ? null : _login,
              child: _loading
                  ? const CircularProgressIndicator()
                  : const Text('เข้าสู่ระบบ'),
            ),
          ),
          const SizedBox(height: 8),
          SizedBox(
            width: double.infinity,
            child: OutlinedButton(
              onPressed: () async {
                await FirebaseAuth.instance.signInAnonymously();
              },
              child: const Text('เข้าใช้แบบไม่ระบุตัวตน'),
            ),
          ),
        ],
      ),
    );
  }
}

class RegisterTab extends StatefulWidget {
  const RegisterTab({super.key});

  @override
  State<RegisterTab> createState() => _RegisterTabState();
}

class _RegisterTabState extends State<RegisterTab> {
  final _emailCtrl = TextEditingController();
  final _passCtrl = TextEditingController();
  final _nameCtrl = TextEditingController();
  bool _loading = false;

  @override
  void dispose() {
    _emailCtrl.dispose();
    _passCtrl.dispose();
    _nameCtrl.dispose();
    super.dispose();
  }

  Future<void> _register() async {
    setState(() => _loading = true);
    try {
      final credential =
          await FirebaseAuth.instance.createUserWithEmailAndPassword(
        email: _emailCtrl.text.trim(),
        password: _passCtrl.text,
      );
      await credential.user?.updateDisplayName(_nameCtrl.text.trim());
      await credential.user?.sendEmailVerification();
    } on FirebaseAuthException catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(e.message ?? 'เกิดข้อผิดพลาด'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        children: [
          TextField(
            controller: _nameCtrl,
            decoration: const InputDecoration(
              labelText: 'ชื่อ',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.person),
            ),
          ),
          const SizedBox(height: 16),
          TextField(
            controller: _emailCtrl,
            decoration: const InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.email),
            ),
          ),
          const SizedBox(height: 16),
          TextField(
            controller: _passCtrl,
            decoration: const InputDecoration(
              labelText: 'รหัสผ่าน',
              border: OutlineInputBorder(),
              prefixIcon: Icon(Icons.lock),
            ),
            obscureText: true,
          ),
          const SizedBox(height: 24),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: _loading ? null : _register,
              child: _loading
                  ? const CircularProgressIndicator()
                  : const Text('สมัครสมาชิก'),
            ),
          ),
        ],
      ),
    );
  }
}

class DashboardPage extends StatelessWidget {
  const DashboardPage({super.key});

  @override
  Widget build(BuildContext context) {
    final user = FirebaseAuth.instance.currentUser!;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Dashboard'),
        backgroundColor: Colors.deepPurple,
        foregroundColor: Colors.white,
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: () => FirebaseAuth.instance.signOut(),
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 50,
              backgroundImage: user.photoURL != null
                  ? NetworkImage(user.photoURL!)
                  : null,
              child:
                  user.photoURL == null ? const Icon(Icons.person, size: 50) : null,
            ),
            const SizedBox(height: 16),
            Text(
              user.displayName ?? 'ไม่มีชื่อ',
              style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Text(
              user.email ?? 'Anonymous',
              style: const TextStyle(color: Colors.grey),
            ),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(
                  user.emailVerified ? Icons.verified : Icons.warning,
                  color: user.emailVerified ? Colors.green : Colors.orange,
                ),
                const SizedBox(width: 4),
                Text(
                  user.emailVerified ? 'Email ยืนยันแล้ว' : 'ยังไม่ได้ยืนยัน Email',
                  style: TextStyle(
                    color: user.emailVerified ? Colors.green : Colors.orange,
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Email/Password Auth**: การ Register/Login ด้วย Email
- **Google Sign-In**: การ Login ด้วย Google Account
- **Sign in with Apple**: การ Login ด้วย Apple ID
- **Phone Auth (OTP)**: การยืนยันด้วยหมายเลขโทรศัพท์
- **Anonymous Auth**: การเข้าใช้โดยไม่ระบุตัวตน
- **Auth State Changes**: การติดตาม State การ Login
- **User Profile**: การจัดการข้อมูลโปรไฟล์
- **Password Reset**: การรีเซ็ตรหัสผ่าน
- **Email Verification**: การยืนยัน Email

## แบบฝึกหัด

1. เพิ่ม Facebook Login ใน Auth Page
2. Implement Re-authentication ก่อนเปลี่ยนรหัสผ่าน
3. สร้าง Profile Page ที่ให้อัพโหลดรูปโปรไฟล์ได้
4. เพิ่ม Remember Me ฟังก์ชัน
5. Implement Multi-Factor Authentication (MFA)

---

[⬅️ Part 22](part_22.md) | [Part 24 ➡️](part_24.md)

# Part 36: Internationalization (i18n)
## ขั้นตอนที่ 351-360

---

## สารบัญ
1. [Internationalization คืออะไร?](#internationalization-คืออะไร)
2. [flutter_localizations Setup](#flutter_localizations-setup)
3. [ARB Files](#arb-files)
4. [l10n.yaml Configuration](#l10nyaml-configuration)
5. [intl Package](#intl-package)
6. [Pluralization](#pluralization)
7. [Date/Time/Number Formatting](#datetime-formatting)
8. [RTL Support](#rtl-support)
9. [Locale-specific Assets](#locale-specific-assets)
10. [Translation Management Workflow](#translation-management)

---

## ขั้นตอนที่ 351: Internationalization คืออะไร?

Internationalization (i18n) คือการออกแบบแอปให้รองรับหลายภาษาและ locale

```
i18n = Internationalization (18 ตัวอักษรระหว่าง i และ n)
l10n = Localization (10 ตัวอักษรระหว่าง l และ n)

ความแตกต่าง:
Internationalization (i18n) = ออกแบบแอปให้รองรับ multiple languages
Localization (l10n) = การแปลและปรับแต่งสำหรับ locale หนึ่งๆ

ตัวอย่าง:
- แสดงข้อความภาษาไทย/อังกฤษ/ญี่ปุ่น
- วันที่แบบ MM/DD/YYYY หรือ DD/MM/YYYY
- ตัวเลข: 1,234.56 หรือ 1.234,56
- สกุลเงิน: $10 หรือ ฿310 หรือ ¥1,500
- RTL text สำหรับ Arabic/Hebrew
```

---

## ขั้นตอนที่ 352: flutter_localizations Setup

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.18.1

flutter:
  generate: true  # สำคัญมาก! ต้องเปิด code generation
```

### main.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My Internationalized App',
      
      // กำหนด locales ที่รองรับ
      supportedLocales: AppLocalizations.supportedLocales,
      
      // Flutter's built-in localizations delegates
      localizationsDelegates: const [
        AppLocalizations.delegate,                // Generated delegate
        GlobalMaterialLocalizations.delegate,     // Material widgets
        GlobalWidgetsLocalizations.delegate,      // Base widgets
        GlobalCupertinoLocalizations.delegate,    // Cupertino widgets
      ],
      
      // เลือก locale อัตโนมัติจาก device settings
      localeResolutionCallback: (locale, supportedLocales) {
        if (locale == null) return supportedLocales.first;
        
        // ตรวจสอบว่า locale ที่ขอรองรับหรือเปล่า
        for (final supportedLocale in supportedLocales) {
          if (supportedLocale.languageCode == locale.languageCode) {
            return supportedLocale;
          }
        }
        
        // Fallback ไปที่ locale แรก (en)
        return supportedLocales.first;
      },
      
      home: const HomePage(),
    );
  }
}
```

```dart
// ใช้ localizations ใน Widget
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    // ดึง localizations instance
    final l10n = AppLocalizations.of(context)!;
    
    return Scaffold(
      appBar: AppBar(
        title: Text(l10n.appTitle),
      ),
      body: Center(
        child: Text(l10n.welcomeMessage),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        tooltip: l10n.addProduct,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 353: ARB Files

ARB (Application Resource Bundle) เป็นรูปแบบไฟล์สำหรับเก็บ translations

### โครงสร้าง

```
lib/
└── l10n/
    ├── app_en.arb     # English (ภาษาหลัก)
    ├── app_th.arb     # Thai
    ├── app_ja.arb     # Japanese
    ├── app_zh.arb     # Chinese
    └── app_ar.arb     # Arabic
```

### app_en.arb (English - ภาษาหลัก)

```json
{
  "@@locale": "en",
  "appTitle": "My Store",
  "@appTitle": {
    "description": "The application title shown in the app bar"
  },
  
  "welcomeMessage": "Welcome to My Store!",
  "@welcomeMessage": {
    "description": "Welcome message on the home screen"
  },
  
  "greeting": "Hello, {name}!",
  "@greeting": {
    "description": "Greeting message with user name",
    "placeholders": {
      "name": {
        "type": "String",
        "example": "John"
      }
    }
  },
  
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "@itemCount": {
    "description": "Number of items with pluralization",
    "placeholders": {
      "count": {
        "type": "int",
        "example": "5"
      }
    }
  },
  
  "price": "Price: {amount}",
  "@price": {
    "description": "Price label",
    "placeholders": {
      "amount": {
        "type": "String"
      }
    }
  },
  
  "lastUpdated": "Last updated: {date}",
  "@lastUpdated": {
    "description": "Last updated date",
    "placeholders": {
      "date": {
        "type": "DateTime",
        "format": "yMMMMd",
        "isCustomDateFormat": "false"
      }
    }
  },
  
  "addProduct": "Add Product",
  "editProduct": "Edit Product",
  "deleteProduct": "Delete Product",
  "confirmDelete": "Are you sure you want to delete \"{productName}\"?",
  "@confirmDelete": {
    "placeholders": {
      "productName": {
        "type": "String"
      }
    }
  },
  
  "yes": "Yes",
  "no": "No",
  "cancel": "Cancel",
  "save": "Save",
  "loading": "Loading...",
  "error": "An error occurred",
  "retry": "Retry",
  "noResults": "No results found",
  
  "login": "Login",
  "logout": "Logout",
  "register": "Register",
  "email": "Email",
  "password": "Password",
  "confirmPassword": "Confirm Password",
  
  "emailRequired": "Email is required",
  "emailInvalid": "Please enter a valid email",
  "passwordRequired": "Password is required",
  "passwordTooShort": "Password must be at least {minLength} characters",
  "@passwordTooShort": {
    "placeholders": {
      "minLength": {
        "type": "int"
      }
    }
  },
  
  "searchHint": "Search products...",
  "filterBy": "Filter by",
  "sortBy": "Sort by",
  "category": "Category",
  "price": "Price",
  "rating": "Rating",
  
  "inStock": "In Stock",
  "outOfStock": "Out of Stock",
  "addToCart": "Add to Cart",
  "buyNow": "Buy Now",
  
  "cart": "Cart",
  "checkout": "Checkout",
  "total": "Total",
  "subtotal": "Subtotal",
  "shipping": "Shipping",
  "tax": "Tax",
  
  "orderSuccess": "Order placed successfully!",
  "orderNumber": "Order #{orderNumber}",
  "@orderNumber": {
    "placeholders": {
      "orderNumber": {
        "type": "String"
      }
    }
  }
}
```

### app_th.arb (Thai)

```json
{
  "@@locale": "th",
  "appTitle": "ร้านค้าของฉัน",
  "welcomeMessage": "ยินดีต้อนรับสู่ร้านค้าของฉัน!",
  "greeting": "สวัสดี, {name}!",
  "itemCount": "{count, plural, =0{ไม่มีรายการ} =1{1 รายการ} other{{count} รายการ}}",
  "price": "ราคา: {amount}",
  "lastUpdated": "อัปเดตล่าสุด: {date}",
  "addProduct": "เพิ่มสินค้า",
  "editProduct": "แก้ไขสินค้า",
  "deleteProduct": "ลบสินค้า",
  "confirmDelete": "คุณแน่ใจหรือไม่ว่าต้องการลบ \"{productName}\"?",
  "yes": "ใช่",
  "no": "ไม่",
  "cancel": "ยกเลิก",
  "save": "บันทึก",
  "loading": "กำลังโหลด...",
  "error": "เกิดข้อผิดพลาด",
  "retry": "ลองอีกครั้ง",
  "noResults": "ไม่พบผลลัพธ์",
  "login": "เข้าสู่ระบบ",
  "logout": "ออกจากระบบ",
  "register": "สมัครสมาชิก",
  "email": "อีเมล",
  "password": "รหัสผ่าน",
  "confirmPassword": "ยืนยันรหัสผ่าน",
  "emailRequired": "กรุณากรอกอีเมล",
  "emailInvalid": "กรุณากรอกอีเมลที่ถูกต้อง",
  "passwordRequired": "กรุณากรอกรหัสผ่าน",
  "passwordTooShort": "รหัสผ่านต้องมีอย่างน้อย {minLength} ตัวอักษร",
  "searchHint": "ค้นหาสินค้า...",
  "filterBy": "กรองตาม",
  "sortBy": "เรียงตาม",
  "category": "หมวดหมู่",
  "rating": "คะแนน",
  "inStock": "มีสินค้า",
  "outOfStock": "สินค้าหมด",
  "addToCart": "เพิ่มในตะกร้า",
  "buyNow": "ซื้อเลย",
  "cart": "ตะกร้าสินค้า",
  "checkout": "ชำระเงิน",
  "total": "ยอดรวม",
  "subtotal": "ราคารวม",
  "shipping": "ค่าจัดส่ง",
  "tax": "ภาษี",
  "orderSuccess": "สั่งซื้อสำเร็จ!",
  "orderNumber": "หมายเลขคำสั่งซื้อ #{orderNumber}"
}
```

### app_ar.arb (Arabic - RTL)

```json
{
  "@@locale": "ar",
  "appTitle": "متجري",
  "welcomeMessage": "مرحباً بك في متجري!",
  "greeting": "مرحباً، {name}!",
  "itemCount": "{count, plural, =0{لا توجد عناصر} =1{عنصر واحد} =2{عنصران} few{{count} عناصر} many{{count} عنصراً} other{{count} عنصر}}",
  "addProduct": "إضافة منتج",
  "cancel": "إلغاء",
  "save": "حفظ",
  "loading": "جارٍ التحميل...",
  "inStock": "متوفر",
  "outOfStock": "غير متوفر",
  "addToCart": "أضف إلى السلة",
  "total": "الإجمالي"
}
```

---

## ขั้นตอนที่ 354: l10n.yaml Configuration

```yaml
# l10n.yaml (ที่ root ของ project)
arb-dir: lib/l10n           # ที่อยู่ ARB files
template-arb-file: app_en.arb # Template (ภาษาหลัก)
output-localization-file: app_localizations.dart # Output file name
output-dir: lib/generated/l10n # ที่อยู่ generated files

# Optional settings
nullable-getter: false        # AppLocalizations.of(context)! หรือ ?
preferred-supported-locales:  # เรียงลำดับ locales
  - en
  - th
  - ja
  - zh
  - ar

# Generate untranslated messages ด้วยภาษา fallback
untranslated-messages-file: untranslated.txt

# Synthetic package
synthetic-package: false      # ใช้ lib/generated แทน .dart_tool
```

```bash
# Generate localization code
flutter gen-l10n

# หรือ watch mode
flutter gen-l10n --watch
```

---

## ขั้นตอนที่ 355: intl Package

```dart
// pubspec.yaml
// intl: ^0.18.1

import 'package:intl/intl.dart';

class FormattingHelper {
  // Currency Formatting
  static String formatCurrency(
    double amount,
    String locale, {
    String? symbol,
    int decimalDigits = 2,
  }) {
    final format = NumberFormat.currency(
      locale: locale,
      symbol: symbol,
      decimalDigits: decimalDigits,
    );
    return format.format(amount);
  }

  // Number Formatting
  static String formatNumber(double number, String locale) {
    return NumberFormat('#,##0.##', locale).format(number);
  }

  // Compact Number (1K, 1M, 1B)
  static String formatCompact(int number, String locale) {
    return NumberFormat.compact(locale: locale).format(number);
  }

  // Percentage
  static String formatPercent(double value, String locale) {
    return NumberFormat.percentPattern(locale).format(value);
  }

  // Date Formatting
  static String formatDate(DateTime date, String locale) {
    return DateFormat.yMMMMd(locale).format(date);
  }

  static String formatDateTime(DateTime dateTime, String locale) {
    return DateFormat.yMMMMd(locale).add_jm().format(dateTime);
  }

  static String formatRelativeTime(DateTime date, String locale) {
    final now = DateTime.now();
    final difference = now.difference(date);

    if (difference.inDays > 365) {
      return '${(difference.inDays / 365).floor()} years ago';
    } else if (difference.inDays > 30) {
      return '${(difference.inDays / 30).floor()} months ago';
    } else if (difference.inDays > 0) {
      return '${difference.inDays} days ago';
    } else if (difference.inHours > 0) {
      return '${difference.inHours} hours ago';
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes} minutes ago';
    } else {
      return 'Just now';
    }
  }
}
```

### การใช้งานใน Widget

```dart
class PriceWidget extends StatelessWidget {
  final double price;
  
  const PriceWidget({super.key, required this.price});

  @override
  Widget build(BuildContext context) {
    final locale = Localizations.localeOf(context).toString();
    
    // แสดงราคาตาม locale
    final formattedPrice = switch (locale) {
      'th' => NumberFormat.currency(
          locale: 'th_TH', 
          symbol: '฿',
          decimalDigits: 0,
        ).format(price),
      'ja' => NumberFormat.currency(
          locale: 'ja_JP',
          symbol: '¥',
          decimalDigits: 0,
        ).format(price),
      'ar' => NumberFormat.currency(
          locale: 'ar_SA',
          symbol: 'ر.س',
        ).format(price),
      _ => NumberFormat.currency(
          locale: 'en_US',
          symbol: '\$',
        ).format(price),
    };
    
    return Text(formattedPrice);
  }
}
```

---

## ขั้นตอนที่ 356: Pluralization

```json
// app_en.arb
{
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "@itemCount": {
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },
  
  "reviewCount": "{count, plural, =0{No reviews} =1{1 review} other{{count} reviews}}",
  
  "daysAgo": "{days, plural, =0{Today} =1{Yesterday} other{{days} days ago}}",
  "@daysAgo": {
    "placeholders": {
      "days": {
        "type": "int"
      }
    }
  },
  
  "cartItems": "You have {count, plural, =0{nothing} =1{1 item} other{{count} items}} in your cart",
  "@cartItems": {
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  }
}
```

```json
// app_th.arb - ภาษาไทยไม่มี plural (ใช้ other เสมอ)
{
  "itemCount": "{count, plural, =0{ไม่มีรายการ} other{{count} รายการ}}",
  "reviewCount": "{count, plural, =0{ยังไม่มีรีวิว} other{{count} รีวิว}}",
  "daysAgo": "{days, plural, =0{วันนี้} =1{เมื่อวาน} other{{days} วันที่แล้ว}}",
  "cartItems": "คุณมี{count, plural, =0{ไม่มีสินค้า} other{{count} รายการ}}ในตะกร้า"
}
```

```json
// app_ar.arb - Arabic มี plural forms หลายแบบ
{
  "itemCount": "{count, plural, =0{لا توجد عناصر} =1{عنصر واحد} =2{عنصران} few{{count} عناصر} many{{count} عنصراً} other{{count} عنصر}}"
}
```

### การใช้งาน Plural

```dart
class CartBadge extends StatelessWidget {
  final int itemCount;
  
  const CartBadge({super.key, required this.itemCount});

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    
    return Badge(
      label: Text('$itemCount'),
      child: Semantics(
        label: l10n.itemCount(itemCount), // Automatically handles plural
        child: const Icon(Icons.shopping_cart),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 357: Date/Time/Number Formatting

```dart
// lib/core/utils/locale_utils.dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class LocaleUtils {
  static String formatDate(
    BuildContext context,
    DateTime date, {
    bool showTime = false,
  }) {
    final locale = Localizations.localeOf(context).toString();
    
    if (showTime) {
      return DateFormat.yMMMMd(locale).add_jm().format(date);
    }
    return DateFormat.yMMMMd(locale).format(date);
  }

  static String formatShortDate(BuildContext context, DateTime date) {
    final locale = Localizations.localeOf(context).toString();
    return DateFormat.yMd(locale).format(date);
  }

  static String formatCurrency(
    BuildContext context,
    double amount, {
    String? currencyCode,
  }) {
    final locale = Localizations.localeOf(context);
    
    // กำหนด currency ตาม locale
    final currency = currencyCode ?? _getCurrencyForLocale(locale);
    
    return NumberFormat.simpleCurrency(
      locale: locale.toString(),
      name: currency,
    ).format(amount);
  }

  static String _getCurrencyForLocale(Locale locale) {
    return switch (locale.countryCode ?? locale.languageCode) {
      'TH' || 'th' => 'THB',
      'JP' || 'ja' => 'JPY',
      'CN' || 'zh' => 'CNY',
      'KR' || 'ko' => 'KRW',
      'GB' => 'GBP',
      'EU' || 'de' || 'fr' => 'EUR',
      _ => 'USD',
    };
  }

  static String formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    if (bytes < 1024 * 1024 * 1024) {
      return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
    }
    return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(1)} GB';
  }

  static String formatRelativeTime(
    BuildContext context,
    DateTime dateTime,
  ) {
    final l10n = AppLocalizations.of(context)!;
    final now = DateTime.now();
    final diff = now.difference(dateTime);
    
    if (diff.inDays > 0) {
      return l10n.daysAgo(diff.inDays);
    } else if (diff.inHours > 0) {
      return l10n.hoursAgo(diff.inHours);
    } else {
      return l10n.minutesAgo(diff.inMinutes);
    }
  }
}
```

---

## ขั้นตอนที่ 358: RTL Support

```dart
// Flutter จัดการ RTL อัตโนมัติสำหรับ Arabic/Hebrew
// แต่ต้องระวัง hard-coded layouts

// ❌ Hard-coded LTR Layout
Padding(
  padding: const EdgeInsets.only(left: 16), // ❌ ไม่รองรับ RTL
  child: const Icon(Icons.arrow_forward),   // ❌ ทิศทางผิดใน RTL
)

// ✅ Direction-aware Layout
Padding(
  padding: const EdgeInsetsDirectional.only(start: 16), // ✅ start/end แทน left/right
  child: const Icon(Icons.arrow_forward),
)

// ✅ หรือใช้ Directionality-aware widgets
Row(
  children: [
    const Padding(
      padding: EdgeInsetsDirectional.only(start: 8, end: 16),
      child: Icon(Icons.person),
    ),
    const Expanded(child: Text('User Name')),
  ],
)
```

### EdgeInsetsDirectional

```dart
// EdgeInsetsDirectional สำหรับ RTL-aware padding
const padding = EdgeInsetsDirectional.fromSTEB(
  start: 16,   // left ใน LTR, right ใน RTL
  top: 8,
  end: 16,     // right ใน LTR, left ใน RTL
  bottom: 8,
);

// Symmetric
const padding2 = EdgeInsetsDirectional.symmetric(
  horizontal: 16,
  vertical: 8,
);
```

### ตรวจสอบ Text Direction

```dart
class DirectionalWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final isRTL = Directionality.of(context) == TextDirection.rtl;
    
    return Row(
      // TextDirection จัดการ Row direction อัตโนมัติ
      children: [
        if (!isRTL) const Icon(Icons.chevron_left),
        const Text('Back'),
        if (isRTL) const Icon(Icons.chevron_right),
      ],
    );
  }
}
```

### Icons สำหรับ RTL

```dart
// ✅ ใช้ Icons.arrow_forward_ios แล้วจัดการ RTL
Widget backButton(BuildContext context) {
  final isRTL = Directionality.of(context) == TextDirection.rtl;
  
  return IconButton(
    icon: Icon(
      isRTL ? Icons.arrow_forward_ios : Icons.arrow_back_ios,
    ),
    onPressed: () => Navigator.pop(context),
  );
}

// หรือใช้ BackButton ที่จัดการ RTL อัตโนมัติ
BackButton()  // ✅ RTL-aware
```

### RTL App Bar

```dart
// AppBar จัดการ RTL อัตโนมัติ
// - Leading widget อยู่ขวาใน RTL
// - Actions อยู่ซ้ายใน RTL
AppBar(
  leading: const BackButton(), // อยู่ขวาใน RTL
  title: const Text('Title'),
  actions: [
    IconButton(
      icon: const Icon(Icons.settings),
      onPressed: () {},
    ),
  ],
)
```

### ทดสอบ RTL

```dart
// Force RTL ใน Debug
void main() {
  runApp(
    Directionality(
      textDirection: TextDirection.rtl, // Force RTL for testing
      child: const MaterialApp(
        home: MyHomePage(),
      ),
    ),
  );
}
```

---

## ขั้นตอนที่ 359: Locale-specific Assets

```yaml
# pubspec.yaml
flutter:
  assets:
    - assets/images/
    - assets/images/en/
    - assets/images/th/
    - assets/images/ar/
```

```dart
// lib/core/utils/asset_helper.dart
class AssetHelper {
  static String getLocalizedImagePath(
    BuildContext context,
    String filename,
  ) {
    final locale = Localizations.localeOf(context);
    final localizedPath = 'assets/images/${locale.languageCode}/$filename';
    
    // Check ว่าไฟล์มีหรือเปล่า, ถ้าไม่มีใช้ default
    return localizedPath;
  }
  
  static String getDefaultImagePath(String filename) {
    return 'assets/images/en/$filename';
  }
}

// Widget
class LocalizedImage extends StatelessWidget {
  final String filename;
  
  const LocalizedImage({super.key, required this.filename});

  @override
  Widget build(BuildContext context) {
    final locale = Localizations.localeOf(context);
    
    return Image.asset(
      'assets/images/${locale.languageCode}/$filename',
      // Fallback เมื่อ localized image ไม่มี
      errorBuilder: (_, __, ___) => Image.asset(
        'assets/images/en/$filename',
        errorBuilder: (_, __, ___) => const Icon(Icons.image_not_supported),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 360: Translation Management Workflow

### Language Switcher

```dart
// lib/core/providers/locale_provider.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';

final localeProvider = StateNotifierProvider<LocaleNotifier, Locale>((ref) {
  return LocaleNotifier();
});

class LocaleNotifier extends StateNotifier<Locale> {
  static const String _localeKey = 'selected_locale';

  LocaleNotifier() : super(const Locale('en')) {
    _loadSavedLocale();
  }

  Future<void> _loadSavedLocale() async {
    final prefs = await SharedPreferences.getInstance();
    final savedLocale = prefs.getString(_localeKey);
    if (savedLocale != null) {
      state = Locale(savedLocale);
    }
  }

  Future<void> setLocale(Locale locale) async {
    state = locale;
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_localeKey, locale.languageCode);
  }

  Future<void> setLocaleFromString(String languageCode) async {
    await setLocale(Locale(languageCode));
  }
}
```

```dart
// Language Selector Widget
class LanguageSelector extends ConsumerWidget {
  const LanguageSelector({super.key});

  static const _supportedLanguages = [
    ('en', 'English', '🇺🇸'),
    ('th', 'ภาษาไทย', '🇹🇭'),
    ('ja', '日本語', '🇯🇵'),
    ('zh', '中文', '🇨🇳'),
    ('ar', 'العربية', '🇸🇦'),
  ];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentLocale = ref.watch(localeProvider);
    
    return PopupMenuButton<String>(
      onSelected: (languageCode) {
        ref.read(localeProvider.notifier).setLocaleFromString(languageCode);
      },
      itemBuilder: (_) => _supportedLanguages.map((lang) {
        final (code, name, flag) = lang;
        return PopupMenuItem(
          value: code,
          child: Row(
            children: [
              Text(flag, style: const TextStyle(fontSize: 24)),
              const SizedBox(width: 12),
              Text(name),
              if (currentLocale.languageCode == code)
                const Padding(
                  padding: EdgeInsets.only(left: 8),
                  child: Icon(Icons.check, size: 16, color: Colors.green),
                ),
            ],
          ),
        );
      }).toList(),
      child: Padding(
        padding: const EdgeInsets.all(8),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              _supportedLanguages
                  .firstWhere((l) => l.$1 == currentLocale.languageCode,
                      orElse: () => _supportedLanguages.first)
                  .$3,
              style: const TextStyle(fontSize: 20),
            ),
            const Icon(Icons.arrow_drop_down),
          ],
        ),
      ),
    );
  }
}
```

### Main App ด้วย Riverpod + Locale

```dart
// main.dart
class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final locale = ref.watch(localeProvider);
    
    return MaterialApp(
      title: 'My App',
      locale: locale,
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      home: const HomePage(),
    );
  }
}
```

### Workshop: Multi-language Product Page

```dart
// WORKSHOP: Product Page ที่รองรับหลายภาษา
class LocalizedProductPage extends StatelessWidget {
  final Product product;
  
  const LocalizedProductPage({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    final locale = Localizations.localeOf(context).toString();
    
    return Scaffold(
      appBar: AppBar(
        title: Text(l10n.appTitle),
        actions: const [LanguageSelector()],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Product Name
            Text(
              product.localizedName(locale) ?? product.name,
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            
            const SizedBox(height: 8),
            
            // Price - formatted per locale
            Text(
              NumberFormat.simpleCurrency(locale: locale).format(product.price),
              style: Theme.of(context).textTheme.titleLarge!.copyWith(
                color: Colors.green,
                fontWeight: FontWeight.bold,
              ),
            ),
            
            const SizedBox(height: 8),
            
            // Stock Status
            Row(
              children: [
                Icon(
                  product.isInStock ? Icons.check_circle : Icons.cancel,
                  color: product.isInStock ? Colors.green : Colors.red,
                  size: 16,
                ),
                const SizedBox(width: 4),
                Text(
                  product.isInStock ? l10n.inStock : l10n.outOfStock,
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            
            // Review count with pluralization
            Text(l10n.reviewCount(product.reviewCount)),
            
            const SizedBox(height: 8),
            
            // Last updated date
            Text(
              l10n.lastUpdated(product.lastUpdated),
              style: Theme.of(context).textTheme.bodySmall,
            ),
            
            const SizedBox(height: 24),
            
            // Add to cart button
            SizedBox(
              width: double.infinity,
              height: 48,
              child: ElevatedButton(
                onPressed: product.isInStock ? () {} : null,
                child: Text(l10n.addToCart),
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

## สรุป (Summary)

```
Internationalization Workflow:

1. Setup:
   - เพิ่ม flutter_localizations ใน pubspec.yaml
   - สร้าง l10n.yaml
   - สร้าง ARB files สำหรับแต่ละภาษา

2. ARB File Structure:
   - "key": "translated text"
   - "@key": { "description": "..." }
   - Placeholders: "{name}"
   - Plural: "{count, plural, ...}"
   - Dates: type DateTime + format

3. Generate:
   - flutter gen-l10n

4. Use:
   - AppLocalizations.of(context)!.key

5. Testing:
   - Test ด้วย locale ต่างๆ
   - ตรวจสอบ RTL layouts
   - ทดสอบ text overflow กับ long translations

Common Pitfalls:
❌ Hard-coded strings ในโค้ด
❌ Hard-coded left/right padding (ใช้ start/end)
❌ Hard-coded arrow icons (ใช้ BackButton)
❌ Assuming date format
❌ Missing translations (ทำให้ app crash)

Best Practices:
✅ ทุก string ต้องอยู่ใน ARB file
✅ ใช้ EdgeInsetsDirectional สำหรับ padding
✅ Test ด้วย longest translation (German มักยาวที่สุด)
✅ Use Semantics labels ที่ localized ด้วย
✅ Format dates/numbers ตาม locale
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. Setup flutter_localizations และสร้าง ARB files สำหรับ EN และ TH
2. แปลง hard-coded strings ทั้งหมดในแอปไปใช้ l10n
3. สร้าง Language Selector widget

**ระดับกลาง:**
4. Implement pluralization สำหรับ cart items
5. Format dates และ numbers ตาม locale
6. สร้าง locale-specific images

**ระดับสูง:**
7. เพิ่มรองรับ Arabic (RTL) แล้วตรวจสอบ layout ทุกหน้า
8. Implement locale persistence ด้วย SharedPreferences
9. Setup Translation Memory workflow กับ Crowdin หรือ Lokalise

---

## การนำทาง
- [← Part 35: Accessibility](part_35.md)
- [→ Part 37: Flutter Web](part_37.md)

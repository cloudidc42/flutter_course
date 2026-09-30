# Part 18: JSON Serialization
## ขั้นตอนที่ 171-180

---

## สารบัญ
1. [dart:convert พื้นฐาน](#ขั้นตอนที่-171-dartconvert-พื้นฐาน)
2. [fromJson/toJson Patterns](#ขั้นตอนที่-172-fromjsontoJsonPatterns)
3. [Nested Objects](#ขั้นตอนที่-173-nested-objects)
4. [Lists และ Maps](#ขั้นตอนที่-174-lists-และ-maps)
5. [Null Safety ใน JSON](#ขั้นตอนที่-175-null-safety-ใน-json)
6. [json_serializable](#ขั้นตอนที่-176-json_serializable)
7. [freezed Package](#ขั้นตอนที่-177-freezed-package)
8. [Code Generation Setup](#ขั้นตอนที่-178-code-generation-setup)
9. [Custom Converters](#ขั้นตอนที่-179-custom-converters)
10. [Error Handling ใน JSON Parsing](#ขั้นตอนที่-180-error-handling-ใน-json-parsing)
11. [Workshop: Product Catalog App](#workshop-product-catalog-app)

---

## ขั้นตอนที่ 171: dart:convert พื้นฐาน

```dart
import 'dart:convert';

void main() {
  // jsonEncode - แปลง Dart object เป็น JSON string
  final map = {
    'name': 'Somchai',
    'age': 25,
    'isActive': true,
    'scores': [90, 85, 95],
    'address': {
      'city': 'Bangkok',
      'country': 'Thailand',
    },
  };

  final jsonString = jsonEncode(map);
  print(jsonString);
  // {"name":"Somchai","age":25,"isActive":true,"scores":[90,85,95],"address":{"city":"Bangkok","country":"Thailand"}}

  // jsonDecode - แปลง JSON string เป็น Dart object
  final decoded = jsonDecode(jsonString) as Map<String, dynamic>;
  print(decoded['name']); // Somchai
  print(decoded['age']); // 25

  // decode array
  final jsonArray = '[1, 2, 3, "hello", true]';
  final list = jsonDecode(jsonArray) as List<dynamic>;
  print(list[3]); // hello

  // Pretty print JSON
  const encoder = JsonEncoder.withIndent('  ');
  final prettyJson = encoder.convert(map);
  print(prettyJson);
  /*
  {
    "name": "Somchai",
    "age": 25,
    ...
  }
  */

  // จัดการ special types
  final special = {
    'date': DateTime.now().toIso8601String(), // DateTime ต้องแปลงเป็น String ก่อน
    'bigNumber': 9007199254740991, // int/double ปกติ
    'decimal': 3.14,
  };
  print(jsonEncode(special));
}
```

---

## ขั้นตอนที่ 172: fromJson/toJson Patterns

```dart
// Pattern 1: Manual fromJson/toJson
class User {
  final int id;
  final String name;
  final String email;
  final DateTime createdAt;
  final UserRole role;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
    required this.role,
  });

  // สร้าง User จาก JSON
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      createdAt: DateTime.parse(json['created_at'] as String),
      role: UserRole.fromJson(json['role'] as String),
    );
  }

  // แปลง User เป็น JSON
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'created_at': createdAt.toIso8601String(),
      'role': role.toJson(),
    };
  }

  // copyWith สำหรับ immutable updates
  User copyWith({
    int? id,
    String? name,
    String? email,
    DateTime? createdAt,
    UserRole? role,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      createdAt: createdAt ?? this.createdAt,
      role: role ?? this.role,
    );
  }

  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is User && other.id == id;
  }

  @override
  int get hashCode => id.hashCode;
}

enum UserRole {
  admin,
  user,
  moderator;

  factory UserRole.fromJson(String value) {
    return UserRole.values.firstWhere(
      (e) => e.name == value,
      orElse: () => UserRole.user,
    );
  }

  String toJson() => name;
}

// การใช้งาน
void useUser() {
  final json = {
    'id': 1,
    'name': 'Somchai Jaidee',
    'email': 'somchai@example.com',
    'created_at': '2024-01-15T10:30:00.000Z',
    'role': 'admin',
  };

  final user = User.fromJson(json);
  print(user.name); // Somchai Jaidee
  print(user.role); // UserRole.admin

  final encoded = user.toJson();
  print(encoded);
}

// Pattern 2: Factory หลายรูปแบบ
class Product {
  final int id;
  final String name;
  final double price;
  final String? imageUrl;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    this.imageUrl,
  });

  factory Product.fromJson(Map<String, dynamic> json) => Product(
    id: json['id'] as int,
    name: json['name'] as String,
    price: (json['price'] as num).toDouble(),
    imageUrl: json['image_url'] as String?,
  );

  // สร้างจาก API response โครงสร้างอื่น
  factory Product.fromApiResponse(Map<String, dynamic> json) => Product(
    id: json['product_id'] as int,
    name: json['product_name'] as String,
    price: double.parse(json['price'].toString()),
    imageUrl: json['images']?[0]?['url'] as String?,
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
    if (imageUrl != null) 'image_url': imageUrl,
  };
}
```

---

## ขั้นตอนที่ 173: Nested Objects

```dart
// Nested objects - Order กับ User และ List<OrderItem>
class Address {
  final String street;
  final String city;
  final String country;
  final String postalCode;

  const Address({
    required this.street,
    required this.city,
    required this.country,
    required this.postalCode,
  });

  factory Address.fromJson(Map<String, dynamic> json) => Address(
    street: json['street'] as String,
    city: json['city'] as String,
    country: json['country'] as String,
    postalCode: json['postal_code'] as String,
  );

  Map<String, dynamic> toJson() => {
    'street': street,
    'city': city,
    'country': country,
    'postal_code': postalCode,
  };
}

class OrderItem {
  final int productId;
  final String productName;
  final int quantity;
  final double unitPrice;

  const OrderItem({
    required this.productId,
    required this.productName,
    required this.quantity,
    required this.unitPrice,
  });

  double get totalPrice => quantity * unitPrice;

  factory OrderItem.fromJson(Map<String, dynamic> json) => OrderItem(
    productId: json['product_id'] as int,
    productName: json['product_name'] as String,
    quantity: json['quantity'] as int,
    unitPrice: (json['unit_price'] as num).toDouble(),
  );

  Map<String, dynamic> toJson() => {
    'product_id': productId,
    'product_name': productName,
    'quantity': quantity,
    'unit_price': unitPrice,
  };
}

class Order {
  final String orderId;
  final User customer;
  final Address shippingAddress;
  final List<OrderItem> items;
  final OrderStatus status;
  final DateTime createdAt;

  const Order({
    required this.orderId,
    required this.customer,
    required this.shippingAddress,
    required this.items,
    required this.status,
    required this.createdAt,
  });

  double get totalAmount => items.fold(0, (sum, item) => sum + item.totalPrice);

  factory Order.fromJson(Map<String, dynamic> json) => Order(
    orderId: json['order_id'] as String,
    // nested object
    customer: User.fromJson(json['customer'] as Map<String, dynamic>),
    shippingAddress: Address.fromJson(
      json['shipping_address'] as Map<String, dynamic>,
    ),
    // list ของ nested objects
    items: (json['items'] as List<dynamic>)
        .map((item) => OrderItem.fromJson(item as Map<String, dynamic>))
        .toList(),
    status: OrderStatus.fromJson(json['status'] as String),
    createdAt: DateTime.parse(json['created_at'] as String),
  );

  Map<String, dynamic> toJson() => {
    'order_id': orderId,
    'customer': customer.toJson(),
    'shipping_address': shippingAddress.toJson(),
    'items': items.map((item) => item.toJson()).toList(),
    'status': status.toJson(),
    'created_at': createdAt.toIso8601String(),
    'total_amount': totalAmount,
  };
}

enum OrderStatus {
  pending,
  confirmed,
  shipped,
  delivered,
  cancelled;

  factory OrderStatus.fromJson(String value) => OrderStatus.values.firstWhere(
    (e) => e.name == value,
    orElse: () => OrderStatus.pending,
  );

  String toJson() => name;
}
```

---

## ขั้นตอนที่ 174: Lists และ Maps

```dart
// Lists ใน JSON
List<User> parseUserList(String jsonString) {
  final data = jsonDecode(jsonString);

  // กรณี root เป็น array
  if (data is List) {
    return data
        .map((item) => User.fromJson(item as Map<String, dynamic>))
        .toList();
  }

  // กรณีอยู่ใน object
  if (data is Map<String, dynamic>) {
    final list = data['users'] as List<dynamic>? ?? [];
    return list
        .map((item) => User.fromJson(item as Map<String, dynamic>))
        .toList();
  }

  return [];
}

// Map ใน JSON
Map<String, Product> parseProductMap(String jsonString) {
  final data = jsonDecode(jsonString) as Map<String, dynamic>;
  return data.map((key, value) => MapEntry(
    key,
    Product.fromJson(value as Map<String, dynamic>),
  ));
}

// Pagination response
class PaginatedResponse<T> {
  final List<T> items;
  final int total;
  final int page;
  final int perPage;
  final bool hasNextPage;

  const PaginatedResponse({
    required this.items,
    required this.total,
    required this.page,
    required this.perPage,
    required this.hasNextPage,
  });

  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    final items = (json['data'] as List<dynamic>)
        .map((item) => fromJsonT(item as Map<String, dynamic>))
        .toList();

    final total = json['total'] as int;
    final page = json['page'] as int;
    final perPage = json['per_page'] as int;

    return PaginatedResponse(
      items: items,
      total: total,
      page: page,
      perPage: perPage,
      hasNextPage: (page * perPage) < total,
    );
  }
}

// การใช้งาน PaginatedResponse
Future<PaginatedResponse<Product>> getProducts(int page) async {
  // simulate API call
  final jsonData = {
    'data': [
      {'id': 1, 'name': 'สินค้า 1', 'price': 99.0},
      {'id': 2, 'name': 'สินค้า 2', 'price': 149.0},
    ],
    'total': 100,
    'page': page,
    'per_page': 20,
  };

  return PaginatedResponse.fromJson(
    jsonData,
    Product.fromJson,
  );
}
```

---

## ขั้นตอนที่ 175: Null Safety ใน JSON

```dart
// Handle nulls อย่างปลอดภัย
class SafeParser {
  // String
  static String parseString(dynamic value, {String defaultValue = ''}) {
    if (value == null) return defaultValue;
    return value.toString();
  }

  // int
  static int parseInt(dynamic value, {int defaultValue = 0}) {
    if (value == null) return defaultValue;
    if (value is int) return value;
    if (value is double) return value.toInt();
    return int.tryParse(value.toString()) ?? defaultValue;
  }

  // double
  static double parseDouble(dynamic value, {double defaultValue = 0.0}) {
    if (value == null) return defaultValue;
    if (value is double) return value;
    if (value is int) return value.toDouble();
    return double.tryParse(value.toString()) ?? defaultValue;
  }

  // bool
  static bool parseBool(dynamic value, {bool defaultValue = false}) {
    if (value == null) return defaultValue;
    if (value is bool) return value;
    if (value is String) {
      return value.toLowerCase() == 'true' || value == '1';
    }
    if (value is int) return value != 0;
    return defaultValue;
  }

  // DateTime
  static DateTime? parseDateTime(dynamic value) {
    if (value == null) return null;
    if (value is DateTime) return value;
    if (value is String && value.isNotEmpty) {
      return DateTime.tryParse(value);
    }
    if (value is int) {
      return DateTime.fromMillisecondsSinceEpoch(value);
    }
    return null;
  }

  // List
  static List<T> parseList<T>(
    dynamic value,
    T Function(Map<String, dynamic>) fromJson,
  ) {
    if (value == null) return [];
    if (value is! List) return [];
    return value
        .whereType<Map<String, dynamic>>()
        .map(fromJson)
        .toList();
  }
}

// Model ที่ใช้ SafeParser
class UserProfile {
  final int id;
  final String username;
  final String? displayName;
  final String? avatarUrl;
  final int followersCount;
  final bool isVerified;
  final DateTime? lastSeen;
  final List<String> interests;

  const UserProfile({
    required this.id,
    required this.username,
    this.displayName,
    this.avatarUrl,
    required this.followersCount,
    required this.isVerified,
    this.lastSeen,
    required this.interests,
  });

  factory UserProfile.fromJson(Map<String, dynamic> json) => UserProfile(
    id: SafeParser.parseInt(json['id']),
    username: SafeParser.parseString(json['username']),
    displayName: json['display_name'] as String?,
    avatarUrl: json['avatar_url'] as String?,
    followersCount: SafeParser.parseInt(json['followers_count']),
    isVerified: SafeParser.parseBool(json['is_verified']),
    lastSeen: SafeParser.parseDateTime(json['last_seen']),
    interests: (json['interests'] as List<dynamic>?)
            ?.map((e) => e.toString())
            .toList() ??
        [],
  );
}
```

---

## ขั้นตอนที่ 176: json_serializable

```yaml
# pubspec.yaml
dependencies:
  json_annotation: ^4.9.0

dev_dependencies:
  build_runner: ^2.4.8
  json_serializable: ^6.8.0
```

```dart
// lib/models/article.dart
import 'package:json_annotation/json_annotation.dart';

part 'article.g.dart'; // generated file

@JsonSerializable(
  explicitToJson: true, // สำหรับ nested objects
  includeIfNull: false, // ไม่ include null fields
)
class Article {
  final int id;
  final String title;

  @JsonKey(name: 'author_name') // map field ชื่อต่างกัน
  final String authorName;

  @JsonKey(name: 'published_at')
  final DateTime publishedAt;

  @JsonKey(defaultValue: 0) // default value เมื่อ null
  final int viewCount;

  @JsonKey(includeFromJson: false, includeToJson: false) // ignore field
  final bool isBookmarked;

  final ArticleCategory category;
  final List<String> tags;
  final ArticleContent content;

  const Article({
    required this.id,
    required this.title,
    required this.authorName,
    required this.publishedAt,
    required this.viewCount,
    required this.isBookmarked,
    required this.category,
    required this.tags,
    required this.content,
  });

  // Generated factories
  factory Article.fromJson(Map<String, dynamic> json) => _$ArticleFromJson(json);
  Map<String, dynamic> toJson() => _$ArticleToJson(this);
}

enum ArticleCategory {
  @JsonValue('tech') tech,
  @JsonValue('lifestyle') lifestyle,
  @JsonValue('sports') sports,
}

@JsonSerializable()
class ArticleContent {
  final String body;
  final int wordCount;
  final String? summary;

  const ArticleContent({
    required this.body,
    required this.wordCount,
    this.summary,
  });

  factory ArticleContent.fromJson(Map<String, dynamic> json) =>
      _$ArticleContentFromJson(json);
  Map<String, dynamic> toJson() => _$ArticleContentToJson(this);
}
```

```bash
# รัน code generation
flutter pub run build_runner build --delete-conflicting-outputs

# หรือ watch mode สำหรับ development
flutter pub run build_runner watch --delete-conflicting-outputs
```

---

## ขั้นตอนที่ 177: freezed Package

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^2.4.1
  json_annotation: ^4.9.0

dev_dependencies:
  build_runner: ^2.4.8
  freezed: ^2.5.2
  json_serializable: ^6.8.0
```

```dart
// lib/models/weather.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'weather.freezed.dart';
part 'weather.g.dart';

@freezed
class Weather with _$Weather {
  const factory Weather({
    required int id,
    required String city,
    required double temperature,
    required double humidity,
    @Default('') String description,
    WeatherCondition? condition,
    List<HourlyForecast>? hourlyForecast,
  }) = _Weather;

  factory Weather.fromJson(Map<String, dynamic> json) => _$WeatherFromJson(json);
}

@freezed
class HourlyForecast with _$HourlyForecast {
  const factory HourlyForecast({
    required DateTime time,
    required double temperature,
    required String icon,
  }) = _HourlyForecast;

  factory HourlyForecast.fromJson(Map<String, dynamic> json) =>
      _$HourlyForecastFromJson(json);
}

enum WeatherCondition {
  sunny,
  cloudy,
  rainy,
  stormy,
  snowy,
}

// Freezed Union Types (Sealed classes)
@freezed
class ApiState<T> with _$ApiState<T> {
  const factory ApiState.initial() = _Initial;
  const factory ApiState.loading() = _Loading;
  const factory ApiState.success(T data) = _Success;
  const factory ApiState.error(String message, {int? statusCode}) = _Error;
}

// การใช้งาน Union Types
void handleApiState(ApiState<List<Weather>> state) {
  // Pattern matching
  state.when(
    initial: () => print('Initial state'),
    loading: () => print('Loading...'),
    success: (data) => print('Got ${data.length} weather items'),
    error: (message, statusCode) => print('Error: $message'),
  );

  // maybeWhen - handle only some cases
  state.maybeWhen(
    success: (data) => print('Data: $data'),
    orElse: () => print('Not success'),
  );

  // map - transform state
  final newState = state.map(
    initial: (_) => const ApiState<String>.initial(),
    loading: (_) => const ApiState<String>.loading(),
    success: (s) => ApiState<String>.success(s.data.toString()),
    error: (e) => ApiState<String>.error(e.message),
  );
}

// Freezed copyWith
void weatherCopyWith() {
  const weather = Weather(
    id: 1,
    city: 'Bangkok',
    temperature: 35.5,
    humidity: 80,
  );

  // สร้าง copy กับ field ที่เปลี่ยน
  final updatedWeather = weather.copyWith(
    temperature: 33.0,
    description: 'Partly cloudy',
  );

  print(updatedWeather.temperature); // 33.0
  print(updatedWeather.city); // Bangkok (ยังเหมือนเดิม)

  // Deep copy กับ nested
  final withForecast = weather.copyWith(
    hourlyForecast: [
      HourlyForecast(
        time: DateTime.now(),
        temperature: 34.0,
        icon: 'sunny',
      ),
    ],
  );
}
```

---

## ขั้นตอนที่ 178: Code Generation Setup

```dart
// build.yaml - config สำหรับ build_runner
// อยู่ที่ root project

/*
targets:
  $default:
    builders:
      json_serializable:
        options:
          any_map: false
          checked: true
          create_factory: true
          create_to_json: true
          disallow_unrecognized_keys: false
          explicit_to_json: true
          field_rename: snake
          include_if_null: false
*/

// Annotation options
import 'package:json_annotation/json_annotation.dart';

// ใช้ snake_case naming สำหรับ JSON fields อัตโนมัติ
@JsonSerializable(fieldRename: FieldRename.snake)
class UserSettings {
  final bool darkModeEnabled;
  final String notificationSound;
  final int autoSaveInterval;

  const UserSettings({
    required this.darkModeEnabled,
    required this.notificationSound,
    required this.autoSaveInterval,
  });

  // JSON key จะเป็น: dark_mode_enabled, notification_sound, auto_save_interval

  factory UserSettings.fromJson(Map<String, dynamic> json) =>
      _$UserSettingsFromJson(json);
  Map<String, dynamic> toJson() => _$UserSettingsToJson(this);
}

// Custom naming
@JsonSerializable()
class ApiConfig {
  @JsonKey(name: 'api_url')
  final String apiUrl;

  @JsonKey(name: 'max_retry')
  final int maxRetry;

  @JsonKey(name: 'cache_ttl_seconds')
  final int cacheTtlSeconds;

  const ApiConfig({
    required this.apiUrl,
    required this.maxRetry,
    required this.cacheTtlSeconds,
  });

  factory ApiConfig.fromJson(Map<String, dynamic> json) =>
      _$ApiConfigFromJson(json);
  Map<String, dynamic> toJson() => _$ApiConfigToJson(this);
}

// ตัวอย่าง generated code (เพื่อทำความเข้าใจ)
// UserSettings _$UserSettingsFromJson(Map<String, dynamic> json) =>
//     UserSettings(
//       darkModeEnabled: json['dark_mode_enabled'] as bool,
//       notificationSound: json['notification_sound'] as String,
//       autoSaveInterval: json['auto_save_interval'] as int,
//     );
//
// Map<String, dynamic> _$UserSettingsToJson(UserSettings instance) =>
//     <String, dynamic>{
//       'dark_mode_enabled': instance.darkModeEnabled,
//       'notification_sound': instance.notificationSound,
//       'auto_save_interval': instance.autoSaveInterval,
//     };
```

---

## ขั้นตอนที่ 179: Custom Converters

```dart
import 'package:json_annotation/json_annotation.dart';

// Custom converter สำหรับ DateTime format พิเศษ
class DateConverter implements JsonConverter<DateTime, String> {
  const DateConverter();

  @override
  DateTime fromJson(String json) {
    // รองรับหลาย format
    try {
      return DateTime.parse(json);
    } catch (_) {
      // format: dd/MM/yyyy
      final parts = json.split('/');
      if (parts.length == 3) {
        return DateTime(
          int.parse(parts[2]),
          int.parse(parts[1]),
          int.parse(parts[0]),
        );
      }
      throw FormatException('Invalid date: $json');
    }
  }

  @override
  String toJson(DateTime object) => object.toIso8601String();
}

// Custom converter สำหรับ Color
class ColorConverter implements JsonConverter<int, String> {
  const ColorConverter();

  @override
  int fromJson(String json) {
    final hex = json.replaceAll('#', '');
    return int.parse('FF$hex', radix: 16);
  }

  @override
  String toJson(int object) {
    return '#${object.toRadixString(16).substring(2).toUpperCase()}';
  }
}

// Custom converter สำหรับ Enum ที่ซับซ้อน
class StatusConverter implements JsonConverter<UserStatus, int> {
  const StatusConverter();

  @override
  UserStatus fromJson(int json) {
    return switch (json) {
      0 => UserStatus.inactive,
      1 => UserStatus.active,
      2 => UserStatus.suspended,
      _ => UserStatus.unknown,
    };
  }

  @override
  int toJson(UserStatus object) {
    return switch (object) {
      UserStatus.inactive => 0,
      UserStatus.active => 1,
      UserStatus.suspended => 2,
      UserStatus.unknown => -1,
    };
  }
}

enum UserStatus { active, inactive, suspended, unknown }

// การใช้ converters ใน model
@JsonSerializable()
class Event {
  final int id;
  final String title;

  @DateConverter()
  final DateTime startDate;

  @DateConverter()
  final DateTime? endDate;

  @ColorConverter()
  final int themeColor;

  @StatusConverter()
  final UserStatus status;

  const Event({
    required this.id,
    required this.title,
    required this.startDate,
    this.endDate,
    required this.themeColor,
    required this.status,
  });

  factory Event.fromJson(Map<String, dynamic> json) => _$EventFromJson(json);
  Map<String, dynamic> toJson() => _$EventToJson(this);
}
```

---

## ขั้นตอนที่ 180: Error Handling ใน JSON Parsing

```dart
import 'dart:convert';

// Safe JSON parsing
T? safeParseJson<T>(
  String jsonString,
  T Function(Map<String, dynamic>) fromJson,
) {
  try {
    final data = jsonDecode(jsonString);
    if (data is Map<String, dynamic>) {
      return fromJson(data);
    }
    return null;
  } on FormatException catch (e) {
    print('Invalid JSON format: $e');
    return null;
  } on TypeError catch (e) {
    print('Type mismatch: $e');
    return null;
  } catch (e) {
    print('Parsing error: $e');
    return null;
  }
}

// Validated JSON parsing
class JsonParseError {
  final String field;
  final String message;
  final dynamic actualValue;

  const JsonParseError({
    required this.field,
    required this.message,
    this.actualValue,
  });

  @override
  String toString() => 'JsonParseError: $field - $message (got: $actualValue)';
}

class JsonParser {
  final Map<String, dynamic> _json;
  final List<JsonParseError> _errors = [];

  JsonParser(this._json);

  String requireString(String key) {
    final value = _json[key];
    if (value == null) {
      _errors.add(JsonParseError(field: key, message: 'required field missing'));
      return '';
    }
    if (value is! String) {
      _errors.add(JsonParseError(
        field: key,
        message: 'expected String',
        actualValue: value.runtimeType,
      ));
      return value.toString();
    }
    return value;
  }

  int requireInt(String key) {
    final value = _json[key];
    if (value == null) {
      _errors.add(JsonParseError(field: key, message: 'required field missing'));
      return 0;
    }
    if (value is int) return value;
    if (value is String) return int.tryParse(value) ?? 0;
    _errors.add(JsonParseError(
      field: key,
      message: 'expected int',
      actualValue: value,
    ));
    return 0;
  }

  String? optionalString(String key) => _json[key] as String?;
  bool get hasErrors => _errors.isNotEmpty;
  List<JsonParseError> get errors => List.unmodifiable(_errors);
}

// Versioned JSON - handle schema changes
class VersionedConfig {
  final String version;
  final String theme;
  final String language;
  final bool notificationsEnabled;

  const VersionedConfig({
    required this.version,
    required this.theme,
    required this.language,
    required this.notificationsEnabled,
  });

  factory VersionedConfig.fromJson(Map<String, dynamic> json) {
    final version = json['version'] as String? ?? '1.0';

    if (version.startsWith('1.')) {
      // v1 format
      return VersionedConfig(
        version: version,
        theme: json['theme'] as String? ?? 'light',
        language: json['lang'] as String? ?? 'th', // v1 used 'lang'
        notificationsEnabled: json['notifications'] as bool? ?? true,
      );
    } else {
      // v2+ format
      return VersionedConfig(
        version: version,
        theme: json['theme'] as String? ?? 'light',
        language: json['language'] as String? ?? 'th', // v2 uses 'language'
        notificationsEnabled: json['notifications_enabled'] as bool? ?? true,
      );
    }
  }
}
```

---

## Workshop: Product Catalog App

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Models
class Category {
  final int id;
  final String name;
  final String? imageUrl;

  const Category({
    required this.id,
    required this.name,
    this.imageUrl,
  });

  factory Category.fromJson(Map<String, dynamic> json) => Category(
    id: json['id'] as int,
    name: json['name'] as String,
    imageUrl: json['image'] as String?,
  );
}

class ProductImage {
  final String url;
  final bool isPrimary;

  const ProductImage({required this.url, required this.isPrimary});

  factory ProductImage.fromJson(Map<String, dynamic> json) => ProductImage(
    url: json['url'] as String,
    isPrimary: json['isPrimary'] as bool? ?? false,
  );

  Map<String, dynamic> toJson() => {'url': url, 'isPrimary': isPrimary};
}

class CatalogProduct {
  final int id;
  final String title;
  final String description;
  final double price;
  final double discountPercentage;
  final double rating;
  final int stock;
  final String brand;
  final String category;
  final String thumbnail;
  final List<String> images;

  const CatalogProduct({
    required this.id,
    required this.title,
    required this.description,
    required this.price,
    required this.discountPercentage,
    required this.rating,
    required this.stock,
    required this.brand,
    required this.category,
    required this.thumbnail,
    required this.images,
  });

  double get discountedPrice => price * (1 - discountPercentage / 100);

  factory CatalogProduct.fromJson(Map<String, dynamic> json) {
    return CatalogProduct(
      id: json['id'] as int,
      title: json['title'] as String,
      description: json['description'] as String,
      price: (json['price'] as num).toDouble(),
      discountPercentage: (json['discountPercentage'] as num?)?.toDouble() ?? 0,
      rating: (json['rating'] as num?)?.toDouble() ?? 0,
      stock: json['stock'] as int? ?? 0,
      brand: json['brand'] as String? ?? 'Unknown',
      category: json['category'] as String,
      thumbnail: json['thumbnail'] as String,
      images: (json['images'] as List<dynamic>?)
              ?.map((e) => e.toString())
              .toList() ??
          [],
    );
  }
}

class ProductsResponse {
  final List<CatalogProduct> products;
  final int total;
  final int skip;
  final int limit;

  const ProductsResponse({
    required this.products,
    required this.total,
    required this.skip,
    required this.limit,
  });

  factory ProductsResponse.fromJson(Map<String, dynamic> json) {
    return ProductsResponse(
      products: (json['products'] as List<dynamic>)
          .map((p) => CatalogProduct.fromJson(p as Map<String, dynamic>))
          .toList(),
      total: json['total'] as int,
      skip: json['skip'] as int,
      limit: json['limit'] as int,
    );
  }
}

// Repository
class ProductCatalogRepository {
  static const _baseUrl = 'https://dummyjson.com';
  final http.Client _client = http.Client();

  Future<ProductsResponse> getProducts({int skip = 0, int limit = 20}) async {
    final uri = Uri.parse('$_baseUrl/products').replace(
      queryParameters: {
        'skip': skip.toString(),
        'limit': limit.toString(),
      },
    );

    final response = await _client.get(uri);
    if (response.statusCode == 200) {
      return ProductsResponse.fromJson(
        jsonDecode(response.body) as Map<String, dynamic>,
      );
    }
    throw Exception('Failed to load products');
  }

  Future<ProductsResponse> searchProducts(String query) async {
    final uri = Uri.parse('$_baseUrl/products/search').replace(
      queryParameters: {'q': query},
    );

    final response = await _client.get(uri);
    if (response.statusCode == 200) {
      return ProductsResponse.fromJson(
        jsonDecode(response.body) as Map<String, dynamic>,
      );
    }
    throw Exception('Search failed');
  }

  Future<List<String>> getCategories() async {
    final response = await _client.get(
      Uri.parse('$_baseUrl/products/categories'),
    );
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body) as List<dynamic>;
      return data.map((e) => e.toString()).toList();
    }
    throw Exception('Failed to load categories');
  }

  void dispose() => _client.close();
}

// App
class ProductCatalogApp extends StatelessWidget {
  const ProductCatalogApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Product Catalog',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const ProductCatalogScreen(),
    );
  }
}

class ProductCatalogScreen extends StatefulWidget {
  const ProductCatalogScreen({super.key});

  @override
  State<ProductCatalogScreen> createState() => _ProductCatalogScreenState();
}

class _ProductCatalogScreenState extends State<ProductCatalogScreen> {
  final _repo = ProductCatalogRepository();
  final _searchController = TextEditingController();

  List<CatalogProduct> _products = [];
  bool _isLoading = false;
  String? _error;
  bool _isSearching = false;

  @override
  void initState() {
    super.initState();
    _loadProducts();
  }

  @override
  void dispose() {
    _repo.dispose();
    _searchController.dispose();
    super.dispose();
  }

  Future<void> _loadProducts() async {
    setState(() { _isLoading = true; _error = null; });

    try {
      final response = await _repo.getProducts();
      if (mounted) {
        setState(() {
          _products = response.products;
          _isLoading = false;
          _isSearching = false;
        });
      }
    } catch (e) {
      if (mounted) setState(() { _error = e.toString(); _isLoading = false; });
    }
  }

  Future<void> _search(String query) async {
    if (query.trim().isEmpty) {
      _loadProducts();
      return;
    }

    setState(() { _isLoading = true; _error = null; _isSearching = true; });

    try {
      final response = await _repo.searchProducts(query);
      if (mounted) {
        setState(() {
          _products = response.products;
          _isLoading = false;
        });
      }
    } catch (e) {
      if (mounted) setState(() { _error = e.toString(); _isLoading = false; });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Product Catalog'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(56),
          child: Padding(
            padding: const EdgeInsets.fromLTRB(16, 0, 16, 8),
            child: TextField(
              controller: _searchController,
              decoration: InputDecoration(
                hintText: 'ค้นหาสินค้า...',
                prefixIcon: const Icon(Icons.search),
                suffixIcon: _searchController.text.isNotEmpty
                    ? IconButton(
                        icon: const Icon(Icons.clear),
                        onPressed: () {
                          _searchController.clear();
                          _loadProducts();
                        },
                      )
                    : null,
                filled: true,
                fillColor: Colors.white,
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(24),
                  borderSide: BorderSide.none,
                ),
                contentPadding: const EdgeInsets.symmetric(horizontal: 16),
              ),
              onSubmitted: _search,
              onChanged: (value) {
                setState(() {});
                if (value.isEmpty) _loadProducts();
              },
              textInputAction: TextInputAction.search,
            ),
          ),
        ),
      ),
      body: _buildBody(),
    );
  }

  Widget _buildBody() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, color: Colors.red, size: 48),
            const SizedBox(height: 8),
            Text(_error!),
            ElevatedButton(onPressed: _loadProducts, child: const Text('ลองใหม่')),
          ],
        ),
      );
    }

    if (_products.isEmpty) {
      return Center(
        child: Text(_isSearching ? 'ไม่พบสินค้า' : 'ยังไม่มีสินค้า'),
      );
    }

    return GridView.builder(
      padding: const EdgeInsets.all(8),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        childAspectRatio: 0.65,
        crossAxisSpacing: 8,
        mainAxisSpacing: 8,
      ),
      itemCount: _products.length,
      itemBuilder: (context, index) => _buildProductCard(_products[index]),
    );
  }

  Widget _buildProductCard(CatalogProduct product) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: () => _showProductDetail(product),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Image
            AspectRatio(
              aspectRatio: 1,
              child: Image.network(
                product.thumbnail,
                fit: BoxFit.cover,
                errorBuilder: (_, __, ___) => Container(
                  color: Colors.grey[200],
                  child: const Icon(Icons.image_not_supported),
                ),
              ),
            ),

            // Info
            Expanded(
              child: Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      product.title,
                      style: const TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 13,
                      ),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                    const Spacer(),
                    // Rating
                    Row(
                      children: [
                        const Icon(Icons.star, size: 12, color: Colors.amber),
                        Text(
                          ' ${product.rating.toStringAsFixed(1)}',
                          style: const TextStyle(fontSize: 12),
                        ),
                      ],
                    ),
                    // Price
                    if (product.discountPercentage > 0) ...[
                      Text(
                        '฿${product.price.toStringAsFixed(0)}',
                        style: const TextStyle(
                          decoration: TextDecoration.lineThrough,
                          color: Colors.grey,
                          fontSize: 11,
                        ),
                      ),
                      Text(
                        '฿${product.discountedPrice.toStringAsFixed(0)}',
                        style: const TextStyle(
                          color: Colors.red,
                          fontWeight: FontWeight.bold,
                          fontSize: 14,
                        ),
                      ),
                    ] else
                      Text(
                        '฿${product.price.toStringAsFixed(0)}',
                        style: const TextStyle(
                          fontWeight: FontWeight.bold,
                          fontSize: 14,
                        ),
                      ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _showProductDetail(CatalogProduct product) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
      ),
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.7,
        expand: false,
        builder: (context, scrollController) => ListView(
          controller: scrollController,
          padding: const EdgeInsets.all(16),
          children: [
            Center(
              child: Container(
                width: 40,
                height: 4,
                decoration: BoxDecoration(
                  color: Colors.grey[300],
                  borderRadius: BorderRadius.circular(2),
                ),
              ),
            ),
            const SizedBox(height: 16),
            Image.network(product.thumbnail, height: 200, fit: BoxFit.contain),
            const SizedBox(height: 16),
            Text(product.title, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Text(product.brand, style: const TextStyle(color: Colors.grey)),
            const SizedBox(height: 8),
            Row(children: [
              const Icon(Icons.star, size: 16, color: Colors.amber),
              Text(' ${product.rating}  '),
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 2),
                decoration: BoxDecoration(
                  color: product.stock > 0 ? Colors.green : Colors.red,
                  borderRadius: BorderRadius.circular(4),
                ),
                child: Text(
                  product.stock > 0 ? 'มีสินค้า (${product.stock})' : 'สินค้าหมด',
                  style: const TextStyle(color: Colors.white, fontSize: 12),
                ),
              ),
            ]),
            const SizedBox(height: 8),
            Text(product.description, style: const TextStyle(color: Colors.grey)),
            const SizedBox(height: 16),
            Text(
              '฿${product.discountedPrice.toStringAsFixed(2)}',
              style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold, color: Colors.indigo),
            ),
          ],
        ),
      ),
    );
  }
}

void main() => runApp(const ProductCatalogApp());
```

---

## สรุป (Summary)

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| dart:convert | jsonEncode, jsonDecode |
| fromJson/toJson | manual serialization patterns |
| Nested Objects | recursive deserialization |
| Lists & Maps | parsing arrays and maps |
| Null Safety | safe parsing patterns |
| json_serializable | code generation |
| freezed | immutable models, union types |
| Custom Converters | DateTime, Color, Enum |
| Error Handling | safe parsing, validation |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง model สำหรับ `Todo` พร้อม `fromJson` และ `toJson`
2. **ปานกลาง**: ใช้ `json_serializable` สร้าง models ที่มี nested objects และ custom converters
3. **ยาก**: สร้าง app ที่ fetch ข้อมูลจาก API, parse JSON ด้วย `freezed`, และแสดงผลด้วย union types

---

[← Part 17: HTTP & REST APIs](part_17.md) | [Part 19: SQLite & Local Database →](part_19.md)

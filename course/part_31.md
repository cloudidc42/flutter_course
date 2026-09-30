# Part 31: Clean Architecture
## ขั้นตอนที่ 301-310

---

## สารบัญ
1. [Clean Architecture คืออะไร?](#clean-architecture-คืออะไร)
2. [Layers ของ Clean Architecture](#layers-ของ-clean-architecture)
3. [โครงสร้างโฟลเดอร์](#โครงสร้างโฟลเดอร์)
4. [Entities vs Models](#entities-vs-models)
5. [Repository Pattern (Abstract + Concrete)](#repository-pattern)
6. [Use Cases](#use-cases)
7. [Dependency Injection ด้วย get_it](#dependency-injection-ด้วย-get_it)
8. [Dependency Inversion Principle](#dependency-inversion-principle)
9. [การเชื่อมต่อ Presentation Layer](#การเชื่อมต่อ-presentation-layer)
10. [Complete Feature Implementation](#complete-feature-implementation)

---

## ขั้นตอนที่ 301: Clean Architecture คืออะไร?

Clean Architecture เป็นแนวคิดที่คิดค้นโดย Robert C. Martin (Uncle Bob) เพื่อแยกส่วนต่างๆ ของแอปพลิเคชันออกจากกัน ทำให้โค้ดดูแลรักษาง่าย ทดสอบได้ และขยายได้

### หลักการสำคัญ

```
หลักการของ Clean Architecture:
1. Independent of Frameworks   - ไม่ผูกติดกับ Framework ใดๆ
2. Testable                    - ทดสอบได้โดยไม่ต้องมี UI, Database
3. Independent of UI           - UI เปลี่ยนได้โดยไม่กระทบ Business Logic
4. Independent of Database     - เปลี่ยน Database ได้โดยไม่กระทบ Business Rule
5. Independent of External     - Business Rule ไม่รู้จักโลกภายนอก
```

### The Dependency Rule

```
┌─────────────────────────────────────────────────┐
│                                                 │
│    ┌───────────────────────────────────────┐   │
│    │         Presentation Layer            │   │
│    │   (UI, Widgets, ViewModels/Blocs)     │   │
│    └────────────────┬──────────────────────┘   │
│                     │ depends on               │
│    ┌────────────────▼──────────────────────┐   │
│    │           Domain Layer                │   │
│    │  (Entities, Use Cases, Repositories)  │   │
│    └────────────────┬──────────────────────┘   │
│                     │ depends on               │
│    ┌────────────────▼──────────────────────┐   │
│    │            Data Layer                 │   │
│    │  (Models, Repositories Impl, APIs)    │   │
│    └───────────────────────────────────────┘   │
│                                                 │
│    Dependencies flow INWARD only               │
└─────────────────────────────────────────────────┘
```

### ทำไมต้องใช้ Clean Architecture?

```dart
// ❌ แบบไม่ใช้ Clean Architecture - ทุกอย่างปนกัน
class UserScreen extends StatefulWidget {
  @override
  _UserScreenState createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  List<Map<String, dynamic>> users = [];
  bool isLoading = false;

  @override
  void initState() {
    super.initState();
    _loadUsers();
  }

  Future<void> _loadUsers() async {
    setState(() => isLoading = true);
    
    // Business Logic ปนกับ UI
    final response = await http.get(Uri.parse('https://api.example.com/users'));
    final data = json.decode(response.body);
    
    // Data Transformation ปนกับ UI
    setState(() {
      users = List<Map<String, dynamic>>.from(data['users']);
      isLoading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    // UI ปนกับ Logic
    return ListView.builder(
      itemCount: users.length,
      itemBuilder: (_, i) => ListTile(title: Text(users[i]['name'])),
    );
  }
}

// ✅ แบบใช้ Clean Architecture - แยกส่วนชัดเจน
// แต่ละ Layer มีหน้าที่ของตัวเอง
```

---

## ขั้นตอนที่ 302: Layers ของ Clean Architecture

### Domain Layer (ชั้นกลาง - ไม่รู้จักภายนอก)

Domain Layer คือหัวใจของแอปพลิเคชัน ประกอบด้วย:
- **Entities**: Business Objects
- **Use Cases**: Business Rules / Application Logic
- **Repository Interfaces**: Contracts สำหรับ Data Layer

```dart
// lib/features/user/domain/entities/user.dart
// Entity - Pure Dart, ไม่มี dependency ใดๆ
class User {
  final String id;
  final String name;
  final String email;
  final DateTime createdAt;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.createdAt,
  });

  // Business Logic ที่เกี่ยวกับ User
  bool get isVerified => email.contains('@');
  
  String get displayName => name.isEmpty ? email.split('@').first : name;

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is User && runtimeType == other.runtimeType && id == other.id;

  @override
  int get hashCode => id.hashCode;

  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';
}
```

```dart
// lib/features/user/domain/repositories/user_repository.dart
// Repository Interface - กำหนด Contract
import 'package:dartz/dartz.dart';
import '../entities/user.dart';
import '../../../../core/error/failures.dart';

abstract class UserRepository {
  Future<Either<Failure, List<User>>> getUsers();
  Future<Either<Failure, User>> getUserById(String id);
  Future<Either<Failure, User>> createUser({
    required String name,
    required String email,
  });
  Future<Either<Failure, User>> updateUser(User user);
  Future<Either<Failure, void>> deleteUser(String id);
}
```

### Data Layer (ชั้นนอกสุด)

```dart
// lib/features/user/data/models/user_model.dart
// Model - มี logic สำหรับ JSON serialization
import '../../domain/entities/user.dart';

class UserModel extends User {
  const UserModel({
    required String id,
    required String name,
    required String email,
    required DateTime createdAt,
  }) : super(
          id: id,
          name: name,
          email: email,
          createdAt: createdAt,
        );

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String,
      name: json['name'] as String,
      email: json['email'] as String,
      createdAt: DateTime.parse(json['created_at'] as String),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'created_at': createdAt.toIso8601String(),
    };
  }

  factory UserModel.fromEntity(User user) {
    return UserModel(
      id: user.id,
      name: user.name,
      email: user.email,
      createdAt: user.createdAt,
    );
  }
}
```

---

## ขั้นตอนที่ 303: โครงสร้างโฟลเดอร์

```
lib/
├── core/
│   ├── error/
│   │   ├── exceptions.dart       # Custom Exceptions
│   │   └── failures.dart         # Failure classes (Either pattern)
│   ├── network/
│   │   ├── network_info.dart     # Network connectivity check
│   │   └── api_client.dart       # HTTP client wrapper
│   ├── usecases/
│   │   └── usecase.dart          # Base UseCase interface
│   ├── utils/
│   │   ├── constants.dart        # App constants
│   │   └── extensions.dart       # Dart extensions
│   └── injection_container.dart  # Dependency injection setup
│
├── features/
│   ├── user/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── user_local_datasource.dart
│   │   │   │   └── user_remote_datasource.dart
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   └── repositories/
│   │   │       └── user_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── user_repository.dart
│   │   │   └── usecases/
│   │   │       ├── get_users.dart
│   │   │       ├── get_user_by_id.dart
│   │   │       ├── create_user.dart
│   │   │       ├── update_user.dart
│   │   │       └── delete_user.dart
│   │   └── presentation/
│   │       ├── bloc/
│   │       │   ├── user_bloc.dart
│   │       │   ├── user_event.dart
│   │       │   └── user_state.dart
│   │       ├── pages/
│   │       │   ├── user_list_page.dart
│   │       │   └── user_detail_page.dart
│   │       └── widgets/
│   │           ├── user_card.dart
│   │           └── user_form.dart
│   │
│   └── post/                     # Feature อื่นๆ มีโครงสร้างเหมือนกัน
│       ├── data/
│       ├── domain/
│       └── presentation/
│
└── main.dart
```

### การตั้งค่า pubspec.yaml

```yaml
name: clean_arch_app
description: Flutter app with Clean Architecture

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  
  # State Management
  flutter_bloc: ^8.1.3
  
  # Dependency Injection
  get_it: ^7.6.4
  injectable: ^2.3.2
  
  # Functional Programming (Either)
  dartz: ^0.10.1
  
  # Network
  dio: ^5.3.2
  
  # Local Storage
  shared_preferences: ^2.2.1
  
  # Connectivity
  connectivity_plus: ^5.0.1
  
  # Freezed (for immutable classes)
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  
  # Code Generation
  build_runner: ^2.4.7
  injectable_generator: ^2.4.1
  freezed: ^2.4.5
  json_serializable: ^6.7.1
  
  # Testing
  mockito: ^5.4.3
  bloc_test: ^9.1.5
```

---

## ขั้นตอนที่ 304: Core Layer Setup

```dart
// lib/core/error/failures.dart
import 'package:equatable/equatable.dart';

abstract class Failure extends Equatable {
  final String message;
  const Failure(this.message);
  
  @override
  List<Object> get props => [message];
}

class ServerFailure extends Failure {
  const ServerFailure(String message) : super(message);
}

class NetworkFailure extends Failure {
  const NetworkFailure(String message) : super(message);
}

class CacheFailure extends Failure {
  const CacheFailure(String message) : super(message);
}

class NotFoundFailure extends Failure {
  const NotFoundFailure(String message) : super(message);
}

class ValidationFailure extends Failure {
  const ValidationFailure(String message) : super(message);
}
```

```dart
// lib/core/error/exceptions.dart
class ServerException implements Exception {
  final String message;
  final int? statusCode;
  
  const ServerException({required this.message, this.statusCode});
  
  @override
  String toString() => 'ServerException: $message (status: $statusCode)';
}

class NetworkException implements Exception {
  final String message;
  const NetworkException({required this.message});
}

class CacheException implements Exception {
  final String message;
  const CacheException({required this.message});
}

class NotFoundException implements Exception {
  final String message;
  const NotFoundException({required this.message});
}
```

```dart
// lib/core/usecases/usecase.dart
import 'package:dartz/dartz.dart';
import '../error/failures.dart';

// Base UseCase สำหรับ Use Cases ที่ต้องการ Parameters
abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

// Base UseCase สำหรับ Use Cases ที่ไม่ต้องการ Parameters
abstract class UseCaseNoParams<Type> {
  Future<Either<Failure, Type>> call();
}

// NoParams class สำหรับ Use Cases ที่ไม่ต้องการ params
class NoParams {
  const NoParams();
}
```

```dart
// lib/core/network/network_info.dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:injectable/injectable.dart';

abstract class NetworkInfo {
  Future<bool> get isConnected;
}

@LazySingleton(as: NetworkInfo)
class NetworkInfoImpl implements NetworkInfo {
  final Connectivity connectivity;

  NetworkInfoImpl(this.connectivity);

  @override
  Future<bool> get isConnected async {
    final result = await connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

---

## ขั้นตอนที่ 305: Entities vs Models

การแยก Entity กับ Model เป็นสิ่งสำคัญมากใน Clean Architecture

```dart
// Entity - Domain Layer
// - Pure Dart class
// - ไม่มี JSON serialization
// - มี Business Logic
// - Immutable

// lib/features/product/domain/entities/product.dart
class Product {
  final String id;
  final String name;
  final double price;
  final int stockQuantity;
  final String category;
  final List<String> tags;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.stockQuantity,
    required this.category,
    required this.tags,
  });

  // Business Logic
  bool get isInStock => stockQuantity > 0;
  bool get isLowStock => stockQuantity > 0 && stockQuantity <= 5;
  bool get isOutOfStock => stockQuantity == 0;
  
  double get discountedPrice => tags.contains('sale') ? price * 0.9 : price;
  
  bool hasTag(String tag) => tags.contains(tag.toLowerCase());
  
  Product copyWith({
    String? id,
    String? name,
    double? price,
    int? stockQuantity,
    String? category,
    List<String>? tags,
  }) {
    return Product(
      id: id ?? this.id,
      name: name ?? this.name,
      price: price ?? this.price,
      stockQuantity: stockQuantity ?? this.stockQuantity,
      category: category ?? this.category,
      tags: tags ?? this.tags,
    );
  }

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Product && runtimeType == other.runtimeType && id == other.id;

  @override
  int get hashCode => id.hashCode;

  @override
  String toString() => 'Product(id: $id, name: $name, price: $price)';
}
```

```dart
// Model - Data Layer
// - Extends Entity (หรือ Map จาก Entity)
// - มี JSON serialization
// - มี fromJson/toJson

// lib/features/product/data/models/product_model.dart
import '../../domain/entities/product.dart';

class ProductModel extends Product {
  const ProductModel({
    required String id,
    required String name,
    required double price,
    required int stockQuantity,
    required String category,
    required List<String> tags,
  }) : super(
          id: id,
          name: name,
          price: price,
          stockQuantity: stockQuantity,
          category: category,
          tags: tags,
        );

  factory ProductModel.fromJson(Map<String, dynamic> json) {
    return ProductModel(
      id: json['id'] as String,
      name: json['name'] as String,
      price: (json['price'] as num).toDouble(),
      stockQuantity: json['stock_quantity'] as int,
      category: json['category'] as String,
      tags: List<String>.from(json['tags'] as List),
    );
  }

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
    'stock_quantity': stockQuantity,
    'category': category,
    'tags': tags,
  };

  // แปลงจาก Entity เป็น Model
  factory ProductModel.fromEntity(Product product) => ProductModel(
    id: product.id,
    name: product.name,
    price: product.price,
    stockQuantity: product.stockQuantity,
    category: product.category,
    tags: product.tags,
  );

  // สร้าง Empty Model สำหรับ Testing
  factory ProductModel.empty() => const ProductModel(
    id: '',
    name: '',
    price: 0.0,
    stockQuantity: 0,
    category: '',
    tags: [],
  );
}
```

---

## ขั้นตอนที่ 306: Repository Pattern

### Abstract Repository (Domain Layer)

```dart
// lib/features/product/domain/repositories/product_repository.dart
import 'package:dartz/dartz.dart';
import '../entities/product.dart';
import '../../../../core/error/failures.dart';

abstract class ProductRepository {
  // อ่านข้อมูล
  Future<Either<Failure, List<Product>>> getProducts({
    String? category,
    int page = 1,
    int limit = 20,
  });
  
  Future<Either<Failure, Product>> getProductById(String id);
  
  Future<Either<Failure, List<Product>>> searchProducts(String query);
  
  // เขียนข้อมูล
  Future<Either<Failure, Product>> createProduct(Product product);
  Future<Either<Failure, Product>> updateProduct(Product product);
  Future<Either<Failure, void>> deleteProduct(String id);
  
  // Cache Management
  Future<Either<Failure, void>> clearCache();
}
```

### Remote Data Source

```dart
// lib/features/product/data/datasources/product_remote_datasource.dart
import 'package:dio/dio.dart';
import 'package:injectable/injectable.dart';
import '../models/product_model.dart';
import '../../../../core/error/exceptions.dart';

abstract class ProductRemoteDataSource {
  Future<List<ProductModel>> getProducts({String? category, int page = 1, int limit = 20});
  Future<ProductModel> getProductById(String id);
  Future<List<ProductModel>> searchProducts(String query);
  Future<ProductModel> createProduct(Map<String, dynamic> productData);
  Future<ProductModel> updateProduct(String id, Map<String, dynamic> productData);
  Future<void> deleteProduct(String id);
}

@LazySingleton(as: ProductRemoteDataSource)
class ProductRemoteDataSourceImpl implements ProductRemoteDataSource {
  final Dio _dio;

  ProductRemoteDataSourceImpl(this._dio);

  @override
  Future<List<ProductModel>> getProducts({
    String? category,
    int page = 1,
    int limit = 20,
  }) async {
    try {
      final queryParams = <String, dynamic>{
        'page': page,
        'limit': limit,
        if (category != null) 'category': category,
      };

      final response = await _dio.get(
        '/products',
        queryParameters: queryParams,
      );

      if (response.statusCode == 200) {
        final List<dynamic> data = response.data['products'];
        return data.map((json) => ProductModel.fromJson(json)).toList();
      }
      
      throw ServerException(
        message: 'Failed to load products',
        statusCode: response.statusCode,
      );
    } on DioException catch (e) {
      throw ServerException(
        message: e.message ?? 'Network error',
        statusCode: e.response?.statusCode,
      );
    }
  }

  @override
  Future<ProductModel> getProductById(String id) async {
    try {
      final response = await _dio.get('/products/$id');
      
      if (response.statusCode == 200) {
        return ProductModel.fromJson(response.data);
      }
      
      if (response.statusCode == 404) {
        throw NotFoundException(message: 'Product not found: $id');
      }
      
      throw ServerException(
        message: 'Failed to get product',
        statusCode: response.statusCode,
      );
    } on DioException catch (e) {
      throw ServerException(
        message: e.message ?? 'Network error',
        statusCode: e.response?.statusCode,
      );
    }
  }

  @override
  Future<List<ProductModel>> searchProducts(String query) async {
    try {
      final response = await _dio.get(
        '/products/search',
        queryParameters: {'q': query},
      );

      if (response.statusCode == 200) {
        final List<dynamic> data = response.data['products'];
        return data.map((json) => ProductModel.fromJson(json)).toList();
      }

      throw ServerException(
        message: 'Search failed',
        statusCode: response.statusCode,
      );
    } on DioException catch (e) {
      throw ServerException(
        message: e.message ?? 'Network error',
      );
    }
  }

  @override
  Future<ProductModel> createProduct(Map<String, dynamic> productData) async {
    try {
      final response = await _dio.post('/products', data: productData);
      
      if (response.statusCode == 201) {
        return ProductModel.fromJson(response.data);
      }
      
      throw ServerException(
        message: 'Failed to create product',
        statusCode: response.statusCode,
      );
    } on DioException catch (e) {
      throw ServerException(message: e.message ?? 'Network error');
    }
  }

  @override
  Future<ProductModel> updateProduct(
    String id,
    Map<String, dynamic> productData,
  ) async {
    try {
      final response = await _dio.put('/products/$id', data: productData);
      
      if (response.statusCode == 200) {
        return ProductModel.fromJson(response.data);
      }
      
      throw ServerException(
        message: 'Failed to update product',
        statusCode: response.statusCode,
      );
    } on DioException catch (e) {
      throw ServerException(message: e.message ?? 'Network error');
    }
  }

  @override
  Future<void> deleteProduct(String id) async {
    try {
      final response = await _dio.delete('/products/$id');
      
      if (response.statusCode != 200 && response.statusCode != 204) {
        throw ServerException(
          message: 'Failed to delete product',
          statusCode: response.statusCode,
        );
      }
    } on DioException catch (e) {
      throw ServerException(message: e.message ?? 'Network error');
    }
  }
}
```

### Local Data Source (Cache)

```dart
// lib/features/product/data/datasources/product_local_datasource.dart
import 'dart:convert';
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';
import '../models/product_model.dart';
import '../../../../core/error/exceptions.dart';

abstract class ProductLocalDataSource {
  Future<List<ProductModel>> getCachedProducts();
  Future<void> cacheProducts(List<ProductModel> products);
  Future<void> clearCache();
}

@LazySingleton(as: ProductLocalDataSource)
class ProductLocalDataSourceImpl implements ProductLocalDataSource {
  final SharedPreferences _prefs;
  static const String _cachedProductsKey = 'CACHED_PRODUCTS';
  static const Duration _cacheExpiry = Duration(hours: 1);
  static const String _cacheTimeKey = 'CACHED_PRODUCTS_TIME';

  ProductLocalDataSourceImpl(this._prefs);

  @override
  Future<List<ProductModel>> getCachedProducts() async {
    // ตรวจสอบว่า Cache หมดอายุหรือยัง
    final cacheTimeStr = _prefs.getString(_cacheTimeKey);
    if (cacheTimeStr != null) {
      final cacheTime = DateTime.parse(cacheTimeStr);
      if (DateTime.now().difference(cacheTime) > _cacheExpiry) {
        await clearCache();
        throw const CacheException(message: 'Cache expired');
      }
    }

    final jsonStr = _prefs.getString(_cachedProductsKey);
    if (jsonStr == null) {
      throw const CacheException(message: 'No cached products');
    }

    try {
      final List<dynamic> jsonList = json.decode(jsonStr);
      return jsonList.map((json) => ProductModel.fromJson(json)).toList();
    } catch (e) {
      throw const CacheException(message: 'Failed to decode cached products');
    }
  }

  @override
  Future<void> cacheProducts(List<ProductModel> products) async {
    final jsonStr = json.encode(products.map((p) => p.toJson()).toList());
    await _prefs.setString(_cachedProductsKey, jsonStr);
    await _prefs.setString(_cacheTimeKey, DateTime.now().toIso8601String());
  }

  @override
  Future<void> clearCache() async {
    await _prefs.remove(_cachedProductsKey);
    await _prefs.remove(_cacheTimeKey);
  }
}
```

### Repository Implementation

```dart
// lib/features/product/data/repositories/product_repository_impl.dart
import 'package:dartz/dartz.dart';
import 'package:injectable/injectable.dart';
import '../../domain/entities/product.dart';
import '../../domain/repositories/product_repository.dart';
import '../datasources/product_local_datasource.dart';
import '../datasources/product_remote_datasource.dart';
import '../models/product_model.dart';
import '../../../../core/error/exceptions.dart';
import '../../../../core/error/failures.dart';
import '../../../../core/network/network_info.dart';

@LazySingleton(as: ProductRepository)
class ProductRepositoryImpl implements ProductRepository {
  final ProductRemoteDataSource _remoteDataSource;
  final ProductLocalDataSource _localDataSource;
  final NetworkInfo _networkInfo;

  ProductRepositoryImpl(
    this._remoteDataSource,
    this._localDataSource,
    this._networkInfo,
  );

  @override
  Future<Either<Failure, List<Product>>> getProducts({
    String? category,
    int page = 1,
    int limit = 20,
  }) async {
    if (await _networkInfo.isConnected) {
      try {
        final products = await _remoteDataSource.getProducts(
          category: category,
          page: page,
          limit: limit,
        );
        
        // Cache เฉพาะ page แรก
        if (page == 1) {
          await _localDataSource.cacheProducts(products);
        }
        
        return Right(products);
      } on ServerException catch (e) {
        return Left(ServerFailure(e.message));
      }
    } else {
      // Offline - ดึงจาก Cache
      try {
        final cachedProducts = await _localDataSource.getCachedProducts();
        return Right(cachedProducts);
      } on CacheException catch (e) {
        return Left(CacheFailure(e.message));
      }
    }
  }

  @override
  Future<Either<Failure, Product>> getProductById(String id) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('No internet connection'));
    }

    try {
      final product = await _remoteDataSource.getProductById(id);
      return Right(product);
    } on NotFoundException catch (e) {
      return Left(NotFoundFailure(e.message));
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }

  @override
  Future<Either<Failure, List<Product>>> searchProducts(String query) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('No internet connection'));
    }

    try {
      final products = await _remoteDataSource.searchProducts(query);
      return Right(products);
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }

  @override
  Future<Either<Failure, Product>> createProduct(Product product) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('No internet connection'));
    }

    try {
      final model = ProductModel.fromEntity(product);
      final createdProduct = await _remoteDataSource.createProduct(model.toJson());
      return Right(createdProduct);
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }

  @override
  Future<Either<Failure, Product>> updateProduct(Product product) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('No internet connection'));
    }

    try {
      final model = ProductModel.fromEntity(product);
      final updatedProduct = await _remoteDataSource.updateProduct(
        product.id,
        model.toJson(),
      );
      return Right(updatedProduct);
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }

  @override
  Future<Either<Failure, void>> deleteProduct(String id) async {
    if (!await _networkInfo.isConnected) {
      return const Left(NetworkFailure('No internet connection'));
    }

    try {
      await _remoteDataSource.deleteProduct(id);
      return const Right(null);
    } on ServerException catch (e) {
      return Left(ServerFailure(e.message));
    }
  }

  @override
  Future<Either<Failure, void>> clearCache() async {
    try {
      await _localDataSource.clearCache();
      return const Right(null);
    } on CacheException catch (e) {
      return Left(CacheFailure(e.message));
    }
  }
}
```

---

## ขั้นตอนที่ 307: Use Cases

Use Cases คือ Application Business Rules - แต่ละ Use Case ทำสิ่งเดียว (Single Responsibility)

```dart
// lib/features/product/domain/usecases/get_products.dart
import 'package:dartz/dartz.dart';
import 'package:equatable/equatable.dart';
import 'package:injectable/injectable.dart';
import '../entities/product.dart';
import '../repositories/product_repository.dart';
import '../../../../core/error/failures.dart';
import '../../../../core/usecases/usecase.dart';

@lazySingleton
class GetProducts implements UseCase<List<Product>, GetProductsParams> {
  final ProductRepository _repository;

  GetProducts(this._repository);

  @override
  Future<Either<Failure, List<Product>>> call(GetProductsParams params) {
    return _repository.getProducts(
      category: params.category,
      page: params.page,
      limit: params.limit,
    );
  }
}

class GetProductsParams extends Equatable {
  final String? category;
  final int page;
  final int limit;

  const GetProductsParams({
    this.category,
    this.page = 1,
    this.limit = 20,
  });

  @override
  List<Object?> get props => [category, page, limit];
}
```

```dart
// lib/features/product/domain/usecases/get_product_by_id.dart
import 'package:dartz/dartz.dart';
import 'package:equatable/equatable.dart';
import 'package:injectable/injectable.dart';
import '../entities/product.dart';
import '../repositories/product_repository.dart';
import '../../../../core/error/failures.dart';
import '../../../../core/usecases/usecase.dart';

@lazySingleton
class GetProductById implements UseCase<Product, GetProductByIdParams> {
  final ProductRepository _repository;

  GetProductById(this._repository);

  @override
  Future<Either<Failure, Product>> call(GetProductByIdParams params) {
    return _repository.getProductById(params.id);
  }
}

class GetProductByIdParams extends Equatable {
  final String id;
  const GetProductByIdParams(this.id);

  @override
  List<Object> get props => [id];
}
```

```dart
// lib/features/product/domain/usecases/search_products.dart
import 'package:dartz/dartz.dart';
import 'package:equatable/equatable.dart';
import 'package:injectable/injectable.dart';
import '../entities/product.dart';
import '../repositories/product_repository.dart';
import '../../../../core/error/failures.dart';
import '../../../../core/usecases/usecase.dart';

@lazySingleton
class SearchProducts implements UseCase<List<Product>, SearchProductsParams> {
  final ProductRepository _repository;

  SearchProducts(this._repository);

  @override
  Future<Either<Failure, List<Product>>> call(SearchProductsParams params) async {
    // Validation ใน Use Case
    if (params.query.trim().isEmpty) {
      return const Left(ValidationFailure('Search query cannot be empty'));
    }
    
    if (params.query.length < 2) {
      return const Left(ValidationFailure('Search query must be at least 2 characters'));
    }

    return _repository.searchProducts(params.query.trim());
  }
}

class SearchProductsParams extends Equatable {
  final String query;
  const SearchProductsParams(this.query);

  @override
  List<Object> get props => [query];
}
```

---

## ขั้นตอนที่ 308: Dependency Injection ด้วย get_it

```dart
// lib/core/injection_container.dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:dio/dio.dart';
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'injection_container.config.dart';

final getIt = GetIt.instance;

@InjectableInit(
  initializerName: r'$initGetIt',
  preferRelativeImports: true,
  asExtension: false,
)
Future<void> configureDependencies() async => $initGetIt(getIt);

// ถ้าไม่ใช้ injectable, ทำ manual setup แบบนี้
Future<void> setupDependencies() async {
  // External
  final sharedPreferences = await SharedPreferences.getInstance();
  getIt.registerLazySingleton<SharedPreferences>(() => sharedPreferences);
  
  getIt.registerLazySingleton<Connectivity>(() => Connectivity());
  
  getIt.registerLazySingleton<Dio>(() {
    final dio = Dio(
      BaseOptions(
        baseUrl: 'https://api.example.com',
        connectTimeout: const Duration(seconds: 30),
        receiveTimeout: const Duration(seconds: 30),
        headers: {'Content-Type': 'application/json'},
      ),
    );
    
    // Interceptors
    dio.interceptors.add(LogInterceptor(
      requestBody: true,
      responseBody: true,
    ));
    
    return dio;
  });

  // Core
  getIt.registerLazySingleton<NetworkInfo>(
    () => NetworkInfoImpl(getIt<Connectivity>()),
  );

  // Data Sources
  getIt.registerLazySingleton<ProductRemoteDataSource>(
    () => ProductRemoteDataSourceImpl(getIt<Dio>()),
  );
  
  getIt.registerLazySingleton<ProductLocalDataSource>(
    () => ProductLocalDataSourceImpl(getIt<SharedPreferences>()),
  );

  // Repositories
  getIt.registerLazySingleton<ProductRepository>(
    () => ProductRepositoryImpl(
      getIt<ProductRemoteDataSource>(),
      getIt<ProductLocalDataSource>(),
      getIt<NetworkInfo>(),
    ),
  );

  // Use Cases
  getIt.registerLazySingleton(() => GetProducts(getIt<ProductRepository>()));
  getIt.registerLazySingleton(() => GetProductById(getIt<ProductRepository>()));
  getIt.registerLazySingleton(() => SearchProducts(getIt<ProductRepository>()));
  getIt.registerLazySingleton(() => CreateProduct(getIt<ProductRepository>()));
  getIt.registerLazySingleton(() => UpdateProduct(getIt<ProductRepository>()));
  getIt.registerLazySingleton(() => DeleteProduct(getIt<ProductRepository>()));

  // BLoCs/Cubits
  getIt.registerFactory(
    () => ProductBloc(
      getProducts: getIt<GetProducts>(),
      getProductById: getIt<GetProductById>(),
      searchProducts: getIt<SearchProducts>(),
    ),
  );
}
```

---

## ขั้นตอนที่ 309: Presentation Layer - Bloc

```dart
// lib/features/product/presentation/bloc/product_event.dart
import 'package:equatable/equatable.dart';

abstract class ProductEvent extends Equatable {
  const ProductEvent();

  @override
  List<Object?> get props => [];
}

class LoadProductsEvent extends ProductEvent {
  final String? category;
  final bool refresh;

  const LoadProductsEvent({this.category, this.refresh = false});

  @override
  List<Object?> get props => [category, refresh];
}

class LoadMoreProductsEvent extends ProductEvent {
  const LoadMoreProductsEvent();
}

class GetProductByIdEvent extends ProductEvent {
  final String id;
  const GetProductByIdEvent(this.id);

  @override
  List<Object> get props => [id];
}

class SearchProductsEvent extends ProductEvent {
  final String query;
  const SearchProductsEvent(this.query);

  @override
  List<Object> get props => [query];
}

class ClearSearchEvent extends ProductEvent {
  const ClearSearchEvent();
}
```

```dart
// lib/features/product/presentation/bloc/product_state.dart
import 'package:equatable/equatable.dart';
import '../../domain/entities/product.dart';

abstract class ProductState extends Equatable {
  const ProductState();

  @override
  List<Object?> get props => [];
}

class ProductInitial extends ProductState {
  const ProductInitial();
}

class ProductLoading extends ProductState {
  const ProductLoading();
}

class ProductLoaded extends ProductState {
  final List<Product> products;
  final bool hasMore;
  final int currentPage;
  final String? searchQuery;

  const ProductLoaded({
    required this.products,
    this.hasMore = false,
    this.currentPage = 1,
    this.searchQuery,
  });

  ProductLoaded copyWith({
    List<Product>? products,
    bool? hasMore,
    int? currentPage,
    String? searchQuery,
  }) {
    return ProductLoaded(
      products: products ?? this.products,
      hasMore: hasMore ?? this.hasMore,
      currentPage: currentPage ?? this.currentPage,
      searchQuery: searchQuery ?? this.searchQuery,
    );
  }

  @override
  List<Object?> get props => [products, hasMore, currentPage, searchQuery];
}

class ProductDetailLoaded extends ProductState {
  final Product product;
  const ProductDetailLoaded(this.product);

  @override
  List<Object> get props => [product];
}

class ProductError extends ProductState {
  final String message;
  const ProductError(this.message);

  @override
  List<Object> get props => [message];
}

class ProductLoadingMore extends ProductLoaded {
  const ProductLoadingMore({
    required super.products,
    required super.hasMore,
    required super.currentPage,
  });
}
```

```dart
// lib/features/product/presentation/bloc/product_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../domain/usecases/get_products.dart';
import '../../domain/usecases/get_product_by_id.dart';
import '../../domain/usecases/search_products.dart';
import 'product_event.dart';
import 'product_state.dart';
import 'package:injectable/injectable.dart';
import 'dart:async';

@injectable
class ProductBloc extends Bloc<ProductEvent, ProductState> {
  final GetProducts _getProducts;
  final GetProductById _getProductById;
  final SearchProducts _searchProducts;
  
  Timer? _searchDebounce;

  ProductBloc({
    required GetProducts getProducts,
    required GetProductById getProductById,
    required SearchProducts searchProducts,
  })  : _getProducts = getProducts,
        _getProductById = getProductById,
        _searchProducts = searchProducts,
        super(const ProductInitial()) {
    on<LoadProductsEvent>(_onLoadProducts);
    on<LoadMoreProductsEvent>(_onLoadMoreProducts);
    on<GetProductByIdEvent>(_onGetProductById);
    on<SearchProductsEvent>(_onSearchProducts);
    on<ClearSearchEvent>(_onClearSearch);
  }

  Future<void> _onLoadProducts(
    LoadProductsEvent event,
    Emitter<ProductState> emit,
  ) async {
    if (!event.refresh && state is ProductLoaded) return;
    
    emit(const ProductLoading());

    final result = await _getProducts(
      GetProductsParams(category: event.category, page: 1),
    );

    result.fold(
      (failure) => emit(ProductError(failure.message)),
      (products) => emit(ProductLoaded(
        products: products,
        hasMore: products.length == 20,
        currentPage: 1,
      )),
    );
  }

  Future<void> _onLoadMoreProducts(
    LoadMoreProductsEvent event,
    Emitter<ProductState> emit,
  ) async {
    if (state is! ProductLoaded) return;
    
    final currentState = state as ProductLoaded;
    if (!currentState.hasMore) return;

    emit(ProductLoadingMore(
      products: currentState.products,
      hasMore: currentState.hasMore,
      currentPage: currentState.currentPage,
    ));

    final nextPage = currentState.currentPage + 1;
    final result = await _getProducts(GetProductsParams(page: nextPage));

    result.fold(
      (failure) => emit(ProductError(failure.message)),
      (newProducts) => emit(currentState.copyWith(
        products: [...currentState.products, ...newProducts],
        hasMore: newProducts.length == 20,
        currentPage: nextPage,
      )),
    );
  }

  Future<void> _onGetProductById(
    GetProductByIdEvent event,
    Emitter<ProductState> emit,
  ) async {
    emit(const ProductLoading());

    final result = await _getProductById(GetProductByIdParams(event.id));

    result.fold(
      (failure) => emit(ProductError(failure.message)),
      (product) => emit(ProductDetailLoaded(product)),
    );
  }

  Future<void> _onSearchProducts(
    SearchProductsEvent event,
    Emitter<ProductState> emit,
  ) async {
    _searchDebounce?.cancel();
    _searchDebounce = Timer(const Duration(milliseconds: 500), () async {
      add(SearchProductsEvent(event.query));
    });
    
    if (event.query.isEmpty) {
      add(const LoadProductsEvent());
      return;
    }

    emit(const ProductLoading());

    final result = await _searchProducts(SearchProductsParams(event.query));

    result.fold(
      (failure) => emit(ProductError(failure.message)),
      (products) => emit(ProductLoaded(
        products: products,
        searchQuery: event.query,
      )),
    );
  }

  Future<void> _onClearSearch(
    ClearSearchEvent event,
    Emitter<ProductState> emit,
  ) async {
    add(const LoadProductsEvent(refresh: true));
  }

  @override
  Future<void> close() {
    _searchDebounce?.cancel();
    return super.close();
  }
}
```

---

## ขั้นตอนที่ 310: Workshop - Complete Feature Implementation

ลองสร้าง Feature "Todo" แบบสมบูรณ์ด้วย Clean Architecture:

```dart
// WORKSHOP: Todo Feature - Clean Architecture

// 1. Entity
// lib/features/todo/domain/entities/todo.dart
class Todo {
  final String id;
  final String title;
  final String description;
  final bool isCompleted;
  final DateTime createdAt;
  final DateTime? completedAt;
  final String userId;
  final int priority; // 1=Low, 2=Medium, 3=High

  const Todo({
    required this.id,
    required this.title,
    required this.description,
    required this.isCompleted,
    required this.createdAt,
    this.completedAt,
    required this.userId,
    this.priority = 2,
  });

  bool get isHighPriority => priority == 3;
  bool get isOverdue => !isCompleted && 
    createdAt.difference(DateTime.now()).inDays < -7;

  Todo complete() => copyWith(
    isCompleted: true,
    completedAt: DateTime.now(),
  );

  Todo copyWith({
    String? id,
    String? title,
    String? description,
    bool? isCompleted,
    DateTime? createdAt,
    DateTime? completedAt,
    String? userId,
    int? priority,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
      completedAt: completedAt ?? this.completedAt,
      userId: userId ?? this.userId,
      priority: priority ?? this.priority,
    );
  }
}
```

```dart
// 2. Use Case - Toggle Todo
// lib/features/todo/domain/usecases/toggle_todo.dart
class ToggleTodo implements UseCase<Todo, ToggleTodoParams> {
  final TodoRepository _repository;
  ToggleTodo(this._repository);

  @override
  Future<Either<Failure, Todo>> call(ToggleTodoParams params) async {
    // Get current todo
    final todoResult = await _repository.getTodoById(params.id);
    
    return todoResult.fold(
      (failure) => Left(failure),
      (todo) async {
        final updatedTodo = todo.isCompleted
            ? todo.copyWith(isCompleted: false, completedAt: null)
            : todo.complete();
        
        return _repository.updateTodo(updatedTodo);
      },
    );
  }
}

class ToggleTodoParams extends Equatable {
  final String id;
  const ToggleTodoParams(this.id);
  
  @override
  List<Object> get props => [id];
}
```

```dart
// 3. Presentation - TodoListPage
// lib/features/todo/presentation/pages/todo_list_page.dart
class TodoListPage extends StatelessWidget {
  const TodoListPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => getIt<TodoBloc>()..add(const LoadTodosEvent()),
      child: const TodoListView(),
    );
  }
}

class TodoListView extends StatelessWidget {
  const TodoListView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('My Todos'),
        actions: [
          IconButton(
            icon: const Icon(Icons.filter_list),
            onPressed: () => _showFilterDialog(context),
          ),
        ],
      ),
      body: BlocBuilder<TodoBloc, TodoState>(
        builder: (context, state) {
          if (state is TodoLoading) {
            return const Center(child: CircularProgressIndicator());
          }
          
          if (state is TodoError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, size: 48, color: Colors.red),
                  const SizedBox(height: 16),
                  Text(state.message, textAlign: TextAlign.center),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () => context.read<TodoBloc>()
                        .add(const LoadTodosEvent()),
                    child: const Text('Retry'),
                  ),
                ],
              ),
            );
          }
          
          if (state is TodoLoaded) {
            if (state.todos.isEmpty) {
              return const Center(
                child: Text('No todos yet. Add one!'),
              );
            }
            
            return ListView.builder(
              itemCount: state.todos.length,
              itemBuilder: (context, index) {
                final todo = state.todos[index];
                return TodoCard(
                  todo: todo,
                  onToggle: () => context.read<TodoBloc>()
                      .add(ToggleTodoEvent(todo.id)),
                  onDelete: () => context.read<TodoBloc>()
                      .add(DeleteTodoEvent(todo.id)),
                );
              },
            );
          }
          
          return const SizedBox.shrink();
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.pushNamed(context, '/todo/create'),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showFilterDialog(BuildContext context) {
    showModalBottomSheet(
      context: context,
      builder: (_) => BlocProvider.value(
        value: context.read<TodoBloc>(),
        child: const TodoFilterSheet(),
      ),
    );
  }
}
```

---

## สรุป (Summary)

```
Clean Architecture สรุปได้ดังนี้:

┌─────────────────────────────────────────────┐
│  Presentation Layer                         │
│  - Widgets, Pages                           │
│  - BLoC/Cubit/Provider                      │
│  - ViewModels                               │
├─────────────────────────────────────────────┤
│  Domain Layer (Core Business Rules)         │
│  - Entities (Pure Dart)                     │
│  - Use Cases (Application Logic)            │
│  - Repository Interfaces (Contracts)        │
├─────────────────────────────────────────────┤
│  Data Layer                                 │
│  - Models (JSON serializable)               │
│  - Repository Implementations               │
│  - Remote Data Sources (API)                │
│  - Local Data Sources (Cache/DB)            │
└─────────────────────────────────────────────┘

กฎสำคัญ:
- Dependencies ไหลเข้าหา Domain เท่านั้น
- Domain ไม่รู้จัก Data/Presentation
- ใช้ Either<Failure, Success> แทน throw Exception
- Each Use Case = Single Responsibility
- Repository Interface อยู่ใน Domain
- Repository Implementation อยู่ใน Data
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง `WeatherEntity` และ `WeatherModel` สำหรับ app ดูอากาศ
2. สร้าง `GetCurrentWeather` Use Case
3. สร้าง `WeatherRepository` interface และ implementation

**ระดับกลาง:**
4. เพิ่ม Offline Support ใน Repository ด้วย Cache Strategy
5. สร้าง `GetWeatherForecast` Use Case ที่ validate input
6. Implement `WeatherBloc` ที่ handle loading/success/error states

**ระดับสูง:**
7. สร้าง Feature "Note Taking" แบบ Complete ด้วย CRUD operations
8. เพิ่ม Pagination ใน List Use Case
9. Implement Optimistic Update (อัพเดท UI ก่อน API respond)

---

## การนำทาง
- [← Part 30: State Management Advanced](part_30.md)
- [→ Part 32: TDD & Testing](part_32.md)

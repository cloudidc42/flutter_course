# Part 50: Complete Project - World-class E-commerce App
## ขั้นตอนที่ 491-500 (Capstone Project)

---

## สารบัญ
1. [Project Setup & Architecture Overview](#ขั้นตอนที่-491-project-setup)
2. [Clean Architecture & Domain Layer](#ขั้นตอนที่-492-domain-layer)
3. [Firebase Backend Integration](#ขั้นตอนที่-493-firebase-backend)
4. [State Management with Riverpod](#ขั้นตอนที่-494-riverpod-state)
5. [Payment Integration](#ขั้นตอนที่-495-payment)
6. [Push Notifications & Offline Support](#ขั้นตอนที่-496-notifications-offline)
7. [Localization, Dark Mode & Accessibility](#ขั้นตอนที่-497-localization)
8. [Analytics, CI/CD & Testing](#ขั้นตอนที่-498-analytics-cicd)
9. [Performance Optimization](#ขั้นตอนที่-499-performance)
10. [Workshop: Full App Integration](#ขั้นตอนที่-500-workshop)

---

## ขั้นตอนที่ 491: Project Setup & Architecture Overview

### โครงสร้างโปรเจกต์

```
shopkrub/                         # World-class E-commerce App
├── android/
├── ios/
├── lib/
│   ├── core/
│   │   ├── constants/
│   │   │   ├── app_constants.dart
│   │   │   └── api_constants.dart
│   │   ├── di/
│   │   │   ├── injection.dart
│   │   │   └── injection.config.dart
│   │   ├── error/
│   │   │   ├── failure.dart
│   │   │   └── exception.dart
│   │   ├── network/
│   │   │   ├── api_client.dart
│   │   │   └── network_info.dart
│   │   ├── router/
│   │   │   └── app_router.dart
│   │   └── utils/
│   │       ├── date_utils.dart
│   │       └── price_utils.dart
│   ├── features/
│   │   ├── auth/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   ├── products/
│   │   │   ├── data/
│   │   │   ├── domain/
│   │   │   └── presentation/
│   │   ├── cart/
│   │   ├── orders/
│   │   ├── profile/
│   │   └── home/
│   ├── shared/
│   │   ├── design_system/
│   │   ├── widgets/
│   │   └── extensions/
│   └── main.dart
├── test/
│   ├── unit/
│   ├── widget/
│   └── integration/
├── pubspec.yaml
└── .github/workflows/
```

### pubspec.yaml

```yaml
name: shopkrub
description: World-class Flutter E-commerce App
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  
  # Firebase
  firebase_core: ^3.0.0
  firebase_auth: ^5.0.0
  cloud_firestore: ^5.0.0
  firebase_storage: ^12.0.0
  firebase_messaging: ^15.0.0
  firebase_analytics: ^11.0.0
  firebase_performance: ^0.10.0
  firebase_crashlytics: ^4.0.0
  firebase_remote_config: ^5.0.0
  
  # State Management
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  
  # Navigation
  go_router: ^14.0.0
  
  # DI
  get_it: ^8.0.0
  injectable: ^2.4.0
  
  # Network
  dio: ^5.4.0
  retrofit: ^4.1.0
  
  # Local Storage
  drift: ^2.18.0
  hive_flutter: ^1.1.0
  flutter_secure_storage: ^9.0.0
  shared_preferences: ^2.2.0
  
  # UI / Image
  cached_network_image: ^3.3.0
  shimmer: ^3.0.0
  flutter_svg: ^2.0.0
  image_picker: ^1.1.0
  
  # Payment
  stripe_flutter: ^10.0.0
  
  # Notifications
  flutter_local_notifications: ^17.0.0
  
  # Utils
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0
  equatable: ^2.0.0
  dartz: ^0.10.1
  intl: ^0.19.0
  connectivity_plus: ^6.0.0
  package_info_plus: ^8.0.0
  url_launcher: ^6.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  riverpod_generator: ^2.4.0
  injectable_generator: ^2.4.0
  retrofit_generator: ^9.1.0
  drift_dev: ^2.18.0
  hive_generator: ^2.0.0
  mockito: ^5.4.0
  bloc_test: ^9.1.0
```

---

## ขั้นตอนที่ 492: Clean Architecture & Domain Layer

### Entities

```dart
// lib/features/products/domain/entities/product.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'product.freezed.dart';

@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required String description,
    required double price,
    double? originalPrice,
    required String imageUrl,
    List<String>? additionalImages,
    required String category,
    required String brand,
    required bool isAvailable,
    required int stockQuantity,
    double? rating,
    int? reviewCount,
    Map<String, dynamic>? attributes,
    required DateTime createdAt,
    DateTime? updatedAt,
  }) = _Product;
  
  const Product._();
  
  bool get hasDiscount =>
      originalPrice != null && originalPrice! > price;
  
  int get discountPercent {
    if (!hasDiscount) return 0;
    return ((originalPrice! - price) / originalPrice! * 100).round();
  }
  
  bool get isInStock => isAvailable && stockQuantity > 0;
}

// lib/features/orders/domain/entities/order.dart
@freezed
class Order with _$Order {
  const factory Order({
    required String id,
    required String userId,
    required List<OrderItem> items,
    required OrderStatus status,
    required ShippingAddress shippingAddress,
    required PaymentInfo paymentInfo,
    required double subtotal,
    required double shippingFee,
    required double discount,
    required double total,
    String? promoCode,
    String? note,
    required DateTime createdAt,
    DateTime? updatedAt,
    DateTime? deliveredAt,
  }) = _Order;
  
  const Order._();
  
  int get itemCount => items.fold(0, (sum, item) => sum + item.quantity);
}

enum OrderStatus {
  pending,
  confirmed,
  processing,
  shipped,
  outForDelivery,
  delivered,
  cancelled,
  refunded,
}

@freezed
class OrderItem with _$OrderItem {
  const factory OrderItem({
    required String productId,
    required String productName,
    required String productImageUrl,
    required double price,
    required int quantity,
    Map<String, String>? selectedOptions,
  }) = _OrderItem;
  
  const OrderItem._();
  
  double get subtotal => price * quantity;
}

@freezed
class ShippingAddress with _$ShippingAddress {
  const factory ShippingAddress({
    required String name,
    required String phone,
    required String address,
    required String district,
    required String province,
    required String postalCode,
    String? note,
  }) = _ShippingAddress;
}
```

### Repository Contracts

```dart
// lib/features/products/domain/repositories/product_repository.dart
import 'package:dartz/dartz.dart';

abstract class ProductRepository {
  Future<Either<Failure, List<Product>>> getProducts({
    String? category,
    String? searchQuery,
    double? minPrice,
    double? maxPrice,
    ProductSortOption? sortBy,
    int limit = 20,
    String? lastDocumentId,
  });
  
  Future<Either<Failure, Product>> getProductById(String id);
  
  Future<Either<Failure, List<Product>>> getFeaturedProducts();
  
  Future<Either<Failure, List<Product>>> getRecommendations(String userId);
  
  Future<Either<Failure, List<Product>>> getByCategory(String category);
  
  Stream<Either<Failure, Product>> watchProduct(String id);
}

enum ProductSortOption {
  newest,
  priceAsc,
  priceDesc,
  topRated,
  mostPopular,
}

// lib/core/error/failure.dart
abstract class Failure extends Equatable {
  final String message;
  
  const Failure(this.message);
  
  @override
  List<Object> get props => [message];
}

class ServerFailure extends Failure {
  const ServerFailure([super.message = 'เกิดข้อผิดพลาดจากเซิร์ฟเวอร์']);
}

class NetworkFailure extends Failure {
  const NetworkFailure([super.message = 'ไม่มีการเชื่อมต่ออินเทอร์เน็ต']);
}

class CacheFailure extends Failure {
  const CacheFailure([super.message = 'เกิดข้อผิดพลาดในการโหลดข้อมูล']);
}

class AuthFailure extends Failure {
  const AuthFailure([super.message = 'กรุณาเข้าสู่ระบบอีกครั้ง']);
}
```

### Use Cases

```dart
// lib/features/products/domain/use_cases/get_products_usecase.dart
class GetProductsParams extends Equatable {
  final String? category;
  final String? searchQuery;
  final double? minPrice;
  final double? maxPrice;
  final ProductSortOption? sortBy;
  final int limit;
  final String? lastDocumentId;
  
  const GetProductsParams({
    this.category,
    this.searchQuery,
    this.minPrice,
    this.maxPrice,
    this.sortBy,
    this.limit = 20,
    this.lastDocumentId,
  });
  
  @override
  List<Object?> get props => [
    category, searchQuery, minPrice, maxPrice, sortBy, limit, lastDocumentId
  ];
}

class GetProductsUseCase
    implements UseCase<List<Product>, GetProductsParams> {
  final ProductRepository _repository;
  
  GetProductsUseCase(this._repository);
  
  @override
  Future<Either<Failure, List<Product>>> call(GetProductsParams params) {
    return _repository.getProducts(
      category: params.category,
      searchQuery: params.searchQuery,
      minPrice: params.minPrice,
      maxPrice: params.maxPrice,
      sortBy: params.sortBy,
      limit: params.limit,
      lastDocumentId: params.lastDocumentId,
    );
  }
}

abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

// lib/features/cart/domain/use_cases/add_to_cart_usecase.dart
class AddToCartParams extends Equatable {
  final String productId;
  final int quantity;
  final Map<String, String>? selectedOptions;
  
  const AddToCartParams({
    required this.productId,
    this.quantity = 1,
    this.selectedOptions,
  });
  
  @override
  List<Object?> get props => [productId, quantity, selectedOptions];
}

class AddToCartUseCase implements UseCase<Cart, AddToCartParams> {
  final CartRepository _repository;
  final ProductRepository _productRepository;
  
  AddToCartUseCase(this._repository, this._productRepository);
  
  @override
  Future<Either<Failure, Cart>> call(AddToCartParams params) async {
    // Validate product exists and is available
    final productResult = await _productRepository.getProductById(params.productId);
    
    return productResult.fold(
      (failure) => Left(failure),
      (product) async {
        if (!product.isInStock) {
          return const Left(ServerFailure('สินค้าหมดชั่วคราว'));
        }
        
        if (product.stockQuantity < params.quantity) {
          return Left(ServerFailure('สินค้าคงเหลือ ${product.stockQuantity} ชิ้น'));
        }
        
        return _repository.addItem(
          productId: params.productId,
          quantity: params.quantity,
          selectedOptions: params.selectedOptions,
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 493: Firebase Backend Integration

### Firebase Auth Service

```dart
// lib/features/auth/data/datasources/firebase_auth_datasource.dart
class FirebaseAuthDataSource {
  final FirebaseAuth _auth;
  final FirebaseFirestore _firestore;
  
  FirebaseAuthDataSource(this._auth, this._firestore);
  
  Future<UserModel> signInWithEmailPassword(
    String email,
    String password,
  ) async {
    try {
      final credential = await _auth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );
      
      final user = credential.user!;
      await _updateLastSeen(user.uid);
      
      return UserModel.fromFirebaseUser(user);
    } on FirebaseAuthException catch (e) {
      throw AuthException(_mapFirebaseAuthError(e.code));
    }
  }
  
  Future<UserModel> signInWithGoogle() async {
    final googleSignIn = GoogleSignIn();
    final googleUser = await googleSignIn.signIn();
    
    if (googleUser == null) {
      throw const AuthException('ยกเลิกการเข้าสู่ระบบด้วย Google');
    }
    
    final googleAuth = await googleUser.authentication;
    final credential = GoogleAuthProvider.credential(
      accessToken: googleAuth.accessToken,
      idToken: googleAuth.idToken,
    );
    
    final userCredential = await _auth.signInWithCredential(credential);
    final user = userCredential.user!;
    
    // Save user to Firestore if new
    if (userCredential.additionalUserInfo?.isNewUser ?? false) {
      await _createUserProfile(user);
    }
    
    return UserModel.fromFirebaseUser(user);
  }
  
  Future<UserModel> registerWithEmailPassword(
    String email,
    String password,
    String displayName,
  ) async {
    try {
      final credential = await _auth.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );
      
      final user = credential.user!;
      await user.updateDisplayName(displayName);
      await _createUserProfile(user);
      
      await user.sendEmailVerification();
      
      return UserModel.fromFirebaseUser(user);
    } on FirebaseAuthException catch (e) {
      throw AuthException(_mapFirebaseAuthError(e.code));
    }
  }
  
  Future<void> _createUserProfile(User user) async {
    await _firestore.collection('users').doc(user.uid).set({
      'id': user.uid,
      'email': user.email,
      'displayName': user.displayName,
      'photoUrl': user.photoURL,
      'createdAt': FieldValue.serverTimestamp(),
      'lastSeen': FieldValue.serverTimestamp(),
      'role': 'customer',
    });
  }
  
  Future<void> _updateLastSeen(String userId) async {
    await _firestore.collection('users').doc(userId).update({
      'lastSeen': FieldValue.serverTimestamp(),
    });
  }
  
  Stream<User?> get authStateChanges => _auth.authStateChanges();
  
  Future<void> signOut() async {
    await GoogleSignIn().signOut();
    await _auth.signOut();
  }
  
  String _mapFirebaseAuthError(String code) {
    return switch (code) {
      'user-not-found' => 'ไม่พบบัญชีผู้ใช้นี้',
      'wrong-password' => 'รหัสผ่านไม่ถูกต้อง',
      'email-already-in-use' => 'อีเมลนี้ถูกใช้แล้ว',
      'invalid-email' => 'รูปแบบอีเมลไม่ถูกต้อง',
      'weak-password' => 'รหัสผ่านต้องมีอย่างน้อย 6 ตัวอักษร',
      'too-many-requests' => 'คุณลองมากเกินไป กรุณารอสักครู่',
      _ => 'เกิดข้อผิดพลาด กรุณาลองใหม่',
    };
  }
}

// Product Repository Implementation
// lib/features/products/data/repositories/product_repository_impl.dart
class ProductRepositoryImpl implements ProductRepository {
  final FirebaseFirestore _firestore;
  final ProductLocalDataSource _localDataSource;
  final NetworkInfo _networkInfo;
  
  ProductRepositoryImpl(
    this._firestore,
    this._localDataSource,
    this._networkInfo,
  );
  
  @override
  Future<Either<Failure, List<Product>>> getProducts({
    String? category,
    String? searchQuery,
    double? minPrice,
    double? maxPrice,
    ProductSortOption? sortBy,
    int limit = 20,
    String? lastDocumentId,
  }) async {
    if (await _networkInfo.isConnected) {
      try {
        Query<Map<String, dynamic>> query = _firestore.collection('products')
            .where('isAvailable', isEqualTo: true)
            .limit(limit);
        
        if (category != null) {
          query = query.where('category', isEqualTo: category);
        }
        
        if (minPrice != null) {
          query = query.where('price', isGreaterThanOrEqualTo: minPrice);
        }
        
        if (maxPrice != null) {
          query = query.where('price', isLessThanOrEqualTo: maxPrice);
        }
        
        query = switch (sortBy) {
          ProductSortOption.priceAsc => query.orderBy('price'),
          ProductSortOption.priceDesc => query.orderBy('price', descending: true),
          ProductSortOption.topRated => query.orderBy('rating', descending: true),
          ProductSortOption.mostPopular => query.orderBy('reviewCount', descending: true),
          _ => query.orderBy('createdAt', descending: true),
        };
        
        if (lastDocumentId != null) {
          final lastDoc = await _firestore
              .collection('products')
              .doc(lastDocumentId)
              .get();
          query = query.startAfterDocument(lastDoc);
        }
        
        final snapshot = await query.get();
        final products = snapshot.docs
            .map((doc) => ProductModel.fromFirestore(doc))
            .where((p) => searchQuery == null ||
                p.name.toLowerCase().contains(searchQuery.toLowerCase()))
            .toList();
        
        // Cache locally
        await _localDataSource.cacheProducts(products);
        
        return Right(products);
      } catch (e) {
        return Left(ServerFailure('เกิดข้อผิดพลาดในการโหลดสินค้า: $e'));
      }
    } else {
      try {
        final cachedProducts = await _localDataSource.getCachedProducts(
          category: category,
          searchQuery: searchQuery,
        );
        return Right(cachedProducts);
      } catch (e) {
        return const Left(CacheFailure());
      }
    }
  }
  
  @override
  Stream<Either<Failure, Product>> watchProduct(String id) {
    return _firestore.collection('products').doc(id).snapshots().map((doc) {
      if (!doc.exists) {
        return const Left(ServerFailure('ไม่พบสินค้า'));
      }
      return Right(ProductModel.fromFirestore(doc) as Product);
    }).handleError((_) => const Left(ServerFailure()));
  }
}
```

---

## ขั้นตอนที่ 494: State Management with Riverpod

### Providers

```dart
// lib/features/products/presentation/providers/products_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'products_provider.g.dart';

@riverpod
class ProductsNotifier extends _$ProductsNotifier {
  @override
  AsyncValue<ProductsState> build() {
    return const AsyncValue.data(ProductsState.initial());
  }
  
  Future<void> loadProducts({
    String? category,
    String? searchQuery,
    bool refresh = false,
  }) async {
    if (refresh) {
      state = const AsyncValue.loading();
    }
    
    final currentState = state.valueOrNull ?? ProductsState.initial();
    
    if (!refresh && !currentState.hasMore) return;
    
    state = AsyncValue.loading();
    
    final useCase = ref.read(getProductsUseCaseProvider);
    final result = await useCase(GetProductsParams(
      category: category ?? currentState.selectedCategory,
      searchQuery: searchQuery ?? currentState.searchQuery,
      sortBy: currentState.sortOption,
      lastDocumentId: refresh ? null : currentState.lastDocumentId,
    ));
    
    result.fold(
      (failure) => state = AsyncValue.error(failure, StackTrace.current),
      (products) {
        final newProducts = refresh
            ? products
            : [...(state.valueOrNull?.products ?? []), ...products];
        
        state = AsyncValue.data(ProductsState(
          products: newProducts,
          selectedCategory: category ?? currentState.selectedCategory,
          searchQuery: searchQuery ?? currentState.searchQuery,
          sortOption: currentState.sortOption,
          hasMore: products.length == 20,
          lastDocumentId: products.isNotEmpty ? products.last.id : null,
          isLoading: false,
        ));
      },
    );
  }
  
  Future<void> applyFilter({
    String? category,
    ProductSortOption? sortOption,
    double? minPrice,
    double? maxPrice,
  }) async {
    final currentState = state.valueOrNull ?? ProductsState.initial();
    
    state = AsyncValue.data(currentState.copyWith(
      selectedCategory: category ?? currentState.selectedCategory,
      sortOption: sortOption ?? currentState.sortOption,
      minPrice: minPrice,
      maxPrice: maxPrice,
    ));
    
    await loadProducts(refresh: true);
  }
  
  Future<void> search(String query) async {
    await loadProducts(searchQuery: query, refresh: true);
  }
}

@freezed
class ProductsState with _$ProductsState {
  const factory ProductsState({
    required List<Product> products,
    String? selectedCategory,
    String? searchQuery,
    required ProductSortOption sortOption,
    required bool hasMore,
    String? lastDocumentId,
    required bool isLoading,
    double? minPrice,
    double? maxPrice,
  }) = _ProductsState;
  
  factory ProductsState.initial() => const ProductsState(
    products: [],
    sortOption: ProductSortOption.newest,
    hasMore: true,
    isLoading: false,
  );
}

// Cart Notifier
@riverpod
class CartNotifier extends _$CartNotifier {
  @override
  AsyncValue<Cart> build() {
    ref.listen(authStateProvider, (prev, next) {
      if (next.value == null) state = AsyncValue.data(Cart.empty());
    });
    return const AsyncValue.loading();
  }
  
  Future<void> loadCart() async {
    state = const AsyncValue.loading();
    
    final userId = ref.read(currentUserProvider)?.id;
    if (userId == null) {
      state = AsyncValue.data(Cart.empty());
      return;
    }
    
    final repository = ref.read(cartRepositoryProvider);
    final result = await repository.getCart(userId);
    
    result.fold(
      (failure) => state = AsyncValue.error(failure, StackTrace.current),
      (cart) => state = AsyncValue.data(cart),
    );
  }
  
  Future<void> addItem(
    String productId,
    int quantity, {
    Map<String, String>? options,
  }) async {
    final useCase = ref.read(addToCartUseCaseProvider);
    final result = await useCase(AddToCartParams(
      productId: productId,
      quantity: quantity,
      selectedOptions: options,
    ));
    
    result.fold(
      (failure) => throw CartException(failure.message),
      (cart) => state = AsyncValue.data(cart),
    );
  }
  
  Future<void> updateQuantity(String itemId, int newQuantity) async {
    if (newQuantity <= 0) {
      await removeItem(itemId);
      return;
    }
    
    final currentCart = state.value;
    if (currentCart == null) return;
    
    // Optimistic update
    state = AsyncValue.data(currentCart.updateItemQuantity(itemId, newQuantity));
    
    final repository = ref.read(cartRepositoryProvider);
    final result = await repository.updateItemQuantity(itemId, newQuantity);
    
    result.fold(
      (failure) {
        // Rollback on failure
        state = AsyncValue.data(currentCart);
        throw CartException(failure.message);
      },
      (cart) => state = AsyncValue.data(cart),
    );
  }
  
  Future<void> removeItem(String itemId) async {
    final currentCart = state.value;
    if (currentCart == null) return;
    
    // Optimistic update
    state = AsyncValue.data(currentCart.removeItem(itemId));
    
    final repository = ref.read(cartRepositoryProvider);
    final result = await repository.removeItem(itemId);
    
    result.fold(
      (failure) {
        state = AsyncValue.data(currentCart);
        throw CartException(failure.message);
      },
      (cart) => state = AsyncValue.data(cart),
    );
  }
  
  Future<void> applyPromoCode(String code) async {
    final repository = ref.read(cartRepositoryProvider);
    final result = await repository.applyPromoCode(code);
    
    result.fold(
      (failure) => throw CartException(failure.message),
      (cart) => state = AsyncValue.data(cart),
    );
  }
  
  Future<void> clearCart() async {
    state = AsyncValue.data(Cart.empty());
    await ref.read(cartRepositoryProvider).clearCart();
  }
}
```

---

## ขั้นตอนที่ 495: Payment Integration

### Stripe Payment

```dart
// lib/features/payment/data/services/stripe_service.dart
import 'package:stripe_flutter/stripe_flutter.dart';

class StripeService {
  final Dio _dio;
  final String _backendUrl;
  
  StripeService(this._dio, this._backendUrl) {
    Stripe.publishableKey = const String.fromEnvironment('STRIPE_PUBLISHABLE_KEY');
  }
  
  Future<PaymentResult> processPayment({
    required double amount,
    required String currency,
    required String orderId,
    required BillingDetails billingDetails,
  }) async {
    try {
      // 1. Create Payment Intent on backend
      final response = await _dio.post(
        '$_backendUrl/create-payment-intent',
        data: {
          'amount': (amount * 100).toInt(), // Stripe uses cents
          'currency': currency,
          'orderId': orderId,
          'metadata': {'orderId': orderId},
        },
      );
      
      final clientSecret = response.data['clientSecret'] as String;
      
      // 2. Initialize Payment Sheet
      await Stripe.instance.initPaymentSheet(
        paymentSheetParameters: SetupPaymentSheetParameters(
          paymentIntentClientSecret: clientSecret,
          merchantDisplayName: 'ShopKrub',
          billingDetails: billingDetails,
          allowsDelayedPaymentMethods: true,
          style: ThemeMode.system,
        ),
      );
      
      // 3. Present Payment Sheet
      await Stripe.instance.presentPaymentSheet();
      
      return const PaymentResult.success();
    } on StripeException catch (e) {
      return PaymentResult.failed(
        message: e.error.localizedMessage ?? 'การชำระเงินล้มเหลว',
        code: e.error.code.name,
      );
    } catch (e) {
      return PaymentResult.failed(message: 'เกิดข้อผิดพลาด: $e');
    }
  }
  
  Future<PaymentResult> saveCard({
    required String customerId,
  }) async {
    try {
      final response = await _dio.post(
        '$_backendUrl/create-setup-intent',
        data: {'customerId': customerId},
      );
      
      final clientSecret = response.data['clientSecret'] as String;
      
      await Stripe.instance.initPaymentSheet(
        paymentSheetParameters: SetupPaymentSheetParameters(
          setupIntentClientSecret: clientSecret,
          merchantDisplayName: 'ShopKrub',
        ),
      );
      
      await Stripe.instance.presentPaymentSheet();
      
      return const PaymentResult.success();
    } on StripeException catch (e) {
      return PaymentResult.failed(
        message: e.error.localizedMessage ?? 'ไม่สามารถบันทึกบัตรได้',
      );
    }
  }
}

@freezed
class PaymentResult with _$PaymentResult {
  const factory PaymentResult.success() = _PaymentSuccess;
  const factory PaymentResult.failed({
    required String message,
    String? code,
  }) = _PaymentFailed;
}

// Checkout Screen
class CheckoutScreen extends ConsumerStatefulWidget {
  const CheckoutScreen({super.key});
  
  @override
  ConsumerState<CheckoutScreen> createState() => _CheckoutScreenState();
}

class _CheckoutScreenState extends ConsumerState<CheckoutScreen> {
  bool _isProcessing = false;
  
  @override
  Widget build(BuildContext context) {
    final cartAsync = ref.watch(cartNotifierProvider);
    final selectedAddress = ref.watch(selectedAddressProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('ชำระเงิน')),
      body: cartAsync.when(
        data: (cart) => _buildCheckoutForm(cart, selectedAddress),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text('$e')),
      ),
      bottomNavigationBar: _buildPayButton(cartAsync.value),
    );
  }
  
  Widget _buildPayButton(Cart? cart) {
    if (cart == null) return const SizedBox.shrink();
    
    return SafeArea(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                const Text('ยอดรวมทั้งหมด'),
                Text(
                  '฿${cart.total.toStringAsFixed(0)}',
                  style: Theme.of(context).textTheme.titleLarge?.copyWith(
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                ),
              ],
            ),
            const SizedBox(height: 12),
            SizedBox(
              width: double.infinity,
              height: 52,
              child: FilledButton(
                onPressed: _isProcessing ? null : _processPayment,
                child: _isProcessing
                    ? const SizedBox(
                        width: 20,
                        height: 20,
                        child: CircularProgressIndicator(
                          strokeWidth: 2,
                          color: Colors.white,
                        ),
                      )
                    : Text(
                        'ชำระเงิน ฿${cart.total.toStringAsFixed(0)}',
                        style: const TextStyle(fontSize: 16),
                      ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Future<void> _processPayment() async {
    setState(() => _isProcessing = true);
    
    try {
      final cart = ref.read(cartNotifierProvider).value!;
      final address = ref.read(selectedAddressProvider)!;
      final user = ref.read(currentUserProvider)!;
      
      // Create Order first
      final orderResult = await ref.read(createOrderUseCaseProvider)(
        CreateOrderParams(
          cart: cart,
          shippingAddress: address,
        ),
      );
      
      await orderResult.fold(
        (failure) => throw Exception(failure.message),
        (order) async {
          // Process Payment
          final stripeService = ref.read(stripeServiceProvider);
          final paymentResult = await stripeService.processPayment(
            amount: order.total,
            currency: 'thb',
            orderId: order.id,
            billingDetails: BillingDetails(
              name: user.displayName,
              email: user.email,
              phone: address.phone,
            ),
          );
          
          paymentResult.when(
            success: () async {
              // Update order status
              await ref.read(orderRepositoryProvider).updateStatus(
                order.id,
                OrderStatus.confirmed,
              );
              
              // Clear cart
              await ref.read(cartNotifierProvider.notifier).clearCart();
              
              if (mounted) {
                context.go('/orders/${order.id}/success');
              }
            },
            failed: (message, _) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text(message), backgroundColor: Colors.red),
              );
            },
          );
        },
      );
    } finally {
      if (mounted) setState(() => _isProcessing = false);
    }
  }
  
  Widget _buildCheckoutForm(Cart cart, ShippingAddress? address) {
    return ListView(
      padding: const EdgeInsets.all(16),
      children: [
        // Order Summary
        _CheckoutSection(
          title: 'รายการสินค้า',
          child: Column(
            children: cart.items.map((item) => _CartItemRow(item: item)).toList(),
          ),
        ),
        const SizedBox(height: 16),
        
        // Shipping Address
        _CheckoutSection(
          title: 'ที่อยู่จัดส่ง',
          action: TextButton(
            onPressed: () => context.push('/addresses'),
            child: const Text('เปลี่ยน'),
          ),
          child: address != null
              ? _AddressCard(address: address)
              : OutlinedButton.icon(
                  onPressed: () => context.push('/addresses/add'),
                  icon: const Icon(Icons.add),
                  label: const Text('เพิ่มที่อยู่'),
                ),
        ),
        const SizedBox(height: 16),
        
        // Promo Code
        _PromoCodeSection(),
        const SizedBox(height: 16),
        
        // Price Breakdown
        _PriceBreakdown(cart: cart),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 496: Push Notifications & Offline Support

### Firebase Cloud Messaging

```dart
// lib/features/notifications/notification_service.dart
class NotificationService {
  final FirebaseMessaging _messaging;
  final FlutterLocalNotificationsPlugin _localNotifications;
  final FirebaseFirestore _firestore;
  
  NotificationService(
    this._messaging,
    this._localNotifications,
    this._firestore,
  );
  
  Future<void> initialize() async {
    // Request permission
    final settings = await _messaging.requestPermission(
      alert: true,
      badge: true,
      sound: true,
      provisional: false,
    );
    
    if (settings.authorizationStatus != AuthorizationStatus.authorized) {
      return;
    }
    
    // Get FCM Token
    final token = await _messaging.getToken();
    if (token != null) await _saveFcmToken(token);
    
    // Refresh token listener
    _messaging.onTokenRefresh.listen(_saveFcmToken);
    
    // Initialize local notifications
    await _initLocalNotifications();
    
    // Handle messages
    FirebaseMessaging.onMessage.listen(_handleForegroundMessage);
    FirebaseMessaging.onMessageOpenedApp.listen(_handleNotificationTap);
    
    // Handle background messages
    FirebaseMessaging.onBackgroundMessage(_handleBackgroundMessage);
    
    // Check initial message (app opened from notification)
    final initialMessage = await _messaging.getInitialMessage();
    if (initialMessage != null) {
      _handleNotificationTap(initialMessage);
    }
  }
  
  Future<void> _initLocalNotifications() async {
    const androidSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings(
      requestAlertPermission: false,
      requestBadgePermission: false,
      requestSoundPermission: false,
    );
    
    const initSettings = InitializationSettings(
      android: androidSettings,
      iOS: iosSettings,
    );
    
    await _localNotifications.initialize(
      initSettings,
      onDidReceiveNotificationResponse: (response) {
        _handleLocalNotificationTap(response.payload);
      },
    );
    
    // Create notification channels (Android)
    const orderChannel = AndroidNotificationChannel(
      'orders',
      'การสั่งซื้อ',
      description: 'แจ้งเตือนสถานะการสั่งซื้อ',
      importance: Importance.high,
    );
    
    const promoChannel = AndroidNotificationChannel(
      'promotions',
      'โปรโมชั่น',
      description: 'โปรโมชั่นและส่วนลดพิเศษ',
      importance: Importance.defaultImportance,
    );
    
    await _localNotifications
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(orderChannel);
    
    await _localNotifications
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(promoChannel);
  }
  
  Future<void> _handleForegroundMessage(RemoteMessage message) async {
    final notification = message.notification;
    if (notification == null) return;
    
    const androidDetails = AndroidNotificationDetails(
      'orders',
      'การสั่งซื้อ',
      importance: Importance.high,
      priority: Priority.high,
    );
    
    const iosDetails = DarwinNotificationDetails();
    
    await _localNotifications.show(
      notification.hashCode,
      notification.title,
      notification.body,
      const NotificationDetails(android: androidDetails, iOS: iosDetails),
      payload: message.data.toString(),
    );
  }
  
  void _handleNotificationTap(RemoteMessage message) {
    final data = message.data;
    _navigateFromNotification(data);
  }
  
  void _handleLocalNotificationTap(String? payload) {
    if (payload == null) return;
    // Parse and navigate
  }
  
  void _navigateFromNotification(Map<String, dynamic> data) {
    final type = data['type'] as String?;
    final id = data['id'] as String?;
    
    final router = getIt<GoRouter>();
    
    switch (type) {
      case 'order_update':
        router.go('/orders/$id');
      case 'promotion':
        router.go('/promotions/$id');
      case 'product':
        router.go('/products/$id');
    }
  }
  
  Future<void> _saveFcmToken(String token) async {
    final userId = getIt<FirebaseAuth>().currentUser?.uid;
    if (userId == null) return;
    
    await _firestore.collection('users').doc(userId).update({
      'fcmTokens': FieldValue.arrayUnion([token]),
      'fcmTokenUpdatedAt': FieldValue.serverTimestamp(),
    });
  }
  
  Future<void> subscribeToTopic(String topic) async {
    await _messaging.subscribeToTopic(topic);
  }
  
  Future<void> unsubscribeFromTopic(String topic) async {
    await _messaging.unsubscribeFromTopic(topic);
  }
}

@pragma('vm:entry-point')
Future<void> _handleBackgroundMessage(RemoteMessage message) async {
  // Must be a top-level function
  await Firebase.initializeApp();
}

// Offline Support - Cached Products
class ProductLocalDataSourceImpl implements ProductLocalDataSource {
  final AppDatabase _db;
  
  ProductLocalDataSourceImpl(this._db);
  
  @override
  Future<List<Product>> getCachedProducts({
    String? category,
    String? searchQuery,
  }) async {
    var query = _db.select(_db.productTable);
    
    if (category != null) {
      query = query..where((p) => p.category.equals(category));
    }
    
    final rows = await query.get();
    var products = rows.map(ProductTableMapper.fromRow).toList();
    
    if (searchQuery != null) {
      final lower = searchQuery.toLowerCase();
      products = products
          .where((p) => p.name.toLowerCase().contains(lower))
          .toList();
    }
    
    return products;
  }
  
  @override
  Future<void> cacheProducts(List<Product> products) async {
    await _db.transaction(() async {
      for (final product in products) {
        await _db.into(_db.productTable).insertOnConflictUpdate(
          ProductTableMapper.toCompanion(product),
        );
      }
    });
  }
}
```

---

## ขั้นตอนที่ 497: Localization, Dark Mode & Accessibility

### Localization Setup

```dart
// lib/l10n/app_th.arb
{
  "@@locale": "th",
  "appTitle": "ช้อปกรูบ",
  "products": "สินค้า",
  "cart": "ตะกร้า",
  "profile": "โปรไฟล์",
  "orders": "การสั่งซื้อ",
  "search": "ค้นหา",
  "searchHint": "ค้นหาสินค้า...",
  "addToCart": "เพิ่มในตะกร้า",
  "buyNow": "ซื้อเลย",
  "checkout": "ชำระเงิน",
  "total": "รวม",
  "subtotal": "ราคาสินค้า",
  "shippingFee": "ค่าจัดส่ง",
  "discount": "ส่วนลด",
  "orderTotal": "ยอดรวมทั้งหมด",
  "outOfStock": "สินค้าหมด",
  "loading": "กำลังโหลด...",
  "noProducts": "ไม่พบสินค้า",
  "retry": "ลองใหม่",
  "signIn": "เข้าสู่ระบบ",
  "signUp": "สมัครสมาชิก",
  "signOut": "ออกจากระบบ",
  "email": "อีเมล",
  "password": "รหัสผ่าน",
  "confirmPassword": "ยืนยันรหัสผ่าน",
  "name": "ชื่อ-นามสกุล",
  "phone": "เบอร์โทรศัพท์",
  "address": "ที่อยู่",
  "orderStatus_pending": "รอยืนยัน",
  "orderStatus_confirmed": "ยืนยันแล้ว",
  "orderStatus_processing": "กำลังจัดเตรียม",
  "orderStatus_shipped": "จัดส่งแล้ว",
  "orderStatus_delivered": "ได้รับสินค้าแล้ว",
  "orderStatus_cancelled": "ยกเลิกแล้ว",
  "itemCount": "{count, plural, =1{1 รายการ} other{{count} รายการ}}",
  "@itemCount": {
    "placeholders": {
      "count": {"type": "int"}
    }
  }
}

// lib/l10n/app_en.arb
{
  "@@locale": "en",
  "appTitle": "ShopKrub",
  "products": "Products",
  "cart": "Cart",
  "profile": "Profile",
  "orders": "Orders",
  "search": "Search",
  "searchHint": "Search products...",
  "addToCart": "Add to Cart",
  "buyNow": "Buy Now",
  "checkout": "Checkout",
  "total": "Total",
  "subtotal": "Subtotal",
  "shippingFee": "Shipping Fee",
  "discount": "Discount",
  "orderTotal": "Order Total",
  "outOfStock": "Out of Stock",
  "loading": "Loading...",
  "noProducts": "No products found",
  "retry": "Retry",
  "signIn": "Sign In",
  "signUp": "Sign Up",
  "signOut": "Sign Out",
  "email": "Email",
  "password": "Password",
  "confirmPassword": "Confirm Password",
  "name": "Full Name",
  "phone": "Phone",
  "address": "Address",
  "orderStatus_pending": "Pending",
  "orderStatus_confirmed": "Confirmed",
  "orderStatus_processing": "Processing",
  "orderStatus_shipped": "Shipped",
  "orderStatus_delivered": "Delivered",
  "orderStatus_cancelled": "Cancelled",
  "itemCount": "{count, plural, =1{1 item} other{{count} items}}",
  "@itemCount": {
    "placeholders": {
      "count": {"type": "int"}
    }
  }
}

// lib/core/router/app_router.dart
@riverpod
GoRouter appRouter(AppRouterRef ref) {
  final authState = ref.watch(authStateProvider);
  
  return GoRouter(
    initialLocation: '/',
    redirect: (context, state) {
      final isAuthenticated = authState.value != null;
      final isAuthRoute = state.matchedLocation.startsWith('/auth');
      
      if (!isAuthenticated && !isAuthRoute) {
        return '/auth/login';
      }
      
      if (isAuthenticated && isAuthRoute) {
        return '/';
      }
      
      return null;
    },
    routes: [
      ShellRoute(
        builder: (context, state, child) => MainScaffold(child: child),
        routes: [
          GoRoute(
            path: '/',
            builder: (context, state) => const HomeScreen(),
          ),
          GoRoute(
            path: '/products',
            builder: (context, state) => const ProductsScreen(),
            routes: [
              GoRoute(
                path: ':id',
                builder: (context, state) => ProductDetailScreen(
                  productId: state.pathParameters['id']!,
                ),
              ),
            ],
          ),
          GoRoute(
            path: '/cart',
            builder: (context, state) => const CartScreen(),
          ),
          GoRoute(
            path: '/orders',
            builder: (context, state) => const OrdersScreen(),
            routes: [
              GoRoute(
                path: ':id',
                builder: (context, state) => OrderDetailScreen(
                  orderId: state.pathParameters['id']!,
                ),
              ),
              GoRoute(
                path: ':id/success',
                builder: (context, state) => OrderSuccessScreen(
                  orderId: state.pathParameters['id']!,
                ),
              ),
            ],
          ),
          GoRoute(
            path: '/profile',
            builder: (context, state) => const ProfileScreen(),
          ),
        ],
      ),
      GoRoute(
        path: '/auth/login',
        builder: (context, state) => const LoginScreen(),
      ),
      GoRoute(
        path: '/auth/register',
        builder: (context, state) => const RegisterScreen(),
      ),
      GoRoute(
        path: '/checkout',
        builder: (context, state) => const CheckoutScreen(),
      ),
    ],
  );
}

// Dark Mode Provider
@riverpod
class ThemeModeNotifier extends _$ThemeModeNotifier {
  static const String _key = 'theme_mode';
  
  @override
  ThemeMode build() {
    final prefs = ref.watch(sharedPreferencesProvider);
    final value = prefs.getString(_key);
    return switch (value) {
      'dark' => ThemeMode.dark,
      'light' => ThemeMode.light,
      _ => ThemeMode.system,
    };
  }
  
  Future<void> setThemeMode(ThemeMode mode) async {
    final prefs = ref.read(sharedPreferencesProvider);
    await prefs.setString(_key, mode.name);
    state = mode;
  }
  
  void toggle() {
    setThemeMode(state == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark);
  }
}
```

---

## ขั้นตอนที่ 498: Analytics, CI/CD & Testing

### Unit Tests

```dart
// test/unit/features/cart/add_to_cart_usecase_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:dartz/dartz.dart';

@GenerateMocks([CartRepository, ProductRepository])
import 'add_to_cart_usecase_test.mocks.dart';

void main() {
  late AddToCartUseCase useCase;
  late MockCartRepository mockCartRepository;
  late MockProductRepository mockProductRepository;
  
  setUp(() {
    mockCartRepository = MockCartRepository();
    mockProductRepository = MockProductRepository();
    useCase = AddToCartUseCase(mockCartRepository, mockProductRepository);
  });
  
  group('AddToCartUseCase', () {
    final tProduct = Product(
      id: 'p1',
      name: 'iPhone 15',
      description: 'Test product',
      price: 35000,
      imageUrl: 'https://example.com/image.jpg',
      category: 'electronics',
      brand: 'Apple',
      isAvailable: true,
      stockQuantity: 10,
      createdAt: DateTime.now(),
    );
    
    final tCart = Cart(
      id: 'cart1',
      userId: 'user1',
      items: [
        CartItem(
          id: 'item1',
          productId: 'p1',
          productName: 'iPhone 15',
          productImageUrl: 'https://example.com/image.jpg',
          price: 35000,
          quantity: 1,
        ),
      ],
    );
    
    test('should return cart when product is available', () async {
      // Arrange
      when(mockProductRepository.getProductById('p1'))
          .thenAnswer((_) async => Right(tProduct));
      when(mockCartRepository.addItem(
        productId: 'p1',
        quantity: 1,
        selectedOptions: null,
      )).thenAnswer((_) async => Right(tCart));
      
      // Act
      final result = await useCase(const AddToCartParams(
        productId: 'p1',
        quantity: 1,
      ));
      
      // Assert
      expect(result, Right(tCart));
      verify(mockProductRepository.getProductById('p1'));
      verify(mockCartRepository.addItem(
        productId: 'p1',
        quantity: 1,
        selectedOptions: null,
      ));
    });
    
    test('should return failure when product is out of stock', () async {
      // Arrange
      final outOfStockProduct = tProduct.copyWith(
        isAvailable: false,
        stockQuantity: 0,
      );
      
      when(mockProductRepository.getProductById('p1'))
          .thenAnswer((_) async => Right(outOfStockProduct));
      
      // Act
      final result = await useCase(const AddToCartParams(
        productId: 'p1',
        quantity: 1,
      ));
      
      // Assert
      expect(result, const Left(ServerFailure('สินค้าหมดชั่วคราว')));
      verifyNever(mockCartRepository.addItem(
        productId: anyNamed('productId'),
        quantity: anyNamed('quantity'),
      ));
    });
    
    test('should return failure when quantity exceeds stock', () async {
      // Arrange
      when(mockProductRepository.getProductById('p1'))
          .thenAnswer((_) async => Right(tProduct)); // stock = 10
      
      // Act
      final result = await useCase(const AddToCartParams(
        productId: 'p1',
        quantity: 15, // > 10
      ));
      
      // Assert
      expect(result.isLeft(), true);
    });
  });
}

// Widget Tests
// test/widget/features/products/product_card_test.dart
void main() {
  group('ProductCardOrganism', () {
    testWidgets('should display product name and price', (tester) async {
      final product = Product(
        id: '1',
        name: 'Nike Air Max',
        description: 'Running shoes',
        price: 4500,
        originalPrice: 5000,
        imageUrl: 'https://example.com/shoe.jpg',
        category: 'shoes',
        brand: 'Nike',
        isAvailable: true,
        stockQuantity: 5,
        createdAt: DateTime.now(),
      );
      
      await tester.pumpWidget(
        ProviderScope(
          child: MaterialApp(
            home: Scaffold(
              body: ProductCardOrganism(
                product: product,
                onTap: () {},
                onAddToCart: () {},
                isFavorite: false,
                onToggleFavorite: () {},
              ),
            ),
          ),
        ),
      );
      
      expect(find.text('Nike Air Max'), findsOneWidget);
      expect(find.text('฿4,500'), findsOneWidget);
      expect(find.text('-10%'), findsOneWidget);
    });
    
    testWidgets('should show "หมด" overlay when out of stock', (tester) async {
      final outOfStockProduct = Product(
        id: '1',
        name: 'Test Product',
        description: 'Test',
        price: 100,
        imageUrl: 'https://example.com/img.jpg',
        category: 'test',
        brand: 'Test',
        isAvailable: false,
        stockQuantity: 0,
        createdAt: DateTime.now(),
      );
      
      await tester.pumpWidget(
        ProviderScope(
          child: MaterialApp(
            home: Scaffold(
              body: ProductCardOrganism(
                product: outOfStockProduct,
                onTap: () {},
                onAddToCart: () {},
                isFavorite: false,
                onToggleFavorite: () {},
              ),
            ),
          ),
        ),
      );
      
      expect(find.text('หมด'), findsOneWidget);
    });
  });
}
```

### CI/CD Pipeline

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  FLUTTER_VERSION: '3.24.0'

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter pub run build_runner build --delete-conflicting-outputs
      
      - name: Run unit tests
        run: flutter test test/unit/ --coverage
      
      - name: Run widget tests
        run: flutter test test/widget/ --coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info
  
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .
      
      - name: Run analyzer
        run: flutter analyze
  
  build-android:
    name: Build Android
    needs: [test, lint]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter pub run build_runner build --delete-conflicting-outputs
      
      - name: Decode keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks
      
      - name: Build APK
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          STRIPE_PUBLISHABLE_KEY: ${{ secrets.STRIPE_PUBLISHABLE_KEY }}
        run: |
          flutter build apk --release \
            --dart-define=STRIPE_PUBLISHABLE_KEY=$STRIPE_PUBLISHABLE_KEY
      
      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_ANDROID_APP_ID }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
          groups: testers
          file: build/app/outputs/flutter-apk/app-release.apk
  
  build-ios:
    name: Build iOS
    needs: [test, lint]
    runs-on: macos-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Generate code
        run: flutter pub run build_runner build --delete-conflicting-outputs
      
      - name: Build iOS (no codesign)
        run: flutter build ios --release --no-codesign
```

---

## ขั้นตอนที่ 499: Performance Optimization

### App Performance

```dart
// lib/core/utils/performance_monitor.dart
class AppPerformanceMonitor {
  static final Map<String, Stopwatch> _activeTraces = {};
  
  static void startTrace(String name) {
    _activeTraces[name] = Stopwatch()..start();
  }
  
  static Duration? stopTrace(String name) {
    final stopwatch = _activeTraces.remove(name);
    if (stopwatch == null) return null;
    
    stopwatch.stop();
    final duration = stopwatch.elapsed;
    
    // Log slow operations
    if (duration.inMilliseconds > 1000) {
      debugPrint('[PERFORMANCE] Slow: $name took ${duration.inMilliseconds}ms');
      FirebasePerformance.instance.newTrace(name).then((trace) async {
        await trace.start();
        await trace.stop();
      });
    }
    
    return duration;
  }
}

// Image Caching Configuration
class AppImageCacheConfig {
  static void configure() {
    // Set cache size limits
    PaintingBinding.instance.imageCache.maximumSize = 200;
    PaintingBinding.instance.imageCache.maximumSizeBytes = 50 * 1024 * 1024; // 50 MB
  }
}

// Lazy Loading Provider
@riverpod
class LazyProductsNotifier extends _$LazyProductsNotifier {
  static const _pageSize = 20;
  final List<Product> _allProducts = [];
  String? _lastDocumentId;
  bool _hasMore = true;
  bool _isLoading = false;
  
  @override
  AsyncValue<List<Product>> build() => const AsyncValue.data([]);
  
  Future<void> loadMore() async {
    if (_isLoading || !_hasMore) return;
    
    _isLoading = true;
    
    final useCase = ref.read(getProductsUseCaseProvider);
    final result = await useCase(GetProductsParams(
      limit: _pageSize,
      lastDocumentId: _lastDocumentId,
    ));
    
    result.fold(
      (failure) {
        state = AsyncValue.error(failure, StackTrace.current);
      },
      (products) {
        _allProducts.addAll(products);
        _hasMore = products.length == _pageSize;
        _lastDocumentId = products.isNotEmpty ? products.last.id : null;
        state = AsyncValue.data(List.unmodifiable(_allProducts));
      },
    );
    
    _isLoading = false;
  }
  
  void reset() {
    _allProducts.clear();
    _lastDocumentId = null;
    _hasMore = true;
    _isLoading = false;
    state = const AsyncValue.data([]);
  }
}

// Memory Management
class ImagePreloader {
  static Future<void> preloadImages(
    BuildContext context,
    List<String> urls,
  ) async {
    await Future.wait(
      urls.take(5).map(
        (url) => precacheImage(
          CachedNetworkImageProvider(url),
          context,
        ),
      ),
    );
  }
}

// Smooth Scrolling
class SmoothProductGrid extends StatefulWidget {
  final List<Product> products;
  final VoidCallback onLoadMore;
  final bool hasMore;
  
  const SmoothProductGrid({
    super.key,
    required this.products,
    required this.onLoadMore,
    required this.hasMore,
  });
  
  @override
  State<SmoothProductGrid> createState() => _SmoothProductGridState();
}

class _SmoothProductGridState extends State<SmoothProductGrid> {
  final _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _scrollController.addListener(() {
      if (_scrollController.position.pixels >
          _scrollController.position.maxScrollExtent * 0.8) {
        widget.onLoadMore();
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      controller: _scrollController,
      physics: const BouncingScrollPhysics(),
      slivers: [
        SliverPadding(
          padding: const EdgeInsets.all(12),
          sliver: SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              childAspectRatio: 0.68,
              crossAxisSpacing: 10,
              mainAxisSpacing: 10,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                if (index == widget.products.length) {
                  return widget.hasMore
                      ? const Center(child: CircularProgressIndicator())
                      : const SizedBox.shrink();
                }
                
                return RepaintBoundary(
                  key: ValueKey(widget.products[index].id),
                  child: ConsumerWidget(
                    builder: (context, ref, _) {
                      final product = widget.products[index];
                      final isFavorite = ref.watch(
                        isFavoriteProvider(product.id),
                      );
                      
                      return ProductCardOrganism(
                        product: product,
                        onTap: () => context.push('/products/${product.id}'),
                        onAddToCart: () => _addToCart(ref, product),
                        isFavorite: isFavorite,
                        onToggleFavorite: () =>
                            ref.read(favoritesNotifierProvider.notifier)
                                .toggle(product.id),
                      );
                    },
                  ),
                );
              },
              childCount: widget.products.length + (widget.hasMore ? 1 : 0),
            ),
          ),
        ),
      ],
    );
  }
  
  Future<void> _addToCart(WidgetRef ref, Product product) async {
    try {
      await ref.read(cartNotifierProvider.notifier).addItem(product.id, 1);
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('เพิ่ม ${product.name} ในตะกร้าแล้ว'),
            action: SnackBarAction(
              label: 'ดูตะกร้า',
              onPressed: () => context.go('/cart'),
            ),
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('$e'),
            backgroundColor: Colors.red,
          ),
        );
      }
    }
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 500: Workshop - Full App Integration

### Main App Entry Point

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Lock orientation to portrait
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  
  // Initialize Firebase
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  // Setup Crashlytics
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };
  
  // Configure image cache
  AppImageCacheConfig.configure();
  
  // Setup DI
  await configureDependencies();
  
  // Initialize notification service
  await getIt<NotificationService>().initialize();
  
  runApp(
    ProviderScope(
      child: const ShopKrubApp(),
    ),
  );
}

class ShopKrubApp extends ConsumerWidget {
  const ShopKrubApp({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);
    final themeMode = ref.watch(themeModeNotifierProvider);
    final locale = ref.watch(localeProvider);
    
    return MaterialApp.router(
      title: 'ShopKrub',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      themeMode: themeMode,
      locale: locale,
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      routerConfig: router,
      builder: (context, child) {
        // Apply text scaling limits
        return MediaQuery(
          data: MediaQuery.of(context).copyWith(
            textScaler: TextScaler.linear(
              MediaQuery.of(context).textScaler.scale(1.0).clamp(0.8, 1.4),
            ),
          ),
          child: child ?? const SizedBox.shrink(),
        );
      },
    );
  }
}

// AppTheme
class AppTheme {
  static ThemeData get lightTheme {
    const primaryColor = Color(0xFF1A73E8);
    
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: primaryColor,
        brightness: Brightness.light,
      ),
      fontFamily: 'Sarabun',
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
        scrolledUnderElevation: 1,
      ),
      cardTheme: CardThemeData(
        elevation: 0,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
          side: BorderSide(color: Colors.grey.shade200),
        ),
      ),
      inputDecorationTheme: InputDecorationTheme(
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
        ),
        contentPadding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 14,
        ),
      ),
      filledButtonTheme: FilledButtonThemeData(
        style: FilledButton.styleFrom(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          minimumSize: const Size(0, 48),
        ),
      ),
    );
  }
  
  static ThemeData get darkTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: const Color(0xFF8AB4F8),
        brightness: Brightness.dark,
      ),
      fontFamily: 'Sarabun',
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
      ),
    );
  }
}

// Home Screen
class HomeScreen extends ConsumerStatefulWidget {
  const HomeScreen({super.key});
  
  @override
  ConsumerState<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends ConsumerState<HomeScreen> {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addPostFrameCallback((_) {
      ref.read(lazyProductsNotifierProvider.notifier).loadMore();
    });
  }
  
  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context)!;
    
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            floating: true,
            snap: true,
            title: Image.asset('assets/images/logo.png', height: 32),
            actions: [
              IconButton(
                icon: const Icon(Icons.search),
                onPressed: () => context.push('/search'),
              ),
              Consumer(
                builder: (context, ref, _) {
                  final cartAsync = ref.watch(cartNotifierProvider);
                  final itemCount = cartAsync.value?.itemCount ?? 0;
                  
                  return badges.Badge(
                    showBadge: itemCount > 0,
                    badgeContent: Text(
                      '$itemCount',
                      style: const TextStyle(
                        color: Colors.white,
                        fontSize: 10,
                      ),
                    ),
                    child: IconButton(
                      icon: const Icon(Icons.shopping_cart_outlined),
                      onPressed: () => context.go('/cart'),
                    ),
                  );
                },
              ),
            ],
          ),
          
          // Hero Banner
          SliverToBoxAdapter(
            child: _HeroBannerWidget(),
          ),
          
          // Categories
          SliverToBoxAdapter(
            child: _CategoriesWidget(),
          ),
          
          // Featured Products
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    'สินค้าแนะนำ',
                    style: Theme.of(context).textTheme.titleLarge?.copyWith(
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  TextButton(
                    onPressed: () => context.go('/products'),
                    child: const Text('ดูทั้งหมด'),
                  ),
                ],
              ),
            ),
          ),
          
          // Products Grid
          Consumer(
            builder: (context, ref, _) {
              final productsAsync = ref.watch(lazyProductsNotifierProvider);
              
              return productsAsync.when(
                data: (products) => SliverPadding(
                  padding: const EdgeInsets.symmetric(horizontal: 12),
                  sliver: SliverGrid(
                    gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: 2,
                      childAspectRatio: 0.68,
                      crossAxisSpacing: 10,
                      mainAxisSpacing: 10,
                    ),
                    delegate: SliverChildBuilderDelegate(
                      (context, index) {
                        final product = products[index];
                        return RepaintBoundary(
                          key: ValueKey(product.id),
                          child: ProductCardOrganism(
                            product: product,
                            onTap: () => context.push('/products/${product.id}'),
                            onAddToCart: () => _addToCart(ref, product),
                            isFavorite: ref.watch(isFavoriteProvider(product.id)),
                            onToggleFavorite: () => ref
                                .read(favoritesNotifierProvider.notifier)
                                .toggle(product.id),
                          ),
                        );
                      },
                      childCount: products.length,
                    ),
                  ),
                ),
                loading: () => const SliverFillRemaining(
                  child: Center(child: CircularProgressIndicator()),
                ),
                error: (e, _) => SliverFillRemaining(
                  child: Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        const Icon(Icons.error_outline, size: 48),
                        const SizedBox(height: 12),
                        Text('$e'),
                        const SizedBox(height: 12),
                        FilledButton(
                          onPressed: () => ref
                              .read(lazyProductsNotifierProvider.notifier)
                              .loadMore(),
                          child: const Text('ลองใหม่'),
                        ),
                      ],
                    ),
                  ),
                ),
              );
            },
          ),
        ],
      ),
    );
  }
  
  Future<void> _addToCart(WidgetRef ref, Product product) async {
    try {
      await ref.read(cartNotifierProvider.notifier).addItem(product.id, 1);
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('เพิ่ม "${product.name}" ในตะกร้าแล้ว'),
            behavior: SnackBarBehavior.floating,
            action: SnackBarAction(
              label: 'ดูตะกร้า',
              onPressed: () => context.go('/cart'),
            ),
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('ไม่สามารถเพิ่มสินค้าได้: $e'),
            backgroundColor: Theme.of(context).colorScheme.error,
          ),
        );
      }
    }
  }
}

// Main Scaffold with Bottom Navigation
class MainScaffold extends ConsumerWidget {
  final Widget child;
  
  const MainScaffold({super.key, required this.child});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final currentRoute = GoRouterState.of(context).matchedLocation;
    
    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: _getSelectedIndex(currentRoute),
        onDestinationSelected: (index) => _onNavTap(context, index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'หน้าหลัก',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: 'ค้นหา',
          ),
          NavigationDestination(
            icon: Icon(Icons.shopping_cart_outlined),
            selectedIcon: Icon(Icons.shopping_cart),
            label: 'ตะกร้า',
          ),
          NavigationDestination(
            icon: Icon(Icons.receipt_long_outlined),
            selectedIcon: Icon(Icons.receipt_long),
            label: 'คำสั่งซื้อ',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'โปรไฟล์',
          ),
        ],
      ),
    );
  }
  
  int _getSelectedIndex(String route) {
    if (route.startsWith('/')) {
      if (route == '/') return 0;
      if (route.startsWith('/products')) return 1;
      if (route.startsWith('/cart')) return 2;
      if (route.startsWith('/orders')) return 3;
      if (route.startsWith('/profile')) return 4;
    }
    return 0;
  }
  
  void _onNavTap(BuildContext context, int index) {
    switch (index) {
      case 0: context.go('/');
      case 1: context.go('/products');
      case 2: context.go('/cart');
      case 3: context.go('/orders');
      case 4: context.go('/profile');
    }
  }
}
```

### Product Detail Screen

```dart
// lib/features/products/presentation/screens/product_detail_screen.dart
class ProductDetailScreen extends ConsumerStatefulWidget {
  final String productId;
  
  const ProductDetailScreen({super.key, required this.productId});
  
  @override
  ConsumerState<ProductDetailScreen> createState() =>
      _ProductDetailScreenState();
}

class _ProductDetailScreenState extends ConsumerState<ProductDetailScreen> {
  int _selectedImageIndex = 0;
  int _quantity = 1;
  
  @override
  Widget build(BuildContext context) {
    final productAsync = ref.watch(productProvider(widget.productId));
    final isFavorite = ref.watch(isFavoriteProvider(widget.productId));
    
    return Scaffold(
      body: productAsync.when(
        data: (product) => _buildBody(product, isFavorite),
        loading: () => const _ProductDetailSkeleton(),
        error: (e, _) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.error_outline, size: 48),
              const SizedBox(height: 12),
              Text('ไม่สามารถโหลดสินค้าได้'),
              const SizedBox(height: 12),
              FilledButton(
                onPressed: () => ref.invalidate(productProvider(widget.productId)),
                child: const Text('ลองใหม่'),
              ),
            ],
          ),
        ),
      ),
    );
  }
  
  Widget _buildBody(Product product, bool isFavorite) {
    return Stack(
      children: [
        CustomScrollView(
          slivers: [
            // Image Gallery
            SliverToBoxAdapter(
              child: _ProductImageGallery(
                product: product,
                selectedIndex: _selectedImageIndex,
                onIndexChanged: (i) =>
                    setState(() => _selectedImageIndex = i),
                isFavorite: isFavorite,
                onFavoriteToggle: () =>
                    ref.read(favoritesNotifierProvider.notifier)
                        .toggle(product.id),
              ),
            ),
            
            // Product Info
            SliverPadding(
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 120),
              sliver: SliverList(
                delegate: SliverChildListDelegate([
                  Row(
                    children: [
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              product.brand,
                              style: Theme.of(context).textTheme.bodySmall?.copyWith(
                                color: Theme.of(context).colorScheme.primary,
                              ),
                            ),
                            const SizedBox(height: 4),
                            Text(
                              product.name,
                              style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                          ],
                        ),
                      ),
                      // Quantity Selector
                      _QuantitySelector(
                        quantity: _quantity,
                        max: product.stockQuantity,
                        onChanged: (q) => setState(() => _quantity = q),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  
                  ProductPriceMolecule(
                    price: product.price,
                    originalPrice: product.originalPrice,
                  ),
                  
                  if (product.rating != null) ...[
                    const SizedBox(height: 12),
                    RatingMolecule(
                      rating: product.rating!,
                      reviewCount: product.reviewCount ?? 0,
                    ),
                  ],
                  
                  const Divider(height: 32),
                  
                  // Description
                  Text(
                    'รายละเอียดสินค้า',
                    style: Theme.of(context).textTheme.titleMedium?.copyWith(
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    product.description,
                    style: Theme.of(context).textTheme.bodyMedium,
                  ),
                  
                  // Reviews Section
                  const SizedBox(height: 24),
                  _ProductReviewsSection(productId: product.id),
                ]),
              ),
            ),
          ],
        ),
        
        // Bottom Action Bar
        Positioned(
          bottom: 0,
          left: 0,
          right: 0,
          child: _BottomActionBar(
            product: product,
            quantity: _quantity,
          ),
        ),
      ],
    );
  }
}

class _BottomActionBar extends ConsumerWidget {
  final Product product;
  final int quantity;
  
  const _BottomActionBar({
    required this.product,
    required this.quantity,
  });
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Container(
      padding: EdgeInsets.fromLTRB(
        16,
        12,
        16,
        MediaQuery.of(context).padding.bottom + 12,
      ),
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.08),
            blurRadius: 8,
            offset: const Offset(0, -2),
          ),
        ],
      ),
      child: Row(
        children: [
          Expanded(
            child: OutlinedButton.icon(
              onPressed: product.isInStock
                  ? () => _addToCart(context, ref)
                  : null,
              icon: const Icon(Icons.shopping_cart_outlined),
              label: const Text('ใส่ตะกร้า'),
              style: OutlinedButton.styleFrom(
                minimumSize: const Size(0, 48),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: FilledButton(
              onPressed: product.isInStock
                  ? () => _buyNow(context, ref)
                  : null,
              style: FilledButton.styleFrom(
                minimumSize: const Size(0, 48),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
              child: Text(product.isInStock ? 'ซื้อเลย' : 'สินค้าหมด'),
            ),
          ),
        ],
      ),
    );
  }
  
  Future<void> _addToCart(BuildContext context, WidgetRef ref) async {
    try {
      await ref.read(cartNotifierProvider.notifier).addItem(
        product.id,
        quantity,
      );
      
      if (context.mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('เพิ่มสินค้าในตะกร้าแล้ว'),
            behavior: SnackBarBehavior.floating,
          ),
        );
      }
    } catch (e) {
      if (context.mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('$e'),
            backgroundColor: Colors.red,
          ),
        );
      }
    }
  }
  
  Future<void> _buyNow(BuildContext context, WidgetRef ref) async {
    await _addToCart(context, ref);
    if (context.mounted) {
      context.push('/checkout');
    }
  }
}
```

---

## สรุปหลักสูตร Flutter 50 บท

ยินดีด้วย! คุณได้เรียนจบหลักสูตร Flutter ทั้ง 50 บทแล้ว นี่คือสิ่งที่คุณได้เรียนรู้:

### Foundations (Part 1-10)
- Dart language basics, OOP, async/await
- Flutter widgets, layouts, navigation
- State management เบื้องต้น
- HTTP networking, JSON parsing

### Intermediate (Part 11-20)
- ListView, GridView, Custom ScrollView
- Forms & Validation
- Local storage (SQLite, SharedPreferences)
- Firebase integration

### Advanced (Part 21-30)
- BLoC pattern, Provider, Riverpod
- Advanced animations
- Testing (Unit, Widget, Integration)
- Performance optimization

### Expert (Part 31-40)
- Clean Architecture
- CI/CD pipelines
- App Store deployment
- Internationalization

### Master (Part 41-50)
- Micro Frontend Architecture
- GraphQL integration
- WebSocket & real-time apps
- Offline-first architecture
- Security best practices
- Machine Learning & AI
- AR & Advanced hardware features
- Enterprise Architecture
- Scalable App Architecture
- Complete E-commerce Capstone Project

## แบบฝึกหัด Capstone

1. สร้าง E-commerce app ตามโครงสร้างที่เรียนมา
2. Implement Authentication ด้วย Firebase Auth
3. เชื่อม Firestore สำหรับ Products และ Orders
4. ตั้งค่า Stripe payment integration
5. เพิ่ม Push Notifications ด้วย FCM
6. รองรับ Thai/English localization
7. Implement Dark Mode
8. เขียน Unit + Widget Tests ให้ครอบคลุม 80%+
9. ตั้งค่า CI/CD ด้วย GitHub Actions
10. Deploy ขึ้น App Store & Google Play

---

[⬅️ Part 49](part_49.md) | [🏠 Course Index](../README.md)

---

*หลักสูตร Flutter ฉบับสมบูรณ์ - จาก Hello World สู่ World-class App*

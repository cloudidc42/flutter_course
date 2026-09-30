# Part 32: TDD & Testing
## ขั้นตอนที่ 311-320

---

## สารบัญ
1. [Introduction to TDD](#introduction-to-tdd)
2. [Unit Tests ด้วย flutter_test](#unit-tests)
3. [Widget Tests](#widget-tests)
4. [Integration Tests](#integration-tests)
5. [Mockito & Mocktail](#mockito--mocktail)
6. [Test Fixtures](#test-fixtures)
7. [BDD Style Tests](#bdd-style-tests)
8. [Testing Async Code](#testing-async-code)
9. [Testing Bloc/Provider](#testing-blocprovider)
10. [Coverage Reports & CI Setup](#coverage-reports--ci-setup)

---

## ขั้นตอนที่ 311: Introduction to TDD

TDD (Test-Driven Development) คือการเขียน Test ก่อน แล้วค่อยเขียน Code

### Red-Green-Refactor Cycle

```
TDD Cycle:
┌─────────────────────────────────────────────────┐
│                                                 │
│   1. RED    → เขียน Test ที่ fail ก่อน          │
│       ↓                                         │
│   2. GREEN  → เขียน Code น้อยที่สุดให้ pass     │
│       ↓                                         │
│   3. REFACTOR → ปรับปรุง Code โดยไม่ให้ Test fail│
│       ↓                                         │
│   (repeat)                                      │
│                                                 │
└─────────────────────────────────────────────────┘
```

### ประโยชน์ของ TDD

```
ประโยชน์:
✅ ได้ Documentation ที่ Update เสมอ (Tests คือ Specs)
✅ Confidence เวลา Refactor
✅ Design ดีขึ้น (Testable = Good Design)
✅ Bug น้อยลง
✅ Regression Protection

โทษของการไม่มี Tests:
❌ กลัวการ Refactor
❌ Bug ซ้ำๆ
❌ Code เปราะ (Brittle)
❌ ต้อง Test Manual ทุกครั้ง
```

### การตั้งค่า pubspec.yaml สำหรับ Testing

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  
  # Unit Testing
  mockito: ^5.4.3
  mocktail: ^1.0.3
  
  # BLoC Testing
  bloc_test: ^9.1.5
  
  # Integration Testing
  integration_test:
    sdk: flutter
  patrol: ^3.6.1
  
  # Code Coverage
  coverage: ^1.6.4
  
  # Matchers
  matcher: ^0.12.16
```

---

## ขั้นตอนที่ 312: Unit Tests

Unit Tests ทดสอบ Logic เดี่ยวๆ โดยไม่ต้องมี UI

```dart
// test/features/product/domain/entities/product_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/features/product/domain/entities/product.dart';

void main() {
  group('Product Entity', () {
    late Product product;

    setUp(() {
      product = const Product(
        id: '1',
        name: 'Test Product',
        price: 100.0,
        stockQuantity: 10,
        category: 'Electronics',
        tags: ['new', 'featured'],
      );
    });

    group('isInStock', () {
      test('should return true when stockQuantity > 0', () {
        expect(product.isInStock, true);
      });

      test('should return false when stockQuantity is 0', () {
        final outOfStockProduct = product.copyWith(stockQuantity: 0);
        expect(outOfStockProduct.isInStock, false);
      });
    });

    group('isLowStock', () {
      test('should return true when stockQuantity is between 1 and 5', () {
        final lowStockProduct = product.copyWith(stockQuantity: 3);
        expect(lowStockProduct.isLowStock, true);
      });

      test('should return false when stockQuantity is more than 5', () {
        expect(product.isLowStock, false);
      });

      test('should return false when stockQuantity is 0', () {
        final outOfStockProduct = product.copyWith(stockQuantity: 0);
        expect(outOfStockProduct.isLowStock, false);
      });
    });

    group('discountedPrice', () {
      test('should apply 10% discount when product has sale tag', () {
        final saleProduct = product.copyWith(tags: ['sale', 'new']);
        expect(saleProduct.discountedPrice, 90.0);
      });

      test('should return original price when no sale tag', () {
        expect(product.discountedPrice, 100.0);
      });
    });

    group('hasTag', () {
      test('should return true for existing tag', () {
        expect(product.hasTag('new'), true);
      });

      test('should return false for non-existing tag', () {
        expect(product.hasTag('sale'), false);
      });

      test('should be case-insensitive', () {
        expect(product.hasTag('NEW'), true);
        expect(product.hasTag('Featured'), true);
      });
    });

    group('equality', () {
      test('products with same id should be equal', () {
        final sameProduct = product.copyWith(name: 'Different Name');
        expect(product, sameProduct);
      });

      test('products with different id should not be equal', () {
        final differentProduct = product.copyWith(id: '2');
        expect(product, isNot(equals(differentProduct)));
      });
    });
  });
}
```

```dart
// test/features/product/domain/usecases/search_products_test.dart
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:my_app/core/error/failures.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import 'package:my_app/features/product/domain/repositories/product_repository.dart';
import 'package:my_app/features/product/domain/usecases/search_products.dart';

import 'search_products_test.mocks.dart';

@GenerateMocks([ProductRepository])
void main() {
  late SearchProducts useCase;
  late MockProductRepository mockRepository;

  setUp(() {
    mockRepository = MockProductRepository();
    useCase = SearchProducts(mockRepository);
  });

  final tProducts = [
    const Product(
      id: '1',
      name: 'iPhone',
      price: 999.0,
      stockQuantity: 5,
      category: 'Electronics',
      tags: ['apple'],
    ),
  ];

  group('SearchProducts UseCase', () {
    test('should return products from repository on valid query', () async {
      // Arrange
      when(mockRepository.searchProducts(any))
          .thenAnswer((_) async => Right(tProducts));

      // Act
      final result = await useCase(const SearchProductsParams('iPhone'));

      // Assert
      expect(result, Right(tProducts));
      verify(mockRepository.searchProducts('iPhone')).called(1);
      verifyNoMoreInteractions(mockRepository);
    });

    test('should return ValidationFailure when query is empty', () async {
      // Act
      final result = await useCase(const SearchProductsParams(''));

      // Assert
      expect(result, isA<Left<Failure, List<Product>>>());
      result.fold(
        (failure) => expect(failure, isA<ValidationFailure>()),
        (_) => fail('Should have returned failure'),
      );
      verifyZeroInteractions(mockRepository);
    });

    test('should return ValidationFailure when query is less than 2 chars', () async {
      // Act
      final result = await useCase(const SearchProductsParams('i'));

      // Assert
      expect(result, isA<Left>());
      result.fold(
        (failure) => expect(failure, isA<ValidationFailure>()),
        (_) => fail('Should have returned failure'),
      );
      verifyZeroInteractions(mockRepository);
    });

    test('should trim query before searching', () async {
      // Arrange
      when(mockRepository.searchProducts('iPhone'))
          .thenAnswer((_) async => Right(tProducts));

      // Act
      await useCase(const SearchProductsParams('  iPhone  '));

      // Assert
      verify(mockRepository.searchProducts('iPhone')).called(1);
    });

    test('should return ServerFailure when repository fails', () async {
      // Arrange
      when(mockRepository.searchProducts(any))
          .thenAnswer((_) async => const Left(ServerFailure('Server error')));

      // Act
      final result = await useCase(const SearchProductsParams('iPhone'));

      // Assert
      expect(result, const Left(ServerFailure('Server error')));
    });
  });
}
```

---

## ขั้นตอนที่ 313: Widget Tests

Widget Tests ทดสอบ UI Components

```dart
// test/features/product/presentation/widgets/product_card_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import 'package:my_app/features/product/presentation/widgets/product_card.dart';

void main() {
  const tProduct = Product(
    id: '1',
    name: 'Test Product',
    price: 100.0,
    stockQuantity: 5,
    category: 'Electronics',
    tags: ['new'],
  );

  Widget makeTestableWidget(Widget child) {
    return MaterialApp(
      home: Scaffold(body: child),
    );
  }

  group('ProductCard Widget', () {
    testWidgets('should display product name', (WidgetTester tester) async {
      // Arrange & Act
      await tester.pumpWidget(
        makeTestableWidget(
          ProductCard(
            product: tProduct,
            onTap: () {},
          ),
        ),
      );

      // Assert
      expect(find.text('Test Product'), findsOneWidget);
    });

    testWidgets('should display product price', (WidgetTester tester) async {
      await tester.pumpWidget(
        makeTestableWidget(
          ProductCard(product: tProduct, onTap: () {}),
        ),
      );

      expect(find.text('฿100.00'), findsOneWidget);
    });

    testWidgets('should show out of stock when stockQuantity is 0', (WidgetTester tester) async {
      const outOfStockProduct = Product(
        id: '2',
        name: 'OOS Product',
        price: 50.0,
        stockQuantity: 0,
        category: 'Electronics',
        tags: [],
      );

      await tester.pumpWidget(
        makeTestableWidget(
          ProductCard(product: outOfStockProduct, onTap: () {}),
        ),
      );

      expect(find.text('Out of Stock'), findsOneWidget);
    });

    testWidgets('should call onTap when tapped', (WidgetTester tester) async {
      bool tapped = false;

      await tester.pumpWidget(
        makeTestableWidget(
          ProductCard(
            product: tProduct,
            onTap: () => tapped = true,
          ),
        ),
      );

      await tester.tap(find.byType(ProductCard));
      await tester.pump();

      expect(tapped, true);
    });

    testWidgets('should show sale badge when product has sale tag', (WidgetTester tester) async {
      const saleProduct = Product(
        id: '3',
        name: 'Sale Product',
        price: 100.0,
        stockQuantity: 10,
        category: 'Electronics',
        tags: ['sale'],
      );

      await tester.pumpWidget(
        makeTestableWidget(
          ProductCard(product: saleProduct, onTap: () {}),
        ),
      );

      expect(find.text('SALE'), findsOneWidget);
    });
  });
}
```

```dart
// test/features/product/presentation/pages/product_list_page_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import 'package:my_app/features/product/presentation/bloc/product_bloc.dart';
import 'package:my_app/features/product/presentation/bloc/product_event.dart';
import 'package:my_app/features/product/presentation/bloc/product_state.dart';
import 'package:my_app/features/product/presentation/pages/product_list_page.dart';

class MockProductBloc extends MockBloc<ProductEvent, ProductState>
    implements ProductBloc {}

void main() {
  late MockProductBloc mockBloc;

  setUp(() {
    mockBloc = MockProductBloc();
  });

  final tProducts = List.generate(
    5,
    (i) => Product(
      id: '$i',
      name: 'Product $i',
      price: (i + 1) * 10.0,
      stockQuantity: i * 2,
      category: 'Test',
      tags: const [],
    ),
  );

  Widget makeTestableWidget() {
    return MaterialApp(
      home: BlocProvider<ProductBloc>.value(
        value: mockBloc,
        child: const ProductListPage(),
      ),
    );
  }

  group('ProductListPage', () {
    testWidgets('should show loading indicator when state is loading',
        (WidgetTester tester) async {
      when(() => mockBloc.state).thenReturn(const ProductLoading());

      await tester.pumpWidget(makeTestableWidget());

      expect(find.byType(CircularProgressIndicator), findsOneWidget);
    });

    testWidgets('should show list of products when state is loaded',
        (WidgetTester tester) async {
      when(() => mockBloc.state).thenReturn(
        ProductLoaded(products: tProducts),
      );

      await tester.pumpWidget(makeTestableWidget());
      await tester.pump();

      expect(find.byType(ListView), findsOneWidget);
      expect(find.text('Product 0'), findsOneWidget);
      expect(find.text('Product 4'), findsOneWidget);
    });

    testWidgets('should show error message when state is error',
        (WidgetTester tester) async {
      when(() => mockBloc.state).thenReturn(
        const ProductError('Something went wrong'),
      );

      await tester.pumpWidget(makeTestableWidget());

      expect(find.text('Something went wrong'), findsOneWidget);
      expect(find.text('Retry'), findsOneWidget);
    });

    testWidgets('should dispatch LoadProductsEvent on retry tap',
        (WidgetTester tester) async {
      when(() => mockBloc.state).thenReturn(
        const ProductError('Error'),
      );

      await tester.pumpWidget(makeTestableWidget());
      await tester.tap(find.text('Retry'));
      await tester.pump();

      verify(() => mockBloc.add(const LoadProductsEvent(refresh: true))).called(1);
    });

    testWidgets('should show empty state when no products', (WidgetTester tester) async {
      when(() => mockBloc.state).thenReturn(
        const ProductLoaded(products: []),
      );

      await tester.pumpWidget(makeTestableWidget());

      expect(find.text('No products found'), findsOneWidget);
    });
  });
}
```

---

## ขั้นตอนที่ 314: Integration Tests

Integration Tests ทดสอบการทำงานร่วมกันของทุก Component

```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('App Integration Tests', () {
    testWidgets('full product search flow', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle(const Duration(seconds: 3));

      // ตรวจสอบ Home Page โหลดสำเร็จ
      expect(find.text('Products'), findsOneWidget);

      // กด Search
      await tester.tap(find.byIcon(Icons.search));
      await tester.pumpAndSettle();

      // พิมพ์ search query
      await tester.enterText(find.byType(TextField), 'iPhone');
      await tester.pumpAndSettle(const Duration(seconds: 2));

      // ตรวจสอบว่า search results แสดงขึ้นมา
      expect(find.textContaining('iPhone'), findsWidgets);
    });

    testWidgets('navigate to product detail', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle(const Duration(seconds: 3));

      // แตะที่ product แรก
      await tester.tap(find.byType(ProductCard).first);
      await tester.pumpAndSettle();

      // ตรวจสอบว่าอยู่ที่ Detail Page
      expect(find.text('Product Details'), findsOneWidget);
      expect(find.byType(BackButton), findsOneWidget);
    });
  });
}
```

```dart
// integration_test/patrol_test.dart (ใช้ patrol package)
import 'package:flutter_test/flutter_test.dart';
import 'package:patrol/patrol.dart';
import 'package:my_app/main.dart' as app;

void main() {
  patrolTest(
    'Complete user journey test',
    ($) async {
      app.main();
      await $.pumpAndSettle();

      // Login Flow
      await $(#emailField).enterText('test@example.com');
      await $(#passwordField).enterText('password123');
      await $(#loginButton).tap();
      await $.pumpAndSettle(const Duration(seconds: 2));

      // ตรวจสอบว่า Login สำเร็จ
      expect($(#homeScreen), findsOneWidget);

      // Navigation
      await $(NavigationBar).tap(find.text('Products'));
      await $.pumpAndSettle();

      expect($('Products'), findsOneWidget);
    },
  );
}
```

---

## ขั้นตอนที่ 315: Mockito & Mocktail

### ใช้ Mockito

```dart
// ต้องสร้าง Mock ด้วย @GenerateMocks annotation
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

@GenerateMocks([ProductRepository, NetworkInfo])
void main() {}

// รัน: flutter pub run build_runner build

// ใช้งาน Mock
class ProductRepositoryTest {
  test('example with mockito', () async {
    final mockRepo = MockProductRepository();
    
    // กำหนด behavior
    when(mockRepo.getProducts())
        .thenAnswer((_) async => Right([product1, product2]));
    
    // ทดสอบ
    final result = await mockRepo.getProducts();
    
    // Verify
    verify(mockRepo.getProducts()).called(1);
    verifyNoMoreInteractions(mockRepo);
  });
}
```

### ใช้ Mocktail (ไม่ต้อง Code Generation)

```dart
// Mocktail ง่ายกว่า ไม่ต้อง generate
import 'package:mocktail/mocktail.dart';

class MockProductRepository extends Mock implements ProductRepository {}
class MockNetworkInfo extends Mock implements NetworkInfo {}

void main() {
  late MockProductRepository mockRepository;
  late MockNetworkInfo mockNetworkInfo;

  setUpAll(() {
    // Register fallback values สำหรับ custom types
    registerFallbackValue(const GetProductsParams());
    registerFallbackValue(const Product(
      id: '',
      name: '',
      price: 0,
      stockQuantity: 0,
      category: '',
      tags: [],
    ));
  });

  setUp(() {
    mockRepository = MockProductRepository();
    mockNetworkInfo = MockNetworkInfo();
  });

  group('ProductRepository', () {
    test('should return products when online', () async {
      // Arrange
      when(() => mockNetworkInfo.isConnected).thenAnswer((_) async => true);
      when(() => mockRepository.getProducts())
          .thenAnswer((_) async => Right(tProducts));

      // Act
      final result = await mockRepository.getProducts();

      // Assert
      expect(result, Right(tProducts));
    });

    test('should throw when called with wrong params', () async {
      // ทดสอบว่า throw exception
      when(() => mockRepository.getProductById(any()))
          .thenThrow(const ServerException(message: 'Server Error'));

      expect(
        () => mockRepository.getProductById('invalid_id'),
        throwsA(isA<ServerException>()),
      );
    });

    test('matchers ต่างๆ', () async {
      when(() => mockRepository.searchProducts(
        any(that: isA<String>()),
      )).thenAnswer((_) async => Right(tProducts));

      // ใช้ captureAny เพื่อ capture argument
      when(() => mockRepository.searchProducts(
        captureAny(),
      )).thenAnswer((_) async => Right(tProducts));

      await mockRepository.searchProducts('test');

      final captured = verify(
        () => mockRepository.searchProducts(captureAny()),
      ).captured;
      
      expect(captured.first, 'test');
    });
  });
}
```

---

## ขั้นตอนที่ 316: Test Fixtures

```dart
// test/fixtures/fixture_reader.dart
import 'dart:io';

String fixture(String name) =>
    File('test/fixtures/$name').readAsStringSync();
```

```json
// test/fixtures/products.json
{
  "products": [
    {
      "id": "1",
      "name": "iPhone 15",
      "price": 999.00,
      "stock_quantity": 10,
      "category": "Electronics",
      "tags": ["apple", "smartphone"],
      "created_at": "2024-01-01T00:00:00.000Z"
    },
    {
      "id": "2",
      "name": "Samsung S24",
      "price": 799.00,
      "stock_quantity": 0,
      "category": "Electronics",
      "tags": ["samsung", "smartphone"],
      "created_at": "2024-01-02T00:00:00.000Z"
    }
  ]
}
```

```dart
// test/features/product/data/models/product_model_test.dart
import 'dart:convert';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/features/product/data/models/product_model.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import '../../../fixtures/fixture_reader.dart';

void main() {
  const tProductModel = ProductModel(
    id: '1',
    name: 'iPhone 15',
    price: 999.00,
    stockQuantity: 10,
    category: 'Electronics',
    tags: ['apple', 'smartphone'],
  );

  group('ProductModel', () {
    test('should be a subclass of Product entity', () {
      expect(tProductModel, isA<Product>());
    });

    group('fromJson', () {
      test('should return a valid model from JSON', () {
        // Arrange
        final Map<String, dynamic> jsonMap = json.decode(
          fixture('products.json'),
        )['products'][0];

        // Act
        final result = ProductModel.fromJson(jsonMap);

        // Assert
        expect(result, tProductModel);
      });

      test('should handle missing optional fields', () {
        final jsonMap = {
          'id': '1',
          'name': 'Test',
          'price': 100.0,
          'stock_quantity': 5,
          'category': 'Test',
          'tags': [],
        };

        expect(
          () => ProductModel.fromJson(jsonMap),
          returnsNormally,
        );
      });
    });

    group('toJson', () {
      test('should return a JSON map containing proper data', () {
        // Act
        final result = tProductModel.toJson();

        // Assert
        expect(result['id'], '1');
        expect(result['name'], 'iPhone 15');
        expect(result['price'], 999.00);
        expect(result['stock_quantity'], 10);
        expect(result['category'], 'Electronics');
        expect(result['tags'], ['apple', 'smartphone']);
      });
    });
  });
}
```

---

## ขั้นตอนที่ 317: BDD Style Tests

BDD (Behavior Driven Development) เขียน Test แบบ Given-When-Then

```dart
// test/features/product/domain/usecases/get_products_bdd_test.dart
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import 'package:my_app/features/product/domain/repositories/product_repository.dart';
import 'package:my_app/features/product/domain/usecases/get_products.dart';

class MockProductRepository extends Mock implements ProductRepository {}

void main() {
  late GetProducts useCase;
  late MockProductRepository mockRepository;

  setUp(() {
    mockRepository = MockProductRepository();
    useCase = GetProducts(mockRepository);
  });

  group('GetProducts', () {
    // Feature: Get Products
    // As a user
    // I want to see a list of products
    // So that I can browse and purchase items

    group('Scenario: User loads products successfully', () {
      // Given the user has internet connection
      // And the server is available
      // When the user opens the product list
      // Then they should see a list of products

      final tProducts = [
        const Product(
          id: '1',
          name: 'iPhone',
          price: 999.0,
          stockQuantity: 5,
          category: 'Electronics',
          tags: [],
        ),
      ];

      test('GIVEN valid connection WHEN loading products THEN returns product list',
          () async {
        // Given
        when(() => mockRepository.getProducts())
            .thenAnswer((_) async => Right(tProducts));

        // When
        final result = await useCase(const GetProductsParams());

        // Then
        expect(result, Right(tProducts));
        verify(() => mockRepository.getProducts()).called(1);
      });
    });

    group('Scenario: Server is unavailable', () {
      // Given the user has internet connection
      // But the server is down
      // When the user opens the product list
      // Then they should see an error message

      test('GIVEN server error WHEN loading products THEN returns server failure',
          () async {
        // Given
        when(() => mockRepository.getProducts()).thenAnswer(
          (_) async => const Left(ServerFailure('Server unavailable')),
        );

        // When
        final result = await useCase(const GetProductsParams());

        // Then
        expect(result, isA<Left>());
        result.fold(
          (failure) {
            expect(failure, isA<ServerFailure>());
            expect(failure.message, 'Server unavailable');
          },
          (_) => fail('Expected failure'),
        );
      });
    });

    group('Scenario: No internet connection', () {
      // Given the user has no internet connection
      // When the user opens the product list
      // Then they should see cached data or offline message

      test('GIVEN no internet WHEN loading products THEN returns cached or network failure',
          () async {
        // Given
        when(() => mockRepository.getProducts()).thenAnswer(
          (_) async => const Left(NetworkFailure('No internet')),
        );

        // When
        final result = await useCase(const GetProductsParams());

        // Then
        expect(result, isA<Left>());
        result.fold(
          (failure) => expect(failure, isA<NetworkFailure>()),
          (_) => fail('Expected failure'),
        );
      });
    });
  });
}
```

---

## ขั้นตอนที่ 318: Testing Async Code

```dart
// test/core/network/network_info_test.dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:my_app/core/network/network_info.dart';

class MockConnectivity extends Mock implements Connectivity {}

void main() {
  late NetworkInfoImpl networkInfo;
  late MockConnectivity mockConnectivity;

  setUp(() {
    mockConnectivity = MockConnectivity();
    networkInfo = NetworkInfoImpl(mockConnectivity);
  });

  group('NetworkInfo', () {
    group('isConnected', () {
      test('should return true when connectivity is wifi', () async {
        // Arrange
        when(() => mockConnectivity.checkConnectivity())
            .thenAnswer((_) async => ConnectivityResult.wifi);

        // Act
        final result = await networkInfo.isConnected;

        // Assert
        expect(result, true);
        verify(() => mockConnectivity.checkConnectivity()).called(1);
      });

      test('should return true when connectivity is mobile', () async {
        when(() => mockConnectivity.checkConnectivity())
            .thenAnswer((_) async => ConnectivityResult.mobile);

        final result = await networkInfo.isConnected;
        expect(result, true);
      });

      test('should return false when no connectivity', () async {
        when(() => mockConnectivity.checkConnectivity())
            .thenAnswer((_) async => ConnectivityResult.none);

        final result = await networkInfo.isConnected;
        expect(result, false);
      });
    });
  });
}
```

```dart
// test/helpers/async_helpers.dart
// Helper functions สำหรับ test async code

// ทดสอบ Stream
import 'package:flutter_test/flutter_test.dart';
import 'dart:async';

void testStream() {
  test('should emit values in correct order', () async {
    final controller = StreamController<int>();
    
    // เพิ่ม values
    controller.add(1);
    controller.add(2);
    controller.add(3);
    controller.close();

    // ตรวจสอบ stream values
    expect(
      controller.stream,
      emitsInOrder([1, 2, 3, emitsDone]),
    );
  });

  test('should handle stream errors', () async {
    final controller = StreamController<int>();
    
    controller.addError('Stream Error');
    controller.close();

    expect(
      controller.stream,
      emitsError('Stream Error'),
    );
  });
}

// ทดสอบ Future ที่ throw Exception
void testException() {
  test('should throw when validation fails', () async {
    final useCase = SomeUseCase();
    
    expect(
      () async => await useCase.execute(''),
      throwsA(isA<ValidationException>()),
    );
    
    // หรือใช้ expectLater
    await expectLater(
      useCase.execute('invalid'),
      throwsA(
        isA<ServerException>().having(
          (e) => e.statusCode,
          'statusCode',
          404,
        ),
      ),
    );
  });
}

// ทดสอบ Future ที่ timeout
void testTimeout() {
  test('should complete within timeout', () async {
    final result = await Future.delayed(
      const Duration(milliseconds: 100),
      () => 'success',
    ).timeout(const Duration(seconds: 5));

    expect(result, 'success');
  });
}
```

---

## ขั้นตอนที่ 319: Testing Bloc/Provider

```dart
// test/features/product/presentation/bloc/product_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:dartz/dartz.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:my_app/core/error/failures.dart';
import 'package:my_app/features/product/domain/entities/product.dart';
import 'package:my_app/features/product/domain/usecases/get_products.dart';
import 'package:my_app/features/product/domain/usecases/get_product_by_id.dart';
import 'package:my_app/features/product/domain/usecases/search_products.dart';
import 'package:my_app/features/product/presentation/bloc/product_bloc.dart';
import 'package:my_app/features/product/presentation/bloc/product_event.dart';
import 'package:my_app/features/product/presentation/bloc/product_state.dart';

class MockGetProducts extends Mock implements GetProducts {}
class MockGetProductById extends Mock implements GetProductById {}
class MockSearchProducts extends Mock implements SearchProducts {}

void main() {
  late ProductBloc bloc;
  late MockGetProducts mockGetProducts;
  late MockGetProductById mockGetProductById;
  late MockSearchProducts mockSearchProducts;

  setUpAll(() {
    registerFallbackValue(const GetProductsParams());
    registerFallbackValue(GetProductByIdParams(''));
    registerFallbackValue(SearchProductsParams(''));
  });

  setUp(() {
    mockGetProducts = MockGetProducts();
    mockGetProductById = MockGetProductById();
    mockSearchProducts = MockSearchProducts();
    
    bloc = ProductBloc(
      getProducts: mockGetProducts,
      getProductById: mockGetProductById,
      searchProducts: mockSearchProducts,
    );
  });

  tearDown(() {
    bloc.close();
  });

  final tProducts = List.generate(
    3,
    (i) => Product(
      id: '$i',
      name: 'Product $i',
      price: (i + 1) * 100.0,
      stockQuantity: i * 3,
      category: 'Test',
      tags: const [],
    ),
  );

  group('ProductBloc', () {
    test('initial state should be ProductInitial', () {
      expect(bloc.state, const ProductInitial());
    });

    group('LoadProductsEvent', () {
      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, Loaded] when products are loaded successfully',
        build: () {
          when(() => mockGetProducts(any()))
              .thenAnswer((_) async => Right(tProducts));
          return bloc;
        },
        act: (bloc) => bloc.add(const LoadProductsEvent()),
        expect: () => [
          const ProductLoading(),
          ProductLoaded(
            products: tProducts,
            hasMore: false,
            currentPage: 1,
          ),
        ],
        verify: (_) {
          verify(() => mockGetProducts(const GetProductsParams())).called(1);
        },
      );

      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, Error] when loading products fails',
        build: () {
          when(() => mockGetProducts(any())).thenAnswer(
            (_) async => const Left(ServerFailure('Server Error')),
          );
          return bloc;
        },
        act: (bloc) => bloc.add(const LoadProductsEvent()),
        expect: () => [
          const ProductLoading(),
          const ProductError('Server Error'),
        ],
      );

      blocTest<ProductBloc, ProductState>(
        'should not reload if already loaded and not refreshing',
        build: () => bloc,
        seed: () => ProductLoaded(products: tProducts),
        act: (bloc) => bloc.add(const LoadProductsEvent()),
        expect: () => [],
      );

      blocTest<ProductBloc, ProductState>(
        'should reload when refresh is true',
        build: () {
          when(() => mockGetProducts(any()))
              .thenAnswer((_) async => Right(tProducts));
          return bloc;
        },
        seed: () => ProductLoaded(products: tProducts),
        act: (bloc) => bloc.add(const LoadProductsEvent(refresh: true)),
        expect: () => [
          const ProductLoading(),
          ProductLoaded(products: tProducts),
        ],
      );
    });

    group('SearchProductsEvent', () {
      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, Loaded] when search is successful',
        build: () {
          when(() => mockSearchProducts(any()))
              .thenAnswer((_) async => Right(tProducts));
          return bloc;
        },
        act: (bloc) => bloc.add(const SearchProductsEvent('iPhone')),
        wait: const Duration(milliseconds: 600), // debounce wait
        expect: () => [
          const ProductLoading(),
          ProductLoaded(
            products: tProducts,
            searchQuery: 'iPhone',
          ),
        ],
      );

      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, Error] when search fails',
        build: () {
          when(() => mockSearchProducts(any())).thenAnswer(
            (_) async => const Left(ValidationFailure('Query too short')),
          );
          return bloc;
        },
        act: (bloc) => bloc.add(const SearchProductsEvent('i')),
        wait: const Duration(milliseconds: 600),
        expect: () => [
          const ProductLoading(),
          const ProductError('Query too short'),
        ],
      );
    });

    group('GetProductByIdEvent', () {
      const tProduct = Product(
        id: '1',
        name: 'iPhone',
        price: 999.0,
        stockQuantity: 5,
        category: 'Electronics',
        tags: [],
      );

      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, DetailLoaded] when product is found',
        build: () {
          when(() => mockGetProductById(any()))
              .thenAnswer((_) async => const Right(tProduct));
          return bloc;
        },
        act: (bloc) => bloc.add(const GetProductByIdEvent('1')),
        expect: () => [
          const ProductLoading(),
          const ProductDetailLoaded(tProduct),
        ],
      );

      blocTest<ProductBloc, ProductState>(
        'should emit [Loading, Error] when product not found',
        build: () {
          when(() => mockGetProductById(any())).thenAnswer(
            (_) async => const Left(NotFoundFailure('Product not found')),
          );
          return bloc;
        },
        act: (bloc) => bloc.add(const GetProductByIdEvent('999')),
        expect: () => [
          const ProductLoading(),
          const ProductError('Product not found'),
        ],
      );
    });
  });
}
```

---

## ขั้นตอนที่ 320: Coverage Reports & CI Setup

### การรัน Tests ด้วย Coverage

```bash
# รัน unit tests ด้วย coverage
flutter test --coverage

# แปลง coverage report เป็น HTML
genhtml coverage/lcov.info -o coverage/html

# เปิด report ใน browser
open coverage/html/index.html
```

### การตั้งค่า GitHub Actions CI

```yaml
# .github/workflows/test.yml
name: Flutter Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          channel: 'stable'
          cache: true

      - name: Get dependencies
        run: flutter pub get

      - name: Run code generation
        run: flutter pub run build_runner build --delete-conflicting-outputs

      - name: Verify formatting
        run: dart format --output=none --set-exit-if-changed .

      - name: Analyze code
        run: flutter analyze --fatal-infos

      - name: Run unit and widget tests
        run: flutter test --coverage --reporter=expanded

      - name: Check coverage threshold
        uses: VeryGoodOpenSource/very_good_coverage@v2
        with:
          min_coverage: 80

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info

  integration_test:
    name: Integration Tests
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
          channel: 'stable'

      - name: Get dependencies
        run: flutter pub get

      - name: Run integration tests on iOS Simulator
        run: |
          open -a Simulator
          xcrun simctl list devices | grep "iPhone 15"
          flutter test integration_test/app_test.dart \
            -d "iPhone 15"
```

### Makefile สำหรับ Development

```makefile
# Makefile
.PHONY: test test-unit test-widget test-integration coverage clean

test-unit:
	flutter test test/ --exclude-tags integration

test-widget:
	flutter test test/ --tags widget

test-integration:
	flutter test integration_test/

test: test-unit test-widget

coverage:
	flutter test --coverage
	genhtml coverage/lcov.info -o coverage/html
	open coverage/html/index.html

analyze:
	flutter analyze

format:
	dart format lib/ test/

generate:
	flutter pub run build_runner build --delete-conflicting-outputs

clean:
	flutter clean
	flutter pub get
```

### Workshop: TDD ตั้งแต่เริ่มต้น

```dart
// WORKSHOP: TDD สำหรับ Calculator Feature

// Step 1: เขียน Test ก่อน (RED)
// test/calculator_test.dart
void main() {
  group('Calculator', () {
    test('should add two numbers correctly', () {
      final calculator = Calculator();
      expect(calculator.add(2, 3), 5);
    });

    test('should subtract correctly', () {
      final calculator = Calculator();
      expect(calculator.subtract(5, 3), 2);
    });

    test('should throw when dividing by zero', () {
      final calculator = Calculator();
      expect(
        () => calculator.divide(10, 0),
        throwsA(isA<ArgumentError>()),
      );
    });
  });
}

// Step 2: เขียน Code (GREEN)
// lib/calculator.dart
class Calculator {
  double add(double a, double b) => a + b;
  double subtract(double a, double b) => a - b;
  double multiply(double a, double b) => a * b;
  
  double divide(double a, double b) {
    if (b == 0) throw ArgumentError('Cannot divide by zero');
    return a / b;
  }
}

// Step 3: Refactor (ไม่กระทบ Tests)
// เพิ่ม History feature
class Calculator {
  final List<String> _history = [];
  List<String> get history => List.unmodifiable(_history);

  double add(double a, double b) {
    final result = a + b;
    _history.add('$a + $b = $result');
    return result;
  }
  
  // ...
}
```

---

## สรุป (Summary)

```
Testing Pyramid:
                   ┌──────────┐
                   │    E2E   │  (น้อย - แพง - ช้า)
                  ─┴──────────┴─
                 ─┴────────────┴─
                 │  Integration │  (ปานกลาง)
                ─┴──────────────┴─
               ─┴────────────────┴─
               │    Unit Tests    │  (เยอะ - ถูก - เร็ว)
              ─┴──────────────────┴─

Best Practices:
- AAA Pattern: Arrange, Act, Assert
- One assertion per test (ideally)
- Descriptive test names: "should [expected] when [condition]"
- Test behavior, not implementation
- Use test doubles (Mock, Stub, Fake) appropriately
- Aim for 80%+ coverage
- Tests should be deterministic (no flakiness)
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. เขียน Unit Tests สำหรับ `CartEntity` (add, remove, total)
2. เขียน Widget Tests สำหรับ `LoginForm` widget
3. Mock `AuthRepository` และเขียน tests สำหรับ `LoginUseCase`

**ระดับกลาง:**
4. เขียน BLoC tests สำหรับ `AuthBloc` (login, logout, register)
5. สร้าง Test Fixtures สำหรับ JSON responses
6. เขียน Integration Test สำหรับ Login Flow

**ระดับสูง:**
7. Setup GitHub Actions CI/CD สำหรับ run tests อัตโนมัติ
8. เพิ่ม Coverage threshold check ใน CI
9. สร้าง Custom Matcher สำหรับ Testing

---

## การนำทาง
- [← Part 31: Clean Architecture](part_31.md)
- [→ Part 33: CI/CD Pipeline](part_33.md)

# Part 48: Enterprise Architecture
## ขั้นตอนที่ 471-480

---

## สารบัญ
1. [SOLID Principles ใน Flutter](#ขั้นตอนที่-471-solid-principles)
2. [Dependency Injection Containers](#ขั้นตอนที่-472-dependency-injection)
3. [Event-driven Architecture](#ขั้นตอนที่-473-event-driven)
4. [CQRS Pattern](#ขั้นตอนที่-474-cqrs-pattern)
5. [Microservices Communication](#ขั้นตอนที่-475-microservices)
6. [Enterprise SSO/SAML Integration](#ขั้นตอนที่-476-sso-saml)
7. [Audit Logging](#ขั้นตอนที่-477-audit-logging)
8. [Multi-tenancy Support](#ขั้นตอนที่-478-multi-tenancy)
9. [Enterprise Testing Strategies](#ขั้นตอนที่-479-enterprise-testing)
10. [Workshop: Enterprise App Architecture](#ขั้นตอนที่-480-workshop)

---

## ขั้นตอนที่ 471: SOLID Principles ใน Flutter

### S - Single Responsibility Principle (SRP)

```dart
// ❌ ผิด - class มีหลาย responsibilities
class UserManager {
  Future<void> createUser(User user) async { /* ... */ }
  Future<void> sendWelcomeEmail(User user) async { /* ... */ }
  Future<void> saveToDatabase(User user) async { /* ... */ }
  Future<void> logActivity(String activity) async { /* ... */ }
  void validateUser(User user) { /* ... */ }
}

// ✅ ถูก - แยก responsibility ออกมา
class UserService {
  final UserRepository _repository;
  final EmailService _emailService;
  final ActivityLogger _logger;
  final UserValidator _validator;
  
  UserService(this._repository, this._emailService, this._logger, this._validator);
  
  Future<void> createUser(User user) async {
    _validator.validate(user);
    await _repository.save(user);
    await _emailService.sendWelcome(user);
    await _logger.log('User created: ${user.id}');
  }
}

class UserRepository {
  Future<void> save(User user) async { /* DB logic only */ }
  Future<User?> findById(String id) async { /* DB logic only */ }
}

class EmailService {
  Future<void> sendWelcome(User user) async { /* Email logic only */ }
}

class ActivityLogger {
  Future<void> log(String activity) async { /* Logging logic only */ }
}

class UserValidator {
  void validate(User user) { /* Validation logic only */ }
}
```

### O - Open/Closed Principle (OCP)

```dart
// ❌ ผิด - ต้อง modify เมื่อเพิ่ม payment method
class PaymentService {
  Future<void> processPayment(String method, double amount) async {
    if (method == 'credit_card') {
      // Process credit card
    } else if (method == 'bank_transfer') {
      // Process bank transfer
    }
    // ต้อง modify เมื่อเพิ่ม PromptPay
  }
}

// ✅ ถูก - Open for extension, Closed for modification
abstract class PaymentProcessor {
  String get name;
  Future<PaymentResult> process(double amount, Map<String, dynamic> details);
}

class CreditCardProcessor implements PaymentProcessor {
  @override
  String get name => 'credit_card';
  
  @override
  Future<PaymentResult> process(double amount, Map<String, dynamic> details) async {
    // Credit card processing
    return PaymentResult.success(transactionId: 'CC_${DateTime.now().millisecondsSinceEpoch}');
  }
}

class BankTransferProcessor implements PaymentProcessor {
  @override
  String get name => 'bank_transfer';
  
  @override
  Future<PaymentResult> process(double amount, Map<String, dynamic> details) async {
    // Bank transfer processing
    return PaymentResult.success(transactionId: 'BT_${DateTime.now().millisecondsSinceEpoch}');
  }
}

class PromptPayProcessor implements PaymentProcessor {
  @override
  String get name => 'promptpay';
  
  @override
  Future<PaymentResult> process(double amount, Map<String, dynamic> details) async {
    // PromptPay processing - เพิ่มโดยไม่ต้อง modify PaymentService
    return PaymentResult.success(transactionId: 'PP_${DateTime.now().millisecondsSinceEpoch}');
  }
}

class PaymentService {
  final Map<String, PaymentProcessor> _processors;
  
  PaymentService(List<PaymentProcessor> processors)
      : _processors = {for (final p in processors) p.name: p};
  
  Future<PaymentResult> process(
    String method,
    double amount,
    Map<String, dynamic> details,
  ) async {
    final processor = _processors[method];
    if (processor == null) {
      throw UnsupportedPaymentMethodException(method);
    }
    return processor.process(amount, details);
  }
  
  void registerProcessor(PaymentProcessor processor) {
    _processors[processor.name] = processor;
  }
}
```

### L - Liskov Substitution Principle (LSP)

```dart
// ✅ ถูก - subclass สามารถแทนที่ base class ได้
abstract class Shape {
  double get area;
  double get perimeter;
}

class Rectangle extends Shape {
  final double width;
  final double height;
  
  Rectangle(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
}

class Circle extends Shape {
  final double radius;
  
  Circle(this.radius);
  
  @override
  double get area => math.pi * radius * radius;
  
  @override
  double get perimeter => 2 * math.pi * radius;
}

// สามารถใช้ Shape แทนที่ Rectangle หรือ Circle ได้
void printShapeInfo(Shape shape) {
  print('Area: ${shape.area}');
  print('Perimeter: ${shape.perimeter}');
}
```

### I - Interface Segregation Principle (ISP)

```dart
// ❌ ผิด - interface ใหญ่เกินไป
abstract class Worker {
  void work();
  void eat();
  void sleep();
  void code();
  void design();
  void manage();
}

// ✅ ถูก - แยก interface ที่เฉพาะเจาะจง
abstract class Workable {
  void work();
}

abstract class Eatable {
  void eat();
}

abstract class Sleepable {
  void sleep();
}

abstract class Codeable {
  void code();
}

// แต่ละ role implement เฉพาะ interface ที่ต้องการ
class Developer implements Workable, Eatable, Sleepable, Codeable {
  @override void work() { }
  @override void eat() { }
  @override void sleep() { }
  @override void code() { }
}
```

### D - Dependency Inversion Principle (DIP)

```dart
// ❌ ผิด - depend on concrete implementation
class OrderService {
  final MySQLOrderRepository _repository = MySQLOrderRepository();
  
  Future<void> createOrder(Order order) async {
    await _repository.save(order);
  }
}

// ✅ ถูก - depend on abstraction
abstract class OrderRepository {
  Future<void> save(Order order);
  Future<Order?> findById(String id);
  Future<List<Order>> findByUserId(String userId);
}

class MySQLOrderRepository implements OrderRepository {
  @override Future<void> save(Order order) async { /* MySQL */ }
  @override Future<Order?> findById(String id) async { /* MySQL */ }
  @override Future<List<Order>> findByUserId(String userId) async { /* MySQL */ }
}

class FirestoreOrderRepository implements OrderRepository {
  @override Future<void> save(Order order) async { /* Firestore */ }
  @override Future<Order?> findById(String id) async { /* Firestore */ }
  @override Future<List<Order>> findByUserId(String userId) async { /* Firestore */ }
}

class OrderService {
  final OrderRepository _repository; // Depend on abstraction
  
  OrderService(this._repository); // Inject dependency
  
  Future<void> createOrder(Order order) async {
    await _repository.save(order);
  }
}
```

---

## ขั้นตอนที่ 472: Dependency Injection Containers

### injectable + get_it

```yaml
dependencies:
  get_it: ^7.6.0
  injectable: ^2.3.0

dev_dependencies:
  injectable_generator: ^2.4.0
  build_runner: ^2.4.0
```

### การ Setup

```dart
// lib/di/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'injection.config.dart';

final getIt = GetIt.instance;

@InjectableInit(
  initializerName: 'init',
  preferRelativeImports: true,
  asExtension: true,
)
Future<void> configureDependencies() async {
  await getIt.init();
}
```

### การ Register Dependencies

```dart
// lib/repositories/user_repository.dart
@injectable
class UserRepositoryImpl implements UserRepository {
  final ApiService _api;
  final LocalDatabase _db;
  
  UserRepositoryImpl(this._api, this._db);
  
  @override
  Future<User?> getUserById(String id) async {
    // Try local first
    final local = await _db.users.findById(id);
    if (local != null) return local;
    
    // Fallback to API
    return _api.getUser(id);
  }
}

// Singleton service
@singleton
class AuthService {
  final UserRepository _userRepository;
  final SecureStorageService _storage;
  
  AuthService(this._userRepository, this._storage);
  
  // ...
}

// Factory - new instance each time
@injectable
class CreateOrderUseCase {
  final OrderRepository _orderRepository;
  final InventoryService _inventoryService;
  
  CreateOrderUseCase(this._orderRepository, this._inventoryService);
  
  // ...
}

// Environment-specific registration
@dev
@injectable
class MockPaymentService implements PaymentService {
  // Mock implementation for development
}

@prod
@injectable
class ProductionPaymentService implements PaymentService {
  // Real implementation for production
}
```

### Module Registration

```dart
// lib/di/app_module.dart
@module
abstract class AppModule {
  @singleton
  @preResolve
  Future<SharedPreferences> prefs() async {
    return SharedPreferences.getInstance();
  }
  
  @singleton
  Dio get dio {
    return Dio(BaseOptions(
      baseUrl: AppConfig.apiUrl,
      connectTimeout: const Duration(seconds: 30),
    ));
  }
  
  @singleton
  FirebaseFirestore get firestore => FirebaseFirestore.instance;
  
  @singleton
  FirebaseAuth get firebaseAuth => FirebaseAuth.instance;
}
```

### การใช้ DI

```dart
// ใน main.dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await configureDependencies();
  runApp(const MyApp());
}

// ในตัวอย่าง widget
class ProductScreen extends StatelessWidget {
  // Inject usecase โดยตรง
  final GetProductsUseCase _getProducts = getIt<GetProductsUseCase>();
  
  @override
  Widget build(BuildContext context) { /* ... */ }
}

// ใน Riverpod
@riverpod
ProductRepository productRepository(ProductRepositoryRef ref) {
  return getIt<ProductRepository>();
}
```

---

## ขั้นตอนที่ 473: Event-driven Architecture

### Domain Events

```dart
// lib/domain/events/domain_event.dart
abstract class DomainEvent {
  final String eventId;
  final DateTime occurredAt;
  final String entityId;
  
  DomainEvent({
    required this.entityId,
    String? eventId,
    DateTime? occurredAt,
  })  : eventId = eventId ?? const Uuid().v4(),
        occurredAt = occurredAt ?? DateTime.now();
  
  String get eventType;
  Map<String, dynamic> toJson();
}

// Order Events
class OrderCreatedEvent extends DomainEvent {
  final String userId;
  final double totalAmount;
  final List<String> productIds;
  
  OrderCreatedEvent({
    required String orderId,
    required this.userId,
    required this.totalAmount,
    required this.productIds,
  }) : super(entityId: orderId);
  
  @override
  String get eventType => 'order.created';
  
  @override
  Map<String, dynamic> toJson() => {
    'eventType': eventType,
    'orderId': entityId,
    'userId': userId,
    'totalAmount': totalAmount,
    'productIds': productIds,
    'occurredAt': occurredAt.toIso8601String(),
  };
}

class OrderShippedEvent extends DomainEvent {
  final String trackingNumber;
  final String carrier;
  
  OrderShippedEvent({
    required String orderId,
    required this.trackingNumber,
    required this.carrier,
  }) : super(entityId: orderId);
  
  @override
  String get eventType => 'order.shipped';
  
  @override
  Map<String, dynamic> toJson() => {
    'eventType': eventType,
    'orderId': entityId,
    'trackingNumber': trackingNumber,
    'carrier': carrier,
  };
}
```

### Domain Event Bus

```dart
// lib/domain/events/event_bus.dart
typedef EventHandler<T extends DomainEvent> = Future<void> Function(T event);

class DomainEventBus {
  static final DomainEventBus _instance = DomainEventBus._internal();
  factory DomainEventBus() => _instance;
  DomainEventBus._internal();
  
  final Map<Type, List<Function>> _handlers = {};
  
  void subscribe<T extends DomainEvent>(EventHandler<T> handler) {
    _handlers.putIfAbsent(T, () => []).add(handler);
  }
  
  void unsubscribe<T extends DomainEvent>(EventHandler<T> handler) {
    _handlers[T]?.remove(handler);
  }
  
  Future<void> publish(DomainEvent event) async {
    final handlers = _handlers[event.runtimeType] ?? [];
    
    for (final handler in handlers) {
      try {
        await (handler as Function)(event);
      } catch (e) {
        print('Event handler error for ${event.eventType}: $e');
      }
    }
  }
  
  Future<void> publishAll(List<DomainEvent> events) async {
    for (final event in events) {
      await publish(event);
    }
  }
}

// Domain Aggregate สร้าง events
abstract class AggregateRoot {
  final List<DomainEvent> _domainEvents = [];
  
  void addDomainEvent(DomainEvent event) {
    _domainEvents.add(event);
  }
  
  List<DomainEvent> get domainEvents => List.unmodifiable(_domainEvents);
  
  void clearDomainEvents() => _domainEvents.clear();
}

class Order extends AggregateRoot {
  String id;
  String userId;
  List<OrderItem> items;
  OrderStatus status;
  double totalAmount;
  
  Order({
    required this.id,
    required this.userId,
    required this.items,
    this.status = OrderStatus.pending,
  }) : totalAmount = items.fold(0, (sum, item) => sum + item.price * item.quantity);
  
  static Order create({
    required String userId,
    required List<OrderItem> items,
  }) {
    final order = Order(
      id: const Uuid().v4(),
      userId: userId,
      items: items,
    );
    
    order.addDomainEvent(OrderCreatedEvent(
      orderId: order.id,
      userId: userId,
      totalAmount: order.totalAmount,
      productIds: items.map((i) => i.productId).toList(),
    ));
    
    return order;
  }
  
  void ship(String trackingNumber, String carrier) {
    if (status != OrderStatus.processing) {
      throw StateError('Order must be processing to ship');
    }
    
    status = OrderStatus.shipped;
    
    addDomainEvent(OrderShippedEvent(
      orderId: id,
      trackingNumber: trackingNumber,
      carrier: carrier,
    ));
  }
}

// Event Handlers
class SendOrderConfirmationEmailHandler {
  final EmailService _emailService;
  
  SendOrderConfirmationEmailHandler(this._emailService);
  
  Future<void> handle(OrderCreatedEvent event) async {
    await _emailService.sendOrderConfirmation(
      userId: event.userId,
      orderId: event.entityId,
      totalAmount: event.totalAmount,
    );
  }
}

class UpdateInventoryHandler {
  final InventoryService _inventoryService;
  
  UpdateInventoryHandler(this._inventoryService);
  
  Future<void> handle(OrderCreatedEvent event) async {
    for (final productId in event.productIds) {
      await _inventoryService.decrementStock(productId, 1);
    }
  }
}
```

---

## ขั้นตอนที่ 474: CQRS Pattern

### Command Query Responsibility Segregation

```
CQRS:
Write Side (Commands):         Read Side (Queries):
User ──> Command ──>           User ──> Query ──>
         Command Handler                Query Handler
              │                              │
         Write Model                    Read Model
              │                              │
         Event Store ──────────────> Read Store (Projections)
```

### Commands

```dart
// lib/domain/commands/order_commands.dart
abstract class Command {}

class CreateOrderCommand implements Command {
  final String userId;
  final List<OrderItemDto> items;
  final String shippingAddress;
  
  const CreateOrderCommand({
    required this.userId,
    required this.items,
    required this.shippingAddress,
  });
}

class CancelOrderCommand implements Command {
  final String orderId;
  final String reason;
  
  const CancelOrderCommand({
    required this.orderId,
    required this.reason,
  });
}

// Command Handlers
abstract class CommandHandler<T extends Command> {
  Future<void> handle(T command);
}

class CreateOrderCommandHandler implements CommandHandler<CreateOrderCommand> {
  final OrderRepository _orderRepository;
  final ProductRepository _productRepository;
  final DomainEventBus _eventBus;
  
  CreateOrderCommandHandler(
    this._orderRepository,
    this._productRepository,
    this._eventBus,
  );
  
  @override
  Future<void> handle(CreateOrderCommand command) async {
    // Validate products exist and have stock
    final products = await Future.wait(
      command.items.map((item) => _productRepository.findById(item.productId)),
    );
    
    for (int i = 0; i < products.length; i++) {
      final product = products[i];
      if (product == null) {
        throw ProductNotFoundException(command.items[i].productId);
      }
      if (product.stock < command.items[i].quantity) {
        throw InsufficientStockException(product.id, product.stock);
      }
    }
    
    // Create order aggregate
    final order = Order.create(
      userId: command.userId,
      items: command.items.map((item) {
        final product = products.firstWhere((p) => p!.id == item.productId)!;
        return OrderItem(
          productId: item.productId,
          quantity: item.quantity,
          price: product.price,
        );
      }).toList(),
    );
    
    // Save
    await _orderRepository.save(order);
    
    // Publish events
    await _eventBus.publishAll(order.domainEvents);
    order.clearDomainEvents();
  }
}
```

### Queries

```dart
// lib/domain/queries/order_queries.dart
abstract class Query<T> {
  Type get resultType => T;
}

class GetOrderByIdQuery implements Query<OrderDetailDto> {
  final String orderId;
  const GetOrderByIdQuery(this.orderId);
}

class GetOrdersByUserQuery implements Query<List<OrderSummaryDto>> {
  final String userId;
  final int page;
  final int limit;
  
  const GetOrdersByUserQuery({
    required this.userId,
    this.page = 1,
    this.limit = 20,
  });
}

// Query Handlers
abstract class QueryHandler<Q extends Query<T>, T> {
  Future<T> handle(Q query);
}

class GetOrderByIdQueryHandler
    implements QueryHandler<GetOrderByIdQuery, OrderDetailDto> {
  final ReadOrderRepository _readRepository;
  
  GetOrderByIdQueryHandler(this._readRepository);
  
  @override
  Future<OrderDetailDto> handle(GetOrderByIdQuery query) async {
    final order = await _readRepository.getOrderDetail(query.orderId);
    if (order == null) throw OrderNotFoundException(query.orderId);
    return order;
  }
}

// Command/Query Bus
class CommandBus {
  final Map<Type, CommandHandler> _handlers = {};
  
  void register<T extends Command>(CommandHandler<T> handler) {
    _handlers[T] = handler;
  }
  
  Future<void> dispatch<T extends Command>(T command) async {
    final handler = _handlers[T] as CommandHandler<T>?;
    if (handler == null) throw UnregisteredCommandException(T);
    await handler.handle(command);
  }
}

class QueryBus {
  final Map<Type, QueryHandler> _handlers = {};
  
  void register<Q extends Query<T>, T>(QueryHandler<Q, T> handler) {
    _handlers[Q] = handler;
  }
  
  Future<T> dispatch<Q extends Query<T>, T>(Q query) async {
    final handler = _handlers[Q] as QueryHandler<Q, T>?;
    if (handler == null) throw UnregisteredQueryException(Q);
    return handler.handle(query);
  }
}
```

---

## ขั้นตอนที่ 475: Microservices Communication

### API Gateway Pattern

```dart
// lib/services/api_gateway.dart
class ApiGateway {
  final Map<String, ServiceProxy> _services;
  final LoadBalancer _loadBalancer;
  final CircuitBreaker _circuitBreaker;
  
  ApiGateway({
    required List<ServiceProxy> services,
    required LoadBalancer loadBalancer,
  })  : _services = {for (final s in services) s.name: s},
        _loadBalancer = loadBalancer,
        _circuitBreaker = CircuitBreaker();
  
  Future<Response> route(Request request) async {
    // Route based on path
    final service = _resolveService(request.path);
    if (service == null) {
      return Response.notFound();
    }
    
    // Check circuit breaker
    if (_circuitBreaker.isOpen(service.name)) {
      return Response.serviceUnavailable('Service ${service.name} is unavailable');
    }
    
    try {
      final response = await _loadBalancer.call(service, request);
      _circuitBreaker.recordSuccess(service.name);
      return response;
    } catch (e) {
      _circuitBreaker.recordFailure(service.name);
      rethrow;
    }
  }
  
  ServiceProxy? _resolveService(String path) {
    if (path.startsWith('/api/users')) return _services['user-service'];
    if (path.startsWith('/api/orders')) return _services['order-service'];
    if (path.startsWith('/api/products')) return _services['product-service'];
    return null;
  }
}

// Circuit Breaker
class CircuitBreaker {
  final Map<String, CircuitBreakerState> _states = {};
  static const int _failureThreshold = 5;
  static const Duration _openDuration = Duration(seconds: 30);
  
  bool isOpen(String service) {
    final state = _states[service];
    if (state == null) return false;
    
    if (state.status == CircuitStatus.open) {
      if (DateTime.now().isAfter(state.openedAt!.add(_openDuration))) {
        // Transition to half-open
        _states[service] = state.copyWith(status: CircuitStatus.halfOpen);
        return false;
      }
      return true;
    }
    
    return false;
  }
  
  void recordSuccess(String service) {
    _states[service] = CircuitBreakerState.closed();
  }
  
  void recordFailure(String service) {
    final state = _states[service] ?? CircuitBreakerState.closed();
    final newFailureCount = state.failureCount + 1;
    
    if (newFailureCount >= _failureThreshold) {
      _states[service] = CircuitBreakerState.open();
    } else {
      _states[service] = state.copyWith(failureCount: newFailureCount);
    }
  }
}
```

---

## ขั้นตอนที่ 476: Enterprise SSO/SAML Integration

### OAuth 2.0 / OpenID Connect

```dart
// lib/security/sso_service.dart
import 'package:flutter_appauth/flutter_appauth.dart';

class SSOService {
  static const FlutterAppAuth _appAuth = FlutterAppAuth();
  
  // OIDC Configuration
  static const _clientId = 'your-client-id';
  static const _redirectUrl = 'com.example.app://oauth/callback';
  static const _issuer = 'https://your-identity-provider.com';
  static const _scopes = ['openid', 'profile', 'email', 'offline_access'];
  
  Future<AuthState?> signIn() async {
    try {
      final result = await _appAuth.authorizeAndExchangeCode(
        AuthorizationTokenRequest(
          _clientId,
          _redirectUrl,
          issuer: _issuer,
          scopes: _scopes,
          promptValues: ['login'],
        ),
      );
      
      if (result == null) return null;
      
      return AuthState(
        accessToken: result.accessToken!,
        refreshToken: result.refreshToken,
        idToken: result.idToken,
        expiresAt: result.accessTokenExpirationDateTime,
      );
    } catch (e) {
      print('SSO Sign-in error: $e');
      return null;
    }
  }
  
  Future<AuthState?> refreshToken(String refreshToken) async {
    try {
      final result = await _appAuth.token(
        TokenRequest(
          _clientId,
          _redirectUrl,
          issuer: _issuer,
          refreshToken: refreshToken,
          scopes: _scopes,
        ),
      );
      
      if (result == null) return null;
      
      return AuthState(
        accessToken: result.accessToken!,
        refreshToken: result.refreshToken ?? refreshToken,
        idToken: result.idToken,
        expiresAt: result.accessTokenExpirationDateTime,
      );
    } catch (e) {
      print('Token refresh error: $e');
      return null;
    }
  }
  
  Future<void> signOut(String idToken) async {
    await _appAuth.endSession(
      EndSessionRequest(
        idTokenHint: idToken,
        issuer: _issuer,
        postLogoutRedirectUrl: _redirectUrl,
      ),
    );
  }
  
  Map<String, dynamic>? decodeIdToken(String idToken) {
    try {
      final parts = idToken.split('.');
      if (parts.length != 3) return null;
      
      final payload = parts[1];
      final normalized = base64.normalize(payload);
      final decoded = utf8.decode(base64.decode(normalized));
      return jsonDecode(decoded) as Map<String, dynamic>;
    } catch (_) {
      return null;
    }
  }
}

// Active Directory / Azure AD Integration
class AzureADService {
  static const _tenantId = 'your-tenant-id';
  static const _clientId = 'your-client-id';
  static const _authority = 'https://login.microsoftonline.com/$_tenantId';
  
  late PublicClientApplication _pca;
  
  Future<void> initialize() async {
    _pca = await PublicClientApplication.createPublicClientApplication(
      _clientId,
      authority: _authority,
    );
  }
  
  Future<IAuthenticationResult?> signIn() async {
    try {
      return await _pca.acquireToken(
        scopes: ['User.Read'],
        prompt: Prompt.selectAccount,
      );
    } catch (e) {
      print('Azure AD sign-in error: $e');
      return null;
    }
  }
}
```

---

## ขั้นตอนที่ 477: Audit Logging

### Comprehensive Audit System

```dart
// lib/audit/audit_log.dart
enum AuditAction {
  create,
  read,
  update,
  delete,
  login,
  logout,
  export,
  import,
  approve,
  reject,
}

class AuditEntry {
  final String id;
  final String userId;
  final String? userEmail;
  final AuditAction action;
  final String entityType;
  final String? entityId;
  final Map<String, dynamic>? before;
  final Map<String, dynamic>? after;
  final Map<String, dynamic>? metadata;
  final DateTime timestamp;
  final String? ipAddress;
  final String? userAgent;
  final bool isSuccess;
  final String? errorMessage;
  
  const AuditEntry({
    required this.id,
    required this.userId,
    this.userEmail,
    required this.action,
    required this.entityType,
    this.entityId,
    this.before,
    this.after,
    this.metadata,
    required this.timestamp,
    this.ipAddress,
    this.userAgent,
    this.isSuccess = true,
    this.errorMessage,
  });
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'userId': userId,
    'userEmail': userEmail,
    'action': action.name,
    'entityType': entityType,
    'entityId': entityId,
    'before': before,
    'after': after,
    'metadata': metadata,
    'timestamp': timestamp.toIso8601String(),
    'ipAddress': ipAddress,
    'userAgent': userAgent,
    'isSuccess': isSuccess,
    'errorMessage': errorMessage,
  };
}

abstract class AuditLogger {
  Future<void> log(AuditEntry entry);
  Future<List<AuditEntry>> query(AuditQuery query);
}

class FirestoreAuditLogger implements AuditLogger {
  final FirebaseFirestore _firestore;
  
  FirestoreAuditLogger(this._firestore);
  
  @override
  Future<void> log(AuditEntry entry) async {
    await _firestore
        .collection('audit_logs')
        .doc(entry.id)
        .set(entry.toJson());
  }
  
  @override
  Future<List<AuditEntry>> query(AuditQuery query) async {
    var q = _firestore.collection('audit_logs') as Query;
    
    if (query.userId != null) {
      q = q.where('userId', isEqualTo: query.userId);
    }
    
    if (query.entityType != null) {
      q = q.where('entityType', isEqualTo: query.entityType);
    }
    
    if (query.action != null) {
      q = q.where('action', isEqualTo: query.action!.name);
    }
    
    if (query.from != null) {
      q = q.where('timestamp', isGreaterThanOrEqualTo: query.from!.toIso8601String());
    }
    
    if (query.to != null) {
      q = q.where('timestamp', isLessThanOrEqualTo: query.to!.toIso8601String());
    }
    
    q = q.orderBy('timestamp', descending: true);
    q = q.limit(query.limit ?? 100);
    
    final snapshot = await q.get();
    return snapshot.docs.map((doc) => _fromFirestore(doc)).toList();
  }
  
  AuditEntry _fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return AuditEntry(
      id: doc.id,
      userId: data['userId'] as String,
      userEmail: data['userEmail'] as String?,
      action: AuditAction.values.byName(data['action'] as String),
      entityType: data['entityType'] as String,
      entityId: data['entityId'] as String?,
      timestamp: DateTime.parse(data['timestamp'] as String),
      isSuccess: data['isSuccess'] as bool? ?? true,
    );
  }
}

// Audit Decorator สำหรับ Repository
class AuditableOrderRepository implements OrderRepository {
  final OrderRepository _inner;
  final AuditLogger _auditLogger;
  final AuthService _authService;
  
  AuditableOrderRepository(this._inner, this._auditLogger, this._authService);
  
  @override
  Future<void> save(Order order) async {
    final user = _authService.currentUser;
    
    await _inner.save(order);
    
    await _auditLogger.log(AuditEntry(
      id: const Uuid().v4(),
      userId: user?.id ?? 'system',
      userEmail: user?.email,
      action: AuditAction.create,
      entityType: 'Order',
      entityId: order.id,
      after: order.toJson(),
      timestamp: DateTime.now(),
    ));
  }
  
  @override
  Future<Order?> findById(String id) async {
    final user = _authService.currentUser;
    final order = await _inner.findById(id);
    
    await _auditLogger.log(AuditEntry(
      id: const Uuid().v4(),
      userId: user?.id ?? 'system',
      action: AuditAction.read,
      entityType: 'Order',
      entityId: id,
      timestamp: DateTime.now(),
    ));
    
    return order;
  }
}
```

---

## ขั้นตอนที่ 478: Multi-tenancy Support

### Tenant Context

```dart
// lib/multi_tenancy/tenant_context.dart
class Tenant {
  final String id;
  final String name;
  final String subdomain;
  final TenantPlan plan;
  final TenantConfig config;
  
  const Tenant({
    required this.id,
    required this.name,
    required this.subdomain,
    required this.plan,
    required this.config,
  });
}

enum TenantPlan { free, starter, professional, enterprise }

class TenantConfig {
  final String primaryColor;
  final String logoUrl;
  final String? customDomain;
  final Map<String, bool> features;
  final int maxUsers;
  final int storageQuotaGB;
  
  const TenantConfig({
    required this.primaryColor,
    required this.logoUrl,
    this.customDomain,
    required this.features,
    required this.maxUsers,
    required this.storageQuotaGB,
  });
}

class TenantContext {
  static Tenant? _currentTenant;
  
  static Tenant get current {
    if (_currentTenant == null) {
      throw StateError('No tenant context set');
    }
    return _currentTenant!;
  }
  
  static void setTenant(Tenant tenant) {
    _currentTenant = tenant;
  }
  
  static void clearTenant() {
    _currentTenant = null;
  }
  
  static bool get hasTenant => _currentTenant != null;
  
  static bool hasFeature(String featureName) {
    return _currentTenant?.config.features[featureName] ?? false;
  }
}

// Tenant-aware Repository
class TenantAwareFirestoreRepository<T> {
  final FirebaseFirestore _firestore;
  final String _collection;
  final T Function(Map<String, dynamic>) _fromJson;
  final Map<String, dynamic> Function(T) _toJson;
  
  TenantAwareFirestoreRepository({
    required FirebaseFirestore firestore,
    required String collection,
    required T Function(Map<String, dynamic>) fromJson,
    required Map<String, dynamic> Function(T) toJson,
  })  : _firestore = firestore,
        _collection = collection,
        _fromJson = fromJson,
        _toJson = toJson;
  
  // All queries are scoped to current tenant
  CollectionReference get _tenantCollection {
    final tenantId = TenantContext.current.id;
    return _firestore.collection('tenants/$tenantId/$_collection');
  }
  
  Future<void> save(String id, T entity) async {
    await _tenantCollection.doc(id).set({
      ..._toJson(entity),
      '_tenantId': TenantContext.current.id,
      '_updatedAt': FieldValue.serverTimestamp(),
    });
  }
  
  Future<T?> findById(String id) async {
    final doc = await _tenantCollection.doc(id).get();
    if (!doc.exists) return null;
    return _fromJson(doc.data() as Map<String, dynamic>);
  }
  
  Stream<List<T>> watchAll() {
    return _tenantCollection
        .orderBy('_updatedAt', descending: true)
        .snapshots()
        .map((snapshot) {
      return snapshot.docs
          .map((doc) => _fromJson(doc.data() as Map<String, dynamic>))
          .toList();
    });
  }
}
```

### Tenant Resolution

```dart
// lib/multi_tenancy/tenant_resolver.dart
class TenantResolver {
  final TenantRepository _tenantRepository;
  
  TenantResolver(this._tenantRepository);
  
  Future<Tenant?> resolveFromSubdomain(String url) async {
    final uri = Uri.parse(url);
    final subdomain = _extractSubdomain(uri.host);
    
    if (subdomain == null) return null;
    
    return _tenantRepository.findBySubdomain(subdomain);
  }
  
  Future<Tenant?> resolveFromUser(String userId) async {
    final user = await _tenantRepository.getUserTenant(userId);
    return user;
  }
  
  String? _extractSubdomain(String host) {
    final parts = host.split('.');
    if (parts.length < 3) return null;
    return parts.first;
  }
}

// Multi-tenant App Widget
class MultiTenantApp extends ConsumerWidget {
  const MultiTenantApp({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final tenantAsync = ref.watch(currentTenantProvider);
    
    return tenantAsync.when(
      data: (tenant) {
        if (tenant == null) {
          return const TenantSelectionScreen();
        }
        
        TenantContext.setTenant(tenant);
        
        return MaterialApp.router(
          title: tenant.name,
          theme: ThemeData(
            colorScheme: ColorScheme.fromSeed(
              seedColor: Color(
                int.parse(tenant.config.primaryColor.replaceFirst('#', '0xFF')),
              ),
            ),
          ),
          routerConfig: AppRouter.router,
        );
      },
      loading: () => const MaterialApp(
        home: Scaffold(
          body: Center(child: CircularProgressIndicator()),
        ),
      ),
      error: (error, _) => MaterialApp(
        home: Scaffold(
          body: Center(child: Text('Error: $error')),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 479: Enterprise Testing Strategies

### Test Pyramid

```
         /\
        /  \
       / E2E\          (UI Tests / Integration)
      /------\
     /  Integ \        (Service Integration Tests)
    /----------\
   /  Unit Tests\      (Domain Logic, Use Cases)
  /--------------\
```

### Contract Testing

```dart
// test/contracts/order_service_contract_test.dart
// Consumer-Driven Contract Testing
class OrderServiceContractTest {
  // Mock de contrat (provider side)
  late MockOrderApi mockOrderApi;
  late OrderRepository repository;
  
  setUp(() {
    mockOrderApi = MockOrderApi();
    repository = OrderRepositoryImpl(mockOrderApi);
  });
  
  // Test contract - consumer defines expected behavior
  test('should return order when valid ID provided', () async {
    // Given - สร้าง contract
    when(mockOrderApi.getOrder('order-123')).thenAnswer(
      (_) async => OrderApiResponse(
        id: 'order-123',
        status: 'pending',
        totalAmount: 1500.0,
      ),
    );
    
    // When
    final order = await repository.findById('order-123');
    
    // Then - verify contract
    expect(order, isNotNull);
    expect(order!.id, equals('order-123'));
    expect(order.status, equals(OrderStatus.pending));
    expect(order.totalAmount, equals(1500.0));
  });
}
```

---

## ขั้นตอนที่ 480: Workshop - Enterprise App Architecture

### Complete Enterprise App Structure

```
enterprise_app/
├── lib/
│   ├── core/
│   │   ├── di/              # Dependency injection
│   │   ├── errors/          # Error handling
│   │   ├── network/         # HTTP clients
│   │   └── security/        # Security utilities
│   ├── domain/
│   │   ├── entities/        # Domain models
│   │   ├── events/          # Domain events
│   │   ├── repositories/    # Repository interfaces
│   │   └── use_cases/       # Business logic
│   ├── infrastructure/
│   │   ├── repositories/    # Repository implementations
│   │   ├── services/        # External services
│   │   └── persistence/     # Local database
│   ├── application/
│   │   ├── commands/        # CQRS commands
│   │   ├── queries/         # CQRS queries
│   │   └── handlers/        # Command/Query handlers
│   └── presentation/
│       ├── screens/
│       ├── widgets/
│       └── view_models/
```

### Enterprise App Bootstrap

```dart
// lib/main.dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Firebase
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  // Initialize dependencies
  await configureDependencies();
  
  // Initialize background services
  await BackgroundServiceManager.initialize();
  
  // Setup error tracking
  FlutterError.onError = (error) {
    getIt<ErrorTrackingService>().capture(error);
  };
  
  PlatformDispatcher.instance.onError = (error, stack) {
    getIt<ErrorTrackingService>().captureException(error, stack);
    return true;
  };
  
  runApp(
    ProviderScope(
      child: const EnterpriseApp(),
    ),
  );
}

class EnterpriseApp extends ConsumerWidget {
  const EnterpriseApp({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return MaterialApp.router(
      title: 'Enterprise App',
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      themeMode: ref.watch(themeModeProvider),
      routerConfig: AppRouter.router,
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      supportedLocales: AppLocalizations.supportedLocales,
      builder: (context, child) {
        return MultiProvider(
          providers: [
            Provider.value(value: getIt<AuthService>()),
            Provider.value(value: getIt<AuditLogger>()),
          ],
          child: SecurityGate(child: child!),
        );
      },
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **SOLID Principles**: หลักการออกแบบ OOP ที่ดี
- **Dependency Injection**: injectable + get_it
- **Event-driven Architecture**: Domain Events และ Event Bus
- **CQRS Pattern**: แยก Read/Write operations
- **Microservices**: Circuit Breaker และ API Gateway patterns
- **SSO/SAML**: OAuth 2.0, OIDC, Azure AD integration
- **Audit Logging**: บันทึก activity ทุกอย่าง
- **Multi-tenancy**: รองรับหลาย tenant

## แบบฝึกหัด

1. สร้าง domain model ที่ implement SOLID principles ครบ
2. Setup dependency injection ด้วย injectable
3. Implement CQRS สำหรับ Order management
4. สร้าง audit log system ที่บันทึก user actions
5. Implement multi-tenant architecture พื้นฐาน

---

[⬅️ Part 47](part_47.md) | [Part 49 ➡️](part_49.md)

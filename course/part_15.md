# Part 15: Provider Pattern
## ขั้นตอนที่ 141-150

---

## สารบัญ
1. [ติดตั้ง Provider Package](#ขั้นตอนที่-141-ติดตั้ง-provider-package)
2. [ChangeNotifierProvider](#ขั้นตอนที่-142-changenotifierprovider)
3. [Consumer Widget](#ขั้นตอนที่-143-consumer-widget)
4. [Selector Widget](#ขั้นตอนที่-144-selector-widget)
5. [MultiProvider](#ขั้นตอนที่-145-multiprovider)
6. [ProxyProvider](#ขั้นตอนที่-146-proxyprovider)
7. [context.watch และ context.read และ context.select](#ขั้นตอนที่-147-contextwatch-read-select)
8. [Provider กับ Dependency Injection](#ขั้นตอนที่-148-provider-กับ-dependency-injection)
9. [Testing กับ Provider](#ขั้นตอนที่-149-testing-กับ-provider)
10. [Provider Best Practices](#ขั้นตอนที่-150-provider-best-practices)
11. [Workshop: E-Commerce App](#workshop-e-commerce-app)

---

## ขั้นตอนที่ 141: ติดตั้ง Provider Package

ก่อนอื่นต้องเพิ่ม package ใน `pubspec.yaml`:

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.1
```

```bash
flutter pub get
```

### ทำความเข้าใจ Provider

```
Provider package คือ InheritedWidget wrapper ที่ใช้งานง่ายกว่า

Widget Tree:
┌─────────────────────────────────────────┐
│  ChangeNotifierProvider<CartModel>      │  ← Provider ไว้ด้านบน
│  ┌─────────────────────────────────────┐│
│  │         MaterialApp                 ││
│  │  ┌───────────────────────────────┐  ││
│  │  │         HomePage              │  ││
│  │  │  ┌─────────────────────────┐  │  ││
│  │  │  │   ProductList           │  │  ││
│  │  │  │  context.read<CartModel>│  │  ││  ← เข้าถึง Provider
│  │  │  └─────────────────────────┘  │  ││
│  │  └───────────────────────────────┘  ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

---

## ขั้นตอนที่ 142: ChangeNotifierProvider

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// 1. สร้าง Model (ChangeNotifier)
class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }

  void decrement() {
    _count--;
    notifyListeners();
  }

  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// 2. Wrap app ด้วย Provider
void main() {
  runApp(
    // ChangeNotifierProvider สร้าง instance และทำลายเมื่อ Widget ถูกทำลาย
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: const MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Provider Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const CounterScreen(),
    );
  }
}

// 3. ใช้งาน Provider ใน Widget
class CounterScreen extends StatelessWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Provider Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Counter:'),
            // context.watch - rebuild เมื่อ model เปลี่ยน
            Text(
              '${context.watch<CounterModel>().count}',
              style: Theme.of(context).textTheme.displayLarge,
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // context.read - ไม่ rebuild (ใช้ใน onPressed)
                FloatingActionButton(
                  heroTag: 'dec',
                  onPressed: () => context.read<CounterModel>().decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                FloatingActionButton(
                  heroTag: 'inc',
                  onPressed: () => context.read<CounterModel>().increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
            TextButton(
              onPressed: () => context.read<CounterModel>().reset(),
              child: const Text('Reset'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Provider Types ต่างๆ

```dart
// 1. Provider<T> - สำหรับค่าที่ไม่เปลี่ยนแปลง
Provider<ApiService>(
  create: (context) => ApiService(baseUrl: 'https://api.example.com'),
  child: MyApp(),
)

// 2. ChangeNotifierProvider<T> - สำหรับ ChangeNotifier
ChangeNotifierProvider<CartModel>(
  create: (context) => CartModel(),
  child: MyApp(),
)

// 3. FutureProvider<T> - สำหรับ Future
FutureProvider<List<Product>>(
  create: (context) => ProductRepository().fetchAll(),
  initialData: const [],
  child: MyApp(),
)

// 4. StreamProvider<T> - สำหรับ Stream
StreamProvider<List<Message>>(
  create: (context) => chatRepository.messagesStream(),
  initialData: const [],
  child: MyApp(),
)

// 5. ChangeNotifierProvider.value - ใช้ instance ที่มีอยู่แล้ว
ChangeNotifierProvider.value(
  value: existingCartModel,
  child: CartScreen(),
)
```

---

## ขั้นตอนที่ 143: Consumer Widget

`Consumer` rebuild เฉพาะส่วนที่อยู่ใน builder เมื่อ model เปลี่ยนแปลง

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class UserModel extends ChangeNotifier {
  String _name = 'สมชาย';
  int _age = 25;
  String _email = 'somchai@example.com';
  bool _isPremium = false;

  String get name => _name;
  int get age => _age;
  String get email => _email;
  bool get isPremium => _isPremium;

  void updateName(String name) {
    _name = name;
    notifyListeners();
  }

  void togglePremium() {
    _isPremium = !_isPremium;
    notifyListeners();
  }
}

class ConsumerExample extends StatelessWidget {
  const ConsumerExample({super.key});

  @override
  Widget build(BuildContext context) {
    // ❌ Bad: context.watch ที่ระดับ build ทำให้ rebuild ทุก widget ใน tree
    // final user = context.watch<UserModel>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Consumer Example'),
        // ✅ Consumer เฉพาะส่วนที่ต้องการ
        actions: [
          Consumer<UserModel>(
            builder: (context, user, child) {
              return user.isPremium
                  ? const Icon(Icons.star, color: Colors.amber)
                  : const Icon(Icons.star_border);
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // ✅ Consumer เฉพาะ header - ไม่ต้อง rebuild ส่วนอื่น
          Consumer<UserModel>(
            builder: (context, user, child) {
              return UserHeader(user: user);
            },
          ),

          const Divider(),

          // ✅ Consumer กับ child ที่ไม่เปลี่ยนแปลง
          Consumer<UserModel>(
            builder: (context, user, child) {
              return Column(
                children: [
                  Text('${user.name} (${user.age} ปี)'),
                  child!, // child ถูก build ครั้งเดียว ไม่ rebuild
                ],
              );
            },
            // child ที่ไม่ขึ้นกับ model
            child: const Padding(
              padding: EdgeInsets.all(16),
              child: Text('ข้อมูลคงที่ที่ไม่เปลี่ยนแปลง'),
            ),
          ),

          const SizedBox(height: 16),

          // Consumer2 - ฟัง 2 models พร้อมกัน
          // Consumer2<UserModel, CartModel>(
          //   builder: (context, user, cart, child) {
          //     return Text('${user.name} มี ${cart.itemCount} รายการในตะกร้า');
          //   },
          // ),

          Padding(
            padding: const EdgeInsets.all(16),
            child: ElevatedButton(
              onPressed: () {
                context.read<UserModel>().togglePremium();
              },
              child: const Text('Toggle Premium'),
            ),
          ),
        ],
      ),
    );
  }
}

class UserHeader extends StatelessWidget {
  final UserModel user;

  const UserHeader({super.key, required this.user});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          CircleAvatar(
            backgroundColor: user.isPremium ? Colors.amber : Colors.blue,
            child: Text(user.name[0]),
          ),
          const SizedBox(width: 16),
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(
                children: [
                  Text(user.name, style: const TextStyle(fontWeight: FontWeight.bold)),
                  if (user.isPremium)
                    const Padding(
                      padding: EdgeInsets.only(left: 8),
                      child: Icon(Icons.verified, color: Colors.amber, size: 16),
                    ),
                ],
              ),
              Text(user.email, style: const TextStyle(color: Colors.grey)),
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 144: Selector Widget

`Selector` rebuild เฉพาะเมื่อค่าที่ select เปลี่ยนแปลงเท่านั้น - ประหยัด rebuild มากที่สุด

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class ProductListModel extends ChangeNotifier {
  List<String> _products = List.generate(20, (i) => 'Product ${i + 1}');
  int _selectedIndex = -1;
  String _sortOrder = 'asc';
  String _searchQuery = '';

  List<String> get products => _products;
  int get selectedIndex => _selectedIndex;
  String get sortOrder => _sortOrder;
  String get searchQuery => _searchQuery;

  List<String> get filteredProducts {
    var result = _products
        .where((p) => p.toLowerCase().contains(_searchQuery.toLowerCase()))
        .toList();
    if (_sortOrder == 'desc') result = result.reversed.toList();
    return result;
  }

  void select(int index) {
    _selectedIndex = index;
    notifyListeners(); // แจ้งทุก listener
  }

  void toggleSort() {
    _sortOrder = _sortOrder == 'asc' ? 'desc' : 'asc';
    notifyListeners();
  }

  void search(String query) {
    _searchQuery = query;
    notifyListeners();
  }
}

class SelectorExample extends StatelessWidget {
  const SelectorExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Selector'),
        actions: [
          // Selector ฟัง sortOrder เท่านั้น
          Selector<ProductListModel, String>(
            selector: (context, model) => model.sortOrder,
            builder: (context, sortOrder, child) {
              print('Sort icon rebuilt'); // จะเห็นว่า rebuild เฉพาะตอน sort เปลี่ยน
              return IconButton(
                icon: Icon(
                  sortOrder == 'asc' ? Icons.arrow_upward : Icons.arrow_downward,
                ),
                onPressed: () => context.read<ProductListModel>().toggleSort(),
              );
            },
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: TextField(
              onChanged: (q) => context.read<ProductListModel>().search(q),
              decoration: const InputDecoration(
                hintText: 'ค้นหา...',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(),
              ),
            ),
          ),
          Expanded(
            child: Selector<ProductListModel, List<String>>(
              // select เฉพาะ filteredProducts - rebuild เมื่อ list เปลี่ยน
              selector: (context, model) => model.filteredProducts,
              builder: (context, products, child) {
                print('Product list rebuilt');
                return ListView.builder(
                  itemCount: products.length,
                  itemBuilder: (context, index) {
                    return _ProductItem(
                      product: products[index],
                      index: index,
                    );
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class _ProductItem extends StatelessWidget {
  final String product;
  final int index;

  const _ProductItem({required this.product, required this.index});

  @override
  Widget build(BuildContext context) {
    // Selector เฉพาะ selectedIndex สำหรับแต่ละ item
    return Selector<ProductListModel, bool>(
      selector: (context, model) => model.selectedIndex == index,
      builder: (context, isSelected, child) {
        print('Item $index rebuilt'); // rebuild เฉพาะ item ที่ selected เปลี่ยน
        return ListTile(
          title: Text(product),
          selected: isSelected,
          selectedColor: Colors.blue,
          onTap: () => context.read<ProductListModel>().select(index),
          trailing: isSelected ? const Icon(Icons.check) : null,
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 145: MultiProvider

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Models
class AuthModel extends ChangeNotifier {
  bool _isLoggedIn = false;
  String? _userName;

  bool get isLoggedIn => _isLoggedIn;
  String? get userName => _userName;

  Future<void> login(String name, String password) async {
    await Future.delayed(const Duration(seconds: 1));
    if (password == '1234') {
      _isLoggedIn = true;
      _userName = name;
      notifyListeners();
    } else {
      throw Exception('รหัสผ่านไม่ถูกต้อง');
    }
  }

  void logout() {
    _isLoggedIn = false;
    _userName = null;
    notifyListeners();
  }
}

class ThemeModel extends ChangeNotifier {
  bool _isDark = false;
  Color _primaryColor = Colors.blue;

  bool get isDark => _isDark;
  Color get primaryColor => _primaryColor;

  void toggleTheme() {
    _isDark = !_isDark;
    notifyListeners();
  }

  void setColor(Color color) {
    _primaryColor = color;
    notifyListeners();
  }
}

class NotificationModel extends ChangeNotifier {
  int _unreadCount = 5;
  List<String> _notifications = List.generate(5, (i) => 'การแจ้งเตือน ${i + 1}');

  int get unreadCount => _unreadCount;
  List<String> get notifications => List.unmodifiable(_notifications);

  void markAllRead() {
    _unreadCount = 0;
    notifyListeners();
  }
}

// Multi-Provider Setup
void multiProviderMain() {
  runApp(
    MultiProvider(
      providers: [
        // จัดเรียงตามลำดับ dependency
        ChangeNotifierProvider(create: (_) => AuthModel()),
        ChangeNotifierProvider(create: (_) => ThemeModel()),
        ChangeNotifierProvider(create: (_) => NotificationModel()),
        // Provider สำหรับ service
        Provider(create: (_) => ApiService()),
      ],
      child: const MultiProviderApp(),
    ),
  );
}

class ApiService {
  Future<List<String>> fetchProducts() async {
    await Future.delayed(const Duration(seconds: 1));
    return List.generate(10, (i) => 'สินค้า ${i + 1}');
  }
}

class MultiProviderApp extends StatelessWidget {
  const MultiProviderApp({super.key});

  @override
  Widget build(BuildContext context) {
    final isDark = context.select<ThemeModel, bool>((t) => t.isDark);
    final primaryColor = context.select<ThemeModel, Color>((t) => t.primaryColor);

    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: primaryColor,
          brightness: isDark ? Brightness.dark : Brightness.light,
        ),
        useMaterial3: true,
      ),
      home: const MultiProviderHome(),
    );
  }
}

class MultiProviderHome extends StatelessWidget {
  const MultiProviderHome({super.key});

  @override
  Widget build(BuildContext context) {
    final auth = context.watch<AuthModel>();
    final notif = context.watch<NotificationModel>();

    return Scaffold(
      appBar: AppBar(
        title: Text(auth.isLoggedIn ? 'สวัสดี ${auth.userName}' : 'หน้าหลัก'),
        actions: [
          // Theme toggle
          IconButton(
            icon: Icon(context.select<ThemeModel, bool>((t) => t.isDark)
                ? Icons.light_mode
                : Icons.dark_mode),
            onPressed: () => context.read<ThemeModel>().toggleTheme(),
          ),
          // Notification badge
          Badge(
            label: Text('${notif.unreadCount}'),
            isLabelVisible: notif.unreadCount > 0,
            child: IconButton(
              icon: const Icon(Icons.notifications),
              onPressed: notif.markAllRead,
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          if (!auth.isLoggedIn)
            Padding(
              padding: const EdgeInsets.all(16),
              child: ElevatedButton(
                onPressed: () async {
                  try {
                    await context.read<AuthModel>().login('สมชาย', '1234');
                  } catch (e) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('$e')),
                    );
                  }
                },
                child: const Text('เข้าสู่ระบบ'),
              ),
            )
          else
            TextButton(
              onPressed: () => context.read<AuthModel>().logout(),
              child: const Text('ออกจากระบบ'),
            ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 146: ProxyProvider

`ProxyProvider` สร้าง provider ที่ขึ้นกับ provider อื่น

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Service ที่ต้องการ AuthToken
class AuthToken {
  final String token;
  const AuthToken(this.token);
}

class AuthProvider extends ChangeNotifier {
  AuthToken? _token;
  AuthToken? get token => _token;
  bool get isAuthenticated => _token != null;

  void setToken(String token) {
    _token = AuthToken(token);
    notifyListeners();
  }

  void clearToken() {
    _token = null;
    notifyListeners();
  }
}

// UserService ขึ้นกับ AuthToken
class UserService {
  final AuthToken? authToken;

  const UserService({this.authToken});

  Future<Map<String, dynamic>?> getCurrentUser() async {
    if (authToken == null) return null;
    await Future.delayed(const Duration(milliseconds: 500));
    return {
      'id': 1,
      'name': 'สมชาย',
      'email': 'somchai@example.com',
      'token': authToken!.token,
    };
  }
}

// OrderService ขึ้นกับทั้ง AuthToken และ UserService
class OrderService {
  final AuthToken? authToken;
  final UserService userService;

  const OrderService({
    required this.authToken,
    required this.userService,
  });

  Future<List<String>> getOrders() async {
    if (authToken == null) return [];
    final user = await userService.getCurrentUser();
    if (user == null) return [];
    return List.generate(5, (i) => 'คำสั่งซื้อ ${i + 1} ของ ${user["name"]}');
  }
}

void proxyProviderMain() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),

        // ProxyProvider - สร้าง UserService จาก AuthToken
        ProxyProvider<AuthProvider, UserService>(
          update: (context, auth, previous) {
            return UserService(authToken: auth.token);
          },
        ),

        // ProxyProvider2 - ขึ้นกับ 2 providers
        ProxyProvider2<AuthProvider, UserService, OrderService>(
          update: (context, auth, userService, previous) {
            return OrderService(
              authToken: auth.token,
              userService: userService,
            );
          },
        ),
      ],
      child: const ProxyProviderApp(),
    ),
  );
}

class ProxyProviderApp extends StatelessWidget {
  const ProxyProviderApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('ProxyProvider')),
        body: Column(
          children: [
            // Auth section
            Consumer<AuthProvider>(
              builder: (context, auth, _) {
                return Padding(
                  padding: const EdgeInsets.all(16),
                  child: auth.isAuthenticated
                      ? Row(
                          children: [
                            Text('Token: ${auth.token?.token}'),
                            TextButton(
                              onPressed: auth.clearToken,
                              child: const Text('Logout'),
                            ),
                          ],
                        )
                      : ElevatedButton(
                          onPressed: () => auth.setToken('my-secret-token-123'),
                          child: const Text('Login'),
                        ),
                );
              },
            ),

            // User section
            FutureBuilder<Map<String, dynamic>?>(
              future: context.watch<UserService>().getCurrentUser(),
              builder: (context, snapshot) {
                if (snapshot.connectionState == ConnectionState.waiting) {
                  return const CircularProgressIndicator();
                }
                if (snapshot.data == null) {
                  return const Text('ยังไม่ได้เข้าสู่ระบบ');
                }
                return ListTile(
                  title: Text(snapshot.data!['name'] as String),
                  subtitle: Text(snapshot.data!['email'] as String),
                );
              },
            ),

            // Orders section
            FutureBuilder<List<String>>(
              future: context.watch<OrderService>().getOrders(),
              builder: (context, snapshot) {
                if (!snapshot.hasData) return const SizedBox();
                return Column(
                  children: snapshot.data!
                      .map((o) => ListTile(title: Text(o)))
                      .toList(),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 147: context.watch, read, select

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class StockModel extends ChangeNotifier {
  double _price = 100.0;
  int _quantity = 0;
  String _status = 'ปกติ';

  double get price => _price;
  int get quantity => _quantity;
  String get status => _status;
  double get totalValue => _price * _quantity;

  void updatePrice(double price) {
    _price = price;
    _status = price > 100 ? 'ขึ้น' : price < 100 ? 'ลง' : 'ปกติ';
    notifyListeners();
  }

  void buy(int qty) {
    _quantity += qty;
    notifyListeners();
  }

  void sell(int qty) {
    if (_quantity >= qty) {
      _quantity -= qty;
      notifyListeners();
    }
  }
}

class ContextMethodsScreen extends StatelessWidget {
  const ContextMethodsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('context methods')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 1. context.watch<T>() - subscribe ทุก change
            //    Widget นี้จะ rebuild ทุกครั้งที่ StockModel notifyListeners()
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text('context.watch (rebuild ทุกครั้ง):',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    Builder(
                      builder: (context) {
                        final stock = context.watch<StockModel>();
                        return Column(
                          children: [
                            Text('ราคา: ${stock.price.toStringAsFixed(2)} บาท'),
                            Text('จำนวน: ${stock.quantity} หุ้น'),
                            Text('มูลค่ารวม: ${stock.totalValue.toStringAsFixed(2)} บาท'),
                          ],
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // 2. context.select<T, R>() - rebuild เฉพาะเมื่อ selected value เปลี่ยน
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text('context.select (rebuild เฉพาะ price):',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    Builder(
                      builder: (context) {
                        // rebuild เฉพาะเมื่อ price เปลี่ยน
                        final price = context.select<StockModel, double>(
                          (stock) => stock.price,
                        );
                        return Text(
                          'ราคา: ${price.toStringAsFixed(2)} บาท',
                          style: TextStyle(
                            color: price > 100 ? Colors.green : Colors.red,
                            fontWeight: FontWeight.bold,
                          ),
                        );
                      },
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // 3. context.read<T>() - ใช้ใน callbacks (ไม่ rebuild)
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Text('context.read (ใน callbacks):',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                      children: [
                        ElevatedButton(
                          // ✅ ถูก: context.read ใน onPressed
                          onPressed: () => context.read<StockModel>().buy(10),
                          child: const Text('ซื้อ 10 หุ้น'),
                        ),
                        ElevatedButton(
                          onPressed: () => context.read<StockModel>().sell(5),
                          child: const Text('ขาย 5 หุ้น'),
                        ),
                      ],
                    ),
                    Slider(
                      value: context.select<StockModel, double>((s) => s.price),
                      min: 50,
                      max: 200,
                      // ✅ ถูก: context.read ใน onChanged
                      onChanged: (v) => context.read<StockModel>().updatePrice(v),
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
}
```

---

## ขั้นตอนที่ 148: Provider กับ Dependency Injection

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Interfaces (Abstract classes)
abstract class IProductRepository {
  Future<List<Product>> getProducts();
  Future<Product> getProductById(String id);
}

abstract class ICartRepository {
  Future<void> saveCart(List<CartItem> items);
  Future<List<CartItem>> loadCart();
}

// Implementations
class RemoteProductRepository implements IProductRepository {
  @override
  Future<List<Product>> getProducts() async {
    await Future.delayed(const Duration(seconds: 1));
    return List.generate(
      10,
      (i) => Product(
        id: 'remote_$i',
        name: 'Remote Product ${i + 1}',
        price: (i + 1) * 199.0,
      ),
    );
  }

  @override
  Future<Product> getProductById(String id) async {
    final products = await getProducts();
    return products.firstWhere((p) => p.id == id);
  }
}

class MockProductRepository implements IProductRepository {
  @override
  Future<List<Product>> getProducts() async {
    return [
      const Product(id: 'mock_1', name: 'Mock Product 1', price: 100),
      const Product(id: 'mock_2', name: 'Mock Product 2', price: 200),
    ];
  }

  @override
  Future<Product> getProductById(String id) async {
    return const Product(id: 'mock_1', name: 'Mock Product 1', price: 100);
  }
}

class Product {
  final String id;
  final String name;
  final double price;
  const Product({required this.id, required this.name, required this.price});
}

class CartItem {
  final Product product;
  int quantity;
  CartItem({required this.product, this.quantity = 1});
}

// ViewModel/Store ที่ใช้ Repository
class ProductStore extends ChangeNotifier {
  final IProductRepository _repository;

  ProductStore({required IProductRepository repository})
      : _repository = repository;

  List<Product> _products = [];
  bool _isLoading = false;
  String? _error;

  List<Product> get products => _products;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadProducts() async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _products = await _repository.getProducts();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}

// Setup ใน main()
void diMain() {
  const isProduction = bool.fromEnvironment('PRODUCTION', defaultValue: true);

  runApp(
    MultiProvider(
      providers: [
        Provider<IProductRepository>(
          create: (_) => isProduction
              ? RemoteProductRepository()
              : MockProductRepository(),
        ),
        ChangeNotifierProxyProvider<IProductRepository, ProductStore>(
          create: (context) => ProductStore(
            repository: context.read<IProductRepository>(),
          ),
          update: (context, repo, previous) =>
              previous ?? ProductStore(repository: repo),
        ),
      ],
      child: const DIApp(),
    ),
  );
}

class DIApp extends StatelessWidget {
  const DIApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Dependency Injection')),
        body: Consumer<ProductStore>(
          builder: (context, store, _) {
            if (store.products.isEmpty && !store.isLoading) {
              return Center(
                child: ElevatedButton(
                  onPressed: store.loadProducts,
                  child: const Text('โหลดสินค้า'),
                ),
              );
            }
            if (store.isLoading) return const Center(child: CircularProgressIndicator());
            if (store.error != null) return Center(child: Text('Error: ${store.error}'));
            return ListView.builder(
              itemCount: store.products.length,
              itemBuilder: (context, index) {
                final p = store.products[index];
                return ListTile(title: Text(p.name), subtitle: Text('฿${p.price}'));
              },
            );
          },
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 149: Testing กับ Provider

```dart
// test/provider_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:provider/provider.dart';

// Model ที่ต้อง test
class TodoModel extends ChangeNotifier {
  final List<String> _todos = [];
  List<String> get todos => List.unmodifiable(_todos);

  void addTodo(String todo) {
    _todos.add(todo);
    notifyListeners();
  }

  void removeTodo(int index) {
    _todos.removeAt(index);
    notifyListeners();
  }
}

// Widget ที่ใช้ Provider
class TodoList extends StatelessWidget {
  const TodoList({super.key});

  @override
  Widget build(BuildContext context) {
    final todos = context.watch<TodoModel>().todos;
    return Column(
      children: [
        Text('${todos.length} todos', key: const Key('count')),
        ...todos.asMap().entries.map(
          (e) => ListTile(
            key: Key('todo_${e.key}'),
            title: Text(e.value),
          ),
        ),
      ],
    );
  }
}

// Helper สำหรับ test
Widget createTestWidget({required Widget child, TodoModel? model}) {
  return MaterialApp(
    home: ChangeNotifierProvider<TodoModel>.value(
      value: model ?? TodoModel(),
      child: Scaffold(body: child),
    ),
  );
}

void main() {
  group('TodoModel', () {
    test('เริ่มที่ list ว่าง', () {
      final model = TodoModel();
      expect(model.todos, isEmpty);
    });

    test('addTodo เพิ่มรายการ', () {
      final model = TodoModel();
      model.addTodo('Task 1');
      expect(model.todos.length, 1);
      expect(model.todos.first, 'Task 1');
    });

    test('notifyListeners ถูกเรียกเมื่อ add', () {
      final model = TodoModel();
      bool notified = false;
      model.addListener(() => notified = true);
      model.addTodo('Task');
      expect(notified, true);
    });
  });

  group('TodoList Widget', () {
    testWidgets('แสดง count ถูกต้อง', (tester) async {
      final model = TodoModel()
        ..addTodo('Task 1')
        ..addTodo('Task 2');

      await tester.pumpWidget(
        createTestWidget(child: const TodoList(), model: model),
      );

      expect(find.text('2 todos'), findsOneWidget);
    });

    testWidgets('แสดง todos ทั้งหมด', (tester) async {
      final model = TodoModel()
        ..addTodo('งานที่ 1')
        ..addTodo('งานที่ 2');

      await tester.pumpWidget(
        createTestWidget(child: const TodoList(), model: model),
      );

      expect(find.text('งานที่ 1'), findsOneWidget);
      expect(find.text('งานที่ 2'), findsOneWidget);
    });
  });
}
```

---

## ขั้นตอนที่ 150: Provider Best Practices

```dart
// ✅ Best Practices สำหรับ Provider

// 1. ใช้ context.read ใน callbacks เท่านั้น
ElevatedButton(
  onPressed: () => context.read<CartModel>().addItem(product), // ✅
  child: const Text('เพิ่ม'),
)

// ❌ ห้ามใช้ context.watch ใน callbacks
// onPressed: () => context.watch<CartModel>().addItem(product), // ❌

// 2. ใช้ Selector เพื่อลด rebuild
Selector<CartModel, int>(
  selector: (_, cart) => cart.itemCount, // เฉพาะ itemCount
  builder: (_, count, __) => Badge(label: Text('$count')),
)

// 3. อย่า create provider ใน build method
// ❌ ผิด
Widget build(BuildContext context) {
  return ChangeNotifierProvider(
    create: (_) => CartModel(), // สร้างใหม่ทุก build!
    child: CartScreen(),
  );
}

// ✅ ถูก - ใช้ .value สำหรับ existing instance
Widget build(BuildContext context) {
  return ChangeNotifierProvider.value(
    value: _cart, // _cart ถูกสร้างใน initState
    child: const CartScreen(),
  );
}

// 4. dispose ChangeNotifier
class _MyState extends State<MyWidget> {
  final _model = MyModel();

  @override
  void dispose() {
    _model.dispose(); // สำคัญ!
    super.dispose();
  }
}

// 5. ใช้ listen: false เมื่อไม่ต้องการ rebuild
final cart = Provider.of<CartModel>(context, listen: false);
// เหมือนกับ context.read<CartModel>()
```

---

## Workshop: E-Commerce App

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

// Models
class Product {
  final String id;
  final String name;
  final double price;
  final String category;
  final String emoji;
  final double rating;
  final int reviewCount;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.category,
    required this.emoji,
    this.rating = 4.5,
    this.reviewCount = 100,
  });
}

class CartItem {
  final Product product;
  int quantity;
  CartItem({required this.product, this.quantity = 1});
  double get total => product.price * quantity;
}

// Providers
class ProductProvider extends ChangeNotifier {
  static const _products = [
    Product(id: '1', name: 'iPhone 15 Pro', price: 49900, category: 'มือถือ', emoji: '📱', rating: 4.8),
    Product(id: '2', name: 'Samsung S24', price: 35900, category: 'มือถือ', emoji: '📱', rating: 4.5),
    Product(id: '3', name: 'MacBook Air', price: 42900, category: 'คอมพิวเตอร์', emoji: '💻', rating: 4.9),
    Product(id: '4', name: 'iPad Pro', price: 32900, category: 'แท็บเล็ต', emoji: '📱', rating: 4.7),
    Product(id: '5', name: 'AirPods Pro', price: 9990, category: 'หูฟัง', emoji: '🎧', rating: 4.6),
    Product(id: '6', name: 'Sony WH-1000XM5', price: 12990, category: 'หูฟัง', emoji: '🎧', rating: 4.8),
  ];

  String _selectedCategory = 'ทั้งหมด';
  String _searchQuery = '';
  String _sortBy = 'name';

  String get selectedCategory => _selectedCategory;

  List<String> get categories {
    return ['ทั้งหมด', ..._products.map((p) => p.category).toSet().toList()];
  }

  List<Product> get filteredProducts {
    var result = _products.where((p) {
      final matchesCat = _selectedCategory == 'ทั้งหมด' || p.category == _selectedCategory;
      final matchesSearch = _searchQuery.isEmpty ||
          p.name.toLowerCase().contains(_searchQuery.toLowerCase());
      return matchesCat && matchesSearch;
    }).toList();

    switch (_sortBy) {
      case 'price_asc':
        result.sort((a, b) => a.price.compareTo(b.price));
        break;
      case 'price_desc':
        result.sort((a, b) => b.price.compareTo(a.price));
        break;
      case 'rating':
        result.sort((a, b) => b.rating.compareTo(a.rating));
        break;
      default:
        result.sort((a, b) => a.name.compareTo(b.name));
    }

    return result;
  }

  void setCategory(String category) {
    _selectedCategory = category;
    notifyListeners();
  }

  void setSearch(String query) {
    _searchQuery = query;
    notifyListeners();
  }

  void setSort(String sort) {
    _sortBy = sort;
    notifyListeners();
  }
}

class CartProvider extends ChangeNotifier {
  final Map<String, CartItem> _items = {};

  Map<String, CartItem> get items => Map.unmodifiable(_items);
  int get itemCount => _items.values.fold(0, (s, i) => s + i.quantity);
  double get subtotal => _items.values.fold(0, (s, i) => s + i.total);
  double get total => subtotal;

  void add(Product product) {
    if (_items.containsKey(product.id)) {
      _items[product.id]!.quantity++;
    } else {
      _items[product.id] = CartItem(product: product);
    }
    notifyListeners();
  }

  void remove(String productId) {
    _items.remove(productId);
    notifyListeners();
  }

  void updateQty(String productId, int qty) {
    if (qty <= 0) {
      remove(productId);
    } else {
      _items[productId]?.quantity = qty;
      notifyListeners();
    }
  }

  bool isInCart(String productId) => _items.containsKey(productId);
  int getQty(String productId) => _items[productId]?.quantity ?? 0;
}

class WishlistProvider extends ChangeNotifier {
  final Set<String> _wishlist = {};

  bool isWishlisted(String productId) => _wishlist.contains(productId);

  void toggle(String productId) {
    if (_wishlist.contains(productId)) {
      _wishlist.remove(productId);
    } else {
      _wishlist.add(productId);
    }
    notifyListeners();
  }
}

// Main App
void ecommerceMain() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => ProductProvider()),
        ChangeNotifierProvider(create: (_) => CartProvider()),
        ChangeNotifierProvider(create: (_) => WishlistProvider()),
      ],
      child: const ECommerceApp(),
    ),
  );
}

class ECommerceApp extends StatelessWidget {
  const ECommerceApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'E-Commerce',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.purple),
        useMaterial3: true,
      ),
      home: const ECommerceHome(),
    );
  }
}

class ECommerceHome extends StatelessWidget {
  const ECommerceHome({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('🛍️ E-Commerce'),
        actions: [
          // Wishlist button
          Selector<WishlistProvider, int>(
            selector: (_, w) => 0, // จำนวน wishlist items
            builder: (_, __, ___) => IconButton(
              icon: const Icon(Icons.favorite_border),
              onPressed: () {},
            ),
          ),
          // Cart button with badge
          Selector<CartProvider, int>(
            selector: (_, cart) => cart.itemCount,
            builder: (_, count, __) => Badge(
              label: Text('$count'),
              isLabelVisible: count > 0,
              child: IconButton(
                icon: const Icon(Icons.shopping_cart),
                onPressed: () => _showCart(context),
              ),
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          // Search
          Padding(
            padding: const EdgeInsets.all(8),
            child: TextField(
              onChanged: (q) => context.read<ProductProvider>().setSearch(q),
              decoration: const InputDecoration(
                hintText: 'ค้นหาสินค้า...',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(12)),
                ),
                isDense: true,
              ),
            ),
          ),

          // Categories
          Selector<ProductProvider, String>(
            selector: (_, p) => p.selectedCategory,
            builder: (context, selected, _) {
              final categories = context.read<ProductProvider>().categories;
              return SizedBox(
                height: 40,
                child: ListView.builder(
                  scrollDirection: Axis.horizontal,
                  padding: const EdgeInsets.symmetric(horizontal: 8),
                  itemCount: categories.length,
                  itemBuilder: (context, index) {
                    final cat = categories[index];
                    return Padding(
                      padding: const EdgeInsets.only(right: 8),
                      child: FilterChip(
                        label: Text(cat),
                        selected: cat == selected,
                        onSelected: (_) =>
                            context.read<ProductProvider>().setCategory(cat),
                      ),
                    );
                  },
                ),
              );
            },
          ),

          const SizedBox(height: 8),

          // Products
          Expanded(
            child: Selector<ProductProvider, List<Product>>(
              selector: (_, p) => p.filteredProducts,
              builder: (context, products, _) {
                return GridView.builder(
                  padding: const EdgeInsets.all(8),
                  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: 2,
                    crossAxisSpacing: 8,
                    mainAxisSpacing: 8,
                    childAspectRatio: 0.72,
                  ),
                  itemCount: products.length,
                  itemBuilder: (context, index) {
                    return _ProductCard(product: products[index]);
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  void _showCart(BuildContext context) {
    showModalBottomSheet(
      context: context,
      builder: (ctx) => MultiProvider(
        providers: [
          ChangeNotifierProvider.value(value: context.read<CartProvider>()),
        ],
        child: const _CartSheet(),
      ),
    );
  }
}

class _ProductCard extends StatelessWidget {
  final Product product;
  const _ProductCard({required this.product});

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Stack(
              children: [
                Container(
                  decoration: BoxDecoration(
                    color: Colors.grey[100],
                    borderRadius: const BorderRadius.vertical(top: Radius.circular(12)),
                  ),
                  alignment: Alignment.center,
                  child: Text(product.emoji, style: const TextStyle(fontSize: 48)),
                ),
                Positioned(
                  top: 4,
                  right: 4,
                  child: Selector<WishlistProvider, bool>(
                    selector: (_, w) => w.isWishlisted(product.id),
                    builder: (context, isWishlisted, _) {
                      return GestureDetector(
                        onTap: () => context.read<WishlistProvider>().toggle(product.id),
                        child: Icon(
                          isWishlisted ? Icons.favorite : Icons.favorite_border,
                          color: isWishlisted ? Colors.red : Colors.grey,
                        ),
                      );
                    },
                  ),
                ),
              ],
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product.name,
                  style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13),
                  overflow: TextOverflow.ellipsis,
                ),
                Row(
                  children: [
                    const Icon(Icons.star, color: Colors.amber, size: 14),
                    Text(' ${product.rating}', style: const TextStyle(fontSize: 12)),
                    Text(' (${product.reviewCount})', style: const TextStyle(fontSize: 11, color: Colors.grey)),
                  ],
                ),
                Text('฿${product.price.toStringAsFixed(0)}',
                    style: const TextStyle(color: Colors.red, fontWeight: FontWeight.bold)),
                Selector<CartProvider, int>(
                  selector: (_, cart) => cart.getQty(product.id),
                  builder: (context, qty, _) {
                    if (qty > 0) {
                      return Row(
                        children: [
                          IconButton(
                            icon: const Icon(Icons.remove, size: 16),
                            padding: EdgeInsets.zero,
                            constraints: const BoxConstraints(),
                            onPressed: () => context.read<CartProvider>().updateQty(product.id, qty - 1),
                          ),
                          Expanded(child: Text('$qty', textAlign: TextAlign.center, style: const TextStyle(fontWeight: FontWeight.bold))),
                          IconButton(
                            icon: const Icon(Icons.add, size: 16),
                            padding: EdgeInsets.zero,
                            constraints: const BoxConstraints(),
                            onPressed: () => context.read<CartProvider>().add(product),
                          ),
                        ],
                      );
                    }
                    return SizedBox(
                      width: double.infinity,
                      child: ElevatedButton(
                        onPressed: () => context.read<CartProvider>().add(product),
                        style: ElevatedButton.styleFrom(padding: const EdgeInsets.symmetric(vertical: 4)),
                        child: const Text('เพิ่ม', style: TextStyle(fontSize: 12)),
                      ),
                    );
                  },
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _CartSheet extends StatelessWidget {
  const _CartSheet();

  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartProvider>();
    return Column(
      children: [
        Padding(
          padding: const EdgeInsets.all(16),
          child: Text('ตะกร้า (${cart.itemCount} รายการ)',
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        ),
        Expanded(
          child: cart.items.isEmpty
              ? const Center(child: Text('ตะกร้าว่างเปล่า'))
              : ListView.builder(
                  itemCount: cart.items.length,
                  itemBuilder: (context, index) {
                    final item = cart.items.values.toList()[index];
                    return ListTile(
                      leading: Text(item.product.emoji, style: const TextStyle(fontSize: 28)),
                      title: Text(item.product.name),
                      subtitle: Text('฿${item.total.toStringAsFixed(0)}'),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(icon: const Icon(Icons.remove), onPressed: () => context.read<CartProvider>().updateQty(item.product.id, item.quantity - 1)),
                          Text('${item.quantity}'),
                          IconButton(icon: const Icon(Icons.add), onPressed: () => context.read<CartProvider>().add(item.product)),
                        ],
                      ),
                    );
                  },
                ),
        ),
        Padding(
          padding: const EdgeInsets.all(16),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text('รวม: ฿${cart.total.toStringAsFixed(0)}', style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              ElevatedButton(onPressed: () {}, child: const Text('ชำระเงิน')),
            ],
          ),
        ),
      ],
    );
  }
}

void main() => ecommerceMain();
```

---

## สรุป (Summary)

| Widget/Method | ใช้เมื่อ |
|---------------|---------|
| ChangeNotifierProvider | จัดการ state ด้วย ChangeNotifier |
| Consumer | rebuild เฉพาะ subtree |
| Selector | rebuild เมื่อ selected value เปลี่ยน |
| MultiProvider | จัดการหลาย providers |
| ProxyProvider | provider ที่ขึ้นกับ provider อื่น |
| context.watch | subscribe ทุก change |
| context.read | อ่านค่าโดยไม่ subscribe |
| context.select | subscribe เฉพาะ value ที่ต้องการ |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง theme provider ที่เปลี่ยน dark/light mode ได้จากทุก screen
2. **ปานกลาง**: สร้าง note-taking app ด้วย Provider (CRUD + search + sort)
3. **ยาก**: สร้าง authentication flow ด้วย MultiProvider (login, user profile, permissions)

---

[← Part 14: State Management](part_14.md) | [Part 16: Riverpod →](part_16.md)

# Part 16: Riverpod
## ขั้นตอนที่ 151-160

---

## สารบัญ
1. [flutter_riverpod Setup](#ขั้นตอนที่-151-flutter_riverpod-setup)
2. [Provider Types พื้นฐาน](#ขั้นตอนที่-152-provider-types-พื้นฐาน)
3. [StateProvider และ StateNotifierProvider](#ขั้นตอนที่-153-stateprovider-และ-statenotifierprovider)
4. [FutureProvider และ StreamProvider](#ขั้นตอนที่-154-futureprovider-และ-streamprovider)
5. [NotifierProvider (Riverpod 2.0)](#ขั้นตอนที่-155-notifierprovider-riverpod-20)
6. [ConsumerWidget และ ConsumerStatefulWidget](#ขั้นตอนที่-156-consumerwidget-และ-consumerstatefulwidget)
7. [ref.watch, ref.read, ref.listen](#ขั้นตอนที่-157-refwatch-refread-reflisten)
8. [family และ autoDispose](#ขั้นตอนที่-158-family-และ-autodispose)
9. [Async Notifiers](#ขั้นตอนที่-159-async-notifiers)
10. [Riverpod Testing](#ขั้นตอนที่-160-riverpod-testing)
11. [Workshop: Task Manager App](#workshop-task-manager-app)

---

## ขั้นตอนที่ 151: flutter_riverpod Setup

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.4  # สำหรับ code generation (optional)

dev_dependencies:
  build_runner: ^2.4.8
  riverpod_generator: ^2.4.0  # สำหรับ code generation (optional)
```

```bash
flutter pub get
```

### Setup ใน main()

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // ProviderScope ต้องครอบ app ทั้งหมด
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Demo',
      home: const HomeScreen(),
    );
  }
}
```

### ข้อดีของ Riverpod เทียบกับ Provider

```
Riverpod vs Provider:
┌────────────────────┬─────────────┬──────────────┐
│ Feature            │ Provider    │ Riverpod     │
├────────────────────┼─────────────┼──────────────┤
│ Compile-time safe  │ ❌          │ ✅           │
│ No BuildContext    │ ❌          │ ✅           │
│ Multiple providers │ ❌          │ ✅           │
│ Testing            │ 🟡          │ ✅           │
│ Auto dispose       │ ❌          │ ✅           │
│ Family             │ 🟡          │ ✅           │
│ Async support      │ 🟡          │ ✅           │
└────────────────────┴─────────────┴──────────────┘
```

---

## ขั้นตอนที่ 152: Provider Types พื้นฐาน

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 1. Provider<T> - ค่าที่ไม่เปลี่ยนแปลง (immutable)
final greetingProvider = Provider<String>((ref) {
  return 'สวัสดี Riverpod!';
});

// Provider ที่ขึ้นกับ provider อื่น
final nameProvider = Provider<String>((ref) {
  return 'สมชาย';
});

final fullGreetingProvider = Provider<String>((ref) {
  final name = ref.watch(nameProvider); // watch provider อื่น
  return 'สวัสดี $name!';
});

// 2. Provider สำหรับ services
class ApiService {
  final String baseUrl;
  ApiService(this.baseUrl);
  
  Future<List<String>> getItems() async {
    await Future.delayed(const Duration(seconds: 1));
    return ['Item 1', 'Item 2', 'Item 3'];
  }
}

final apiServiceProvider = Provider<ApiService>((ref) {
  return ApiService('https://api.example.com');
});

// การใช้งานใน Widget
class ProviderBasicsScreen extends ConsumerWidget {
  const ProviderBasicsScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final greeting = ref.watch(greetingProvider);
    final fullGreeting = ref.watch(fullGreetingProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Provider Basics')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(greeting, style: const TextStyle(fontSize: 24)),
            Text(fullGreeting, style: const TextStyle(fontSize: 20)),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 153: StateProvider และ StateNotifierProvider

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// StateProvider - สำหรับ state ง่ายๆ (int, bool, String, etc.)
final counterProvider = StateProvider<int>((ref) => 0);
final isDarkProvider = StateProvider<bool>((ref) => false);
final selectedTabProvider = StateProvider<int>((ref) => 0);

// StateNotifierProvider - สำหรับ state ที่ซับซ้อน
// 1. State class (immutable)
class TodoState {
  final List<Todo> todos;
  final bool isLoading;
  final String? error;

  const TodoState({
    this.todos = const [],
    this.isLoading = false,
    this.error,
  });

  TodoState copyWith({
    List<Todo>? todos,
    bool? isLoading,
    String? error,
  }) {
    return TodoState(
      todos: todos ?? this.todos,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
}

class Todo {
  final String id;
  final String title;
  final bool isDone;

  const Todo({
    required this.id,
    required this.title,
    this.isDone = false,
  });

  Todo copyWith({String? title, bool? isDone}) {
    return Todo(
      id: id,
      title: title ?? this.title,
      isDone: isDone ?? this.isDone,
    );
  }
}

// 2. StateNotifier
class TodoNotifier extends StateNotifier<TodoState> {
  TodoNotifier() : super(const TodoState()) {
    _loadTodos();
  }

  Future<void> _loadTodos() async {
    state = state.copyWith(isLoading: true);
    await Future.delayed(const Duration(milliseconds: 500));
    state = state.copyWith(
      todos: [
        Todo(id: '1', title: 'เรียน Flutter'),
        Todo(id: '2', title: 'ทำ Project', isDone: true),
        Todo(id: '3', title: 'เขียน Tests'),
      ],
      isLoading: false,
    );
  }

  void addTodo(String title) {
    state = state.copyWith(
      todos: [
        ...state.todos,
        Todo(
          id: DateTime.now().millisecondsSinceEpoch.toString(),
          title: title,
        ),
      ],
    );
  }

  void toggleTodo(String id) {
    state = state.copyWith(
      todos: state.todos.map((todo) {
        if (todo.id == id) {
          return todo.copyWith(isDone: !todo.isDone);
        }
        return todo;
      }).toList(),
    );
  }

  void deleteTodo(String id) {
    state = state.copyWith(
      todos: state.todos.where((t) => t.id != id).toList(),
    );
  }
}

// 3. Provider
final todoProvider = StateNotifierProvider<TodoNotifier, TodoState>((ref) {
  return TodoNotifier();
});

// Derived providers
final completedTodosProvider = Provider<List<Todo>>((ref) {
  return ref.watch(todoProvider).todos.where((t) => t.isDone).toList();
});

final pendingTodosProvider = Provider<List<Todo>>((ref) {
  return ref.watch(todoProvider).todos.where((t) => !t.isDone).toList();
});

// Screen
class TodoScreen extends ConsumerWidget {
  const TodoScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todoState = ref.watch(todoProvider);
    final completedCount = ref.watch(completedTodosProvider).length;
    final pendingCount = ref.watch(pendingTodosProvider).length;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo App (Riverpod)'),
        actions: [
          Text('✓$completedCount  ○$pendingCount',
              style: const TextStyle(fontSize: 14)),
          const SizedBox(width: 8),
        ],
      ),
      body: todoState.isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: todoState.todos.length,
              itemBuilder: (context, index) {
                final todo = todoState.todos[index];
                return Dismissible(
                  key: Key(todo.id),
                  onDismissed: (_) => ref.read(todoProvider.notifier).deleteTodo(todo.id),
                  child: CheckboxListTile(
                    title: Text(
                      todo.title,
                      style: TextStyle(
                        decoration: todo.isDone ? TextDecoration.lineThrough : null,
                        color: todo.isDone ? Colors.grey : null,
                      ),
                    ),
                    value: todo.isDone,
                    onChanged: (_) =>
                        ref.read(todoProvider.notifier).toggleTodo(todo.id),
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context, WidgetRef ref) {
    final controller = TextEditingController();
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('เพิ่ม Todo'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(hintText: 'ชื่องาน...'),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                ref.read(todoProvider.notifier).addTodo(controller.text);
                Navigator.pop(context);
              }
            },
            child: const Text('เพิ่ม'),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 154: FutureProvider และ StreamProvider

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Service
class WeatherService {
  Future<Map<String, dynamic>> getWeather(String city) async {
    await Future.delayed(const Duration(seconds: 1));
    // จำลองข้อมูลจาก API
    final data = {
      'กรุงเทพ': {'temp': 35, 'humidity': 75, 'condition': 'ร้อนจัด', 'icon': '☀️'},
      'เชียงใหม่': {'temp': 28, 'humidity': 60, 'condition': 'แดดร้อน', 'icon': '⛅'},
      'ภูเก็ต': {'temp': 32, 'humidity': 80, 'condition': 'มีเมฆ', 'icon': '🌤️'},
    };
    return data[city] ?? {'temp': 30, 'humidity': 70, 'condition': 'ปกติ', 'icon': '🌡️'};
  }

  Stream<List<String>> getLiveUpdates() async* {
    final messages = [
      'ข่าวสาร 1: อากาศวันนี้ร้อน',
      'ข่าวสาร 2: มีฝนตกบางพื้นที่',
      'ข่าวสาร 3: อุณหภูมิจะลดลงพรุ่งนี้',
    ];
    for (int i = 0; i < messages.length; i++) {
      await Future.delayed(const Duration(seconds: 2));
      yield messages.sublist(0, i + 1);
    }
  }
}

final weatherServiceProvider = Provider((ref) => WeatherService());

// FutureProvider - สำหรับ async data
final selectedCityProvider = StateProvider<String>((ref) => 'กรุงเทพ');

final weatherProvider = FutureProvider.autoDispose<Map<String, dynamic>>((ref) {
  final city = ref.watch(selectedCityProvider);
  final service = ref.read(weatherServiceProvider);
  return service.getWeather(city);
});

// StreamProvider - สำหรับ stream data
final liveUpdatesProvider = StreamProvider.autoDispose<List<String>>((ref) {
  return ref.read(weatherServiceProvider).getLiveUpdates();
});

class FutureStreamScreen extends ConsumerWidget {
  const FutureStreamScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final weatherAsync = ref.watch(weatherProvider);
    final updatesAsync = ref.watch(liveUpdatesProvider);
    final selectedCity = ref.watch(selectedCityProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Weather App')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // City selector
            SegmentedButton<String>(
              segments: const [
                ButtonSegment(value: 'กรุงเทพ', label: Text('กรุงเทพ')),
                ButtonSegment(value: 'เชียงใหม่', label: Text('เชียงใหม่')),
                ButtonSegment(value: 'ภูเก็ต', label: Text('ภูเก็ต')),
              ],
              selected: {selectedCity},
              onSelectionChanged: (cities) {
                ref.read(selectedCityProvider.notifier).state = cities.first;
              },
            ),

            const SizedBox(height: 24),

            // Weather data
            weatherAsync.when(
              data: (weather) => _buildWeatherCard(weather, selectedCity),
              loading: () => const CircularProgressIndicator(),
              error: (e, st) => Text('Error: $e'),
            ),

            const SizedBox(height: 24),
            const Text('Live Updates:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Stream data
            Expanded(
              child: updatesAsync.when(
                data: (updates) => ListView.builder(
                  itemCount: updates.length,
                  itemBuilder: (context, i) => ListTile(
                    leading: const Icon(Icons.notifications, color: Colors.blue),
                    title: Text(updates[i]),
                  ),
                ),
                loading: () => const Center(child: CircularProgressIndicator()),
                error: (e, st) => Text('Stream Error: $e'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildWeatherCard(Map<String, dynamic> weather, String city) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          children: [
            Text(weather['icon'] as String, style: const TextStyle(fontSize: 64)),
            Text(city, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            Text(
              '${weather['temp']}°C',
              style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
            ),
            Text(weather['condition'] as String),
            Text('ความชื้น: ${weather['humidity']}%'),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 155: NotifierProvider (Riverpod 2.0)

`NotifierProvider` คือ replacement ของ `StateNotifierProvider` ใน Riverpod 2.0

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// State
class CartState {
  final Map<String, int> items; // productId -> quantity
  final bool isProcessing;

  const CartState({
    this.items = const {},
    this.isProcessing = false,
  });

  CartState copyWith({Map<String, int>? items, bool? isProcessing}) {
    return CartState(
      items: items ?? this.items,
      isProcessing: isProcessing ?? this.isProcessing,
    );
  }

  int get totalItems => items.values.fold(0, (sum, qty) => sum + qty);
}

// Notifier (Riverpod 2.0)
class CartNotifier extends Notifier<CartState> {
  @override
  CartState build() {
    return const CartState();
  }

  void addItem(String productId) {
    final current = state.items[productId] ?? 0;
    state = state.copyWith(
      items: {...state.items, productId: current + 1},
    );
  }

  void removeItem(String productId) {
    final newItems = Map<String, int>.from(state.items);
    newItems.remove(productId);
    state = state.copyWith(items: newItems);
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    state = state.copyWith(
      items: {...state.items, productId: quantity},
    );
  }

  Future<bool> checkout() async {
    state = state.copyWith(isProcessing: true);
    await Future.delayed(const Duration(seconds: 2));
    state = const CartState(); // reset
    return true;
  }
}

final cartProvider = NotifierProvider<CartNotifier, CartState>(CartNotifier.new);

// AsyncNotifier - สำหรับ async operations
class UserNotifier extends AsyncNotifier<Map<String, dynamic>?> {
  @override
  Future<Map<String, dynamic>?> build() async {
    // โหลด user จาก storage หรือ API
    return null;
  }

  Future<void> login(String email, String password) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      await Future.delayed(const Duration(seconds: 1));
      if (password == '1234') {
        return {'email': email, 'name': 'สมชาย'};
      }
      throw Exception('รหัสผ่านไม่ถูกต้อง');
    });
  }

  void logout() {
    state = const AsyncData(null);
  }
}

final userProvider = AsyncNotifierProvider<UserNotifier, Map<String, dynamic>?>(
  UserNotifier.new,
);

// Screen
class NotifierScreen extends ConsumerWidget {
  const NotifierScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);
    final userAsync = ref.watch(userProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('NotifierProvider'),
        actions: [
          Badge(
            label: Text('${cart.totalItems}'),
            isLabelVisible: cart.totalItems > 0,
            child: const Icon(Icons.shopping_cart),
          ),
          const SizedBox(width: 16),
        ],
      ),
      body: Column(
        children: [
          // User section
          userAsync.when(
            data: (user) => user == null
                ? ElevatedButton(
                    onPressed: () async {
                      await ref.read(userProvider.notifier).login('test@test.com', '1234');
                    },
                    child: const Text('Login'),
                  )
                : ListTile(
                    title: Text('สวัสดี ${user["name"]}'),
                    trailing: TextButton(
                      onPressed: () => ref.read(userProvider.notifier).logout(),
                      child: const Text('Logout'),
                    ),
                  ),
            loading: () => const CircularProgressIndicator(),
            error: (e, _) => Text('Error: $e'),
          ),

          const Divider(),

          // Products
          Expanded(
            child: GridView.builder(
              padding: const EdgeInsets.all(16),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
                childAspectRatio: 1.5,
              ),
              itemCount: 6,
              itemBuilder: (context, index) {
                final productId = 'product_$index';
                final qty = cart.items[productId] ?? 0;
                return Card(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text('สินค้า ${index + 1}'),
                      if (qty > 0)
                        Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            IconButton(
                              icon: const Icon(Icons.remove, size: 16),
                              onPressed: () => ref.read(cartProvider.notifier).updateQuantity(productId, qty - 1),
                            ),
                            Text('$qty'),
                            IconButton(
                              icon: const Icon(Icons.add, size: 16),
                              onPressed: () => ref.read(cartProvider.notifier).addItem(productId),
                            ),
                          ],
                        )
                      else
                        TextButton(
                          onPressed: () => ref.read(cartProvider.notifier).addItem(productId),
                          child: const Text('เพิ่ม'),
                        ),
                    ],
                  ),
                );
              },
            ),
          ),
        ],
      ),
      floatingActionButton: cart.totalItems > 0
          ? FloatingActionButton.extended(
              onPressed: cart.isProcessing ? null : () async {
                final success = await ref.read(cartProvider.notifier).checkout();
                if (context.mounted && success) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('ชำระเงินสำเร็จ!')),
                  );
                }
              },
              label: cart.isProcessing
                  ? const SizedBox(width: 24, height: 24, child: CircularProgressIndicator(strokeWidth: 2))
                  : Text('Checkout (${cart.totalItems})'),
              icon: cart.isProcessing ? null : const Icon(Icons.payment),
            )
          : null,
    );
  }
}
```

---

## ขั้นตอนที่ 156: ConsumerWidget และ ConsumerStatefulWidget

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final themeProvider = StateProvider<ThemeMode>((ref) => ThemeMode.system);
final counterProvider2 = StateProvider<int>((ref) => 0);

// ConsumerWidget - สำหรับ StatelessWidget ที่ต้องการ WidgetRef
class CounterScreen extends ConsumerWidget {
  const CounterScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref ใช้แทน context ใน Provider
    final count = ref.watch(counterProvider2);

    return Scaffold(
      appBar: AppBar(title: const Text('ConsumerWidget')),
      body: Center(
        child: Text(
          '$count',
          style: Theme.of(context).textTheme.displayLarge,
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider2.notifier).state++,
        child: const Icon(Icons.add),
      ),
    );
  }
}

// ConsumerStatefulWidget - สำหรับ StatefulWidget ที่ต้องการ WidgetRef
class AnimatedCounterScreen extends ConsumerStatefulWidget {
  const AnimatedCounterScreen({super.key});

  @override
  ConsumerState<AnimatedCounterScreen> createState() =>
      _AnimatedCounterScreenState();
}

class _AnimatedCounterScreenState extends ConsumerState<AnimatedCounterScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 200),
    );
    _scaleAnimation = Tween<double>(begin: 1, end: 1.3).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );

    // ฟัง provider ใน initState
    ref.listenManual(counterProvider2, (previous, next) {
      if (next > (previous ?? 0)) {
        _controller.forward().then((_) => _controller.reverse());
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // ใช้ ref ได้ใน build เหมือน ConsumerWidget
    final count = ref.watch(counterProvider2);

    return Scaffold(
      appBar: AppBar(title: const Text('ConsumerStatefulWidget')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ScaleTransition(
              scale: _scaleAnimation,
              child: Text(
                '$count',
                style: Theme.of(context).textTheme.displayLarge,
              ),
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => ref.read(counterProvider2.notifier).state--,
                  child: const Text('-'),
                ),
                const SizedBox(width: 16),
                ElevatedButton(
                  onPressed: () => ref.read(counterProvider2.notifier).state++,
                  child: const Text('+'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

// HookConsumerWidget (ต้องติดตั้ง hooks_riverpod)
// class MyHookWidget extends HookConsumerWidget {
//   Widget build(BuildContext context, WidgetRef ref) {
//     final textController = useTextEditingController();
//     final count = ref.watch(counterProvider);
//     return ...;
//   }
// }
```

---

## ขั้นตอนที่ 157: ref.watch, ref.read, ref.listen

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final stockPriceProvider = StateProvider<double>((ref) => 100.0);
final alertThresholdProvider = StateProvider<double>((ref) => 150.0);

class RefMethodsScreen extends ConsumerStatefulWidget {
  const RefMethodsScreen({super.key});

  @override
  ConsumerState<RefMethodsScreen> createState() => _RefMethodsScreenState();
}

class _RefMethodsScreenState extends ConsumerState<RefMethodsScreen> {
  final List<String> _log = [];

  @override
  void initState() {
    super.initState();

    // ref.listen - ฟังการเปลี่ยนแปลงโดยไม่ rebuild widget
    ref.listen<double>(
      stockPriceProvider,
      (previous, next) {
        final threshold = ref.read(alertThresholdProvider);
        setState(() {
          _log.add('ราคา: $previous → $next');
          if (next > threshold) {
            _log.add('⚠️ ราคาเกิน threshold! ($next > $threshold)');
          }
        });
      },
      fireImmediately: false, // ไม่ต้อง fire ทันที
    );
  }

  @override
  Widget build(BuildContext context) {
    // ref.watch - subscribe และ rebuild เมื่อเปลี่ยน
    final price = ref.watch(stockPriceProvider);
    final threshold = ref.watch(alertThresholdProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('ref methods')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text(
                      'ราคาหุ้น: ${price.toStringAsFixed(2)} บาท',
                      style: TextStyle(
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                        color: price >= threshold ? Colors.red : Colors.green,
                      ),
                    ),
                    Text('Alert threshold: $threshold บาท'),
                    Slider(
                      value: price,
                      min: 50,
                      max: 200,
                      // ref.read ใน callbacks - ไม่ rebuild
                      onChanged: (v) =>
                          ref.read(stockPriceProvider.notifier).state = v,
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),
            const Text('Log:', style: TextStyle(fontWeight: FontWeight.bold)),
            Expanded(
              child: ListView.builder(
                reverse: true,
                itemCount: _log.length,
                itemBuilder: (context, index) => Text(
                  _log[_log.length - 1 - index],
                  style: TextStyle(
                    color: _log[_log.length - 1 - index].startsWith('⚠️')
                        ? Colors.red
                        : null,
                  ),
                ),
              ),
            ),

            ElevatedButton(
              onPressed: () => setState(() => _log.clear()),
              child: const Text('ล้าง log'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 158: family และ autoDispose

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// family - สร้าง provider จาก parameter
final userByIdProvider = FutureProvider.family<Map<String, dynamic>, String>(
  (ref, userId) async {
    await Future.delayed(const Duration(milliseconds: 500));
    // Fetch user by ID
    return {
      'id': userId,
      'name': 'User $userId',
      'email': 'user$userId@example.com',
    };
  },
);

// autoDispose - ทำลาย provider เมื่อไม่มีใช้
final searchResultsProvider = FutureProvider.autoDispose.family<List<String>, String>(
  (ref, query) async {
    // cancel เมื่อ provider ถูก dispose
    ref.onDispose(() => print('Search provider disposed'));

    if (query.isEmpty) return [];
    await Future.delayed(const Duration(milliseconds: 300));
    return List.generate(
      5,
      (i) => 'ผลการค้นหา "$query" #${i + 1}',
    );
  },
);

// keepAlive - ป้องกัน autoDispose (เก็บ cache ไว้)
final cachedDataProvider = FutureProvider.autoDispose<List<String>>((ref) async {
  final link = ref.keepAlive(); // ป้องกันการ dispose
  
  // ยกเลิก keepAlive หลัง 30 วินาที
  Future.delayed(const Duration(seconds: 30), () {
    link.close();
  });

  await Future.delayed(const Duration(seconds: 1));
  return ['Data 1', 'Data 2', 'Data 3'];
});

class FamilyAutoDisposeScreen extends ConsumerStatefulWidget {
  const FamilyAutoDisposeScreen({super.key});

  @override
  ConsumerState<FamilyAutoDisposeScreen> createState() =>
      _FamilyAutoDisposeScreenState();
}

class _FamilyAutoDisposeScreenState
    extends ConsumerState<FamilyAutoDisposeScreen> {
  String _searchQuery = '';
  final _searchController = TextEditingController();

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('family & autoDispose')),
      body: Column(
        children: [
          // Search with autoDispose.family
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: _searchController,
              decoration: const InputDecoration(
                hintText: 'ค้นหา...',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(),
              ),
              onChanged: (query) => setState(() => _searchQuery = query),
            ),
          ),

          if (_searchQuery.isNotEmpty)
            Consumer(
              builder: (context, ref, _) {
                final results = ref.watch(searchResultsProvider(_searchQuery));
                return results.when(
                  data: (items) => Column(
                    children: items
                        .map((item) => ListTile(title: Text(item)))
                        .toList(),
                  ),
                  loading: () => const LinearProgressIndicator(),
                  error: (e, _) => Text('Error: $e'),
                );
              },
            ),

          const Divider(),
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('Users (family):',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),

          // User list with family
          ListView.builder(
            shrinkWrap: true,
            itemCount: 3,
            itemBuilder: (context, index) {
              final userId = 'user_${index + 1}';
              return Consumer(
                builder: (context, ref, _) {
                  final userAsync = ref.watch(userByIdProvider(userId));
                  return userAsync.when(
                    data: (user) => ListTile(
                      leading: CircleAvatar(child: Text('${index + 1}')),
                      title: Text(user['name'] as String),
                      subtitle: Text(user['email'] as String),
                    ),
                    loading: () => const ListTile(title: LinearProgressIndicator()),
                    error: (e, _) => ListTile(title: Text('Error: $e')),
                  );
                },
              );
            },
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 159: Async Notifiers

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class Post {
  final int id;
  final String title;
  final String body;

  const Post({required this.id, required this.title, required this.body});
}

// AsyncNotifier - build() returns Future
class PostsNotifier extends AsyncNotifier<List<Post>> {
  @override
  Future<List<Post>> build() async {
    return _fetchPosts();
  }

  Future<List<Post>> _fetchPosts() async {
    await Future.delayed(const Duration(seconds: 1));
    return List.generate(
      10,
      (i) => Post(
        id: i + 1,
        title: 'โพสต์ที่ ${i + 1}: เรื่องน่าสนใจ',
        body: 'เนื้อหาโพสต์ ${i + 1} ที่มีรายละเอียดครบครัน...',
      ),
    );
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(_fetchPosts);
  }

  Future<void> addPost(String title, String body) async {
    // Optimistic update
    final currentPosts = state.valueOrNull ?? [];
    final newPost = Post(id: DateTime.now().millisecondsSinceEpoch, title: title, body: body);
    state = AsyncData([newPost, ...currentPosts]);

    try {
      // API call (จำลอง)
      await Future.delayed(const Duration(milliseconds: 500));
    } catch (e) {
      // Rollback on error
      state = AsyncData(currentPosts);
      rethrow;
    }
  }

  void deletePost(int postId) {
    state.whenData((posts) {
      state = AsyncData(posts.where((p) => p.id != postId).toList());
    });
  }
}

final postsProvider = AsyncNotifierProvider<PostsNotifier, List<Post>>(
  PostsNotifier.new,
);

class AsyncNotifierScreen extends ConsumerWidget {
  const AsyncNotifierScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final postsAsync = ref.watch(postsProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Async Notifier'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => ref.read(postsProvider.notifier).refresh(),
          ),
        ],
      ),
      body: postsAsync.when(
        data: (posts) => RefreshIndicator(
          onRefresh: () => ref.read(postsProvider.notifier).refresh(),
          child: ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) {
              final post = posts[index];
              return Dismissible(
                key: Key('${post.id}'),
                onDismissed: (_) =>
                    ref.read(postsProvider.notifier).deletePost(post.id),
                child: ListTile(
                  title: Text(post.title),
                  subtitle: Text(
                    post.body,
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                ),
              );
            },
          ),
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, st) => Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Error: $e'),
              ElevatedButton(
                onPressed: () => ref.read(postsProvider.notifier).refresh(),
                child: const Text('ลองอีกครั้ง'),
              ),
            ],
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(postsProvider.notifier).addPost(
          'โพสต์ใหม่ ${DateTime.now().millisecondsSinceEpoch}',
          'เนื้อหาโพสต์ใหม่',
        ),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 160: Riverpod Testing

```dart
// test/riverpod_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';

// Provider ที่ต้อง test
final counterProvider3 = StateProvider<int>((ref) => 0);

class SimpleCounter extends ConsumerWidget {
  const SimpleCounter({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider3);
    return Column(
      children: [
        Text('$count', key: const Key('count')),
        ElevatedButton(
          key: const Key('increment'),
          onPressed: () => ref.read(counterProvider3.notifier).state++,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

void main() {
  group('Counter Tests', () {
    // Unit test สำหรับ provider
    test('counter เริ่มที่ 0', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      expect(container.read(counterProvider3), 0);
    });

    test('counter เพิ่มขึ้น', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      container.read(counterProvider3.notifier).state++;
      expect(container.read(counterProvider3), 1);
    });

    // Widget test
    testWidgets('SimpleCounter widget', (tester) async {
      await tester.pumpWidget(
        const ProviderScope(
          child: MaterialApp(
            home: Scaffold(body: SimpleCounter()),
          ),
        ),
      );

      expect(find.text('0'), findsOneWidget);
      await tester.tap(find.byKey(const Key('increment')));
      await tester.pump();
      expect(find.text('1'), findsOneWidget);
    });

    // Test กับ override provider
    testWidgets('ทดสอบด้วย mock provider', (tester) async {
      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            // Override provider สำหรับ test
            counterProvider3.overrideWith((ref) => 100),
          ],
          child: const MaterialApp(
            home: Scaffold(body: SimpleCounter()),
          ),
        ),
      );

      expect(find.text('100'), findsOneWidget);
    });
  });
}
```

---

## Workshop: Task Manager App

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Models
enum TaskPriority { low, medium, high }
enum TaskStatus { todo, inProgress, done }

class Task {
  final String id;
  final String title;
  final String description;
  final TaskPriority priority;
  final TaskStatus status;
  final DateTime createdAt;
  final DateTime? dueDate;

  const Task({
    required this.id,
    required this.title,
    required this.description,
    required this.priority,
    required this.status,
    required this.createdAt,
    this.dueDate,
  });

  Task copyWith({
    String? title,
    String? description,
    TaskPriority? priority,
    TaskStatus? status,
    DateTime? dueDate,
  }) {
    return Task(
      id: id,
      title: title ?? this.title,
      description: description ?? this.description,
      priority: priority ?? this.priority,
      status: status ?? this.status,
      createdAt: createdAt,
      dueDate: dueDate ?? this.dueDate,
    );
  }
}

// State
class TaskManagerState {
  final List<Task> tasks;
  final TaskStatus filterStatus;
  final TaskPriority? filterPriority;
  final String searchQuery;
  final bool isLoading;

  const TaskManagerState({
    this.tasks = const [],
    this.filterStatus = TaskStatus.todo,
    this.filterPriority,
    this.searchQuery = '',
    this.isLoading = false,
  });

  List<Task> get filteredTasks {
    return tasks.where((task) {
      final matchesStatus = task.status == filterStatus;
      final matchesPriority = filterPriority == null || task.priority == filterPriority;
      final matchesSearch = searchQuery.isEmpty ||
          task.title.toLowerCase().contains(searchQuery.toLowerCase());
      return matchesStatus && matchesPriority && matchesSearch;
    }).toList()
      ..sort((a, b) {
        // Sort by priority (high first)
        return b.priority.index.compareTo(a.priority.index);
      });
  }

  Map<TaskStatus, int> get statusCounts {
    final counts = <TaskStatus, int>{};
    for (final status in TaskStatus.values) {
      counts[status] = tasks.where((t) => t.status == status).length;
    }
    return counts;
  }

  TaskManagerState copyWith({
    List<Task>? tasks,
    TaskStatus? filterStatus,
    TaskPriority? filterPriority,
    String? searchQuery,
    bool? isLoading,
  }) {
    return TaskManagerState(
      tasks: tasks ?? this.tasks,
      filterStatus: filterStatus ?? this.filterStatus,
      filterPriority: filterPriority,
      searchQuery: searchQuery ?? this.searchQuery,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

// Notifier
class TaskManagerNotifier extends Notifier<TaskManagerState> {
  @override
  TaskManagerState build() {
    return TaskManagerState(
      tasks: _initialTasks(),
    );
  }

  List<Task> _initialTasks() {
    return [
      Task(id: '1', title: 'ออกแบบ UI', description: 'สร้าง wireframe', priority: TaskPriority.high, status: TaskStatus.todo, createdAt: DateTime.now()),
      Task(id: '2', title: 'เขียน API', description: 'สร้าง REST endpoints', priority: TaskPriority.high, status: TaskStatus.inProgress, createdAt: DateTime.now()),
      Task(id: '3', title: 'เขียน Tests', description: 'unit และ widget tests', priority: TaskPriority.medium, status: TaskStatus.todo, createdAt: DateTime.now()),
      Task(id: '4', title: 'Deploy', description: 'deploy to production', priority: TaskPriority.low, status: TaskStatus.done, createdAt: DateTime.now()),
    ];
  }

  void addTask(Task task) {
    state = state.copyWith(tasks: [...state.tasks, task]);
  }

  void updateTask(Task task) {
    state = state.copyWith(
      tasks: state.tasks.map((t) => t.id == task.id ? task : t).toList(),
    );
  }

  void deleteTask(String taskId) {
    state = state.copyWith(
      tasks: state.tasks.where((t) => t.id != taskId).toList(),
    );
  }

  void moveTask(String taskId, TaskStatus newStatus) {
    state = state.copyWith(
      tasks: state.tasks.map((t) {
        if (t.id == taskId) return t.copyWith(status: newStatus);
        return t;
      }).toList(),
    );
  }

  void setFilter(TaskStatus status) {
    state = state.copyWith(filterStatus: status);
  }

  void setPriorityFilter(TaskPriority? priority) {
    state = state.copyWith(filterPriority: priority);
  }

  void setSearch(String query) {
    state = state.copyWith(searchQuery: query);
  }
}

final taskManagerProvider = NotifierProvider<TaskManagerNotifier, TaskManagerState>(
  TaskManagerNotifier.new,
);

// App
void taskManagerMain() {
  runApp(const ProviderScope(child: TaskManagerApp()));
}

class TaskManagerApp extends StatelessWidget {
  const TaskManagerApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Task Manager',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
        useMaterial3: true,
      ),
      home: const TaskManagerScreen(),
    );
  }
}

class TaskManagerScreen extends ConsumerStatefulWidget {
  const TaskManagerScreen({super.key});

  @override
  ConsumerState<TaskManagerScreen> createState() => _TaskManagerScreenState();
}

class _TaskManagerScreenState extends ConsumerState<TaskManagerScreen>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
    _tabController.addListener(() {
      if (!_tabController.indexIsChanging) {
        final statuses = [TaskStatus.todo, TaskStatus.inProgress, TaskStatus.done];
        ref.read(taskManagerProvider.notifier).setFilter(statuses[_tabController.index]);
      }
    });
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final taskState = ref.watch(taskManagerProvider);
    final counts = taskState.statusCounts;

    return Scaffold(
      appBar: AppBar(
        title: const Text('📋 Task Manager'),
        bottom: TabBar(
          controller: _tabController,
          tabs: [
            Tab(text: 'Todo (${counts[TaskStatus.todo] ?? 0})'),
            Tab(text: 'In Progress (${counts[TaskStatus.inProgress] ?? 0})'),
            Tab(text: 'Done (${counts[TaskStatus.done] ?? 0})'),
          ],
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: _showSearch,
          ),
          PopupMenuButton<TaskPriority?>(
            icon: const Icon(Icons.filter_list),
            onSelected: (p) => ref.read(taskManagerProvider.notifier).setPriorityFilter(p),
            itemBuilder: (context) => [
              const PopupMenuItem(value: null, child: Text('ทั้งหมด')),
              const PopupMenuItem(value: TaskPriority.high, child: Text('🔴 สูง')),
              const PopupMenuItem(value: TaskPriority.medium, child: Text('🟡 กลาง')),
              const PopupMenuItem(value: TaskPriority.low, child: Text('🟢 ต่ำ')),
            ],
          ),
        ],
      ),
      body: TabBarView(
        controller: _tabController,
        children: [TaskStatus.todo, TaskStatus.inProgress, TaskStatus.done]
            .map((status) => _TaskList(status: status))
            .toList(),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _showAddTask,
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showSearch() {
    showSearch(
      context: context,
      delegate: _TaskSearchDelegate(ref),
    );
  }

  void _showAddTask() {
    final titleController = TextEditingController();
    final descController = TextEditingController();
    TaskPriority selectedPriority = TaskPriority.medium;

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (context) => Padding(
        padding: EdgeInsets.only(
          bottom: MediaQuery.of(context).viewInsets.bottom,
          left: 16, right: 16, top: 16,
        ),
        child: StatefulBuilder(
          builder: (context, setSheet) {
            return Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                const Text('เพิ่มงานใหม่', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                const SizedBox(height: 16),
                TextField(
                  controller: titleController,
                  decoration: const InputDecoration(labelText: 'ชื่องาน', border: OutlineInputBorder()),
                ),
                const SizedBox(height: 8),
                TextField(
                  controller: descController,
                  decoration: const InputDecoration(labelText: 'รายละเอียด', border: OutlineInputBorder()),
                  maxLines: 2,
                ),
                const SizedBox(height: 8),
                SegmentedButton<TaskPriority>(
                  segments: const [
                    ButtonSegment(value: TaskPriority.low, label: Text('ต่ำ')),
                    ButtonSegment(value: TaskPriority.medium, label: Text('กลาง')),
                    ButtonSegment(value: TaskPriority.high, label: Text('สูง')),
                  ],
                  selected: {selectedPriority},
                  onSelectionChanged: (p) => setSheet(() => selectedPriority = p.first),
                ),
                const SizedBox(height: 16),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      if (titleController.text.isNotEmpty) {
                        ref.read(taskManagerProvider.notifier).addTask(
                          Task(
                            id: DateTime.now().millisecondsSinceEpoch.toString(),
                            title: titleController.text,
                            description: descController.text,
                            priority: selectedPriority,
                            status: TaskStatus.todo,
                            createdAt: DateTime.now(),
                          ),
                        );
                        Navigator.pop(context);
                      }
                    },
                    child: const Text('เพิ่ม'),
                  ),
                ),
                const SizedBox(height: 16),
              ],
            );
          },
        ),
      ),
    );
  }
}

class _TaskList extends ConsumerWidget {
  final TaskStatus status;
  const _TaskList({required this.status});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final tasks = ref.watch(
      taskManagerProvider.select((s) => s.filteredTasks),
    );

    if (tasks.isEmpty) {
      return const Center(child: Text('ไม่มีงาน'));
    }

    return ListView.builder(
      padding: const EdgeInsets.all(8),
      itemCount: tasks.length,
      itemBuilder: (context, index) {
        final task = tasks[index];
        return _TaskCard(task: task);
      },
    );
  }
}

class _TaskCard extends ConsumerWidget {
  final Task task;
  const _TaskCard({required this.task});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final priorityColors = {
      TaskPriority.low: Colors.green,
      TaskPriority.medium: Colors.orange,
      TaskPriority.high: Colors.red,
    };

    final priorityLabels = {
      TaskPriority.low: 'ต่ำ',
      TaskPriority.medium: 'กลาง',
      TaskPriority.high: 'สูง',
    };

    return Card(
      margin: const EdgeInsets.symmetric(vertical: 4),
      child: ListTile(
        leading: Container(
          width: 4,
          height: 40,
          color: priorityColors[task.priority],
        ),
        title: Text(task.title, style: const TextStyle(fontWeight: FontWeight.bold)),
        subtitle: Text(task.description, maxLines: 1, overflow: TextOverflow.ellipsis),
        trailing: PopupMenuButton<TaskStatus>(
          onSelected: (status) =>
              ref.read(taskManagerProvider.notifier).moveTask(task.id, status),
          itemBuilder: (context) => TaskStatus.values
              .map((s) => PopupMenuItem(
                    value: s,
                    child: Text(['Todo', 'In Progress', 'Done'][s.index]),
                  ))
              .toList(),
          child: Chip(
            label: Text(
              priorityLabels[task.priority]!,
              style: TextStyle(color: priorityColors[task.priority], fontSize: 11),
            ),
            backgroundColor: priorityColors[task.priority]!.withOpacity(0.1),
          ),
        ),
      ),
    );
  }
}

class _TaskSearchDelegate extends SearchDelegate<String> {
  final WidgetRef ref;
  _TaskSearchDelegate(this.ref);

  @override
  List<Widget>? buildActions(BuildContext context) => [
    IconButton(icon: const Icon(Icons.clear), onPressed: () => query = ''),
  ];

  @override
  Widget? buildLeading(BuildContext context) =>
      IconButton(icon: const Icon(Icons.arrow_back), onPressed: () => close(context, ''));

  @override
  Widget buildResults(BuildContext context) => _buildList();

  @override
  Widget buildSuggestions(BuildContext context) {
    ref.read(taskManagerProvider.notifier).setSearch(query);
    return _buildList();
  }

  Widget _buildList() {
    return Consumer(
      builder: (context, ref, _) {
        final tasks = ref.watch(
          taskManagerProvider.select((s) => s.filteredTasks),
        );
        return ListView.builder(
          itemCount: tasks.length,
          itemBuilder: (context, index) => ListTile(
            title: Text(tasks[index].title),
            subtitle: Text(tasks[index].description),
          ),
        );
      },
    );
  }
}

void main() => taskManagerMain();
```

---

## สรุป (Summary)

| Provider Type | ใช้สำหรับ |
|---------------|---------|
| Provider | ค่าคงที่, services |
| StateProvider | state ง่ายๆ (int, bool, String) |
| StateNotifierProvider | state ซับซ้อน (Riverpod 1.x) |
| NotifierProvider | state ซับซ้อน (Riverpod 2.x) |
| FutureProvider | async data |
| StreamProvider | stream data |
| AsyncNotifierProvider | async state management |
| .family | สร้าง provider จาก parameter |
| .autoDispose | ทำลาย provider อัตโนมัติ |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง counter app ด้วย StateProvider พร้อม reset button
2. **ปานกลาง**: สร้าง weather app ที่ fetch ข้อมูลจาก API ด้วย FutureProvider
3. **ยาก**: สร้าง chat app ด้วย StreamProvider ที่ simulate real-time messages

---

[← Part 15: Provider Pattern](part_15.md) | [Part 17: HTTP & REST APIs →](part_17.md)

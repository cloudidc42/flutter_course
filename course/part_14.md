# Part 14: State Management: setState & InheritedWidget
## ขั้นตอนที่ 131-140

---

## สารบัญ
1. [setState แบบเจาะลึก](#ขั้นตอนที่-131-setstate-แบบเจาะลึก)
2. [Lifting State Up](#ขั้นตอนที่-132-lifting-state-up)
3. [InheritedWidget จากศูนย์](#ขั้นตอนที่-133-inheritedwidget-จากศูนย์)
4. [InheritedModel](#ขั้นตอนที่-134-inheritedmodel)
5. [ValueNotifier และ ValueListenableBuilder](#ขั้นตอนที่-135-valuenotifier-และ-valuelistenablebuilder)
6. [ChangeNotifier พื้นฐาน](#ขั้นตอนที่-136-changenotifier-พื้นฐาน)
7. [ListenableBuilder](#ขั้นตอนที่-137-listenablebuilder)
8. [State Management Patterns](#ขั้นตอนที่-138-state-management-patterns)
9. [Performance Optimization](#ขั้นตอนที่-139-performance-optimization)
10. [Testing Stateful Widgets](#ขั้นตอนที่-140-testing-stateful-widgets)
11. [Workshop: Shopping Cart](#workshop-shopping-cart)

---

## ขั้นตอนที่ 131: setState แบบเจาะลึก

`setState` เป็น mechanism พื้นฐานที่สุดใน Flutter สำหรับอัปเดต UI เมื่อ state เปลี่ยนแปลง

```dart
import 'package:flutter/material.dart';

class SetStateDeepDive extends StatefulWidget {
  const SetStateDeepDive({super.key});

  @override
  State<SetStateDeepDive> createState() => _SetStateDeepDiveState();
}

class _SetStateDeepDiveState extends State<SetStateDeepDive> {
  int _counter = 0;
  String _text = 'เริ่มต้น';
  bool _isLoading = false;
  List<String> _items = [];

  // ❌ ผิด: setState ที่มี async code ข้างใน (ไม่ควรทำ)
  // void _wrongWay() {
  //   setState(() async { // ผิด! setState ไม่รับ async callback
  //     await Future.delayed(Duration(seconds: 1));
  //     _counter++;
  //   });
  // }

  // ✅ ถูก: async แล้วค่อย setState
  Future<void> _rightWay() async {
    setState(() => _isLoading = true);

    await Future.delayed(const Duration(seconds: 1));
    final result = await _fetchData();

    if (mounted) { // ตรวจสอบว่า widget ยังอยู่ใน tree
      setState(() {
        _text = result;
        _isLoading = false;
      });
    }
  }

  Future<String> _fetchData() async {
    return 'ข้อมูลจาก API';
  }

  // setState เฉพาะส่วนที่ต้องการ - ดีกว่า rebuild ทั้งหมด
  void _incrementCounter() {
    setState(() => _counter++); // เปลี่ยนเฉพาะ _counter
  }

  void _addItem() {
    setState(() => _items.add('รายการ ${_items.length + 1}'));
  }

  // ❌ ผิด: เปลี่ยนค่าโดยตรงโดยไม่ setState
  // void _wrong() {
  //   _counter++; // UI จะไม่อัปเดต!
  // }

  @override
  Widget build(BuildContext context) {
    print('build ถูกเรียก'); // เพื่อดูว่า rebuild กี่ครั้ง
    return Scaffold(
      appBar: AppBar(title: const Text('setState Deep Dive')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Counter
            Card(
              child: ListTile(
                title: Text('Counter: $_counter'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.remove),
                      onPressed: () => setState(() => _counter--),
                    ),
                    IconButton(
                      icon: const Icon(Icons.add),
                      onPressed: _incrementCounter,
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Async setState
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    Text('ข้อความ: $_text'),
                    const SizedBox(height: 8),
                    _isLoading
                        ? const CircularProgressIndicator()
                        : ElevatedButton(
                            onPressed: _rightWay,
                            child: const Text('โหลดข้อมูล (async)'),
                          ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // List management
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text('รายการ (${_items.length}):'),
                ElevatedButton(onPressed: _addItem, child: const Text('เพิ่ม')),
              ],
            ),
            Expanded(
              child: ListView.builder(
                itemCount: _items.length,
                itemBuilder: (context, index) => ListTile(
                  title: Text(_items[index]),
                  trailing: IconButton(
                    icon: const Icon(Icons.delete, color: Colors.red),
                    onPressed: () => setState(() => _items.removeAt(index)),
                  ),
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

### ข้อควรระวังใน setState

```dart
// 1. ตรวจสอบ mounted ก่อน setState ใน async method
Future<void> _loadData() async {
  final data = await fetchData();
  if (mounted) { // สำคัญ!
    setState(() => _data = data);
  }
}

// 2. หลีกเลี่ยง setState ใน build()
// ❌ ผิด
@override
Widget build(BuildContext context) {
  setState(() => _counter++); // ทำให้ infinite loop!
  return Container();
}

// 3. setState กับ List - ต้องสร้าง List ใหม่หรือ modify แล้วค่อย setState
void _addItem(String item) {
  setState(() {
    _items = [..._items, item]; // สร้าง list ใหม่
    // หรือ
    // _items.add(item); // modify ใน setState ก็ได้
  });
}
```

---

## ขั้นตอนที่ 132: Lifting State Up

เมื่อ widget หลายตัวต้องการ state เดียวกัน ให้ยก state ขึ้นไปไว้ที่ parent

```dart
import 'package:flutter/material.dart';

// ❌ Bad: state อยู่คนละ widget ทำให้ sync ยาก
class BadDesign extends StatelessWidget {
  const BadDesign({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        _CounterButton(), // มี state ของตัวเอง
        _CounterDisplay(), // มี state ของตัวเอง - ไม่ sync กัน!
      ],
    );
  }
}

// ✅ Good: Lifting state up
class TemperatureConverter extends StatefulWidget {
  const TemperatureConverter({super.key});

  @override
  State<TemperatureConverter> createState() => _TemperatureConverterState();
}

class _TemperatureConverterState extends State<TemperatureConverter> {
  // State อยู่ที่ parent - ทั้ง Celsius และ Fahrenheit ใช้ค่าเดียวกัน
  double _celsius = 0;

  double get _fahrenheit => _celsius * 9 / 5 + 32;
  double get _kelvin => _celsius + 273.15;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Temperature Converter')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Celsius Input - เปลี่ยน state ของ parent
            _TemperatureInput(
              label: 'เซลเซียส (°C)',
              value: _celsius,
              onChanged: (value) => setState(() => _celsius = value),
            ),
            const SizedBox(height: 16),

            // Fahrenheit Input - เปลี่ยน state ของ parent เช่นกัน
            _TemperatureInput(
              label: 'ฟาเรนไฮต์ (°F)',
              value: _fahrenheit,
              onChanged: (value) {
                setState(() => _celsius = (value - 32) * 5 / 9);
              },
            ),
            const SizedBox(height: 16),

            // Kelvin - แค่แสดงผล
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('เคลวิน (K):', style: TextStyle(fontSize: 16)),
                    Text(
                      _kelvin.toStringAsFixed(2),
                      style: const TextStyle(
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                        color: Colors.purple,
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 24),
            Slider(
              value: _celsius.clamp(-50, 150),
              min: -50,
              max: 150,
              divisions: 200,
              label: '${_celsius.toStringAsFixed(1)}°C',
              onChanged: (value) => setState(() => _celsius = value),
            ),
          ],
        ),
      ),
    );
  }
}

// Child widget - รับ callback จาก parent
class _TemperatureInput extends StatelessWidget {
  final String label;
  final double value;
  final ValueChanged<double> onChanged;

  const _TemperatureInput({
    required this.label,
    required this.value,
    required this.onChanged,
  });

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      key: ValueKey(value.toStringAsFixed(2)),
      initialValue: value.toStringAsFixed(2),
      decoration: InputDecoration(
        labelText: label,
        border: const OutlineInputBorder(),
        suffixIcon: const Icon(Icons.thermostat),
      ),
      keyboardType: const TextInputType.numberWithOptions(decimal: true),
      onChanged: (text) {
        final parsed = double.tryParse(text);
        if (parsed != null) onChanged(parsed);
      },
    );
  }
}
```

---

## ขั้นตอนที่ 133: InheritedWidget จากศูนย์

`InheritedWidget` ช่วยส่ง state ลงไปใน widget tree โดยไม่ต้อง pass ผ่าน constructor ทุกระดับ

```dart
import 'package:flutter/material.dart';

// 1. สร้าง Model
class ThemeModel {
  final bool isDarkMode;
  final Color primaryColor;
  final double fontSize;

  const ThemeModel({
    required this.isDarkMode,
    required this.primaryColor,
    required this.fontSize,
  });

  ThemeModel copyWith({
    bool? isDarkMode,
    Color? primaryColor,
    double? fontSize,
  }) {
    return ThemeModel(
      isDarkMode: isDarkMode ?? this.isDarkMode,
      primaryColor: primaryColor ?? this.primaryColor,
      fontSize: fontSize ?? this.fontSize,
    );
  }
}

// 2. สร้าง InheritedWidget
class ThemeInherited extends InheritedWidget {
  final ThemeModel theme;
  final void Function(ThemeModel) onThemeChanged;

  const ThemeInherited({
    super.key,
    required this.theme,
    required this.onThemeChanged,
    required super.child,
  });

  // Static method สำหรับ access จาก child
  static ThemeInherited of(BuildContext context) {
    final result = context.dependOnInheritedWidgetOfExactType<ThemeInherited>();
    assert(result != null, 'ThemeInherited not found in widget tree');
    return result!;
  }

  // อีกแบบ - nullable (safe)
  static ThemeInherited? maybeOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ThemeInherited>();
  }

  // updateShouldNotify - บอกว่าควร notify children เมื่อไหร่
  @override
  bool updateShouldNotify(ThemeInherited oldWidget) {
    return theme != oldWidget.theme;
  }
}

// 3. Provider Widget (ตัวจัดการ state)
class ThemeProvider extends StatefulWidget {
  final Widget child;

  const ThemeProvider({super.key, required this.child});

  @override
  State<ThemeProvider> createState() => _ThemeProviderState();
}

class _ThemeProviderState extends State<ThemeProvider> {
  ThemeModel _theme = const ThemeModel(
    isDarkMode: false,
    primaryColor: Colors.blue,
    fontSize: 16,
  );

  void _updateTheme(ThemeModel newTheme) {
    setState(() => _theme = newTheme);
  }

  @override
  Widget build(BuildContext context) {
    return ThemeInherited(
      theme: _theme,
      onThemeChanged: _updateTheme,
      child: widget.child,
    );
  }
}

// 4. การใช้งาน
class InheritedWidgetApp extends StatelessWidget {
  const InheritedWidgetApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      child: Builder(
        builder: (context) {
          final theme = ThemeInherited.of(context).theme;
          return MaterialApp(
            theme: ThemeData(
              colorScheme: ColorScheme.fromSeed(
                seedColor: theme.primaryColor,
                brightness: theme.isDarkMode ? Brightness.dark : Brightness.light,
              ),
            ),
            home: const InheritedWidgetScreen(),
          );
        },
      ),
    );
  }
}

class InheritedWidgetScreen extends StatelessWidget {
  const InheritedWidgetScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // เข้าถึง InheritedWidget ด้วย of()
    final inherited = ThemeInherited.of(context);
    final theme = inherited.theme;

    return Scaffold(
      appBar: AppBar(title: const Text('InheritedWidget')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // แสดง settings ปัจจุบัน
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    SwitchListTile(
                      title: const Text('Dark Mode'),
                      value: theme.isDarkMode,
                      onChanged: (value) {
                        inherited.onThemeChanged(
                          theme.copyWith(isDarkMode: value),
                        );
                      },
                    ),
                    ListTile(
                      title: const Text('ขนาดตัวอักษร'),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(
                            icon: const Icon(Icons.remove),
                            onPressed: () => inherited.onThemeChanged(
                              theme.copyWith(fontSize: theme.fontSize - 2),
                            ),
                          ),
                          Text(theme.fontSize.toStringAsFixed(0)),
                          IconButton(
                            icon: const Icon(Icons.add),
                            onPressed: () => inherited.onThemeChanged(
                              theme.copyWith(fontSize: theme.fontSize + 2),
                            ),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Widget ที่ใช้ theme
            _NestedWidget(),
          ],
        ),
      ),
    );
  }
}

// Widget ลึก - ยังเข้าถึง InheritedWidget ได้โดยตรง
class _NestedWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final theme = ThemeInherited.of(context).theme;
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Text(
          'Widget ลึก - ใช้ fontSize: ${theme.fontSize}',
          style: TextStyle(fontSize: theme.fontSize),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 134: InheritedModel

`InheritedModel` เหมือน InheritedWidget แต่ rebuild เฉพาะ aspect ที่เปลี่ยน

```dart
import 'package:flutter/material.dart';

// Aspects ที่ model มี
enum SettingsAspect { theme, language, notifications }

class AppSettings {
  final bool darkMode;
  final String language;
  final bool notifications;

  const AppSettings({
    required this.darkMode,
    required this.language,
    required this.notifications,
  });
}

class SettingsModel extends InheritedModel<SettingsAspect> {
  final AppSettings settings;
  final void Function(AppSettings) onChanged;

  const SettingsModel({
    super.key,
    required this.settings,
    required this.onChanged,
    required super.child,
  });

  static SettingsModel of(BuildContext context, SettingsAspect aspect) {
    return InheritedModel.inheritFrom<SettingsModel>(context, aspect: aspect)!;
  }

  @override
  bool updateShouldNotify(SettingsModel old) {
    return settings != old.settings;
  }

  @override
  bool updateShouldNotifyDependent(
    SettingsModel old,
    Set<SettingsAspect> aspects,
  ) {
    // Rebuild เฉพาะ widget ที่ depend on aspect ที่เปลี่ยนแปลง
    if (aspects.contains(SettingsAspect.theme) &&
        settings.darkMode != old.settings.darkMode) {
      return true;
    }
    if (aspects.contains(SettingsAspect.language) &&
        settings.language != old.settings.language) {
      return true;
    }
    if (aspects.contains(SettingsAspect.notifications) &&
        settings.notifications != old.settings.notifications) {
      return true;
    }
    return false;
  }
}

class InheritedModelScreen extends StatefulWidget {
  const InheritedModelScreen({super.key});

  @override
  State<InheritedModelScreen> createState() => _InheritedModelScreenState();
}

class _InheritedModelScreenState extends State<InheritedModelScreen> {
  AppSettings _settings = const AppSettings(
    darkMode: false,
    language: 'ไทย',
    notifications: true,
  );

  @override
  Widget build(BuildContext context) {
    return SettingsModel(
      settings: _settings,
      onChanged: (s) => setState(() => _settings = s),
      child: Scaffold(
        appBar: AppBar(title: const Text('InheritedModel')),
        body: const Column(
          children: [
            _ThemeWidget(),   // rebuild เมื่อ theme เปลี่ยน
            _LanguageWidget(), // rebuild เมื่อ language เปลี่ยน
            _NotifWidget(),   // rebuild เมื่อ notifications เปลี่ยน
            _SettingsControl(),
          ],
        ),
      ),
    );
  }
}

class _ThemeWidget extends StatelessWidget {
  const _ThemeWidget();

  @override
  Widget build(BuildContext context) {
    // depend on theme aspect เท่านั้น
    final model = SettingsModel.of(context, SettingsAspect.theme);
    print('_ThemeWidget rebuilt');
    return ListTile(
      leading: Icon(
        model.settings.darkMode ? Icons.dark_mode : Icons.light_mode,
      ),
      title: Text(model.settings.darkMode ? 'Dark Mode' : 'Light Mode'),
    );
  }
}

class _LanguageWidget extends StatelessWidget {
  const _LanguageWidget();

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context, SettingsAspect.language);
    print('_LanguageWidget rebuilt');
    return ListTile(
      leading: const Icon(Icons.language),
      title: Text('ภาษา: ${model.settings.language}'),
    );
  }
}

class _NotifWidget extends StatelessWidget {
  const _NotifWidget();

  @override
  Widget build(BuildContext context) {
    final model = SettingsModel.of(context, SettingsAspect.notifications);
    print('_NotifWidget rebuilt');
    return ListTile(
      leading: Icon(
        model.settings.notifications
            ? Icons.notifications_active
            : Icons.notifications_off,
      ),
      title: Text('การแจ้งเตือน: ${model.settings.notifications ? "เปิด" : "ปิด"}'),
    );
  }
}

class _SettingsControl extends StatelessWidget {
  const _SettingsControl();

  @override
  Widget build(BuildContext context) {
    // ใช้ aspect ไหนก็ได้เพื่อเข้าถึง model
    final model = SettingsModel.of(context, SettingsAspect.theme);
    final settings = model.settings;

    return Card(
      margin: const EdgeInsets.all(16),
      child: Column(
        children: [
          SwitchListTile(
            title: const Text('Dark Mode'),
            value: settings.darkMode,
            onChanged: (v) => model.onChanged(
              AppSettings(
                darkMode: v,
                language: settings.language,
                notifications: settings.notifications,
              ),
            ),
          ),
          SwitchListTile(
            title: const Text('การแจ้งเตือน'),
            value: settings.notifications,
            onChanged: (v) => model.onChanged(
              AppSettings(
                darkMode: settings.darkMode,
                language: settings.language,
                notifications: v,
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 135: ValueNotifier และ ValueListenableBuilder

`ValueNotifier` เป็น wrapper ง่ายๆ สำหรับ single value ที่ notify เมื่อเปลี่ยน

```dart
import 'package:flutter/material.dart';

class ValueNotifierScreen extends StatelessWidget {
  // ValueNotifier ไม่ต้องอยู่ใน State
  final ValueNotifier<int> _counter = ValueNotifier<int>(0);
  final ValueNotifier<bool> _isDark = ValueNotifier<bool>(false);
  final ValueNotifier<List<String>> _items = ValueNotifier<List<String>>([]);

  ValueNotifierScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ValueNotifier'),
        actions: [
          // ใช้ ValueListenableBuilder สำหรับ icon ใน AppBar
          ValueListenableBuilder<bool>(
            valueListenable: _isDark,
            builder: (context, isDark, child) {
              return IconButton(
                icon: Icon(isDark ? Icons.light_mode : Icons.dark_mode),
                onPressed: () => _isDark.value = !_isDark.value,
              );
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Counter
            ValueListenableBuilder<int>(
              valueListenable: _counter,
              builder: (context, count, child) {
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      children: [
                        Text(
                          '$count',
                          style: const TextStyle(
                            fontSize: 48,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            IconButton(
                              icon: const Icon(Icons.remove),
                              onPressed: () => _counter.value--,
                            ),
                            IconButton(
                              icon: const Icon(Icons.add),
                              onPressed: () => _counter.value++,
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),

            const SizedBox(height: 16),

            // Items list
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                ValueListenableBuilder<List<String>>(
                  valueListenable: _items,
                  builder: (context, items, child) {
                    return Text('รายการ: ${items.length}');
                  },
                ),
                ElevatedButton(
                  onPressed: () {
                    // ต้องสร้าง list ใหม่เพื่อ trigger update
                    _items.value = [..._items.value, 'Item ${_items.value.length + 1}'];
                  },
                  child: const Text('เพิ่ม'),
                ),
              ],
            ),
            Expanded(
              child: ValueListenableBuilder<List<String>>(
                valueListenable: _items,
                builder: (context, items, child) {
                  return ListView.builder(
                    itemCount: items.length,
                    itemBuilder: (context, index) => ListTile(
                      title: Text(items[index]),
                      trailing: IconButton(
                        icon: const Icon(Icons.delete, color: Colors.red),
                        onPressed: () {
                          final newList = [...items];
                          newList.removeAt(index);
                          _items.value = newList;
                        },
                      ),
                    ),
                  );
                },
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

## ขั้นตอนที่ 136: ChangeNotifier พื้นฐาน

`ChangeNotifier` สำหรับ state ที่ซับซ้อนกว่า ValueNotifier

```dart
import 'package:flutter/material.dart';

// ChangeNotifier Model
class CartModel extends ChangeNotifier {
  final List<CartItem> _items = [];
  double _discount = 0;

  List<CartItem> get items => List.unmodifiable(_items);
  int get itemCount => _items.length;
  double get discount => _discount;

  double get subtotal => _items.fold(
    0,
    (sum, item) => sum + item.price * item.quantity,
  );

  double get total => subtotal * (1 - _discount);

  void addItem(Product product) {
    final existingIndex = _items.indexWhere((i) => i.product.id == product.id);
    if (existingIndex >= 0) {
      _items[existingIndex].quantity++;
    } else {
      _items.add(CartItem(product: product, quantity: 1));
    }
    notifyListeners(); // แจ้ง listeners ทุกตัวที่ฟัง
  }

  void removeItem(String productId) {
    _items.removeWhere((i) => i.product.id == productId);
    notifyListeners();
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    final index = _items.indexWhere((i) => i.product.id == productId);
    if (index >= 0) {
      _items[index].quantity = quantity;
      notifyListeners();
    }
  }

  void applyDiscount(double discount) {
    _discount = discount.clamp(0, 1);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    _discount = 0;
    notifyListeners();
  }
}

class Product {
  final String id;
  final String name;
  final double price;
  final String emoji;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.emoji,
  });
}

class CartItem {
  final Product product;
  int quantity;

  CartItem({required this.product, required this.quantity});
}

// Screen ที่ใช้ CartModel โดยตรง (ไม่ผ่าน Provider)
class CartScreen extends StatefulWidget {
  const CartScreen({super.key});

  @override
  State<CartScreen> createState() => _CartScreenState();
}

class _CartScreenState extends State<CartScreen> {
  final CartModel _cart = CartModel();
  final List<Product> _products = const [
    Product(id: '1', name: 'iPhone 15', price: 39900, emoji: '📱'),
    Product(id: '2', name: 'MacBook Air', price: 45900, emoji: '💻'),
    Product(id: '3', name: 'AirPods Pro', price: 9990, emoji: '🎧'),
    Product(id: '4', name: 'iPad Air', price: 22900, emoji: '📱'),
  ];

  @override
  void initState() {
    super.initState();
    _cart.addListener(_onCartChanged);
  }

  @override
  void dispose() {
    _cart.removeListener(_onCartChanged);
    _cart.dispose(); // dispose ChangeNotifier
    super.dispose();
  }

  void _onCartChanged() {
    setState(() {}); // rebuild เมื่อ cart เปลี่ยน
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ตะกร้าสินค้า'),
        actions: [
          Badge(
            label: Text('${_cart.itemCount}'),
            isLabelVisible: _cart.itemCount > 0,
            child: IconButton(
              icon: const Icon(Icons.shopping_cart),
              onPressed: _showCart,
            ),
          ),
        ],
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
          childAspectRatio: 0.85,
        ),
        itemCount: _products.length,
        itemBuilder: (context, index) {
          return _buildProductCard(_products[index]);
        },
      ),
      bottomNavigationBar: _cart.itemCount > 0
          ? BottomAppBar(
              child: Padding(
                padding: const EdgeInsets.all(8),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text(
                      'รวม: ฿${_cart.total.toStringAsFixed(2)}',
                      style: const TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    ElevatedButton(
                      onPressed: () {},
                      child: const Text('ชำระเงิน'),
                    ),
                  ],
                ),
              ),
            )
          : null,
    );
  }

  Widget _buildProductCard(Product product) {
    final inCart = _cart.items.any((i) => i.product.id == product.id);

    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Container(
              color: Colors.grey[100],
              alignment: Alignment.center,
              child: Text(product.emoji, style: const TextStyle(fontSize: 48)),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product.name,
                  style: const TextStyle(fontWeight: FontWeight.bold),
                  overflow: TextOverflow.ellipsis,
                ),
                Text(
                  '฿${product.price.toStringAsFixed(0)}',
                  style: const TextStyle(color: Colors.red),
                ),
                const SizedBox(height: 4),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () => _cart.addItem(product),
                    style: inCart
                        ? ElevatedButton.styleFrom(
                            backgroundColor: Colors.green,
                          )
                        : null,
                    child: Text(inCart ? '✓ เพิ่มแล้ว' : 'เพิ่มลงตะกร้า'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  void _showCart() {
    showModalBottomSheet(
      context: context,
      builder: (context) => StatefulBuilder(
        builder: (context, setModalState) {
          return Column(
            children: [
              Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    const Text('ตะกร้า', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                    TextButton(
                      onPressed: () {
                        _cart.clear();
                        setModalState(() {});
                        Navigator.pop(context);
                      },
                      child: const Text('ล้างทั้งหมด'),
                    ),
                  ],
                ),
              ),
              Expanded(
                child: ListView.builder(
                  itemCount: _cart.items.length,
                  itemBuilder: (context, index) {
                    final item = _cart.items[index];
                    return ListTile(
                      leading: Text(item.product.emoji, style: const TextStyle(fontSize: 24)),
                      title: Text(item.product.name),
                      subtitle: Text('฿${item.product.price}'),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(
                            icon: const Icon(Icons.remove),
                            onPressed: () {
                              _cart.updateQuantity(item.product.id, item.quantity - 1);
                              setModalState(() {});
                            },
                          ),
                          Text('${item.quantity}'),
                          IconButton(
                            icon: const Icon(Icons.add),
                            onPressed: () {
                              _cart.updateQuantity(item.product.id, item.quantity + 1);
                              setModalState(() {});
                            },
                          ),
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
                    Text('รวม: ฿${_cart.total.toStringAsFixed(2)}',
                        style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                    ElevatedButton(
                      onPressed: () {},
                      child: const Text('ชำระเงิน'),
                    ),
                  ],
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

## ขั้นตอนที่ 137: ListenableBuilder

`ListenableBuilder` เป็น widget ใหม่ (Flutter 3.x) ที่ใช้แทน AnimatedBuilder สำหรับ non-animation Listenable

```dart
import 'package:flutter/material.dart';

class CounterNotifier extends ChangeNotifier {
  int _count = 0;
  String _history = '';

  int get count => _count;
  String get history => _history;

  void increment() {
    _count++;
    _history += '+1 (รวม: $_count)\n';
    notifyListeners();
  }

  void decrement() {
    _count--;
    _history += '-1 (รวม: $_count)\n';
    notifyListeners();
  }

  void reset() {
    _count = 0;
    _history += 'Reset\n';
    notifyListeners();
  }
}

class ListenableBuilderScreen extends StatefulWidget {
  const ListenableBuilderScreen({super.key});

  @override
  State<ListenableBuilderScreen> createState() => _ListenableBuilderScreenState();
}

class _ListenableBuilderScreenState extends State<ListenableBuilderScreen> {
  final CounterNotifier _counter = CounterNotifier();

  @override
  void dispose() {
    _counter.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ListenableBuilder')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // ListenableBuilder - rebuild เฉพาะ subtree นี้เมื่อ notifier เปลี่ยน
            ListenableBuilder(
              listenable: _counter,
              builder: (context, child) {
                return Card(
                  child: Padding(
                    padding: const EdgeInsets.all(24),
                    child: Column(
                      children: [
                        Text(
                          '${_counter.count}',
                          style: const TextStyle(
                            fontSize: 72,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            FloatingActionButton(
                              heroTag: 'dec',
                              onPressed: _counter.decrement,
                              child: const Icon(Icons.remove),
                            ),
                            const SizedBox(width: 16),
                            FloatingActionButton(
                              heroTag: 'inc',
                              onPressed: _counter.increment,
                              child: const Icon(Icons.add),
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),

            const SizedBox(height: 16),

            // แสดง history
            Expanded(
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          const Text('ประวัติ:', style: TextStyle(fontWeight: FontWeight.bold)),
                          TextButton(
                            onPressed: _counter.reset,
                            child: const Text('Reset'),
                          ),
                        ],
                      ),
                      Expanded(
                        child: ListenableBuilder(
                          listenable: _counter,
                          builder: (context, child) {
                            return SingleChildScrollView(
                              child: Text(_counter.history),
                            );
                          },
                        ),
                      ),
                    ],
                  ),
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

## ขั้นตอนที่ 138: State Management Patterns

```dart
import 'package:flutter/material.dart';

// Pattern: Scoped State ด้วย StatefulWidget + InheritedWidget
class AppStateData {
  final int counter;
  final List<String> todos;
  final bool isLoggedIn;

  const AppStateData({
    required this.counter,
    required this.todos,
    required this.isLoggedIn,
  });

  AppStateData copyWith({
    int? counter,
    List<String>? todos,
    bool? isLoggedIn,
  }) =>
      AppStateData(
        counter: counter ?? this.counter,
        todos: todos ?? this.todos,
        isLoggedIn: isLoggedIn ?? this.isLoggedIn,
      );
}

// Store pattern
class AppStore extends ChangeNotifier {
  AppStateData _state = const AppStateData(
    counter: 0,
    todos: [],
    isLoggedIn: false,
  );

  AppStateData get state => _state;

  void increment() {
    _state = _state.copyWith(counter: _state.counter + 1);
    notifyListeners();
  }

  void addTodo(String todo) {
    _state = _state.copyWith(todos: [..._state.todos, todo]);
    notifyListeners();
  }

  void removeTodo(int index) {
    final newTodos = [..._state.todos]..removeAt(index);
    _state = _state.copyWith(todos: newTodos);
    notifyListeners();
  }

  void login() {
    _state = _state.copyWith(isLoggedIn: true);
    notifyListeners();
  }

  void logout() {
    _state = _state.copyWith(isLoggedIn: false);
    notifyListeners();
  }
}
```

---

## ขั้นตอนที่ 139: Performance Optimization

```dart
import 'package:flutter/material.dart';

class PerformanceScreen extends StatefulWidget {
  const PerformanceScreen({super.key});

  @override
  State<PerformanceScreen> createState() => _PerformanceScreenState();
}

class _PerformanceScreenState extends State<PerformanceScreen> {
  int _counter = 0;
  String _name = 'Flutter';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Performance Optimization')),
      body: Column(
        children: [
          // 1. const widget ไม่ถูก rebuild
          const _ExpensiveWidget(data: 'ข้อมูลคงที่'),

          // 2. Widget ที่เปลี่ยนแปลง
          Text('Counter: $_counter', style: const TextStyle(fontSize: 24)),
          Text('Name: $_name'),

          // 3. ใช้ const สำหรับ subtree ที่ไม่เปลี่ยนแปลง
          ElevatedButton(
            onPressed: () => setState(() => _counter++),
            child: const Text('Increment'),
          ),
        ],
      ),
    );
  }
}

// Widget ที่ประมวลผลหนัก
class _ExpensiveWidget extends StatelessWidget {
  final String data;

  const _ExpensiveWidget({required this.data}); // const constructor

  @override
  Widget build(BuildContext context) {
    // จำลองการทำงานหนัก
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Text('Expensive: $data'),
    );
  }
}
```

---

## ขั้นตอนที่ 140: Testing Stateful Widgets

```dart
// test/widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

// Widget ที่ต้องการ test
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count', key: const Key('counter_text')),
        ElevatedButton(
          key: const Key('increment_button'),
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// Tests
void main() {
  group('CounterWidget', () {
    testWidgets('เริ่มที่ 0', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(home: Scaffold(body: CounterWidget())),
      );

      expect(find.text('Count: 0'), findsOneWidget);
    });

    testWidgets('เพิ่มขึ้น 1 เมื่อกดปุ่ม', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(home: Scaffold(body: CounterWidget())),
      );

      await tester.tap(find.byKey(const Key('increment_button')));
      await tester.pump();

      expect(find.text('Count: 1'), findsOneWidget);
    });

    testWidgets('เพิ่มขึ้นหลายครั้ง', (tester) async {
      await tester.pumpWidget(
        const MaterialApp(home: Scaffold(body: CounterWidget())),
      );

      for (int i = 0; i < 5; i++) {
        await tester.tap(find.byKey(const Key('increment_button')));
        await tester.pump();
      }

      expect(find.text('Count: 5'), findsOneWidget);
    });
  });
}
```

---

## Workshop: Shopping Cart

```dart
import 'package:flutter/material.dart';

// Models
class Product {
  final String id;
  final String name;
  final double price;
  final String category;
  final String emoji;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.category,
    required this.emoji,
  });
}

class CartItem {
  final Product product;
  int quantity;

  CartItem({required this.product, this.quantity = 1});

  double get total => product.price * quantity;
}

// State Management
class ShoppingCartState extends ChangeNotifier {
  final Map<String, CartItem> _cart = {};
  String _selectedCategory = 'ทั้งหมด';
  String _searchQuery = '';
  bool _isCheckingOut = false;

  Map<String, CartItem> get cart => Map.unmodifiable(_cart);
  String get selectedCategory => _selectedCategory;
  String get searchQuery => _searchQuery;
  bool get isCheckingOut => _isCheckingOut;

  int get itemCount => _cart.values.fold(0, (sum, item) => sum + item.quantity);
  double get subtotal => _cart.values.fold(0, (sum, item) => sum + item.total);
  double get tax => subtotal * 0.07;
  double get total => subtotal + tax;

  void addToCart(Product product) {
    if (_cart.containsKey(product.id)) {
      _cart[product.id]!.quantity++;
    } else {
      _cart[product.id] = CartItem(product: product);
    }
    notifyListeners();
  }

  void removeFromCart(String productId) {
    _cart.remove(productId);
    notifyListeners();
  }

  void updateQuantity(String productId, int quantity) {
    if (quantity <= 0) {
      removeFromCart(productId);
    } else {
      _cart[productId]?.quantity = quantity;
      notifyListeners();
    }
  }

  void setCategory(String category) {
    _selectedCategory = category;
    notifyListeners();
  }

  void setSearchQuery(String query) {
    _searchQuery = query;
    notifyListeners();
  }

  Future<void> checkout() async {
    _isCheckingOut = true;
    notifyListeners();
    await Future.delayed(const Duration(seconds: 2));
    _cart.clear();
    _isCheckingOut = false;
    notifyListeners();
  }

  bool isInCart(String productId) => _cart.containsKey(productId);
  int getQuantity(String productId) => _cart[productId]?.quantity ?? 0;
}

// Product Data
const _allProducts = [
  Product(id: 'p1', name: 'iPhone 15 Pro', price: 49900, category: 'มือถือ', emoji: '📱'),
  Product(id: 'p2', name: 'Samsung S24', price: 35900, category: 'มือถือ', emoji: '📱'),
  Product(id: 'p3', name: 'MacBook Air M3', price: 42900, category: 'คอมพิวเตอร์', emoji: '💻'),
  Product(id: 'p4', name: 'iPad Pro', price: 32900, category: 'แท็บเล็ต', emoji: '📱'),
  Product(id: 'p5', name: 'AirPods Pro', price: 9990, category: 'หูฟัง', emoji: '🎧'),
  Product(id: 'p6', name: 'Sony WH-1000XM5', price: 12990, category: 'หูฟัง', emoji: '🎧'),
  Product(id: 'p7', name: 'Apple Watch', price: 14900, category: 'นาฬิกา', emoji: '⌚'),
  Product(id: 'p8', name: 'Samsung Watch', price: 8990, category: 'นาฬิกา', emoji: '⌚'),
];

// Main App
class ShoppingApp extends StatelessWidget {
  const ShoppingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Shopping Cart',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
        useMaterial3: true,
      ),
      home: const ShoppingScreen(),
    );
  }
}

class ShoppingScreen extends StatefulWidget {
  const ShoppingScreen({super.key});

  @override
  State<ShoppingScreen> createState() => _ShoppingScreenState();
}

class _ShoppingScreenState extends State<ShoppingScreen> {
  final ShoppingCartState _cartState = ShoppingCartState();
  final TextEditingController _searchController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _cartState.addListener(() => setState(() {}));
    _searchController.addListener(() {
      _cartState.setSearchQuery(_searchController.text);
    });
  }

  @override
  void dispose() {
    _cartState.removeListener(() {});
    _cartState.dispose();
    _searchController.dispose();
    super.dispose();
  }

  List<Product> get _filteredProducts {
    return _allProducts.where((p) {
      final matchesCategory = _cartState.selectedCategory == 'ทั้งหมด' ||
          p.category == _cartState.selectedCategory;
      final matchesSearch = _cartState.searchQuery.isEmpty ||
          p.name.toLowerCase().contains(_cartState.searchQuery.toLowerCase());
      return matchesCategory && matchesSearch;
    }).toList();
  }

  List<String> get _categories {
    final cats = _allProducts.map((p) => p.category).toSet().toList();
    return ['ทั้งหมด', ...cats];
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('🛍️ ร้านค้า'),
        actions: [
          Badge(
            label: Text('${_cartState.itemCount}'),
            isLabelVisible: _cartState.itemCount > 0,
            child: IconButton(
              icon: const Icon(Icons.shopping_cart),
              onPressed: _showCartSheet,
            ),
          ),
        ],
      ),
      body: Column(
        children: [
          // Search bar
          Padding(
            padding: const EdgeInsets.all(16),
            child: TextField(
              controller: _searchController,
              decoration: const InputDecoration(
                hintText: 'ค้นหาสินค้า...',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.all(Radius.circular(12)),
                ),
                contentPadding: EdgeInsets.symmetric(vertical: 8),
              ),
            ),
          ),

          // Category filter
          SizedBox(
            height: 40,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 16),
              itemCount: _categories.length,
              itemBuilder: (context, index) {
                final cat = _categories[index];
                final isSelected = cat == _cartState.selectedCategory;
                return Padding(
                  padding: const EdgeInsets.only(right: 8),
                  child: FilterChip(
                    label: Text(cat),
                    selected: isSelected,
                    onSelected: (_) => _cartState.setCategory(cat),
                  ),
                );
              },
            ),
          ),

          const SizedBox(height: 8),

          // Products grid
          Expanded(
            child: GridView.builder(
              padding: const EdgeInsets.all(16),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
                crossAxisSpacing: 12,
                mainAxisSpacing: 12,
                childAspectRatio: 0.8,
              ),
              itemCount: _filteredProducts.length,
              itemBuilder: (context, index) {
                return _buildProductCard(_filteredProducts[index]);
              },
            ),
          ),
        ],
      ),
      bottomNavigationBar: _cartState.itemCount > 0
          ? Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Theme.of(context).colorScheme.surface,
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 8,
                    offset: const Offset(0, -4),
                  ),
                ],
              ),
              child: ElevatedButton(
                onPressed: _showCartSheet,
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                ),
                child: Text(
                  'ดูตะกร้า (${_cartState.itemCount} รายการ) · ฿${_cartState.total.toStringAsFixed(0)}',
                  style: const TextStyle(fontSize: 16),
                ),
              ),
            )
          : null,
    );
  }

  Widget _buildProductCard(Product product) {
    final inCart = _cartState.isInCart(product.id);
    final qty = _cartState.getQuantity(product.id);

    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Container(
              alignment: Alignment.center,
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: const BorderRadius.vertical(top: Radius.circular(12)),
              ),
              child: Text(product.emoji, style: const TextStyle(fontSize: 48)),
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
                Text(
                  '฿${product.price.toStringAsFixed(0)}',
                  style: const TextStyle(color: Colors.red, fontSize: 14),
                ),
                if (inCart)
                  Row(
                    children: [
                      IconButton(
                        icon: const Icon(Icons.remove, size: 18),
                        padding: EdgeInsets.zero,
                        constraints: const BoxConstraints(),
                        onPressed: () => _cartState.updateQuantity(product.id, qty - 1),
                      ),
                      Expanded(
                        child: Text(
                          '$qty',
                          textAlign: TextAlign.center,
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ),
                      IconButton(
                        icon: const Icon(Icons.add, size: 18),
                        padding: EdgeInsets.zero,
                        constraints: const BoxConstraints(),
                        onPressed: () => _cartState.addToCart(product),
                      ),
                    ],
                  )
                else
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () => _cartState.addToCart(product),
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.symmetric(vertical: 4),
                      ),
                      child: const Text('เพิ่ม', style: TextStyle(fontSize: 12)),
                    ),
                  ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  void _showCartSheet() {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      useSafeArea: true,
      builder: (context) => StatefulBuilder(
        builder: (context, setSheet) {
          void rebuild() {
            setState(() {});
            setSheet(() {});
          }

          return DraggableScrollableSheet(
            initialChildSize: 0.7,
            minChildSize: 0.4,
            maxChildSize: 0.95,
            expand: false,
            builder: (context, scrollController) {
              return Column(
                children: [
                  Container(
                    padding: const EdgeInsets.all(16),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text('ตะกร้าสินค้า (${_cartState.itemCount})',
                            style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                        TextButton(
                          onPressed: () {
                            for (final id in _cartState.cart.keys.toList()) {
                              _cartState.removeFromCart(id);
                            }
                            rebuild();
                          },
                          child: const Text('ล้าง'),
                        ),
                      ],
                    ),
                  ),
                  Expanded(
                    child: ListView.builder(
                      controller: scrollController,
                      itemCount: _cartState.cart.length,
                      itemBuilder: (context, index) {
                        final item = _cartState.cart.values.toList()[index];
                        return ListTile(
                          leading: Text(item.product.emoji, style: const TextStyle(fontSize: 28)),
                          title: Text(item.product.name),
                          subtitle: Text('฿${item.total.toStringAsFixed(0)}'),
                          trailing: Row(
                            mainAxisSize: MainAxisSize.min,
                            children: [
                              IconButton(
                                icon: const Icon(Icons.remove),
                                onPressed: () {
                                  _cartState.updateQuantity(item.product.id, item.quantity - 1);
                                  rebuild();
                                },
                              ),
                              Text('${item.quantity}'),
                              IconButton(
                                icon: const Icon(Icons.add),
                                onPressed: () {
                                  _cartState.addToCart(item.product);
                                  rebuild();
                                },
                              ),
                            ],
                          ),
                        );
                      },
                    ),
                  ),
                  Container(
                    padding: const EdgeInsets.all(16),
                    child: Column(
                      children: [
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            const Text('ราคาสินค้า:'),
                            Text('฿${_cartState.subtotal.toStringAsFixed(2)}'),
                          ],
                        ),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            const Text('VAT 7%:'),
                            Text('฿${_cartState.tax.toStringAsFixed(2)}'),
                          ],
                        ),
                        const Divider(),
                        Row(
                          mainAxisAlignment: MainAxisAlignment.spaceBetween,
                          children: [
                            const Text('รวมทั้งสิ้น:', style: TextStyle(fontWeight: FontWeight.bold)),
                            Text(
                              '฿${_cartState.total.toStringAsFixed(2)}',
                              style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 18),
                            ),
                          ],
                        ),
                        const SizedBox(height: 8),
                        SizedBox(
                          width: double.infinity,
                          child: ElevatedButton(
                            onPressed: _cartState.isCheckingOut ? null : () async {
                              await _cartState.checkout();
                              if (mounted) Navigator.pop(context);
                              rebuild();
                            },
                            style: ElevatedButton.styleFrom(
                              padding: const EdgeInsets.symmetric(vertical: 16),
                            ),
                            child: _cartState.isCheckingOut
                                ? const CircularProgressIndicator()
                                : const Text('ชำระเงิน', style: TextStyle(fontSize: 18)),
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              );
            },
          );
        },
      ),
    );
  }
}

void main() => runApp(const ShoppingApp());
```

---

## สรุป (Summary)

| หัวข้อ | เหมาะกับ |
|--------|---------|
| setState | State ภายใน widget เดียว |
| Lifting State Up | State ที่ใช้ร่วมระหว่าง siblings |
| InheritedWidget | State ที่ต้องส่งลึกใน tree |
| InheritedModel | State ที่มีหลาย aspect |
| ValueNotifier | Single value ที่ observable |
| ChangeNotifier | State object ที่ซับซ้อน |
| ListenableBuilder | rebuild เฉพาะส่วนที่ใช้ listenable |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง theme toggle (light/dark) ด้วย ValueNotifier
2. **ปานกลาง**: สร้าง todo list ด้วย ChangeNotifier พร้อม filter และ sort
3. **ยาก**: สร้าง InheritedWidget สำหรับ user authentication state

---

[← Part 13: Gestures](part_13.md) | [Part 15: Provider Pattern →](part_15.md)

# Part 06: Stateful & Stateless Widgets
## ขั้นตอนที่ 51-60

---

## สารบัญ
1. [Widget Lifecycle](#ขั้นตอนที่-51-widget-lifecycle)
2. [StatefulWidget เชิงลึก](#ขั้นตอนที่-52-statefulwidget-เชิงลึก)
3. [Keys ใน Flutter](#ขั้นตอนที่-53-keys-ใน-flutter)
4. [BuildContext](#ขั้นตอนที่-54-buildcontext)
5. [InheritedWidget](#ขั้นตอนที่-55-inheritedwidget)
6. [Widget Composition](#ขั้นตอนที่-56-widget-composition)
7. [Separation of Concerns](#ขั้นตอนที่-57-separation-of-concerns)
8. [Pure Widgets](#ขั้นตอนที่-58-pure-widgets)
9. [Performance Tips](#ขั้นตอนที่-59-performance-tips)
10. [Workshop: Counter App (Advanced)](#ขั้นตอนที่-60-workshop-counter-app-advanced)

---

## ขั้นตอนที่ 51: Widget Lifecycle

ทำความเข้าใจ lifecycle ของ StatefulWidget

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const LifecycleApp());
}

class LifecycleApp extends StatelessWidget {
  const LifecycleApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Widget Lifecycle',
      theme: ThemeData(useMaterial3: true),
      home: const LifecycleDemoScreen(),
    );
  }
}

class LifecycleDemoScreen extends StatefulWidget {
  const LifecycleDemoScreen({super.key});

  @override
  State<LifecycleDemoScreen> createState() => _LifecycleDemoScreenState();
}

class _LifecycleDemoScreenState extends State<LifecycleDemoScreen> {
  bool _showChild = true;
  int _rebuildCount = 0;
  String _lastEvent = 'เริ่มต้น';

  @override
  Widget build(BuildContext context) {
    _rebuildCount++;
    return Scaffold(
      appBar: AppBar(title: const Text('Widget Lifecycle')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // แสดงข้อมูล lifecycle
            Card(
              color: Colors.blue.shade50,
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text('Parent Widget',
                        style: TextStyle(fontWeight: FontWeight.bold)),
                    Text('Build count: $_rebuildCount'),
                    Text('Last event: $_lastEvent'),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),

            // ปุ่มควบคุม
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: () => setState(() {
                    _lastEvent = 'setState called';
                  }),
                  child: const Text('setState'),
                ),
                ElevatedButton(
                  onPressed: () => setState(() {
                    _showChild = !_showChild;
                    _lastEvent = _showChild ? 'child shown' : 'child hidden';
                  }),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: _showChild ? Colors.red : Colors.green,
                    foregroundColor: Colors.white,
                  ),
                  child: Text(_showChild ? 'ซ่อน Child' : 'แสดง Child'),
                ),
              ],
            ),

            const SizedBox(height: 16),

            // แสดง Child Widget
            if (_showChild) const _LifecycleChild(label: 'Child Widget'),
          ],
        ),
      ),
    );
  }
}

// Child Widget ที่แสดง lifecycle events
class _LifecycleChild extends StatefulWidget {
  final String label;

  const _LifecycleChild({required this.label});

  @override
  State<_LifecycleChild> createState() => _LifecycleChildState();
}

class _LifecycleChildState extends State<_LifecycleChild> {
  final List<String> _events = [];
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    // เรียกครั้งเดียวตอน widget สร้าง
    _addEvent('initState() - widget สร้างแล้ว');

    // ตัวอย่างการ setup ใน initState
    // - initialize controllers
    // - subscribe to streams
    // - fetch initial data
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // เรียกหลัง initState และเมื่อ InheritedWidget เปลี่ยน
    _addEvent('didChangeDependencies() - dependency เปลี่ยน');
  }

  @override
  void didUpdateWidget(_LifecycleChild oldWidget) {
    super.didUpdateWidget(oldWidget);
    // เรียกเมื่อ parent rebuild และส่ง widget ใหม่
    if (oldWidget.label != widget.label) {
      _addEvent('didUpdateWidget() - label เปลี่ยนเป็น ${widget.label}');
    }
  }

  @override
  void deactivate() {
    super.deactivate();
    // เรียกเมื่อ widget ถูกลบออกจาก tree ชั่วคราว
    _addEvent('deactivate() - ลบออกชั่วคราว');
  }

  @override
  void dispose() {
    // เรียกครั้งสุดท้ายก่อน widget ถูกทำลาย
    // ต้องยกเลิก: controllers, streams, animations, timers
    _addEvent('dispose() - widget ถูกทำลาย');
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    _addEvent('build() - ครั้งที่ ${_events.where((e) => e.contains("build")).length + 1}');
    return Card(
      color: Colors.green.shade50,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                const Icon(Icons.widgets, color: Colors.green),
                const SizedBox(width: 8),
                Text(
                  widget.label,
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                const Spacer(),
                Text('Counter: $_counter',
                    style: const TextStyle(
                        fontWeight: FontWeight.bold, fontSize: 18)),
                const SizedBox(width: 8),
                IconButton(
                  icon: const Icon(Icons.add_circle, color: Colors.green),
                  onPressed: () => setState(() => _counter++),
                ),
              ],
            ),
            const Divider(),
            const Text('Lifecycle Events:',
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 12)),
            const SizedBox(height: 4),
            SizedBox(
              height: 120,
              child: ListView.builder(
                reverse: true,
                itemCount: _events.length,
                itemBuilder: (_, i) {
                  final index = _events.length - 1 - i;
                  return Padding(
                    padding: const EdgeInsets.symmetric(vertical: 1),
                    child: Text(
                      '• ${_events[index]}',
                      style: TextStyle(
                        fontSize: 11,
                        color: index == _events.length - 1
                            ? Colors.green.shade700
                            : Colors.grey,
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

  void _addEvent(String event) {
    // ใช้ addPostFrameCallback เพื่อ avoid setState during build
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) {
        setState(() => _events.add(event));
      }
    });
  }
}
```

### Lifecycle Methods สรุป

```dart
class _MyWidgetState extends State<MyWidget> {

  // 1. initState - เรียกครั้งเดียวตอนสร้าง widget
  // ใช้สำหรับ: init controllers, fetch data, setup subscriptions
  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _fetchData();
  }

  // 2. didChangeDependencies - หลัง initState และเมื่อ InheritedWidget เปลี่ยน
  // ใช้สำหรับ: อ่านค่าจาก InheritedWidget (Theme, Locale, etc.)
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // final theme = Theme.of(context); // ปลอดภัยที่จะอ่านที่นี่
  }

  // 3. build - rebuild ทุกครั้งที่ setState หรือ parent rebuild
  // ต้องเป็น pure function - ไม่มี side effects
  @override
  Widget build(BuildContext context) {
    return Container();
  }

  // 4. didUpdateWidget - เมื่อ parent rebuild ส่ง widget ใหม่
  @override
  void didUpdateWidget(MyWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // เปรียบเทียบ oldWidget กับ widget ใหม่
    // if (oldWidget.data != widget.data) { refetch(); }
  }

  // 5. deactivate - ก่อนลบออกจาก tree
  @override
  void deactivate() {
    super.deactivate();
  }

  // 6. dispose - ทำลาย widget - cleanup ที่นี่!
  @override
  void dispose() {
    _controller.dispose();
    _subscription.cancel();
    _timer.cancel();
    super.dispose();  // ต้อง call super.dispose() เสมอ
  }
}
```

---

## ขั้นตอนที่ 52: StatefulWidget เชิงลึก

```dart
import 'package:flutter/material.dart';
import 'dart:async';

// ตัวอย่าง StatefulWidget ที่ถูกต้องและมีประสิทธิภาพ

// 1. Counter Widget แบบ basic
class CounterWidget extends StatefulWidget {
  // Widget constructor รับค่าเริ่มต้น
  final int initialCount;
  final int step;
  final ValueChanged<int>? onChanged;

  const CounterWidget({
    super.key,
    this.initialCount = 0,
    this.step = 1,
    this.onChanged,
  });

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  late int _count;

  @override
  void initState() {
    super.initState();
    // ใช้ widget.initialCount ใน initState
    _count = widget.initialCount;
  }

  @override
  void didUpdateWidget(CounterWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // ถ้า initialCount เปลี่ยน reset counter
    if (oldWidget.initialCount != widget.initialCount) {
      _count = widget.initialCount;
    }
  }

  void _increment() {
    setState(() {
      _count += widget.step;
      widget.onChanged?.call(_count);
    });
  }

  void _decrement() {
    setState(() {
      _count -= widget.step;
      widget.onChanged?.call(_count);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        IconButton(
          icon: const Icon(Icons.remove_circle_outline),
          onPressed: _decrement,
        ),
        Padding(
          padding: const EdgeInsets.symmetric(horizontal: 16),
          child: Text(
            '$_count',
            style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
        ),
        IconButton(
          icon: const Icon(Icons.add_circle_outline),
          onPressed: _increment,
        ),
      ],
    );
  }
}

// 2. Stopwatch Widget (ใช้ Timer)
class StopwatchWidget extends StatefulWidget {
  const StopwatchWidget({super.key});

  @override
  State<StopwatchWidget> createState() => _StopwatchWidgetState();
}

class _StopwatchWidgetState extends State<StopwatchWidget> {
  Timer? _timer;
  int _milliseconds = 0;
  bool _isRunning = false;
  final List<int> _laps = [];

  @override
  void dispose() {
    _timer?.cancel();  // ต้องยกเลิก timer ตอน dispose!
    super.dispose();
  }

  void _start() {
    setState(() => _isRunning = true);
    _timer = Timer.periodic(const Duration(milliseconds: 10), (_) {
      if (mounted) {
        setState(() => _milliseconds += 10);
      }
    });
  }

  void _stop() {
    _timer?.cancel();
    setState(() => _isRunning = false);
  }

  void _reset() {
    _timer?.cancel();
    setState(() {
      _milliseconds = 0;
      _isRunning = false;
      _laps.clear();
    });
  }

  void _lap() {
    setState(() => _laps.add(_milliseconds));
  }

  String _formatTime(int ms) {
    final min = (ms ~/ 60000).toString().padLeft(2, '0');
    final sec = ((ms % 60000) ~/ 1000).toString().padLeft(2, '0');
    final hundredths = ((ms % 1000) ~/ 10).toString().padLeft(2, '0');
    return '$min:$sec.$hundredths';
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Display
        Container(
          padding: const EdgeInsets.all(32),
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            color: Colors.black87,
            boxShadow: [
              BoxShadow(
                color: Colors.black.withOpacity(0.3),
                blurRadius: 20,
                offset: const Offset(0, 8),
              ),
            ],
          ),
          child: Text(
            _formatTime(_milliseconds),
            style: const TextStyle(
              color: Colors.white,
              fontSize: 32,
              fontFamily: 'monospace',
              fontWeight: FontWeight.w300,
            ),
          ),
        ),

        const SizedBox(height: 24),

        // Controls
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Reset / Lap
            ElevatedButton(
              onPressed: _isRunning ? _lap : _reset,
              style: ElevatedButton.styleFrom(
                shape: const CircleBorder(),
                padding: const EdgeInsets.all(20),
                backgroundColor: Colors.grey.shade800,
                foregroundColor: Colors.white,
              ),
              child: Text(_isRunning ? 'Lap' : 'Reset'),
            ),

            const SizedBox(width: 24),

            // Start / Stop
            ElevatedButton(
              onPressed: _isRunning ? _stop : _start,
              style: ElevatedButton.styleFrom(
                shape: const CircleBorder(),
                padding: const EdgeInsets.all(24),
                backgroundColor: _isRunning ? Colors.red : Colors.green,
                foregroundColor: Colors.white,
              ),
              child: Text(
                _isRunning ? 'Stop' : 'Start',
                style: const TextStyle(fontSize: 16),
              ),
            ),
          ],
        ),

        const SizedBox(height: 16),

        // Laps
        if (_laps.isNotEmpty) ...[
          const Divider(),
          SizedBox(
            height: 200,
            child: ListView.builder(
              itemCount: _laps.length,
              itemBuilder: (context, index) {
                final lapNum = _laps.length - index;
                final time = _laps[_laps.length - 1 - index];
                final prev = index < _laps.length - 1
                    ? _laps[_laps.length - 2 - index]
                    : 0;
                final diff = time - prev;

                return ListTile(
                  dense: true,
                  title: Text('Lap $lapNum'),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Text(
                        '+${_formatTime(diff)}',
                        style: const TextStyle(
                            color: Colors.grey, fontSize: 12),
                      ),
                      const SizedBox(width: 16),
                      Text(_formatTime(time)),
                    ],
                  ),
                );
              },
            ),
          ),
        ],
      ],
    );
  }
}

// 3. Toggle Widget ด้วย Animation
class AnimatedToggle extends StatefulWidget {
  final bool value;
  final ValueChanged<bool> onChanged;

  const AnimatedToggle({
    super.key,
    required this.value,
    required this.onChanged,
  });

  @override
  State<AnimatedToggle> createState() => _AnimatedToggleState();
}

class _AnimatedToggleState extends State<AnimatedToggle>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 300),
    );
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );
    // ตั้งค่าเริ่มต้นตาม widget.value
    if (widget.value) _controller.value = 1.0;
  }

  @override
  void didUpdateWidget(AnimatedToggle oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.value != widget.value) {
      if (widget.value) {
        _controller.forward();
      } else {
        _controller.reverse();
      }
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => widget.onChanged(!widget.value),
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, _) {
          return Container(
            width: 60,
            height: 32,
            decoration: BoxDecoration(
              color: Color.lerp(
                Colors.grey.shade300,
                Colors.green,
                _animation.value,
              ),
              borderRadius: BorderRadius.circular(16),
            ),
            padding: const EdgeInsets.all(4),
            child: Align(
              alignment: Alignment.lerp(
                Alignment.centerLeft,
                Alignment.centerRight,
                _animation.value,
              )!,
              child: Container(
                width: 24,
                height: 24,
                decoration: const BoxDecoration(
                  color: Colors.white,
                  shape: BoxShape.circle,
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black26,
                      blurRadius: 4,
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Demo Screen
class StatefulDemoScreen extends StatefulWidget {
  const StatefulDemoScreen({super.key});

  @override
  State<StatefulDemoScreen> createState() => _StatefulDemoScreenState();
}

class _StatefulDemoScreenState extends State<StatefulDemoScreen> {
  int _counterValue = 0;
  bool _toggle1 = false;
  bool _toggle2 = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('StatefulWidget Demo')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('Counter Widget:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            Center(
              child: CounterWidget(
                initialCount: 0,
                step: 5,
                onChanged: (value) =>
                    setState(() => _counterValue = value),
              ),
            ),
            Center(child: Text('ค่า: $_counterValue')),

            const SizedBox(height: 24),

            const Text('Stopwatch:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const Center(child: StopwatchWidget()),

            const SizedBox(height: 24),

            const Text('Animated Toggle:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                AnimatedToggle(
                  value: _toggle1,
                  onChanged: (v) => setState(() => _toggle1 = v),
                ),
                const SizedBox(width: 16),
                Text(_toggle1 ? 'เปิด' : 'ปิด'),
              ],
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                AnimatedToggle(
                  value: _toggle2,
                  onChanged: (v) => setState(() => _toggle2 = v),
                ),
                const SizedBox(width: 16),
                Text(_toggle2 ? 'เปิด' : 'ปิด'),
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

## ขั้นตอนที่ 53: Keys ใน Flutter

Keys ช่วย Flutter ระบุ widget ใน tree

```dart
import 'package:flutter/material.dart';

// ปัญหาที่ Keys แก้ไข:
// เมื่อเราสลับ StatefulWidget ใน list โดยไม่ใช้ Key
// Flutter อาจจับคู่ state กับ widget ผิด

class KeysDemoScreen extends StatefulWidget {
  const KeysDemoScreen({super.key});

  @override
  State<KeysDemoScreen> createState() => _KeysDemoScreenState();
}

class _KeysDemoScreenState extends State<KeysDemoScreen> {
  // ลองสลับระหว่าง Key vs No Key
  bool _useKeys = true;

  // รายการ items
  List<int> _items = [1, 2, 3, 4, 5];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Keys Demo')),
      body: Column(
        children: [
          // Toggle Keys
          SwitchListTile(
            title: const Text('ใช้ Keys'),
            subtitle: const Text('เปิด-ปิดเพื่อดูความแตกต่าง'),
            value: _useKeys,
            onChanged: (v) => setState(() => _useKeys = v),
          ),

          const Divider(),

          // Controls
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: () => setState(() {
                    _items = List.from(_items)..shuffle();
                  }),
                  child: const Text('สุ่ม'),
                ),
                ElevatedButton(
                  onPressed: () => setState(() {
                    if (_items.length > 1) _items.removeAt(0);
                  }),
                  child: const Text('ลบแรก'),
                ),
                ElevatedButton(
                  onPressed: () => setState(() {
                    _items.insert(0, _items.length + 1);
                  }),
                  child: const Text('เพิ่มต้น'),
                ),
              ],
            ),
          ),

          const Divider(),

          // List ของ ColorBox
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.all(8),
              itemCount: _items.length,
              itemBuilder: (context, index) {
                return _useKeys
                    ? ColorBox(
                        key: ValueKey(_items[index]),  // ใช้ Key!
                        id: _items[index],
                      )
                    : ColorBox(
                        // ไม่มี Key - Flutter อาจสับสน
                        id: _items[index],
                      );
              },
            ),
          ),
        ],
      ),
    );
  }
}

// Widget ที่มี internal state (สี random)
class ColorBox extends StatefulWidget {
  final int id;

  const ColorBox({super.key, required this.id});

  @override
  State<ColorBox> createState() => _ColorBoxState();
}

class _ColorBoxState extends State<ColorBox> {
  // State นี้ต้องติดกับ widget ที่ถูกต้อง!
  late Color _color;

  @override
  void initState() {
    super.initState();
    // สุ่มสีตอนสร้าง
    _color = Colors.primaries[widget.id % Colors.primaries.length];
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 4),
      child: ListTile(
        leading: Container(
          width: 40,
          height: 40,
          color: _color,
        ),
        title: Text('Item ${widget.id}'),
        subtitle: Text('State color: ${_color.value.toRadixString(16)}'),
        trailing: IconButton(
          icon: const Icon(Icons.refresh),
          onPressed: () => setState(() {
            // เปลี่ยนสีใหม่
            _color = Colors.primaries[
                (widget.id * 3 + 7) % Colors.primaries.length];
          }),
        ),
      ),
    );
  }
}

// ตัวอย่าง Key ชนิดต่างๆ
class KeyTypesDemo extends StatelessWidget {
  const KeyTypesDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Key Types')),
      body: const SingleChildScrollView(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 1. ValueKey - ใช้ value ใดๆ เป็น key
            Text('ValueKey:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            SizedBox(height: 8),

            // ใช้ String
            Text('ValueKey("myWidget")'),
            // ใช้ int
            Text('ValueKey(42)'),
            // ใช้ object
            Text('ValueKey(myObject)'),

            SizedBox(height: 16),

            // 2. ObjectKey - ใช้ object reference เป็น key
            Text('ObjectKey:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            SizedBox(height: 8),
            Text('ObjectKey(myUser) - ต่างถ้า reference ต่าง'),

            SizedBox(height: 16),

            // 3. UniqueKey - สร้าง key ใหม่ทุกครั้ง (force rebuild)
            Text('UniqueKey:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            SizedBox(height: 8),
            Text('UniqueKey() - rebuild ทุกครั้ง (ใช้ระวัง!)'),

            SizedBox(height: 16),

            // 4. GlobalKey - access state จากข้างนอก
            Text('GlobalKey:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            SizedBox(height: 8),
            Text('GlobalKey<FormState>() - เพื่อ validate Form'),
            Text('GlobalKey<ScaffoldState>() - เพื่อเปิด Drawer'),
          ],
        ),
      ),
    );
  }
}

// GlobalKey ตัวอย่าง
class GlobalKeyDemo extends StatefulWidget {
  const GlobalKeyDemo({super.key});

  @override
  State<GlobalKeyDemo> createState() => _GlobalKeyDemoState();
}

class _GlobalKeyDemoState extends State<GlobalKeyDemo> {
  // GlobalKey เพื่อ access Scaffold
  final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();

  // GlobalKey เพื่อ access Form
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: _scaffoldKey,
      appBar: AppBar(
        title: const Text('GlobalKey Demo'),
        actions: [
          // เปิด Drawer ผ่าน GlobalKey (แทนที่จะ swipe)
          IconButton(
            icon: const Icon(Icons.menu),
            onPressed: () => _scaffoldKey.currentState?.openDrawer(),
          ),
        ],
      ),
      drawer: const Drawer(
        child: Center(child: Text('Drawer ผ่าน GlobalKey')),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                decoration: const InputDecoration(labelText: 'ชื่อ'),
                validator: (value) =>
                    value?.isEmpty == true ? 'กรุณากรอกชื่อ' : null,
              ),
              TextFormField(
                decoration: const InputDecoration(labelText: 'อีเมล'),
                validator: (value) {
                  if (value?.isEmpty == true) return 'กรุณากรอกอีเมล';
                  if (!value!.contains('@')) return 'อีเมลไม่ถูกต้อง';
                  return null;
                },
              ),
              const SizedBox(height: 16),
              ElevatedButton(
                onPressed: () {
                  // Validate Form ผ่าน GlobalKey
                  if (_formKey.currentState?.validate() == true) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(content: Text('Form ถูกต้อง!')),
                    );
                  }
                },
                child: const Text('ตรวจสอบ Form'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 54: BuildContext

BuildContext คือ handle ไปยัง widget's position ใน widget tree

```dart
import 'package:flutter/material.dart';

class BuildContextDemo extends StatelessWidget {
  const BuildContextDemo({super.key});

  @override
  Widget build(BuildContext context) {
    // context ให้เข้าถึง:
    // 1. Theme
    final theme = Theme.of(context);
    final colorScheme = theme.colorScheme;

    // 2. MediaQuery
    final mediaQuery = MediaQuery.of(context);
    final screenSize = mediaQuery.size;

    // 3. Navigator
    // Navigator.of(context).push(...)

    // 4. Scaffold
    // Scaffold.of(context)

    // 5. Localizations
    // Localizations.of<AppLocalizations>(context, AppLocalizations)

    return Scaffold(
      appBar: AppBar(title: const Text('BuildContext Demo')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Theme.of(context)
            const Text('Theme.of(context):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            Container(
              padding: const EdgeInsets.all(16),
              color: colorScheme.primaryContainer,
              child: Text(
                'primaryContainer color',
                style: TextStyle(color: colorScheme.onPrimaryContainer),
              ),
            ),

            const SizedBox(height: 16),

            // MediaQuery
            const Text('MediaQuery.of(context):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            Text('Screen: ${screenSize.width.toInt()} x ${screenSize.height.toInt()}'),

            const SizedBox(height: 16),

            // context ใน callback - ระวัง!
            const Text('⚠️ ระวัง: async context:',
                style: TextStyle(fontWeight: FontWeight.bold, color: Colors.orange)),
            const SizedBox(height: 8),
            ElevatedButton(
              onPressed: () async {
                // ❌ อย่าใช้ context หลัง await โดยตรง
                // final result = await someAsync();
                // Navigator.of(context).pop(); // อาจ crash!

                // ✅ ตรวจสอบ mounted ก่อน
                // if (!context.mounted) return;
                // Navigator.of(context).pop();

                // หรือเก็บ navigator ไว้ก่อน await
                final navigator = Navigator.of(context);
                await Future.delayed(const Duration(seconds: 1));
                navigator.pushNamed('/home');
              },
              child: const Text('Async Navigation'),
            ),

            const SizedBox(height: 16),

            // findAncestorWidgetOfExactType
            const Text('Ancestor Widget:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            Builder(
              builder: (innerContext) {
                // Builder ให้ context ใหม่ใต้ Scaffold
                final scaffold = Scaffold.of(innerContext);
                return ElevatedButton(
                  onPressed: () {
                    scaffold.openDrawer();
                  },
                  child: const Text('เปิด Drawer ผ่าน context'),
                );
              },
            ),

            const SizedBox(height: 16),

            // context.size
            const Text('Widget Size:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            Builder(
              builder: (ctx) {
                return ElevatedButton(
                  onPressed: () {
                    // ขนาดของ widget ปัจจุบัน
                    final size = ctx.size;
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('Button size: $size')),
                    );
                  },
                  child: const Text('แสดงขนาด'),
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

## ขั้นตอนที่ 55: InheritedWidget

InheritedWidget ส่งข้อมูลลงไปยัง subtree โดยไม่ต้องส่งผ่าน constructor

```dart
import 'package:flutter/material.dart';

// 1. สร้าง InheritedWidget
class AppThemeData extends InheritedWidget {
  final Color primaryColor;
  final Color accentColor;
  final double fontSize;
  final bool isDarkMode;

  const AppThemeData({
    super.key,
    required this.primaryColor,
    required this.accentColor,
    required this.fontSize,
    required this.isDarkMode,
    required super.child,
  });

  // Static method เพื่อเข้าถึงจาก subtree
  static AppThemeData of(BuildContext context) {
    final result =
        context.dependOnInheritedWidgetOfExactType<AppThemeData>();
    assert(result != null, 'AppThemeData not found in context');
    return result!;
  }

  // ถ้า true = rebuild widget ที่ใช้ค่านี้
  @override
  bool updateShouldNotify(AppThemeData oldWidget) {
    return primaryColor != oldWidget.primaryColor ||
        accentColor != oldWidget.accentColor ||
        fontSize != oldWidget.fontSize ||
        isDarkMode != oldWidget.isDarkMode;
  }
}

// 2. Provider pattern ด้วย StatefulWidget + InheritedWidget
class AppThemeProvider extends StatefulWidget {
  final Widget child;

  const AppThemeProvider({super.key, required this.child});

  @override
  State<AppThemeProvider> createState() => _AppThemeProviderState();
}

class _AppThemeProviderState extends State<AppThemeProvider> {
  Color _primaryColor = Colors.blue;
  bool _isDarkMode = false;
  double _fontSize = 14.0;

  void updatePrimaryColor(Color color) {
    setState(() => _primaryColor = color);
  }

  void toggleDarkMode() {
    setState(() => _isDarkMode = !_isDarkMode);
  }

  void updateFontSize(double size) {
    setState(() => _fontSize = size);
  }

  @override
  Widget build(BuildContext context) {
    return AppThemeData(
      primaryColor: _primaryColor,
      accentColor: _primaryColor.withOpacity(0.5),
      fontSize: _fontSize,
      isDarkMode: _isDarkMode,
      child: widget.child,
    );
  }
}

// 3. Widget ที่ใช้ InheritedWidget
class ThemedCard extends StatelessWidget {
  final String title;
  final String body;

  const ThemedCard({super.key, required this.title, required this.body});

  @override
  Widget build(BuildContext context) {
    // อ่านค่าจาก InheritedWidget
    final themeData = AppThemeData.of(context);

    return Container(
      margin: const EdgeInsets.symmetric(vertical: 8),
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: themeData.isDarkMode
            ? Colors.grey.shade800
            : Colors.white,
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: themeData.primaryColor.withOpacity(0.3)),
        boxShadow: [
          if (!themeData.isDarkMode)
            BoxShadow(
              color: themeData.primaryColor.withOpacity(0.1),
              blurRadius: 8,
              offset: const Offset(0, 2),
            ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            title,
            style: TextStyle(
              fontSize: themeData.fontSize + 4,
              fontWeight: FontWeight.bold,
              color: themeData.primaryColor,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            body,
            style: TextStyle(
              fontSize: themeData.fontSize,
              color:
                  themeData.isDarkMode ? Colors.white70 : Colors.black87,
            ),
          ),
        ],
      ),
    );
  }
}

// 4. Demo Screen
class InheritedWidgetDemo extends StatelessWidget {
  const InheritedWidgetDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: AppThemeProvider(
        child: const _InheritedDemoContent(),
      ),
    );
  }
}

class _InheritedDemoContent extends StatefulWidget {
  const _InheritedDemoContent();

  @override
  State<_InheritedDemoContent> createState() =>
      _InheritedDemoContentState();
}

class _InheritedDemoContentState
    extends State<_InheritedDemoContent> {
  @override
  Widget build(BuildContext context) {
    final themeData = AppThemeData.of(context);

    return Scaffold(
      backgroundColor:
          themeData.isDarkMode ? Colors.grey.shade900 : Colors.grey.shade100,
      appBar: AppBar(
        backgroundColor: themeData.primaryColor,
        foregroundColor: Colors.white,
        title: const Text('InheritedWidget Demo'),
        actions: [
          IconButton(
            icon: Icon(
                themeData.isDarkMode ? Icons.light_mode : Icons.dark_mode),
            onPressed: () {
              // หา provider ใน tree
              final provider = context
                  .findAncestorStateOfType<_AppThemeProviderState>();
              provider?.toggleDarkMode();
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Color picker
            Wrap(
              spacing: 8,
              children: [
                Colors.blue,
                Colors.red,
                Colors.green,
                Colors.purple,
                Colors.orange,
              ].map((color) {
                return GestureDetector(
                  onTap: () {
                    final provider = context
                        .findAncestorStateOfType<_AppThemeProviderState>();
                    provider?.updatePrimaryColor(color);
                  },
                  child: Container(
                    width: 40,
                    height: 40,
                    decoration: BoxDecoration(
                      color: color,
                      shape: BoxShape.circle,
                      border: Border.all(
                        color: themeData.primaryColor == color
                            ? Colors.white
                            : Colors.transparent,
                        width: 3,
                      ),
                    ),
                  ),
                );
              }).toList(),
            ),

            const SizedBox(height: 16),

            // Font size slider
            Row(
              children: [
                const Text('Font Size: '),
                Expanded(
                  child: Slider(
                    value: themeData.fontSize,
                    min: 10,
                    max: 24,
                    divisions: 14,
                    label: themeData.fontSize.toStringAsFixed(0),
                    onChanged: (value) {
                      final provider = context
                          .findAncestorStateOfType<_AppThemeProviderState>();
                      provider?.updateFontSize(value);
                    },
                  ),
                ),
                Text(themeData.fontSize.toStringAsFixed(0)),
              ],
            ),

            const SizedBox(height: 16),

            // Cards ที่ใช้ theme
            const ThemedCard(
              title: 'บทความที่ 1',
              body: 'นี่คือเนื้อหาของบทความที่ใช้ theme จาก InheritedWidget',
            ),
            const ThemedCard(
              title: 'บทความที่ 2',
              body: 'เนื้อหาสอง - สีและขนาดจะเปลี่ยนตาม theme',
            ),
            const ThemedCard(
              title: 'บทความที่ 3',
              body: 'เนื้อหาสาม - ทั้ง dark mode และ light mode รองรับ',
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 56: Widget Composition

การสร้าง widget ที่ซับซ้อนจาก widget ย่อยๆ

```dart
import 'package:flutter/material.dart';

// ========================
// Atomic Widgets (เล็กสุด)
// ========================

// Avatar Widget
class UserAvatar extends StatelessWidget {
  final String? imageUrl;
  final String name;
  final double radius;
  final bool showBadge;

  const UserAvatar({
    super.key,
    this.imageUrl,
    required this.name,
    this.radius = 24,
    this.showBadge = false,
  });

  @override
  Widget build(BuildContext context) {
    return Stack(
      clipBehavior: Clip.none,
      children: [
        CircleAvatar(
          radius: radius,
          backgroundImage:
              imageUrl != null ? NetworkImage(imageUrl!) : null,
          backgroundColor: Colors.blue.shade200,
          child: imageUrl == null
              ? Text(
                  name.isNotEmpty ? name[0].toUpperCase() : '?',
                  style: TextStyle(
                    fontSize: radius * 0.8,
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                )
              : null,
        ),
        if (showBadge)
          Positioned(
            bottom: 0,
            right: 0,
            child: Container(
              width: radius * 0.5,
              height: radius * 0.5,
              decoration: BoxDecoration(
                color: Colors.green,
                shape: BoxShape.circle,
                border: Border.all(color: Colors.white, width: 1.5),
              ),
            ),
          ),
      ],
    );
  }
}

// Rating Widget
class StarRating extends StatelessWidget {
  final double rating;
  final int maxStars;
  final double size;
  final Color? color;

  const StarRating({
    super.key,
    required this.rating,
    this.maxStars = 5,
    this.size = 16,
    this.color,
  });

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: List.generate(maxStars, (index) {
        final starValue = index + 1;
        IconData icon;
        if (rating >= starValue) {
          icon = Icons.star;
        } else if (rating >= starValue - 0.5) {
          icon = Icons.star_half;
        } else {
          icon = Icons.star_border;
        }
        return Icon(
          icon,
          size: size,
          color: color ?? Colors.amber,
        );
      }),
    );
  }
}

// Price Widget
class PriceTag extends StatelessWidget {
  final double price;
  final double? originalPrice;
  final String currency;

  const PriceTag({
    super.key,
    required this.price,
    this.originalPrice,
    this.currency = '฿',
  });

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      crossAxisAlignment: CrossAxisAlignment.baseline,
      textBaseline: TextBaseline.alphabetic,
      children: [
        Text(
          '$currency${price.toStringAsFixed(0)}',
          style: const TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
            color: Colors.deepOrange,
          ),
        ),
        if (originalPrice != null) ...[
          const SizedBox(width: 8),
          Text(
            '$currency${originalPrice!.toStringAsFixed(0)}',
            style: const TextStyle(
              fontSize: 13,
              color: Colors.grey,
              decoration: TextDecoration.lineThrough,
            ),
          ),
        ],
      ],
    );
  }
}

// Badge Widget
class Badge extends StatelessWidget {
  final String text;
  final Color color;

  const Badge({super.key, required this.text, required this.color});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 3),
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Text(
        text,
        style: const TextStyle(
          color: Colors.white,
          fontSize: 11,
          fontWeight: FontWeight.bold,
        ),
      ),
    );
  }
}

// ========================
// Molecule Widgets (รวม atomic)
// ========================

// Product Card
class ProductCard extends StatelessWidget {
  final String imageUrl;
  final String name;
  final double price;
  final double? originalPrice;
  final double rating;
  final int reviewCount;
  final bool isFavorite;
  final String? badge;
  final VoidCallback? onTap;
  final VoidCallback? onFavorite;

  const ProductCard({
    super.key,
    required this.imageUrl,
    required this.name,
    required this.price,
    this.originalPrice,
    this.rating = 0,
    this.reviewCount = 0,
    this.isFavorite = false,
    this.badge,
    this.onTap,
    this.onFavorite,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Card(
        clipBehavior: Clip.antiAlias,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Image + Badge + Favorite
            Stack(
              children: [
                AspectRatio(
                  aspectRatio: 1.0,
                  child: Image.network(
                    imageUrl,
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) => Container(
                      color: Colors.grey.shade200,
                      child:
                          const Icon(Icons.image, size: 48, color: Colors.grey),
                    ),
                  ),
                ),
                if (badge != null)
                  Positioned(
                    top: 8,
                    left: 8,
                    child: Badge(text: badge!, color: Colors.red),
                  ),
                Positioned(
                  top: 4,
                  right: 4,
                  child: IconButton(
                    icon: Icon(
                      isFavorite ? Icons.favorite : Icons.favorite_border,
                      color: isFavorite ? Colors.red : Colors.white,
                    ),
                    onPressed: onFavorite,
                    style: IconButton.styleFrom(
                      backgroundColor: Colors.black26,
                    ),
                  ),
                ),
              ],
            ),

            // Info
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    name,
                    style: const TextStyle(
                      fontWeight: FontWeight.w600,
                      fontSize: 14,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 6),
                  PriceTag(price: price, originalPrice: originalPrice),
                  const SizedBox(height: 4),
                  Row(
                    children: [
                      StarRating(rating: rating, size: 14),
                      const SizedBox(width: 4),
                      Text(
                        '($reviewCount)',
                        style: const TextStyle(
                          fontSize: 11,
                          color: Colors.grey,
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// Comment Widget
class CommentCard extends StatelessWidget {
  final String authorName;
  final String? authorAvatar;
  final String content;
  final String timeAgo;
  final int likes;
  final bool isLiked;
  final VoidCallback? onLike;

  const CommentCard({
    super.key,
    required this.authorName,
    this.authorAvatar,
    required this.content,
    required this.timeAgo,
    this.likes = 0,
    this.isLiked = false,
    this.onLike,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          UserAvatar(
            imageUrl: authorAvatar,
            name: authorName,
            radius: 20,
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Row(
                  children: [
                    Text(
                      authorName,
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const Spacer(),
                    Text(
                      timeAgo,
                      style: const TextStyle(
                          fontSize: 12, color: Colors.grey),
                    ),
                  ],
                ),
                const SizedBox(height: 4),
                Text(content),
                const SizedBox(height: 8),
                GestureDetector(
                  onTap: onLike,
                  child: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Icon(
                        isLiked
                            ? Icons.favorite
                            : Icons.favorite_border,
                        size: 16,
                        color: isLiked ? Colors.red : Colors.grey,
                      ),
                      const SizedBox(width: 4),
                      Text(
                        '$likes',
                        style: const TextStyle(
                            fontSize: 13, color: Colors.grey),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 57: Separation of Concerns

แยก Logic, UI, และ State ออกจากกัน

```dart
import 'package:flutter/material.dart';

// ========================
// Data Layer (Model)
// ========================
class Todo {
  final String id;
  final String title;
  final String description;
  bool isCompleted;
  final DateTime createdAt;

  Todo({
    required this.id,
    required this.title,
    this.description = '',
    this.isCompleted = false,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();

  Todo copyWith({
    String? title,
    String? description,
    bool? isCompleted,
  }) {
    return Todo(
      id: id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt,
    );
  }
}

// ========================
// Business Logic Layer (Controller/ViewModel)
// ========================
class TodoController {
  final List<Todo> _todos = [];

  List<Todo> get todos => List.unmodifiable(_todos);

  List<Todo> get completedTodos =>
      _todos.where((t) => t.isCompleted).toList();

  List<Todo> get activeTodos =>
      _todos.where((t) => !t.isCompleted).toList();

  void addTodo(String title, {String description = ''}) {
    _todos.add(Todo(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      title: title,
      description: description,
    ));
  }

  void toggleTodo(String id) {
    final index = _todos.indexWhere((t) => t.id == id);
    if (index != -1) {
      _todos[index].isCompleted = !_todos[index].isCompleted;
    }
  }

  void deleteTodo(String id) {
    _todos.removeWhere((t) => t.id == id);
  }

  void clearCompleted() {
    _todos.removeWhere((t) => t.isCompleted);
  }
}

// ========================
// UI Layer (View)
// ========================
class TodoApp extends StatefulWidget {
  const TodoApp({super.key});

  @override
  State<TodoApp> createState() => _TodoAppState();
}

class _TodoAppState extends State<TodoApp> {
  // State อยู่ที่ top
  final TodoController _controller = TodoController();
  final TextEditingController _textController = TextEditingController();
  String _filter = 'all'; // all, active, completed

  @override
  void initState() {
    super.initState();
    // เพิ่มข้อมูลตัวอย่าง
    _controller.addTodo('เรียน Flutter', description: 'ศึกษา widget lifecycle');
    _controller.addTodo('สร้าง app แรก', description: 'counter app');
    _controller.addTodo('Deploy ขึ้น store');
  }

  @override
  void dispose() {
    _textController.dispose();
    super.dispose();
  }

  List<Todo> get _filteredTodos {
    switch (_filter) {
      case 'active':
        return _controller.activeTodos;
      case 'completed':
        return _controller.completedTodos;
      default:
        return _controller.todos;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo App'),
        actions: [
          if (_controller.completedTodos.isNotEmpty)
            TextButton(
              onPressed: () => setState(() => _controller.clearCompleted()),
              child: const Text('ล้าง completed'),
            ),
        ],
      ),
      body: Column(
        children: [
          // Input
          _TodoInput(
            controller: _textController,
            onSubmit: (title) {
              setState(() {
                _controller.addTodo(title);
                _textController.clear();
              });
            },
          ),

          // Filter
          _TodoFilter(
            currentFilter: _filter,
            onFilterChanged: (filter) =>
                setState(() => _filter = filter),
            totalCount: _controller.todos.length,
            activeCount: _controller.activeTodos.length,
            completedCount: _controller.completedTodos.length,
          ),

          // List
          Expanded(
            child: _filteredTodos.isEmpty
                ? const Center(
                    child: Text('ไม่มีรายการ',
                        style: TextStyle(color: Colors.grey)))
                : ListView.builder(
                    itemCount: _filteredTodos.length,
                    itemBuilder: (context, index) {
                      final todo = _filteredTodos[index];
                      return _TodoItem(
                        todo: todo,
                        onToggle: () =>
                            setState(() => _controller.toggleTodo(todo.id)),
                        onDelete: () =>
                            setState(() => _controller.deleteTodo(todo.id)),
                      );
                    },
                  ),
          ),
        ],
      ),
    );
  }
}

// Sub-widgets แยกออกมา (presentation only)
class _TodoInput extends StatelessWidget {
  final TextEditingController controller;
  final ValueChanged<String> onSubmit;

  const _TodoInput({required this.controller, required this.onSubmit});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              controller: controller,
              decoration: InputDecoration(
                hintText: 'เพิ่มรายการใหม่...',
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
                contentPadding: const EdgeInsets.symmetric(
                    horizontal: 16, vertical: 12),
              ),
              onSubmitted: (value) {
                if (value.trim().isNotEmpty) {
                  onSubmit(value.trim());
                }
              },
            ),
          ),
          const SizedBox(width: 8),
          IconButton.filled(
            icon: const Icon(Icons.add),
            onPressed: () {
              if (controller.text.trim().isNotEmpty) {
                onSubmit(controller.text.trim());
              }
            },
          ),
        ],
      ),
    );
  }
}

class _TodoFilter extends StatelessWidget {
  final String currentFilter;
  final ValueChanged<String> onFilterChanged;
  final int totalCount;
  final int activeCount;
  final int completedCount;

  const _TodoFilter({
    required this.currentFilter,
    required this.onFilterChanged,
    required this.totalCount,
    required this.activeCount,
    required this.completedCount,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: [
          _FilterChip('ทั้งหมด ($totalCount)', 'all', currentFilter, onFilterChanged),
          const SizedBox(width: 8),
          _FilterChip('ค้างอยู่ ($activeCount)', 'active', currentFilter, onFilterChanged),
          const SizedBox(width: 8),
          _FilterChip('เสร็จ ($completedCount)', 'completed', currentFilter, onFilterChanged),
        ],
      ),
    );
  }
}

class _FilterChip extends StatelessWidget {
  final String label;
  final String value;
  final String current;
  final ValueChanged<String> onChanged;

  const _FilterChip(this.label, this.value, this.current, this.onChanged);

  @override
  Widget build(BuildContext context) {
    final isSelected = current == value;
    return GestureDetector(
      onTap: () => onChanged(value),
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
        decoration: BoxDecoration(
          color: isSelected ? Colors.blue : Colors.grey.shade200,
          borderRadius: BorderRadius.circular(16),
        ),
        child: Text(
          label,
          style: TextStyle(
            color: isSelected ? Colors.white : Colors.black,
            fontSize: 12,
          ),
        ),
      ),
    );
  }
}

class _TodoItem extends StatelessWidget {
  final Todo todo;
  final VoidCallback onToggle;
  final VoidCallback onDelete;

  const _TodoItem({
    required this.todo,
    required this.onToggle,
    required this.onDelete,
  });

  @override
  Widget build(BuildContext context) {
    return Dismissible(
      key: Key(todo.id),
      background: Container(
        color: Colors.red,
        padding: const EdgeInsets.symmetric(horizontal: 16),
        alignment: Alignment.centerRight,
        child: const Icon(Icons.delete, color: Colors.white),
      ),
      direction: DismissDirection.endToStart,
      onDismissed: (_) => onDelete(),
      child: ListTile(
        leading: Checkbox(
          value: todo.isCompleted,
          onChanged: (_) => onToggle(),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(4),
          ),
        ),
        title: Text(
          todo.title,
          style: TextStyle(
            decoration:
                todo.isCompleted ? TextDecoration.lineThrough : null,
            color: todo.isCompleted ? Colors.grey : null,
          ),
        ),
        subtitle: todo.description.isNotEmpty
            ? Text(
                todo.description,
                style: const TextStyle(color: Colors.grey),
              )
            : null,
        trailing: IconButton(
          icon: const Icon(Icons.delete_outline, color: Colors.grey),
          onPressed: onDelete,
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 58: Pure Widgets

Pure widget คือ widget ที่ output ขึ้นอยู่กับ input เท่านั้น (ไม่มี side effects)

```dart
import 'package:flutter/material.dart';

// ========================
// Pure Widgets - deterministic output
// ========================

// ✅ Pure Widget - output ขึ้นกับ input เท่านั้น
class TemperatureDisplay extends StatelessWidget {
  final double celsius;

  const TemperatureDisplay({super.key, required this.celsius});

  double get fahrenheit => celsius * 9 / 5 + 32;

  Color get temperatureColor {
    if (celsius < 10) return Colors.blue;
    if (celsius < 20) return Colors.green;
    if (celsius < 30) return Colors.orange;
    return Colors.red;
  }

  String get temperatureLabel {
    if (celsius < 10) return 'หนาว';
    if (celsius < 20) return 'เย็น';
    if (celsius < 30) return 'อบอุ่น';
    return 'ร้อน';
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: [
            temperatureColor.withOpacity(0.1),
            temperatureColor.withOpacity(0.3),
          ],
        ),
        borderRadius: BorderRadius.circular(16),
        border: Border.all(color: temperatureColor.withOpacity(0.5)),
      ),
      child: Column(
        children: [
          Icon(Icons.thermostat, color: temperatureColor, size: 48),
          Text(
            '${celsius.toStringAsFixed(1)}°C',
            style: TextStyle(
              fontSize: 36,
              fontWeight: FontWeight.bold,
              color: temperatureColor,
            ),
          ),
          Text(
            '${fahrenheit.toStringAsFixed(1)}°F',
            style: TextStyle(fontSize: 16, color: temperatureColor),
          ),
          Chip(
            label: Text(temperatureLabel),
            backgroundColor: temperatureColor.withOpacity(0.2),
            labelStyle: TextStyle(color: temperatureColor),
          ),
        ],
      ),
    );
  }
}

// Pure widget สำหรับ progress
class CircularProgress extends StatelessWidget {
  final double progress; // 0.0 - 1.0
  final double size;
  final Color color;
  final String? label;

  const CircularProgress({
    super.key,
    required this.progress,
    this.size = 80,
    this.color = Colors.blue,
    this.label,
  });

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: size,
      height: size,
      child: Stack(
        alignment: Alignment.center,
        children: [
          CircularProgressIndicator(
            value: progress,
            strokeWidth: size * 0.1,
            backgroundColor: color.withOpacity(0.2),
            valueColor: AlwaysStoppedAnimation<Color>(color),
          ),
          Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(
                '${(progress * 100).toInt()}%',
                style: TextStyle(
                  fontSize: size * 0.22,
                  fontWeight: FontWeight.bold,
                  color: color,
                ),
              ),
              if (label != null)
                Text(
                  label!,
                  style: TextStyle(
                    fontSize: size * 0.13,
                    color: Colors.grey,
                  ),
                ),
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 59: Performance Tips

เทคนิคเพิ่มประสิทธิภาพ widget

```dart
import 'package:flutter/material.dart';

// 1. const constructor - Flutter reuse widget แทนที่จะสร้างใหม่
// ✅ ดี
const Text('Hello'); // Flutter cache และ reuse
// ❌ ไม่ดี (แต่ไม่ผิด)
Text('Hello'); // สร้างใหม่ทุกครั้ง

// 2. แยก StatefulWidget ให้เล็กที่สุด
// ❌ ไม่ดี - rebuild ทั้งหน้าเมื่อ counter เปลี่ยน
class BigScreen extends StatefulWidget {
  const BigScreen({super.key});

  @override
  State<BigScreen> createState() => _BigScreenState();
}

class _BigScreenState extends State<BigScreen> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ❌ ทุก widget ใน Column rebuild เมื่อ _count เปลี่ยน
        const ExpensiveStaticWidget(),
        const AnotherExpensiveWidget(),
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('+'),
        ),
      ],
    );
  }
}

// ✅ ดีกว่า - แยก counter ออกมา
class OptimizedScreen extends StatelessWidget {
  const OptimizedScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        ExpensiveStaticWidget(),  // ไม่ rebuild!
        AnotherExpensiveWidget(), // ไม่ rebuild!
        _CounterSection(),       // เฉพาะส่วนนี้ rebuild
      ],
    );
  }
}

class _CounterSection extends StatefulWidget {
  const _CounterSection();

  @override
  State<_CounterSection> createState() => _CounterSectionState();
}

class _CounterSectionState extends State<_CounterSection> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('+'),
        ),
      ],
    );
  }
}

class ExpensiveStaticWidget extends StatelessWidget {
  const ExpensiveStaticWidget({super.key});
  @override
  Widget build(BuildContext context) => const Text('Expensive');
}

class AnotherExpensiveWidget extends StatelessWidget {
  const AnotherExpensiveWidget({super.key});
  @override
  Widget build(BuildContext context) => const Text('Another');
}

// 3. repaintBoundary - แยก repaint zone
class RepaintBoundaryDemo extends StatelessWidget {
  const RepaintBoundaryDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        RepaintBoundary(
          // ส่วนนี้จะ repaint แยกจากส่วนอื่น
          child: AnimatedWidget(),
        ),
        const StaticContent(),
      ],
    );
  }
}

class AnimatedWidget extends StatefulWidget {
  @override
  State<AnimatedWidget> createState() => _AnimatedWidgetState();
}

class _AnimatedWidgetState extends State<AnimatedWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (_, __) => Transform.rotate(
        angle: _controller.value * 2 * 3.14159,
        child: const Icon(Icons.refresh, size: 48),
      ),
    );
  }
}

class StaticContent extends StatelessWidget {
  const StaticContent({super.key});

  @override
  Widget build(BuildContext context) {
    return const Text('ส่วนนี้ไม่ repaint');
  }
}
```

---

## ขั้นตอนที่ 60: Workshop - Counter App (Advanced)

สร้าง Counter App ที่แสดง concept ทั้งหมดที่เรียนมา

```dart
import 'package:flutter/material.dart';
import 'dart:async';

void main() {
  runApp(const CounterAdvancedApp());
}

class CounterAdvancedApp extends StatelessWidget {
  const CounterAdvancedApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter Advanced',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const CounterHomePage(),
    );
  }
}

// Model
class CounterHistory {
  final int value;
  final String action;
  final DateTime timestamp;

  CounterHistory({
    required this.value,
    required this.action,
  }) : timestamp = DateTime.now();
}

// State management (simplified)
class CounterState {
  final int count;
  final int step;
  final List<CounterHistory> history;
  final bool autoIncrement;

  const CounterState({
    this.count = 0,
    this.step = 1,
    this.history = const [],
    this.autoIncrement = false,
  });

  CounterState copyWith({
    int? count,
    int? step,
    List<CounterHistory>? history,
    bool? autoIncrement,
  }) {
    return CounterState(
      count: count ?? this.count,
      step: step ?? this.step,
      history: history ?? this.history,
      autoIncrement: autoIncrement ?? this.autoIncrement,
    );
  }
}

class CounterHomePage extends StatefulWidget {
  const CounterHomePage({super.key});

  @override
  State<CounterHomePage> createState() => _CounterHomePageState();
}

class _CounterHomePageState extends State<CounterHomePage>
    with TickerProviderStateMixin {
  CounterState _state = const CounterState();
  Timer? _autoTimer;
  late AnimationController _scaleController;
  late Animation<double> _scaleAnimation;

  @override
  void initState() {
    super.initState();
    _scaleController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 200),
    );
    _scaleAnimation = Tween<double>(begin: 1.0, end: 1.3).animate(
      CurvedAnimation(parent: _scaleController, curve: Curves.elasticOut),
    );

    // ค่าเริ่มต้น
    _addToHistory(0, 'เริ่มต้น');
  }

  @override
  void dispose() {
    _autoTimer?.cancel();
    _scaleController.dispose();
    super.dispose();
  }

  void _addToHistory(int value, String action) {
    final newHistory =
        List<CounterHistory>.from(_state.history)
          ..add(CounterHistory(value: value, action: action));
    // เก็บแค่ 20 รายการล่าสุด
    if (newHistory.length > 20) newHistory.removeAt(0);
    setState(() {
      _state = _state.copyWith(history: newHistory);
    });
  }

  void _increment() {
    _scaleController.forward().then((_) => _scaleController.reverse());
    final newCount = _state.count + _state.step;
    setState(() {
      _state = _state.copyWith(count: newCount);
    });
    _addToHistory(newCount, '+${_state.step}');
  }

  void _decrement() {
    _scaleController.forward().then((_) => _scaleController.reverse());
    final newCount = _state.count - _state.step;
    setState(() {
      _state = _state.copyWith(count: newCount);
    });
    _addToHistory(newCount, '-${_state.step}');
  }

  void _reset() {
    setState(() {
      _state = _state.copyWith(count: 0);
    });
    _addToHistory(0, 'reset');
  }

  void _toggleAuto() {
    if (_state.autoIncrement) {
      _autoTimer?.cancel();
      setState(() {
        _state = _state.copyWith(autoIncrement: false);
      });
    } else {
      setState(() {
        _state = _state.copyWith(autoIncrement: true);
      });
      _autoTimer = Timer.periodic(const Duration(milliseconds: 500), (_) {
        if (mounted) _increment();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    final colorScheme = Theme.of(context).colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Advanced Counter'),
        actions: [
          IconButton(
            icon: const Icon(Icons.history),
            onPressed: _showHistory,
            tooltip: 'ประวัติ',
          ),
          IconButton(
            icon: const Icon(Icons.tune),
            onPressed: _showSettings,
            tooltip: 'ตั้งค่า',
          ),
        ],
      ),
      body: Column(
        children: [
          // Main Counter Display
          Expanded(
            flex: 3,
            child: Container(
              width: double.infinity,
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                  colors: [
                    colorScheme.primaryContainer,
                    colorScheme.secondaryContainer,
                  ],
                ),
              ),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  // Auto indicator
                  if (_state.autoIncrement)
                    const Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        SizedBox(
                          width: 12,
                          height: 12,
                          child: CircularProgressIndicator(strokeWidth: 2),
                        ),
                        SizedBox(width: 8),
                        Text('Auto Increment'),
                      ],
                    ),

                  const SizedBox(height: 16),

                  // Counter value with animation
                  ScaleTransition(
                    scale: _scaleAnimation,
                    child: Text(
                      '${_state.count}',
                      style: TextStyle(
                        fontSize: 80,
                        fontWeight: FontWeight.bold,
                        color: colorScheme.onPrimaryContainer,
                      ),
                    ),
                  ),

                  Text(
                    'Step: ${_state.step}',
                    style: TextStyle(
                      color: colorScheme.onPrimaryContainer.withOpacity(0.7),
                    ),
                  ),

                  const SizedBox(height: 32),

                  // Controls
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      // Decrement
                      FloatingActionButton(
                        heroTag: 'dec',
                        onPressed: _decrement,
                        backgroundColor: Colors.red.shade400,
                        child: const Icon(Icons.remove, color: Colors.white),
                      ),
                      const SizedBox(width: 16),

                      // Reset
                      FloatingActionButton.small(
                        heroTag: 'reset',
                        onPressed: _reset,
                        backgroundColor: Colors.grey,
                        child: const Icon(Icons.refresh, color: Colors.white),
                      ),
                      const SizedBox(width: 16),

                      // Auto Toggle
                      FloatingActionButton.small(
                        heroTag: 'auto',
                        onPressed: _toggleAuto,
                        backgroundColor:
                            _state.autoIncrement ? Colors.orange : Colors.blue,
                        child: Icon(
                          _state.autoIncrement
                              ? Icons.stop
                              : Icons.play_arrow,
                          color: Colors.white,
                        ),
                      ),
                      const SizedBox(width: 16),

                      // Increment
                      FloatingActionButton(
                        heroTag: 'inc',
                        onPressed: _increment,
                        backgroundColor: Colors.green.shade400,
                        child: const Icon(Icons.add, color: Colors.white),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),

          // Stats
          Expanded(
            flex: 1,
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  _StatItem(
                    label: 'ค่าปัจจุบัน',
                    value: '${_state.count}',
                    icon: Icons.numbers,
                    color: Colors.blue,
                  ),
                  _StatItem(
                    label: 'step',
                    value: '${_state.step}',
                    icon: Icons.stairs,
                    color: Colors.purple,
                  ),
                  _StatItem(
                    label: 'บันทึก',
                    value: '${_state.history.length}',
                    icon: Icons.history,
                    color: Colors.orange,
                  ),
                  _StatItem(
                    label: 'สถานะ',
                    value: _state.autoIncrement ? 'Auto' : 'Manual',
                    icon: Icons.settings,
                    color: _state.autoIncrement ? Colors.green : Colors.grey,
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  void _showHistory() {
    showModalBottomSheet(
      context: context,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => Column(
        children: [
          const SizedBox(height: 12),
          Container(
            width: 40,
            height: 4,
            decoration: BoxDecoration(
              color: Colors.grey.shade300,
              borderRadius: BorderRadius.circular(2),
            ),
          ),
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('ประวัติ',
                style: TextStyle(
                    fontSize: 18, fontWeight: FontWeight.bold)),
          ),
          Expanded(
            child: ListView.builder(
              reverse: true,
              itemCount: _state.history.length,
              itemBuilder: (context, index) {
                final revIndex =
                    _state.history.length - 1 - index;
                final h = _state.history[revIndex];
                return ListTile(
                  dense: true,
                  leading: CircleAvatar(
                    radius: 16,
                    backgroundColor: h.action.startsWith('+')
                        ? Colors.green.shade100
                        : h.action.startsWith('-')
                            ? Colors.red.shade100
                            : Colors.grey.shade200,
                    child: Text(
                      '${h.value}',
                      style: const TextStyle(fontSize: 11),
                    ),
                  ),
                  title: Text('${h.action} → ${h.value}'),
                  trailing: Text(
                    '${h.timestamp.hour}:${h.timestamp.minute.toString().padLeft(2, '0')}:${h.timestamp.second.toString().padLeft(2, '0')}',
                    style: const TextStyle(
                        fontSize: 11, color: Colors.grey),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }

  void _showSettings() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('ตั้งค่า Step'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [1, 5, 10, 25, 100].map((step) {
            return RadioListTile<int>(
              title: Text('Step: $step'),
              value: step,
              groupValue: _state.step,
              onChanged: (value) {
                if (value != null) {
                  setState(() {
                    _state = _state.copyWith(step: value);
                  });
                  Navigator.pop(context);
                }
              },
            );
          }).toList(),
        ),
      ),
    );
  }
}

class _StatItem extends StatelessWidget {
  final String label;
  final String value;
  final IconData icon;
  final Color color;

  const _StatItem({
    required this.label,
    required this.value,
    required this.icon,
    required this.color,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Icon(icon, color: color, size: 20),
        const SizedBox(height: 4),
        Text(
          value,
          style: TextStyle(
            fontWeight: FontWeight.bold,
            color: color,
          ),
        ),
        Text(
          label,
          style: const TextStyle(fontSize: 11, color: Colors.grey),
        ),
      ],
    );
  }
}
```

---

## สรุป (Summary)

ใน Part 06 เราได้เรียนรู้:

| Concept | สาระสำคัญ |
|---------|-----------|
| Widget Lifecycle | initState → didChangeDependencies → build → didUpdateWidget → dispose |
| StatefulWidget | State แยกจาก Widget, setState triggers rebuild |
| Keys | ช่วย Flutter จับคู่ Widget กับ State ที่ถูกต้อง |
| BuildContext | Handle ไปยัง position ใน widget tree |
| InheritedWidget | ส่งข้อมูลลงไปทั้ง subtree โดยไม่ผ่าน constructor |
| Widget Composition | สร้าง widget ซับซ้อนจาก widget ย่อยๆ |
| Separation of Concerns | แยก Model, Controller, View |
| Pure Widgets | Output ขึ้นกับ input เท่านั้น |

---

## แบบฝึกหัด (Exercises)

1. **Lifecycle Logger**: สร้าง widget ที่แสดง log ของทุก lifecycle method ใน real time

2. **Todo App**: พัฒนา Todo App จากตัวอย่าง เพิ่ม: edit, priority (high/medium/low), due date

3. **Theme Switcher**: สร้าง InheritedWidget ที่เก็บ theme settings และ widget ต่างๆ ใช้ค่าจาก inherited

4. **Stopwatch**: สร้าง stopwatch ที่มี start/stop/reset/lap ใช้ Timer และ dispose ถูกต้อง

5. **Keys Demo**: สร้าง demo ที่แสดงให้เห็นความแตกต่างระหว่างใช้/ไม่ใช้ Key เมื่อ reorder list

---

[← Part 05: Layout Widgets](part_05.md) | [Part 07: Material Design & Theming →](part_07.md)

# Part 13: Gestures & Touch Handling
## ขั้นตอนที่ 121-130

---

## สารบัญ
1. [GestureDetector พื้นฐาน](#ขั้นตอนที่-121-gesturedetector-พื้นฐาน)
2. [Swipe และ Pan Gestures](#ขั้นตอนที่-122-swipe-และ-pan-gestures)
3. [Pinch to Zoom](#ขั้นตอนที่-123-pinch-to-zoom)
4. [InkWell และ InkResponse](#ขั้นตอนที่-124-inkwell-และ-inkresponse)
5. [Draggable และ DragTarget](#ขั้นตอนที่-125-draggable-และ-dragtarget)
6. [ReorderableListView](#ขั้นตอนที่-126-reorderablelistview)
7. [Dismissible](#ขั้นตอนที่-127-dismissible)
8. [Listener และ PointerEvent](#ขั้นตอนที่-128-listener-และ-pointerevent)
9. [Custom Gestures](#ขั้นตอนที่-129-custom-gestures)
10. [Multi-touch Gestures](#ขั้นตอนที่-130-multi-touch-gestures)
11. [Workshop: Drag & Drop Kanban Board](#workshop-drag--drop-kanban-board)

---

## ขั้นตอนที่ 121: GestureDetector พื้นฐาน

`GestureDetector` คือ Widget ที่จัดการ touch event ต่างๆ ทั้ง tap, double tap, long press

```dart
import 'package:flutter/material.dart';

class GestureDetectorScreen extends StatefulWidget {
  const GestureDetectorScreen({super.key});

  @override
  State<GestureDetectorScreen> createState() => _GestureDetectorScreenState();
}

class _GestureDetectorScreenState extends State<GestureDetectorScreen> {
  String _lastGesture = 'ยังไม่มีการสัมผัส';
  Color _boxColor = Colors.blue;
  int _tapCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GestureDetector')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          // แสดง last gesture
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(
              _lastGesture,
              style: const TextStyle(fontSize: 18),
              textAlign: TextAlign.center,
            ),
          ),

          Center(
            child: GestureDetector(
              // Tap
              onTap: () {
                setState(() {
                  _tapCount++;
                  _lastGesture = 'Tap ครั้งที่ $_tapCount';
                  _boxColor = Colors.blue;
                });
              },

              // Double Tap
              onDoubleTap: () {
                setState(() {
                  _lastGesture = 'Double Tap!';
                  _boxColor = Colors.green;
                });
              },

              // Long Press
              onLongPress: () {
                setState(() {
                  _lastGesture = 'Long Press!';
                  _boxColor = Colors.orange;
                });
              },

              // Long Press with position
              onLongPressStart: (details) {
                print('Long press เริ่มที่ ${details.localPosition}');
              },

              onLongPressEnd: (details) {
                print('Long press จบที่ ${details.localPosition}');
              },

              // Tap Down/Up/Cancel
              onTapDown: (details) {
                setState(() => _boxColor = Colors.blue[800]!);
              },

              onTapUp: (details) {
                setState(() => _boxColor = Colors.blue);
              },

              onTapCancel: () {
                setState(() => _boxColor = Colors.blue);
              },

              // Behavior: กำหนดว่า gesture ทำงานที่ไหน
              behavior: HitTestBehavior.opaque,

              child: AnimatedContainer(
                duration: const Duration(milliseconds: 200),
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  color: _boxColor,
                  borderRadius: BorderRadius.circular(16),
                  boxShadow: [
                    BoxShadow(
                      color: _boxColor.withOpacity(0.4),
                      blurRadius: 12,
                      offset: const Offset(0, 6),
                    ),
                  ],
                ),
                child: const Center(
                  child: Text(
                    'แตะที่นี่',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),
          ),
          const SizedBox(height: 32),

          // คำแนะนำ
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 32),
            child: Column(
              children: [
                _buildHint('แตะ 1 ครั้ง', 'Tap (สีน้ำเงิน)'),
                _buildHint('แตะ 2 ครั้ง', 'Double Tap (สีเขียว)'),
                _buildHint('แตะค้าง', 'Long Press (สีส้ม)'),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildHint(String gesture, String result) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(gesture, style: const TextStyle(color: Colors.grey)),
          Text(result, style: const TextStyle(fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 122: Swipe และ Pan Gestures

```dart
import 'package:flutter/material.dart';

class SwipePanScreen extends StatefulWidget {
  const SwipePanScreen({super.key});

  @override
  State<SwipePanScreen> createState() => _SwipePanScreenState();
}

class _SwipePanScreenState extends State<SwipePanScreen> {
  Offset _position = const Offset(150, 300);
  String _lastDirection = '';
  double _velocityX = 0;
  double _velocityY = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Swipe & Pan')),
      body: Stack(
        children: [
          // Swipe detection area
          GestureDetector(
            onHorizontalDragEnd: (details) {
              setState(() {
                final velocity = details.primaryVelocity ?? 0;
                if (velocity > 0) {
                  _lastDirection = '← Swipe ซ้าย (velocity: ${velocity.toInt()})';
                } else {
                  _lastDirection = '→ Swipe ขวา (velocity: ${velocity.toInt()})';
                }
              });
            },
            onVerticalDragEnd: (details) {
              setState(() {
                final velocity = details.primaryVelocity ?? 0;
                if (velocity > 0) {
                  _lastDirection = '↑ Swipe ลง (velocity: ${velocity.toInt()})';
                } else {
                  _lastDirection = '↓ Swipe ขึ้น (velocity: ${velocity.toInt()})';
                }
              });
            },
            child: Container(
              color: Colors.grey[100],
              child: Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    const Text(
                      'Swipe ไปทิศทางใดก็ได้',
                      style: TextStyle(fontSize: 18),
                    ),
                    const SizedBox(height: 16),
                    if (_lastDirection.isNotEmpty)
                      Text(
                        _lastDirection,
                        style: const TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                          color: Colors.blue,
                        ),
                      ),
                  ],
                ),
              ),
            ),
          ),

          // Draggable ball (Pan gesture)
          Positioned(
            left: _position.dx - 30,
            top: _position.dy - 30,
            child: GestureDetector(
              onPanStart: (details) {
                // เริ่ม drag
              },
              onPanUpdate: (details) {
                setState(() {
                  _position += details.delta;
                  // จำกัดไม่ให้ออกนอกจอ
                  final size = MediaQuery.of(context).size;
                  _position = Offset(
                    _position.dx.clamp(30, size.width - 30),
                    _position.dy.clamp(30, size.height - 100),
                  );
                  _velocityX = details.delta.dx;
                  _velocityY = details.delta.dy;
                });
              },
              onPanEnd: (details) {
                setState(() {
                  _velocityX = details.velocity.pixelsPerSecond.dx;
                  _velocityY = details.velocity.pixelsPerSecond.dy;
                });
              },
              child: Container(
                width: 60,
                height: 60,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  gradient: const RadialGradient(
                    colors: [Colors.orange, Colors.red],
                  ),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.orange.withOpacity(0.5),
                      blurRadius: 10,
                      spreadRadius: 2,
                    ),
                  ],
                ),
                child: const Center(
                  child: Icon(Icons.drag_indicator, color: Colors.white),
                ),
              ),
            ),
          ),

          // แสดงค่า velocity
          Positioned(
            bottom: 16,
            left: 16,
            child: Text(
              'Velocity: (${_velocityX.toInt()}, ${_velocityY.toInt()})',
              style: const TextStyle(color: Colors.grey),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 123: Pinch to Zoom

```dart
import 'package:flutter/material.dart';

class PinchZoomScreen extends StatefulWidget {
  const PinchZoomScreen({super.key});

  @override
  State<PinchZoomScreen> createState() => _PinchZoomScreenState();
}

class _PinchZoomScreenState extends State<PinchZoomScreen> {
  double _scale = 1.0;
  double _previousScale = 1.0;
  Offset _offset = Offset.zero;
  Offset _previousOffset = Offset.zero;
  double _rotation = 0.0;
  double _previousRotation = 0.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Pinch & Zoom'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              setState(() {
                _scale = 1.0;
                _offset = Offset.zero;
                _rotation = 0.0;
              });
            },
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: Text(
              'Scale: ${_scale.toStringAsFixed(2)} | '
              'Rotation: ${(_rotation * 180 / 3.14159).toStringAsFixed(1)}°',
              style: const TextStyle(color: Colors.grey),
            ),
          ),
          Expanded(
            child: GestureDetector(
              onScaleStart: (details) {
                _previousScale = _scale;
                _previousOffset = _offset;
                _previousRotation = _rotation;
              },
              onScaleUpdate: (details) {
                setState(() {
                  // Scale
                  _scale = (_previousScale * details.scale).clamp(0.5, 5.0);
                  
                  // Rotation
                  _rotation = _previousRotation + details.rotation;
                  
                  // Pan (เฉพาะเมื่อ scale == 1 คือไม่ได้ pinch)
                  if (details.pointerCount == 1) {
                    _offset = _previousOffset + details.focalPointDelta;
                  }
                });
              },
              onScaleEnd: (details) {
                _previousScale = _scale;
              },
              child: Container(
                color: Colors.grey[200],
                child: Center(
                  child: Transform(
                    transform: Matrix4.identity()
                      ..translate(_offset.dx, _offset.dy)
                      ..scale(_scale)
                      ..rotateZ(_rotation),
                    alignment: Alignment.center,
                    child: Container(
                      width: 200,
                      height: 200,
                      decoration: BoxDecoration(
                        color: Colors.blue,
                        borderRadius: BorderRadius.circular(16),
                        image: const DecorationImage(
                          image: NetworkImage(
                            'https://picsum.photos/200/200?random=5',
                          ),
                          fit: BoxFit.cover,
                        ),
                      ),
                    ),
                  ),
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: const Text(
              'Pinch = Zoom | Twist = Rotate | Drag = Pan',
              style: TextStyle(color: Colors.grey),
              textAlign: TextAlign.center,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 124: InkWell และ InkResponse

`InkWell` และ `InkResponse` ให้ ripple effect เมื่อแตะ ตาม Material Design guidelines

```dart
import 'package:flutter/material.dart';

class InkWellScreen extends StatelessWidget {
  const InkWellScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('InkWell & InkResponse')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // 1. InkWell พื้นฐาน
          const Text('InkWell พื้นฐาน',
              style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          InkWell(
            onTap: () => _showSnackBar(context, 'InkWell tapped!'),
            onDoubleTap: () => _showSnackBar(context, 'Double tap!'),
            onLongPress: () => _showSnackBar(context, 'Long press!'),
            borderRadius: BorderRadius.circular(12),
            splashColor: Colors.blue.withOpacity(0.3),
            highlightColor: Colors.blue.withOpacity(0.1),
            child: Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                border: Border.all(color: Colors.blue),
                borderRadius: BorderRadius.circular(12),
              ),
              child: const Text('แตะที่นี่เพื่อดู ripple effect'),
            ),
          ),

          const SizedBox(height: 24),

          // 2. InkWell กับ Card
          const Text('InkWell กับ Card',
              style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          Card(
            child: InkWell(
              onTap: () => _showSnackBar(context, 'Card tapped!'),
              borderRadius: BorderRadius.circular(12),
              child: const Padding(
                padding: EdgeInsets.all(16),
                child: Row(
                  children: [
                    Icon(Icons.star, color: Colors.amber),
                    SizedBox(width: 16),
                    Text('Card ที่กดได้'),
                    Spacer(),
                    Icon(Icons.arrow_forward_ios, size: 16),
                  ],
                ),
              ),
            ),
          ),

          const SizedBox(height: 24),

          // 3. InkResponse - ripple ขยายออกจากจุดที่แตะ
          const Text('InkResponse (ripple จากจุดแตะ)',
              style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          Center(
            child: Material(
              color: Colors.blue,
              borderRadius: BorderRadius.circular(50),
              child: InkResponse(
                onTap: () => _showSnackBar(context, 'InkResponse tapped!'),
                containedInkWell: false, // ปล่อย ripple ออกนอกขอบ
                highlightShape: BoxShape.circle,
                radius: 60,
                child: const Padding(
                  padding: EdgeInsets.all(16),
                  child: Icon(Icons.favorite, color: Colors.white, size: 32),
                ),
              ),
            ),
          ),

          const SizedBox(height: 24),

          // 4. Custom InkWell shapes
          const Text('Custom Shape InkWell',
              style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 8),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              // Circle
              Material(
                color: Colors.purple[100],
                shape: const CircleBorder(),
                child: InkWell(
                  onTap: () => _showSnackBar(context, 'Circle!'),
                  customBorder: const CircleBorder(),
                  child: const Padding(
                    padding: EdgeInsets.all(16),
                    child: Icon(Icons.circle, color: Colors.purple),
                  ),
                ),
              ),
              // RoundedRect
              Material(
                color: Colors.green[100],
                borderRadius: BorderRadius.circular(8),
                child: InkWell(
                  onTap: () => _showSnackBar(context, 'Rectangle!'),
                  borderRadius: BorderRadius.circular(8),
                  child: const Padding(
                    padding: EdgeInsets.all(16),
                    child: Icon(Icons.square_rounded, color: Colors.green),
                  ),
                ),
              ),
              // Stadium
              Material(
                color: Colors.orange[100],
                shape: const StadiumBorder(),
                child: InkWell(
                  onTap: () => _showSnackBar(context, 'Stadium!'),
                  customBorder: const StadiumBorder(),
                  child: const Padding(
                    padding: EdgeInsets.symmetric(horizontal: 20, vertical: 12),
                    child: Text('Stadium', style: TextStyle(color: Colors.orange)),
                  ),
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  void _showSnackBar(BuildContext context, String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message), duration: const Duration(seconds: 1)),
    );
  }
}
```

---

## ขั้นตอนที่ 125: Draggable และ DragTarget

```dart
import 'package:flutter/material.dart';

class DragDropScreen extends StatefulWidget {
  const DragDropScreen({super.key});

  @override
  State<DragDropScreen> createState() => _DragDropScreenState();
}

class _DragDropScreenState extends State<DragDropScreen> {
  final List<String> _availableItems = ['🍎 แอปเปิ้ล', '🍌 กล้วย', '🍊 ส้ม', '🍇 องุ่น'];
  final List<String> _basket = [];
  String? _hoveredItem;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Drag & Drop')),
      body: Column(
        children: [
          // Source - ผลไม้ที่ลาก
          Expanded(
            child: Column(
              children: [
                const Padding(
                  padding: EdgeInsets.all(16),
                  child: Text(
                    'ลากผลไม้ลงตะกร้า',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                ),
                Wrap(
                  spacing: 16,
                  runSpacing: 16,
                  alignment: WrapAlignment.center,
                  children: _availableItems.map((item) {
                    return Draggable<String>(
                      data: item,
                      // Widget ขณะกำลัง drag
                      feedback: Material(
                        elevation: 6,
                        borderRadius: BorderRadius.circular(12),
                        child: Container(
                          padding: const EdgeInsets.all(12),
                          decoration: BoxDecoration(
                            color: Colors.orange,
                            borderRadius: BorderRadius.circular(12),
                          ),
                          child: Text(
                            item,
                            style: const TextStyle(
                              color: Colors.white,
                              fontSize: 18,
                            ),
                          ),
                        ),
                      ),
                      // Widget ที่เหลืออยู่ที่เดิม (ขณะ drag)
                      childWhenDragging: Opacity(
                        opacity: 0.3,
                        child: _buildFruitCard(item),
                      ),
                      child: _buildFruitCard(item),
                    );
                  }).toList(),
                ),
              ],
            ),
          ),

          const Divider(thickness: 2),

          // Target - ตะกร้า
          Expanded(
            child: Column(
              children: [
                const Padding(
                  padding: EdgeInsets.all(16),
                  child: Text(
                    'ตะกร้าผลไม้',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                ),
                DragTarget<String>(
                  onWillAcceptWithDetails: (details) {
                    setState(() => _hoveredItem = details.data);
                    return true; // รับ item ที่ drag มา
                  },
                  onAcceptWithDetails: (details) {
                    setState(() {
                      _basket.add(details.data);
                      _hoveredItem = null;
                    });
                  },
                  onLeave: (data) {
                    setState(() => _hoveredItem = null);
                  },
                  builder: (context, candidateItems, rejectedItems) {
                    final isHovered = candidateItems.isNotEmpty;
                    return AnimatedContainer(
                      duration: const Duration(milliseconds: 200),
                      width: double.infinity,
                      height: 180,
                      margin: const EdgeInsets.symmetric(horizontal: 16),
                      decoration: BoxDecoration(
                        color: isHovered
                            ? Colors.green[100]
                            : Colors.grey[200],
                        borderRadius: BorderRadius.circular(16),
                        border: Border.all(
                          color: isHovered ? Colors.green : Colors.grey,
                          width: 2,
                          style: BorderStyle.solid,
                        ),
                      ),
                      child: _basket.isEmpty
                          ? Center(
                              child: Text(
                                isHovered ? 'วางที่นี่!' : 'ลากผลไม้มาวางที่นี่',
                                style: TextStyle(
                                  color: isHovered ? Colors.green : Colors.grey,
                                  fontSize: 16,
                                ),
                              ),
                            )
                          : Padding(
                              padding: const EdgeInsets.all(8),
                              child: Wrap(
                                spacing: 8,
                                runSpacing: 8,
                                children: _basket
                                    .map((item) => Chip(
                                          label: Text(item),
                                          onDeleted: () {
                                            setState(() => _basket.remove(item));
                                          },
                                        ))
                                    .toList(),
                              ),
                            ),
                    );
                  },
                ),
                if (_basket.isNotEmpty)
                  Padding(
                    padding: const EdgeInsets.all(8),
                    child: Text(
                      'ในตะกร้า: ${_basket.length} รายการ',
                      style: const TextStyle(color: Colors.grey),
                    ),
                  ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildFruitCard(String item) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(12),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 6,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Text(item, style: const TextStyle(fontSize: 18)),
    );
  }
}
```

---

## ขั้นตอนที่ 126: ReorderableListView

```dart
import 'package:flutter/material.dart';

class ReorderableListScreen extends StatefulWidget {
  const ReorderableListScreen({super.key});

  @override
  State<ReorderableListScreen> createState() => _ReorderableListScreenState();
}

class _ReorderableListScreenState extends State<ReorderableListScreen> {
  List<Map<String, dynamic>> _tasks = [
    {'id': 1, 'title': 'ออกแบบ UI', 'priority': 'สูง', 'done': false},
    {'id': 2, 'title': 'เขียน API', 'priority': 'สูง', 'done': false},
    {'id': 3, 'title': 'เขียน Test', 'priority': 'กลาง', 'done': false},
    {'id': 4, 'title': 'Deploy', 'priority': 'ต่ำ', 'done': false},
    {'id': 5, 'title': 'เขียน Docs', 'priority': 'ต่ำ', 'done': true},
    {'id': 6, 'title': 'Code Review', 'priority': 'กลาง', 'done': false},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Reorderable List'),
        actions: [
          IconButton(
            icon: const Icon(Icons.info_outline),
            onPressed: () {
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('กดค้างเพื่อลากจัดเรียง')),
              );
            },
          ),
        ],
      ),
      body: ReorderableListView.builder(
        padding: const EdgeInsets.all(8),
        itemCount: _tasks.length,
        itemBuilder: (context, index) {
          final task = _tasks[index];
          return _buildTaskItem(task, index);
        },
        // ฟังก์ชันที่เรียกเมื่อมีการจัดเรียงใหม่
        onReorder: (oldIndex, newIndex) {
          setState(() {
            if (newIndex > oldIndex) newIndex--;
            final item = _tasks.removeAt(oldIndex);
            _tasks.insert(newIndex, item);
          });
        },
        // กำหนด proxy decorator (widget ที่แสดงขณะ drag)
        proxyDecorator: (child, index, animation) {
          return AnimatedBuilder(
            animation: animation,
            builder: (context, child) {
              return Material(
                elevation: 4,
                borderRadius: BorderRadius.circular(12),
                child: child,
              );
            },
            child: child,
          );
        },
      ),
    );
  }

  Widget _buildTaskItem(Map<String, dynamic> task, int index) {
    final priorityColors = {
      'สูง': Colors.red,
      'กลาง': Colors.orange,
      'ต่ำ': Colors.green,
    };

    return Card(
      key: ValueKey(task['id']),
      margin: const EdgeInsets.symmetric(vertical: 4),
      child: ListTile(
        leading: Checkbox(
          value: task['done'] as bool,
          onChanged: (value) {
            setState(() => task['done'] = value);
          },
        ),
        title: Text(
          task['title'] as String,
          style: TextStyle(
            decoration: (task['done'] as bool)
                ? TextDecoration.lineThrough
                : null,
            color: (task['done'] as bool) ? Colors.grey : null,
          ),
        ),
        subtitle: Text('ลำดับ: ${index + 1}'),
        trailing: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
              decoration: BoxDecoration(
                color: priorityColors[task['priority']]!.withOpacity(0.1),
                borderRadius: BorderRadius.circular(4),
              ),
              child: Text(
                task['priority'] as String,
                style: TextStyle(
                  color: priorityColors[task['priority']],
                  fontSize: 12,
                ),
              ),
            ),
            const SizedBox(width: 8),
            const Icon(Icons.drag_handle, color: Colors.grey),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 127: Dismissible

`Dismissible` ช่วยให้ swipe to delete item ออกจาก list ได้

```dart
import 'package:flutter/material.dart';

class DismissibleScreen extends StatefulWidget {
  const DismissibleScreen({super.key});

  @override
  State<DismissibleScreen> createState() => _DismissibleScreenState();
}

class _DismissibleScreenState extends State<DismissibleScreen> {
  List<Map<String, dynamic>> _emails = List.generate(
    15,
    (index) => {
      'id': index,
      'sender': ['สมชาย', 'อนงค์', 'บุญรอด', 'กมล', 'ทักษิณ'][index % 5],
      'subject': 'เรื่อง: ประชุมวันที่ ${index + 1}',
      'time': '${index + 1}:00',
      'isRead': index % 3 == 0,
    },
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Email - Swipe to Action'),
        actions: [
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: () {
              setState(() {
                _emails.removeWhere((e) => e['isRead'] as bool);
              });
            },
            tooltip: 'ลบที่อ่านแล้วทั้งหมด',
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: _emails.length,
        itemBuilder: (context, index) {
          final email = _emails[index];
          return _buildDismissibleEmail(email, index);
        },
      ),
    );
  }

  Widget _buildDismissibleEmail(Map<String, dynamic> email, int index) {
    return Dismissible(
      key: ValueKey(email['id']),
      
      // Swipe ซ้าย = Delete
      background: Container(
        color: Colors.red,
        alignment: Alignment.centerRight,
        padding: const EdgeInsets.only(right: 16),
        child: const Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.delete, color: Colors.white),
            Text('ลบ', style: TextStyle(color: Colors.white)),
          ],
        ),
      ),
      
      // Swipe ขวา = Archive
      secondaryBackground: Container(
        color: Colors.green,
        alignment: Alignment.centerLeft,
        padding: const EdgeInsets.only(left: 16),
        child: const Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.archive, color: Colors.white),
            Text('เก็บ', style: TextStyle(color: Colors.white)),
          ],
        ),
      ),
      
      // direction: Dismissible.horizontal, สำหรับ swipe ทั้งสองทาง
      direction: DismissDirection.horizontal,
      
      // confirmDismiss ให้ผู้ใช้ยืนยันก่อน
      confirmDismiss: (direction) async {
        if (direction == DismissDirection.endToStart) {
          // Swipe ซ้าย = Delete - ขอยืนยัน
          return await showDialog<bool>(
            context: context,
            builder: (context) => AlertDialog(
              title: const Text('ยืนยันการลบ?'),
              content: Text('ลบอีเมลจาก ${email['sender']}?'),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context, false),
                  child: const Text('ยกเลิก'),
                ),
                TextButton(
                  onPressed: () => Navigator.pop(context, true),
                  style: TextButton.styleFrom(foregroundColor: Colors.red),
                  child: const Text('ลบ'),
                ),
              ],
            ),
          );
        }
        // Swipe ขวา = Archive - ไม่ต้องยืนยัน
        return true;
      },
      
      // เรียกเมื่อ dismiss เสร็จแล้ว
      onDismissed: (direction) {
        setState(() => _emails.removeAt(index));
        
        final message = direction == DismissDirection.endToStart
            ? 'ลบอีเมลแล้ว'
            : 'เก็บอีเมลแล้ว';
        
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(message),
            action: SnackBarAction(
              label: 'เลิกทำ',
              onPressed: () {
                setState(() => _emails.insert(index, email));
              },
            ),
          ),
        );
      },
      
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: (email['isRead'] as bool)
              ? Colors.grey[300]
              : Colors.blue,
          child: Text(
            (email['sender'] as String)[0],
            style: TextStyle(
              color: (email['isRead'] as bool) ? Colors.grey : Colors.white,
            ),
          ),
        ),
        title: Text(
          email['sender'] as String,
          style: TextStyle(
            fontWeight: (email['isRead'] as bool)
                ? FontWeight.normal
                : FontWeight.bold,
          ),
        ),
        subtitle: Text(
          email['subject'] as String,
          overflow: TextOverflow.ellipsis,
        ),
        trailing: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              email['time'] as String,
              style: const TextStyle(fontSize: 12, color: Colors.grey),
            ),
            if (!(email['isRead'] as bool))
              Container(
                width: 8,
                height: 8,
                decoration: const BoxDecoration(
                  color: Colors.blue,
                  shape: BoxShape.circle,
                ),
              ),
          ],
        ),
        onTap: () {
          setState(() => email['isRead'] = true);
        },
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 128: Listener และ PointerEvent

`Listener` ให้ raw pointer events ที่ level ต่ำกว่า GestureDetector

```dart
import 'package:flutter/material.dart';

class ListenerScreen extends StatefulWidget {
  const ListenerScreen({super.key});

  @override
  State<ListenerScreen> createState() => _ListenerScreenState();
}

class _ListenerScreenState extends State<ListenerScreen> {
  final List<Map<String, dynamic>> _events = [];
  final List<Offset> _trail = [];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Listener & Pointer Events'),
        actions: [
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () => setState(() {
              _events.clear();
              _trail.clear();
            }),
          ),
        ],
      ),
      body: Column(
        children: [
          // Drawing area
          Expanded(
            flex: 2,
            child: Listener(
              onPointerDown: (event) {
                setState(() {
                  _trail.clear();
                  _trail.add(event.localPosition);
                  _addEvent('Down', event.localPosition, event.pointer);
                });
              },
              onPointerMove: (event) {
                setState(() {
                  _trail.add(event.localPosition);
                  _addEvent('Move', event.localPosition, event.pointer);
                });
              },
              onPointerUp: (event) {
                setState(() {
                  _addEvent('Up', event.localPosition, event.pointer);
                });
              },
              onPointerCancel: (event) {
                setState(() {
                  _addEvent('Cancel', event.localPosition, event.pointer);
                });
              },
              // Hover (สำหรับ web/desktop)
              onPointerHover: (event) {
                // เฉพาะ web/desktop
              },
              child: Container(
                color: Colors.grey[100],
                child: CustomPaint(
                  painter: _TrailPainter(trail: _trail),
                  child: const Center(
                    child: Text(
                      'วาดที่นี่',
                      style: TextStyle(color: Colors.grey),
                    ),
                  ),
                ),
              ),
            ),
          ),

          // Event log
          Expanded(
            child: Container(
              color: Colors.black87,
              child: ListView.builder(
                reverse: true,
                padding: const EdgeInsets.all(8),
                itemCount: _events.length,
                itemBuilder: (context, index) {
                  final event = _events[_events.length - 1 - index];
                  return Text(
                    '[Pointer ${event['pointer']}] ${event['type']} at '
                    '(${(event['position'] as Offset).dx.toInt()}, '
                    '${(event['position'] as Offset).dy.toInt()})',
                    style: TextStyle(
                      color: _getEventColor(event['type'] as String),
                      fontSize: 12,
                      fontFamily: 'monospace',
                    ),
                  );
                },
              ),
            ),
          ),
        ],
      ),
    );
  }

  void _addEvent(String type, Offset position, int pointer) {
    _events.add({'type': type, 'position': position, 'pointer': pointer});
    // จำกัด 50 events ล่าสุด
    if (_events.length > 50) _events.removeAt(0);
  }

  Color _getEventColor(String type) {
    switch (type) {
      case 'Down': return Colors.green;
      case 'Move': return Colors.blue;
      case 'Up': return Colors.orange;
      case 'Cancel': return Colors.red;
      default: return Colors.white;
    }
  }
}

class _TrailPainter extends CustomPainter {
  final List<Offset> trail;

  _TrailPainter({required this.trail});

  @override
  void paint(Canvas canvas, Size size) {
    if (trail.length < 2) return;

    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 4
      ..strokeCap = StrokeCap.round
      ..strokeJoin = StrokeJoin.round
      ..style = PaintingStyle.stroke;

    final path = Path();
    path.moveTo(trail.first.dx, trail.first.dy);
    for (int i = 1; i < trail.length; i++) {
      path.lineTo(trail[i].dx, trail[i].dy);
    }
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(_TrailPainter old) => true;
}
```

---

## ขั้นตอนที่ 129: Custom Gestures

```dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';

// Custom GestureRecognizer สำหรับ Double Tap ที่เร็วกว่าปกติ
class FastDoubleTapGestureRecognizer extends DoubleTapGestureRecognizer {
  FastDoubleTapGestureRecognizer() {
    // Override doubleTapTimeout ถ้าต้องการ
  }
}

class CustomGestureScreen extends StatefulWidget {
  const CustomGestureScreen({super.key});

  @override
  State<CustomGestureScreen> createState() => _CustomGestureScreenState();
}

class _CustomGestureScreenState extends State<CustomGestureScreen> {
  int _likeCount = 0;
  bool _isLiked = false;
  String _gestureInfo = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Gestures')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // RichText กับ TapGestureRecognizer
            const Text('RichText with Tap:'),
            const SizedBox(height: 8),
            RichText(
              text: TextSpan(
                style: const TextStyle(color: Colors.black, fontSize: 16),
                children: [
                  const TextSpan(text: 'ยอมรับ '),
                  TextSpan(
                    text: 'เงื่อนไขการใช้งาน',
                    style: const TextStyle(
                      color: Colors.blue,
                      decoration: TextDecoration.underline,
                    ),
                    recognizer: TapGestureRecognizer()
                      ..onTap = () {
                        setState(() => _gestureInfo = 'เปิดเงื่อนไขการใช้งาน');
                      },
                  ),
                  const TextSpan(text: ' และ '),
                  TextSpan(
                    text: 'นโยบายความเป็นส่วนตัว',
                    style: const TextStyle(
                      color: Colors.blue,
                      decoration: TextDecoration.underline,
                    ),
                    recognizer: TapGestureRecognizer()
                      ..onTap = () {
                        setState(() => _gestureInfo = 'เปิดนโยบายความเป็นส่วนตัว');
                      },
                  ),
                ],
              ),
            ),

            const SizedBox(height: 32),

            // Double tap to like (Instagram style)
            const Text('Double tap to like:'),
            const SizedBox(height: 8),
            GestureDetector(
              onDoubleTap: () {
                setState(() {
                  _isLiked = true;
                  _likeCount++;
                  _gestureInfo = '❤️ Like! (${_likeCount} likes)';
                });
              },
              child: Container(
                width: 250,
                height: 250,
                decoration: BoxDecoration(
                  color: Colors.grey[200],
                  borderRadius: BorderRadius.circular(16),
                ),
                child: Stack(
                  alignment: Alignment.center,
                  children: [
                    Image.network(
                      'https://picsum.photos/250/250?random=7',
                      fit: BoxFit.cover,
                      errorBuilder: (_, __, ___) =>
                          const Icon(Icons.image, size: 60),
                    ),
                    if (_isLiked)
                      TweenAnimationBuilder<double>(
                        key: ValueKey(_likeCount),
                        tween: Tween(begin: 1.5, end: 0),
                        duration: const Duration(milliseconds: 800),
                        builder: (context, value, _) {
                          return Opacity(
                            opacity: value.clamp(0, 1),
                            child: Transform.scale(
                              scale: value,
                              child: const Icon(
                                Icons.favorite,
                                color: Colors.white,
                                size: 80,
                              ),
                            ),
                          );
                        },
                        onEnd: () => setState(() => _isLiked = false),
                      ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),
            Text(
              _gestureInfo,
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 130: Multi-touch Gestures

```dart
import 'package:flutter/material.dart';

class MultiTouchScreen extends StatefulWidget {
  const MultiTouchScreen({super.key});

  @override
  State<MultiTouchScreen> createState() => _MultiTouchScreenState();
}

class _MultiTouchScreenState extends State<MultiTouchScreen> {
  final Map<int, Offset> _pointers = {};
  int _maxSimultaneousPointers = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Multi-touch')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              children: [
                Text(
                  'กำลังสัมผัส: ${_pointers.length} นิ้ว',
                  style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                ),
                Text(
                  'สูงสุด: $_maxSimultaneousPointers นิ้ว',
                  style: const TextStyle(color: Colors.grey),
                ),
              ],
            ),
          ),
          Expanded(
            child: Listener(
              onPointerDown: (event) {
                setState(() {
                  _pointers[event.pointer] = event.localPosition;
                  if (_pointers.length > _maxSimultaneousPointers) {
                    _maxSimultaneousPointers = _pointers.length;
                  }
                });
              },
              onPointerMove: (event) {
                setState(() {
                  _pointers[event.pointer] = event.localPosition;
                });
              },
              onPointerUp: (event) {
                setState(() {
                  _pointers.remove(event.pointer);
                });
              },
              onPointerCancel: (event) {
                setState(() {
                  _pointers.remove(event.pointer);
                });
              },
              child: Container(
                color: Colors.grey[100],
                child: CustomPaint(
                  painter: _MultiTouchPainter(pointers: _pointers),
                  child: const Center(
                    child: Text(
                      'แตะหลายนิ้วพร้อมกัน',
                      style: TextStyle(color: Colors.grey),
                    ),
                  ),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _MultiTouchPainter extends CustomPainter {
  final Map<int, Offset> pointers;
  final List<Color> _colors = [
    Colors.red, Colors.blue, Colors.green, Colors.orange,
    Colors.purple, Colors.pink, Colors.teal, Colors.amber,
  ];

  _MultiTouchPainter({required this.pointers});

  @override
  void paint(Canvas canvas, Size size) {
    var colorIndex = 0;
    for (final entry in pointers.entries) {
      final color = _colors[colorIndex % _colors.length];
      colorIndex++;

      // วงกลมใหญ่
      canvas.drawCircle(
        entry.value,
        50,
        Paint()..color = color.withOpacity(0.3),
      );
      // วงกลมเล็ก
      canvas.drawCircle(
        entry.value,
        10,
        Paint()..color = color,
      );

      // ป้าย pointer ID
      final textPainter = TextPainter(
        text: TextSpan(
          text: '#${entry.key}',
          style: TextStyle(color: color, fontSize: 12),
        ),
        textDirection: TextDirection.ltr,
      )..layout();
      textPainter.paint(canvas, entry.value + const Offset(15, -10));
    }
  }

  @override
  bool shouldRepaint(_MultiTouchPainter old) => true;
}
```

---

## Workshop: Drag & Drop Kanban Board

```dart
import 'package:flutter/material.dart';

enum TaskStatus { todo, inProgress, done }

class KanbanTask {
  final String id;
  String title;
  String description;
  TaskStatus status;
  String priority;

  KanbanTask({
    required this.id,
    required this.title,
    required this.description,
    required this.status,
    required this.priority,
  });
}

class KanbanBoardApp extends StatelessWidget {
  const KanbanBoardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Kanban Board',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const KanbanBoardScreen(),
    );
  }
}

class KanbanBoardScreen extends StatefulWidget {
  const KanbanBoardScreen({super.key});

  @override
  State<KanbanBoardScreen> createState() => _KanbanBoardScreenState();
}

class _KanbanBoardScreenState extends State<KanbanBoardScreen> {
  final List<KanbanTask> _tasks = [
    KanbanTask(id: '1', title: 'ออกแบบ UI', description: 'สร้าง wireframe', status: TaskStatus.todo, priority: 'สูง'),
    KanbanTask(id: '2', title: 'สร้าง API', description: 'endpoints CRUD', status: TaskStatus.todo, priority: 'สูง'),
    KanbanTask(id: '3', title: 'เขียน Test', description: 'unit tests', status: TaskStatus.inProgress, priority: 'กลาง'),
    KanbanTask(id: '4', title: 'Code Review', description: 'ตรวจ code', status: TaskStatus.inProgress, priority: 'กลาง'),
    KanbanTask(id: '5', title: 'Deploy Staging', description: 'test environment', status: TaskStatus.done, priority: 'ต่ำ'),
    KanbanTask(id: '6', title: 'เขียน Docs', description: 'API documentation', status: TaskStatus.todo, priority: 'ต่ำ'),
  ];

  List<KanbanTask> _getTasksByStatus(TaskStatus status) {
    return _tasks.where((t) => t.status == status).toList();
  }

  void _moveTask(KanbanTask task, TaskStatus newStatus) {
    setState(() => task.status = newStatus);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('🎯 Kanban Board'),
        actions: [
          IconButton(icon: const Icon(Icons.add), onPressed: _addTask),
        ],
      ),
      body: SingleChildScrollView(
        scrollDirection: Axis.horizontal,
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: TaskStatus.values.map((status) {
            return _buildColumn(status);
          }).toList(),
        ),
      ),
    );
  }

  Widget _buildColumn(TaskStatus status) {
    final tasks = _getTasksByStatus(status);
    final columnData = {
      TaskStatus.todo: ('Todo', Colors.grey, Icons.radio_button_unchecked),
      TaskStatus.inProgress: ('In Progress', Colors.blue, Icons.pending),
      TaskStatus.done: ('Done', Colors.green, Icons.check_circle),
    }[status]!;

    return DragTarget<KanbanTask>(
      onWillAcceptWithDetails: (details) => details.data.status != status,
      onAcceptWithDetails: (details) => _moveTask(details.data, status),
      builder: (context, candidateItems, rejectedItems) {
        final isHovered = candidateItems.isNotEmpty;
        return AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          width: 280,
          margin: const EdgeInsets.all(8),
          decoration: BoxDecoration(
            color: isHovered
                ? columnData.$2.withOpacity(0.1)
                : Colors.grey[100],
            borderRadius: BorderRadius.circular(12),
            border: isHovered
                ? Border.all(color: columnData.$2, width: 2)
                : null,
          ),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              // Column Header
              Container(
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: columnData.$2,
                  borderRadius: const BorderRadius.only(
                    topLeft: Radius.circular(12),
                    topRight: Radius.circular(12),
                  ),
                ),
                child: Row(
                  children: [
                    Icon(columnData.$3, color: Colors.white, size: 20),
                    const SizedBox(width: 8),
                    Text(
                      columnData.$1,
                      style: const TextStyle(
                        color: Colors.white,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const Spacer(),
                    Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 2),
                      decoration: BoxDecoration(
                        color: Colors.white.withOpacity(0.3),
                        borderRadius: BorderRadius.circular(12),
                      ),
                      child: Text(
                        '${tasks.length}',
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                  ],
                ),
              ),

              // Task cards
              Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  children: tasks.map((task) => _buildTaskCard(task)).toList(),
                ),
              ),

              // Drop hint
              if (isHovered)
                Container(
                  margin: const EdgeInsets.all(8),
                  height: 60,
                  decoration: BoxDecoration(
                    border: Border.all(
                      color: columnData.$2,
                      style: BorderStyle.solid,
                      width: 2,
                    ),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Center(
                    child: Text(
                      'วางที่นี่',
                      style: TextStyle(color: columnData.$2),
                    ),
                  ),
                ),
            ],
          ),
        );
      },
    );
  }

  Widget _buildTaskCard(KanbanTask task) {
    final priorityColors = {
      'สูง': Colors.red,
      'กลาง': Colors.orange,
      'ต่ำ': Colors.green,
    };

    return Draggable<KanbanTask>(
      data: task,
      feedback: Material(
        elevation: 8,
        borderRadius: BorderRadius.circular(8),
        child: Container(
          width: 250,
          padding: const EdgeInsets.all(12),
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(8),
          ),
          child: Text(task.title, style: const TextStyle(fontWeight: FontWeight.bold)),
        ),
      ),
      childWhenDragging: Opacity(
        opacity: 0.3,
        child: _buildCardContent(task, priorityColors),
      ),
      child: _buildCardContent(task, priorityColors),
    );
  }

  Widget _buildCardContent(KanbanTask task, Map<String, Color> priorityColors) {
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 4),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Expanded(
                  child: Text(
                    task.title,
                    style: const TextStyle(fontWeight: FontWeight.bold),
                  ),
                ),
                Container(
                  padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                  decoration: BoxDecoration(
                    color: priorityColors[task.priority]!.withOpacity(0.1),
                    borderRadius: BorderRadius.circular(4),
                  ),
                  child: Text(
                    task.priority,
                    style: TextStyle(
                      color: priorityColors[task.priority],
                      fontSize: 11,
                    ),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 4),
            Text(
              task.description,
              style: const TextStyle(color: Colors.grey, fontSize: 12),
            ),
          ],
        ),
      ),
    );
  }

  void _addTask() {
    final controller = TextEditingController();
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('เพิ่มงานใหม่'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(labelText: 'ชื่องาน'),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                setState(() {
                  _tasks.add(KanbanTask(
                    id: DateTime.now().millisecondsSinceEpoch.toString(),
                    title: controller.text,
                    description: 'งานใหม่',
                    status: TaskStatus.todo,
                    priority: 'กลาง',
                  ));
                });
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

void main() => runApp(const KanbanBoardApp());
```

---

## สรุป (Summary)

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| GestureDetector | tap, double tap, long press |
| Pan/Swipe | การ drag และ swipe |
| Pinch to Zoom | scale และ rotation gesture |
| InkWell/InkResponse | ripple effect แบบ Material |
| Draggable/DragTarget | drag and drop |
| ReorderableListView | จัดเรียงรายการ |
| Dismissible | swipe to delete/archive |
| Listener/PointerEvent | raw touch events |
| Custom Gestures | TapGestureRecognizer |
| Multi-touch | จัดการหลาย pointer พร้อมกัน |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง swipe card (เหมือน Tinder) ที่ swipe ซ้าย/ขวาได้
2. **ปานกลาง**: สร้าง drawing app ที่วาดด้วยนิ้ว เปลี่ยนสีและขนาดได้
3. **ยาก**: สร้าง puzzle game ที่ drag ชิ้นส่วนมาวางในช่องที่ถูกต้อง

---

[← Part 12: Animations](part_12.md) | [Part 14: State Management →](part_14.md)

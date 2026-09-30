# Part 12: Animations พื้นฐาน
## ขั้นตอนที่ 111-120

---

## สารบัญ
1. [AnimationController](#ขั้นตอนที่-111-animationcontroller)
2. [Tween และ CurvedAnimation](#ขั้นตอนที่-112-tween-และ-curvedanimation)
3. [AnimatedWidget](#ขั้นตอนที่-113-animatedwidget)
4. [AnimatedBuilder](#ขั้นตอนที่-114-animatedbuilder)
5. [AnimatedContainer](#ขั้นตอนที่-115-animatedcontainer)
6. [AnimatedOpacity และ AnimatedSwitcher](#ขั้นตอนที่-116-animatedopacity-และ-animatedswitcher)
7. [AnimatedCrossFade](#ขั้นตอนที่-117-animatedcrossfade)
8. [Hero Animations](#ขั้นตอนที่-118-hero-animations)
9. [TweenAnimationBuilder](#ขั้นตอนที่-119-tweenanimationbuilder)
10. [Staggered Animations](#ขั้นตอนที่-120-staggered-animations)
11. [Workshop: Animated Dashboard](#workshop-animated-dashboard)

---

## ขั้นตอนที่ 111: AnimationController

`AnimationController` คือ ตัวควบคุม Animation หลักใน Flutter ทำหน้าที่จัดการค่าของ animation ตั้งแต่ 0.0 ถึง 1.0

```dart
import 'package:flutter/material.dart';

class AnimationControllerExample extends StatefulWidget {
  const AnimationControllerExample({super.key});

  @override
  State<AnimationControllerExample> createState() =>
      _AnimationControllerExampleState();
}

class _AnimationControllerExampleState extends State<AnimationControllerExample>
    with SingleTickerProviderStateMixin {
  // SingleTickerProviderStateMixin จำเป็นสำหรับ 1 animation
  // TickerProviderStateMixin สำหรับหลาย animation
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this, // ใช้ this ได้เพราะมี SingleTickerProviderStateMixin
      duration: const Duration(milliseconds: 1000),
      // reverseDuration: ความเร็วตอน reverse (ถ้าไม่กำหนดจะใช้ duration เดิม)
    );

    // Listener สำหรับติดตามค่า
    _controller.addListener(() {
      // _controller.value อยู่ระหว่าง 0.0 - 1.0
    });

    // Status listener
    _controller.addStatusListener((status) {
      switch (status) {
        case AnimationStatus.forward:
          print('Animation กำลังไปข้างหน้า');
          break;
        case AnimationStatus.reverse:
          print('Animation กำลังย้อนกลับ');
          break;
        case AnimationStatus.completed:
          print('Animation เสร็จสิ้น (ค่า = 1.0)');
          break;
        case AnimationStatus.dismissed:
          print('Animation ถูก dismiss (ค่า = 0.0)');
          break;
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose(); // สำคัญ! ต้อง dispose เสมอ
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimationController')),
      body: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          // แสดงค่า animation
          AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Column(
                children: [
                  // Box ที่ขยายตาม animation
                  Container(
                    width: 50 + _controller.value * 200,
                    height: 50 + _controller.value * 200,
                    decoration: BoxDecoration(
                      color: Color.lerp(
                        Colors.blue,
                        Colors.red,
                        _controller.value,
                      ),
                      borderRadius: BorderRadius.circular(
                        _controller.value * 50,
                      ),
                    ),
                  ),
                  const SizedBox(height: 16),
                  Text(
                    'ค่า: ${_controller.value.toStringAsFixed(2)}',
                    style: const TextStyle(fontSize: 18),
                  ),
                  Text(
                    'สถานะ: ${_controller.status.name}',
                    style: const TextStyle(fontSize: 14, color: Colors.grey),
                  ),
                ],
              );
            },
          ),
          const SizedBox(height: 32),
          // ปุ่มควบคุม
          Wrap(
            spacing: 8,
            children: [
              ElevatedButton(
                onPressed: () => _controller.forward(),
                child: const Text('Forward'),
              ),
              ElevatedButton(
                onPressed: () => _controller.reverse(),
                child: const Text('Reverse'),
              ),
              ElevatedButton(
                onPressed: () => _controller.stop(),
                child: const Text('Stop'),
              ),
              ElevatedButton(
                onPressed: () => _controller.reset(),
                child: const Text('Reset'),
              ),
              ElevatedButton(
                onPressed: () => _controller.repeat(reverse: true),
                child: const Text('Repeat'),
              ),
              ElevatedButton(
                onPressed: () => _controller.animateTo(0.5),
                child: const Text('Go to 0.5'),
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

## ขั้นตอนที่ 112: Tween และ CurvedAnimation

`Tween` แปลงค่า 0.0-1.0 ของ AnimationController ไปเป็นค่าที่ต้องการ เช่น สี, ขนาด, ตำแหน่ง

```dart
import 'package:flutter/material.dart';

class TweenExample extends StatefulWidget {
  const TweenExample({super.key});

  @override
  State<TweenExample> createState() => _TweenExampleState();
}

class _TweenExampleState extends State<TweenExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  // Tween ต่างๆ
  late Animation<double> _sizeAnimation;     // ขนาด
  late Animation<Color?> _colorAnimation;    // สี
  late Animation<double> _rotationAnimation; // การหมุน
  late Animation<Offset> _slideAnimation;    // ตำแหน่ง
  late Animation<double> _opacityAnimation;  // ความโปร่งใส

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );

    // CurvedAnimation - ใส่ curve ให้ animation
    final curvedAnimation = CurvedAnimation(
      parent: _controller,
      curve: Curves.elasticOut, // ลองเปลี่ยน curve ดู
      // curve: Curves.bounceOut,
      // curve: Curves.easeInOutCubic,
      // curve: Curves.fastOutSlowIn,
    );

    // Size Tween: 50 → 150
    _sizeAnimation = Tween<double>(begin: 50, end: 150).animate(curvedAnimation);

    // Color Tween: blue → orange
    _colorAnimation = ColorTween(
      begin: Colors.blue,
      end: Colors.orange,
    ).animate(_controller); // ไม่ใส่ curve = linear

    // Rotation Tween: 0 → 360 degrees (ใน radians)
    _rotationAnimation = Tween<double>(
      begin: 0,
      end: 2 * 3.14159, // 360 degrees in radians
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    ));

    // Slide Tween: ซ้าย → ขวา
    _slideAnimation = Tween<Offset>(
      begin: const Offset(-1, 0), // ซ้ายนอกจอ
      end: Offset.zero,           // ตำแหน่งปกติ
    ).animate(CurvedAnimation(
      parent: _controller,
      curve: Curves.easeOut,
    ));

    // Opacity Tween: 0 → 1
    _opacityAnimation = Tween<double>(begin: 0, end: 1)
        .animate(CurvedAnimation(
          parent: _controller,
          curve: const Interval(0.5, 1.0), // เริ่มเมื่อ controller ถึง 0.5
        ));

    _controller.repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Tween & CurvedAnimation')),
      body: AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Size animation
                Column(
                  children: [
                    const Text('Size Animation'),
                    Container(
                      width: _sizeAnimation.value,
                      height: _sizeAnimation.value,
                      color: Colors.blue,
                    ),
                  ],
                ),

                // Color animation
                Column(
                  children: [
                    const Text('Color Animation'),
                    Container(
                      width: 80,
                      height: 80,
                      color: _colorAnimation.value,
                    ),
                  ],
                ),

                // Rotation animation
                Column(
                  children: [
                    const Text('Rotation Animation'),
                    Transform.rotate(
                      angle: _rotationAnimation.value,
                      child: const Icon(Icons.star, size: 60, color: Colors.amber),
                    ),
                  ],
                ),

                // Slide animation
                Column(
                  children: [
                    const Text('Slide Animation'),
                    SlideTransition(
                      position: _slideAnimation,
                      child: Container(
                        width: 80,
                        height: 40,
                        color: Colors.green,
                        alignment: Alignment.center,
                        child: const Text('Slide', style: TextStyle(color: Colors.white)),
                      ),
                    ),
                  ],
                ),

                // Opacity animation
                Column(
                  children: [
                    const Text('Opacity (delay 50%)'),
                    FadeTransition(
                      opacity: _opacityAnimation,
                      child: Container(
                        width: 80,
                        height: 40,
                        color: Colors.purple,
                        alignment: Alignment.center,
                        child: const Text('Fade', style: TextStyle(color: Colors.white)),
                      ),
                    ),
                  ],
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

### Curves ที่ใช้บ่อย

```dart
// Curves ที่ควรรู้จัก
Curves.linear          // ความเร็วคงที่
Curves.easeIn          // เริ่มช้า เร็วขึ้น
Curves.easeOut         // เริ่มเร็ว ช้าลง
Curves.easeInOut       // ช้า เร็ว ช้า
Curves.bounceOut       // เด้งที่ปลาย
Curves.elasticOut      // ยืดหยุ่น
Curves.fastOutSlowIn   // แบบ Material Design
Curves.decelerate      // ชะลอ
```

---

## ขั้นตอนที่ 113: AnimatedWidget

`AnimatedWidget` ทำให้ Widget ฟัง Animation โดยตรง โดยไม่ต้องใช้ AnimatedBuilder

```dart
import 'package:flutter/material.dart';

// Custom AnimatedWidget
class SpinningBox extends AnimatedWidget {
  final Color color;
  final double size;

  const SpinningBox({
    super.key,
    required Animation<double> animation,
    this.color = Colors.blue,
    this.size = 100,
  }) : super(listenable: animation);

  Animation<double> get _animation => listenable as Animation<double>;

  @override
  Widget build(BuildContext context) {
    return Transform.rotate(
      angle: _animation.value * 2 * 3.14159,
      child: Container(
        width: size,
        height: size,
        decoration: BoxDecoration(
          color: color,
          borderRadius: BorderRadius.circular(_animation.value * size / 2),
        ),
        child: const Center(
          child: Icon(Icons.star, color: Colors.white),
        ),
      ),
    );
  }
}

// การใช้งาน
class AnimatedWidgetScreen extends StatefulWidget {
  const AnimatedWidgetScreen({super.key});

  @override
  State<AnimatedWidgetScreen> createState() => _AnimatedWidgetScreenState();
}

class _AnimatedWidgetScreenState extends State<AnimatedWidgetScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();

    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.linear,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedWidget')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            SpinningBox(animation: _animation, color: Colors.blue),
            SpinningBox(animation: _animation, color: Colors.red, size: 80),
            SpinningBox(animation: _animation, color: Colors.green, size: 60),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 114: AnimatedBuilder

`AnimatedBuilder` ยืดหยุ่นกว่า AnimatedWidget เพราะ build เฉพาะส่วนที่เปลี่ยนแปลง

```dart
import 'package:flutter/material.dart';

class AnimatedBuilderExample extends StatefulWidget {
  const AnimatedBuilderExample({super.key});

  @override
  State<AnimatedBuilderExample> createState() => _AnimatedBuilderExampleState();
}

class _AnimatedBuilderExampleState extends State<AnimatedBuilderExample>
    with TickerProviderStateMixin {
  late AnimationController _pulseController;
  late AnimationController _rotateController;
  late Animation<double> _pulseAnimation;
  late Animation<double> _rotateAnimation;

  @override
  void initState() {
    super.initState();
    
    _pulseController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    )..repeat(reverse: true);

    _rotateController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat();

    _pulseAnimation = Tween<double>(begin: 0.8, end: 1.2).animate(
      CurvedAnimation(parent: _pulseController, curve: Curves.easeInOut),
    );

    _rotateAnimation = Tween<double>(begin: 0, end: 2 * 3.14159).animate(
      _rotateController,
    );
  }

  @override
  void dispose() {
    _pulseController.dispose();
    _rotateController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedBuilder')),
      body: Center(
        child: Stack(
          alignment: Alignment.center,
          children: [
            // Rotating ring - ใช้ AnimatedBuilder
            AnimatedBuilder(
              animation: _rotateAnimation,
              builder: (context, child) {
                return Transform.rotate(
                  angle: _rotateAnimation.value,
                  child: child, // child ไม่ถูก rebuild เมื่อ animation เปลี่ยน
                );
              },
              // child ที่ไม่เปลี่ยนแปลง - ถูกสร้างครั้งเดียว
              child: Container(
                width: 150,
                height: 150,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  border: Border.all(
                    color: Colors.blue,
                    width: 4,
                  ),
                ),
              ),
            ),
            
            // Pulsing center - ใช้ AnimatedBuilder อีกตัว
            AnimatedBuilder(
              animation: _pulseAnimation,
              builder: (context, child) {
                return Transform.scale(
                  scale: _pulseAnimation.value,
                  child: child,
                );
              },
              child: Container(
                width: 80,
                height: 80,
                decoration: const BoxDecoration(
                  shape: BoxShape.circle,
                  gradient: RadialGradient(
                    colors: [Colors.orange, Colors.red],
                  ),
                ),
                child: const Center(
                  child: Text(
                    '❤️',
                    style: TextStyle(fontSize: 30),
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

## ขั้นตอนที่ 115: AnimatedContainer

`AnimatedContainer` คือ Container ที่ Animate การเปลี่ยนแปลง property อัตโนมัติ ไม่ต้องใช้ AnimationController

```dart
import 'package:flutter/material.dart';

class AnimatedContainerExample extends StatefulWidget {
  const AnimatedContainerExample({super.key});

  @override
  State<AnimatedContainerExample> createState() =>
      _AnimatedContainerExampleState();
}

class _AnimatedContainerExampleState extends State<AnimatedContainerExample> {
  bool _isExpanded = false;
  bool _isRound = false;
  Color _color = Colors.blue;

  double get _width => _isExpanded ? 300 : 100;
  double get _height => _isExpanded ? 200 : 100;
  double get _borderRadius => _isRound ? 50 : 8;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedContainer')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // AnimatedContainer - animate เมื่อ setState
            AnimatedContainer(
              duration: const Duration(milliseconds: 500),
              curve: Curves.easeInOut,
              width: _width,
              height: _height,
              decoration: BoxDecoration(
                color: _color,
                borderRadius: BorderRadius.circular(_borderRadius),
                boxShadow: [
                  BoxShadow(
                    color: _color.withOpacity(0.4),
                    blurRadius: _isExpanded ? 20 : 5,
                    spreadRadius: _isExpanded ? 5 : 0,
                  ),
                ],
              ),
              child: const Center(
                child: Icon(Icons.touch_app, color: Colors.white, size: 32),
              ),
            ),

            const SizedBox(height: 40),

            // ปุ่มควบคุม
            Wrap(
              spacing: 8,
              children: [
                ElevatedButton.icon(
                  onPressed: () => setState(() => _isExpanded = !_isExpanded),
                  icon: Icon(_isExpanded ? Icons.compress : Icons.expand),
                  label: Text(_isExpanded ? 'ย่อ' : 'ขยาย'),
                ),
                ElevatedButton.icon(
                  onPressed: () => setState(() => _isRound = !_isRound),
                  icon: Icon(_isRound ? Icons.square : Icons.circle),
                  label: Text(_isRound ? 'สี่เหลี่ยม' : 'วงกลม'),
                ),
              ],
            ),

            const SizedBox(height: 16),

            // เลือกสี
            Wrap(
              spacing: 8,
              children: [
                Colors.blue, Colors.red, Colors.green,
                Colors.orange, Colors.purple,
              ].map((color) => GestureDetector(
                onTap: () => setState(() => _color = color),
                child: Container(
                  width: 40,
                  height: 40,
                  decoration: BoxDecoration(
                    color: color,
                    shape: BoxShape.circle,
                    border: Border.all(
                      color: _color == color ? Colors.black : Colors.transparent,
                      width: 3,
                    ),
                  ),
                ),
              )).toList(),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 116: AnimatedOpacity และ AnimatedSwitcher

```dart
import 'package:flutter/material.dart';

class AnimatedOpacityExample extends StatefulWidget {
  const AnimatedOpacityExample({super.key});

  @override
  State<AnimatedOpacityExample> createState() => _AnimatedOpacityExampleState();
}

class _AnimatedOpacityExampleState extends State<AnimatedOpacityExample> {
  bool _isVisible = true;
  int _currentIndex = 0;

  final List<Widget> _widgets = [
    Container(
      key: ValueKey(0),
      width: 150, height: 150,
      color: Colors.blue,
      child: const Center(child: Text('Widget 1', style: TextStyle(color: Colors.white))),
    ),
    Container(
      key: ValueKey(1),
      width: 150, height: 150,
      color: Colors.red,
      child: const Center(child: Text('Widget 2', style: TextStyle(color: Colors.white))),
    ),
    Container(
      key: ValueKey(2),
      width: 150, height: 150,
      decoration: const BoxDecoration(
        color: Colors.green,
        shape: BoxShape.circle,
      ),
      child: const Center(child: Text('Widget 3', style: TextStyle(color: Colors.white))),
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedOpacity & AnimatedSwitcher')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            // AnimatedOpacity
            Column(
              children: [
                const Text('AnimatedOpacity', style: TextStyle(fontWeight: FontWeight.bold)),
                const SizedBox(height: 16),
                AnimatedOpacity(
                  opacity: _isVisible ? 1.0 : 0.0,
                  duration: const Duration(milliseconds: 500),
                  curve: Curves.easeInOut,
                  child: Container(
                    width: 100, height: 100,
                    color: Colors.purple,
                    child: const Center(
                      child: Text('Fade Me', style: TextStyle(color: Colors.white)),
                    ),
                  ),
                ),
                const SizedBox(height: 8),
                ElevatedButton(
                  onPressed: () => setState(() => _isVisible = !_isVisible),
                  child: Text(_isVisible ? 'ซ่อน' : 'แสดง'),
                ),
              ],
            ),

            const Divider(),

            // AnimatedSwitcher - switch ระหว่าง widget ด้วย animation
            Column(
              children: [
                const Text('AnimatedSwitcher', style: TextStyle(fontWeight: FontWeight.bold)),
                const SizedBox(height: 16),
                AnimatedSwitcher(
                  duration: const Duration(milliseconds: 400),
                  // กำหนด transition แบบ custom
                  transitionBuilder: (child, animation) {
                    return ScaleTransition(
                      scale: animation,
                      child: FadeTransition(opacity: animation, child: child),
                    );
                  },
                  child: _widgets[_currentIndex],
                ),
                const SizedBox(height: 8),
                ElevatedButton(
                  onPressed: () => setState(() {
                    _currentIndex = (_currentIndex + 1) % _widgets.length;
                  }),
                  child: const Text('เปลี่ยน Widget'),
                ),
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

## ขั้นตอนที่ 117: AnimatedCrossFade

`AnimatedCrossFade` สำหรับ cross-fade ระหว่าง 2 widget

```dart
import 'package:flutter/material.dart';

class AnimatedCrossFadeExample extends StatefulWidget {
  const AnimatedCrossFadeExample({super.key});

  @override
  State<AnimatedCrossFadeExample> createState() =>
      _AnimatedCrossFadeExampleState();
}

class _AnimatedCrossFadeExampleState extends State<AnimatedCrossFadeExample> {
  bool _showFirst = true;
  bool _isLoading = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedCrossFade')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // ตัวอย่าง 1: Toggle ระหว่าง 2 views
            AnimatedCrossFade(
              duration: const Duration(milliseconds: 500),
              firstCurve: Curves.easeOut,
              secondCurve: Curves.easeIn,
              crossFadeState: _showFirst
                  ? CrossFadeState.showFirst
                  : CrossFadeState.showSecond,
              firstChild: Card(
                child: Padding(
                  padding: const EdgeInsets.all(24),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: const [
                      Icon(Icons.login, size: 48, color: Colors.blue),
                      SizedBox(height: 8),
                      Text('เข้าสู่ระบบ',
                          style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                      SizedBox(height: 4),
                      Text('กรอกข้อมูลเพื่อเข้าสู่ระบบ'),
                    ],
                  ),
                ),
              ),
              secondChild: Card(
                child: Padding(
                  padding: const EdgeInsets.all(24),
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: const [
                      Icon(Icons.person_add, size: 48, color: Colors.green),
                      SizedBox(height: 8),
                      Text('สมัครสมาชิก',
                          style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                      SizedBox(height: 4),
                      Text('สร้างบัญชีใหม่'),
                    ],
                  ),
                ),
              ),
            ),

            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => setState(() => _showFirst = !_showFirst),
              child: Text(_showFirst ? 'สมัครสมาชิก' : 'เข้าสู่ระบบ'),
            ),

            const SizedBox(height: 32),
            const Divider(),
            const SizedBox(height: 16),

            // ตัวอย่าง 2: Loading state
            AnimatedCrossFade(
              duration: const Duration(milliseconds: 300),
              crossFadeState: _isLoading
                  ? CrossFadeState.showFirst
                  : CrossFadeState.showSecond,
              firstChild: const SizedBox(
                height: 60,
                child: Center(child: CircularProgressIndicator()),
              ),
              secondChild: SizedBox(
                height: 60,
                child: Center(
                  child: ElevatedButton(
                    onPressed: () async {
                      setState(() => _isLoading = true);
                      await Future.delayed(const Duration(seconds: 2));
                      setState(() => _isLoading = false);
                    },
                    child: const Text('กดเพื่อโหลด'),
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

## ขั้นตอนที่ 118: Hero Animations

`Hero` animation เชื่อม Widget ระหว่าง 2 หน้าจอด้วย animation ที่ลื่นไหล

```dart
import 'package:flutter/material.dart';

// Model
class HeroItem {
  final String id;
  final String title;
  final String description;
  final Color color;
  final IconData icon;

  const HeroItem({
    required this.id,
    required this.title,
    required this.description,
    required this.color,
    required this.icon,
  });
}

// List Screen
class HeroListScreen extends StatelessWidget {
  const HeroListScreen({super.key});

  static const items = [
    HeroItem(
      id: 'hero_1',
      title: 'Flutter',
      description: 'UI Toolkit จาก Google สำหรับสร้างแอปข้ามแพลตฟอร์ม',
      color: Colors.blue,
      icon: Icons.flutter_dash,
    ),
    HeroItem(
      id: 'hero_2',
      title: 'Dart',
      description: 'ภาษา Programming ที่ใช้ใน Flutter',
      color: Colors.teal,
      icon: Icons.code,
    ),
    HeroItem(
      id: 'hero_3',
      title: 'Firebase',
      description: 'Backend service จาก Google',
      color: Colors.orange,
      icon: Icons.local_fire_department,
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Hero Animation')),
      body: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
          childAspectRatio: 0.9,
        ),
        itemCount: items.length,
        itemBuilder: (context, index) {
          final item = items[index];
          return GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => HeroDetailScreen(item: item),
              ),
            ),
            child: Hero(
              tag: item.id, // tag ต้องไม่ซ้ำกันในแต่ละ Hero pair
              child: Card(
                elevation: 4,
                child: Container(
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(12),
                    gradient: LinearGradient(
                      colors: [item.color, item.color.withOpacity(0.6)],
                      begin: Alignment.topLeft,
                      end: Alignment.bottomRight,
                    ),
                  ),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(item.icon, size: 48, color: Colors.white),
                      const SizedBox(height: 8),
                      Text(
                        item.title,
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// Detail Screen
class HeroDetailScreen extends StatelessWidget {
  final HeroItem item;

  const HeroDetailScreen({super.key, required this.item});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: Text(item.title),
              background: Hero(
                tag: item.id, // ต้องใช้ tag เดิม
                child: Container(
                  decoration: BoxDecoration(
                    gradient: LinearGradient(
                      colors: [item.color, item.color.withOpacity(0.6)],
                      begin: Alignment.topLeft,
                      end: Alignment.bottomRight,
                    ),
                  ),
                  child: Center(
                    child: Icon(item.icon, size: 100, color: Colors.white),
                  ),
                ),
              ),
            ),
          ),
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    item.title,
                    style: Theme.of(context).textTheme.headlineMedium?.copyWith(
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    item.description,
                    style: Theme.of(context).textTheme.bodyLarge,
                  ),
                  const SizedBox(height: 24),
                  // เพิ่ม content ตัวอย่าง
                  ...List.generate(
                    5,
                    (i) => Padding(
                      padding: const EdgeInsets.symmetric(vertical: 8),
                      child: Text('ข้อมูลเพิ่มเติม ${i + 1}: รายละเอียดเกี่ยวกับ ${item.title}'),
                    ),
                  ),
                ],
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

## ขั้นตอนที่ 119: TweenAnimationBuilder

`TweenAnimationBuilder` ช่วยสร้าง animation แบบง่ายโดยไม่ต้องจัดการ AnimationController เอง

```dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

class TweenAnimationBuilderExample extends StatefulWidget {
  const TweenAnimationBuilderExample({super.key});

  @override
  State<TweenAnimationBuilderExample> createState() =>
      _TweenAnimationBuilderExampleState();
}

class _TweenAnimationBuilderExampleState
    extends State<TweenAnimationBuilderExample> {
  double _targetValue = 0;
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('TweenAnimationBuilder')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            // ตัวอย่าง 1: Progress indicator แบบ custom
            TweenAnimationBuilder<double>(
              tween: Tween<double>(begin: 0, end: _targetValue),
              duration: const Duration(milliseconds: 800),
              curve: Curves.easeOut,
              builder: (context, value, child) {
                return Column(
                  children: [
                    const Text('Progress'),
                    const SizedBox(height: 8),
                    SizedBox(
                      width: 100,
                      height: 100,
                      child: Stack(
                        alignment: Alignment.center,
                        children: [
                          // เส้นรอบวง
                          CustomPaint(
                            size: const Size(100, 100),
                            painter: _CircularProgressPainter(
                              progress: value,
                              color: Colors.blue,
                            ),
                          ),
                          Text(
                            '${(value * 100).toInt()}%',
                            style: const TextStyle(
                              fontSize: 18,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ],
                      ),
                    ),
                    const SizedBox(height: 8),
                    Slider(
                      value: _targetValue,
                      onChanged: (v) => setState(() => _targetValue = v),
                    ),
                  ],
                );
              },
            ),

            const Divider(),

            // ตัวอย่าง 2: Counter animation
            Column(
              children: [
                const Text('Animated Counter'),
                TweenAnimationBuilder<int>(
                  tween: IntTween(begin: 0, end: _counter),
                  duration: const Duration(milliseconds: 500),
                  curve: Curves.easeOut,
                  builder: (context, value, child) {
                    return Text(
                      '$value',
                      style: const TextStyle(
                        fontSize: 60,
                        fontWeight: FontWeight.bold,
                        color: Colors.blue,
                      ),
                    );
                  },
                ),
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    IconButton(
                      icon: const Icon(Icons.remove_circle, color: Colors.red),
                      iconSize: 36,
                      onPressed: () => setState(() => _counter--),
                    ),
                    const SizedBox(width: 16),
                    IconButton(
                      icon: const Icon(Icons.add_circle, color: Colors.green),
                      iconSize: 36,
                      onPressed: () => setState(() => _counter++),
                    ),
                  ],
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

class _CircularProgressPainter extends CustomPainter {
  final double progress;
  final Color color;

  _CircularProgressPainter({required this.progress, required this.color});

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 8;

    // Background circle
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;
    canvas.drawCircle(center, radius, bgPaint);

    // Progress arc
    final progressPaint = Paint()
      ..color = color
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8
      ..strokeCap = StrokeCap.round;
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2, // เริ่มจากด้านบน
      2 * math.pi * progress,
      false,
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(_CircularProgressPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}
```

---

## ขั้นตอนที่ 120: Staggered Animations

Staggered Animation คือการเล่น animation หลายตัวต่อเนื่องกันด้วย delay

```dart
import 'package:flutter/material.dart';

class StaggeredAnimationScreen extends StatefulWidget {
  const StaggeredAnimationScreen({super.key});

  @override
  State<StaggeredAnimationScreen> createState() =>
      _StaggeredAnimationScreenState();
}

class _StaggeredAnimationScreenState extends State<StaggeredAnimationScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late List<Animation<double>> _itemAnimations;

  final int _itemCount = 6;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1500),
    );

    // สร้าง animation สำหรับแต่ละ item ด้วย Interval ที่ต่างกัน
    _itemAnimations = List.generate(_itemCount, (index) {
      final start = index / _itemCount;
      final end = (index + 1) / _itemCount;

      return Tween<double>(begin: 0, end: 1).animate(
        CurvedAnimation(
          parent: _controller,
          curve: Interval(start, end, curve: Curves.easeOut),
        ),
      );
    });

    // เริ่ม animation หลัง build
    Future.delayed(const Duration(milliseconds: 300), () {
      _controller.forward();
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Staggered Animations'),
        actions: [
          IconButton(
            icon: const Icon(Icons.replay),
            onPressed: () {
              _controller.reset();
              _controller.forward();
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: List.generate(_itemCount, (index) {
            return AnimatedBuilder(
              animation: _itemAnimations[index],
              builder: (context, child) {
                final value = _itemAnimations[index].value;
                return Opacity(
                  opacity: value,
                  child: Transform.translate(
                    offset: Offset(0, 50 * (1 - value)), // ขยับขึ้นจากด้านล่าง
                    child: child,
                  ),
                );
              },
              child: Card(
                margin: const EdgeInsets.symmetric(vertical: 6),
                child: ListTile(
                  leading: CircleAvatar(
                    backgroundColor: HSLColor.fromAHSL(
                      1.0, index * 60.0, 0.7, 0.5
                    ).toColor(),
                    child: Text('${index + 1}', style: const TextStyle(color: Colors.white)),
                  ),
                  title: Text('รายการ ${index + 1}'),
                  subtitle: Text('ปรากฏด้วย delay ${(index * 250).toInt()} ms'),
                  trailing: const Icon(Icons.arrow_forward_ios, size: 16),
                ),
              ),
            );
          }),
        ),
      ),
    );
  }
}
```

---

## Workshop: Animated Dashboard

```dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

class AnimatedDashboardApp extends StatelessWidget {
  const AnimatedDashboardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Animated Dashboard',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const DashboardScreen(),
    );
  }
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late List<Animation<double>> _cardAnimations;

  final List<Map<String, dynamic>> _stats = [
    {'title': 'ยอดขาย', 'value': 85432, 'icon': Icons.attach_money, 'color': Colors.green, 'change': '+12%'},
    {'title': 'ผู้ใช้ใหม่', 'value': 1284, 'icon': Icons.people, 'color': Colors.blue, 'change': '+8%'},
    {'title': 'ออเดอร์', 'value': 352, 'icon': Icons.shopping_bag, 'color': Colors.orange, 'change': '+5%'},
    {'title': 'รีวิว', 'value': 4.8, 'icon': Icons.star, 'color': Colors.amber, 'change': '+0.2'},
  ];

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    );

    _cardAnimations = List.generate(
      _stats.length + 2, // +2 สำหรับ chart และ list
      (i) => CurvedAnimation(
        parent: _controller,
        curve: Interval(
          i * 0.1,
          math.min(i * 0.1 + 0.4, 1.0),
          curve: Curves.easeOut,
        ),
      ),
    );

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
        title: const Text('Dashboard'),
        backgroundColor: Colors.indigo,
        foregroundColor: Colors.white,
        actions: [
          IconButton(
            icon: const Icon(Icons.notifications_outlined),
            onPressed: () {},
          ),
          const CircleAvatar(
            radius: 16,
            child: Icon(Icons.person, size: 18),
          ),
          const SizedBox(width: 8),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              'ภาพรวมธุรกิจ',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),

            // Stat cards
            GridView.builder(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
                crossAxisSpacing: 12,
                mainAxisSpacing: 12,
                childAspectRatio: 1.4,
              ),
              itemCount: _stats.length,
              itemBuilder: (context, index) {
                return _buildAnimatedCard(index);
              },
            ),

            const SizedBox(height: 24),

            // Chart section
            _buildSectionWithAnimation(
              animationIndex: 4,
              child: _buildChartCard(),
            ),

            const SizedBox(height: 16),

            // Recent transactions
            _buildSectionWithAnimation(
              animationIndex: 5,
              child: _buildRecentTransactions(),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildAnimatedCard(int index) {
    final stat = _stats[index];
    return AnimatedBuilder(
      animation: _cardAnimations[index],
      builder: (context, child) {
        return Opacity(
          opacity: _cardAnimations[index].value,
          child: Transform.translate(
            offset: Offset(0, 30 * (1 - _cardAnimations[index].value)),
            child: child,
          ),
        );
      },
      child: Card(
        elevation: 2,
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Icon(stat['icon'] as IconData, color: stat['color'] as Color),
                  Container(
                    padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                    decoration: BoxDecoration(
                      color: Colors.green[50],
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: Text(
                      stat['change'] as String,
                      style: const TextStyle(
                        color: Colors.green,
                        fontSize: 11,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ],
              ),
              Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  TweenAnimationBuilder<double>(
                    tween: Tween(begin: 0, end: (stat['value'] as num).toDouble()),
                    duration: const Duration(milliseconds: 1500),
                    curve: Curves.easeOut,
                    builder: (context, value, _) {
                      String display;
                      if (value > 1000) {
                        display = '${(value / 1000).toStringAsFixed(1)}K';
                      } else if (value < 10) {
                        display = value.toStringAsFixed(1);
                      } else {
                        display = value.toInt().toString();
                      }
                      return Text(
                        display,
                        style: TextStyle(
                          fontSize: 22,
                          fontWeight: FontWeight.bold,
                          color: stat['color'] as Color,
                        ),
                      );
                    },
                  ),
                  Text(
                    stat['title'] as String,
                    style: const TextStyle(color: Colors.grey, fontSize: 12),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildSectionWithAnimation({
    required int animationIndex,
    required Widget child,
  }) {
    return AnimatedBuilder(
      animation: _cardAnimations[animationIndex],
      builder: (context, _) {
        return Opacity(
          opacity: _cardAnimations[animationIndex].value,
          child: Transform.translate(
            offset: Offset(0, 30 * (1 - _cardAnimations[animationIndex].value)),
            child: child,
          ),
        );
      },
    );
  }

  Widget _buildChartCard() {
    return Card(
      elevation: 2,
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('ยอดขายรายสัปดาห์',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            SizedBox(
              height: 150,
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.end,
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  for (final entry in [
                    ('จ', 0.6), ('อ', 0.8), ('พ', 0.5), ('พฤ', 0.9),
                    ('ศ', 0.7), ('ส', 0.4), ('อา', 0.85),
                  ])
                    TweenAnimationBuilder<double>(
                      tween: Tween(begin: 0, end: entry.$2),
                      duration: const Duration(milliseconds: 1000),
                      curve: Curves.easeOut,
                      builder: (context, value, _) {
                        return Column(
                          mainAxisAlignment: MainAxisAlignment.end,
                          children: [
                            Container(
                              width: 30,
                              height: 120 * value,
                              decoration: BoxDecoration(
                                color: Colors.indigo,
                                borderRadius: BorderRadius.circular(4),
                              ),
                            ),
                            const SizedBox(height: 4),
                            Text(entry.$1, style: const TextStyle(fontSize: 12)),
                          ],
                        );
                      },
                    ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildRecentTransactions() {
    final transactions = [
      ('ซื้อ iPhone 15', '฿45,900', Colors.red, Icons.arrow_upward),
      ('รับเงิน', '฿10,000', Colors.green, Icons.arrow_downward),
      ('ค่าอาหาร', '฿350', Colors.red, Icons.arrow_upward),
      ('โบนัส', '฿5,000', Colors.green, Icons.arrow_downward),
    ];

    return Card(
      elevation: 2,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('รายการล่าสุด',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),
          ...transactions.asMap().entries.map((entry) {
            final i = entry.key;
            final t = entry.value;
            return TweenAnimationBuilder<double>(
              tween: Tween(begin: 0, end: 1),
              duration: Duration(milliseconds: 400 + i * 100),
              curve: Curves.easeOut,
              builder: (context, value, child) {
                return Opacity(
                  opacity: value,
                  child: child,
                );
              },
              child: ListTile(
                leading: CircleAvatar(
                  backgroundColor: t.$3.withOpacity(0.1),
                  child: Icon(t.$4, color: t.$3, size: 20),
                ),
                title: Text(t.$1),
                trailing: Text(
                  t.$2,
                  style: TextStyle(
                    color: t.$3,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            );
          }),
        ],
      ),
    );
  }
}

void main() => runApp(const AnimatedDashboardApp());
```

---

## สรุป (Summary)

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| AnimationController | ตัวควบคุม animation หลัก |
| Tween | แปลงค่า 0-1 เป็นค่าที่ต้องการ |
| CurvedAnimation | ใส่ easing curve ให้ animation |
| AnimatedWidget | Widget ที่ฟัง animation โดยตรง |
| AnimatedBuilder | rebuild เฉพาะส่วนที่เปลี่ยน |
| AnimatedContainer | implicit animation สำหรับ Container |
| AnimatedOpacity | fade in/out |
| AnimatedSwitcher | switch ระหว่าง widget |
| AnimatedCrossFade | cross-fade 2 widget |
| Hero | animation ระหว่าง 2 screen |
| TweenAnimationBuilder | animation ง่ายๆ ไม่ต้องจัดการ controller |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง button ที่ขยาย/ย่อตัวด้วย AnimatedContainer เมื่อกด
2. **ปานกลาง**: สร้าง login screen ที่มี shake animation เมื่อรหัสผ่านผิด
3. **ยาก**: สร้าง card list ที่มี staggered animation เมื่อ scroll เข้ามา

---

[← Part 11: ListView, GridView, Slivers](part_11.md) | [Part 13: Gestures →](part_13.md)

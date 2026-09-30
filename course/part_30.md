# Part 30: Advanced Animations
## ขั้นตอนที่ 291-300

---

## สารบัญ
1. [AnimationController Lifecycle](#animationcontroller-lifecycle)
2. [Staggered Animations](#staggered-animations)
3. [Physics-Based Animations](#physics-based-animations)
4. [Rive Animations](#rive-animations)
5. [Lottie Animations](#lottie-animations)
6. [CustomPainter Animations](#custompaainter-animations)
7. [Page Transitions](#page-transitions)
8. [Drag Animations](#drag-animations)
9. [Spring & Bounce Effects](#spring--bounce-effects)
10. [Hero Animations & Shared Elements](#hero-animations--shared-elements)

---

## ขั้นตอนที่ 291: AnimationController Lifecycle

AnimationController เป็นหัวใจของ Animation System ใน Flutter

```dart
// lib/animations/animation_controller_demo.dart
import 'package:flutter/material.dart';

class AnimationControllerDemo extends StatefulWidget {
  const AnimationControllerDemo({super.key});

  @override
  State<AnimationControllerDemo> createState() =>
      _AnimationControllerDemoState();
}

class _AnimationControllerDemoState extends State<AnimationControllerDemo>
    with TickerProviderStateMixin {

  // Controller หลัก
  late final AnimationController _controller;
  late final AnimationController _rotateController;

  // Animations ต่างๆ
  late final Animation<double> _scaleAnimation;
  late final Animation<double> _opacityAnimation;
  late final Animation<Offset> _slideAnimation;
  late final Animation<Color?> _colorAnimation;

  @override
  void initState() {
    super.initState();

    // สร้าง Controller
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );

    _rotateController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    );

    // Scale Animation
    _scaleAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );

    // Opacity Animation
    _opacityAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeIn),
    );

    // Slide Animation
    _slideAnimation = Tween<Offset>(
      begin: const Offset(0, 1),
      end: Offset.zero,
    ).animate(CurvedAnimation(parent: _controller, curve: Curves.easeOutBack));

    // Color Animation
    _colorAnimation = ColorTween(
      begin: Colors.blue,
      end: Colors.purple,
    ).animate(_controller);

    // Listeners
    _controller.addListener(() {
      // เรียกทุกครั้งที่ค่า Animation เปลี่ยน
    });

    _controller.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        print('Animation completed');
      } else if (status == AnimationStatus.dismissed) {
        print('Animation dismissed');
      }
    });

    // เริ่ม Rotation ทันที
    _rotateController.repeat();
  }

  @override
  void dispose() {
    _controller.dispose();
    _rotateController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animation Controller')),
      body: AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Scale + Opacity
                FadeTransition(
                  opacity: _opacityAnimation,
                  child: ScaleTransition(
                    scale: _scaleAnimation,
                    child: Container(
                      width: 150,
                      height: 150,
                      decoration: BoxDecoration(
                        color: _colorAnimation.value ?? Colors.blue,
                        borderRadius: BorderRadius.circular(20),
                        boxShadow: [
                          BoxShadow(
                            color: (_colorAnimation.value ?? Colors.blue)
                                .withOpacity(0.4),
                            blurRadius: 20,
                            spreadRadius: 5,
                          ),
                        ],
                      ),
                      child: const Icon(
                        Icons.star,
                        color: Colors.white,
                        size: 60,
                      ),
                    ),
                  ),
                ),
                const SizedBox(height: 32),

                // Slide
                SlideTransition(
                  position: _slideAnimation,
                  child: const Text(
                    'Hello Animation!',
                    style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                  ),
                ),
                const SizedBox(height: 32),

                // Rotation
                RotationTransition(
                  turns: _rotateController,
                  child: const Icon(Icons.settings, size: 48),
                ),
                const SizedBox(height: 40),

                // Controls
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    ElevatedButton(
                      onPressed: _controller.forward,
                      child: const Text('Play'),
                    ),
                    ElevatedButton(
                      onPressed: _controller.reverse,
                      child: const Text('Reverse'),
                    ),
                    ElevatedButton(
                      onPressed: () => _controller.repeat(reverse: true),
                      child: const Text('Loop'),
                    ),
                    ElevatedButton(
                      onPressed: _controller.stop,
                      child: const Text('Stop'),
                    ),
                  ],
                ),

                const SizedBox(height: 16),

                // Progress Bar
                Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 40),
                  child: LinearProgressIndicator(
                    value: _controller.value,
                  ),
                ),
                Text(
                  '${(_controller.value * 100).toInt()}%',
                  style: const TextStyle(fontSize: 16),
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

---

## ขั้นตอนที่ 292: Staggered Animations

Staggered Animations คือ Animation หลายตัวที่เริ่มในเวลาต่างกัน

```dart
// lib/animations/staggered_animation.dart
import 'package:flutter/material.dart';

class StaggeredAnimationPage extends StatefulWidget {
  const StaggeredAnimationPage({super.key});

  @override
  State<StaggeredAnimationPage> createState() => _StaggeredAnimationPageState();
}

class _StaggeredAnimationPageState extends State<StaggeredAnimationPage>
    with SingleTickerProviderStateMixin {

  late final AnimationController _controller;
  late final List<Animation<double>> _itemAnimations;

  final int _itemCount = 6;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    );

    // สร้าง Animation สำหรับแต่ละ Item
    // แต่ละตัวเริ่มช้าลง 0.1 interval
    _itemAnimations = List.generate(_itemCount, (index) {
      final start = index * 0.1;
      final end = start + 0.4;

      return Tween<double>(begin: 0.0, end: 1.0).animate(
        CurvedAnimation(
          parent: _controller,
          curve: Interval(start, end.clamp(0.0, 1.0), curve: Curves.easeOut),
        ),
      );
    });

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
      appBar: AppBar(title: const Text('Staggered Animations')),
      body: AnimatedBuilder(
        animation: _controller,
        builder: (context, _) {
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  padding: const EdgeInsets.all(16),
                  itemCount: _itemCount,
                  itemBuilder: (context, index) {
                    return FadeTransition(
                      opacity: _itemAnimations[index],
                      child: SlideTransition(
                        position: Tween<Offset>(
                          begin: const Offset(1, 0),
                          end: Offset.zero,
                        ).animate(_itemAnimations[index]),
                        child: Card(
                          margin: const EdgeInsets.only(bottom: 12),
                          child: ListTile(
                            leading: CircleAvatar(
                              child: Text('${index + 1}'),
                            ),
                            title: Text('Item ${index + 1}'),
                            subtitle: const Text('Staggered animation'),
                          ),
                        ),
                      ),
                    );
                  },
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  children: [
                    Expanded(
                      child: ElevatedButton(
                        onPressed: () {
                          _controller.reset();
                          _controller.forward();
                        },
                        child: const Text('Replay'),
                      ),
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

// Staggered Grid Animation
class StaggeredGridPage extends StatefulWidget {
  const StaggeredGridPage({super.key});

  @override
  State<StaggeredGridPage> createState() => _StaggeredGridPageState();
}

class _StaggeredGridPageState extends State<StaggeredGridPage>
    with SingleTickerProviderStateMixin {

  late final AnimationController _controller;
  final _colors = [
    Colors.red, Colors.blue, Colors.green, Colors.orange,
    Colors.purple, Colors.pink, Colors.teal, Colors.cyan,
    Colors.amber, Colors.indigo, Colors.lime, Colors.brown,
  ];

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 2000),
    )..forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Staggered Grid')),
      body: AnimatedBuilder(
        animation: _controller,
        builder: (context, _) {
          return GridView.builder(
            padding: const EdgeInsets.all(8),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
            ),
            itemCount: _colors.length,
            itemBuilder: (context, index) {
              final delay = index * 0.06;
              final start = delay.clamp(0.0, 1.0);
              final end = (delay + 0.4).clamp(0.0, 1.0);

              final animation = Tween<double>(begin: 0.0, end: 1.0).animate(
                CurvedAnimation(
                  parent: _controller,
                  curve: Interval(start, end, curve: Curves.elasticOut),
                ),
              );

              return ScaleTransition(
                scale: animation,
                child: Container(
                  decoration: BoxDecoration(
                    color: _colors[index],
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Center(
                    child: Text(
                      '${index + 1}',
                      style: const TextStyle(
                        color: Colors.white,
                        fontSize: 24,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _controller.reset();
          _controller.forward();
        },
        child: const Icon(Icons.replay),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 293: Physics-Based Animations

Physics Simulation ทำให้ Animation ดูเป็นธรรมชาติ

```dart
// lib/animations/physics_animation.dart
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

// Spring Simulation
class SpringAnimationPage extends StatefulWidget {
  const SpringAnimationPage({super.key});

  @override
  State<SpringAnimationPage> createState() => _SpringAnimationPageState();
}

class _SpringAnimationPageState extends State<SpringAnimationPage>
    with SingleTickerProviderStateMixin {

  late AnimationController _controller;
  late Animation<double> _animation;
  double _dragPosition = 0;
  double _dragStart = 0;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController.unbounded(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _runSpringSimulation(double position, double velocity) {
    final spring = SpringDescription(
      mass: 1,
      stiffness: 100,
      damping: 10,
    );

    final simulation = SpringSimulation(spring, position, 0, velocity);

    _controller.animateWith(simulation);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Spring Animation')),
      body: Center(
        child: GestureDetector(
          onPanStart: (details) {
            _controller.stop();
            _dragStart = details.localPosition.dy;
          },
          onPanUpdate: (details) {
            setState(() {
              _dragPosition = details.localPosition.dy - _dragStart;
              _controller.value = _dragPosition;
            });
          },
          onPanEnd: (details) {
            _runSpringSimulation(
              _controller.value,
              details.velocity.pixelsPerSecond.dy,
            );
          },
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Transform.translate(
                offset: Offset(0, _controller.value),
                child: child,
              );
            },
            child: Container(
              width: 100,
              height: 100,
              decoration: BoxDecoration(
                color: Colors.blue,
                shape: BoxShape.circle,
                boxShadow: [
                  BoxShadow(
                    color: Colors.blue.withOpacity(0.4),
                    blurRadius: 20,
                    spreadRadius: 5,
                  ),
                ],
              ),
              child: const Icon(Icons.touch_app, color: Colors.white, size: 40),
            ),
          ),
        ),
      ),
    );
  }
}

// Gravity Simulation
class GravityAnimationPage extends StatefulWidget {
  const GravityAnimationPage({super.key});

  @override
  State<GravityAnimationPage> createState() => _GravityAnimationPageState();
}

class _GravityAnimationPageState extends State<GravityAnimationPage>
    with SingleTickerProviderStateMixin {

  late AnimationController _controller;
  Offset _ballPosition = const Offset(150, 0);
  Offset _velocity = Offset.zero;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 10),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _throwBall(Offset position, Offset velocity) {
    _ballPosition = position;
    _velocity = velocity;
    _controller.reset();
    _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Gravity Simulation')),
      body: GestureDetector(
        onTapUp: (details) {
          _throwBall(
            details.localPosition,
            const Offset(-100, -500), // initial velocity
          );
        },
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, _) {
            final t = _controller.value * 2;
            final gravity = 980.0;
            final x = _ballPosition.dx + _velocity.dx * t;
            final y = _ballPosition.dy +
                _velocity.dy * t +
                0.5 * gravity * t * t;

            return Stack(
              children: [
                // Background
                Container(
                  color: Colors.grey[100],
                  child: const Center(
                    child: Text(
                      'แตะเพื่อโยนลูกบอล',
                      style: TextStyle(color: Colors.grey, fontSize: 18),
                    ),
                  ),
                ),
                // Ball
                if (_controller.isAnimating)
                  Positioned(
                    left: x - 20,
                    top: y - 20,
                    child: Container(
                      width: 40,
                      height: 40,
                      decoration: const BoxDecoration(
                        color: Colors.red,
                        shape: BoxShape.circle,
                      ),
                    ),
                  ),
              ],
            );
          },
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 294: Rive Animations

Rive คือ Animation Tool ที่ทรงพลังสำหรับ Flutter

```yaml
# pubspec.yaml
dependencies:
  rive: ^0.12.0
```

```dart
// lib/animations/rive_demo.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:rive/rive.dart';

class RiveDemoPage extends StatefulWidget {
  const RiveDemoPage({super.key});

  @override
  State<RiveDemoPage> createState() => _RiveDemoPageState();
}

class _RiveDemoPageState extends State<RiveDemoPage> {
  // State Machine
  StateMachineController? _controller;
  SMITrigger? _triggerAnimation;
  SMIBool? _isHovered;
  SMINumber? _progress;

  // Simple Animation
  RiveAnimationController? _simpleController;

  Artboard? _artboard;

  @override
  void initState() {
    super.initState();
    _loadRiveFile();
  }

  Future<void> _loadRiveFile() async {
    // โหลดไฟล์ Rive จาก Assets
    try {
      final data =
          await rootBundle.load('assets/animations/my_animation.riv');
      final file = RiveFile.import(data);
      final artboard = file.mainArtboard;

      // ใช้ State Machine
      final controller = StateMachineController.fromArtboard(
        artboard,
        'Main State Machine',
      );

      if (controller != null) {
        artboard.addController(controller);

        // ดึง Inputs
        _triggerAnimation =
            controller.findInput<bool>('trigger') as SMITrigger?;
        _isHovered = controller.findInput<bool>('isHovered') as SMIBool?;
        _progress = controller.findInput<double>('progress') as SMINumber?;

        _controller = controller;
      }

      setState(() => _artboard = artboard);
    } catch (e) {
      debugPrint('Error loading Rive: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Rive Animations')),
      body: Column(
        children: [
          // ใช้ RiveAnimation Widget (วิธีง่าย)
          SizedBox(
            height: 200,
            child: RiveAnimation.asset(
              'assets/animations/my_animation.riv',
              animations: const ['idle'],
              onInit: (artboard) {
                final controller =
                    SimpleAnimation('idle');
                artboard.addController(controller);
              },
            ),
          ),

          const SizedBox(height: 24),

          // Custom Artboard
          if (_artboard != null)
            Expanded(
              child: GestureDetector(
                onTap: () => _triggerAnimation?.fire(),
                onLongPress: () => _isHovered?.value = true,
                onLongPressEnd: (_) => _isHovered?.value = false,
                child: Rive(
                  artboard: _artboard!,
                  fit: BoxFit.contain,
                ),
              ),
            ),

          // Progress Control
          if (_progress != null)
            Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                children: [
                  const Text('Progress Control'),
                  Slider(
                    value: (_progress!.value / 100).clamp(0.0, 1.0),
                    onChanged: (v) {
                      setState(() {
                        _progress!.value = v * 100;
                      });
                    },
                  ),
                ],
              ),
            ),

          Padding(
            padding: const EdgeInsets.all(16),
            child: Wrap(
              spacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () => _triggerAnimation?.fire(),
                  child: const Text('Trigger'),
                ),
                ElevatedButton(
                  onPressed: () {
                    if (_isHovered != null) {
                      _isHovered!.value = !_isHovered!.value;
                    }
                  },
                  child: const Text('Toggle Hover'),
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

## ขั้นตอนที่ 295: Lottie Animations

Lottie ใช้ไฟล์ JSON จาก After Effects

```yaml
# pubspec.yaml
dependencies:
  lottie: ^3.0.0
```

```dart
// lib/animations/lottie_demo.dart
import 'package:flutter/material.dart';
import 'package:lottie/lottie.dart';

class LottieDemoPage extends StatefulWidget {
  const LottieDemoPage({super.key});

  @override
  State<LottieDemoPage> createState() => _LottieDemoPageState();
}

class _LottieDemoPageState extends State<LottieDemoPage>
    with TickerProviderStateMixin {

  late AnimationController _controller;
  bool _isPlaying = true;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Lottie Animations')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // จาก Asset
          const Text(
            'จาก Asset File',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Lottie.asset(
            'assets/animations/loading.json',
            width: 200,
            height: 200,
            fit: BoxFit.contain,
            repeat: true,
            animate: _isPlaying,
            controller: _controller,
            onLoaded: (composition) {
              _controller.duration = composition.duration;
              _controller.forward();
              _controller.repeat();
            },
          ),

          const SizedBox(height: 24),

          // จาก Network
          const Text(
            'จาก Network',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Lottie.network(
            'https://assets3.lottiefiles.com/packages/lf20_UJNc2t.json',
            width: 200,
            height: 200,
            fit: BoxFit.contain,
            errorBuilder: (context, error, stackTrace) {
              return Container(
                width: 200,
                height: 200,
                color: Colors.grey[100],
                child: const Center(
                  child: Icon(Icons.error, color: Colors.red),
                ),
              );
            },
          ),

          const SizedBox(height: 24),

          // Success Animation
          _LottieButton(
            label: 'แสดง Success',
            animationPath: 'assets/animations/success.json',
          ),

          const SizedBox(height: 12),

          // Error Animation
          _LottieButton(
            label: 'แสดง Error',
            animationPath: 'assets/animations/error.json',
          ),

          const SizedBox(height: 24),

          // Controls
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              ElevatedButton(
                onPressed: () {
                  setState(() => _isPlaying = true);
                  _controller.repeat();
                },
                child: const Text('Play'),
              ),
              ElevatedButton(
                onPressed: () {
                  setState(() => _isPlaying = false);
                  _controller.stop();
                },
                child: const Text('Pause'),
              ),
              ElevatedButton(
                onPressed: () {
                  _controller.reset();
                  _controller.forward();
                },
                child: const Text('Reset'),
              ),
            ],
          ),

          const SizedBox(height: 16),
          Text(
            'Speed: ${_controller.value.toStringAsFixed(2)}',
            textAlign: TextAlign.center,
          ),
          Slider(
            value: 1.0,
            min: 0.1,
            max: 3.0,
            onChanged: (v) {
              // ควบคุมความเร็ว
            },
          ),
        ],
      ),
    );
  }
}

class _LottieButton extends StatefulWidget {
  const _LottieButton({
    required this.label,
    required this.animationPath,
  });

  final String label;
  final String animationPath;

  @override
  State<_LottieButton> createState() => _LottieButtonState();
}

class _LottieButtonState extends State<_LottieButton>
    with SingleTickerProviderStateMixin {

  late AnimationController _controller;
  bool _showAnimation = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        if (_showAnimation)
          Lottie.asset(
            widget.animationPath,
            width: 150,
            height: 150,
            controller: _controller,
            onLoaded: (composition) {
              _controller.duration = composition.duration;
              _controller.forward().then((_) {
                setState(() => _showAnimation = false);
              });
            },
          ),
        ElevatedButton(
          onPressed: () {
            _controller.reset();
            setState(() => _showAnimation = true);
          },
          child: Text(widget.label),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 296: CustomPainter Animations

CustomPainter ให้อิสระในการวาด Animation

```dart
// lib/animations/custom_painter_animation.dart
import 'dart:math' as math;
import 'package:flutter/material.dart';

// Animated Wave Painter
class WavePainterPage extends StatefulWidget {
  const WavePainterPage({super.key});

  @override
  State<WavePainterPage> createState() => _WavePainterPageState();
}

class _WavePainterPageState extends State<WavePainterPage>
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
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Painter Animation')),
      body: Column(
        children: [
          // Wave
          AnimatedBuilder(
            animation: _controller,
            builder: (context, _) {
              return CustomPaint(
                painter: WavePainter(
                  animationValue: _controller.value,
                  color: Colors.blue.withOpacity(0.6),
                ),
                size: const Size(double.infinity, 200),
              );
            },
          ),

          const SizedBox(height: 24),

          // Loading Spinner
          AnimatedBuilder(
            animation: _controller,
            builder: (context, _) {
              return CustomPaint(
                painter: CircleLoadingPainter(
                  progress: _controller.value,
                ),
                size: const Size(100, 100),
              );
            },
          ),

          const SizedBox(height: 24),

          // Radar
          AnimatedBuilder(
            animation: _controller,
            builder: (context, _) {
              return CustomPaint(
                painter: RadarPainter(
                  angle: _controller.value * math.pi * 2,
                ),
                size: const Size(200, 200),
              );
            },
          ),
        ],
      ),
    );
  }
}

class WavePainter extends CustomPainter {
  const WavePainter({required this.animationValue, required this.color});

  final double animationValue;
  final Color color;

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    final path = Path();
    final waveHeight = 20.0;
    final waveCount = 2.0;

    path.moveTo(0, size.height / 2);

    for (var x = 0.0; x <= size.width; x++) {
      final y = size.height / 2 +
          waveHeight *
              math.sin(
                (x / size.width * math.pi * 2 * waveCount) +
                    (animationValue * math.pi * 2),
              );
      path.lineTo(x, y);
    }

    path.lineTo(size.width, size.height);
    path.lineTo(0, size.height);
    path.close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(WavePainter oldDelegate) =>
      oldDelegate.animationValue != animationValue;
}

class CircleLoadingPainter extends CustomPainter {
  const CircleLoadingPainter({required this.progress});

  final double progress;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2;
    final startAngle = -math.pi / 2;
    final sweepAngle = math.pi * 2 * progress;

    // Background circle
    final bgPaint = Paint()
      ..color = Colors.grey[200]!
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8;
    canvas.drawCircle(center, radius - 4, bgPaint);

    // Progress arc
    final progressPaint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8
      ..strokeCap = StrokeCap.round;
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius - 4),
      startAngle,
      sweepAngle,
      false,
      progressPaint,
    );

    // Text
    final textPainter = TextPainter(
      text: TextSpan(
        text: '${(progress * 100).toInt()}%',
        style: const TextStyle(color: Colors.black87, fontSize: 18),
      ),
      textDirection: TextDirection.ltr,
    )..layout();
    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );
  }

  @override
  bool shouldRepaint(CircleLoadingPainter oldDelegate) =>
      oldDelegate.progress != progress;
}

class RadarPainter extends CustomPainter {
  const RadarPainter({required this.angle});

  final double angle;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2;

    // Background circles
    final bgPaint = Paint()
      ..color = Colors.green.withOpacity(0.3)
      ..style = PaintingStyle.stroke
      ..strokeWidth = 1;
    for (var i = 1; i <= 4; i++) {
      canvas.drawCircle(center, radius * i / 4, bgPaint);
    }

    // Cross lines
    canvas.drawLine(
      Offset(center.dx - radius, center.dy),
      Offset(center.dx + radius, center.dy),
      bgPaint,
    );
    canvas.drawLine(
      Offset(center.dx, center.dy - radius),
      Offset(center.dx, center.dy + radius),
      bgPaint,
    );

    // Sweep
    final sweepPaint = Paint()
      ..shader = SweepGradient(
        colors: [Colors.green.withOpacity(0), Colors.green.withOpacity(0.6)],
        stops: const [0, 1],
        startAngle: angle - 1.0,
        endAngle: angle,
      ).createShader(Rect.fromCircle(center: center, radius: radius))
      ..style = PaintingStyle.fill;

    canvas.drawCircle(center, radius, sweepPaint);

    // Sweep line
    final linePaint = Paint()
      ..color = Colors.green
      ..strokeWidth = 2;
    canvas.drawLine(
      center,
      Offset(
        center.dx + radius * math.cos(angle),
        center.dy + radius * math.sin(angle),
      ),
      linePaint,
    );
  }

  @override
  bool shouldRepaint(RadarPainter oldDelegate) =>
      oldDelegate.angle != angle;
}
```

---

## ขั้นตอนที่ 297: Page Transitions

```dart
// lib/animations/page_transitions.dart
import 'package:flutter/material.dart';

// Custom Route Transitions

// Fade Transition
class FadeRoute<T> extends PageRouteBuilder<T> {
  FadeRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return FadeTransition(opacity: animation, child: child);
          },
        );

  final Widget page;
}

// Slide Transition
class SlideRoute<T> extends PageRouteBuilder<T> {
  SlideRoute({
    required this.page,
    this.direction = SlideDirection.rightToLeft,
  }) : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final begin = direction.offset;
            const end = Offset.zero;
            const curve = Curves.easeInOut;

            final tween = Tween<Offset>(begin: begin, end: end).chain(
              CurveTween(curve: curve),
            );

            return SlideTransition(
              position: animation.drive(tween),
              child: child,
            );
          },
        );

  final Widget page;
  final SlideDirection direction;
}

enum SlideDirection {
  rightToLeft(Offset(1.0, 0.0)),
  leftToRight(Offset(-1.0, 0.0)),
  bottomToTop(Offset(0.0, 1.0)),
  topToBottom(Offset(0.0, -1.0));

  const SlideDirection(this.offset);
  final Offset offset;
}

// Scale + Fade
class ScaleFadeRoute<T> extends PageRouteBuilder<T> {
  ScaleFadeRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return ScaleTransition(
              scale: CurvedAnimation(
                parent: animation,
                curve: Curves.easeOut,
              ),
              child: FadeTransition(opacity: animation, child: child),
            );
          },
        );

  final Widget page;
}

// Rotate + Scale
class RotateRoute<T> extends PageRouteBuilder<T> {
  RotateRoute({required this.page})
      : super(
          transitionDuration: const Duration(milliseconds: 600),
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return RotationTransition(
              turns: Tween<double>(begin: 0.5, end: 1.0).animate(
                CurvedAnimation(parent: animation, curve: Curves.easeOut),
              ),
              child: ScaleTransition(
                scale: animation,
                child: FadeTransition(opacity: animation, child: child),
              ),
            );
          },
        );

  final Widget page;
}

// ตัวอย่างการใช้งาน
class TransitionsDemoPage extends StatelessWidget {
  const TransitionsDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Page Transitions')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _TransitionButton(
            label: 'Fade Transition',
            color: Colors.blue,
            onTap: () => Navigator.push(
              context,
              FadeRoute(page: _DestinationPage(color: Colors.blue)),
            ),
          ),
          const SizedBox(height: 12),
          _TransitionButton(
            label: 'Slide Right to Left',
            color: Colors.green,
            onTap: () => Navigator.push(
              context,
              SlideRoute(
                page: _DestinationPage(color: Colors.green),
                direction: SlideDirection.rightToLeft,
              ),
            ),
          ),
          const SizedBox(height: 12),
          _TransitionButton(
            label: 'Slide Bottom to Top',
            color: Colors.orange,
            onTap: () => Navigator.push(
              context,
              SlideRoute(
                page: _DestinationPage(color: Colors.orange),
                direction: SlideDirection.bottomToTop,
              ),
            ),
          ),
          const SizedBox(height: 12),
          _TransitionButton(
            label: 'Scale + Fade',
            color: Colors.purple,
            onTap: () => Navigator.push(
              context,
              ScaleFadeRoute(page: _DestinationPage(color: Colors.purple)),
            ),
          ),
          const SizedBox(height: 12),
          _TransitionButton(
            label: 'Rotate',
            color: Colors.red,
            onTap: () => Navigator.push(
              context,
              RotateRoute(page: _DestinationPage(color: Colors.red)),
            ),
          ),
        ],
      ),
    );
  }
}

class _TransitionButton extends StatelessWidget {
  const _TransitionButton({
    required this.label,
    required this.color,
    required this.onTap,
  });

  final String label;
  final Color color;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onTap,
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
        foregroundColor: Colors.white,
        padding: const EdgeInsets.all(16),
      ),
      child: Text(label),
    );
  }
}

class _DestinationPage extends StatelessWidget {
  const _DestinationPage({required this.color});

  final Color color;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: color,
      appBar: AppBar(
        title: const Text('Destination Page'),
        backgroundColor: color.withOpacity(0.8),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.star, color: Colors.white, size: 80),
            const SizedBox(height: 24),
            const Text(
              'หน้าปลายทาง',
              style: TextStyle(color: Colors.white, fontSize: 24),
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('กลับ'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 298: Drag Animations

```dart
// lib/animations/drag_animation.dart
import 'package:flutter/material.dart';

class DragAnimationPage extends StatefulWidget {
  const DragAnimationPage({super.key});

  @override
  State<DragAnimationPage> createState() => _DragAnimationPageState();
}

class _DragAnimationPageState extends State<DragAnimationPage>
    with TickerProviderStateMixin {

  Offset _position = const Offset(150, 300);
  Offset _dragStart = Offset.zero;
  late AnimationController _snapController;
  late Animation<Offset> _snapAnimation;

  @override
  void initState() {
    super.initState();
    _snapController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 400),
    );
  }

  @override
  void dispose() {
    _snapController.dispose();
    super.dispose();
  }

  void _onDragUpdate(DragUpdateDetails details) {
    setState(() {
      _position += details.delta;
    });
  }

  void _onDragEnd(DragEndDetails details) {
    final screenSize = MediaQuery.of(context).size;

    // Snap to nearest corner
    final targetX = _position.dx < screenSize.width / 2 ? 50.0 : screenSize.width - 50;
    final targetY = _position.dy < screenSize.height / 2 ? 100.0 : screenSize.height - 100;

    final target = Offset(targetX, targetY);

    _snapAnimation = Tween<Offset>(
      begin: _position,
      end: target,
    ).animate(
      CurvedAnimation(parent: _snapController, curve: Curves.elasticOut),
    );

    _snapController.reset();
    _snapController.forward();

    _snapAnimation.addListener(() {
      setState(() {
        _position = _snapAnimation.value;
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Drag Animation')),
      body: Stack(
        children: [
          // Drop Zones
          Positioned(
            left: 0, top: 0,
            right: MediaQuery.of(context).size.width / 2,
            bottom: MediaQuery.of(context).size.height / 2,
            child: _DropZone(color: Colors.blue.withOpacity(0.1), label: 'โซน 1'),
          ),
          Positioned(
            right: 0, top: 0,
            left: MediaQuery.of(context).size.width / 2,
            bottom: MediaQuery.of(context).size.height / 2,
            child: _DropZone(color: Colors.green.withOpacity(0.1), label: 'โซน 2'),
          ),
          Positioned(
            left: 0, bottom: 0,
            right: MediaQuery.of(context).size.width / 2,
            top: MediaQuery.of(context).size.height / 2,
            child: _DropZone(color: Colors.orange.withOpacity(0.1), label: 'โซน 3'),
          ),
          Positioned(
            right: 0, bottom: 0,
            left: MediaQuery.of(context).size.width / 2,
            top: MediaQuery.of(context).size.height / 2,
            child: _DropZone(color: Colors.purple.withOpacity(0.1), label: 'โซน 4'),
          ),

          // Draggable Ball
          Positioned(
            left: _position.dx - 40,
            top: _position.dy - 40,
            child: GestureDetector(
              onPanUpdate: _onDragUpdate,
              onPanEnd: _onDragEnd,
              child: Container(
                width: 80,
                height: 80,
                decoration: BoxDecoration(
                  color: Colors.red,
                  shape: BoxShape.circle,
                  boxShadow: [
                    BoxShadow(
                      color: Colors.red.withOpacity(0.4),
                      blurRadius: 15,
                      spreadRadius: 5,
                    ),
                  ],
                ),
                child: const Icon(Icons.drag_indicator,
                    color: Colors.white, size: 36),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

class _DropZone extends StatelessWidget {
  const _DropZone({required this.color, required this.label});

  final Color color;
  final String label;

  @override
  Widget build(BuildContext context) {
    return Container(
      color: color,
      child: Center(
        child: Text(
          label,
          style: const TextStyle(fontSize: 18, color: Colors.black38),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 299: Spring & Bounce Effects

```dart
// lib/animations/spring_bounce.dart
import 'package:flutter/material.dart';

// Bounce Button
class BounceButton extends StatefulWidget {
  const BounceButton({
    super.key,
    required this.onPressed,
    required this.child,
  });

  final VoidCallback onPressed;
  final Widget child;

  @override
  State<BounceButton> createState() => _BounceButtonState();
}

class _BounceButtonState extends State<BounceButton>
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

    _scaleAnimation = Tween<double>(begin: 1.0, end: 0.85).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => _controller.forward(),
      onTapUp: (_) {
        _controller.reverse();
        widget.onPressed();
      },
      onTapCancel: () => _controller.reverse(),
      child: ScaleTransition(
        scale: _scaleAnimation,
        child: widget.child,
      ),
    );
  }
}

// Shake Animation
class ShakeWidget extends StatefulWidget {
  const ShakeWidget({
    super.key,
    required this.child,
    this.duration = const Duration(milliseconds: 500),
  });

  final Widget child;
  final Duration duration;

  @override
  State<ShakeWidget> createState() => ShakeWidgetState();
}

class ShakeWidgetState extends State<ShakeWidget>
    with SingleTickerProviderStateMixin {

  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: widget.duration,
    );

    _animation = Tween<double>(begin: 0, end: 1).animate(_controller)
      ..addListener(() => setState(() {}));
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void shake() {
    _controller.forward(from: 0);
  }

  double _shakeOffset(double t) {
    const shake = 3;
    const count = 5;
    final x = math.sin(t * math.pi * count) * shake;
    return x * (1 - t);
  }

  @override
  Widget build(BuildContext context) {
    return Transform.translate(
      offset: Offset(_shakeOffset(_animation.value), 0),
      child: widget.child,
    );
  }
}

import 'dart:math' as math;

// Pulse Animation
class PulseWidget extends StatefulWidget {
  const PulseWidget({super.key, required this.child, this.color = Colors.red});

  final Widget child;
  final Color color;

  @override
  State<PulseWidget> createState() => _PulseWidgetState();
}

class _PulseWidgetState extends State<PulseWidget>
    with SingleTickerProviderStateMixin {

  late AnimationController _controller;
  late Animation<double> _pulseAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    )..repeat(reverse: true);

    _pulseAnimation = Tween<double>(begin: 1.0, end: 1.2).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
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
      builder: (context, child) {
        return Stack(
          alignment: Alignment.center,
          children: [
            // Pulse ring
            ScaleTransition(
              scale: _pulseAnimation,
              child: Container(
                width: 80,
                height: 80,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: widget.color.withOpacity(
                    0.3 * (1 - _controller.value),
                  ),
                ),
              ),
            ),
            // Main widget
            widget.child,
          ],
        );
      },
    );
  }
}

// Spring & Bounce Demo Page
class SpringBounceDemoPage extends StatefulWidget {
  const SpringBounceDemoPage({super.key});

  @override
  State<SpringBounceDemoPage> createState() => _SpringBounceDemoPageState();
}

class _SpringBounceDemoPageState extends State<SpringBounceDemoPage> {
  final GlobalKey<ShakeWidgetState> _shakeKey = GlobalKey();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Spring & Bounce')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            const Text(
              'Bounce Button',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 12),
            BounceButton(
              onPressed: () => ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Bounced!')),
              ),
              child: Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.blue,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: const Text(
                  'กดฉันดิ!',
                  style: TextStyle(color: Colors.white, fontSize: 18),
                  textAlign: TextAlign.center,
                ),
              ),
            ),
            const SizedBox(height: 32),

            const Text(
              'Shake Effect',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 12),
            ShakeWidget(
              key: _shakeKey,
              child: TextFormField(
                decoration: InputDecoration(
                  border: const OutlineInputBorder(),
                  labelText: 'รหัสผ่าน',
                  suffixIcon: IconButton(
                    icon: const Icon(Icons.error_outline),
                    onPressed: () => _shakeKey.currentState?.shake(),
                  ),
                ),
                obscureText: true,
              ),
            ),
            TextButton(
              onPressed: () => _shakeKey.currentState?.shake(),
              child: const Text('ทดสอบ Shake'),
            ),
            const SizedBox(height: 32),

            const Text(
              'Pulse Effect',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 24),
            Center(
              child: PulseWidget(
                color: Colors.red,
                child: Container(
                  width: 60,
                  height: 60,
                  decoration: const BoxDecoration(
                    color: Colors.red,
                    shape: BoxShape.circle,
                  ),
                  child: const Icon(Icons.notifications, color: Colors.white),
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

## ขั้นตอนที่ 300: Hero Animations & Shared Elements

```dart
// lib/animations/hero_animation.dart
import 'package:flutter/material.dart';

// Hero Animation - Shared Element Transition
class HeroGalleryPage extends StatelessWidget {
  const HeroGalleryPage({super.key});

  static final List<_GalleryItem> items = [
    _GalleryItem(id: '1', color: Colors.red, icon: Icons.favorite, title: 'Item 1'),
    _GalleryItem(id: '2', color: Colors.blue, icon: Icons.star, title: 'Item 2'),
    _GalleryItem(id: '3', color: Colors.green, icon: Icons.flash_on, title: 'Item 3'),
    _GalleryItem(id: '4', color: Colors.orange, icon: Icons.music_note, title: 'Item 4'),
    _GalleryItem(id: '5', color: Colors.purple, icon: Icons.camera, title: 'Item 5'),
    _GalleryItem(id: '6', color: Colors.teal, icon: Icons.palette, title: 'Item 6'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Hero Animations')),
      body: GridView.builder(
        padding: const EdgeInsets.all(12),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
        ),
        itemCount: items.length,
        itemBuilder: (context, index) {
          final item = items[index];
          return GestureDetector(
            onTap: () {
              Navigator.push(
                context,
                PageRouteBuilder(
                  pageBuilder: (ctx, anim, secondAnim) =>
                      HeroDetailPage(item: item),
                  transitionDuration: const Duration(milliseconds: 500),
                  transitionsBuilder: (ctx, anim, secondAnim, child) {
                    return FadeTransition(opacity: anim, child: child);
                  },
                ),
              );
            },
            child: Hero(
              tag: 'card_${item.id}',
              child: Material(
                borderRadius: BorderRadius.circular(16),
                color: item.color,
                elevation: 4,
                child: Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(item.icon, color: Colors.white, size: 48),
                      const SizedBox(height: 12),
                      Text(
                        item.title,
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 18,
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

class HeroDetailPage extends StatelessWidget {
  const HeroDetailPage({super.key, required this.item});

  final _GalleryItem item;

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
                tag: 'card_${item.id}',
                child: Material(
                  color: item.color,
                  child: Center(
                    child: Icon(
                      item.icon,
                      color: Colors.white,
                      size: 100,
                    ),
                  ),
                ),
              ),
            ),
          ),
          SliverPadding(
            padding: const EdgeInsets.all(16),
            sliver: SliverList(
              delegate: SliverChildListDelegate([
                Text(
                  item.title,
                  style: const TextStyle(
                    fontSize: 28,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 16),
                const Text(
                  'รายละเอียดของ item นี้ Hero Animation ทำให้การเปลี่ยนหน้า '
                  'ดูสวยงามและต่อเนื่อง ผู้ใช้จะรู้สึกว่า element เดียวกัน '
                  'เคลื่อนจากหน้าหนึ่งไปอีกหน้าหนึ่ง',
                  style: TextStyle(fontSize: 16, height: 1.6),
                ),
                const SizedBox(height: 24),
                const Divider(),
                const SizedBox(height: 24),
                ...List.generate(
                  5,
                  (i) => ListTile(
                    leading: Icon(Icons.circle, color: item.color),
                    title: Text('รายการที่ ${i + 1}'),
                  ),
                ),
              ]),
            ),
          ),
        ],
      ),
    );
  }
}

class _GalleryItem {
  const _GalleryItem({
    required this.id,
    required this.color,
    required this.icon,
    required this.title,
  });

  final String id;
  final Color color;
  final IconData icon;
  final String title;
}
```

---

## Workshop: Advanced Animation App

```dart
// lib/workshop/animation_showcase.dart
import 'dart:math' as math;
import 'package:flutter/material.dart';

class AnimationShowcasePage extends StatefulWidget {
  const AnimationShowcasePage({super.key});

  @override
  State<AnimationShowcasePage> createState() => _AnimationShowcasePageState();
}

class _AnimationShowcasePageState extends State<AnimationShowcasePage>
    with TickerProviderStateMixin {

  late AnimationController _mainController;
  late AnimationController _idleController;
  late AnimationController _successController;

  int _currentIndex = 0;
  bool _showSuccess = false;

  @override
  void initState() {
    super.initState();

    _mainController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1000),
    );

    _idleController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat(reverse: true);

    _successController = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );
  }

  @override
  void dispose() {
    _mainController.dispose();
    _idleController.dispose();
    _successController.dispose();
    super.dispose();
  }

  void _showSuccessAnimation() async {
    setState(() => _showSuccess = true);
    await _successController.forward();
    await Future.delayed(const Duration(seconds: 1));
    await _successController.reverse();
    setState(() => _showSuccess = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // Background Gradient Animation
          AnimatedBuilder(
            animation: _idleController,
            builder: (context, _) {
              return Container(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                    colors: [
                      Color.lerp(
                        Colors.blue[900],
                        Colors.purple[900],
                        _idleController.value,
                      )!,
                      Color.lerp(
                        Colors.purple[900],
                        Colors.blue[900],
                        _idleController.value,
                      )!,
                    ],
                  ),
                ),
              );
            },
          ),

          // Floating Particles
          ...List.generate(20, (i) {
            return AnimatedBuilder(
              animation: _idleController,
              builder: (context, _) {
                final angle = (i / 20) * math.pi * 2 +
                    _idleController.value * math.pi;
                final radius = 100.0 + math.sin(angle * 3) * 50;
                final x = MediaQuery.of(context).size.width / 2 +
                    radius * math.cos(angle);
                final y = MediaQuery.of(context).size.height / 2 +
                    radius * math.sin(angle);

                return Positioned(
                  left: x,
                  top: y,
                  child: Container(
                    width: 6,
                    height: 6,
                    decoration: BoxDecoration(
                      color: Colors.white.withOpacity(0.3),
                      shape: BoxShape.circle,
                    ),
                  ),
                );
              },
            );
          }),

          // Main Content
          Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                // Animated Logo
                AnimatedBuilder(
                  animation: _idleController,
                  builder: (context, _) {
                    return Transform.scale(
                      scale: 1.0 + _idleController.value * 0.05,
                      child: Container(
                        width: 120,
                        height: 120,
                        decoration: BoxDecoration(
                          color: Colors.white.withOpacity(0.2),
                          shape: BoxShape.circle,
                          border: Border.all(
                            color: Colors.white.withOpacity(0.5),
                            width: 2,
                          ),
                        ),
                        child: const Icon(
                          Icons.flutter_dash,
                          color: Colors.white,
                          size: 60,
                        ),
                      ),
                    );
                  },
                ),

                const SizedBox(height: 40),

                const Text(
                  'Flutter Animations',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 28,
                    fontWeight: FontWeight.bold,
                    letterSpacing: 2,
                  ),
                ),

                const SizedBox(height: 40),

                // Animated Buttons
                ..._buildAnimatedButtons(context),
              ],
            ),
          ),

          // Success Overlay
          if (_showSuccess)
            AnimatedBuilder(
              animation: _successController,
              builder: (context, _) {
                return Positioned.fill(
                  child: Container(
                    color: Colors.black.withOpacity(
                        _successController.value * 0.5),
                    child: Center(
                      child: ScaleTransition(
                        scale: CurvedAnimation(
                          parent: _successController,
                          curve: Curves.elasticOut,
                        ),
                        child: Container(
                          padding: const EdgeInsets.all(32),
                          decoration: BoxDecoration(
                            color: Colors.white,
                            borderRadius: BorderRadius.circular(24),
                          ),
                          child: Column(
                            mainAxisSize: MainAxisSize.min,
                            children: const [
                              Icon(Icons.check_circle,
                                  color: Colors.green, size: 80),
                              SizedBox(height: 16),
                              Text(
                                'สำเร็จ!',
                                style: TextStyle(
                                  fontSize: 24,
                                  fontWeight: FontWeight.bold,
                                ),
                              ),
                            ],
                          ),
                        ),
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

  List<Widget> _buildAnimatedButtons(BuildContext context) {
    final buttons = [
      ('Staggered', Colors.blue, Icons.layers),
      ('Physics', Colors.green, Icons.science),
      ('Hero', Colors.orange, Icons.photo),
      ('Custom Painter', Colors.purple, Icons.brush),
    ];

    return buttons.asMap().entries.map((entry) {
      final index = entry.key;
      final (label, color, icon) = entry.value;

      return Padding(
        padding: const EdgeInsets.only(bottom: 12),
        child: TweenAnimationBuilder<double>(
          tween: Tween(begin: 0, end: 1),
          duration: Duration(milliseconds: 600 + index * 100),
          curve: Curves.easeOut,
          builder: (context, value, child) {
            return Transform.translate(
              offset: Offset((1 - value) * 100, 0),
              child: Opacity(
                opacity: value,
                child: child,
              ),
            );
          },
          child: SizedBox(
            width: 250,
            child: ElevatedButton.icon(
              onPressed: () => _showSuccessAnimation(),
              style: ElevatedButton.styleFrom(
                backgroundColor: color.withOpacity(0.8),
                foregroundColor: Colors.white,
                padding: const EdgeInsets.all(16),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
              icon: Icon(icon),
              label: Text(label),
            ),
          ),
        ),
      );
    }).toList();
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **AnimationController**: Lifecycle, Listeners, Status Callbacks
- **Staggered Animations**: Animation หลายตัวที่เริ่มต่างเวลา
- **Physics Simulations**: SpringSimulation, GravitySimulation
- **Rive Animations**: State Machine, Inputs, Triggers
- **Lottie Animations**: JSON-based animations จาก After Effects
- **CustomPainter**: Wave, Radar, Loading Painter
- **Page Transitions**: Fade, Slide, Scale, Rotate Routes
- **Drag Animations**: Snap to position, Velocity-based
- **Spring & Bounce**: BounceButton, ShakeWidget, PulseWidget
- **Hero Animations**: Shared Element Transitions

## แบบฝึกหัด

1. สร้าง Particle System Animation ด้วย CustomPainter
2. Implement Swipe-to-Delete ด้วย Physics Simulation
3. สร้าง Loading Screen ด้วย Lottie
4. ทำ Morphing Animation ระหว่าง Shapes
5. สร้าง Animated Bottom Navigation Bar ด้วย AnimationController

---

[⬅️ Part 29](part_29.md) | [กลับหน้าหลัก](../README.md)

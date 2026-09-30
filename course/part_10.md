# Part 10: Assets, Images, Icons
## ขั้นตอนที่ 91-100

---

## สารบัญ
1. [การตั้งค่า Assets ใน pubspec.yaml](#การตั้งค่า-assets)
2. [AssetImage และ Image.asset](#assetimage)
3. [NetworkImage และ Image.network](#networkimage)
4. [FadeInImage](#fadeinimage)
5. [CachedNetworkImage](#cachednetworkimage)
6. [Image Effects และ Filters](#image-effects)
7. [Icon และ IconButton](#icon-และ-iconbutton)
8. [SVG Images](#svg-images)
9. [CustomPaint และ ClipPath](#custompaint)
10. [Workshop: Gallery App](#workshop)

---

## ขั้นตอนที่ 91: การตั้งค่า Assets

### โครงสร้างโฟลเดอร์ assets

```
my_app/
├── assets/
│   ├── images/
│   │   ├── logo.png
│   │   ├── banner.jpg
│   │   └── 2.0x/        # สำหรับ high-DPI
│   │       ├── logo.png
│   │       └── banner.jpg
│   │   └── 3.0x/
│   │       ├── logo.png
│   │       └── banner.jpg
│   ├── icons/
│   │   └── app_icon.png
│   ├── fonts/
│   │   ├── Prompt-Regular.ttf
│   │   └── Prompt-Bold.ttf
│   └── data/
│       └── countries.json
└── pubspec.yaml
```

### pubspec.yaml

```yaml
flutter:
  uses-material-design: true

  assets:
    - assets/images/           # ทั้งโฟลเดอร์
    - assets/icons/
    - assets/data/countries.json  # ไฟล์เดียว

  fonts:
    - family: Prompt
      fonts:
        - asset: assets/fonts/Prompt-Regular.ttf
        - asset: assets/fonts/Prompt-Bold.ttf
          weight: 700
        - asset: assets/fonts/Prompt-Italic.ttf
          style: italic
```

### โหลด Asset ใน Code

```dart
import 'package:flutter/services.dart';
import 'dart:convert';

class AssetLoader {
  // โหลด JSON
  static Future<Map<String, dynamic>> loadJson(String path) async {
    final String data = await rootBundle.loadString(path);
    return json.decode(data) as Map<String, dynamic>;
  }

  // โหลด String
  static Future<String> loadText(String path) async {
    return await rootBundle.loadString(path);
  }

  // โหลด bytes
  static Future<List<int>> loadBytes(String path) async {
    final ByteData data = await rootBundle.load(path);
    return data.buffer.asUint8List();
  }
}

// ใช้งาน
Future<void> loadCountries() async {
  final data = await AssetLoader.loadJson('assets/data/countries.json');
  final List countries = data['countries'];
  print('Loaded ${countries.length} countries');
}
```

---

## ขั้นตอนที่ 92: AssetImage

```dart
import 'package:flutter/material.dart';

class AssetImageExamples extends StatelessWidget {
  const AssetImageExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Asset Images')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // วิธีที่ 1: Image.asset
            Image.asset(
              'assets/images/logo.png',
              width: 200,
              height: 200,
              fit: BoxFit.contain,
            ),

            const SizedBox(height: 16),

            // วิธีที่ 2: Image widget กับ AssetImage
            const Image(
              image: AssetImage('assets/images/banner.jpg'),
              width: double.infinity,
              height: 150,
              fit: BoxFit.cover,
            ),

            const SizedBox(height: 16),

            // กำหนด BoxFit ต่างๆ
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: BoxFit.values.map((fit) {
                return Column(
                  children: [
                    Container(
                      width: 80,
                      height: 80,
                      decoration: BoxDecoration(
                        border: Border.all(color: Colors.grey),
                      ),
                      child: Image.asset(
                        'assets/images/logo.png',
                        fit: fit,
                      ),
                    ),
                    Text(fit.name, style: const TextStyle(fontSize: 10)),
                  ],
                );
              }).toList(),
            ),

            const SizedBox(height: 16),

            // Image กับ color overlay
            ColorFiltered(
              colorFilter: ColorFilter.mode(
                Colors.blue.withOpacity(0.5),
                BlendMode.srcATop,
              ),
              child: Image.asset('assets/images/logo.png', width: 150),
            ),

            const SizedBox(height: 16),

            // Image ใน CircleAvatar
            const CircleAvatar(
              radius: 50,
              backgroundImage: AssetImage('assets/images/avatar.png'),
            ),

            const SizedBox(height: 16),

            // Image กับ errorBuilder
            Image.asset(
              'assets/images/missing.png',
              width: 100,
              height: 100,
              errorBuilder: (context, error, stackTrace) {
                return Container(
                  width: 100,
                  height: 100,
                  color: Colors.grey.shade200,
                  child: const Icon(Icons.broken_image, size: 48, color: Colors.grey),
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

## ขั้นตอนที่ 93: NetworkImage

```dart
class NetworkImageExamples extends StatelessWidget {
  const NetworkImageExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Network Images')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // Image.network พื้นฐาน
          Image.network(
            'https://picsum.photos/400/200',
            width: double.infinity,
            height: 200,
            fit: BoxFit.cover,
          ),

          const SizedBox(height: 16),

          // กับ loadingBuilder (progress indicator)
          Image.network(
            'https://picsum.photos/800/400',
            width: double.infinity,
            height: 200,
            fit: BoxFit.cover,
            loadingBuilder: (context, child, loadingProgress) {
              if (loadingProgress == null) return child;

              final progress = loadingProgress.expectedTotalBytes != null
                  ? loadingProgress.cumulativeBytesLoaded /
                      loadingProgress.expectedTotalBytes!
                  : null;

              return Container(
                height: 200,
                color: Colors.grey.shade100,
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      CircularProgressIndicator(value: progress),
                      if (progress != null) ...[
                        const SizedBox(height: 8),
                        Text('${(progress * 100).toInt()}%'),
                      ],
                    ],
                  ),
                ),
              );
            },
            errorBuilder: (context, error, stackTrace) {
              return Container(
                height: 200,
                color: Colors.red.shade50,
                child: const Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.error_outline, size: 48, color: Colors.red),
                    SizedBox(height: 8),
                    Text('โหลดรูปไม่สำเร็จ'),
                  ],
                ),
              );
            },
          ),

          const SizedBox(height: 16),

          // Grid ของรูปจาก network
          GridView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 4,
              mainAxisSpacing: 4,
            ),
            itemCount: 9,
            itemBuilder: (context, index) {
              return ClipRRect(
                borderRadius: BorderRadius.circular(8),
                child: Image.network(
                  'https://picsum.photos/200/200?random=$index',
                  fit: BoxFit.cover,
                  loadingBuilder: (_, child, progress) {
                    if (progress == null) return child;
                    return Container(
                      color: Colors.grey.shade200,
                      child: const Center(
                        child: CircularProgressIndicator(strokeWidth: 2),
                      ),
                    );
                  },
                ),
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

## ขั้นตอนที่ 94: FadeInImage

```dart
class FadeInImageExample extends StatelessWidget {
  const FadeInImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FadeInImage')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // FadeIn จาก placeholder asset
          FadeInImage.assetNetwork(
            placeholder: 'assets/images/placeholder.png',
            image: 'https://picsum.photos/400/300',
            width: double.infinity,
            height: 200,
            fit: BoxFit.cover,
            fadeInDuration: const Duration(milliseconds: 500),
            fadeOutDuration: const Duration(milliseconds: 300),
          ),

          const SizedBox(height: 16),

          // FadeIn จาก MemoryImage (transparent pixel)
          FadeInImage(
            placeholder: MemoryImage(_kTransparentImage),
            image: const NetworkImage('https://picsum.photos/400/300?random=1'),
            width: double.infinity,
            height: 200,
            fit: BoxFit.cover,
            imageErrorBuilder: (context, error, stackTrace) {
              return const Icon(Icons.error);
            },
          ),

          const SizedBox(height: 16),

          // Grid ด้วย FadeInImage
          GridView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
              childAspectRatio: 1.5,
            ),
            itemCount: 6,
            itemBuilder: (context, index) {
              return ClipRRect(
                borderRadius: BorderRadius.circular(12),
                child: FadeInImage(
                  placeholder: MemoryImage(_kTransparentImage),
                  image: NetworkImage(
                    'https://picsum.photos/300/200?random=${index + 10}',
                  ),
                  fit: BoxFit.cover,
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}

// 1x1 transparent PNG
final Uint8List _kTransparentImage = base64Decode(
  'iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8z8BQDwADhQGAWjR9awAAAABJRU5ErkJggg==',
);
```

---

## ขั้นตอนที่ 95: CachedNetworkImage

```yaml
# pubspec.yaml
dependencies:
  cached_network_image: ^3.3.0
```

```dart
import 'package:cached_network_image/cached_network_image.dart';

class CachedImageExample extends StatelessWidget {
  const CachedImageExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Cached Network Image'),
        actions: [
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: () async {
              await CachedNetworkImage.evictFromCache(
                'https://picsum.photos/400/300',
              );
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('Cache cleared')),
              );
            },
          ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // พื้นฐาน
          CachedNetworkImage(
            imageUrl: 'https://picsum.photos/400/300',
            width: double.infinity,
            height: 200,
            fit: BoxFit.cover,
            placeholder: (context, url) => Container(
              height: 200,
              color: Colors.grey.shade200,
              child: const Center(child: CircularProgressIndicator()),
            ),
            errorWidget: (context, url, error) => Container(
              height: 200,
              color: Colors.red.shade50,
              child: const Icon(Icons.error),
            ),
          ),

          const SizedBox(height: 16),

          // กับ CachedNetworkImageProvider (ใน CircleAvatar)
          const CircleAvatar(
            radius: 50,
            backgroundImage: CachedNetworkImageProvider(
              'https://i.pravatar.cc/300',
            ),
          ),

          const SizedBox(height: 16),

          // Product cards ด้วย cached images
          ...List.generate(5, (index) => _ProductCard(index: index)),
        ],
      ),
    );
  }
}

class _ProductCard extends StatelessWidget {
  final int index;
  const _ProductCard({required this.index});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      clipBehavior: Clip.antiAlias,
      child: Row(
        children: [
          CachedNetworkImage(
            imageUrl: 'https://picsum.photos/120/120?random=$index',
            width: 100,
            height: 100,
            fit: BoxFit.cover,
            memCacheWidth: 200,  // ควบคุม memory cache size
            maxWidthDiskCache: 200,
            placeholder: (_, __) => Container(
              width: 100,
              height: 100,
              color: Colors.grey.shade200,
              child: const Center(child: CircularProgressIndicator(strokeWidth: 2)),
            ),
            errorWidget: (_, __, ___) => Container(
              width: 100,
              height: 100,
              color: Colors.grey.shade100,
              child: const Icon(Icons.image_not_supported),
            ),
          ),
          Expanded(
            child: Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('Product ${index + 1}',
                      style: const TextStyle(fontWeight: FontWeight.bold)),
                  const SizedBox(height: 4),
                  Text('฿${(index + 1) * 299}',
                      style: TextStyle(color: Colors.green.shade700)),
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

## ขั้นตอนที่ 96: Image Effects และ Filters

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';

class ImageEffectsPage extends StatefulWidget {
  const ImageEffectsPage({super.key});

  @override
  State<ImageEffectsPage> createState() => _ImageEffectsPageState();
}

class _ImageEffectsPageState extends State<ImageEffectsPage> {
  double _blurRadius = 0;
  ColorFilter? _colorFilter;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Image Effects')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Blur effect
            Text('Blur: ${_blurRadius.toStringAsFixed(1)}'),
            Slider(
              value: _blurRadius,
              max: 20,
              onChanged: (v) => setState(() => _blurRadius = v),
            ),
            ImageFiltered(
              imageFilter: ui.ImageFilter.blur(
                sigmaX: _blurRadius,
                sigmaY: _blurRadius,
              ),
              child: Image.network('https://picsum.photos/400/200', fit: BoxFit.cover),
            ),

            const SizedBox(height: 24),

            // ColorFilter effects
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                _FilterChip(
                  label: 'Normal',
                  onTap: () => setState(() => _colorFilter = null),
                ),
                _FilterChip(
                  label: 'Grayscale',
                  onTap: () => setState(() => _colorFilter = const ColorFilter.matrix([
                    0.2126, 0.7152, 0.0722, 0, 0,
                    0.2126, 0.7152, 0.0722, 0, 0,
                    0.2126, 0.7152, 0.0722, 0, 0,
                    0,      0,      0,      1, 0,
                  ])),
                ),
                _FilterChip(
                  label: 'Sepia',
                  onTap: () => setState(() => _colorFilter = const ColorFilter.matrix([
                    0.393, 0.769, 0.189, 0, 0,
                    0.349, 0.686, 0.168, 0, 0,
                    0.272, 0.534, 0.131, 0, 0,
                    0,     0,     0,     1, 0,
                  ])),
                ),
                _FilterChip(
                  label: 'Invert',
                  onTap: () => setState(() => _colorFilter = const ColorFilter.matrix([
                    -1, 0, 0, 0, 255,
                    0, -1, 0, 0, 255,
                    0, 0, -1, 0, 255,
                    0, 0, 0,  1, 0,
                  ])),
                ),
              ],
            ),

            const SizedBox(height: 16),

            ColorFiltered(
              colorFilter: _colorFilter ?? const ColorFilter.mode(
                Colors.transparent,
                BlendMode.dst,
              ),
              child: Image.network(
                'https://picsum.photos/400/200?random=5',
                width: double.infinity,
                height: 200,
                fit: BoxFit.cover,
              ),
            ),

            const SizedBox(height: 24),

            // ClipRRect - ขอบมน
            ClipRRect(
              borderRadius: BorderRadius.circular(20),
              child: Image.network(
                'https://picsum.photos/300/150?random=6',
                width: 300,
                height: 150,
                fit: BoxFit.cover,
              ),
            ),

            const SizedBox(height: 16),

            // ClipOval - วงกลม
            ClipOval(
              child: Image.network(
                'https://picsum.photos/150/150?random=7',
                width: 150,
                height: 150,
                fit: BoxFit.cover,
              ),
            ),

            const SizedBox(height: 16),

            // ShaderMask - gradient overlay
            ShaderMask(
              shaderCallback: (Rect bounds) {
                return const LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [Colors.transparent, Colors.black],
                ).createShader(bounds);
              },
              blendMode: BlendMode.darken,
              child: Image.network(
                'https://picsum.photos/400/200?random=8',
                width: double.infinity,
                height: 200,
                fit: BoxFit.cover,
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _FilterChip extends StatelessWidget {
  final String label;
  final VoidCallback onTap;
  const _FilterChip({required this.label, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return ActionChip(label: Text(label), onPressed: onTap);
  }
}
```

---

## ขั้นตอนที่ 97: Icon และ IconButton

```dart
class IconExamples extends StatelessWidget {
  const IconExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Icons')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Material Icons
            const Text('Material Icons', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                Icons.home, Icons.search, Icons.favorite, Icons.star,
                Icons.settings, Icons.person, Icons.shopping_cart, Icons.notifications,
              ].map((icon) => Icon(icon, size: 32)).toList(),
            ),

            const SizedBox(height: 24),

            // Icon ปรับแต่ง
            const Text('Styled Icons', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                const Icon(Icons.favorite, color: Colors.red, size: 48),
                Icon(Icons.star, color: Colors.amber, size: 48, shadows: [
                  Shadow(color: Colors.orange.shade200, blurRadius: 10),
                ]),
                ShaderMask(
                  shaderCallback: (bounds) => const LinearGradient(
                    colors: [Colors.purple, Colors.blue],
                  ).createShader(bounds),
                  child: const Icon(Icons.flutter_dash, size: 48, color: Colors.white),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // IconButton
            const Text('IconButtons', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                // Standard
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.share),
                  tooltip: 'Share',
                ),

                // Filled
                IconButton.filled(
                  onPressed: () {},
                  icon: const Icon(Icons.add),
                ),

                // FilledTonal
                IconButton.filledTonal(
                  onPressed: () {},
                  icon: const Icon(Icons.edit),
                ),

                // Outlined
                IconButton.outlined(
                  onPressed: () {},
                  icon: const Icon(Icons.delete),
                  style: IconButton.styleFrom(
                    foregroundColor: Colors.red,
                    side: const BorderSide(color: Colors.red),
                  ),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // Icon ใน Container
            const Text('Icon in Container', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.blue.shade100,
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Icon(Icons.notifications, color: Colors.blue.shade700, size: 28),
                ),
                Container(
                  width: 56,
                  height: 56,
                  decoration: const BoxDecoration(
                    shape: BoxShape.circle,
                    gradient: LinearGradient(
                      colors: [Colors.orange, Colors.red],
                    ),
                  ),
                  child: const Icon(Icons.local_fire_department, color: Colors.white, size: 28),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // Custom font icons
            const Text('Font Icons (ต้องเพิ่ม font ใน pubspec)', style: TextStyle(fontWeight: FontWeight.bold)),
            const Text('ตัวอย่าง: FontAwesome, MaterialCommunityIcons'),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 98: SVG Images

```yaml
# pubspec.yaml
dependencies:
  flutter_svg: ^2.0.9
```

```dart
import 'package:flutter_svg/flutter_svg.dart';

class SvgExamples extends StatelessWidget {
  const SvgExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('SVG Images')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // SVG จาก asset
            SvgPicture.asset(
              'assets/icons/flutter_logo.svg',
              width: 100,
              height: 100,
            ),

            const SizedBox(height: 16),

            // SVG จาก network
            SvgPicture.network(
              'https://www.svgrepo.com/show/331388/dart.svg',
              width: 80,
              height: 80,
              placeholderBuilder: (_) => const CircularProgressIndicator(),
            ),

            const SizedBox(height: 16),

            // SVG กับ colorFilter
            SvgPicture.asset(
              'assets/icons/heart.svg',
              width: 60,
              height: 60,
              colorFilter: const ColorFilter.mode(Colors.red, BlendMode.srcIn),
            ),

            const SizedBox(height: 16),

            // SVG ใน Button
            ElevatedButton.icon(
              onPressed: () {},
              icon: SvgPicture.asset(
                'assets/icons/google.svg',
                width: 24,
                height: 24,
              ),
              label: const Text('Sign in with Google'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.white,
                foregroundColor: Colors.black87,
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
              ),
            ),

            const SizedBox(height: 16),

            // SVG inline string
            SvgPicture.string(
              '''<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                <circle cx="50" cy="50" r="40" fill="blue" opacity="0.7"/>
                <rect x="25" y="25" width="50" height="50" fill="orange" opacity="0.7"/>
              </svg>''',
              width: 100,
              height: 100,
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 99: CustomPaint

```dart
class CustomPaintExamples extends StatelessWidget {
  const CustomPaintExamples({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('CustomPaint')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // วาดรูปทรงพื้นฐาน
            CustomPaint(
              size: const Size(300, 200),
              painter: ShapesPainter(),
            ),

            const SizedBox(height: 24),

            // Chart
            CustomPaint(
              size: const Size(double.infinity, 200),
              painter: BarChartPainter(
                values: [0.6, 0.8, 0.4, 0.9, 0.5, 0.7],
                labels: ['ม.ค.', 'ก.พ.', 'มี.ค.', 'เม.ย.', 'พ.ค.', 'มิ.ย.'],
              ),
            ),

            const SizedBox(height: 24),

            // Progress ring
            SizedBox(
              width: 120,
              height: 120,
              child: CustomPaint(
                painter: RingProgressPainter(progress: 0.75),
                child: const Center(
                  child: Text('75%', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ShapesPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..style = PaintingStyle.fill
      ..color = Colors.blue;

    // วงกลม
    canvas.drawCircle(Offset(size.width * 0.2, size.height * 0.4), 40, paint);

    // สี่เหลี่ยม
    paint.color = Colors.orange;
    canvas.drawRect(
      Rect.fromCenter(center: Offset(size.width * 0.5, size.height * 0.4), width: 70, height: 70),
      paint,
    );

    // สามเหลี่ยม
    paint.color = Colors.green;
    final path = Path()
      ..moveTo(size.width * 0.8, size.height * 0.1)
      ..lineTo(size.width * 0.95, size.height * 0.7)
      ..lineTo(size.width * 0.65, size.height * 0.7)
      ..close();
    canvas.drawPath(path, paint);

    // เส้น
    paint
      ..style = PaintingStyle.stroke
      ..color = Colors.purple
      ..strokeWidth = 3;
    canvas.drawLine(const Offset(10, 180), Offset(size.width - 10, 180), paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}

class BarChartPainter extends CustomPainter {
  final List<double> values;
  final List<String> labels;

  BarChartPainter({required this.values, required this.labels});

  @override
  void paint(Canvas canvas, Size size) {
    final barWidth = size.width / values.length - 8;
    final maxHeight = size.height - 30;

    final paint = Paint()..style = PaintingStyle.fill;
    final textPainter = TextPainter(textDirection: TextDirection.ltr);

    for (int i = 0; i < values.length; i++) {
      final barHeight = values[i] * maxHeight;
      final left = i * (barWidth + 8) + 4;
      final top = maxHeight - barHeight;

      // Gradient bar
      paint.shader = LinearGradient(
        begin: Alignment.topCenter,
        end: Alignment.bottomCenter,
        colors: [Colors.blue.shade300, Colors.blue.shade700],
      ).createShader(Rect.fromLTWH(left, top, barWidth, barHeight));

      canvas.drawRRect(
        RRect.fromRectAndCorners(
          Rect.fromLTWH(left, top, barWidth, barHeight),
          topLeft: const Radius.circular(6),
          topRight: const Radius.circular(6),
        ),
        paint,
      );

      // Label
      textPainter
        ..text = TextSpan(
          text: labels[i],
          style: const TextStyle(color: Colors.grey, fontSize: 11),
        )
        ..layout()
        ..paint(canvas, Offset(left + barWidth / 2 - textPainter.width / 2, size.height - 20));
    }
  }

  @override
  bool shouldRepaint(covariant BarChartPainter old) => old.values != values;
}

class RingProgressPainter extends CustomPainter {
  final double progress;
  RingProgressPainter({required this.progress});

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 10;

    // Background ring
    canvas.drawCircle(
      center,
      radius,
      Paint()
        ..style = PaintingStyle.stroke
        ..strokeWidth = 12
        ..color = Colors.grey.shade200,
    );

    // Progress arc
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -1.5708, // -90 degrees (start from top)
      progress * 2 * 3.14159,
      false,
      Paint()
        ..style = PaintingStyle.stroke
        ..strokeWidth = 12
        ..strokeCap = StrokeCap.round
        ..color = Colors.blue,
    );
  }

  @override
  bool shouldRepaint(covariant RingProgressPainter old) => old.progress != progress;
}
```

---

## ขั้นตอนที่ 100: Image Handling Patterns

```dart
// Placeholder image widget ที่ใช้ซ้ำได้
class AppImage extends StatelessWidget {
  final String? url;
  final String? assetPath;
  final double? width;
  final double? height;
  final BoxFit fit;
  final BorderRadius? borderRadius;
  final Widget? placeholder;
  final Widget? errorWidget;

  const AppImage({
    super.key,
    this.url,
    this.assetPath,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
    this.borderRadius,
    this.placeholder,
    this.errorWidget,
  }) : assert(url != null || assetPath != null, 'Provide url or assetPath');

  @override
  Widget build(BuildContext context) {
    Widget image;

    if (assetPath != null) {
      image = Image.asset(
        assetPath!,
        width: width,
        height: height,
        fit: fit,
        errorBuilder: (_, __, ___) => _buildError(),
      );
    } else {
      image = CachedNetworkImage(
        imageUrl: url!,
        width: width,
        height: height,
        fit: fit,
        placeholder: (_, __) => placeholder ?? _buildPlaceholder(),
        errorWidget: (_, __, ___) => errorWidget ?? _buildError(),
      );
    }

    if (borderRadius != null) {
      return ClipRRect(borderRadius: borderRadius!, child: image);
    }
    return image;
  }

  Widget _buildPlaceholder() {
    return Container(
      width: width,
      height: height,
      color: Colors.grey.shade200,
      child: const Center(
        child: CircularProgressIndicator(strokeWidth: 2),
      ),
    );
  }

  Widget _buildError() {
    return Container(
      width: width,
      height: height,
      color: Colors.grey.shade100,
      child: const Center(
        child: Icon(Icons.image_not_supported, color: Colors.grey),
      ),
    );
  }
}

// ใช้งาน
// AppImage(url: 'https://...', width: 200, height: 150, borderRadius: BorderRadius.circular(12))
// AppImage(assetPath: 'assets/images/logo.png', width: 100)
```

---

## Workshop: Photo Gallery App

```dart
import 'package:flutter/material.dart';
import 'package:cached_network_image/cached_network_image.dart';

void main() => runApp(const GalleryApp());

class GalleryApp extends StatelessWidget {
  const GalleryApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Photo Gallery',
      theme: ThemeData.dark(useMaterial3: true),
      home: const GalleryScreen(),
    );
  }
}

class Photo {
  final int id;
  final String url;
  final String thumbUrl;
  final String title;

  Photo({required this.id, required this.url, required this.thumbUrl, required this.title});

  factory Photo.fromId(int id) => Photo(
    id: id,
    url: 'https://picsum.photos/800/600?random=$id',
    thumbUrl: 'https://picsum.photos/400/300?random=$id',
    title: 'Photo $id',
  );
}

class GalleryScreen extends StatefulWidget {
  const GalleryScreen({super.key});
  @override
  State<GalleryScreen> createState() => _GalleryScreenState();
}

class _GalleryScreenState extends State<GalleryScreen> {
  final List<Photo> photos = List.generate(30, (i) => Photo.fromId(i + 1));
  int _columns = 2;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Photo Gallery'),
        actions: [
          IconButton(
            icon: Icon(_columns == 2 ? Icons.grid_on : Icons.grid_view),
            onPressed: () => setState(() => _columns = _columns == 2 ? 3 : 2),
          ),
        ],
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(4),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: _columns,
          crossAxisSpacing: 4,
          mainAxisSpacing: 4,
        ),
        itemCount: photos.length,
        itemBuilder: (context, index) {
          final photo = photos[index];
          return GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (_) => PhotoDetailScreen(photo: photo),
              ),
            ),
            child: Hero(
              tag: 'photo_${photo.id}',
              child: CachedNetworkImage(
                imageUrl: photo.thumbUrl,
                fit: BoxFit.cover,
                placeholder: (_, __) => Container(color: Colors.grey.shade900),
                errorWidget: (_, __, ___) => const Icon(Icons.error),
              ),
            ),
          );
        },
      ),
    );
  }
}

class PhotoDetailScreen extends StatelessWidget {
  final Photo photo;
  const PhotoDetailScreen({super.key, required this.photo});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        foregroundColor: Colors.white,
        title: Text(photo.title),
      ),
      body: Center(
        child: Hero(
          tag: 'photo_${photo.id}',
          child: InteractiveViewer(
            child: CachedNetworkImage(
              imageUrl: photo.url,
              fit: BoxFit.contain,
              placeholder: (_, __) => const CircularProgressIndicator(),
              errorWidget: (_, __, ___) => const Icon(Icons.error, color: Colors.white),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## สรุป Part 10

✅ ตั้งค่า Assets ใน pubspec.yaml
✅ Image.asset และ AssetImage
✅ Image.network พร้อม loading/error builders
✅ FadeInImage สำหรับ smooth loading
✅ CachedNetworkImage สำหรับ performance
✅ Image effects: blur, color filters, clip
✅ Icon และ IconButton ทุกสไตล์
✅ SVG ด้วย flutter_svg
✅ CustomPaint วาดรูปและ charts
✅ สร้าง Photo Gallery App ด้วย Hero animation

## แบบฝึกหัด

1. สร้าง Widget ที่แสดงรูปโปรไฟล์พร้อม badge สถานะออนไลน์
2. สร้าง image carousel ด้วย PageView และ dots indicator
3. สร้าง custom progress bar ด้วย CustomPaint
4. เพิ่มฟีเจอร์ zoom และ pan บน detail screen ด้วย InteractiveViewer

---

**ก่อนหน้า:** [Part 09 - Navigation & Routing](part_09.md)
**ต่อไป:** [Part 11 - ListView, GridView, Slivers →](part_11.md)

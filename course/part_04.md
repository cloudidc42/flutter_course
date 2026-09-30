# Part 04: Flutter Widget พื้นฐาน
## ขั้นตอนที่ 31-40

---

## สารบัญ
1. [Text Widget](#ขั้นตอนที่-31-text-widget)
2. [Icon Widget](#ขั้นตอนที่-32-icon-widget)
3. [Image Widget](#ขั้นตอนที่-33-image-widget)
4. [Container Widget](#ขั้นตอนที่-34-container-widget)
5. [Row & Column เบื้องต้น](#ขั้นตอนที่-35-row--column-เบื้องต้น)
6. [Button Widgets](#ขั้นตอนที่-36-button-widgets)
7. [AppBar Widget](#ขั้นตอนที่-37-appbar-widget)
8. [Scaffold Widget](#ขั้นตอนที่-38-scaffold-widget)
9. [SafeArea Widget](#ขั้นตอนที่-39-safearea-widget)
10. [Workshop: Profile Card App](#ขั้นตอนที่-40-workshop-profile-card-app)

---

## ขั้นตอนที่ 31: Text Widget

Text widget เป็น widget พื้นฐานที่ใช้แสดงข้อความใน Flutter รองรับการตกแต่งด้วย TextStyle และ RichText

### Text Widget พื้นฐาน

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Text Widget Demo',
      home: Scaffold(
        appBar: AppBar(title: const Text('Text Widget')),
        body: const TextDemoScreen(),
      ),
    );
  }
}

class TextDemoScreen extends StatelessWidget {
  const TextDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Text พื้นฐาน
          const Text('สวัสดี Flutter!'),

          const SizedBox(height: 16),

          // Text พร้อม style
          const Text(
            'ข้อความใหญ่และหนา',
            style: TextStyle(
              fontSize: 24,           // ขนาดตัวอักษร
              fontWeight: FontWeight.bold,  // ความหนา
              color: Colors.blue,      // สี
              letterSpacing: 2,        // ระยะห่างระหว่างตัวอักษร
            ),
          ),

          const SizedBox(height: 16),

          // Text หลายรูปแบบ
          const Text(
            'ข้อความเอียง',
            style: TextStyle(
              fontStyle: FontStyle.italic,
              color: Colors.grey,
            ),
          ),

          const SizedBox(height: 16),

          // Text มีเส้นใต้
          const Text(
            'ข้อความมีเส้นใต้',
            style: TextStyle(
              decoration: TextDecoration.underline,
              decorationColor: Colors.red,
              decorationStyle: TextDecorationStyle.wavy,
            ),
          ),

          const SizedBox(height: 16),

          // Text เส้นขีดทับ
          const Text(
            'ราคาเดิม 500 บาท',
            style: TextStyle(
              decoration: TextDecoration.lineThrough,
              color: Colors.grey,
              fontSize: 18,
            ),
          ),

          const SizedBox(height: 16),

          // Text Overflow - ข้อความยาวเกิน
          Container(
            width: 200,
            color: Colors.yellow.shade100,
            child: const Text(
              'ข้อความที่ยาวมากและควรจะถูกตัดทิ้งเมื่อเกินความกว้างที่กำหนด',
              overflow: TextOverflow.ellipsis,  // ตัดและใส่ ...
              maxLines: 1,
            ),
          ),

          const SizedBox(height: 8),

          // Text Overflow Clip
          Container(
            width: 200,
            color: Colors.green.shade100,
            child: const Text(
              'ข้อความที่ยาวมากและจะถูกตัดทิ้งเมื่อเกินความกว้าง',
              overflow: TextOverflow.clip,
              maxLines: 1,
            ),
          ),

          const SizedBox(height: 16),

          // Text หลายบรรทัด
          const Text(
            'นี่คือข้อความหลายบรรทัด\nบรรทัดที่สอง\nบรรทัดที่สาม',
            style: TextStyle(
              height: 1.5,  // ระยะห่างระหว่างบรรทัด
            ),
          ),

          const SizedBox(height: 16),

          // Text จัดตำแหน่ง
          const SizedBox(
            width: double.infinity,
            child: Text(
              'ข้อความอยู่ตรงกลาง',
              textAlign: TextAlign.center,
              style: TextStyle(fontSize: 18),
            ),
          ),

          const SizedBox(height: 8),

          const SizedBox(
            width: double.infinity,
            child: Text(
              'ข้อความชิดขวา',
              textAlign: TextAlign.right,
              style: TextStyle(fontSize: 18),
            ),
          ),

          const SizedBox(height: 16),

          // Text พร้อม Shadow
          const Text(
            'ข้อความมีเงา',
            style: TextStyle(
              fontSize: 32,
              color: Colors.purple,
              shadows: [
                Shadow(
                  color: Colors.grey,
                  offset: Offset(2, 2),
                  blurRadius: 4,
                ),
              ],
            ),
          ),

          const SizedBox(height: 16),

          // RichText - ข้อความหลายสไตล์ในบรรทัดเดียว
          RichText(
            text: const TextSpan(
              style: TextStyle(
                fontSize: 18,
                color: Colors.black,
              ),
              children: [
                TextSpan(text: 'ฉันชอบ '),
                TextSpan(
                  text: 'Flutter',
                  style: TextStyle(
                    color: Colors.blue,
                    fontWeight: FontWeight.bold,
                    fontSize: 22,
                  ),
                ),
                TextSpan(text: ' มาก!'),
              ],
            ),
          ),

          const SizedBox(height: 16),

          // Text.rich shorthand
          const Text.rich(
            TextSpan(
              children: [
                TextSpan(
                  text: 'ราคา: ',
                  style: TextStyle(fontSize: 16),
                ),
                TextSpan(
                  text: '฿199',
                  style: TextStyle(
                    fontSize: 20,
                    color: Colors.red,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                TextSpan(
                  text: ' /เดือน',
                  style: TextStyle(
                    fontSize: 14,
                    color: Colors.grey,
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

### TextStyle Properties ทั้งหมด

```dart
const TextStyle(
  // ขนาดและน้ำหนัก
  fontSize: 16.0,
  fontWeight: FontWeight.w400,    // w100 ถึง w900
  fontStyle: FontStyle.normal,    // normal, italic

  // สี
  color: Colors.black,
  backgroundColor: Colors.transparent,

  // Font Family
  fontFamily: 'Roboto',
  fontFamilyFallback: ['Arial', 'sans-serif'],

  // Spacing
  letterSpacing: 0.0,   // ระยะห่างตัวอักษร
  wordSpacing: 0.0,     // ระยะห่างคำ
  height: 1.0,          // ความสูงบรรทัด (เป็น multiplier ของ fontSize)

  // Decoration
  decoration: TextDecoration.none,
  decorationColor: Colors.black,
  decorationStyle: TextDecorationStyle.solid,
  decorationThickness: 1.0,

  // Shadow
  shadows: [
    Shadow(color: Colors.grey, offset: Offset(1, 1), blurRadius: 2),
  ],

  // Overflow
  overflow: TextOverflow.clip,

  // Locale
  locale: Locale('th', 'TH'),
);
```

---

## ขั้นตอนที่ 32: Icon Widget

Icon widget ใช้แสดงไอคอนจาก Material Icons library หรือ custom icon fonts

```dart
import 'package:flutter/material.dart';

class IconDemoScreen extends StatelessWidget {
  const IconDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Icon Widget')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Icon พื้นฐาน
            const Text('Icon พื้นฐาน:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Row(
              children: [
                Icon(Icons.home),
                SizedBox(width: 16),
                Icon(Icons.favorite),
                SizedBox(width: 16),
                Icon(Icons.star),
                SizedBox(width: 16),
                Icon(Icons.settings),
              ],
            ),

            const SizedBox(height: 24),

            // Icon พร้อมสีและขนาด
            const Text('Icon พร้อมสีและขนาด:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Row(
              children: [
                Icon(Icons.favorite, color: Colors.red, size: 32),
                SizedBox(width: 16),
                Icon(Icons.star, color: Colors.amber, size: 40),
                SizedBox(width: 16),
                Icon(Icons.check_circle, color: Colors.green, size: 48),
                SizedBox(width: 16),
                Icon(Icons.warning, color: Colors.orange, size: 56),
              ],
            ),

            const SizedBox(height: 24),

            // Icon ใน Container (ให้ background)
            const Text('Icon ใน Container:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                // วงกลม
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: const BoxDecoration(
                    color: Colors.blue,
                    shape: BoxShape.circle,
                  ),
                  child: const Icon(Icons.phone, color: Colors.white, size: 24),
                ),
                const SizedBox(width: 16),
                // มน
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.green,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Icon(Icons.message, color: Colors.white, size: 24),
                ),
                const SizedBox(width: 16),
                // สี่เหลี่ยม border
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    border: Border.all(color: Colors.purple, width: 2),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Icon(Icons.email, color: Colors.purple, size: 24),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // Cupertino Icons (iOS style)
            const Text('Cupertino Icons (iOS Style):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // ต้องเพิ่ม cupertino_icons ใน pubspec.yaml
            // const Row(
            //   children: [
            //     Icon(CupertinoIcons.home),
            //     Icon(CupertinoIcons.heart),
            //     Icon(CupertinoIcons.star),
            //   ],
            // ),

            // Icon ทั่วไปจาก Material
            Wrap(
              spacing: 16,
              runSpacing: 16,
              children: [
                _buildIconItem(Icons.access_alarm, 'alarm'),
                _buildIconItem(Icons.account_circle, 'account'),
                _buildIconItem(Icons.add_shopping_cart, 'cart'),
                _buildIconItem(Icons.article, 'article'),
                _buildIconItem(Icons.badge, 'badge'),
                _buildIconItem(Icons.camera_alt, 'camera'),
                _buildIconItem(Icons.dashboard, 'dashboard'),
                _buildIconItem(Icons.download, 'download'),
                _buildIconItem(Icons.filter_list, 'filter'),
                _buildIconItem(Icons.gps_fixed, 'gps'),
                _buildIconItem(Icons.history, 'history'),
                _buildIconItem(Icons.info, 'info'),
              ],
            ),

            const SizedBox(height: 24),

            // Icon as Button
            const Text('Icon ทำเป็น Button:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                IconButton(
                  icon: const Icon(Icons.favorite_border),
                  onPressed: () {},
                  color: Colors.red,
                ),
                IconButton(
                  icon: const Icon(Icons.share),
                  onPressed: () {},
                  color: Colors.blue,
                ),
                IconButton(
                  icon: const Icon(Icons.bookmark_border),
                  onPressed: () {},
                  color: Colors.orange,
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildIconItem(IconData icon, String label) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Icon(icon, size: 32, color: Colors.deepPurple),
        const SizedBox(height: 4),
        Text(label, style: const TextStyle(fontSize: 10)),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 33: Image Widget

Image widget ใช้แสดงรูปภาพจากหลายแหล่ง: network, assets, file, memory

```dart
import 'package:flutter/material.dart';

class ImageDemoScreen extends StatelessWidget {
  const ImageDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Image Widget')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            // 1. Network Image
            const Text('1. Network Image:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Image.network(
              'https://picsum.photos/300/200',
              width: 300,
              height: 200,
              fit: BoxFit.cover,
              // Loading builder - แสดง loading indicator
              loadingBuilder: (context, child, loadingProgress) {
                if (loadingProgress == null) return child;
                return SizedBox(
                  width: 300,
                  height: 200,
                  child: Center(
                    child: CircularProgressIndicator(
                      value: loadingProgress.expectedTotalBytes != null
                          ? loadingProgress.cumulativeBytesLoaded /
                              loadingProgress.expectedTotalBytes!
                          : null,
                    ),
                  ),
                );
              },
              // Error builder - แสดงเมื่อโหลดไม่ได้
              errorBuilder: (context, error, stackTrace) {
                return Container(
                  width: 300,
                  height: 200,
                  color: Colors.grey.shade200,
                  child: const Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(Icons.broken_image, size: 48, color: Colors.grey),
                      Text('โหลดรูปไม่ได้'),
                    ],
                  ),
                );
              },
            ),

            const SizedBox(height: 24),

            // 2. BoxFit ต่างๆ
            const Text('2. BoxFit ต่างๆ:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Row(
              children: [
                _buildBoxFitDemo('cover', BoxFit.cover),
                const SizedBox(width: 8),
                _buildBoxFitDemo('contain', BoxFit.contain),
                const SizedBox(width: 8),
                _buildBoxFitDemo('fill', BoxFit.fill),
              ],
            ),

            const SizedBox(height: 8),

            Row(
              children: [
                _buildBoxFitDemo('fitWidth', BoxFit.fitWidth),
                const SizedBox(width: 8),
                _buildBoxFitDemo('fitHeight', BoxFit.fitHeight),
                const SizedBox(width: 8),
                _buildBoxFitDemo('none', BoxFit.none),
              ],
            ),

            const SizedBox(height: 24),

            // 3. Image เป็นวงกลม (ClipOval)
            const Text('3. Image วงกลม:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                ClipOval(
                  child: Image.network(
                    'https://picsum.photos/100/100?random=1',
                    width: 100,
                    height: 100,
                    fit: BoxFit.cover,
                  ),
                ),
                const SizedBox(width: 16),
                // CircleAvatar ง่ายกว่า
                const CircleAvatar(
                  radius: 50,
                  backgroundImage: NetworkImage(
                    'https://picsum.photos/100/100?random=2',
                  ),
                ),
                const SizedBox(width: 16),
                const CircleAvatar(
                  radius: 50,
                  backgroundColor: Colors.blue,
                  child: Text('AB', style: TextStyle(color: Colors.white, fontSize: 24)),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // 4. Image พร้อม decoration
            const Text('4. Image ใน Container:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Container(
              width: double.infinity,
              height: 200,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.3),
                    blurRadius: 8,
                    offset: const Offset(0, 4),
                  ),
                ],
              ),
              child: ClipRRect(
                borderRadius: BorderRadius.circular(16),
                child: Image.network(
                  'https://picsum.photos/400/200?random=3',
                  fit: BoxFit.cover,
                ),
              ),
            ),

            const SizedBox(height: 24),

            // 5. Asset Image (ต้องเพิ่มใน pubspec.yaml ก่อน)
            const Text('5. Asset Image (ตัวอย่าง):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text(
              '// ใน pubspec.yaml:\n'
              'flutter:\n'
              '  assets:\n'
              '    - assets/images/logo.png\n\n'
              '// ในโค้ด:\n'
              'Image.asset("assets/images/logo.png")',
              style: TextStyle(
                fontFamily: 'monospace',
                fontSize: 12,
                backgroundColor: Color(0xFFEEEEEE),
              ),
            ),

            const SizedBox(height: 24),

            // 6. Fade In Image
            const Text('6. FadeInImage:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            FadeInImage(
              placeholder: const AssetImage('assets/images/placeholder.png'),
              image: const NetworkImage('https://picsum.photos/300/200?random=4'),
              width: 300,
              height: 200,
              fit: BoxFit.cover,
              imageErrorBuilder: (context, error, stackTrace) {
                return Container(
                  width: 300,
                  height: 200,
                  color: Colors.grey.shade300,
                  child: const Icon(Icons.broken_image, size: 48),
                );
              },
            ),

            const SizedBox(height: 24),

            // 7. Image พร้อม Stack (overlay)
            const Text('7. Image พร้อม Overlay:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Stack(
              children: [
                ClipRRect(
                  borderRadius: BorderRadius.circular(12),
                  child: Image.network(
                    'https://picsum.photos/400/200?random=5',
                    width: double.infinity,
                    height: 200,
                    fit: BoxFit.cover,
                  ),
                ),
                Positioned.fill(
                  child: Container(
                    decoration: BoxDecoration(
                      borderRadius: BorderRadius.circular(12),
                      gradient: LinearGradient(
                        begin: Alignment.topCenter,
                        end: Alignment.bottomCenter,
                        colors: [
                          Colors.transparent,
                          Colors.black.withOpacity(0.7),
                        ],
                      ),
                    ),
                  ),
                ),
                const Positioned(
                  bottom: 16,
                  left: 16,
                  child: Text(
                    'ชื่อรูปภาพ',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildBoxFitDemo(String label, BoxFit fit) {
    return Column(
      children: [
        Container(
          width: 100,
          height: 80,
          decoration: BoxDecoration(
            border: Border.all(color: Colors.grey),
          ),
          child: Image.network(
            'https://picsum.photos/200/100',
            fit: fit,
          ),
        ),
        Text(label, style: const TextStyle(fontSize: 11)),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 34: Container Widget

Container เป็น widget อเนกประสงค์ที่รวม padding, margin, decoration, size, alignment ไว้ในที่เดียว

```dart
import 'package:flutter/material.dart';

class ContainerDemoScreen extends StatelessWidget {
  const ContainerDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Container Widget')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [

            // Container พื้นฐาน
            Container(
              width: 200,
              height: 100,
              color: Colors.blue,
              child: const Center(
                child: Text('Container พื้นฐาน',
                    style: TextStyle(color: Colors.white)),
              ),
            ),

            const SizedBox(height: 16),

            // Container พร้อม Padding และ Margin
            Container(
              margin: const EdgeInsets.all(8),         // ระยะภายนอก
              padding: const EdgeInsets.all(16),        // ระยะภายใน
              color: Colors.green.shade100,
              child: const Text('Padding และ Margin'),
            ),

            const SizedBox(height: 16),

            // Container พร้อม BoxDecoration
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(20),
              decoration: BoxDecoration(
                // สี gradient
                gradient: const LinearGradient(
                  colors: [Colors.purple, Colors.blue],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(16),
                boxShadow: [
                  BoxShadow(
                    color: Colors.purple.withOpacity(0.4),
                    blurRadius: 10,
                    offset: const Offset(0, 4),
                  ),
                ],
              ),
              child: const Text(
                'Container พร้อม Gradient',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
                textAlign: TextAlign.center,
              ),
            ),

            const SizedBox(height: 16),

            // Container พร้อม Border
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Colors.white,
                border: Border.all(color: Colors.orange, width: 2),
                borderRadius: BorderRadius.circular(8),
              ),
              child: const Text('Container พร้อม Border'),
            ),

            const SizedBox(height: 16),

            // Container พร้อม Border แบบเฉพาะด้าน
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              decoration: const BoxDecoration(
                color: Colors.white,
                border: Border(
                  left: BorderSide(color: Colors.red, width: 4),
                ),
              ),
              child: const Text('Border ด้านซ้ายอย่างเดียว'),
            ),

            const SizedBox(height: 16),

            // Container พร้อม Image Background
            Container(
              width: double.infinity,
              height: 150,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(12),
                image: const DecorationImage(
                  image: NetworkImage('https://picsum.photos/400/150'),
                  fit: BoxFit.cover,
                ),
              ),
              child: Container(
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(12),
                  color: Colors.black.withOpacity(0.3),
                ),
                child: const Center(
                  child: Text(
                    'Background Image',
                    style: TextStyle(
                      color: Colors.white,
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Container วงกลม
            Container(
              width: 100,
              height: 100,
              decoration: const BoxDecoration(
                color: Colors.teal,
                shape: BoxShape.circle,
              ),
              child: const Center(
                child: Text(
                  'วงกลม',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Container Alignment
            Container(
              width: 200,
              height: 100,
              color: Colors.amber.shade100,
              alignment: Alignment.bottomRight,  // จัดตำแหน่ง child
              child: const Text('Bottom Right'),
            ),

            const SizedBox(height: 16),

            // ConstrainedBox (จำกัดขนาดแบบ min/max)
            Container(
              color: Colors.pink.shade100,
              constraints: const BoxConstraints(
                minWidth: 100,
                maxWidth: 300,
                minHeight: 50,
                maxHeight: 150,
              ),
              child: const Padding(
                padding: EdgeInsets.all(8),
                child: Text('ConstrainedBox: ขนาดจะอยู่ใน range ที่กำหนด'),
              ),
            ),

            const SizedBox(height: 16),

            // Transform - หมุน/ย่อ/ขยาย container
            Transform.rotate(
              angle: 0.1,  // หน่วยเป็น radian
              child: Container(
                width: 200,
                height: 80,
                color: Colors.cyan.shade200,
                child: const Center(child: Text('Rotated Container')),
              ),
            ),

            const SizedBox(height: 32),

            // Card (container สำเร็จรูป)
            Card(
              elevation: 8,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
              ),
              child: const Padding(
                padding: EdgeInsets.all(20),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Card Title',
                        style: TextStyle(
                            fontSize: 18, fontWeight: FontWeight.bold)),
                    SizedBox(height: 8),
                    Text('Card ก็คือ Container ที่มี elevation และ shape สำเร็จรูป'),
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

## ขั้นตอนที่ 35: Row & Column เบื้องต้น

Row จัดเรียง widget แนวนอน, Column จัดเรียงแนวตั้ง

```dart
import 'package:flutter/material.dart';

class RowColumnDemoScreen extends StatelessWidget {
  const RowColumnDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Row & Column')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Row พื้นฐาน
            const Text('Row พื้นฐาน:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                _buildBox(Colors.red, 'A'),
                _buildBox(Colors.green, 'B'),
                _buildBox(Colors.blue, 'C'),
              ],
            ),

            const SizedBox(height: 24),

            // MainAxisAlignment
            const Text('MainAxisAlignment (แกนหลัก Row = แนวนอน):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            _buildRowDemo('start', MainAxisAlignment.start),
            _buildRowDemo('center', MainAxisAlignment.center),
            _buildRowDemo('end', MainAxisAlignment.end),
            _buildRowDemo('spaceBetween', MainAxisAlignment.spaceBetween),
            _buildRowDemo('spaceAround', MainAxisAlignment.spaceAround),
            _buildRowDemo('spaceEvenly', MainAxisAlignment.spaceEvenly),

            const SizedBox(height: 24),

            // CrossAxisAlignment
            const Text('CrossAxisAlignment (แกนรอง Row = แนวตั้ง):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Container(
              height: 80,
              color: Colors.grey.shade100,
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _buildBox(Colors.red, 'start', height: 40),
                  _buildBox(Colors.green, 'start', height: 60),
                  _buildBox(Colors.blue, 'start', height: 50),
                ],
              ),
            ),

            const SizedBox(height: 8),

            Container(
              height: 80,
              color: Colors.grey.shade100,
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  _buildBox(Colors.red, 'center', height: 40),
                  _buildBox(Colors.green, 'center', height: 60),
                  _buildBox(Colors.blue, 'center', height: 50),
                ],
              ),
            ),

            const SizedBox(height: 8),

            Container(
              height: 80,
              color: Colors.grey.shade100,
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.end,
                children: [
                  _buildBox(Colors.red, 'end', height: 40),
                  _buildBox(Colors.green, 'end', height: 60),
                  _buildBox(Colors.blue, 'end', height: 50),
                ],
              ),
            ),

            const SizedBox(height: 24),

            // Column
            const Text('Column:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            SizedBox(
              height: 200,
              width: double.infinity,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  _buildBox(Colors.red, 'Item 1', width: 150),
                  _buildBox(Colors.green, 'Item 2', width: 200),
                  _buildBox(Colors.blue, 'Item 3', width: 100),
                ],
              ),
            ),

            const SizedBox(height: 24),

            // Flexible
            const Text('Flexible:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                Flexible(
                  flex: 2,
                  child: Container(
                    height: 50,
                    color: Colors.red.shade200,
                    child: const Center(child: Text('flex: 2')),
                  ),
                ),
                Flexible(
                  flex: 1,
                  child: Container(
                    height: 50,
                    color: Colors.green.shade200,
                    child: const Center(child: Text('flex: 1')),
                  ),
                ),
                Flexible(
                  flex: 1,
                  child: Container(
                    height: 50,
                    color: Colors.blue.shade200,
                    child: const Center(child: Text('flex: 1')),
                  ),
                ),
              ],
            ),

            const SizedBox(height: 16),

            // Expanded
            const Text('Expanded:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Row(
              children: [
                Container(
                  width: 60,
                  height: 50,
                  color: Colors.purple.shade200,
                  child: const Center(child: Text('Fixed')),
                ),
                Expanded(
                  child: Container(
                    height: 50,
                    color: Colors.orange.shade200,
                    child: const Center(child: Text('Expanded (เต็มพื้นที่)')),
                  ),
                ),
                Container(
                  width: 60,
                  height: 50,
                  color: Colors.teal.shade200,
                  child: const Center(child: Text('Fixed')),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildBox(Color color, String label,
      {double? width, double? height}) {
    return Container(
      width: width ?? 60,
      height: height ?? 60,
      margin: const EdgeInsets.all(4),
      color: color,
      child: Center(
        child: Text(
          label,
          style: const TextStyle(
            color: Colors.white,
            fontWeight: FontWeight.bold,
            fontSize: 11,
          ),
        ),
      ),
    );
  }

  Widget _buildRowDemo(String label, MainAxisAlignment alignment) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(label, style: const TextStyle(fontSize: 12, color: Colors.grey)),
        Container(
          color: Colors.grey.shade100,
          child: Row(
            mainAxisAlignment: alignment,
            children: [
              _buildBox(Colors.red, 'A'),
              _buildBox(Colors.green, 'B'),
              _buildBox(Colors.blue, 'C'),
            ],
          ),
        ),
        const SizedBox(height: 8),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 36: Button Widgets

Flutter มี Button widget หลายประเภทสำหรับ use case ต่างๆ

```dart
import 'package:flutter/material.dart';

class ButtonDemoScreen extends StatefulWidget {
  const ButtonDemoScreen({super.key});

  @override
  State<ButtonDemoScreen> createState() => _ButtonDemoScreenState();
}

class _ButtonDemoScreenState extends State<ButtonDemoScreen> {
  String _lastPressed = 'ยังไม่ได้กดปุ่ม';

  void _showPressed(String button) {
    setState(() {
      _lastPressed = 'กด: $button';
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Button Widgets')),
      // FAB - FloatingActionButton
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showPressed('FAB'),
        child: const Icon(Icons.add),
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // แสดงปุ่มที่กดล่าสุด
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(12),
              color: Colors.grey.shade100,
              child: Text(_lastPressed,
                  style: const TextStyle(fontSize: 16)),
            ),

            const SizedBox(height: 24),

            // 1. ElevatedButton
            const Text('1. ElevatedButton:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showPressed('ElevatedButton'),
              child: const Text('ElevatedButton พื้นฐาน'),
            ),

            const SizedBox(height: 8),

            // ElevatedButton แบบกำหนด style
            ElevatedButton(
              onPressed: () => _showPressed('ElevatedButton สีม่วง'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.purple,
                foregroundColor: Colors.white,
                padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(30),
                ),
                elevation: 8,
              ),
              child: const Text('ElevatedButton Custom', fontSize: 16),
            ),

            const SizedBox(height: 8),

            // ElevatedButton พร้อม Icon
            ElevatedButton.icon(
              onPressed: () => _showPressed('ElevatedButton Icon'),
              icon: const Icon(Icons.send),
              label: const Text('ส่งข้อมูล'),
            ),

            const SizedBox(height: 8),

            // ElevatedButton ปิดการใช้งาน
            const ElevatedButton(
              onPressed: null,  // null = disabled
              child: Text('Disabled Button'),
            ),

            const SizedBox(height: 24),

            // 2. TextButton
            const Text('2. TextButton:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            TextButton(
              onPressed: () => _showPressed('TextButton'),
              child: const Text('TextButton พื้นฐาน'),
            ),

            TextButton.icon(
              onPressed: () => _showPressed('TextButton Icon'),
              icon: const Icon(Icons.favorite),
              label: const Text('ถูกใจ'),
            ),

            TextButton(
              onPressed: () => _showPressed('TextButton Custom'),
              style: TextButton.styleFrom(
                foregroundColor: Colors.red,
                textStyle: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                  decoration: TextDecoration.underline,
                ),
              ),
              child: const Text('TextButton Custom Style'),
            ),

            const SizedBox(height: 24),

            // 3. OutlinedButton
            const Text('3. OutlinedButton:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            OutlinedButton(
              onPressed: () => _showPressed('OutlinedButton'),
              child: const Text('OutlinedButton พื้นฐาน'),
            ),

            const SizedBox(height: 8),

            OutlinedButton(
              onPressed: () => _showPressed('OutlinedButton Custom'),
              style: OutlinedButton.styleFrom(
                foregroundColor: Colors.green,
                side: const BorderSide(color: Colors.green, width: 2),
                padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(8),
                ),
              ),
              child: const Text('OutlinedButton Custom'),
            ),

            const SizedBox(height: 8),

            OutlinedButton.icon(
              onPressed: () => _showPressed('OutlinedButton Icon'),
              icon: const Icon(Icons.download),
              label: const Text('ดาวน์โหลด'),
            ),

            const SizedBox(height: 24),

            // 4. IconButton
            const Text('4. IconButton:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Row(
              children: [
                IconButton(
                  icon: const Icon(Icons.favorite_border),
                  onPressed: () => _showPressed('IconButton Heart'),
                  color: Colors.red,
                  tooltip: 'ถูกใจ',
                ),
                IconButton(
                  icon: const Icon(Icons.share),
                  onPressed: () => _showPressed('IconButton Share'),
                  color: Colors.blue,
                  tooltip: 'แชร์',
                ),
                IconButton(
                  icon: const Icon(Icons.delete),
                  onPressed: () => _showPressed('IconButton Delete'),
                  color: Colors.grey,
                  tooltip: 'ลบ',
                ),
                // IconButton ใหญ่
                IconButton(
                  icon: const Icon(Icons.play_arrow),
                  iconSize: 48,
                  onPressed: () => _showPressed('IconButton Large'),
                  color: Colors.green,
                  style: IconButton.styleFrom(
                    backgroundColor: Colors.green.shade100,
                  ),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // 5. FloatingActionButton Variants
            const Text('5. FAB Variants:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                // Small FAB
                FloatingActionButton.small(
                  heroTag: 'fab_small',
                  onPressed: () => _showPressed('FAB Small'),
                  child: const Icon(Icons.add),
                ),
                // Regular FAB
                FloatingActionButton(
                  heroTag: 'fab_regular',
                  onPressed: () => _showPressed('FAB Regular'),
                  backgroundColor: Colors.orange,
                  child: const Icon(Icons.edit),
                ),
                // Large FAB
                FloatingActionButton.large(
                  heroTag: 'fab_large',
                  onPressed: () => _showPressed('FAB Large'),
                  backgroundColor: Colors.purple,
                  child: const Icon(Icons.camera_alt),
                ),
                // Extended FAB
                FloatingActionButton.extended(
                  heroTag: 'fab_extended',
                  onPressed: () => _showPressed('FAB Extended'),
                  icon: const Icon(Icons.navigation),
                  label: const Text('Navigate'),
                ),
              ],
            ),

            const SizedBox(height: 24),

            // 6. Custom Button (GestureDetector)
            const Text('6. Custom Button:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            GestureDetector(
              onTap: () => _showPressed('Custom GestureDetector'),
              child: Container(
                padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
                decoration: BoxDecoration(
                  gradient: const LinearGradient(
                    colors: [Colors.orange, Colors.red],
                  ),
                  borderRadius: BorderRadius.circular(30),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.orange.withOpacity(0.4),
                      blurRadius: 8,
                      offset: const Offset(0, 4),
                    ),
                  ],
                ),
                child: const Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Icon(Icons.rocket_launch, color: Colors.white),
                    SizedBox(width: 8),
                    Text(
                      'Custom Button',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // InkWell (ripple effect)
            InkWell(
              onTap: () => _showPressed('InkWell'),
              borderRadius: BorderRadius.circular(8),
              child: Container(
                padding: const EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.teal.shade100,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Text(
                  'InkWell - มี ripple effect',
                  style: TextStyle(fontSize: 16),
                ),
              ),
            ),

            const SizedBox(height: 80), // space for FAB
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 37: AppBar Widget

AppBar เป็น top navigation bar ที่ใช้บ่อยใน Material Design

```dart
import 'package:flutter/material.dart';

// ตัวอย่าง AppBar หลายแบบ
class AppBarDemoScreen extends StatefulWidget {
  const AppBarDemoScreen({super.key});

  @override
  State<AppBarDemoScreen> createState() => _AppBarDemoScreenState();
}

class _AppBarDemoScreenState extends State<AppBarDemoScreen> {
  int _currentDemo = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // เลือก AppBar ตาม demo
      appBar: _buildAppBar(),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          const Text(
            'เลือก AppBar Style:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          ...List.generate(7, (index) {
            return RadioListTile<int>(
              title: Text(_getDemoName(index)),
              value: index,
              groupValue: _currentDemo,
              onChanged: (value) => setState(() => _currentDemo = value!),
            );
          }),
        ],
      ),
    );
  }

  String _getDemoName(int index) {
    const names = [
      'Basic AppBar',
      'AppBar พร้อม Actions',
      'AppBar พร้อม Leading',
      'Centered Title',
      'Colored AppBar',
      'AppBar พร้อม Bottom (TabBar)',
      'SliverAppBar (Collapsible)',
    ];
    return names[index];
  }

  PreferredSizeWidget _buildAppBar() {
    switch (_currentDemo) {
      case 0:
        // Basic AppBar
        return AppBar(
          title: const Text('Basic AppBar'),
        );

      case 1:
        // AppBar พร้อม Actions
        return AppBar(
          title: const Text('AppBar + Actions'),
          actions: [
            IconButton(
              icon: const Icon(Icons.search),
              onPressed: () {},
              tooltip: 'ค้นหา',
            ),
            IconButton(
              icon: const Icon(Icons.notifications),
              onPressed: () {},
              tooltip: 'แจ้งเตือน',
            ),
            PopupMenuButton<String>(
              onSelected: (value) {},
              itemBuilder: (context) => [
                const PopupMenuItem(value: 'profile', child: Text('โปรไฟล์')),
                const PopupMenuItem(value: 'settings', child: Text('ตั้งค่า')),
                const PopupMenuItem(value: 'logout', child: Text('ออกจากระบบ')),
              ],
            ),
          ],
        );

      case 2:
        // AppBar พร้อม Leading
        return AppBar(
          leading: IconButton(
            icon: const Icon(Icons.menu),
            onPressed: () {},
          ),
          title: const Text('AppBar + Leading'),
        );

      case 3:
        // Centered Title
        return AppBar(
          title: const Text('Centered Title'),
          centerTitle: true,
          actions: [
            IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
          ],
        );

      case 4:
        // Colored AppBar
        return AppBar(
          title: const Text(
            'Colored AppBar',
            style: TextStyle(color: Colors.white),
          ),
          backgroundColor: Colors.deepPurple,
          iconTheme: const IconThemeData(color: Colors.white),
          actions: [
            IconButton(
              icon: const Icon(Icons.favorite),
              onPressed: () {},
            ),
          ],
        );

      case 5:
        // AppBar พร้อม TabBar
        return AppBar(
          title: const Text('AppBar + Tabs'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.home), text: 'หน้าแรก'),
              Tab(icon: Icon(Icons.search), text: 'ค้นหา'),
              Tab(icon: Icon(Icons.person), text: 'โปรไฟล์'),
            ],
          ),
        ) as PreferredSizeWidget;

      default:
        return AppBar(title: const Text('AppBar'));
    }
  }
}

// SliverAppBar ตัวอย่าง
class SliverAppBarDemo extends StatelessWidget {
  const SliverAppBarDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 200,           // ความสูงเมื่อขยาย
            floating: false,               // AppBar จะ float ไหมเมื่อ scroll up
            pinned: true,                  // คงอยู่ด้านบนเมื่อ scroll
            snap: false,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('SliverAppBar'),
              background: Image.network(
                'https://picsum.photos/400/200',
                fit: BoxFit.cover,
              ),
            ),
          ),
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                title: Text('Item ${index + 1}'),
              ),
              childCount: 30,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 38: Scaffold Widget

Scaffold เป็น structure หลักของหน้า Material Design รวม AppBar, Body, Drawer, FAB, SnackBar

```dart
import 'package:flutter/material.dart';

class ScaffoldDemoScreen extends StatefulWidget {
  const ScaffoldDemoScreen({super.key});

  @override
  State<ScaffoldDemoScreen> createState() => _ScaffoldDemoScreenState();
}

class _ScaffoldDemoScreenState extends State<ScaffoldDemoScreen> {
  int _selectedIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // AppBar
      appBar: AppBar(
        title: const Text('Scaffold Demo'),
        actions: [
          IconButton(
            icon: const Icon(Icons.info_outline),
            onPressed: () {
              // แสดง SnackBar
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: const Text('นี่คือ SnackBar!'),
                  action: SnackBarAction(
                    label: 'ปิด',
                    onPressed: () {
                      ScaffoldMessenger.of(context).hideCurrentSnackBar();
                    },
                  ),
                  behavior: SnackBarBehavior.floating,
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(10),
                  ),
                ),
              );
            },
          ),
        ],
      ),

      // Drawer (เมนูด้านซ้าย)
      drawer: Drawer(
        child: ListView(
          padding: EdgeInsets.zero,
          children: [
            const DrawerHeader(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  colors: [Colors.blue, Colors.purple],
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisAlignment: MainAxisAlignment.end,
                children: [
                  CircleAvatar(
                    radius: 30,
                    backgroundColor: Colors.white,
                    child: Icon(Icons.person, size: 36, color: Colors.blue),
                  ),
                  SizedBox(height: 8),
                  Text('สมชาย ใจดี',
                      style: TextStyle(color: Colors.white, fontSize: 16)),
                  Text('somchai@example.com',
                      style: TextStyle(color: Colors.white70, fontSize: 12)),
                ],
              ),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('หน้าแรก'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.person),
              title: const Text('โปรไฟล์'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('ตั้งค่า'),
              onTap: () => Navigator.pop(context),
            ),
            const Divider(),
            ListTile(
              leading: const Icon(Icons.logout, color: Colors.red),
              title: const Text('ออกจากระบบ',
                  style: TextStyle(color: Colors.red)),
              onTap: () => Navigator.pop(context),
            ),
          ],
        ),
      ),

      // End Drawer (เมนูด้านขวา)
      endDrawer: Drawer(
        child: ListView(
          children: [
            const DrawerHeader(
              decoration: BoxDecoration(color: Colors.teal),
              child: Text('End Drawer',
                  style: TextStyle(color: Colors.white, fontSize: 20)),
            ),
            const ListTile(title: Text('Item 1')),
            const ListTile(title: Text('Item 2')),
          ],
        ),
      ),

      // Body หลัก
      body: _buildBody(),

      // Bottom Navigation Bar
      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (index) =>
            setState(() => _selectedIndex = index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'หน้าแรก',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: 'ค้นหา',
          ),
          NavigationDestination(
            icon: Icon(Icons.favorite_outline),
            selectedIcon: Icon(Icons.favorite),
            label: 'ถูกใจ',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'โปรไฟล์',
          ),
        ],
      ),

      // FAB
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _showBottomSheet(context);
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildBody() {
    const pages = [
      Center(child: Text('หน้าแรก', style: TextStyle(fontSize: 24))),
      Center(child: Text('ค้นหา', style: TextStyle(fontSize: 24))),
      Center(child: Text('ถูกใจ', style: TextStyle(fontSize: 24))),
      Center(child: Text('โปรไฟล์', style: TextStyle(fontSize: 24))),
    ];
    return pages[_selectedIndex];
  }

  void _showBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => Container(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Container(
              width: 40,
              height: 4,
              decoration: BoxDecoration(
                color: Colors.grey.shade300,
                borderRadius: BorderRadius.circular(2),
              ),
            ),
            const SizedBox(height: 20),
            const Text('เพิ่มรายการใหม่',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 16),
            ListTile(
              leading: const Icon(Icons.photo_camera),
              title: const Text('ถ่ายรูป'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.photo_library),
              title: const Text('เลือกจากอัลบั้ม'),
              onTap: () => Navigator.pop(context),
            ),
            ListTile(
              leading: const Icon(Icons.folder),
              title: const Text('เลือกจากไฟล์'),
              onTap: () => Navigator.pop(context),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 39: SafeArea Widget

SafeArea ป้องกันไม่ให้ content ถูกบัง notch, status bar, home indicator

```dart
import 'package:flutter/material.dart';

class SafeAreaDemoScreen extends StatelessWidget {
  const SafeAreaDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // MediaQuery - ข้อมูลหน้าจอ
    final mediaQuery = MediaQuery.of(context);
    final padding = mediaQuery.padding;
    final size = mediaQuery.size;

    return Scaffold(
      // ไม่ใช้ AppBar เพื่อ demo SafeArea
      body: Column(
        children: [
          // ไม่มี SafeArea - เนื้อหาอาจทับ status bar
          Container(
            color: Colors.red.shade100,
            height: 50,
            child: const Center(
              child: Text('ไม่มี SafeArea - อาจทับ status bar!'),
            ),
          ),

          // มี SafeArea - ปลอดภัย
          SafeArea(
            // กำหนดว่าต้องการ safe จากด้านไหน
            top: true,
            bottom: true,
            left: true,
            right: true,
            child: Container(
              color: Colors.green.shade100,
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'SafeArea ตัวอย่าง',
                    style: TextStyle(
                      fontSize: 20,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 16),
                  Text('Screen Size: ${size.width.toInt()} x ${size.height.toInt()}'),
                  Text('Status Bar Height: ${padding.top}'),
                  Text('Bottom Bar Height: ${padding.bottom}'),
                  Text('Pixel Ratio: ${mediaQuery.devicePixelRatio}'),
                  Text('Text Scale: ${mediaQuery.textScaler}'),
                  Text('Orientation: ${mediaQuery.orientation.name}'),
                  Text('Platform: ${Theme.of(context).platform.name}'),

                  const SizedBox(height: 16),

                  // SafeArea แบบเลือกด้าน
                  const Text('SafeArea เฉพาะด้านล่าง:', style: TextStyle(fontWeight: FontWeight.bold)),
                ],
              ),
            ),
          ),

          const Expanded(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('การใช้ SafeArea ที่ถูกต้อง:'),
                  SizedBox(height: 8),
                  Text(
                    '1. ใช้กับหน้าที่ไม่มี AppBar\n'
                    '2. ใช้กับ BottomSheet ที่มีปุ่มสำคัญ\n'
                    '3. ใช้กับ Custom navigation\n'
                    '4. ใช้กับ Full-screen content\n'
                    '5. ไม่จำเป็นต้องใช้ถ้ามี AppBar + Scaffold\n\n'
                    'Scaffold จะจัดการ SafeArea ให้อัตโนมัติสำหรับ AppBar',
                  ),
                ],
              ),
            ),
          ),

          // Bottom SafeArea
          SafeArea(
            top: false,  // ไม่ต้องการด้านบน
            child: Container(
              color: Colors.blue.shade100,
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              child: const Text(
                'SafeArea เฉพาะด้านล่าง - ป้องกัน home indicator',
                textAlign: TextAlign.center,
              ),
            ),
          ),
        ],
      ),
    );
  }
}

// ตัวอย่างการใช้ MediaQuery เต็มรูปแบบ
class MediaQueryDemoScreen extends StatelessWidget {
  const MediaQueryDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final mq = MediaQuery.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('MediaQuery Info')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _buildInfoCard('Size', '${mq.size.width.toInt()} x ${mq.size.height.toInt()}'),
          _buildInfoCard('Pixel Ratio', '${mq.devicePixelRatio}'),
          _buildInfoCard('Text Scale', '${mq.textScaler}'),
          _buildInfoCard('Orientation', mq.orientation.name),
          _buildInfoCard('Platform Brightness',
              mq.platformBrightness.name),
          _buildInfoCard('Padding (top)', '${mq.padding.top}'),
          _buildInfoCard('Padding (bottom)', '${mq.padding.bottom}'),
          _buildInfoCard('View Insets Bottom',
              '${mq.viewInsets.bottom}'),
          _buildInfoCard('System Gesture Insets',
              mq.systemGestureInsets.toString()),
        ],
      ),
    );
  }

  Widget _buildInfoCard(String label, String value) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(label, style: const TextStyle(fontWeight: FontWeight.bold)),
            Text(value, style: const TextStyle(color: Colors.blue)),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 40: Workshop - Profile Card App

สร้างแอป Profile Card ที่รวมทุก widget ที่เรียนมา

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ProfileCardApp());
}

class ProfileCardApp extends StatelessWidget {
  const ProfileCardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Profile Card App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const ProfileScreen(),
    );
  }
}

// Model สำหรับ User
class UserProfile {
  final String name;
  final String title;
  final String location;
  final String bio;
  final String avatarUrl;
  final int followers;
  final int following;
  final int posts;
  final List<String> skills;
  final List<SocialLink> socialLinks;

  const UserProfile({
    required this.name,
    required this.title,
    required this.location,
    required this.bio,
    required this.avatarUrl,
    required this.followers,
    required this.following,
    required this.posts,
    required this.skills,
    required this.socialLinks,
  });
}

class SocialLink {
  final IconData icon;
  final String label;
  final Color color;

  const SocialLink({
    required this.icon,
    required this.label,
    required this.color,
  });
}

class ProfileScreen extends StatefulWidget {
  const ProfileScreen({super.key});

  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  bool _isFollowing = false;

  final UserProfile _user = const UserProfile(
    name: 'สมชาย พัฒนาดี',
    title: 'Flutter Developer',
    location: 'กรุงเทพมหานคร, ไทย',
    bio: 'นักพัฒนาแอปพลิเคชันมือถือด้วย Flutter ชอบสร้างสิ่งใหม่ๆ'
        ' มีประสบการณ์ 5 ปีในการพัฒนาแอปสำหรับ iOS และ Android',
    avatarUrl: 'https://picsum.photos/200/200?random=10',
    followers: 1250,
    following: 380,
    posts: 47,
    skills: ['Flutter', 'Dart', 'Firebase', 'REST API', 'Git', 'UI/UX'],
    socialLinks: [
      SocialLink(icon: Icons.code, label: 'GitHub', color: Colors.black87),
      SocialLink(icon: Icons.link, label: 'LinkedIn', color: Color(0xFF0077B5)),
      SocialLink(icon: Icons.web, label: 'Website', color: Colors.blue),
      SocialLink(icon: Icons.email, label: 'Email', color: Colors.red),
    ],
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Collapsible Header
          SliverAppBar(
            expandedHeight: 280,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              background: _buildHeader(),
            ),
            actions: [
              IconButton(
                icon: const Icon(Icons.share, color: Colors.white),
                onPressed: () => _showShareDialog(),
              ),
              IconButton(
                icon: const Icon(Icons.more_vert, color: Colors.white),
                onPressed: () => _showOptionsMenu(),
              ),
            ],
          ),

          // Body Content
          SliverToBoxAdapter(
            child: Column(
              children: [
                _buildProfileInfo(),
                const Divider(height: 1),
                _buildStats(),
                const Divider(height: 1),
                _buildActions(),
                const Divider(height: 1),
                _buildSkills(),
                const Divider(height: 1),
                _buildSocialLinks(),
                const Divider(height: 1),
                _buildPosts(),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildHeader() {
    return Stack(
      fit: StackFit.expand,
      children: [
        // Background gradient
        Container(
          decoration: const BoxDecoration(
            gradient: LinearGradient(
              begin: Alignment.topLeft,
              end: Alignment.bottomRight,
              colors: [Color(0xFF6C63FF), Color(0xFF3B82F6)],
            ),
          ),
        ),

        // Pattern overlay
        Opacity(
          opacity: 0.1,
          child: GridView.builder(
            physics: const NeverScrollableScrollPhysics(),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 8,
            ),
            itemBuilder: (_, __) => const Icon(Icons.circle_outlined,
                color: Colors.white, size: 20),
            itemCount: 100,
          ),
        ),

        // Avatar and Name
        Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const SizedBox(height: 40),
            // Avatar
            Container(
              decoration: BoxDecoration(
                shape: BoxShape.circle,
                border: Border.all(color: Colors.white, width: 4),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.3),
                    blurRadius: 10,
                  ),
                ],
              ),
              child: CircleAvatar(
                radius: 50,
                backgroundImage: NetworkImage(_user.avatarUrl),
              ),
            ),
            const SizedBox(height: 12),
            Text(
              _user.name,
              style: const TextStyle(
                color: Colors.white,
                fontSize: 22,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              _user.title,
              style: const TextStyle(
                color: Colors.white70,
                fontSize: 16,
              ),
            ),
            const SizedBox(height: 4),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.location_on, color: Colors.white70, size: 14),
                const SizedBox(width: 4),
                Text(
                  _user.location,
                  style: const TextStyle(
                    color: Colors.white70,
                    fontSize: 13,
                  ),
                ),
              ],
            ),
          ],
        ),
      ],
    );
  }

  Widget _buildProfileInfo() {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'เกี่ยวกับ',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 8),
          Text(
            _user.bio,
            style: const TextStyle(fontSize: 14, height: 1.6, color: Colors.black87),
          ),
        ],
      ),
    );
  }

  Widget _buildStats() {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
          _buildStatItem(_user.posts.toString(), 'โพสต์'),
          _buildDividerVertical(),
          _buildStatItem(_formatNumber(_user.followers), 'ผู้ติดตาม'),
          _buildDividerVertical(),
          _buildStatItem(_user.following.toString(), 'กำลังติดตาม'),
        ],
      ),
    );
  }

  Widget _buildStatItem(String value, String label) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 22,
            fontWeight: FontWeight.bold,
            color: Colors.deepPurple,
          ),
        ),
        Text(
          label,
          style: const TextStyle(fontSize: 13, color: Colors.grey),
        ),
      ],
    );
  }

  Widget _buildDividerVertical() {
    return Container(
      height: 40,
      width: 1,
      color: Colors.grey.shade300,
    );
  }

  Widget _buildActions() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          Expanded(
            child: ElevatedButton(
              onPressed: () => setState(() => _isFollowing = !_isFollowing),
              style: ElevatedButton.styleFrom(
                backgroundColor: _isFollowing ? Colors.grey.shade200 : Colors.deepPurple,
                foregroundColor: _isFollowing ? Colors.black : Colors.white,
                padding: const EdgeInsets.symmetric(vertical: 14),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(10),
                ),
              ),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(_isFollowing ? Icons.person_remove : Icons.person_add),
                  const SizedBox(width: 8),
                  Text(_isFollowing ? 'เลิกติดตาม' : 'ติดตาม'),
                ],
              ),
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: OutlinedButton(
              onPressed: () => _showMessageDialog(),
              style: OutlinedButton.styleFrom(
                padding: const EdgeInsets.symmetric(vertical: 14),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(10),
                ),
              ),
              child: const Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.message_outlined),
                  SizedBox(width: 8),
                  Text('ข้อความ'),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildSkills() {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'ทักษะ',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 12),
          Wrap(
            spacing: 8,
            runSpacing: 8,
            children: _user.skills.map((skill) {
              return Chip(
                label: Text(skill),
                backgroundColor: Colors.deepPurple.shade50,
                labelStyle: const TextStyle(
                  color: Colors.deepPurple,
                  fontWeight: FontWeight.w500,
                ),
                side: BorderSide(color: Colors.deepPurple.shade200),
              );
            }).toList(),
          ),
        ],
      ),
    );
  }

  Widget _buildSocialLinks() {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'ช่องทางติดต่อ',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 12),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceAround,
            children: _user.socialLinks.map((link) {
              return InkWell(
                onTap: () {},
                borderRadius: BorderRadius.circular(12),
                child: Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: link.color.withOpacity(0.1),
                    borderRadius: BorderRadius.circular(12),
                    border: Border.all(color: link.color.withOpacity(0.3)),
                  ),
                  child: Column(
                    children: [
                      Icon(link.icon, color: link.color, size: 28),
                      const SizedBox(height: 4),
                      Text(
                        link.label,
                        style: TextStyle(
                          fontSize: 11,
                          color: link.color,
                          fontWeight: FontWeight.w500,
                        ),
                      ),
                    ],
                  ),
                ),
              );
            }).toList(),
          ),
        ],
      ),
    );
  }

  Widget _buildPosts() {
    return Padding(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'โพสต์ล่าสุด',
            style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 12),
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
                borderRadius: BorderRadius.circular(4),
                child: Image.network(
                  'https://picsum.photos/150/150?random=${index + 20}',
                  fit: BoxFit.cover,
                ),
              );
            },
          ),
        ],
      ),
    );
  }

  String _formatNumber(int n) {
    if (n >= 1000) {
      return '${(n / 1000).toStringAsFixed(1)}K';
    }
    return n.toString();
  }

  void _showShareDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('แชร์โปรไฟล์'),
        content: const Text('ต้องการแชร์โปรไฟล์นี้หรือไม่?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('แชร์'),
          ),
        ],
      ),
    );
  }

  void _showOptionsMenu() {
    showModalBottomSheet(
      context: context,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
      ),
      builder: (context) => Column(
        mainAxisSize: MainAxisSize.min,
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
          const SizedBox(height: 8),
          ListTile(
            leading: const Icon(Icons.block, color: Colors.orange),
            title: const Text('บล็อกผู้ใช้'),
            onTap: () => Navigator.pop(context),
          ),
          ListTile(
            leading: const Icon(Icons.report_gmailerrorred, color: Colors.red),
            title: const Text('รายงาน'),
            onTap: () => Navigator.pop(context),
          ),
          const SizedBox(height: 8),
        ],
      ),
    );
  }

  void _showMessageDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('ส่งข้อความถึง ${_user.name}'),
        content: const TextField(
          maxLines: 3,
          decoration: InputDecoration(
            hintText: 'พิมพ์ข้อความ...',
            border: OutlineInputBorder(),
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () {
              Navigator.pop(context);
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('ส่งข้อความแล้ว!')),
              );
            },
            child: const Text('ส่ง'),
          ),
        ],
      ),
    );
  }
}
```

---

## สรุป (Summary)

ใน Part 04 นี้เราได้เรียนรู้ Widget พื้นฐานของ Flutter:

| Widget | ใช้สำหรับ |
|--------|-----------|
| `Text` | แสดงข้อความ, รองรับ rich text และ overflow |
| `Icon` | แสดงไอคอน Material/Cupertino |
| `Image` | แสดงรูปจาก network, asset, file, memory |
| `Container` | กล่องอเนกประสงค์ (padding, margin, decoration) |
| `Row/Column` | จัดเรียง widget แนวนอน/ตั้ง |
| `ElevatedButton` | ปุ่มที่มี elevation |
| `TextButton` | ปุ่มแบบข้อความ (ไม่มี background) |
| `OutlinedButton` | ปุ่มแบบมีกรอบ |
| `IconButton` | ปุ่มที่เป็น icon |
| `FloatingActionButton` | ปุ่มลอยอยู่มุม |
| `AppBar` | แถบด้านบน |
| `Scaffold` | โครงสร้างหลักของหน้า |
| `SafeArea` | ป้องกัน content ทับ system UI |

---

## แบบฝึกหัด (Exercises)

1. **Text Styling**: สร้าง widget ที่แสดงบทความสั้นๆ พร้อม title, subtitle, body text ด้วย TextStyle ที่แตกต่างกัน

2. **Icon Grid**: สร้าง grid 4x4 ของ icon ต่างๆ พร้อม label ด้านล่าง เมื่อกดให้แสดง SnackBar บอกชื่อ icon

3. **Image Gallery**: สร้าง horizontal scroll list ของรูปภาพจาก network พร้อม error handling

4. **Card UI**: สร้าง product card ที่มี: รูปสินค้า, ชื่อ, ราคา, rating (ด้วย Icon star), ปุ่ม "เพิ่มลงตะกร้า"

5. **Navigation App**: สร้างแอปที่มี Scaffold พร้อม Drawer (5 เมนู), BottomNavigationBar (4 tabs), FAB, และ SnackBar

---

[← Part 03: Dart OOP](part_03.md) | [Part 05: Layout Widgets →](part_05.md)

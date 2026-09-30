# Part 05: Layout Widgets
## ขั้นตอนที่ 41-50

---

## สารบัญ
1. [Row & Column (Advanced)](#ขั้นตอนที่-41-row--column-advanced)
2. [Stack & Positioned](#ขั้นตอนที่-42-stack--positioned)
3. [Wrap Widget](#ขั้นตอนที่-43-wrap-widget)
4. [GridView](#ขั้นตอนที่-44-gridview)
5. [ListView](#ขั้นตอนที่-45-listview)
6. [SizedBox & Padding](#ขั้นตอนที่-46-sizedbox--padding)
7. [Align & Center](#ขั้นตอนที่-47-align--center)
8. [ConstrainedBox & AspectRatio](#ขั้นตอนที่-48-constrainedbox--aspectratio)
9. [FractionallySizedBox & LayoutBuilder](#ขั้นตอนที่-49-fractionallysizedbox--layoutbuilder)
10. [Workshop: Dashboard UI](#ขั้นตอนที่-50-workshop-dashboard-ui)

---

## ขั้นตอนที่ 41: Row & Column (Advanced)

เรียนรู้การจัดการ Row/Column ในเชิงลึก: Flexible, Expanded, IntrinsicHeight, IntrinsicWidth

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LayoutApp());

class LayoutApp extends StatelessWidget {
  const LayoutApp({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Layout Demo',
        theme: ThemeData(useMaterial3: true),
        home: const RowColumnAdvancedScreen(),
      );
}

class RowColumnAdvancedScreen extends StatelessWidget {
  const RowColumnAdvancedScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Row & Column Advanced')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            _sectionTitle('1. Flexible vs Expanded'),

            // Flexible ยืดหยุ่น แต่ไม่บังคับเต็มพื้นที่
            const Text('Flexible (fit: FlexFit.loose - ไม่บังคับเต็ม):'),
            const SizedBox(height: 4),
            Row(
              children: [
                Flexible(
                  flex: 1,
                  child: _box(Colors.red, 'ข้อความสั้น'),
                ),
                Flexible(
                  flex: 2,
                  child: _box(Colors.green, 'ข้อความที่ยาวกว่านิดหน่อย'),
                ),
                Flexible(
                  flex: 1,
                  child: _box(Colors.blue, 'สั้น'),
                ),
              ],
            ),

            const SizedBox(height: 16),

            // Expanded บังคับเต็มพื้นที่ที่ได้รับ
            const Text('Expanded (บังคับเต็มพื้นที่):'),
            const SizedBox(height: 4),
            Row(
              children: [
                Expanded(
                  flex: 1,
                  child: _box(Colors.red, 'flex:1'),
                ),
                Expanded(
                  flex: 2,
                  child: _box(Colors.green, 'flex:2'),
                ),
                Expanded(
                  flex: 1,
                  child: _box(Colors.blue, 'flex:1'),
                ),
              ],
            ),

            const SizedBox(height: 24),

            _sectionTitle('2. IntrinsicHeight (ความสูงเท่ากัน)'),

            IntrinsicHeight(
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  Expanded(
                    child: Container(
                      color: Colors.blue.shade100,
                      padding: const EdgeInsets.all(16),
                      child: const Text(
                        'คอลัมน์ซ้าย\nมีหลายบรรทัด\nดังนั้นจะสูงกว่า',
                        style: TextStyle(height: 1.5),
                      ),
                    ),
                  ),
                  const SizedBox(width: 8),
                  Expanded(
                    child: Container(
                      color: Colors.green.shade100,
                      padding: const EdgeInsets.all(16),
                      child: const Text('คอลัมน์ขวา\nสั้นกว่า'),
                    ),
                  ),
                ],
              ),
            ),

            const SizedBox(height: 8),
            const Text(
              'IntrinsicHeight ทำให้ทั้งสองคอลัมน์มีความสูงเท่ากัน',
              style: TextStyle(color: Colors.grey, fontSize: 12),
            ),

            const SizedBox(height: 24),

            _sectionTitle('3. MainAxisSize'),

            // mainAxisSize: min - ขนาดตามเนื้อหา
            const Text('mainAxisSize: min'),
            Container(
              color: Colors.grey.shade100,
              child: Row(
                mainAxisSize: MainAxisSize.min,  // หดตามเนื้อหา
                children: [
                  _box(Colors.red, 'A'),
                  _box(Colors.green, 'B'),
                  _box(Colors.blue, 'C'),
                ],
              ),
            ),

            const SizedBox(height: 8),

            // mainAxisSize: max - เต็มความกว้าง (default)
            const Text('mainAxisSize: max (default)'),
            Container(
              color: Colors.grey.shade100,
              child: Row(
                mainAxisSize: MainAxisSize.max,  // เต็มความกว้าง
                children: [
                  _box(Colors.red, 'A'),
                  _box(Colors.green, 'B'),
                  _box(Colors.blue, 'C'),
                ],
              ),
            ),

            const SizedBox(height: 24),

            _sectionTitle('4. crossAxisAlignment: stretch'),

            Container(
              height: 100,
              color: Colors.grey.shade100,
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  _box(Colors.red, 'stretch\nA'),
                  _box(Colors.green, 'stretch\nB'),
                  _box(Colors.blue, 'stretch\nC'),
                ],
              ),
            ),

            const SizedBox(height: 24),

            _sectionTitle('5. Nested Row/Column'),

            // Row ซ้อน Column
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  children: [
                    ClipRRect(
                      borderRadius: BorderRadius.circular(8),
                      child: Image.network(
                        'https://picsum.photos/80/80?random=50',
                        width: 80,
                        height: 80,
                        fit: BoxFit.cover,
                      ),
                    ),
                    const SizedBox(width: 16),
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          const Text(
                            'ชื่อผลิตภัณฑ์',
                            style: TextStyle(
                              fontWeight: FontWeight.bold,
                              fontSize: 16,
                            ),
                          ),
                          const SizedBox(height: 4),
                          const Text(
                            'รายละเอียดสินค้าสั้นๆ',
                            style: TextStyle(color: Colors.grey),
                          ),
                          const SizedBox(height: 8),
                          Row(
                            children: [
                              const Icon(Icons.star,
                                  color: Colors.amber, size: 16),
                              const Text('4.8'),
                              const Spacer(),
                              const Text(
                                '฿299',
                                style: TextStyle(
                                  color: Colors.deepOrange,
                                  fontWeight: FontWeight.bold,
                                  fontSize: 16,
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
            ),

            const SizedBox(height: 24),

            _sectionTitle('6. Spacer widget'),

            Row(
              children: [
                _box(Colors.red, 'ซ้าย'),
                const Spacer(),  // ดันให้ห่างกัน
                _box(Colors.green, 'กลาง'),
                const Spacer(flex: 2),  // Spacer ใหญ่กว่า
                _box(Colors.blue, 'ขวา'),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _sectionTitle(String title) => Padding(
        padding: const EdgeInsets.symmetric(vertical: 8),
        child: Text(
          title,
          style: const TextStyle(
            fontWeight: FontWeight.bold,
            fontSize: 16,
            color: Colors.deepPurple,
          ),
        ),
      );

  Widget _box(Color color, String label) => Container(
        margin: const EdgeInsets.all(2),
        padding: const EdgeInsets.all(8),
        color: color.withOpacity(0.7),
        child: Text(
          label,
          style: const TextStyle(color: Colors.white, fontSize: 12),
        ),
      );
}
```

---

## ขั้นตอนที่ 42: Stack & Positioned

Stack ซ้อน widget ทับกัน, Positioned กำหนดตำแหน่งใน Stack

```dart
import 'package:flutter/material.dart';

class StackDemoScreen extends StatelessWidget {
  const StackDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Stack & Positioned')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            const Text('1. Stack พื้นฐาน:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Stack พื้นฐาน - children ซ้อนทับกัน
            SizedBox(
              width: 200,
              height: 200,
              child: Stack(
                children: [
                  Container(color: Colors.blue, width: 150, height: 150),
                  Container(
                    color: Colors.red.withOpacity(0.7),
                    width: 100,
                    height: 100,
                  ),
                  Container(
                    color: Colors.green.withOpacity(0.7),
                    width: 60,
                    height: 60,
                  ),
                ],
              ),
            ),

            const SizedBox(height: 24),

            const Text('2. Stack + Positioned:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            SizedBox(
              width: double.infinity,
              height: 200,
              child: Stack(
                children: [
                  // Background
                  Positioned.fill(
                    child: Container(color: Colors.grey.shade200),
                  ),

                  // Top-left
                  const Positioned(
                    top: 8,
                    left: 8,
                    child: Text('Top Left'),
                  ),

                  // Top-right
                  const Positioned(
                    top: 8,
                    right: 8,
                    child: Text('Top Right'),
                  ),

                  // Center
                  const Positioned.fill(
                    child: Center(
                      child: Text(
                        'Center',
                        style: TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),

                  // Bottom-left
                  const Positioned(
                    bottom: 8,
                    left: 8,
                    child: Text('Bottom Left'),
                  ),

                  // Bottom-right
                  const Positioned(
                    bottom: 8,
                    right: 8,
                    child: Text('Bottom Right'),
                  ),
                ],
              ),
            ),

            const SizedBox(height: 24),

            const Text('3. Stack Alignment:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Stack.alignment ใช้สำหรับ non-positioned children
            SizedBox(
              width: 200,
              height: 200,
              child: Stack(
                alignment: Alignment.center,  // center alignment
                children: [
                  Container(
                    width: 200,
                    height: 200,
                    decoration: BoxDecoration(
                      color: Colors.blue.shade100,
                      borderRadius: BorderRadius.circular(16),
                    ),
                  ),
                  Container(
                    width: 120,
                    height: 120,
                    decoration: BoxDecoration(
                      color: Colors.blue.shade300,
                      borderRadius: BorderRadius.circular(12),
                    ),
                  ),
                  Container(
                    width: 60,
                    height: 60,
                    decoration: BoxDecoration(
                      color: Colors.blue.shade600,
                      borderRadius: BorderRadius.circular(8),
                    ),
                  ),
                  const Icon(Icons.star, color: Colors.white, size: 30),
                ],
              ),
            ),

            const SizedBox(height: 24),

            const Text('4. Stack ใช้งานจริง (Card + Badge):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Product card with badge
            Row(
              children: [
                _buildProductCard(
                  'สินค้า A',
                  '฿199',
                  badgeText: 'NEW',
                  badgeColor: Colors.green,
                ),
                const SizedBox(width: 16),
                _buildProductCard(
                  'สินค้า B',
                  '฿299',
                  badgeText: '-20%',
                  badgeColor: Colors.red,
                ),
                const SizedBox(width: 16),
                _buildProductCard(
                  'สินค้า C',
                  '฿149',
                ),
              ],
            ),

            const SizedBox(height: 24),

            const Text('5. Profile Avatar + Status:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Row(
              children: [
                _buildAvatarWithStatus(
                    'https://picsum.photos/60/60?random=1', true),
                const SizedBox(width: 16),
                _buildAvatarWithStatus(
                    'https://picsum.photos/60/60?random=2', false),
                const SizedBox(width: 16),
                _buildAvatarWithBadge(
                    'https://picsum.photos/60/60?random=3', 5),
              ],
            ),

            const SizedBox(height: 24),

            const Text('6. Image + Text Overlay:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            ClipRRect(
              borderRadius: BorderRadius.circular(16),
              child: Stack(
                children: [
                  Image.network(
                    'https://picsum.photos/400/200?random=60',
                    width: double.infinity,
                    height: 200,
                    fit: BoxFit.cover,
                  ),
                  Positioned.fill(
                    child: Container(
                      decoration: BoxDecoration(
                        gradient: LinearGradient(
                          begin: Alignment.topCenter,
                          end: Alignment.bottomCenter,
                          colors: [
                            Colors.transparent,
                            Colors.black.withOpacity(0.8),
                          ],
                        ),
                      ),
                    ),
                  ),
                  const Positioned(
                    top: 16,
                    right: 16,
                    child: Icon(Icons.bookmark_border, color: Colors.white, size: 28),
                  ),
                  const Positioned(
                    bottom: 16,
                    left: 16,
                    right: 16,
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          'หัวข้อบทความ',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 18,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        SizedBox(height: 4),
                        Row(
                          children: [
                            Icon(Icons.access_time,
                                color: Colors.white70, size: 14),
                            SizedBox(width: 4),
                            Text('5 นาทีที่แล้ว',
                                style: TextStyle(
                                    color: Colors.white70, fontSize: 12)),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildProductCard(String name, String price,
      {String? badgeText, Color badgeColor = Colors.red}) {
    return Stack(
      children: [
        Container(
          width: 100,
          height: 140,
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(12),
            boxShadow: [
              BoxShadow(
                color: Colors.grey.shade200,
                blurRadius: 4,
                offset: const Offset(0, 2),
              ),
            ],
          ),
          child: Column(
            children: [
              ClipRRect(
                borderRadius: const BorderRadius.vertical(
                    top: Radius.circular(12)),
                child: Image.network(
                  'https://picsum.photos/100/80?random=${name.hashCode}',
                  width: 100,
                  height: 80,
                  fit: BoxFit.cover,
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  children: [
                    Text(name,
                        style: const TextStyle(fontWeight: FontWeight.bold)),
                    Text(price,
                        style: const TextStyle(color: Colors.deepOrange)),
                  ],
                ),
              ),
            ],
          ),
        ),
        if (badgeText != null)
          Positioned(
            top: 8,
            right: 8,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
              decoration: BoxDecoration(
                color: badgeColor,
                borderRadius: BorderRadius.circular(4),
              ),
              child: Text(
                badgeText,
                style: const TextStyle(
                  color: Colors.white,
                  fontSize: 10,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
          ),
      ],
    );
  }

  Widget _buildAvatarWithStatus(String url, bool isOnline) {
    return Stack(
      children: [
        CircleAvatar(
          radius: 30,
          backgroundImage: NetworkImage(url),
        ),
        Positioned(
          bottom: 0,
          right: 0,
          child: Container(
            width: 16,
            height: 16,
            decoration: BoxDecoration(
              color: isOnline ? Colors.green : Colors.grey,
              shape: BoxShape.circle,
              border: Border.all(color: Colors.white, width: 2),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildAvatarWithBadge(String url, int count) {
    return Stack(
      clipBehavior: Clip.none,
      children: [
        CircleAvatar(
          radius: 30,
          backgroundImage: NetworkImage(url),
        ),
        Positioned(
          top: -4,
          right: -4,
          child: Container(
            padding: const EdgeInsets.all(4),
            decoration: const BoxDecoration(
              color: Colors.red,
              shape: BoxShape.circle,
            ),
            child: Text(
              '$count',
              style: const TextStyle(
                color: Colors.white,
                fontSize: 10,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 43: Wrap Widget

Wrap จัดเรียง widget อัตโนมัติ เมื่อไม่พอในบรรทัดจะขึ้นบรรทัดใหม่

```dart
import 'package:flutter/material.dart';

class WrapDemoScreen extends StatefulWidget {
  const WrapDemoScreen({super.key});

  @override
  State<WrapDemoScreen> createState() => _WrapDemoScreenState();
}

class _WrapDemoScreenState extends State<WrapDemoScreen> {
  final List<String> _selectedTags = [];
  final List<String> _allTags = [
    'Flutter', 'Dart', 'Firebase', 'REST API', 'GraphQL',
    'Git', 'Docker', 'Kubernetes', 'AWS', 'iOS', 'Android',
    'UI/UX', 'Testing', 'CI/CD', 'Agile', 'Scrum',
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Wrap Widget')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            const Text('1. Wrap พื้นฐาน (Tags):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Wrap(
              spacing: 8,       // ระยะห่างแนวนอน
              runSpacing: 8,    // ระยะห่างแนวตั้ง (ระหว่าง row)
              children: [
                'Flutter', 'Dart', 'Mobile', 'iOS', 'Android',
                'Web', 'Desktop', 'Firebase', 'REST API', 'GraphQL',
              ].map((tag) => Chip(label: Text(tag))).toList(),
            ),

            const SizedBox(height: 24),

            const Text('2. Wrap alignment:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Wrap จัดตำแหน่ง center
            Container(
              color: Colors.grey.shade100,
              width: double.infinity,
              child: Wrap(
                spacing: 8,
                runSpacing: 8,
                alignment: WrapAlignment.center,  // จัดกลาง
                children: ['A', 'BB', 'CCC', 'DDDD', 'EE', 'F', 'GGG']
                    .map((text) => Container(
                          padding: const EdgeInsets.symmetric(
                              horizontal: 16, vertical: 8),
                          color: Colors.blue.withOpacity(0.2),
                          child: Text(text),
                        ))
                    .toList(),
              ),
            ),

            const SizedBox(height: 16),

            // Wrap จัดตำแหน่ง end
            Container(
              color: Colors.grey.shade100,
              width: double.infinity,
              child: Wrap(
                spacing: 8,
                runSpacing: 8,
                alignment: WrapAlignment.end,  // ชิดขวา
                children: ['A', 'BB', 'CCC', 'DDDD', 'EE', 'F', 'GGG']
                    .map((text) => Container(
                          padding: const EdgeInsets.symmetric(
                              horizontal: 16, vertical: 8),
                          color: Colors.green.withOpacity(0.2),
                          child: Text(text),
                        ))
                    .toList(),
              ),
            ),

            const SizedBox(height: 24),

            const Text('3. Wrap แนวตั้ง (direction: vertical):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            SizedBox(
              height: 150,
              child: Wrap(
                direction: Axis.vertical,  // แนวตั้ง
                spacing: 8,
                runSpacing: 8,
                children: List.generate(
                  12,
                  (i) => Container(
                    width: 40,
                    height: 40,
                    color: Colors.primaries[i % Colors.primaries.length]
                        .withOpacity(0.5),
                    child: Center(
                      child: Text('${i + 1}',
                          style: const TextStyle(color: Colors.white)),
                    ),
                  ),
                ),
              ),
            ),

            const SizedBox(height: 24),

            const Text('4. Interactive Tags (เลือก/ยกเลิก):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: _allTags.map((tag) {
                final isSelected = _selectedTags.contains(tag);
                return FilterChip(
                  label: Text(tag),
                  selected: isSelected,
                  onSelected: (selected) {
                    setState(() {
                      if (selected) {
                        _selectedTags.add(tag);
                      } else {
                        _selectedTags.remove(tag);
                      }
                    });
                  },
                  selectedColor: Colors.deepPurple.shade100,
                  checkmarkColor: Colors.deepPurple,
                );
              }).toList(),
            ),

            const SizedBox(height: 16),

            if (_selectedTags.isNotEmpty)
              Container(
                width: double.infinity,
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: Colors.deepPurple.shade50,
                  borderRadius: BorderRadius.circular(8),
                  border: Border.all(color: Colors.deepPurple.shade200),
                ),
                child: Text(
                  'เลือกแล้ว: ${_selectedTags.join(", ")}',
                  style: const TextStyle(color: Colors.deepPurple),
                ),
              ),

            const SizedBox(height: 24),

            const Text('5. Wrap + runAlignment:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // runAlignment จัดตำแหน่งระหว่าง rows
            Container(
              height: 150,
              color: Colors.grey.shade100,
              child: Wrap(
                spacing: 8,
                runSpacing: 8,
                runAlignment: WrapAlignment.spaceAround,
                children: List.generate(
                  6,
                  (i) => Container(
                    width: 80,
                    height: 40,
                    color: Colors.primaries[i].shade200,
                    child: Center(child: Text('Item $i')),
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

## ขั้นตอนที่ 44: GridView

GridView แสดงข้อมูลในรูปแบบตาราง

```dart
import 'package:flutter/material.dart';

class GridViewDemoScreen extends StatelessWidget {
  const GridViewDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 4,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('GridView'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'Count'),
              Tab(text: 'Extent'),
              Tab(text: 'Builder'),
              Tab(text: 'Custom'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            _GridCountDemo(),
            _GridExtentDemo(),
            _GridBuilderDemo(),
            _GridCustomDemo(),
          ],
        ),
      ),
    );
  }
}

class _GridCountDemo extends StatelessWidget {
  const _GridCountDemo();

  @override
  Widget build(BuildContext context) {
    return GridView.count(
      // กำหนดจำนวน column
      crossAxisCount: 3,
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      padding: const EdgeInsets.all(16),
      children: List.generate(
        18,
        (index) => Container(
          decoration: BoxDecoration(
            color: Colors.primaries[index % Colors.primaries.length].shade200,
            borderRadius: BorderRadius.circular(8),
          ),
          child: Center(
            child: Text(
              'Item\n${index + 1}',
              textAlign: TextAlign.center,
              style: const TextStyle(
                fontWeight: FontWeight.bold,
                color: Colors.white,
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class _GridExtentDemo extends StatelessWidget {
  const _GridExtentDemo();

  @override
  Widget build(BuildContext context) {
    return GridView.extent(
      // กำหนดความกว้างสูงสุดของแต่ละ item
      maxCrossAxisExtent: 150,
      crossAxisSpacing: 8,
      mainAxisSpacing: 8,
      childAspectRatio: 1.2,  // width/height ratio
      padding: const EdgeInsets.all(16),
      children: List.generate(
        20,
        (index) => Card(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(
                Icons.widgets,
                size: 40,
                color: Colors.primaries[index % Colors.primaries.length],
              ),
              const SizedBox(height: 8),
              Text('Widget ${index + 1}'),
            ],
          ),
        ),
      ),
    );
  }
}

class _GridBuilderDemo extends StatelessWidget {
  const _GridBuilderDemo();

  // ข้อมูล mock
  static final List<Map<String, dynamic>> _products = List.generate(
    30,
    (i) => {
      'name': 'สินค้า ${i + 1}',
      'price': (i + 1) * 50,
      'rating': (3.5 + (i % 5) * 0.3).clamp(1.0, 5.0),
      'image': 'https://picsum.photos/150/150?random=${i + 100}',
    },
  );

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      padding: const EdgeInsets.all(16),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 12,
        mainAxisSpacing: 12,
        childAspectRatio: 0.75,  // ความสูงมากกว่าความกว้าง
      ),
      itemCount: _products.length,
      itemBuilder: (context, index) {
        final product = _products[index];
        return _buildProductCard(product);
      },
    );
  }

  Widget _buildProductCard(Map<String, dynamic> product) {
    return Card(
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // รูป
          Expanded(
            flex: 3,
            child: Image.network(
              product['image'] as String,
              width: double.infinity,
              fit: BoxFit.cover,
              errorBuilder: (_, __, ___) => Container(
                color: Colors.grey.shade200,
                child: const Icon(Icons.image, size: 48, color: Colors.grey),
              ),
            ),
          ),
          // ข้อมูล
          Expanded(
            flex: 2,
            child: Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product['name'] as String,
                    style: const TextStyle(fontWeight: FontWeight.bold),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const Spacer(),
                  Row(
                    children: [
                      const Icon(Icons.star, color: Colors.amber, size: 14),
                      Text(
                        (product['rating'] as double).toStringAsFixed(1),
                        style: const TextStyle(fontSize: 12),
                      ),
                    ],
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '฿${product['price']}',
                    style: const TextStyle(
                      color: Colors.deepOrange,
                      fontWeight: FontWeight.bold,
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

class _GridCustomDemo extends StatelessWidget {
  const _GridCustomDemo();

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      padding: const EdgeInsets.all(8),
      gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
        maxCrossAxisExtent: 120,
        crossAxisSpacing: 4,
        mainAxisSpacing: 4,
        childAspectRatio: 1.0,
      ),
      itemCount: 30,
      itemBuilder: (context, index) {
        return ClipRRect(
          borderRadius: BorderRadius.circular(8),
          child: Stack(
            fit: StackFit.expand,
            children: [
              Image.network(
                'https://picsum.photos/120/120?random=${index + 200}',
                fit: BoxFit.cover,
              ),
              Positioned(
                bottom: 0,
                left: 0,
                right: 0,
                child: Container(
                  color: Colors.black54,
                  padding: const EdgeInsets.symmetric(vertical: 4),
                  child: Text(
                    'Photo ${index + 1}',
                    textAlign: TextAlign.center,
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 10,
                    ),
                  ),
                ),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 45: ListView

ListView แสดงข้อมูลในรูปแบบรายการแนวตั้ง

```dart
import 'package:flutter/material.dart';

class ListViewDemoScreen extends StatelessWidget {
  const ListViewDemoScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 5,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('ListView'),
          bottom: const TabBar(
            isScrollable: true,
            tabs: [
              Tab(text: 'Basic'),
              Tab(text: 'Builder'),
              Tab(text: 'Separated'),
              Tab(text: 'Horizontal'),
              Tab(text: 'Custom Scroll'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            _ListBasicDemo(),
            _ListBuilderDemo(),
            _ListSeparatedDemo(),
            _ListHorizontalDemo(),
            _CustomScrollDemo(),
          ],
        ),
      ),
    );
  }
}

// 1. ListView พื้นฐาน
class _ListBasicDemo extends StatelessWidget {
  const _ListBasicDemo();

  @override
  Widget build(BuildContext context) {
    return ListView(
      padding: const EdgeInsets.all(8),
      children: [
        // ListTile สำเร็จรูป
        ListTile(
          leading: const CircleAvatar(child: Icon(Icons.person)),
          title: const Text('สมชาย ใจดี'),
          subtitle: const Text('Flutter Developer'),
          trailing: const Icon(Icons.arrow_forward_ios, size: 16),
          onTap: () {},
        ),
        const Divider(),
        ListTile(
          leading: const Icon(Icons.notifications, color: Colors.orange),
          title: const Text('การแจ้งเตือน'),
          trailing: Switch(value: true, onChanged: (_) {}),
        ),
        const Divider(),
        ListTile(
          leading: const Icon(Icons.dark_mode, color: Colors.indigo),
          title: const Text('โหมดมืด'),
          trailing: Switch(value: false, onChanged: (_) {}),
        ),
        const Divider(),
        // CheckboxListTile
        CheckboxListTile(
          secondary: const Icon(Icons.check_box_outline_blank),
          title: const Text('ยอมรับเงื่อนไข'),
          value: true,
          onChanged: (_) {},
        ),
        const Divider(),
        // RadioListTile
        RadioListTile<int>(
          title: const Text('ตัวเลือกที่ 1'),
          value: 1,
          groupValue: 1,
          onChanged: (_) {},
        ),
        RadioListTile<int>(
          title: const Text('ตัวเลือกที่ 2'),
          value: 2,
          groupValue: 1,
          onChanged: (_) {},
        ),
        const Divider(),
        // ExpansionTile
        ExpansionTile(
          leading: const Icon(Icons.info_outline),
          title: const Text('รายละเอียดเพิ่มเติม'),
          children: const [
            Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'นี่คือรายละเอียดที่ซ่อนอยู่ เมื่อกดแล้วจะขยายออกมา',
              ),
            ),
          ],
        ),
      ],
    );
  }
}

// 2. ListView.builder - สำหรับรายการยาว
class _ListBuilderDemo extends StatelessWidget {
  const _ListBuilderDemo();

  static final List<Map<String, dynamic>> _messages = List.generate(
    50,
    (i) => {
      'sender': 'ผู้ใช้ ${i + 1}',
      'message': 'ข้อความที่ ${i + 1}: สวัสดี Flutter!',
      'time': '${i % 24}:${(i * 3 % 60).toString().padLeft(2, '0')}',
      'unread': i % 3 == 0,
      'avatar': 'https://picsum.photos/50/50?random=${i + 50}',
    },
  );

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: _messages.length,
      itemBuilder: (context, index) {
        final msg = _messages[index];
        return ListTile(
          leading: Stack(
            children: [
              CircleAvatar(
                backgroundImage: NetworkImage(msg['avatar'] as String),
              ),
              if (msg['unread'] as bool)
                Positioned(
                  top: 0,
                  right: 0,
                  child: Container(
                    width: 12,
                    height: 12,
                    decoration: const BoxDecoration(
                      color: Colors.green,
                      shape: BoxShape.circle,
                    ),
                  ),
                ),
            ],
          ),
          title: Text(
            msg['sender'] as String,
            style: TextStyle(
              fontWeight: (msg['unread'] as bool)
                  ? FontWeight.bold
                  : FontWeight.normal,
            ),
          ),
          subtitle: Text(
            msg['message'] as String,
            maxLines: 1,
            overflow: TextOverflow.ellipsis,
          ),
          trailing: Text(
            msg['time'] as String,
            style: TextStyle(
              fontSize: 12,
              color: (msg['unread'] as bool) ? Colors.blue : Colors.grey,
              fontWeight: (msg['unread'] as bool)
                  ? FontWeight.bold
                  : FontWeight.normal,
            ),
          ),
          onTap: () {},
        );
      },
    );
  }
}

// 3. ListView.separated - มี divider ระหว่าง item
class _ListSeparatedDemo extends StatelessWidget {
  const _ListSeparatedDemo();

  @override
  Widget build(BuildContext context) {
    final items = List.generate(
      20,
      (i) => {'title': 'หัวข้อที่ ${i + 1}', 'subtitle': 'รายละเอียด...'},
    );

    return ListView.separated(
      padding: const EdgeInsets.all(8),
      itemCount: items.length,
      separatorBuilder: (context, index) {
        // Custom separator
        if (index % 5 == 4) {
          return Container(
            height: 8,
            color: Colors.grey.shade200,
          );
        }
        return const Divider(height: 1);
      },
      itemBuilder: (context, index) {
        return ListTile(
          leading: CircleAvatar(
            backgroundColor: Colors.primaries[index % Colors.primaries.length]
                .shade200,
            child: Text(
              '${index + 1}',
              style: const TextStyle(color: Colors.white),
            ),
          ),
          title: Text(items[index]['title']!),
          subtitle: Text(items[index]['subtitle']!),
          trailing: IconButton(
            icon: const Icon(Icons.more_vert),
            onPressed: () {},
          ),
          onTap: () {},
        );
      },
    );
  }
}

// 4. Horizontal ListView
class _ListHorizontalDemo extends StatelessWidget {
  const _ListHorizontalDemo();

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Category chips
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('หมวดหมู่:',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),
          SizedBox(
            height: 50,
            child: ListView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 16),
              children: [
                'ทั้งหมด', 'อาหาร', 'เทคโนโลยี', 'กีฬา', 'บันเทิง', 'สุขภาพ',
              ].asMap().entries.map((entry) {
                final isFirst = entry.key == 0;
                return Padding(
                  padding: const EdgeInsets.only(right: 8),
                  child: FilterChip(
                    label: Text(entry.value),
                    selected: isFirst,
                    onSelected: (_) {},
                  ),
                );
              }).toList(),
            ),
          ),

          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('แนะนำ:',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),

          // Horizontal scroll cards
          SizedBox(
            height: 200,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 16),
              itemCount: 10,
              itemBuilder: (context, index) {
                return Container(
                  width: 160,
                  margin: const EdgeInsets.only(right: 12),
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(12),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.grey.shade300,
                        blurRadius: 4,
                        offset: const Offset(0, 2),
                      ),
                    ],
                  ),
                  child: ClipRRect(
                    borderRadius: BorderRadius.circular(12),
                    child: Stack(
                      children: [
                        Image.network(
                          'https://picsum.photos/160/200?random=${index + 150}',
                          fit: BoxFit.cover,
                          width: 160,
                          height: 200,
                        ),
                        Positioned(
                          bottom: 0,
                          left: 0,
                          right: 0,
                          child: Container(
                            padding: const EdgeInsets.all(8),
                            decoration: const BoxDecoration(
                              gradient: LinearGradient(
                                begin: Alignment.topCenter,
                                end: Alignment.bottomCenter,
                                colors: [Colors.transparent, Colors.black87],
                              ),
                            ),
                            child: Text(
                              'รายการที่ ${index + 1}',
                              style: const TextStyle(
                                color: Colors.white,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                          ),
                        ),
                      ],
                    ),
                  ),
                );
              },
            ),
          ),

          // Grid-like in scroll
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('ล่าสุด:',
                style: TextStyle(fontWeight: FontWeight.bold)),
          ),

          ListView.builder(
            shrinkWrap: true,
            physics: const NeverScrollableScrollPhysics(),
            itemCount: 8,
            itemBuilder: (context, index) {
              return Card(
                margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                child: ListTile(
                  leading: Image.network(
                    'https://picsum.photos/60/60?random=${index + 170}',
                    width: 60,
                    height: 60,
                    fit: BoxFit.cover,
                  ),
                  title: Text('บทความที่ ${index + 1}'),
                  subtitle: const Text('คลิกอ่านเพิ่มเติม...'),
                  trailing: const Icon(Icons.arrow_forward_ios, size: 14),
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}

// 5. CustomScrollView
class _CustomScrollDemo extends StatelessWidget {
  const _CustomScrollDemo();

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        // SliverAppBar
        SliverAppBar(
          expandedHeight: 150,
          floating: true,
          pinned: false,
          flexibleSpace: FlexibleSpaceBar(
            title: const Text('Custom Scroll'),
            background: Image.network(
              'https://picsum.photos/400/150?random=300',
              fit: BoxFit.cover,
            ),
          ),
        ),

        // SliverGrid
        SliverPadding(
          padding: const EdgeInsets.all(8),
          sliver: SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 4,
              mainAxisSpacing: 4,
              childAspectRatio: 1.0,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                color: Colors.primaries[index % Colors.primaries.length]
                    .shade200,
                child: Center(child: Text('${index + 1}')),
              ),
              childCount: 9,
            ),
          ),
        ),

        // SliverList
        SliverList(
          delegate: SliverChildBuilderDelegate(
            (context, index) => ListTile(
              leading: CircleAvatar(child: Text('${index + 1}')),
              title: Text('รายการที่ ${index + 1}'),
              subtitle: const Text('SliverList item'),
            ),
            childCount: 20,
          ),
        ),

        // SliverFillRemaining
        const SliverFillRemaining(
          hasScrollBody: false,
          child: Center(
            child: Padding(
              padding: EdgeInsets.all(32),
              child: Text(
                'สิ้นสุดรายการ',
                style: TextStyle(color: Colors.grey),
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 46: SizedBox & Padding

Widget สำหรับจัดระยะห่างและขนาด

```dart
import 'package:flutter/material.dart';

class SizedBoxPaddingDemo extends StatelessWidget {
  const SizedBoxPaddingDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('SizedBox & Padding')),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            // SizedBox ใช้งานต่างๆ
            const Padding(
              padding: EdgeInsets.all(16),
              child: Text('SizedBox:',
                  style: TextStyle(fontWeight: FontWeight.bold)),
            ),

            // SizedBox เป็น spacer
            const Row(
              children: [
                SizedBox(width: 16),
                Text('Item A'),
                SizedBox(width: 32),  // ระยะห่าง
                Text('Item B'),
                SizedBox(width: 16),
              ],
            ),

            const SizedBox(height: 16),  // vertical space

            // SizedBox กำหนดขนาด
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: SizedBox(
                width: 200,
                height: 80,
                child: ElevatedButton(
                  onPressed: () {},
                  child: const Text('ปุ่มขนาดคงที่ 200x80'),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // SizedBox.expand เต็มพื้นที่ parent
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: SizedBox(
                height: 100,
                child: SizedBox.expand(
                  child: Container(
                    color: Colors.blue.shade100,
                    child: const Center(child: Text('SizedBox.expand')),
                  ),
                ),
              ),
            ),

            const SizedBox(height: 24),

            // Padding แบบต่างๆ
            const Padding(
              padding: EdgeInsets.all(16),
              child: Text('Padding:',
                  style: TextStyle(fontWeight: FontWeight.bold)),
            ),

            // EdgeInsets ต่างๆ
            Container(
              color: Colors.grey.shade100,
              child: Padding(
                padding: const EdgeInsets.all(16),  // ทุกด้าน
                child: Container(color: Colors.blue, height: 40),
              ),
            ),

            const SizedBox(height: 8),

            Container(
              color: Colors.grey.shade100,
              child: Padding(
                padding: const EdgeInsets.symmetric(
                    horizontal: 32, vertical: 8),  // แกน
                child: Container(color: Colors.green, height: 40),
              ),
            ),

            const SizedBox(height: 8),

            Container(
              color: Colors.grey.shade100,
              child: Padding(
                padding: const EdgeInsets.only(
                    left: 16, top: 8, right: 32, bottom: 24),  // เฉพาะด้าน
                child: Container(color: Colors.orange, height: 40),
              ),
            ),

            const SizedBox(height: 8),

            Container(
              color: Colors.grey.shade100,
              child: Padding(
                padding: const EdgeInsets.fromLTRB(8, 16, 32, 8),  // LTRB
                child: Container(color: Colors.red, height: 40),
              ),
            ),

            const SizedBox(height: 24),

            // EdgeInsets.lerp
            const Padding(
              padding: EdgeInsets.all(16),
              child: Text('Padding ใน Card:',
                  style: TextStyle(fontWeight: FontWeight.bold)),
            ),

            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Card(
                child: Column(
                  children: [
                    Padding(
                      padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
                      child: Row(
                        children: [
                          const Icon(Icons.info, color: Colors.blue),
                          const SizedBox(width: 8),
                          const Text('Title',
                              style: TextStyle(
                                  fontWeight: FontWeight.bold, fontSize: 16)),
                          const Spacer(),
                          IconButton(
                            icon: const Icon(Icons.close),
                            onPressed: () {},
                            padding: EdgeInsets.zero,
                            constraints: const BoxConstraints(),
                          ),
                        ],
                      ),
                    ),
                    const Padding(
                      padding: EdgeInsets.fromLTRB(16, 0, 16, 16),
                      child: Text('เนื้อหาของการ์ด ใช้ Padding จัดระยะห่าง'),
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

## ขั้นตอนที่ 47: Align & Center

Widget สำหรับจัดตำแหน่ง child

```dart
import 'package:flutter/material.dart';

class AlignCenterDemo extends StatelessWidget {
  const AlignCenterDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Align & Center')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('Align positions:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // แสดง Align ต่างๆ
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                _AlignBox('topLeft', Alignment.topLeft),
                _AlignBox('topCenter', Alignment.topCenter),
                _AlignBox('topRight', Alignment.topRight),
                _AlignBox('centerLeft', Alignment.centerLeft),
                _AlignBox('center', Alignment.center),
                _AlignBox('centerRight', Alignment.centerRight),
                _AlignBox('bottomLeft', Alignment.bottomLeft),
                _AlignBox('bottomCenter', Alignment.bottomCenter),
                _AlignBox('bottomRight', Alignment.bottomRight),
              ],
            ),

            const SizedBox(height: 24),
            const Text('Center (shorthand):', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Container(
              width: double.infinity,
              height: 100,
              color: Colors.blue.shade50,
              child: const Center(
                child: Text('Center Widget', style: TextStyle(fontSize: 18)),
              ),
            ),

            const SizedBox(height: 16),

            // Align + widthFactor, heightFactor
            const Text('Align with factors:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Container(
              color: Colors.grey.shade100,
              child: Align(
                alignment: Alignment.centerLeft,
                widthFactor: 2.0,   // กว้างเป็น 2x ของ child
                heightFactor: 3.0,  // สูงเป็น 3x ของ child
                child: Container(
                  width: 60,
                  height: 40,
                  color: Colors.blue,
                  child: const Center(child: Text('Child')),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // FractionalOffset
            const Text('FractionalOffset:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Container(
              width: double.infinity,
              height: 100,
              color: Colors.amber.shade50,
              child: const Align(
                // FractionalOffset(0, 0) = topLeft
                // FractionalOffset(1, 1) = bottomRight
                // FractionalOffset(0.5, 0.5) = center
                alignment: Alignment(0.5, -0.5),  // custom position
                child: Icon(Icons.star, color: Colors.amber, size: 32),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _AlignBox extends StatelessWidget {
  final String label;
  final AlignmentGeometry alignment;

  const _AlignBox(this.label, this.alignment);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(
          width: 90,
          height: 70,
          color: Colors.grey.shade200,
          child: Align(
            alignment: alignment,
            child: Container(
              width: 20,
              height: 20,
              color: Colors.deepPurple,
            ),
          ),
        ),
        Text(label, style: const TextStyle(fontSize: 9)),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 48: ConstrainedBox & AspectRatio

Widget สำหรับจำกัดและกำหนดสัดส่วน

```dart
import 'package:flutter/material.dart';

class ConstrainedBoxDemo extends StatelessWidget {
  const ConstrainedBoxDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ConstrainedBox & AspectRatio')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            const Text('ConstrainedBox:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // ConstrainedBox จำกัด min/max ขนาด
            ConstrainedBox(
              constraints: const BoxConstraints(
                minWidth: 100,
                maxWidth: 300,
                minHeight: 50,
                maxHeight: 150,
              ),
              child: Container(
                color: Colors.blue.shade100,
                child: const Text(
                    'ข้อความนี้อยู่ใน ConstrainedBox\nmax 300x150 / min 100x50'),
              ),
            ),

            const SizedBox(height: 16),

            // UnconstrainedBox ปลดล็อคข้อจำกัด
            UnconstrainedBox(
              child: Container(
                width: 500,  // เกินหน้าจอ แต่ UnconstrainedBox อนุญาต
                height: 60,
                color: Colors.orange.shade100,
                child: const Center(child: Text('UnconstrainedBox - width 500')),
              ),
            ),

            const SizedBox(height: 24),

            const Text('AspectRatio:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // AspectRatio 16:9
            AspectRatio(
              aspectRatio: 16 / 9,
              child: Container(
                color: Colors.black,
                child: const Center(
                  child: Text(
                    '16:9 (วิดีโอ)',
                    style: TextStyle(color: Colors.white, fontSize: 18),
                  ),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // AspectRatio 1:1
            Row(
              children: [
                AspectRatio(
                  aspectRatio: 1.0,
                  child: Container(
                    color: Colors.blue.shade200,
                    child: const Center(child: Text('1:1\n(Square)')),
                  ),
                ),
                const SizedBox(width: 8),
                AspectRatio(
                  aspectRatio: 4 / 3,
                  child: Container(
                    color: Colors.green.shade200,
                    child: const Center(child: Text('4:3\n(Classic)')),
                  ),
                ),
              ],
            ),

            const SizedBox(height: 24),

            const Text('LimitedBox (overflow limit):',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // LimitedBox จำกัด unconstrained
            ListView(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              children: List.generate(
                5,
                (i) => LimitedBox(
                  maxHeight: 60,
                  child: Container(
                    color:
                        Colors.primaries[i % Colors.primaries.length].shade100,
                    child: Padding(
                      padding: const EdgeInsets.all(8),
                      child: Text('LimitedBox item ${i + 1}'),
                    ),
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

## ขั้นตอนที่ 49: FractionallySizedBox & LayoutBuilder

```dart
import 'package:flutter/material.dart';

class FractionallySizedBoxDemo extends StatelessWidget {
  const FractionallySizedBoxDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FractionallySizedBox & LayoutBuilder')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [

            const Text('FractionallySizedBox:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // FractionallySizedBox - ขนาดเป็น % ของ parent
            Container(
              height: 100,
              color: Colors.grey.shade200,
              child: FractionallySizedBox(
                widthFactor: 0.6,  // 60% ของ parent
                heightFactor: 0.8,  // 80% ของ parent
                alignment: Alignment.centerLeft,
                child: Container(
                  color: Colors.blue,
                  child: const Center(
                    child: Text(
                      '60% x 80%',
                      style: TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Progress bar ด้วย FractionallySizedBox
            const Text('Progress Bar:', style: TextStyle(color: Colors.grey)),
            const SizedBox(height: 8),

            _buildProgressBar('HTML/CSS', 0.90),
            const SizedBox(height: 8),
            _buildProgressBar('Flutter', 0.85),
            const SizedBox(height: 8),
            _buildProgressBar('Dart', 0.75),
            const SizedBox(height: 8),
            _buildProgressBar('Firebase', 0.60),

            const SizedBox(height: 24),

            const Text('LayoutBuilder:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text(
              'LayoutBuilder รู้ขนาดของ parent และสามารถ build ต่างกันได้',
              style: TextStyle(color: Colors.grey, fontSize: 12),
            ),
            const SizedBox(height: 8),

            // LayoutBuilder - responsive widget
            LayoutBuilder(
              builder: (context, constraints) {
                final width = constraints.maxWidth;

                // เปลี่ยน layout ตามความกว้าง
                if (width > 600) {
                  return _buildWideLayout();
                } else if (width > 400) {
                  return _buildMediumLayout();
                } else {
                  return _buildNarrowLayout();
                }
              },
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildProgressBar(String label, double progress) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(label),
            Text('${(progress * 100).toInt()}%'),
          ],
        ),
        const SizedBox(height: 4),
        Container(
          height: 12,
          width: double.infinity,
          decoration: BoxDecoration(
            color: Colors.grey.shade200,
            borderRadius: BorderRadius.circular(6),
          ),
          child: FractionallySizedBox(
            widthFactor: progress,
            alignment: Alignment.centerLeft,
            child: Container(
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Colors.blue, Colors.purple],
                ),
                borderRadius: BorderRadius.circular(6),
              ),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildWideLayout() {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.green.shade50,
      child: Row(
        children: [
          const Icon(Icons.desktop_windows, size: 32),
          const SizedBox(width: 16),
          const Expanded(
            child: Text('Wide Layout (>600px)\nใช้ Row layout'),
          ),
          ElevatedButton(onPressed: () {}, child: const Text('Action')),
        ],
      ),
    );
  }

  Widget _buildMediumLayout() {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.orange.shade50,
      child: Row(
        children: [
          const Icon(Icons.tablet, size: 28),
          const SizedBox(width: 8),
          const Expanded(
            child: Text('Medium Layout (400-600px)'),
          ),
          TextButton(onPressed: () {}, child: const Text('Action')),
        ],
      ),
    );
  }

  Widget _buildNarrowLayout() {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.blue.shade50,
      child: Column(
        children: [
          const Row(
            children: [
              Icon(Icons.smartphone, size: 24),
              SizedBox(width: 8),
              Text('Narrow Layout (<400px)'),
            ],
          ),
          const SizedBox(height: 8),
          SizedBox(
            width: double.infinity,
            child: ElevatedButton(
              onPressed: () {},
              child: const Text('Action (Full Width)'),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 50: Workshop - Dashboard UI

สร้าง Dashboard UI ที่ใช้ Layout Widgets ต่างๆ

```dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

void main() {
  runApp(const DashboardApp());
}

class DashboardApp extends StatelessWidget {
  const DashboardApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dashboard',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
        useMaterial3: true,
      ),
      home: const DashboardScreen(),
    );
  }
}

// Data Models
class StatCard {
  final String title;
  final String value;
  final String change;
  final bool isPositive;
  final IconData icon;
  final Color color;

  const StatCard({
    required this.title,
    required this.value,
    required this.change,
    required this.isPositive,
    required this.icon,
    required this.color,
  });
}

class Transaction {
  final String name;
  final String date;
  final double amount;
  final bool isIncome;
  final IconData icon;

  const Transaction({
    required this.name,
    required this.date,
    required this.amount,
    required this.isIncome,
    required this.icon,
  });
}

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  int _selectedIndex = 0;

  final List<StatCard> _stats = const [
    StatCard(
      title: 'รายได้รวม',
      value: '฿125,430',
      change: '+12.5%',
      isPositive: true,
      icon: Icons.trending_up,
      color: Colors.green,
    ),
    StatCard(
      title: 'ค่าใช้จ่าย',
      value: '฿43,210',
      change: '-3.2%',
      isPositive: true,
      icon: Icons.trending_down,
      color: Colors.orange,
    ),
    StatCard(
      title: 'คำสั่งซื้อ',
      value: '1,284',
      change: '+8.7%',
      isPositive: true,
      icon: Icons.shopping_cart,
      color: Colors.blue,
    ),
    StatCard(
      title: 'ลูกค้าใหม่',
      value: '342',
      change: '-1.5%',
      isPositive: false,
      icon: Icons.people,
      color: Colors.purple,
    ),
  ];

  final List<Transaction> _transactions = const [
    Transaction(
      name: 'คำสั่งซื้อ #1234',
      date: '30 ก.ย. 2026',
      amount: 2500,
      isIncome: true,
      icon: Icons.shopping_bag,
    ),
    Transaction(
      name: 'ค่าโฆษณา',
      date: '29 ก.ย. 2026',
      amount: 1200,
      isIncome: false,
      icon: Icons.campaign,
    ),
    Transaction(
      name: 'คำสั่งซื้อ #1233',
      date: '28 ก.ย. 2026',
      amount: 890,
      isIncome: true,
      icon: Icons.shopping_bag,
    ),
    Transaction(
      name: 'ค่า Hosting',
      date: '27 ก.ย. 2026',
      amount: 499,
      isIncome: false,
      icon: Icons.cloud,
    ),
    Transaction(
      name: 'คำสั่งซื้อ #1232',
      date: '26 ก.ย. 2026',
      amount: 3200,
      isIncome: true,
      icon: Icons.shopping_bag,
    ),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF5F7FA),
      appBar: _buildAppBar(),
      drawer: _buildDrawer(),
      body: _buildBody(),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (i) => setState(() => _selectedIndex = i),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.dashboard_outlined),
            selectedIcon: Icon(Icons.dashboard),
            label: 'Dashboard',
          ),
          NavigationDestination(
            icon: Icon(Icons.analytics_outlined),
            selectedIcon: Icon(Icons.analytics),
            label: 'Analytics',
          ),
          NavigationDestination(
            icon: Icon(Icons.receipt_long_outlined),
            selectedIcon: Icon(Icons.receipt_long),
            label: 'Orders',
          ),
          NavigationDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: 'Settings',
          ),
        ],
      ),
    );
  }

  PreferredSizeWidget _buildAppBar() {
    return AppBar(
      title: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text('Dashboard', style: TextStyle(fontWeight: FontWeight.bold)),
          Text(
            'ยินดีต้อนรับกลับ, สมชาย',
            style: TextStyle(fontSize: 12, color: Colors.grey.shade600),
          ),
        ],
      ),
      backgroundColor: Colors.white,
      elevation: 0,
      actions: [
        Stack(
          children: [
            IconButton(
              icon: const Icon(Icons.notifications_outlined),
              onPressed: () {},
            ),
            Positioned(
              top: 8,
              right: 8,
              child: Container(
                width: 8,
                height: 8,
                decoration: const BoxDecoration(
                  color: Colors.red,
                  shape: BoxShape.circle,
                ),
              ),
            ),
          ],
        ),
        Padding(
          padding: const EdgeInsets.only(right: 12),
          child: GestureDetector(
            onTap: () {},
            child: const CircleAvatar(
              radius: 18,
              backgroundImage:
                  NetworkImage('https://picsum.photos/36/36?random=1'),
            ),
          ),
        ),
      ],
    );
  }

  Widget _buildDrawer() {
    return Drawer(
      child: Column(
        children: [
          UserAccountsDrawerHeader(
            accountName: const Text('สมชาย พัฒนาดี'),
            accountEmail: const Text('somchai@example.com'),
            currentAccountPicture: const CircleAvatar(
              backgroundImage: NetworkImage(
                  'https://picsum.photos/60/60?random=1'),
            ),
            decoration: BoxDecoration(
              gradient: LinearGradient(
                colors: [Colors.indigo.shade700, Colors.indigo.shade400],
              ),
            ),
          ),
          const ListTile(
              leading: Icon(Icons.dashboard), title: Text('Dashboard')),
          const ListTile(
              leading: Icon(Icons.analytics), title: Text('Analytics')),
          const ListTile(
              leading: Icon(Icons.people), title: Text('ลูกค้า')),
          const ListTile(
              leading: Icon(Icons.inventory), title: Text('สินค้า')),
          const ListTile(
              leading: Icon(Icons.receipt_long), title: Text('คำสั่งซื้อ')),
          const Divider(),
          const ListTile(
              leading: Icon(Icons.settings), title: Text('ตั้งค่า')),
          const ListTile(
              leading: Icon(Icons.help_outline), title: Text('ช่วยเหลือ')),
          const Spacer(),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.logout, color: Colors.red),
            title: const Text('ออกจากระบบ',
                style: TextStyle(color: Colors.red)),
            onTap: () {},
          ),
        ],
      ),
    );
  }

  Widget _buildBody() {
    return CustomScrollView(
      slivers: [
        SliverPadding(
          padding: const EdgeInsets.all(16),
          sliver: SliverList(
            delegate: SliverChildListDelegate([
              // Stats Grid
              _buildStatsGrid(),

              const SizedBox(height: 20),

              // Revenue Chart (mock)
              _buildRevenueChart(),

              const SizedBox(height: 20),

              // Quick Actions
              _buildQuickActions(),

              const SizedBox(height: 20),

              // Recent Transactions
              _buildTransactions(),

              const SizedBox(height: 20),

              // Top Products
              _buildTopProducts(),

              const SizedBox(height: 20),
            ]),
          ),
        ),
      ],
    );
  }

  Widget _buildStatsGrid() {
    return GridView.count(
      crossAxisCount: 2,
      crossAxisSpacing: 12,
      mainAxisSpacing: 12,
      childAspectRatio: 1.5,
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      children: _stats.map(_buildStatCard).toList(),
    );
  }

  Widget _buildStatCard(StatCard stat) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              Container(
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: stat.color.withOpacity(0.1),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: Icon(stat.icon, color: stat.color, size: 20),
              ),
              const Spacer(),
              Container(
                padding:
                    const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
                decoration: BoxDecoration(
                  color: stat.isPositive
                      ? Colors.green.shade50
                      : Colors.red.shade50,
                  borderRadius: BorderRadius.circular(4),
                ),
                child: Text(
                  stat.change,
                  style: TextStyle(
                    fontSize: 11,
                    color: stat.isPositive ? Colors.green : Colors.red,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
            ],
          ),
          const Spacer(),
          Text(
            stat.value,
            style: const TextStyle(
              fontSize: 18,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            stat.title,
            style: TextStyle(
              fontSize: 12,
              color: Colors.grey.shade600,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildRevenueChart() {
    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text(
                'รายได้รายเดือน',
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
              DropdownButton<String>(
                value: '2026',
                isDense: true,
                items: ['2024', '2025', '2026']
                    .map((y) => DropdownMenuItem(value: y, child: Text(y)))
                    .toList(),
                onChanged: (_) {},
              ),
            ],
          ),
          const SizedBox(height: 20),
          // Mock chart bars
          SizedBox(
            height: 120,
            child: Row(
              crossAxisAlignment: CrossAxisAlignment.end,
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                _buildBar('ม.ค.', 0.6, Colors.indigo),
                _buildBar('ก.พ.', 0.8, Colors.indigo),
                _buildBar('มี.ค.', 0.5, Colors.indigo),
                _buildBar('เม.ย.', 0.9, Colors.indigo),
                _buildBar('พ.ค.', 0.7, Colors.indigo),
                _buildBar('มิ.ย.', 0.85, Colors.indigo),
                _buildBar('ก.ค.', 0.95, Colors.indigo),
                _buildBar('ส.ค.', 0.65, Colors.indigo),
                _buildBar('ก.ย.', 1.0, Colors.deepOrange),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildBar(String label, double height, Color color) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.end,
      children: [
        Container(
          width: 28,
          height: 100 * height,
          decoration: BoxDecoration(
            color: color.withOpacity(height == 1.0 ? 1.0 : 0.5),
            borderRadius: BorderRadius.circular(4),
          ),
        ),
        const SizedBox(height: 4),
        Text(label, style: const TextStyle(fontSize: 10, color: Colors.grey)),
      ],
    );
  }

  Widget _buildQuickActions() {
    final actions = [
      {'icon': Icons.add_shopping_cart, 'label': 'สั่งซื้อ', 'color': Colors.blue},
      {'icon': Icons.inventory, 'label': 'สต็อก', 'color': Colors.green},
      {'icon': Icons.people, 'label': 'ลูกค้า', 'color': Colors.orange},
      {'icon': Icons.bar_chart, 'label': 'รายงาน', 'color': Colors.purple},
    ];

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        const Text(
          'เมนูด่วน',
          style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 12),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: actions.map((action) {
            final color = action['color'] as Color;
            return Column(
              children: [
                Container(
                  width: 60,
                  height: 60,
                  decoration: BoxDecoration(
                    color: color.withOpacity(0.1),
                    borderRadius: BorderRadius.circular(16),
                    border: Border.all(
                        color: color.withOpacity(0.3), width: 1),
                  ),
                  child: Icon(action['icon'] as IconData,
                      color: color, size: 28),
                ),
                const SizedBox(height: 8),
                Text(
                  action['label'] as String,
                  style: const TextStyle(fontSize: 12),
                ),
              ],
            );
          }).toList(),
        ),
      ],
    );
  }

  Widget _buildTransactions() {
    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text(
                'รายการล่าสุด',
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.bold,
                ),
              ),
              TextButton(
                onPressed: () {},
                child: const Text('ดูทั้งหมด'),
              ),
            ],
          ),
          const SizedBox(height: 8),
          ..._transactions.map((t) => _buildTransactionItem(t)),
        ],
      ),
    );
  }

  Widget _buildTransactionItem(Transaction t) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8),
      child: Row(
        children: [
          Container(
            width: 44,
            height: 44,
            decoration: BoxDecoration(
              color: t.isIncome
                  ? Colors.green.shade50
                  : Colors.red.shade50,
              borderRadius: BorderRadius.circular(12),
            ),
            child: Icon(
              t.icon,
              color: t.isIncome ? Colors.green : Colors.red,
              size: 22,
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(t.name,
                    style: const TextStyle(fontWeight: FontWeight.w500)),
                Text(t.date,
                    style: const TextStyle(
                        fontSize: 12, color: Colors.grey)),
              ],
            ),
          ),
          Text(
            '${t.isIncome ? '+' : '-'}฿${t.amount.toStringAsFixed(0)}',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              color: t.isIncome ? Colors.green : Colors.red,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildTopProducts() {
    final products = [
      {'name': 'สินค้า A', 'sold': 234, 'revenue': 58500, 'progress': 0.9},
      {'name': 'สินค้า B', 'sold': 189, 'revenue': 47250, 'progress': 0.73},
      {'name': 'สินค้า C', 'sold': 145, 'revenue': 36250, 'progress': 0.56},
      {'name': 'สินค้า D', 'sold': 98, 'revenue': 24500, 'progress': 0.38},
    ];

    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(16),
        boxShadow: [
          BoxShadow(
            color: Colors.grey.withOpacity(0.1),
            blurRadius: 8,
            offset: const Offset(0, 2),
          ),
        ],
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          const Text(
            'สินค้าขายดี',
            style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 16),
          ...products.map((p) {
            final progress = p['progress'] as double;
            return Padding(
              padding: const EdgeInsets.only(bottom: 16),
              child: Column(
                children: [
                  Row(
                    children: [
                      Expanded(
                        child: Text(p['name'] as String,
                            style: const TextStyle(
                                fontWeight: FontWeight.w500)),
                      ),
                      Text(
                        '${p['sold']} ชิ้น',
                        style: const TextStyle(
                            fontSize: 12, color: Colors.grey),
                      ),
                      const SizedBox(width: 16),
                      Text(
                        '฿${p['revenue']}',
                        style: const TextStyle(fontWeight: FontWeight.bold),
                      ),
                    ],
                  ),
                  const SizedBox(height: 6),
                  ClipRRect(
                    borderRadius: BorderRadius.circular(4),
                    child: LinearProgressIndicator(
                      value: progress,
                      minHeight: 6,
                      backgroundColor: Colors.grey.shade200,
                      valueColor:
                          const AlwaysStoppedAnimation<Color>(Colors.indigo),
                    ),
                  ),
                ],
              ),
            );
          }),
        ],
      ),
    );
  }
}
```

---

## สรุป (Summary)

ใน Part 05 เราได้เรียน Layout Widgets:

| Widget | ใช้งาน |
|--------|--------|
| `Row/Column` | จัดเรียงแนวนอน/แนวตั้ง พร้อม Flexible/Expanded |
| `Stack/Positioned` | ซ้อน widget ทับกัน |
| `Wrap` | จัดเรียงอัตโนมัติ ขึ้นบรรทัดใหม่เมื่อไม่พอ |
| `GridView` | แสดงข้อมูลในตาราง |
| `ListView` | แสดงรายการแนวตั้ง/แนวนอน |
| `SizedBox` | กำหนดขนาดหรือสร้างช่องว่าง |
| `Padding` | เพิ่มระยะห่างภายใน |
| `Align/Center` | จัดตำแหน่ง child |
| `ConstrainedBox` | จำกัดขนาด min/max |
| `AspectRatio` | กำหนดสัดส่วน |
| `FractionallySizedBox` | ขนาดเป็น % ของ parent |
| `LayoutBuilder` | สร้าง responsive layout |

---

## แบบฝึกหัด (Exercises)

1. **Responsive Grid**: สร้าง GridView ที่เปลี่ยนจาก 2 column เป็น 3 column ตามความกว้างหน้าจอ โดยใช้ LayoutBuilder

2. **Chat UI**: สร้าง chat interface ด้วย ListView.builder พร้อม bubble message ของ sender และ receiver

3. **News Feed**: สร้าง news feed ด้วย CustomScrollView + SliverAppBar + SliverList ที่มีรูปภาพ header

4. **Stack Overlay**: สร้าง image card ด้วย Stack ที่มี: รูปภาพ, gradient overlay, title ล่าง, badge "HOT" บนขวา

5. **Progress Dashboard**: สร้าง skill progress page ด้วย Column + FractionallySizedBox สำหรับ progress bars

---

[← Part 04: Flutter Widget พื้นฐาน](part_04.md) | [Part 06: Stateful & Stateless Widgets →](part_06.md)

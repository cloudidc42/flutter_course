# Part 34: Performance Optimization
## ขั้นตอนที่ 331-340

---

## สารบัญ
1. [Flutter DevTools](#flutter-devtools)
2. [Performance Profiling](#performance-profiling)
3. [Widget Rebuilds Optimization](#widget-rebuilds-optimization)
4. [const Widget](#const-widget)
5. [RepaintBoundary](#repaintboundary)
6. [Expensive build() Anti-patterns](#expensive-build-anti-patterns)
7. [Image Optimization](#image-optimization)
8. [List Performance](#list-performance)
9. [compute() สำหรับ Heavy Computation](#compute)
10. [Memory Leaks Detection](#memory-leaks-detection)

---

## ขั้นตอนที่ 331: Flutter DevTools

Flutter DevTools คือชุดเครื่องมือสำหรับ Debug และ Profile Flutter Apps

### การเปิด DevTools

```bash
# วิธีที่ 1: ผ่าน Command Line
flutter run
# กด 'v' ใน terminal เพื่อเปิด DevTools

# วิธีที่ 2: ผ่าน VS Code
# Run > Open DevTools

# วิธีที่ 3: ผ่าน URL
# เปิด http://127.0.0.1:9101 หลังจาก run app

# วิธีที่ 4: ใช้ dart devtools command
dart devtools
```

### DevTools Panels

```
DevTools Panels:
┌────────────────────────────────────────────────────────────┐
│ Flutter DevTools                                           │
├────────────┬───────────┬────────────┬──────────┬──────────┤
│  Flutter   │ Timeline  │  Memory    │  CPU     │ Network  │
│ Inspector  │ (Perf)    │ Profiler   │ Profiler │          │
│            │           │            │          │          │
│ Widget Tree│ Frame Time│ Heap Snap  │ Call     │ HTTP     │
│ Properties │ Flame     │ Allocation │ Tree     │ Requests │
│ Layout     │ Chart     │ Retainer   │ Bottom   │          │
│ Explorer   │           │ Tree       │ Up       │          │
└────────────┴───────────┴────────────┴──────────┴──────────┘
```

### Performance Overlay

```dart
// เปิด Performance Overlay ใน code
import 'package:flutter/material.dart';

void main() {
  runApp(
    MaterialApp(
      showPerformanceOverlay: true, // เปิด overlay
      checkerboardRasterCacheImages: true, // แสดง raster cache
      checkerboardOffscreenLayers: true,   // แสดง offscreen layers
      home: const MyApp(),
    ),
  );
}
```

```
Performance Overlay แสดง 2 bars:
┌─────────────────────────────────────────────────┐
│ GPU Thread  ████████░░░░░░░░░░░░  16ms          │
│ UI Thread   ██████░░░░░░░░░░░░░░  16ms          │
└─────────────────────────────────────────────────┘
เส้นสีแดง = 16ms threshold (60fps)
เส้นสีเหลือง = 8ms threshold (120fps)
ถ้า bar เกิน threshold = dropped frames
```

---

## ขั้นตอนที่ 332: Performance Profiling

### Profile Mode vs Debug Mode

```bash
# Debug Mode - ช้ากว่า Production จริง
flutter run

# Profile Mode - ใกล้เคียง Production แต่มี Profiling
flutter run --profile

# Release Mode - Production Performance
flutter run --release
```

### การ Profile ด้วย Timeline

```dart
// ใส่ Timeline events เพื่อ track performance
import 'dart:developer' as developer;

Future<void> loadData() async {
  // เริ่ม timeline event
  developer.Timeline.startSync('Load Data');
  
  try {
    final data = await fetchFromServer();
    processData(data);
  } finally {
    // จบ timeline event
    developer.Timeline.finishSync();
  }
}

// หรือใช้ timeSync
Future<void> processLargeData(List<dynamic> data) async {
  developer.Timeline.timeSync(
    'Process Large Data',
    () {
      // code ที่ต้องการ profile
      final processed = data.map((item) => transform(item)).toList();
      return processed;
    },
  );
}
```

### PerformanceMode Widget

```dart
// ใช้ PerformanceMode เพื่อ hint ให้ Flutter optimize
class AnimationScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return PerformanceMode(
      requestedMode: DartPerformanceMode.latency,
      child: YourAnimationWidget(),
    );
  }
}
```

---

## ขั้นตอนที่ 333: Widget Rebuilds Optimization

### ทำความเข้าใจ Widget Rebuild

```dart
// ❌ ปัญหา: Widget rebuild ทุกครั้งที่ state เปลี่ยน
class BadCounter extends StatefulWidget {
  @override
  State<BadCounter> createState() => _BadCounterState();
}

class _BadCounterState extends State<BadCounter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    print('BadCounter rebuild!'); // rebuild ทุกอย่าง
    
    return Column(
      children: [
        // Widget นี้ไม่เปลี่ยนแปลงเลย แต่ก็ยัง rebuild
        ExpensiveStaticWidget(), // ❌ rebuild ทุกครั้ง!
        
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

```dart
// ✅ แก้ด้วยการแยก Widget
class GoodCounter extends StatefulWidget {
  @override
  State<GoodCounter> createState() => _GoodCounterState();
}

class _GoodCounterState extends State<GoodCounter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const ExpensiveStaticWidget(), // ✅ ไม่ rebuild เพราะ const
        
        CounterDisplay(count: _count), // rebuild เฉพาะส่วนนี้
        ElevatedButton(
          onPressed: () => setState(() => _count++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// แยก Widget ที่เปลี่ยนแปลงออกมา
class CounterDisplay extends StatelessWidget {
  final int count;
  const CounterDisplay({super.key, required this.count});

  @override
  Widget build(BuildContext context) {
    return Text('Count: $count');
  }
}
```

### ใช้ ValueNotifier + ValueListenableBuilder

```dart
// ✅ rebuild เฉพาะส่วนที่จำเป็น
class OptimizedCounter extends StatefulWidget {
  @override
  State<OptimizedCounter> createState() => _OptimizedCounterState();
}

class _OptimizedCounterState extends State<OptimizedCounter> {
  final _count = ValueNotifier<int>(0);

  @override
  void dispose() {
    _count.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    print('OptimizedCounter build'); // จะ print แค่ครั้งเดียว!
    
    return Column(
      children: [
        const HeavyStaticWidget(),
        
        // Rebuild เฉพาะ Text เมื่อ count เปลี่ยน
        ValueListenableBuilder<int>(
          valueListenable: _count,
          builder: (context, value, _) {
            print('CounterText rebuild'); // print ทุกครั้งที่กด
            return Text('Count: $value');
          },
        ),
        
        ElevatedButton(
          onPressed: () => _count.value++,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### ใช้ Selector กับ Provider

```dart
// ✅ Selector rebuild เฉพาะเมื่อ selected value เปลี่ยน
class ProductPrice extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // rebuild เฉพาะเมื่อ price เปลี่ยน ไม่ใช่ทั้ง ProductModel
    return Selector<ProductViewModel, double>(
      selector: (_, viewModel) => viewModel.price,
      builder: (_, price, __) {
        return Text('฿${price.toStringAsFixed(2)}');
      },
    );
  }
}
```

---

## ขั้นตอนที่ 334: const Widget

การใช้ `const` เป็นวิธีที่ง่ายที่สุดในการ optimize performance

```dart
// ❌ ไม่ใช้ const - Widget ถูกสร้างใหม่ทุกครั้ง
Widget build(BuildContext context) {
  return Container(
    padding: EdgeInsets.all(16),      // สร้างใหม่ทุก build
    child: Text('Hello'),              // สร้างใหม่ทุก build
  );
}

// ✅ ใช้ const - Widget ถูก reuse
Widget build(BuildContext context) {
  return const Padding(
    padding: EdgeInsets.all(16),       // ✅ const
    child: Text('Hello'),              // ✅ const
  );
}
```

```dart
// const ใช้ได้กับ Widget, EdgeInsets, Color, TextStyle, etc.
class ProductCard extends StatelessWidget {
  final Product product;
  
  const ProductCard({super.key, required this.product}); // ✅ const constructor

  static const _padding = EdgeInsets.all(12.0);           // ✅ static const
  static const _titleStyle = TextStyle(                   // ✅ static const
    fontSize: 16,
    fontWeight: FontWeight.bold,
  );
  static const _divider = Divider(height: 1);            // ✅ static const

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: _padding,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(product.name, style: _titleStyle),
            _divider,
            Text('฿${product.price}'),
            const SizedBox(height: 8),                    // ✅ const
            if (product.isInStock)
              const Text(                                  // ✅ const
                'In Stock',
                style: TextStyle(color: Colors.green),
              )
            else
              const Text(                                  // ✅ const
                'Out of Stock',
                style: TextStyle(color: Colors.red),
              ),
          ],
        ),
      ),
    );
  }
}
```

### const ใน Lists

```dart
// ❌ List ถูกสร้างใหม่ทุกครั้ง
return Row(
  children: [
    Icon(Icons.star),
    Icon(Icons.star),
    Icon(Icons.star),
  ],
);

// ✅ const List
return const Row(
  children: [
    Icon(Icons.star),
    Icon(Icons.star),
    Icon(Icons.star),
  ],
);
```

---

## ขั้นตอนที่ 335: RepaintBoundary

`RepaintBoundary` แยก Widget Tree ออกเป็น Layer แยกต่างหาก ทำให้ animation ไม่กระทบส่วนอื่น

```dart
// ❌ ปัญหา: Animation ทำให้ทั้งหน้า repaint
class BadAnimationPage extends StatefulWidget {
  @override
  State<BadAnimationPage> createState() => _BadAnimationPageState();
}

class _BadAnimationPageState extends State<BadAnimationPage>
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
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Static content - ถูก repaint ทุก frame เพราะ animation ด้านล่าง!
        ExpensiveStaticList(),
        
        // Animated widget
        AnimatedBuilder(
          animation: _controller,
          builder: (_, __) => RotatingLogo(progress: _controller.value),
        ),
      ],
    );
  }
}

// ✅ แก้ด้วย RepaintBoundary
class GoodAnimationPage extends StatefulWidget {
  @override
  State<GoodAnimationPage> createState() => _GoodAnimationPageState();
}

class _GoodAnimationPageState extends State<GoodAnimationPage>
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
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Static content ใส่ใน RepaintBoundary
        RepaintBoundary(
          child: ExpensiveStaticList(), // ✅ ไม่ถูก repaint
        ),
        
        // Animation ใส่ใน RepaintBoundary แยก
        RepaintBoundary(
          child: AnimatedBuilder(
            animation: _controller,
            builder: (_, __) => RotatingLogo(progress: _controller.value),
          ),
        ),
      ],
    );
  }
}
```

### RepaintBoundary สำหรับ Complex Widgets

```dart
// ✅ ใส่ RepaintBoundary รอบ Complex Widgets
class ProductGrid extends StatelessWidget {
  final List<Product> products;
  
  const ProductGrid({super.key, required this.products});

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        childAspectRatio: 0.75,
      ),
      itemCount: products.length,
      itemBuilder: (_, index) {
        return RepaintBoundary(
          // แต่ละ card เป็น separate layer
          child: ProductCard(product: products[index]),
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 336: Expensive build() Anti-patterns

```dart
// ❌ Anti-patterns ที่ทำให้ build() ช้า

class BadWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ❌ 1. Heavy computation ใน build()
    final sortedItems = items.toList()
      ..sort((a, b) => a.name.compareTo(b.name)); // sort ทุก rebuild!
    
    // ❌ 2. Regular expressions ใน build()
    final RegExp emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$'); // สร้างใหม่ทุก build!
    
    // ❌ 3. Large objects creation
    final gradient = LinearGradient(
      colors: colors.map((c) => Color(c)).toList(), // allocation ทุก build!
    );
    
    // ❌ 4. String interpolation ที่ซับซ้อน
    final displayText = '${user.firstName} ${user.lastName} (${user.age} ปี)';
    
    return Column(children: [/* ... */]);
  }
}
```

```dart
// ✅ แก้ไข anti-patterns

class GoodWidget extends StatelessWidget {
  // ✅ Compute once, use many times
  static final RegExp _emailRegex = RegExp(
    r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$',
  );
  
  static const _gradient = LinearGradient(
    colors: [Colors.blue, Colors.purple],
  );

  @override
  Widget build(BuildContext context) {
    // ✅ ย้าย computation ออกจาก build
    // หรือใช้ memo/computed property
    return Column(children: [/* ... */]);
  }
}

// ✅ ใช้ StatefulWidget เพื่อ cache computation
class CachedSortWidget extends StatefulWidget {
  final List<Item> items;
  const CachedSortWidget({super.key, required this.items});

  @override
  State<CachedSortWidget> createState() => _CachedSortWidgetState();
}

class _CachedSortWidgetState extends State<CachedSortWidget> {
  late List<Item> _sortedItems;

  @override
  void initState() {
    super.initState();
    _sortedItems = widget.items.toList()
      ..sort((a, b) => a.name.compareTo(b.name));
  }

  @override
  void didUpdateWidget(CachedSortWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Sort อีกครั้งเฉพาะเมื่อ items เปลี่ยน
    if (oldWidget.items != widget.items) {
      _sortedItems = widget.items.toList()
        ..sort((a, b) => a.name.compareTo(b.name));
    }
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: _sortedItems.length,
      itemBuilder: (_, i) => ItemTile(item: _sortedItems[i]),
    );
  }
}
```

---

## ขั้นตอนที่ 337: Image Optimization

```dart
// ❌ การโหลด Image แบบไม่ optimize
Image.network(
  'https://example.com/large-image.jpg',
  // ไม่มี cache, ไม่มี size limit
)

// ✅ Optimize การโหลด Image
Image.network(
  'https://example.com/large-image.jpg',
  
  // กำหนดขนาดเพื่อ cache อย่างมีประสิทธิภาพ
  width: 200,
  height: 200,
  
  // Resize image ตาม pixel ratio ของ device
  cacheWidth: (200 * MediaQuery.devicePixelRatioOf(context)).round(),
  cacheHeight: (200 * MediaQuery.devicePixelRatioOf(context)).round(),
  
  // Fit type
  fit: BoxFit.cover,
  
  // Loading placeholder
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Center(
      child: CircularProgressIndicator(
        value: loadingProgress.expectedTotalBytes != null
            ? loadingProgress.cumulativeBytesLoaded /
                loadingProgress.expectedTotalBytes!
            : null,
      ),
    );
  },
  
  // Error placeholder
  errorBuilder: (context, error, stackTrace) {
    return const Icon(Icons.broken_image, size: 48);
  },
)
```

### ใช้ cached_network_image

```dart
// pubspec.yaml
// cached_network_image: ^3.3.0

import 'package:cached_network_image/cached_network_image.dart';

class OptimizedProductImage extends StatelessWidget {
  final String imageUrl;
  final double size;

  const OptimizedProductImage({
    super.key,
    required this.imageUrl,
    this.size = 100,
  });

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: imageUrl,
      width: size,
      height: size,
      fit: BoxFit.cover,
      
      // Fade in animation
      fadeInDuration: const Duration(milliseconds: 300),
      
      // Placeholder
      placeholder: (context, url) => Container(
        width: size,
        height: size,
        color: Colors.grey[200],
        child: const Center(
          child: CircularProgressIndicator(strokeWidth: 2),
        ),
      ),
      
      // Error widget
      errorWidget: (context, url, error) => Container(
        width: size,
        height: size,
        color: Colors.grey[200],
        child: const Icon(Icons.image_not_supported),
      ),
      
      // MemCache สำหรับ performance
      memCacheWidth: (size * MediaQuery.devicePixelRatioOf(context)).round(),
      memCacheHeight: (size * MediaQuery.devicePixelRatioOf(context)).round(),
    );
  }
}
```

### การ preload Images

```dart
// Preload images ที่จะใช้เร็วๆ นี้
class ImagePreloader extends StatefulWidget {
  final List<String> imageUrls;
  final Widget child;

  const ImagePreloader({
    super.key,
    required this.imageUrls,
    required this.child,
  });

  @override
  State<ImagePreloader> createState() => _ImagePreloaderState();
}

class _ImagePreloaderState extends State<ImagePreloader> {
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    _preloadImages();
  }

  Future<void> _preloadImages() async {
    for (final url in widget.imageUrls) {
      await precacheImage(
        CachedNetworkImageProvider(url),
        context,
      );
    }
  }

  @override
  Widget build(BuildContext context) => widget.child;
}
```

---

## ขั้นตอนที่ 338: List Performance

```dart
// ❌ ใช้ Column + overflow แบบนี้กับ data เยอะๆ
Column(
  children: items.map((item) => ItemWidget(item: item)).toList(),
)

// ✅ ใช้ ListView.builder - Lazy loading
ListView.builder(
  itemCount: items.length,
  
  // กำหนด itemExtent ถ้าทุก item มีความสูงเท่ากัน (เร็วมาก!)
  itemExtent: 72.0,
  
  // prototypeItem แทน itemExtent สำหรับ items ที่ซับซ้อน
  // prototypeItem: ItemWidget(item: items[0]),
  
  itemBuilder: (context, index) {
    return ItemWidget(key: ValueKey(items[index].id), item: items[index]);
  },
)
```

```dart
// ✅ ListView.separated สำหรับ List ที่มี Divider
ListView.separated(
  itemCount: products.length,
  separatorBuilder: (_, __) => const Divider(height: 1), // ✅ const Divider
  itemBuilder: (_, index) => ProductTile(product: products[index]),
)
```

### Sliver List สำหรับ Complex Layouts

```dart
// ✅ CustomScrollView + SliverList สำหรับ Performance สูงสุด
CustomScrollView(
  slivers: [
    // Header
    SliverToBoxAdapter(
      child: CategoryHeader(category: selectedCategory),
    ),
    
    // ✅ SliverList - render ที่เห็นเท่านั้น
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) {
          return RepaintBoundary(
            child: ProductCard(product: products[index]),
          );
        },
        childCount: products.length,
        
        // addAutomaticKeepAlives ปิดถ้าไม่ต้องการ keep state
        addAutomaticKeepAlives: false,
        addRepaintBoundaries: true,
      ),
    ),
    
    // Loading more indicator
    SliverToBoxAdapter(
      child: isLoadingMore
          ? const Center(child: CircularProgressIndicator())
          : const SizedBox.shrink(),
    ),
  ],
)
```

### Pagination ด้วย ScrollController

```dart
class InfiniteProductList extends StatefulWidget {
  @override
  State<InfiniteProductList> createState() => _InfiniteProductListState();
}

class _InfiniteProductListState extends State<InfiniteProductList> {
  final ScrollController _scrollController = ScrollController();
  final List<Product> _products = [];
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 1;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent * 0.8) {
      // Load more เมื่อ scroll ถึง 80% ของ list
      if (!_isLoading && _hasMore) {
        _loadMore();
      }
    }
  }

  Future<void> _loadMore() async {
    if (_isLoading) return;
    
    setState(() => _isLoading = true);
    
    try {
      final newProducts = await fetchProducts(page: _page);
      setState(() {
        _products.addAll(newProducts);
        _page++;
        _hasMore = newProducts.length == 20;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: _products.length + (_isLoading ? 1 : 0),
      itemBuilder: (context, index) {
        if (index == _products.length) {
          return const Center(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: CircularProgressIndicator(),
            ),
          );
        }
        return ProductCard(product: _products[index]);
      },
    );
  }
}
```

---

## ขั้นตอนที่ 339: compute() สำหรับ Heavy Computation

`compute()` ส่ง computation ไปรันใน separate Isolate เพื่อไม่บล็อก UI Thread

```dart
import 'package:flutter/foundation.dart';

// ❌ ปัญหา: Heavy computation บน UI Thread
Future<void> processDataBad() async {
  setState(() => isLoading = true);
  
  // รันใน UI Thread - ทำให้ UI กระตุก!
  final result = await _parseHugeJson(rawData); // ❌
  
  setState(() {
    data = result;
    isLoading = false;
  });
}

// ✅ ใช้ compute() ส่ง task ไป Isolate
Future<void> processDataGood() async {
  setState(() => isLoading = true);
  
  // รันใน separate Isolate - UI ไม่กระตุก!
  final result = await compute(_parseHugeJson, rawData); // ✅
  
  setState(() {
    data = result;
    isLoading = false;
  });
}

// Function ต้องเป็น top-level หรือ static
List<Product> _parseHugeJson(String jsonString) {
  final data = json.decode(jsonString) as List;
  return data.map((item) => Product.fromJson(item)).toList();
}
```

```dart
// ตัวอย่างการใช้ compute() กับ data หลายประเภท
class DataProcessor {
  // Process large list
  static Future<List<ProcessedItem>> processItems(
    List<RawItem> items,
  ) async {
    return compute(_processItemsList, items);
  }
  
  static List<ProcessedItem> _processItemsList(List<RawItem> items) {
    return items
        .where((item) => item.isValid)
        .map((item) => ProcessedItem(
              id: item.id,
              name: item.name.trim().toUpperCase(),
              score: _calculateScore(item),
            ))
        .toList();
  }

  // Parse JSON
  static Future<UserData> parseUserData(String jsonStr) async {
    return compute(_parseUserJson, jsonStr);
  }

  static UserData _parseUserJson(String jsonStr) {
    final map = json.decode(jsonStr) as Map<String, dynamic>;
    return UserData.fromJson(map);
  }

  // Image processing
  static Future<Uint8List> compressImage(Uint8List imageBytes) async {
    return compute(_compressImageBytes, imageBytes);
  }

  static Uint8List _compressImageBytes(Uint8List bytes) {
    // Heavy image processing here
    return processedBytes;
  }
}
```

### Isolate สำหรับ Long Running Tasks

```dart
import 'dart:isolate';

// สำหรับ task ที่ต้องรันนาน และส่งผลลัพธ์กลับมาเรื่อยๆ
class BackgroundProcessor {
  ReceivePort? _receivePort;
  Isolate? _isolate;

  Future<void> startProcessing(List<Item> items) async {
    _receivePort = ReceivePort();
    
    _isolate = await Isolate.spawn(
      _processInBackground,
      {
        'sendPort': _receivePort!.sendPort,
        'items': items,
      },
    );
    
    _receivePort!.listen((message) {
      if (message is ProcessingProgress) {
        updateProgress(message.progress);
      } else if (message is ProcessingComplete) {
        handleComplete(message.results);
      }
    });
  }

  void stop() {
    _isolate?.kill();
    _receivePort?.close();
    _isolate = null;
    _receivePort = null;
  }
}

// Top-level function สำหรับ Isolate
void _processInBackground(Map<String, dynamic> params) {
  final sendPort = params['sendPort'] as SendPort;
  final items = params['items'] as List<Item>;
  
  final results = <ProcessedItem>[];
  
  for (var i = 0; i < items.length; i++) {
    results.add(processItem(items[i]));
    
    // ส่ง progress กลับมา
    if (i % 10 == 0) {
      sendPort.send(ProcessingProgress(
        progress: i / items.length,
      ));
    }
  }
  
  sendPort.send(ProcessingComplete(results: results));
}
```

---

## ขั้นตอนที่ 340: Memory Leaks Detection

### ประเภทของ Memory Leaks ใน Flutter

```dart
// ❌ Memory Leak 1: ไม่ dispose Controller
class BadWidget extends StatefulWidget {
  @override
  State<BadWidget> createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  final TextEditingController _controller = TextEditingController(); // ❌ ไม่ dispose!
  final AnimationController _animController = AnimationController(vsync: this); // ❌

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
  
  // ขาด @override void dispose()!
}

// ✅ แก้ไข: dispose ทุก controller
class GoodWidget extends StatefulWidget {
  @override
  State<GoodWidget> createState() => _GoodWidgetState();
}

class _GoodWidgetState extends State<GoodWidget> with SingleTickerProviderStateMixin {
  final _controller = TextEditingController();
  late final AnimationController _animController;

  @override
  void initState() {
    super.initState();
    _animController = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose(); // ✅
    _animController.dispose(); // ✅
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(controller: _controller);
  }
}
```

```dart
// ❌ Memory Leak 2: ใช้ setState หลัง dispose
class AsyncWidget extends StatefulWidget {
  @override
  State<AsyncWidget> createState() => _AsyncWidgetState();
}

class _AsyncWidgetState extends State<AsyncWidget> {
  String? _data;

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  Future<void> _loadData() async {
    final data = await fetchData(); // Widget อาจถูก dispose ระหว่างรอ!
    
    // ❌ อาจ throw exception ถ้า widget ถูก dispose ไปแล้ว
    setState(() => _data = data);
  }

  @override
  Widget build(BuildContext context) {
    return Text(_data ?? 'Loading...');
  }
}

// ✅ แก้ด้วยการ check mounted
class SafeAsyncWidget extends StatefulWidget {
  @override
  State<SafeAsyncWidget> createState() => _SafeAsyncWidgetState();
}

class _SafeAsyncWidgetState extends State<SafeAsyncWidget> {
  String? _data;

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  Future<void> _loadData() async {
    final data = await fetchData();
    
    // ✅ Check mounted ก่อน setState
    if (mounted) {
      setState(() => _data = data);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Text(_data ?? 'Loading...');
  }
}
```

```dart
// ❌ Memory Leak 3: Stream ไม่ cancel subscription
class StreamWidget extends StatefulWidget {
  @override
  State<StreamWidget> createState() => _StreamWidgetState();
}

class _StreamWidgetState extends State<StreamWidget> {
  StreamSubscription? _subscription; // ❌ ไม่ cancel!

  @override
  void initState() {
    super.initState();
    dataStream.listen((data) {
      setState(() {/* update */});
    });
  }

  // ขาด dispose!
}

// ✅ Cancel stream subscription
class SafeStreamWidget extends StatefulWidget {
  @override
  State<SafeStreamWidget> createState() => _SafeStreamWidgetState();
}

class _SafeStreamWidgetState extends State<SafeStreamWidget> {
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = dataStream.listen((data) {
      if (mounted) setState(() {/* update */});
    });
  }

  @override
  void dispose() {
    _subscription?.cancel(); // ✅
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Container();
}
```

### ตรวจจับ Memory Leaks ด้วย DevTools

```dart
// ใช้ MemoryAllocations เพื่อ track object lifecycle
import 'package:flutter/foundation.dart';

class TrackableObject with ChangeNotifier {
  TrackableObject() {
    if (kDebugMode) {
      MemoryAllocations.instance.dispatchObjectCreated(
        library: 'my_library',
        className: 'TrackableObject',
        object: this,
      );
    }
  }

  @override
  void dispose() {
    if (kDebugMode) {
      MemoryAllocations.instance.dispatchObjectDisposed(object: this);
    }
    super.dispose();
  }
}
```

### Workshop: Performance Audit

```dart
// Workshop: ตรวจสอบ Performance ของ App

// Step 1: Enable Performance Overlay
MaterialApp(
  showPerformanceOverlay: true,
  home: MyApp(),
)

// Step 2: Run ใน Profile Mode
// flutter run --profile

// Step 3: เปิด DevTools
// flutter pub global activate devtools
// flutter pub global run devtools

// Step 4: เก็บ Timeline ด้วย dart:developer
import 'dart:developer' as dev;

Widget buildExpensiveWidget() {
  dev.Timeline.startSync('Build ExpensiveWidget');
  try {
    return _actualBuild();
  } finally {
    dev.Timeline.finishSync();
  }
}

// Step 5: ตรวจสอบ Widget Rebuilds
// ใช้ debugRepaintRainbowEnabled
import 'package:flutter/rendering.dart';

void main() {
  debugRepaintRainbowEnabled = true; // แต่ละ repaint เปลี่ยนสี
  runApp(const MyApp());
}
```

---

## สรุป (Summary)

```
Performance Checklist:

✅ Widget Optimization:
  - ใช้ const constructor/variables ทุกที่ที่ทำได้
  - แยก Widget เล็กๆ แทนการ rebuild Widget ใหญ่
  - ใช้ RepaintBoundary สำหรับ Animation
  - ใช้ ValueListenableBuilder, Selector แทน setState

✅ List Performance:
  - ใช้ ListView.builder แทน Column + map
  - กำหนด itemExtent ถ้า items มีความสูงเท่ากัน
  - ใช้ addRepaintBoundaries: true ใน SliverChildBuilderDelegate

✅ Image Performance:
  - กำหนด cacheWidth/cacheHeight
  - ใช้ CachedNetworkImage
  - Preload images ที่จะใช้เร็วๆ นี้

✅ Computation:
  - ใช้ compute() สำหรับ heavy computation
  - อย่าทำ heavy work ใน build()
  - Cache computed values

✅ Memory:
  - dispose() ทุก Controller, Animation
  - cancel() Stream subscriptions
  - Check mounted ก่อน setState async
  - ใช้ DevTools Memory Profiler

Tools:
- Flutter DevTools: ตรวจ Frame time, Widget rebuilds
- Timeline: Track specific operations
- Memory Profiler: ตรวจ Memory leaks
- Performance Overlay: ดู GPU/UI thread usage
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. เพิ่ม `const` keyword ให้ทุก Widget ที่สามารถทำได้ในโปรเจคของคุณ
2. แปลง `Column + map` เป็น `ListView.builder`
3. เพิ่ม `dispose()` method ที่ทุก StatefulWidget

**ระดับกลาง:**
4. ใช้ `RepaintBoundary` สำหรับ Animation Widget
5. ย้าย JSON parsing ไปใช้ `compute()`
6. Implement Infinite Scroll ด้วย ScrollController

**ระดับสูง:**
7. Profile App ด้วย Flutter DevTools และหา bottlenecks
8. ลด Widget rebuilds โดยใช้ Selector แทน Consumer
9. Implement Image caching strategy ที่ effective

---

## การนำทาง
- [← Part 33: CI/CD Pipeline](part_33.md)
- [→ Part 35: Accessibility](part_35.md)

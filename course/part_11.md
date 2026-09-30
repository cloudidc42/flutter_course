# Part 11: ListView, GridView, Slivers
## ขั้นตอนที่ 101-110

---

## สารบัญ
1. [ListView.builder แบบเจาะลึก](#ขั้นตอนที่-101-listviewbuilder-แบบเจาะลึก)
2. [Pull-to-Refresh ด้วย RefreshIndicator](#ขั้นตอนที่-102-pull-to-refresh-ด้วย-refreshindicator)
3. [Infinite Scroll](#ขั้นตอนที่-103-infinite-scroll)
4. [GridView.builder](#ขั้นตอนที่-104-gridviewbuilder)
5. [SliverList](#ขั้นตอนที่-105-sliverlist)
6. [SliverGrid](#ขั้นตอนที่-106-slivergrid)
7. [SliverAppBar](#ขั้นตอนที่-107-sliverappbar)
8. [CustomScrollView](#ขั้นตอนที่-108-customscrollview)
9. [NestedScrollView](#ขั้นตอนที่-109-nestedscrollview)
10. [Sticky Headers](#ขั้นตอนที่-110-sticky-headers)
11. [Workshop: News Feed App](#workshop-news-feed-app)

---

## ขั้นตอนที่ 101: ListView.builder แบบเจาะลึก

`ListView.builder` เป็นวิธีที่มีประสิทธิภาพในการแสดงรายการข้อมูลจำนวนมาก เพราะสร้าง Widget เฉพาะที่มองเห็นเท่านั้น (Lazy Loading)

### ความแตกต่างระหว่าง ListView แบบต่างๆ

```dart
// 1. ListView ธรรมดา - สร้าง children ทั้งหมดทันที
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    // ...
  ],
)

// 2. ListView.builder - สร้าง item เฉพาะที่มองเห็น (แนะนำสำหรับข้อมูลมาก)
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
)

// 3. ListView.separated - มี separator ระหว่าง item
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
  separatorBuilder: (context, index) => Divider(),
)

// 4. ListView.custom - กำหนด SliverChildDelegate เอง
ListView.custom(
  childrenDelegate: SliverChildBuilderDelegate(
    (context, index) => ListTile(title: Text(items[index])),
    childCount: items.length,
  ),
)
```

### ตัวอย่าง ListView.builder แบบสมบูรณ์

```dart
import 'package:flutter/material.dart';

class Product {
  final int id;
  final String name;
  final String category;
  final double price;
  final String imageUrl;
  bool isFavorite;

  Product({
    required this.id,
    required this.name,
    required this.category,
    required this.price,
    required this.imageUrl,
    this.isFavorite = false,
  });
}

class ProductListScreen extends StatefulWidget {
  const ProductListScreen({super.key});

  @override
  State<ProductListScreen> createState() => _ProductListScreenState();
}

class _ProductListScreenState extends State<ProductListScreen> {
  final List<Product> _products = List.generate(
    50,
    (index) => Product(
      id: index + 1,
      name: 'Product ${index + 1}',
      category: ['Electronics', 'Clothing', 'Food', 'Books'][index % 4],
      price: (index + 1) * 99.99,
      imageUrl: 'https://picsum.photos/200/200?random=$index',
    ),
  );

  final ScrollController _scrollController = ScrollController();
  bool _showFab = false;

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(() {
      setState(() {
        _showFab = _scrollController.offset > 200;
      });
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('รายการสินค้า'),
        actions: [
          IconButton(
            icon: const Icon(Icons.sort),
            onPressed: _sortProducts,
          ),
        ],
      ),
      body: ListView.builder(
        controller: _scrollController,
        itemCount: _products.length,
        // กำหนด itemExtent เพื่อเพิ่มประสิทธิภาพ (เมื่อ item มีความสูงเท่ากัน)
        // itemExtent: 80,
        padding: const EdgeInsets.all(8),
        itemBuilder: (context, index) {
          final product = _products[index];
          return _buildProductCard(product, index);
        },
      ),
      floatingActionButton: _showFab
          ? FloatingActionButton(
              onPressed: () {
                _scrollController.animateTo(
                  0,
                  duration: const Duration(milliseconds: 500),
                  curve: Curves.easeOut,
                );
              },
              child: const Icon(Icons.arrow_upward),
            )
          : null,
    );
  }

  Widget _buildProductCard(Product product, int index) {
    return Card(
      margin: const EdgeInsets.symmetric(vertical: 4, horizontal: 8),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Theme.of(context).colorScheme.primaryContainer,
          child: Text(
            '${product.id}',
            style: TextStyle(
              color: Theme.of(context).colorScheme.onPrimaryContainer,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
        title: Text(product.name),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(product.category),
            Text(
              '฿${product.price.toStringAsFixed(2)}',
              style: TextStyle(
                color: Theme.of(context).colorScheme.primary,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
        trailing: IconButton(
          icon: Icon(
            product.isFavorite ? Icons.favorite : Icons.favorite_border,
            color: product.isFavorite ? Colors.red : null,
          ),
          onPressed: () {
            setState(() {
              product.isFavorite = !product.isFavorite;
            });
          },
        ),
        onTap: () {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('เลือก: ${product.name}')),
          );
        },
      ),
    );
  }

  void _sortProducts() {
    setState(() {
      _products.sort((a, b) => a.price.compareTo(b.price));
    });
  }
}
```

### การจัดการ ScrollController และ ScrollPhysics

```dart
// ScrollPhysics แบบต่างๆ
ListView.builder(
  // ฟิสิกส์การ scroll แบบ iOS (ดีดขึ้นขอบ)
  physics: const BouncingScrollPhysics(),
  
  // ฟิสิกส์การ scroll แบบ Android (หยุดที่ขอบ)
  // physics: const ClampingScrollPhysics(),
  
  // ปิดการ scroll
  // physics: const NeverScrollableScrollPhysics(),
  
  itemCount: 20,
  itemBuilder: (context, index) => ListTile(title: Text('Item $index')),
)
```

---

## ขั้นตอนที่ 102: Pull-to-Refresh ด้วย RefreshIndicator

`RefreshIndicator` ช่วยให้ผู้ใช้ดึงหน้าจอลงเพื่อรีเฟรชข้อมูล ซึ่งเป็น UX pattern ที่ผู้ใช้คุ้นเคย

```dart
import 'package:flutter/material.dart';

class NewsListScreen extends StatefulWidget {
  const NewsListScreen({super.key});

  @override
  State<NewsListScreen> createState() => _NewsListScreenState();
}

class _NewsListScreenState extends State<NewsListScreen> {
  List<Map<String, String>> _news = [];
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadNews();
  }

  // จำลองการโหลดข้อมูลจาก API
  Future<void> _loadNews() async {
    setState(() => _isLoading = true);
    
    // จำลอง network delay
    await Future.delayed(const Duration(seconds: 2));
    
    setState(() {
      _news = List.generate(
        20,
        (index) => {
          'title': 'ข่าวที่ ${index + 1}: ${_getRandomTitle(index)}',
          'description': 'รายละเอียดข่าว ${index + 1} ที่น่าสนใจและมีประโยชน์...',
          'time': '${index + 1} ชั่วโมงที่แล้ว',
          'category': ['กีฬา', 'การเมือง', 'บันเทิง', 'เทคโนโลยี'][index % 4],
        },
      );
      _isLoading = false;
    });
  }

  String _getRandomTitle(int index) {
    final titles = [
      'เหตุการณ์สำคัญประจำวัน',
      'ข่าวด่วนจากทั่วโลก',
      'อัปเดตล่าสุด',
      'รายงานพิเศษ',
    ];
    return titles[index % titles.length];
  }

  // ฟังก์ชันสำหรับ refresh
  Future<void> _onRefresh() async {
    // ล้างข้อมูลเก่าและโหลดใหม่
    await _loadNews();
    
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('รีเฟรชข้อมูลสำเร็จ!'),
          duration: Duration(seconds: 2),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ข่าวล่าสุด'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _onRefresh,
          ),
        ],
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : RefreshIndicator(
              onRefresh: _onRefresh,
              // กำหนดสีของ indicator
              color: Theme.of(context).colorScheme.primary,
              backgroundColor: Theme.of(context).colorScheme.surface,
              // ระยะที่ต้องดึงก่อน trigger refresh
              displacement: 60,
              child: ListView.builder(
                itemCount: _news.length,
                itemBuilder: (context, index) {
                  final newsItem = _news[index];
                  return _buildNewsCard(newsItem);
                },
              ),
            ),
    );
  }

  Widget _buildNewsCard(Map<String, String> newsItem) {
    final categoryColors = {
      'กีฬา': Colors.green,
      'การเมือง': Colors.blue,
      'บันเทิง': Colors.purple,
      'เทคโนโลยี': Colors.orange,
    };

    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Container(
                  padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                  decoration: BoxDecoration(
                    color: categoryColors[newsItem['category']] ?? Colors.grey,
                    borderRadius: BorderRadius.circular(4),
                  ),
                  child: Text(
                    newsItem['category'] ?? '',
                    style: const TextStyle(color: Colors.white, fontSize: 12),
                  ),
                ),
                Text(
                  newsItem['time'] ?? '',
                  style: Theme.of(context).textTheme.bodySmall,
                ),
              ],
            ),
            const SizedBox(height: 8),
            Text(
              newsItem['title'] ?? '',
              style: Theme.of(context).textTheme.titleMedium?.copyWith(
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              newsItem['description'] ?? '',
              style: Theme.of(context).textTheme.bodyMedium,
              maxLines: 2,
              overflow: TextOverflow.ellipsis,
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 103: Infinite Scroll

Infinite Scroll หรือ Pagination คือการโหลดข้อมูลเพิ่มเติมเมื่อผู้ใช้ scroll ถึงด้านล่าง

```dart
import 'package:flutter/material.dart';

class InfiniteScrollScreen extends StatefulWidget {
  const InfiniteScrollScreen({super.key});

  @override
  State<InfiniteScrollScreen> createState() => _InfiniteScrollScreenState();
}

class _InfiniteScrollScreenState extends State<InfiniteScrollScreen> {
  final List<Map<String, dynamic>> _items = [];
  final ScrollController _scrollController = ScrollController();
  bool _isLoading = false;
  bool _hasMore = true;
  int _page = 1;
  static const int _pageSize = 20;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.removeListener(_onScroll);
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      // โหลดข้อมูลเพิ่มเมื่อเหลือ 200 pixels ถึงด้านล่าง
      _loadMore();
    }
  }

  Future<void> _loadMore() async {
    if (_isLoading || !_hasMore) return;

    setState(() => _isLoading = true);

    // จำลอง API call
    await Future.delayed(const Duration(seconds: 1));

    final newItems = List.generate(
      _pageSize,
      (index) {
        final itemIndex = (_page - 1) * _pageSize + index + 1;
        return {
          'id': itemIndex,
          'title': 'รายการที่ $itemIndex',
          'subtitle': 'รายละเอียดรายการที่ $itemIndex',
          'value': itemIndex * 10,
        };
      },
    );

    setState(() {
      _items.addAll(newItems);
      _page++;
      _isLoading = false;
      // จำกัดข้อมูลที่ 100 รายการ
      _hasMore = _items.length < 100;
    });
  }

  Future<void> _refresh() async {
    setState(() {
      _items.clear();
      _page = 1;
      _hasMore = true;
    });
    await _loadMore();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Infinite Scroll')),
      body: RefreshIndicator(
        onRefresh: _refresh,
        child: ListView.builder(
          controller: _scrollController,
          itemCount: _items.length + (_hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            // แสดง Loading indicator ที่ด้านล่าง
            if (index == _items.length) {
              return _buildLoadingIndicator();
            }

            final item = _items[index];
            return _buildItem(item);
          },
        ),
      ),
    );
  }

  Widget _buildItem(Map<String, dynamic> item) {
    return ListTile(
      leading: CircleAvatar(
        child: Text('${item['id']}'),
      ),
      title: Text(item['title'] as String),
      subtitle: Text(item['subtitle'] as String),
      trailing: Chip(
        label: Text('${item['value']}'),
      ),
    );
  }

  Widget _buildLoadingIndicator() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Center(
        child: _isLoading
            ? const Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 8),
                  Text('กำลังโหลด...'),
                ],
              )
            : const Text('โหลดข้อมูลทั้งหมดแล้ว'),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 104: GridView.builder

`GridView.builder` ใช้สำหรับแสดงข้อมูลในรูปแบบตาราง เหมาะสำหรับ gallery, product catalog เป็นต้น

```dart
import 'package:flutter/material.dart';

class PhotoGalleryScreen extends StatefulWidget {
  const PhotoGalleryScreen({super.key});

  @override
  State<PhotoGalleryScreen> createState() => _PhotoGalleryScreenState();
}

class _PhotoGalleryScreenState extends State<PhotoGalleryScreen> {
  final List<Map<String, dynamic>> _photos = List.generate(
    50,
    (index) => {
      'id': index + 1,
      'url': 'https://picsum.photos/300/300?random=$index',
      'title': 'รูปภาพ ${index + 1}',
      'isSelected': false,
    },
  );

  int _crossAxisCount = 3;
  bool _isSelectionMode = false;
  final Set<int> _selectedItems = {};

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: _isSelectionMode
            ? Text('เลือกแล้ว ${_selectedItems.length} รายการ')
            : const Text('แกลเลอรี่'),
        actions: [
          if (_isSelectionMode) ...[
            IconButton(
              icon: const Icon(Icons.delete),
              onPressed: _deleteSelected,
            ),
            IconButton(
              icon: const Icon(Icons.close),
              onPressed: () {
                setState(() {
                  _isSelectionMode = false;
                  _selectedItems.clear();
                });
              },
            ),
          ] else ...[
            // Toggle column count
            IconButton(
              icon: const Icon(Icons.grid_view),
              onPressed: () {
                setState(() {
                  _crossAxisCount = _crossAxisCount == 2 ? 3 : 2;
                });
              },
            ),
          ],
        ],
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(4),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: _crossAxisCount,
          crossAxisSpacing: 4,
          mainAxisSpacing: 4,
          // childAspectRatio: ความกว้าง / ความสูง
          childAspectRatio: 1.0,
        ),
        itemCount: _photos.length,
        itemBuilder: (context, index) {
          final photo = _photos[index];
          final isSelected = _selectedItems.contains(index);

          return _buildPhotoItem(photo, index, isSelected);
        },
      ),
    );
  }

  Widget _buildPhotoItem(
      Map<String, dynamic> photo, int index, bool isSelected) {
    return GestureDetector(
      onTap: () {
        if (_isSelectionMode) {
          setState(() {
            if (isSelected) {
              _selectedItems.remove(index);
              if (_selectedItems.isEmpty) _isSelectionMode = false;
            } else {
              _selectedItems.add(index);
            }
          });
        } else {
          // เปิดดูรูปเต็ม
          _openPhoto(photo);
        }
      },
      onLongPress: () {
        setState(() {
          _isSelectionMode = true;
          _selectedItems.add(index);
        });
      },
      child: Stack(
        fit: StackFit.expand,
        children: [
          // รูปภาพ
          Container(
            color: Colors.grey[200],
            child: Image.network(
              photo['url'] as String,
              fit: BoxFit.cover,
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
              errorBuilder: (context, error, stackTrace) {
                return Container(
                  color: Colors.grey[300],
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Icon(Icons.broken_image, color: Colors.grey),
                      Text(
                        'โหลดไม่ได้',
                        style: TextStyle(color: Colors.grey[600], fontSize: 10),
                      ),
                    ],
                  ),
                );
              },
            ),
          ),
          // Overlay สำหรับ selection mode
          if (_isSelectionMode)
            Container(
              color: isSelected
                  ? Colors.blue.withOpacity(0.4)
                  : Colors.transparent,
            ),
          // Checkbox
          if (_isSelectionMode)
            Positioned(
              top: 4,
              right: 4,
              child: Container(
                width: 24,
                height: 24,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: isSelected ? Colors.blue : Colors.white,
                  border: Border.all(
                    color: isSelected ? Colors.blue : Colors.grey,
                    width: 2,
                  ),
                ),
                child: isSelected
                    ? const Icon(Icons.check, color: Colors.white, size: 16)
                    : null,
              ),
            ),
        ],
      ),
    );
  }

  void _openPhoto(Map<String, dynamic> photo) {
    showDialog(
      context: context,
      builder: (context) => Dialog(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Image.network(photo['url'] as String),
            Padding(
              padding: const EdgeInsets.all(8),
              child: Text(photo['title'] as String),
            ),
          ],
        ),
      ),
    );
  }

  void _deleteSelected() {
    setState(() {
      final sortedIndices = _selectedItems.toList()..sort((a, b) => b.compareTo(a));
      for (final index in sortedIndices) {
        _photos.removeAt(index);
      }
      _selectedItems.clear();
      _isSelectionMode = false;
    });
  }
}
```

### GridView กับ SliverGridDelegateWithMaxCrossAxisExtent

```dart
// ใช้เมื่อต้องการกำหนดความกว้างสูงสุดของแต่ละ cell
GridView.builder(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 200, // แต่ละ cell กว้างสูงสุด 200 pixels
    crossAxisSpacing: 8,
    mainAxisSpacing: 8,
    childAspectRatio: 1.5, // กว้าง:สูง = 1.5:1
  ),
  itemCount: 50,
  itemBuilder: (context, index) => Card(
    child: Center(child: Text('Item $index')),
  ),
)
```

---

## ขั้นตอนที่ 105: SliverList

`SliverList` คือ List ที่ทำงานภายใน `CustomScrollView` ซึ่งช่วยให้ combine List, Grid, AppBar ได้อย่างอิสระ

```dart
import 'package:flutter/material.dart';

class SliverListExample extends StatelessWidget {
  const SliverListExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // SliverAppBar
          const SliverAppBar(
            title: Text('SliverList Example'),
            floating: true,
            snap: true,
          ),
          // SliverList แบบ builder (แนะนำสำหรับ list ยาว)
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return ListTile(
                  leading: CircleAvatar(child: Text('$index')),
                  title: Text('รายการที่ $index'),
                  subtitle: Text('รายละเอียดรายการที่ $index'),
                );
              },
              childCount: 30,
            ),
          ),
        ],
      ),
    );
  }
}

// SliverList แบบ fixed list (สำหรับ list สั้นๆ)
SliverList(
  delegate: SliverChildListDelegate([
    const ListTile(title: Text('Item 1')),
    const Divider(),
    const ListTile(title: Text('Item 2')),
    const Divider(),
    const ListTile(title: Text('Item 3')),
  ]),
)
```

---

## ขั้นตอนที่ 106: SliverGrid

```dart
import 'package:flutter/material.dart';

class SliverGridExample extends StatelessWidget {
  const SliverGridExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          const SliverAppBar(
            title: Text('SliverGrid'),
            pinned: true,
          ),
          // Header section
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: const Text(
                'หมวดหมู่สินค้า',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
            ),
          ),
          // Grid ของ categories
          SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
              childAspectRatio: 2.0,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                final categories = [
                  ('อิเล็กทรอนิกส์', Icons.devices, Colors.blue),
                  ('เสื้อผ้า', Icons.checkroom, Colors.pink),
                  ('อาหาร', Icons.restaurant, Colors.orange),
                  ('หนังสือ', Icons.book, Colors.green),
                  ('กีฬา', Icons.sports_soccer, Colors.red),
                  ('ความงาม', Icons.face, Colors.purple),
                ];
                final (name, icon, color) = categories[index % categories.length];
                return Card(
                  color: color.withOpacity(0.1),
                  child: Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(icon, color: color),
                      const SizedBox(width: 8),
                      Text(
                        name,
                        style: TextStyle(
                          color: color,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),
                );
              },
              childCount: 6,
            ),
          ),
          // Divider
          const SliverToBoxAdapter(child: Divider()),
          // รายการสินค้า
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: const Text(
                'สินค้าแนะนำ',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
            ),
          ),
          SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 8,
              mainAxisSpacing: 8,
              childAspectRatio: 0.75,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return Card(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Expanded(
                        child: Container(
                          color: Colors.grey[200],
                          child: Center(
                            child: Text(
                              '🖼️',
                              style: const TextStyle(fontSize: 40),
                            ),
                          ),
                        ),
                      ),
                      Padding(
                        padding: const EdgeInsets.all(8),
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              'สินค้า ${index + 1}',
                              style: const TextStyle(fontWeight: FontWeight.bold),
                            ),
                            Text(
                              '฿${(index + 1) * 99}',
                              style: const TextStyle(color: Colors.red),
                            ),
                          ],
                        ),
                      ),
                    ],
                  ),
                );
              },
              childCount: 20,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 107: SliverAppBar

`SliverAppBar` คือ AppBar ที่ขยาย/ย่อตัวได้เมื่อ scroll ซึ่งให้ผลลัพธ์ที่สวยงาม

```dart
import 'package:flutter/material.dart';

class SliverAppBarExample extends StatelessWidget {
  const SliverAppBarExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            // ความสูงที่ขยายออก
            expandedHeight: 250,
            
            // ติดอยู่ด้านบนเมื่อ scroll ขึ้น
            pinned: true,
            
            // โผล่มาเมื่อ scroll ขึ้น
            floating: false,
            
            // snap กลับทันทีเมื่อ scroll ขึ้น (ต้องใช้กับ floating: true)
            // snap: true,
            
            backgroundColor: Theme.of(context).colorScheme.primaryContainer,
            
            // FlexibleSpaceBar - ส่วนที่ขยายออก
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('โปรไฟล์ผู้ใช้'),
              background: Stack(
                fit: StackFit.expand,
                children: [
                  // Background image
                  Image.network(
                    'https://picsum.photos/800/400?random=1',
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) => Container(
                      color: Colors.blue[200],
                    ),
                  ),
                  // Gradient overlay
                  Container(
                    decoration: BoxDecoration(
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
                  // Profile info
                  Positioned(
                    bottom: 60,
                    left: 16,
                    child: Row(
                      children: [
                        const CircleAvatar(
                          radius: 30,
                          backgroundImage: NetworkImage(
                            'https://picsum.photos/100/100?random=2',
                          ),
                        ),
                        const SizedBox(width: 12),
                        Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          mainAxisSize: MainAxisSize.min,
                          children: const [
                            Text(
                              'สมชาย ใจดี',
                              style: TextStyle(
                                color: Colors.white,
                                fontSize: 18,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                            Text(
                              '@somchai_jaidee',
                              style: TextStyle(
                                color: Colors.white70,
                                fontSize: 14,
                              ),
                            ),
                          ],
                        ),
                      ],
                    ),
                  ),
                ],
              ),
              // ปรับ title เมื่อ collapse
              titlePadding: const EdgeInsets.only(left: 16, bottom: 16),
              collapseMode: CollapseMode.parallax,
            ),
            
            actions: [
              IconButton(
                icon: const Icon(Icons.share),
                onPressed: () {},
              ),
              IconButton(
                icon: const Icon(Icons.more_vert),
                onPressed: () {},
              ),
            ],
          ),
          
          // Stats bar
          SliverToBoxAdapter(
            child: Container(
              padding: const EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  _buildStat('โพสต์', '128'),
                  _buildStat('ผู้ติดตาม', '4.2K'),
                  _buildStat('กำลังติดตาม', '356'),
                ],
              ),
            ),
          ),
          
          const SliverToBoxAdapter(child: Divider()),
          
          // Content list
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                leading: const Icon(Icons.article),
                title: Text('โพสต์ที่ ${index + 1}'),
                subtitle: Text('เนื้อหาโพสต์ ${index + 1}...'),
                trailing: const Icon(Icons.chevron_right),
              ),
              childCount: 30,
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildStat(String label, String value) {
    return Column(
      children: [
        Text(
          value,
          style: const TextStyle(
            fontSize: 20,
            fontWeight: FontWeight.bold,
          ),
        ),
        Text(
          label,
          style: const TextStyle(color: Colors.grey),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 108: CustomScrollView

`CustomScrollView` ให้อิสระในการจัด Sliver ต่างๆ ตามต้องการ

```dart
import 'package:flutter/material.dart';

class CustomScrollViewExample extends StatelessWidget {
  const CustomScrollViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        // Physics การ scroll
        physics: const BouncingScrollPhysics(),
        
        slivers: [
          // 1. SliverAppBar
          SliverAppBar(
            expandedHeight: 200,
            floating: false,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('แอปซื้อของ'),
              background: Container(
                decoration: const BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Colors.purple, Colors.deepPurple],
                  ),
                ),
              ),
            ),
          ),
          
          // 2. Padding wrapper
          const SliverPadding(
            padding: EdgeInsets.all(16),
            sliver: SliverToBoxAdapter(
              child: Text(
                'แบนเนอร์โปรโมชัน',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
            ),
          ),
          
          // 3. SliverToBoxAdapter สำหรับ Banner
          SliverToBoxAdapter(
            child: SizedBox(
              height: 150,
              child: PageView.builder(
                itemCount: 5,
                itemBuilder: (context, index) {
                  final colors = [
                    Colors.red,
                    Colors.blue,
                    Colors.green,
                    Colors.orange,
                    Colors.purple,
                  ];
                  return Container(
                    margin: const EdgeInsets.symmetric(horizontal: 8),
                    decoration: BoxDecoration(
                      color: colors[index],
                      borderRadius: BorderRadius.circular(12),
                    ),
                    child: Center(
                      child: Text(
                        'แบนเนอร์ ${index + 1}',
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  );
                },
              ),
            ),
          ),
          
          // 4. SliverPadding + SliverGrid สำหรับ categories
          SliverPadding(
            padding: const EdgeInsets.all(16),
            sliver: SliverGrid(
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 4,
                crossAxisSpacing: 8,
                mainAxisSpacing: 8,
                childAspectRatio: 0.8,
              ),
              delegate: SliverChildBuilderDelegate(
                (context, index) {
                  final items = [
                    ('แฟชั่น', '👗'),
                    ('อาหาร', '🍜'),
                    ('เกม', '🎮'),
                    ('ของใช้', '🏠'),
                  ];
                  return Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        items[index % 4].$2,
                        style: const TextStyle(fontSize: 28),
                      ),
                      const SizedBox(height: 4),
                      Text(
                        items[index % 4].$1,
                        style: const TextStyle(fontSize: 12),
                        textAlign: TextAlign.center,
                      ),
                    ],
                  );
                },
                childCount: 8,
              ),
            ),
          ),
          
          // 5. Section header
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.fromLTRB(16, 8, 16, 8),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  const Text(
                    'สินค้ายอดนิยม',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  TextButton(onPressed: () {}, child: const Text('ดูทั้งหมด')),
                ],
              ),
            ),
          ),
          
          // 6. Horizontal list ด้วย SliverToBoxAdapter
          SliverToBoxAdapter(
            child: SizedBox(
              height: 180,
              child: ListView.builder(
                scrollDirection: Axis.horizontal,
                padding: const EdgeInsets.symmetric(horizontal: 8),
                itemCount: 10,
                itemBuilder: (context, index) {
                  return Container(
                    width: 130,
                    margin: const EdgeInsets.symmetric(horizontal: 4),
                    child: Card(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Expanded(
                            child: Container(color: Colors.grey[200]),
                          ),
                          Padding(
                            padding: const EdgeInsets.all(8),
                            child: Column(
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                Text('สินค้า ${index + 1}'),
                                Text(
                                  '฿${(index + 1) * 199}',
                                  style: const TextStyle(
                                    color: Colors.red,
                                    fontWeight: FontWeight.bold,
                                  ),
                                ),
                              ],
                            ),
                          ),
                        ],
                      ),
                    ),
                  );
                },
              ),
            ),
          ),
          
          // 7. Section header
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'สินค้าทั้งหมด',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
            ),
          ),
          
          // 8. SliverList สำหรับ all products
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => ListTile(
                leading: Container(
                  width: 56,
                  height: 56,
                  color: Colors.grey[200],
                  child: const Icon(Icons.image),
                ),
                title: Text('สินค้า ${index + 1}'),
                subtitle: Text('หมวดหมู่: ${['อิเล็กทรอนิกส์', 'เสื้อผ้า', 'อาหาร'][index % 3]}'),
                trailing: Text('฿${(index + 1) * 149}'),
              ),
              childCount: 20,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 109: NestedScrollView

`NestedScrollView` ใช้สำหรับ layout ที่มี header scroll ร่วมกับ TabBarView

```dart
import 'package:flutter/material.dart';

class NestedScrollViewExample extends StatelessWidget {
  const NestedScrollViewExample({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        body: NestedScrollView(
          headerSliverBuilder: (context, innerBoxIsScrolled) {
            return [
              SliverAppBar(
                expandedHeight: 200,
                floating: true,
                pinned: true,
                snap: true,
                // innerBoxIsScrolled บอกว่า inner scroll view ถูก scroll หรือยัง
                forceElevated: innerBoxIsScrolled,
                flexibleSpace: FlexibleSpaceBar(
                  title: const Text('NestedScrollView'),
                  background: Container(
                    decoration: const BoxDecoration(
                      gradient: LinearGradient(
                        colors: [Colors.teal, Colors.cyan],
                      ),
                    ),
                  ),
                ),
                bottom: const TabBar(
                  tabs: [
                    Tab(text: 'ทั้งหมด', icon: Icon(Icons.apps)),
                    Tab(text: 'ยอดนิยม', icon: Icon(Icons.trending_up)),
                    Tab(text: 'ใหม่ล่าสุด', icon: Icon(Icons.new_releases)),
                  ],
                ),
              ),
            ];
          },
          body: TabBarView(
            children: [
              // Tab 1: ทั้งหมด
              ListView.builder(
                itemCount: 30,
                itemBuilder: (context, index) => ListTile(
                  title: Text('รายการ ${index + 1}'),
                  leading: const Icon(Icons.article),
                ),
              ),
              // Tab 2: ยอดนิยม
              GridView.builder(
                gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: 2,
                  crossAxisSpacing: 8,
                  mainAxisSpacing: 8,
                ),
                padding: const EdgeInsets.all(8),
                itemCount: 20,
                itemBuilder: (context, index) => Card(
                  child: Center(child: Text('ยอดนิยม ${index + 1}')),
                ),
              ),
              // Tab 3: ใหม่ล่าสุด
              ListView.builder(
                itemCount: 15,
                itemBuilder: (context, index) => Card(
                  margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
                  child: ListTile(
                    title: Text('ใหม่ล่าสุด ${index + 1}'),
                    trailing: const Chip(label: Text('ใหม่')),
                  ),
                ),
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

## ขั้นตอนที่ 110: Sticky Headers

Sticky Headers คือ header ที่ติดอยู่ด้านบนเมื่อ scroll ข้ามหมวดหมู่ต่างๆ

```dart
import 'package:flutter/material.dart';

// ข้อมูลสำหรับ contacts แบบมี section
class ContactsWithStickyHeader extends StatelessWidget {
  const ContactsWithStickyHeader({super.key});

  // จัดกลุ่ม contacts ตามตัวอักษรแรก
  Map<String, List<String>> get _groupedContacts {
    final contacts = [
      'อนุชา สุขใจ', 'อรุณ พรมมา', 'อิทธิพล วงศ์ษา',
      'กนกวรรณ ชาติ', 'กมล พันธ์ดี', 'กิตติ อยู่ดี',
      'ขนิษฐา ไทยใจ', 'ขจร สุขสม',
      'จันทร์เพ็ญ แสงทอง', 'จิรา วงค์แก้ว',
      'ชลิตา ดีมาก', 'ชัย มีสุข',
      'ณัฐพล ใจกว้าง', 'ณัฐวุฒิ สดใส',
      'ทักษิณ ชัยชนะ', 'ทัศนา ศรีสุข',
      'ธนา วรรณา', 'ธิดา สวัสดิ์',
      'นนท์ ศรีทอง', 'นภา สุดาวรรณ',
    ];

    final Map<String, List<String>> grouped = {};
    for (final contact in contacts) {
      final key = contact[0];
      grouped.putIfAbsent(key, () => []).add(contact);
    }

    // เรียงตามตัวอักษร
    return Map.fromEntries(
      grouped.entries.toList()..sort((a, b) => a.key.compareTo(b.key)),
    );
  }

  @override
  Widget build(BuildContext context) {
    final grouped = _groupedContacts;
    final sections = grouped.keys.toList();

    return Scaffold(
      appBar: AppBar(
        title: const Text('รายชื่อติดต่อ'),
        actions: [
          IconButton(icon: const Icon(Icons.search), onPressed: () {}),
        ],
      ),
      body: CustomScrollView(
        slivers: [
          for (final section in sections) ...[
            // Sticky Header
            SliverPersistentHeader(
              pinned: true,
              delegate: _SectionHeaderDelegate(
                title: section,
                minHeight: 36,
                maxHeight: 36,
              ),
            ),
            // Items ในแต่ละ section
            SliverList(
              delegate: SliverChildBuilderDelegate(
                (context, index) {
                  final contact = grouped[section]![index];
                  return ListTile(
                    leading: CircleAvatar(
                      backgroundColor: _getColor(section),
                      child: Text(
                        contact[0],
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                    title: Text(contact),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        IconButton(
                          icon: const Icon(Icons.call, color: Colors.green),
                          onPressed: () {},
                        ),
                        IconButton(
                          icon: const Icon(Icons.message, color: Colors.blue),
                          onPressed: () {},
                        ),
                      ],
                    ),
                  );
                },
                childCount: grouped[section]!.length,
              ),
            ),
          ],
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: const Icon(Icons.person_add),
      ),
    );
  }

  Color _getColor(String letter) {
    final colors = [
      Colors.red, Colors.pink, Colors.purple, Colors.deepPurple,
      Colors.indigo, Colors.blue, Colors.lightBlue, Colors.cyan,
      Colors.teal, Colors.green, Colors.lightGreen, Colors.lime,
      Colors.yellow, Colors.amber, Colors.orange, Colors.deepOrange,
    ];
    return colors[letter.codeUnitAt(0) % colors.length];
  }
}

// Custom SliverPersistentHeaderDelegate สำหรับ Sticky Header
class _SectionHeaderDelegate extends SliverPersistentHeaderDelegate {
  final String title;
  final double minHeight;
  final double maxHeight;

  _SectionHeaderDelegate({
    required this.title,
    required this.minHeight,
    required this.maxHeight,
  });

  @override
  double get minExtent => minHeight;

  @override
  double get maxExtent => maxHeight;

  @override
  Widget build(
    BuildContext context,
    double shrinkOffset,
    bool overlapsContent,
  ) {
    return Container(
      color: Colors.grey[100],
      alignment: Alignment.centerLeft,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Text(
        title,
        style: TextStyle(
          fontWeight: FontWeight.bold,
          color: Theme.of(context).colorScheme.primary,
          fontSize: 14,
        ),
      ),
    );
  }

  @override
  bool shouldRebuild(covariant _SectionHeaderDelegate oldDelegate) {
    return oldDelegate.title != title ||
        oldDelegate.minHeight != minHeight ||
        oldDelegate.maxHeight != maxHeight;
  }
}
```

---

## Workshop: News Feed App

มาสร้าง News Feed App ที่ใช้ทุกอย่างที่เรียนมา

```dart
import 'package:flutter/material.dart';

// Models
class NewsArticle {
  final int id;
  final String title;
  final String description;
  final String category;
  final String author;
  final String imageUrl;
  final DateTime publishedAt;
  bool isBookmarked;

  NewsArticle({
    required this.id,
    required this.title,
    required this.description,
    required this.category,
    required this.author,
    required this.imageUrl,
    required this.publishedAt,
    this.isBookmarked = false,
  });
}

// Data source
class NewsRepository {
  static final List<String> categories = [
    'ทั้งหมด', 'เทคโนโลยี', 'กีฬา', 'การเมือง', 'บันเทิง', 'สุขภาพ'
  ];

  static List<NewsArticle> generateArticles(int count, {String? category}) {
    return List.generate(count, (index) {
      final cats = ['เทคโนโลยี', 'กีฬา', 'การเมือง', 'บันเทิง', 'สุขภาพ'];
      final cat = category ?? cats[index % cats.length];
      return NewsArticle(
        id: index + 1,
        title: 'ข่าว$cat ประจำวันที่ ${index + 1}: เหตุการณ์สำคัญที่น่าติดตาม',
        description: 'รายละเอียดข่าว$cat ที่ครอบคลุมและน่าสนใจ ครบถ้วนทุกแง่มุม...',
        category: cat,
        author: ['สมชาย', 'สมหญิง', 'อนงค์', 'อาคม', 'บุญรอด'][index % 5],
        imageUrl: 'https://picsum.photos/400/250?random=${index + 10}',
        publishedAt: DateTime.now().subtract(Duration(hours: index * 2)),
      );
    });
  }
}

// Main Screen
class NewsFeedApp extends StatelessWidget {
  const NewsFeedApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'News Feed',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const NewsFeedScreen(),
    );
  }
}

class NewsFeedScreen extends StatefulWidget {
  const NewsFeedScreen({super.key});

  @override
  State<NewsFeedScreen> createState() => _NewsFeedScreenState();
}

class _NewsFeedScreenState extends State<NewsFeedScreen>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;
  final Map<String, List<NewsArticle>> _articlesByCategory = {};
  final Map<String, bool> _isLoadingMore = {};
  final Map<String, bool> _hasMore = {};
  final Map<String, ScrollController> _scrollControllers = {};
  bool _isRefreshing = false;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(
      length: NewsRepository.categories.length,
      vsync: this,
    );

    // Initialize data for each category
    for (final cat in NewsRepository.categories) {
      _articlesByCategory[cat] = NewsRepository.generateArticles(
        15,
        category: cat == 'ทั้งหมด' ? null : cat,
      );
      _isLoadingMore[cat] = false;
      _hasMore[cat] = true;
      _scrollControllers[cat] = ScrollController()
        ..addListener(() => _onScroll(cat));
    }
  }

  @override
  void dispose() {
    _tabController.dispose();
    for (final controller in _scrollControllers.values) {
      controller.dispose();
    }
    super.dispose();
  }

  void _onScroll(String category) {
    final controller = _scrollControllers[category]!;
    if (controller.position.pixels >= controller.position.maxScrollExtent - 300) {
      _loadMore(category);
    }
  }

  Future<void> _loadMore(String category) async {
    if (_isLoadingMore[category]! || !_hasMore[category]!) return;

    setState(() => _isLoadingMore[category] = true);
    await Future.delayed(const Duration(seconds: 1));

    final current = _articlesByCategory[category]!;
    if (current.length >= 60) {
      setState(() {
        _hasMore[category] = false;
        _isLoadingMore[category] = false;
      });
      return;
    }

    final newArticles = NewsRepository.generateArticles(
      10,
      category: category == 'ทั้งหมด' ? null : category,
    );

    setState(() {
      _articlesByCategory[category]!.addAll(newArticles);
      _isLoadingMore[category] = false;
    });
  }

  Future<void> _refresh() async {
    setState(() => _isRefreshing = true);
    await Future.delayed(const Duration(seconds: 2));

    setState(() {
      for (final cat in NewsRepository.categories) {
        _articlesByCategory[cat] = NewsRepository.generateArticles(
          15,
          category: cat == 'ทั้งหมด' ? null : cat,
        );
        _hasMore[cat] = true;
      }
      _isRefreshing = false;
    });

    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('อัปเดตข่าวล่าสุดแล้ว!')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: NestedScrollView(
        headerSliverBuilder: (context, innerBoxIsScrolled) => [
          SliverAppBar(
            expandedHeight: 100,
            floating: true,
            pinned: true,
            snap: true,
            forceElevated: innerBoxIsScrolled,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('📰 ข่าวสด'),
              background: Container(
                decoration: const BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Color(0xFF1565C0), Color(0xFF42A5F5)],
                  ),
                ),
              ),
            ),
            actions: [
              IconButton(icon: const Icon(Icons.search), onPressed: () {}),
              IconButton(icon: const Icon(Icons.notifications), onPressed: () {}),
              IconButton(icon: const Icon(Icons.bookmark), onPressed: () {}),
            ],
            bottom: TabBar(
              controller: _tabController,
              isScrollable: true,
              tabs: NewsRepository.categories
                  .map((cat) => Tab(text: cat))
                  .toList(),
            ),
          ),
        ],
        body: TabBarView(
          controller: _tabController,
          children: NewsRepository.categories
              .map((cat) => _buildNewsList(cat))
              .toList(),
        ),
      ),
    );
  }

  Widget _buildNewsList(String category) {
    final articles = _articlesByCategory[category] ?? [];

    return RefreshIndicator(
      onRefresh: _refresh,
      child: CustomScrollView(
        controller: _scrollControllers[category],
        slivers: [
          // Breaking news banner
          if (category == 'ทั้งหมด')
            SliverToBoxAdapter(child: _buildBreakingNewsBanner()),

          // Featured article
          if (articles.isNotEmpty)
            SliverToBoxAdapter(
              child: _buildFeaturedArticle(articles.first),
            ),

          // Article list
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                if (index >= articles.length) {
                  return _buildLoadingTile(category);
                }
                return _buildArticleTile(articles[index]);
              },
              childCount: articles.length + (_hasMore[category]! ? 1 : 0),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildBreakingNewsBanner() {
    return Container(
      height: 50,
      color: Colors.red[700],
      child: Row(
        children: [
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 12),
            color: Colors.red[900],
            alignment: Alignment.center,
            child: const Text(
              'ด่วน!',
              style: TextStyle(
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          Expanded(
            child: SingleChildScrollView(
              scrollDirection: Axis.horizontal,
              child: Padding(
                padding: const EdgeInsets.symmetric(horizontal: 16),
                child: Text(
                  '🔴 ข่าวด่วน: เหตุการณ์สำคัญที่ทุกคนต้องติดตาม...',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildFeaturedArticle(NewsArticle article) {
    return Container(
      height: 220,
      margin: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(16),
        color: Colors.grey[200],
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: 8,
            offset: const Offset(0, 4),
          ),
        ],
      ),
      child: ClipRRect(
        borderRadius: BorderRadius.circular(16),
        child: Stack(
          fit: StackFit.expand,
          children: [
            Image.network(
              article.imageUrl,
              fit: BoxFit.cover,
              errorBuilder: (_, __, ___) => Container(color: Colors.blue[100]),
            ),
            Container(
              decoration: BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [Colors.transparent, Colors.black.withOpacity(0.8)],
                ),
              ),
            ),
            Positioned(
              bottom: 16,
              left: 16,
              right: 16,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisSize: MainAxisSize.min,
                children: [
                  Container(
                    padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                    decoration: BoxDecoration(
                      color: Colors.blue,
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: Text(
                      article.category,
                      style: const TextStyle(color: Colors.white, fontSize: 12),
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(
                    article.title,
                    style: const TextStyle(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                      fontSize: 16,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildArticleTile(NewsArticle article) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      child: InkWell(
        borderRadius: BorderRadius.circular(12),
        onTap: () {},
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Thumbnail
              ClipRRect(
                borderRadius: BorderRadius.circular(8),
                child: SizedBox(
                  width: 90,
                  height: 70,
                  child: Image.network(
                    article.imageUrl,
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) =>
                        Container(color: Colors.grey[200]),
                  ),
                ),
              ),
              const SizedBox(width: 12),
              // Content
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Row(
                      children: [
                        Container(
                          padding: const EdgeInsets.symmetric(
                              horizontal: 6, vertical: 2),
                          decoration: BoxDecoration(
                            color: Colors.blue[50],
                            borderRadius: BorderRadius.circular(4),
                          ),
                          child: Text(
                            article.category,
                            style: TextStyle(
                              color: Colors.blue[700],
                              fontSize: 10,
                            ),
                          ),
                        ),
                        const Spacer(),
                        IconButton(
                          icon: Icon(
                            article.isBookmarked
                                ? Icons.bookmark
                                : Icons.bookmark_border,
                            size: 18,
                            color: article.isBookmarked ? Colors.blue : null,
                          ),
                          constraints: const BoxConstraints(),
                          padding: EdgeInsets.zero,
                          onPressed: () {
                            setState(() {
                              article.isBookmarked = !article.isBookmarked;
                            });
                          },
                        ),
                      ],
                    ),
                    const SizedBox(height: 4),
                    Text(
                      article.title,
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '${article.author} • '
                      '${_formatTime(article.publishedAt)}',
                      style: TextStyle(
                        color: Colors.grey[600],
                        fontSize: 12,
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildLoadingTile(String category) {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Center(
        child: _isLoadingMore[category]!
            ? const CircularProgressIndicator()
            : const Text('โหลดข้อมูลทั้งหมดแล้ว'),
      ),
    );
  }

  String _formatTime(DateTime dateTime) {
    final diff = DateTime.now().difference(dateTime);
    if (diff.inMinutes < 60) return '${diff.inMinutes} นาทีที่แล้ว';
    if (diff.inHours < 24) return '${diff.inHours} ชั่วโมงที่แล้ว';
    return '${diff.inDays} วันที่แล้ว';
  }
}

void main() => runApp(const NewsFeedApp());
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้:

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| ListView.builder | การสร้าง List อย่างมีประสิทธิภาพด้วย Lazy Loading |
| RefreshIndicator | Pull-to-refresh pattern |
| Infinite Scroll | การโหลดข้อมูลเพิ่มเมื่อ scroll ถึงด้านล่าง |
| GridView.builder | การแสดงข้อมูลแบบตาราง |
| SliverList/Grid | การใช้ Sliver ใน CustomScrollView |
| SliverAppBar | AppBar ที่ขยาย/ย่อตัวได้ |
| CustomScrollView | การรวม Sliver ต่างๆ |
| NestedScrollView | Scroll ซ้อนกับ TabBarView |
| Sticky Headers | Header ที่ติดอยู่ขณะ scroll |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง ListView.separated ที่แสดงรายการ To-Do พร้อม checkbox
2. **ปานกลาง**: สร้าง GridView gallery ที่เลือกหลายรูปได้พร้อมปุ่ม share
3. **ยาก**: สร้าง CustomScrollView ที่มี SliverAppBar, SliverGrid สำหรับ featured และ SliverList สำหรับรายการทั้งหมด

---

[← Part 10: Navigation & Routing](part_10.md) | [Part 12: Animations →](part_12.md)

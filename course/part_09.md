# Part 09: Navigation & Routing พื้นฐาน
## ขั้นตอนที่ 81-90

---

## สารบัญ
1. [Navigator 1.0 พื้นฐาน](#ขั้นตอนที่-81-navigator-10-พื้นฐาน)
2. [Named Routes](#ขั้นตอนที่-82-named-routes)
3. [Route Generation (onGenerateRoute)](#ขั้นตอนที่-83-route-generation)
4. [ส่งข้อมูลระหว่างหน้า](#ขั้นตอนที่-84-ส่งข้อมูลระหว่างหน้า)
5. [MaterialPageRoute & CupertinoPageRoute](#ขั้นตอนที่-85-materialpageroute--cupertinopageroute)
6. [Custom Page Transitions](#ขั้นตอนที่-86-custom-page-transitions)
7. [WillPopScope & PopScope](#ขั้นตอนที่-87-willpopscope--popscope)
8. [Bottom Navigation & Tab Navigation](#ขั้นตอนที่-88-bottom-navigation--tab-navigation)
9. [Navigator 2.0 เบื้องต้น](#ขั้นตอนที่-89-navigator-20-เบื้องต้น)
10. [Workshop: Multi-screen App](#ขั้นตอนที่-90-workshop-multi-screen-app)

---

## ขั้นตอนที่ 81: Navigator 1.0 พื้นฐาน

Navigator จัดการ stack ของ screens

```dart
import 'package:flutter/material.dart';

void main() => runApp(const NavigationApp());

class NavigationApp extends StatelessWidget {
  const NavigationApp({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: ThemeData(useMaterial3: true),
        home: const HomeScreen(),
      );
}

// ===== Home Screen =====
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // push - ไปหน้าใหม่
            ElevatedButton.icon(
              icon: const Icon(Icons.arrow_forward),
              label: const Text('push → Detail'),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const DetailScreen(),
                  ),
                );
              },
            ),

            const SizedBox(height: 12),

            // push และรับค่ากลับ
            ElevatedButton.icon(
              icon: const Icon(Icons.arrow_forward),
              label: const Text('push → Result Screen'),
              onPressed: () async {
                // รอรับค่ากลับด้วย await
                final result = await Navigator.push<String>(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ResultScreen(),
                  ),
                );

                if (result != null && context.mounted) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('ได้รับค่า: $result')),
                  );
                }
              },
            ),

            const SizedBox(height: 12),

            // pushReplacement - แทนที่หน้าปัจจุบัน (ไม่มี back)
            ElevatedButton.icon(
              icon: const Icon(Icons.swap_horiz),
              label: const Text('pushReplacement → Login'),
              onPressed: () {
                Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const LoginScreen(),
                  ),
                );
              },
            ),

            const SizedBox(height: 12),

            // pushAndRemoveUntil - ล้าง stack ทั้งหมด
            ElevatedButton.icon(
              icon: const Icon(Icons.refresh),
              label: const Text('pushAndRemoveUntil'),
              onPressed: () {
                Navigator.pushAndRemoveUntil(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HomeScreen(),
                  ),
                  (route) => false, // ลบทุก route ใน stack
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

// ===== Detail Screen =====
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Detail')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Detail Screen'),
            const SizedBox(height: 16),

            // pop - กลับหน้าก่อน
            ElevatedButton.icon(
              icon: const Icon(Icons.arrow_back),
              label: const Text('pop'),
              onPressed: () => Navigator.pop(context),
            ),

            const SizedBox(height: 8),

            // canPop - ตรวจสอบก่อน pop
            Text(
              'canPop: ${Navigator.canPop(context)}',
              style: const TextStyle(color: Colors.grey),
            ),
          ],
        ),
      ),
    );
  }
}

// ===== Result Screen =====
class ResultScreen extends StatelessWidget {
  const ResultScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('เลือกตัวเลือก')),
      body: ListView(
        children: ['ตัวเลือก A', 'ตัวเลือก B', 'ตัวเลือก C'].map((option) {
          return ListTile(
            title: Text(option),
            onTap: () => Navigator.pop(context, option), // ส่งค่ากลับ
          );
        }).toList(),
      ),
    );
  }
}

// ===== Login Screen =====
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('กลับ'),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 82: Named Routes

Named Routes ใช้ชื่อแทน route ทำให้จัดการง่าย

```dart
import 'package:flutter/material.dart';

// กำหนด route names เป็น constants
abstract class AppRoutes {
  static const home = '/';
  static const login = '/login';
  static const register = '/register';
  static const profile = '/profile';
  static const settings = '/settings';
  static const productList = '/products';
  static const productDetail = '/products/detail';
}

class NamedRoutesApp extends StatelessWidget {
  const NamedRoutesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      initialRoute: AppRoutes.home,
      routes: {
        AppRoutes.home: (context) => const HomeNamedScreen(),
        AppRoutes.login: (context) => const LoginNamedScreen(),
        AppRoutes.register: (context) => const RegisterScreen(),
        AppRoutes.profile: (context) => const ProfileScreen(),
        AppRoutes.settings: (context) => const SettingsScreen(),
        AppRoutes.productList: (context) => const ProductListScreen(),
        // ไม่รองรับ arguments ใน routes map โดยตรง
        // ใช้ onGenerateRoute แทน
      },
    );
  }
}

class HomeNamedScreen extends StatelessWidget {
  const HomeNamedScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Named Routes')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          ListTile(
            leading: const Icon(Icons.login),
            title: const Text('ไปหน้า Login'),
            onTap: () =>
                Navigator.pushNamed(context, AppRoutes.login),
          ),

          ListTile(
            leading: const Icon(Icons.person_add),
            title: const Text('ไปหน้า Register'),
            onTap: () =>
                Navigator.pushNamed(context, AppRoutes.register),
          ),

          ListTile(
            leading: const Icon(Icons.person),
            title: const Text('ไปหน้า Profile'),
            onTap: () =>
                Navigator.pushNamed(context, AppRoutes.profile),
          ),

          ListTile(
            leading: const Icon(Icons.settings),
            title: const Text('ไปหน้า Settings'),
            onTap: () =>
                Navigator.pushNamed(context, AppRoutes.settings),
          ),

          ListTile(
            leading: const Icon(Icons.shopping_bag),
            title: const Text('ไปหน้า Products'),
            onTap: () =>
                Navigator.pushNamed(context, AppRoutes.productList),
          ),

          // pushReplacementNamed
          ListTile(
            leading: const Icon(Icons.swap_horiz),
            title: const Text('pushReplacementNamed'),
            onTap: () =>
                Navigator.pushReplacementNamed(context, AppRoutes.login),
          ),

          // popAndPushNamed
          ListTile(
            leading: const Icon(Icons.refresh),
            title: const Text('popAndPushNamed'),
            onTap: () =>
                Navigator.popAndPushNamed(context, AppRoutes.home),
          ),
        ],
      ),
    );
  }
}

class LoginNamedScreen extends StatelessWidget {
  const LoginNamedScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Login')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text('Login Screen'),
              ElevatedButton(
                onPressed: () => Navigator.pushReplacementNamed(
                    context, AppRoutes.home),
                child: const Text('Login สำเร็จ → Home'),
              ),
            ],
          ),
        ),
      );
}

class RegisterScreen extends StatelessWidget {
  const RegisterScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Register')),
        body: const Center(child: Text('Register Screen')),
      );
}

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Profile')),
        body: const Center(child: Text('Profile Screen')),
      );
}

class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Settings')),
        body: const Center(child: Text('Settings Screen')),
      );
}

class ProductListScreen extends StatelessWidget {
  const ProductListScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Products')),
        body: const Center(child: Text('Products Screen')),
      );
}
```

---

## ขั้นตอนที่ 83: Route Generation

`onGenerateRoute` จัดการ routes แบบ dynamic พร้อมรับ arguments

```dart
import 'package:flutter/material.dart';

// Route arguments classes
class ProductDetailArgs {
  final int productId;
  final String productName;
  const ProductDetailArgs({
    required this.productId,
    required this.productName,
  });
}

class UserProfileArgs {
  final String userId;
  final bool isOwner;
  const UserProfileArgs({required this.userId, this.isOwner = false});
}

class GeneratedRoutesApp extends StatelessWidget {
  const GeneratedRoutesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      initialRoute: '/',
      onGenerateRoute: _generateRoute,
      onUnknownRoute: (settings) {
        // Fallback สำหรับ route ที่ไม่รู้จัก
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
    );
  }

  Route<dynamic>? _generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case '/':
        return MaterialPageRoute(
          settings: settings,
          builder: (_) => const MainScreen(),
        );

      case '/product-detail':
        final args = settings.arguments as ProductDetailArgs?;
        if (args == null) {
          return MaterialPageRoute(
            builder: (_) => const ErrorScreen(
                message: 'ต้องส่ง ProductDetailArgs'),
          );
        }
        return MaterialPageRoute(
          settings: settings,
          builder: (_) => ProductDetailScreen(args: args),
        );

      case '/user-profile':
        final args = settings.arguments as UserProfileArgs?;
        return MaterialPageRoute(
          settings: settings,
          builder: (_) =>
              UserProfileScreen(args: args ?? const UserProfileArgs(userId: '')),
        );

      // Dynamic route - /post/123
      default:
        final name = settings.name ?? '';
        if (name.startsWith('/post/')) {
          final postId = name.replaceFirst('/post/', '');
          return MaterialPageRoute(
            builder: (_) => PostScreen(postId: postId),
          );
        }
        return null;
    }
  }
}

class MainScreen extends StatelessWidget {
  const MainScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('onGenerateRoute')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // ส่ง args ผ่าน named route
          ElevatedButton(
            onPressed: () {
              Navigator.pushNamed(
                context,
                '/product-detail',
                arguments: const ProductDetailArgs(
                  productId: 42,
                  productName: 'iPhone 16 Pro',
                ),
              );
            },
            child: const Text('Product Detail (with args)'),
          ),

          const SizedBox(height: 8),

          ElevatedButton(
            onPressed: () {
              Navigator.pushNamed(
                context,
                '/user-profile',
                arguments: const UserProfileArgs(
                  userId: 'user123',
                  isOwner: true,
                ),
              );
            },
            child: const Text('User Profile (with args)'),
          ),

          const SizedBox(height: 8),

          // Dynamic route
          ElevatedButton(
            onPressed: () {
              Navigator.pushNamed(context, '/post/abc-456');
            },
            child: const Text('Dynamic Route (/post/abc-456)'),
          ),

          const SizedBox(height: 8),

          // Unknown route
          ElevatedButton(
            onPressed: () {
              Navigator.pushNamed(context, '/unknown-page');
            },
            child: const Text('Unknown Route (404)'),
          ),
        ],
      ),
    );
  }
}

class ProductDetailScreen extends StatelessWidget {
  final ProductDetailArgs args;
  const ProductDetailScreen({super.key, required this.args});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text(args.productName)),
        body: Center(
          child: Text('Product ID: ${args.productId}\nชื่อ: ${args.productName}'),
        ),
      );
}

class UserProfileScreen extends StatelessWidget {
  final UserProfileArgs args;
  const UserProfileScreen({super.key, required this.args});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('User: ${args.userId}')),
        body: Center(
          child: Text(
              'userId: ${args.userId}\nisOwner: ${args.isOwner}'),
        ),
      );
}

class PostScreen extends StatelessWidget {
  final String postId;
  const PostScreen({super.key, required this.postId});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('Post: $postId')),
        body: Center(child: Text('Post ID: $postId')),
      );
}

class NotFoundScreen extends StatelessWidget {
  const NotFoundScreen({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('404')),
        body: const Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.error_outline, size: 64, color: Colors.red),
              SizedBox(height: 16),
              Text('ไม่พบหน้าที่ต้องการ', style: TextStyle(fontSize: 20)),
            ],
          ),
        ),
      );
}

class ErrorScreen extends StatelessWidget {
  final String message;
  const ErrorScreen({super.key, required this.message});
  @override
  Widget build(BuildContext context) => Scaffold(
        body: Center(child: Text('Error: $message')),
      );
}
```

---

## ขั้นตอนที่ 84: ส่งข้อมูลระหว่างหน้า

วิธีส่งข้อมูลระหว่าง screens

```dart
import 'package:flutter/material.dart';

// Model
class Product {
  final int id;
  final String name;
  final double price;
  final String image;
  bool isFavorite;

  Product({
    required this.id,
    required this.name,
    required this.price,
    required this.image,
    this.isFavorite = false,
  });
}

// ===== ส่งข้อมูลไป =====
class ProductListPage extends StatefulWidget {
  const ProductListPage({super.key});

  @override
  State<ProductListPage> createState() => _ProductListPageState();
}

class _ProductListPageState extends State<ProductListPage> {
  final List<Product> _products = [
    Product(id: 1, name: 'MacBook Pro', price: 79900, image: '💻'),
    Product(id: 2, name: 'iPhone 16', price: 35900, image: '📱'),
    Product(id: 3, name: 'iPad Air', price: 23900, image: '📲'),
    Product(id: 4, name: 'AirPods Pro', price: 9900, image: '🎧'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('สินค้า')),
      body: ListView.builder(
        itemCount: _products.length,
        itemBuilder: (context, index) {
          final product = _products[index];
          return ListTile(
            leading: Text(product.image, style: const TextStyle(fontSize: 32)),
            title: Text(product.name),
            subtitle: Text('฿${product.price.toStringAsFixed(0)}'),
            trailing: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                IconButton(
                  icon: Icon(
                    product.isFavorite
                        ? Icons.favorite
                        : Icons.favorite_border,
                    color: product.isFavorite ? Colors.red : null,
                  ),
                  onPressed: () {
                    setState(() => product.isFavorite = !product.isFavorite);
                  },
                ),
                const Icon(Icons.arrow_forward_ios, size: 14),
              ],
            ),
            onTap: () async {
              // ส่ง object ไปพร้อมกัน และรับค่ากลับ
              final updatedProduct =
                  await Navigator.push<Product>(
                context,
                MaterialPageRoute(
                  builder: (context) =>
                      ProductDetailPage(product: product),
                ),
              );

              // อัปเดตถ้ามีการเปลี่ยนแปลง
              if (updatedProduct != null) {
                setState(() {
                  _products[index] = updatedProduct;
                });
              }
            },
          );
        },
      ),
    );
  }
}

// ===== รับข้อมูลมา / ส่งกลับ =====
class ProductDetailPage extends StatefulWidget {
  final Product product;
  const ProductDetailPage({super.key, required this.product});

  @override
  State<ProductDetailPage> createState() => _ProductDetailPageState();
}

class _ProductDetailPageState extends State<ProductDetailPage> {
  late Product _product;

  @override
  void initState() {
    super.initState();
    // copy product เพื่อไม่ mutate ของเดิม
    _product = Product(
      id: widget.product.id,
      name: widget.product.name,
      price: widget.product.price,
      image: widget.product.image,
      isFavorite: widget.product.isFavorite,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_product.name),
        actions: [
          IconButton(
            icon: Icon(
              _product.isFavorite ? Icons.favorite : Icons.favorite_border,
              color: _product.isFavorite ? Colors.red : null,
            ),
            onPressed: () {
              setState(() => _product.isFavorite = !_product.isFavorite);
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(_product.image, style: const TextStyle(fontSize: 96)),
            const SizedBox(height: 16),
            Text(_product.name,
                style: const TextStyle(
                    fontSize: 24, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            Text(
              '฿${_product.price.toStringAsFixed(0)}',
              style: const TextStyle(
                  fontSize: 18, color: Colors.green),
            ),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              icon: const Icon(Icons.shopping_cart),
              label: const Text('เพิ่มลงตะกร้า'),
              onPressed: () {
                // ส่ง updated product กลับ
                Navigator.pop(context, _product);
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

## ขั้นตอนที่ 85: MaterialPageRoute & CupertinoPageRoute

```dart
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

class RouteTypesDemo extends StatelessWidget {
  const RouteTypesDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route Types')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // 1. MaterialPageRoute (Android style - slide from right)
          ListTile(
            leading: const Icon(Icons.android, color: Colors.green),
            title: const Text('MaterialPageRoute'),
            subtitle: const Text('Slide จากขวา (Android)'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => const SamplePage(title: 'Material'),
                  fullscreenDialog: false,  // ถ้า true จะเป็น modal
                ),
              );
            },
          ),

          // 2. MaterialPageRoute fullscreenDialog (slide from bottom)
          ListTile(
            leading: const Icon(Icons.open_in_full),
            title: const Text('MaterialPageRoute (Dialog)'),
            subtitle: const Text('Slide จากล่าง (modal)'),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) =>
                      const SamplePage(title: 'Material Dialog'),
                  fullscreenDialog: true,
                ),
              );
            },
          ),

          // 3. CupertinoPageRoute (iOS style)
          ListTile(
            leading: const Icon(Icons.apple, color: Colors.grey),
            title: const Text('CupertinoPageRoute'),
            subtitle: const Text('iOS style slide'),
            onTap: () {
              Navigator.push(
                context,
                CupertinoPageRoute(
                  builder: (_) => const SamplePage(title: 'Cupertino'),
                ),
              );
            },
          ),

          // 4. CupertinoModalPopupRoute
          ListTile(
            leading: const Icon(Icons.vertical_align_top),
            title: const Text('CupertinoModalPopup'),
            subtitle: const Text('Sheet จากล่าง (iOS)'),
            onTap: () {
              showCupertinoModalPopup(
                context: context,
                builder: (_) => CupertinoActionSheet(
                  title: const Text('เลือกตัวเลือก'),
                  actions: [
                    CupertinoActionSheetAction(
                      onPressed: () => Navigator.pop(context),
                      child: const Text('ตัวเลือก 1'),
                    ),
                    CupertinoActionSheetAction(
                      onPressed: () => Navigator.pop(context),
                      child: const Text('ตัวเลือก 2'),
                    ),
                  ],
                  cancelButton: CupertinoActionSheetAction(
                    isDefaultAction: true,
                    onPressed: () => Navigator.pop(context),
                    child: const Text('ยกเลิก'),
                  ),
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}

class SamplePage extends StatelessWidget {
  final String title;
  const SamplePage({super.key, required this.title});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text(title)),
        body: Center(
          child: ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('กลับ'),
          ),
        ),
      );
}
```

---

## ขั้นตอนที่ 86: Custom Page Transitions

PageRouteBuilder สร้าง transition เอง

```dart
import 'package:flutter/material.dart';

// Custom transition helper
class CustomRoutes {
  // Fade transition
  static Route<T> fade<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, _) => page,
      transitionDuration: const Duration(milliseconds: 400),
      transitionsBuilder: (context, animation, _, child) {
        return FadeTransition(opacity: animation, child: child);
      },
    );
  }

  // Scale transition
  static Route<T> scale<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, _) => page,
      transitionDuration: const Duration(milliseconds: 300),
      transitionsBuilder: (context, animation, _, child) {
        final curvedAnim =
            CurvedAnimation(parent: animation, curve: Curves.elasticOut);
        return ScaleTransition(scale: curvedAnim, child: child);
      },
    );
  }

  // Slide from bottom
  static Route<T> slideFromBottom<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, _) => page,
      transitionDuration: const Duration(milliseconds: 350),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        const begin = Offset(0.0, 1.0);
        const end = Offset.zero;
        final tween = Tween(begin: begin, end: end)
            .chain(CurveTween(curve: Curves.easeOutCubic));
        return SlideTransition(
          position: animation.drive(tween),
          child: child,
        );
      },
    );
  }

  // Slide from right
  static Route<T> slideFromRight<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, _) => page,
      transitionDuration: const Duration(milliseconds: 300),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        final slideIn = Tween(
          begin: const Offset(1.0, 0.0),
          end: Offset.zero,
        ).animate(CurvedAnimation(parent: animation, curve: Curves.easeOut));

        final slideOut = Tween(
          begin: Offset.zero,
          end: const Offset(-0.3, 0.0),
        ).animate(CurvedAnimation(
            parent: secondaryAnimation, curve: Curves.easeIn));

        return SlideTransition(
          position: slideIn,
          child: SlideTransition(position: slideOut, child: child),
        );
      },
    );
  }

  // Rotation + Fade
  static Route<T> rotateFade<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (context, animation, _) => page,
      transitionDuration: const Duration(milliseconds: 500),
      transitionsBuilder: (context, animation, _, child) {
        return FadeTransition(
          opacity: animation,
          child: RotationTransition(
            turns: Tween(begin: 0.05, end: 0.0).animate(
              CurvedAnimation(parent: animation, curve: Curves.easeOut),
            ),
            child: child,
          ),
        );
      },
    );
  }
}

class TransitionsDemo extends StatelessWidget {
  const TransitionsDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Transitions')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _TransitionTile(
            title: 'Fade',
            onTap: () => Navigator.push(
              context,
              CustomRoutes.fade(const SamplePage(title: 'Fade')),
            ),
          ),
          _TransitionTile(
            title: 'Scale (Elastic)',
            onTap: () => Navigator.push(
              context,
              CustomRoutes.scale(const SamplePage(title: 'Scale')),
            ),
          ),
          _TransitionTile(
            title: 'Slide From Bottom',
            onTap: () => Navigator.push(
              context,
              CustomRoutes.slideFromBottom(
                  const SamplePage(title: 'Slide Bottom')),
            ),
          ),
          _TransitionTile(
            title: 'Slide From Right (with secondary)',
            onTap: () => Navigator.push(
              context,
              CustomRoutes.slideFromRight(
                  const SamplePage(title: 'Slide Right')),
            ),
          ),
          _TransitionTile(
            title: 'Rotate + Fade',
            onTap: () => Navigator.push(
              context,
              CustomRoutes.rotateFade(
                  const SamplePage(title: 'Rotate Fade')),
            ),
          ),
        ],
      ),
    );
  }
}

class _TransitionTile extends StatelessWidget {
  final String title;
  final VoidCallback onTap;
  const _TransitionTile({required this.title, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: ListTile(
        title: Text(title),
        trailing: const Icon(Icons.arrow_forward_ios, size: 14),
        onTap: onTap,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 87: WillPopScope & PopScope

ควบคุมการกด Back

```dart
import 'package:flutter/material.dart';

class FormWithBackGuard extends StatefulWidget {
  const FormWithBackGuard({super.key});

  @override
  State<FormWithBackGuard> createState() => _FormWithBackGuardState();
}

class _FormWithBackGuardState extends State<FormWithBackGuard> {
  final _formKey = GlobalKey<FormState>();
  bool _hasChanges = false;

  @override
  Widget build(BuildContext context) {
    // PopScope แทน WillPopScope (Flutter 3.12+)
    return PopScope(
      canPop: !_hasChanges,  // false = ป้องกันการ pop
      onPopInvoked: (didPop) async {
        if (didPop) return;  // pop สำเร็จแล้ว

        // แสดง dialog ถามผู้ใช้
        final shouldPop = await _showExitDialog();
        if (shouldPop && context.mounted) {
          Navigator.pop(context);
        }
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Form พร้อม Back Guard'),
          actions: [
            if (_hasChanges)
              Container(
                margin: const EdgeInsets.only(right: 8),
                padding: const EdgeInsets.symmetric(
                    horizontal: 8, vertical: 4),
                decoration: BoxDecoration(
                  color: Colors.orange.shade100,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: const Text('มีการแก้ไข',
                    style: TextStyle(color: Colors.orange, fontSize: 12)),
              ),
          ],
        ),
        body: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Form(
            key: _formKey,
            onChanged: () => setState(() => _hasChanges = true),
            child: Column(
              children: [
                const TextField(
                  decoration: InputDecoration(
                    labelText: 'ชื่อ',
                    border: OutlineInputBorder(),
                  ),
                ),
                const SizedBox(height: 16),
                const TextField(
                  maxLines: 3,
                  decoration: InputDecoration(
                    labelText: 'รายละเอียด',
                    border: OutlineInputBorder(),
                    alignLabelWithHint: true,
                  ),
                ),
                const SizedBox(height: 24),
                Row(
                  children: [
                    Expanded(
                      child: ElevatedButton(
                        onPressed: () {
                          setState(() => _hasChanges = false);
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(
                                content: Text('บันทึกแล้ว!')),
                          );
                          Navigator.pop(context);
                        },
                        child: const Text('บันทึก'),
                      ),
                    ),
                    const SizedBox(width: 8),
                    Expanded(
                      child: OutlinedButton(
                        onPressed: () async {
                          if (_hasChanges) {
                            final shouldDiscard =
                                await _showExitDialog();
                            if (shouldDiscard && context.mounted) {
                              Navigator.pop(context);
                            }
                          } else {
                            Navigator.pop(context);
                          }
                        },
                        child: const Text('ยกเลิก'),
                      ),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }

  Future<bool> _showExitDialog() async {
    final result = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('ออกโดยไม่บันทึก?'),
        content: const Text('ข้อมูลที่แก้ไขจะหายไปทั้งหมด'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('อยู่ต่อ'),
          ),
          ElevatedButton(
            style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red),
            onPressed: () => Navigator.pop(context, true),
            child: const Text('ออก', style: TextStyle(color: Colors.white)),
          ),
        ],
      ),
    );
    return result ?? false;
  }
}
```

---

## ขั้นตอนที่ 88: Bottom Navigation & Tab Navigation

```dart
import 'package:flutter/material.dart';

class BottomNavApp extends StatefulWidget {
  const BottomNavApp({super.key});

  @override
  State<BottomNavApp> createState() => _BottomNavAppState();
}

class _BottomNavAppState extends State<BottomNavApp> {
  int _currentIndex = 0;

  // Keep pages alive เมื่อ switch tabs
  final _pages = [
    const PageWrapper(child: FeedPage(), key: PageStorageKey('feed')),
    const PageWrapper(child: SearchPage(), key: PageStorageKey('search')),
    const PageWrapper(child: CartPage(), key: PageStorageKey('cart')),
    const PageWrapper(child: ProfilePage(), key: PageStorageKey('profile')),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // IndexedStack เพื่อ keep state ของแต่ละ tab
      body: IndexedStack(
        index: _currentIndex,
        children: _pages,
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) =>
            setState(() => _currentIndex = index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'หน้าแรก',
          ),
          NavigationDestination(
            icon: Icon(Icons.search),
            label: 'ค้นหา',
          ),
          NavigationDestination(
            icon: Badge(
              label: Text('3'),
              child: Icon(Icons.shopping_cart_outlined),
            ),
            selectedIcon: Icon(Icons.shopping_cart),
            label: 'ตะกร้า',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'โปรไฟล์',
          ),
        ],
      ),
    );
  }
}

// Wrapper เพื่อ keep Navigator state ของแต่ละ tab
class PageWrapper extends StatelessWidget {
  final Widget child;
  const PageWrapper({super.key, required this.child});

  @override
  Widget build(BuildContext context) => Navigator(
        onGenerateRoute: (settings) => MaterialPageRoute(
          builder: (_) => child,
        ),
      );
}

class FeedPage extends StatelessWidget {
  const FeedPage({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Feed')),
        body: ListView.builder(
          itemCount: 10,
          itemBuilder: (context, index) => ListTile(
            title: Text('โพสต์ ${index + 1}'),
            leading: CircleAvatar(child: Text('${index + 1}')),
          ),
        ),
      );
}

class SearchPage extends StatelessWidget {
  const SearchPage({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const TextField(
            decoration: InputDecoration(
              hintText: 'ค้นหา...',
              border: InputBorder.none,
            ),
          ),
        ),
        body: const Center(child: Text('ค้นหา')),
      );
}

class CartPage extends StatelessWidget {
  const CartPage({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('ตะกร้า')),
        body: const Center(child: Text('ตะกร้าสินค้า')),
      );
}

class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key});
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('โปรไฟล์')),
        body: const Center(child: Text('โปรไฟล์')),
      );
}

// ===== Tab Navigation =====
class TabNavigation extends StatelessWidget {
  const TabNavigation({super.key});

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Tab Navigation'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.bolt), text: 'ล่าสุด'),
              Tab(icon: Icon(Icons.trending_up), text: 'ยอดนิยม'),
              Tab(icon: Icon(Icons.bookmark), text: 'บันทึกไว้'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            Center(child: Text('ล่าสุด')),
            Center(child: Text('ยอดนิยม')),
            Center(child: Text('บันทึกไว้')),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 89: Navigator 2.0 เบื้องต้น

Navigator 2.0 ใช้ RouterDelegate และ RouteInformationParser

```dart
import 'package:flutter/material.dart';

// ===== App State =====
enum AppPage { home, detail, settings }

class AppState extends ChangeNotifier {
  AppPage _currentPage = AppPage.home;
  String? _selectedItemId;

  AppPage get currentPage => _currentPage;
  String? get selectedItemId => _selectedItemId;

  void goHome() {
    _currentPage = AppPage.home;
    _selectedItemId = null;
    notifyListeners();
  }

  void goToDetail(String id) {
    _currentPage = AppPage.detail;
    _selectedItemId = id;
    notifyListeners();
  }

  void goToSettings() {
    _currentPage = AppPage.settings;
    notifyListeners();
  }

  void goBack() {
    if (_currentPage != AppPage.home) {
      _currentPage = AppPage.home;
      _selectedItemId = null;
      notifyListeners();
    }
  }
}

// ===== Router Delegate =====
class AppRouterDelegate extends RouterDelegate<AppPage>
    with ChangeNotifier, PopNavigatorRouterDelegateMixin<AppPage> {
  @override
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  final AppState appState;

  AppRouterDelegate(this.appState) {
    appState.addListener(notifyListeners);
  }

  @override
  AppPage get currentConfiguration => appState.currentPage;

  @override
  Widget build(BuildContext context) {
    return Navigator(
      key: navigatorKey,
      pages: [
        // Home always in stack
        MaterialPage(
          key: const ValueKey('home'),
          child: Nav2HomeScreen(state: appState),
        ),

        // Detail - add to stack if needed
        if (appState.currentPage == AppPage.detail)
          MaterialPage(
            key: ValueKey('detail-${appState.selectedItemId}'),
            child: Nav2DetailScreen(
              id: appState.selectedItemId ?? '',
              state: appState,
            ),
          ),

        // Settings
        if (appState.currentPage == AppPage.settings)
          MaterialPage(
            key: const ValueKey('settings'),
            child: Nav2SettingsScreen(state: appState),
          ),
      ],
      onPopPage: (route, result) {
        if (!route.didPop(result)) return false;
        appState.goBack();
        return true;
      },
    );
  }

  @override
  Future<void> setNewRoutePath(AppPage configuration) async {
    // Handle deep links / URL changes
    switch (configuration) {
      case AppPage.home:
        appState.goHome();
      case AppPage.settings:
        appState.goToSettings();
      case AppPage.detail:
        break;
    }
  }
}

// ===== Screens =====
class Nav2HomeScreen extends StatelessWidget {
  final AppState state;
  const Nav2HomeScreen({super.key, required this.state});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Navigator 2.0'),
        actions: [
          IconButton(
            icon: const Icon(Icons.settings),
            onPressed: state.goToSettings,
          ),
        ],
      ),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: 5,
        itemBuilder: (context, index) {
          return Card(
            margin: const EdgeInsets.only(bottom: 8),
            child: ListTile(
              title: Text('รายการ ${index + 1}'),
              trailing: const Icon(Icons.arrow_forward_ios, size: 14),
              onTap: () => state.goToDetail('item-$index'),
            ),
          );
        },
      ),
    );
  }
}

class Nav2DetailScreen extends StatelessWidget {
  final String id;
  final AppState state;
  const Nav2DetailScreen(
      {super.key, required this.id, required this.state});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('รายละเอียด: $id'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: state.goBack,
        ),
      ),
      body: Center(child: Text('ID: $id')),
    );
  }
}

class Nav2SettingsScreen extends StatelessWidget {
  final AppState state;
  const Nav2SettingsScreen({super.key, required this.state});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings'),
        leading: IconButton(
          icon: const Icon(Icons.arrow_back),
          onPressed: state.goBack,
        ),
      ),
      body: const Center(child: Text('Settings Screen')),
    );
  }
}

// ===== App Entry =====
class Nav2App extends StatelessWidget {
  const Nav2App({super.key});

  @override
  Widget build(BuildContext context) {
    final appState = AppState();
    return MaterialApp.router(
      routerDelegate: AppRouterDelegate(appState),
      routeInformationParser: const _AppRouteInformationParser(),
    );
  }
}

class _AppRouteInformationParser
    extends RouteInformationParser<AppPage> {
  const _AppRouteInformationParser();

  @override
  Future<AppPage> parseRouteInformation(
      RouteInformation routeInformation) async {
    final uri = routeInformation.uri;
    if (uri.pathSegments.isEmpty) return AppPage.home;
    switch (uri.pathSegments.first) {
      case 'settings':
        return AppPage.settings;
      default:
        return AppPage.home;
    }
  }

  @override
  RouteInformation restoreRouteInformation(AppPage configuration) {
    switch (configuration) {
      case AppPage.home:
        return RouteInformation(uri: Uri.parse('/'));
      case AppPage.settings:
        return RouteInformation(uri: Uri.parse('/settings'));
      case AppPage.detail:
        return RouteInformation(uri: Uri.parse('/detail'));
    }
  }
}
```

---

## ขั้นตอนที่ 90: Workshop - Multi-screen App

แอปสมบูรณ์ที่ใช้ navigation หลายรูปแบบ

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ShopApp());
}

// ===== Models =====
class ShopProduct {
  final int id;
  final String name;
  final double price;
  final String category;
  final String emoji;
  final String description;
  bool isFavorite;

  ShopProduct({
    required this.id,
    required this.name,
    required this.price,
    required this.category,
    required this.emoji,
    required this.description,
    this.isFavorite = false,
  });
}

class CartItem {
  final ShopProduct product;
  int quantity;
  CartItem({required this.product, this.quantity = 1});
}

// ===== App State =====
class ShopAppState extends ChangeNotifier {
  final List<ShopProduct> products = [
    ShopProduct(id: 1, name: 'MacBook Pro 14"', price: 79900,
        category: 'คอมพิวเตอร์', emoji: '💻',
        description: 'MacBook Pro รุ่นล่าสุด ชิป M3 Pro'),
    ShopProduct(id: 2, name: 'iPhone 16 Pro', price: 44900,
        category: 'มือถือ', emoji: '📱',
        description: 'iPhone รุ่นใหม่ กล้อง 48MP'),
    ShopProduct(id: 3, name: 'iPad Air M2', price: 23900,
        category: 'แท็บเล็ต', emoji: '📲',
        description: 'iPad Air ชิป M2 จอ 11 นิ้ว'),
    ShopProduct(id: 4, name: 'AirPods Pro 2', price: 9900,
        category: 'อุปกรณ์เสียง', emoji: '🎧',
        description: 'หูฟัง True Wireless พร้อม ANC'),
    ShopProduct(id: 5, name: 'Apple Watch Ultra 2', price: 32900,
        category: 'นาฬิกา', emoji: '⌚',
        description: 'สมาร์ทวอทช์สำหรับนักผจญภัย'),
    ShopProduct(id: 6, name: 'Magic Keyboard', price: 4900,
        category: 'อุปกรณ์เสริม', emoji: '⌨️',
        description: 'คีย์บอร์ดไร้สายพร้อม Touch ID'),
  ];

  final List<CartItem> _cart = [];

  List<CartItem> get cart => List.unmodifiable(_cart);

  int get cartCount =>
      _cart.fold(0, (sum, item) => sum + item.quantity);

  double get totalPrice =>
      _cart.fold(0, (sum, item) => sum + item.product.price * item.quantity);

  void toggleFavorite(int id) {
    final product = products.firstWhere((p) => p.id == id);
    product.isFavorite = !product.isFavorite;
    notifyListeners();
  }

  void addToCart(ShopProduct product) {
    final existing = _cart.where((i) => i.product.id == product.id);
    if (existing.isNotEmpty) {
      existing.first.quantity++;
    } else {
      _cart.add(CartItem(product: product));
    }
    notifyListeners();
  }

  void removeFromCart(int productId) {
    _cart.removeWhere((i) => i.product.id == productId);
    notifyListeners();
  }

  void updateQuantity(int productId, int quantity) {
    if (quantity <= 0) {
      removeFromCart(productId);
    } else {
      _cart.firstWhere((i) => i.product.id == productId).quantity = quantity;
    }
    notifyListeners();
  }
}

// ===== State Provider =====
class ShopProvider extends InheritedNotifier<ShopAppState> {
  const ShopProvider({
    super.key,
    required ShopAppState state,
    required super.child,
  }) : super(notifier: state);

  static ShopAppState of(BuildContext context) {
    return context
        .dependOnInheritedWidgetOfExactType<ShopProvider>()!
        .notifier!;
  }
}

// ===== App =====
class ShopApp extends StatelessWidget {
  const ShopApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ShopProvider(
      state: ShopAppState(),
      child: MaterialApp(
        title: 'Shop App',
        theme: ThemeData(
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        ),
        initialRoute: '/',
        onGenerateRoute: _generateRoute,
      ),
    );
  }

  Route<dynamic>? _generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case '/':
        return MaterialPageRoute(
            builder: (_) => const ShopMainScreen());
      case '/product-detail':
        final product = settings.arguments as ShopProduct;
        return MaterialPageRoute(
            builder: (_) => ShopDetailScreen(product: product));
      case '/cart':
        return MaterialPageRoute(
            builder: (_) => const ShopCartScreen());
      case '/checkout':
        return MaterialPageRoute(
          builder: (_) => const ShopCheckoutScreen(),
          fullscreenDialog: true,
        );
      case '/order-success':
        return CustomRoutes.slideFromBottom(
            const OrderSuccessScreen());
      default:
        return null;
    }
  }
}

// ===== Custom Routes (reuse from step 86) =====
class CustomRoutes {
  static Route<T> slideFromBottom<T>(Widget page) {
    return PageRouteBuilder<T>(
      pageBuilder: (_, animation, __) => page,
      transitionDuration: const Duration(milliseconds: 400),
      transitionsBuilder: (_, animation, __, child) {
        final tween = Tween(
          begin: const Offset(0, 1),
          end: Offset.zero,
        ).chain(CurveTween(curve: Curves.easeOutCubic));
        return SlideTransition(
          position: animation.drive(tween),
          child: child,
        );
      },
    );
  }
}

// ===== Main Screen (Bottom Nav) =====
class ShopMainScreen extends StatefulWidget {
  const ShopMainScreen({super.key});

  @override
  State<ShopMainScreen> createState() => _ShopMainScreenState();
}

class _ShopMainScreenState extends State<ShopMainScreen> {
  int _currentTab = 0;

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);

    return Scaffold(
      body: IndexedStack(
        index: _currentTab,
        children: const [
          ShopProductListTab(),
          ShopFavoritesTab(),
        ],
      ),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentTab,
        onDestinationSelected: (i) => setState(() => _currentTab = i),
        destinations: [
          const NavigationDestination(
            icon: Icon(Icons.store_outlined),
            selectedIcon: Icon(Icons.store),
            label: 'ร้านค้า',
          ),
          NavigationDestination(
            icon: Badge(
              isLabelVisible:
                  state.products.any((p) => p.isFavorite),
              child: const Icon(Icons.favorite_outline),
            ),
            selectedIcon: const Icon(Icons.favorite),
            label: 'โปรด',
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () => Navigator.pushNamed(context, '/cart'),
        icon: Badge(
          label: Text('${state.cartCount}'),
          isLabelVisible: state.cartCount > 0,
          child: const Icon(Icons.shopping_cart),
        ),
        label: Text('฿${state.totalPrice.toStringAsFixed(0)}'),
      ),
    );
  }
}

// ===== Product List Tab =====
class ShopProductListTab extends StatelessWidget {
  const ShopProductListTab({super.key});

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('สินค้าทั้งหมด'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () {},
          ),
        ],
      ),
      body: GridView.builder(
        padding: const EdgeInsets.all(16),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
          crossAxisSpacing: 12,
          mainAxisSpacing: 12,
        ),
        itemCount: state.products.length,
        itemBuilder: (context, index) {
          final product = state.products[index];
          return _ProductCard(product: product);
        },
      ),
    );
  }
}

class _ProductCard extends StatelessWidget {
  final ShopProduct product;
  const _ProductCard({required this.product});

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);

    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: () => Navigator.pushNamed(
          context,
          '/product-detail',
          arguments: product,
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Image area
            Expanded(
              child: Stack(
                fit: StackFit.expand,
                children: [
                  Container(
                    color: Colors.grey.shade50,
                    child: Center(
                      child: Text(
                        product.emoji,
                        style: const TextStyle(fontSize: 56),
                      ),
                    ),
                  ),
                  Positioned(
                    top: 4,
                    right: 4,
                    child: IconButton(
                      icon: Icon(
                        product.isFavorite
                            ? Icons.favorite
                            : Icons.favorite_border,
                        color: product.isFavorite ? Colors.red : Colors.grey,
                      ),
                      onPressed: () =>
                          state.toggleFavorite(product.id),
                    ),
                  ),
                ],
              ),
            ),
            // Info
            Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: const TextStyle(
                        fontWeight: FontWeight.bold, fontSize: 13),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  Text(
                    '฿${product.price.toStringAsFixed(0)}',
                    style: TextStyle(
                      color: Theme.of(context).colorScheme.primary,
                      fontWeight: FontWeight.w600,
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
}

// ===== Favorites Tab =====
class ShopFavoritesTab extends StatelessWidget {
  const ShopFavoritesTab({super.key});

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);
    final favorites =
        state.products.where((p) => p.isFavorite).toList();

    return Scaffold(
      appBar: AppBar(title: const Text('รายการโปรด')),
      body: favorites.isEmpty
          ? const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.favorite_border,
                      size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('ยังไม่มีรายการโปรด',
                      style: TextStyle(color: Colors.grey)),
                ],
              ),
            )
          : ListView.builder(
              itemCount: favorites.length,
              itemBuilder: (context, index) {
                final product = favorites[index];
                return ListTile(
                  leading: Text(product.emoji,
                      style: const TextStyle(fontSize: 32)),
                  title: Text(product.name),
                  subtitle:
                      Text('฿${product.price.toStringAsFixed(0)}'),
                  trailing: IconButton(
                    icon: const Icon(Icons.favorite, color: Colors.red),
                    onPressed: () =>
                        state.toggleFavorite(product.id),
                  ),
                  onTap: () => Navigator.pushNamed(
                    context,
                    '/product-detail',
                    arguments: product,
                  ),
                );
              },
            ),
    );
  }
}

// ===== Product Detail Screen =====
class ShopDetailScreen extends StatelessWidget {
  final ShopProduct product;
  const ShopDetailScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);

    return Scaffold(
      appBar: AppBar(
        title: Text(product.name),
        actions: [
          IconButton(
            icon: Icon(
              product.isFavorite
                  ? Icons.favorite
                  : Icons.favorite_border,
              color: product.isFavorite ? Colors.red : null,
            ),
            onPressed: () => state.toggleFavorite(product.id),
          ),
        ],
      ),
      body: SingleChildScrollView(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Container(
              height: 280,
              color: Colors.grey.shade50,
              child: Center(
                child: Text(product.emoji,
                    style: const TextStyle(fontSize: 120)),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: const TextStyle(
                        fontSize: 24, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  Chip(label: Text(product.category)),
                  const SizedBox(height: 16),
                  Text(
                    '฿${product.price.toStringAsFixed(0)}',
                    style: TextStyle(
                      fontSize: 28,
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).colorScheme.primary,
                    ),
                  ),
                  const SizedBox(height: 16),
                  const Text('รายละเอียด',
                      style: TextStyle(
                          fontSize: 18, fontWeight: FontWeight.bold)),
                  const SizedBox(height: 8),
                  Text(product.description,
                      style: const TextStyle(fontSize: 15)),
                ],
              ),
            ),
          ],
        ),
      ),
      bottomNavigationBar: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: ElevatedButton.icon(
            icon: const Icon(Icons.shopping_cart),
            label: const Text('เพิ่มลงตะกร้า'),
            style: ElevatedButton.styleFrom(
              minimumSize: const Size.fromHeight(48),
            ),
            onPressed: () {
              state.addToCart(product);
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('เพิ่ม ${product.name} ลงตะกร้าแล้ว'),
                  action: SnackBarAction(
                    label: 'ดูตะกร้า',
                    onPressed: () =>
                        Navigator.pushNamed(context, '/cart'),
                  ),
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}

// ===== Cart Screen =====
class ShopCartScreen extends StatelessWidget {
  const ShopCartScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final state = ShopProvider.of(context);
    final cart = state.cart;

    return Scaffold(
      appBar: AppBar(
        title: Text('ตะกร้า (${state.cartCount} รายการ)'),
      ),
      body: cart.isEmpty
          ? const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.shopping_cart_outlined,
                      size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('ตะกร้าว่าง',
                      style: TextStyle(color: Colors.grey, fontSize: 18)),
                ],
              ),
            )
          : Column(
              children: [
                Expanded(
                  child: ListView.builder(
                    itemCount: cart.length,
                    itemBuilder: (context, index) {
                      final item = cart[index];
                      return ListTile(
                        leading: Text(item.product.emoji,
                            style: const TextStyle(fontSize: 32)),
                        title: Text(item.product.name),
                        subtitle: Text(
                            '฿${item.product.price.toStringAsFixed(0)}'),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              icon: const Icon(Icons.remove),
                              onPressed: () => state.updateQuantity(
                                  item.product.id, item.quantity - 1),
                            ),
                            Text('${item.quantity}',
                                style: const TextStyle(
                                    fontWeight: FontWeight.bold)),
                            IconButton(
                              icon: const Icon(Icons.add),
                              onPressed: () => state.updateQuantity(
                                  item.product.id, item.quantity + 1),
                            ),
                            IconButton(
                              icon: const Icon(Icons.delete_outline,
                                  color: Colors.red),
                              onPressed: () =>
                                  state.removeFromCart(item.product.id),
                            ),
                          ],
                        ),
                      );
                    },
                  ),
                ),
                Container(
                  padding: const EdgeInsets.all(16),
                  decoration: BoxDecoration(
                    color: Colors.white,
                    boxShadow: [
                      BoxShadow(
                        color: Colors.grey.withOpacity(0.2),
                        blurRadius: 8,
                        offset: const Offset(0, -2),
                      ),
                    ],
                  ),
                  child: Column(
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          const Text('ยอดรวม:',
                              style: TextStyle(fontSize: 18)),
                          Text(
                            '฿${state.totalPrice.toStringAsFixed(0)}',
                            style: const TextStyle(
                              fontSize: 22,
                              fontWeight: FontWeight.bold,
                              color: Colors.green,
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 12),
                      SizedBox(
                        width: double.infinity,
                        child: ElevatedButton(
                          onPressed: () => Navigator.pushNamed(
                              context, '/checkout'),
                          style: ElevatedButton.styleFrom(
                              padding:
                                  const EdgeInsets.symmetric(vertical: 14)),
                          child: const Text('ดำเนินการสั่งซื้อ',
                              style: TextStyle(fontSize: 16)),
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

// ===== Checkout Screen =====
class ShopCheckoutScreen extends StatefulWidget {
  const ShopCheckoutScreen({super.key});

  @override
  State<ShopCheckoutScreen> createState() => _ShopCheckoutScreenState();
}

class _ShopCheckoutScreenState extends State<ShopCheckoutScreen> {
  bool _isProcessing = false;
  String _payment = 'credit';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ชำระเงิน')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('ที่อยู่จัดส่ง',
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
            const SizedBox(height: 8),
            const TextField(
              decoration: InputDecoration(
                labelText: 'ที่อยู่',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            const Text('วิธีชำระเงิน',
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
            const SizedBox(height: 8),
            ...['credit', 'promptpay', 'cod'].map((method) {
              final labels = {
                'credit': 'บัตรเครดิต/เดบิต',
                'promptpay': 'พร้อมเพย์',
                'cod': 'เก็บเงินปลายทาง',
              };
              return RadioListTile<String>(
                title: Text(labels[method]!),
                value: method,
                groupValue: _payment,
                onChanged: (v) => setState(() => _payment = v!),
                contentPadding: EdgeInsets.zero,
              );
            }),
          ],
        ),
      ),
      bottomNavigationBar: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: ElevatedButton(
            onPressed: _isProcessing ? null : _processPayment,
            style: ElevatedButton.styleFrom(
              minimumSize: const Size.fromHeight(48),
              backgroundColor: Colors.green,
            ),
            child: _isProcessing
                ? const SizedBox(
                    height: 20,
                    width: 20,
                    child: CircularProgressIndicator(
                        strokeWidth: 2, color: Colors.white),
                  )
                : const Text('ยืนยันการสั่งซื้อ',
                    style: TextStyle(color: Colors.white, fontSize: 16)),
          ),
        ),
      ),
    );
  }

  Future<void> _processPayment() async {
    setState(() => _isProcessing = true);
    await Future.delayed(const Duration(seconds: 2));

    if (!mounted) return;
    // ไปหน้า success และล้าง stack
    Navigator.pushNamedAndRemoveUntil(
      context,
      '/order-success',
      (route) => route.settings.name == '/',
    );
  }
}

// ===== Order Success =====
class OrderSuccessScreen extends StatelessWidget {
  const OrderSuccessScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.check_circle, size: 96, color: Colors.green),
            const SizedBox(height: 24),
            const Text('สั่งซื้อสำเร็จ!',
                style: TextStyle(
                    fontSize: 28, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('ขอบคุณที่ใช้บริการ',
                style: TextStyle(color: Colors.grey, fontSize: 16)),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () {
                Navigator.popUntil(
                    context, (route) => route.settings.name == '/');
              },
              child: const Text('กลับหน้าแรก'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## สรุป (Summary)

| Navigator Method | การใช้งาน |
|-----------------|-----------|
| `push` | ไปหน้าใหม่ เก็บ stack |
| `pop` | กลับหน้าก่อน ลบจาก stack |
| `pushReplacement` | แทนที่หน้าปัจจุบัน |
| `pushAndRemoveUntil` | ไปหน้าใหม่และล้าง stack |
| `pushNamed` | ไปด้วยชื่อ route |
| `onGenerateRoute` | จัดการ route แบบ dynamic |
| `MaterialPageRoute` | Android transition |
| `CupertinoPageRoute` | iOS transition |
| `PageRouteBuilder` | Custom transition |
| `PopScope` | ควบคุมการกด Back |
| `NavigationBar` | Bottom navigation M3 |
| `TabBar/TabBarView` | Tab navigation |
| Navigator 2.0 | Declarative routing |

---

## แบบฝึกหัด (Exercises)

1. **Auth Flow**: สร้าง Login → Register → Forgot Password โดยใช้ pushReplacement ให้ถูกต้อง

2. **Deep Linking**: ตั้งค่า onGenerateRoute ให้รับ `/user/123`, `/post/abc`, `/category/tech`

3. **Transition Library**: สร้าง class TransitionRoutes พร้อม transitions อย่างน้อย 5 แบบ

4. **Nested Navigation**: สร้างแอปที่มี Bottom nav และแต่ละ tab มี navigator ของตัวเอง

5. **Navigator 2.0 Shop**: แปลง ShopApp ให้ใช้ Navigator 2.0 + URL routing สำหรับ web

---

[← Part 08: Forms และ Input Handling](part_08.md) | [Part 10: Assets, Images, Icons →](part_10.md)

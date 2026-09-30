# Part 37: Flutter Web
## ขั้นตอนที่ 361-370

---

## สารบัญ
1. [Flutter Web คืออะไร?](#flutter-web-คืออะไร)
2. [Flutter Web Setup](#flutter-web-setup)
3. [Web-specific Widgets](#web-specific-widgets)
4. [Responsive Design](#responsive-design)
5. [URL Routing ด้วย go_router](#url-routing)
6. [Web Performance Optimization](#web-performance)
7. [SEO Considerations](#seo-considerations)
8. [PWA Setup](#pwa-setup)
9. [Deploy ไปยัง Firebase Hosting](#deploy-firebase)
10. [Deploy ไปยัง Vercel/Netlify](#deploy-vercel)

---

## ขั้นตอนที่ 361: Flutter Web คืออะไร?

Flutter Web ช่วยให้เราเอา Flutter app ไปรันบน Browser ได้

```
Flutter Web Architecture:
┌─────────────────────────────────────────────────────────┐
│                    Browser (Chrome/Safari/Firefox)       │
├─────────────────────────────────────────────────────────┤
│              Flutter Web Runtime                         │
│  ┌────────────────┐  ┌─────────────────────────────┐   │
│  │  HTML Renderer │  │      CanvasKit Renderer      │   │
│  │  (DOM-based)   │  │  (WebGL + WebAssembly)       │   │
│  │  - เร็ว load   │  │  - Pixel-perfect              │   │
│  │  - SEO ดีกว่า │  │  - Better performance         │   │
│  └────────────────┘  └─────────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│                   Flutter Framework                      │
│              (ใช้โค้ดเดียวกับ Mobile)                    │
└─────────────────────────────────────────────────────────┘

Renderer เลือกอัตโนมัติ:
- Mobile: CanvasKit
- Desktop: CanvasKit
- ถ้า size เล็ก: HTML Renderer
```

### ข้อดีและข้อจำกัด

```
ข้อดี:
✅ ใช้โค้ดเดียวกับ Mobile (Code reuse)
✅ Pixel-perfect rendering
✅ Rich UI animations
✅ PWA support

ข้อจำกัด:
❌ Bundle size ใหญ่ (~2MB compressed)
❌ Initial load ช้ากว่า traditional web
❌ SEO ยาก (แต่แก้ได้)
❌ บาง HTML/CSS features ไม่รองรับ
```

---

## ขั้นตอนที่ 362: Flutter Web Setup

```bash
# ตรวจสอบว่ารองรับ web
flutter doctor

# เพิ่ม web support
flutter create --platforms web .

# หรือสร้าง project ใหม่ที่รองรับ web
flutter create --platforms android,ios,web my_web_app

# รัน on web
flutter run -d chrome
flutter run -d web-server --web-port 8080

# Build สำหรับ production
flutter build web --release
flutter build web --release --web-renderer canvaskit  # เต็มคุณภาพ
flutter build web --release --web-renderer html       # เร็วกว่า, SEO ดีกว่า
```

### pubspec.yaml สำหรับ Web

```yaml
name: my_web_app
description: Flutter Web Application

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  
  # Routing
  go_router: ^13.0.0
  
  # State Management
  flutter_riverpod: ^2.4.9
  
  # SEO (สำหรับ web)
  flutter_meta_seo: ^1.0.0
  
  # Web-specific packages
  url_launcher: ^6.2.2
  
  # Analytics
  firebase_analytics: ^10.7.4

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/web/
```

### web/index.html

```html
<!DOCTYPE html>
<html>
<head>
  <!--
    If you are serving your web app in a path other than the root, change the
    href value below to reflect the base path you are serving from.
    The path provided below has to start and end with a slash "/" in order for
    it to work correctly.
  -->
  <base href="/">

  <meta charset="UTF-8">
  <meta content="IE=Edge" http-equiv="X-UA-Compatible">
  
  <!-- SEO Meta Tags -->
  <meta name="description" content="My Flutter Web App - A cross-platform app">
  <meta name="keywords" content="flutter, web, app">
  <meta name="author" content="My Name">
  
  <!-- Open Graph -->
  <meta property="og:title" content="My Flutter Web App">
  <meta property="og:description" content="Description of my app">
  <meta property="og:image" content="/icons/og-image.jpg">
  <meta property="og:url" content="https://myapp.com">
  <meta property="og:type" content="website">
  
  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="My Flutter Web App">
  <meta name="twitter:description" content="Description">
  <meta name="twitter:image" content="/icons/og-image.jpg">
  
  <!-- PWA -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">
  <meta name="apple-mobile-web-app-title" content="My App">
  <link rel="apple-touch-icon" href="/icons/Icon-192.png">

  <!-- Favicon -->
  <link rel="icon" type="image/png" href="/favicon.png"/>

  <title>My Flutter Web App</title>
  <link rel="manifest" href="manifest.json">
</head>
<body>
  <!-- Loading Screen -->
  <div id="loading">
    <style>
      body { 
        margin: 0;
        background: #ffffff;
      }
      #loading {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        flex-direction: column;
      }
      .spinner {
        width: 40px;
        height: 40px;
        border: 4px solid #f3f3f3;
        border-top: 4px solid #3498db;
        border-radius: 50%;
        animation: spin 1s linear infinite;
      }
      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
    </style>
    <div class="spinner"></div>
    <p style="margin-top: 16px; color: #666;">Loading...</p>
  </div>

  <script>
    window.addEventListener('flutter-first-frame', function () {
      document.getElementById('loading').remove();
    });
  </script>

  <script src="flutter_bootstrap.js" async></script>
</body>
</html>
```

---

## ขั้นตอนที่ 363: Web-specific Widgets

```dart
// ใช้ kIsWeb เพื่อตรวจสอบว่ารัน Web หรือเปล่า
import 'package:flutter/foundation.dart';

class PlatformAwareWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    if (kIsWeb) {
      return const WebSpecificWidget();
    } else {
      return const MobileSpecificWidget();
    }
  }
}
```

```dart
// Mouse Hover สำหรับ Web
class HoverableCard extends StatefulWidget {
  final Widget child;
  final VoidCallback? onTap;

  const HoverableCard({super.key, required this.child, this.onTap});

  @override
  State<HoverableCard> createState() => _HoverableCardState();
}

class _HoverableCardState extends State<HoverableCard> {
  bool _isHovered = false;

  @override
  Widget build(BuildContext context) {
    return MouseRegion(
      onEnter: (_) => setState(() => _isHovered = true),
      onExit: (_) => setState(() => _isHovered = false),
      cursor: SystemMouseCursors.click,
      child: GestureDetector(
        onTap: widget.onTap,
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          transform: Matrix4.identity()
            ..scale(_isHovered ? 1.02 : 1.0),
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(12),
            boxShadow: _isHovered
                ? [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.15),
                      blurRadius: 12,
                      offset: const Offset(0, 4),
                    ),
                  ]
                : [
                    BoxShadow(
                      color: Colors.black.withOpacity(0.05),
                      blurRadius: 4,
                      offset: const Offset(0, 2),
                    ),
                  ],
          ),
          child: widget.child,
        ),
      ),
    );
  }
}
```

```dart
// SelectableText สำหรับ Web (copy ได้)
class SelectableContent extends StatelessWidget {
  final String text;
  
  const SelectableContent({super.key, required this.text});

  @override
  Widget build(BuildContext context) {
    return SelectableText(
      text,
      style: Theme.of(context).textTheme.bodyLarge,
      // สำหรับ web - enable copy/paste
      enableInteractiveSelection: true,
    );
  }
}
```

```dart
// ScrollBehavior สำหรับ Web (mouse scrolling)
class WebScrollBehavior extends MaterialScrollBehavior {
  @override
  Set<PointerDeviceKind> get dragDevices => {
    PointerDeviceKind.touch,
    PointerDeviceKind.mouse,     // Allow mouse drag
    PointerDeviceKind.stylus,
    PointerDeviceKind.invertedStylus,
  };
}

// ใช้ใน MaterialApp
MaterialApp(
  scrollBehavior: WebScrollBehavior(),
  // ...
)
```

---

## ขั้นตอนที่ 364: Responsive Design

```dart
// Breakpoints
class Breakpoints {
  static const double mobile = 600;
  static const double tablet = 900;
  static const double desktop = 1200;
  static const double widescreen = 1800;
}

// Responsive helper
class ResponsiveLayout extends StatelessWidget {
  final Widget mobile;
  final Widget? tablet;
  final Widget? desktop;

  const ResponsiveLayout({
    super.key,
    required this.mobile,
    this.tablet,
    this.desktop,
  });

  static bool isMobile(BuildContext context) =>
      MediaQuery.sizeOf(context).width < Breakpoints.mobile;

  static bool isTablet(BuildContext context) =>
      MediaQuery.sizeOf(context).width >= Breakpoints.mobile &&
      MediaQuery.sizeOf(context).width < Breakpoints.desktop;

  static bool isDesktop(BuildContext context) =>
      MediaQuery.sizeOf(context).width >= Breakpoints.desktop;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= Breakpoints.desktop) {
          return desktop ?? tablet ?? mobile;
        }
        if (constraints.maxWidth >= Breakpoints.mobile) {
          return tablet ?? mobile;
        }
        return mobile;
      },
    );
  }
}
```

```dart
// Responsive Grid
class ResponsiveGrid extends StatelessWidget {
  final List<Widget> children;
  final double childAspectRatio;

  const ResponsiveGrid({
    super.key,
    required this.children,
    this.childAspectRatio = 1.0,
  });

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final width = constraints.maxWidth;
        
        int crossAxisCount;
        if (width >= Breakpoints.widescreen) {
          crossAxisCount = 6;
        } else if (width >= Breakpoints.desktop) {
          crossAxisCount = 4;
        } else if (width >= Breakpoints.tablet) {
          crossAxisCount = 3;
        } else if (width >= Breakpoints.mobile) {
          crossAxisCount = 2;
        } else {
          crossAxisCount = 1;
        }

        return GridView.count(
          crossAxisCount: crossAxisCount,
          childAspectRatio: childAspectRatio,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
          children: children,
        );
      },
    );
  }
}
```

### Responsive Scaffold

```dart
// Adaptive Navigation
class AdaptiveScaffold extends StatefulWidget {
  final Widget body;
  final List<NavigationDestination> destinations;
  final int selectedIndex;
  final ValueChanged<int> onDestinationSelected;

  const AdaptiveScaffold({
    super.key,
    required this.body,
    required this.destinations,
    required this.selectedIndex,
    required this.onDestinationSelected,
  });

  @override
  State<AdaptiveScaffold> createState() => _AdaptiveScaffoldState();
}

class _AdaptiveScaffoldState extends State<AdaptiveScaffold> {
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= Breakpoints.desktop) {
          // Desktop: Side Navigation Rail
          return Scaffold(
            body: Row(
              children: [
                NavigationRail(
                  destinations: widget.destinations
                      .map((d) => NavigationRailDestination(
                            icon: d.icon,
                            label: Text(d.label),
                          ))
                      .toList(),
                  selectedIndex: widget.selectedIndex,
                  onDestinationSelected: widget.onDestinationSelected,
                  extended: constraints.maxWidth >= Breakpoints.widescreen,
                  labelType: constraints.maxWidth >= Breakpoints.widescreen
                      ? NavigationRailLabelType.none
                      : NavigationRailLabelType.all,
                ),
                const VerticalDivider(width: 1),
                Expanded(child: widget.body),
              ],
            ),
          );
        }
        
        // Mobile/Tablet: Bottom Navigation Bar
        return Scaffold(
          body: widget.body,
          bottomNavigationBar: NavigationBar(
            destinations: widget.destinations,
            selectedIndex: widget.selectedIndex,
            onDestinationSelected: widget.onDestinationSelected,
          ),
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 365: URL Routing ด้วย go_router

```dart
// pubspec.yaml
// go_router: ^13.0.0

// lib/core/router/app_router.dart
import 'package:go_router/go_router.dart';

final appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  
  // Error handling
  errorBuilder: (context, state) => ErrorPage(error: state.error),
  
  // Redirect (Auth guard)
  redirect: (context, state) {
    final isLoggedIn = AuthService.instance.isLoggedIn;
    final isAuthRoute = state.matchedLocation.startsWith('/auth');
    
    if (!isLoggedIn && !isAuthRoute) {
      return '/auth/login?redirect=${state.uri}';
    }
    
    if (isLoggedIn && isAuthRoute) {
      return '/';
    }
    
    return null;
  },
  
  routes: [
    // Home
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomePage(),
    ),
    
    // Products
    GoRoute(
      path: '/products',
      name: 'products',
      builder: (context, state) {
        final category = state.uri.queryParameters['category'];
        final search = state.uri.queryParameters['q'];
        return ProductListPage(category: category, search: search);
      },
      routes: [
        // Product Detail - /products/:id
        GoRoute(
          path: ':id',
          name: 'product-detail',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return ProductDetailPage(productId: id);
          },
        ),
      ],
    ),
    
    // Cart
    GoRoute(
      path: '/cart',
      name: 'cart',
      builder: (context, state) => const CartPage(),
    ),
    
    // Auth Routes (Shell)
    ShellRoute(
      builder: (context, state, child) => AuthShell(child: child),
      routes: [
        GoRoute(
          path: '/auth/login',
          name: 'login',
          builder: (context, state) {
            final redirect = state.uri.queryParameters['redirect'] ?? '/';
            return LoginPage(redirectUrl: redirect);
          },
        ),
        GoRoute(
          path: '/auth/register',
          name: 'register',
          builder: (context, state) => const RegisterPage(),
        ),
      ],
    ),
    
    // Profile
    GoRoute(
      path: '/profile',
      name: 'profile',
      builder: (context, state) => const ProfilePage(),
      routes: [
        GoRoute(
          path: 'edit',
          name: 'edit-profile',
          builder: (context, state) => const EditProfilePage(),
        ),
        GoRoute(
          path: 'orders',
          name: 'orders',
          builder: (context, state) => const OrdersPage(),
        ),
      ],
    ),
  ],
);
```

### การ Navigate

```dart
// ใน Widget
class NavigationExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // ไปที่ route ด้วย path
        ElevatedButton(
          onPressed: () => context.go('/products'),
          child: const Text('Products'),
        ),
        
        // ไปด้วย name
        ElevatedButton(
          onPressed: () => context.goNamed('product-detail',
            pathParameters: {'id': '123'},
          ),
          child: const Text('Product Detail'),
        ),
        
        // Push (เพิ่ม history entry)
        ElevatedButton(
          onPressed: () => context.push('/products/456'),
          child: const Text('Push Product'),
        ),
        
        // กลับ
        ElevatedButton(
          onPressed: () => context.pop(),
          child: const Text('Back'),
        ),
        
        // Navigate พร้อม query parameters
        ElevatedButton(
          onPressed: () => context.go(
            '/products',
            extra: {'category': 'electronics'},
          ),
          child: const Text('Electronics'),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 366: Web Performance Optimization

### Lazy Loading (deferred imports)

```dart
// ใช้ deferred imports เพื่อ lazy load
import 'package:my_app/features/heavy_feature/heavy_page.dart' deferred as heavyPage;

class LazyLoadingPage extends StatefulWidget {
  @override
  State<LazyLoadingPage> createState() => _LazyLoadingPageState();
}

class _LazyLoadingPageState extends State<LazyLoadingPage> {
  bool _isLoaded = false;

  Future<void> _loadFeature() async {
    await heavyPage.loadLibrary();
    setState(() => _isLoaded = true);
  }

  @override
  Widget build(BuildContext context) {
    if (!_isLoaded) {
      return ElevatedButton(
        onPressed: _loadFeature,
        child: const Text('Load Heavy Feature'),
      );
    }
    
    return heavyPage.HeavyPage();
  }
}
```

### Tree Shaking

```bash
# Build optimized web bundle
flutter build web --release --no-source-maps

# ตรวจสอบ bundle size
ls -la build/web/

# Analyze bundle
flutter build web --analyze-size
```

### Service Worker Cache

```javascript
// web/flutter_service_worker.js
const CACHE_NAME = 'my-app-cache-v1';
const ASSETS_TO_CACHE = [
  '/',
  '/index.html',
  '/flutter.js',
  '/main.dart.js',
  '/manifest.json',
  '/icons/Icon-192.png',
  '/icons/Icon-512.png',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(ASSETS_TO_CACHE);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

---

## ขั้นตอนที่ 367: SEO Considerations

Flutter Web ด้วย HTML Renderer มี SEO ดีกว่า CanvasKit

```dart
// ใส่ Meta Tags แบบ dynamic
class SeoHelper {
  static void setMetaTags({
    required String title,
    required String description,
    String? imageUrl,
    String? canonical,
  }) {
    if (!kIsWeb) return;
    
    // Update document title
    html.document.title = title;
    
    // Update meta description
    _updateMetaTag('description', description);
    
    // Update OG tags
    _updateMetaProperty('og:title', title);
    _updateMetaProperty('og:description', description);
    if (imageUrl != null) {
      _updateMetaProperty('og:image', imageUrl);
    }
    
    // Canonical URL
    if (canonical != null) {
      _updateLinkTag('canonical', canonical);
    }
  }
  
  static void _updateMetaTag(String name, String content) {
    var meta = html.document.querySelector('meta[name="$name"]') 
        as html.MetaElement?;
    
    if (meta == null) {
      meta = html.MetaElement()
        ..name = name;
      html.document.head!.append(meta);
    }
    
    meta.content = content;
  }
  
  static void _updateMetaProperty(String property, String content) {
    var meta = html.document.querySelector('meta[property="$property"]')
        as html.MetaElement?;
    
    if (meta == null) {
      meta = html.MetaElement()
        ..setAttribute('property', property);
      html.document.head!.append(meta);
    }
    
    meta.content = content;
  }
  
  static void _updateLinkTag(String rel, String href) {
    var link = html.document.querySelector('link[rel="$rel"]')
        as html.LinkElement?;
    
    if (link == null) {
      link = html.LinkElement()..rel = rel;
      html.document.head!.append(link);
    }
    
    link.href = href;
  }
}
```

```dart
// ใช้ใน Page
class ProductDetailPage extends StatefulWidget {
  final String productId;
  const ProductDetailPage({super.key, required this.productId});

  @override
  State<ProductDetailPage> createState() => _ProductDetailPageState();
}

class _ProductDetailPageState extends State<ProductDetailPage> {
  @override
  void initState() {
    super.initState();
    _loadProduct();
  }

  Future<void> _loadProduct() async {
    final product = await getProduct(widget.productId);
    
    // Update SEO meta tags
    SeoHelper.setMetaTags(
      title: '${product.name} - My Store',
      description: product.description,
      imageUrl: product.imageUrl,
      canonical: 'https://mystore.com/products/${product.id}',
    );
  }
  
  @override
  Widget build(BuildContext context) => Container();
}
```

### Structured Data (JSON-LD)

```dart
// เพิ่ม Structured Data สำหรับ Products
void addProductStructuredData(Product product) {
  if (!kIsWeb) return;
  
  final jsonLd = {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": product.name,
    "description": product.description,
    "image": product.imageUrl,
    "offers": {
      "@type": "Offer",
      "price": product.price,
      "priceCurrency": "THB",
      "availability": product.isInStock
          ? "https://schema.org/InStock"
          : "https://schema.org/OutOfStock",
    },
  };
  
  final script = html.ScriptElement()
    ..type = 'application/ld+json'
    ..text = json.encode(jsonLd);
  
  html.document.head!.append(script);
}
```

---

## ขั้นตอนที่ 368: PWA Setup

```json
// web/manifest.json
{
  "name": "My Flutter App",
  "short_name": "MyApp",
  "description": "A Flutter Progressive Web App",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#1976D2",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "icons/Icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "icons/Icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "screenshots": [
    {
      "src": "screenshots/home.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Home Screen"
    }
  ],
  "categories": ["shopping", "lifestyle"],
  "lang": "en",
  "dir": "ltr"
}
```

### Install PWA Prompt

```dart
// ตรวจสอบว่า PWA installable หรือเปล่า
class PwaInstallBanner extends StatefulWidget {
  @override
  State<PwaInstallBanner> createState() => _PwaInstallBannerState();
}

class _PwaInstallBannerState extends State<PwaInstallBanner> {
  bool _showBanner = false;
  
  @override
  void initState() {
    super.initState();
    if (kIsWeb) {
      _checkInstallable();
    }
  }
  
  void _checkInstallable() {
    // Listen for beforeinstallprompt event
    html.window.on['beforeinstallprompt'].listen((event) {
      setState(() => _showBanner = true);
    });
  }

  @override
  Widget build(BuildContext context) {
    if (!_showBanner) return const SizedBox.shrink();
    
    return Container(
      color: Theme.of(context).primaryColor,
      padding: const EdgeInsets.all(12),
      child: Row(
        children: [
          const Icon(Icons.install_mobile, color: Colors.white),
          const SizedBox(width: 12),
          Expanded(
            child: Text(
              'Install My App for a better experience',
              style: const TextStyle(color: Colors.white),
            ),
          ),
          TextButton(
            onPressed: () {
              // Trigger install
              setState(() => _showBanner = false);
            },
            child: const Text('Install', style: TextStyle(color: Colors.white)),
          ),
          IconButton(
            icon: const Icon(Icons.close, color: Colors.white),
            onPressed: () => setState(() => _showBanner = false),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 369: Deploy ไปยัง Firebase Hosting

```bash
# ติดตั้ง Firebase CLI
npm install -g firebase-tools

# Login
firebase login

# Initialize Firebase ในโปรเจค
firebase init hosting

# firebase.json จะถูกสร้าง
```

```json
// firebase.json
{
  "hosting": {
    "public": "build/web",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000"
          }
        ]
      },
      {
        "source": "index.html",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      }
    ]
  }
}
```

```bash
# Build Flutter Web
flutter build web --release

# Deploy
firebase deploy --only hosting

# Preview ก่อน Deploy (ให้คนอื่น preview ได้)
firebase hosting:channel:deploy preview
```

### GitHub Actions สำหรับ Firebase Deploy

```yaml
# .github/workflows/deploy-web.yml
name: Deploy to Firebase Hosting

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.0'
      
      - name: Get dependencies
        run: flutter pub get
      
      - name: Build Web
        run: flutter build web --release --web-renderer html
      
      - name: Deploy to Firebase
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
          channelId: live
          projectId: my-flutter-app
```

---

## ขั้นตอนที่ 370: Deploy ไปยัง Vercel/Netlify

### Vercel Deployment

```json
// vercel.json
{
  "buildCommand": "flutter build web --release",
  "outputDirectory": "build/web",
  "framework": null,
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ],
  "headers": [
    {
      "source": "/flutter.js",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

```bash
# ติดตั้ง Vercel CLI
npm install -g vercel

# Deploy
vercel --prod

# หรือ auto deploy ด้วย GitHub integration
# ไปที่ vercel.com > New Project > Import from GitHub
```

### Netlify Deployment

```toml
# netlify.toml
[build]
  command = "flutter build web --release"
  publish = "build/web"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "*.js"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "index.html"
  [headers.values]
    Cache-Control = "no-cache, no-store, must-revalidate"
```

```bash
# ติดตั้ง Netlify CLI
npm install -g netlify-cli

# Build และ Deploy
flutter build web --release
netlify deploy --dir=build/web --prod
```

### Workshop: Complete Web App

```dart
// Workshop: Product Catalog Web App

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  runApp(
    ProviderScope(
      child: const MyApp(),
    ),
  );
}

class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return MaterialApp.router(
      title: 'Flutter Web Store',
      scrollBehavior: WebScrollBehavior(),
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      routerConfig: appRouter,
      builder: (context, child) {
        return Column(
          children: [
            if (kIsWeb) const PwaInstallBanner(),
            Expanded(child: child ?? const SizedBox()),
          ],
        );
      },
    );
  }
}
```

---

## สรุป (Summary)

```
Flutter Web Checklist:

✅ Setup:
  - เพิ่ม flutter/dart สำหรับ web
  - ตั้งค่า web/index.html
  - เลือก renderer ที่เหมาะสม

✅ Responsive:
  - ใช้ LayoutBuilder + Breakpoints
  - AdaptiveScaffold สำหรับ navigation
  - Test หลาย screen sizes

✅ Web UX:
  - Mouse hover effects (MouseRegion)
  - SelectableText
  - ScrollBehavior สำหรับ mouse wheel
  - Cursor changes

✅ Routing:
  - ใช้ go_router สำหรับ URL-based routing
  - Deep linking support
  - Browser back/forward ทำงาน

✅ Performance:
  - Deferred imports สำหรับ lazy loading
  - Service Worker Cache
  - Tree shaking
  - Optimize bundle size

✅ SEO:
  - Meta tags ที่ถูกต้อง
  - Open Graph tags
  - Structured Data (JSON-LD)
  - HTML renderer สำหรับ SEO

✅ PWA:
  - manifest.json
  - Service Worker
  - Install prompt
  - Offline support

✅ Deployment:
  - Firebase Hosting (แนะนำ)
  - Vercel
  - Netlify
  - GitHub Pages
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง Flutter Web app และรันบน Chrome
2. Implement Responsive Layout ที่แสดงได้ทั้ง Mobile, Tablet, Desktop
3. Setup go_router สำหรับ URL routing

**ระดับกลาง:**
4. เพิ่ม Mouse Hover effects สำหรับ cards
5. Implement SEO meta tags ที่ dynamic
6. Deploy ไปยัง Firebase Hosting

**ระดับสูง:**
7. สร้าง PWA ที่ work offline
8. Optimize bundle size โดยใช้ deferred imports
9. Setup CI/CD สำหรับ auto-deploy ไปยัง Firebase

---

## การนำทาง
- [← Part 36: Internationalization](part_36.md)
- [→ Part 38: Flutter Desktop](part_38.md)

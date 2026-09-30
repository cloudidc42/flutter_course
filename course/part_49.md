# Part 49: Scalable App Architecture
## ขั้นตอนที่ 481-490

---

## สารบัญ
1. [Feature-first Folder Structure](#ขั้นตอนที่-481-feature-first-structure)
2. [Atomic Design กับ Flutter Widgets](#ขั้นตอนที่-482-atomic-design)
3. [Design Systems ใน Flutter](#ขั้นตอนที่-483-design-systems)
4. [Plugin Architecture](#ขั้นตอนที่-484-plugin-architecture)
5. [Hot Configuration & Feature Flags](#ขั้นตอนที่-485-feature-flags)
6. [A/B Testing](#ขั้นตอนที่-486-ab-testing)
7. [Analytics Abstraction Layer](#ขั้นตอนที่-487-analytics-layer)
8. [App Health Monitoring](#ขั้นตอนที่-488-health-monitoring)
9. [Performance Profiling](#ขั้นตอนที่-489-performance-profiling)
10. [Workshop: Scalable Architecture Demo](#ขั้นตอนที่-490-workshop)

---

## ขั้นตอนที่ 481: Feature-first Folder Structure

### Feature-first vs Layer-first

```
Layer-first (ไม่แนะนำสำหรับ app ใหญ่):
lib/
├── models/
│   ├── user.dart
│   ├── product.dart
│   └── order.dart
├── services/
│   ├── user_service.dart
│   ├── product_service.dart
│   └── order_service.dart
└── screens/
    ├── user_screen.dart
    ├── product_screen.dart
    └── order_screen.dart

Feature-first (แนะนำ):
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── repositories/
│   │   │   └── sources/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── use_cases/
│   │   └── presentation/
│   │       ├── screens/
│   │       ├── widgets/
│   │       └── providers/
│   ├── products/
│   └── orders/
├── core/                    # Shared utilities
│   ├── network/
│   ├── storage/
│   ├── di/
│   └── utils/
└── shared/                  # Shared widgets
    ├── design_system/
    └── widgets/
```

### Feature Module Structure

```
features/products/
├── data/
│   ├── datasources/
│   │   ├── product_remote_datasource.dart
│   │   └── product_local_datasource.dart
│   ├── models/
│   │   ├── product_model.dart           # Data Transfer Object
│   │   └── product_model.g.dart        # Generated JSON
│   └── repositories/
│       └── product_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── product.dart                # Domain Entity
│   ├── repositories/
│   │   └── product_repository.dart     # Abstract interface
│   └── use_cases/
│       ├── get_products_usecase.dart
│       ├── get_product_by_id_usecase.dart
│       └── search_products_usecase.dart
└── presentation/
    ├── screens/
    │   ├── products_screen.dart
    │   └── product_detail_screen.dart
    ├── widgets/
    │   ├── product_card.dart
    │   ├── product_list.dart
    │   └── product_filter_bar.dart
    └── providers/
        ├── products_provider.dart
        └── product_detail_provider.dart
```

### Barrel Files

```dart
// features/products/products.dart - Public API
// เฉพาะสิ่งที่ต้องการ expose ออกมา

// Domain
export 'domain/entities/product.dart';
export 'domain/use_cases/get_products_usecase.dart';

// Presentation
export 'presentation/screens/products_screen.dart';
export 'presentation/screens/product_detail_screen.dart';
export 'presentation/providers/products_provider.dart';

// ซ่อน implementation details
// ไม่ export: repositories, datasources, models
```

### Feature Registration

```dart
// lib/app_features.dart
class AppFeatures {
  static void register() {
    // Register all features
    FeatureRegistry.register(AuthFeature());
    FeatureRegistry.register(ProductsFeature());
    FeatureRegistry.register(OrdersFeature());
    FeatureRegistry.register(ProfileFeature());
  }
}

abstract class AppFeature {
  String get name;
  String get version;
  
  void registerDependencies(GetIt container);
  List<GoRoute> get routes;
  List<NavigationItem>? get navigationItems;
}

class ProductsFeature implements AppFeature {
  @override
  String get name => 'products';
  
  @override
  String get version => '2.1.0';
  
  @override
  void registerDependencies(GetIt container) {
    container
      ..registerLazySingleton<ProductRemoteDataSource>(
        () => ProductRemoteDataSourceImpl(container<ApiClient>()),
      )
      ..registerLazySingleton<ProductLocalDataSource>(
        () => ProductLocalDataSourceImpl(container<AppDatabase>()),
      )
      ..registerLazySingleton<ProductRepository>(
        () => ProductRepositoryImpl(
          container<ProductRemoteDataSource>(),
          container<ProductLocalDataSource>(),
        ),
      )
      ..registerLazySingleton<GetProductsUseCase>(
        () => GetProductsUseCase(container<ProductRepository>()),
      );
  }
  
  @override
  List<GoRoute> get routes => [
    GoRoute(
      path: '/products',
      builder: (context, state) => const ProductsScreen(),
    ),
    GoRoute(
      path: '/products/:id',
      builder: (context, state) => ProductDetailScreen(
        productId: state.pathParameters['id']!,
      ),
    ),
  ];
  
  @override
  List<NavigationItem>? get navigationItems => [
    NavigationItem(
      label: 'สินค้า',
      icon: Icons.shopping_bag_outlined,
      selectedIcon: Icons.shopping_bag,
      path: '/products',
    ),
  ];
}
```

---

## ขั้นตอนที่ 482: Atomic Design กับ Flutter Widgets

### Atomic Design Hierarchy

```
Atoms:      ปุ่ม, input, icon, text
Molecules:  Form field (label + input + error), Card header
Organisms:  Product card, Navigation bar, Login form
Templates:  Page layouts
Pages:      Complete screens
```

### Atoms

```dart
// lib/shared/design_system/atoms/app_button.dart
enum AppButtonVariant { primary, secondary, outlined, text }
enum AppButtonSize { small, medium, large }

class AppButton extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final AppButtonVariant variant;
  final AppButtonSize size;
  final IconData? leadingIcon;
  final IconData? trailingIcon;
  final bool isLoading;
  final bool isFullWidth;
  
  const AppButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.variant = AppButtonVariant.primary,
    this.size = AppButtonSize.medium,
    this.leadingIcon,
    this.trailingIcon,
    this.isLoading = false,
    this.isFullWidth = false,
  });
  
  @override
  Widget build(BuildContext context) {
    final tokens = AppDesignTokens.of(context);
    
    final padding = switch (size) {
      AppButtonSize.small => EdgeInsets.symmetric(
          horizontal: tokens.spacing.sm,
          vertical: tokens.spacing.xs,
        ),
      AppButtonSize.medium => EdgeInsets.symmetric(
          horizontal: tokens.spacing.md,
          vertical: tokens.spacing.sm,
        ),
      AppButtonSize.large => EdgeInsets.symmetric(
          horizontal: tokens.spacing.lg,
          vertical: tokens.spacing.md,
        ),
    };
    
    final textStyle = switch (size) {
      AppButtonSize.small => tokens.typography.labelSmall,
      AppButtonSize.medium => tokens.typography.labelMedium,
      AppButtonSize.large => tokens.typography.labelLarge,
    };
    
    Widget buttonContent = Row(
      mainAxisSize: isFullWidth ? MainAxisSize.max : MainAxisSize.min,
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        if (isLoading)
          SizedBox(
            width: 16,
            height: 16,
            child: CircularProgressIndicator(
              strokeWidth: 2,
              color: _getContentColor(context),
            ),
          )
        else ...[
          if (leadingIcon != null) ...[
            Icon(leadingIcon, size: 18),
            SizedBox(width: tokens.spacing.xs),
          ],
          Text(label, style: textStyle),
          if (trailingIcon != null) ...[
            SizedBox(width: tokens.spacing.xs),
            Icon(trailingIcon, size: 18),
          ],
        ],
      ],
    );
    
    return switch (variant) {
      AppButtonVariant.primary => FilledButton(
          onPressed: isLoading ? null : onPressed,
          style: FilledButton.styleFrom(padding: padding),
          child: buttonContent,
        ),
      AppButtonVariant.secondary => FilledButton.tonal(
          onPressed: isLoading ? null : onPressed,
          style: FilledButton.styleFrom(padding: padding),
          child: buttonContent,
        ),
      AppButtonVariant.outlined => OutlinedButton(
          onPressed: isLoading ? null : onPressed,
          style: OutlinedButton.styleFrom(padding: padding),
          child: buttonContent,
        ),
      AppButtonVariant.text => TextButton(
          onPressed: isLoading ? null : onPressed,
          style: TextButton.styleFrom(padding: padding),
          child: buttonContent,
        ),
    };
  }
  
  Color _getContentColor(BuildContext context) {
    return switch (variant) {
      AppButtonVariant.primary => Theme.of(context).colorScheme.onPrimary,
      _ => Theme.of(context).colorScheme.primary,
    };
  }
}

// Atom: AppInput
class AppInput extends StatelessWidget {
  final String? label;
  final String? hint;
  final String? error;
  final String? helperText;
  final TextEditingController? controller;
  final ValueChanged<String>? onChanged;
  final TextInputType keyboardType;
  final bool obscureText;
  final IconData? prefixIcon;
  final Widget? suffixWidget;
  final bool enabled;
  final int maxLines;
  final String? Function(String?)? validator;
  
  const AppInput({
    super.key,
    this.label,
    this.hint,
    this.error,
    this.helperText,
    this.controller,
    this.onChanged,
    this.keyboardType = TextInputType.text,
    this.obscureText = false,
    this.prefixIcon,
    this.suffixWidget,
    this.enabled = true,
    this.maxLines = 1,
    this.validator,
  });
  
  @override
  Widget build(BuildContext context) {
    return TextFormField(
      controller: controller,
      onChanged: onChanged,
      keyboardType: keyboardType,
      obscureText: obscureText,
      enabled: enabled,
      maxLines: obscureText ? 1 : maxLines,
      validator: validator,
      decoration: InputDecoration(
        labelText: label,
        hintText: hint,
        errorText: error,
        helperText: helperText,
        prefixIcon: prefixIcon != null ? Icon(prefixIcon) : null,
        suffix: suffixWidget,
        border: const OutlineInputBorder(),
      ),
    );
  }
}
```

### Molecules

```dart
// lib/shared/design_system/molecules/form_field_molecule.dart
class FormFieldMolecule extends StatelessWidget {
  final String label;
  final String? hint;
  final bool isRequired;
  final TextEditingController controller;
  final String? Function(String?) validator;
  
  const FormFieldMolecule({
    super.key,
    required this.label,
    this.hint,
    this.isRequired = false,
    required this.controller,
    required this.validator,
  });
  
  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        RichText(
          text: TextSpan(
            text: label,
            style: Theme.of(context).textTheme.labelMedium,
            children: [
              if (isRequired)
                TextSpan(
                  text: ' *',
                  style: TextStyle(
                    color: Theme.of(context).colorScheme.error,
                  ),
                ),
            ],
          ),
        ),
        const SizedBox(height: 4),
        AppInput(
          hint: hint,
          controller: controller,
          validator: validator,
        ),
      ],
    );
  }
}

// Molecule: ProductPrice
class ProductPriceMolecule extends StatelessWidget {
  final double price;
  final double? originalPrice;
  final String currency;
  
  const ProductPriceMolecule({
    super.key,
    required this.price,
    this.originalPrice,
    this.currency = '฿',
  });
  
  @override
  Widget build(BuildContext context) {
    final hasDiscount = originalPrice != null && originalPrice! > price;
    final discountPercent = hasDiscount
        ? ((originalPrice! - price) / originalPrice! * 100).round()
        : 0;
    
    return Row(
      children: [
        Text(
          '$currency${_formatPrice(price)}',
          style: Theme.of(context).textTheme.titleLarge?.copyWith(
            color: hasDiscount
                ? Theme.of(context).colorScheme.error
                : null,
            fontWeight: FontWeight.bold,
          ),
        ),
        if (hasDiscount) ...[
          const SizedBox(width: 8),
          Text(
            '$currency${_formatPrice(originalPrice!)}',
            style: Theme.of(context).textTheme.bodyMedium?.copyWith(
              decoration: TextDecoration.lineThrough,
              color: Colors.grey,
            ),
          ),
          const SizedBox(width: 4),
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 4, vertical: 2),
            decoration: BoxDecoration(
              color: Theme.of(context).colorScheme.errorContainer,
              borderRadius: BorderRadius.circular(4),
            ),
            child: Text(
              '-$discountPercent%',
              style: TextStyle(
                fontSize: 11,
                color: Theme.of(context).colorScheme.onErrorContainer,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ],
      ],
    );
  }
  
  String _formatPrice(double price) {
    return price.toStringAsFixed(price.truncate() == price ? 0 : 2);
  }
}
```

### Organisms

```dart
// lib/shared/design_system/organisms/product_card_organism.dart
class ProductCardOrganism extends StatelessWidget {
  final Product product;
  final VoidCallback onTap;
  final VoidCallback onAddToCart;
  final bool isFavorite;
  final VoidCallback onToggleFavorite;
  
  const ProductCardOrganism({
    super.key,
    required this.product,
    required this.onTap,
    required this.onAddToCart,
    required this.isFavorite,
    required this.onToggleFavorite,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Product Image
            AspectRatio(
              aspectRatio: 1,
              child: Stack(
                fit: StackFit.expand,
                children: [
                  ProductImageAtom(
                    imageUrl: product.imageUrl,
                    heroTag: 'product_${product.id}',
                  ),
                  Positioned(
                    top: 8,
                    right: 8,
                    child: FavoriteButtonAtom(
                      isFavorite: isFavorite,
                      onToggle: onToggleFavorite,
                    ),
                  ),
                  if (!product.isAvailable)
                    Positioned.fill(
                      child: Container(
                        color: Colors.black45,
                        child: const Center(
                          child: Text(
                            'หมด',
                            style: TextStyle(
                              color: Colors.white,
                              fontWeight: FontWeight.bold,
                              fontSize: 18,
                            ),
                          ),
                        ),
                      ),
                    ),
                ],
              ),
            ),
            
            // Product Info
            Padding(
              padding: const EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product.name,
                    style: Theme.of(context).textTheme.titleSmall,
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  ProductPriceMolecule(
                    price: product.price,
                    originalPrice: product.originalPrice,
                  ),
                  const SizedBox(height: 8),
                  
                  // Rating
                  if (product.rating != null)
                    RatingMolecule(
                      rating: product.rating!,
                      reviewCount: product.reviewCount ?? 0,
                    ),
                  
                  const SizedBox(height: 8),
                  
                  // Add to Cart Button
                  SizedBox(
                    width: double.infinity,
                    child: AppButton(
                      label: 'เพิ่มในตะกร้า',
                      onPressed: product.isAvailable ? onAddToCart : null,
                      variant: AppButtonVariant.secondary,
                      size: AppButtonSize.small,
                      leadingIcon: Icons.shopping_cart_outlined,
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
```

---

## ขั้นตอนที่ 483: Design Systems ใน Flutter

### Design Tokens

```dart
// lib/shared/design_system/tokens/app_design_tokens.dart
class AppDesignTokens {
  final AppColorTokens colors;
  final AppSpacingTokens spacing;
  final AppTypographyTokens typography;
  final AppBorderRadiusTokens borderRadius;
  final AppElevationTokens elevation;
  final AppDurationTokens duration;
  
  const AppDesignTokens({
    required this.colors,
    required this.spacing,
    required this.typography,
    required this.borderRadius,
    required this.elevation,
    required this.duration,
  });
  
  static AppDesignTokens of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<AppDesignTokensProvider>()!
        .tokens;
  }
  
  static AppDesignTokens get light => AppDesignTokens(
    colors: AppColorTokens.light(),
    spacing: AppSpacingTokens.defaults(),
    typography: AppTypographyTokens.defaults(),
    borderRadius: AppBorderRadiusTokens.defaults(),
    elevation: AppElevationTokens.defaults(),
    duration: AppDurationTokens.defaults(),
  );
  
  static AppDesignTokens get dark => AppDesignTokens(
    colors: AppColorTokens.dark(),
    spacing: AppSpacingTokens.defaults(),
    typography: AppTypographyTokens.defaults(),
    borderRadius: AppBorderRadiusTokens.defaults(),
    elevation: AppElevationTokens.defaults(),
    duration: AppDurationTokens.defaults(),
  );
}

class AppColorTokens {
  // Brand Colors
  final Color primary;
  final Color primaryVariant;
  final Color secondary;
  final Color accent;
  
  // Semantic Colors
  final Color success;
  final Color warning;
  final Color error;
  final Color info;
  
  // Neutral Colors
  final Color surface;
  final Color background;
  final Color onSurface;
  final Color onBackground;
  final Color divider;
  final Color disabled;
  
  const AppColorTokens._({
    required this.primary,
    required this.primaryVariant,
    required this.secondary,
    required this.accent,
    required this.success,
    required this.warning,
    required this.error,
    required this.info,
    required this.surface,
    required this.background,
    required this.onSurface,
    required this.onBackground,
    required this.divider,
    required this.disabled,
  });
  
  factory AppColorTokens.light() => const AppColorTokens._(
    primary: Color(0xFF1A73E8),
    primaryVariant: Color(0xFF1557B0),
    secondary: Color(0xFF34A853),
    accent: Color(0xFFFF6D00),
    success: Color(0xFF34A853),
    warning: Color(0xFFFBBC04),
    error: Color(0xFFEA4335),
    info: Color(0xFF1A73E8),
    surface: Color(0xFFFFFFFF),
    background: Color(0xFFF8F9FA),
    onSurface: Color(0xFF202124),
    onBackground: Color(0xFF202124),
    divider: Color(0xFFE8EAED),
    disabled: Color(0xFF9AA0A6),
  );
  
  factory AppColorTokens.dark() => const AppColorTokens._(
    primary: Color(0xFF8AB4F8),
    primaryVariant: Color(0xFF6B9FE4),
    secondary: Color(0xFF81C995),
    accent: Color(0xFFFFB74D),
    success: Color(0xFF81C995),
    warning: Color(0xFFFDD663),
    error: Color(0xFFF28B82),
    info: Color(0xFF8AB4F8),
    surface: Color(0xFF292A2D),
    background: Color(0xFF202124),
    onSurface: Color(0xFFE8EAED),
    onBackground: Color(0xFFE8EAED),
    divider: Color(0xFF3C4043),
    disabled: Color(0xFF5F6368),
  );
}

class AppSpacingTokens {
  final double xs;  // 4
  final double sm;  // 8
  final double md;  // 16
  final double lg;  // 24
  final double xl;  // 32
  final double xxl; // 48
  final double xxxl; // 64
  
  const AppSpacingTokens._({
    required this.xs,
    required this.sm,
    required this.md,
    required this.lg,
    required this.xl,
    required this.xxl,
    required this.xxxl,
  });
  
  factory AppSpacingTokens.defaults() => const AppSpacingTokens._(
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
    xxl: 48,
    xxxl: 64,
  );
}

// Design System Provider
class AppDesignTokensProvider extends InheritedWidget {
  final AppDesignTokens tokens;
  
  const AppDesignTokensProvider({
    super.key,
    required this.tokens,
    required super.child,
  });
  
  @override
  bool updateShouldNotify(AppDesignTokensProvider oldWidget) {
    return tokens != oldWidget.tokens;
  }
}
```

---

## ขั้นตอนที่ 484: Plugin Architecture

### Plugin System

```dart
// lib/core/plugins/plugin_manager.dart
abstract class AppPlugin {
  String get id;
  String get name;
  String get version;
  
  Future<void> install();
  Future<void> activate();
  Future<void> deactivate();
  Future<void> uninstall();
  
  // Extension points
  List<GoRoute> get additionalRoutes => [];
  List<NavigationItem> get additionalNavItems => [];
  Widget? buildSettingsPage() => null;
}

class PluginManager {
  final Map<String, AppPlugin> _plugins = {};
  final Map<String, bool> _activeStates = {};
  
  Future<void> install(AppPlugin plugin) async {
    await plugin.install();
    _plugins[plugin.id] = plugin;
    _activeStates[plugin.id] = false;
  }
  
  Future<void> activate(String pluginId) async {
    final plugin = _plugins[pluginId];
    if (plugin == null) throw PluginNotFoundException(pluginId);
    
    await plugin.activate();
    _activeStates[pluginId] = true;
  }
  
  Future<void> deactivate(String pluginId) async {
    final plugin = _plugins[pluginId];
    if (plugin == null) throw PluginNotFoundException(pluginId);
    
    await plugin.deactivate();
    _activeStates[pluginId] = false;
  }
  
  bool isActive(String pluginId) => _activeStates[pluginId] ?? false;
  
  List<AppPlugin> get activePlugins {
    return _plugins.entries
        .where((e) => _activeStates[e.key] ?? false)
        .map((e) => e.value)
        .toList();
  }
  
  List<GoRoute> get allAdditionalRoutes {
    return activePlugins.expand((p) => p.additionalRoutes).toList();
  }
  
  List<NavigationItem> get allAdditionalNavItems {
    return activePlugins.expand((p) => p.additionalNavItems).toList();
  }
}
```

---

## ขั้นตอนที่ 485: Hot Configuration & Feature Flags

### Firebase Remote Config

```dart
// lib/features/feature_flags/remote_config_service.dart
import 'package:firebase_remote_config/firebase_remote_config.dart';

class RemoteConfigService {
  late final FirebaseRemoteConfig _remoteConfig;
  
  Future<void> initialize() async {
    _remoteConfig = FirebaseRemoteConfig.instance;
    
    await _remoteConfig.setConfigSettings(
      RemoteConfigSettings(
        fetchTimeout: const Duration(minutes: 1),
        minimumFetchInterval: kDebugMode
            ? Duration.zero
            : const Duration(hours: 1),
      ),
    );
    
    // Default values
    await _remoteConfig.setDefaults({
      'enable_new_checkout': false,
      'enable_ai_recommendations': false,
      'max_cart_items': 50,
      'app_maintenance_mode': false,
      'promo_banner_text': '',
    });
    
    // Fetch and activate
    await _remoteConfig.fetchAndActivate();
    
    // Listen for real-time updates
    _remoteConfig.onConfigUpdated.listen((event) async {
      await _remoteConfig.activate();
    });
  }
  
  bool getBool(String key) => _remoteConfig.getBool(key);
  String getString(String key) => _remoteConfig.getString(key);
  int getInt(String key) => _remoteConfig.getInt(key);
  double getDouble(String key) => _remoteConfig.getDouble(key);
  
  // Feature flags
  bool get isNewCheckoutEnabled => getBool('enable_new_checkout');
  bool get isAIRecommendationsEnabled => getBool('enable_ai_recommendations');
  bool get isMaintenanceMode => getBool('app_maintenance_mode');
  int get maxCartItems => getInt('max_cart_items');
  String get promoBannerText => getString('promo_banner_text');
}

// Feature Flag Widget
class FeatureGate extends ConsumerWidget {
  final String featureFlag;
  final Widget child;
  final Widget? fallback;
  
  const FeatureGate({
    super.key,
    required this.featureFlag,
    required this.child,
    this.fallback,
  });
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final config = ref.watch(remoteConfigProvider);
    final isEnabled = config.getBool(featureFlag);
    
    if (isEnabled) return child;
    return fallback ?? const SizedBox.shrink();
  }
}

// ใช้งาน
FeatureGate(
  featureFlag: 'enable_new_checkout',
  child: const NewCheckoutScreen(),
  fallback: const OldCheckoutScreen(),
)
```

---

## ขั้นตอนที่ 486: A/B Testing

### Firebase A/B Testing

```dart
// lib/features/ab_testing/ab_test_service.dart
class ABTestService {
  final RemoteConfigService _remoteConfig;
  final AnalyticsService _analytics;
  
  ABTestService(this._remoteConfig, this._analytics);
  
  String getVariant(String experimentName) {
    return _remoteConfig.getString('experiment_$experimentName');
  }
  
  Widget runExperiment({
    required String experimentName,
    required Map<String, Widget> variants,
    required Widget control,
  }) {
    final variant = getVariant(experimentName);
    
    // Track impression
    _analytics.trackEvent('experiment_impression', {
      'experiment': experimentName,
      'variant': variant.isEmpty ? 'control' : variant,
    });
    
    if (variant.isEmpty) return control;
    return variants[variant] ?? control;
  }
}

// Provider
@riverpod
ABTestService abTestService(ABTestServiceRef ref) {
  return ABTestService(
    ref.watch(remoteConfigProvider),
    ref.watch(analyticsServiceProvider),
  );
}

// Widget Usage
class HomeBannerWidget extends ConsumerWidget {
  const HomeBannerWidget({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final abTest = ref.watch(abTestServiceProvider);
    
    return abTest.runExperiment(
      experimentName: 'home_banner_style',
      variants: {
        'variant_a': const ImageBanner(),
        'variant_b': const VideoAutplayBanner(),
      },
      control: const TextBanner(),
    );
  }
}
```

---

## ขั้นตอนที่ 487: Analytics Abstraction Layer

### Analytics Abstraction

```dart
// lib/analytics/analytics_service.dart
abstract class AnalyticsProvider {
  Future<void> initialize();
  Future<void> track(String event, Map<String, dynamic>? properties);
  Future<void> identify(String userId, Map<String, dynamic>? traits);
  Future<void> page(String screenName, Map<String, dynamic>? properties);
  Future<void> reset();
}

class FirebaseAnalyticsProvider implements AnalyticsProvider {
  final FirebaseAnalytics _analytics = FirebaseAnalytics.instance;
  
  @override
  Future<void> initialize() async { }
  
  @override
  Future<void> track(String event, Map<String, dynamic>? properties) async {
    await _analytics.logEvent(
      name: _sanitizeEventName(event),
      parameters: properties?.map(
        (key, value) => MapEntry(key, value?.toString() ?? ''),
      ),
    );
  }
  
  @override
  Future<void> identify(String userId, Map<String, dynamic>? traits) async {
    await _analytics.setUserId(id: userId);
    if (traits != null) {
      for (final entry in traits.entries) {
        await _analytics.setUserProperty(
          name: entry.key,
          value: entry.value?.toString(),
        );
      }
    }
  }
  
  @override
  Future<void> page(String screenName, Map<String, dynamic>? properties) async {
    await _analytics.logScreenView(screenName: screenName);
  }
  
  @override
  Future<void> reset() async {
    await _analytics.setUserId(id: null);
  }
  
  String _sanitizeEventName(String name) {
    return name.replaceAll(RegExp(r'[^a-zA-Z0-9_]'), '_').substring(0, min(40, name.length));
  }
}

class MixpanelProvider implements AnalyticsProvider {
  // Mixpanel implementation
  @override Future<void> initialize() async { }
  @override Future<void> track(String event, Map<String, dynamic>? properties) async { }
  @override Future<void> identify(String userId, Map<String, dynamic>? traits) async { }
  @override Future<void> page(String screenName, Map<String, dynamic>? properties) async { }
  @override Future<void> reset() async { }
}

// Multi-provider Analytics
class AnalyticsService {
  final List<AnalyticsProvider> _providers;
  
  AnalyticsService(this._providers);
  
  Future<void> initialize() async {
    await Future.wait(_providers.map((p) => p.initialize()));
  }
  
  Future<void> trackEvent(
    String event, [
    Map<String, dynamic>? properties,
  ]) async {
    await Future.wait(
      _providers.map((p) => p.track(event, properties)),
    );
  }
  
  Future<void> identify(
    String userId, [
    Map<String, dynamic>? traits,
  ]) async {
    await Future.wait(
      _providers.map((p) => p.identify(userId, traits)),
    );
  }
  
  Future<void> trackScreen(
    String screenName, [
    Map<String, dynamic>? properties,
  ]) async {
    await Future.wait(
      _providers.map((p) => p.page(screenName, properties)),
    );
  }
  
  Future<void> reset() async {
    await Future.wait(_providers.map((p) => p.reset()));
  }
  
  // Predefined events for type safety
  Future<void> trackProductViewed(Product product) async {
    await trackEvent('product_viewed', {
      'product_id': product.id,
      'product_name': product.name,
      'product_category': product.category,
      'price': product.price,
    });
  }
  
  Future<void> trackAddToCart(Product product, int quantity) async {
    await trackEvent('add_to_cart', {
      'product_id': product.id,
      'product_name': product.name,
      'quantity': quantity,
      'total_value': product.price * quantity,
    });
  }
  
  Future<void> trackCheckoutStarted(double cartTotal) async {
    await trackEvent('checkout_started', {
      'cart_total': cartTotal,
    });
  }
  
  Future<void> trackOrderCompleted({
    required String orderId,
    required double total,
    required List<String> productIds,
  }) async {
    await trackEvent('order_completed', {
      'order_id': orderId,
      'total': total,
      'product_count': productIds.length,
    });
  }
}

// Analytics Observer for Router
class AnalyticsRouterObserver extends NavigatorObserver {
  final AnalyticsService _analytics;
  
  AnalyticsRouterObserver(this._analytics);
  
  @override
  void didPush(Route route, Route? previousRoute) {
    _trackScreen(route);
  }
  
  @override
  void didReplace({Route? newRoute, Route? oldRoute}) {
    if (newRoute != null) _trackScreen(newRoute);
  }
  
  void _trackScreen(Route route) {
    final name = route.settings.name;
    if (name != null) {
      _analytics.trackScreen(name);
    }
  }
}
```

---

## ขั้นตอนที่ 488: App Health Monitoring

### App Performance Monitoring

```dart
// lib/monitoring/health_monitor.dart
class AppHealthMonitor {
  final FirebasePerformance _performance = FirebasePerformance.instance;
  final FirebaseCrashlytics _crashlytics = FirebaseCrashlytics.instance;
  
  // Custom traces
  Trace startTrace(String name) {
    return _performance.newTrace(name);
  }
  
  Future<T> measureOperation<T>(
    String operationName,
    Future<T> Function() operation, {
    Map<String, String>? attributes,
  }) async {
    final trace = _performance.newTrace(operationName);
    
    if (attributes != null) {
      attributes.forEach(trace.putAttribute);
    }
    
    await trace.start();
    
    try {
      final result = await operation();
      trace.putAttribute('success', 'true');
      return result;
    } catch (e, stack) {
      trace.putAttribute('success', 'false');
      trace.putAttribute('error', e.toString().substring(0, min(100, e.toString().length)));
      await _crashlytics.recordError(e, stack, fatal: false);
      rethrow;
    } finally {
      await trace.stop();
    }
  }
  
  // HTTP Monitoring
  HttpMetric createHttpMetric(String url, HttpMethod method) {
    return _performance.newHttpMetric(url, method);
  }
  
  // Crashlytics
  Future<void> setCrashlyticsUser(String userId) async {
    await _crashlytics.setUserIdentifier(userId);
  }
  
  Future<void> log(String message) async {
    await _crashlytics.log(message);
  }
  
  Future<void> recordError(
    dynamic exception,
    StackTrace? stack, {
    bool fatal = false,
    Map<String, dynamic>? context,
  }) async {
    if (context != null) {
      for (final entry in context.entries) {
        await _crashlytics.setCustomKey(entry.key, entry.value.toString());
      }
    }
    
    await _crashlytics.recordError(exception, stack, fatal: fatal);
  }
}

// Health Check Widget
class AppHealthStatus extends ConsumerWidget {
  const AppHealthStatus({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final healthAsync = ref.watch(appHealthProvider);
    
    return healthAsync.when(
      data: (health) => _buildStatus(context, health),
      loading: () => const SizedBox.shrink(),
      error: (_, __) => const SizedBox.shrink(),
    );
  }
  
  Widget _buildStatus(BuildContext context, AppHealth health) {
    if (health.isHealthy) return const SizedBox.shrink();
    
    return MaterialBanner(
      backgroundColor: Colors.orange,
      content: Text(health.message),
      actions: [
        TextButton(
          onPressed: () => {},
          child: const Text('ปิด', style: TextStyle(color: Colors.white)),
        ),
      ],
    );
  }
}

class AppHealth {
  final bool isHealthy;
  final String message;
  final DateTime checkedAt;
  
  const AppHealth({
    required this.isHealthy,
    required this.message,
    required this.checkedAt,
  });
}
```

---

## ขั้นตอนที่ 489: Performance Profiling

### Flutter Performance Best Practices

```dart
// lib/utils/performance_utils.dart
class PerformanceUtils {
  // Use const constructors
  static const Widget constWidget = SizedBox.shrink(); // ✅
  
  // Avoid rebuilding expensive widgets
  static Widget buildCachedWidget(
    String key,
    Widget Function() builder,
  ) {
    return Builder(
      builder: (context) => builder(),
    );
  }
  
  // Image optimization
  static Widget buildOptimizedImage(
    String url, {
    double? width,
    double? height,
    BoxFit fit = BoxFit.cover,
  }) {
    return CachedNetworkImage(
      imageUrl: url,
      width: width,
      height: height,
      fit: fit,
      memCacheWidth: width?.toInt(),
      memCacheHeight: height?.toInt(),
      placeholder: (context, url) => const ShimmerWidget(),
      errorWidget: (context, url, error) => const Icon(Icons.error),
    );
  }
}

// Sliver-based list for performance
class PerformantProductGrid extends StatelessWidget {
  final List<Product> products;
  
  const PerformantProductGrid({super.key, required this.products});
  
  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: [
        SliverPadding(
          padding: const EdgeInsets.all(16),
          sliver: SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              childAspectRatio: 0.75,
              crossAxisSpacing: 12,
              mainAxisSpacing: 12,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return RepaintBoundary(
                  key: ValueKey(products[index].id),
                  child: ProductCardOrganism(
                    product: products[index],
                    onTap: () {},
                    onAddToCart: () {},
                    isFavorite: false,
                    onToggleFavorite: () {},
                  ),
                );
              },
              childCount: products.length,
            ),
          ),
        ),
      ],
    );
  }
}

// Lazy loading with pagination
class InfiniteScrollList<T> extends ConsumerStatefulWidget {
  final AsyncValue<List<T>> items;
  final Widget Function(T item) itemBuilder;
  final Future<void> Function() onLoadMore;
  final bool hasMore;
  final Widget? emptyWidget;
  
  const InfiniteScrollList({
    super.key,
    required this.items,
    required this.itemBuilder,
    required this.onLoadMore,
    required this.hasMore,
    this.emptyWidget,
  });
  
  @override
  ConsumerState<InfiniteScrollList<T>> createState() =>
      _InfiniteScrollListState<T>();
}

class _InfiniteScrollListState<T>
    extends ConsumerState<InfiniteScrollList<T>> {
  final ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      if (widget.hasMore) {
        widget.onLoadMore();
      }
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return widget.items.when(
      data: (items) {
        if (items.isEmpty) {
          return widget.emptyWidget ?? const Center(child: Text('ไม่มีข้อมูล'));
        }
        
        return ListView.builder(
          controller: _scrollController,
          itemCount: items.length + (widget.hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == items.length) {
              return const Center(
                child: Padding(
                  padding: EdgeInsets.all(16),
                  child: CircularProgressIndicator(),
                ),
              );
            }
            return widget.itemBuilder(items[index]);
          },
        );
      },
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(child: Text('Error: $error')),
    );
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 490: Workshop - Scalable Architecture Demo

### Complete Scalable App

```dart
// lib/main.dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  
  await configureDependencies();
  await getIt<RemoteConfigService>().initialize();
  await getIt<AnalyticsService>().initialize();
  
  AppFeatures.register();
  
  runApp(
    ProviderScope(
      overrides: [
        // Dependency overrides for testing
      ],
      child: const ScalableApp(),
    ),
  );
}

class ScalableApp extends ConsumerWidget {
  const ScalableApp({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final themeMode = ref.watch(themeModeProvider);
    final locale = ref.watch(localeProvider);
    
    return AppDesignTokensProvider(
      tokens: themeMode == ThemeMode.dark
          ? AppDesignTokens.dark
          : AppDesignTokens.light,
      child: MaterialApp.router(
        title: 'Scalable Flutter App',
        theme: buildLightTheme(),
        darkTheme: buildDarkTheme(),
        themeMode: themeMode,
        locale: locale,
        routerConfig: ref.watch(appRouterProvider).router,
        localizationsDelegates: AppLocalizations.localizationsDelegates,
        supportedLocales: AppLocalizations.supportedLocales,
        builder: (context, child) {
          return AppHealthStatus(
            child: child ?? const SizedBox.shrink(),
          );
        },
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **Feature-first Structure**: โครงสร้างที่ scale ได้
- **Atomic Design**: สร้าง widget library อย่างเป็นระบบ
- **Design Systems**: Tokens, Components, Documentation
- **Plugin Architecture**: ขยาย app โดยไม่แก้ core
- **Feature Flags**: Remote config และ feature gates
- **A/B Testing**: ทดสอบ variant ของ UI
- **Analytics Abstraction**: ใช้หลาย analytics providers
- **Health Monitoring**: Performance tracking, Crashlytics

## แบบฝึกหัด

1. สร้าง feature-first folder structure ที่สมบูรณ์
2. สร้าง Design System ที่มี Tokens, Atoms, Molecules, Organisms
3. Implement Feature Flag system ด้วย Firebase Remote Config
4. เพิ่ม Analytics abstraction ที่รองรับ Firebase + Mixpanel
5. ตั้งค่า Performance monitoring ด้วย Firebase Performance

---

[⬅️ Part 48](part_48.md) | [Part 50 ➡️](part_50.md)

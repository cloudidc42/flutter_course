# Part 35: Accessibility
## ขั้นตอนที่ 341-350

---

## สารบัญ
1. [Accessibility คืออะไร?](#accessibility-คืออะไร)
2. [Semantics Widget](#semantics-widget)
3. [MergeSemantics & ExcludeSemantics](#mergesemantics--excludesemantics)
4. [Semantic Labels](#semantic-labels)
5. [Touch Targets (48x48)](#touch-targets)
6. [Color Contrast Requirements](#color-contrast)
7. [Screen Reader Support](#screen-reader-support)
8. [Dynamic Text Sizes](#dynamic-text-sizes)
9. [Focus Management](#focus-management)
10. [Accessibility Testing](#accessibility-testing)

---

## ขั้นตอนที่ 341: Accessibility คืออะไร?

Accessibility (a11y) คือการออกแบบแอปให้ผู้ใช้ทุกคนสามารถใช้งานได้ รวมถึงผู้พิการ

### กลุ่มผู้ใช้ที่ต้องคำนึงถึง

```
ประเภทความพิการที่ต้องคำนึงถึง:

👁️ Visual Impairment:
  - ตาบอด → ใช้ Screen Reader (TalkBack/VoiceOver)
  - ตาพร่ามัว → ใช้ Large Text, High Contrast
  - ตาบอดสี → อย่าใช้สีเพียงอย่างเดียวเพื่อสื่อความหมาย

🤚 Motor Impairment:
  - ใช้มือไม่ได้เต็มที่ → Touch Target ต้องใหญ่พอ
  - Switch Access
  - Voice Control

👂 Hearing Impairment:
  - ใส่ Captions/Subtitles สำหรับ Video/Audio

🧠 Cognitive Impairment:
  - Simple language
  - Clear navigation
  - Consistent patterns
```

### WCAG Guidelines

```
WCAG 2.1 มี 4 หลักการ (POUR):

P - Perceivable  (รับรู้ได้)
  - ทุกข้อมูลต้องรับรู้ได้โดยไม่ขึ้นอยู่กับ sense เดียว
  - Alt text สำหรับ Images

O - Operable  (ใช้งานได้)
  - ทุกฟังก์ชันต้องใช้ keyboard ได้
  - No time limits ที่ไม่จำเป็น

U - Understandable  (เข้าใจได้)
  - ภาษาที่ชัดเจน
  - Predictable behavior

R - Robust  (แข็งแกร่ง)
  - ทำงานได้กับ assistive technologies
```

---

## ขั้นตอนที่ 342: Semantics Widget

`Semantics` widget เพิ่ม metadata ให้ screen readers

```dart
// ❌ Icon ไม่มี semantic information
IconButton(
  icon: const Icon(Icons.delete),
  onPressed: () => deleteItem(item),
)

// ✅ เพิ่ม Semantics label
IconButton(
  icon: const Icon(Icons.delete),
  onPressed: () => deleteItem(item),
  tooltip: 'Delete item',  // แสดงเป็น tooltip และ semantic
)

// หรือใช้ Semantics widget โดยตรง
Semantics(
  label: 'Delete ${item.name}',
  hint: 'Double tap to delete this item',
  button: true,
  onTap: () => deleteItem(item),
  child: IconButton(
    icon: const Icon(Icons.delete),
    onPressed: () => deleteItem(item),
  ),
)
```

### Semantics Properties

```dart
Semantics(
  // Text ที่ Screen Reader จะอ่าน
  label: 'Add to cart button',
  
  // คำใบ้สำหรับ action
  hint: 'Double tap to add product to your cart',
  
  // State information
  checked: isChecked,           // checkbox state
  selected: isSelected,         // selected state
  enabled: isEnabled,           // enabled/disabled
  
  // Type hints
  button: true,                 // เป็น button
  link: false,                  // เป็น link
  image: false,                 // เป็น image
  header: false,                // เป็น header
  
  // Value (สำหรับ slider, progress bar)
  value: '${(progress * 100).round()}%',
  increasedValue: '${((progress + 0.1) * 100).round()}%',
  decreasedValue: '${((progress - 0.1) * 100).round()}%',
  
  // Actions
  onTap: handleTap,
  onLongPress: handleLongPress,
  onScrollUp: handleScrollUp,
  onScrollDown: handleScrollDown,
  
  // เพื่อเพิ่ม/ลด value (slider)
  onIncrease: handleIncrease,
  onDecrease: handleDecrease,
  
  child: YourWidget(),
)
```

### ตัวอย่างจริง

```dart
// Rating Widget ที่ accessible
class AccessibleRating extends StatelessWidget {
  final double rating;
  final int maxRating;
  final ValueChanged<int>? onRatingChanged;

  const AccessibleRating({
    super.key,
    required this.rating,
    this.maxRating = 5,
    this.onRatingChanged,
  });

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: 'Rating',
      value: '${rating.toStringAsFixed(1)} out of $maxRating stars',
      hint: onRatingChanged != null ? 'Swipe to change rating' : null,
      slider: onRatingChanged != null,
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: List.generate(maxRating, (index) {
          final starValue = index + 1;
          return Semantics(
            // ExcludeSemantics หรือ label สำหรับแต่ละดาว
            label: '$starValue star${starValue == 1 ? '' : 's'}',
            button: onRatingChanged != null,
            selected: starValue <= rating,
            child: GestureDetector(
              onTap: () => onRatingChanged?.call(starValue),
              child: Icon(
                starValue <= rating ? Icons.star : Icons.star_border,
                color: Colors.amber,
                size: 24,
              ),
            ),
          );
        }),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 343: MergeSemantics & ExcludeSemantics

```dart
// MergeSemantics รวม Semantics ของ children เข้าด้วยกัน
// ทำให้ Screen Reader อ่านทั้งหมดในครั้งเดียว

// ❌ ปัญหา: Screen Reader อ่านแยก 3 ครั้ง
Row(
  children: [
    Icon(Icons.favorite),
    Text('42'),
    Text('likes'),
  ],
)

// ✅ MergeSemantics ทำให้อ่านครั้งเดียว: "42 likes"
MergeSemantics(
  child: Row(
    children: [
      const ExcludeSemantics(
        child: Icon(Icons.favorite),  // Icon ไม่ต้องการ semantic แยก
      ),
      const Text('42'),
      const Text(' likes'),
    ],
  ),
)
```

```dart
// ตัวอย่าง Product Card ที่ดี
class AccessibleProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback onTap;

  const AccessibleProductCard({
    super.key,
    required this.product,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Semantics(
      // รวม info ทั้งหมดไว้ใน label เดียว
      label: '${product.name}, '
             'price ${product.price} baht, '
             '${product.isInStock ? "in stock" : "out of stock"}',
      button: true,
      onTap: onTap,
      child: ExcludeSemantics(
        // ซ่อน semantics ของ children ที่ไม่ต้องการ
        child: GestureDetector(
          onTap: onTap,
          child: Card(
            child: Column(
              children: [
                Image.network(product.imageUrl),
                Text(product.name),
                Text('฿${product.price}'),
                if (product.isInStock)
                  const Text('In Stock', style: TextStyle(color: Colors.green))
                else
                  const Text('Out of Stock', style: TextStyle(color: Colors.red)),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

### ExcludeSemantics

```dart
// ใช้ ExcludeSemantics เพื่อซ่อน decorative elements
Column(
  children: [
    // Decorative icon - ไม่ต้องการ semantic
    const ExcludeSemantics(
      child: Icon(Icons.star, color: Colors.amber),
    ),
    
    // Text ที่ต้องการ semantic
    const Text('Premium Product'),
    
    // Background decoration
    const ExcludeSemantics(
      child: DecorativeBackground(),
    ),
  ],
)
```

---

## ขั้นตอนที่ 344: Semantic Labels

```dart
// Image Semantics
Image.network(
  product.imageUrl,
  semanticLabel: 'Product image of ${product.name}', // ✅
)

// หรือใช้ Semantics widget
Semantics(
  label: 'Product image of ${product.name}',
  image: true,
  child: Image.network(product.imageUrl),
)

// Container/Icon Button
Semantics(
  label: 'Close dialog',
  button: true,
  child: IconButton(
    icon: const Icon(Icons.close),
    onPressed: () => Navigator.pop(context),
  ),
)

// Checkbox
Semantics(
  label: 'Remember me',
  checked: rememberMe,
  child: Checkbox(
    value: rememberMe,
    onChanged: (val) => setState(() => rememberMe = val ?? false),
  ),
)

// Switch
SwitchListTile(
  title: const Text('Dark Mode'),
  value: isDarkMode,
  onChanged: toggleDarkMode,
  // SwitchListTile จัดการ Semantics อัตโนมัติ
)
```

### Live Region Semantics

```dart
// Live Region - แจ้ง Screen Reader เมื่อ content เปลี่ยน
class StatusMessage extends StatelessWidget {
  final String message;
  
  const StatusMessage({super.key, required this.message});

  @override
  Widget build(BuildContext context) {
    return Semantics(
      liveRegion: true,       // แจ้ง Screen Reader ทันทีเมื่อ text เปลี่ยน
      child: Text(message),
    );
  }
}

// ใช้งาน
StatusMessage(
  message: isLoading ? 'Loading products...' : 'Products loaded',
)
```

---

## ขั้นตอนที่ 345: Touch Targets (48x48)

WCAG กำหนดว่า touch target ต้องมีขนาดอย่างน้อย 48x48 logical pixels

```dart
// ❌ Touch target เล็กเกินไป
IconButton(
  iconSize: 16,  // ❌ เล็กมาก
  icon: const Icon(Icons.delete),
  onPressed: deleteItem,
)

// ✅ ขนาดที่เหมาะสม (default 48x48)
IconButton(
  icon: const Icon(Icons.delete),
  onPressed: deleteItem,
  // iconSize default = 24, padding = 8, total = 48x48 ✅
)

// สำหรับ Custom Touch Targets
class MinimumTouchTarget extends StatelessWidget {
  final Widget child;
  final VoidCallback? onTap;
  static const double kMinimumSize = 48.0;

  const MinimumTouchTarget({
    super.key,
    required this.child,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: ConstrainedBox(
        constraints: const BoxConstraints(
          minWidth: kMinimumSize,
          minHeight: kMinimumSize,
        ),
        child: Center(child: child),
      ),
    );
  }
}
```

```dart
// ตรวจสอบขนาด Touch Target อัตโนมัติ
class CheckboxWithLabel extends StatelessWidget {
  final bool value;
  final String label;
  final ValueChanged<bool?> onChanged;

  const CheckboxWithLabel({
    super.key,
    required this.value,
    required this.label,
    required this.onChanged,
  });

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: () => onChanged(!value),
      // กำหนดขนาด minimum
      child: ConstrainedBox(
        constraints: const BoxConstraints(minHeight: 48),
        child: Row(
          children: [
            Checkbox(value: value, onChanged: onChanged),
            const SizedBox(width: 8),
            Expanded(child: Text(label)),
          ],
        ),
      ),
    );
  }
}
```

### Spacing ระหว่าง Touch Targets

```dart
// ❌ ปุ่มชิดกันเกินไป
Row(
  children: [
    TextButton(onPressed: cancel, child: const Text('Cancel')),
    TextButton(onPressed: confirm, child: const Text('Confirm')),
  ],
)

// ✅ มี spacing เพียงพอ
Row(
  children: [
    TextButton(onPressed: cancel, child: const Text('Cancel')),
    const SizedBox(width: 8), // ✅ space ระหว่างปุ่ม
    TextButton(onPressed: confirm, child: const Text('Confirm')),
  ],
)
```

---

## ขั้นตอนที่ 346: Color Contrast Requirements

WCAG กำหนด Contrast Ratio:
- ข้อความปกติ: อย่างน้อย 4.5:1
- ข้อความใหญ่ (18pt+): อย่างน้อย 3:1
- UI Components: อย่างน้อย 3:1

```dart
// ❌ Low contrast - ยากต่อการอ่าน
Text(
  'Please enter your email',
  style: TextStyle(
    color: Colors.grey[400], // Contrast ratio กับ white background ประมาณ 1.9:1 ❌
  ),
)

// ✅ High contrast
Text(
  'Please enter your email',
  style: TextStyle(
    color: Colors.grey[700], // Contrast ratio กับ white background ประมาณ 5.9:1 ✅
  ),
)
```

```dart
// Helper function ตรวจสอบ Contrast
double calculateContrastRatio(Color foreground, Color background) {
  final fgLuminance = foreground.computeLuminance();
  final bgLuminance = background.computeLuminance();
  
  final lighter = math.max(fgLuminance, bgLuminance);
  final darker = math.min(fgLuminance, bgLuminance);
  
  return (lighter + 0.05) / (darker + 0.05);
}

bool meetsWCAGAA(Color foreground, Color background, {bool largeText = false}) {
  final ratio = calculateContrastRatio(foreground, background);
  return largeText ? ratio >= 3.0 : ratio >= 4.5;
}

// ใช้งาน
void checkContrast() {
  final textColor = const Color(0xFF6B7280); // Gray-500
  final bgColor = Colors.white;
  
  final ratio = calculateContrastRatio(textColor, bgColor);
  final passes = meetsWCAGAA(textColor, bgColor);
  
  print('Contrast ratio: ${ratio.toStringAsFixed(2)}:1');
  print('Passes WCAG AA: $passes');
}
```

### Error Messages ที่ไม่พึ่งสีอย่างเดียว

```dart
// ❌ ใช้สีแดงอย่างเดียว - ผู้ใช้ตาบอดสีอาจไม่เห็น
Text(
  hasError ? 'Email is invalid' : '',
  style: const TextStyle(color: Colors.red),
)

// ✅ ใช้สี + icon + text
if (hasError)
  Row(
    children: const [
      Icon(Icons.error_outline, color: Colors.red, size: 16),
      SizedBox(width: 4),
      Text(
        'Email is invalid',
        style: TextStyle(color: Colors.red),
      ),
    ],
  )
```

---

## ขั้นตอนที่ 347: Screen Reader Support

### TalkBack (Android) & VoiceOver (iOS)

```dart
// ทดสอบ Screen Reader
// Android: Settings > Accessibility > TalkBack
// iOS: Settings > Accessibility > VoiceOver

// หรือใช้ Keyboard Shortcut:
// Android: Volume Up + Down พร้อมกัน
// iOS: Triple click Home/Side button
```

```dart
// Custom Action สำหรับ Screen Reader
Semantics(
  customSemanticsActions: {
    const CustomSemanticsAction(label: 'Add to favorites'): () {
      addToFavorites(product);
    },
    const CustomSemanticsAction(label: 'Share product'): () {
      shareProduct(product);
    },
  },
  child: ProductCard(product: product),
)
```

### Announcement สำหรับ Screen Reader

```dart
// ประกาศข้อความให้ Screen Reader อ่าน
Future<void> announceForAccessibility(BuildContext context, String message) async {
  await SemanticsService.announce(
    message,
    TextDirection.ltr,
    assertiveness: Assertiveness.polite, // หรือ assertive
  );
}

// ใช้งาน
Future<void> _onAddToCart() async {
  await addToCart(product);
  if (mounted) {
    await announceForAccessibility(
      context,
      '${product.name} added to cart',
    );
  }
}
```

### Accessible Navigation

```dart
// ใส่ Semantics ที่ meaningful สำหรับ Navigation
BottomNavigationBar(
  items: [
    BottomNavigationBarItem(
      icon: const Icon(Icons.home),
      label: 'Home',               // ✅ label จะใช้เป็น semantic
      activeIcon: const Icon(Icons.home_filled),
      tooltip: 'Go to Home',       // ✅ tooltip เพิ่ม hint
    ),
    BottomNavigationBarItem(
      icon: const Icon(Icons.shopping_cart),
      label: 'Cart',
      tooltip: 'View shopping cart',
    ),
  ],
  currentIndex: _selectedIndex,
  onTap: _onTabTapped,
)
```

---

## ขั้นตอนที่ 348: Dynamic Text Sizes

```dart
// ✅ ใช้ TextScaler จาก MediaQuery
class AccessibleText extends StatelessWidget {
  final String text;
  final TextStyle? style;
  final double maxScaleFactor;

  const AccessibleText({
    super.key,
    required this.text,
    this.style,
    this.maxScaleFactor = 2.0,
  });

  @override
  Widget build(BuildContext context) {
    return Text(
      text,
      style: style,
      // อนุญาต text scaling แต่จำกัดไว้ที่ maxScaleFactor
      textScaler: TextScaler.linear(
        MediaQuery.textScalerOf(context).scale(1.0).clamp(1.0, maxScaleFactor),
      ),
    );
  }
}
```

```dart
// ✅ Layout ที่ accommodate large text
class FlexibleLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final textScaler = MediaQuery.textScalerOf(context);
    final isLargeText = textScaler.scale(1.0) > 1.3;

    return isLargeText
        ? Column(  // Stack vertically สำหรับ large text
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              _buildLabel(),
              const SizedBox(height: 4),
              _buildValue(),
            ],
          )
        : Row(     // Side by side สำหรับ normal text
            children: [
              _buildLabel(),
              const Spacer(),
              _buildValue(),
            ],
          );
  }
  
  Widget _buildLabel() => const Text('Price');
  Widget _buildValue() => const Text('฿999');
}
```

```dart
// ✅ Container ที่ขยายตาม text size
class AdaptiveCard extends StatelessWidget {
  final String title;
  final String description;

  const AdaptiveCard({
    super.key,
    required this.title,
    required this.description,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          // ✅ ไม่กำหนดความสูงตายตัว ให้ขยายตาม content
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              title,
              style: Theme.of(context).textTheme.titleMedium,
              // ✅ ไม่จำกัดจำนวน lines สำหรับ large text
              maxLines: null,
            ),
            const SizedBox(height: 8),
            Text(
              description,
              style: Theme.of(context).textTheme.bodyMedium,
              maxLines: null, // ✅ ไม่ overflow
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 349: Focus Management

```dart
// ✅ กำหนด Focus Order ที่ถูกต้อง
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _emailFocusNode = FocusNode();
  final _passwordFocusNode = FocusNode();
  final _loginButtonFocusNode = FocusNode();

  @override
  void dispose() {
    _emailFocusNode.dispose();
    _passwordFocusNode.dispose();
    _loginButtonFocusNode.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextFormField(
          focusNode: _emailFocusNode,
          decoration: const InputDecoration(
            labelText: 'Email',
            hintText: 'Enter your email address',
          ),
          keyboardType: TextInputType.emailAddress,
          textInputAction: TextInputAction.next,
          onFieldSubmitted: (_) => _passwordFocusNode.requestFocus(), // ✅ ย้าย focus
        ),
        
        const SizedBox(height: 16),
        
        TextFormField(
          focusNode: _passwordFocusNode,
          decoration: const InputDecoration(
            labelText: 'Password',
          ),
          obscureText: true,
          textInputAction: TextInputAction.done,
          onFieldSubmitted: (_) => _loginButtonFocusNode.requestFocus(),
        ),
        
        const SizedBox(height: 24),
        
        Focus(
          focusNode: _loginButtonFocusNode,
          child: ElevatedButton(
            onPressed: _handleLogin,
            child: const Text('Login'),
          ),
        ),
      ],
    );
  }
}
```

### Dialog Focus Trap

```dart
// ✅ Focus ควรอยู่ใน Dialog เมื่อมันเปิดอยู่
Future<void> showAccessibleDialog(BuildContext context) async {
  return showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: const Text('Confirm Delete'),
      content: const Text('Are you sure you want to delete this item?'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          autofocus: true, // ✅ Focus ที่ปุ่มแรกเมื่อ dialog เปิด
          child: const Text('Cancel'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context, true),
          child: const Text('Delete'),
        ),
      ],
    ),
  );
}
```

### FocusTraversalGroup

```dart
// ✅ กำหนดกลุ่ม Focus ที่ถูกต้อง
class AccessibleForm extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FocusTraversalGroup(
      // กำหนด traversal order
      policy: OrderedTraversalPolicy(),
      child: Column(
        children: [
          FocusTraversalOrder(
            order: const NumericFocusOrder(1),
            child: TextFormField(
              decoration: const InputDecoration(labelText: 'First Name'),
            ),
          ),
          FocusTraversalOrder(
            order: const NumericFocusOrder(2),
            child: TextFormField(
              decoration: const InputDecoration(labelText: 'Last Name'),
            ),
          ),
          FocusTraversalOrder(
            order: const NumericFocusOrder(3),
            child: TextFormField(
              decoration: const InputDecoration(labelText: 'Email'),
            ),
          ),
          FocusTraversalOrder(
            order: const NumericFocusOrder(4),
            child: ElevatedButton(
              onPressed: () {},
              child: const Text('Submit'),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 350: Accessibility Testing

### Widget Tests สำหรับ Accessibility

```dart
// test/accessibility_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('Accessibility Tests', () {
    testWidgets('ProductCard should have semantic labels', (tester) async {
      const product = Product(
        id: '1',
        name: 'iPhone 15',
        price: 999.0,
        stockQuantity: 5,
        category: 'Electronics',
        tags: [],
      );

      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: AccessibleProductCard(
              product: product,
              onTap: () {},
            ),
          ),
        ),
      );

      // ตรวจสอบ semantic label
      expect(
        tester.getSemantics(find.byType(AccessibleProductCard)),
        matchesSemantics(
          label: contains('iPhone 15'),
          isButton: true,
          hasTapAction: true,
        ),
      );
    });

    testWidgets('Button should have minimum touch target size', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: IconButton(
              icon: const Icon(Icons.delete),
              onPressed: () {},
            ),
          ),
        ),
      );

      // ตรวจสอบขนาด touch target
      final renderObject = tester.renderObject<RenderBox>(
        find.byType(IconButton),
      );
      
      expect(renderObject.size.width, greaterThanOrEqualTo(48));
      expect(renderObject.size.height, greaterThanOrEqualTo(48));
    });
  });
}
```

### Accessibility Audit Tool

```dart
// ใช้ flutter_accessibility_service package หรือ
// ตรวจสอบด้วย SemanticsDebugger

// ใน main() สำหรับ development
void main() {
  runApp(
    SemanticsDebugger(
      child: MaterialApp(
        home: MyHomePage(),
      ),
    ),
  );
}
```

### Automated Accessibility Checks

```dart
// ตรวจสอบ accessibility ใน tests
void main() {
  testWidgets('App passes accessibility checks', (tester) async {
    final SemanticsHandle handle = tester.ensureSemantics();
    
    await tester.pumpWidget(const MaterialApp(home: MyHomePage()));
    await tester.pump();
    
    // ตรวจสอบว่าไม่มี accessibility violations
    await expectLater(tester, meetsGuideline(androidTapTargetGuideline));
    await expectLater(tester, meetsGuideline(iOSTapTargetGuideline));
    await expectLater(tester, meetsGuideline(labeledTapTargetGuideline));
    await expectLater(tester, meetsGuideline(textContrastGuideline));
    
    handle.dispose();
  });
}
```

### Workshop: Accessible Form

```dart
// WORKSHOP: สร้าง Accessible Registration Form

class AccessibleRegistrationForm extends StatefulWidget {
  @override
  State<AccessibleRegistrationForm> createState() =>
      _AccessibleRegistrationFormState();
}

class _AccessibleRegistrationFormState
    extends State<AccessibleRegistrationForm> {
  final _formKey = GlobalKey<FormState>();
  final _nameFocus = FocusNode();
  final _emailFocus = FocusNode();
  final _passwordFocus = FocusNode();
  
  String _name = '';
  String _email = '';
  String _password = '';
  bool _showPassword = false;
  bool _isSubmitting = false;
  String? _successMessage;

  @override
  void dispose() {
    _nameFocus.dispose();
    _emailFocus.dispose();
    _passwordFocus.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (!(_formKey.currentState?.validate() ?? false)) return;
    
    setState(() => _isSubmitting = true);
    
    // Announce loading
    await SemanticsService.announce(
      'Submitting your registration',
      TextDirection.ltr,
    );

    await Future.delayed(const Duration(seconds: 2)); // simulate API call
    
    setState(() {
      _isSubmitting = false;
      _successMessage = 'Registration successful! Welcome $_name!';
    });
    
    // Announce success
    await SemanticsService.announce(
      _successMessage!,
      TextDirection.ltr,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Create Account'),
        // ✅ Back button semantic
        leading: BackButton(
          onPressed: () => Navigator.pop(context),
        ),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // ✅ Live region สำหรับ success message
              if (_successMessage != null)
                Semantics(
                  liveRegion: true,
                  child: Container(
                    padding: const EdgeInsets.all(12),
                    color: Colors.green[100],
                    child: Row(
                      children: [
                        const Icon(Icons.check_circle, color: Colors.green),
                        const SizedBox(width: 8),
                        Expanded(
                          child: Text(
                            _successMessage!,
                            style: const TextStyle(color: Colors.green),
                          ),
                        ),
                      ],
                    ),
                  ),
                ),

              // ✅ Name field
              TextFormField(
                focusNode: _nameFocus,
                decoration: const InputDecoration(
                  labelText: 'Full Name *',
                  hintText: 'Enter your full name',
                  prefixIcon: Icon(Icons.person),
                ),
                textInputAction: TextInputAction.next,
                onFieldSubmitted: (_) => _emailFocus.requestFocus(),
                onChanged: (val) => _name = val,
                validator: (val) {
                  if (val == null || val.trim().isEmpty) {
                    return 'Name is required';
                  }
                  if (val.trim().length < 2) {
                    return 'Name must be at least 2 characters';
                  }
                  return null;
                },
              ),
              
              const SizedBox(height: 16),
              
              // ✅ Email field
              TextFormField(
                focusNode: _emailFocus,
                decoration: const InputDecoration(
                  labelText: 'Email Address *',
                  hintText: 'Enter your email',
                  prefixIcon: Icon(Icons.email),
                ),
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                onFieldSubmitted: (_) => _passwordFocus.requestFocus(),
                onChanged: (val) => _email = val,
                validator: (val) {
                  if (val == null || val.isEmpty) {
                    return 'Email is required';
                  }
                  if (!RegExp(r'^[^@]+@[^@]+\.[^@]+').hasMatch(val)) {
                    return 'Please enter a valid email address';
                  }
                  return null;
                },
              ),
              
              const SizedBox(height: 16),
              
              // ✅ Password field
              TextFormField(
                focusNode: _passwordFocus,
                decoration: InputDecoration(
                  labelText: 'Password *',
                  hintText: 'At least 8 characters',
                  prefixIcon: const Icon(Icons.lock),
                  suffixIcon: IconButton(
                    icon: Icon(
                      _showPassword ? Icons.visibility_off : Icons.visibility,
                    ),
                    tooltip: _showPassword ? 'Hide password' : 'Show password',
                    onPressed: () {
                      setState(() => _showPassword = !_showPassword);
                    },
                  ),
                ),
                obscureText: !_showPassword,
                textInputAction: TextInputAction.done,
                onFieldSubmitted: (_) => _submit(),
                onChanged: (val) => _password = val,
                validator: (val) {
                  if (val == null || val.isEmpty) {
                    return 'Password is required';
                  }
                  if (val.length < 8) {
                    return 'Password must be at least 8 characters';
                  }
                  return null;
                },
              ),
              
              const SizedBox(height: 32),
              
              // ✅ Submit button
              Semantics(
                label: _isSubmitting ? 'Submitting registration...' : 'Register',
                button: true,
                enabled: !_isSubmitting,
                child: ElevatedButton(
                  onPressed: _isSubmitting ? null : _submit,
                  style: ElevatedButton.styleFrom(
                    minimumSize: const Size(double.infinity, 48), // ✅ min height 48
                  ),
                  child: _isSubmitting
                      ? const SizedBox(
                          height: 20,
                          width: 20,
                          child: CircularProgressIndicator(strokeWidth: 2),
                        )
                      : const Text('Create Account'),
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

## สรุป (Summary)

```
Accessibility Checklist:

✅ Semantics:
  - ใส่ semantic labels สำหรับ Icons, Images, Buttons
  - ใช้ MergeSemantics สำหรับ related widgets
  - ใช้ ExcludeSemantics สำหรับ decorative elements
  - เพิ่ม liveRegion: true สำหรับ dynamic content

✅ Touch Targets:
  - ขนาดอย่างน้อย 48x48 dp
  - Spacing ระหว่างปุ่มอย่างน้อย 8dp

✅ Color:
  - Contrast ratio ≥ 4.5:1 สำหรับ normal text
  - Contrast ratio ≥ 3:1 สำหรับ large text
  - ไม่ใช้สีเป็น sole indicator
  - รองรับ high contrast mode

✅ Text:
  - ไม่ใช้ fixed font sizes
  - รองรับ text scaling up to 200%
  - ไม่ตัดข้อความออก

✅ Navigation:
  - Focus order ที่สมเหตุสมผล
  - Focus trap ใน Dialog
  - ประกาศ page changes สำหรับ screen reader

✅ Testing:
  - ใช้ meetsGuideline() ใน widget tests
  - ทดสอบกับ TalkBack/VoiceOver จริงๆ
  - ใช้ SemanticsDebugger
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. เพิ่ม semantic labels ให้ทุก IconButton ในแอป
2. ตรวจสอบและแก้ไข touch targets ที่เล็กกว่า 48x48
3. ตรวจสอบ contrast ratio ของ colors ในแอป

**ระดับกลาง:**
4. Implement AccessibleProductCard ที่ใช้ MergeSemantics
5. เพิ่ม liveRegion สำหรับ loading/error messages
6. สร้าง Widget test ที่ตรวจสอบ accessibility guidelines

**ระดับสูง:**
7. ทดสอบแอปกับ TalkBack บน Android และ VoiceOver บน iOS
8. Implement complete Focus Management สำหรับ Dialog flow
9. สร้าง Accessibility audit report สำหรับแอปทั้งหมด

---

## การนำทาง
- [← Part 34: Performance Optimization](part_34.md)
- [→ Part 36: Internationalization](part_36.md)

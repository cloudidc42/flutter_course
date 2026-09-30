# Part 07: Material Design & Theming
## ขั้นตอนที่ 61-70

---

## สารบัญ
1. [MaterialApp & ThemeData](#ขั้นตอนที่-61-materialapp--themedata)
2. [ColorScheme & Typography](#ขั้นตอนที่-62-colorscheme--typography)
3. [Material 3](#ขั้นตอนที่-63-material-3)
4. [Dark Mode](#ขั้นตอนที่-64-dark-mode)
5. [Custom Themes](#ขั้นตอนที่-65-custom-themes)
6. [Card, Chip, Dialog](#ขั้นตอนที่-66-card-chip-dialog)
7. [BottomSheet & Snackbar](#ขั้นตอนที่-67-bottomsheet--snackbar)
8. [ProgressIndicator & Divider](#ขั้นตอนที่-68-progressindicator--divider)
9. [Drawer & BottomNavigationBar](#ขั้นตอนที่-69-drawer--bottomnavigationbar)
10. [Workshop: Shopping App Theme](#ขั้นตอนที่-70-workshop-shopping-app-theme)

---

## ขั้นตอนที่ 61: MaterialApp & ThemeData

MaterialApp เป็น root widget ของแอป Material Design พร้อม ThemeData สำหรับตกแต่ง

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ThemeApp());
}

class ThemeApp extends StatelessWidget {
  const ThemeApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // =====================
      // App Identity
      // =====================
      title: 'My Flutter App',
      debugShowCheckedModeBanner: false,  // ซ่อน debug banner

      // =====================
      // Theme Configuration
      // =====================
      theme: _buildLightTheme(),
      darkTheme: _buildDarkTheme(),
      themeMode: ThemeMode.system,  // system, light, dark

      // =====================
      // Locale
      // =====================
      locale: const Locale('th', 'TH'),
      supportedLocales: const [
        Locale('th', 'TH'),
        Locale('en', 'US'),
      ],

      // =====================
      // Navigation
      // =====================
      initialRoute: '/',
      routes: {
        '/': (context) => const HomeScreen(),
        '/settings': (context) => const SettingsScreen(),
      },
    );
  }

  ThemeData _buildLightTheme() {
    return ThemeData(
      useMaterial3: true,
      // ColorScheme จาก seed color
      colorScheme: ColorScheme.fromSeed(
        seedColor: Colors.deepPurple,
        brightness: Brightness.light,
      ),

      // Typography
      textTheme: const TextTheme(
        displayLarge: TextStyle(
          fontSize: 57,
          fontWeight: FontWeight.w400,
          letterSpacing: -0.25,
        ),
        headlineMedium: TextStyle(
          fontSize: 28,
          fontWeight: FontWeight.w400,
        ),
        titleLarge: TextStyle(
          fontSize: 22,
          fontWeight: FontWeight.w500,
        ),
        bodyLarge: TextStyle(
          fontSize: 16,
          fontWeight: FontWeight.w400,
        ),
        bodyMedium: TextStyle(
          fontSize: 14,
          fontWeight: FontWeight.w400,
        ),
        labelLarge: TextStyle(
          fontSize: 14,
          fontWeight: FontWeight.w500,
        ),
      ),

      // AppBar Theme
      appBarTheme: const AppBarTheme(
        centerTitle: true,
        elevation: 0,
        scrolledUnderElevation: 2,
        backgroundColor: Colors.transparent,
      ),

      // Card Theme
      cardTheme: CardThemeData(
        elevation: 2,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),

      // Input Decoration Theme
      inputDecorationTheme: InputDecorationTheme(
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
        ),
        filled: true,
      ),

      // Button Themes
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
        ),
      ),
    );
  }

  ThemeData _buildDarkTheme() {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: Colors.deepPurple,
        brightness: Brightness.dark,
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final colorScheme = theme.colorScheme;

    return Scaffold(
      appBar: AppBar(
        title: Text('Theme Demo',
            style: TextStyle(color: colorScheme.onSurface)),
        backgroundColor: colorScheme.surface,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // แสดง colors ทั้งหมด
            Text('Color Palette:',
                style: theme.textTheme.titleLarge),
            const SizedBox(height: 8),
            _ColorGrid(colorScheme: colorScheme),

            const SizedBox(height: 24),

            Text('Typography:', style: theme.textTheme.titleLarge),
            const SizedBox(height: 8),
            _TypographyDemo(theme: theme),

            const SizedBox(height: 24),

            Text('Components:', style: theme.textTheme.titleLarge),
            const SizedBox(height: 8),
            _ComponentsDemo(),
          ],
        ),
      ),
    );
  }
}

class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ตั้งค่า')),
      body: const Center(child: Text('Settings Page')),
    );
  }
}

class _ColorGrid extends StatelessWidget {
  final ColorScheme colorScheme;
  const _ColorGrid({required this.colorScheme});

  @override
  Widget build(BuildContext context) {
    final colors = [
      ('primary', colorScheme.primary, colorScheme.onPrimary),
      ('secondary', colorScheme.secondary, colorScheme.onSecondary),
      ('tertiary', colorScheme.tertiary, colorScheme.onTertiary),
      ('error', colorScheme.error, colorScheme.onError),
      ('surface', colorScheme.surface, colorScheme.onSurface),
      ('primaryContainer', colorScheme.primaryContainer,
          colorScheme.onPrimaryContainer),
    ];

    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: colors.map((c) {
        return Container(
          width: 160,
          height: 60,
          padding: const EdgeInsets.all(8),
          decoration: BoxDecoration(
            color: c.$2,
            borderRadius: BorderRadius.circular(8),
          ),
          child: Text(
            c.$1,
            style: TextStyle(color: c.$3, fontSize: 12),
          ),
        );
      }).toList(),
    );
  }
}

class _TypographyDemo extends StatelessWidget {
  final ThemeData theme;
  const _TypographyDemo({required this.theme});

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('displayLarge', style: theme.textTheme.displayLarge?.copyWith(fontSize: 24)),
        Text('headlineMedium', style: theme.textTheme.headlineMedium),
        Text('titleLarge', style: theme.textTheme.titleLarge),
        Text('bodyLarge', style: theme.textTheme.bodyLarge),
        Text('bodyMedium', style: theme.textTheme.bodyMedium),
        Text('labelLarge', style: theme.textTheme.labelLarge),
      ],
    );
  }
}

class _ComponentsDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Wrap(
      spacing: 8,
      runSpacing: 8,
      children: [
        ElevatedButton(onPressed: () {}, child: const Text('Elevated')),
        TextButton(onPressed: () {}, child: const Text('Text')),
        OutlinedButton(onPressed: () {}, child: const Text('Outlined')),
        FilledButton(onPressed: () {}, child: const Text('Filled')),
        FilledButton.tonal(onPressed: () {}, child: const Text('Tonal')),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 62: ColorScheme & Typography

```dart
import 'package:flutter/material.dart';

class ColorSchemeDemo extends StatefulWidget {
  const ColorSchemeDemo({super.key});

  @override
  State<ColorSchemeDemo> createState() => _ColorSchemeDemoState();
}

class _ColorSchemeDemoState extends State<ColorSchemeDemo> {
  Color _seedColor = Colors.deepPurple;
  Brightness _brightness = Brightness.light;

  // seed colors to choose
  final List<Color> _seeds = [
    Colors.deepPurple,
    Colors.blue,
    Colors.teal,
    Colors.green,
    Colors.orange,
    Colors.red,
    Colors.pink,
    Colors.indigo,
  ];

  @override
  Widget build(BuildContext context) {
    // สร้าง ColorScheme จาก seed
    final colorScheme = ColorScheme.fromSeed(
      seedColor: _seedColor,
      brightness: _brightness,
    );

    return Theme(
      data: ThemeData(colorScheme: colorScheme, useMaterial3: true),
      child: Builder(
        builder: (context) {
          return Scaffold(
            backgroundColor: colorScheme.surface,
            appBar: AppBar(
              title: const Text('ColorScheme Demo'),
              backgroundColor: colorScheme.primaryContainer,
              foregroundColor: colorScheme.onPrimaryContainer,
              actions: [
                IconButton(
                  icon: Icon(
                    _brightness == Brightness.light
                        ? Icons.dark_mode
                        : Icons.light_mode,
                  ),
                  onPressed: () => setState(() {
                    _brightness = _brightness == Brightness.light
                        ? Brightness.dark
                        : Brightness.light;
                  }),
                ),
              ],
            ),
            body: SingleChildScrollView(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Seed Color Picker
                  Text('Seed Color:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        color: colorScheme.onSurface,
                      )),
                  const SizedBox(height: 8),
                  Wrap(
                    spacing: 8,
                    children: _seeds.map((color) {
                      final isSelected = color.value == _seedColor.value;
                      return GestureDetector(
                        onTap: () =>
                            setState(() => _seedColor = color),
                        child: Container(
                          width: 36,
                          height: 36,
                          decoration: BoxDecoration(
                            color: color,
                            shape: BoxShape.circle,
                            border: Border.all(
                              color: isSelected
                                  ? Colors.white
                                  : Colors.transparent,
                              width: 3,
                            ),
                            boxShadow: [
                              if (isSelected)
                                BoxShadow(
                                  color: color.withOpacity(0.5),
                                  blurRadius: 8,
                                ),
                            ],
                          ),
                          child: isSelected
                              ? const Icon(Icons.check,
                                  color: Colors.white, size: 16)
                              : null,
                        ),
                      );
                    }).toList(),
                  ),

                  const SizedBox(height: 24),

                  // Color Roles
                  Text('Color Roles:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        color: colorScheme.onSurface,
                      )),
                  const SizedBox(height: 8),

                  _buildColorPairs(colorScheme),

                  const SizedBox(height: 24),

                  // Typography Scale
                  Text('Typography:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        color: colorScheme.onSurface,
                      )),
                  const SizedBox(height: 8),

                  _buildTypographyScale(colorScheme),
                ],
              ),
            ),
          );
        },
      ),
    );
  }

  Widget _buildColorPairs(ColorScheme cs) {
    final pairs = [
      ('primary / onPrimary', cs.primary, cs.onPrimary),
      ('primaryContainer / onPrimaryContainer', cs.primaryContainer,
          cs.onPrimaryContainer),
      ('secondary / onSecondary', cs.secondary, cs.onSecondary),
      ('tertiary / onTertiary', cs.tertiary, cs.onTertiary),
      ('error / onError', cs.error, cs.onError),
      ('surface / onSurface', cs.surface, cs.onSurface),
      ('surfaceContainerHigh', cs.surfaceContainerHigh,
          cs.onSurface),
    ];

    return Column(
      children: pairs.map((p) {
        return Container(
          margin: const EdgeInsets.only(bottom: 4),
          height: 48,
          padding: const EdgeInsets.symmetric(horizontal: 12),
          decoration: BoxDecoration(
            color: p.$2,
            borderRadius: BorderRadius.circular(8),
          ),
          child: Align(
            alignment: Alignment.centerLeft,
            child: Text(
              p.$1,
              style: TextStyle(color: p.$3, fontSize: 13),
            ),
          ),
        );
      }).toList(),
    );
  }

  Widget _buildTypographyScale(ColorScheme cs) {
    final styles = [
      ('Display Large', const TextStyle(fontSize: 57, fontWeight: FontWeight.w400)),
      ('Display Medium', const TextStyle(fontSize: 45, fontWeight: FontWeight.w400)),
      ('Display Small', const TextStyle(fontSize: 36, fontWeight: FontWeight.w400)),
      ('Headline Large', const TextStyle(fontSize: 32, fontWeight: FontWeight.w400)),
      ('Title Large', const TextStyle(fontSize: 22, fontWeight: FontWeight.w500)),
      ('Body Large', const TextStyle(fontSize: 16, fontWeight: FontWeight.w400)),
      ('Body Medium', const TextStyle(fontSize: 14, fontWeight: FontWeight.w400)),
      ('Label Large', const TextStyle(fontSize: 14, fontWeight: FontWeight.w500)),
      ('Label Small', const TextStyle(fontSize: 11, fontWeight: FontWeight.w500)),
    ];

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: styles.map((s) {
        return Padding(
          padding: const EdgeInsets.symmetric(vertical: 4),
          child: Text(
            s.$1,
            style: s.$2.copyWith(
              color: cs.onSurface,
              // จำกัดขนาดเพื่อ display
              fontSize: s.$2.fontSize!.clamp(11, 28),
            ),
          ),
        );
      }).toList(),
    );
  }
}
```

---

## ขั้นตอนที่ 63: Material 3

```dart
import 'package:flutter/material.dart';

// Material 3 Components
class Material3Demo extends StatefulWidget {
  const Material3Demo({super.key});

  @override
  State<Material3Demo> createState() => _Material3DemoState();
}

class _Material3DemoState extends State<Material3Demo> {
  bool _useMaterial3 = true;
  int _selectedIndex = 0;
  bool _switchValue = true;
  double _sliderValue = 0.5;
  bool _checkboxValue = true;
  String _radioValue = 'option1';

  @override
  Widget build(BuildContext context) {
    return Theme(
      data: ThemeData(
        useMaterial3: _useMaterial3,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.teal,
          brightness: Brightness.light,
        ),
      ),
      child: Builder(builder: (context) {
        final theme = Theme.of(context);
        return Scaffold(
          appBar: AppBar(
            title: Text('Material ${_useMaterial3 ? '3' : '2'}'),
            actions: [
              Row(
                children: [
                  const Text('M3'),
                  Switch(
                    value: _useMaterial3,
                    onChanged: (v) => setState(() => _useMaterial3 = v),
                  ),
                ],
              ),
            ],
          ),
          body: SingleChildScrollView(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Buttons
                Text('Buttons:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                Wrap(
                  spacing: 8,
                  runSpacing: 8,
                  children: [
                    ElevatedButton(
                        onPressed: () {}, child: const Text('Elevated')),
                    FilledButton(
                        onPressed: () {}, child: const Text('Filled')),
                    FilledButton.tonal(
                        onPressed: () {}, child: const Text('Tonal')),
                    OutlinedButton(
                        onPressed: () {}, child: const Text('Outlined')),
                    TextButton(onPressed: () {}, child: const Text('Text')),
                  ],
                ),

                const SizedBox(height: 24),

                // Icon Buttons
                Text('Icon Buttons:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                Row(
                  children: [
                    IconButton(
                      icon: const Icon(Icons.favorite),
                      onPressed: () {},
                    ),
                    IconButton.filled(
                      icon: const Icon(Icons.add),
                      onPressed: () {},
                    ),
                    IconButton.filledTonal(
                      icon: const Icon(Icons.edit),
                      onPressed: () {},
                    ),
                    IconButton.outlined(
                      icon: const Icon(Icons.share),
                      onPressed: () {},
                    ),
                  ],
                ),

                const SizedBox(height: 24),

                // FAB Variants
                Text('FAB:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    FloatingActionButton.small(
                      heroTag: 'm3_fab_s',
                      onPressed: () {},
                      child: const Icon(Icons.add),
                    ),
                    FloatingActionButton(
                      heroTag: 'm3_fab_r',
                      onPressed: () {},
                      child: const Icon(Icons.add),
                    ),
                    FloatingActionButton.large(
                      heroTag: 'm3_fab_l',
                      onPressed: () {},
                      child: const Icon(Icons.add),
                    ),
                    FloatingActionButton.extended(
                      heroTag: 'm3_fab_e',
                      onPressed: () {},
                      icon: const Icon(Icons.add),
                      label: const Text('New'),
                    ),
                  ],
                ),

                const SizedBox(height: 24),

                // Cards
                Text('Cards:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                Row(
                  children: [
                    Expanded(
                      child: Card(
                        child: Padding(
                          padding: const EdgeInsets.all(16),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text('Elevated Card',
                                  style: theme.textTheme.titleSmall),
                              const Text('elevation: 1'),
                            ],
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(width: 8),
                    Expanded(
                      child: Card.filled(
                        child: Padding(
                          padding: const EdgeInsets.all(16),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text('Filled Card',
                                  style: theme.textTheme.titleSmall),
                              const Text('no elevation'),
                            ],
                          ),
                        ),
                      ),
                    ),
                    const SizedBox(width: 8),
                    Expanded(
                      child: Card.outlined(
                        child: Padding(
                          padding: const EdgeInsets.all(16),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text('Outlined Card',
                                  style: theme.textTheme.titleSmall),
                              const Text('border'),
                            ],
                          ),
                        ),
                      ),
                    ),
                  ],
                ),

                const SizedBox(height: 24),

                // Chips
                Text('Chips:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                Wrap(
                  spacing: 8,
                  runSpacing: 8,
                  children: [
                    const Chip(label: Text('Chip')),
                    ActionChip(
                      label: const Text('Action'),
                      onPressed: () {},
                    ),
                    FilterChip(
                      label: const Text('Filter'),
                      selected: true,
                      onSelected: (_) {},
                    ),
                    ChoiceChip(
                      label: const Text('Choice'),
                      selected: true,
                      onSelected: (_) {},
                    ),
                    InputChip(
                      label: const Text('Input'),
                      onPressed: () {},
                      onDeleted: () {},
                    ),
                  ],
                ),

                const SizedBox(height: 24),

                // Selection Controls
                Text('Selection Controls:',
                    style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),

                Row(
                  children: [
                    Checkbox(
                      value: _checkboxValue,
                      onChanged: (v) =>
                          setState(() => _checkboxValue = v!),
                    ),
                    const Text('Checkbox'),
                    const SizedBox(width: 16),
                    Switch(
                      value: _switchValue,
                      onChanged: (v) =>
                          setState(() => _switchValue = v),
                    ),
                    const Text('Switch'),
                  ],
                ),

                Row(
                  children: [
                    Radio<String>(
                      value: 'option1',
                      groupValue: _radioValue,
                      onChanged: (v) =>
                          setState(() => _radioValue = v!),
                    ),
                    const Text('Option 1'),
                    Radio<String>(
                      value: 'option2',
                      groupValue: _radioValue,
                      onChanged: (v) =>
                          setState(() => _radioValue = v!),
                    ),
                    const Text('Option 2'),
                  ],
                ),

                Slider(
                  value: _sliderValue,
                  onChanged: (v) => setState(() => _sliderValue = v),
                  label: _sliderValue.toStringAsFixed(2),
                ),

                const SizedBox(height: 24),

                // Navigation
                Text('Navigation Bar:', style: theme.textTheme.titleMedium),
                const SizedBox(height: 8),
                NavigationBar(
                  selectedIndex: _selectedIndex,
                  onDestinationSelected: (i) =>
                      setState(() => _selectedIndex = i),
                  destinations: const [
                    NavigationDestination(
                      icon: Icon(Icons.home_outlined),
                      selectedIcon: Icon(Icons.home),
                      label: 'Home',
                    ),
                    NavigationDestination(
                      icon: Icon(Icons.search_outlined),
                      selectedIcon: Icon(Icons.search),
                      label: 'Search',
                    ),
                    NavigationDestination(
                      icon: Icon(Icons.person_outline),
                      selectedIcon: Icon(Icons.person),
                      label: 'Profile',
                    ),
                  ],
                ),
              ],
            ),
          ),
        );
      }),
    );
  }
}
```

---

## ขั้นตอนที่ 64: Dark Mode

```dart
import 'package:flutter/material.dart';

// ใช้ ValueNotifier สำหรับ theme mode
class ThemeModeController extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;

  ThemeMode get themeMode => _themeMode;

  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    notifyListeners();
  }

  void toggleDarkMode() {
    _themeMode = _themeMode == ThemeMode.dark
        ? ThemeMode.light
        : ThemeMode.dark;
    notifyListeners();
  }
}

class DarkModeApp extends StatefulWidget {
  const DarkModeApp({super.key});

  @override
  State<DarkModeApp> createState() => _DarkModeAppState();
}

class _DarkModeAppState extends State<DarkModeApp> {
  ThemeMode _themeMode = ThemeMode.system;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dark Mode Demo',
      themeMode: _themeMode,
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.indigo,
          brightness: Brightness.light,
        ),
      ),
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.indigo,
          brightness: Brightness.dark,
        ),
      ),
      home: DarkModeScreen(
        themeMode: _themeMode,
        onThemeModeChanged: (mode) =>
            setState(() => _themeMode = mode),
      ),
    );
  }
}

class DarkModeScreen extends StatelessWidget {
  final ThemeMode themeMode;
  final ValueChanged<ThemeMode> onThemeModeChanged;

  const DarkModeScreen({
    super.key,
    required this.themeMode,
    required this.onThemeModeChanged,
  });

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final isDark = theme.brightness == Brightness.dark;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Dark Mode'),
        actions: [
          // Theme mode toggle
          PopupMenuButton<ThemeMode>(
            icon: Icon(
              themeMode == ThemeMode.dark
                  ? Icons.dark_mode
                  : themeMode == ThemeMode.light
                      ? Icons.light_mode
                      : Icons.brightness_auto,
            ),
            onSelected: onThemeModeChanged,
            itemBuilder: (context) => [
              const PopupMenuItem(
                value: ThemeMode.system,
                child: Row(children: [
                  Icon(Icons.brightness_auto),
                  SizedBox(width: 8),
                  Text('ตามระบบ'),
                ]),
              ),
              const PopupMenuItem(
                value: ThemeMode.light,
                child: Row(children: [
                  Icon(Icons.light_mode),
                  SizedBox(width: 8),
                  Text('สว่าง'),
                ]),
              ),
              const PopupMenuItem(
                value: ThemeMode.dark,
                child: Row(children: [
                  Icon(Icons.dark_mode),
                  SizedBox(width: 8),
                  Text('มืด'),
                ]),
              ),
            ],
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // Status card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Row(
                  children: [
                    Icon(
                      isDark ? Icons.dark_mode : Icons.light_mode,
                      size: 32,
                      color: isDark ? Colors.yellow : Colors.orange,
                    ),
                    const SizedBox(width: 16),
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          isDark ? 'Dark Mode' : 'Light Mode',
                          style: theme.textTheme.titleLarge,
                        ),
                        Text(
                          'ThemeMode: ${themeMode.name}',
                          style: theme.textTheme.bodySmall,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Content ที่ adapt ตาม theme
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Content ที่ใช้ Theme:',
                        style: theme.textTheme.titleMedium),
                    const SizedBox(height: 8),

                    // ใช้ colorScheme แทนการ hardcode สี
                    Container(
                      padding: const EdgeInsets.all(12),
                      decoration: BoxDecoration(
                        // ✅ ดี - ใช้ theme color
                        color: theme.colorScheme.primaryContainer,
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: Text(
                        'primaryContainer - ปรับตาม theme อัตโนมัติ',
                        style: TextStyle(
                          color: theme.colorScheme.onPrimaryContainer,
                        ),
                      ),
                    ),

                    const SizedBox(height: 8),

                    Container(
                      padding: const EdgeInsets.all(12),
                      decoration: BoxDecoration(
                        // ✅ ดี
                        color: theme.colorScheme.surfaceContainerHigh,
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: Text(
                        'surfaceContainerHigh - ปรับตาม theme',
                        style: TextStyle(
                          color: theme.colorScheme.onSurface,
                        ),
                      ),
                    ),

                    const SizedBox(height: 8),

                    // ❌ ไม่ดี - hardcode สี
                    Container(
                      padding: const EdgeInsets.all(12),
                      decoration: BoxDecoration(
                        color: Colors.white, // ❌ ไม่ดีใน dark mode
                        borderRadius: BorderRadius.circular(8),
                      ),
                      child: const Text(
                        'Hardcoded white - ไม่ดีใน dark mode',
                        style: TextStyle(color: Colors.black),
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Tips
            Card(
              color: theme.colorScheme.tertiaryContainer,
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      'Tips สำหรับ Dark Mode:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        color: theme.colorScheme.onTertiaryContainer,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      '• ใช้ colorScheme แทนการ hardcode สี\n'
                      '• ใช้ Theme.of(context).brightness เช็คสถานะ\n'
                      '• ทดสอบทั้ง light และ dark mode\n'
                      '• หลีกเลี่ยง Colors.white/black โดยตรง\n'
                      '• ใช้ adaptive widgets เมื่อเป็นไปได้',
                      style: TextStyle(
                        color: theme.colorScheme.onTertiaryContainer,
                        height: 1.6,
                      ),
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

## ขั้นตอนที่ 65: Custom Themes

```dart
import 'package:flutter/material.dart';

// Custom Brand Theme
class BrandTheme {
  // Brand Colors
  static const Color primary = Color(0xFF6C63FF);
  static const Color secondary = Color(0xFF03DAC6);
  static const Color accent = Color(0xFFFF6B6B);
  static const Color surface = Color(0xFFF8F9FE);
  static const Color background = Color(0xFFFFFFFF);

  // สร้าง Light Theme
  static ThemeData light() {
    final colorScheme = ColorScheme.fromSeed(
      seedColor: primary,
      brightness: Brightness.light,
    ).copyWith(
      primary: primary,
      secondary: secondary,
      error: accent,
      surface: surface,
    );

    return ThemeData(
      useMaterial3: true,
      colorScheme: colorScheme,
      fontFamily: 'Prompt', // ต้องเพิ่มใน pubspec.yaml

      // AppBar
      appBarTheme: AppBarTheme(
        centerTitle: true,
        backgroundColor: Colors.white,
        foregroundColor: primary,
        elevation: 0,
        shadowColor: Colors.transparent,
        surfaceTintColor: Colors.transparent,
        titleTextStyle: const TextStyle(
          color: primary,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),

      // Buttons
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          backgroundColor: primary,
          foregroundColor: Colors.white,
          elevation: 0,
          padding:
              const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
          textStyle: const TextStyle(
            fontSize: 16,
            fontWeight: FontWeight.w600,
          ),
        ),
      ),

      outlinedButtonTheme: OutlinedButtonThemeData(
        style: OutlinedButton.styleFrom(
          foregroundColor: primary,
          side: const BorderSide(color: primary, width: 1.5),
          padding:
              const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
          ),
        ),
      ),

      textButtonTheme: TextButtonThemeData(
        style: TextButton.styleFrom(
          foregroundColor: primary,
          textStyle: const TextStyle(fontWeight: FontWeight.w600),
        ),
      ),

      // Input
      inputDecorationTheme: InputDecorationTheme(
        filled: true,
        fillColor: const Color(0xFFF0F0FF),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(16),
          borderSide: BorderSide.none,
        ),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(16),
          borderSide: BorderSide.none,
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(16),
          borderSide: const BorderSide(color: primary, width: 2),
        ),
        errorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(16),
          borderSide: const BorderSide(color: accent, width: 1),
        ),
        contentPadding:
            const EdgeInsets.symmetric(horizontal: 20, vertical: 16),
        labelStyle: const TextStyle(color: Colors.grey),
      ),

      // Card
      cardTheme: CardThemeData(
        elevation: 0,
        color: Colors.white,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(20),
          side: BorderSide(color: Colors.grey.shade200, width: 1),
        ),
      ),

      // Chip
      chipTheme: ChipThemeData(
        backgroundColor: const Color(0xFFF0F0FF),
        selectedColor: primary.withOpacity(0.2),
        labelStyle: const TextStyle(color: primary),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        side: BorderSide.none,
      ),

      // Typography
      textTheme: const TextTheme(
        headlineLarge: TextStyle(
          fontWeight: FontWeight.bold,
          color: Color(0xFF1A1A2E),
        ),
        headlineMedium: TextStyle(
          fontWeight: FontWeight.bold,
          color: Color(0xFF1A1A2E),
        ),
        titleLarge: TextStyle(
          fontWeight: FontWeight.w600,
          color: Color(0xFF1A1A2E),
        ),
        bodyLarge: TextStyle(
          color: Color(0xFF4A4A6A),
          height: 1.6,
        ),
        bodyMedium: TextStyle(
          color: Color(0xFF4A4A6A),
          height: 1.5,
        ),
      ),
    );
  }

  // สร้าง Dark Theme
  static ThemeData dark() {
    final colorScheme = ColorScheme.fromSeed(
      seedColor: primary,
      brightness: Brightness.dark,
    ).copyWith(
      primary: const Color(0xFF9C94FF),
      secondary: secondary,
    );

    return ThemeData(
      useMaterial3: true,
      colorScheme: colorScheme,
      scaffoldBackgroundColor: const Color(0xFF0D0D1A),
      cardTheme: CardThemeData(
        color: const Color(0xFF1A1A2E),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(20),
        ),
      ),
    );
  }
}

// Demo Screen ที่ใช้ Brand Theme
class CustomThemeDemo extends StatelessWidget {
  const CustomThemeDemo({super.key});

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Scaffold(
      appBar: AppBar(title: const Text('Custom Brand Theme')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Form Example
            Card(
              child: Padding(
                padding: const EdgeInsets.all(20),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    Text('เข้าสู่ระบบ', style: theme.textTheme.titleLarge),
                    const SizedBox(height: 20),
                    const TextField(
                      decoration: InputDecoration(
                        labelText: 'อีเมล',
                        prefixIcon: Icon(Icons.email_outlined),
                      ),
                    ),
                    const SizedBox(height: 16),
                    const TextField(
                      obscureText: true,
                      decoration: InputDecoration(
                        labelText: 'รหัสผ่าน',
                        prefixIcon: Icon(Icons.lock_outlined),
                      ),
                    ),
                    const SizedBox(height: 24),
                    ElevatedButton(
                      onPressed: () {},
                      child: const Text('เข้าสู่ระบบ'),
                    ),
                    const SizedBox(height: 12),
                    OutlinedButton(
                      onPressed: () {},
                      child: const Text('สมัครสมาชิก'),
                    ),
                    TextButton(
                      onPressed: () {},
                      child: const Text('ลืมรหัสผ่าน?'),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 20),

            // Chips
            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                const Chip(label: Text('Flutter')),
                FilterChip(
                  label: const Text('Dart'),
                  selected: true,
                  onSelected: (_) {},
                ),
                ActionChip(
                  label: const Text('Firebase'),
                  onPressed: () {},
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 66: Card, Chip, Dialog

```dart
import 'package:flutter/material.dart';

class MaterialComponentsDemo extends StatefulWidget {
  const MaterialComponentsDemo({super.key});

  @override
  State<MaterialComponentsDemo> createState() =>
      _MaterialComponentsDemoState();
}

class _MaterialComponentsDemoState extends State<MaterialComponentsDemo> {
  bool _chip1 = false;
  bool _chip2 = true;
  String _choice = 'A';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Card, Chip, Dialog')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ========= CARDS =========
            const Text('Cards:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Basic Card
            const Card(
              child: ListTile(
                leading: Icon(Icons.info),
                title: Text('Basic Card'),
                subtitle: Text('Card พื้นฐาน'),
              ),
            ),

            const SizedBox(height: 8),

            // Card พร้อม Image
            Card(
              clipBehavior: Clip.antiAlias,
              child: Column(
                children: [
                  Image.network(
                    'https://picsum.photos/400/150?random=66',
                    height: 150,
                    width: double.infinity,
                    fit: BoxFit.cover,
                  ),
                  const Padding(
                    padding: EdgeInsets.all(16),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Card Title',
                            style: TextStyle(
                                fontWeight: FontWeight.bold, fontSize: 16)),
                        Text('รายละเอียดของ card'),
                      ],
                    ),
                  ),
                  OverflowBar(
                    children: [
                      TextButton(onPressed: () {}, child: const Text('อ่านเพิ่ม')),
                      TextButton(onPressed: () {}, child: const Text('แชร์')),
                    ],
                  ),
                ],
              ),
            ),

            const SizedBox(height: 24),

            // ========= CHIPS =========
            const Text('Chips:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                // Chip พื้นฐาน
                const Chip(
                  avatar: CircleAvatar(child: Text('F')),
                  label: Text('Flutter'),
                  deleteIcon: Icon(Icons.cancel, size: 16),
                  onDeleted: null,
                ),

                // FilterChip
                FilterChip(
                  label: const Text('Filter 1'),
                  selected: _chip1,
                  onSelected: (v) => setState(() => _chip1 = v),
                ),
                FilterChip(
                  label: const Text('Filter 2'),
                  selected: _chip2,
                  onSelected: (v) => setState(() => _chip2 = v),
                ),

                // ChoiceChip
                ChoiceChip(
                  label: const Text('A'),
                  selected: _choice == 'A',
                  onSelected: (_) => setState(() => _choice = 'A'),
                ),
                ChoiceChip(
                  label: const Text('B'),
                  selected: _choice == 'B',
                  onSelected: (_) => setState(() => _choice = 'B'),
                ),
                ChoiceChip(
                  label: const Text('C'),
                  selected: _choice == 'C',
                  onSelected: (_) => setState(() => _choice = 'C'),
                ),

                // ActionChip
                ActionChip(
                  avatar: const Icon(Icons.add, size: 18),
                  label: const Text('Add Tag'),
                  onPressed: () {},
                ),

                // InputChip
                InputChip(
                  avatar: const CircleAvatar(
                    backgroundImage:
                        NetworkImage('https://picsum.photos/32/32?random=1'),
                  ),
                  label: const Text('สมชาย'),
                  onPressed: () {},
                  onDeleted: () {},
                ),
              ],
            ),

            const SizedBox(height: 24),

            // ========= DIALOGS =========
            const Text('Dialogs:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Wrap(
              spacing: 8,
              runSpacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () => _showAlertDialog(context),
                  child: const Text('Alert Dialog'),
                ),
                ElevatedButton(
                  onPressed: () => _showSimpleDialog(context),
                  child: const Text('Simple Dialog'),
                ),
                ElevatedButton(
                  onPressed: () => _showCustomDialog(context),
                  child: const Text('Custom Dialog'),
                ),
                ElevatedButton(
                  onPressed: () => _showFullScreenDialog(context),
                  child: const Text('Full Screen'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  void _showAlertDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        icon: const Icon(Icons.warning_amber, color: Colors.orange, size: 48),
        title: const Text('ยืนยันการลบ'),
        content: const Text('คุณแน่ใจหรือไม่ว่าต้องการลบรายการนี้?\nไม่สามารถกู้คืนได้'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(context, true),
            style: FilledButton.styleFrom(backgroundColor: Colors.red),
            child: const Text('ลบ'),
          ),
        ],
      ),
    );
  }

  void _showSimpleDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => SimpleDialog(
        title: const Text('เลือกตัวเลือก'),
        children: [
          SimpleDialogOption(
            onPressed: () => Navigator.pop(context, 'view'),
            child: const ListTile(
              leading: Icon(Icons.visibility),
              title: Text('ดู'),
            ),
          ),
          SimpleDialogOption(
            onPressed: () => Navigator.pop(context, 'edit'),
            child: const ListTile(
              leading: Icon(Icons.edit),
              title: Text('แก้ไข'),
            ),
          ),
          SimpleDialogOption(
            onPressed: () => Navigator.pop(context, 'share'),
            child: const ListTile(
              leading: Icon(Icons.share),
              title: Text('แชร์'),
            ),
          ),
          SimpleDialogOption(
            onPressed: () => Navigator.pop(context, 'delete'),
            child: const ListTile(
              leading: Icon(Icons.delete, color: Colors.red),
              title: Text('ลบ', style: TextStyle(color: Colors.red)),
            ),
          ),
        ],
      ),
    );
  }

  void _showCustomDialog(BuildContext context) {
    showDialog(
      context: context,
      barrierDismissible: true,
      builder: (context) {
        String _input = '';
        return AlertDialog(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(20),
          ),
          title: Row(
            children: [
              Container(
                padding: const EdgeInsets.all(8),
                decoration: BoxDecoration(
                  color: Colors.blue.shade50,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Icon(Icons.edit, color: Colors.blue),
              ),
              const SizedBox(width: 12),
              const Text('แก้ไขชื่อ'),
            ],
          ),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                autofocus: true,
                decoration: const InputDecoration(
                  labelText: 'ชื่อใหม่',
                  hintText: 'กรอกชื่อ...',
                  border: OutlineInputBorder(),
                ),
                onChanged: (v) => _input = v,
              ),
            ],
          ),
          actions: [
            OutlinedButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('ยกเลิก'),
            ),
            ElevatedButton(
              onPressed: () => Navigator.pop(context, _input),
              child: const Text('บันทึก'),
            ),
          ],
        );
      },
    );
  }

  void _showFullScreenDialog(BuildContext context) {
    showDialog(
      context: context,
      builder: (context) => Dialog.fullscreen(
        child: Scaffold(
          appBar: AppBar(
            title: const Text('Full Screen Dialog'),
            leading: IconButton(
              icon: const Icon(Icons.close),
              onPressed: () => Navigator.pop(context),
            ),
            actions: [
              TextButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('บันทึก'),
              ),
            ],
          ),
          body: const Center(
            child: Text('Full Screen Dialog Content'),
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 67: BottomSheet & Snackbar

```dart
import 'package:flutter/material.dart';

class BottomSheetSnackbarDemo extends StatelessWidget {
  const BottomSheetSnackbarDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('BottomSheet & Snackbar')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // BottomSheet
            const Text('BottomSheet:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showModalBottomSheet(context),
              child: const Text('Modal BottomSheet'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showDraggableBottomSheet(context),
              child: const Text('Draggable BottomSheet'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showScrollableBottomSheet(context),
              child: const Text('Scrollable BottomSheet'),
            ),

            const SizedBox(height: 24),

            // Snackbar
            const Text('Snackbar:', style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showBasicSnackbar(context),
              child: const Text('Basic Snackbar'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showActionSnackbar(context),
              child: const Text('Snackbar + Action'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showFloatingSnackbar(context),
              child: const Text('Floating Snackbar'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showColoredSnackbar(context, 'success'),
              style: ElevatedButton.styleFrom(backgroundColor: Colors.green),
              child: const Text('Success Snackbar'),
            ),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showColoredSnackbar(context, 'error'),
              style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
              child: const Text('Error Snackbar'),
            ),
          ],
        ),
      ),
    );
  }

  void _showModalBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
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
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text('Modal BottomSheet',
                style: TextStyle(
                    fontSize: 18, fontWeight: FontWeight.bold)),
          ),
          const Divider(height: 1),
          ListTile(
            leading: const Icon(Icons.photo_camera),
            title: const Text('ถ่ายรูป'),
            onTap: () => Navigator.pop(context),
          ),
          ListTile(
            leading: const Icon(Icons.photo_library),
            title: const Text('เลือกจากคลัง'),
            onTap: () => Navigator.pop(context),
          ),
          ListTile(
            leading: const Icon(Icons.file_upload),
            title: const Text('อัปโหลดไฟล์'),
            onTap: () => Navigator.pop(context),
          ),
          const SizedBox(height: 16),
        ],
      ),
    );
  }

  void _showDraggableBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,  // เต็มหน้าจอได้
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.4,    // เริ่มที่ 40%
        minChildSize: 0.2,         // ต่ำสุด 20%
        maxChildSize: 0.9,         // สูงสุด 90%
        expand: false,
        builder: (context, scrollController) => Column(
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
            const Padding(
              padding: EdgeInsets.all(16),
              child: Text('Draggable BottomSheet',
                  style: TextStyle(
                      fontSize: 18, fontWeight: FontWeight.bold)),
            ),
            Expanded(
              child: ListView.builder(
                controller: scrollController,
                itemCount: 30,
                itemBuilder: (context, index) => ListTile(
                  leading: CircleAvatar(child: Text('${index + 1}')),
                  title: Text('รายการที่ ${index + 1}'),
                  subtitle: const Text('รายละเอียด...'),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _showScrollableBottomSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      useSafeArea: true,
      builder: (context) => Padding(
        padding: EdgeInsets.only(
          bottom: MediaQuery.of(context).viewInsets.bottom,
        ),
        child: const _SearchBottomSheet(),
      ),
    );
  }

  void _showBasicSnackbar(BuildContext context) {
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('Basic Snackbar')),
    );
  }

  void _showActionSnackbar(BuildContext context) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: const Text('ลบรายการแล้ว'),
        action: SnackBarAction(
          label: 'Undo',
          textColor: Colors.yellow,
          onPressed: () {
            // กู้คืนรายการ
          },
        ),
        duration: const Duration(seconds: 4),
      ),
    );
  }

  void _showFloatingSnackbar(BuildContext context) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: const Row(
          children: [
            Icon(Icons.check_circle, color: Colors.white),
            SizedBox(width: 8),
            Text('บันทึกสำเร็จ!'),
          ],
        ),
        behavior: SnackBarBehavior.floating,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(10),
        ),
        margin: const EdgeInsets.all(16),
      ),
    );
  }

  void _showColoredSnackbar(BuildContext context, String type) {
    final isSuccess = type == 'success';
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        backgroundColor: isSuccess ? Colors.green : Colors.red,
        content: Row(
          children: [
            Icon(
              isSuccess ? Icons.check_circle : Icons.error,
              color: Colors.white,
            ),
            const SizedBox(width: 8),
            Text(
              isSuccess ? 'ดำเนินการสำเร็จ!' : 'เกิดข้อผิดพลาด!',
              style: const TextStyle(color: Colors.white),
            ),
          ],
        ),
        behavior: SnackBarBehavior.floating,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(10),
        ),
        margin: const EdgeInsets.all(16),
      ),
    );
  }
}

class _SearchBottomSheet extends StatelessWidget {
  const _SearchBottomSheet();

  @override
  Widget build(BuildContext context) {
    return Column(
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
        const Padding(
          padding: EdgeInsets.all(16),
          child: TextField(
            autofocus: true,
            decoration: InputDecoration(
              hintText: 'ค้นหา...',
              prefixIcon: Icon(Icons.search),
              border: OutlineInputBorder(),
            ),
          ),
        ),
        const SizedBox(height: 8),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 68: ProgressIndicator & Divider

```dart
import 'package:flutter/material.dart';
import 'dart:async';

class ProgressDividerDemo extends StatefulWidget {
  const ProgressDividerDemo({super.key});

  @override
  State<ProgressDividerDemo> createState() => _ProgressDividerDemoState();
}

class _ProgressDividerDemoState extends State<ProgressDividerDemo> {
  double _progress = 0.0;
  bool _loading = false;
  Timer? _timer;

  void _startProgress() {
    setState(() {
      _progress = 0.0;
      _loading = true;
    });
    _timer = Timer.periodic(const Duration(milliseconds: 100), (timer) {
      if (mounted) {
        setState(() => _progress += 0.02);
        if (_progress >= 1.0) {
          timer.cancel();
          setState(() => _loading = false);
        }
      }
    });
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Progress & Divider')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('LinearProgressIndicator:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Indeterminate (ไม่รู้ % ที่แน่นอน)
            const LinearProgressIndicator(),
            const SizedBox(height: 8),
            const Text('Indeterminate', style: TextStyle(fontSize: 12, color: Colors.grey)),

            const SizedBox(height: 16),

            // Determinate (รู้ % ที่แน่นอน)
            LinearProgressIndicator(value: _progress),
            const SizedBox(height: 4),
            Text('${(_progress * 100).toInt()}%',
                style: const TextStyle(fontSize: 12, color: Colors.grey)),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: _loading ? null : _startProgress,
              child: Text(_loading ? 'กำลังโหลด...' : 'เริ่ม'),
            ),

            const SizedBox(height: 16),

            // Custom styled
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: LinearProgressIndicator(
                value: 0.7,
                minHeight: 12,
                backgroundColor: Colors.grey.shade200,
                valueColor:
                    const AlwaysStoppedAnimation<Color>(Colors.green),
              ),
            ),

            const SizedBox(height: 24),

            const Text('CircularProgressIndicator:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                // Indeterminate
                const CircularProgressIndicator(),

                // Determinate
                CircularProgressIndicator(value: _progress),

                // Custom
                CircularProgressIndicator(
                  value: 0.75,
                  strokeWidth: 8,
                  backgroundColor: Colors.blue.shade100,
                  valueColor: const AlwaysStoppedAnimation<Color>(
                      Colors.blue),
                ),

                // Adaptive (iOS = CupertinoActivityIndicator)
                const CircularProgressIndicator.adaptive(),
              ],
            ),

            const SizedBox(height: 24),

            const Text('RefreshProgressIndicator:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Center(child: RefreshProgressIndicator()),

            const SizedBox(height: 24),

            const Text('Divider:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            // Basic Divider
            const Divider(),
            const Text('Basic Divider'),

            // Colored Divider
            const Divider(color: Colors.blue, thickness: 2),
            const Text('Colored Divider (blue, 2px)'),

            // Indented Divider
            const Divider(indent: 40, endIndent: 40),
            const Text('Indented Divider'),

            // Vertical Divider
            SizedBox(
              height: 50,
              child: Row(
                children: [
                  const Text('Left'),
                  const VerticalDivider(
                    width: 32,
                    thickness: 2,
                    color: Colors.grey,
                  ),
                  const Text('Right'),
                ],
              ),
            ),

            const SizedBox(height: 24),

            const Text('Loading Overlay:',
                style: TextStyle(fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),

            ElevatedButton(
              onPressed: () => _showLoadingOverlay(context),
              child: const Text('แสดง Loading Overlay'),
            ),
          ],
        ),
      ),
    );
  }

  void _showLoadingOverlay(BuildContext context) async {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => PopScope(
        canPop: false,
        child: Dialog(
          backgroundColor: Colors.transparent,
          elevation: 0,
          child: Container(
            padding: const EdgeInsets.all(24),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(16),
            ),
            child: const Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                CircularProgressIndicator(),
                SizedBox(height: 16),
                Text('กำลังโหลด...',
                    style: TextStyle(fontSize: 16)),
              ],
            ),
          ),
        ),
      ),
    );

    // จำลองการโหลด 2 วินาที
    await Future.delayed(const Duration(seconds: 2));
    if (context.mounted) Navigator.pop(context);
  }
}
```

---

## ขั้นตอนที่ 69: Drawer & BottomNavigationBar

```dart
import 'package:flutter/material.dart';

class DrawerNavigationDemo extends StatefulWidget {
  const DrawerNavigationDemo({super.key});

  @override
  State<DrawerNavigationDemo> createState() =>
      _DrawerNavigationDemoState();
}

class _DrawerNavigationDemoState extends State<DrawerNavigationDemo> {
  int _currentIndex = 0;
  String _currentMenu = 'หน้าแรก';

  final List<Map<String, dynamic>> _navItems = const [
    {'icon': Icons.home, 'label': 'หน้าแรก'},
    {'icon': Icons.explore, 'label': 'สำรวจ'},
    {'icon': Icons.notifications, 'label': 'แจ้งเตือน'},
    {'icon': Icons.person, 'label': 'โปรไฟล์'},
  ];

  final List<Map<String, dynamic>> _drawerItems = const [
    {'icon': Icons.home, 'label': 'หน้าแรก', 'route': 'home'},
    {'icon': Icons.article, 'label': 'บทความ', 'route': 'articles'},
    {'icon': Icons.favorite, 'label': 'รายการโปรด', 'route': 'favorites'},
    {'icon': Icons.history, 'label': 'ประวัติ', 'route': 'history'},
    {'icon': Icons.shopping_bag, 'label': 'คำสั่งซื้อ', 'route': 'orders'},
    null, // divider
    {'icon': Icons.settings, 'label': 'ตั้งค่า', 'route': 'settings'},
    {'icon': Icons.help_outline, 'label': 'ช่วยเหลือ', 'route': 'help'},
    {'icon': Icons.logout, 'label': 'ออกจากระบบ', 'route': 'logout', 'color': Colors.red},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_currentMenu),
        centerTitle: true,
      ),

      // Drawer
      drawer: NavigationDrawer(
        onDestinationSelected: (index) {
          Navigator.pop(context);
          // handle selection
        },
        children: [
          // Header
          Padding(
            padding: const EdgeInsets.fromLTRB(28, 16, 16, 10),
            child: Row(
              children: [
                const CircleAvatar(
                  radius: 32,
                  backgroundImage:
                      NetworkImage('https://picsum.photos/64/64?random=1'),
                ),
                const SizedBox(width: 16),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text('สมชาย ใจดี',
                          style: TextStyle(
                              fontWeight: FontWeight.bold, fontSize: 16)),
                      Text('somchai@example.com',
                          style: TextStyle(
                              fontSize: 12, color: Colors.grey.shade600)),
                    ],
                  ),
                ),
              ],
            ),
          ),

          const Divider(indent: 28, endIndent: 28),

          // Navigation Items
          const NavigationDrawerDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: Text('หน้าแรก'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.article_outlined),
            selectedIcon: Icon(Icons.article),
            label: Text('บทความ'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.favorite_outline),
            selectedIcon: Icon(Icons.favorite),
            label: Text('รายการโปรด'),
          ),

          const Divider(indent: 28, endIndent: 28),

          Padding(
            padding: const EdgeInsets.fromLTRB(28, 16, 28, 10),
            child: Text('การตั้งค่า',
                style: Theme.of(context).textTheme.labelSmall),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.settings_outlined),
            selectedIcon: Icon(Icons.settings),
            label: Text('ตั้งค่า'),
          ),
          const NavigationDrawerDestination(
            icon: Icon(Icons.logout),
            label: Text('ออกจากระบบ'),
          ),
        ],
      ),

      body: IndexedStack(
        index: _currentIndex,
        children: [
          const Center(child: Text('หน้าแรก', style: TextStyle(fontSize: 24))),
          const Center(child: Text('สำรวจ', style: TextStyle(fontSize: 24))),
          const Center(
              child: Text('แจ้งเตือน', style: TextStyle(fontSize: 24))),
          const Center(
              child: Text('โปรไฟล์', style: TextStyle(fontSize: 24))),
        ],
      ),

      // Navigation Bar (Material 3)
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) {
          setState(() {
            _currentIndex = index;
            _currentMenu = _navItems[index]['label'] as String;
          });
        },
        destinations: _navItems.map((item) {
          return NavigationDestination(
            icon: Icon(item['icon'] as IconData),
            label: item['label'] as String,
          );
        }).toList(),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 70: Workshop - Shopping App Theme

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const ShoppingApp());
}

// Custom Shopping App Theme
class ShoppingTheme {
  static const Color primary = Color(0xFFFF6B6B);
  static const Color primaryDark = Color(0xFFE55555);
  static const Color secondary = Color(0xFF4ECDC4);
  static const Color background = Color(0xFFF7F8FC);
  static const Color textPrimary = Color(0xFF1A1A2E);
  static const Color textSecondary = Color(0xFF6B7280);

  static ThemeData get light {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: primary,
        brightness: Brightness.light,
      ).copyWith(
        primary: primary,
        secondary: secondary,
        surface: Colors.white,
        background: background,
      ),
      scaffoldBackgroundColor: background,
      appBarTheme: const AppBarTheme(
        backgroundColor: Colors.white,
        foregroundColor: textPrimary,
        elevation: 0,
        centerTitle: false,
        titleTextStyle: TextStyle(
          color: textPrimary,
          fontSize: 20,
          fontWeight: FontWeight.bold,
        ),
      ),
      cardTheme: CardThemeData(
        elevation: 0,
        color: Colors.white,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(16),
        ),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          backgroundColor: primary,
          foregroundColor: Colors.white,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          padding: const EdgeInsets.symmetric(vertical: 16),
          elevation: 0,
        ),
      ),
      chipTheme: ChipThemeData(
        backgroundColor: background,
        selectedColor: primary.withOpacity(0.15),
        labelStyle: const TextStyle(color: textPrimary),
        side: BorderSide(color: Colors.grey.shade300),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
    );
  }
}

class ShoppingApp extends StatelessWidget {
  const ShoppingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Shopping App',
      theme: ShoppingTheme.light,
      debugShowCheckedModeBanner: false,
      home: const ShoppingHomeScreen(),
    );
  }
}

class ShoppingHomeScreen extends StatefulWidget {
  const ShoppingHomeScreen({super.key});

  @override
  State<ShoppingHomeScreen> createState() => _ShoppingHomeScreenState();
}

class _ShoppingHomeScreenState extends State<ShoppingHomeScreen> {
  int _cartCount = 0;
  int _bottomNavIndex = 0;
  String _selectedCategory = 'ทั้งหมด';
  final List<bool> _favorites = List.filled(8, false);

  final List<String> _categories = [
    'ทั้งหมด', 'เสื้อผ้า', 'อุปกรณ์', 'รองเท้า', 'กระเป๋า'
  ];

  static final List<Map<String, dynamic>> _products = [
    {
      'name': 'เสื้อ Oversize',
      'price': 390.0,
      'original': 590.0,
      'rating': 4.8,
      'reviews': 234,
      'image': 'https://picsum.photos/200/200?random=70',
      'badge': 'ลด 33%',
    },
    {
      'name': 'กางเกงยีนส์',
      'price': 690.0,
      'original': null,
      'rating': 4.6,
      'reviews': 189,
      'image': 'https://picsum.photos/200/200?random=71',
      'badge': 'ใหม่',
    },
    {
      'name': 'รองเท้าผ้าใบ',
      'price': 1290.0,
      'original': 1590.0,
      'rating': 4.9,
      'reviews': 456,
      'image': 'https://picsum.photos/200/200?random=72',
      'badge': null,
    },
    {
      'name': 'กระเป๋าเป้',
      'price': 890.0,
      'original': null,
      'rating': 4.7,
      'reviews': 123,
      'image': 'https://picsum.photos/200/200?random=73',
      'badge': 'ฮิต',
    },
    {
      'name': 'แจ็กเก็ต',
      'price': 1490.0,
      'original': 1990.0,
      'rating': 4.5,
      'reviews': 89,
      'image': 'https://picsum.photos/200/200?random=74',
      'badge': 'ลด 25%',
    },
    {
      'name': 'หมวก Cap',
      'price': 290.0,
      'original': null,
      'rating': 4.3,
      'reviews': 67,
      'image': 'https://picsum.photos/200/200?random=75',
      'badge': null,
    },
    {
      'name': 'ถุงเท้า 3 คู่',
      'price': 149.0,
      'original': 199.0,
      'rating': 4.1,
      'reviews': 312,
      'image': 'https://picsum.photos/200/200?random=76',
      'badge': null,
    },
    {
      'name': 'เข็มขัด',
      'price': 350.0,
      'original': null,
      'rating': 4.4,
      'reviews': 45,
      'image': 'https://picsum.photos/200/200?random=77',
      'badge': null,
    },
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: _buildAppBar(context),
      body: _buildBody(),
      bottomNavigationBar: _buildBottomNav(),
    );
  }

  PreferredSizeWidget _buildAppBar(BuildContext context) {
    return AppBar(
      title: const Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('สวัสดี สมชาย 👋',
              style: TextStyle(fontSize: 14, color: Colors.grey)),
          Text('ค้นหาสินค้า', style: TextStyle(fontSize: 20)),
        ],
      ),
      actions: [
        Stack(
          children: [
            IconButton(
              icon: const Icon(Icons.shopping_cart_outlined),
              onPressed: () => _showCart(context),
            ),
            if (_cartCount > 0)
              Positioned(
                top: 6,
                right: 6,
                child: Container(
                  width: 18,
                  height: 18,
                  decoration: const BoxDecoration(
                    color: ShoppingTheme.primary,
                    shape: BoxShape.circle,
                  ),
                  child: Center(
                    child: Text(
                      '$_cartCount',
                      style: const TextStyle(
                          color: Colors.white, fontSize: 10),
                    ),
                  ),
                ),
              ),
          ],
        ),
        const CircleAvatar(
          radius: 16,
          backgroundImage:
              NetworkImage('https://picsum.photos/32/32?random=1'),
        ),
        const SizedBox(width: 12),
      ],
    );
  }

  Widget _buildBody() {
    return CustomScrollView(
      slivers: [
        SliverToBoxAdapter(
          child: Column(
            children: [
              _buildSearchBar(),
              _buildBanner(),
              _buildCategories(),
            ],
          ),
        ),
        SliverPadding(
          padding: const EdgeInsets.all(16),
          sliver: SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              crossAxisSpacing: 12,
              mainAxisSpacing: 12,
              childAspectRatio: 0.72,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) => _buildProductCard(index),
              childCount: _products.length,
            ),
          ),
        ),
        const SliverPadding(padding: EdgeInsets.only(bottom: 16)),
      ],
    );
  }

  Widget _buildSearchBar() {
    return Padding(
      padding: const EdgeInsets.fromLTRB(16, 8, 16, 0),
      child: Container(
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
        child: TextField(
          decoration: InputDecoration(
            hintText: 'ค้นหาสินค้า...',
            prefixIcon: const Icon(Icons.search, color: Colors.grey),
            suffixIcon: Container(
              margin: const EdgeInsets.all(8),
              decoration: BoxDecoration(
                color: ShoppingTheme.primary,
                borderRadius: BorderRadius.circular(10),
              ),
              child: const Icon(Icons.tune, color: Colors.white, size: 20),
            ),
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(16),
              borderSide: BorderSide.none,
            ),
            filled: true,
            fillColor: Colors.white,
          ),
        ),
      ),
    );
  }

  Widget _buildBanner() {
    return Container(
      margin: const EdgeInsets.all(16),
      height: 160,
      child: PageView.builder(
        itemCount: 3,
        itemBuilder: (context, index) {
          final colors = [
            [const Color(0xFF6C63FF), const Color(0xFF3B82F6)],
            [const Color(0xFFFF6B6B), const Color(0xFFFF8E53)],
            [const Color(0xFF4ECDC4), const Color(0xFF44A08D)],
          ];
          final titles = ['Flash Sale', 'New Arrivals', 'Free Shipping'];
          final subtitles = ['ลดสูงสุด 70%', 'สินค้าใหม่มาแล้ว', 'สั่งครบ ฿500'];

          return Container(
            margin: const EdgeInsets.symmetric(horizontal: 4),
            decoration: BoxDecoration(
              gradient: LinearGradient(colors: colors[index]),
              borderRadius: BorderRadius.circular(20),
            ),
            child: Stack(
              children: [
                Positioned(
                  right: -20,
                  bottom: -20,
                  child: Container(
                    width: 150,
                    height: 150,
                    decoration: BoxDecoration(
                      color: Colors.white.withOpacity(0.1),
                      shape: BoxShape.circle,
                    ),
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.all(24),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        subtitles[index],
                        style: const TextStyle(
                          color: Colors.white70,
                          fontSize: 14,
                        ),
                      ),
                      Text(
                        titles[index],
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 28,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 12),
                      Container(
                        padding: const EdgeInsets.symmetric(
                            horizontal: 16, vertical: 8),
                        decoration: BoxDecoration(
                          color: Colors.white,
                          borderRadius: BorderRadius.circular(20),
                        ),
                        child: Text(
                          'ช้อปเลย',
                          style: TextStyle(
                            color: colors[index][0],
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _buildCategories() {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              const Text('หมวดหมู่',
                  style: TextStyle(
                      fontSize: 18, fontWeight: FontWeight.bold)),
              TextButton(onPressed: () {}, child: const Text('ดูทั้งหมด')),
            ],
          ),
          const SizedBox(height: 8),
          SizedBox(
            height: 40,
            child: ListView.separated(
              scrollDirection: Axis.horizontal,
              itemCount: _categories.length,
              separatorBuilder: (_, __) => const SizedBox(width: 8),
              itemBuilder: (context, index) {
                final cat = _categories[index];
                final isSelected = cat == _selectedCategory;
                return ChoiceChip(
                  label: Text(cat),
                  selected: isSelected,
                  onSelected: (_) =>
                      setState(() => _selectedCategory = cat),
                  selectedColor: ShoppingTheme.primary.withOpacity(0.2),
                  labelStyle: TextStyle(
                    color: isSelected
                        ? ShoppingTheme.primary
                        : ShoppingTheme.textPrimary,
                    fontWeight: isSelected
                        ? FontWeight.bold
                        : FontWeight.normal,
                  ),
                  side: isSelected
                      ? const BorderSide(
                          color: ShoppingTheme.primary, width: 1.5)
                      : null,
                );
              },
            ),
          ),
          const SizedBox(height: 8),
        ],
      ),
    );
  }

  Widget _buildProductCard(int index) {
    final product = _products[index];
    final isFav = _favorites[index];

    return Card(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            flex: 3,
            child: Stack(
              fit: StackFit.expand,
              children: [
                ClipRRect(
                  borderRadius: const BorderRadius.vertical(
                      top: Radius.circular(16)),
                  child: Image.network(
                    product['image'] as String,
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) => Container(
                      color: Colors.grey.shade100,
                    ),
                  ),
                ),
                if (product['badge'] != null)
                  Positioned(
                    top: 8,
                    left: 8,
                    child: Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 4),
                      decoration: BoxDecoration(
                        color: ShoppingTheme.primary,
                        borderRadius: BorderRadius.circular(6),
                      ),
                      child: Text(
                        product['badge'] as String,
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 10,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                Positioned(
                  top: 4,
                  right: 4,
                  child: IconButton(
                    icon: Icon(
                      isFav ? Icons.favorite : Icons.favorite_border,
                      color: isFav ? ShoppingTheme.primary : Colors.white,
                    ),
                    onPressed: () =>
                        setState(() => _favorites[index] = !isFav),
                    style: IconButton.styleFrom(
                      backgroundColor: Colors.black26,
                      minimumSize: const Size(32, 32),
                    ),
                  ),
                ),
              ],
            ),
          ),
          Expanded(
            flex: 2,
            child: Padding(
              padding: const EdgeInsets.all(10),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    product['name'] as String,
                    style: const TextStyle(fontWeight: FontWeight.w600),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 2),
                  Row(
                    children: [
                      const Icon(Icons.star,
                          color: Colors.amber, size: 12),
                      Text(
                        ' ${product['rating']}',
                        style: const TextStyle(fontSize: 11),
                      ),
                      Text(
                        ' (${product['reviews']})',
                        style: const TextStyle(
                            fontSize: 11, color: Colors.grey),
                      ),
                    ],
                  ),
                  const Spacer(),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          Text(
                            '฿${(product['price'] as double).toStringAsFixed(0)}',
                            style: const TextStyle(
                              color: ShoppingTheme.primary,
                              fontWeight: FontWeight.bold,
                              fontSize: 15,
                            ),
                          ),
                          if (product['original'] != null)
                            Text(
                              '฿${(product['original'] as double).toStringAsFixed(0)}',
                              style: const TextStyle(
                                color: Colors.grey,
                                fontSize: 11,
                                decoration: TextDecoration.lineThrough,
                              ),
                            ),
                        ],
                      ),
                      GestureDetector(
                        onTap: () {
                          setState(() => _cartCount++);
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(
                              content: Text(
                                  '${product['name']} เพิ่มในตะกร้าแล้ว'),
                              behavior: SnackBarBehavior.floating,
                              shape: RoundedRectangleBorder(
                                borderRadius: BorderRadius.circular(10),
                              ),
                              duration: const Duration(seconds: 2),
                            ),
                          );
                        },
                        child: Container(
                          width: 32,
                          height: 32,
                          decoration: BoxDecoration(
                            color: ShoppingTheme.primary,
                            borderRadius: BorderRadius.circular(8),
                          ),
                          child: const Icon(Icons.add,
                              color: Colors.white, size: 18),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildBottomNav() {
    return NavigationBar(
      selectedIndex: _bottomNavIndex,
      onDestinationSelected: (i) =>
          setState(() => _bottomNavIndex = i),
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
    );
  }

  void _showCart(BuildContext context) {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.5,
        maxChildSize: 0.9,
        expand: false,
        builder: (_, controller) => Column(
          children: [
            const SizedBox(height: 8),
            Container(
              width: 40,
              height: 4,
              decoration: BoxDecoration(
                color: Colors.grey.shade300,
                borderRadius: BorderRadius.circular(2),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                children: [
                  const Text('ตะกร้าสินค้า',
                      style: TextStyle(
                          fontSize: 18, fontWeight: FontWeight.bold)),
                  const Spacer(),
                  Text(
                    '$_cartCount รายการ',
                    style: const TextStyle(color: Colors.grey),
                  ),
                ],
              ),
            ),
            const Divider(height: 1),
            if (_cartCount == 0)
              const Expanded(
                child: Center(
                  child: Column(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Icon(Icons.shopping_cart_outlined,
                          size: 64, color: Colors.grey),
                      SizedBox(height: 16),
                      Text('ตะกร้าว่างเปล่า',
                          style: TextStyle(color: Colors.grey)),
                    ],
                  ),
                ),
              )
            else
              const Expanded(
                child: Center(
                  child: Text('มีสินค้าในตะกร้า'),
                ),
              ),
            Padding(
              padding: const EdgeInsets.all(16),
              child: SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _cartCount > 0 ? () {} : null,
                  child: Text('ชำระเงิน (฿${_cartCount * 390})'),
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

## สรุป (Summary)

ใน Part 07 เราเรียน Material Design & Theming:

| หัวข้อ | สาระสำคัญ |
|--------|-----------|
| MaterialApp | Root widget พร้อม theme, routes, locale |
| ThemeData | กำหนดรูปลักษณ์ทั้งแอป |
| ColorScheme | 30+ color roles ตาม Material 3 |
| Dark Mode | themeMode: ThemeMode.system/light/dark |
| Custom Theme | สร้าง brand theme ของตัวเอง |
| Card | 3 แบบ: elevated, filled, outlined |
| Chip | 5 แบบ: Chip, Filter, Choice, Action, Input |
| Dialog | AlertDialog, SimpleDialog, Dialog.fullscreen |
| BottomSheet | Modal, Draggable, Persistent |
| SnackBar | Basic, Action, Floating, Colored |

---

## แบบฝึกหัด (Exercises)

1. **Brand Theme**: สร้าง theme สำหรับแบรนด์สมมติ พร้อม light/dark mode เปลี่ยน primary color, typography, border radius

2. **E-commerce Product Page**: สร้างหน้า product detail พร้อม image, title, price, reviews, add to cart button ใช้ theme colors ทั้งหมด

3. **Settings Screen**: สร้าง settings page ที่มี switches, sliders, dropdowns พร้อม theme preview

4. **Notification Feed**: สร้าง notification list พร้อม chips เพื่อกรอง (ทั้งหมด, อ่านแล้ว, ยังไม่ได้อ่าน)

5. **Custom Dialog**: สร้าง dialog แบบ custom ที่มี animation และ form input เพื่อเพิ่มรายการ

---

[← Part 06: Stateful & Stateless Widgets](part_06.md) | [Part 08: Forms และ Input Handling →](part_08.md)

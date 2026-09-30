# Part 38: Flutter Desktop
## ขั้นตอนที่ 371-380

---

## สารบัญ
1. [Flutter Desktop Overview](#flutter-desktop-overview)
2. [macOS/Windows/Linux Setup](#setup)
3. [Window Management](#window-management)
4. [System Tray](#system-tray)
5. [Native Menus](#native-menus)
6. [Keyboard Shortcuts](#keyboard-shortcuts)
7. [File System Access](#file-system-access)
8. [Platform-specific Features](#platform-specific-features)
9. [Packaging (MSIX, DMG)](#packaging)
10. [Desktop-specific Layouts](#desktop-layouts)

---

## ขั้นตอนที่ 371: Flutter Desktop Overview

Flutter Desktop ช่วยให้เราสร้าง Native Desktop apps สำหรับ macOS, Windows, Linux

```
Desktop Platforms:
┌─────────────────────────────────────────────────────────┐
│  macOS    → .app bundle, DMG installer                  │
│  Windows  → .exe, MSIX package (Windows Store)          │
│  Linux    → Snap, AppImage, .deb, .rpm                  │
└─────────────────────────────────────────────────────────┘

Requirements:
macOS:  - Xcode 13+, macOS 12+
        - Flutter macOS SDK
        
Windows: - Visual Studio 2019+ with Desktop workload
         - Windows 10 SDK
         
Linux:   - clang, cmake, ninja-build, pkg-config
         - GTK 3.0, libblkid, liblzma
```

### pubspec.yaml สำหรับ Desktop

```yaml
name: my_desktop_app
description: Flutter Desktop Application

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  
  # Window Management
  window_manager: ^0.3.8
  
  # System Tray
  tray_manager: ^0.2.0
  
  # File System
  file_picker: ^6.1.1
  path_provider: ^2.1.1
  path: ^1.8.3
  
  # Hotkeys
  hotkey_manager: ^0.2.0
  
  # Local Notifications
  local_notifier: ^0.1.6
  
  # Screen utilities
  screen_retriever: ^0.1.9
  
  # Database (Desktop)
  drift: ^2.14.1
  sqlite3_flutter_libs: ^0.5.18
  
  # Clipboard
  super_clipboard: ^0.7.5
```

---

## ขั้นตอนที่ 372: macOS/Windows/Linux Setup

```bash
# ตรวจสอบ Flutter Desktop support
flutter doctor

# เพิ่ม Desktop support
flutter create --platforms macos .
flutter create --platforms windows .
flutter create --platforms linux .

# หรือเพิ่มทั้งหมด
flutter create --platforms macos,windows,linux .

# รัน Desktop app
flutter run -d macos
flutter run -d windows
flutter run -d linux

# Build
flutter build macos
flutter build windows
flutter build linux
```

### macOS Entitlements

```xml
<!-- macos/Runner/DebugProfile.entitlements -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <!-- อนุญาต network access -->
  <key>com.apple.security.network.client</key>
  <true/>
  <!-- อนุญาต file access -->
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
  <!-- อนุญาต Downloads folder -->
  <key>com.apple.security.files.downloads.read-write</key>
  <true/>
</dict>
</plist>
```

```xml
<!-- macos/Runner/Release.entitlements (for App Store) -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.network.client</key>
  <true/>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
</plist>
```

### Windows Manifest

```xml
<!-- windows/runner/Runner.manifest -->
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
    <application>
      <!-- Windows 10 -->
      <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"/>
    </application>
  </compatibility>
  <application xmlns="urn:schemas-microsoft-com:asm.v3">
    <windowsSettings>
      <dpiAware xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">
        true/pm
      </dpiAware>
      <dpiAwareness xmlns="http://schemas.microsoft.com/SMI/2016/WindowsSettings">
        PerMonitorV2
      </dpiAwareness>
    </windowsSettings>
  </application>
</assembly>
```

---

## ขั้นตอนที่ 373: Window Management

```dart
// lib/core/window/window_manager_helper.dart
import 'package:flutter/material.dart';
import 'package:window_manager/window_manager.dart';

class WindowManagerHelper {
  static Future<void> initialize() async {
    await windowManager.ensureInitialized();
    
    const windowOptions = WindowOptions(
      size: Size(1280, 800),
      minimumSize: Size(800, 600),
      center: true,
      backgroundColor: Colors.transparent,
      skipTaskbar: false,
      titleBarStyle: TitleBarStyle.normal,
      title: 'My Desktop App',
    );
    
    await windowManager.waitUntilReadyToShow(windowOptions, () async {
      await windowManager.show();
      await windowManager.focus();
    });
  }
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  if (isDesktop) {
    await WindowManagerHelper.initialize();
  }
  
  runApp(const MyApp());
}

bool get isDesktop =>
    defaultTargetPlatform == TargetPlatform.macOS ||
    defaultTargetPlatform == TargetPlatform.windows ||
    defaultTargetPlatform == TargetPlatform.linux;
```

### Custom Title Bar

```dart
// Custom Title Bar Widget
class CustomTitleBar extends StatefulWidget with PreferredSizeWidget {
  final String title;
  
  const CustomTitleBar({super.key, required this.title});

  @override
  Size get preferredSize => const Size.fromHeight(40);

  @override
  State<CustomTitleBar> createState() => _CustomTitleBarState();
}

class _CustomTitleBarState extends State<CustomTitleBar> with WindowListener {
  bool _isMaximized = false;
  bool _isAlwaysOnTop = false;

  @override
  void initState() {
    super.initState();
    windowManager.addListener(this);
    _checkMaximized();
  }

  @override
  void dispose() {
    windowManager.removeListener(this);
    super.dispose();
  }

  Future<void> _checkMaximized() async {
    _isMaximized = await windowManager.isMaximized();
    setState(() {});
  }

  @override
  void onWindowMaximize() {
    setState(() => _isMaximized = true);
  }

  @override
  void onWindowUnmaximize() {
    setState(() => _isMaximized = false);
  }

  @override
  Widget build(BuildContext context) {
    return DragToMoveArea(
      child: Container(
        color: Theme.of(context).colorScheme.surface,
        child: Row(
          children: [
            // App Icon
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 12),
              child: Image.asset('assets/icon.png', width: 20, height: 20),
            ),
            
            // Title
            Expanded(
              child: Text(
                widget.title,
                style: Theme.of(context).textTheme.labelMedium,
                overflow: TextOverflow.ellipsis,
              ),
            ),
            
            // Window Controls (Windows style)
            if (defaultTargetPlatform == TargetPlatform.windows)
              _WindowsControls(isMaximized: _isMaximized),
          ],
        ),
      ),
    );
  }
}

class _WindowsControls extends StatelessWidget {
  final bool isMaximized;
  
  const _WindowsControls({required this.isMaximized});

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // Minimize
        _ControlButton(
          icon: Icons.minimize,
          tooltip: 'Minimize',
          onPressed: () => windowManager.minimize(),
        ),
        
        // Maximize/Restore
        _ControlButton(
          icon: isMaximized ? Icons.fullscreen_exit : Icons.fullscreen,
          tooltip: isMaximized ? 'Restore' : 'Maximize',
          onPressed: () {
            if (isMaximized) {
              windowManager.unmaximize();
            } else {
              windowManager.maximize();
            }
          },
        ),
        
        // Close
        _ControlButton(
          icon: Icons.close,
          tooltip: 'Close',
          hoverColor: Colors.red,
          onPressed: () => windowManager.close(),
        ),
      ],
    );
  }
}

class _ControlButton extends StatefulWidget {
  final IconData icon;
  final String tooltip;
  final VoidCallback onPressed;
  final Color? hoverColor;

  const _ControlButton({
    required this.icon,
    required this.tooltip,
    required this.onPressed,
    this.hoverColor,
  });

  @override
  State<_ControlButton> createState() => _ControlButtonState();
}

class _ControlButtonState extends State<_ControlButton> {
  bool _isHovered = false;

  @override
  Widget build(BuildContext context) {
    return Tooltip(
      message: widget.tooltip,
      child: MouseRegion(
        onEnter: (_) => setState(() => _isHovered = true),
        onExit: (_) => setState(() => _isHovered = false),
        child: GestureDetector(
          onTap: widget.onPressed,
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 100),
            width: 46,
            height: 40,
            color: _isHovered
                ? (widget.hoverColor ?? Theme.of(context).hoverColor)
                : Colors.transparent,
            child: Icon(widget.icon, size: 18),
          ),
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 374: System Tray

```dart
// lib/core/tray/tray_manager_helper.dart
import 'package:flutter/foundation.dart';
import 'package:tray_manager/tray_manager.dart';

class TrayManagerHelper with TrayListener {
  TrayManagerHelper._();
  static final instance = TrayManagerHelper._();

  Future<void> initialize() async {
    trayManager.addListener(this);
    
    // Set tray icon
    await trayManager.setIcon(
      defaultTargetPlatform == TargetPlatform.windows
          ? 'assets/tray_icon.ico'
          : 'assets/tray_icon.png',
    );
    
    // Set tooltip
    await trayManager.setToolTip('My Desktop App');
    
    // Set context menu
    await _setupContextMenu();
  }

  Future<void> _setupContextMenu() async {
    final menu = Menu(
      items: [
        MenuItem(
          key: 'show_window',
          label: 'Show Window',
        ),
        MenuItem.separator(),
        MenuItem(
          key: 'preferences',
          label: 'Preferences',
        ),
        MenuItem(
          key: 'check_updates',
          label: 'Check for Updates',
        ),
        MenuItem.separator(),
        MenuItem(
          key: 'quit',
          label: 'Quit',
        ),
      ],
    );
    
    await trayManager.setContextMenu(menu);
  }

  @override
  void onTrayIconMouseDown() {
    // Show/hide window on tray icon click
    windowManager.show();
    windowManager.focus();
  }

  @override
  void onTrayIconRightMouseDown() {
    trayManager.popUpContextMenu();
  }

  @override
  void onTrayMenuItemClick(MenuItem menuItem) {
    switch (menuItem.key) {
      case 'show_window':
        windowManager.show();
        windowManager.focus();
        break;
      case 'preferences':
        // Navigate to preferences
        break;
      case 'check_updates':
        // Check for updates
        break;
      case 'quit':
        windowManager.destroy();
        break;
    }
  }

  void dispose() {
    trayManager.removeListener(this);
  }
}
```

### Minimize to Tray

```dart
// ซ่อน app ไปที่ System Tray แทนการปิด
class _AppState extends State<App> with WindowListener {
  @override
  void initState() {
    super.initState();
    windowManager.addListener(this);
  }

  @override
  void dispose() {
    windowManager.removeListener(this);
    super.dispose();
  }

  @override
  void onWindowClose() async {
    // แสดง dialog ถามว่าจะปิดหรือ minimize to tray
    final action = await showDialog<String>(
      context: navigatorKey.currentContext!,
      builder: (context) => AlertDialog(
        title: const Text('Close App'),
        content: const Text('Would you like to minimize to tray or quit?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, 'minimize'),
            child: const Text('Minimize to Tray'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, 'quit'),
            child: const Text('Quit'),
          ),
        ],
      ),
    );
    
    if (action == 'minimize') {
      await windowManager.hide(); // ซ่อน window แต่ยังทำงานอยู่
    } else {
      await windowManager.destroy(); // ปิดจริงๆ
    }
  }
}
```

---

## ขั้นตอนที่ 375: Native Menus

```dart
// Native Menu Bar สำหรับ macOS/Windows/Linux
class DesktopMenuBar extends StatelessWidget {
  const DesktopMenuBar({super.key});

  @override
  Widget build(BuildContext context) {
    return PlatformMenuBar(
      menus: [
        // App Menu (macOS only)
        if (defaultTargetPlatform == TargetPlatform.macOS)
          PlatformMenu(
            label: 'My App',
            menus: [
              PlatformMenuItem(
                label: 'About My App',
                onSelected: () => _showAboutDialog(context),
              ),
              PlatformMenuItem(
                label: 'Preferences',
                shortcut: const SingleActivator(LogicalKeyboardKey.comma, meta: true),
                onSelected: () => _openPreferences(context),
              ),
              const PlatformMenuDivider(),
              PlatformMenuItem(
                label: 'Quit My App',
                shortcut: const SingleActivator(LogicalKeyboardKey.keyQ, meta: true),
                onSelected: () => windowManager.destroy(),
              ),
            ],
          ),
        
        // File Menu
        PlatformMenu(
          label: 'File',
          menus: [
            PlatformMenuItem(
              label: 'New',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyN,
                  control: true, meta: true),
              onSelected: () => _createNew(context),
            ),
            PlatformMenuItem(
              label: 'Open...',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyO,
                  control: true, meta: true),
              onSelected: () => _openFile(context),
            ),
            PlatformMenuItem(
              label: 'Save',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyS,
                  control: true, meta: true),
              onSelected: () => _saveFile(context),
            ),
            PlatformMenuItem(
              label: 'Save As...',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyS,
                  shift: true, control: true, meta: true),
              onSelected: () => _saveFileAs(context),
            ),
            const PlatformMenuDivider(),
            if (defaultTargetPlatform != TargetPlatform.macOS)
              PlatformMenuItem(
                label: 'Exit',
                shortcut: const SingleActivator(LogicalKeyboardKey.keyQ,
                    control: true),
                onSelected: () => windowManager.destroy(),
              ),
          ],
        ),
        
        // Edit Menu
        PlatformMenu(
          label: 'Edit',
          menus: [
            PlatformMenuItem(
              label: 'Undo',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyZ,
                  control: true, meta: true),
              onSelected: () => _undo(context),
            ),
            PlatformMenuItem(
              label: 'Redo',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyZ,
                  shift: true, control: true, meta: true),
              onSelected: () => _redo(context),
            ),
            const PlatformMenuDivider(),
            PlatformMenuItem(
              label: 'Cut',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyX,
                  control: true, meta: true),
              onSelected: () {},
            ),
            PlatformMenuItem(
              label: 'Copy',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyC,
                  control: true, meta: true),
              onSelected: () {},
            ),
            PlatformMenuItem(
              label: 'Paste',
              shortcut: const SingleActivator(LogicalKeyboardKey.keyV,
                  control: true, meta: true),
              onSelected: () {},
            ),
          ],
        ),
        
        // View Menu
        PlatformMenu(
          label: 'View',
          menus: [
            PlatformMenuItem(
              label: 'Zoom In',
              shortcut: const SingleActivator(LogicalKeyboardKey.equal,
                  control: true, meta: true),
              onSelected: () => _zoomIn(context),
            ),
            PlatformMenuItem(
              label: 'Zoom Out',
              shortcut: const SingleActivator(LogicalKeyboardKey.minus,
                  control: true, meta: true),
              onSelected: () => _zoomOut(context),
            ),
            PlatformMenuItem(
              label: 'Reset Zoom',
              shortcut: const SingleActivator(LogicalKeyboardKey.digit0,
                  control: true, meta: true),
              onSelected: () => _resetZoom(context),
            ),
          ],
        ),
        
        // Help Menu
        PlatformMenu(
          label: 'Help',
          menus: [
            PlatformMenuItem(
              label: 'Documentation',
              onSelected: () => _openDocumentation(),
            ),
            PlatformMenuItem(
              label: 'Check for Updates',
              onSelected: () => _checkUpdates(context),
            ),
          ],
        ),
      ],
      child: const Scaffold(
        body: MainContent(),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 376: Keyboard Shortcuts

```dart
// lib/core/shortcuts/keyboard_shortcuts.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:hotkey_manager/hotkey_manager.dart';

class AppShortcuts {
  // Global Hotkeys (ทำงานแม้ app ไม่ได้ focus)
  static Future<void> registerGlobalHotkeys() async {
    await hotKeyManager.unregisterAll();
    
    // Ctrl+Shift+Space - Show/Hide Window
    await hotKeyManager.register(
      HotKey(
        key: PhysicalKeyboardKey.space,
        modifiers: [HotKeyModifier.control, HotKeyModifier.shift],
        scope: HotKeyScope.system,
      ),
      keyDownHandler: (_) async {
        final isVisible = await windowManager.isVisible();
        if (isVisible) {
          await windowManager.hide();
        } else {
          await windowManager.show();
          await windowManager.focus();
        }
      },
    );
  }
}

// In-app Shortcuts
class AppKeyboardShortcuts extends StatelessWidget {
  final Widget child;
  
  const AppKeyboardShortcuts({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return Shortcuts(
      shortcuts: const {
        // File operations
        SingleActivator(LogicalKeyboardKey.keyN, control: true): NewFileIntent(),
        SingleActivator(LogicalKeyboardKey.keyO, control: true): OpenFileIntent(),
        SingleActivator(LogicalKeyboardKey.keyS, control: true): SaveIntent(),
        SingleActivator(LogicalKeyboardKey.keyS, control: true, shift: true): SaveAsIntent(),
        
        // Edit operations
        SingleActivator(LogicalKeyboardKey.keyZ, control: true): UndoTextIntent(SelectionChangedCause.keyboard),
        SingleActivator(LogicalKeyboardKey.keyZ, control: true, shift: true): RedoTextIntent(SelectionChangedCause.keyboard),
        
        // Navigation
        SingleActivator(LogicalKeyboardKey.f5): RefreshIntent(),
        
        // macOS alternatives (meta key)
        SingleActivator(LogicalKeyboardKey.keyN, meta: true): NewFileIntent(),
        SingleActivator(LogicalKeyboardKey.keyO, meta: true): OpenFileIntent(),
        SingleActivator(LogicalKeyboardKey.keyS, meta: true): SaveIntent(),
      },
      child: Actions(
        actions: {
          NewFileIntent: CallbackAction<NewFileIntent>(
            onInvoke: (_) => _createNewFile(context),
          ),
          OpenFileIntent: CallbackAction<OpenFileIntent>(
            onInvoke: (_) => _openFile(context),
          ),
          SaveIntent: CallbackAction<SaveIntent>(
            onInvoke: (_) => _saveFile(context),
          ),
          RefreshIntent: CallbackAction<RefreshIntent>(
            onInvoke: (_) => _refresh(context),
          ),
        },
        child: Focus(
          autofocus: true,
          child: child,
        ),
      ),
    );
  }
  
  Object? _createNewFile(BuildContext context) {
    // TODO: implement
    return null;
  }
  
  Object? _openFile(BuildContext context) {
    // TODO: implement
    return null;
  }
  
  Object? _saveFile(BuildContext context) {
    // TODO: implement
    return null;
  }
  
  Object? _refresh(BuildContext context) {
    // TODO: implement
    return null;
  }
}

// Custom Intents
class NewFileIntent extends Intent { const NewFileIntent(); }
class OpenFileIntent extends Intent { const OpenFileIntent(); }
class SaveIntent extends Intent { const SaveIntent(); }
class SaveAsIntent extends Intent { const SaveAsIntent(); }
class RefreshIntent extends Intent { const RefreshIntent(); }
```

---

## ขั้นตอนที่ 377: File System Access

```dart
// lib/core/file/file_service.dart
import 'dart:io';
import 'package:file_picker/file_picker.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class FileService {
  // เปิด file picker
  static Future<File?> pickFile({
    List<String>? allowedExtensions,
    String? dialogTitle,
  }) async {
    final result = await FilePicker.platform.pickFiles(
      dialogTitle: dialogTitle ?? 'Select File',
      type: allowedExtensions != null ? FileType.custom : FileType.any,
      allowedExtensions: allowedExtensions,
      allowMultiple: false,
    );
    
    if (result == null || result.files.isEmpty) return null;
    
    final filePath = result.files.first.path;
    if (filePath == null) return null;
    
    return File(filePath);
  }

  // เลือก directory
  static Future<Directory?> pickDirectory({String? dialogTitle}) async {
    final path = await FilePicker.platform.getDirectoryPath(
      dialogTitle: dialogTitle ?? 'Select Directory',
    );
    
    if (path == null) return null;
    return Directory(path);
  }

  // บันทึกไฟล์
  static Future<File?> saveFile({
    required String defaultName,
    List<String>? allowedExtensions,
    String? dialogTitle,
    List<int>? bytes,
    String? content,
  }) async {
    final outputFile = await FilePicker.platform.saveFile(
      dialogTitle: dialogTitle ?? 'Save File',
      fileName: defaultName,
      type: allowedExtensions != null ? FileType.custom : FileType.any,
      allowedExtensions: allowedExtensions,
      bytes: bytes != null ? Uint8List.fromList(bytes) : null,
    );
    
    if (outputFile == null) return null;
    
    final file = File(outputFile);
    
    if (content != null) {
      await file.writeAsString(content);
    }
    
    return file;
  }

  // อ่านไฟล์ Text
  static Future<String?> readTextFile(File file) async {
    try {
      return await file.readAsString();
    } catch (e) {
      debugPrint('Error reading file: $e');
      return null;
    }
  }

  // อ่านไฟล์ JSON
  static Future<Map<String, dynamic>?> readJsonFile(File file) async {
    final content = await readTextFile(file);
    if (content == null) return null;
    
    try {
      return json.decode(content) as Map<String, dynamic>;
    } catch (e) {
      debugPrint('Error parsing JSON: $e');
      return null;
    }
  }

  // App Support Directory
  static Future<Directory> getAppSupportDirectory() async {
    return await getApplicationSupportDirectory();
  }

  // Documents Directory
  static Future<Directory> getDocumentsDirectory() async {
    return await getApplicationDocumentsDirectory();
  }

  // Desktop Directory
  static Future<Directory?> getDesktopDirectory() async {
    if (!Platform.isWindows && !Platform.isMacOS && !Platform.isLinux) {
      return null;
    }
    
    final homeDir = Platform.environment['HOME'] ??
        Platform.environment['USERPROFILE'];
    
    if (homeDir == null) return null;
    
    final desktopPath = path.join(homeDir, 'Desktop');
    final dir = Directory(desktopPath);
    
    return await dir.exists() ? dir : null;
  }

  // ตรวจสอบ file size
  static String formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) {
      return '${(bytes / 1024).toStringAsFixed(1)} KB';
    }
    return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
  }
}
```

```dart
// File Manager Widget
class FileManagerPage extends StatefulWidget {
  const FileManagerPage({super.key});

  @override
  State<FileManagerPage> createState() => _FileManagerPageState();
}

class _FileManagerPageState extends State<FileManagerPage> {
  Directory? _currentDirectory;
  List<FileSystemEntity> _entities = [];
  bool _isLoading = false;

  @override
  void initState() {
    super.initState();
    _loadInitialDirectory();
  }

  Future<void> _loadInitialDirectory() async {
    final docsDir = await FileService.getDocumentsDirectory();
    await _loadDirectory(docsDir);
  }

  Future<void> _loadDirectory(Directory dir) async {
    setState(() => _isLoading = true);
    
    try {
      final entities = await dir.list().toList();
      entities.sort((a, b) {
        if (a is Directory && b is File) return -1;
        if (a is File && b is Directory) return 1;
        return path.basename(a.path).compareTo(path.basename(b.path));
      });
      
      setState(() {
        _currentDirectory = dir;
        _entities = entities;
        _isLoading = false;
      });
    } catch (e) {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_currentDirectory?.path ?? 'Files'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () => _currentDirectory != null
                ? _loadDirectory(_currentDirectory!)
                : null,
          ),
        ],
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: _entities.length,
              itemBuilder: (context, index) {
                final entity = _entities[index];
                return ListTile(
                  leading: Icon(
                    entity is Directory ? Icons.folder : Icons.insert_drive_file,
                    color: entity is Directory ? Colors.amber : Colors.blue,
                  ),
                  title: Text(path.basename(entity.path)),
                  subtitle: entity is File
                      ? FutureBuilder<FileStat>(
                          future: entity.stat(),
                          builder: (context, snapshot) {
                            if (!snapshot.hasData) return const SizedBox.shrink();
                            return Text(
                              FileService.formatFileSize(snapshot.data!.size),
                            );
                          },
                        )
                      : null,
                  onTap: () {
                    if (entity is Directory) {
                      _loadDirectory(entity);
                    } else {
                      _openFile(entity as File);
                    }
                  },
                );
              },
            ),
    );
  }

  Future<void> _openFile(File file) async {
    // เปิดไฟล์ด้วย default app
    // ใช้ url_launcher หรือ open_file package
  }
}
```

---

## ขั้นตอนที่ 378: Platform-specific Features

```dart
// lib/core/platform/platform_features.dart
import 'dart:io';
import 'package:flutter/foundation.dart';

class PlatformFeatures {
  // ตรวจสอบ Platform
  static bool get isMacOS => !kIsWeb && Platform.isMacOS;
  static bool get isWindows => !kIsWeb && Platform.isWindows;
  static bool get isLinux => !kIsWeb && Platform.isLinux;
  static bool get isDesktop => isMacOS || isWindows || isLinux;
  static bool get isMobile => !kIsWeb && (Platform.isAndroid || Platform.isIOS);

  // macOS-specific: Haptic Feedback
  static Future<void> hapticFeedback() async {
    if (isMacOS) {
      // NSHapticFeedbackManager
    }
  }

  // macOS-specific: Badge count
  static Future<void> setBadgeCount(int count) async {
    if (isMacOS) {
      // macOS Dock badge
    }
  }

  // Windows-specific: Jump List
  static void setupJumpList() {
    if (isWindows) {
      // Windows Taskbar Jump List items
    }
  }

  // Notifications
  static Future<void> showNotification({
    required String title,
    required String body,
    String? iconPath,
  }) async {
    if (isDesktop) {
      await LocalNotifier.instance.notify(
        LocalNotification(
          title: title,
          body: body,
          identifier: DateTime.now().millisecondsSinceEpoch.toString(),
        ),
      );
    }
  }
}
```

### Context Menu

```dart
// Right-click Context Menu
class ContextMenuRegion extends StatelessWidget {
  final Widget child;
  final List<ContextMenuButtonItem> contextMenuItems;

  const ContextMenuRegion({
    super.key,
    required this.child,
    required this.contextMenuItems,
  });

  @override
  Widget build(BuildContext context) {
    return ContextMenuController.removeAny == null
        ? child
        : GestureDetector(
            onSecondaryTapDown: (details) {
              _showContextMenu(context, details.globalPosition);
            },
            child: child,
          );
  }

  void _showContextMenu(BuildContext context, Offset position) {
    final RenderBox overlay =
        Overlay.of(context).context.findRenderObject() as RenderBox;
    
    showMenu(
      context: context,
      position: RelativeRect.fromRect(
        Rect.fromLTWH(position.dx, position.dy, 0, 0),
        Rect.fromLTWH(0, 0, overlay.size.width, overlay.size.height),
      ),
      items: contextMenuItems.map((item) {
        return PopupMenuItem(
          onTap: item.onPressed,
          child: Row(
            children: [
              if (item.leadingIcon != null)
                Padding(
                  padding: const EdgeInsets.only(right: 8),
                  child: Icon(item.leadingIcon, size: 16),
                ),
              Text(item.label ?? ''),
              if (item.trailingText != null) ...[
                const Spacer(),
                Text(
                  item.trailingText!,
                  style: TextStyle(color: Colors.grey[600], fontSize: 12),
                ),
              ],
            ],
          ),
        );
      }).toList(),
    );
  }
}

// ContextMenuButtonItem class
class ContextMenuButtonItem {
  final String? label;
  final VoidCallback? onPressed;
  final IconData? leadingIcon;
  final String? trailingText;

  const ContextMenuButtonItem({
    this.label,
    this.onPressed,
    this.leadingIcon,
    this.trailingText,
  });
}
```

---

## ขั้นตอนที่ 379: Packaging

### Android MSIX (Windows Store)

```yaml
# pubspec.yaml
dev_dependencies:
  msix: ^3.16.5

msix_config:
  display_name: My Desktop App
  publisher_display_name: My Company
  identity_name: com.mycompany.mydesktopapp
  msix_version: 1.0.0.0
  logo_path: assets/logo.png
  capabilities: 'internetClient,privateNetworkClientServer'
  file_extension: '.myext'
  protocol_activation: myapp
  sign_msix: true
  certificate_path: cert/certificate.pfx
  certificate_password: my-certificate-password
```

```bash
# Build MSIX
dart pub run msix:create

# หรือ
flutter pub run msix:create --release
```

### macOS DMG

```bash
# Build macOS app
flutter build macos --release

# สร้าง DMG ด้วย create-dmg
brew install create-dmg

create-dmg \
  --volname "My App Installer" \
  --window-pos 200 120 \
  --window-size 800 400 \
  --icon-size 100 \
  --icon "MyApp.app" 200 190 \
  --hide-extension "MyApp.app" \
  --app-drop-link 600 185 \
  "MyApp-Installer.dmg" \
  "build/macos/Build/Products/Release/MyApp.app"
```

### Linux AppImage

```bash
# Build Linux app
flutter build linux --release

# สร้าง AppImage
wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage
chmod +x appimagetool-x86_64.AppImage

# สร้าง AppDir structure
mkdir -p MyApp.AppDir/usr/bin
mkdir -p MyApp.AppDir/usr/lib
cp -r build/linux/x64/release/bundle/* MyApp.AppDir/usr/bin/

./appimagetool-x86_64.AppImage MyApp.AppDir MyApp-x86_64.AppImage
```

---

## ขั้นตอนที่ 380: Desktop-specific Layouts

```dart
// Desktop Sidebar Layout
class DesktopSidebarLayout extends StatefulWidget {
  final Widget sidebar;
  final Widget body;
  final Widget? detail;

  const DesktopSidebarLayout({
    super.key,
    required this.sidebar,
    required this.body,
    this.detail,
  });

  @override
  State<DesktopSidebarLayout> createState() => _DesktopSidebarLayoutState();
}

class _DesktopSidebarLayoutState extends State<DesktopSidebarLayout> {
  double _sidebarWidth = 250;
  double _detailWidth = 350;
  bool _isSidebarCollapsed = false;
  bool _isDetailVisible = true;

  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        // Sidebar
        AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          width: _isSidebarCollapsed ? 60 : _sidebarWidth,
          child: widget.sidebar,
        ),
        
        // Resizable Divider (Sidebar)
        _ResizableDivider(
          onResize: (delta) {
            setState(() {
              _sidebarWidth = (_sidebarWidth + delta).clamp(200, 400);
            });
          },
        ),
        
        // Main Body
        Expanded(child: widget.body),
        
        // Detail Panel
        if (widget.detail != null && _isDetailVisible) ...[
          _ResizableDivider(
            onResize: (delta) {
              setState(() {
                _detailWidth = (_detailWidth - delta).clamp(250, 500);
              });
            },
          ),
          AnimatedContainer(
            duration: const Duration(milliseconds: 200),
            width: _detailWidth,
            child: widget.detail!,
          ),
        ],
      ],
    );
  }
}

// Resizable Divider
class _ResizableDivider extends StatefulWidget {
  final ValueChanged<double> onResize;

  const _ResizableDivider({required this.onResize});

  @override
  State<_ResizableDivider> createState() => _ResizableDividerState();
}

class _ResizableDividerState extends State<_ResizableDivider> {
  bool _isHovered = false;

  @override
  Widget build(BuildContext context) {
    return MouseRegion(
      cursor: SystemMouseCursors.resizeColumn,
      onEnter: (_) => setState(() => _isHovered = true),
      onExit: (_) => setState(() => _isHovered = false),
      child: GestureDetector(
        onHorizontalDragUpdate: (details) {
          widget.onResize(details.delta.dx);
        },
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 100),
          width: 4,
          color: _isHovered
              ? Theme.of(context).colorScheme.primary.withOpacity(0.5)
              : Theme.of(context).dividerColor,
        ),
      ),
    );
  }
}
```

### Desktop App Layout Complete Example

```dart
class DesktopApp extends StatefulWidget {
  const DesktopApp({super.key});

  @override
  State<DesktopApp> createState() => _DesktopAppState();
}

class _DesktopAppState extends State<DesktopApp> {
  int _selectedIndex = 0;
  String? _selectedItemId;

  final _navItems = const [
    _NavItem(icon: Icons.inbox, label: 'Inbox'),
    _NavItem(icon: Icons.send, label: 'Sent'),
    _NavItem(icon: Icons.star, label: 'Starred'),
    _NavItem(icon: Icons.delete, label: 'Trash'),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // Custom Title Bar
          if (!Platform.isMacOS)
            CustomTitleBar(title: 'My Desktop App'),
          
          Expanded(
            child: DesktopSidebarLayout(
              sidebar: _buildSidebar(),
              body: _buildMailList(),
              detail: _selectedItemId != null
                  ? _buildMailDetail(_selectedItemId!)
                  : const Center(child: Text('Select an email')),
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildSidebar() {
    return NavigationRail(
      destinations: _navItems
          .map((item) => NavigationRailDestination(
                icon: Icon(item.icon),
                label: Text(item.label),
              ))
          .toList(),
      selectedIndex: _selectedIndex,
      onDestinationSelected: (i) => setState(() => _selectedIndex = i),
      extended: true,
    );
  }

  Widget _buildMailList() {
    return ListView.builder(
      itemCount: 20,
      itemBuilder: (context, index) => ListTile(
        selected: _selectedItemId == '$index',
        onTap: () => setState(() => _selectedItemId = '$index'),
        title: Text('Email Subject $index'),
        subtitle: Text('Preview of email $index content...'),
        trailing: Text('${index + 1}h ago'),
      ),
    );
  }

  Widget _buildMailDetail(String id) {
    return Padding(
      padding: const EdgeInsets.all(24),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('Email Subject $id',
              style: Theme.of(context).textTheme.headlineSmall),
          const SizedBox(height: 16),
          Text('From: sender@example.com'),
          Text('To: me@example.com'),
          const Divider(height: 32),
          Expanded(child: SingleChildScrollView(
            child: Text('Email body content for email $id...'),
          )),
        ],
      ),
    );
  }
}
```

---

## สรุป (Summary)

```
Flutter Desktop Checklist:

✅ Platform Setup:
  - เพิ่ม desktop platform support
  - กำหนด minimum system requirements
  - ตั้งค่า entitlements (macOS)

✅ Window Management:
  - ใช้ window_manager package
  - Custom title bar (optional)
  - Minimize to tray
  - Remember window position/size

✅ Native Integration:
  - System Tray (tray_manager)
  - Native Menu Bar (PlatformMenuBar)
  - Keyboard shortcuts
  - Context menus

✅ File System:
  - file_picker สำหรับ Open/Save dialogs
  - path_provider สำหรับ app directories
  - ตรวจสอบ permissions (macOS sandbox)

✅ Layout:
  - Sidebar + Body + Detail panel
  - Resizable panels
  - Desktop-appropriate typography (larger)
  - Mouse interactions (hover, right-click)

✅ Packaging:
  - Windows: MSIX (Windows Store)
  - macOS: DMG / App Store
  - Linux: AppImage / Snap / Flatpak

Platform Differences:
macOS: ⌘ key, Global menu bar, Sandbox
Windows: Ctrl key, Taskbar integration, MSIX
Linux: Varies by desktop environment
```

---

## แบบฝึกหัด (Exercises)

**ระดับพื้นฐาน:**
1. สร้าง Flutter Desktop app และรันบน macOS/Windows
2. ตั้งค่า window size, title, และ minimum size
3. เพิ่ม Custom Title Bar

**ระดับกลาง:**
4. Implement System Tray ด้วย context menu
5. เพิ่ม Keyboard Shortcuts สำหรับ common actions
6. Implement File Open/Save dialog

**ระดับสูง:**
7. สร้าง Resizable Panel Layout
8. Package app เป็น MSIX (Windows) หรือ DMG (macOS)
9. Implement Auto-updater

---

## การนำทาง
- [← Part 37: Flutter Web](part_37.md)
- [→ Part 39: Package Publishing](part_39.md)

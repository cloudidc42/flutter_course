# Part 47: AR & Advanced Features
## ขั้นตอนที่ 461-470

---

## สารบัญ
1. [AR Foundations](#ขั้นตอนที่-461-ar-foundations)
2. [ARCore (Android)](#ขั้นตอนที่-462-arcore-android)
3. [ARKit (iOS)](#ขั้นตอนที่-463-arkit-ios)
4. [Plane Detection & AR Object Placement](#ขั้นตอนที่-464-plane-detection)
5. [Custom AR Experiences](#ขั้นตอนที่-465-custom-ar)
6. [Bluetooth กับ Flutter](#ขั้นตอนที่-466-bluetooth)
7. [NFC](#ขั้นตอนที่-467-nfc)
8. [QR/Barcode Scanner](#ขั้นตอนที่-468-qr-barcode-scanner)
9. [Background Services](#ขั้นตอนที่-469-background-services)
10. [Workshop: AR Furniture App](#ขั้นตอนที่-470-workshop)

---

## ขั้นตอนที่ 461: AR Foundations

### AR คืออะไร?

Augmented Reality (AR) คือเทคโนโลยีที่เพิ่ม virtual objects เข้าไปใน real world ผ่านกล้องของอุปกรณ์

```
AR Components:
├── Camera Feed (Real World)
├── AR Tracking
│   ├── Plane Detection (ตรวจจับพื้นผิว)
│   ├── Image Tracking (ตามรอยรูปภาพ)
│   ├── Face Tracking (ติดตามใบหน้า)
│   └── World Tracking (ติดตาม 3D space)
├── 3D Rendering
│   ├── 3D Models (.glb, .gltf, .obj)
│   ├── Animations
│   └── Lighting
└── User Interaction
    ├── Tap to place
    ├── Pinch to scale
    └── Drag to move
```

### Dependencies

```yaml
# pubspec.yaml
dependencies:
  # AR (Choose based on platform)
  ar_flutter_plugin: ^0.7.3
  
  # 3D Models
  flutter_3d_controller: ^1.4.0
  
  # Camera
  camera: ^0.10.0
  
  # Bluetooth
  flutter_blue_plus: ^1.29.0
  
  # NFC
  flutter_nfc_kit: ^3.4.0
  
  # Barcode/QR
  mobile_scanner: ^3.5.6
  qr_flutter: ^4.1.0
  
  # Background Services
  flutter_background_service: ^5.0.5
  workmanager: ^0.5.2
```

### AR Architecture

```
Flutter App
├── AR View Layer
│   ├── Camera Feed
│   └── AR Overlay
├── AR Manager
│   ├── Session Management
│   ├── Node Management
│   └── Hit Testing
├── 3D Content
│   ├── Model Loader
│   ├── Scene Builder
│   └── Animation Controller
└── Platform Bridge
    ├── Android (ARCore)
    └── iOS (ARKit)
```

---

## ขั้นตอนที่ 462: ARCore (Android)

### ARCore Setup

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  
  <!-- AR Required -->
  <uses-permission android:name="android.permission.CAMERA"/>
  <uses-feature android:name="android.hardware.camera.ar" android:required="true"/>
  
  <!-- AR Optional -->
  <!-- <uses-feature android:name="android.hardware.camera.ar" android:required="false"/> -->
  
  <application>
    <meta-data
        android:name="com.google.ar.core"
        android:value="required"/> <!-- or "optional" -->
  </application>
  
</manifest>
```

### AR Flutter Plugin

```dart
// lib/features/ar/screens/ar_screen.dart
import 'package:ar_flutter_plugin/ar_flutter_plugin.dart';
import 'package:vector_math/vector_math_64.dart' as vector;

class ARScreen extends StatefulWidget {
  const ARScreen({super.key});
  
  @override
  State<ARScreen> createState() => _ARScreenState();
}

class _ARScreenState extends State<ARScreen> {
  ARSessionManager? arSessionManager;
  ARObjectManager? arObjectManager;
  ARAnchorManager? arAnchorManager;
  ARLocationManager? arLocationManager;
  
  List<ARNode> nodes = [];
  List<ARAnchor> anchors = [];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('AR Experience'),
        actions: [
          IconButton(
            onPressed: _clearNodes,
            icon: const Icon(Icons.delete_outline),
          ),
        ],
      ),
      body: Stack(
        children: [
          ARView(
            onARViewCreated: onARViewCreated,
            planeDetectionConfig: PlaneDetectionConfig.horizontalAndVertical,
          ),
          
          // UI Overlay
          Positioned(
            bottom: 100,
            left: 0,
            right: 0,
            child: _buildModelSelector(),
          ),
          
          // Instructions
          Positioned(
            top: 16,
            left: 0,
            right: 0,
            child: Center(
              child: Container(
                padding: const EdgeInsets.symmetric(
                  horizontal: 16,
                  vertical: 8,
                ),
                decoration: BoxDecoration(
                  color: Colors.black54,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Text(
                  'แตะพื้นเพื่อวาง 3D object',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  void onARViewCreated(
    ARSessionManager arSessionManager,
    ARObjectManager arObjectManager,
    ARAnchorManager arAnchorManager,
    ARLocationManager arLocationManager,
  ) {
    this.arSessionManager = arSessionManager;
    this.arObjectManager = arObjectManager;
    this.arAnchorManager = arAnchorManager;
    this.arLocationManager = arLocationManager;
    
    this.arSessionManager!.onInitialize(
      showAnimatedGuide: true,
      showFeaturePoints: false,
      showPlanes: true,
      customPlaneTexturePath: 'assets/triangle.png',
      showWorldOrigin: false,
      handlePans: true,
      handleRotation: true,
    );
    
    this.arObjectManager!.onInitialize();
    
    this.arSessionManager!.onPlaneOrPointTap = onPlaneOrPointTapped;
    this.arObjectManager!.onNodeTap = onNodeTapped;
  }
  
  Future<void> onPlaneOrPointTapped(List<ARHitTestResult> hitTestResults) async {
    final singleHitTestResult = hitTestResults.firstOrNull;
    if (singleHitTestResult == null) return;
    
    // Create anchor at tap location
    final newAnchor = ARPlaneAnchor(
      transformation: singleHitTestResult.worldTransform,
    );
    
    final didAddAnchor = await arAnchorManager!.addAnchor(newAnchor);
    if (didAddAnchor == null || !didAddAnchor) return;
    
    anchors.add(newAnchor);
    
    // Create node (3D object)
    final newNode = ARNode(
      type: NodeType.webGLB,
      uri: 'https://github.com/KhronosGroup/glTF-Sample-Models/raw/master/2.0/Duck/glTF-Binary/Duck.glb',
      scale: vector.Vector3(0.2, 0.2, 0.2),
      position: vector.Vector3(0.0, 0.0, 0.0),
      rotation: vector.Vector4(1.0, 0.0, 0.0, 0.0),
    );
    
    final didAddNodeToAnchor = await arObjectManager!.addNode(
      newNode,
      planeAnchor: newAnchor,
    );
    
    if (didAddNodeToAnchor == null || !didAddNodeToAnchor) {
      arAnchorManager!.removeAnchor(newAnchor);
    } else {
      nodes.add(newNode);
    }
  }
  
  void onNodeTapped(List<String> nodeNames) {
    // Show options for tapped node
    showModalBottomSheet(
      context: context,
      builder: (context) => Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ListTile(
            leading: const Icon(Icons.delete),
            title: const Text('ลบ Object'),
            onTap: () {
              Navigator.pop(context);
              _removeNode(nodeNames.first);
            },
          ),
          ListTile(
            leading: const Icon(Icons.scale),
            title: const Text('ปรับขนาด'),
            onTap: () {
              Navigator.pop(context);
              _showScaleDialog(nodeNames.first);
            },
          ),
        ],
      ),
    );
  }
  
  Future<void> _removeNode(String nodeName) async {
    final nodeToRemove = nodes.firstWhere((n) => n.name == nodeName);
    await arObjectManager!.removeNode(nodeToRemove);
    nodes.remove(nodeToRemove);
  }
  
  Future<void> _showScaleDialog(String nodeName) async {
    double scale = 1.0;
    
    await showDialog<void>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('ปรับขนาด'),
        content: StatefulBuilder(
          builder: (context, setDialogState) => Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text('${(scale * 100).round()}%'),
              Slider(
                value: scale,
                min: 0.1,
                max: 3.0,
                onChanged: (value) {
                  setDialogState(() => scale = value);
                },
              ),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยืนยัน'),
          ),
        ],
      ),
    );
  }
  
  Future<void> _clearNodes() async {
    for (final anchor in anchors) {
      arAnchorManager!.removeAnchor(anchor);
    }
    anchors.clear();
    nodes.clear();
  }
  
  Widget _buildModelSelector() {
    final models = [
      {'name': 'เก้าอี้', 'url': 'chair.glb', 'icon': Icons.chair},
      {'name': 'โต๊ะ', 'url': 'table.glb', 'icon': Icons.table_restaurant},
      {'name': 'โคมไฟ', 'url': 'lamp.glb', 'icon': Icons.light},
    ];
    
    return SingleChildScrollView(
      scrollDirection: Axis.horizontal,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Row(
        children: models.map((model) {
          return Padding(
            padding: const EdgeInsets.only(right: 8),
            child: ElevatedButton.icon(
              onPressed: () {
                // Set current model to place
              },
              icon: Icon(model['icon'] as IconData),
              label: Text(model['name'] as String),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.white,
                foregroundColor: Colors.black,
              ),
            ),
          );
        }).toList(),
      ),
    );
  }
  
  @override
  void dispose() {
    arSessionManager?.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 463: ARKit (iOS)

```dart
// lib/features/ar/screens/arkit_screen.dart
// ใช้ arkit_plugin สำหรับ iOS โดยเฉพาะ

import 'package:arkit_plugin/arkit_plugin.dart';
import 'package:vector_math/vector_math_64.dart' as vector;

class ARKitScreen extends StatefulWidget {
  const ARKitScreen({super.key});
  
  @override
  State<ARKitScreen> createState() => _ARKitScreenState();
}

class _ARKitScreenState extends State<ARKitScreen> {
  late ARKitController arkitController;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AR (iOS)')),
      body: ARKitSceneView(
        planeDetection: ARPlaneDetection.horizontalAndVertical,
        onARKitViewCreated: onARKitViewCreated,
        showFeaturePoints: true,
        showWorldOrigin: false,
      ),
    );
  }
  
  void onARKitViewCreated(ARKitController arkitController) {
    this.arkitController = arkitController;
    
    // Listen for plane detection
    arkitController.onAddNodeForAnchor = _handleAddAnchor;
    
    // Listen for taps
    arkitController.onNodeTap = (nodes) {
      if (nodes.isNotEmpty) {
        print('Tapped node: ${nodes.first}');
      }
    };
  }
  
  void _handleAddAnchor(ARKitAnchor anchor) {
    if (anchor is ARKitPlaneAnchor) {
      _addPlaneNode(anchor);
    }
  }
  
  void _addPlaneNode(ARKitPlaneAnchor anchor) {
    // Visualize detected plane
    final material = ARKitMaterial(
      diffuse: ARKitMaterialProperty.color(Colors.blue.withOpacity(0.3)),
    );
    
    final plane = ARKitPlane(
      width: anchor.extent.x,
      height: anchor.extent.z,
      materials: [material],
    );
    
    final node = ARKitNode(
      geometry: plane,
      position: vector.Vector3(
        anchor.center.x,
        0,
        anchor.center.z,
      ),
      rotation: vector.Vector4(1, 0, 0, -math.pi / 2),
    );
    
    arkitController.add(node, parentNodeName: anchor.nodeName);
  }
  
  Future<void> add3DObject({
    required vector.Vector3 position,
    required String modelUrl,
  }) async {
    // Add 3D model to scene
    final node = ARKitReferenceNode(
      url: modelUrl,
      position: position,
      scale: vector.Vector3.all(0.1),
    );
    
    arkitController.add(node);
  }
  
  void addTextNode(String text, vector.Vector3 position) {
    final textGeometry = ARKitText(
      string: text,
      extrusionDepth: 1,
      materials: [
        ARKitMaterial(
          diffuse: ARKitMaterialProperty.color(Colors.red),
        ),
      ],
    );
    
    final textNode = ARKitNode(
      geometry: textGeometry,
      position: position,
      scale: vector.Vector3.all(0.01),
    );
    
    arkitController.add(textNode);
  }
  
  @override
  void dispose() {
    arkitController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 464: Plane Detection & AR Object Placement

```dart
// lib/features/ar/services/ar_placement_service.dart
class ARPlacementService {
  final List<PlacedObject> _placedObjects = [];
  
  List<PlacedObject> get placedObjects => List.unmodifiable(_placedObjects);
  
  Future<PlacedObject?> placeObject({
    required ARHitTestResult hitResult,
    required ARObjectType objectType,
    required ARObjectManager objectManager,
    required ARAnchorManager anchorManager,
  }) async {
    // Create anchor at hit point
    final anchor = ARPlaneAnchor(
      transformation: hitResult.worldTransform,
    );
    
    final anchorAdded = await anchorManager.addAnchor(anchor);
    if (anchorAdded == null || !anchorAdded) return null;
    
    // Load 3D model
    final modelUrl = _getModelUrl(objectType);
    final scale = _getModelScale(objectType);
    
    final node = ARNode(
      type: NodeType.webGLB,
      uri: modelUrl,
      scale: vector.Vector3.all(scale),
      position: vector.Vector3.zero(),
    );
    
    final nodeAdded = await objectManager.addNode(
      node,
      planeAnchor: anchor,
    );
    
    if (nodeAdded == null || !nodeAdded) {
      anchorManager.removeAnchor(anchor);
      return null;
    }
    
    final placedObject = PlacedObject(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      type: objectType,
      anchor: anchor,
      node: node,
      placedAt: DateTime.now(),
    );
    
    _placedObjects.add(placedObject);
    return placedObject;
  }
  
  Future<void> removeObject(
    PlacedObject object,
    ARObjectManager objectManager,
    ARAnchorManager anchorManager,
  ) async {
    await objectManager.removeNode(object.node);
    anchorManager.removeAnchor(object.anchor);
    _placedObjects.remove(object);
  }
  
  String _getModelUrl(ARObjectType type) {
    return switch (type) {
      ARObjectType.chair => 'assets/models/chair.glb',
      ARObjectType.table => 'assets/models/table.glb',
      ARObjectType.sofa => 'assets/models/sofa.glb',
      ARObjectType.lamp => 'assets/models/lamp.glb',
    };
  }
  
  double _getModelScale(ARObjectType type) {
    return switch (type) {
      ARObjectType.chair => 0.003,
      ARObjectType.table => 0.004,
      ARObjectType.sofa => 0.005,
      ARObjectType.lamp => 0.002,
    };
  }
}

enum ARObjectType { chair, table, sofa, lamp }

class PlacedObject {
  final String id;
  final ARObjectType type;
  final ARAnchor anchor;
  final ARNode node;
  final DateTime placedAt;
  
  const PlacedObject({
    required this.id,
    required this.type,
    required this.anchor,
    required this.node,
    required this.placedAt,
  });
}
```

---

## ขั้นตอนที่ 465: Custom AR Experiences

### AR Image Tracking

```dart
// lib/features/ar/screens/ar_image_tracking_screen.dart
class ARImageTrackingScreen extends StatefulWidget {
  const ARImageTrackingScreen({super.key});
  
  @override
  State<ARImageTrackingScreen> createState() =>
      _ARImageTrackingScreenState();
}

class _ARImageTrackingScreenState extends State<ARImageTrackingScreen> {
  ARSessionManager? _arSessionManager;
  ARObjectManager? _arObjectManager;
  ARAnchorManager? _arAnchorManager;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AR Image Tracking')),
      body: ARView(
        onARViewCreated: _onARViewCreated,
        planeDetectionConfig: PlaneDetectionConfig.none, // No plane detection needed
      ),
    );
  }
  
  void _onARViewCreated(
    ARSessionManager sessionManager,
    ARObjectManager objectManager,
    ARAnchorManager anchorManager,
    ARLocationManager locationManager,
  ) {
    _arSessionManager = sessionManager;
    _arObjectManager = objectManager;
    _arAnchorManager = anchorManager;
    
    _arSessionManager!.onInitialize(
      showFeaturePoints: false,
      showPlanes: false,
      handlePans: false,
      handleRotation: false,
    );
    
    _arObjectManager!.onInitialize();
    
    // Set up image anchors
    _arAnchorManager!.initGoogleCloudAnchorMode();
    
    _arSessionManager!.onPlaneOrPointTap = (hits) {
      // Handle taps on tracked images
    };
    
    // Add image tracking reference
    _setupImageTracking();
  }
  
  Future<void> _setupImageTracking() async {
    // Load reference image
    final imageBytes = await rootBundle.load('assets/tracking_image.png');
    
    final imageAnchor = ARImageAnchor(
      // Configure tracking target
    );
    
    // When image is detected, show 3D content
    _arAnchorManager!.initGoogleCloudAnchorMode();
  }
}

// AR with Custom Shaders
class ARCustomEffectScreen extends StatefulWidget {
  const ARCustomEffectScreen({super.key});
  
  @override
  State<ARCustomEffectScreen> createState() =>
      _ARCustomEffectScreenState();
}

class _ARCustomEffectScreenState extends State<ARCustomEffectScreen>
    with TickerProviderStateMixin {
  late AnimationController _animationController;
  
  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AR Effects')),
      body: AnimatedBuilder(
        animation: _animationController,
        builder: (context, child) {
          return Stack(
            children: [
              // Camera preview
              const CameraPreviewWidget(),
              
              // AR overlay effect
              CustomPaint(
                painter: AREffectPainter(
                  animation: _animationController.value,
                ),
                size: Size.infinite,
              ),
            ],
          );
        },
      ),
    );
  }
  
  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 466: Bluetooth กับ Flutter

### Flutter Blue Plus

```dart
// lib/features/bluetooth/services/bluetooth_service.dart
import 'package:flutter_blue_plus/flutter_blue_plus.dart';

class BluetoothService {
  Stream<BluetoothAdapterState> get adapterStateStream {
    return FlutterBluePlus.adapterState;
  }
  
  Future<bool> get isBluetoothOn async {
    return FlutterBluePlus.adapterStateNow == BluetoothAdapterState.on;
  }
  
  // Scan for devices
  Stream<List<ScanResult>> scanDevices({
    Duration timeout = const Duration(seconds: 10),
    List<Guid> withServices = const [],
  }) async* {
    await FlutterBluePlus.startScan(
      timeout: timeout,
      withServices: withServices,
    );
    
    yield* FlutterBluePlus.scanResults;
  }
  
  Future<void> stopScan() async {
    await FlutterBluePlus.stopScan();
  }
  
  // Connect to device
  Future<BluetoothDevice> connect(BluetoothDevice device) async {
    await device.connect(
      timeout: const Duration(seconds: 30),
      autoConnect: false,
    );
    return device;
  }
  
  Future<void> disconnect(BluetoothDevice device) async {
    await device.disconnect();
  }
  
  // Discover services
  Future<List<BluetoothService>> discoverServices(
    BluetoothDevice device,
  ) async {
    return device.discoverServices();
  }
  
  // Read characteristic
  Future<List<int>> readCharacteristic(
    BluetoothCharacteristic characteristic,
  ) async {
    return characteristic.read();
  }
  
  // Write characteristic
  Future<void> writeCharacteristic(
    BluetoothCharacteristic characteristic,
    List<int> data, {
    bool withResponse = true,
  }) async {
    await characteristic.write(
      data,
      withoutResponse: !withResponse,
    );
  }
  
  // Notify/Subscribe to characteristic
  Stream<List<int>> subscribeToCharacteristic(
    BluetoothCharacteristic characteristic,
  ) async* {
    await characteristic.setNotifyValue(true);
    yield* characteristic.lastValueStream;
  }
}

// Bluetooth Device List Screen
class BluetoothDevicesScreen extends ConsumerStatefulWidget {
  const BluetoothDevicesScreen({super.key});
  
  @override
  ConsumerState<BluetoothDevicesScreen> createState() =>
      _BluetoothDevicesScreenState();
}

class _BluetoothDevicesScreenState
    extends ConsumerState<BluetoothDevicesScreen> {
  final BluetoothService _bluetoothService = BluetoothService();
  List<ScanResult> _scanResults = [];
  bool _isScanning = false;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('อุปกรณ์ Bluetooth'),
        actions: [
          IconButton(
            onPressed: _isScanning ? _stopScan : _startScan,
            icon: Icon(_isScanning ? Icons.stop : Icons.search),
          ),
        ],
      ),
      body: Column(
        children: [
          if (_isScanning)
            const LinearProgressIndicator(),
          Expanded(
            child: ListView.builder(
              itemCount: _scanResults.length,
              itemBuilder: (context, index) {
                final result = _scanResults[index];
                return ListTile(
                  leading: const Icon(Icons.bluetooth),
                  title: Text(
                    result.device.platformName.isNotEmpty
                        ? result.device.platformName
                        : 'Unknown Device',
                  ),
                  subtitle: Text(result.device.remoteId.str),
                  trailing: Text('${result.rssi} dBm'),
                  onTap: () => _connectToDevice(result.device),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
  
  Future<void> _startScan() async {
    setState(() {
      _isScanning = true;
      _scanResults.clear();
    });
    
    _bluetoothService.scanDevices().listen((results) {
      setState(() => _scanResults = results);
    });
  }
  
  Future<void> _stopScan() async {
    await _bluetoothService.stopScan();
    setState(() => _isScanning = false);
  }
  
  Future<void> _connectToDevice(BluetoothDevice device) async {
    try {
      await _bluetoothService.connect(device);
      if (mounted) {
        context.push('/bluetooth/device', extra: device);
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('เชื่อมต่อล้มเหลว: $e')),
        );
      }
    }
  }
  
  @override
  void dispose() {
    _bluetoothService.stopScan();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 467: NFC

```dart
// lib/features/nfc/services/nfc_service.dart
import 'package:flutter_nfc_kit/flutter_nfc_kit.dart';
import 'package:ndef/ndef.dart' as ndef;

class NFCService {
  Future<NFCAvailability> checkAvailability() async {
    return FlutterNfcKit.nfcAvailability;
  }
  
  Future<NFCTag?> readTag({
    Duration timeout = const Duration(seconds: 30),
    String iosAlertMessage = 'ใกล้ NFC tag',
  }) async {
    try {
      final tag = await FlutterNfcKit.poll(
        timeout: timeout,
        iosAlertMessage: iosAlertMessage,
        androidCheckNDEF: true,
      );
      return tag;
    } on PlatformException catch (e) {
      print('NFC read error: $e');
      return null;
    }
  }
  
  Future<List<ndef.NDEFRecord>?> readNDEF() async {
    final tag = await readTag();
    if (tag == null) return null;
    
    try {
      if (tag.ndefAvailable ?? false) {
        final records = await FlutterNfcKit.readNDEFRecords(cached: false);
        return records;
      }
      return null;
    } finally {
      await FlutterNfcKit.finish();
    }
  }
  
  Future<void> writeNDEF(List<ndef.NDEFRecord> records) async {
    final tag = await readTag(
      iosAlertMessage: 'ใกล้ NFC tag เพื่อเขียนข้อมูล',
    );
    
    if (tag == null) return;
    
    try {
      await FlutterNfcKit.writeNDEFRecords(records);
    } finally {
      await FlutterNfcKit.finish(iosAlertMessage: 'เขียนข้อมูลสำเร็จ!');
    }
  }
  
  Future<void> writeTextRecord(String text) async {
    final record = ndef.TextRecord(text: text, language: 'th');
    await writeNDEF([record]);
  }
  
  Future<void> writeUrlRecord(String url) async {
    final record = ndef.UriRecord.fromString(url);
    await writeNDEF([record]);
  }
}

// NFC Screen
class NFCScreen extends StatefulWidget {
  const NFCScreen({super.key});
  
  @override
  State<NFCScreen> createState() => _NFCScreenState();
}

class _NFCScreenState extends State<NFCScreen> {
  final NFCService _nfcService = NFCService();
  String _status = 'พร้อม';
  List<String> _readData = [];
  NFCMode _mode = NFCMode.read;
  final TextEditingController _writeController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('NFC'),
        actions: [
          SegmentedButton<NFCMode>(
            segments: const [
              ButtonSegment(value: NFCMode.read, label: Text('อ่าน')),
              ButtonSegment(value: NFCMode.write, label: Text('เขียน')),
            ],
            selected: {_mode},
            onSelectionChanged: (modes) {
              setState(() => _mode = modes.first);
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            // Status card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    const Icon(Icons.nfc, size: 64),
                    const SizedBox(height: 8),
                    Text(
                      _status,
                      style: Theme.of(context).textTheme.titleMedium,
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 16),
            
            if (_mode == NFCMode.write) ...[
              TextField(
                controller: _writeController,
                decoration: const InputDecoration(
                  labelText: 'ข้อความที่จะเขียน',
                  border: OutlineInputBorder(),
                ),
                maxLines: 3,
              ),
              const SizedBox(height: 8),
            ],
            
            ElevatedButton.icon(
              onPressed: _mode == NFCMode.read ? _readNFC : _writeNFC,
              icon: Icon(_mode == NFCMode.read ? Icons.nfc : Icons.edit),
              label: Text(_mode == NFCMode.read ? 'อ่าน NFC Tag' : 'เขียน NFC Tag'),
            ),
            
            if (_readData.isNotEmpty) ...[
              const SizedBox(height: 16),
              const Text('ข้อมูลที่อ่านได้:'),
              ..._readData.map((data) => Card(
                child: ListTile(
                  leading: const Icon(Icons.article),
                  title: Text(data),
                ),
              )),
            ],
          ],
        ),
      ),
    );
  }
  
  Future<void> _readNFC() async {
    setState(() => _status = 'กำลังอ่าน... ใกล้ NFC tag');
    
    try {
      final records = await _nfcService.readNDEF();
      
      if (records != null && records.isNotEmpty) {
        final readData = records.map((record) {
          if (record is ndef.TextRecord) {
            return 'ข้อความ: ${record.text}';
          } else if (record is ndef.UriRecord) {
            return 'URL: ${record.uri}';
          }
          return 'ข้อมูล: ${record.payload?.toHexString()}';
        }).toList();
        
        setState(() {
          _readData = readData;
          _status = 'อ่านสำเร็จ!';
        });
      } else {
        setState(() => _status = 'ไม่พบข้อมูล NDEF');
      }
    } catch (e) {
      setState(() => _status = 'เกิดข้อผิดพลาด: $e');
    }
  }
  
  Future<void> _writeNFC() async {
    final text = _writeController.text.trim();
    if (text.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('กรุณากรอกข้อความ')),
      );
      return;
    }
    
    setState(() => _status = 'กำลังเขียน... ใกล้ NFC tag');
    
    try {
      await _nfcService.writeTextRecord(text);
      setState(() => _status = 'เขียนสำเร็จ!');
    } catch (e) {
      setState(() => _status = 'เกิดข้อผิดพลาด: $e');
    }
  }
  
  @override
  void dispose() {
    _writeController.dispose();
    super.dispose();
  }
}

enum NFCMode { read, write }
```

---

## ขั้นตอนที่ 468: QR/Barcode Scanner

```dart
// lib/features/scanner/screens/scanner_screen.dart
import 'package:mobile_scanner/mobile_scanner.dart';
import 'package:qr_flutter/qr_flutter.dart';

class QRScannerScreen extends StatefulWidget {
  final Function(String) onScanned;
  
  const QRScannerScreen({super.key, required this.onScanned});
  
  @override
  State<QRScannerScreen> createState() => _QRScannerScreenState();
}

class _QRScannerScreenState extends State<QRScannerScreen> {
  late MobileScannerController _controller;
  bool _hasScanned = false;
  bool _flashOn = false;
  
  @override
  void initState() {
    super.initState();
    _controller = MobileScannerController(
      detectionSpeed: DetectionSpeed.normal,
      facing: CameraFacing.back,
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        backgroundColor: Colors.black,
        foregroundColor: Colors.white,
        title: const Text('สแกน QR Code'),
        actions: [
          IconButton(
            onPressed: () {
              _controller.toggleTorch();
              setState(() => _flashOn = !_flashOn);
            },
            icon: Icon(_flashOn ? Icons.flash_on : Icons.flash_off),
          ),
          IconButton(
            onPressed: () => _controller.switchCamera(),
            icon: const Icon(Icons.flip_camera_android),
          ),
        ],
      ),
      body: Stack(
        children: [
          MobileScanner(
            controller: _controller,
            onDetect: (capture) {
              if (_hasScanned) return;
              
              final barcodes = capture.barcodes;
              final barcode = barcodes.firstOrNull;
              
              if (barcode?.rawValue != null) {
                _hasScanned = true;
                
                // Vibrate
                HapticFeedback.mediumImpact();
                
                widget.onScanned(barcode!.rawValue!);
                Navigator.pop(context);
              }
            },
          ),
          
          // Scan overlay
          CustomPaint(
            painter: ScannerOverlayPainter(),
            size: Size.infinite,
          ),
          
          // Instructions
          Positioned(
            bottom: 50,
            left: 0,
            right: 0,
            child: const Center(
              child: Text(
                'จัด QR Code ให้อยู่ในกรอบ',
                style: TextStyle(color: Colors.white, fontSize: 16),
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

// Scanner Overlay Painter
class ScannerOverlayPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.black54
      ..style = PaintingStyle.fill;
    
    final scanAreaSize = size.width * 0.7;
    final scanAreaLeft = (size.width - scanAreaSize) / 2;
    final scanAreaTop = (size.height - scanAreaSize) / 2;
    
    final scanRect = Rect.fromLTWH(
      scanAreaLeft,
      scanAreaTop,
      scanAreaSize,
      scanAreaSize,
    );
    
    // Draw dark overlay
    final fullPath = Path()
      ..addRect(Rect.fromLTWH(0, 0, size.width, size.height));
    final scanPath = Path()
      ..addRRect(RRect.fromRectAndRadius(scanRect, const Radius.circular(12)));
    
    final overlayPath = Path.combine(
      PathOperation.difference,
      fullPath,
      scanPath,
    );
    
    canvas.drawPath(overlayPath, paint);
    
    // Draw corner markers
    final markerPaint = Paint()
      ..color = Colors.green
      ..style = PaintingStyle.stroke
      ..strokeWidth = 3;
    
    const markerLength = 20.0;
    
    // Top-left corner
    canvas.drawLine(
      Offset(scanAreaLeft, scanAreaTop + markerLength),
      Offset(scanAreaLeft, scanAreaTop),
      markerPaint,
    );
    canvas.drawLine(
      Offset(scanAreaLeft, scanAreaTop),
      Offset(scanAreaLeft + markerLength, scanAreaTop),
      markerPaint,
    );
    
    // Draw other corners...
  }
  
  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}

// QR Code Generator
class QRCodeGenerator extends StatelessWidget {
  final String data;
  final double size;
  
  const QRCodeGenerator({
    super.key,
    required this.data,
    this.size = 200,
  });
  
  @override
  Widget build(BuildContext context) {
    return QrImageView(
      data: data,
      version: QrVersions.auto,
      size: size,
      backgroundColor: Colors.white,
      errorCorrectionLevel: QrErrorCorrectLevel.H,
      embeddedImage: const AssetImage('assets/logo.png'),
      embeddedImageStyle: QrEmbeddedImageStyle(
        size: Size(size * 0.2, size * 0.2),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 469: Background Services

```dart
// lib/services/background_service.dart
import 'package:flutter_background_service/flutter_background_service.dart';

@pragma('vm:entry-point')
void onStart(ServiceInstance service) async {
  DartPluginRegistrant.ensureInitialized();
  
  if (service is AndroidServiceInstance) {
    service.on('setAsForeground').listen((event) {
      service.setAsForegroundService();
    });
    
    service.on('setAsBackground').listen((event) {
      service.setAsBackgroundService();
    });
  }
  
  service.on('stopService').listen((event) {
    service.stopSelf();
  });
  
  // Background tasks
  Timer.periodic(const Duration(seconds: 10), (timer) async {
    if (service is AndroidServiceInstance) {
      if (await service.isForegroundService()) {
        service.setForegroundNotificationInfo(
          title: 'App is running',
          content: 'Running in background',
        );
      }
    }
    
    // Do background work
    await _performBackgroundWork();
    
    // Send data to UI
    service.invoke('update', {
      'timestamp': DateTime.now().toIso8601String(),
    });
  });
}

Future<void> _performBackgroundWork() async {
  // Background sync, location tracking, etc.
  print('Background work: ${DateTime.now()}');
}

class BackgroundServiceManager {
  static Future<void> initialize() async {
    final service = FlutterBackgroundService();
    
    await service.configure(
      androidConfiguration: AndroidConfiguration(
        onStart: onStart,
        isForegroundMode: true,
        autoStart: true,
        notificationChannelId: 'my_foreground',
        initialNotificationTitle: 'App Service',
        initialNotificationContent: 'กำลังทำงาน',
        foregroundServiceNotificationId: 888,
      ),
      iosConfiguration: IosConfiguration(
        autoStart: true,
        onForeground: onStart,
        onBackground: onIosBackground,
      ),
    );
  }
  
  static Future<void> start() async {
    final service = FlutterBackgroundService();
    var isRunning = await service.isRunning();
    if (!isRunning) {
      service.startService();
    }
  }
  
  static Future<void> stop() async {
    final service = FlutterBackgroundService();
    service.invoke('stopService');
  }
  
  static Future<bool> isRunning() async {
    final service = FlutterBackgroundService();
    return service.isRunning();
  }
  
  // Listen to updates from background service
  static Stream<Map<String, dynamic>?> get updateStream {
    final service = FlutterBackgroundService();
    return service.on('update');
  }
}

@pragma('vm:entry-point')
Future<bool> onIosBackground(ServiceInstance service) async {
  WidgetsFlutterBinding.ensureInitialized();
  DartPluginRegistrant.ensureInitialized();
  return true;
}

// Background Service Widget
class BackgroundServiceWidget extends ConsumerStatefulWidget {
  const BackgroundServiceWidget({super.key});
  
  @override
  ConsumerState<BackgroundServiceWidget> createState() =>
      _BackgroundServiceWidgetState();
}

class _BackgroundServiceWidgetState
    extends ConsumerState<BackgroundServiceWidget> {
  bool _isRunning = false;
  String _lastUpdate = '-';
  late StreamSubscription _subscription;
  
  @override
  void initState() {
    super.initState();
    _checkStatus();
    _subscribeToUpdates();
  }
  
  Future<void> _checkStatus() async {
    final running = await BackgroundServiceManager.isRunning();
    setState(() => _isRunning = running);
  }
  
  void _subscribeToUpdates() {
    _subscription = BackgroundServiceManager.updateStream.listen((data) {
      if (data != null) {
        setState(() {
          _lastUpdate = data['timestamp'] as String? ?? '-';
        });
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Background Service')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: ListTile(
                leading: Icon(
                  _isRunning ? Icons.check_circle : Icons.cancel,
                  color: _isRunning ? Colors.green : Colors.red,
                ),
                title: Text(_isRunning ? 'กำลังทำงาน' : 'หยุดทำงาน'),
                subtitle: Text('อัปเดตล่าสุด: $_lastUpdate'),
              ),
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton(
                    onPressed: _isRunning ? null : _startService,
                    child: const Text('เริ่มต้น'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton(
                    onPressed: _isRunning ? _stopService : null,
                    child: const Text('หยุด'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  Future<void> _startService() async {
    await BackgroundServiceManager.start();
    setState(() => _isRunning = true);
  }
  
  Future<void> _stopService() async {
    await BackgroundServiceManager.stop();
    setState(() => _isRunning = false);
  }
  
  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 470: Workshop - AR Furniture App

### สร้าง AR Furniture Placement App

```dart
// lib/features/ar_furniture/screens/ar_furniture_screen.dart
class ARFurnitureScreen extends StatefulWidget {
  const ARFurnitureScreen({super.key});
  
  @override
  State<ARFurnitureScreen> createState() => _ARFurnitureScreenState();
}

class _ARFurnitureScreenState extends State<ARFurnitureScreen> {
  ARSessionManager? arSessionManager;
  ARObjectManager? arObjectManager;
  ARAnchorManager? arAnchorManager;
  
  final List<PlacedObject> _placedObjects = [];
  ARObjectType _selectedType = ARObjectType.chair;
  
  static const Map<ARObjectType, Map<String, dynamic>> furnitureItems = {
    ARObjectType.chair: {
      'name': 'เก้าอี้',
      'icon': Icons.chair,
      'color': Colors.brown,
      'price': 2500,
    },
    ARObjectType.table: {
      'name': 'โต๊ะ',
      'icon': Icons.table_restaurant,
      'color': Colors.teal,
      'price': 5000,
    },
    ARObjectType.sofa: {
      'name': 'โซฟา',
      'icon': Icons.weekend,
      'color': Colors.purple,
      'price': 15000,
    },
    ARObjectType.lamp: {
      'name': 'โคมไฟ',
      'icon': Icons.light,
      'color': Colors.amber,
      'price': 1200,
    },
  };
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // AR View
          ARView(
            onARViewCreated: _onARViewCreated,
            planeDetectionConfig: PlaneDetectionConfig.horizontalAndVertical,
          ),
          
          // Top bar
          SafeArea(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                children: [
                  IconButton.filled(
                    onPressed: () => Navigator.pop(context),
                    icon: const Icon(Icons.arrow_back),
                    style: IconButton.styleFrom(
                      backgroundColor: Colors.white,
                      foregroundColor: Colors.black,
                    ),
                  ),
                  const Spacer(),
                  Container(
                    padding: const EdgeInsets.symmetric(
                      horizontal: 12,
                      vertical: 6,
                    ),
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(20),
                    ),
                    child: Text(
                      '${_placedObjects.length} ชิ้น',
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                  ),
                  const SizedBox(width: 8),
                  IconButton.filled(
                    onPressed: _clearAll,
                    icon: const Icon(Icons.delete_outline),
                    style: IconButton.styleFrom(
                      backgroundColor: Colors.white,
                      foregroundColor: Colors.black,
                    ),
                  ),
                ],
              ),
            ),
          ),
          
          // Instructions
          Positioned(
            top: 100,
            left: 0,
            right: 0,
            child: Center(
              child: Container(
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: Colors.black54,
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Text(
                  'แตะพื้นเพื่อวางเฟอร์นิเจอร์',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ),
          ),
          
          // Furniture selector at bottom
          Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            child: _buildFurniturePicker(),
          ),
        ],
      ),
    );
  }
  
  Widget _buildFurniturePicker() {
    return Container(
      padding: const EdgeInsets.all(16),
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
      child: SafeArea(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text(
              'เลือกเฟอร์นิเจอร์',
              style: TextStyle(
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 12),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: furnitureItems.entries.map((entry) {
                final isSelected = _selectedType == entry.key;
                final item = entry.value;
                
                return GestureDetector(
                  onTap: () => setState(() => _selectedType = entry.key),
                  child: AnimatedContainer(
                    duration: const Duration(milliseconds: 200),
                    padding: const EdgeInsets.all(12),
                    decoration: BoxDecoration(
                      color: isSelected
                          ? (item['color'] as Color)
                          : Colors.white.withOpacity(0.2),
                      borderRadius: BorderRadius.circular(12),
                      border: Border.all(
                        color: isSelected
                            ? Colors.white
                            : Colors.transparent,
                        width: 2,
                      ),
                    ),
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Icon(
                          item['icon'] as IconData,
                          color: Colors.white,
                          size: 32,
                        ),
                        const SizedBox(height: 4),
                        Text(
                          item['name'] as String,
                          style: const TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                          ),
                        ),
                        Text(
                          '฿${item['price']}',
                          style: TextStyle(
                            color: Colors.white.withOpacity(0.7),
                            fontSize: 10,
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
      ),
    );
  }
  
  void _onARViewCreated(
    ARSessionManager sessionManager,
    ARObjectManager objectManager,
    ARAnchorManager anchorManager,
    ARLocationManager locationManager,
  ) {
    arSessionManager = sessionManager;
    arObjectManager = objectManager;
    arAnchorManager = anchorManager;
    
    arSessionManager!.onInitialize(
      showPlanes: true,
      showFeaturePoints: false,
    );
    
    arObjectManager!.onInitialize();
    arSessionManager!.onPlaneOrPointTap = _onTap;
  }
  
  Future<void> _onTap(List<ARHitTestResult> hitTestResults) async {
    final hit = hitTestResults.firstOrNull;
    if (hit == null) return;
    
    final service = ARPlacementService();
    final placed = await service.placeObject(
      hitResult: hit,
      objectType: _selectedType,
      objectManager: arObjectManager!,
      anchorManager: arAnchorManager!,
    );
    
    if (placed != null) {
      setState(() => _placedObjects.add(placed));
    }
  }
  
  Future<void> _clearAll() async {
    for (final obj in _placedObjects) {
      await arObjectManager!.removeNode(obj.node);
      arAnchorManager!.removeAnchor(obj.anchor);
    }
    setState(() => _placedObjects.clear());
  }
  
  @override
  void dispose() {
    arSessionManager?.dispose();
    super.dispose();
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **AR Foundations**: หลักการของ Augmented Reality
- **ARCore/ARKit**: Plane detection และ object placement
- **Bluetooth**: การ scan, connect และสื่อสารกับอุปกรณ์ BLE
- **NFC**: การอ่านและเขียน NFC tags
- **QR/Barcode Scanner**: mobile_scanner และ qr_flutter
- **Background Services**: flutter_background_service

## แบบฝึกหัด

1. สร้าง AR app ที่วาง virtual sticky notes ในพื้นที่จริง
2. Implement BLE heart rate monitor reader
3. สร้าง NFC business card ที่เขียนข้อมูลติดต่อ
4. สร้าง QR code scanner ที่รองรับหลาย action (URL, contact, WiFi)
5. Implement background location tracking service

---

[⬅️ Part 46](part_46.md) | [Part 48 ➡️](part_48.md)

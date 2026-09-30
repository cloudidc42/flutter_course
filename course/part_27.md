# Part 27: Maps & Geolocation
## ขั้นตอนที่ 261-270

---

## สารบัญ
1. [google_maps_flutter Setup](#google_maps_flutter-setup)
2. [geolocator & Permissions](#geolocator--permissions)
3. [Showing Current Location](#showing-current-location)
4. [Markers](#markers)
5. [Polylines](#polylines)
6. [Polygons](#polygons)
7. [InfoWindows](#infowindows)
8. [Custom Map Styles](#custom-map-styles)
9. [Geocoding](#geocoding)
10. [Place Picker & Directions API](#place-picker--directions-api)

---

## ขั้นตอนที่ 261: google_maps_flutter Setup

```yaml
# pubspec.yaml
dependencies:
  google_maps_flutter: ^2.5.3
  geolocator: ^11.0.0
  geocoding: ^3.0.0
  permission_handler: ^11.3.0
  http: ^1.2.0
  flutter_polyline_points: ^2.0.0
```

### ตั้งค่า Android

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest>
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    
    <application>
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="YOUR_ANDROID_API_KEY"/>
    </application>
</manifest>
```

### ตั้งค่า iOS

```xml
<!-- ios/Runner/Info.plist -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>แอปนี้ต้องการเข้าถึงตำแหน่งของคุณเพื่อแสดงแผนที่</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>แอปนี้ต้องการเข้าถึงตำแหน่งของคุณเสมอ</string>
```

```swift
// ios/Runner/AppDelegate.swift - เพิ่ม Google Maps
import GoogleMaps

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        GMSServices.provideAPIKey("YOUR_IOS_API_KEY")
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}
```

---

## ขั้นตอนที่ 262: geolocator & Permissions

```dart
// lib/services/location_service.dart
import 'package:geolocator/geolocator.dart';
import 'package:permission_handler/permission_handler.dart';

class LocationService {
  // ขอ Permission และดึง Location
  static Future<Position?> getCurrentLocation() async {
    // ตรวจสอบว่า Service เปิดอยู่
    bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceEnabled) {
      throw Exception('Location service ปิดอยู่ กรุณาเปิด GPS');
    }

    // ตรวจสอบ Permission
    LocationPermission permission = await Geolocator.checkPermission();

    if (permission == LocationPermission.denied) {
      permission = await Geolocator.requestPermission();
      if (permission == LocationPermission.denied) {
        throw Exception('ไม่ได้รับอนุญาตให้เข้าถึงตำแหน่ง');
      }
    }

    if (permission == LocationPermission.deniedForever) {
      throw Exception(
          'ถูกปิดการเข้าถึงตำแหน่งถาวร กรุณาเปิดใน Settings');
    }

    // ดึง Position
    return await Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
    );
  }

  // Stream ตำแหน่งแบบ Real-time
  static Stream<Position> getPositionStream() {
    return Geolocator.getPositionStream(
      locationSettings: const LocationSettings(
        accuracy: LocationAccuracy.high,
        distanceFilter: 10, // Update ทุก 10 เมตร
      ),
    );
  }

  // คำนวณระยะทาง
  static double calculateDistance({
    required double startLat,
    required double startLng,
    required double endLat,
    required double endLng,
  }) {
    return Geolocator.distanceBetween(startLat, startLng, endLat, endLng);
  }

  // คำนวณ Bearing (ทิศทาง)
  static double calculateBearing({
    required double startLat,
    required double startLng,
    required double endLat,
    required double endLng,
  }) {
    return Geolocator.bearingBetween(startLat, startLng, endLat, endLng);
  }

  // เปิด Location Settings
  static Future<bool> openLocationSettings() {
    return Geolocator.openLocationSettings();
  }

  // เปิด App Settings
  static Future<bool> openAppSettings() {
    return Geolocator.openAppSettings();
  }
}
```

---

## ขั้นตอนที่ 263: Showing Current Location

```dart
// lib/pages/map_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:geolocator/geolocator.dart';
import '../services/location_service.dart';

class MapPage extends StatefulWidget {
  const MapPage({super.key});

  @override
  State<MapPage> createState() => _MapPageState();
}

class _MapPageState extends State<MapPage> {
  GoogleMapController? _mapController;
  Position? _currentPosition;
  bool _isLoading = true;
  String? _error;

  static const CameraPosition _initialPosition = CameraPosition(
    target: LatLng(13.7563, 100.5018), // กรุงเทพฯ
    zoom: 12,
  );

  @override
  void initState() {
    super.initState();
    _getCurrentLocation();
  }

  @override
  void dispose() {
    _mapController?.dispose();
    super.dispose();
  }

  Future<void> _getCurrentLocation() async {
    try {
      final position = await LocationService.getCurrentLocation();
      setState(() {
        _currentPosition = position;
        _isLoading = false;
      });

      // ย้าย Camera ไปยังตำแหน่งปัจจุบัน
      _mapController?.animateCamera(
        CameraUpdate.newCameraPosition(
          CameraPosition(
            target: LatLng(position!.latitude, position.longitude),
            zoom: 15,
          ),
        ),
      );
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_error != null) {
      return Scaffold(
        appBar: AppBar(title: const Text('แผนที่')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.location_off, size: 80, color: Colors.red),
              const SizedBox(height: 16),
              Text(_error!),
              const SizedBox(height: 16),
              ElevatedButton(
                onPressed: () {
                  setState(() {
                    _error = null;
                    _isLoading = true;
                  });
                  _getCurrentLocation();
                },
                child: const Text('ลองใหม่'),
              ),
              ElevatedButton(
                onPressed: LocationService.openLocationSettings,
                child: const Text('เปิดการตั้งค่า Location'),
              ),
            ],
          ),
        ),
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: const Text('แผนที่'),
        actions: [
          IconButton(
            icon: const Icon(Icons.my_location),
            onPressed: _getCurrentLocation,
          ),
        ],
      ),
      body: Stack(
        children: [
          GoogleMap(
            initialCameraPosition: _initialPosition,
            onMapCreated: (controller) {
              _mapController = controller;
              if (_currentPosition != null) {
                controller.animateCamera(
                  CameraUpdate.newLatLngZoom(
                    LatLng(
                      _currentPosition!.latitude,
                      _currentPosition!.longitude,
                    ),
                    15,
                  ),
                );
              }
            },
            myLocationEnabled: true,
            myLocationButtonEnabled: false,
            mapType: MapType.normal,
            zoomControlsEnabled: false,
            compassEnabled: true,
          ),
          if (_isLoading)
            const Center(child: CircularProgressIndicator()),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _getCurrentLocation,
        child: const Icon(Icons.my_location),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 264: Markers

```dart
// lib/pages/markers_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'dart:ui' as ui;
import 'dart:typed_data';

class MarkersPage extends StatefulWidget {
  const MarkersPage({super.key});

  @override
  State<MarkersPage> createState() => _MarkersPageState();
}

class _MarkersPageState extends State<MarkersPage> {
  GoogleMapController? _mapController;
  final Set<Marker> _markers = {};
  BitmapDescriptor? _customIcon;

  static const LatLng _center = LatLng(13.7563, 100.5018);

  @override
  void initState() {
    super.initState();
    _createCustomIcon();
    _addDefaultMarkers();
  }

  Future<void> _createCustomIcon() async {
    // สร้าง Custom Icon จาก Widget
    final icon = await _createMarkerIcon(
      color: Colors.red,
      size: 80,
      text: 'A',
    );
    setState(() => _customIcon = icon);
  }

  Future<BitmapDescriptor> _createMarkerIcon({
    required Color color,
    required double size,
    required String text,
  }) async {
    final recorder = ui.PictureRecorder();
    final canvas = Canvas(recorder);
    final paint = Paint()..color = color;

    // วาด Circle
    canvas.drawCircle(
      Offset(size / 2, size / 2),
      size / 2,
      paint,
    );

    // วาด Text
    final textPainter = TextPainter(
      text: TextSpan(
        text: text,
        style: TextStyle(
          color: Colors.white,
          fontSize: size * 0.4,
          fontWeight: FontWeight.bold,
        ),
      ),
      textDirection: TextDirection.ltr,
    );
    textPainter.layout();
    textPainter.paint(
      canvas,
      Offset(
        (size - textPainter.width) / 2,
        (size - textPainter.height) / 2,
      ),
    );

    final picture = recorder.endRecording();
    final image = await picture.toImage(size.toInt(), size.toInt());
    final bytes = await image.toByteData(format: ui.ImageByteFormat.png);

    return BitmapDescriptor.fromBytes(bytes!.buffer.asUint8List());
  }

  void _addDefaultMarkers() {
    final locations = [
      {'name': 'สยามพารากอน', 'lat': 13.7466, 'lng': 100.5347},
      {'name': 'เซ็นทรัลเวิลด์', 'lat': 13.7464, 'lng': 100.5394},
      {'name': 'เมเจอร์ซีนีเพล็กซ์ สุขุมวิท', 'lat': 13.7256, 'lng': 100.5974},
      {'name': 'ไอคอนสยาม', 'lat': 13.7267, 'lng': 100.5097},
    ];

    for (final loc in locations) {
      _markers.add(
        Marker(
          markerId: MarkerId(loc['name'] as String),
          position: LatLng(loc['lat'] as double, loc['lng'] as double),
          infoWindow: InfoWindow(
            title: loc['name'] as String,
            snippet: 'แตะเพื่อดูรายละเอียด',
            onTap: () => _showPlaceDetail(loc['name'] as String),
          ),
          icon: BitmapDescriptor.defaultMarkerWithHue(
            BitmapDescriptor.hueBlue,
          ),
          onTap: () {},
        ),
      );
    }
    setState(() {});
  }

  void _addMarkerOnTap(LatLng position) {
    final id = 'custom_${DateTime.now().millisecondsSinceEpoch}';
    setState(() {
      _markers.add(
        Marker(
          markerId: MarkerId(id),
          position: position,
          infoWindow: InfoWindow(
            title: 'จุดที่เลือก',
            snippet: '${position.latitude.toStringAsFixed(4)}, ${position.longitude.toStringAsFixed(4)}',
          ),
          icon: _customIcon ??
              BitmapDescriptor.defaultMarkerWithHue(BitmapDescriptor.hueGreen),
          draggable: true,
          onDragEnd: (newPosition) {
            debugPrint('Marker moved to: $newPosition');
          },
        ),
      );
    });
  }

  void _showPlaceDetail(String name) {
    showModalBottomSheet(
      context: context,
      builder: (ctx) => Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              name,
              style: const TextStyle(
                  fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 8),
            Text(
              'กรุงเทพมหานคร, ประเทศไทย',
              style: TextStyle(color: Colors.grey[600]),
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () => Navigator.pop(ctx),
                    icon: const Icon(Icons.directions),
                    label: const Text('นำทาง'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: () => Navigator.pop(ctx),
                    icon: const Icon(Icons.bookmark),
                    label: const Text('บันทึก'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Markers'),
        actions: [
          IconButton(
            icon: const Icon(Icons.clear),
            onPressed: () => setState(() => _markers.clear()),
          ),
        ],
      ),
      body: GoogleMap(
        initialCameraPosition: const CameraPosition(
          target: _center,
          zoom: 12,
        ),
        onMapCreated: (controller) => _mapController = controller,
        markers: _markers,
        onTap: _addMarkerOnTap,
        myLocationEnabled: true,
        myLocationButtonEnabled: true,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 265: Polylines

```dart
// lib/pages/polylines_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'dart:math';

class PolylinesPage extends StatefulWidget {
  const PolylinesPage({super.key});

  @override
  State<PolylinesPage> createState() => _PolylinesPageState();
}

class _PolylinesPageState extends State<PolylinesPage> {
  GoogleMapController? _mapController;
  final Set<Polyline> _polylines = {};
  final Set<Marker> _markers = {};

  // ตำแหน่ง BTS สายสีเขียว (บางส่วน)
  static const List<LatLng> _btsGreenLine = [
    LatLng(13.7250, 100.5175), // สยาม
    LatLng(13.7302, 100.5245), // ชิดลม
    LatLng(13.7368, 100.5297), // เพลินจิต
    LatLng(13.7438, 100.5328), // นานา
    LatLng(13.7504, 100.5374), // อโศก
    LatLng(13.7565, 100.5412), // พร้อมพงษ์
    LatLng(13.7621, 100.5468), // ทองหล่อ
  ];

  @override
  void initState() {
    super.initState();
    _addBTSRoute();
  }

  void _addBTSRoute() {
    // เพิ่ม Stations เป็น Markers
    for (var i = 0; i < _btsGreenLine.length; i++) {
      _markers.add(
        Marker(
          markerId: MarkerId('station_$i'),
          position: _btsGreenLine[i],
          icon: BitmapDescriptor.defaultMarkerWithHue(
            BitmapDescriptor.hueGreen,
          ),
          infoWindow: InfoWindow(title: 'สถานี ${i + 1}'),
        ),
      );
    }

    // เพิ่ม Polyline
    _polylines.add(
      Polyline(
        polylineId: const PolylineId('bts_green_line'),
        points: _btsGreenLine,
        color: Colors.green,
        width: 5,
        patterns: [], // Solid line
        startCap: Cap.roundCap,
        endCap: Cap.roundCap,
        jointType: JointType.round,
      ),
    );

    setState(() {});
  }

  void _addCustomRoute(List<LatLng> points) {
    final id = 'route_${DateTime.now().millisecondsSinceEpoch}';
    _polylines.add(
      Polyline(
        polylineId: PolylineId(id),
        points: points,
        color: Colors.blue,
        width: 4,
        patterns: [
          PatternItem.dash(20), // เส้นประ
          PatternItem.gap(10),
        ],
      ),
    );
    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('เส้นทาง BTS')),
      body: GoogleMap(
        initialCameraPosition: const CameraPosition(
          target: LatLng(13.7438, 100.5328),
          zoom: 13,
        ),
        onMapCreated: (controller) => _mapController = controller,
        polylines: _polylines,
        markers: _markers,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 266: Polygons

```dart
// lib/pages/polygons_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class PolygonsPage extends StatefulWidget {
  const PolygonsPage({super.key});

  @override
  State<PolygonsPage> createState() => _PolygonsPageState();
}

class _PolygonsPageState extends State<PolygonsPage> {
  final Set<Polygon> _polygons = {};
  final Set<Circle> _circles = {};

  @override
  void initState() {
    super.initState();
    _addPolygons();
  }

  void _addPolygons() {
    // สี่เหลี่ยมรอบสยาม
    _polygons.add(
      Polygon(
        polygonId: const PolygonId('siam_area'),
        points: const [
          LatLng(13.7400, 100.5280),
          LatLng(13.7400, 100.5450),
          LatLng(13.7550, 100.5450),
          LatLng(13.7550, 100.5280),
        ],
        fillColor: Colors.blue.withOpacity(0.2),
        strokeColor: Colors.blue,
        strokeWidth: 2,
        consumeTapEvents: true,
        onTap: () {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('แถบสยาม-ราชประสงค์')),
          );
        },
      ),
    );

    // Circle รอบจุดสนใจ
    _circles.add(
      Circle(
        circleId: const CircleId('radius_circle'),
        center: const LatLng(13.7466, 100.5347),
        radius: 500, // 500 เมตร
        fillColor: Colors.red.withOpacity(0.1),
        strokeColor: Colors.red,
        strokeWidth: 2,
      ),
    );

    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Polygons & Circles')),
      body: GoogleMap(
        initialCameraPosition: const CameraPosition(
          target: LatLng(13.7466, 100.5347),
          zoom: 13,
        ),
        polygons: _polygons,
        circles: _circles,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 267: InfoWindows

```dart
// lib/pages/info_windows_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class InfoWindowsPage extends StatefulWidget {
  const InfoWindowsPage({super.key});

  @override
  State<InfoWindowsPage> createState() => _InfoWindowsPageState();
}

class _InfoWindowsPageState extends State<InfoWindowsPage> {
  GoogleMapController? _mapController;
  final Set<Marker> _markers = {};

  static const List<Map<String, dynamic>> _places = [
    {
      'id': 'thai_food',
      'name': 'ร้านอาหารไทย',
      'address': '123 ถ.สุขุมวิท',
      'rating': 4.5,
      'type': 'อาหาร',
      'lat': 13.7380,
      'lng': 100.5590,
    },
    {
      'id': 'coffee',
      'name': 'คาเฟ่แสนงาม',
      'address': '456 ถ.ทองหล่อ',
      'rating': 4.8,
      'type': 'คาเฟ่',
      'lat': 13.7290,
      'lng': 100.5840,
    },
    {
      'id': 'hotel',
      'name': 'โรงแรมสุขสบาย',
      'address': '789 ถ.สีลม',
      'rating': 4.2,
      'type': 'ที่พัก',
      'lat': 13.7258,
      'lng': 100.5278,
    },
  ];

  @override
  void initState() {
    super.initState();
    _addMarkers();
  }

  void _addMarkers() {
    for (final place in _places) {
      _markers.add(
        Marker(
          markerId: MarkerId(place['id'] as String),
          position: LatLng(place['lat'] as double, place['lng'] as double),
          infoWindow: InfoWindow(
            title: place['name'] as String,
            snippet: '⭐ ${place['rating']} • ${place['type']}',
            onTap: () => _showDetailBottomSheet(place),
          ),
          icon: _getMarkerIcon(place['type'] as String),
        ),
      );
    }
    setState(() {});
  }

  BitmapDescriptor _getMarkerIcon(String type) {
    switch (type) {
      case 'อาหาร':
        return BitmapDescriptor.defaultMarkerWithHue(BitmapDescriptor.hueRed);
      case 'คาเฟ่':
        return BitmapDescriptor.defaultMarkerWithHue(BitmapDescriptor.hueOrange);
      case 'ที่พัก':
        return BitmapDescriptor.defaultMarkerWithHue(BitmapDescriptor.hueBlue);
      default:
        return BitmapDescriptor.defaultMarker;
    }
  }

  void _showDetailBottomSheet(Map<String, dynamic> place) {
    showModalBottomSheet(
      context: context,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      builder: (ctx) => Container(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                Expanded(
                  child: Text(
                    place['name'] as String,
                    style: const TextStyle(
                      fontSize: 22,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
                Container(
                  padding: const EdgeInsets.symmetric(
                      horizontal: 8, vertical: 4),
                  decoration: BoxDecoration(
                    color: Colors.amber[100],
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      const Icon(Icons.star, color: Colors.amber, size: 16),
                      Text(' ${place['rating']}'),
                    ],
                  ),
                ),
              ],
            ),
            const SizedBox(height: 8),
            Text(
              place['address'] as String,
              style: TextStyle(color: Colors.grey[600], fontSize: 16),
            ),
            const SizedBox(height: 4),
            Chip(
              label: Text(place['type'] as String),
              backgroundColor: Colors.blue[100],
            ),
            const SizedBox(height: 16),
            Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () {},
                    icon: const Icon(Icons.directions),
                    label: const Text('นำทาง'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: () {},
                    icon: const Icon(Icons.phone),
                    label: const Text('โทร'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: () {},
                    icon: const Icon(Icons.share),
                    label: const Text('แชร์'),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('InfoWindows')),
      body: GoogleMap(
        initialCameraPosition: const CameraPosition(
          target: LatLng(13.7300, 100.5500),
          zoom: 12,
        ),
        onMapCreated: (controller) => _mapController = controller,
        markers: _markers,
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 268: Custom Map Styles

```dart
// lib/services/map_style_service.dart
class MapStyles {
  // Night Mode Style
  static const String nightStyle = '''
  [
    {
      "elementType": "geometry",
      "stylers": [{"color": "#212121"}]
    },
    {
      "elementType": "labels.text.fill",
      "stylers": [{"color": "#757575"}]
    },
    {
      "elementType": "labels.text.stroke",
      "stylers": [{"color": "#212121"}]
    },
    {
      "featureType": "road",
      "elementType": "geometry",
      "stylers": [{"color": "#373737"}]
    },
    {
      "featureType": "water",
      "elementType": "geometry",
      "stylers": [{"color": "#000000"}]
    }
  ]
  ''';

  // Minimal Style
  static const String minimalStyle = '''
  [
    {
      "featureType": "all",
      "elementType": "labels",
      "stylers": [{"visibility": "off"}]
    },
    {
      "featureType": "road",
      "elementType": "geometry",
      "stylers": [{"color": "#ffffff"}]
    },
    {
      "featureType": "water",
      "elementType": "geometry",
      "stylers": [{"color": "#93c4d0"}]
    }
  ]
  ''';

  // Retro Style
  static const String retroStyle = '''
  [
    {
      "elementType": "geometry",
      "stylers": [{"color": "#ebe3cd"}]
    },
    {
      "featureType": "road",
      "elementType": "geometry",
      "stylers": [{"color": "#f5f1e6"}]
    },
    {
      "featureType": "water",
      "elementType": "geometry.fill",
      "stylers": [{"color": "#b9d3c2"}]
    }
  ]
  ''';
}

// lib/pages/custom_style_map_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class CustomStyleMapPage extends StatefulWidget {
  const CustomStyleMapPage({super.key});

  @override
  State<CustomStyleMapPage> createState() => _CustomStyleMapPageState();
}

class _CustomStyleMapPageState extends State<CustomStyleMapPage> {
  GoogleMapController? _mapController;
  String _currentStyle = 'default';

  void _setMapStyle(String style) async {
    if (_mapController == null) return;
    switch (style) {
      case 'night':
        await _mapController!.setMapStyle(MapStyles.nightStyle);
        break;
      case 'minimal':
        await _mapController!.setMapStyle(MapStyles.minimalStyle);
        break;
      case 'retro':
        await _mapController!.setMapStyle(MapStyles.retroStyle);
        break;
      default:
        await _mapController!.setMapStyle(null);
    }
    setState(() => _currentStyle = style);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Custom Map Styles')),
      body: Stack(
        children: [
          GoogleMap(
            initialCameraPosition: const CameraPosition(
              target: LatLng(13.7563, 100.5018),
              zoom: 12,
            ),
            onMapCreated: (controller) {
              _mapController = controller;
            },
          ),
          Positioned(
            bottom: 16,
            left: 0,
            right: 0,
            child: SingleChildScrollView(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                children: [
                  _StyleButton('ปกติ', 'default'),
                  const SizedBox(width: 8),
                  _StyleButton('กลางคืน', 'night'),
                  const SizedBox(width: 8),
                  _StyleButton('เรียบง่าย', 'minimal'),
                  const SizedBox(width: 8),
                  _StyleButton('คลาสสิก', 'retro'),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  Widget _StyleButton(String label, String style) {
    final isSelected = _currentStyle == style;
    return ElevatedButton(
      onPressed: () => _setMapStyle(style),
      style: ElevatedButton.styleFrom(
        backgroundColor: isSelected ? Colors.blue : Colors.white,
        foregroundColor: isSelected ? Colors.white : Colors.black,
      ),
      child: Text(label),
    );
  }
}
```

---

## ขั้นตอนที่ 269: Geocoding

```dart
// lib/services/geocoding_service.dart
import 'package:geocoding/geocoding.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class GeocodingService {
  // แปลงที่อยู่เป็นพิกัด (Forward Geocoding)
  static Future<LatLng?> getCoordinatesFromAddress(String address) async {
    try {
      final locations = await locationFromAddress(address);
      if (locations.isNotEmpty) {
        return LatLng(
          locations.first.latitude,
          locations.first.longitude,
        );
      }
      return null;
    } catch (e) {
      throw Exception('ค้นหาที่อยู่ไม่พบ: $e');
    }
  }

  // แปลงพิกัดเป็นที่อยู่ (Reverse Geocoding)
  static Future<String?> getAddressFromCoordinates(
      double lat, double lng) async {
    try {
      final placemarks = await placemarkFromCoordinates(lat, lng);
      if (placemarks.isNotEmpty) {
        final place = placemarks.first;
        return _formatAddress(place);
      }
      return null;
    } catch (e) {
      throw Exception('ค้นหาที่อยู่ไม่สำเร็จ: $e');
    }
  }

  static String _formatAddress(Placemark place) {
    final parts = [
      place.street,
      place.subLocality,
      place.locality,
      place.administrativeArea,
      place.country,
    ].where((p) => p != null && p.isNotEmpty);

    return parts.join(', ');
  }
}

// lib/pages/geocoding_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class GeocodingPage extends StatefulWidget {
  const GeocodingPage({super.key});

  @override
  State<GeocodingPage> createState() => _GeocodingPageState();
}

class _GeocodingPageState extends State<GeocodingPage> {
  GoogleMapController? _mapController;
  final _searchController = TextEditingController();
  final Set<Marker> _markers = {};
  String? _address;
  bool _isLoading = false;

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  Future<void> _searchAddress() async {
    final query = _searchController.text.trim();
    if (query.isEmpty) return;

    setState(() => _isLoading = true);
    try {
      final coords = await GeocodingService.getCoordinatesFromAddress(query);
      if (coords != null) {
        setState(() {
          _markers.clear();
          _markers.add(
            Marker(
              markerId: const MarkerId('search_result'),
              position: coords,
              infoWindow: InfoWindow(title: query),
            ),
          );
        });

        _mapController?.animateCamera(
          CameraUpdate.newCameraPosition(
            CameraPosition(target: coords, zoom: 15),
          ),
        );

        // Reverse geocode เพื่อแสดงที่อยู่จริง
        final address = await GeocodingService.getAddressFromCoordinates(
          coords.latitude,
          coords.longitude,
        );
        setState(() => _address = address);
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('$e'), backgroundColor: Colors.red),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  Future<void> _onMapTap(LatLng position) async {
    setState(() {
      _markers.clear();
      _markers.add(
        Marker(
          markerId: const MarkerId('tapped'),
          position: position,
        ),
      );
      _isLoading = true;
    });

    try {
      final address = await GeocodingService.getAddressFromCoordinates(
        position.latitude,
        position.longitude,
      );
      setState(() => _address = address);
    } catch (e) {
      setState(() => _address = 'ไม่พบที่อยู่');
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Geocoding')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _searchController,
                    decoration: const InputDecoration(
                      hintText: 'ค้นหาที่อยู่...',
                      border: OutlineInputBorder(),
                      prefixIcon: Icon(Icons.search),
                    ),
                    onSubmitted: (_) => _searchAddress(),
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _isLoading ? null : _searchAddress,
                  child: _isLoading
                      ? const SizedBox(
                          width: 20,
                          height: 20,
                          child: CircularProgressIndicator(strokeWidth: 2),
                        )
                      : const Icon(Icons.search),
                ),
              ],
            ),
          ),
          if (_address != null)
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
              color: Colors.blue[50],
              child: Row(
                children: [
                  const Icon(Icons.location_on, color: Colors.blue),
                  const SizedBox(width: 8),
                  Expanded(child: Text(_address!)),
                ],
              ),
            ),
          Expanded(
            child: GoogleMap(
              initialCameraPosition: const CameraPosition(
                target: LatLng(13.7563, 100.5018),
                zoom: 12,
              ),
              onMapCreated: (controller) => _mapController = controller,
              markers: _markers,
              onTap: _onMapTap,
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 270: Place Picker & Directions API

```dart
// lib/services/directions_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:flutter_polyline_points/flutter_polyline_points.dart';

class DirectionsService {
  static const String _apiKey = 'YOUR_GOOGLE_MAPS_API_KEY';
  static const String _baseUrl =
      'https://maps.googleapis.com/maps/api/directions/json';

  static Future<DirectionsResult?> getDirections({
    required LatLng origin,
    required LatLng destination,
    String travelMode = 'driving',
  }) async {
    final url = Uri.parse(
      '$_baseUrl?origin=${origin.latitude},${origin.longitude}'
      '&destination=${destination.latitude},${destination.longitude}'
      '&mode=$travelMode'
      '&language=th'
      '&key=$_apiKey',
    );

    try {
      final response = await http.get(url);
      if (response.statusCode != 200) return null;

      final data = jsonDecode(response.body) as Map<String, dynamic>;
      if (data['status'] != 'OK') return null;

      final routes = data['routes'] as List;
      if (routes.isEmpty) return null;

      final route = routes[0] as Map<String, dynamic>;
      final leg = (route['legs'] as List)[0] as Map<String, dynamic>;

      // Decode Polyline
      final polylinePoints = PolylinePoints();
      final points = polylinePoints.decodePolyline(
        route['overview_polyline']['points'] as String,
      );

      final latLngPoints = points
          .map((point) => LatLng(point.latitude, point.longitude))
          .toList();

      return DirectionsResult(
        distance: leg['distance']['text'] as String,
        duration: leg['duration']['text'] as String,
        polylinePoints: latLngPoints,
        steps: (leg['steps'] as List).map((step) {
          return DirectionStep(
            instruction: _stripHtml(step['html_instructions'] as String),
            distance: step['distance']['text'] as String,
            duration: step['duration']['text'] as String,
          );
        }).toList(),
      );
    } catch (e) {
      throw Exception('ดึงข้อมูลเส้นทางล้มเหลว: $e');
    }
  }

  static String _stripHtml(String html) {
    return html.replaceAll(RegExp(r'<[^>]*>'), '');
  }
}

class DirectionsResult {
  const DirectionsResult({
    required this.distance,
    required this.duration,
    required this.polylinePoints,
    required this.steps,
  });

  final String distance;
  final String duration;
  final List<LatLng> polylinePoints;
  final List<DirectionStep> steps;
}

class DirectionStep {
  const DirectionStep({
    required this.instruction,
    required this.distance,
    required this.duration,
  });

  final String instruction;
  final String distance;
  final String duration;
}

// lib/pages/navigation_page.dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import '../services/location_service.dart';

class NavigationPage extends StatefulWidget {
  const NavigationPage({super.key});

  @override
  State<NavigationPage> createState() => _NavigationPageState();
}

class _NavigationPageState extends State<NavigationPage> {
  GoogleMapController? _mapController;
  final Set<Marker> _markers = {};
  final Set<Polyline> _polylines = {};
  DirectionsResult? _directions;
  bool _isLoading = false;
  LatLng? _origin;
  LatLng? _destination;

  static const _destinationOptions = [
    {'name': 'สนามบินสุวรรณภูมิ', 'lat': 13.6900, 'lng': 100.7501},
    {'name': 'วัดพระแก้ว', 'lat': 13.7516, 'lng': 100.4926},
    {'name': 'เขาเขียว', 'lat': 13.1942, 'lng': 101.1553},
  ];

  Future<void> _getDirections(LatLng destination) async {
    setState(() => _isLoading = true);
    try {
      final position = await LocationService.getCurrentLocation();
      if (position == null) return;

      _origin = LatLng(position.latitude, position.longitude);
      _destination = destination;

      final result = await DirectionsService.getDirections(
        origin: _origin!,
        destination: _destination!,
      );

      if (result != null) {
        setState(() {
          _directions = result;
          _markers.clear();
          _markers.addAll([
            Marker(
              markerId: const MarkerId('origin'),
              position: _origin!,
              icon: BitmapDescriptor.defaultMarkerWithHue(
                BitmapDescriptor.hueGreen,
              ),
              infoWindow: const InfoWindow(title: 'จุดเริ่มต้น'),
            ),
            Marker(
              markerId: const MarkerId('destination'),
              position: destination,
              infoWindow: const InfoWindow(title: 'จุดหมาย'),
            ),
          ]);

          _polylines.clear();
          _polylines.add(
            Polyline(
              polylineId: const PolylineId('route'),
              points: result.polylinePoints,
              color: Colors.blue,
              width: 5,
            ),
          );
        });

        // Fit bounds
        final bounds = LatLngBounds(
          southwest: LatLng(
            [_origin!.latitude, destination.latitude].reduce((a, b) => a < b ? a : b),
            [_origin!.longitude, destination.longitude].reduce((a, b) => a < b ? a : b),
          ),
          northeast: LatLng(
            [_origin!.latitude, destination.latitude].reduce((a, b) => a > b ? a : b),
            [_origin!.longitude, destination.longitude].reduce((a, b) => a > b ? a : b),
          ),
        );
        _mapController?.animateCamera(
          CameraUpdate.newLatLngBounds(bounds, 80),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('$e'), backgroundColor: Colors.red),
        );
      }
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('นำทาง')),
      body: Stack(
        children: [
          GoogleMap(
            initialCameraPosition: const CameraPosition(
              target: LatLng(13.7563, 100.5018),
              zoom: 11,
            ),
            onMapCreated: (controller) => _mapController = controller,
            markers: _markers,
            polylines: _polylines,
            myLocationEnabled: true,
          ),
          if (_isLoading)
            const Center(child: CircularProgressIndicator()),
          Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            child: Column(
              children: [
                if (_directions != null)
                  Container(
                    margin: const EdgeInsets.all(8),
                    padding: const EdgeInsets.all(12),
                    decoration: BoxDecoration(
                      color: Colors.white,
                      borderRadius: BorderRadius.circular(8),
                      boxShadow: [
                        BoxShadow(
                          color: Colors.black.withOpacity(0.1),
                          blurRadius: 8,
                        ),
                      ],
                    ),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceAround,
                      children: [
                        Column(
                          children: [
                            const Icon(Icons.directions_car),
                            Text(
                              _directions!.distance,
                              style: const TextStyle(fontWeight: FontWeight.bold),
                            ),
                            const Text('ระยะทาง'),
                          ],
                        ),
                        Column(
                          children: [
                            const Icon(Icons.access_time),
                            Text(
                              _directions!.duration,
                              style: const TextStyle(fontWeight: FontWeight.bold),
                            ),
                            const Text('เวลา'),
                          ],
                        ),
                      ],
                    ),
                  ),
                Container(
                  color: Colors.white,
                  padding: const EdgeInsets.all(8),
                  child: SingleChildScrollView(
                    scrollDirection: Axis.horizontal,
                    child: Row(
                      children: _destinationOptions.map((dest) {
                        return Padding(
                          padding: const EdgeInsets.only(right: 8),
                          child: ElevatedButton(
                            onPressed: () => _getDirections(
                              LatLng(dest['lat'] as double, dest['lng'] as double),
                            ),
                            child: Text(dest['name'] as String),
                          ),
                        );
                      }).toList(),
                    ),
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
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Google Maps Flutter**: การแสดงแผนที่
- **Geolocator**: การดึงตำแหน่ง GPS
- **Markers**: การเพิ่ม/จัดการหมุดบนแผนที่
- **Polylines**: การวาดเส้นทาง
- **Polygons & Circles**: การวาดพื้นที่
- **InfoWindows**: การแสดงข้อมูลเมื่อแตะ Marker
- **Custom Styles**: การปรับแต่งรูปแบบแผนที่
- **Geocoding**: การแปลงที่อยู่↔พิกัด
- **Directions API**: การนำทาง

## แบบฝึกหัด

1. สร้าง Nearby Places App ที่แสดงร้านอาหารใกล้เคียง
2. Implement Real-time Tracking ของ Location
3. สร้าง Geo-fence Alert เมื่อผู้ใช้เข้าพื้นที่ที่กำหนด
4. เพิ่ม Traffic Layer บนแผนที่
5. สร้าง Route Comparison ระหว่าง Walk/Drive/Transit

---

[⬅️ Part 26](part_26.md) | [Part 28 ➡️](part_28.md)

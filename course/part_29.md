# Part 29: Platform Channels (Native Code)
## ขั้นตอนที่ 281-290

---

## สารบัญ
1. [MethodChannel](#methodchannel)
2. [EventChannel](#eventchannel)
3. [BasicMessageChannel](#basicmessagechannel)
4. [Calling Android (Kotlin)](#calling-android-kotlin)
5. [Calling Android (Java)](#calling-android-java)
6. [Calling iOS (Swift)](#calling-ios-swift)
7. [Calling iOS (Obj-C)](#calling-ios-obj-c)
8. [Returning Results & Error Handling](#returning-results--error-handling)
9. [Platform-Specific UI](#platform-specific-ui)
10. [FFI Basics](#ffi-basics)

---

## ขั้นตอนที่ 281: MethodChannel

MethodChannel ใช้สำหรับเรียก Method บน Native Platform และรับผลลัพธ์กลับมา

```dart
// lib/services/platform_service.dart
import 'package:flutter/services.dart';
import 'package:flutter/foundation.dart';

class PlatformService {
  static const MethodChannel _channel =
      MethodChannel('com.example.myapp/platform');

  // เรียก Native Method
  static Future<String> getPlatformVersion() async {
    try {
      final version =
          await _channel.invokeMethod<String>('getPlatformVersion');
      return version ?? 'Unknown';
    } on PlatformException catch (e) {
      throw Exception('Platform error: ${e.message}');
    }
  }

  // ดึงข้อมูล Device
  static Future<Map<String, dynamic>> getDeviceInfo() async {
    try {
      final result =
          await _channel.invokeMethod<Map<dynamic, dynamic>>('getDeviceInfo');
      if (result == null) return {};
      return result.map((k, v) => MapEntry(k.toString(), v));
    } on PlatformException catch (e) {
      throw Exception('Device info error: ${e.message}');
    }
  }

  // เปิด URL ด้วย Native Browser
  static Future<bool> openUrl(String url) async {
    try {
      final result = await _channel.invokeMethod<bool>('openUrl', {'url': url});
      return result ?? false;
    } on PlatformException catch (e) {
      throw Exception('Open URL error: ${e.message}');
    }
  }

  // Share ข้อความ
  static Future<void> shareText(String text) async {
    try {
      await _channel.invokeMethod('shareText', {'text': text});
    } on PlatformException catch (e) {
      throw Exception('Share error: ${e.message}');
    }
  }

  // ตรวจสอบ Platform
  static bool get isAndroid => defaultTargetPlatform == TargetPlatform.android;
  static bool get isIOS => defaultTargetPlatform == TargetPlatform.iOS;
  static bool get isWeb => kIsWeb;
  static bool get isDesktop => defaultTargetPlatform == TargetPlatform.macOS ||
      defaultTargetPlatform == TargetPlatform.windows ||
      defaultTargetPlatform == TargetPlatform.linux;
}
```

```dart
// lib/pages/platform_channel_page.dart
import 'package:flutter/material.dart';
import '../services/platform_service.dart';

class PlatformChannelPage extends StatefulWidget {
  const PlatformChannelPage({super.key});

  @override
  State<PlatformChannelPage> createState() => _PlatformChannelPageState();
}

class _PlatformChannelPageState extends State<PlatformChannelPage> {
  String _platformVersion = 'กำลังโหลด...';
  Map<String, dynamic> _deviceInfo = {};

  @override
  void initState() {
    super.initState();
    _loadPlatformInfo();
  }

  Future<void> _loadPlatformInfo() async {
    try {
      final version = await PlatformService.getPlatformVersion();
      final info = await PlatformService.getDeviceInfo();
      setState(() {
        _platformVersion = version;
        _deviceInfo = info;
      });
    } catch (e) {
      setState(() => _platformVersion = 'Error: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Platform Channels')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Platform Version',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  Text(_platformVersion),
                ],
              ),
            ),
          ),
          const SizedBox(height: 16),
          Card(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text(
                    'Device Info',
                    style: TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 8),
                  ..._deviceInfo.entries.map(
                    (e) => Padding(
                      padding: const EdgeInsets.only(bottom: 4),
                      child: Row(
                        children: [
                          Text(
                            '${e.key}: ',
                            style: const TextStyle(fontWeight: FontWeight.bold),
                          ),
                          Expanded(child: Text('${e.value}')),
                        ],
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
          const SizedBox(height: 16),
          ElevatedButton.icon(
            onPressed: () => PlatformService.shareText('แชร์จาก Flutter App!'),
            icon: const Icon(Icons.share),
            label: const Text('Share'),
          ),
          const SizedBox(height: 8),
          ElevatedButton.icon(
            onPressed: () =>
                PlatformService.openUrl('https://flutter.dev'),
            icon: const Icon(Icons.open_in_browser),
            label: const Text('เปิด Flutter.dev'),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 282: EventChannel

EventChannel ใช้สำหรับรับ Stream ของ Events จาก Native Platform

```dart
// lib/services/sensor_service.dart
import 'package:flutter/services.dart';

class SensorService {
  static const EventChannel _accelerometerChannel =
      EventChannel('com.example.myapp/accelerometer');

  static const EventChannel _batteryChannel =
      EventChannel('com.example.myapp/battery');

  // Stream ข้อมูล Accelerometer
  static Stream<AccelerometerData> get accelerometerStream {
    return _accelerometerChannel
        .receiveBroadcastStream()
        .map((dynamic data) {
      final map = Map<String, dynamic>.from(data as Map);
      return AccelerometerData(
        x: (map['x'] as num).toDouble(),
        y: (map['y'] as num).toDouble(),
        z: (map['z'] as num).toDouble(),
      );
    });
  }

  // Stream ข้อมูล Battery
  static Stream<int> get batteryLevelStream {
    return _batteryChannel
        .receiveBroadcastStream()
        .map((dynamic level) => level as int);
  }
}

class AccelerometerData {
  const AccelerometerData({
    required this.x,
    required this.y,
    required this.z,
  });

  final double x;
  final double y;
  final double z;

  @override
  String toString() =>
      'x: ${x.toStringAsFixed(2)}, y: ${y.toStringAsFixed(2)}, z: ${z.toStringAsFixed(2)}';
}

// lib/pages/sensor_page.dart
import 'package:flutter/material.dart';

class SensorPage extends StatelessWidget {
  const SensorPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Sensor Data')),
      body: Column(
        children: [
          // Accelerometer
          Expanded(
            child: StreamBuilder<AccelerometerData>(
              stream: SensorService.accelerometerStream,
              builder: (context, snapshot) {
                if (snapshot.hasError) {
                  return Center(child: Text('Error: ${snapshot.error}'));
                }
                if (!snapshot.hasData) {
                  return const Center(child: CircularProgressIndicator());
                }
                final data = snapshot.data!;
                return Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      const Text(
                        'Accelerometer',
                        style: TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 24),
                      _SensorBar(
                          label: 'X', value: data.x, color: Colors.red),
                      const SizedBox(height: 8),
                      _SensorBar(
                          label: 'Y', value: data.y, color: Colors.green),
                      const SizedBox(height: 8),
                      _SensorBar(
                          label: 'Z', value: data.z, color: Colors.blue),
                    ],
                  ),
                );
              },
            ),
          ),

          // Battery
          StreamBuilder<int>(
            stream: SensorService.batteryLevelStream,
            builder: (context, snapshot) {
              final level = snapshot.data ?? 0;
              return Container(
                padding: const EdgeInsets.all(16),
                color: Colors.grey[100],
                child: Row(
                  children: [
                    Icon(
                      level > 20 ? Icons.battery_std : Icons.battery_alert,
                      color: level > 20 ? Colors.green : Colors.red,
                    ),
                    const SizedBox(width: 8),
                    Text('Battery: $level%'),
                  ],
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}

class _SensorBar extends StatelessWidget {
  const _SensorBar({
    required this.label,
    required this.value,
    required this.color,
  });

  final String label;
  final double value;
  final Color color;

  @override
  Widget build(BuildContext context) {
    final normalized = (value + 10) / 20;
    return Row(
      children: [
        SizedBox(
          width: 30,
          child: Text(label, style: const TextStyle(fontWeight: FontWeight.bold)),
        ),
        Expanded(
          child: LinearProgressIndicator(
            value: normalized.clamp(0.0, 1.0),
            backgroundColor: Colors.grey[300],
            valueColor: AlwaysStoppedAnimation<Color>(color),
            minHeight: 16,
          ),
        ),
        const SizedBox(width: 8),
        SizedBox(
          width: 50,
          child: Text(value.toStringAsFixed(2)),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 283: BasicMessageChannel

BasicMessageChannel ใช้สำหรับรับส่งข้อความแบบง่ายๆ

```dart
// lib/services/message_channel_service.dart
import 'package:flutter/services.dart';

class MessageChannelService {
  static const BasicMessageChannel<String?> _channel =
      BasicMessageChannel<String?>(
    'com.example.myapp/messages',
    StringCodec(),
  );

  static Future<void> initialize() async {
    // รับ Messages จาก Native
    _channel.setMessageHandler((String? message) async {
      if (message == null) return null;

      print('Received from native: $message');

      // ส่ง Reply กลับ
      return 'Received: $message';
    });
  }

  // ส่ง Message ไป Native
  static Future<String?> sendMessage(String message) async {
    final reply = await _channel.send(message);
    return reply;
  }
}
```

---

## ขั้นตอนที่ 284: Calling Android (Kotlin)

```kotlin
// android/app/src/main/kotlin/com/example/myapp/MainActivity.kt
package com.example.myapp

import android.content.Intent
import android.net.Uri
import android.os.Build
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.EventChannel
import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {
    
    private val PLATFORM_CHANNEL = "com.example.myapp/platform"
    private val EVENT_CHANNEL = "com.example.myapp/accelerometer"
    
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        
        // MethodChannel
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, PLATFORM_CHANNEL)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getPlatformVersion" -> {
                        result.success("Android ${Build.VERSION.RELEASE}")
                    }
                    "getDeviceInfo" -> {
                        val info = mapOf(
                            "brand" to Build.BRAND,
                            "model" to Build.MODEL,
                            "manufacturer" to Build.MANUFACTURER,
                            "sdk" to Build.VERSION.SDK_INT.toString(),
                            "release" to Build.VERSION.RELEASE
                        )
                        result.success(info)
                    }
                    "openUrl" -> {
                        val url = call.argument<String>("url")
                        if (url != null) {
                            try {
                                val intent = Intent(Intent.ACTION_VIEW, Uri.parse(url))
                                startActivity(intent)
                                result.success(true)
                            } catch (e: Exception) {
                                result.error("OPEN_URL_ERROR", e.message, null)
                            }
                        } else {
                            result.error("INVALID_URL", "URL is null", null)
                        }
                    }
                    "shareText" -> {
                        val text = call.argument<String>("text")
                        if (text != null) {
                            val sendIntent = Intent().apply {
                                action = Intent.ACTION_SEND
                                putExtra(Intent.EXTRA_TEXT, text)
                                type = "text/plain"
                            }
                            startActivity(Intent.createChooser(sendIntent, "Share"))
                            result.success(null)
                        } else {
                            result.error("INVALID_TEXT", "Text is null", null)
                        }
                    }
                    "getBatteryLevel" -> {
                        val batteryLevel = getBatteryLevel()
                        if (batteryLevel != -1) {
                            result.success(batteryLevel)
                        } else {
                            result.error("UNAVAILABLE", "Battery level not available", null)
                        }
                    }
                    else -> result.notImplemented()
                }
            }
        
        // EventChannel สำหรับ Battery Events
        EventChannel(
            flutterEngine.dartExecutor.binaryMessenger,
            "com.example.myapp/battery"
        ).setStreamHandler(object : EventChannel.StreamHandler {
            private var eventSink: EventChannel.EventSink? = null
            
            override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
                eventSink = events
                // Start sending battery updates
                startBatteryMonitoring()
            }
            
            override fun onCancel(arguments: Any?) {
                eventSink = null
            }
            
            private fun startBatteryMonitoring() {
                // ส่ง Battery level ทุก 5 วินาที
                Thread {
                    while (eventSink != null) {
                        val level = getBatteryLevel()
                        runOnUiThread {
                            eventSink?.success(level)
                        }
                        Thread.sleep(5000)
                    }
                }.start()
            }
        })
    }
    
    private fun getBatteryLevel(): Int {
        val batteryManager = getSystemService(BATTERY_SERVICE) 
            as android.os.BatteryManager
        return batteryManager.getIntProperty(
            android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY
        )
    }
}
```

---

## ขั้นตอนที่ 285: Calling Android (Java)

```java
// android/app/src/main/java/com/example/myapp/MainActivityJava.java
package com.example.myapp;

import android.content.Intent;
import android.net.Uri;
import android.os.Build;
import io.flutter.embedding.android.FlutterActivity;
import io.flutter.embedding.engine.FlutterEngine;
import io.flutter.plugin.common.MethodCall;
import io.flutter.plugin.common.MethodChannel;
import java.util.HashMap;
import java.util.Map;

public class MainActivityJava extends FlutterActivity {
    
    private static final String CHANNEL = "com.example.myapp/platform";
    
    @Override
    public void configureFlutterEngine(FlutterEngine flutterEngine) {
        super.configureFlutterEngine(flutterEngine);
        
        new MethodChannel(
            flutterEngine.getDartExecutor().getBinaryMessenger(),
            CHANNEL
        ).setMethodCallHandler(new MethodChannel.MethodCallHandler() {
            @Override
            public void onMethodCall(MethodCall call, MethodChannel.Result result) {
                if (call.method.equals("getPlatformVersion")) {
                    result.success("Android " + Build.VERSION.RELEASE);
                } else if (call.method.equals("getDeviceInfo")) {
                    Map<String, String> info = new HashMap<>();
                    info.put("brand", Build.BRAND);
                    info.put("model", Build.MODEL);
                    info.put("manufacturer", Build.MANUFACTURER);
                    result.success(info);
                } else if (call.method.equals("openUrl")) {
                    String url = call.argument("url");
                    if (url != null) {
                        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
                        startActivity(intent);
                        result.success(true);
                    } else {
                        result.error("INVALID_URL", "URL is null", null);
                    }
                } else {
                    result.notImplemented();
                }
            }
        });
    }
}
```

---

## ขั้นตอนที่ 286: Calling iOS (Swift)

```swift
// ios/Runner/AppDelegate.swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        
        let controller = window?.rootViewController as! FlutterViewController
        setupMethodChannel(controller: controller)
        setupEventChannel(controller: controller)
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
    
    private func setupMethodChannel(controller: FlutterViewController) {
        let channel = FlutterMethodChannel(
            name: "com.example.myapp/platform",
            binaryMessenger: controller.binaryMessenger
        )
        
        channel.setMethodCallHandler { [weak self] (call, result) in
            switch call.method {
            case "getPlatformVersion":
                let version = UIDevice.current.systemVersion
                result("iOS \(version)")
                
            case "getDeviceInfo":
                let device = UIDevice.current
                let info: [String: String] = [
                    "name": device.name,
                    "model": device.model,
                    "systemName": device.systemName,
                    "systemVersion": device.systemVersion,
                    "identifierForVendor": device.identifierForVendor?.uuidString ?? "unknown"
                ]
                result(info)
                
            case "openUrl":
                guard let args = call.arguments as? [String: Any],
                      let urlString = args["url"] as? String,
                      let url = URL(string: urlString) else {
                    result(FlutterError(code: "INVALID_URL", message: "URL invalid", details: nil))
                    return
                }
                if UIApplication.shared.canOpenURL(url) {
                    UIApplication.shared.open(url, options: [:]) { success in
                        result(success)
                    }
                } else {
                    result(false)
                }
                
            case "shareText":
                guard let args = call.arguments as? [String: Any],
                      let text = args["text"] as? String else {
                    result(FlutterError(code: "INVALID_TEXT", message: "Text invalid", details: nil))
                    return
                }
                let activityVC = UIActivityViewController(
                    activityItems: [text],
                    applicationActivities: nil
                )
                if let rootVC = UIApplication.shared.keyWindow?.rootViewController {
                    rootVC.present(activityVC, animated: true)
                }
                result(nil)
                
            case "getBatteryLevel":
                UIDevice.current.isBatteryMonitoringEnabled = true
                let level = Int(UIDevice.current.batteryLevel * 100)
                result(level)
                
            default:
                result(FlutterMethodNotImplemented)
            }
        }
    }
    
    private func setupEventChannel(controller: FlutterViewController) {
        let batteryChannel = FlutterEventChannel(
            name: "com.example.myapp/battery",
            binaryMessenger: controller.binaryMessenger
        )
        batteryChannel.setStreamHandler(BatteryStreamHandler())
    }
}

// Stream Handler สำหรับ Battery
class BatteryStreamHandler: NSObject, FlutterStreamHandler {
    private var eventSink: FlutterEventSink?
    private var timer: Timer?
    
    func onListen(withArguments arguments: Any?, eventSink events: @escaping FlutterEventSink) -> FlutterError? {
        self.eventSink = events
        UIDevice.current.isBatteryMonitoringEnabled = true
        
        // ส่ง Battery level ทุก 5 วินาที
        timer = Timer.scheduledTimer(withTimeInterval: 5, repeats: true) { _ in
            let level = Int(UIDevice.current.batteryLevel * 100)
            self.eventSink?(level)
        }
        
        // ส่งค่าเริ่มต้นทันที
        let initialLevel = Int(UIDevice.current.batteryLevel * 100)
        events(initialLevel)
        
        return nil
    }
    
    func onCancel(withArguments arguments: Any?) -> FlutterError? {
        timer?.invalidate()
        timer = nil
        eventSink = nil
        UIDevice.current.isBatteryMonitoringEnabled = false
        return nil
    }
}
```

---

## ขั้นตอนที่ 287: Calling iOS (Obj-C)

```objc
// ios/Runner/AppDelegate.m
#import "AppDelegate.h"
#import <Flutter/Flutter.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    FlutterViewController* controller = (FlutterViewController*)self.window.rootViewController;
    
    FlutterMethodChannel* platformChannel =
        [FlutterMethodChannel methodChannelWithName:@"com.example.myapp/platform"
                                    binaryMessenger:controller.binaryMessenger];
    
    [platformChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
        if ([@"getPlatformVersion" isEqualToString:call.method]) {
            NSString* version = [[UIDevice currentDevice] systemVersion];
            result([@"iOS " stringByAppendingString:version]);
        } else if ([@"getDeviceInfo" isEqualToString:call.method]) {
            NSDictionary* info = @{
                @"name": [[UIDevice currentDevice] name],
                @"model": [[UIDevice currentDevice] model],
                @"systemVersion": [[UIDevice currentDevice] systemVersion]
            };
            result(info);
        } else {
            result(FlutterMethodNotImplemented);
        }
    }];
    
    [GeneratedPluginRegistrant registerWithRegistry:self];
    return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end
```

---

## ขั้นตอนที่ 288: Returning Results & Error Handling

```dart
// lib/services/robust_platform_service.dart
import 'package:flutter/services.dart';

class RobustPlatformService {
  static const MethodChannel _channel =
      MethodChannel('com.example.myapp/platform');

  // Pattern: Safe Call with Error Handling
  static Future<T?> safeCall<T>(
    String method, [
    Map<String, dynamic>? arguments,
  ]) async {
    try {
      final result = await _channel.invokeMethod<T>(method, arguments);
      return result;
    } on PlatformException catch (e) {
      debugPrint('PlatformException: ${e.code} - ${e.message}');
      switch (e.code) {
        case 'UNAVAILABLE':
          throw UnavailableException(e.message ?? 'Service unavailable');
        case 'PERMISSION_DENIED':
          throw PermissionDeniedException(e.message ?? 'Permission denied');
        case 'INVALID_ARGUMENT':
          throw ArgumentError(e.message ?? 'Invalid argument');
        default:
          throw PlatformOperationException(
              e.code, e.message ?? 'Unknown error');
      }
    } on MissingPluginException catch (e) {
      debugPrint('Plugin not implemented: $e');
      throw UnimplementedError('This feature is not available on this platform');
    } catch (e) {
      debugPrint('Unexpected error: $e');
      rethrow;
    }
  }

  // ส่ง Arguments ที่ซับซ้อน
  static Future<Map<String, dynamic>> getComplexData(
      String query, int limit) async {
    final result = await safeCall<Map<dynamic, dynamic>>(
      'getComplexData',
      {'query': query, 'limit': limit, 'timestamp': DateTime.now().toIso8601String()},
    );

    if (result == null) return {};
    return result.map((k, v) => MapEntry(k.toString(), v));
  }
}

// Custom Exceptions
class UnavailableException implements Exception {
  const UnavailableException(this.message);
  final String message;

  @override
  String toString() => 'UnavailableException: $message';
}

class PermissionDeniedException implements Exception {
  const PermissionDeniedException(this.message);
  final String message;

  @override
  String toString() => 'PermissionDeniedException: $message';
}

class PlatformOperationException implements Exception {
  const PlatformOperationException(this.code, this.message);
  final String code;
  final String message;

  @override
  String toString() => 'PlatformOperationException[$code]: $message';
}

import 'package:flutter/foundation.dart';
```

---

## ขั้นตอนที่ 289: Platform-Specific UI

```dart
// lib/widgets/platform_ui_widgets.dart
import 'dart:io';
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';

// Platform-specific Switch
class PlatformSwitch extends StatelessWidget {
  const PlatformSwitch({
    super.key,
    required this.value,
    required this.onChanged,
  });

  final bool value;
  final Function(bool) onChanged;

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoSwitch(value: value, onChanged: onChanged);
    }
    return Switch(value: value, onChanged: onChanged);
  }
}

// Platform-specific Slider
class PlatformSlider extends StatelessWidget {
  const PlatformSlider({
    super.key,
    required this.value,
    required this.onChanged,
    this.min = 0.0,
    this.max = 1.0,
  });

  final double value;
  final Function(double) onChanged;
  final double min;
  final double max;

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoSlider(
        value: value,
        onChanged: onChanged,
        min: min,
        max: max,
      );
    }
    return Slider(
      value: value,
      onChanged: onChanged,
      min: min,
      max: max,
    );
  }
}

// Platform-specific Dialog
Future<bool?> showPlatformDialog(
  BuildContext context, {
  required String title,
  required String message,
  String confirmLabel = 'ตกลง',
  String cancelLabel = 'ยกเลิก',
}) async {
  if (Platform.isIOS) {
    return showCupertinoDialog<bool>(
      context: context,
      builder: (ctx) => CupertinoAlertDialog(
        title: Text(title),
        content: Text(message),
        actions: [
          CupertinoDialogAction(
            isDestructiveAction: true,
            onPressed: () => Navigator.pop(ctx, false),
            child: Text(cancelLabel),
          ),
          CupertinoDialogAction(
            onPressed: () => Navigator.pop(ctx, true),
            child: Text(confirmLabel),
          ),
        ],
      ),
    );
  }

  return showDialog<bool>(
    context: context,
    builder: (ctx) => AlertDialog(
      title: Text(title),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(ctx, false),
          child: Text(cancelLabel),
        ),
        ElevatedButton(
          onPressed: () => Navigator.pop(ctx, true),
          child: Text(confirmLabel),
        ),
      ],
    ),
  );
}

// Platform-specific Button
class PlatformButton extends StatelessWidget {
  const PlatformButton({
    super.key,
    required this.onPressed,
    required this.child,
  });

  final VoidCallback onPressed;
  final Widget child;

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoButton(
        onPressed: onPressed,
        child: child,
      );
    }
    return ElevatedButton(
      onPressed: onPressed,
      child: child,
    );
  }
}

// Platform-specific Page Scaffold
class PlatformScaffold extends StatelessWidget {
  const PlatformScaffold({
    super.key,
    required this.title,
    required this.body,
    this.actions,
    this.leading,
  });

  final String title;
  final Widget body;
  final List<Widget>? actions;
  final Widget? leading;

  @override
  Widget build(BuildContext context) {
    if (Platform.isIOS) {
      return CupertinoPageScaffold(
        navigationBar: CupertinoNavigationBar(
          middle: Text(title),
          leading: leading,
          trailing: actions != null ? Row(
            mainAxisSize: MainAxisSize.min,
            children: actions!,
          ) : null,
        ),
        child: body,
      );
    }

    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        leading: leading,
        actions: actions,
      ),
      body: body,
    );
  }
}

// ตัวอย่างการใช้ Platform-specific Styling
class PlatformAwarePage extends StatefulWidget {
  const PlatformAwarePage({super.key});

  @override
  State<PlatformAwarePage> createState() => _PlatformAwarePageState();
}

class _PlatformAwarePageState extends State<PlatformAwarePage> {
  bool _switchValue = false;
  double _sliderValue = 0.5;

  @override
  Widget build(BuildContext context) {
    return PlatformScaffold(
      title: 'Platform UI',
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Text(
            Platform.isIOS ? 'iOS Style' : 'Android Style',
            style: const TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          const SizedBox(height: 20),

          ListTile(
            title: const Text('การตั้งค่า'),
            trailing: PlatformSwitch(
              value: _switchValue,
              onChanged: (v) => setState(() => _switchValue = v),
            ),
          ),

          const SizedBox(height: 16),
          Text('ระดับเสียง: ${(_sliderValue * 100).toInt()}%'),
          PlatformSlider(
            value: _sliderValue,
            onChanged: (v) => setState(() => _sliderValue = v),
          ),

          const SizedBox(height: 24),
          PlatformButton(
            onPressed: () => showPlatformDialog(
              context,
              title: 'ยืนยัน',
              message: 'ต้องการดำเนินการหรือไม่?',
            ),
            child: const Text('แสดง Dialog'),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 290: FFI Basics

```dart
// lib/services/ffi_service.dart
import 'dart:ffi';
import 'dart:io';
import 'package:ffi/ffi.dart';

// FFI สำหรับเรียก Native C Functions

// ประกาศ Native Function Types
typedef NativeAddFunc = Int32 Function(Int32 a, Int32 b);
typedef DartAddFunc = int Function(int a, int b);

typedef NativeStringFunc = Pointer<Utf8> Function(Pointer<Utf8> input);
typedef DartStringFunc = Pointer<Utf8> Function(Pointer<Utf8> input);

class FFIService {
  static late final DynamicLibrary _lib;
  static late final DartAddFunc _addNumbers;
  static late final DartStringFunc _processString;

  static void initialize() {
    // โหลด Library ตาม Platform
    _lib = Platform.isAndroid
        ? DynamicLibrary.open('libmy_native.so')
        : Platform.isIOS
            ? DynamicLibrary.process()
            : DynamicLibrary.open('my_native.dll');

    // Lookup Functions
    _addNumbers = _lib
        .lookup<NativeFunction<NativeAddFunc>>('add_numbers')
        .asFunction<DartAddFunc>();

    _processString = _lib
        .lookup<NativeFunction<NativeStringFunc>>('process_string')
        .asFunction<DartStringFunc>();
  }

  // เรียก C Function: int add_numbers(int a, int b)
  static int addNumbers(int a, int b) {
    return _addNumbers(a, b);
  }

  // เรียก C Function ที่รับและส่งคืน String
  static String processString(String input) {
    final nativeInput = input.toNativeUtf8();
    try {
      final nativeResult = _processString(nativeInput);
      return nativeResult.toDartString();
    } finally {
      malloc.free(nativeInput);
    }
  }
}

// ตัวอย่าง C Code ที่ต้อง compile
/*
// native/my_native.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// ฟังก์ชันบวกเลข
int add_numbers(int a, int b) {
    return a + b;
}

// ฟังก์ชันจัดการ String
char* process_string(const char* input) {
    size_t len = strlen(input);
    char* result = (char*)malloc(len + 20);
    snprintf(result, len + 20, "Processed: %s", input);
    return result;
}
*/

// lib/pages/ffi_demo_page.dart
import 'package:flutter/material.dart';

class FFIDemoPage extends StatefulWidget {
  const FFIDemoPage({super.key});

  @override
  State<FFIDemoPage> createState() => _FFIDemoPageState();
}

class _FFIDemoPageState extends State<FFIDemoPage> {
  String _result = 'กด ทดสอบ FFI';

  void _testFFI() {
    try {
      // ตัวอย่างการเรียก FFI
      // final sum = FFIService.addNumbers(10, 32);
      // final processed = FFIService.processString('Hello');
      // setState(() => _result = 'Sum: $sum\nProcessed: $processed');

      // Demo ไม่ใช้ FFI จริง
      setState(() {
        _result = 'FFI Demo:\n'
            '10 + 32 = 42\n'
            'Processed: Hello World';
      });
    } catch (e) {
      setState(() => _result = 'Error: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FFI Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.memory, size: 80, color: Colors.purple),
            const SizedBox(height: 24),
            const Text(
              'Flutter Foreign Function Interface',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 16),
            Container(
              padding: const EdgeInsets.all(16),
              margin: const EdgeInsets.symmetric(horizontal: 24),
              decoration: BoxDecoration(
                color: Colors.grey[100],
                borderRadius: BorderRadius.circular(8),
              ),
              child: Text(
                _result,
                textAlign: TextAlign.center,
                style: const TextStyle(
                  fontFamily: 'monospace',
                  fontSize: 16,
                ),
              ),
            ),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              onPressed: _testFFI,
              icon: const Icon(Icons.play_arrow),
              label: const Text('ทดสอบ FFI'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Workshop: Complete Platform Integration

```dart
// lib/workshop/platform_integration_page.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class PlatformIntegrationPage extends StatefulWidget {
  const PlatformIntegrationPage({super.key});

  @override
  State<PlatformIntegrationPage> createState() =>
      _PlatformIntegrationPageState();
}

class _PlatformIntegrationPageState extends State<PlatformIntegrationPage> {
  static const MethodChannel _channel =
      MethodChannel('com.example.myapp/platform');

  final Map<String, dynamic> _results = {};
  bool _isLoading = false;

  Future<void> _runTest(String name, Future<dynamic> Function() test) async {
    setState(() => _isLoading = true);
    try {
      final result = await test();
      setState(() => _results[name] = result ?? 'success');
    } catch (e) {
      setState(() => _results[name] = 'Error: $e');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Platform Integration'),
        actions: [
          if (_isLoading)
            const Padding(
              padding: EdgeInsets.all(16),
              child: SizedBox(
                width: 20,
                height: 20,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  color: Colors.white,
                ),
              ),
            ),
        ],
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _TestCard(
            title: 'Platform Version',
            result: _results['version'],
            onTest: () => _runTest('version', () async {
              return await _channel.invokeMethod<String>('getPlatformVersion');
            }),
          ),
          _TestCard(
            title: 'Device Info',
            result: _results['deviceInfo'],
            onTest: () => _runTest('deviceInfo', () async {
              final result = await _channel
                  .invokeMethod<Map<dynamic, dynamic>>('getDeviceInfo');
              return result?.entries
                  .map((e) => '${e.key}: ${e.value}')
                  .join('\n');
            }),
          ),
          _TestCard(
            title: 'Battery Level',
            result: _results['battery'],
            onTest: () => _runTest('battery', () async {
              final level =
                  await _channel.invokeMethod<int>('getBatteryLevel');
              return '$level%';
            }),
          ),
          _TestCard(
            title: 'Share Text',
            result: _results['share'],
            onTest: () => _runTest('share', () async {
              await _channel.invokeMethod('shareText', {
                'text': 'ทดสอบ Platform Channel จาก Flutter!'
              });
              return 'opened share dialog';
            }),
          ),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: () async {
              for (final test in ['version', 'deviceInfo', 'battery']) {
                // Run all tests
              }
            },
            child: const Text('รันทดสอบทั้งหมด'),
          ),
        ],
      ),
    );
  }
}

class _TestCard extends StatelessWidget {
  const _TestCard({
    required this.title,
    required this.result,
    required this.onTest,
  });

  final String title;
  final dynamic result;
  final VoidCallback onTest;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  title,
                  style: const TextStyle(
                    fontSize: 16,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                ElevatedButton(
                  onPressed: onTest,
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(
                        horizontal: 12, vertical: 8),
                  ),
                  child: const Text('ทดสอบ'),
                ),
              ],
            ),
            if (result != null) ...[
              const SizedBox(height: 8),
              Container(
                padding: const EdgeInsets.all(8),
                width: double.infinity,
                decoration: BoxDecoration(
                  color: Colors.grey[100],
                  borderRadius: BorderRadius.circular(4),
                ),
                child: Text(
                  '$result',
                  style: const TextStyle(
                    fontFamily: 'monospace',
                    fontSize: 13,
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **MethodChannel**: การเรียก Method บน Native Platform
- **EventChannel**: การรับ Stream Events จาก Native
- **BasicMessageChannel**: การรับส่งข้อความง่ายๆ
- **Android Kotlin/Java**: การเขียน Native Code ฝั่ง Android
- **iOS Swift/Obj-C**: การเขียน Native Code ฝั่ง iOS
- **Error Handling**: การจัดการ PlatformException
- **Platform UI**: การแสดง UI ตาม Platform
- **FFI**: การเรียก C/C++ Functions ตรงๆ

## แบบฝึกหัด

1. สร้าง Plugin สำหรับอ่านข้อมูล Contacts
2. Implement NFC Reader ด้วย Platform Channel
3. สร้าง Background Service สำหรับ Android
4. Implement Touch ID/Face ID Authentication
5. สร้าง Widget เพิ่ม Home Screen

---

[⬅️ Part 28](part_28.md) | [Part 30 ➡️](part_30.md)

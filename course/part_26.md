# Part 26: Push Notifications
## ขั้นตอนที่ 251-260

---

## สารบัญ
1. [firebase_messaging Setup](#firebase_messaging-setup)
2. [FCM Setup Android](#fcm-setup-android)
3. [FCM Setup iOS](#fcm-setup-ios)
4. [Foreground Notifications](#foreground-notifications)
5. [Background Notifications](#background-notifications)
6. [Terminated State Notifications](#terminated-state-notifications)
7. [Local Notifications](#local-notifications)
8. [Notification Channels](#notification-channels)
9. [Deep Links from Notifications](#deep-links-from-notifications)
10. [Notification Handling](#notification-handling)

---

## ขั้นตอนที่ 251: firebase_messaging Setup

Firebase Cloud Messaging (FCM) ให้บริการส่ง Push Notifications ไปยัง Mobile Devices

```yaml
# pubspec.yaml
dependencies:
  firebase_messaging: ^14.7.10
  flutter_local_notifications: ^17.0.0
  go_router: ^12.1.3  # สำหรับ Deep Links
```

```dart
// lib/services/notification_service.dart
import 'dart:convert';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:flutter/material.dart';

// Handler สำหรับ Background Messages (ต้องเป็น top-level function)
@pragma('vm:entry-point')
Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  // Initialize Firebase ถ้ายังไม่ได้ทำ
  // await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  debugPrint('Background message: ${message.messageId}');
  // บันทึก notification ลง local หรือทำ background work
}

class NotificationService {
  static final FirebaseMessaging _messaging = FirebaseMessaging.instance;
  static final FlutterLocalNotificationsPlugin _localNotifications =
      FlutterLocalNotificationsPlugin();

  // Callback สำหรับเมื่อผู้ใช้แตะ Notification
  static Function(Map<String, dynamic>)? onNotificationTap;

  static Future<void> initialize() async {
    // Setup background handler
    FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);

    // ขอ Permission
    await _requestPermission();

    // Setup Local Notifications
    await _setupLocalNotifications();

    // Setup FCM Handlers
    _setupForegroundHandler();
    _setupMessageOpenedHandler();

    // ดึง Token
    await getToken();
  }

  static Future<void> _requestPermission() async {
    final settings = await _messaging.requestPermission(
      alert: true,
      announcement: false,
      badge: true,
      carPlay: false,
      criticalAlert: false,
      provisional: false,
      sound: true,
    );

    debugPrint('Permission status: ${settings.authorizationStatus}');
  }

  static Future<String?> getToken() async {
    final token = await _messaging.getToken();
    debugPrint('FCM Token: $token');

    // บันทึก Token ใน Firestore
    // await FirebaseFirestore.instance
    //     .collection('users')
    //     .doc(FirebaseAuth.instance.currentUser?.uid)
    //     .update({'fcmToken': token});

    // Subscribe to token refreshes
    _messaging.onTokenRefresh.listen((newToken) {
      debugPrint('Token refreshed: $newToken');
      // อัพเดต Token ใหม่ใน Backend
    });

    return token;
  }

  static Future<void> _setupLocalNotifications() async {
    const androidSettings =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings(
      requestAlertPermission: true,
      requestBadgePermission: true,
      requestSoundPermission: true,
    );

    const initSettings = InitializationSettings(
      android: androidSettings,
      iOS: iosSettings,
    );

    await _localNotifications.initialize(
      initSettings,
      onDidReceiveNotificationResponse: (response) {
        // Handle tap on local notification
        if (response.payload != null) {
          final data = jsonDecode(response.payload!);
          onNotificationTap?.call(data as Map<String, dynamic>);
        }
      },
    );

    // Create Notification Channels for Android
    await _createNotificationChannels();
  }

  static Future<void> _createNotificationChannels() async {
    const channels = [
      AndroidNotificationChannel(
        'high_importance_channel',
        'การแจ้งเตือนสำคัญ',
        description: 'ช่องสำหรับการแจ้งเตือนที่สำคัญ',
        importance: Importance.high,
        playSound: true,
        enableVibration: true,
      ),
      AndroidNotificationChannel(
        'messages_channel',
        'ข้อความ',
        description: 'ช่องสำหรับการแจ้งเตือนข้อความ',
        importance: Importance.defaultImportance,
        playSound: true,
      ),
      AndroidNotificationChannel(
        'promotions_channel',
        'โปรโมชั่น',
        description: 'ช่องสำหรับการแจ้งเตือนโปรโมชั่น',
        importance: Importance.low,
        playSound: false,
      ),
    ];

    for (final channel in channels) {
      await _localNotifications
          .resolvePlatformSpecificImplementation<
              AndroidFlutterLocalNotificationsPlugin>()
          ?.createNotificationChannel(channel);
    }
  }

  static void _setupForegroundHandler() {
    FirebaseMessaging.onMessage.listen((RemoteMessage message) {
      debugPrint('Foreground message: ${message.notification?.title}');
      // แสดง Local Notification เมื่อแอปอยู่ foreground
      _showLocalNotification(message);
    });
  }

  static void _setupMessageOpenedHandler() {
    // แอปเปิดจาก Background
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
      debugPrint('Opened from background: ${message.data}');
      onNotificationTap?.call(message.data);
    });

    // แอปเปิดจาก Terminated State
    _messaging.getInitialMessage().then((message) {
      if (message != null) {
        debugPrint('Opened from terminated: ${message.data}');
        onNotificationTap?.call(message.data);
      }
    });
  }

  static Future<void> _showLocalNotification(RemoteMessage message) async {
    final notification = message.notification;
    if (notification == null) return;

    final channelId = message.data['channelId'] ?? 'high_importance_channel';

    await _localNotifications.show(
      notification.hashCode,
      notification.title,
      notification.body,
      NotificationDetails(
        android: AndroidNotificationDetails(
          channelId,
          channelId == 'high_importance_channel'
              ? 'การแจ้งเตือนสำคัญ'
              : 'ข้อความ',
          importance: Importance.high,
          priority: Priority.high,
          showWhen: true,
          styleInformation: BigTextStyleInformation(notification.body ?? ''),
        ),
        iOS: const DarwinNotificationDetails(
          presentAlert: true,
          presentBadge: true,
          presentSound: true,
        ),
      ),
      payload: jsonEncode(message.data),
    );
  }

  // Subscribe/Unsubscribe Topics
  static Future<void> subscribeToTopic(String topic) async {
    await _messaging.subscribeToTopic(topic);
    debugPrint('Subscribed to topic: $topic');
  }

  static Future<void> unsubscribeFromTopic(String topic) async {
    await _messaging.unsubscribeFromTopic(topic);
    debugPrint('Unsubscribed from topic: $topic');
  }
}
```

---

## ขั้นตอนที่ 252: FCM Setup Android

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.VIBRATE" />

    <application
        android:label="my_app"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher">
        
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:launchMode="singleTop"
            android:theme="@style/LaunchTheme">
            
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
            
            <!-- Deep Link Intent Filter -->
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="myapp" android:host="open" />
            </intent-filter>
        </activity>
        
        <!-- FCM Service -->
        <service
            android:name="com.google.firebase.messaging.FirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        
        <!-- Notification Icon -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_icon"
            android:resource="@drawable/ic_notification" />
        
        <!-- Notification Color -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_color"
            android:resource="@color/notification_color" />
        
        <!-- Default Channel -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_channel_id"
            android:value="high_importance_channel" />
        
    </application>
</manifest>
```

```kotlin
// android/app/src/main/kotlin/com/example/myapp/MainActivity.kt
package com.example.myapp

import io.flutter.embedding.android.FlutterActivity

class MainActivity: FlutterActivity() {
    // ไม่ต้องเพิ่มโค้ดอะไร FlutterFire จัดการให้
}
```

---

## ขั้นตอนที่ 253: FCM Setup iOS

```xml
<!-- ios/Runner/Info.plist - เพิ่ม -->
<key>UIBackgroundModes</key>
<array>
    <string>fetch</string>
    <string>remote-notification</string>
</array>

<!-- For Local Notifications -->
<key>NSUserNotificationUsageDescription</key>
<string>แอปนี้ต้องการส่งการแจ้งเตือนเพื่อแจ้งข้อมูลสำคัญ</string>
```

```swift
// ios/Runner/AppDelegate.swift
import UIKit
import Flutter
import FirebaseCore
import FirebaseMessaging
import UserNotifications

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        FirebaseApp.configure()
        
        // Set Messaging Delegate
        Messaging.messaging().delegate = self
        
        // Request Permission
        UNUserNotificationCenter.current().delegate = self
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        UNUserNotificationCenter.current().requestAuthorization(
            options: authOptions,
            completionHandler: { _, _ in }
        )
        application.registerForRemoteNotifications()
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}

extension AppDelegate: MessagingDelegate {
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        print("FCM Token: \(fcmToken ?? "nil")")
        // ส่ง Token ไปยัง Server
    }
}

extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        // แสดง Notification ขณะแอปอยู่ Foreground
        completionHandler([.alert, .badge, .sound])
    }
}
```

---

## ขั้นตอนที่ 254: Foreground Notifications

```dart
// lib/pages/notification_demo_page.dart
import 'package:flutter/material.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import '../services/notification_service.dart';

class NotificationDemoPage extends StatefulWidget {
  const NotificationDemoPage({super.key});

  @override
  State<NotificationDemoPage> createState() => _NotificationDemoPageState();
}

class _NotificationDemoPageState extends State<NotificationDemoPage> {
  final List<RemoteMessage> _messages = [];
  String? _fcmToken;

  @override
  void initState() {
    super.initState();
    _initialize();
  }

  Future<void> _initialize() async {
    // ดึง Token
    final token = await NotificationService.getToken();
    setState(() => _fcmToken = token);

    // ฟัง Foreground Messages
    FirebaseMessaging.onMessage.listen((message) {
      setState(() => _messages.insert(0, message));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Push Notifications'),
        backgroundColor: Colors.deepPurple,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          // Token Display
          Container(
            margin: const EdgeInsets.all(16),
            padding: const EdgeInsets.all(12),
            decoration: BoxDecoration(
              color: Colors.grey[100],
              borderRadius: BorderRadius.circular(8),
              border: Border.all(color: Colors.grey.shade300),
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'FCM Token:',
                  style: TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 4),
                Text(
                  _fcmToken ?? 'กำลังโหลด...',
                  style: const TextStyle(fontSize: 11, fontFamily: 'monospace'),
                  maxLines: 3,
                  overflow: TextOverflow.ellipsis,
                ),
                const SizedBox(height: 8),
                ElevatedButton.icon(
                  onPressed: _fcmToken != null
                      ? () {
                          // Copy to clipboard
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('Copied!')),
                          );
                        }
                      : null,
                  icon: const Icon(Icons.copy, size: 16),
                  label: const Text('คัดลอก Token'),
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(
                        horizontal: 12, vertical: 8),
                  ),
                ),
              ],
            ),
          ),

          // Topic Subscription
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: Card(
              child: Column(
                children: [
                  ListTile(
                    title: const Text('หัวข้อ: ข่าวสาร'),
                    trailing: Switch(
                      value: true,
                      onChanged: (value) {
                        if (value) {
                          NotificationService.subscribeToTopic('news');
                        } else {
                          NotificationService.unsubscribeFromTopic('news');
                        }
                      },
                    ),
                  ),
                  ListTile(
                    title: const Text('หัวข้อ: โปรโมชั่น'),
                    trailing: Switch(
                      value: false,
                      onChanged: (value) {
                        if (value) {
                          NotificationService.subscribeToTopic('promotions');
                        } else {
                          NotificationService.unsubscribeFromTopic('promotions');
                        }
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),

          const Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'การแจ้งเตือนที่ได้รับ',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ),

          // Messages List
          Expanded(
            child: _messages.isEmpty
                ? const Center(child: Text('ยังไม่มีการแจ้งเตือน'))
                : ListView.builder(
                    padding: const EdgeInsets.symmetric(horizontal: 16),
                    itemCount: _messages.length,
                    itemBuilder: (context, index) {
                      final message = _messages[index];
                      return _MessageCard(message: message);
                    },
                  ),
          ),
        ],
      ),
    );
  }
}

class _MessageCard extends StatelessWidget {
  const _MessageCard({required this.message});
  final RemoteMessage message;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: ExpansionTile(
        leading: const Icon(Icons.notifications, color: Colors.deepPurple),
        title: Text(
          message.notification?.title ?? 'ไม่มีหัวข้อ',
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        subtitle: Text(message.notification?.body ?? ''),
        children: [
          if (message.data.isNotEmpty)
            Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('Data:',
                      style: TextStyle(fontWeight: FontWeight.bold)),
                  ...message.data.entries.map(
                    (entry) => Text('  ${entry.key}: ${entry.value}'),
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

## ขั้นตอนที่ 255: Background Notifications

```dart
// lib/main.dart - ต้องตั้งค่า Background Handler ก่อน runApp
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'firebase_options.dart';
import 'services/notification_service.dart';

// Top-level function สำหรับ Background Handler
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  debugPrint('Background message received:');
  debugPrint('  Title: ${message.notification?.title}');
  debugPrint('  Body: ${message.notification?.body}');
  debugPrint('  Data: ${message.data}');

  // ทำงาน Background เช่น:
  // - บันทึกใน Local Database
  // - Update Badge Count
  // - Download Content
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  // ลงทะเบียน Background Handler
  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

  // Initialize Notification Service
  await NotificationService.initialize();

  runApp(const MyApp());
}
```

---

## ขั้นตอนที่ 256: Terminated State Notifications

```dart
// lib/main.dart - Handle Terminated State
import 'package:flutter/material.dart';
import 'package:firebase_messaging/firebase_messaging.dart';

class NotificationRouter {
  static String? _pendingRoute;

  static void handleInitialMessage() {
    FirebaseMessaging.instance.getInitialMessage().then((message) {
      if (message != null) {
        _pendingRoute = _getRouteFromData(message.data);
      }
    });
  }

  static String? consumePendingRoute() {
    final route = _pendingRoute;
    _pendingRoute = null;
    return route;
  }

  static String? _getRouteFromData(Map<String, dynamic> data) {
    final screen = data['screen'] as String?;
    final id = data['id'] as String?;

    switch (screen) {
      case 'post':
        return '/posts/$id';
      case 'chat':
        return '/chat/$id';
      case 'profile':
        return '/profile/$id';
      default:
        return null;
    }
  }
}

// ใน main()
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();

  // Handle terminated state
  NotificationRouter.handleInitialMessage();

  runApp(const MyApp());
}

// ใน Root Widget
class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();

    // ตรวจสอบ Pending Route หลัง build
    WidgetsBinding.instance.addPostFrameCallback((_) {
      final route = NotificationRouter.consumePendingRoute();
      if (route != null && mounted) {
        // Navigate to route
        Navigator.pushNamed(context, route);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Notification App',
      routes: {
        '/posts': (ctx) => const PostsPage(),
        '/chat': (ctx) => const ChatPage(),
      },
      home: const HomePage(),
    );
  }
}

class PostsPage extends StatelessWidget {
  const PostsPage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: Center(child: Text('Posts')));
}

class ChatPage extends StatelessWidget {
  const ChatPage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: Center(child: Text('Chat')));
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});
  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: Center(child: Text('Home')));
}
```

---

## ขั้นตอนที่ 257: Local Notifications

```dart
// lib/services/local_notification_service.dart
import 'dart:typed_data';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/timezone.dart' as tz;
import 'package:timezone/data/latest.dart' as tz;

class LocalNotificationService {
  static final FlutterLocalNotificationsPlugin _plugin =
      FlutterLocalNotificationsPlugin();

  static Future<void> initialize() async {
    tz.initializeTimeZones();

    const androidSettings =
        AndroidInitializationSettings('@mipmap/ic_launcher');
    const iosSettings = DarwinInitializationSettings();
    const settings =
        InitializationSettings(android: androidSettings, iOS: iosSettings);

    await _plugin.initialize(settings);
  }

  // แสดง Notification ทันที
  static Future<void> showNotification({
    required int id,
    required String title,
    required String body,
    String? payload,
    String channelId = 'default_channel',
  }) async {
    final androidDetails = AndroidNotificationDetails(
      channelId,
      'Default Channel',
      importance: Importance.high,
      priority: Priority.high,
      ticker: 'ticker',
      styleInformation: BigTextStyleInformation(body),
    );

    const iosDetails = DarwinNotificationDetails(
      presentAlert: true,
      presentBadge: true,
      presentSound: true,
    );

    final details = NotificationDetails(
      android: androidDetails,
      iOS: iosDetails,
    );

    await _plugin.show(id, title, body, details, payload: payload);
  }

  // Schedule Notification
  static Future<void> scheduleNotification({
    required int id,
    required String title,
    required String body,
    required DateTime scheduledDate,
    String? payload,
  }) async {
    final details = NotificationDetails(
      android: AndroidNotificationDetails(
        'scheduled_channel',
        'Scheduled Notifications',
        importance: Importance.high,
      ),
      iOS: const DarwinNotificationDetails(),
    );

    await _plugin.zonedSchedule(
      id,
      title,
      body,
      tz.TZDateTime.from(scheduledDate, tz.local),
      details,
      payload: payload,
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
    );
  }

  // Periodic Notification
  static Future<void> showPeriodicNotification({
    required int id,
    required String title,
    required String body,
    required RepeatInterval interval,
  }) async {
    const details = NotificationDetails(
      android: AndroidNotificationDetails(
        'periodic_channel',
        'Periodic Notifications',
        importance: Importance.low,
      ),
      iOS: DarwinNotificationDetails(),
    );

    await _plugin.periodicallyShow(
      id,
      title,
      body,
      interval,
      details,
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
    );
  }

  // Notification with Action Buttons
  static Future<void> showNotificationWithActions({
    required int id,
    required String title,
    required String body,
  }) async {
    const androidActions = [
      AndroidNotificationAction('accept', 'รับ',
          showsUserInterface: false),
      AndroidNotificationAction('decline', 'ปฏิเสธ',
          showsUserInterface: false),
    ];

    final androidDetails = AndroidNotificationDetails(
      'action_channel',
      'Action Notifications',
      importance: Importance.high,
      actions: androidActions,
    );

    await _plugin.show(
      id,
      title,
      body,
      NotificationDetails(android: androidDetails),
    );
  }

  // Big Picture Notification
  static Future<void> showBigPictureNotification({
    required int id,
    required String title,
    required String body,
    required String imageUrl,
  }) async {
    final bigPictureStyleInformation = BigPictureStyleInformation(
      DrawableResourceAndroidBitmap('@drawable/notification_image'),
      contentTitle: title,
      summaryText: body,
    );

    final androidDetails = AndroidNotificationDetails(
      'picture_channel',
      'Picture Notifications',
      styleInformation: bigPictureStyleInformation,
    );

    await _plugin.show(
      id,
      title,
      body,
      NotificationDetails(android: androidDetails),
    );
  }

  // Progress Notification
  static Future<void> showProgressNotification({
    required int id,
    required String title,
    required int progress,
    int maxProgress = 100,
    bool indeterminate = false,
  }) async {
    final androidDetails = AndroidNotificationDetails(
      'progress_channel',
      'Progress Notifications',
      channelShowBadge: false,
      importance: Importance.low,
      priority: Priority.low,
      onlyAlertOnce: true,
      showProgress: true,
      maxProgress: maxProgress,
      progress: progress,
      indeterminate: indeterminate,
    );

    await _plugin.show(
      id,
      title,
      '$progress/$maxProgress',
      NotificationDetails(android: androidDetails),
    );
  }

  // ยกเลิก Notification
  static Future<void> cancelNotification(int id) async {
    await _plugin.cancel(id);
  }

  // ยกเลิกทุก Notification
  static Future<void> cancelAllNotifications() async {
    await _plugin.cancelAll();
  }

  // ดึง Pending Notifications
  static Future<List<PendingNotificationRequest>>
      getPendingNotifications() async {
    return await _plugin.pendingNotificationRequests();
  }
}
```

---

## ขั้นตอนที่ 258: Notification Channels

```dart
// lib/config/notification_channels.dart
class NotificationChannels {
  // Channel IDs
  static const String highImportance = 'high_importance_channel';
  static const String messages = 'messages_channel';
  static const String promotions = 'promotions_channel';
  static const String reminders = 'reminders_channel';
  static const String system = 'system_channel';

  static const Map<String, NotificationChannelConfig> channels = {
    highImportance: NotificationChannelConfig(
      id: highImportance,
      name: 'การแจ้งเตือนสำคัญ',
      description: 'สำหรับการแจ้งเตือนที่สำคัญและเร่งด่วน',
      importance: 'high',
      sound: true,
      vibration: true,
      lights: true,
    ),
    messages: NotificationChannelConfig(
      id: messages,
      name: 'ข้อความ',
      description: 'การแจ้งเตือนข้อความและแชท',
      importance: 'default',
      sound: true,
      vibration: true,
      lights: false,
    ),
    promotions: NotificationChannelConfig(
      id: promotions,
      name: 'โปรโมชั่น',
      description: 'ข้อเสนอพิเศษและโปรโมชั่น',
      importance: 'low',
      sound: false,
      vibration: false,
      lights: false,
    ),
  };
}

class NotificationChannelConfig {
  const NotificationChannelConfig({
    required this.id,
    required this.name,
    required this.description,
    required this.importance,
    required this.sound,
    required this.vibration,
    required this.lights,
  });

  final String id;
  final String name;
  final String description;
  final String importance;
  final bool sound;
  final bool vibration;
  final bool lights;
}

// lib/pages/notification_settings_page.dart
import 'package:flutter/material.dart';

class NotificationSettingsPage extends StatefulWidget {
  const NotificationSettingsPage({super.key});

  @override
  State<NotificationSettingsPage> createState() =>
      _NotificationSettingsPageState();
}

class _NotificationSettingsPageState extends State<NotificationSettingsPage> {
  final Map<String, bool> _channelEnabled = {
    NotificationChannels.highImportance: true,
    NotificationChannels.messages: true,
    NotificationChannels.promotions: false,
    NotificationChannels.reminders: true,
  };

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('การตั้งค่าการแจ้งเตือน')),
      body: ListView(
        children: [
          const Padding(
            padding: EdgeInsets.all(16),
            child: Text(
              'ประเภทการแจ้งเตือน',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
          ...NotificationChannels.channels.entries.map(
            (entry) => _ChannelToggleTile(
              config: entry.value,
              isEnabled: _channelEnabled[entry.key] ?? false,
              onChanged: (value) {
                setState(() => _channelEnabled[entry.key] = value);
                if (value) {
                  NotificationService.subscribeToTopic(entry.key);
                } else {
                  NotificationService.unsubscribeFromTopic(entry.key);
                }
              },
            ),
          ),
          const Divider(),
          ListTile(
            leading: const Icon(Icons.notification_add),
            title: const Text('ทดสอบการแจ้งเตือน'),
            onTap: () {
              LocalNotificationService.showNotification(
                id: DateTime.now().millisecondsSinceEpoch ~/ 1000,
                title: 'ทดสอบการแจ้งเตือน',
                body: 'นี่คือการแจ้งเตือนทดสอบ',
              );
            },
          ),
        ],
      ),
    );
  }
}

class _ChannelToggleTile extends StatelessWidget {
  const _ChannelToggleTile({
    required this.config,
    required this.isEnabled,
    required this.onChanged,
  });

  final NotificationChannelConfig config;
  final bool isEnabled;
  final Function(bool) onChanged;

  @override
  Widget build(BuildContext context) {
    return SwitchListTile(
      title: Text(config.name),
      subtitle: Text(config.description),
      secondary: Icon(
        isEnabled ? Icons.notifications_active : Icons.notifications_off,
        color: isEnabled ? Colors.blue : Colors.grey,
      ),
      value: isEnabled,
      onChanged: onChanged,
    );
  }
}
```

---

## ขั้นตอนที่ 259: Deep Links from Notifications

```dart
// lib/services/deep_link_service.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class DeepLinkService {
  static final GlobalKey<NavigatorState> navigatorKey =
      GlobalKey<NavigatorState>();

  static void handleNotificationData(Map<String, dynamic> data) {
    final screen = data['screen'] as String?;
    final id = data['id'] as String?;

    WidgetsBinding.instance.addPostFrameCallback((_) {
      final context = navigatorKey.currentContext;
      if (context == null) return;

      switch (screen) {
        case 'post':
          context.go('/posts/$id');
          break;
        case 'chat':
          context.go('/chat/$id');
          break;
        case 'profile':
          context.go('/profile/$id');
          break;
        case 'order':
          context.go('/orders/$id');
          break;
        default:
          context.go('/');
      }
    });
  }
}

// lib/router/app_router.dart
import 'package:go_router/go_router.dart';

class AppRouter {
  static final GoRouter router = GoRouter(
    navigatorKey: DeepLinkService.navigatorKey,
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const HomePage(),
      ),
      GoRoute(
        path: '/posts/:id',
        builder: (context, state) {
          final postId = state.pathParameters['id']!;
          return PostDetailPage(postId: postId);
        },
      ),
      GoRoute(
        path: '/chat/:id',
        builder: (context, state) {
          final chatId = state.pathParameters['id']!;
          return ChatRoomPage(chatId: chatId);
        },
      ),
      GoRoute(
        path: '/profile/:id',
        builder: (context, state) {
          final userId = state.pathParameters['id']!;
          return UserProfilePage(userId: userId);
        },
      ),
      GoRoute(
        path: '/orders/:id',
        builder: (context, state) {
          final orderId = state.pathParameters['id']!;
          return OrderDetailPage(orderId: orderId);
        },
      ),
    ],
  );
}

// Placeholder pages
class PostDetailPage extends StatelessWidget {
  const PostDetailPage({super.key, required this.postId});
  final String postId;
  @override
  Widget build(BuildContext context) =>
      Scaffold(appBar: AppBar(title: Text('Post $postId')));
}

class ChatRoomPage extends StatelessWidget {
  const ChatRoomPage({super.key, required this.chatId});
  final String chatId;
  @override
  Widget build(BuildContext context) =>
      Scaffold(appBar: AppBar(title: Text('Chat $chatId')));
}

class UserProfilePage extends StatelessWidget {
  const UserProfilePage({super.key, required this.userId});
  final String userId;
  @override
  Widget build(BuildContext context) =>
      Scaffold(appBar: AppBar(title: Text('User $userId')));
}

class OrderDetailPage extends StatelessWidget {
  const OrderDetailPage({super.key, required this.orderId});
  final String orderId;
  @override
  Widget build(BuildContext context) =>
      Scaffold(appBar: AppBar(title: Text('Order $orderId')));
}
```

---

## ขั้นตอนที่ 260: Notification Handling

```dart
// lib/main.dart - Complete Notification Setup
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'firebase_options.dart';
import 'services/notification_service.dart';
import 'services/local_notification_service.dart';
import 'services/deep_link_service.dart';

@pragma('vm:entry-point')
Future<void> _backgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  debugPrint('Background: ${message.notification?.title}');
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  FirebaseMessaging.onBackgroundMessage(_backgroundHandler);

  await NotificationService.initialize();
  await LocalNotificationService.initialize();

  // Handle notification taps
  NotificationService.onNotificationTap = (data) {
    DeepLinkService.handleNotificationData(data);
  };

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Notification App',
      routerConfig: AppRouter.router,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
    );
  }
}

// Notification Tester Widget
class NotificationTesterPage extends StatelessWidget {
  const NotificationTesterPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ทดสอบ Notifications')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          _TestButton(
            label: 'แสดง Notification ทันที',
            icon: Icons.notification_add,
            onTap: () => LocalNotificationService.showNotification(
              id: 1,
              title: 'ทดสอบ',
              body: 'นี่คือ Local Notification',
            ),
          ),
          _TestButton(
            label: 'Notification ใน 5 วินาที',
            icon: Icons.timer,
            onTap: () => LocalNotificationService.scheduleNotification(
              id: 2,
              title: 'แจ้งเตือนล่าช้า',
              body: 'ตั้งเวลาให้แสดงใน 5 วินาที',
              scheduledDate: DateTime.now().add(const Duration(seconds: 5)),
            ),
          ),
          _TestButton(
            label: 'Notification พร้อม Action',
            icon: Icons.touch_app,
            onTap: () => LocalNotificationService.showNotificationWithActions(
              id: 3,
              title: 'คุณมีคำเชิญใหม่',
              body: 'สมชายเชิญคุณเข้าร่วมกลุ่ม Flutter Dev',
            ),
          ),
          _TestButton(
            label: 'Progress Notification',
            icon: Icons.download,
            onTap: () async {
              for (var i = 0; i <= 100; i += 10) {
                await LocalNotificationService.showProgressNotification(
                  id: 4,
                  title: 'กำลังดาวน์โหลด...',
                  progress: i,
                );
                await Future.delayed(const Duration(milliseconds: 300));
              }
            },
          ),
          _TestButton(
            label: 'ยกเลิก Notification ทั้งหมด',
            icon: Icons.clear_all,
            color: Colors.red,
            onTap: () => LocalNotificationService.cancelAllNotifications(),
          ),
        ],
      ),
    );
  }
}

class _TestButton extends StatelessWidget {
  const _TestButton({
    required this.label,
    required this.icon,
    required this.onTap,
    this.color,
  });

  final String label;
  final IconData icon;
  final VoidCallback onTap;
  final Color? color;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.only(bottom: 8),
      child: ListTile(
        leading: Icon(icon, color: color ?? Colors.deepPurple),
        title: Text(label),
        trailing: const Icon(Icons.chevron_right),
        onTap: onTap,
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **FCM Setup**: การตั้งค่า Firebase Messaging บน Android และ iOS
- **Foreground**: การรับ Notification ขณะแอปอยู่หน้า
- **Background**: การรับ Notification ขณะแอปอยู่เบื้องหลัง
- **Terminated**: การรับ Notification เมื่อแอปปิดอยู่
- **Local Notifications**: การแสดง Notification จากภายใน
- **Notification Channels**: การจัดกลุ่ม Notification
- **Deep Links**: การ Navigate เมื่อแตะ Notification
- **Notification Handling**: การจัดการ Notification อย่างครบถ้วน

## แบบฝึกหัด

1. ส่ง Notification ไปยัง User เฉพาะคนจาก Cloud Functions
2. สร้าง Notification History Page
3. Implement DND (Do Not Disturb) Mode
4. สร้าง Notification Preferences ให้ผู้ใช้ปรับแต่งได้
5. ทดสอบ Deep Link จาก Notification

---

[⬅️ Part 25](part_25.md) | [Part 27 ➡️](part_27.md)

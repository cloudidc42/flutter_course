# Part 43: WebSocket & Real-time Apps
## ขั้นตอนที่ 421-430

---

## สารบัญ
1. [web_socket_channel Package](#ขั้นตอนที่-421-web_socket_channel)
2. [WebSocket Connection Management](#ขั้นตอนที่-422-connection-management)
3. [Real-time Chat App](#ขั้นตอนที่-423-real-time-chat-app)
4. [Reconnection Strategy](#ขั้นตอนที่-424-reconnection-strategy)
5. [Socket.io กับ Flutter](#ขั้นตอนที่-425-socketio-กับ-flutter)
6. [Collaborative Editing](#ขั้นตอนที่-426-collaborative-editing)
7. [Presence Indicators](#ขั้นตอนที่-427-presence-indicators)
8. [Real-time Notifications](#ขั้นตอนที่-428-real-time-notifications)
9. [Performance Optimization](#ขั้นตอนที่-429-performance-optimization)
10. [Workshop: Real-time Dashboard](#ขั้นตอนที่-430-workshop)

---

## ขั้นตอนที่ 421: web_socket_channel Package

### WebSocket คืออะไร?

WebSocket เป็น protocol ที่ช่วยให้ client และ server สื่อสารแบบ full-duplex (สองทาง) ผ่าน TCP connection เดียว เหมาะสำหรับ real-time applications

```
HTTP (Request-Response):
Client ──── Request ────> Server
Client <─── Response ─── Server

WebSocket (Bidirectional):
Client <══════════════════> Server
       ←── Message ────────
       ──── Message ───────>
       ←── Message ────────
```

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  web_socket_channel: ^2.4.0
  stream_channel: ^2.1.0
```

### การใช้งานพื้นฐาน

```dart
// lib/services/websocket_service.dart
import 'package:web_socket_channel/web_socket_channel.dart';
import 'package:web_socket_channel/status.dart' as status;

class WebSocketService {
  WebSocketChannel? _channel;
  final String _url;
  
  WebSocketService(this._url);
  
  // เชื่อมต่อ WebSocket
  Future<void> connect() async {
    final uri = Uri.parse(_url);
    _channel = WebSocketChannel.connect(uri);
    
    // รอให้ connection พร้อม
    await _channel!.ready;
    print('WebSocket connected to $_url');
  }
  
  // ส่ง message
  void send(String message) {
    _channel?.sink.add(message);
  }
  
  // ส่ง JSON
  void sendJson(Map<String, dynamic> data) {
    _channel?.sink.add(jsonEncode(data));
  }
  
  // รับ messages
  Stream<dynamic> get messages {
    return _channel?.stream ?? const Stream.empty();
  }
  
  // รับ typed messages
  Stream<Map<String, dynamic>> get jsonMessages {
    return messages.map((data) {
      if (data is String) {
        return jsonDecode(data) as Map<String, dynamic>;
      }
      return data as Map<String, dynamic>;
    });
  }
  
  // ปิด connection
  Future<void> disconnect() async {
    await _channel?.sink.close(status.goingAway);
    _channel = null;
    print('WebSocket disconnected');
  }
  
  bool get isConnected => _channel != null;
}
```

### WebSocket ใน Flutter Widget

```dart
// lib/widgets/websocket_listener.dart
class WebSocketListener extends StatefulWidget {
  final String url;
  final Widget Function(dynamic data) builder;
  final Widget Function(Object error)? errorBuilder;
  
  const WebSocketListener({
    super.key,
    required this.url,
    required this.builder,
    this.errorBuilder,
  });
  
  @override
  State<WebSocketListener> createState() => _WebSocketListenerState();
}

class _WebSocketListenerState extends State<WebSocketListener> {
  WebSocketChannel? _channel;
  
  @override
  void initState() {
    super.initState();
    _connect();
  }
  
  void _connect() {
    _channel = WebSocketChannel.connect(Uri.parse(widget.url));
  }
  
  @override
  Widget build(BuildContext context) {
    return StreamBuilder(
      stream: _channel?.stream,
      builder: (context, snapshot) {
        if (snapshot.hasError) {
          return widget.errorBuilder?.call(snapshot.error!) ??
              Text('Error: ${snapshot.error}');
        }
        
        if (!snapshot.hasData) {
          return const CircularProgressIndicator();
        }
        
        return widget.builder(snapshot.data);
      },
    );
  }
  
  @override
  void dispose() {
    _channel?.sink.close();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 422: Connection Management

### WebSocket Connection Manager

```dart
// lib/services/websocket_manager.dart
import 'dart:async';

enum WebSocketState {
  disconnected,
  connecting,
  connected,
  reconnecting,
  error,
}

class WebSocketManager {
  static final WebSocketManager _instance = WebSocketManager._internal();
  factory WebSocketManager() => _instance;
  WebSocketManager._internal();
  
  WebSocketChannel? _channel;
  Timer? _pingTimer;
  Timer? _reconnectTimer;
  
  final _stateController = StreamController<WebSocketState>.broadcast();
  final _messageController = StreamController<dynamic>.broadcast();
  
  WebSocketState _state = WebSocketState.disconnected;
  String? _url;
  Map<String, dynamic>? _headers;
  int _reconnectAttempts = 0;
  static const int _maxReconnectAttempts = 5;
  
  Stream<WebSocketState> get stateStream => _stateController.stream;
  Stream<dynamic> get messageStream => _messageController.stream;
  WebSocketState get state => _state;
  
  Future<void> connect(
    String url, {
    Map<String, dynamic>? headers,
  }) async {
    _url = url;
    _headers = headers;
    _reconnectAttempts = 0;
    await _doConnect();
  }
  
  Future<void> _doConnect() async {
    _setState(WebSocketState.connecting);
    
    try {
      _channel = WebSocketChannel.connect(
        Uri.parse(_url!),
        // headers: _headers, // depends on implementation
      );
      
      await _channel!.ready;
      _setState(WebSocketState.connected);
      _reconnectAttempts = 0;
      
      // Start ping/pong to keep connection alive
      _startPing();
      
      // Listen for messages
      _channel!.stream.listen(
        (data) => _messageController.add(data),
        onError: (error) {
          print('WebSocket error: $error');
          _handleDisconnect();
        },
        onDone: () {
          print('WebSocket connection closed');
          _handleDisconnect();
        },
      );
    } catch (e) {
      print('WebSocket connection failed: $e');
      _setState(WebSocketState.error);
      _scheduleReconnect();
    }
  }
  
  void _handleDisconnect() {
    _stopPing();
    _channel = null;
    _scheduleReconnect();
  }
  
  void _scheduleReconnect() {
    if (_reconnectAttempts >= _maxReconnectAttempts) {
      _setState(WebSocketState.error);
      print('Max reconnect attempts reached');
      return;
    }
    
    _setState(WebSocketState.reconnecting);
    _reconnectAttempts++;
    
    // Exponential backoff
    final delay = Duration(
      seconds: (2 ^ _reconnectAttempts).clamp(1, 30),
    );
    
    print('Reconnecting in ${delay.inSeconds}s (attempt $_reconnectAttempts)');
    
    _reconnectTimer = Timer(delay, _doConnect);
  }
  
  void _startPing() {
    _pingTimer = Timer.periodic(const Duration(seconds: 30), (_) {
      if (_state == WebSocketState.connected) {
        send(jsonEncode({'type': 'ping'}));
      }
    });
  }
  
  void _stopPing() {
    _pingTimer?.cancel();
    _pingTimer = null;
  }
  
  void _setState(WebSocketState newState) {
    _state = newState;
    _stateController.add(newState);
  }
  
  void send(dynamic data) {
    if (_state == WebSocketState.connected) {
      _channel?.sink.add(data);
    } else {
      print('Cannot send: WebSocket not connected (state: $_state)');
    }
  }
  
  Future<void> disconnect() async {
    _reconnectTimer?.cancel();
    _stopPing();
    await _channel?.sink.close();
    _channel = null;
    _setState(WebSocketState.disconnected);
  }
  
  void dispose() {
    disconnect();
    _stateController.close();
    _messageController.close();
  }
}
```

### Connection State Widget

```dart
// lib/widgets/connection_status_widget.dart
class ConnectionStatusWidget extends StatelessWidget {
  final WebSocketManager wsManager;
  final Widget child;
  
  const ConnectionStatusWidget({
    super.key,
    required this.wsManager,
    required this.child,
  });
  
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<WebSocketState>(
      stream: wsManager.stateStream,
      initialData: wsManager.state,
      builder: (context, snapshot) {
        final state = snapshot.data ?? WebSocketState.disconnected;
        
        return Column(
          children: [
            if (state != WebSocketState.connected)
              _buildStatusBanner(state),
            Expanded(child: child),
          ],
        );
      },
    );
  }
  
  Widget _buildStatusBanner(WebSocketState state) {
    final (color, message, icon) = switch (state) {
      WebSocketState.connecting => (Colors.orange, 'กำลังเชื่อมต่อ...', Icons.sync),
      WebSocketState.reconnecting => (Colors.orange, 'กำลังเชื่อมต่อใหม่...', Icons.refresh),
      WebSocketState.disconnected => (Colors.grey, 'ออฟไลน์', Icons.wifi_off),
      WebSocketState.error => (Colors.red, 'เชื่อมต่อล้มเหลว', Icons.error_outline),
      _ => (Colors.green, 'เชื่อมต่อแล้ว', Icons.wifi),
    };
    
    return AnimatedContainer(
      duration: const Duration(milliseconds: 300),
      color: color,
      padding: const EdgeInsets.symmetric(vertical: 8, horizontal: 16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(icon, color: Colors.white, size: 16),
          const SizedBox(width: 8),
          Text(message, style: const TextStyle(color: Colors.white)),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 423: Real-time Chat App

### Chat Message Model

```dart
// lib/features/chat/models/chat_message.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'chat_message.freezed.dart';
part 'chat_message.g.dart';

enum MessageType {
  text,
  image,
  file,
  system,
}

enum MessageStatus {
  sending,
  sent,
  delivered,
  read,
  failed,
}

@freezed
class ChatMessage with _$ChatMessage {
  const factory ChatMessage({
    required String id,
    required String roomId,
    required String senderId,
    required String content,
    required MessageType type,
    required DateTime createdAt,
    @Default(MessageStatus.sending) MessageStatus status,
    String? replyToId,
    List<String>? attachmentUrls,
    Map<String, List<String>>? reactions,
  }) = _ChatMessage;
  
  factory ChatMessage.fromJson(Map<String, dynamic> json) =>
      _$ChatMessageFromJson(json);
}

@freezed
class ChatRoom with _$ChatRoom {
  const factory ChatRoom({
    required String id,
    required String name,
    required List<String> participantIds,
    ChatMessage? lastMessage,
    @Default(0) int unreadCount,
    DateTime? updatedAt,
  }) = _ChatRoom;
  
  factory ChatRoom.fromJson(Map<String, dynamic> json) =>
      _$ChatRoomFromJson(json);
}
```

### Chat Service

```dart
// lib/features/chat/services/chat_service.dart
class ChatService {
  final WebSocketManager _wsManager;
  final ChatRepository _repository;
  
  final Map<String, StreamController<ChatMessage>> _roomControllers = {};
  
  ChatService(this._wsManager, this._repository) {
    _setupMessageHandling();
  }
  
  void _setupMessageHandling() {
    _wsManager.messageStream.listen((data) {
      final json = jsonDecode(data as String) as Map<String, dynamic>;
      final type = json['type'] as String;
      
      switch (type) {
        case 'message':
          _handleNewMessage(json['data'] as Map<String, dynamic>);
        case 'typing':
          _handleTyping(json['data'] as Map<String, dynamic>);
        case 'read_receipt':
          _handleReadReceipt(json['data'] as Map<String, dynamic>);
        case 'user_online':
          _handleUserOnline(json['data'] as Map<String, dynamic>);
      }
    });
  }
  
  void _handleNewMessage(Map<String, dynamic> data) {
    final message = ChatMessage.fromJson(data);
    final controller = _roomControllers[message.roomId];
    controller?.add(message);
  }
  
  Stream<ChatMessage> messagesStream(String roomId) {
    if (!_roomControllers.containsKey(roomId)) {
      _roomControllers[roomId] = StreamController<ChatMessage>.broadcast();
      
      // Join room
      _wsManager.send(jsonEncode({
        'type': 'join_room',
        'roomId': roomId,
      }));
    }
    return _roomControllers[roomId]!.stream;
  }
  
  Future<void> sendMessage({
    required String roomId,
    required String content,
    MessageType type = MessageType.text,
    String? replyToId,
  }) async {
    final tempId = 'temp_${DateTime.now().millisecondsSinceEpoch}';
    
    // Optimistic: เพิ่ม message ทันที
    final optimisticMessage = ChatMessage(
      id: tempId,
      roomId: roomId,
      senderId: 'me',
      content: content,
      type: type,
      createdAt: DateTime.now(),
      status: MessageStatus.sending,
      replyToId: replyToId,
    );
    
    _roomControllers[roomId]?.add(optimisticMessage);
    
    try {
      // ส่งผ่าน WebSocket
      _wsManager.send(jsonEncode({
        'type': 'send_message',
        'tempId': tempId,
        'roomId': roomId,
        'content': content,
        'messageType': type.name,
        if (replyToId != null) 'replyToId': replyToId,
      }));
    } catch (e) {
      // Update status เป็น failed
      final failedMessage = optimisticMessage.copyWith(
        status: MessageStatus.failed,
      );
      _roomControllers[roomId]?.add(failedMessage);
    }
  }
  
  void sendTypingIndicator(String roomId, bool isTyping) {
    _wsManager.send(jsonEncode({
      'type': 'typing',
      'roomId': roomId,
      'isTyping': isTyping,
    }));
  }
  
  void markAsRead(String roomId, String messageId) {
    _wsManager.send(jsonEncode({
      'type': 'read_receipt',
      'roomId': roomId,
      'messageId': messageId,
    }));
  }
  
  void _handleTyping(Map<String, dynamic> data) {
    // Handle typing indicator
  }
  
  void _handleReadReceipt(Map<String, dynamic> data) {
    // Handle read receipts
  }
  
  void _handleUserOnline(Map<String, dynamic> data) {
    // Handle user online status
  }
  
  void dispose() {
    for (final controller in _roomControllers.values) {
      controller.close();
    }
    _roomControllers.clear();
  }
}
```

### Chat Room Screen

```dart
// lib/features/chat/screens/chat_room_screen.dart
class ChatRoomScreen extends ConsumerStatefulWidget {
  final String roomId;
  final String roomName;
  
  const ChatRoomScreen({
    super.key,
    required this.roomId,
    required this.roomName,
  });
  
  @override
  ConsumerState<ChatRoomScreen> createState() => _ChatRoomScreenState();
}

class _ChatRoomScreenState extends ConsumerState<ChatRoomScreen> {
  final TextEditingController _messageController = TextEditingController();
  final ScrollController _scrollController = ScrollController();
  final List<ChatMessage> _messages = [];
  late StreamSubscription _messageSubscription;
  Timer? _typingTimer;
  bool _isTyping = false;
  
  @override
  void initState() {
    super.initState();
    _loadMessages();
    _subscribeToMessages();
  }
  
  Future<void> _loadMessages() async {
    final chatService = ref.read(chatServiceProvider);
    final messages = await chatService.getMessages(widget.roomId);
    setState(() {
      _messages.addAll(messages);
    });
    _scrollToBottom();
  }
  
  void _subscribeToMessages() {
    final chatService = ref.read(chatServiceProvider);
    _messageSubscription = chatService
        .messagesStream(widget.roomId)
        .listen((message) {
          setState(() {
            // Replace optimistic message or add new
            final index = _messages.indexWhere((m) => m.id == message.id);
            if (index >= 0) {
              _messages[index] = message;
            } else {
              _messages.add(message);
            }
          });
          _scrollToBottom();
        });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(widget.roomName),
            _buildTypingIndicator(),
          ],
        ),
        actions: [
          IconButton(
            onPressed: () => context.push('/chat/${widget.roomId}/info'),
            icon: const Icon(Icons.info_outline),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              controller: _scrollController,
              padding: const EdgeInsets.all(8),
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final message = _messages[index];
                final previousMessage = index > 0 ? _messages[index - 1] : null;
                
                return ChatBubble(
                  message: message,
                  showAvatar: previousMessage?.senderId != message.senderId,
                  isMe: message.senderId == ref.watch(currentUserProvider)?.id,
                );
              },
            ),
          ),
          _buildInputArea(),
        ],
      ),
    );
  }
  
  Widget _buildTypingIndicator() {
    return StreamBuilder<List<String>>(
      stream: ref.read(chatServiceProvider).typingUsersStream(widget.roomId),
      builder: (context, snapshot) {
        final users = snapshot.data ?? [];
        if (users.isEmpty) return const SizedBox.shrink();
        
        return Text(
          users.length == 1
              ? '${users.first} กำลังพิมพ์...'
              : '${users.length} คนกำลังพิมพ์...',
          style: const TextStyle(fontSize: 12, fontStyle: FontStyle.italic),
        );
      },
    );
  }
  
  Widget _buildInputArea() {
    return Container(
      padding: const EdgeInsets.all(8),
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surface,
        border: Border(
          top: BorderSide(
            color: Theme.of(context).dividerColor,
          ),
        ),
      ),
      child: SafeArea(
        child: Row(
          children: [
            IconButton(
              onPressed: _pickAttachment,
              icon: const Icon(Icons.attach_file),
            ),
            Expanded(
              child: TextField(
                controller: _messageController,
                decoration: const InputDecoration(
                  hintText: 'พิมพ์ข้อความ...',
                  border: InputBorder.none,
                  contentPadding: EdgeInsets.symmetric(horizontal: 16),
                ),
                maxLines: null,
                onChanged: _onTextChanged,
                textInputAction: TextInputAction.send,
                onSubmitted: (_) => _sendMessage(),
              ),
            ),
            IconButton(
              onPressed: _sendMessage,
              icon: const Icon(Icons.send),
              color: Theme.of(context).colorScheme.primary,
            ),
          ],
        ),
      ),
    );
  }
  
  void _onTextChanged(String text) {
    final chatService = ref.read(chatServiceProvider);
    
    if (!_isTyping) {
      _isTyping = true;
      chatService.sendTypingIndicator(widget.roomId, true);
    }
    
    _typingTimer?.cancel();
    _typingTimer = Timer(const Duration(seconds: 2), () {
      _isTyping = false;
      chatService.sendTypingIndicator(widget.roomId, false);
    });
  }
  
  void _sendMessage() {
    final content = _messageController.text.trim();
    if (content.isEmpty) return;
    
    _messageController.clear();
    _typingTimer?.cancel();
    _isTyping = false;
    
    ref.read(chatServiceProvider).sendMessage(
      roomId: widget.roomId,
      content: content,
    );
  }
  
  Future<void> _pickAttachment() async {
    // Pick image/file
  }
  
  void _scrollToBottom() {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (_scrollController.hasClients) {
        _scrollController.animateTo(
          _scrollController.position.maxScrollExtent,
          duration: const Duration(milliseconds: 300),
          curve: Curves.easeOut,
        );
      }
    });
  }
  
  @override
  void dispose() {
    _messageController.dispose();
    _scrollController.dispose();
    _messageSubscription.cancel();
    _typingTimer?.cancel();
    super.dispose();
  }
}
```

### Chat Bubble Widget

```dart
// lib/features/chat/widgets/chat_bubble.dart
class ChatBubble extends StatelessWidget {
  final ChatMessage message;
  final bool isMe;
  final bool showAvatar;
  
  const ChatBubble({
    super.key,
    required this.message,
    required this.isMe,
    this.showAvatar = true,
  });
  
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.only(
        top: showAvatar ? 8 : 2,
        bottom: 2,
      ),
      child: Row(
        mainAxisAlignment:
            isMe ? MainAxisAlignment.end : MainAxisAlignment.start,
        crossAxisAlignment: CrossAxisAlignment.end,
        children: [
          if (!isMe) ...[
            if (showAvatar)
              CircleAvatar(
                radius: 16,
                child: Text(message.senderId[0].toUpperCase()),
              )
            else
              const SizedBox(width: 32),
            const SizedBox(width: 8),
          ],
          Flexible(
            child: Container(
              constraints: BoxConstraints(
                maxWidth: MediaQuery.of(context).size.width * 0.75,
              ),
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
              decoration: BoxDecoration(
                color: isMe
                    ? Theme.of(context).colorScheme.primary
                    : Theme.of(context).colorScheme.secondaryContainer,
                borderRadius: BorderRadius.only(
                  topLeft: const Radius.circular(16),
                  topRight: const Radius.circular(16),
                  bottomLeft: Radius.circular(isMe ? 16 : 4),
                  bottomRight: Radius.circular(isMe ? 4 : 16),
                ),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  if (message.type == MessageType.text)
                    Text(
                      message.content,
                      style: TextStyle(
                        color: isMe
                            ? Colors.white
                            : Theme.of(context).colorScheme.onSecondaryContainer,
                      ),
                    ),
                  const SizedBox(height: 4),
                  Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Text(
                        _formatTime(message.createdAt),
                        style: TextStyle(
                          fontSize: 10,
                          color: (isMe ? Colors.white : Colors.black)
                              .withOpacity(0.6),
                        ),
                      ),
                      if (isMe) ...[
                        const SizedBox(width: 4),
                        _buildStatusIcon(),
                      ],
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
  
  Widget _buildStatusIcon() {
    final icon = switch (message.status) {
      MessageStatus.sending => Icons.access_time,
      MessageStatus.sent => Icons.check,
      MessageStatus.delivered => Icons.done_all,
      MessageStatus.read => Icons.done_all,
      MessageStatus.failed => Icons.error_outline,
    };
    
    final color = message.status == MessageStatus.read
        ? Colors.blue
        : Colors.white.withOpacity(0.7);
    
    return Icon(icon, size: 12, color: color);
  }
  
  String _formatTime(DateTime time) {
    return '${time.hour}:${time.minute.toString().padLeft(2, '0')}';
  }
}
```

---

## ขั้นตอนที่ 424: Reconnection Strategy

### Exponential Backoff Reconnection

```dart
// lib/services/reconnection_strategy.dart
class ReconnectionStrategy {
  final int maxAttempts;
  final Duration initialDelay;
  final Duration maxDelay;
  final double multiplier;
  
  int _attempts = 0;
  
  const ReconnectionStrategy({
    this.maxAttempts = 10,
    this.initialDelay = const Duration(seconds: 1),
    this.maxDelay = const Duration(minutes: 2),
    this.multiplier = 1.5,
  });
  
  bool get canRetry => _attempts < maxAttempts;
  int get attempts => _attempts;
  
  Duration get nextDelay {
    final delay = initialDelay * (multiplier * _attempts).ceil();
    return delay > maxDelay ? maxDelay : delay;
  }
  
  void recordAttempt() => _attempts++;
  void reset() => _attempts = 0;
}

class RobustWebSocketClient {
  final String url;
  final ReconnectionStrategy reconnectionStrategy;
  WebSocketChannel? _channel;
  StreamSubscription? _subscription;
  final _stateController = StreamController<WebSocketState>.broadcast();
  final _messageController = StreamController<dynamic>.broadcast();
  bool _shouldReconnect = true;
  
  RobustWebSocketClient({
    required this.url,
    ReconnectionStrategy? strategy,
  }) : reconnectionStrategy = strategy ?? const ReconnectionStrategy();
  
  Stream<WebSocketState> get stateStream => _stateController.stream;
  Stream<dynamic> get messageStream => _messageController.stream;
  
  Future<void> connect() async {
    _shouldReconnect = true;
    await _connect();
  }
  
  Future<void> _connect() async {
    if (!_shouldReconnect) return;
    
    _stateController.add(WebSocketState.connecting);
    
    try {
      _channel = WebSocketChannel.connect(Uri.parse(url));
      await _channel!.ready;
      
      reconnectionStrategy.reset();
      _stateController.add(WebSocketState.connected);
      
      _subscription = _channel!.stream.listen(
        (data) => _messageController.add(data),
        onError: (_) => _handleDisconnect(),
        onDone: _handleDisconnect,
      );
    } catch (e) {
      await _handleDisconnect();
    }
  }
  
  Future<void> _handleDisconnect() async {
    _subscription?.cancel();
    _channel = null;
    
    if (!_shouldReconnect) return;
    
    if (!reconnectionStrategy.canRetry) {
      _stateController.add(WebSocketState.error);
      return;
    }
    
    reconnectionStrategy.recordAttempt();
    _stateController.add(WebSocketState.reconnecting);
    
    print('Reconnecting in ${reconnectionStrategy.nextDelay.inSeconds}s...');
    await Future.delayed(reconnectionStrategy.nextDelay);
    
    await _connect();
  }
  
  void send(dynamic data) {
    _channel?.sink.add(data);
  }
  
  Future<void> disconnect() async {
    _shouldReconnect = false;
    await _channel?.sink.close();
    _stateController.add(WebSocketState.disconnected);
  }
  
  void dispose() {
    disconnect();
    _stateController.close();
    _messageController.close();
  }
}
```

---

## ขั้นตอนที่ 425: Socket.io กับ Flutter

### socket_io_client Package

```yaml
dependencies:
  socket_io_client: ^2.0.3
```

```dart
// lib/services/socket_io_service.dart
import 'package:socket_io_client/socket_io_client.dart' as IO;

class SocketIOService {
  late IO.Socket _socket;
  final String _serverUrl;
  
  SocketIOService(this._serverUrl);
  
  void connect({Map<String, dynamic>? auth}) {
    _socket = IO.io(
      _serverUrl,
      IO.OptionBuilder()
          .setTransports(['websocket', 'polling'])
          .setAuth(auth ?? {})
          .enableAutoConnect()
          .enableReconnection()
          .setReconnectionAttempts(5)
          .setReconnectionDelay(1000)
          .build(),
    );
    
    _setupListeners();
    _socket.connect();
  }
  
  void _setupListeners() {
    _socket.onConnect((_) {
      print('Socket.IO connected: ${_socket.id}');
    });
    
    _socket.onDisconnect((_) {
      print('Socket.IO disconnected');
    });
    
    _socket.onConnectError((error) {
      print('Socket.IO connection error: $error');
    });
    
    _socket.onReconnect((attempt) {
      print('Socket.IO reconnected after $attempt attempts');
    });
  }
  
  // Emit event
  void emit(String event, dynamic data) {
    _socket.emit(event, data);
  }
  
  // Emit with acknowledgement
  void emitWithAck(String event, dynamic data, Function(dynamic) callback) {
    _socket.emitWithAck(event, data, ack: callback);
  }
  
  // Listen to event
  void on(String event, Function(dynamic) handler) {
    _socket.on(event, handler);
  }
  
  // Remove listener
  void off(String event) {
    _socket.off(event);
  }
  
  // Join room
  void joinRoom(String room) {
    _socket.emit('join', room);
  }
  
  // Leave room
  void leaveRoom(String room) {
    _socket.emit('leave', room);
  }
  
  bool get isConnected => _socket.connected;
  
  void disconnect() {
    _socket.disconnect();
  }
  
  void dispose() {
    _socket.dispose();
  }
}

// การใช้งาน
class ChatWithSocketIO {
  final SocketIOService _socketService;
  
  ChatWithSocketIO(this._socketService) {
    // Setup handlers
    _socketService.on('chat:message', (data) {
      print('New message: $data');
    });
    
    _socketService.on('user:joined', (data) {
      print('User joined: $data');
    });
  }
  
  void sendMessage(String roomId, String message) {
    _socketService.emitWithAck(
      'chat:send',
      {'roomId': roomId, 'message': message},
      (response) {
        if (response['status'] == 'ok') {
          print('Message sent successfully');
        }
      },
    );
  }
}
```

---

## ขั้นตอนที่ 426: Collaborative Editing

### Operational Transformation (OT) แบบง่าย

```dart
// lib/features/collab/models/operation.dart
abstract class TextOperation {
  int get position;
  void apply(StringBuffer buffer);
  TextOperation inverse();
}

class InsertOperation extends TextOperation {
  @override
  final int position;
  final String text;
  
  InsertOperation({required this.position, required this.text});
  
  @override
  void apply(StringBuffer buffer) {
    buffer.insert(position, text);
  }
  
  @override
  TextOperation inverse() {
    return DeleteOperation(
      position: position,
      length: text.length,
    );
  }
  
  Map<String, dynamic> toJson() => {
    'type': 'insert',
    'position': position,
    'text': text,
  };
}

class DeleteOperation extends TextOperation {
  @override
  final int position;
  final int length;
  
  DeleteOperation({required this.position, required this.length});
  
  @override
  void apply(StringBuffer buffer) {
    buffer.delete(position, position + length);
  }
  
  @override
  TextOperation inverse() {
    return InsertOperation(position: position, text: ''); // simplified
  }
  
  Map<String, dynamic> toJson() => {
    'type': 'delete',
    'position': position,
    'length': length,
  };
}

// Collaborative Document
class CollaborativeDocument {
  String _content;
  final String documentId;
  final String userId;
  final WebSocketManager _wsManager;
  int _revision = 0;
  
  final _contentController = StreamController<String>.broadcast();
  
  CollaborativeDocument({
    required this.documentId,
    required this.userId,
    required WebSocketManager wsManager,
    String initialContent = '',
  })  : _content = initialContent,
        _wsManager = wsManager {
    _listenToChanges();
  }
  
  String get content => _content;
  Stream<String> get contentStream => _contentController.stream;
  
  void _listenToChanges() {
    _wsManager.messageStream.listen((data) {
      final json = jsonDecode(data as String) as Map<String, dynamic>;
      if (json['type'] == 'doc_operation' &&
          json['documentId'] == documentId) {
        _applyRemoteOperation(json);
      }
    });
  }
  
  void _applyRemoteOperation(Map<String, dynamic> data) {
    if (data['userId'] == userId) return; // Skip own operations
    
    final opType = data['operation']['type'];
    
    if (opType == 'insert') {
      final op = InsertOperation(
        position: data['operation']['position'] as int,
        text: data['operation']['text'] as String,
      );
      final buffer = StringBuffer(_content);
      op.apply(buffer);
      _content = buffer.toString();
      _contentController.add(_content);
    }
    
    _revision = data['revision'] as int;
  }
  
  void applyOperation(TextOperation op) {
    final buffer = StringBuffer(_content);
    op.apply(buffer);
    _content = buffer.toString();
    
    _revision++;
    
    // Send to server
    Map<String, dynamic> opJson;
    if (op is InsertOperation) {
      opJson = op.toJson();
    } else if (op is DeleteOperation) {
      opJson = op.toJson();
    } else {
      return;
    }
    
    _wsManager.send(jsonEncode({
      'type': 'doc_operation',
      'documentId': documentId,
      'userId': userId,
      'revision': _revision,
      'operation': opJson,
    }));
  }
}
```

---

## ขั้นตอนที่ 427: Presence Indicators

### Online Presence System

```dart
// lib/features/presence/presence_service.dart
class UserPresence {
  final String userId;
  final bool isOnline;
  final DateTime lastSeen;
  final String? currentActivity;
  
  const UserPresence({
    required this.userId,
    required this.isOnline,
    required this.lastSeen,
    this.currentActivity,
  });
  
  factory UserPresence.fromJson(Map<String, dynamic> json) {
    return UserPresence(
      userId: json['userId'] as String,
      isOnline: json['isOnline'] as bool,
      lastSeen: DateTime.parse(json['lastSeen'] as String),
      currentActivity: json['currentActivity'] as String?,
    );
  }
}

class PresenceService {
  final WebSocketManager _wsManager;
  final Map<String, UserPresence> _presences = {};
  final _presenceController =
      StreamController<Map<String, UserPresence>>.broadcast();
  
  PresenceService(this._wsManager) {
    _setupPresenceTracking();
  }
  
  Stream<Map<String, UserPresence>> get presenceStream =>
      _presenceController.stream;
  
  UserPresence? getPresence(String userId) => _presences[userId];
  
  bool isOnline(String userId) => _presences[userId]?.isOnline ?? false;
  
  void _setupPresenceTracking() {
    _wsManager.messageStream.listen((data) {
      final json = jsonDecode(data as String) as Map<String, dynamic>;
      
      if (json['type'] == 'presence_update') {
        final presence = UserPresence.fromJson(
          json['data'] as Map<String, dynamic>,
        );
        _presences[presence.userId] = presence;
        _presenceController.add(Map.from(_presences));
      }
    });
    
    // Subscribe to presence updates
    _wsManager.send(jsonEncode({
      'type': 'subscribe_presence',
    }));
  }
  
  void subscribeToUser(String userId) {
    _wsManager.send(jsonEncode({
      'type': 'watch_user',
      'userId': userId,
    }));
  }
  
  void updateMyStatus({
    bool isOnline = true,
    String? activity,
  }) {
    _wsManager.send(jsonEncode({
      'type': 'update_presence',
      'isOnline': isOnline,
      if (activity != null) 'activity': activity,
    }));
  }
}

// Presence Indicator Widget
class PresenceIndicator extends StatelessWidget {
  final String userId;
  
  const PresenceIndicator({super.key, required this.userId});
  
  @override
  Widget build(BuildContext context) {
    final presenceService = context.read<PresenceService>();
    
    return StreamBuilder<Map<String, UserPresence>>(
      stream: presenceService.presenceStream,
      builder: (context, snapshot) {
        final isOnline = presenceService.isOnline(userId);
        
        return Container(
          width: 12,
          height: 12,
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            color: isOnline ? Colors.green : Colors.grey,
            border: Border.all(
              color: Theme.of(context).colorScheme.surface,
              width: 2,
            ),
          ),
        );
      },
    );
  }
}

// User Avatar with Presence
class UserAvatarWithPresence extends StatelessWidget {
  final String userId;
  final String? avatarUrl;
  final double radius;
  
  const UserAvatarWithPresence({
    super.key,
    required this.userId,
    this.avatarUrl,
    this.radius = 20,
  });
  
  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        CircleAvatar(
          radius: radius,
          backgroundImage:
              avatarUrl != null ? NetworkImage(avatarUrl!) : null,
        ),
        Positioned(
          bottom: 0,
          right: 0,
          child: PresenceIndicator(userId: userId),
        ),
      ],
    );
  }
}
```

---

## ขั้นตอนที่ 428: Real-time Notifications

### Notification Service ด้วย WebSocket

```dart
// lib/services/realtime_notification_service.dart
class AppNotification {
  final String id;
  final String type;
  final String title;
  final String body;
  final Map<String, dynamic>? data;
  final DateTime createdAt;
  bool isRead;
  
  AppNotification({
    required this.id,
    required this.type,
    required this.title,
    required this.body,
    this.data,
    required this.createdAt,
    this.isRead = false,
  });
  
  factory AppNotification.fromJson(Map<String, dynamic> json) {
    return AppNotification(
      id: json['id'] as String,
      type: json['type'] as String,
      title: json['title'] as String,
      body: json['body'] as String,
      data: json['data'] as Map<String, dynamic>?,
      createdAt: DateTime.parse(json['createdAt'] as String),
      isRead: json['isRead'] as bool? ?? false,
    );
  }
}

class RealtimeNotificationService {
  final WebSocketManager _wsManager;
  final List<AppNotification> _notifications = [];
  final _notificationController =
      StreamController<AppNotification>.broadcast();
  final _countController = StreamController<int>.broadcast();
  
  RealtimeNotificationService(this._wsManager) {
    _setupNotificationHandling();
  }
  
  Stream<AppNotification> get notificationStream =>
      _notificationController.stream;
  
  Stream<int> get unreadCountStream => _countController.stream;
  
  int get unreadCount =>
      _notifications.where((n) => !n.isRead).length;
  
  List<AppNotification> get notifications =>
      List.unmodifiable(_notifications);
  
  void _setupNotificationHandling() {
    _wsManager.messageStream.listen((data) {
      final json = jsonDecode(data as String) as Map<String, dynamic>;
      
      if (json['type'] == 'notification') {
        final notification = AppNotification.fromJson(
          json['data'] as Map<String, dynamic>,
        );
        
        _notifications.insert(0, notification);
        _notificationController.add(notification);
        _countController.add(unreadCount);
        
        // Show in-app notification banner
        _showInAppBanner(notification);
      }
    });
  }
  
  void _showInAppBanner(AppNotification notification) {
    // This would be called from UI layer
    print('New notification: ${notification.title}');
  }
  
  void markAsRead(String notificationId) {
    final index = _notifications.indexWhere((n) => n.id == notificationId);
    if (index >= 0) {
      _notifications[index].isRead = true;
      _countController.add(unreadCount);
      
      _wsManager.send(jsonEncode({
        'type': 'mark_notification_read',
        'notificationId': notificationId,
      }));
    }
  }
  
  void markAllAsRead() {
    for (final notification in _notifications) {
      notification.isRead = true;
    }
    _countController.add(0);
    
    _wsManager.send(jsonEncode({
      'type': 'mark_all_notifications_read',
    }));
  }
  
  void dispose() {
    _notificationController.close();
    _countController.close();
  }
}

// Notification Bell Widget
class NotificationBell extends ConsumerWidget {
  const NotificationBell({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final notificationService = ref.watch(notificationServiceProvider);
    
    return StreamBuilder<int>(
      stream: notificationService.unreadCountStream,
      initialData: notificationService.unreadCount,
      builder: (context, snapshot) {
        final count = snapshot.data ?? 0;
        
        return Stack(
          children: [
            IconButton(
              onPressed: () => context.push('/notifications'),
              icon: const Icon(Icons.notifications_outlined),
            ),
            if (count > 0)
              Positioned(
                right: 6,
                top: 6,
                child: Container(
                  padding: const EdgeInsets.all(2),
                  decoration: BoxDecoration(
                    color: Colors.red,
                    borderRadius: BorderRadius.circular(10),
                  ),
                  constraints: const BoxConstraints(
                    minWidth: 16,
                    minHeight: 16,
                  ),
                  child: Text(
                    count > 99 ? '99+' : '$count',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 10,
                      fontWeight: FontWeight.bold,
                    ),
                    textAlign: TextAlign.center,
                  ),
                ),
              ),
          ],
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 429: Performance Optimization

### Message Batching

```dart
// lib/services/message_batcher.dart
class MessageBatcher {
  final Duration batchInterval;
  final WebSocketManager _wsManager;
  final List<Map<String, dynamic>> _batch = [];
  Timer? _batchTimer;
  
  MessageBatcher({
    required WebSocketManager wsManager,
    this.batchInterval = const Duration(milliseconds: 100),
  }) : _wsManager = wsManager;
  
  void queue(Map<String, dynamic> message) {
    _batch.add(message);
    
    _batchTimer ??= Timer(batchInterval, _flush);
  }
  
  void _flush() {
    if (_batch.isEmpty) return;
    
    final messages = List.from(_batch);
    _batch.clear();
    _batchTimer = null;
    
    if (messages.length == 1) {
      _wsManager.send(jsonEncode(messages.first));
    } else {
      _wsManager.send(jsonEncode({
        'type': 'batch',
        'messages': messages,
      }));
    }
  }
  
  void dispose() {
    _batchTimer?.cancel();
  }
}
```

---

## ขั้นตอนที่ 430: Workshop - Real-time Dashboard

### Real-time Stock Dashboard

```dart
// lib/features/dashboard/screens/realtime_dashboard.dart
class RealtimeDashboard extends ConsumerStatefulWidget {
  const RealtimeDashboard({super.key});
  
  @override
  ConsumerState<RealtimeDashboard> createState() =>
      _RealtimeDashboardState();
}

class _RealtimeDashboardState extends ConsumerState<RealtimeDashboard> {
  final Map<String, double> _prices = {};
  final Map<String, double> _changes = {};
  late StreamSubscription _priceSubscription;
  
  final List<String> _watchlist = ['AAPL', 'GOOGL', 'MSFT', 'AMZN', 'META'];
  
  @override
  void initState() {
    super.initState();
    _connectToMarketData();
  }
  
  void _connectToMarketData() {
    final wsManager = ref.read(webSocketManagerProvider);
    
    wsManager.send(jsonEncode({
      'type': 'subscribe_prices',
      'symbols': _watchlist,
    }));
    
    _priceSubscription = wsManager.messageStream.listen((data) {
      final json = jsonDecode(data as String) as Map<String, dynamic>;
      
      if (json['type'] == 'price_update') {
        final symbol = json['symbol'] as String;
        final price = (json['price'] as num).toDouble();
        final previousPrice = _prices[symbol] ?? price;
        
        setState(() {
          _prices[symbol] = price;
          _changes[symbol] = ((price - previousPrice) / previousPrice) * 100;
        });
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Real-time Dashboard'),
        actions: [
          const ConnectionStatusBadge(),
        ],
      ),
      body: RefreshIndicator(
        onRefresh: () async {
          // Force refresh
        },
        child: ListView.builder(
          padding: const EdgeInsets.all(16),
          itemCount: _watchlist.length,
          itemBuilder: (context, index) {
            final symbol = _watchlist[index];
            final price = _prices[symbol];
            final change = _changes[symbol] ?? 0;
            
            return StockCard(
              symbol: symbol,
              price: price,
              changePercent: change,
            );
          },
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _priceSubscription.cancel();
    super.dispose();
  }
}

class StockCard extends StatelessWidget {
  final String symbol;
  final double? price;
  final double changePercent;
  
  const StockCard({
    super.key,
    required this.symbol,
    required this.price,
    required this.changePercent,
  });
  
  @override
  Widget build(BuildContext context) {
    final isPositive = changePercent >= 0;
    
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          child: Text(symbol[0]),
        ),
        title: Text(
          symbol,
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        trailing: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.end,
          children: [
            Text(
              price != null ? '\$${price!.toStringAsFixed(2)}' : '-',
              style: const TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
              ),
            ),
            Text(
              '${isPositive ? '+' : ''}${changePercent.toStringAsFixed(2)}%',
              style: TextStyle(
                color: isPositive ? Colors.green : Colors.red,
                fontSize: 12,
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

## สรุป

ในบทนี้เราได้เรียนรู้:

- **WebSocket Basics**: web_socket_channel package และการใช้งาน
- **Connection Management**: การจัดการ connection state
- **Real-time Chat**: สร้าง chat app ที่สมบูรณ์
- **Reconnection Strategy**: Exponential backoff pattern
- **Socket.IO**: ใช้ socket_io_client สำหรับ advanced features
- **Collaborative Editing**: Operational Transformation concept
- **Presence Indicators**: Online status แบบ real-time
- **Real-time Notifications**: Push notifications ผ่าน WebSocket

## แบบฝึกหัด

1. สร้าง WebSocket server เล็กๆ ด้วย Node.js และเชื่อมกับ Flutter
2. Implement reconnection strategy พร้อม exponential backoff
3. สร้าง chat room ที่มี typing indicators และ read receipts
4. เพิ่ม presence system แสดงสถานะออนไลน์ของ users
5. สร้าง real-time notification center

---

[⬅️ Part 42](part_42.md) | [Part 44 ➡️](part_44.md)

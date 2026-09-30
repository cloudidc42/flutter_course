# Part 17: HTTP & REST APIs
## ขั้นตอนที่ 161-170

---

## สารบัญ
1. [http Package พื้นฐาน](#ขั้นตอนที่-161-http-package-พื้นฐาน)
2. [dio Package](#ขั้นตอนที่-162-dio-package)
3. [GET Requests](#ขั้นตอนที่-163-get-requests)
4. [POST, PUT, DELETE Requests](#ขั้นตอนที่-164-post-put-delete-requests)
5. [Headers และ Authentication](#ขั้นตอนที่-165-headers-และ-authentication)
6. [Query Parameters](#ขั้นตอนที่-166-query-parameters)
7. [Interceptors](#ขั้นตอนที่-167-interceptors)
8. [Loading States และ Error Handling](#ขั้นตอนที่-168-loading-states-และ-error-handling)
9. [Retry Logic](#ขั้นตอนที่-169-retry-logic)
10. [API Client Pattern](#ขั้นตอนที่-170-api-client-pattern)
11. [Workshop: GitHub Profile App](#workshop-github-profile-app)

---

## ขั้นตอนที่ 161: http Package พื้นฐาน

```yaml
# pubspec.yaml
dependencies:
  http: ^1.2.1
```

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

// GET request พื้นฐาน
Future<void> basicGet() async {
  final uri = Uri.parse('https://jsonplaceholder.typicode.com/posts/1');

  try {
    final response = await http.get(
      uri,
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body) as Map<String, dynamic>;
      print('Title: ${data['title']}');
    } else {
      print('Error: ${response.statusCode}');
    }
  } catch (e) {
    print('Exception: $e');
  }
}

// POST request
Future<Map<String, dynamic>?> createPost(String title, String body) async {
  final uri = Uri.parse('https://jsonplaceholder.typicode.com/posts');

  final response = await http.post(
    uri,
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({
      'title': title,
      'body': body,
      'userId': 1,
    }),
  );

  if (response.statusCode == 201) {
    return jsonDecode(response.body) as Map<String, dynamic>;
  }
  return null;
}

// ใช้ Client สำหรับ request หลายครั้ง (ประหยัด connection)
Future<void> useClient() async {
  final client = http.Client();
  try {
    final response1 = await client.get(
      Uri.parse('https://jsonplaceholder.typicode.com/posts/1'),
    );
    final response2 = await client.get(
      Uri.parse('https://jsonplaceholder.typicode.com/posts/2'),
    );
    print('Response 1: ${response1.statusCode}');
    print('Response 2: ${response2.statusCode}');
  } finally {
    client.close(); // สำคัญ! ต้องปิด client
  }
}

// Timeout
Future<String?> getWithTimeout(String url) async {
  try {
    final response = await http.get(
      Uri.parse(url),
    ).timeout(
      const Duration(seconds: 10),
      onTimeout: () => throw TimeoutException('Request timed out'),
    );
    return response.body;
  } on TimeoutException {
    print('Request timed out');
    return null;
  }
}

class TimeoutException implements Exception {
  final String message;
  TimeoutException(this.message);
}
```

---

## ขั้นตอนที่ 162: dio Package

Dio มี features มากกว่า http package เช่น interceptors, cancellation, progress tracking

```yaml
# pubspec.yaml
dependencies:
  dio: ^5.4.1
```

```dart
import 'package:dio/dio.dart';

// สร้าง Dio instance
final dio = Dio(BaseOptions(
  baseUrl: 'https://jsonplaceholder.typicode.com',
  connectTimeout: const Duration(seconds: 10),
  receiveTimeout: const Duration(seconds: 10),
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
));

// GET พื้นฐาน
Future<void> dioGet() async {
  try {
    final response = await dio.get('/posts/1');
    print('Data: ${response.data}');
    print('Status: ${response.statusCode}');
  } on DioException catch (e) {
    _handleDioError(e);
  }
}

// POST พื้นฐาน
Future<void> dioPost() async {
  try {
    final response = await dio.post(
      '/posts',
      data: {
        'title': 'New Post',
        'body': 'Content',
        'userId': 1,
      },
    );
    print('Created: ${response.data}');
  } on DioException catch (e) {
    _handleDioError(e);
  }
}

// File upload กับ progress
Future<void> uploadFile(String filePath) async {
  final formData = FormData.fromMap({
    'file': await MultipartFile.fromFile(
      filePath,
      filename: 'upload.jpg',
    ),
    'description': 'My file',
  });

  await dio.post(
    '/upload',
    data: formData,
    onSendProgress: (sent, total) {
      final progress = (sent / total * 100).toStringAsFixed(1);
      print('Upload progress: $progress%');
    },
  );
}

// Download กับ progress
Future<void> downloadFile(String url, String savePath) async {
  await dio.download(
    url,
    savePath,
    onReceiveProgress: (received, total) {
      if (total != -1) {
        final progress = (received / total * 100).toStringAsFixed(1);
        print('Download: $progress%');
      }
    },
  );
}

// Cancel request
Future<void> cancelableRequest() async {
  final cancelToken = CancelToken();

  // Cancel หลัง 2 วินาที
  Future.delayed(const Duration(seconds: 2), () {
    cancelToken.cancel('User cancelled');
  });

  try {
    await dio.get('/posts', cancelToken: cancelToken);
  } on DioException catch (e) {
    if (CancelToken.isCancel(e)) {
      print('Request cancelled: ${e.message}');
    }
  }
}

void _handleDioError(DioException e) {
  switch (e.type) {
    case DioExceptionType.connectionTimeout:
      print('Connection timeout');
      break;
    case DioExceptionType.receiveTimeout:
      print('Receive timeout');
      break;
    case DioExceptionType.badResponse:
      print('Bad response: ${e.response?.statusCode}');
      break;
    case DioExceptionType.cancel:
      print('Request cancelled');
      break;
    default:
      print('Unknown error: ${e.message}');
  }
}
```

---

## ขั้นตอนที่ 163: GET Requests

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Model
class Post {
  final int id;
  final int userId;
  final String title;
  final String body;

  const Post({
    required this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}

// Repository
class PostRepository {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';

  // GET all posts
  Future<List<Post>> getPosts() async {
    final response = await http.get(
      Uri.parse('$baseUrl/posts'),
    );

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body) as List;
      return data.map((json) => Post.fromJson(json as Map<String, dynamic>)).toList();
    }
    throw Exception('Failed to load posts: ${response.statusCode}');
  }

  // GET single post
  Future<Post> getPost(int id) async {
    final response = await http.get(
      Uri.parse('$baseUrl/posts/$id'),
    );

    if (response.statusCode == 200) {
      return Post.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
    }
    throw Exception('Post not found');
  }

  // GET with query parameters
  Future<List<Post>> getPostsByUser(int userId) async {
    final uri = Uri.parse('$baseUrl/posts').replace(
      queryParameters: {'userId': userId.toString()},
    );

    final response = await http.get(uri);
    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body) as List;
      return data.map((json) => Post.fromJson(json as Map<String, dynamic>)).toList();
    }
    throw Exception('Failed to load user posts');
  }
}

// Screen
class PostsScreen extends StatefulWidget {
  const PostsScreen({super.key});

  @override
  State<PostsScreen> createState() => _PostsScreenState();
}

class _PostsScreenState extends State<PostsScreen> {
  final _repository = PostRepository();
  List<Post> _posts = [];
  bool _isLoading = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadPosts();
  }

  Future<void> _loadPosts() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });

    try {
      final posts = await _repository.getPosts();
      if (mounted) {
        setState(() {
          _posts = posts;
          _isLoading = false;
        });
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _error = e.toString();
          _isLoading = false;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Posts'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _loadPosts,
          ),
        ],
      ),
      body: _buildBody(),
    );
  }

  Widget _buildBody() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, color: Colors.red, size: 48),
            const SizedBox(height: 8),
            Text(_error!, textAlign: TextAlign.center),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: _loadPosts,
              child: const Text('ลองใหม่'),
            ),
          ],
        ),
      );
    }

    return RefreshIndicator(
      onRefresh: _loadPosts,
      child: ListView.builder(
        itemCount: _posts.length,
        itemBuilder: (context, index) {
          final post = _posts[index];
          return ListTile(
            leading: CircleAvatar(child: Text('${post.id}')),
            title: Text(post.title),
            subtitle: Text(
              post.body,
              maxLines: 2,
              overflow: TextOverflow.ellipsis,
            ),
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => PostDetailScreen(postId: post.id),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class PostDetailScreen extends StatefulWidget {
  final int postId;
  const PostDetailScreen({super.key, required this.postId});

  @override
  State<PostDetailScreen> createState() => _PostDetailScreenState();
}

class _PostDetailScreenState extends State<PostDetailScreen> {
  final _repository = PostRepository();
  Post? _post;
  bool _isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadPost();
  }

  Future<void> _loadPost() async {
    try {
      final post = await _repository.getPost(widget.postId);
      if (mounted) setState(() { _post = post; _isLoading = false; });
    } catch (e) {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Post #${widget.postId}')),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _post == null
              ? const Center(child: Text('ไม่พบข้อมูล'))
              : Padding(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(_post!.title, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
                      const SizedBox(height: 8),
                      Text(_post!.body),
                    ],
                  ),
                ),
    );
  }
}
```

---

## ขั้นตอนที่ 164: POST, PUT, DELETE Requests

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class PostService {
  static const String baseUrl = 'https://jsonplaceholder.typicode.com';
  final http.Client _client = http.Client();

  // POST - สร้างข้อมูลใหม่
  Future<Map<String, dynamic>> createPost({
    required String title,
    required String body,
    required int userId,
  }) async {
    final response = await _client.post(
      Uri.parse('$baseUrl/posts'),
      headers: {'Content-Type': 'application/json; charset=UTF-8'},
      body: jsonEncode({
        'title': title,
        'body': body,
        'userId': userId,
      }),
    );

    if (response.statusCode == 201) {
      return jsonDecode(response.body) as Map<String, dynamic>;
    }
    throw HttpException('Failed to create post', statusCode: response.statusCode);
  }

  // PUT - อัปเดตข้อมูลทั้งหมด
  Future<Map<String, dynamic>> updatePost({
    required int id,
    required String title,
    required String body,
    required int userId,
  }) async {
    final response = await _client.put(
      Uri.parse('$baseUrl/posts/$id'),
      headers: {'Content-Type': 'application/json; charset=UTF-8'},
      body: jsonEncode({
        'id': id,
        'title': title,
        'body': body,
        'userId': userId,
      }),
    );

    if (response.statusCode == 200) {
      return jsonDecode(response.body) as Map<String, dynamic>;
    }
    throw HttpException('Failed to update post', statusCode: response.statusCode);
  }

  // PATCH - อัปเดตบางฟิลด์
  Future<Map<String, dynamic>> patchPost({
    required int id,
    String? title,
    String? body,
  }) async {
    final patchData = <String, dynamic>{};
    if (title != null) patchData['title'] = title;
    if (body != null) patchData['body'] = body;

    final response = await _client.patch(
      Uri.parse('$baseUrl/posts/$id'),
      headers: {'Content-Type': 'application/json; charset=UTF-8'},
      body: jsonEncode(patchData),
    );

    if (response.statusCode == 200) {
      return jsonDecode(response.body) as Map<String, dynamic>;
    }
    throw HttpException('Failed to patch post', statusCode: response.statusCode);
  }

  // DELETE - ลบข้อมูล
  Future<bool> deletePost(int id) async {
    final response = await _client.delete(
      Uri.parse('$baseUrl/posts/$id'),
      headers: {'Content-Type': 'application/json; charset=UTF-8'},
    );

    return response.statusCode == 200;
  }

  void dispose() {
    _client.close();
  }
}

class HttpException implements Exception {
  final String message;
  final int statusCode;

  HttpException(this.message, {required this.statusCode});

  @override
  String toString() => '$message (HTTP $statusCode)';
}
```

---

## ขั้นตอนที่ 165: Headers และ Authentication

```dart
import 'dart:convert';
import 'package:dio/dio.dart';

class AuthenticatedApiClient {
  late Dio _dio;
  String? _accessToken;
  String? _refreshToken;

  AuthenticatedApiClient() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com/v1',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'App-Version': '1.0.0',
        'Platform': 'flutter',
      },
    ));

    _setupInterceptors();
  }

  void _setupInterceptors() {
    _dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) {
          // เพิ่ม Authorization header
          if (_accessToken != null) {
            options.headers['Authorization'] = 'Bearer $_accessToken';
          }
          print('📤 ${options.method} ${options.uri}');
          handler.next(options);
        },
        onResponse: (response, handler) {
          print('📥 ${response.statusCode} ${response.requestOptions.uri}');
          handler.next(response);
        },
        onError: (error, handler) async {
          if (error.response?.statusCode == 401) {
            // Token หมดอายุ - refresh token
            try {
              await _refreshAccessToken();
              // Retry request ด้วย token ใหม่
              final retryResponse = await _retryRequest(error.requestOptions);
              return handler.resolve(retryResponse);
            } catch (e) {
              // Refresh failed - logout
              _clearTokens();
            }
          }
          handler.next(error);
        },
      ),
    );
  }

  Future<void> login(String email, String password) async {
    final response = await _dio.post(
      '/auth/login',
      data: {'email': email, 'password': password},
      // ไม่ใส่ Authorization header สำหรับ login
      options: Options(headers: {'Authorization': null}),
    );

    _accessToken = response.data['access_token'] as String;
    _refreshToken = response.data['refresh_token'] as String;
  }

  Future<void> _refreshAccessToken() async {
    final response = await Dio().post(
      'https://api.example.com/v1/auth/refresh',
      data: {'refresh_token': _refreshToken},
    );
    _accessToken = response.data['access_token'] as String;
  }

  Future<Response<dynamic>> _retryRequest(RequestOptions requestOptions) async {
    requestOptions.headers['Authorization'] = 'Bearer $_accessToken';
    return _dio.fetch(requestOptions);
  }

  void _clearTokens() {
    _accessToken = null;
    _refreshToken = null;
  }

  // API Methods
  Future<Map<String, dynamic>> getCurrentUser() async {
    final response = await _dio.get('/users/me');
    return response.data as Map<String, dynamic>;
  }

  Future<List<dynamic>> getProtectedData() async {
    final response = await _dio.get('/protected/data');
    return response.data as List<dynamic>;
  }
}

// Basic Auth
Future<void> basicAuthExample() async {
  final dio = Dio();
  final response = await dio.get(
    'https://httpbin.org/basic-auth/user/pass',
    options: Options(
      headers: {
        'Authorization':
            'Basic ${base64Encode(utf8.encode('user:pass'))}',
      },
    ),
  );
  print(response.data);
}

// API Key
Future<void> apiKeyExample() async {
  final dio = Dio(BaseOptions(
    headers: {
      'X-API-Key': 'your-api-key-here',
      // หรือ
      // 'Authorization': 'ApiKey your-api-key',
    },
  ));
  final response = await dio.get('https://api.example.com/data');
  print(response.data);
}
```

---

## ขั้นตอนที่ 166: Query Parameters

```dart
import 'package:dio/dio.dart';

class SearchService {
  final Dio _dio = Dio(BaseOptions(
    baseUrl: 'https://api.themoviedb.org/3',
  ));

  // GET กับ query parameters
  Future<List<Map<String, dynamic>>> searchMovies({
    required String query,
    int page = 1,
    String language = 'th-TH',
    bool includeAdult = false,
  }) async {
    final response = await _dio.get(
      '/search/movie',
      queryParameters: {
        'api_key': 'your-api-key',
        'query': query,
        'page': page,
        'language': language,
        'include_adult': includeAdult,
      },
    );

    final results = response.data['results'] as List;
    return results.cast<Map<String, dynamic>>();
  }

  // Pagination
  Future<Map<String, dynamic>> getPaginatedData({
    int page = 1,
    int perPage = 20,
    String? sortBy,
    String? sortOrder,
    Map<String, dynamic>? filters,
  }) async {
    final queryParams = <String, dynamic>{
      'page': page,
      'per_page': perPage,
    };

    if (sortBy != null) queryParams['sort_by'] = sortBy;
    if (sortOrder != null) queryParams['sort_order'] = sortOrder;
    if (filters != null) queryParams.addAll(filters);

    final response = await _dio.get(
      '/items',
      queryParameters: queryParams,
    );

    return {
      'data': response.data['data'],
      'total': response.data['total'],
      'page': response.data['page'],
      'hasMore': (response.data['page'] as int) < (response.data['total_pages'] as int),
    };
  }
}

// การสร้าง URI พร้อม query parameters ด้วย http package
Uri buildSearchUri({
  required String query,
  List<String>? tags,
  int? limit,
}) {
  return Uri.https(
    'api.example.com',
    '/search',
    {
      'q': query,
      if (tags != null) 'tags': tags.join(','),
      if (limit != null) 'limit': limit.toString(),
    },
  );
}
```

---

## ขั้นตอนที่ 167: Interceptors

```dart
import 'package:dio/dio.dart';

// Logging Interceptor
class LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('┌─────────────────────────────────────────────────');
    print('│ 📤 REQUEST: ${options.method} ${options.uri}');
    print('│ Headers: ${options.headers}');
    if (options.data != null) print('│ Body: ${options.data}');
    print('└─────────────────────────────────────────────────');
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print('┌─────────────────────────────────────────────────');
    print('│ 📥 RESPONSE: ${response.statusCode} ${response.requestOptions.uri}');
    print('│ Data: ${response.data.toString().substring(0, response.data.toString().length.clamp(0, 200))}...');
    print('└─────────────────────────────────────────────────');
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print('┌─────────────────────────────────────────────────');
    print('│ ❌ ERROR: ${err.type} ${err.requestOptions.uri}');
    print('│ Message: ${err.message}');
    print('└─────────────────────────────────────────────────');
    handler.next(err);
  }
}

// Auth Interceptor
class AuthInterceptor extends Interceptor {
  final String Function() getToken;
  final Future<void> Function() onUnauthorized;

  AuthInterceptor({
    required this.getToken,
    required this.onUnauthorized,
  });

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final token = getToken();
    if (token.isNotEmpty) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      await onUnauthorized();
    }
    handler.next(err);
  }
}

// Cache Interceptor (simple in-memory cache)
class CacheInterceptor extends Interceptor {
  final Map<String, _CacheEntry> _cache = {};
  final Duration cacheDuration;

  CacheInterceptor({this.cacheDuration = const Duration(minutes: 5)});

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (options.method != 'GET') {
      handler.next(options);
      return;
    }

    final key = options.uri.toString();
    final cached = _cache[key];

    if (cached != null && !cached.isExpired) {
      print('Cache HIT: $key');
      handler.resolve(
        Response(
          requestOptions: options,
          data: cached.data,
          statusCode: 200,
        ),
      );
      return;
    }

    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    if (response.requestOptions.method == 'GET' && response.statusCode == 200) {
      final key = response.requestOptions.uri.toString();
      _cache[key] = _CacheEntry(data: response.data, duration: cacheDuration);
      print('Cache STORE: $key');
    }
    handler.next(response);
  }

  void clearCache() => _cache.clear();
}

class _CacheEntry {
  final dynamic data;
  final DateTime expiresAt;

  _CacheEntry({required this.data, required Duration duration})
      : expiresAt = DateTime.now().add(duration);

  bool get isExpired => DateTime.now().isAfter(expiresAt);
}

// Setup Dio กับ Interceptors ทั้งหมด
Dio createDio({required String Function() getToken}) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.example.com',
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 10),
  ));

  dio.interceptors.addAll([
    LoggingInterceptor(),
    AuthInterceptor(
      getToken: getToken,
      onUnauthorized: () async {
        // Navigate to login
      },
    ),
    CacheInterceptor(cacheDuration: const Duration(minutes: 10)),
  ]);

  return dio;
}
```

---

## ขั้นตอนที่ 168: Loading States และ Error Handling

```dart
import 'package:flutter/material.dart';

// Result type สำหรับ handle success/error
sealed class ApiResult<T> {
  const ApiResult();
}

class ApiSuccess<T> extends ApiResult<T> {
  final T data;
  const ApiSuccess(this.data);
}

class ApiError<T> extends ApiResult<T> {
  final String message;
  final int? statusCode;
  final dynamic originalError;

  const ApiError({
    required this.message,
    this.statusCode,
    this.originalError,
  });
}

class ApiLoading<T> extends ApiResult<T> {
  const ApiLoading();
}

// Extension สำหรับ handle ApiResult
extension ApiResultExtension<T> on ApiResult<T> {
  bool get isLoading => this is ApiLoading<T>;
  bool get isSuccess => this is ApiSuccess<T>;
  bool get isError => this is ApiError<T>;

  T? get data => this is ApiSuccess<T> ? (this as ApiSuccess<T>).data : null;
  String? get errorMessage => this is ApiError<T> ? (this as ApiError<T>).message : null;

  R when<R>({
    required R Function() loading,
    required R Function(T data) success,
    required R Function(String message, int? statusCode) error,
  }) {
    if (this is ApiLoading<T>) return loading();
    if (this is ApiSuccess<T>) return success((this as ApiSuccess<T>).data);
    final e = this as ApiError<T>;
    return error(e.message, e.statusCode);
  }
}

// Error types
class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
  @override String toString() => 'NetworkException: $message';
}

class ServerException implements Exception {
  final String message;
  final int statusCode;
  ServerException(this.message, {required this.statusCode});
  @override String toString() => 'ServerException [$statusCode]: $message';
}

class ValidationException implements Exception {
  final Map<String, List<String>> errors;
  ValidationException(this.errors);
  @override String toString() => 'ValidationException: $errors';
}

// Error handling utility
String getErrorMessage(dynamic error) {
  if (error is NetworkException) {
    return 'ไม่สามารถเชื่อมต่อได้ กรุณาตรวจสอบอินเทอร์เน็ต';
  }
  if (error is ServerException) {
    switch (error.statusCode) {
      case 400: return 'ข้อมูลไม่ถูกต้อง';
      case 401: return 'กรุณาเข้าสู่ระบบก่อน';
      case 403: return 'ไม่มีสิทธิ์เข้าถึง';
      case 404: return 'ไม่พบข้อมูลที่ต้องการ';
      case 500: return 'เซิร์ฟเวอร์มีปัญหา กรุณาลองใหม่';
      default: return 'เกิดข้อผิดพลาด (${error.statusCode})';
    }
  }
  if (error is ValidationException) {
    return error.errors.values.expand((e) => e).join(', ');
  }
  return 'เกิดข้อผิดพลาดที่ไม่รู้จัก';
}

// Widget สำหรับแสดง API result
class ApiResultWidget<T> extends StatelessWidget {
  final ApiResult<T> result;
  final Widget Function(T data) builder;
  final Widget? loadingWidget;
  final Widget Function(String message)? errorBuilder;

  const ApiResultWidget({
    super.key,
    required this.result,
    required this.builder,
    this.loadingWidget,
    this.errorBuilder,
  });

  @override
  Widget build(BuildContext context) {
    return result.when(
      loading: () => loadingWidget ?? const Center(child: CircularProgressIndicator()),
      success: builder,
      error: (message, statusCode) {
        if (errorBuilder != null) return errorBuilder!(message);
        return Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.error_outline, color: Colors.red, size: 48),
              const SizedBox(height: 8),
              Text(message, textAlign: TextAlign.center),
            ],
          ),
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 169: Retry Logic

```dart
import 'dart:async';
import 'package:dio/dio.dart';

// Retry กับ exponential backoff
class RetryInterceptor extends Interceptor {
  final Dio dio;
  final int maxRetries;
  final Duration initialDelay;

  RetryInterceptor({
    required this.dio,
    this.maxRetries = 3,
    this.initialDelay = const Duration(seconds: 1),
  });

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    int retryCount = err.requestOptions.extra['retry_count'] as int? ?? 0;

    // Retry เฉพาะ network errors หรือ 5xx
    final shouldRetry = _shouldRetry(err) && retryCount < maxRetries;

    if (shouldRetry) {
      retryCount++;
      err.requestOptions.extra['retry_count'] = retryCount;

      // Exponential backoff: 1s, 2s, 4s...
      final delay = initialDelay * (1 << (retryCount - 1));
      print('Retry attempt $retryCount after ${delay.inMilliseconds}ms');

      await Future.delayed(delay);

      try {
        final response = await dio.fetch(err.requestOptions);
        handler.resolve(response);
      } on DioException catch (e) {
        handler.next(e);
      }
    } else {
      handler.next(err);
    }
  }

  bool _shouldRetry(DioException err) {
    return err.type == DioExceptionType.connectionTimeout ||
        err.type == DioExceptionType.receiveTimeout ||
        err.type == DioExceptionType.connectionError ||
        (err.response?.statusCode != null && err.response!.statusCode! >= 500);
  }
}

// Manual retry สำหรับ specific operations
class RetryableOperation<T> {
  final Future<T> Function() operation;
  final int maxAttempts;
  final Duration delay;
  final bool Function(Exception)? shouldRetry;

  const RetryableOperation({
    required this.operation,
    this.maxAttempts = 3,
    this.delay = const Duration(seconds: 1),
    this.shouldRetry,
  });

  Future<T> execute() async {
    int attempt = 0;
    while (true) {
      try {
        return await operation();
      } on Exception catch (e) {
        attempt++;
        if (attempt >= maxAttempts) rethrow;
        if (shouldRetry != null && !shouldRetry!(e)) rethrow;

        print('Attempt $attempt failed, retrying...');
        await Future.delayed(delay * attempt);
      }
    }
  }
}

// การใช้งาน
Future<void> retryExample() async {
  final retryable = RetryableOperation(
    operation: () async {
      // API call ที่อาจ fail
      final response = await Dio().get('https://api.example.com/data');
      return response.data;
    },
    maxAttempts: 3,
    delay: const Duration(seconds: 2),
  );

  try {
    final data = await retryable.execute();
    print('Success: $data');
  } catch (e) {
    print('Failed after all retries: $e');
  }
}
```

---

## ขั้นตอนที่ 170: API Client Pattern

```dart
import 'dart:convert';
import 'package:dio/dio.dart';

// Base API Client
abstract class BaseApiClient {
  late final Dio _dio;

  BaseApiClient({required String baseUrl, Map<String, dynamic>? headers}) {
    _dio = Dio(BaseOptions(
      baseUrl: baseUrl,
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        ...?headers,
      },
    ));

    setupInterceptors(_dio);
  }

  void setupInterceptors(Dio dio) {
    // Override ใน subclass
  }

  Future<T> get<T>(
    String path, {
    Map<String, dynamic>? queryParameters,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.get(
        path,
        queryParameters: queryParameters,
      );
      if (fromJson != null) return fromJson(response.data);
      return response.data as T;
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<T> post<T>(
    String path, {
    dynamic data,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.post(path, data: data);
      if (fromJson != null) return fromJson(response.data);
      return response.data as T;
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<T> put<T>(
    String path, {
    dynamic data,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.put(path, data: data);
      if (fromJson != null) return fromJson(response.data);
      return response.data as T;
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Future<void> delete(String path) async {
    try {
      await _dio.delete(path);
    } on DioException catch (e) {
      throw _mapError(e);
    }
  }

  Exception _mapError(DioException e) {
    if (e.response != null) {
      return ServerException(
        e.response?.data?['message'] as String? ?? 'Server error',
        statusCode: e.response!.statusCode!,
      );
    }
    return NetworkException('Network error: ${e.message}');
  }
}

class ServerException implements Exception {
  final String message;
  final int statusCode;
  ServerException(this.message, {required this.statusCode});
}

class NetworkException implements Exception {
  final String message;
  NetworkException(this.message);
}

// Concrete API Client
class JsonPlaceholderClient extends BaseApiClient {
  JsonPlaceholderClient() : super(baseUrl: 'https://jsonplaceholder.typicode.com');

  Future<List<PostModel>> getPosts() async {
    return get<List<PostModel>>(
      '/posts',
      fromJson: (data) => (data as List)
          .map((json) => PostModel.fromJson(json as Map<String, dynamic>))
          .toList(),
    );
  }

  Future<PostModel> getPost(int id) async {
    return get<PostModel>(
      '/posts/$id',
      fromJson: (data) => PostModel.fromJson(data as Map<String, dynamic>),
    );
  }

  Future<PostModel> createPost(PostModel post) async {
    return post<PostModel>(
      '/posts',
      data: post.toJson(),
      fromJson: (data) => PostModel.fromJson(data as Map<String, dynamic>),
    );
  }

  Future<void> deletePost(int id) async {
    return delete('/posts/$id');
  }
}

class PostModel {
  final int? id;
  final int userId;
  final String title;
  final String body;

  const PostModel({
    this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  factory PostModel.fromJson(Map<String, dynamic> json) {
    return PostModel(
      id: json['id'] as int?,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }

  Map<String, dynamic> toJson() => {
    if (id != null) 'id': id,
    'userId': userId,
    'title': title,
    'body': body,
  };
}
```

---

## Workshop: GitHub Profile App

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Models
class GitHubUser {
  final String login;
  final String name;
  final String? avatarUrl;
  final String? bio;
  final int publicRepos;
  final int followers;
  final int following;
  final String? location;
  final String? company;
  final String? blog;

  const GitHubUser({
    required this.login,
    required this.name,
    this.avatarUrl,
    this.bio,
    required this.publicRepos,
    required this.followers,
    required this.following,
    this.location,
    this.company,
    this.blog,
  });

  factory GitHubUser.fromJson(Map<String, dynamic> json) {
    return GitHubUser(
      login: json['login'] as String,
      name: json['name'] as String? ?? json['login'] as String,
      avatarUrl: json['avatar_url'] as String?,
      bio: json['bio'] as String?,
      publicRepos: json['public_repos'] as int? ?? 0,
      followers: json['followers'] as int? ?? 0,
      following: json['following'] as int? ?? 0,
      location: json['location'] as String?,
      company: json['company'] as String?,
      blog: json['blog'] as String?,
    );
  }
}

class GitHubRepo {
  final int id;
  final String name;
  final String? description;
  final String language;
  final int stargazersCount;
  final int forksCount;
  final bool isPrivate;
  final DateTime updatedAt;

  const GitHubRepo({
    required this.id,
    required this.name,
    this.description,
    required this.language,
    required this.stargazersCount,
    required this.forksCount,
    required this.isPrivate,
    required this.updatedAt,
  });

  factory GitHubRepo.fromJson(Map<String, dynamic> json) {
    return GitHubRepo(
      id: json['id'] as int,
      name: json['name'] as String,
      description: json['description'] as String?,
      language: json['language'] as String? ?? 'Unknown',
      stargazersCount: json['stargazers_count'] as int? ?? 0,
      forksCount: json['forks_count'] as int? ?? 0,
      isPrivate: json['private'] as bool? ?? false,
      updatedAt: DateTime.parse(json['updated_at'] as String),
    );
  }
}

// API Service
class GitHubApiService {
  static const _baseUrl = 'https://api.github.com';
  final http.Client _client;

  GitHubApiService() : _client = http.Client();

  Map<String, String> get _headers => {
    'Accept': 'application/vnd.github.v3+json',
    'User-Agent': 'Flutter-GitHub-App',
  };

  Future<GitHubUser> getUser(String username) async {
    final response = await _client.get(
      Uri.parse('$_baseUrl/users/$username'),
      headers: _headers,
    );

    if (response.statusCode == 200) {
      return GitHubUser.fromJson(
        jsonDecode(response.body) as Map<String, dynamic>,
      );
    } else if (response.statusCode == 404) {
      throw Exception('ไม่พบผู้ใช้ "$username"');
    }
    throw Exception('เกิดข้อผิดพลาด: ${response.statusCode}');
  }

  Future<List<GitHubRepo>> getUserRepos(
    String username, {
    int page = 1,
    String sort = 'updated',
  }) async {
    final uri = Uri.parse('$_baseUrl/users/$username/repos').replace(
      queryParameters: {
        'page': page.toString(),
        'per_page': '20',
        'sort': sort,
      },
    );

    final response = await _client.get(uri, headers: _headers);

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body) as List;
      return data
          .map((json) => GitHubRepo.fromJson(json as Map<String, dynamic>))
          .toList();
    }
    throw Exception('ไม่สามารถโหลด repositories ได้');
  }

  void dispose() => _client.close();
}

// App
class GitHubApp extends StatelessWidget {
  const GitHubApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'GitHub Profile',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF24292E)),
        useMaterial3: true,
      ),
      home: const GitHubSearchScreen(),
    );
  }
}

class GitHubSearchScreen extends StatefulWidget {
  const GitHubSearchScreen({super.key});

  @override
  State<GitHubSearchScreen> createState() => _GitHubSearchScreenState();
}

class _GitHubSearchScreenState extends State<GitHubSearchScreen> {
  final _service = GitHubApiService();
  final _controller = TextEditingController();
  GitHubUser? _user;
  List<GitHubRepo> _repos = [];
  bool _isLoading = false;
  String? _error;

  @override
  void dispose() {
    _service.dispose();
    _controller.dispose();
    super.dispose();
  }

  Future<void> _search(String username) async {
    if (username.trim().isEmpty) return;

    setState(() {
      _isLoading = true;
      _error = null;
      _user = null;
      _repos = [];
    });

    try {
      final user = await _service.getUser(username.trim());
      final repos = await _service.getUserRepos(username.trim());

      if (mounted) {
        setState(() {
          _user = user;
          _repos = repos;
          _isLoading = false;
        });
      }
    } catch (e) {
      if (mounted) {
        setState(() {
          _error = e.toString().replaceAll('Exception: ', '');
          _isLoading = false;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
        backgroundColor: const Color(0xFF24292E),
        foregroundColor: Colors.white,
        title: const Text('GitHub Profile'),
      ),
      body: Column(
        children: [
          // Search bar
          Container(
            padding: const EdgeInsets.all(16),
            color: const Color(0xFF24292E),
            child: TextField(
              controller: _controller,
              style: const TextStyle(color: Colors.white),
              decoration: InputDecoration(
                hintText: 'ค้นหา GitHub username...',
                hintStyle: const TextStyle(color: Colors.white54),
                prefixIcon: const Icon(Icons.search, color: Colors.white54),
                suffixIcon: IconButton(
                  icon: const Icon(Icons.send, color: Colors.white),
                  onPressed: () => _search(_controller.text),
                ),
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8),
                  borderSide: BorderSide(color: Colors.white.withOpacity(0.3)),
                ),
                enabledBorder: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(8),
                  borderSide: BorderSide(color: Colors.white.withOpacity(0.3)),
                ),
                filled: true,
                fillColor: Colors.white.withOpacity(0.1),
              ),
              onSubmitted: _search,
              textInputAction: TextInputAction.search,
            ),
          ),

          // Content
          Expanded(
            child: _buildContent(),
          ),
        ],
      ),
    );
  }

  Widget _buildContent() {
    if (_isLoading) {
      return const Center(child: CircularProgressIndicator());
    }

    if (_error != null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.person_off, size: 64, color: Colors.grey),
            const SizedBox(height: 16),
            Text(
              _error!,
              style: const TextStyle(fontSize: 16),
              textAlign: TextAlign.center,
            ),
          ],
        ),
      );
    }

    if (_user == null) {
      return const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.search, size: 64, color: Colors.grey),
            SizedBox(height: 16),
            Text('ค้นหา GitHub username'),
          ],
        ),
      );
    }

    return ListView(
      children: [
        // Profile header
        _buildProfileHeader(_user!),
        const Divider(height: 1),

        // Repositories
        Padding(
          padding: const EdgeInsets.all(16),
          child: Text(
            'Repositories (${_repos.length})',
            style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
          ),
        ),
        ..._repos.map((repo) => _buildRepoCard(repo)),
      ],
    );
  }

  Widget _buildProfileHeader(GitHubUser user) {
    return Container(
      padding: const EdgeInsets.all(16),
      color: Colors.white,
      child: Row(
        children: [
          // Avatar
          ClipRRect(
            borderRadius: BorderRadius.circular(40),
            child: user.avatarUrl != null
                ? Image.network(
                    user.avatarUrl!,
                    width: 80,
                    height: 80,
                    fit: BoxFit.cover,
                    errorBuilder: (_, __, ___) => const CircleAvatar(
                      radius: 40,
                      child: Icon(Icons.person, size: 40),
                    ),
                  )
                : const CircleAvatar(
                    radius: 40,
                    child: Icon(Icons.person, size: 40),
                  ),
          ),
          const SizedBox(width: 16),

          // Info
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  user.name,
                  style: const TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  '@${user.login}',
                  style: const TextStyle(color: Colors.grey),
                ),
                if (user.bio != null) ...[
                  const SizedBox(height: 4),
                  Text(user.bio!, maxLines: 2, overflow: TextOverflow.ellipsis),
                ],
                if (user.location != null) ...[
                  const SizedBox(height: 4),
                  Row(
                    children: [
                      const Icon(Icons.location_on, size: 14, color: Colors.grey),
                      Text(user.location!, style: const TextStyle(color: Colors.grey, fontSize: 12)),
                    ],
                  ),
                ],
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildRepoCard(GitHubRepo repo) {
    final languageColors = {
      'Dart': const Color(0xFF00B4AB),
      'JavaScript': const Color(0xFFF1E05A),
      'Python': const Color(0xFF3572A5),
      'Java': const Color(0xFFB07219),
      'Kotlin': const Color(0xFFA97BFF),
      'Swift': const Color(0xFFFA7343),
    };

    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Row(
              children: [
                const Icon(Icons.book_outlined, size: 16, color: Colors.grey),
                const SizedBox(width: 8),
                Expanded(
                  child: Text(
                    repo.name,
                    style: const TextStyle(
                      fontWeight: FontWeight.bold,
                      color: Colors.blue,
                    ),
                  ),
                ),
                if (repo.isPrivate)
                  Container(
                    padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 2),
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.grey),
                      borderRadius: BorderRadius.circular(4),
                    ),
                    child: const Text('Private', style: TextStyle(fontSize: 11)),
                  ),
              ],
            ),
            if (repo.description != null) ...[
              const SizedBox(height: 4),
              Text(
                repo.description!,
                style: const TextStyle(color: Colors.grey, fontSize: 13),
                maxLines: 2,
                overflow: TextOverflow.ellipsis,
              ),
            ],
            const SizedBox(height: 8),
            Row(
              children: [
                // Language
                if (repo.language != 'Unknown') ...[
                  Container(
                    width: 12,
                    height: 12,
                    decoration: BoxDecoration(
                      color: languageColors[repo.language] ?? Colors.grey,
                      shape: BoxShape.circle,
                    ),
                  ),
                  const SizedBox(width: 4),
                  Text(repo.language, style: const TextStyle(fontSize: 12)),
                  const SizedBox(width: 16),
                ],
                // Stars
                const Icon(Icons.star_border, size: 14, color: Colors.grey),
                Text(' ${repo.stargazersCount}', style: const TextStyle(fontSize: 12)),
                const SizedBox(width: 12),
                // Forks
                const Icon(Icons.call_split, size: 14, color: Colors.grey),
                Text(' ${repo.forksCount}', style: const TextStyle(fontSize: 12)),
                const Spacer(),
                // Updated
                Text(
                  'อัปเดต: ${_formatDate(repo.updatedAt)}',
                  style: const TextStyle(fontSize: 11, color: Colors.grey),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  String _formatDate(DateTime date) {
    final diff = DateTime.now().difference(date);
    if (diff.inDays < 1) return 'วันนี้';
    if (diff.inDays < 7) return '${diff.inDays} วันที่แล้ว';
    if (diff.inDays < 30) return '${diff.inDays ~/ 7} สัปดาห์ที่แล้ว';
    return '${diff.inDays ~/ 30} เดือนที่แล้ว';
  }
}

void main() => runApp(const GitHubApp());
```

---

## สรุป (Summary)

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| http package | GET, POST, PUT, DELETE พื้นฐาน |
| dio package | advanced HTTP features |
| Headers & Auth | Bearer token, API key, Basic auth |
| Query Parameters | การส่ง parameters ใน URL |
| Interceptors | logging, auth, caching |
| Error Handling | Result type, exception handling |
| Retry Logic | exponential backoff |
| API Client Pattern | reusable API client |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง app ที่ fetch รายการ users จาก JSONPlaceholder และแสดงใน ListView
2. **ปานกลาง**: สร้าง weather app ที่ใช้ OpenWeatherMap API พร้อม search และ loading states
3. **ยาก**: สร้าง REST client ที่มี token refresh, retry logic, และ offline caching

---

[← Part 16: Riverpod](part_16.md) | [Part 18: JSON Serialization →](part_18.md)

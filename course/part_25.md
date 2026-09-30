# Part 25: Firebase Storage
## ขั้นตอนที่ 241-250

---

## สารบัญ
1. [firebase_storage Package](#firebase_storage-package)
2. [Uploading Files](#uploading-files)
3. [Download URLs](#download-urls)
4. [Delete Files](#delete-files)
5. [Progress Monitoring](#progress-monitoring)
6. [Resumable Uploads](#resumable-uploads)
7. [Metadata](#metadata)
8. [Security Rules](#security-rules)
9. [Image Compression Before Upload](#image-compression-before-upload)
10. [Gallery Management](#gallery-management)

---

## ขั้นตอนที่ 241: firebase_storage Package

Firebase Storage ให้บริการเก็บไฟล์ เช่น รูปภาพ วิดีโอ เอกสาร ได้อย่างปลอดภัย

```yaml
# pubspec.yaml
dependencies:
  firebase_storage: ^11.6.0
  image_picker: ^1.0.7
  flutter_image_compress: ^2.1.0
  path_provider: ^2.1.2
  path: ^1.8.3
  mime: ^1.0.5
```

### โครงสร้างใน Firebase Storage

```
Firebase Storage Bucket
├── users/
│   ├── user_abc123/
│   │   ├── profile.jpg
│   │   └── documents/
│   │       └── resume.pdf
│   └── user_xyz789/
│       └── profile.jpg
├── posts/
│   ├── post_001/
│   │   ├── cover.jpg
│   │   └── images/
│   │       ├── img1.jpg
│   │       └── img2.jpg
└── public/
    └── banners/
        └── banner1.jpg
```

```dart
// lib/services/storage_service.dart
import 'package:firebase_storage/firebase_storage.dart';

class StorageService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // Reference ไปยัง folder หลัก
  static Reference get rootRef => _storage.ref();
  static Reference get usersRef => _storage.ref().child('users');
  static Reference get postsRef => _storage.ref().child('posts');
  static Reference get publicRef => _storage.ref().child('public');

  // สร้าง Reference สำหรับไฟล์
  static Reference getUserProfileRef(String userId) {
    return usersRef.child(userId).child('profile.jpg');
  }

  static Reference getPostImageRef(String postId, String fileName) {
    return postsRef.child(postId).child('images').child(fileName);
  }
}
```

---

## ขั้นตอนที่ 242: Uploading Files

```dart
// lib/services/upload_service.dart
import 'dart:io';
import 'dart:typed_data';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:path/path.dart' as path;
import 'package:mime/mime.dart';

class UploadService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // Upload File (ไฟล์จาก Device)
  static Future<String> uploadFile({
    required File file,
    required String storagePath,
    Map<String, String>? customMetadata,
  }) async {
    final ref = _storage.ref().child(storagePath);

    final metadata = SettableMetadata(
      contentType: lookupMimeType(file.path) ?? 'application/octet-stream',
      customMetadata: customMetadata,
    );

    final uploadTask = ref.putFile(file, metadata);
    final snapshot = await uploadTask;

    return await snapshot.ref.getDownloadURL();
  }

  // Upload Bytes (ไฟล์จาก Memory)
  static Future<String> uploadBytes({
    required Uint8List bytes,
    required String storagePath,
    required String contentType,
  }) async {
    final ref = _storage.ref().child(storagePath);
    final metadata = SettableMetadata(contentType: contentType);

    final uploadTask = ref.putData(bytes, metadata);
    final snapshot = await uploadTask;

    return await snapshot.ref.getDownloadURL();
  }

  // Upload รูปโปรไฟล์
  static Future<String> uploadProfilePhoto({
    required String userId,
    required File imageFile,
  }) async {
    final extension = path.extension(imageFile.path);
    final storagePath = 'users/$userId/profile$extension';

    return await uploadFile(
      file: imageFile,
      storagePath: storagePath,
      customMetadata: {
        'uploadedBy': userId,
        'type': 'profile_photo',
      },
    );
  }

  // Upload รูปสำหรับ Post
  static Future<List<String>> uploadPostImages({
    required String postId,
    required List<File> images,
  }) async {
    final urls = <String>[];

    for (var i = 0; i < images.length; i++) {
      final extension = path.extension(images[i].path);
      final fileName = 'image_$i$extension';
      final storagePath = 'posts/$postId/images/$fileName';

      final url = await uploadFile(
        file: images[i],
        storagePath: storagePath,
      );
      urls.add(url);
    }

    return urls;
  }
}
```

### Widget สำหรับอัพโหลด

```dart
// lib/widgets/image_upload_widget.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import '../services/upload_service.dart';

class ImageUploadWidget extends StatefulWidget {
  const ImageUploadWidget({
    super.key,
    required this.userId,
    this.currentImageUrl,
    required this.onUploaded,
  });

  final String userId;
  final String? currentImageUrl;
  final Function(String url) onUploaded;

  @override
  State<ImageUploadWidget> createState() => _ImageUploadWidgetState();
}

class _ImageUploadWidgetState extends State<ImageUploadWidget> {
  File? _selectedImage;
  bool _isUploading = false;
  double _uploadProgress = 0;

  Future<void> _pickImage() async {
    final picker = ImagePicker();
    final pickedFile = await picker.pickImage(
      source: ImageSource.gallery,
      imageQuality: 80,
      maxWidth: 1000,
      maxHeight: 1000,
    );

    if (pickedFile != null) {
      setState(() {
        _selectedImage = File(pickedFile.path);
      });
      await _uploadImage();
    }
  }

  Future<void> _uploadImage() async {
    if (_selectedImage == null) return;

    setState(() {
      _isUploading = true;
      _uploadProgress = 0;
    });

    try {
      // Upload with progress
      final ref = FirebaseStorage.instance
          .ref()
          .child('users/${widget.userId}/profile.jpg');

      final uploadTask = ref.putFile(_selectedImage!);

      uploadTask.snapshotEvents.listen((snapshot) {
        if (snapshot.state == TaskState.running) {
          setState(() {
            _uploadProgress = snapshot.bytesTransferred / snapshot.totalBytes;
          });
        }
      });

      final taskSnapshot = await uploadTask;
      final url = await taskSnapshot.ref.getDownloadURL();

      widget.onUploaded(url);

      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('อัพโหลดสำเร็จ!'),
            backgroundColor: Colors.green,
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('อัพโหลดล้มเหลว: $e'),
            backgroundColor: Colors.red,
          ),
        );
      }
    } finally {
      if (mounted) {
        setState(() {
          _isUploading = false;
          _uploadProgress = 0;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _isUploading ? null : _pickImage,
      child: Stack(
        alignment: Alignment.center,
        children: [
          CircleAvatar(
            radius: 60,
            backgroundImage: _selectedImage != null
                ? FileImage(_selectedImage!)
                : (widget.currentImageUrl != null
                    ? NetworkImage(widget.currentImageUrl!) as ImageProvider
                    : null),
            child: _selectedImage == null && widget.currentImageUrl == null
                ? const Icon(Icons.person, size: 60)
                : null,
          ),
          if (_isUploading)
            CircularProgressIndicator(
              value: _uploadProgress > 0 ? _uploadProgress : null,
              strokeWidth: 4,
              backgroundColor: Colors.white.withOpacity(0.7),
            ),
          if (!_isUploading)
            Positioned(
              bottom: 0,
              right: 0,
              child: CircleAvatar(
                radius: 18,
                backgroundColor: Colors.blue,
                child: const Icon(
                  Icons.camera_alt,
                  size: 18,
                  color: Colors.white,
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 243: Download URLs

```dart
// lib/services/download_service.dart
import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class DownloadService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // ดึง Download URL จาก Path
  static Future<String> getDownloadUrl(String storagePath) async {
    final ref = _storage.ref().child(storagePath);
    return await ref.getDownloadURL();
  }

  // ดาวน์โหลดไฟล์ไปยัง Device
  static Future<File> downloadFile({
    required String storagePath,
    String? localFileName,
  }) async {
    final ref = _storage.ref().child(storagePath);
    final fileName = localFileName ?? path.basename(storagePath);

    final directory = await getApplicationDocumentsDirectory();
    final localFile = File('${directory.path}/$fileName');

    await ref.writeToFile(localFile);
    return localFile;
  }

  // ดาวน์โหลดเป็น Bytes
  static Future<Uint8List?> downloadBytes(String storagePath,
      {int maxSize = 10 * 1024 * 1024}) async {
    final ref = _storage.ref().child(storagePath);
    return await ref.getData(maxSize);
  }
}

import 'dart:typed_data';
// lib/widgets/storage_image_widget.dart
import 'package:flutter/material.dart';

class StorageImage extends StatelessWidget {
  const StorageImage({
    super.key,
    required this.url,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
    this.placeholder,
    this.errorWidget,
  });

  final String url;
  final double? width;
  final double? height;
  final BoxFit fit;
  final Widget? placeholder;
  final Widget? errorWidget;

  @override
  Widget build(BuildContext context) {
    return Image.network(
      url,
      width: width,
      height: height,
      fit: fit,
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) return child;
        return placeholder ??
            Center(
              child: CircularProgressIndicator(
                value: loadingProgress.expectedTotalBytes != null
                    ? loadingProgress.cumulativeBytesLoaded /
                        loadingProgress.expectedTotalBytes!
                    : null,
              ),
            );
      },
      errorBuilder: (context, error, stackTrace) {
        return errorWidget ??
            const Center(
              child: Icon(Icons.broken_image, color: Colors.grey, size: 48),
            );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 244: Delete Files

```dart
// lib/services/delete_service.dart
import 'package:firebase_storage/firebase_storage.dart';

class DeleteService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // ลบไฟล์เดียว
  static Future<void> deleteFile(String storagePath) async {
    try {
      await _storage.ref().child(storagePath).delete();
    } on FirebaseException catch (e) {
      if (e.code == 'object-not-found') {
        // ไฟล์ไม่มีอยู่ - ถือว่าสำเร็จ
        return;
      }
      throw Exception('ลบไฟล์ล้มเหลว: ${e.message}');
    }
  }

  // ลบไฟล์จาก URL
  static Future<void> deleteFileByUrl(String downloadUrl) async {
    try {
      final ref = _storage.refFromURL(downloadUrl);
      await ref.delete();
    } catch (e) {
      throw Exception('ลบไฟล์ล้มเหลว: $e');
    }
  }

  // ลบทุกไฟล์ใน Folder
  static Future<void> deleteFolder(String folderPath) async {
    try {
      final ref = _storage.ref().child(folderPath);
      final listResult = await ref.listAll();

      // ลบไฟล์ทั้งหมด
      await Future.wait(
        listResult.items.map((item) => item.delete()),
      );

      // ลบ subfolders (recursive)
      await Future.wait(
        listResult.prefixes.map((prefix) => deleteFolder(prefix.fullPath)),
      );
    } catch (e) {
      throw Exception('ลบ folder ล้มเหลว: $e');
    }
  }

  // ลบรูปโปรไฟล์เก่า แล้วอัพโหลดใหม่
  static Future<String> replaceProfilePhoto({
    required String userId,
    required File newPhoto,
    String? oldPhotoUrl,
  }) async {
    // ลบรูปเก่า (ถ้ามี)
    if (oldPhotoUrl != null) {
      try {
        await deleteFileByUrl(oldPhotoUrl);
      } catch (_) {
        // Ignore error ถ้าลบรูปเก่าไม่ได้
      }
    }

    // อัพโหลดรูปใหม่
    final ref = _storage.ref().child('users/$userId/profile.jpg');
    await ref.putFile(newPhoto);
    return await ref.getDownloadURL();
  }
}
```

---

## ขั้นตอนที่ 245: Progress Monitoring

```dart
// lib/widgets/upload_progress_widget.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:firebase_storage/firebase_storage.dart';

class UploadProgressWidget extends StatefulWidget {
  const UploadProgressWidget({super.key, required this.file, required this.path});
  final File file;
  final String path;

  @override
  State<UploadProgressWidget> createState() => _UploadProgressWidgetState();
}

class _UploadProgressWidgetState extends State<UploadProgressWidget> {
  UploadTask? _uploadTask;
  double _progress = 0;
  TaskState _state = TaskState.running;

  @override
  void initState() {
    super.initState();
    _startUpload();
  }

  void _startUpload() {
    final ref = FirebaseStorage.instance.ref().child(widget.path);
    _uploadTask = ref.putFile(widget.file);

    _uploadTask!.snapshotEvents.listen(
      (TaskSnapshot snapshot) {
        if (mounted) {
          setState(() {
            _state = snapshot.state;
            if (snapshot.totalBytes > 0) {
              _progress = snapshot.bytesTransferred / snapshot.totalBytes;
            }
          });
        }
      },
      onError: (error) {
        if (mounted) {
          setState(() => _state = TaskState.error);
        }
      },
    );
  }

  void _pauseUpload() {
    _uploadTask?.pause();
  }

  void _resumeUpload() {
    _uploadTask?.resume();
  }

  void _cancelUpload() {
    _uploadTask?.cancel();
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.all(16),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'อัพโหลด: ${widget.path.split('/').last}',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 12),
            LinearProgressIndicator(
              value: _state == TaskState.running ? _progress : null,
              backgroundColor: Colors.grey[300],
              valueColor: AlwaysStoppedAnimation<Color>(
                _state == TaskState.error ? Colors.red : Colors.blue,
              ),
            ),
            const SizedBox(height: 8),
            Row(
              children: [
                Text(
                  '${(_progress * 100).toStringAsFixed(0)}%',
                  style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                const SizedBox(width: 8),
                _buildStateChip(),
                const Spacer(),
                if (_state == TaskState.running)
                  IconButton(
                    icon: const Icon(Icons.pause),
                    onPressed: _pauseUpload,
                    tooltip: 'หยุดชั่วคราว',
                  ),
                if (_state == TaskState.paused)
                  IconButton(
                    icon: const Icon(Icons.play_arrow),
                    onPressed: _resumeUpload,
                    tooltip: 'ทำต่อ',
                  ),
                if (_state != TaskState.success)
                  IconButton(
                    icon: const Icon(Icons.cancel, color: Colors.red),
                    onPressed: _cancelUpload,
                    tooltip: 'ยกเลิก',
                  ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildStateChip() {
    final Map<TaskState, (String, Color)> stateMap = {
      TaskState.running: ('กำลังอัพโหลด', Colors.blue),
      TaskState.paused: ('หยุดชั่วคราว', Colors.orange),
      TaskState.success: ('สำเร็จ', Colors.green),
      TaskState.error: ('ล้มเหลว', Colors.red),
      TaskState.canceled: ('ยกเลิก', Colors.grey),
    };

    final (label, color) = stateMap[_state] ?? ('ไม่ทราบ', Colors.grey);

    return Chip(
      label: Text(label, style: TextStyle(color: Colors.white, fontSize: 12)),
      backgroundColor: color,
      materialTapTargetSize: MaterialTapTargetSize.shrinkWrap,
    );
  }
}

// lib/pages/multi_upload_page.dart - อัพโหลดหลายไฟล์พร้อมกัน
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';

class MultiUploadPage extends StatefulWidget {
  const MultiUploadPage({super.key});

  @override
  State<MultiUploadPage> createState() => _MultiUploadPageState();
}

class _MultiUploadPageState extends State<MultiUploadPage> {
  final List<File> _files = [];
  final Map<int, double> _progressMap = {};
  final Map<int, TaskState> _stateMap = {};

  Future<void> _pickImages() async {
    final picker = ImagePicker();
    final pickedFiles = await picker.pickMultiImage(imageQuality: 70);

    if (pickedFiles.isNotEmpty) {
      setState(() {
        _files.addAll(pickedFiles.map((f) => File(f.path)));
      });
    }
  }

  Future<void> _uploadAll() async {
    for (var i = 0; i < _files.length; i++) {
      final index = i;
      final file = _files[i];
      final fileName = file.path.split('/').last;
      final ref =
          FirebaseStorage.instance.ref().child('uploads/$fileName');
      final task = ref.putFile(file);

      task.snapshotEvents.listen((snapshot) {
        if (mounted) {
          setState(() {
            _stateMap[index] = snapshot.state;
            if (snapshot.totalBytes > 0) {
              _progressMap[index] =
                  snapshot.bytesTransferred / snapshot.totalBytes;
            }
          });
        }
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Multi Upload')),
      body: Column(
        children: [
          if (_files.isEmpty)
            const Expanded(
              child: Center(child: Text('เลือกรูปภาพเพื่ออัพโหลด')),
            )
          else
            Expanded(
              child: ListView.builder(
                itemCount: _files.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    leading: Image.file(
                      _files[index],
                      width: 50,
                      height: 50,
                      fit: BoxFit.cover,
                    ),
                    title: Text(_files[index].path.split('/').last),
                    subtitle: LinearProgressIndicator(
                      value: _progressMap[index],
                    ),
                    trailing: Icon(
                      _stateMap[index] == TaskState.success
                          ? Icons.check_circle
                          : Icons.pending,
                      color: _stateMap[index] == TaskState.success
                          ? Colors.green
                          : Colors.orange,
                    ),
                  );
                },
              ),
            ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _pickImages,
                    icon: const Icon(Icons.photo_library),
                    label: const Text('เลือกรูป'),
                  ),
                ),
                const SizedBox(width: 16),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _files.isEmpty ? null : _uploadAll,
                    icon: const Icon(Icons.upload),
                    label: const Text('อัพโหลดทั้งหมด'),
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

## ขั้นตอนที่ 246: Resumable Uploads

```dart
// lib/services/resumable_upload_service.dart
import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:shared_preferences/shared_preferences.dart';

class ResumableUploadService {
  static UploadTask? _currentTask;
  static const String _resumeTokenKey = 'upload_resume_token';

  // เริ่ม/ดำเนินการต่อ Upload
  static Future<String?> uploadWithResume({
    required File file,
    required String storagePath,
    Function(double)? onProgress,
  }) async {
    final ref = FirebaseStorage.instance.ref().child(storagePath);

    // สร้าง Upload Task
    _currentTask = ref.putFile(file);

    // ติดตาม Progress
    _currentTask!.snapshotEvents.listen((snapshot) {
      if (snapshot.state == TaskState.running) {
        final progress =
            snapshot.bytesTransferred / snapshot.totalBytes;
        onProgress?.call(progress);
      }
    });

    try {
      final snapshot = await _currentTask!;
      return await snapshot.ref.getDownloadURL();
    } on FirebaseException catch (e) {
      if (e.code == 'canceled') {
        return null; // Upload ถูกยกเลิก
      }
      rethrow;
    }
  }

  static void pause() {
    _currentTask?.pause();
  }

  static void resume() {
    _currentTask?.resume();
  }

  static void cancel() {
    _currentTask?.cancel();
    _currentTask = null;
  }

  static TaskState? get currentState => _currentTask?.snapshot.state;
}
```

---

## ขั้นตอนที่ 247: Metadata

```dart
// lib/services/metadata_service.dart
import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';

class MetadataService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // อัพโหลดพร้อม Metadata
  static Future<String> uploadWithMetadata({
    required File file,
    required String storagePath,
    required String userId,
    String? description,
    List<String>? tags,
  }) async {
    final ref = _storage.ref().child(storagePath);

    final metadata = SettableMetadata(
      contentType: 'image/jpeg',
      cacheControl: 'public, max-age=31536000', // Cache 1 ปี
      customMetadata: {
        'uploadedBy': userId,
        'uploadedAt': DateTime.now().toIso8601String(),
        if (description != null) 'description': description,
        if (tags != null) 'tags': tags.join(','),
      },
    );

    final snapshot = await ref.putFile(file, metadata);
    return await snapshot.ref.getDownloadURL();
  }

  // ดึง Metadata
  static Future<FullMetadata> getMetadata(String storagePath) async {
    return await _storage.ref().child(storagePath).getMetadata();
  }

  // อัพเดต Metadata
  static Future<FullMetadata> updateMetadata({
    required String storagePath,
    Map<String, String>? customMetadata,
    String? cacheControl,
  }) async {
    final metadata = SettableMetadata(
      customMetadata: customMetadata,
      cacheControl: cacheControl,
    );
    return await _storage.ref().child(storagePath).updateMetadata(metadata);
  }

  // แสดงข้อมูล Metadata
  static String formatMetadata(FullMetadata metadata) {
    return '''
Path: ${metadata.fullPath}
Name: ${metadata.name}
Size: ${_formatBytes(metadata.size ?? 0)}
Content Type: ${metadata.contentType ?? 'unknown'}
Created: ${metadata.timeCreated?.toString() ?? 'unknown'}
Updated: ${metadata.updated?.toString() ?? 'unknown'}
MD5 Hash: ${metadata.md5Hash ?? 'unknown'}
Custom: ${metadata.customMetadata?.toString() ?? 'none'}
''';
  }

  static String _formatBytes(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    if (bytes < 1024 * 1024 * 1024) {
      return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
    }
    return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(1)} GB';
  }
}
```

---

## ขั้นตอนที่ 248: Security Rules

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return request.auth.uid == userId;
    }
    
    function isImageFile() {
      return request.resource.contentType.matches('image/.*');
    }
    
    function isSmallEnough() {
      return request.resource.size < 5 * 1024 * 1024; // 5MB
    }
    
    function isAdmin() {
      return request.auth.token.admin == true;
    }
    
    // User profile photos
    match /users/{userId}/{allPaths=**} {
      allow read: if isAuthenticated();
      allow write: if isAuthenticated() && isOwner(userId)
        && isImageFile() && isSmallEnough();
      allow delete: if isAuthenticated() && (isOwner(userId) || isAdmin());
    }
    
    // Post images
    match /posts/{postId}/{allPaths=**} {
      allow read: if true; // public readable
      allow write: if isAuthenticated() && isImageFile()
        && request.resource.size < 10 * 1024 * 1024; // 10MB
      allow delete: if isAuthenticated();
    }
    
    // Public assets
    match /public/{allPaths=**} {
      allow read: if true;
      allow write: if isAdmin();
    }
  }
}
```

---

## ขั้นตอนที่ 249: Image Compression Before Upload

```dart
// lib/services/image_compression_service.dart
import 'dart:io';
import 'dart:typed_data';
import 'package:flutter_image_compress/flutter_image_compress.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class ImageCompressionService {
  // Compress ไฟล์ Image ก่อนอัพโหลด
  static Future<File> compressImage({
    required File file,
    int quality = 70,
    int? maxWidth,
    int? maxHeight,
  }) async {
    final dir = await getTemporaryDirectory();
    final fileName = path.basenameWithoutExtension(file.path);
    final targetPath = '${dir.path}/compressed_$fileName.jpg';

    final result = await FlutterImageCompress.compressAndGetFile(
      file.path,
      targetPath,
      quality: quality,
      minWidth: maxWidth ?? 1080,
      minHeight: maxHeight ?? 1080,
      format: CompressFormat.jpeg,
    );

    if (result == null) throw Exception('Compress ล้มเหลว');
    return File(result.path);
  }

  // Compress เป็น Bytes
  static Future<Uint8List> compressToBytes({
    required File file,
    int quality = 70,
    int maxWidth = 800,
    int maxHeight = 800,
  }) async {
    final result = await FlutterImageCompress.compressWithFile(
      file.path,
      quality: quality,
      minWidth: maxWidth,
      minHeight: maxHeight,
    );

    if (result == null) throw Exception('Compress ล้มเหลว');
    return result;
  }

  // Thumbnail
  static Future<Uint8List> createThumbnail({
    required File file,
    int size = 200,
    int quality = 50,
  }) async {
    return await compressToBytes(
      file: file,
      quality: quality,
      maxWidth: size,
      maxHeight: size,
    );
  }

  // แสดงขนาดไฟล์
  static String getFileSizeString(File file) {
    final bytes = file.lengthSync();
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) {
      return '${(bytes / 1024).toStringAsFixed(1)} KB';
    }
    return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
  }
}

// lib/services/smart_upload_service.dart - Upload พร้อม Compression
import 'package:firebase_storage/firebase_storage.dart';

class SmartUploadService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  static Future<String> uploadImage({
    required File originalFile,
    required String storagePath,
    int quality = 70,
    bool createThumbnail = true,
    Function(double)? onProgress,
  }) async {
    // Compress Image ก่อนอัพโหลด
    final compressedFile = await ImageCompressionService.compressImage(
      file: originalFile,
      quality: quality,
    );

    final originalSize =
        ImageCompressionService.getFileSizeString(originalFile);
    final compressedSize =
        ImageCompressionService.getFileSizeString(compressedFile);
    debugPrint('Original: $originalSize → Compressed: $compressedSize');

    // อัพโหลดรูปหลัก
    final ref = _storage.ref().child(storagePath);
    final task = ref.putFile(
      compressedFile,
      SettableMetadata(contentType: 'image/jpeg'),
    );

    task.snapshotEvents.listen((snapshot) {
      if (snapshot.state == TaskState.running && snapshot.totalBytes > 0) {
        onProgress?.call(snapshot.bytesTransferred / snapshot.totalBytes);
      }
    });

    final snapshot = await task;
    final url = await snapshot.ref.getDownloadURL();

    // อัพโหลด Thumbnail ถ้าต้องการ
    if (createThumbnail) {
      final thumbnailBytes = await ImageCompressionService.createThumbnail(
        file: originalFile,
      );
      final thumbPath = storagePath.replaceAll('.jpg', '_thumb.jpg');
      await _storage.ref().child(thumbPath).putData(
            thumbnailBytes,
            SettableMetadata(contentType: 'image/jpeg'),
          );
    }

    return url;
  }
}

import 'package:flutter/material.dart';
void debugPrint(String message) => print(message);
```

---

## ขั้นตอนที่ 250: Gallery Management

```dart
// lib/services/gallery_service.dart
import 'package:firebase_storage/firebase_storage.dart';

class GalleryItem {
  const GalleryItem({
    required this.ref,
    required this.name,
    required this.url,
    required this.size,
    required this.updatedAt,
  });

  final Reference ref;
  final String name;
  final String url;
  final int size;
  final DateTime? updatedAt;
}

class GalleryService {
  static final FirebaseStorage _storage = FirebaseStorage.instance;

  // แสดงรายการไฟล์ใน Folder
  static Future<List<GalleryItem>> listFiles(String folderPath) async {
    final ref = _storage.ref().child(folderPath);
    final result = await ref.listAll();

    final items = <GalleryItem>[];
    for (final item in result.items) {
      try {
        final url = await item.getDownloadURL();
        final metadata = await item.getMetadata();
        items.add(GalleryItem(
          ref: item,
          name: metadata.name ?? item.name,
          url: url,
          size: metadata.size ?? 0,
          updatedAt: metadata.updated,
        ));
      } catch (_) {
        // Skip ถ้าดึงข้อมูลไม่ได้
      }
    }
    return items;
  }
}

// lib/pages/gallery_page.dart
import 'package:flutter/material.dart';

class GalleryPage extends StatefulWidget {
  const GalleryPage({super.key, required this.userId});
  final String userId;

  @override
  State<GalleryPage> createState() => _GalleryPageState();
}

class _GalleryPageState extends State<GalleryPage> {
  late Future<List<GalleryItem>> _galleryFuture;

  @override
  void initState() {
    super.initState();
    _loadGallery();
  }

  void _loadGallery() {
    _galleryFuture =
        GalleryService.listFiles('users/${widget.userId}/gallery');
  }

  Future<void> _deleteItem(GalleryItem item) async {
    final confirm = await showDialog<bool>(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('ลบรูปภาพ'),
        content: Text('ต้องการลบ "${item.name}" หรือไม่?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx, false),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            style: ElevatedButton.styleFrom(backgroundColor: Colors.red),
            onPressed: () => Navigator.pop(ctx, true),
            child: const Text('ลบ', style: TextStyle(color: Colors.white)),
          ),
        ],
      ),
    );

    if (confirm == true) {
      await item.ref.delete();
      setState(() => _loadGallery());
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(
            content: Text('ลบรูปภาพแล้ว'),
            backgroundColor: Colors.green,
          ),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('แกลเลอรี่'),
        actions: [
          IconButton(
            icon: const Icon(Icons.add_photo_alternate),
            onPressed: () async {
              // Pick and upload
              final picker = ImagePicker();
              final image = await picker.pickImage(source: ImageSource.gallery);
              if (image != null) {
                final file = File(image.path);
                final fileName = image.name;
                await SmartUploadService.uploadImage(
                  originalFile: file,
                  storagePath: 'users/${widget.userId}/gallery/$fileName',
                );
                setState(() => _loadGallery());
              }
            },
          ),
        ],
      ),
      body: FutureBuilder<List<GalleryItem>>(
        future: _galleryFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          }

          final items = snapshot.data ?? [];

          if (items.isEmpty) {
            return const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.photo_library, size: 80, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('ยังไม่มีรูปภาพ'),
                ],
              ),
            );
          }

          return GridView.builder(
            padding: const EdgeInsets.all(8),
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 3,
              crossAxisSpacing: 4,
              mainAxisSpacing: 4,
            ),
            itemCount: items.length,
            itemBuilder: (context, index) {
              final item = items[index];
              return GestureDetector(
                onLongPress: () => _deleteItem(item),
                child: Stack(
                  fit: StackFit.expand,
                  children: [
                    Image.network(
                      item.url,
                      fit: BoxFit.cover,
                      loadingBuilder: (context, child, progress) {
                        if (progress == null) return child;
                        return const Center(
                          child: CircularProgressIndicator(strokeWidth: 2),
                        );
                      },
                    ),
                    Positioned(
                      bottom: 0,
                      left: 0,
                      right: 0,
                      child: Container(
                        padding: const EdgeInsets.all(4),
                        color: Colors.black54,
                        child: Text(
                          item.name,
                          style: const TextStyle(
                            color: Colors.white,
                            fontSize: 10,
                          ),
                          overflow: TextOverflow.ellipsis,
                        ),
                      ),
                    ),
                  ],
                ),
              );
            },
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
- **Uploading Files**: การอัพโหลดรูปภาพและเอกสาร
- **Download URLs**: การดึง URL สำหรับแสดงไฟล์
- **Delete Files**: การลบไฟล์จาก Storage
- **Progress Monitoring**: การติดตามความคืบหน้า
- **Resumable Uploads**: การหยุดและดำเนินการต่อ
- **Metadata**: การจัดการข้อมูลเพิ่มเติมของไฟล์
- **Security Rules**: การกำหนดสิทธิ์การเข้าถึง
- **Image Compression**: การบีบอัดรูปภาพก่อนอัพโหลด

## แบบฝึกหัด

1. สร้าง Profile Photo Upload ที่ Compress รูปก่อนอัพโหลด
2. สร้าง Image Gallery ที่แสดงรูปภาพจาก Storage
3. Implement Upload Queue ที่อัพโหลดทีละไฟล์
4. สร้าง Document Upload ที่รองรับหลาย File Types
5. เพิ่ม Preview ก่อนอัพโหลด

---

[⬅️ Part 24](part_24.md) | [Part 26 ➡️](part_26.md)

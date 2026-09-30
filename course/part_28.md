# Part 28: Camera & Media
## ขั้นตอนที่ 271-280

---

## สารบัญ
1. [camera Package Setup](#camera-package-setup)
2. [image_picker](#image_picker)
3. [Taking Photos](#taking-photos)
4. [Taking Videos](#taking-videos)
5. [Video Player](#video-player)
6. [Audio Recording](#audio-recording)
7. [Gallery Picker](#gallery-picker)
8. [Image Cropping](#image-cropping)
9. [Compression](#compression)
10. [File Management & Permissions](#file-management--permissions)

---

## ขั้นตอนที่ 271: camera Package Setup

```yaml
# pubspec.yaml
dependencies:
  camera: ^0.10.5+9
  image_picker: ^1.0.7
  video_player: ^2.8.3
  record: ^5.1.1
  image_cropper: ^7.0.3
  flutter_image_compress: ^2.1.0
  path_provider: ^2.1.2
  path: ^1.8.3
  permission_handler: ^11.3.0
  mime: ^1.0.5
```

### ตั้งค่า Android

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
```

### ตั้งค่า iOS

```xml
<!-- ios/Runner/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>แอปนี้ต้องการเข้าถึงกล้องเพื่อถ่ายรูป</string>
<key>NSMicrophoneUsageDescription</key>
<string>แอปนี้ต้องการเข้าถึงไมโครโฟนเพื่อบันทึกเสียง</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>แอปนี้ต้องการเข้าถึงคลังรูปภาพ</string>
```

---

## ขั้นตอนที่ 272: image_picker

```dart
// lib/services/media_picker_service.dart
import 'dart:io';
import 'package:image_picker/image_picker.dart';

class MediaPickerService {
  static final ImagePicker _picker = ImagePicker();

  // เลือกรูปจาก Gallery
  static Future<File?> pickImageFromGallery({
    int imageQuality = 80,
    double? maxWidth,
    double? maxHeight,
  }) async {
    final XFile? picked = await _picker.pickImage(
      source: ImageSource.gallery,
      imageQuality: imageQuality,
      maxWidth: maxWidth,
      maxHeight: maxHeight,
    );
    return picked != null ? File(picked.path) : null;
  }

  // ถ่ายรูปด้วยกล้อง
  static Future<File?> takePhotoWithCamera({
    int imageQuality = 80,
    CameraDevice preferredCamera = CameraDevice.rear,
  }) async {
    final XFile? taken = await _picker.pickImage(
      source: ImageSource.camera,
      imageQuality: imageQuality,
      preferredCameraDevice: preferredCamera,
    );
    return taken != null ? File(taken.path) : null;
  }

  // เลือกหลายรูปจาก Gallery
  static Future<List<File>> pickMultipleImages({int imageQuality = 80}) async {
    final List<XFile> picked = await _picker.pickMultiImage(
      imageQuality: imageQuality,
    );
    return picked.map((f) => File(f.path)).toList();
  }

  // เลือกวิดีโอจาก Gallery
  static Future<File?> pickVideoFromGallery({
    Duration? maxDuration,
  }) async {
    final XFile? picked = await _picker.pickVideo(
      source: ImageSource.gallery,
      maxDuration: maxDuration,
    );
    return picked != null ? File(picked.path) : null;
  }

  // ถ่ายวิดีโอด้วยกล้อง
  static Future<File?> recordVideoWithCamera({
    Duration? maxDuration,
  }) async {
    final XFile? recorded = await _picker.pickVideo(
      source: ImageSource.camera,
      maxDuration: maxDuration ?? const Duration(minutes: 1),
    );
    return recorded != null ? File(recorded.path) : null;
  }

  // เลือกไฟล์ Media (ทั้งรูปและวิดีโอ)
  static Future<List<File>> pickMultipleMedia() async {
    final List<XFile> picked = await _picker.pickMultipleMedia();
    return picked.map((f) => File(f.path)).toList();
  }
}
```

```dart
// lib/widgets/media_picker_widget.dart
import 'dart:io';
import 'package:flutter/material.dart';
import '../services/media_picker_service.dart';

class MediaPickerWidget extends StatefulWidget {
  const MediaPickerWidget({super.key, this.onFilePicked});

  final Function(File)? onFilePicked;

  @override
  State<MediaPickerWidget> createState() => _MediaPickerWidgetState();
}

class _MediaPickerWidgetState extends State<MediaPickerWidget> {
  File? _selectedFile;
  bool _isVideo = false;

  void _showPickerDialog() {
    showModalBottomSheet(
      context: context,
      builder: (ctx) => SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              const Text(
                'เลือกมีเดีย',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 16),
              ListTile(
                leading: const Icon(Icons.photo_camera, color: Colors.blue),
                title: const Text('ถ่ายรูป'),
                onTap: () async {
                  Navigator.pop(ctx);
                  final file = await MediaPickerService.takePhotoWithCamera();
                  if (file != null) {
                    setState(() {
                      _selectedFile = file;
                      _isVideo = false;
                    });
                    widget.onFilePicked?.call(file);
                  }
                },
              ),
              ListTile(
                leading: const Icon(Icons.videocam, color: Colors.red),
                title: const Text('ถ่ายวิดีโอ'),
                onTap: () async {
                  Navigator.pop(ctx);
                  final file = await MediaPickerService.recordVideoWithCamera();
                  if (file != null) {
                    setState(() {
                      _selectedFile = file;
                      _isVideo = true;
                    });
                    widget.onFilePicked?.call(file);
                  }
                },
              ),
              ListTile(
                leading: const Icon(Icons.photo_library, color: Colors.green),
                title: const Text('เลือกจาก Gallery'),
                onTap: () async {
                  Navigator.pop(ctx);
                  final file = await MediaPickerService.pickImageFromGallery();
                  if (file != null) {
                    setState(() {
                      _selectedFile = file;
                      _isVideo = false;
                    });
                    widget.onFilePicked?.call(file);
                  }
                },
              ),
            ],
          ),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _showPickerDialog,
      child: Container(
        height: 200,
        decoration: BoxDecoration(
          border: Border.all(
            color: Colors.grey.shade300,
            style: BorderStyle.solid,
          ),
          borderRadius: BorderRadius.circular(12),
        ),
        child: _selectedFile != null
            ? ClipRRect(
                borderRadius: BorderRadius.circular(12),
                child: _isVideo
                    ? const Center(
                        child: Icon(Icons.video_file, size: 80, color: Colors.grey),
                      )
                    : Image.file(_selectedFile!, fit: BoxFit.cover),
              )
            : const Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.add_photo_alternate, size: 48, color: Colors.grey),
                  SizedBox(height: 8),
                  Text('แตะเพื่อเลือกรูปหรือวิดีโอ',
                      style: TextStyle(color: Colors.grey)),
                ],
              ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 273: Taking Photos

```dart
// lib/pages/camera_page.dart
import 'package:flutter/material.dart';
import 'package:camera/camera.dart';
import 'dart:io';

class CameraPage extends StatefulWidget {
  const CameraPage({super.key});

  @override
  State<CameraPage> createState() => _CameraPageState();
}

class _CameraPageState extends State<CameraPage>
    with WidgetsBindingObserver {
  CameraController? _controller;
  List<CameraDescription>? _cameras;
  int _currentCameraIndex = 0;
  bool _isInitialized = false;
  bool _isTakingPhoto = false;
  FlashMode _flashMode = FlashMode.auto;
  double _currentZoom = 1.0;
  double _minZoom = 1.0;
  double _maxZoom = 1.0;
  File? _lastPhoto;

  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    _initializeCamera();
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    _controller?.dispose();
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    final controller = _controller;
    if (controller == null || !controller.value.isInitialized) return;

    if (state == AppLifecycleState.inactive) {
      controller.dispose();
    } else if (state == AppLifecycleState.resumed) {
      _initializeCamera();
    }
  }

  Future<void> _initializeCamera() async {
    _cameras = await availableCameras();
    if (_cameras!.isEmpty) return;

    await _setupCamera(_cameras![_currentCameraIndex]);
  }

  Future<void> _setupCamera(CameraDescription camera) async {
    final controller = CameraController(
      camera,
      ResolutionPreset.high,
      enableAudio: false,
      imageFormatGroup: ImageFormatGroup.jpeg,
    );

    await controller.initialize();
    if (!mounted) return;

    _maxZoom = await controller.getMaxZoomLevel();
    _minZoom = await controller.getMinZoomLevel();

    setState(() {
      _controller = controller;
      _isInitialized = true;
    });
  }

  Future<void> _takePicture() async {
    if (_controller == null || !_controller!.value.isInitialized) return;
    if (_isTakingPhoto) return;

    setState(() => _isTakingPhoto = true);

    try {
      final XFile photo = await _controller!.takePicture();
      setState(() => _lastPhoto = File(photo.path));

      if (mounted) {
        // แสดง Preview
        showDialog(
          context: context,
          builder: (ctx) => AlertDialog(
            content: Image.file(File(photo.path)),
            actions: [
              TextButton(
                onPressed: () {
                  Navigator.pop(ctx);
                  setState(() => _lastPhoto = null);
                },
                child: const Text('ถ่ายใหม่'),
              ),
              ElevatedButton(
                onPressed: () {
                  Navigator.pop(ctx);
                  Navigator.pop(context, File(photo.path));
                },
                child: const Text('ใช้รูปนี้'),
              ),
            ],
          ),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('ถ่ายรูปล้มเหลว: $e')),
        );
      }
    } finally {
      if (mounted) setState(() => _isTakingPhoto = false);
    }
  }

  void _switchCamera() async {
    if (_cameras == null || _cameras!.length < 2) return;
    _currentCameraIndex = (_currentCameraIndex + 1) % _cameras!.length;
    await _setupCamera(_cameras![_currentCameraIndex]);
  }

  void _setFlashMode() {
    final modes = [FlashMode.auto, FlashMode.always, FlashMode.off];
    final nextIndex = (modes.indexOf(_flashMode) + 1) % modes.length;
    setState(() => _flashMode = modes[nextIndex]);
    _controller?.setFlashMode(_flashMode);
  }

  IconData _getFlashIcon() {
    switch (_flashMode) {
      case FlashMode.auto:
        return Icons.flash_auto;
      case FlashMode.always:
        return Icons.flash_on;
      case FlashMode.off:
        return Icons.flash_off;
      default:
        return Icons.flash_auto;
    }
  }

  @override
  Widget build(BuildContext context) {
    if (!_isInitialized || _controller == null) {
      return const Scaffold(
        backgroundColor: Colors.black,
        body: Center(child: CircularProgressIndicator(color: Colors.white)),
      );
    }

    return Scaffold(
      backgroundColor: Colors.black,
      body: Stack(
        children: [
          // Camera Preview
          Center(
            child: GestureDetector(
              onScaleUpdate: (details) async {
                double zoom = (_currentZoom * details.scale)
                    .clamp(_minZoom, _maxZoom);
                await _controller!.setZoomLevel(zoom);
                setState(() => _currentZoom = zoom);
              },
              child: CameraPreview(
                _controller!,
                child: const SizedBox.expand(),
              ),
            ),
          ),

          // Top Controls
          Positioned(
            top: MediaQuery.of(context).padding.top + 8,
            left: 0,
            right: 0,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                IconButton(
                  icon: Icon(_getFlashIcon(),
                      color: Colors.white, size: 30),
                  onPressed: _setFlashMode,
                ),
                const Spacer(),
                IconButton(
                  icon: const Icon(Icons.flip_camera_ios,
                      color: Colors.white, size: 30),
                  onPressed: _switchCamera,
                ),
              ],
            ),
          ),

          // Bottom Controls
          Positioned(
            bottom: MediaQuery.of(context).padding.bottom + 24,
            left: 0,
            right: 0,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: [
                // Thumbnail
                if (_lastPhoto != null)
                  ClipRRect(
                    borderRadius: BorderRadius.circular(8),
                    child: Image.file(
                      _lastPhoto!,
                      width: 60,
                      height: 60,
                      fit: BoxFit.cover,
                    ),
                  )
                else
                  const SizedBox(width: 60),

                // Shutter Button
                GestureDetector(
                  onTap: _isTakingPhoto ? null : _takePicture,
                  child: Container(
                    width: 80,
                    height: 80,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      border: Border.all(color: Colors.white, width: 4),
                      color: _isTakingPhoto
                          ? Colors.grey
                          : Colors.white.withOpacity(0.8),
                    ),
                    child: _isTakingPhoto
                        ? const CircularProgressIndicator(color: Colors.grey)
                        : null,
                  ),
                ),

                // Gallery Button
                IconButton(
                  icon: const Icon(Icons.photo_library,
                      color: Colors.white, size: 40),
                  onPressed: () async {
                    final file = await MediaPickerService.pickImageFromGallery();
                    if (file != null && mounted) {
                      Navigator.pop(context, file);
                    }
                  },
                ),
              ],
            ),
          ),

          // Zoom Indicator
          if (_currentZoom > 1.0)
            Positioned(
              top: MediaQuery.of(context).size.height * 0.4,
              right: 16,
              child: Container(
                padding: const EdgeInsets.symmetric(
                    horizontal: 8, vertical: 4),
                decoration: BoxDecoration(
                  color: Colors.black54,
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Text(
                  '${_currentZoom.toStringAsFixed(1)}x',
                  style: const TextStyle(color: Colors.white),
                ),
              ),
            ),

          // Back Button
          Positioned(
            top: MediaQuery.of(context).padding.top + 8,
            left: 8,
            child: IconButton(
              icon: const Icon(Icons.arrow_back, color: Colors.white),
              onPressed: () => Navigator.pop(context),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 274: Taking Videos

```dart
// lib/pages/video_camera_page.dart
import 'package:flutter/material.dart';
import 'package:camera/camera.dart';
import 'dart:io';

class VideoCameraPage extends StatefulWidget {
  const VideoCameraPage({super.key});

  @override
  State<VideoCameraPage> createState() => _VideoCameraPageState();
}

class _VideoCameraPageState extends State<VideoCameraPage> {
  CameraController? _controller;
  bool _isRecording = false;
  bool _isInitialized = false;
  int _recordingSeconds = 0;
  Timer? _timer;

  @override
  void initState() {
    super.initState();
    _initCamera();
  }

  @override
  void dispose() {
    _controller?.dispose();
    _timer?.cancel();
    super.dispose();
  }

  Future<void> _initCamera() async {
    final cameras = await availableCameras();
    if (cameras.isEmpty) return;

    final controller = CameraController(
      cameras.first,
      ResolutionPreset.high,
      enableAudio: true,
    );

    await controller.initialize();
    if (mounted) {
      setState(() {
        _controller = controller;
        _isInitialized = true;
      });
    }
  }

  Future<void> _startRecording() async {
    if (_controller == null) return;
    await _controller!.startVideoRecording();
    setState(() {
      _isRecording = true;
      _recordingSeconds = 0;
    });

    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      if (mounted) {
        setState(() => _recordingSeconds++);
      }
    });
  }

  Future<void> _stopRecording() async {
    if (_controller == null || !_isRecording) return;
    _timer?.cancel();

    final XFile video = await _controller!.stopVideoRecording();
    setState(() => _isRecording = false);

    if (mounted) {
      Navigator.pop(context, File(video.path));
    }
  }

  String _formatDuration(int seconds) {
    final mins = seconds ~/ 60;
    final secs = seconds % 60;
    return '${mins.toString().padLeft(2, '0')}:${secs.toString().padLeft(2, '0')}';
  }

  @override
  Widget build(BuildContext context) {
    if (!_isInitialized || _controller == null) {
      return const Scaffold(
        backgroundColor: Colors.black,
        body: Center(child: CircularProgressIndicator()),
      );
    }

    return Scaffold(
      backgroundColor: Colors.black,
      body: Stack(
        children: [
          CameraPreview(_controller!),

          if (_isRecording)
            Positioned(
              top: MediaQuery.of(context).padding.top + 16,
              left: 16,
              child: Row(
                children: [
                  const Icon(Icons.circle, color: Colors.red, size: 12),
                  const SizedBox(width: 4),
                  Text(
                    _formatDuration(_recordingSeconds),
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ],
              ),
            ),

          Positioned(
            bottom: MediaQuery.of(context).padding.bottom + 40,
            left: 0,
            right: 0,
            child: Center(
              child: GestureDetector(
                onTap: _isRecording ? _stopRecording : _startRecording,
                child: Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    border: Border.all(color: Colors.white, width: 4),
                    color: _isRecording ? Colors.red : Colors.white.withOpacity(0.3),
                  ),
                  child: _isRecording
                      ? const Icon(Icons.stop, color: Colors.white, size: 36)
                      : const Icon(Icons.fiber_manual_record,
                          color: Colors.red, size: 36),
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

import 'dart:async';
```

---

## ขั้นตอนที่ 275: Video Player

```dart
// lib/pages/video_player_page.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:video_player/video_player.dart';

class VideoPlayerPage extends StatefulWidget {
  const VideoPlayerPage({
    super.key,
    required this.videoSource,
    this.isNetwork = false,
  });

  final String videoSource;
  final bool isNetwork;

  @override
  State<VideoPlayerPage> createState() => _VideoPlayerPageState();
}

class _VideoPlayerPageState extends State<VideoPlayerPage> {
  late VideoPlayerController _controller;
  bool _isInitialized = false;
  bool _isPlaying = false;
  bool _showControls = true;

  @override
  void initState() {
    super.initState();
    _initializePlayer();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _initializePlayer() async {
    if (widget.isNetwork) {
      _controller = VideoPlayerController.networkUrl(
        Uri.parse(widget.videoSource),
      );
    } else {
      _controller = VideoPlayerController.file(
        File(widget.videoSource),
      );
    }

    await _controller.initialize();
    _controller.addListener(() {
      if (mounted) setState(() {});
    });

    setState(() => _isInitialized = true);
  }

  void _togglePlayPause() {
    if (_controller.value.isPlaying) {
      _controller.pause();
    } else {
      _controller.play();
    }
    setState(() => _isPlaying = !_isPlaying);
  }

  String _formatDuration(Duration duration) {
    final minutes = duration.inMinutes.remainder(60).toString().padLeft(2, '0');
    final seconds = duration.inSeconds.remainder(60).toString().padLeft(2, '0');
    return '$minutes:$seconds';
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        backgroundColor: Colors.black,
        foregroundColor: Colors.white,
        title: const Text('วิดีโอ'),
      ),
      body: _isInitialized
          ? GestureDetector(
              onTap: () => setState(() => _showControls = !_showControls),
              child: Stack(
                alignment: Alignment.center,
                children: [
                  AspectRatio(
                    aspectRatio: _controller.value.aspectRatio,
                    child: VideoPlayer(_controller),
                  ),
                  if (_showControls) _buildControls(),
                ],
              ),
            )
          : const Center(child: CircularProgressIndicator()),
      bottomNavigationBar: _isInitialized
          ? Container(
              color: Colors.black,
              padding: const EdgeInsets.all(8),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  VideoProgressIndicator(
                    _controller,
                    allowScrubbing: true,
                    colors: const VideoProgressColors(
                      playedColor: Colors.red,
                      backgroundColor: Colors.grey,
                      bufferedColor: Colors.white30,
                    ),
                  ),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        _formatDuration(_controller.value.position),
                        style: const TextStyle(color: Colors.white),
                      ),
                      Text(
                        _formatDuration(_controller.value.duration),
                        style: const TextStyle(color: Colors.grey),
                      ),
                    ],
                  ),
                ],
              ),
            )
          : null,
    );
  }

  Widget _buildControls() {
    return Container(
      color: Colors.black45,
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          IconButton(
            icon: const Icon(Icons.replay_10, color: Colors.white, size: 40),
            onPressed: () {
              _controller.seekTo(
                _controller.value.position - const Duration(seconds: 10),
              );
            },
          ),
          const SizedBox(width: 16),
          IconButton(
            icon: Icon(
              _controller.value.isPlaying ? Icons.pause : Icons.play_arrow,
              color: Colors.white,
              size: 60,
            ),
            onPressed: _togglePlayPause,
          ),
          const SizedBox(width: 16),
          IconButton(
            icon: const Icon(Icons.forward_10, color: Colors.white, size: 40),
            onPressed: () {
              _controller.seekTo(
                _controller.value.position + const Duration(seconds: 10),
              );
            },
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 276: Audio Recording

```dart
// lib/services/audio_recorder_service.dart
import 'dart:io';
import 'package:record/record.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

class AudioRecorderService {
  static final AudioRecorder _recorder = AudioRecorder();
  static String? _recordingPath;

  // ตรวจสอบ Permission
  static Future<bool> checkPermission() async {
    return await _recorder.hasPermission();
  }

  // เริ่มบันทึก
  static Future<void> startRecording() async {
    if (!await checkPermission()) {
      throw Exception('ไม่ได้รับอนุญาตให้เข้าถึงไมโครโฟน');
    }

    final directory = await getApplicationDocumentsDirectory();
    final fileName = 'audio_${DateTime.now().millisecondsSinceEpoch}.m4a';
    _recordingPath = path.join(directory.path, fileName);

    await _recorder.start(
      RecordConfig(
        encoder: AudioEncoder.aacLc,
        bitRate: 128000,
        sampleRate: 44100,
      ),
      path: _recordingPath!,
    );
  }

  // หยุดบันทึก
  static Future<File?> stopRecording() async {
    final filePath = await _recorder.stop();
    if (filePath != null) {
      return File(filePath);
    }
    return null;
  }

  // หยุดชั่วคราว
  static Future<void> pauseRecording() async {
    await _recorder.pause();
  }

  // ดำเนินการต่อ
  static Future<void> resumeRecording() async {
    await _recorder.resume();
  }

  // ยกเลิก
  static Future<void> cancelRecording() async {
    await _recorder.cancel();
  }

  // ตรวจสอบสถานะ
  static Future<RecordState> getState() async {
    return await _recorder.getState();
  }

  // Stream Amplitude (สำหรับ Waveform)
  static Stream<Amplitude> getAmplitudeStream() {
    return _recorder.onAmplitudeChanged(const Duration(milliseconds: 100));
  }

  static void dispose() {
    _recorder.dispose();
  }
}

// lib/widgets/audio_recorder_widget.dart
import 'package:flutter/material.dart';

class AudioRecorderWidget extends StatefulWidget {
  const AudioRecorderWidget({super.key, this.onRecorded});

  final Function(File)? onRecorded;

  @override
  State<AudioRecorderWidget> createState() => _AudioRecorderWidgetState();
}

class _AudioRecorderWidgetState extends State<AudioRecorderWidget> {
  RecordState _state = RecordState.stop;
  int _duration = 0;
  Timer? _timer;
  double _amplitude = 0;

  @override
  void dispose() {
    _timer?.cancel();
    AudioRecorderService.dispose();
    super.dispose();
  }

  Future<void> _toggleRecording() async {
    if (_state == RecordState.stop || _state == RecordState.pause) {
      await _startRecording();
    } else if (_state == RecordState.record) {
      await _stopRecording();
    }
  }

  Future<void> _startRecording() async {
    await AudioRecorderService.startRecording();
    setState(() {
      _state = RecordState.record;
      _duration = 0;
    });

    _timer = Timer.periodic(const Duration(seconds: 1), (_) {
      if (mounted) setState(() => _duration++);
    });

    // Amplitude stream
    AudioRecorderService.getAmplitudeStream().listen((amp) {
      if (mounted) setState(() => _amplitude = amp.current);
    });
  }

  Future<void> _stopRecording() async {
    _timer?.cancel();
    final file = await AudioRecorderService.stopRecording();
    setState(() => _state = RecordState.stop);

    if (file != null) {
      widget.onRecorded?.call(file);
    }
  }

  String _formatDuration(int seconds) {
    final mins = (seconds ~/ 60).toString().padLeft(2, '0');
    final secs = (seconds % 60).toString().padLeft(2, '0');
    return '$mins:$secs';
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(16),
      ),
      child: Column(
        children: [
          // Waveform Visualization
          Container(
            height: 60,
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(8),
            ),
            child: Center(
              child: _state == RecordState.record
                  ? AnimatedContainer(
                      duration: const Duration(milliseconds: 100),
                      width: (_amplitude + 160) / 2,
                      height: (_amplitude + 160) / 2,
                      decoration: const BoxDecoration(
                        color: Colors.red,
                        shape: BoxShape.circle,
                      ),
                    )
                  : const Icon(Icons.mic, size: 40, color: Colors.grey),
            ),
          ),
          const SizedBox(height: 16),

          // Duration
          Text(
            _formatDuration(_duration),
            style: const TextStyle(
              fontSize: 32,
              fontWeight: FontWeight.bold,
              fontFamily: 'monospace',
            ),
          ),
          const SizedBox(height: 16),

          // Controls
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              FloatingActionButton(
                heroTag: 'record',
                onPressed: _toggleRecording,
                backgroundColor:
                    _state == RecordState.record ? Colors.red : Colors.blue,
                child: Icon(
                  _state == RecordState.record ? Icons.stop : Icons.mic,
                ),
              ),
              if (_state == RecordState.record) ...[
                const SizedBox(width: 16),
                FloatingActionButton(
                  heroTag: 'pause',
                  mini: true,
                  onPressed: () async {
                    await AudioRecorderService.pauseRecording();
                    setState(() => _state = RecordState.pause);
                    _timer?.cancel();
                  },
                  child: const Icon(Icons.pause),
                ),
              ],
            ],
          ),
        ],
      ),
    );
  }
}

import 'dart:async';
```

---

## ขั้นตอนที่ 277: Gallery Picker

```dart
// lib/pages/gallery_picker_page.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';

class GalleryPickerPage extends StatefulWidget {
  const GalleryPickerPage({super.key});

  @override
  State<GalleryPickerPage> createState() => _GalleryPickerPageState();
}

class _GalleryPickerPageState extends State<GalleryPickerPage> {
  final List<File> _selectedFiles = [];
  bool _isLoading = false;

  Future<void> _pickMultipleImages() async {
    setState(() => _isLoading = true);
    try {
      final files = await MediaPickerService.pickMultipleImages();
      setState(() => _selectedFiles.addAll(files));
    } finally {
      setState(() => _isLoading = false);
    }
  }

  Future<void> _pickMedia() async {
    setState(() => _isLoading = true);
    try {
      final files = await MediaPickerService.pickMultipleMedia();
      setState(() => _selectedFiles.addAll(files));
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('เลือกรูป (${_selectedFiles.length})'),
        actions: [
          if (_selectedFiles.isNotEmpty)
            TextButton(
              onPressed: () => Navigator.pop(context, _selectedFiles),
              child: const Text('ใช้'),
            ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8),
            child: Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _isLoading ? null : _pickMultipleImages,
                    icon: const Icon(Icons.photo),
                    label: const Text('เลือกรูปภาพ'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _isLoading ? null : _pickMedia,
                    icon: const Icon(Icons.perm_media),
                    label: const Text('เลือก Media'),
                  ),
                ),
              ],
            ),
          ),
          Expanded(
            child: _isLoading
                ? const Center(child: CircularProgressIndicator())
                : _selectedFiles.isEmpty
                    ? const Center(child: Text('ยังไม่ได้เลือกรูป'))
                    : GridView.builder(
                        padding: const EdgeInsets.all(8),
                        gridDelegate:
                            const SliverGridDelegateWithFixedCrossAxisCount(
                          crossAxisCount: 3,
                          crossAxisSpacing: 4,
                          mainAxisSpacing: 4,
                        ),
                        itemCount: _selectedFiles.length,
                        itemBuilder: (context, index) {
                          return Stack(
                            fit: StackFit.expand,
                            children: [
                              Image.file(
                                _selectedFiles[index],
                                fit: BoxFit.cover,
                              ),
                              Positioned(
                                top: 4,
                                right: 4,
                                child: GestureDetector(
                                  onTap: () => setState(
                                      () => _selectedFiles.removeAt(index)),
                                  child: Container(
                                    decoration: const BoxDecoration(
                                      color: Colors.red,
                                      shape: BoxShape.circle,
                                    ),
                                    child: const Icon(
                                      Icons.close,
                                      color: Colors.white,
                                      size: 18,
                                    ),
                                  ),
                                ),
                              ),
                            ],
                          );
                        },
                      ),
          ),
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 278: Image Cropping

```dart
// lib/services/image_cropper_service.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_cropper/image_cropper.dart';

class ImageCropperService {
  static Future<File?> cropImage({
    required File imageFile,
    CropAspectRatio? aspectRatio,
    List<CropAspectRatioPreset>? presets,
  }) async {
    final croppedFile = await ImageCropper().cropImage(
      sourcePath: imageFile.path,
      aspectRatio: aspectRatio,
      aspectRatioPresets: presets ??
          [
            CropAspectRatioPreset.square,
            CropAspectRatioPreset.ratio3x2,
            CropAspectRatioPreset.original,
            CropAspectRatioPreset.ratio4x3,
            CropAspectRatioPreset.ratio16x9,
          ],
      uiSettings: [
        AndroidUiSettings(
          toolbarTitle: 'ครอปรูปภาพ',
          toolbarColor: Colors.deepPurple,
          toolbarWidgetColor: Colors.white,
          initAspectRatio: CropAspectRatioPreset.original,
          lockAspectRatio: false,
          activeControlsWidgetColor: Colors.deepPurple,
        ),
        IOSUiSettings(
          title: 'ครอปรูปภาพ',
          cancelButtonTitle: 'ยกเลิก',
          doneButtonTitle: 'ตกลง',
        ),
      ],
    );

    return croppedFile != null ? File(croppedFile.path) : null;
  }

  // ครอปสำหรับโปรไฟล์ (1:1)
  static Future<File?> cropForProfile(File imageFile) async {
    return cropImage(
      imageFile: imageFile,
      aspectRatio: const CropAspectRatio(ratioX: 1, ratioY: 1),
      presets: [CropAspectRatioPreset.square],
    );
  }

  // ครอปสำหรับ Banner (16:9)
  static Future<File?> cropForBanner(File imageFile) async {
    return cropImage(
      imageFile: imageFile,
      aspectRatio: const CropAspectRatio(ratioX: 16, ratioY: 9),
      presets: [CropAspectRatioPreset.ratio16x9],
    );
  }
}
```

---

## ขั้นตอนที่ 279: Compression

```dart
// lib/services/media_compression_service.dart
import 'dart:io';
import 'dart:typed_data';
import 'package:flutter_image_compress/flutter_image_compress.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;
import 'package:mime/mime.dart';

class MediaCompressionService {
  // Compress Image
  static Future<File> compressImage({
    required File file,
    int quality = 70,
    int maxWidth = 1920,
    int maxHeight = 1080,
  }) async {
    final dir = await getTemporaryDirectory();
    final ext = path.extension(file.path).toLowerCase();
    final outputPath = '${dir.path}/compressed_${DateTime.now().millisecondsSinceEpoch}$ext';

    CompressFormat format;
    switch (ext) {
      case '.png':
        format = CompressFormat.png;
        break;
      case '.webp':
        format = CompressFormat.webp;
        break;
      default:
        format = CompressFormat.jpeg;
    }

    final result = await FlutterImageCompress.compressAndGetFile(
      file.path,
      outputPath,
      quality: quality,
      minWidth: maxWidth,
      minHeight: maxHeight,
      format: format,
    );

    if (result == null) throw Exception('Compress ล้มเหลว');

    final compressed = File(result.path);
    final originalSize = await file.length();
    final compressedSize = await compressed.length();

    debugPrint(
      'Compressed: ${_formatBytes(originalSize)} → ${_formatBytes(compressedSize)} '
      '(${((1 - compressedSize / originalSize) * 100).toStringAsFixed(1)}% reduction)',
    );

    return compressed;
  }

  // Compress to Bytes
  static Future<Uint8List> compressToBytes({
    required File file,
    int quality = 70,
  }) async {
    final result = await FlutterImageCompress.compressWithFile(
      file.path,
      quality: quality,
    );
    if (result == null) throw Exception('Compress ล้มเหลว');
    return result;
  }

  // Compress Video (requires video_compress)
  static String _formatBytes(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
  }
}

import 'package:flutter/foundation.dart';
```

---

## ขั้นตอนที่ 280: File Management & Permissions

```dart
// lib/services/file_manager_service.dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;
import 'package:mime/mime.dart';

class FileManagerService {
  // ดึง Directories
  static Future<Directory> getDocumentsDirectory() async {
    return await getApplicationDocumentsDirectory();
  }

  static Future<Directory> getTemporaryDirectory() async {
    return await getTempDirectory();
  }

  static Future<Directory> getTempDirectory() async {
    return Directory.systemTemp;
  }

  // สร้าง Folder
  static Future<Directory> createFolder(String folderName) async {
    final docs = await getDocumentsDirectory();
    final folder = Directory(path.join(docs.path, folderName));
    return await folder.create(recursive: true);
  }

  // คัดลอกไฟล์
  static Future<File> copyFile(File source, String destinationPath) async {
    return await source.copy(destinationPath);
  }

  // บันทึกไฟล์ไปยัง Documents
  static Future<File> saveToDocuments(File file, {String? subfolder}) async {
    final docs = await getDocumentsDirectory();
    final fileName = path.basename(file.path);

    final targetDir = subfolder != null
        ? Directory(path.join(docs.path, subfolder))
        : docs;

    await targetDir.create(recursive: true);
    return await file.copy(path.join(targetDir.path, fileName));
  }

  // แสดงขนาดไฟล์
  static String getFileSizeString(File file) {
    final bytes = file.lengthSync();
    return _formatBytes(bytes);
  }

  // ลบไฟล์
  static Future<void> deleteFile(File file) async {
    if (await file.exists()) {
      await file.delete();
    }
  }

  // ลบไฟล์ชั่วคราว
  static Future<void> cleanTemporaryFiles() async {
    final temp = await getTemporaryDirectory();
    final files = temp.listSync();
    for (final file in files) {
      if (file is File) {
        try {
          await file.delete();
        } catch (_) {}
      }
    }
  }

  // ตรวจสอบ MIME type
  static String? getMimeType(String filePath) {
    return lookupMimeType(filePath);
  }

  static bool isImage(String filePath) {
    final mime = getMimeType(filePath);
    return mime?.startsWith('image/') ?? false;
  }

  static bool isVideo(String filePath) {
    final mime = getMimeType(filePath);
    return mime?.startsWith('video/') ?? false;
  }

  static bool isAudio(String filePath) {
    final mime = getMimeType(filePath);
    return mime?.startsWith('audio/') ?? false;
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

// lib/services/permission_service.dart
import 'package:permission_handler/permission_handler.dart';

class PermissionService {
  static Future<bool> requestCameraPermission() async {
    final status = await Permission.camera.request();
    return status.isGranted;
  }

  static Future<bool> requestMicrophonePermission() async {
    final status = await Permission.microphone.request();
    return status.isGranted;
  }

  static Future<bool> requestPhotosPermission() async {
    final status = await Permission.photos.request();
    return status.isGranted;
  }

  static Future<Map<Permission, PermissionStatus>>
      requestCameraAndMicPermissions() async {
    return await [
      Permission.camera,
      Permission.microphone,
    ].request();
  }

  static Future<bool> openSettings() async {
    return await openAppSettings();
  }
}
```

---

## Workshop: Camera App สมบูรณ์

```dart
// lib/workshop/complete_camera_app.dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:camera/camera.dart';
import 'package:image_picker/image_picker.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final cameras = await availableCameras();
  runApp(CompleteCameraApp(cameras: cameras));
}

class CompleteCameraApp extends StatelessWidget {
  const CompleteCameraApp({super.key, required this.cameras});
  final List<CameraDescription> cameras;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Camera App',
      theme: ThemeData.dark(),
      home: cameras.isEmpty
          ? const Scaffold(
              body: Center(child: Text('ไม่พบกล้อง')),
            )
          : CameraHomePage(cameras: cameras),
    );
  }
}

class CameraHomePage extends StatefulWidget {
  const CameraHomePage({super.key, required this.cameras});
  final List<CameraDescription> cameras;

  @override
  State<CameraHomePage> createState() => _CameraHomePageState();
}

class _CameraHomePageState extends State<CameraHomePage> {
  late CameraController _controller;
  bool _isInitialized = false;
  final List<File> _capturedMedia = [];
  bool _isVideo = false;
  bool _isRecording = false;

  @override
  void initState() {
    super.initState();
    _initCamera(widget.cameras.first);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _initCamera(CameraDescription camera) async {
    final controller = CameraController(
      camera,
      ResolutionPreset.high,
      enableAudio: true,
    );
    await controller.initialize();
    if (mounted) {
      setState(() {
        _controller = controller;
        _isInitialized = true;
      });
    }
  }

  Future<void> _capture() async {
    if (_isVideo) {
      if (_isRecording) {
        final file = await _controller.stopVideoRecording();
        setState(() {
          _capturedMedia.add(File(file.path));
          _isRecording = false;
        });
      } else {
        await _controller.startVideoRecording();
        setState(() => _isRecording = true);
      }
    } else {
      final photo = await _controller.takePicture();
      setState(() => _capturedMedia.add(File(photo.path)));
    }
  }

  @override
  Widget build(BuildContext context) {
    if (!_isInitialized) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    return Scaffold(
      body: Stack(
        children: [
          CameraPreview(_controller),

          // Mode Toggle
          Positioned(
            top: MediaQuery.of(context).padding.top + 8,
            left: 0,
            right: 0,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                _ModeButton(
                  label: 'รูป',
                  isSelected: !_isVideo,
                  onTap: () => setState(() => _isVideo = false),
                ),
                const SizedBox(width: 16),
                _ModeButton(
                  label: 'วิดีโอ',
                  isSelected: _isVideo,
                  onTap: () => setState(() => _isVideo = true),
                ),
              ],
            ),
          ),

          // Recording Indicator
          if (_isRecording)
            const Positioned(
              top: 80,
              left: 16,
              child: Row(
                children: [
                  Icon(Icons.circle, color: Colors.red, size: 12),
                  SizedBox(width: 4),
                  Text('REC', style: TextStyle(color: Colors.white)),
                ],
              ),
            ),

          // Bottom Controls
          Positioned(
            bottom: MediaQuery.of(context).padding.bottom + 24,
            left: 0,
            right: 0,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Gallery
                GestureDetector(
                  onTap: () => Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) => MediaGalleryPage(files: _capturedMedia),
                    ),
                  ),
                  child: Container(
                    width: 60,
                    height: 60,
                    decoration: BoxDecoration(
                      border: Border.all(color: Colors.white, width: 2),
                      borderRadius: BorderRadius.circular(8),
                    ),
                    child: _capturedMedia.isNotEmpty
                        ? ClipRRect(
                            borderRadius: BorderRadius.circular(6),
                            child: Image.file(
                              _capturedMedia.last,
                              fit: BoxFit.cover,
                            ),
                          )
                        : const Icon(Icons.photo_library,
                            color: Colors.white),
                  ),
                ),

                // Shutter
                GestureDetector(
                  onTap: _capture,
                  child: Container(
                    width: 80,
                    height: 80,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      border: Border.all(color: Colors.white, width: 4),
                      color: _isRecording ? Colors.red : Colors.transparent,
                    ),
                    child: _isVideo
                        ? Icon(
                            _isRecording ? Icons.stop : Icons.fiber_manual_record,
                            color: _isRecording ? Colors.white : Colors.red,
                            size: 36,
                          )
                        : null,
                  ),
                ),

                // Flip Camera
                IconButton(
                  icon: const Icon(Icons.flip_camera_ios,
                      color: Colors.white, size: 36),
                  onPressed: () {
                    final cameras = widget.cameras;
                    if (cameras.length < 2) return;
                    final current = _controller.description;
                    final next = cameras.firstWhere(
                      (c) => c.lensDirection != current.lensDirection,
                      orElse: () => cameras.first,
                    );
                    _initCamera(next);
                  },
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _ModeButton extends StatelessWidget {
  const _ModeButton({
    required this.label,
    required this.isSelected,
    required this.onTap,
  });

  final String label;
  final bool isSelected;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        decoration: BoxDecoration(
          color: isSelected ? Colors.white : Colors.transparent,
          borderRadius: BorderRadius.circular(20),
        ),
        child: Text(
          label,
          style: TextStyle(
            color: isSelected ? Colors.black : Colors.white,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  }
}

class MediaGalleryPage extends StatelessWidget {
  const MediaGalleryPage({super.key, required this.files});
  final List<File> files;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('คลัง (${files.length})'),
        backgroundColor: Colors.black,
        foregroundColor: Colors.white,
      ),
      backgroundColor: Colors.black,
      body: files.isEmpty
          ? const Center(
              child: Text('ยังไม่มีรูป/วิดีโอ',
                  style: TextStyle(color: Colors.white)),
            )
          : GridView.builder(
              padding: const EdgeInsets.all(4),
              gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 3,
                crossAxisSpacing: 2,
                mainAxisSpacing: 2,
              ),
              itemCount: files.length,
              itemBuilder: (context, index) {
                return Image.file(
                  files[index],
                  fit: BoxFit.cover,
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
- **camera Package**: การใช้กล้องใน Flutter
- **image_picker**: การเลือกรูปภาพ/วิดีโอ
- **Taking Photos**: การถ่ายรูปด้วย Camera Controller
- **Taking Videos**: การบันทึกวิดีโอ
- **Video Player**: การเล่นวิดีโอ
- **Audio Recording**: การบันทึกเสียง
- **Image Cropping**: การครอปรูปภาพ
- **Compression**: การบีบอัดไฟล์
- **File Management**: การจัดการไฟล์และ Permissions

## แบบฝึกหัด

1. สร้าง Instagram-like Camera ที่มี Filters
2. Implement QR Code Scanner
3. สร้าง Voice Recorder ที่บันทึกหลาย Track
4. สร้าง Image Editor ที่ Crop, Rotate, Brightness
5. เพิ่ม Live Face Detection ด้วย ML Kit

---

[⬅️ Part 27](part_27.md) | [Part 29 ➡️](part_29.md)

# Part 46: Machine Learning & AI Integration
## ขั้นตอนที่ 451-460

---

## สารบัญ
1. [Google ML Kit - Text Recognition](#ขั้นตอนที่-451-text-recognition)
2. [Face Detection](#ขั้นตอนที่-452-face-detection)
3. [Barcode Scanning](#ขั้นตอนที่-453-barcode-scanning)
4. [Pose Detection](#ขั้นตอนที่-454-pose-detection)
5. [Object Detection](#ขั้นตอนที่-455-object-detection)
6. [TensorFlow Lite](#ขั้นตอนที่-456-tensorflow-lite)
7. [On-device ML](#ขั้นตอนที่-457-on-device-ml)
8. [OpenAI API Integration](#ขั้นตอนที่-458-openai-api)
9. [Image Classification](#ขั้นตอนที่-459-image-classification)
10. [NLP Features](#ขั้นตอนที่-460-nlp-features)

---

## ขั้นตอนที่ 451: Google ML Kit - Text Recognition

### ML Kit คืออะไร?

Google ML Kit คือชุด Machine Learning APIs สำหรับ mobile apps ที่ทำงานได้ทั้ง on-device และ cloud-based

```
ML Kit Features:
├── Vision
│   ├── Text Recognition (OCR)
│   ├── Face Detection
│   ├── Barcode Scanning
│   ├── Image Labeling
│   ├── Object Detection
│   ├── Pose Detection
│   └── Document Scanner
├── Natural Language
│   ├── Language Identification
│   ├── Translation
│   ├── Smart Reply
│   └── Entity Extraction
└── Custom Models
    ├── TensorFlow Lite
    └── AutoML Vision Edge
```

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  # ML Kit Vision
  google_mlkit_text_recognition: ^0.13.0
  google_mlkit_face_detection: ^0.11.0
  google_mlkit_barcode_scanning: ^0.12.0
  google_mlkit_pose_detection: ^0.11.0
  google_mlkit_object_detection: ^0.13.0
  google_mlkit_image_labeling: ^0.11.0
  
  # Camera
  camera: ^0.10.0
  image_picker: ^1.0.0
  
  # Image processing
  image: ^4.1.0
```

### Text Recognition (OCR)

```dart
// lib/features/ml/services/text_recognition_service.dart
import 'package:google_mlkit_text_recognition/google_mlkit_text_recognition.dart';
import 'package:image_picker/image_picker.dart';

class TextRecognitionService {
  final TextRecognizer _recognizer;
  
  TextRecognitionService()
      : _recognizer = TextRecognizer(script: TextRecognitionScript.latin);
  
  Future<RecognizedTextResult> recognizeFromImage(XFile imageFile) async {
    final inputImage = InputImage.fromFile(File(imageFile.path));
    
    try {
      final recognizedText = await _recognizer.processImage(inputImage);
      
      return RecognizedTextResult(
        fullText: recognizedText.text,
        blocks: recognizedText.blocks.map((block) {
          return TextBlock(
            text: block.text,
            boundingBox: block.boundingBox,
            lines: block.lines.map((line) {
              return TextLine(
                text: line.text,
                boundingBox: line.boundingBox,
                elements: line.elements.map((element) {
                  return TextElement(
                    text: element.text,
                    boundingBox: element.boundingBox,
                    confidence: element.confidence ?? 0.0,
                  );
                }).toList(),
              );
            }).toList(),
          );
        }).toList(),
      );
    } catch (e) {
      throw TextRecognitionException('Text recognition failed: $e');
    }
  }
  
  Future<RecognizedTextResult?> recognizeFromCamera() async {
    final picker = ImagePicker();
    final image = await picker.pickImage(
      source: ImageSource.camera,
      imageQuality: 90,
    );
    
    if (image == null) return null;
    return recognizeFromImage(image);
  }
  
  Future<RecognizedTextResult?> recognizeFromGallery() async {
    final picker = ImagePicker();
    final image = await picker.pickImage(
      source: ImageSource.gallery,
    );
    
    if (image == null) return null;
    return recognizeFromImage(image);
  }
  
  void dispose() {
    _recognizer.close();
  }
}

class RecognizedTextResult {
  final String fullText;
  final List<TextBlock> blocks;
  
  const RecognizedTextResult({
    required this.fullText,
    required this.blocks,
  });
}

class TextBlock {
  final String text;
  final Rect boundingBox;
  final List<TextLine> lines;
  
  const TextBlock({
    required this.text,
    required this.boundingBox,
    required this.lines,
  });
}

class TextLine {
  final String text;
  final Rect boundingBox;
  final List<TextElement> elements;
  
  const TextLine({
    required this.text,
    required this.boundingBox,
    required this.elements,
  });
}

class TextElement {
  final String text;
  final Rect boundingBox;
  final double confidence;
  
  const TextElement({
    required this.text,
    required this.boundingBox,
    required this.confidence,
  });
}
```

### OCR Screen

```dart
// lib/features/ml/screens/ocr_screen.dart
class OCRScreen extends ConsumerStatefulWidget {
  const OCRScreen({super.key});
  
  @override
  ConsumerState<OCRScreen> createState() => _OCRScreenState();
}

class _OCRScreenState extends ConsumerState<OCRScreen> {
  final _textRecognitionService = TextRecognitionService();
  RecognizedTextResult? _result;
  bool _isProcessing = false;
  File? _selectedImage;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ถ่ายข้อความ (OCR)'),
        actions: [
          if (_result != null)
            IconButton(
              onPressed: _copyText,
              icon: const Icon(Icons.copy),
            ),
        ],
      ),
      body: Column(
        children: [
          // Image preview area
          Expanded(
            flex: 2,
            child: GestureDetector(
              onTap: _pickImage,
              child: Container(
                width: double.infinity,
                color: Colors.black12,
                child: _selectedImage != null
                    ? Stack(
                        fit: StackFit.expand,
                        children: [
                          Image.file(_selectedImage!, fit: BoxFit.contain),
                          if (_result != null)
                            _buildTextOverlay(),
                        ],
                      )
                    : const Center(
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            Icon(Icons.add_photo_alternate, size: 64),
                            SizedBox(height: 8),
                            Text('แตะเพื่อเลือกรูปภาพ'),
                          ],
                        ),
                      ),
              ),
            ),
          ),
          
          // Buttons
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _isProcessing ? null : _captureFromCamera,
                    icon: const Icon(Icons.camera_alt),
                    label: const Text('ถ่ายรูป'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _isProcessing ? null : _pickImage,
                    icon: const Icon(Icons.photo_library),
                    label: const Text('เลือกรูป'),
                  ),
                ),
              ],
            ),
          ),
          
          // Recognized text
          Expanded(
            flex: 2,
            child: Container(
              width: double.infinity,
              padding: const EdgeInsets.all(16),
              child: _isProcessing
                  ? const Center(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          CircularProgressIndicator(),
                          SizedBox(height: 16),
                          Text('กำลังประมวลผล...'),
                        ],
                      ),
                    )
                  : _result != null
                      ? SingleChildScrollView(
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text(
                                'ข้อความที่พบ:',
                                style: Theme.of(context).textTheme.titleMedium,
                              ),
                              const SizedBox(height: 8),
                              SelectableText(_result!.fullText),
                            ],
                          ),
                        )
                      : const Center(
                          child: Text('ยังไม่มีผลลัพธ์'),
                        ),
            ),
          ),
        ],
      ),
    );
  }
  
  Widget _buildTextOverlay() {
    // Show bounding boxes on image
    return CustomPaint(
      painter: TextBoundingBoxPainter(result: _result!),
    );
  }
  
  Future<void> _captureFromCamera() async {
    setState(() => _isProcessing = true);
    
    try {
      final result = await _textRecognitionService.recognizeFromCamera();
      
      if (result != null) {
        setState(() {
          _result = result;
        });
      }
    } catch (e) {
      _showError('OCR ล้มเหลว: $e');
    } finally {
      setState(() => _isProcessing = false);
    }
  }
  
  Future<void> _pickImage() async {
    final picker = ImagePicker();
    final image = await picker.pickImage(source: ImageSource.gallery);
    
    if (image == null) return;
    
    setState(() {
      _selectedImage = File(image.path);
      _isProcessing = true;
    });
    
    try {
      final result = await _textRecognitionService.recognizeFromImage(image);
      setState(() => _result = result);
    } catch (e) {
      _showError('OCR ล้มเหลว: $e');
    } finally {
      setState(() => _isProcessing = false);
    }
  }
  
  void _copyText() {
    if (_result != null) {
      Clipboard.setData(ClipboardData(text: _result!.fullText));
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('คัดลอกข้อความแล้ว')),
      );
    }
  }
  
  void _showError(String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(message), backgroundColor: Colors.red),
    );
  }
  
  @override
  void dispose() {
    _textRecognitionService.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 452: Face Detection

```dart
// lib/features/ml/services/face_detection_service.dart
import 'package:google_mlkit_face_detection/google_mlkit_face_detection.dart';

class FaceDetectionService {
  late final FaceDetector _detector;
  
  FaceDetectionService() {
    _detector = FaceDetector(
      options: FaceDetectorOptions(
        enableClassification: true, // Smiling, eyes open
        enableLandmarks: true,      // Nose, ears, eyes positions
        enableContours: true,       // Face outline
        enableTracking: true,       // Track faces across frames
        minFaceSize: 0.15,
        performanceMode: FaceDetectorMode.accurate,
      ),
    );
  }
  
  Future<List<DetectedFace>> detectFaces(InputImage inputImage) async {
    final faces = await _detector.processImage(inputImage);
    
    return faces.map((face) {
      return DetectedFace(
        boundingBox: face.boundingBox,
        trackingId: face.trackingId,
        smilingProbability: face.smilingProbability,
        leftEyeOpenProbability: face.leftEyeOpenProbability,
        rightEyeOpenProbability: face.rightEyeOpenProbability,
        headEulerAngleX: face.headEulerAngleX,
        headEulerAngleY: face.headEulerAngleY,
        headEulerAngleZ: face.headEulerAngleZ,
        landmarks: _extractLandmarks(face),
      );
    }).toList();
  }
  
  Map<String, FaceLandmark?> _extractLandmarks(Face face) {
    return {
      'leftEye': face.landmarks[FaceLandmarkType.leftEye],
      'rightEye': face.landmarks[FaceLandmarkType.rightEye],
      'nose': face.landmarks[FaceLandmarkType.noseBase],
      'leftMouth': face.landmarks[FaceLandmarkType.leftMouth],
      'rightMouth': face.landmarks[FaceLandmarkType.rightMouth],
    };
  }
  
  void dispose() => _detector.close();
}

class DetectedFace {
  final Rect boundingBox;
  final int? trackingId;
  final double? smilingProbability;
  final double? leftEyeOpenProbability;
  final double? rightEyeOpenProbability;
  final double? headEulerAngleX;
  final double? headEulerAngleY;
  final double? headEulerAngleZ;
  final Map<String, FaceLandmark?> landmarks;
  
  const DetectedFace({
    required this.boundingBox,
    this.trackingId,
    this.smilingProbability,
    this.leftEyeOpenProbability,
    this.rightEyeOpenProbability,
    this.headEulerAngleX,
    this.headEulerAngleY,
    this.headEulerAngleZ,
    required this.landmarks,
  });
  
  bool get isSmiling => (smilingProbability ?? 0) > 0.8;
  bool get hasEyesOpen =>
      (leftEyeOpenProbability ?? 0) > 0.8 &&
      (rightEyeOpenProbability ?? 0) > 0.8;
}

// Face Detection with Camera
class FaceDetectionCamera extends StatefulWidget {
  final Function(List<DetectedFace>) onFacesDetected;
  
  const FaceDetectionCamera({super.key, required this.onFacesDetected});
  
  @override
  State<FaceDetectionCamera> createState() => _FaceDetectionCameraState();
}

class _FaceDetectionCameraState extends State<FaceDetectionCamera> {
  CameraController? _cameraController;
  final FaceDetectionService _faceService = FaceDetectionService();
  bool _isProcessing = false;
  List<DetectedFace> _faces = [];
  
  @override
  void initState() {
    super.initState();
    _initCamera();
  }
  
  Future<void> _initCamera() async {
    final cameras = await availableCameras();
    final frontCamera = cameras.firstWhere(
      (c) => c.lensDirection == CameraLensDirection.front,
      orElse: () => cameras.first,
    );
    
    _cameraController = CameraController(
      frontCamera,
      ResolutionPreset.medium,
      imageFormatGroup: Platform.isAndroid
          ? ImageFormatGroup.nv21
          : ImageFormatGroup.bgra8888,
    );
    
    await _cameraController!.initialize();
    
    _cameraController!.startImageStream(_processImage);
    
    if (mounted) setState(() {});
  }
  
  Future<void> _processImage(CameraImage image) async {
    if (_isProcessing) return;
    _isProcessing = true;
    
    try {
      final inputImage = _convertToInputImage(image);
      if (inputImage == null) return;
      
      final faces = await _faceService.detectFaces(inputImage);
      
      if (mounted) {
        setState(() => _faces = faces);
        widget.onFacesDetected(faces);
      }
    } finally {
      _isProcessing = false;
    }
  }
  
  InputImage? _convertToInputImage(CameraImage image) {
    final WriteBuffer allBytes = WriteBuffer();
    for (final Plane plane in image.planes) {
      allBytes.putUint8List(plane.bytes);
    }
    final bytes = allBytes.done().buffer.asUint8List();
    
    final Size imageSize = Size(
      image.width.toDouble(),
      image.height.toDouble(),
    );
    
    final camera = _cameraController!.description;
    final imageRotation = InputImageRotationValue.fromRawValue(
      camera.sensorOrientation,
    );
    
    if (imageRotation == null) return null;
    
    final inputImageFormat = InputImageFormatValue.fromRawValue(
      image.format.raw,
    );
    if (inputImageFormat == null) return null;
    
    final planeData = image.planes.map((Plane plane) {
      return InputImagePlaneMetadata(
        bytesPerRow: plane.bytesPerRow,
        height: plane.height,
        width: plane.width,
      );
    }).toList();
    
    final inputImageData = InputImageData(
      size: imageSize,
      imageRotation: imageRotation,
      inputImageFormat: inputImageFormat,
      planeData: planeData,
    );
    
    return InputImage.fromBytes(
      bytes: bytes,
      inputImageData: inputImageData,
    );
  }
  
  @override
  Widget build(BuildContext context) {
    if (_cameraController == null || !_cameraController!.value.isInitialized) {
      return const Center(child: CircularProgressIndicator());
    }
    
    return Stack(
      children: [
        CameraPreview(_cameraController!),
        CustomPaint(
          painter: FaceBoundingBoxPainter(
            faces: _faces,
            imageSize: Size(
              _cameraController!.value.previewSize!.height,
              _cameraController!.value.previewSize!.width,
            ),
          ),
        ),
      ],
    );
  }
  
  @override
  void dispose() {
    _cameraController?.dispose();
    _faceService.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 453: Barcode Scanning

```dart
// lib/features/ml/services/barcode_scanner_service.dart
import 'package:google_mlkit_barcode_scanning/google_mlkit_barcode_scanning.dart';

class BarcodeScannerService {
  late final BarcodeScanner _scanner;
  
  BarcodeScannerService() {
    _scanner = BarcodeScanner(
      formats: [
        BarcodeFormat.qrCode,
        BarcodeFormat.code128,
        BarcodeFormat.code39,
        BarcodeFormat.ean13,
        BarcodeFormat.ean8,
        BarcodeFormat.pdf417,
        BarcodeFormat.aztec,
        BarcodeFormat.dataMatrix,
      ],
    );
  }
  
  Future<List<ScannedBarcode>> scanFromImage(InputImage inputImage) async {
    final barcodes = await _scanner.processImage(inputImage);
    
    return barcodes.map((barcode) {
      return ScannedBarcode(
        rawValue: barcode.rawValue ?? '',
        type: barcode.type,
        format: barcode.format,
        boundingBox: barcode.boundingBox,
        data: _extractBarcodeData(barcode),
      );
    }).toList();
  }
  
  BarcodeData? _extractBarcodeData(Barcode barcode) {
    return switch (barcode.type) {
      BarcodeType.url => UrlData(barcode.value?.url?.url ?? ''),
      BarcodeType.contactInfo => ContactData(
        name: barcode.value?.contactInfo?.name?.toString() ?? '',
        phone: barcode.value?.contactInfo?.phones?.firstOrNull?.number ?? '',
        email: barcode.value?.contactInfo?.emails?.firstOrNull?.address ?? '',
      ),
      BarcodeType.wifi => WifiData(
        ssid: barcode.value?.wifi?.ssid ?? '',
        password: barcode.value?.wifi?.password ?? '',
      ),
      BarcodeType.geoPoint => LocationData(
        latitude: barcode.value?.geoPoint?.latitude ?? 0,
        longitude: barcode.value?.geoPoint?.longitude ?? 0,
      ),
      _ => null,
    };
  }
  
  void dispose() => _scanner.close();
}

// Barcode Scanner Widget
class BarcodeScannerWidget extends StatefulWidget {
  final Function(ScannedBarcode) onBarcodeDetected;
  
  const BarcodeScannerWidget({
    super.key,
    required this.onBarcodeDetected,
  });
  
  @override
  State<BarcodeScannerWidget> createState() => _BarcodeScannerWidgetState();
}

class _BarcodeScannerWidgetState extends State<BarcodeScannerWidget> {
  late MobileScannerController _controller;
  
  @override
  void initState() {
    super.initState();
    _controller = MobileScannerController(
      detectionSpeed: DetectionSpeed.normal,
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return MobileScanner(
      controller: _controller,
      onDetect: (capture) {
        final barcodes = capture.barcodes;
        for (final barcode in barcodes) {
          if (barcode.rawValue != null) {
            widget.onBarcodeDetected(ScannedBarcode(
              rawValue: barcode.rawValue!,
              type: barcode.type,
              format: barcode.format,
              boundingBox: barcode.boundingBox,
              data: null,
            ));
          }
        }
      },
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 454: Pose Detection

```dart
// lib/features/ml/services/pose_detection_service.dart
import 'package:google_mlkit_pose_detection/google_mlkit_pose_detection.dart';

class PoseDetectionService {
  late final PoseDetector _detector;
  
  PoseDetectionService() {
    _detector = PoseDetector(
      options: PoseDetectorOptions(
        model: PoseDetectionModel.accurate,
        mode: PoseDetectionMode.stream,
      ),
    );
  }
  
  Future<List<PoseData>> detectPoses(InputImage inputImage) async {
    final poses = await _detector.processImage(inputImage);
    
    return poses.map((pose) {
      final landmarks = <PoseLandmarkType, LandmarkData>{};
      
      for (final landmark in pose.landmarks.values) {
        landmarks[landmark.type] = LandmarkData(
          type: landmark.type,
          x: landmark.x,
          y: landmark.y,
          z: landmark.z,
          likelihood: landmark.likelihood,
        );
      }
      
      return PoseData(landmarks: landmarks);
    }).toList();
  }
  
  // ตรวจสอบท่า squat
  bool isSquatPosition(PoseData pose) {
    final leftHip = pose.landmarks[PoseLandmarkType.leftHip];
    final leftKnee = pose.landmarks[PoseLandmarkType.leftKnee];
    final leftAnkle = pose.landmarks[PoseLandmarkType.leftAnkle];
    
    if (leftHip == null || leftKnee == null || leftAnkle == null) {
      return false;
    }
    
    // Calculate knee angle
    final kneeAngle = _calculateAngle(
      leftHip.toOffset(),
      leftKnee.toOffset(),
      leftAnkle.toOffset(),
    );
    
    // Squat: knee angle between 60-90 degrees
    return kneeAngle >= 60 && kneeAngle <= 90;
  }
  
  double _calculateAngle(Offset a, Offset b, Offset c) {
    final radians = math.atan2(c.dy - b.dy, c.dx - b.dx) -
        math.atan2(a.dy - b.dy, a.dx - b.dx);
    double angle = (radians * 180 / math.pi).abs();
    if (angle > 180) angle = 360 - angle;
    return angle;
  }
  
  void dispose() => _detector.close();
}

class PoseData {
  final Map<PoseLandmarkType, LandmarkData> landmarks;
  
  const PoseData({required this.landmarks});
}

class LandmarkData {
  final PoseLandmarkType type;
  final double x;
  final double y;
  final double z;
  final double likelihood;
  
  const LandmarkData({
    required this.type,
    required this.x,
    required this.y,
    required this.z,
    required this.likelihood,
  });
  
  Offset toOffset() => Offset(x, y);
}
```

---

## ขั้นตอนที่ 455: Object Detection

```dart
// lib/features/ml/services/object_detection_service.dart
import 'package:google_mlkit_object_detection/google_mlkit_object_detection.dart';

class ObjectDetectionService {
  late final ObjectDetector _detector;
  
  ObjectDetectionService() {
    _detector = ObjectDetector(
      options: ObjectDetectorOptions(
        mode: DetectionMode.stream,
        classifyObjects: true,
        multipleObjects: true,
      ),
    );
  }
  
  Future<List<DetectedObject>> detectObjects(InputImage inputImage) async {
    final objects = await _detector.processImage(inputImage);
    
    return objects.map((obj) {
      return DetectedObject(
        trackingId: obj.trackingId,
        boundingBox: obj.boundingBox,
        labels: obj.labels.map((label) {
          return ObjectLabel(
            text: label.text,
            confidence: label.confidence,
            index: label.index,
          );
        }).toList(),
      );
    }).toList();
  }
  
  void dispose() => _detector.close();
}

class DetectedObject {
  final int? trackingId;
  final Rect boundingBox;
  final List<ObjectLabel> labels;
  
  const DetectedObject({
    this.trackingId,
    required this.boundingBox,
    required this.labels,
  });
  
  ObjectLabel? get topLabel =>
      labels.isEmpty ? null : labels.reduce(
        (a, b) => a.confidence > b.confidence ? a : b,
      );
}

class ObjectLabel {
  final String text;
  final double confidence;
  final int index;
  
  const ObjectLabel({
    required this.text,
    required this.confidence,
    required this.index,
  });
}
```

---

## ขั้นตอนที่ 456: TensorFlow Lite

### TFLite Integration

```yaml
dependencies:
  tflite_flutter: ^0.10.4
  tflite_flutter_helper: ^0.4.0
```

```dart
// lib/features/ml/services/tflite_service.dart
import 'package:tflite_flutter/tflite_flutter.dart';

class TFLiteClassifier {
  Interpreter? _interpreter;
  List<String>? _labels;
  
  Future<void> initialize(String modelPath, String labelsPath) async {
    // Load model
    final options = InterpreterOptions()..threads = 4;
    _interpreter = await Interpreter.fromAsset(
      modelPath,
      options: options,
    );
    
    // Load labels
    final labelsData = await rootBundle.loadString(labelsPath);
    _labels = labelsData.split('\n');
  }
  
  Future<List<ClassificationResult>> classify(Uint8List imageBytes) async {
    if (_interpreter == null) throw StateError('Model not initialized');
    
    // Preprocess image
    final input = _preprocessImage(imageBytes);
    
    // Run inference
    final outputShape = _interpreter!.getOutputTensor(0).shape;
    final output = List.filled(outputShape[1], 0.0)
        .reshape([1, outputShape[1]]);
    
    _interpreter!.run(input, output);
    
    // Post-process results
    final probabilities = List<double>.from(output[0] as List);
    
    final results = <ClassificationResult>[];
    for (int i = 0; i < probabilities.length; i++) {
      if (probabilities[i] > 0.05) { // Threshold
        results.add(ClassificationResult(
          label: _labels?[i] ?? 'Unknown',
          confidence: probabilities[i],
          index: i,
        ));
      }
    }
    
    results.sort((a, b) => b.confidence.compareTo(a.confidence));
    return results.take(5).toList(); // Top 5
  }
  
  List<List<List<List<double>>>> _preprocessImage(Uint8List imageBytes) {
    // Convert to Float32 tensor [1, 224, 224, 3]
    // Normalize to [-1, 1] or [0, 1] depending on model
    
    final img = image_lib.decodeImage(imageBytes)!;
    final resized = image_lib.copyResize(img, width: 224, height: 224);
    
    const inputSize = 224;
    final result = List.generate(
      1,
      (_) => List.generate(
        inputSize,
        (y) => List.generate(
          inputSize,
          (x) {
            final pixel = resized.getPixel(x, y);
            return [
              (image_lib.getRed(pixel) - 127.5) / 127.5,
              (image_lib.getGreen(pixel) - 127.5) / 127.5,
              (image_lib.getBlue(pixel) - 127.5) / 127.5,
            ];
          },
        ),
      ),
    );
    
    return result;
  }
  
  void dispose() {
    _interpreter?.close();
  }
}

class ClassificationResult {
  final String label;
  final double confidence;
  final int index;
  
  const ClassificationResult({
    required this.label,
    required this.confidence,
    required this.index,
  });
}
```

---

## ขั้นตอนที่ 457: On-device ML

### On-device vs Cloud ML

```
On-device ML:                     Cloud ML:
✅ No internet required           ✅ More powerful models
✅ Fast (no network latency)       ✅ Can be updated easily
✅ Privacy (data stays on device)  ✅ No device resource limits
✅ Lower cost (no API calls)       ✅ Better accuracy generally
❌ Limited model size              ❌ Requires internet
❌ Limited device resources        ❌ Latency
```

### Custom TFLite Model

```dart
// lib/features/ml/models/custom_model.dart
class CustomMLModel {
  static const String _modelName = 'custom_model.tflite';
  static const String _labelsName = 'custom_labels.txt';
  
  late Interpreter _interpreter;
  late List<String> _labels;
  
  static Future<CustomMLModel> create() async {
    final model = CustomMLModel();
    await model._initialize();
    return model;
  }
  
  Future<void> _initialize() async {
    // Use GPU delegate if available
    final options = InterpreterOptions();
    
    if (Platform.isAndroid) {
      options.addDelegate(GpuDelegateV2(
        options: GpuDelegateOptionsV2(
          isPrecisionLossAllowed: false,
          inferencePreference: TfLiteGpuInferenceUsage.fastSingleAnswer,
        ),
      ));
    } else if (Platform.isIOS) {
      options.addDelegate(GpuDelegate(
        options: GpuDelegateOptions(allowPrecisionLoss: true),
      ));
    }
    
    _interpreter = await Interpreter.fromAsset(
      'assets/models/$_modelName',
      options: options,
    );
    
    final labelsData = await rootBundle.loadString(
      'assets/models/$_labelsName',
    );
    _labels = labelsData.split('\n').where((l) => l.isNotEmpty).toList();
  }
  
  Future<List<ClassificationResult>> predict(Uint8List imageBytes) async {
    // Implementation
    return [];
  }
  
  void close() => _interpreter.close();
}
```

---

## ขั้นตอนที่ 458: OpenAI API Integration

```dart
// lib/features/ai/services/openai_service.dart
import 'package:dart_openai/dart_openai.dart';

class OpenAIService {
  static void initialize(String apiKey) {
    OpenAI.apiKey = apiKey;
    OpenAI.baseUrl = 'https://api.openai.com';
  }
  
  // Chat Completion
  Future<String> chat({
    required String userMessage,
    List<Map<String, String>> history = const [],
    String systemPrompt = 'คุณเป็นผู้ช่วยที่เป็นประโยชน์',
    String model = 'gpt-4o-mini',
  }) async {
    final messages = [
      OpenAIChatCompletionChoiceMessageModel(
        content: [
          OpenAIChatCompletionChoiceMessageContentItemModel.text(systemPrompt),
        ],
        role: OpenAIChatMessageRole.system,
      ),
      // Add history
      ...history.map((msg) => OpenAIChatCompletionChoiceMessageModel(
        content: [
          OpenAIChatCompletionChoiceMessageContentItemModel.text(msg['content']!),
        ],
        role: msg['role'] == 'user'
            ? OpenAIChatMessageRole.user
            : OpenAIChatMessageRole.assistant,
      )),
      // Add current message
      OpenAIChatCompletionChoiceMessageModel(
        content: [
          OpenAIChatCompletionChoiceMessageContentItemModel.text(userMessage),
        ],
        role: OpenAIChatMessageRole.user,
      ),
    ];
    
    final response = await OpenAI.instance.chat.create(
      model: model,
      messages: messages,
      maxTokens: 2000,
      temperature: 0.7,
    );
    
    return response.choices.first.message.content?.first.text ?? '';
  }
  
  // Streaming Chat
  Stream<String> streamChat({
    required String userMessage,
    String systemPrompt = 'คุณเป็นผู้ช่วยที่เป็นประโยชน์',
  }) {
    return OpenAI.instance.chat.createStream(
      model: 'gpt-4o-mini',
      messages: [
        OpenAIChatCompletionChoiceMessageModel(
          content: [
            OpenAIChatCompletionChoiceMessageContentItemModel.text(systemPrompt),
          ],
          role: OpenAIChatMessageRole.system,
        ),
        OpenAIChatCompletionChoiceMessageModel(
          content: [
            OpenAIChatCompletionChoiceMessageContentItemModel.text(userMessage),
          ],
          role: OpenAIChatMessageRole.user,
        ),
      ],
    ).map((chunk) {
      return chunk.choices.first.delta.content?.first.text ?? '';
    });
  }
  
  // Image Analysis (Vision)
  Future<String> analyzeImage({
    required String imageUrl,
    required String prompt,
  }) async {
    final response = await OpenAI.instance.chat.create(
      model: 'gpt-4o',
      messages: [
        OpenAIChatCompletionChoiceMessageModel(
          content: [
            OpenAIChatCompletionChoiceMessageContentItemModel.text(prompt),
            OpenAIChatCompletionChoiceMessageContentItemModel.imageUrl(imageUrl),
          ],
          role: OpenAIChatMessageRole.user,
        ),
      ],
    );
    
    return response.choices.first.message.content?.first.text ?? '';
  }
  
  // Text Embedding
  Future<List<double>> createEmbedding(String text) async {
    final response = await OpenAI.instance.embedding.create(
      model: 'text-embedding-3-small',
      input: text,
    );
    
    return response.data.first.embeddings;
  }
  
  // Image Generation
  Future<String> generateImage({
    required String prompt,
    String size = '1024x1024',
  }) async {
    final response = await OpenAI.instance.image.create(
      prompt: prompt,
      n: 1,
      size: OpenAIImageSize.size1024,
      responseFormat: OpenAIImageResponseFormat.url,
    );
    
    return response.data.first.url ?? '';
  }
}

// AI Chat Screen
class AIChatScreen extends ConsumerStatefulWidget {
  const AIChatScreen({super.key});
  
  @override
  ConsumerState<AIChatScreen> createState() => _AIChatScreenState();
}

class _AIChatScreenState extends ConsumerState<AIChatScreen> {
  final TextEditingController _controller = TextEditingController();
  final List<ChatMessage> _messages = [];
  final OpenAIService _aiService = OpenAIService();
  bool _isLoading = false;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AI Assistant')),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.all(16),
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final message = _messages[index];
                return _buildMessageBubble(message);
              },
            ),
          ),
          if (_isLoading)
            const Padding(
              padding: EdgeInsets.all(8),
              child: Row(
                children: [
                  SizedBox(width: 16),
                  SizedBox(
                    width: 20,
                    height: 20,
                    child: CircularProgressIndicator(strokeWidth: 2),
                  ),
                  SizedBox(width: 8),
                  Text('AI กำลังคิด...'),
                ],
              ),
            ),
          _buildInputArea(),
        ],
      ),
    );
  }
  
  Widget _buildMessageBubble(ChatMessage message) {
    final isUser = message.senderId == 'user';
    
    return Align(
      alignment: isUser ? Alignment.centerRight : Alignment.centerLeft,
      child: Container(
        margin: const EdgeInsets.only(bottom: 8),
        padding: const EdgeInsets.all(12),
        constraints: BoxConstraints(
          maxWidth: MediaQuery.of(context).size.width * 0.75,
        ),
        decoration: BoxDecoration(
          color: isUser
              ? Theme.of(context).colorScheme.primary
              : Theme.of(context).colorScheme.secondaryContainer,
          borderRadius: BorderRadius.circular(16),
        ),
        child: MarkdownBody(
          data: message.content,
          styleSheet: MarkdownStyleSheet(
            p: TextStyle(
              color: isUser ? Colors.white : null,
            ),
          ),
        ),
      ),
    );
  }
  
  Widget _buildInputArea() {
    return Container(
      padding: const EdgeInsets.all(8),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              controller: _controller,
              decoration: const InputDecoration(
                hintText: 'ถามคำถาม...',
                border: OutlineInputBorder(),
              ),
              maxLines: null,
            ),
          ),
          const SizedBox(width: 8),
          IconButton.filled(
            onPressed: _isLoading ? null : _sendMessage,
            icon: const Icon(Icons.send),
          ),
        ],
      ),
    );
  }
  
  Future<void> _sendMessage() async {
    final text = _controller.text.trim();
    if (text.isEmpty) return;
    
    _controller.clear();
    
    setState(() {
      _messages.add(ChatMessage(
        id: DateTime.now().toString(),
        roomId: 'ai',
        senderId: 'user',
        content: text,
        type: MessageType.text,
        createdAt: DateTime.now(),
      ));
      _isLoading = true;
    });
    
    try {
      final response = await _aiService.chat(
        userMessage: text,
        systemPrompt: 'คุณเป็นผู้ช่วย Flutter Developer ที่เชี่ยวชาญ ตอบเป็นภาษาไทย',
      );
      
      setState(() {
        _messages.add(ChatMessage(
          id: DateTime.now().toString(),
          roomId: 'ai',
          senderId: 'ai',
          content: response,
          type: MessageType.text,
          createdAt: DateTime.now(),
        ));
      });
    } catch (e) {
      setState(() {
        _messages.add(ChatMessage(
          id: DateTime.now().toString(),
          roomId: 'ai',
          senderId: 'ai',
          content: 'เกิดข้อผิดพลาด: $e',
          type: MessageType.text,
          createdAt: DateTime.now(),
        ));
      });
    } finally {
      setState(() => _isLoading = false);
    }
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 459: Image Classification

### Custom Image Classifier

```dart
// lib/features/ml/screens/image_classifier_screen.dart
class ImageClassifierScreen extends StatefulWidget {
  const ImageClassifierScreen({super.key});
  
  @override
  State<ImageClassifierScreen> createState() => _ImageClassifierScreenState();
}

class _ImageClassifierScreenState extends State<ImageClassifierScreen> {
  final TFLiteClassifier _classifier = TFLiteClassifier();
  File? _selectedImage;
  List<ClassificationResult> _results = [];
  bool _isClassifying = false;
  
  @override
  void initState() {
    super.initState();
    _initClassifier();
  }
  
  Future<void> _initClassifier() async {
    await _classifier.initialize(
      'assets/models/mobilenet_v2.tflite',
      'assets/models/imagenet_labels.txt',
    );
  }
  
  Future<void> _classifyImage(File imageFile) async {
    setState(() {
      _selectedImage = imageFile;
      _isClassifying = true;
    });
    
    try {
      final bytes = await imageFile.readAsBytes();
      final results = await _classifier.classify(bytes);
      setState(() => _results = results);
    } finally {
      setState(() => _isClassifying = false);
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('จำแนกรูปภาพ')),
      body: Column(
        children: [
          // Image display
          Expanded(
            child: _selectedImage != null
                ? Image.file(_selectedImage!, fit: BoxFit.contain)
                : const Center(
                    child: Text('เลือกรูปภาพเพื่อจำแนก'),
                  ),
          ),
          
          // Results
          if (_results.isNotEmpty)
            Container(
              height: 200,
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    'ผลการจำแนก:',
                    style: Theme.of(context).textTheme.titleMedium,
                  ),
                  const SizedBox(height: 8),
                  Expanded(
                    child: ListView.builder(
                      itemCount: _results.length,
                      itemBuilder: (context, index) {
                        final result = _results[index];
                        return Row(
                          children: [
                            Expanded(
                              flex: 3,
                              child: Text(result.label),
                            ),
                            Expanded(
                              flex: 4,
                              child: LinearProgressIndicator(
                                value: result.confidence,
                                backgroundColor: Colors.grey.shade200,
                              ),
                            ),
                            SizedBox(
                              width: 60,
                              child: Text(
                                '${(result.confidence * 100).toStringAsFixed(1)}%',
                                textAlign: TextAlign.right,
                                style: const TextStyle(fontSize: 12),
                              ),
                            ),
                          ],
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),
          
          // Buttons
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: _isClassifying ? null : _captureFromCamera,
                    icon: const Icon(Icons.camera),
                    label: const Text('ถ่ายรูป'),
                  ),
                ),
                const SizedBox(width: 8),
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _isClassifying ? null : _pickFromGallery,
                    icon: const Icon(Icons.photo),
                    label: const Text('เลือกรูป'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
  
  Future<void> _captureFromCamera() async {
    final picker = ImagePicker();
    final image = await picker.pickImage(source: ImageSource.camera);
    if (image != null) await _classifyImage(File(image.path));
  }
  
  Future<void> _pickFromGallery() async {
    final picker = ImagePicker();
    final image = await picker.pickImage(source: ImageSource.gallery);
    if (image != null) await _classifyImage(File(image.path));
  }
  
  @override
  void dispose() {
    _classifier.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 460: NLP Features

### Natural Language Processing

```dart
// lib/features/ml/services/nlp_service.dart
import 'package:google_mlkit_language_id/google_mlkit_language_id.dart';
import 'package:google_mlkit_translation/google_mlkit_translation.dart';

class NLPService {
  final LanguageIdentifier _languageIdentifier;
  final Map<String, OnDeviceTranslator> _translators = {};
  
  NLPService()
      : _languageIdentifier = LanguageIdentifier(confidenceThreshold: 0.5);
  
  // Language Detection
  Future<String?> detectLanguage(String text) async {
    return _languageIdentifier.identifyLanguage(text);
  }
  
  Future<List<IdentifiedLanguage>> detectLanguages(String text) async {
    return _languageIdentifier.identifyPossibleLanguages(text);
  }
  
  // Translation
  Future<String> translate(
    String text, {
    required String fromLanguage,
    required String toLanguage,
  }) async {
    final key = '${fromLanguage}_$toLanguage';
    
    if (!_translators.containsKey(key)) {
      final source = TranslateLanguage.values.firstWhere(
        (l) => l.bcpCode == fromLanguage,
      );
      final target = TranslateLanguage.values.firstWhere(
        (l) => l.bcpCode == toLanguage,
      );
      
      _translators[key] = OnDeviceTranslator(
        sourceLanguage: source,
        targetLanguage: target,
      );
      
      // Download model if needed
      final modelManager = OnDeviceTranslatorModelManager();
      final isDownloaded = await modelManager.isModelDownloaded(target);
      
      if (!isDownloaded) {
        await modelManager.downloadModel(target);
      }
    }
    
    return _translators[key]!.translateText(text);
  }
  
  // Smart Reply
  Future<List<String>> getSmartReplies(
    List<ConversationMessage> conversation,
  ) async {
    final smartReply = SmartReply();
    
    final history = conversation.map((msg) {
      return msg.isLocal
          ? SmartReplySuggestion.localUserMessage(
              msg.text,
              timestamp: msg.timestamp.millisecondsSinceEpoch,
            )
          : SmartReplySuggestion.remoteUserMessage(
              msg.text,
              timestamp: msg.timestamp.millisecondsSinceEpoch,
              userId: msg.userId,
            );
    }).toList();
    
    final response = await smartReply.suggestReplies(history);
    
    await smartReply.close();
    
    return response.suggestions.map((s) => s.text).toList();
  }
  
  void dispose() {
    _languageIdentifier.close();
    for (final translator in _translators.values) {
      translator.close();
    }
  }
}

// Smart Reply Widget
class SmartReplyWidget extends StatefulWidget {
  final List<ConversationMessage> conversation;
  final Function(String) onReplySelected;
  
  const SmartReplyWidget({
    super.key,
    required this.conversation,
    required this.onReplySelected,
  });
  
  @override
  State<SmartReplyWidget> createState() => _SmartReplyWidgetState();
}

class _SmartReplyWidgetState extends State<SmartReplyWidget> {
  final NLPService _nlpService = NLPService();
  List<String> _suggestions = [];
  
  @override
  void initState() {
    super.initState();
    _loadSuggestions();
  }
  
  Future<void> _loadSuggestions() async {
    try {
      final suggestions = await _nlpService.getSmartReplies(
        widget.conversation,
      );
      setState(() => _suggestions = suggestions);
    } catch (e) {
      print('Smart reply error: $e');
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (_suggestions.isEmpty) return const SizedBox.shrink();
    
    return SingleChildScrollView(
      scrollDirection: Axis.horizontal,
      padding: const EdgeInsets.symmetric(horizontal: 8),
      child: Row(
        children: _suggestions.map((suggestion) {
          return Padding(
            padding: const EdgeInsets.only(right: 8),
            child: ActionChip(
              label: Text(suggestion),
              onPressed: () => widget.onReplySelected(suggestion),
            ),
          );
        }).toList(),
      ),
    );
  }
  
  @override
  void dispose() {
    _nlpService.dispose();
    super.dispose();
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **Google ML Kit**: Text Recognition, Face Detection, Barcode Scanning
- **Pose Detection**: ตรวจจับท่าทางร่างกาย
- **Object Detection**: ตรวจจับ object ในรูปภาพ
- **TensorFlow Lite**: Run custom ML models on-device
- **On-device ML**: ประโยชน์ของ ML บนอุปกรณ์
- **OpenAI API**: Chat, Vision, Image Generation, Embeddings
- **Image Classification**: จำแนกรูปภาพด้วย TFLite
- **NLP Features**: Language detection, Translation, Smart Reply

## แบบฝึกหัด

1. สร้าง OCR app ที่ถ่ายรูปแล้ว extract ข้อความ
2. สร้าง QR Code scanner ที่รองรับหลาย format
3. Implement image classifier ด้วย MobileNet model
4. สร้าง AI chatbot โดยใช้ OpenAI API
5. เพิ่ม language translation ด้วย ML Kit Translation

---

[⬅️ Part 45](part_45.md) | [Part 47 ➡️](part_47.md)

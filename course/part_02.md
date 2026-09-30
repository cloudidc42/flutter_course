# Part 02: Dart Language Fundamentals
## ขั้นตอนที่ 11-20

---

## สารบัญ
1. [Dart คืออะไร](#dart-คืออะไร)
2. [Variables และ Data Types](#variables-และ-data-types)
3. [Operators](#operators)
4. [Control Flow](#control-flow)
5. [Functions](#functions)
6. [Null Safety](#null-safety)
7. [Collections (List, Set, Map)](#collections)
8. [String Interpolation](#string-interpolation)
9. [Type System](#type-system)
10. [Error Handling](#error-handling)

---

## ขั้นตอนที่ 11: Dart คืออะไร?

Dart เป็นภาษาโปรแกรมมิ่งที่พัฒนาโดย Google ออกแบบมาสำหรับการพัฒนาแอปพลิเคชัน โดยเฉพาะร่วมกับ Flutter

### ลักษณะเด่นของ Dart

```
Dart Features:
├── Strongly Typed (แต่สามารถ infer ได้)
├── Object-Oriented
├── Null Safety (Sound Null Safety)
├── Async/Await Support
├── AOT & JIT Compilation
├── Garbage Collection
└── Modern Syntax (คล้าย Java, C#, JavaScript)
```

### ทดลองใช้ Dart

```dart
// ไฟล์ hello.dart - รันได้เลยด้วย: dart hello.dart

void main() {
  // แสดงข้อความ
  print('สวัสดี Dart!');
  print('Hello, World!');
  
  // คำนวณพื้นฐาน
  int result = 10 + 20;
  print('10 + 20 = $result');
  
  // String
  String name = 'Flutter Developer';
  print('คุณคือ: $name');
  
  // Boolean
  bool isFlutterAwesome = true;
  print('Flutter เจ๋งไหม? $isFlutterAwesome');
}
```

---

## ขั้นตอนที่ 12: Variables และ Data Types

### การประกาศตัวแปร

```dart
void main() {
  // ===== วิธีที่ 1: ระบุ Type โดยตรง =====
  int age = 25;
  double height = 175.5;
  String name = 'สมชาย';
  bool isStudent = true;
  
  // ===== วิธีที่ 2: ใช้ var (Type Inference) =====
  var city = 'Bangkok';       // String
  var year = 2024;            // int
  var pi = 3.14159;           // double
  var isDeveloper = true;     // bool
  
  // ===== วิธีที่ 3: ใช้ final (ค่าเปลี่ยนได้ครั้งเดียว) =====
  final String country = 'Thailand';
  final currentYear = DateTime.now().year;  // กำหนดตอน runtime
  
  // ===== วิธีที่ 4: ใช้ const (Compile-time constant) =====
  const double gravity = 9.81;
  const String appName = 'MyApp';
  const int maxRetries = 3;
  
  // ===== var vs final vs const =====
  var mutableVar = 'เปลี่ยนได้';
  mutableVar = 'เปลี่ยนแล้ว';  // OK
  
  final String immutableFinal = 'ไม่เปลี่ยน';
  // immutableFinal = 'ผิด!';  // Error!
  
  const int immutableConst = 42;
  // immutableConst = 43;  // Error!
  
  print('Age: $age, Name: $name, City: $city');
}
```

### Data Types หลัก

```dart
void main() {
  // ===== int =====
  int positiveInt = 100;
  int negativeInt = -50;
  int hexInt = 0xFF;      // Hexadecimal
  int binaryInt = 0b1010; // Binary
  
  print('int: $positiveInt, $negativeInt');
  print('hex: $hexInt');       // 255
  print('binary: $binaryInt'); // 10
  
  // ===== double =====
  double pi = 3.14159265358979;
  double scientific = 1.42e5;   // 142000.0
  double infinity = double.infinity;
  double notANumber = double.nan;
  
  print('pi: $pi');
  print('scientific: $scientific');
  
  // ===== String =====
  String singleQuote = 'Single quotes';
  String doubleQuote = "Double quotes";
  String multiLine = '''
    นี่คือ
    ข้อความ
    หลายบรรทัด
  ''';
  String rawString = r'Raw: \n ไม่ escape';
  
  print(multiLine);
  print(rawString);  // แสดง \n จริงๆ
  
  // ===== bool =====
  bool trueValue = true;
  bool falseValue = false;
  bool comparison = 5 > 3;  // true
  bool equality = 'a' == 'a';  // true
  
  // ===== num =====
  // num เป็น parent class ของ int และ double
  num anyNumber = 42;     // สามารถเป็น int หรือ double
  anyNumber = 3.14;       // OK
  
  // ===== dynamic =====
  // Avoid using dynamic unless necessary!
  dynamic anything = 'string';
  anything = 42;        // OK แต่ไม่แนะนำ
  anything = [1, 2, 3]; // OK แต่ไม่แนะนำ
}
```

### Type Conversion

```dart
void main() {
  // String to int
  String strNum = '42';
  int parsedInt = int.parse(strNum);
  int? tryParseInt = int.tryParse('not a number');  // returns null
  
  print('Parsed: $parsedInt');       // 42
  print('Try parse: $tryParseInt');  // null
  
  // String to double
  String strDouble = '3.14';
  double parsedDouble = double.parse(strDouble);
  
  // int to String
  int number = 100;
  String numStr = number.toString();
  String numStr2 = '$number';  // String interpolation
  
  // int to double
  int intVal = 5;
  double doubleVal = intVal.toDouble();
  
  // double to int (ตัดทศนิยม)
  double doubleNum = 9.99;
  int intNum = doubleNum.toInt();      // 9 (ไม่ปัด)
  int roundNum = doubleNum.round();    // 10 (ปัด)
  int floorNum = doubleNum.floor();    // 9 (ปัดลง)
  int ceilNum = doubleNum.ceil();      // 10 (ปัดขึ้น)
  
  print('double to int: $intNum, round: $roundNum');
  print('floor: $floorNum, ceil: $ceilNum');
  
  // String methods
  String text = '  Hello, Flutter!  ';
  print(text.trim());           // ตัด whitespace
  print(text.toUpperCase());    // ตัวใหญ่ทั้งหมด
  print(text.toLowerCase());    // ตัวเล็กทั้งหมด
  print(text.contains('Flutter')); // true
  print(text.replaceAll('Flutter', 'Dart')); // แทนที่
  print(text.split(','));       // แบ่งเป็น List
  print(text.length);           // ความยาว
  print(text.isEmpty);          // ว่างเปล่าหรือไม่
  print(text.isNotEmpty);       // ไม่ว่างเปล่าหรือไม่
  print(text.startsWith('  ')); // ขึ้นต้นด้วย
  print(text.endsWith('  '));   // ลงท้ายด้วย
  print(text.indexOf('o'));     // หา index แรก
  print(text.substring(2, 7)); // ตัดข้อความ
}
```

---

## ขั้นตอนที่ 13: Operators

```dart
void main() {
  // ===== Arithmetic Operators =====
  int a = 10, b = 3;
  
  print(a + b);    // 13 (บวก)
  print(a - b);    // 7  (ลบ)
  print(a * b);    // 30 (คูณ)
  print(a / b);    // 3.3333... (หาร -> double)
  print(a ~/ b);   // 3  (หารเอาผลลัพธ์เป็น int)
  print(a % b);    // 1  (modulo/เศษ)
  
  double result = a / b;
  print(result);  // 3.3333...
  
  // ===== Assignment Operators =====
  int x = 5;
  x += 3;   // x = x + 3 = 8
  x -= 2;   // x = x - 2 = 6
  x *= 4;   // x = x * 4 = 24
  x ~/= 5;  // x = x ~/ 5 = 4
  x %= 3;   // x = x % 3 = 1
  
  // ===== Comparison Operators =====
  int p = 10, q = 20;
  
  print(p == q);   // false (เท่ากัน)
  print(p != q);   // true  (ไม่เท่ากัน)
  print(p > q);    // false (มากกว่า)
  print(p < q);    // true  (น้อยกว่า)
  print(p >= q);   // false (มากกว่าหรือเท่ากัน)
  print(p <= q);   // true  (น้อยกว่าหรือเท่ากัน)
  
  // ===== Logical Operators =====
  bool t = true, f = false;
  
  print(t && f);   // false (AND)
  print(t || f);   // true  (OR)
  print(!t);       // false (NOT)
  
  // Short-circuit evaluation
  // && จะไม่ evaluate ด้านขวาถ้าด้านซ้าย false
  // || จะไม่ evaluate ด้านขวาถ้าด้านซ้าย true
  
  // ===== Bitwise Operators =====
  int m = 0b1010;  // 10
  int n = 0b1100;  // 12
  
  print(m & n);    // 0b1000 = 8   (AND)
  print(m | n);    // 0b1110 = 14  (OR)
  print(m ^ n);    // 0b0110 = 6   (XOR)
  print(~m);       // bitwise NOT
  print(m << 1);   // 0b10100 = 20 (left shift)
  print(m >> 1);   // 0b0101 = 5   (right shift)
  
  // ===== Type Test Operators =====
  dynamic value = 'Hello';
  
  print(value is String);      // true
  print(value is int);         // false
  print(value is! String);     // false (ไม่ใช่ String)
  
  // ===== Conditional Operators =====
  int age = 20;
  
  // Ternary operator
  String status = age >= 18 ? 'ผู้ใหญ่' : 'เด็ก';
  print(status);  // ผู้ใหญ่
  
  // Null-aware operators
  String? nullableStr = null;
  
  // ?? (null coalescing)
  String result2 = nullableStr ?? 'Default Value';
  print(result2);  // Default Value
  
  // ??= (null-aware assignment)
  String? str;
  str ??= 'Assigned if null';
  print(str);  // Assigned if null
  str ??= 'Not assigned';  // str ไม่ใช่ null แล้ว
  print(str);  // Assigned if null (ไม่เปลี่ยน)
  
  // ?. (null-safe member access)
  String? maybeNull = null;
  int? length = maybeNull?.length;  // null แทนที่จะ crash
  print(length);  // null
  
  // Cascade operator (..)
  // ใช้เรียก method หลายๆ ครั้งบน object เดียว
  StringBuffer buffer = StringBuffer()
    ..write('Hello')
    ..write(', ')
    ..write('World')
    ..write('!');
  print(buffer.toString());  // Hello, World!
  
  // ===== Spread Operators =====
  List<int> list1 = [1, 2, 3];
  List<int> list2 = [4, 5, 6];
  List<int> combined = [...list1, ...list2];
  print(combined);  // [1, 2, 3, 4, 5, 6]
  
  List<int>? nullableList = null;
  List<int> safe = [...?nullableList, 7, 8];  // ปลอดภัยกับ null
  print(safe);  // [7, 8]
}
```

---

## ขั้นตอนที่ 14: Control Flow

```dart
void main() {
  // ===== if-else =====
  int score = 85;
  
  if (score >= 90) {
    print('เกรด A');
  } else if (score >= 80) {
    print('เกรด B');
  } else if (score >= 70) {
    print('เกรด C');
  } else if (score >= 60) {
    print('เกรด D');
  } else {
    print('เกรด F');
  }
  
  // ===== switch-case =====
  String day = 'Monday';
  
  switch (day) {
    case 'Monday':
    case 'Tuesday':
    case 'Wednesday':
    case 'Thursday':
    case 'Friday':
      print('วันทำงาน');
      break;
    case 'Saturday':
    case 'Sunday':
      print('วันหยุด');
      break;
    default:
      print('วันไม่ถูกต้อง');
  }
  
  // Dart 3.0+ Pattern Matching in switch
  String result = switch (score) {
    >= 90 => 'A',
    >= 80 => 'B',
    >= 70 => 'C',
    >= 60 => 'D',
    _ => 'F',
  };
  print('Grade: $result');
  
  // ===== for loop =====
  for (int i = 0; i < 5; i++) {
    print('i = $i');
  }
  
  // for-in loop
  List<String> fruits = ['apple', 'banana', 'cherry'];
  for (String fruit in fruits) {
    print('Fruit: $fruit');
  }
  
  // forEach
  fruits.forEach((fruit) => print('- $fruit'));
  
  // ===== while loop =====
  int count = 0;
  while (count < 3) {
    print('count = $count');
    count++;
  }
  
  // ===== do-while loop =====
  int num = 0;
  do {
    print('num = $num');
    num++;
  } while (num < 3);
  
  // ===== break & continue =====
  for (int i = 0; i < 10; i++) {
    if (i == 3) continue;  // ข้ามรอบนี้
    if (i == 7) break;     // ออกจาก loop
    print('i = $i');
  }
  // Output: 0, 1, 2, 4, 5, 6
  
  // ===== Labeled loops =====
  outer:
  for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
      if (j == 1) continue outer;  // continue outer loop
      print('i=$i, j=$j');
    }
  }
}
```

---

## ขั้นตอนที่ 15: Functions

```dart
// ===== Basic Function =====
int add(int a, int b) {
  return a + b;
}

// ===== Return Type Void =====
void greet(String name) {
  print('Hello, $name!');
}

// ===== Optional Parameters =====
// 1. Named Parameters (แนะนำ!)
void createUser({
  required String name,          // required - ต้องส่งมา
  int age = 18,                  // optional with default
  String? email,                 // optional nullable
}) {
  print('Name: $name, Age: $age, Email: $email');
}

// 2. Positional Optional Parameters
String buildAddress(String street, [String? city, String country = 'Thailand']) {
  return '$street, ${city ?? 'Unknown City'}, $country';
}

// ===== Arrow Function (=>)  =====
int multiply(int a, int b) => a * b;
String formatName(String first, String last) => '$first $last';
bool isEven(int n) => n % 2 == 0;

// ===== Higher-Order Functions =====
void executeFunction(Function callback) {
  print('Before calling...');
  callback();
  print('After calling...');
}

int applyOperation(int a, int b, int Function(int, int) operation) {
  return operation(a, b);
}

// ===== Closures =====
Function makeCounter() {
  int count = 0;
  return () {
    count++;
    return count;
  };
}

// ===== Recursive Functions =====
int factorial(int n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

int fibonacci(int n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

void main() {
  // Basic function calls
  print(add(5, 3));        // 8
  greet('Flutter');         // Hello, Flutter!
  
  // Named parameters
  createUser(name: 'สมชาย', age: 25, email: 'somchai@example.com');
  createUser(name: 'มาลี');  // age ใช้ default = 18
  
  // Positional optional
  print(buildAddress('123 Main St'));              // 123 Main St, Unknown City, Thailand
  print(buildAddress('456 Oak Ave', 'Bangkok'));  // 456 Oak Ave, Bangkok, Thailand
  
  // Arrow functions
  print(multiply(4, 6));     // 24
  print(formatName('John', 'Doe'));  // John Doe
  print(isEven(7));          // false
  
  // Higher-order functions
  executeFunction(() => print('Executing!'));
  
  int result = applyOperation(10, 5, (a, b) => a + b);
  print('Result: $result');  // 15
  
  int result2 = applyOperation(10, 5, (a, b) => a * b);
  print('Result2: $result2');  // 50
  
  // Closures
  var counter = makeCounter();
  print(counter());  // 1
  print(counter());  // 2
  print(counter());  // 3
  
  // Recursive
  print(factorial(5));    // 120
  print(fibonacci(10));   // 55
  
  // ===== Anonymous Functions =====
  List<int> numbers = [3, 1, 4, 1, 5, 9, 2, 6];
  
  // sort with custom comparator
  numbers.sort((a, b) => a.compareTo(b));
  print(numbers);  // [1, 1, 2, 3, 4, 5, 6, 9]
  
  // map - transform each element
  List<int> doubled = numbers.map((n) => n * 2).toList();
  print(doubled);  // [2, 2, 4, 6, 8, 10, 12, 18]
  
  // filter - keep elements matching condition
  List<int> evenNumbers = numbers.where((n) => n % 2 == 0).toList();
  print(evenNumbers);  // [2, 4, 6]
  
  // reduce - fold to single value
  int sum = numbers.reduce((acc, n) => acc + n);
  print('Sum: $sum');  // 31
  
  // fold with initial value
  int product = numbers.fold(1, (acc, n) => acc * n);
  print('Product: $product');
  
  // ===== Function as Variable =====
  int Function(int, int) addFunc = add;
  print(addFunc(3, 4));  // 7
  
  // typedef
  // typedef Operation = int Function(int, int);
  // Operation myOp = add;
}
```

---

## ขั้นตอนที่ 16: Null Safety

Null Safety เป็นหนึ่งในฟีเจอร์สำคัญที่สุดของ Dart (เวอร์ชัน 2.12+)

```dart
void main() {
  // ===== Non-nullable vs Nullable =====
  
  // Non-nullable: ต้องมีค่าเสมอ
  String name = 'Flutter';
  // name = null;  // Compile Error!
  
  // Nullable: อาจเป็น null ได้ (เพิ่ม ?)
  String? nullableName = 'Dart';
  nullableName = null;  // OK
  
  int nonNullableAge = 25;
  int? nullableAge = null;
  
  // ===== Null Check =====
  String? userInput = getUserInput();
  
  // วิธีที่ 1: if check
  if (userInput != null) {
    print(userInput.length);  // Safe!
  }
  
  // วิธีที่ 2: ?. (null-safe access)
  print(userInput?.length);  // null ถ้า userInput เป็น null
  
  // วิธีที่ 3: ?? (null coalescing)
  String result = userInput ?? 'Default';
  print(result);
  
  // วิธีที่ 4: ! (null assertion - ใช้เมื่อแน่ใจ 100%)
  // ระวัง! ถ้า null จะ throw exception
  if (userInput != null) {
    String definitelyNotNull = userInput!;  // ยืนยันว่าไม่ null
    print(definitelyNotNull.length);
  }
  
  // ===== Late Variables =====
  // late บอกว่าจะ initialize ทีหลัง (ก่อน use)
  late String lateVar;
  lateVar = 'Initialized later';  // ต้อง initialize ก่อน access
  print(lateVar);
  
  // late ใน class
  // late String _name; // จะ initialize ใน initState() หรือ constructor
  
  // ===== Null Safety Patterns =====
  
  // Pattern 1: Early return
  String? text = getOptionalText();
  if (text == null) return;
  print(text.length);  // Dart รู้ว่า text ไม่ใช่ null แล้ว
  
  // Pattern 2: Cascade with null safety
  StringBuffer? maybeBuffer;
  maybeBuffer
    ?..write('Hello')  // ไม่ทำอะไรถ้า maybeBuffer เป็น null
    ..write(' World');
  
  // Pattern 3: List operations
  List<String?> mixedList = ['a', null, 'b', null, 'c'];
  List<String> nonNullList = mixedList
      .where((item) => item != null)
      .map((item) => item!)  // ปลอดภัยเพราะผ่าน where แล้ว
      .toList();
  print(nonNullList);  // [a, b, c]
  
  // หรือใช้ whereType
  List<String> nonNullList2 = mixedList.whereType<String>().toList();
  print(nonNullList2);  // [a, b, c]
}

String? getUserInput() {
  return null;  // Simulate user not typing anything
}

String? getOptionalText() {
  return 'Some text';
}

// ===== Null Safety in Functions =====
String greetUser(String? name) {
  // วิธีที่ 1: ใช้ ?? 
  return 'Hello, ${name ?? 'Guest'}!';
}

// วิธีที่ 2: early return
String processInput(String? input) {
  if (input == null || input.isEmpty) {
    return 'No input provided';
  }
  // ณ จุดนี้ Dart รู้ว่า input ไม่ใช่ null และไม่ใช่ empty string
  return input.toUpperCase();
}

// Required vs Optional in constructor
class UserProfile {
  final String name;         // required non-nullable
  final String? bio;         // optional nullable
  final int age;             // required non-nullable
  
  const UserProfile({
    required this.name,
    this.bio,              // ไม่มี required = optional
    required this.age,
  });
}
```

---

## ขั้นตอนที่ 17: Collections

```dart
void main() {
  // ===================================================
  // LIST
  // ===================================================
  
  // สร้าง List
  List<int> numbers = [1, 2, 3, 4, 5];
  List<String> fruits = ['apple', 'banana', 'cherry'];
  List<dynamic> mixed = [1, 'hello', true, 3.14];
  
  // var ก็ได้
  var colors = ['red', 'green', 'blue'];
  
  // Fixed-length list
  List<int> fixedList = List.filled(5, 0);  // [0, 0, 0, 0, 0]
  
  // Generate list
  List<int> generated = List.generate(5, (index) => index * 2);
  // [0, 2, 4, 6, 8]
  
  // Empty list
  List<String> emptyList = [];
  List<String> emptyList2 = <String>[];
  
  // ===== CRUD Operations =====
  
  // Create / Add
  fruits.add('date');              // เพิ่มท้าย
  fruits.addAll(['elderberry', 'fig']);  // เพิ่มหลายตัว
  fruits.insert(1, 'avocado');    // แทรกที่ index 1
  fruits.insertAll(0, ['acai', 'apricot']);  // แทรกหลายตัวที่ index 0
  
  // Read
  print(fruits[0]);              // เข้าถึงด้วย index
  print(fruits.first);           // ตัวแรก
  print(fruits.last);            // ตัวสุดท้าย
  print(fruits.length);          // ความยาว
  print(fruits.isEmpty);         // ว่างเปล่า?
  print(fruits.isNotEmpty);      // ไม่ว่างเปล่า?
  
  // Update
  fruits[0] = 'APPLE';           // แก้ไขที่ index 0
  
  // Delete
  fruits.remove('banana');       // ลบด้วยค่า
  fruits.removeAt(0);            // ลบด้วย index
  fruits.removeLast();           // ลบตัวสุดท้าย
  fruits.removeRange(0, 2);      // ลบ range
  fruits.clear();                // ลบทั้งหมด
  
  // ===== Useful List Methods =====
  List<int> nums = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
  
  print(nums.contains(4));        // true
  print(nums.indexOf(5));         // 4 (index แรก)
  print(nums.lastIndexOf(5));     // 10 (index สุดท้าย)
  print(nums.sublist(2, 5));      // [4, 1, 5]
  
  // Functional methods
  List<int> doubled = nums.map((n) => n * 2).toList();
  List<int> evens = nums.where((n) => n % 2 == 0).toList();
  int sum = nums.reduce((a, b) => a + b);
  bool anyNegative = nums.any((n) => n < 0);      // false
  bool allPositive = nums.every((n) => n > 0);    // true
  
  // Sort
  List<int> sorted = [...nums]..sort();           // copy แล้ว sort
  List<int> reversed = sorted.reversed.toList();
  
  // Unique values
  List<int> unique = nums.toSet().toList();
  
  print('Sum: $sum, Evens: $evens');
  
  // ===================================================
  // SET
  // ===================================================
  
  // Set ไม่มีค่าซ้ำ และไม่มี order
  Set<String> tags = {'flutter', 'dart', 'mobile'};
  Set<int> uniqueNums = {1, 2, 3, 2, 1};  // {1, 2, 3}
  
  // Empty set
  Set<String> emptySet = {};        // ระวัง! {} เป็น Map
  Set<String> emptySet2 = <String>{};  // ชัดเจนกว่า
  Set<String> emptySet3 = Set<String>();
  
  // Add/Remove
  tags.add('ios');
  tags.addAll(['android', 'web']);
  tags.remove('ios');
  
  // Set Operations
  Set<int> setA = {1, 2, 3, 4, 5};
  Set<int> setB = {3, 4, 5, 6, 7};
  
  print(setA.union(setB));         // {1, 2, 3, 4, 5, 6, 7}
  print(setA.intersection(setB));  // {3, 4, 5}
  print(setA.difference(setB));    // {1, 2}
  
  // ===================================================
  // MAP
  // ===================================================
  
  // Map คือ Key-Value pairs (Dictionary)
  Map<String, int> ages = {
    'Alice': 25,
    'Bob': 30,
    'Charlie': 22,
  };
  
  Map<String, dynamic> person = {
    'name': 'สมชาย',
    'age': 28,
    'isStudent': false,
    'hobbies': ['reading', 'coding'],
  };
  
  // Empty map
  Map<String, int> emptyMap = {};
  Map<String, int> emptyMap2 = <String, int>{};
  
  // ===== CRUD Operations =====
  
  // Create / Add
  ages['Diana'] = 27;              // เพิ่ม key-value ใหม่
  ages.addAll({'Eve': 24, 'Frank': 31});  // เพิ่มหลายคู่
  
  // Read
  print(ages['Alice']);            // 25
  print(ages['Unknown']);          // null (ไม่มี key นี้)
  print(ages['Unknown'] ?? 0);    // 0 (ใช้ default)
  
  // putIfAbsent - เพิ่มเฉพาะถ้ายังไม่มี key
  ages.putIfAbsent('Grace', () => 29);
  
  // Update
  ages['Alice'] = 26;             // update ค่า
  ages.update('Bob', (v) => v + 1);  // update ด้วย function
  ages.updateAll((key, value) => value + 1);  // update ทุก value
  
  // Delete
  ages.remove('Frank');           // ลบ key
  ages.removeWhere((k, v) => v < 25);  // ลบตาม condition
  
  // Check
  print(ages.containsKey('Alice'));    // true
  print(ages.containsValue(30));      // true
  print(ages.length);
  print(ages.isEmpty);
  
  // Iterate
  ages.forEach((key, value) {
    print('$key: $value');
  });
  
  // Keys and Values
  print(ages.keys.toList());      // [Alice, Bob, ...]
  print(ages.values.toList());    // [26, 31, ...]
  print(ages.entries.toList());   // [MapEntry(Alice: 26), ...]
  
  // Map operations
  Map<String, int> filtered = Map.fromEntries(
    ages.entries.where((e) => e.value > 25),
  );
  print(filtered);
  
  // ===================================================
  // COLLECTION CONTROL FLOW
  // ===================================================
  
  bool showExtra = true;
  List<String> menu = [
    'Home',
    'About',
    if (showExtra) 'Extra Page',  // conditional
    'Contact',
  ];
  print(menu);  // [Home, About, Extra Page, Contact]
  
  List<int> nums2 = [1, 2, 3];
  List<int> expanded = [
    0,
    for (int n in nums2) n * 10,  // for loop in literal
    40,
  ];
  print(expanded);  // [0, 10, 20, 30, 40]
}
```

---

## ขั้นตอนที่ 18: String Interpolation และ Multi-line Strings

```dart
void main() {
  String name = 'Flutter';
  int version = 3;
  double rating = 4.8;
  
  // ===== String Interpolation =====
  print('Hello, $name!');
  print('Version: $version');
  print('Rating: $rating');
  
  // Expression ใน ${}
  print('${name.toUpperCase()} v$version');
  print('${name.length} characters');
  print('${version * 2} times');
  
  // Complex expressions
  List<String> items = ['a', 'b', 'c'];
  print('Items: ${items.join(', ')}');
  print('Count: ${items.length}');
  print('Is empty: ${items.isEmpty ? "yes" : "no"}');
  
  // ===== Multi-line Strings =====
  String paragraph = '''
Flutter คือ UI Toolkit จาก Google
ที่ช่วยให้สร้างแอปข้ามแพลตฟอร์มได้
จาก Codebase เดียวกัน
  ''';
  
  String json = """
  {
    "name": "Flutter",
    "version": 3,
    "features": ["fast", "beautiful", "productive"]
  }
  """;
  
  print(paragraph);
  print(json);
  
  // ===== Raw Strings =====
  String regex = r'\d+\.\d+';         // regex pattern
  String path = r'C:\Users\Flutter';   // Windows path
  print(regex);   // \d+\.\d+  (ไม่ escape)
  print(path);    // C:\Users\Flutter
  
  // ===== String Methods =====
  String text = 'Hello, Flutter World!';
  
  // Case
  print(text.toUpperCase());      // HELLO, FLUTTER WORLD!
  print(text.toLowerCase());      // hello, flutter world!
  
  // Trim
  String padded = '   spaces   ';
  print(padded.trim());           // 'spaces'
  print(padded.trimLeft());       // 'spaces   '
  print(padded.trimRight());      // '   spaces'
  
  // Split & Join
  List<String> words = text.split(' ');
  print(words);  // [Hello,, Flutter, World!]
  print(words.join('-'));  // Hello,-Flutter-World!
  
  // Replace
  print(text.replaceAll('Flutter', 'Dart'));  // Hello, Dart World!
  print(text.replaceFirst('l', 'L'));          // HeLlo, Flutter World!
  print(text.replaceRange(0, 5, 'Goodbye'));   // Goodbye, Flutter World!
  
  // Search
  print(text.contains('Flutter'));     // true
  print(text.startsWith('Hello'));     // true
  print(text.endsWith('World!'));      // true
  print(text.indexOf('Flutter'));      // 7
  print(text.lastIndexOf('l'));        // 14
  
  // Substring
  print(text.substring(7, 14));       // Flutter
  
  // Padding
  String num = '42';
  print(num.padLeft(5));              // '   42'
  print(num.padRight(5));             // '42   '
  print(num.padLeft(5, '0'));         // '00042'
  
  // ===== String Buffer (ประสิทธิภาพสูงกว่า) =====
  StringBuffer sb = StringBuffer();
  for (int i = 0; i < 5; i++) {
    sb.write('Line $i\n');
  }
  print(sb.toString());
  
  // ===== Format Numbers =====
  double price = 1234567.89;
  
  // toStringAsFixed
  print(price.toStringAsFixed(2));    // 1234567.89
  print(price.toStringAsFixed(0));    // 1234568
  
  // toStringAsPrecision
  print(price.toStringAsPrecision(6)); // 1.23457e+6
}
```

---

## ขั้นตอนที่ 19: Type System

```dart
void main() {
  // ===== Type Inference =====
  var number = 42;           // inferred as int
  var text = 'Hello';        // inferred as String
  var list = [1, 2, 3];      // inferred as List<int>
  var map = {'a': 1, 'b': 2}; // inferred as Map<String, int>
  
  // เช็ค type ด้วย runtimeType
  print(number.runtimeType);  // int
  print(text.runtimeType);    // String
  print(list.runtimeType);    // List<int>
  
  // ===== Type Casting =====
  num anyNum = 3.14;
  
  // is check
  if (anyNum is double) {
    print('It is a double: $anyNum');
  }
  
  // as cast (อาจ throw exception ถ้าไม่ตรง)
  double asDouble = anyNum as double;
  print(asDouble);
  
  // Safe cast pattern
  dynamic value = 'Hello';
  if (value is String) {
    String s = value;  // auto-cast ใน block นี้
    print(s.toUpperCase());
  }
  
  // ===== Generic Types =====
  List<String> stringList = ['a', 'b', 'c'];
  List<int> intList = [1, 2, 3];
  
  // Generic function
  T first<T>(List<T> list) {
    return list.first;
  }
  
  print(first(stringList));  // a
  print(first(intList));     // 1
  
  // ===== typedef =====
  // typedef สำหรับ function types
  typedef Callback = void Function(String);
  typedef Transform<T> = T Function(T);
  
  void doSomething(Callback callback) {
    callback('Hello from typedef');
  }
  
  doSomething((s) => print(s));
}

// Generic Class
class Pair<A, B> {
  final A first;
  final B second;
  
  const Pair(this.first, this.second);
  
  @override
  String toString() => 'Pair($first, $second)';
}

// Generic Method
extension ListExtensions<T> on List<T> {
  T? safeGet(int index) {
    if (index < 0 || index >= length) return null;
    return this[index];
  }
}
```

---

## ขั้นตอนที่ 20: Error Handling

```dart
// ===== Custom Exceptions =====
class ValidationException implements Exception {
  final String message;
  ValidationException(this.message);
  
  @override
  String toString() => 'ValidationException: $message';
}

class NetworkException implements Exception {
  final int statusCode;
  final String message;
  
  NetworkException({required this.statusCode, required this.message});
  
  @override
  String toString() => 'NetworkException($statusCode): $message';
}

// ===== Functions that might throw =====
int divideNumbers(int a, int b) {
  if (b == 0) {
    throw ArgumentError('Cannot divide by zero');
  }
  return a ~/ b;
}

void validateAge(int age) {
  if (age < 0) {
    throw ValidationException('Age cannot be negative');
  }
  if (age > 150) {
    throw ValidationException('Age is unrealistically large');
  }
}

void main() {
  // ===== try-catch-finally =====
  try {
    int result = divideNumbers(10, 0);
    print('Result: $result');
  } catch (e) {
    print('Error: $e');
  }
  
  // ===== catch specific exception types =====
  try {
    validateAge(-5);
  } on ValidationException catch (e) {
    print('Validation Error: ${e.message}');
  } on ArgumentError catch (e) {
    print('Argument Error: $e');
  } catch (e) {
    print('Unknown Error: $e');
  }
  
  // ===== with stack trace =====
  try {
    divideNumbers(10, 0);
  } catch (e, stackTrace) {
    print('Error: $e');
    print('Stack Trace:\n$stackTrace');
  }
  
  // ===== finally block =====
  void readFile() {
    // Simulate file operation
    try {
      print('Opening file...');
      throw Exception('File not found');
    } catch (e) {
      print('Error reading file: $e');
    } finally {
      print('Closing file (always runs)');
    }
  }
  
  readFile();
  
  // ===== rethrow =====
  void processData(String data) {
    try {
      if (data.isEmpty) throw ValidationException('Data is empty');
      print('Processing: $data');
    } catch (e) {
      print('Logging error...');
      rethrow;  //던 exception ต่อ
    }
  }
  
  try {
    processData('');
  } catch (e) {
    print('Caught at top level: $e');
  }
  
  // ===== Result Pattern (Functional Error Handling) =====
  // Pattern นิยมใช้ใน Flutter
  
  sealed class Result<T> {}
  
  class Success<T> extends Result<T> {
    final T value;
    Success(this.value);
  }
  
  class Failure<T> extends Result<T> {
    final Exception error;
    Failure(this.error);
  }
  
  Result<int> safeDivide(int a, int b) {
    if (b == 0) {
      return Failure(ArgumentError('Division by zero'));
    }
    return Success(a ~/ b);
  }
  
  Result<int> result = safeDivide(10, 2);
  
  switch (result) {
    case Success(:final value):
      print('Success: $value');
    case Failure(:final error):
      print('Error: $error');
  }
  
  // ===== Assert (Development Only) =====
  // assert ทำงานเฉพาะใน debug mode
  int age = 25;
  assert(age >= 0, 'Age must be non-negative');
  assert(age <= 150, 'Age must be realistic');
  
  // assert ใน constructor
  // const User(this.name, this.age) : assert(age >= 0, 'Age must be >= 0');
}
```

---

## Workshop: Calculator App

สร้าง Calculator App ที่ใช้ความรู้จาก Part 02:

```dart
// lib/main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(const CalculatorApp());
}

class CalculatorApp extends StatelessWidget {
  const CalculatorApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Dart Calculator',
      theme: ThemeData.dark().copyWith(
        colorScheme: ColorScheme.dark(
          primary: Colors.orange,
          secondary: Colors.orangeAccent,
        ),
      ),
      home: const CalculatorScreen(),
    );
  }
}

class CalculatorScreen extends StatefulWidget {
  const CalculatorScreen({super.key});

  @override
  State<CalculatorScreen> createState() => _CalculatorScreenState();
}

class _CalculatorScreenState extends State<CalculatorScreen> {
  String _display = '0';
  double _firstOperand = 0;
  String _operator = '';
  bool _shouldResetDisplay = false;

  void _onDigitPressed(String digit) {
    setState(() {
      if (_shouldResetDisplay || _display == '0') {
        _display = digit;
        _shouldResetDisplay = false;
      } else {
        if (_display.length < 12) {
          _display += digit;
        }
      }
    });
  }

  void _onDecimalPressed() {
    setState(() {
      if (_shouldResetDisplay) {
        _display = '0.';
        _shouldResetDisplay = false;
      } else if (!_display.contains('.')) {
        _display += '.';
      }
    });
  }

  void _onOperatorPressed(String op) {
    setState(() {
      _firstOperand = double.tryParse(_display) ?? 0;
      _operator = op;
      _shouldResetDisplay = true;
    });
  }

  void _onEqualsPressed() {
    if (_operator.isEmpty) return;
    
    setState(() {
      double secondOperand = double.tryParse(_display) ?? 0;
      double result = _calculate(_firstOperand, _operator, secondOperand);
      
      _display = _formatResult(result);
      _operator = '';
      _shouldResetDisplay = true;
    });
  }

  double _calculate(double a, String op, double b) {
    switch (op) {
      case '+': return a + b;
      case '-': return a - b;
      case '×': return a * b;
      case '÷':
        if (b == 0) throw Exception('Division by zero');
        return a / b;
      default: return b;
    }
  }

  String _formatResult(double result) {
    if (result == result.toInt()) {
      return result.toInt().toString();
    }
    String str = result.toStringAsFixed(8);
    while (str.endsWith('0')) str = str.substring(0, str.length - 1);
    if (str.endsWith('.')) str = str.substring(0, str.length - 1);
    return str;
  }

  void _onClear() {
    setState(() {
      _display = '0';
      _firstOperand = 0;
      _operator = '';
      _shouldResetDisplay = false;
    });
  }

  void _onBackspace() {
    setState(() {
      if (_display.length > 1) {
        _display = _display.substring(0, _display.length - 1);
      } else {
        _display = '0';
      }
    });
  }

  void _onPlusMinus() {
    setState(() {
      if (_display != '0') {
        if (_display.startsWith('-')) {
          _display = _display.substring(1);
        } else {
          _display = '-$_display';
        }
      }
    });
  }

  void _onPercent() {
    setState(() {
      double value = double.tryParse(_display) ?? 0;
      _display = _formatResult(value / 100);
    });
  }

  Widget _buildButton(
    String text, {
    Color? color,
    Color? textColor,
    VoidCallback? onPressed,
    int flex = 1,
  }) {
    return Expanded(
      flex: flex,
      child: Padding(
        padding: const EdgeInsets.all(4),
        child: ElevatedButton(
          onPressed: onPressed,
          style: ElevatedButton.styleFrom(
            backgroundColor: color ?? const Color(0xFF333333),
            foregroundColor: textColor ?? Colors.white,
            padding: const EdgeInsets.symmetric(vertical: 20),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(12),
            ),
          ),
          child: Text(
            text,
            style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
          ),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: SafeArea(
        child: Column(
          children: [
            // Display
            Expanded(
              flex: 2,
              child: Container(
                width: double.infinity,
                padding: const EdgeInsets.all(24),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.end,
                  mainAxisAlignment: MainAxisAlignment.end,
                  children: [
                    if (_operator.isNotEmpty)
                      Text(
                        '${_firstOperand % 1 == 0 ? _firstOperand.toInt() : _firstOperand} $_operator',
                        style: const TextStyle(
                          color: Colors.grey,
                          fontSize: 18,
                        ),
                      ),
                    Text(
                      _display,
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: _display.length > 10 ? 36 : 56,
                        fontWeight: FontWeight.w200,
                      ),
                    ),
                  ],
                ),
              ),
            ),
            
            // Buttons
            Expanded(
              flex: 4,
              child: Padding(
                padding: const EdgeInsets.all(8),
                child: Column(
                  children: [
                    // Row 1
                    Row(children: [
                      _buildButton('AC', color: Colors.grey.shade700, onPressed: _onClear),
                      _buildButton('+/-', color: Colors.grey.shade700, onPressed: _onPlusMinus),
                      _buildButton('%', color: Colors.grey.shade700, onPressed: _onPercent),
                      _buildButton('÷', color: Colors.orange, onPressed: () => _onOperatorPressed('÷')),
                    ]),
                    // Row 2
                    Row(children: [
                      _buildButton('7', onPressed: () => _onDigitPressed('7')),
                      _buildButton('8', onPressed: () => _onDigitPressed('8')),
                      _buildButton('9', onPressed: () => _onDigitPressed('9')),
                      _buildButton('×', color: Colors.orange, onPressed: () => _onOperatorPressed('×')),
                    ]),
                    // Row 3
                    Row(children: [
                      _buildButton('4', onPressed: () => _onDigitPressed('4')),
                      _buildButton('5', onPressed: () => _onDigitPressed('5')),
                      _buildButton('6', onPressed: () => _onDigitPressed('6')),
                      _buildButton('-', color: Colors.orange, onPressed: () => _onOperatorPressed('-')),
                    ]),
                    // Row 4
                    Row(children: [
                      _buildButton('1', onPressed: () => _onDigitPressed('1')),
                      _buildButton('2', onPressed: () => _onDigitPressed('2')),
                      _buildButton('3', onPressed: () => _onDigitPressed('3')),
                      _buildButton('+', color: Colors.orange, onPressed: () => _onOperatorPressed('+')),
                    ]),
                    // Row 5
                    Row(children: [
                      _buildButton('⌫', onPressed: _onBackspace),
                      _buildButton('0', onPressed: () => _onDigitPressed('0')),
                      _buildButton('.', onPressed: _onDecimalPressed),
                      _buildButton('=', color: Colors.orange, onPressed: _onEqualsPressed),
                    ]),
                  ],
                ),
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

## สรุป Part 02

ในส่วนนี้คุณได้เรียนรู้:

✅ Variables และ Data Types ของ Dart
✅ Operators ทุกประเภท
✅ Control Flow: if-else, switch, loops
✅ Functions: named params, optional, arrow functions
✅ Null Safety อย่างถูกต้อง
✅ Collections: List, Set, Map
✅ String Interpolation และ methods
✅ Type System และ Generics
✅ Error Handling แบบ professional
✅ สร้าง Calculator App ได้!

## แบบฝึกหัด

1. สร้าง function `calculateBMI(weight, height)` ที่คำนวณ BMI
2. สร้าง function ที่รับ List<int> และคืน median
3. สร้าง Map ที่เก็บนักศึกษาและคะแนน แล้ว sort ตามคะแนน
4. สร้าง custom Exception สำหรับ validation form

---

**ก่อนหน้า:** [Part 01 - การติดตั้งและตั้งค่า Flutter](part_01.md)  
**ต่อไป:** [Part 03 - Dart OOP และ Advanced Types →](part_03.md)

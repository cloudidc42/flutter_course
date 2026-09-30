# Part 03: Dart OOP และ Advanced Types
## ขั้นตอนที่ 21-30

---

## สารบัญ
1. [Classes และ Objects](#classes-และ-objects)
2. [Constructors](#constructors)
3. [Inheritance](#inheritance)
4. [Abstract Classes & Interfaces](#abstract-classes--interfaces)
5. [Mixins](#mixins)
6. [Enums](#enums)
7. [Extensions](#extensions)
8. [Sealed Classes](#sealed-classes)
9. [Async Programming](#async-programming)
10. [Stream](#stream)

---

## ขั้นตอนที่ 21: Classes และ Objects

```dart
// ===== Basic Class =====
class Person {
  // Fields (Properties)
  String name;
  int age;
  String? email;
  
  // Static field (ใช้ร่วมกันทุก instance)
  static int totalPersons = 0;
  
  // Constructor
  Person(this.name, this.age, {this.email}) {
    totalPersons++;
  }
  
  // Getter
  String get greeting => 'สวัสดี ฉันชื่อ $name อายุ $age ปี';
  
  bool get isAdult => age >= 18;
  
  // Setter
  set setAge(int newAge) {
    if (newAge >= 0 && newAge <= 150) {
      age = newAge;
    }
  }
  
  // Methods
  void introduce() {
    print(greeting);
  }
  
  String getInfo() {
    return 'Name: $name, Age: $age${email != null ? ", Email: $email" : ""}';
  }
  
  // Static method
  static void resetCounter() {
    totalPersons = 0;
  }
  
  // Override toString for debugging
  @override
  String toString() => 'Person(name: $name, age: $age)';
  
  // Override == for equality
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is Person &&
          runtimeType == other.runtimeType &&
          name == other.name &&
          age == other.age;
  
  @override
  int get hashCode => Object.hash(name, age);
}

void main() {
  // Create objects
  Person person1 = Person('สมชาย', 25);
  Person person2 = Person('มาลี', 30, email: 'mali@example.com');
  
  // Access properties
  print(person1.name);        // สมชาย
  print(person1.age);         // 25
  print(person1.greeting);    // สวัสดี ฉันชื่อ สมชาย อายุ 25 ปี
  print(person1.isAdult);     // true
  
  // Call methods
  person1.introduce();
  print(person2.getInfo());
  
  // Use setter
  person1.setAge = 26;
  print(person1.age);  // 26
  
  // Static access
  print(Person.totalPersons);  // 2
  
  // Override toString
  print(person1);  // Person(name: สมชาย, age: 26)
  
  // Equality
  Person person3 = Person('สมชาย', 26);
  print(person1 == person3);  // true (same name and age)
}
```

### Immutable Objects ด้วย final fields

```dart
// Immutable class (ค่าเปลี่ยนไม่ได้หลัง create)
class ImmutablePoint {
  final double x;
  final double y;
  
  const ImmutablePoint(this.x, this.y);
  
  // สร้าง instance ใหม่จาก modification
  ImmutablePoint copyWith({double? x, double? y}) {
    return ImmutablePoint(x ?? this.x, y ?? this.y);
  }
  
  double distanceTo(ImmutablePoint other) {
    final dx = x - other.x;
    final dy = y - other.y;
    return (dx * dx + dy * dy);  // squared distance
  }
  
  @override
  String toString() => 'Point($x, $y)';
}

// Value Objects ด้วย const constructor
class Color {
  final int r, g, b;
  
  const Color(this.r, this.g, this.b);
  
  const Color.red() : this(255, 0, 0);
  const Color.green() : this(0, 255, 0);
  const Color.blue() : this(0, 0, 255);
  
  String toHex() => '#${r.toRadixString(16).padLeft(2, '0')}'
      '${g.toRadixString(16).padLeft(2, '0')}'
      '${b.toRadixString(16).padLeft(2, '0')}';
  
  @override
  String toString() => 'Color($r, $g, $b)';
}
```

---

## ขั้นตอนที่ 22: Constructors

```dart
class Rectangle {
  final double width;
  final double height;
  
  // ===== Default Constructor =====
  Rectangle(this.width, this.height);
  
  // ===== Named Constructor =====
  Rectangle.square(double size) : width = size, height = size;
  
  Rectangle.fromArea(double area, double aspectRatio)
      : width = (area * aspectRatio),
        height = area / width;
  
  // ===== Const Constructor =====
  // ทุก field ต้องเป็น final
  const Rectangle.constant(this.width, this.height);
  
  // ===== Factory Constructor =====
  // ควบคุมการสร้าง object (อาจ return existing instance หรือ subclass)
  factory Rectangle.fromMap(Map<String, double> map) {
    return Rectangle(map['width'] ?? 0, map['height'] ?? 0);
  }
  
  // ===== Redirecting Constructor =====
  Rectangle.defaultSize() : this(100, 50);
  
  // ===== Initializer List =====
  Rectangle.validated(double w, double h)
      : assert(w > 0, 'Width must be positive'),
        assert(h > 0, 'Height must be positive'),
        width = w,
        height = h;
  
  // Properties
  double get area => width * height;
  double get perimeter => 2 * (width + height);
  bool get isSquare => width == height;
  
  @override
  String toString() => 'Rectangle(${width}x${height})';
}

// Singleton Pattern ด้วย Factory Constructor
class Database {
  static Database? _instance;
  
  Database._internal();  // Private named constructor
  
  factory Database() {
    _instance ??= Database._internal();
    return _instance!;
  }
  
  void query(String sql) {
    print('Executing: $sql');
  }
}

void main() {
  // Default
  Rectangle r1 = Rectangle(10, 5);
  print(r1.area);        // 50.0
  print(r1.perimeter);   // 30.0
  
  // Named constructors
  Rectangle square = Rectangle.square(8);
  print(square.isSquare);  // true
  
  // Factory
  Rectangle fromMap = Rectangle.fromMap({'width': 15.0, 'height': 7.0});
  print(fromMap);  // Rectangle(15.0x7.0)
  
  // Const
  const r2 = Rectangle.constant(20, 10);
  const r3 = Rectangle.constant(20, 10);
  print(identical(r2, r3));  // true (same object in memory!)
  
  // Singleton
  Database db1 = Database();
  Database db2 = Database();
  print(identical(db1, db2));  // true (same instance)
}
```

---

## ขั้นตอนที่ 23: Inheritance

```dart
// ===== Base Class =====
abstract class Animal {
  String name;
  int age;
  
  Animal(this.name, this.age);
  
  // Abstract method (ต้อง override)
  String makeSound();
  
  // Concrete method (อาจ override)
  void eat(String food) {
    print('$name กำลังกิน $food');
  }
  
  // Template method
  void dailyRoutine() {
    print('$name เริ่มวันใหม่');
    eat(getFavoriteFood());
    makeSound();
    print('$name จบวัน');
  }
  
  String getFavoriteFood() => 'อาหาร';
  
  @override
  String toString() => '$runtimeType($name, age: $age)';
}

// ===== Subclasses =====
class Dog extends Animal {
  String breed;
  
  Dog(String name, int age, this.breed) : super(name, age);
  
  @override
  String makeSound() => '$name: วู้ฟๆ!';
  
  @override
  String getFavoriteFood() => 'กระดูก';
  
  void fetch(String item) {
    print('$name วิ่งไปเอา $item');
  }
  
  @override
  String toString() => 'Dog($name, $breed)';
}

class Cat extends Animal {
  bool isIndoor;
  
  Cat(String name, int age, {this.isIndoor = true}) : super(name, age);
  
  @override
  String makeSound() => '$name: เมี๊ยวๆ!';
  
  @override
  String getFavoriteFood() => 'ปลา';
  
  void purr() {
    print('$name: กรรรรร...');
  }
}

class Parrot extends Animal {
  List<String> vocabulary;
  
  Parrot(String name, int age, this.vocabulary) : super(name, age);
  
  @override
  String makeSound() {
    if (vocabulary.isNotEmpty) {
      return '$name: ${vocabulary[DateTime.now().microsecond % vocabulary.length]}';
    }
    return '$name: กรี๊ก!';
  }
  
  void learnWord(String word) {
    vocabulary.add(word);
    print('$name เรียนรู้คำว่า: $word');
  }
}

void main() {
  // Polymorphism
  List<Animal> animals = [
    Dog('บัดดี้', 3, 'Golden Retriever'),
    Cat('มิ้ว', 2),
    Parrot('พอลลี่', 5, ['Hello!', 'Polly wants a cracker', 'สวัสดี!']),
  ];
  
  for (Animal animal in animals) {
    print(animal.makeSound());  // Calls correct override
  }
  
  // Type checking
  for (Animal animal in animals) {
    if (animal is Dog) {
      animal.fetch('บอล');  // auto-cast to Dog
    } else if (animal is Cat) {
      animal.purr();
    }
  }
  
  // super keyword
  Dog myDog = Dog('แม็กซ์', 4, 'Labrador');
  myDog.dailyRoutine();
  
  // instanceof check
  print(myDog is Animal);  // true
  print(myDog is Dog);     // true
  print(myDog is Cat);     // false
}

// ===== extends vs implements =====

// Abstract class as Interface
abstract class Flyable {
  void fly();
  double get maxAltitude;
}

abstract class Swimmable {
  void swim();
  double get maxDepth;
}

// Multiple "interfaces" ด้วย implements
class Duck extends Animal implements Flyable, Swimmable {
  Duck(String name, int age) : super(name, age);
  
  @override
  String makeSound() => '$name: กๆๆ!';
  
  @override
  void fly() => print('$name กำลังบิน!');
  
  @override
  double get maxAltitude => 100;
  
  @override
  void swim() => print('$name กำลังว่ายน้ำ!');
  
  @override
  double get maxDepth => 2;
}
```

---

## ขั้นตอนที่ 24: Abstract Classes & Interfaces

```dart
// ===== Abstract Class =====
abstract class Shape {
  // Abstract getter (ไม่มี implementation)
  double get area;
  double get perimeter;
  
  // Concrete method
  void describe() {
    print('${runtimeType}: area=${area.toStringAsFixed(2)}, '
        'perimeter=${perimeter.toStringAsFixed(2)}');
  }
  
  // Abstract method
  bool containsPoint(double x, double y);
  
  // Factory constructor ที่ return subclass
  factory Shape.circle(double radius) => Circle(radius);
  factory Shape.rectangle(double w, double h) => Rect(w, h);
}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);
  
  @override
  double get area => 3.14159 * radius * radius;
  
  @override
  double get perimeter => 2 * 3.14159 * radius;
  
  @override
  bool containsPoint(double x, double y) {
    return x * x + y * y <= radius * radius;
  }
}

class Rect extends Shape {
  final double width, height;
  Rect(this.width, this.height);
  
  @override
  double get area => width * height;
  
  @override
  double get perimeter => 2 * (width + height);
  
  @override
  bool containsPoint(double x, double y) {
    return x >= 0 && x <= width && y >= 0 && y <= height;
  }
}

// ===== Interface Pattern =====
// Dart ไม่มี interface keyword แต่ใช้ abstract class + implements

abstract class Repository<T> {
  Future<T?> findById(String id);
  Future<List<T>> findAll();
  Future<void> save(T entity);
  Future<void> delete(String id);
}

class User {
  final String id;
  final String name;
  User(this.id, this.name);
}

// Concrete implementation
class InMemoryUserRepository implements Repository<User> {
  final Map<String, User> _store = {};
  
  @override
  Future<User?> findById(String id) async => _store[id];
  
  @override
  Future<List<User>> findAll() async => _store.values.toList();
  
  @override
  Future<void> save(User entity) async => _store[entity.id] = entity;
  
  @override
  Future<void> delete(String id) async => _store.remove(id);
}

void main() async {
  // Use factory
  Shape c = Shape.circle(5);
  Shape r = Shape.rectangle(4, 6);
  
  c.describe();  // Circle: area=78.54, perimeter=31.42
  r.describe();  // Rect: area=24.00, perimeter=20.00
  
  // Repository pattern
  Repository<User> userRepo = InMemoryUserRepository();
  
  await userRepo.save(User('1', 'Alice'));
  await userRepo.save(User('2', 'Bob'));
  
  User? alice = await userRepo.findById('1');
  print(alice?.name);  // Alice
  
  List<User> all = await userRepo.findAll();
  print('Users: ${all.map((u) => u.name).join(', ')}');
}
```

---

## ขั้นตอนที่ 25: Mixins

```dart
// ===== Mixins =====
// Mixin เป็นวิธี reuse code โดยไม่ต้อง inherit

mixin Loggable {
  void log(String message) {
    print('[${runtimeType}] ${DateTime.now()}: $message');
  }
  
  void logError(String error) {
    print('[ERROR][${runtimeType}] $error');
  }
}

mixin Cacheable {
  final Map<String, dynamic> _cache = {};
  
  void cacheSet(String key, dynamic value) {
    _cache[key] = value;
  }
  
  T? cacheGet<T>(String key) {
    return _cache[key] as T?;
  }
  
  void cacheClear() => _cache.clear();
}

mixin Serializable {
  Map<String, dynamic> toJson();
  
  String serialize() {
    return toJson().entries
        .map((e) => '"${e.key}": "${e.value}"')
        .join(', ');
  }
}

mixin Validatable {
  List<String> validate();
  
  bool get isValid => validate().isEmpty;
  
  void throwIfInvalid() {
    final errors = validate();
    if (errors.isNotEmpty) {
      throw ValidationError(errors);
    }
  }
}

class ValidationError implements Exception {
  final List<String> errors;
  ValidationError(this.errors);
  
  @override
  String toString() => 'ValidationError: ${errors.join(', ')}';
}

// Class ที่ใช้ mixins
class UserService with Loggable, Cacheable {
  Future<User?> getUser(String id) async {
    // Check cache first
    User? cached = cacheGet<User>('user_$id');
    if (cached != null) {
      log('Returning cached user: $id');
      return cached;
    }
    
    log('Fetching user from database: $id');
    // Simulate DB fetch
    User? user = User(id, 'User $id');
    
    if (user != null) {
      cacheSet('user_$id', user);
    }
    
    return user;
  }
}

class ProductModel with Serializable, Validatable {
  String name;
  double price;
  int quantity;
  
  ProductModel({
    required this.name,
    required this.price,
    required this.quantity,
  });
  
  @override
  Map<String, dynamic> toJson() => {
    'name': name,
    'price': price,
    'quantity': quantity,
  };
  
  @override
  List<String> validate() {
    List<String> errors = [];
    if (name.isEmpty) errors.add('Name is required');
    if (price <= 0) errors.add('Price must be positive');
    if (quantity < 0) errors.add('Quantity cannot be negative');
    return errors;
  }
}

// Mixin with constraint (on keyword)
mixin CanBorrow on LibraryItem {
  void borrow(String userId) {
    print('${title} borrowed by $userId');
  }
}

abstract class LibraryItem {
  String title;
  LibraryItem(this.title);
}

class Book extends LibraryItem with CanBorrow {
  String author;
  Book(String title, this.author) : super(title);
}

void main() async {
  // UserService with logging and caching
  UserService service = UserService();
  
  User? user = await service.getUser('123');
  User? userCached = await service.getUser('123');  // From cache
  
  // Product validation
  ProductModel product = ProductModel(
    name: 'Flutter Book',
    price: 29.99,
    quantity: 100,
  );
  
  print(product.isValid);   // true
  print(product.serialize());  // "name": "Flutter Book", ...
  
  ProductModel invalidProduct = ProductModel(
    name: '',
    price: -10,
    quantity: -1,
  );
  
  print(invalidProduct.validate());  // [Name is required, Price must be positive, ...]
  
  // Library
  Book book = Book('Flutter in Action', 'John Author');
  book.borrow('user456');
}
```

---

## ขั้นตอนที่ 26: Enums

```dart
// ===== Basic Enum =====
enum Direction {
  north,
  south,
  east,
  west,
}

// ===== Enhanced Enum (Dart 2.17+) =====
enum Status {
  pending(label: 'รอดำเนินการ', color: 0xFFFF9800),
  active(label: 'ดำเนินการอยู่', color: 0xFF4CAF50),
  completed(label: 'เสร็จสิ้น', color: 0xFF2196F3),
  cancelled(label: 'ยกเลิก', color: 0xFFF44336);
  
  final String label;
  final int color;
  
  const Status({required this.label, required this.color});
  
  // Methods on enum
  bool get isTerminal => this == completed || this == cancelled;
  
  Status? get nextStatus => switch (this) {
    Status.pending => Status.active,
    Status.active => Status.completed,
    _ => null,
  };
  
  static Status fromString(String s) {
    return Status.values.firstWhere(
      (e) => e.name.toLowerCase() == s.toLowerCase(),
      orElse: () => Status.pending,
    );
  }
}

enum OrderType {
  dineIn,
  takeOut,
  delivery;
  
  String get displayName => switch (this) {
    OrderType.dineIn => 'ทานที่ร้าน',
    OrderType.takeOut => 'ซื้อกลับบ้าน',
    OrderType.delivery => 'จัดส่ง',
  };
  
  bool get requiresAddress => this == delivery;
}

void main() {
  // Basic enum
  Direction dir = Direction.north;
  print(dir);         // Direction.north
  print(dir.name);    // north
  print(dir.index);   // 0
  
  // switch on enum
  String description = switch (dir) {
    Direction.north => 'ทิศเหนือ',
    Direction.south => 'ทิศใต้',
    Direction.east => 'ทิศตะวันออก',
    Direction.west => 'ทิศตะวันตก',
  };
  print(description);
  
  // Enhanced enum
  Status currentStatus = Status.pending;
  print(currentStatus.label);   // รอดำเนินการ
  print(currentStatus.isTerminal);  // false
  
  Status? next = currentStatus.nextStatus;
  print(next?.label);  // ดำเนินการอยู่
  
  // Iterate all values
  for (Status s in Status.values) {
    print('${s.name}: ${s.label}');
  }
  
  // From string
  Status status = Status.fromString('active');
  print(status);  // Status.active
  
  // OrderType
  OrderType orderType = OrderType.delivery;
  print(orderType.displayName);          // จัดส่ง
  print(orderType.requiresAddress);      // true
  
  // Enum in conditions
  if (orderType.requiresAddress) {
    print('โปรดระบุที่อยู่จัดส่ง');
  }
}
```

---

## ขั้นตอนที่ 27: Extensions

```dart
// Extensions ช่วยเพิ่ม method ให้กับ class ที่มีอยู่แล้ว

// ===== String Extensions =====
extension StringExtensions on String {
  // Capitalize first letter
  String get capitalize {
    if (isEmpty) return this;
    return '${this[0].toUpperCase()}${substring(1)}';
  }
  
  // Capitalize all words
  String get titleCase {
    return split(' ')
        .map((word) => word.capitalize)
        .join(' ');
  }
  
  // Check if valid email
  bool get isValidEmail {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);
  }
  
  // Check if valid Thai ID
  bool get isValidThaiId {
    if (length != 13) return false;
    return RegExp(r'^\d{13}$').hasMatch(this);
  }
  
  // Convert to int safely
  int? toIntOrNull() => int.tryParse(this);
  
  // Convert to double safely
  double? toDoubleOrNull() => double.tryParse(this);
  
  // Remove whitespace
  String get removeSpaces => replaceAll(' ', '');
  
  // Truncate with ellipsis
  String truncate(int maxLength) {
    if (length <= maxLength) return this;
    return '${substring(0, maxLength)}...';
  }
  
  // Repeat string
  String repeat(int times) => List.filled(times, this).join();
  
  // Wrap in tags (HTML-like)
  String wrap(String tag) => '<$tag>$this</$tag>';
}

// ===== int Extensions =====
extension IntExtensions on int {
  // Duration helpers
  Duration get seconds => Duration(seconds: this);
  Duration get minutes => Duration(minutes: this);
  Duration get hours => Duration(hours: this);
  Duration get days => Duration(days: this);
  
  // Range check
  bool isBetween(int min, int max) => this >= min && this <= max;
  
  // Currency format (Thai Baht)
  String get baht {
    return '฿${toString().padLeft(1)}';
  }
  
  // Ordinal (1st, 2nd, ...)
  String get ordinal {
    if (this % 100 >= 11 && this % 100 <= 13) return '${this}th';
    switch (this % 10) {
      case 1: return '${this}st';
      case 2: return '${this}nd';
      case 3: return '${this}rd';
      default: return '${this}th';
    }
  }
  
  // Generate list
  List<int> get range => List.generate(this, (i) => i);
  
  // Factorial
  int get factorial {
    if (this <= 1) return 1;
    return this * (this - 1).factorial;
  }
}

// ===== List Extensions =====
extension ListExtensions<T> on List<T> {
  // Safe get
  T? safeGet(int index) {
    if (index < 0 || index >= length) return null;
    return this[index];
  }
  
  // Chunk into smaller lists
  List<List<T>> chunk(int size) {
    List<List<T>> result = [];
    for (int i = 0; i < length; i += size) {
      int end = (i + size < length) ? i + size : length;
      result.add(sublist(i, end));
    }
    return result;
  }
  
  // Zip with another list
  List<(T, R)> zip<R>(List<R> other) {
    int len = length < other.length ? length : other.length;
    return List.generate(len, (i) => (this[i], other[i]));
  }
  
  // Flatten one level (if T is List)
  List flatten() {
    return expand((e) => e is Iterable ? e : [e]).toList();
  }
  
  // Random element
  T? random() {
    if (isEmpty) return null;
    return this[DateTime.now().millisecondsSinceEpoch % length];
  }
  
  // Group by key
  Map<K, List<T>> groupBy<K>(K Function(T) keySelector) {
    Map<K, List<T>> result = {};
    for (T item in this) {
      K key = keySelector(item);
      (result[key] ??= []).add(item);
    }
    return result;
  }
}

// ===== DateTime Extensions =====
extension DateTimeExtensions on DateTime {
  String get formatted => '$day/$month/$year';
  
  String get timeFormatted =>
      '${hour.toString().padLeft(2, '0')}:${minute.toString().padLeft(2, '0')}';
  
  bool get isToday {
    DateTime now = DateTime.now();
    return year == now.year && month == now.month && day == now.day;
  }
  
  bool get isPast => isBefore(DateTime.now());
  bool get isFuture => isAfter(DateTime.now());
  
  String get relativeTime {
    Duration diff = DateTime.now().difference(this);
    if (diff.inSeconds < 60) return 'เมื่อสักครู่';
    if (diff.inMinutes < 60) return '${diff.inMinutes} นาทีที่แล้ว';
    if (diff.inHours < 24) return '${diff.inHours} ชั่วโมงที่แล้ว';
    return '${diff.inDays} วันที่แล้ว';
  }
}

void main() {
  // String extensions
  print('hello world'.capitalize);  // Hello world
  print('hello world'.titleCase);   // Hello World
  print('test@email.com'.isValidEmail);  // true
  print('This is a long text for testing'.truncate(20));  // This is a long text...
  print('dart'.repeat(3));  // dartdartdart
  
  // int extensions
  print(5.seconds);           // Duration: 0:00:05
  print(3.range);             // [0, 1, 2]
  print(5.factorial);         // 120
  print(3.ordinal);           // 3rd
  print(42.isBetween(1, 100)); // true
  
  // List extensions
  List<int> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  print(numbers.chunk(3));    // [[1,2,3], [4,5,6], [7,8,9], [10]]
  print(numbers.safeGet(15)); // null (no crash)
  
  List<String> names = ['Alice', 'Bob', 'Charlie'];
  List<(int, String)> zipped = numbers.zip(names);
  print(zipped);  // [(1, Alice), (2, Bob), (3, Charlie)]
  
  // Group by
  List<String> words = ['apple', 'avocado', 'banana', 'blueberry', 'cherry'];
  Map<String, List<String>> grouped = words.groupBy((w) => w[0]);
  print(grouped);  // {a: [apple, avocado], b: [banana, blueberry], c: [cherry]}
  
  // DateTime extensions
  DateTime now = DateTime.now();
  print(now.formatted);     // DD/MM/YYYY
  print(now.timeFormatted); // HH:MM
  print(now.isToday);       // true
  
  DateTime past = DateTime.now().subtract(2.hours);
  print(past.relativeTime); // 2 ชั่วโมงที่แล้ว
}
```

---

## ขั้นตอนที่ 28: Sealed Classes

```dart
// Sealed class ใน Dart 3.0+ ใช้สำหรับ Discriminated Unions
// ทุก subclass ต้องอยู่ใน library เดียวกัน
// เหมาะมากสำหรับ state management และ error handling

sealed class ApiResult<T> {}

class ApiSuccess<T> extends ApiResult<T> {
  final T data;
  final int statusCode;
  
  ApiSuccess(this.data, {this.statusCode = 200});
}

class ApiError<T> extends ApiResult<T> {
  final String message;
  final int statusCode;
  
  ApiError(this.message, {this.statusCode = 500});
}

class ApiLoading<T> extends ApiResult<T> {}

// Pattern matching ด้วย switch
String handleResult<T>(ApiResult<T> result) {
  return switch (result) {
    ApiSuccess(:final data, :final statusCode) =>
        'Success ($statusCode): $data',
    ApiError(:final message, :final statusCode) =>
        'Error ($statusCode): $message',
    ApiLoading() => 'Loading...',
  };
}

// ===== Sealed class สำหรับ State Management =====
sealed class AuthState {}

class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState {
  final String userId;
  final String email;
  AuthAuthenticated(this.userId, this.email);
}
class AuthUnauthenticated extends AuthState {
  final String? errorMessage;
  AuthUnauthenticated([this.errorMessage]);
}

// Widget ที่ใช้ sealed class
String buildAuthUI(AuthState state) {
  return switch (state) {
    AuthInitial() => 'แสดงหน้า Splash Screen',
    AuthLoading() => 'แสดง Loading Spinner',
    AuthAuthenticated(:final email) => 'ยินดีต้อนรับ $email',
    AuthUnauthenticated(errorMessage: null) => 'แสดงหน้า Login',
    AuthUnauthenticated(:final errorMessage?) => 'Error: $errorMessage',
  };
}

// ===== Sealed class สำหรับ Navigation Events =====
sealed class NavigationEvent {}
class NavigateToHome extends NavigationEvent {}
class NavigateToProfile extends NavigationEvent { final String userId; NavigateToProfile(this.userId); }
class NavigateBack extends NavigationEvent {}
class ShowDialog extends NavigationEvent { final String message; ShowDialog(this.message); }

void handleNavigation(NavigationEvent event) {
  switch (event) {
    case NavigateToHome():
      print('Navigating to Home');
    case NavigateToProfile(:final userId):
      print('Navigating to Profile of $userId');
    case NavigateBack():
      print('Going back');
    case ShowDialog(:final message):
      print('Showing dialog: $message');
  }
}

void main() {
  // API Results
  ApiResult<String> success = ApiSuccess('User data here');
  ApiResult<String> error = ApiError('Not Found', statusCode: 404);
  ApiResult<String> loading = ApiLoading();
  
  print(handleResult(success));  // Success (200): User data here
  print(handleResult(error));    // Error (404): Not Found
  print(handleResult(loading));  // Loading...
  
  // Auth states
  List<AuthState> states = [
    AuthInitial(),
    AuthLoading(),
    AuthAuthenticated('user123', 'user@email.com'),
    AuthUnauthenticated('Invalid credentials'),
    AuthUnauthenticated(),
  ];
  
  for (AuthState state in states) {
    print(buildAuthUI(state));
  }
  
  // Navigation events
  handleNavigation(NavigateToHome());
  handleNavigation(NavigateToProfile('user456'));
  handleNavigation(ShowDialog('Are you sure?'));
}
```

---

## ขั้นตอนที่ 29: Async Programming

```dart
import 'dart:async';

// ===== Future =====
// Future แทนค่าที่จะได้รับในอนาคต

Future<String> fetchUserName(String id) async {
  // Simulate API call
  await Future.delayed(Duration(seconds: 1));
  
  if (id == 'invalid') {
    throw Exception('User not found');
  }
  
  return 'User $id';
}

Future<int> calculateComplexValue() async {
  await Future.delayed(Duration(milliseconds: 500));
  return 42;
}

// ===== async/await =====
Future<void> main() async {
  print('Start');
  
  // await รอ Future ให้เสร็จ
  String userName = await fetchUserName('123');
  print('User: $userName');
  
  // Error handling
  try {
    String invalid = await fetchUserName('invalid');
  } catch (e) {
    print('Error: $e');
  }
  
  // ===== Future.wait - รอหลาย Future พร้อมกัน =====
  print('Fetching multiple users...');
  
  List<Future<String>> futures = [
    fetchUserName('1'),
    fetchUserName('2'),
    fetchUserName('3'),
  ];
  
  List<String> users = await Future.wait(futures);
  print('Users: $users');
  
  // ===== Future.any - รอ Future แรกที่เสร็จ =====
  String fastest = await Future.any([
    Future.delayed(Duration(seconds: 3), () => 'slow'),
    Future.delayed(Duration(seconds: 1), () => 'fast'),
  ]);
  print('Fastest: $fastest');  // fast
  
  // ===== Chaining Futures =====
  String result = await Future.value('hello')
      .then((s) => s.toUpperCase())
      .then((s) => '$s World');
  print(result);  // HELLO World
  
  // ===== Future.delayed =====
  await Future.delayed(Duration(seconds: 1), () {
    print('Delayed execution');
  });
  
  // ===== Compute timeout =====
  try {
    String data = await fetchUserName('slow').timeout(
      Duration(milliseconds: 500),
      onTimeout: () => 'Timeout!',
    );
    print('Data: $data');
  } catch (e) {
    print('Timed out!');
  }
  
  print('End');
}

// ===== Completer =====
Future<String> processWithCompleter() {
  Completer<String> completer = Completer();
  
  Timer(Duration(seconds: 1), () {
    if (DateTime.now().second % 2 == 0) {
      completer.complete('Success!');
    } else {
      completer.completeError(Exception('Failed!'));
    }
  });
  
  return completer.future;
}

// ===== Async Generators (async*) =====
Stream<int> countAsync(int from, int to) async* {
  for (int i = from; i <= to; i++) {
    await Future.delayed(Duration(milliseconds: 100));
    yield i;
  }
}

Future<void> exampleAsync() async {
  await for (int i in countAsync(1, 5)) {
    print('Count: $i');
  }
}
```

---

## ขั้นตอนที่ 30: Stream

```dart
import 'dart:async';

// ===== Stream Types =====
void main() async {
  // ===== Single-subscription Stream =====
  Stream<int> numberStream = Stream.fromIterable([1, 2, 3, 4, 5]);
  
  await for (int n in numberStream) {
    print('Number: $n');
  }
  
  // ===== Broadcast Stream =====
  // Multiple listeners can subscribe
  StreamController<String> controller = StreamController<String>.broadcast();
  
  // Subscriber 1
  controller.stream.listen(
    (data) => print('Listener 1: $data'),
    onError: (e) => print('Listener 1 Error: $e'),
    onDone: () => print('Listener 1: Stream closed'),
  );
  
  // Subscriber 2
  controller.stream.listen(
    (data) => print('Listener 2: $data'),
  );
  
  // Add data to stream
  controller.add('Hello');
  controller.add('World');
  controller.addError(Exception('Something went wrong'));
  
  await Future.delayed(Duration(milliseconds: 100));
  await controller.close();
  
  // ===== Stream Transformations =====
  Stream<int> nums = Stream.periodic(
    Duration(milliseconds: 200),
    (count) => count,
  ).take(10);
  
  // map
  Stream<String> strings = nums.map((n) => 'Item $n');
  
  // where (filter)
  Stream<int> evenNums = nums.where((n) => n % 2 == 0);
  
  // expand (flatMap)
  Stream<int> expanded = nums.expand((n) => [n, n * 10]);
  
  // take / skip
  Stream<int> first5 = nums.take(5);
  Stream<int> skip3 = nums.skip(3);
  
  // distinct
  Stream<int> repeating = Stream.fromIterable([1, 1, 2, 2, 3, 3]);
  Stream<int> distinct = repeating.distinct();
  
  // reduce / fold
  int sum = await nums.reduce((a, b) => a + b);
  int result = await nums.fold(0, (acc, n) => acc + n);
  
  // toList / toSet
  List<int> list = await Stream.fromIterable([1, 2, 3]).toList();
  
  // ===== StreamTransformer =====
  StreamTransformer<int, String> transformer = StreamTransformer.fromHandlers(
    handleData: (data, sink) {
      sink.add('Transformed: $data');
    },
    handleError: (error, stackTrace, sink) {
      sink.addError('Caught error: $error');
    },
  );
  
  await Stream.fromIterable([1, 2, 3])
      .transform(transformer)
      .forEach(print);
  
  // ===== StreamBuilder ใน Flutter (preview) =====
  // ใน Widget จะใช้แบบนี้:
  // StreamBuilder<int>(
  //   stream: myStream,
  //   builder: (context, snapshot) {
  //     if (snapshot.hasError) return Text('Error: ${snapshot.error}');
  //     if (!snapshot.hasData) return CircularProgressIndicator();
  //     return Text('Value: ${snapshot.data}');
  //   },
  // )
  
  // ===== Practical: Stock Price Stream =====
  Stream<double> stockPriceStream = _createStockStream('AAPL');
  
  stockPriceStream
      .where((price) => price > 150)  // Only prices above $150
      .take(5)                         // Take first 5
      .listen(
        (price) => print('Stock price: \$${price.toStringAsFixed(2)}'),
        onDone: () => print('Stream complete'),
      );
  
  await Future.delayed(Duration(seconds: 3));
}

Stream<double> _createStockStream(String symbol) async* {
  double price = 148.50;
  
  while (true) {
    await Future.delayed(Duration(milliseconds: 200));
    price += (DateTime.now().millisecond % 10 - 5) * 0.1;
    yield price;
  }
}
```

---

## Workshop: สร้าง Todo App ด้วย OOP

```dart
// ===== Model =====
enum Priority { low, medium, high }

class Todo {
  final String id;
  String title;
  String? description;
  bool isCompleted;
  Priority priority;
  DateTime createdAt;
  DateTime? completedAt;
  List<String> tags;
  
  Todo({
    String? id,
    required this.title,
    this.description,
    this.isCompleted = false,
    this.priority = Priority.medium,
    List<String>? tags,
  })  : id = id ?? DateTime.now().millisecondsSinceEpoch.toString(),
        createdAt = DateTime.now(),
        tags = tags ?? [];
  
  void complete() {
    isCompleted = true;
    completedAt = DateTime.now();
  }
  
  void uncomplete() {
    isCompleted = false;
    completedAt = null;
  }
  
  Todo copyWith({
    String? title,
    String? description,
    bool? isCompleted,
    Priority? priority,
    List<String>? tags,
  }) {
    return Todo(
      id: id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      priority: priority ?? this.priority,
      tags: tags ?? this.tags,
    );
  }
  
  Map<String, dynamic> toJson() => {
    'id': id,
    'title': title,
    'description': description,
    'isCompleted': isCompleted,
    'priority': priority.name,
    'createdAt': createdAt.toIso8601String(),
    'completedAt': completedAt?.toIso8601String(),
    'tags': tags,
  };
  
  factory Todo.fromJson(Map<String, dynamic> json) {
    return Todo(
      id: json['id'],
      title: json['title'],
      description: json['description'],
      isCompleted: json['isCompleted'] ?? false,
      priority: Priority.values.firstWhere(
        (p) => p.name == json['priority'],
        orElse: () => Priority.medium,
      ),
      tags: List<String>.from(json['tags'] ?? []),
    );
  }
  
  @override
  String toString() => 'Todo($title, $priority, completed: $isCompleted)';
}

// ===== Repository =====
abstract class TodoRepository {
  Future<List<Todo>> getAll();
  Future<Todo?> getById(String id);
  Future<void> add(Todo todo);
  Future<void> update(Todo todo);
  Future<void> delete(String id);
  Future<void> toggleComplete(String id);
}

class InMemoryTodoRepository implements TodoRepository {
  final Map<String, Todo> _todos = {};
  
  @override
  Future<List<Todo>> getAll() async => _todos.values.toList();
  
  @override
  Future<Todo?> getById(String id) async => _todos[id];
  
  @override
  Future<void> add(Todo todo) async => _todos[todo.id] = todo;
  
  @override
  Future<void> update(Todo todo) async {
    if (!_todos.containsKey(todo.id)) {
      throw Exception('Todo not found: ${todo.id}');
    }
    _todos[todo.id] = todo;
  }
  
  @override
  Future<void> delete(String id) async => _todos.remove(id);
  
  @override
  Future<void> toggleComplete(String id) async {
    final todo = _todos[id];
    if (todo == null) throw Exception('Todo not found: $id');
    
    if (todo.isCompleted) {
      todo.uncomplete();
    } else {
      todo.complete();
    }
  }
}

// ===== Service =====
class TodoService {
  final TodoRepository _repository;
  
  TodoService(this._repository);
  
  Future<List<Todo>> getAllTodos() => _repository.getAll();
  
  Future<List<Todo>> getByPriority(Priority priority) async {
    final todos = await _repository.getAll();
    return todos.where((t) => t.priority == priority).toList();
  }
  
  Future<List<Todo>> getPending() async {
    final todos = await _repository.getAll();
    return todos.where((t) => !t.isCompleted).toList();
  }
  
  Future<List<Todo>> getCompleted() async {
    final todos = await _repository.getAll();
    return todos.where((t) => t.isCompleted).toList();
  }
  
  Future<List<Todo>> searchByTag(String tag) async {
    final todos = await _repository.getAll();
    return todos.where((t) => t.tags.contains(tag)).toList();
  }
  
  Future<Todo> addTodo(String title, {
    String? description,
    Priority priority = Priority.medium,
    List<String>? tags,
  }) async {
    final todo = Todo(
      title: title,
      description: description,
      priority: priority,
      tags: tags,
    );
    await _repository.add(todo);
    return todo;
  }
  
  Future<void> completeTodo(String id) => _repository.toggleComplete(id);
  
  Future<void> deleteTodo(String id) => _repository.delete(id);
  
  Future<Map<String, int>> getStats() async {
    final todos = await _repository.getAll();
    return {
      'total': todos.length,
      'completed': todos.where((t) => t.isCompleted).length,
      'pending': todos.where((t) => !t.isCompleted).length,
      'high': todos.where((t) => t.priority == Priority.high).length,
    };
  }
}

// ===== Main =====
Future<void> testTodoApp() async {
  TodoService service = TodoService(InMemoryTodoRepository());
  
  // Add todos
  Todo todo1 = await service.addTodo(
    'เรียน Flutter',
    priority: Priority.high,
    tags: ['education', 'coding'],
  );
  
  Todo todo2 = await service.addTodo(
    'ออกกำลังกาย',
    description: 'วิ่ง 5km',
    priority: Priority.medium,
    tags: ['health'],
  );
  
  Todo todo3 = await service.addTodo(
    'ซื้อของ',
    priority: Priority.low,
    tags: ['shopping'],
  );
  
  print('=== All Todos ===');
  List<Todo> all = await service.getAllTodos();
  all.forEach(print);
  
  print('\n=== Complete a Todo ===');
  await service.completeTodo(todo2.id);
  
  print('\n=== Stats ===');
  Map<String, int> stats = await service.getStats();
  stats.forEach((key, value) => print('$key: $value'));
  
  print('\n=== High Priority ===');
  List<Todo> highPriority = await service.getByPriority(Priority.high);
  highPriority.forEach(print);
  
  print('\n=== Search by Tag "health" ===');
  List<Todo> healthTodos = await service.searchByTag('health');
  healthTodos.forEach(print);
}
```

---

## สรุป Part 03

ในส่วนนี้คุณได้เรียนรู้:

✅ Classes, Objects, Properties, Methods
✅ Constructors ทุกประเภท (Default, Named, Factory, Const, Redirecting)
✅ Inheritance, Polymorphism, Method Overriding
✅ Abstract Classes และ Interfaces
✅ Mixins สำหรับ code reuse
✅ Enhanced Enums
✅ Extensions เพิ่ม functionality ให้ existing classes
✅ Sealed Classes สำหรับ discriminated unions
✅ Async/Await และ Future
✅ Stream programming
✅ สร้าง Todo App ด้วย OOP patterns

---

**ก่อนหน้า:** [Part 02 - Dart Language Fundamentals](part_02.md)  
**ต่อไป:** [Part 04 - Flutter Widget พื้นฐาน →](part_04.md)

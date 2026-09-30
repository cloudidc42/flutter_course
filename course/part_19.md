# Part 19: SQLite & Local Database
## ขั้นตอนที่ 181-190

---

## สารบัญ
1. [sqflite Package Setup](#ขั้นตอนที่-181-sqflite-package-setup)
2. [สร้างและเปิด Database](#ขั้นตอนที่-182-สร้างและเปิด-database)
3. [สร้าง Tables](#ขั้นตอนที่-183-สร้าง-tables)
4. [CREATE (Insert)](#ขั้นตอนที่-184-create-insert)
5. [READ (Query)](#ขั้นตอนที่-185-read-query)
6. [UPDATE](#ขั้นตอนที่-186-update)
7. [DELETE](#ขั้นตอนที่-187-delete)
8. [Complex Queries](#ขั้นตอนที่-188-complex-queries)
9. [Transactions](#ขั้นตอนที่-189-transactions)
10. [Database Helper Pattern](#ขั้นตอนที่-190-database-helper-pattern)
11. [Workshop: Note-Taking App](#workshop-note-taking-app)

---

## ขั้นตอนที่ 181: sqflite Package Setup

```yaml
# pubspec.yaml
dependencies:
  sqflite: ^2.3.3+1
  path: ^1.9.0
  path_provider: ^2.1.3
```

```dart
// ตรวจสอบว่า sqflite ทำงานได้
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as path;
import 'package:path_provider/path_provider.dart';

Future<String> getDatabasePath(String dbName) async {
  // วิธีที่ 1: ใช้ getDatabasesPath() (แนะนำ)
  final databasesPath = await getDatabasesPath();
  return path.join(databasesPath, dbName);

  // วิธีที่ 2: ใช้ path_provider
  // final dir = await getApplicationDocumentsDirectory();
  // return path.join(dir.path, dbName);
}

// ตรวจสอบ database version
Future<void> checkDatabase() async {
  final dbPath = await getDatabasePath('myapp.db');
  final db = await openDatabase(dbPath);

  // ดู version
  final version = await db.getVersion();
  print('Database version: $version');

  // ดูว่า table มีอะไรบ้าง
  final tables = await db.rawQuery(
    "SELECT name FROM sqlite_master WHERE type='table'",
  );
  print('Tables: ${tables.map((t) => t['name']).toList()}');

  await db.close();
}
```

---

## ขั้นตอนที่ 182: สร้างและเปิด Database

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as path;

class DatabaseManager {
  static Database? _database;
  static const int _version = 1;
  static const String _dbName = 'myapp.db';

  // Singleton pattern
  static Future<Database> get database async {
    _database ??= await _initDatabase();
    return _database!;
  }

  static Future<Database> _initDatabase() async {
    final dbPath = await getDatabasesPath();
    final fullPath = path.join(dbPath, _dbName);

    return openDatabase(
      fullPath,
      version: _version,
      onCreate: _onCreate,
      onUpgrade: _onUpgrade,
      onDowngrade: onDatabaseDowngradeDelete, // หรือ handle manually
      onOpen: (db) async {
        // เปิด foreign keys support
        await db.execute('PRAGMA foreign_keys = ON');
      },
    );
  }

  static Future<void> _onCreate(Database db, int version) async {
    // สร้าง tables ทั้งหมดใน version 1
    await db.execute('''
      CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TEXT NOT NULL,
        is_active INTEGER NOT NULL DEFAULT 1
      )
    ''');

    await db.execute('''
      CREATE TABLE notes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        user_id INTEGER NOT NULL,
        created_at TEXT NOT NULL,
        updated_at TEXT NOT NULL,
        FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
      )
    ''');

    // Index สำหรับเพิ่มความเร็ว query
    await db.execute(
      'CREATE INDEX idx_notes_user_id ON notes(user_id)',
    );
  }

  static Future<void> _onUpgrade(Database db, int oldVersion, int newVersion) async {
    if (oldVersion < 2) {
      // Migration จาก v1 ไป v2
      await db.execute('ALTER TABLE notes ADD COLUMN is_pinned INTEGER DEFAULT 0');
      await db.execute('ALTER TABLE notes ADD COLUMN tags TEXT');
    }
    if (oldVersion < 3) {
      // Migration จาก v2 ไป v3
      await db.execute('''
        CREATE TABLE categories (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT NOT NULL,
          color TEXT NOT NULL
        )
      ''');
      await db.execute('ALTER TABLE notes ADD COLUMN category_id INTEGER');
    }
  }

  // ลบ database ทั้งหมด (เช่น ตอน logout)
  static Future<void> deleteDatabase() async {
    final dbPath = await getDatabasesPath();
    final fullPath = path.join(dbPath, _dbName);
    await databaseFactory.deleteDatabase(fullPath);
    _database = null;
  }

  // Close database
  static Future<void> close() async {
    if (_database != null) {
      await _database!.close();
      _database = null;
    }
  }
}
```

---

## ขั้นตอนที่ 183: สร้าง Tables

```dart
// Table schemas และ model classes

// Users Table
class UserTable {
  static const String tableName = 'users';
  static const String colId = 'id';
  static const String colName = 'name';
  static const String colEmail = 'email';
  static const String colCreatedAt = 'created_at';
  static const String colIsActive = 'is_active';

  static String get createTableSQL => '''
    CREATE TABLE IF NOT EXISTS $tableName (
      $colId INTEGER PRIMARY KEY AUTOINCREMENT,
      $colName TEXT NOT NULL,
      $colEmail TEXT UNIQUE NOT NULL,
      $colCreatedAt TEXT NOT NULL DEFAULT (datetime('now')),
      $colIsActive INTEGER NOT NULL DEFAULT 1
    )
  ''';
}

// Notes Table
class NoteTable {
  static const String tableName = 'notes';

  static String get createTableSQL => '''
    CREATE TABLE IF NOT EXISTS $tableName (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      content TEXT NOT NULL,
      user_id INTEGER NOT NULL,
      is_pinned INTEGER NOT NULL DEFAULT 0,
      color TEXT,
      tags TEXT,
      created_at TEXT NOT NULL DEFAULT (datetime('now')),
      updated_at TEXT NOT NULL DEFAULT (datetime('now')),
      FOREIGN KEY (user_id) REFERENCES ${UserTable.tableName}(id) ON DELETE CASCADE
    )
  ''';

  static String get createIndexSQL =>
      'CREATE INDEX IF NOT EXISTS idx_notes_user_id ON $tableName(user_id)';
}

// Model
class NoteModel {
  final int? id;
  final String title;
  final String content;
  final int userId;
  final bool isPinned;
  final String? color;
  final List<String> tags;
  final DateTime createdAt;
  final DateTime updatedAt;

  const NoteModel({
    this.id,
    required this.title,
    required this.content,
    required this.userId,
    this.isPinned = false,
    this.color,
    this.tags = const [],
    required this.createdAt,
    required this.updatedAt,
  });

  Map<String, dynamic> toMap() => {
    if (id != null) 'id': id,
    'title': title,
    'content': content,
    'user_id': userId,
    'is_pinned': isPinned ? 1 : 0,
    if (color != null) 'color': color,
    'tags': tags.join(','),
    'created_at': createdAt.toIso8601String(),
    'updated_at': updatedAt.toIso8601String(),
  };

  factory NoteModel.fromMap(Map<String, dynamic> map) => NoteModel(
    id: map['id'] as int?,
    title: map['title'] as String,
    content: map['content'] as String,
    userId: map['user_id'] as int,
    isPinned: (map['is_pinned'] as int?) == 1,
    color: map['color'] as String?,
    tags: (map['tags'] as String?)?.split(',').where((t) => t.isNotEmpty).toList() ?? [],
    createdAt: DateTime.parse(map['created_at'] as String),
    updatedAt: DateTime.parse(map['updated_at'] as String),
  );

  NoteModel copyWith({
    int? id,
    String? title,
    String? content,
    int? userId,
    bool? isPinned,
    String? color,
    List<String>? tags,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) => NoteModel(
    id: id ?? this.id,
    title: title ?? this.title,
    content: content ?? this.content,
    userId: userId ?? this.userId,
    isPinned: isPinned ?? this.isPinned,
    color: color ?? this.color,
    tags: tags ?? this.tags,
    createdAt: createdAt ?? this.createdAt,
    updatedAt: updatedAt ?? this.updatedAt,
  );
}
```

---

## ขั้นตอนที่ 184: CREATE (Insert)

```dart
import 'package:sqflite/sqflite.dart';

class NoteDao {
  final Database db;
  NoteDao(this.db);

  // Insert หนึ่ง record
  Future<int> insertNote(NoteModel note) async {
    return db.insert(
      NoteTable.tableName,
      note.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace, // หรือ abort, ignore, fail, rollback
    );
  }

  // Insert ด้วย rawInsert
  Future<int> insertNoteRaw({
    required String title,
    required String content,
    required int userId,
  }) async {
    return db.rawInsert(
      '''
      INSERT INTO ${NoteTable.tableName} (title, content, user_id, created_at, updated_at)
      VALUES (?, ?, ?, ?, ?)
      ''',
      [
        title,
        content,
        userId,
        DateTime.now().toIso8601String(),
        DateTime.now().toIso8601String(),
      ],
    );
  }

  // Bulk insert
  Future<void> insertMultipleNotes(List<NoteModel> notes) async {
    final batch = db.batch();

    for (final note in notes) {
      batch.insert(
        NoteTable.tableName,
        note.toMap(),
        conflictAlgorithm: ConflictAlgorithm.ignore,
      );
    }

    await batch.commit(noResult: true);
  }
}
```

---

## ขั้นตอนที่ 185: READ (Query)

```dart
class NoteDao {
  final Database db;
  NoteDao(this.db);

  // Query ทั้งหมด
  Future<List<NoteModel>> getAllNotes() async {
    final maps = await db.query(NoteTable.tableName);
    return maps.map(NoteModel.fromMap).toList();
  }

  // Query ด้วย conditions
  Future<List<NoteModel>> getNotesByUser(int userId) async {
    final maps = await db.query(
      NoteTable.tableName,
      where: 'user_id = ?',
      whereArgs: [userId],
      orderBy: 'is_pinned DESC, updated_at DESC',
    );
    return maps.map(NoteModel.fromMap).toList();
  }

  // Query single record
  Future<NoteModel?> getNoteById(int id) async {
    final maps = await db.query(
      NoteTable.tableName,
      where: 'id = ?',
      whereArgs: [id],
      limit: 1,
    );
    return maps.isEmpty ? null : NoteModel.fromMap(maps.first);
  }

  // Query กับ multiple conditions
  Future<List<NoteModel>> searchNotes({
    required int userId,
    String? query,
    bool? isPinned,
    String? color,
  }) async {
    final conditions = <String>['user_id = ?'];
    final args = <dynamic>[userId];

    if (query != null && query.isNotEmpty) {
      conditions.add('(title LIKE ? OR content LIKE ?)');
      args.addAll(['%$query%', '%$query%']);
    }

    if (isPinned != null) {
      conditions.add('is_pinned = ?');
      args.add(isPinned ? 1 : 0);
    }

    if (color != null) {
      conditions.add('color = ?');
      args.add(color);
    }

    final maps = await db.query(
      NoteTable.tableName,
      where: conditions.join(' AND '),
      whereArgs: args,
      orderBy: 'updated_at DESC',
    );

    return maps.map(NoteModel.fromMap).toList();
  }

  // Raw query
  Future<List<NoteModel>> getRecentNotes(int userId, {int limit = 10}) async {
    final maps = await db.rawQuery(
      '''
      SELECT * FROM ${NoteTable.tableName}
      WHERE user_id = ?
      ORDER BY updated_at DESC
      LIMIT ?
      ''',
      [userId, limit],
    );
    return maps.map(NoteModel.fromMap).toList();
  }

  // COUNT
  Future<int> getNoteCount(int userId) async {
    final result = await db.rawQuery(
      'SELECT COUNT(*) as count FROM ${NoteTable.tableName} WHERE user_id = ?',
      [userId],
    );
    return Sqflite.firstIntValue(result) ?? 0;
  }

  // Pagination
  Future<List<NoteModel>> getNotesPage({
    required int userId,
    required int page,
    int pageSize = 20,
  }) async {
    final maps = await db.query(
      NoteTable.tableName,
      where: 'user_id = ?',
      whereArgs: [userId],
      orderBy: 'updated_at DESC',
      limit: pageSize,
      offset: page * pageSize,
    );
    return maps.map(NoteModel.fromMap).toList();
  }
}
```

---

## ขั้นตอนที่ 186: UPDATE

```dart
class NoteDao {
  final Database db;
  NoteDao(this.db);

  // Update ทั้ง record
  Future<int> updateNote(NoteModel note) async {
    return db.update(
      NoteTable.tableName,
      note.copyWith(updatedAt: DateTime.now()).toMap(),
      where: 'id = ?',
      whereArgs: [note.id],
    );
  }

  // Update เฉพาะบาง fields
  Future<int> updateNoteTitle(int id, String title) async {
    return db.update(
      NoteTable.tableName,
      {
        'title': title,
        'updated_at': DateTime.now().toIso8601String(),
      },
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  // Toggle pin
  Future<int> togglePin(int id) async {
    return db.rawUpdate(
      '''
      UPDATE ${NoteTable.tableName}
      SET is_pinned = CASE WHEN is_pinned = 1 THEN 0 ELSE 1 END,
          updated_at = ?
      WHERE id = ?
      ''',
      [DateTime.now().toIso8601String(), id],
    );
  }

  // Bulk update
  Future<void> updateColor(List<int> ids, String color) async {
    final batch = db.batch();
    for (final id in ids) {
      batch.update(
        NoteTable.tableName,
        {'color': color, 'updated_at': DateTime.now().toIso8601String()},
        where: 'id = ?',
        whereArgs: [id],
      );
    }
    await batch.commit(noResult: true);
  }
}
```

---

## ขั้นตอนที่ 187: DELETE

```dart
class NoteDao {
  final Database db;
  NoteDao(this.db);

  // Delete หนึ่ง record
  Future<int> deleteNote(int id) async {
    return db.delete(
      NoteTable.tableName,
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  // Delete หลาย records
  Future<int> deleteNotes(List<int> ids) async {
    final placeholders = ids.map((_) => '?').join(', ');
    return db.rawDelete(
      'DELETE FROM ${NoteTable.tableName} WHERE id IN ($placeholders)',
      ids,
    );
  }

  // Delete ทั้งหมดของ user
  Future<int> deleteAllUserNotes(int userId) async {
    return db.delete(
      NoteTable.tableName,
      where: 'user_id = ?',
      whereArgs: [userId],
    );
  }

  // Soft delete (mark as deleted แทนที่จะลบจริง)
  Future<int> softDeleteNote(int id) async {
    return db.update(
      NoteTable.tableName,
      {
        'is_deleted': 1,
        'deleted_at': DateTime.now().toIso8601String(),
      },
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}
```

---

## ขั้นตอนที่ 188: Complex Queries

```dart
// JOIN queries
class NoteWithUser {
  final NoteModel note;
  final String userName;
  final String userEmail;

  const NoteWithUser({
    required this.note,
    required this.userName,
    required this.userEmail,
  });

  factory NoteWithUser.fromMap(Map<String, dynamic> map) => NoteWithUser(
    note: NoteModel.fromMap(map),
    userName: map['user_name'] as String,
    userEmail: map['user_email'] as String,
  );
}

class AdvancedNoteDao {
  final Database db;
  AdvancedNoteDao(this.db);

  // JOIN
  Future<List<NoteWithUser>> getNotesWithUsers() async {
    final maps = await db.rawQuery('''
      SELECT n.*, u.name as user_name, u.email as user_email
      FROM notes n
      INNER JOIN users u ON n.user_id = u.id
      ORDER BY n.updated_at DESC
    ''');
    return maps.map(NoteWithUser.fromMap).toList();
  }

  // Aggregate queries
  Future<Map<String, dynamic>> getNoteStats(int userId) async {
    final result = await db.rawQuery('''
      SELECT
        COUNT(*) as total_notes,
        SUM(CASE WHEN is_pinned = 1 THEN 1 ELSE 0 END) as pinned_count,
        MIN(created_at) as first_note_date,
        MAX(updated_at) as last_updated,
        COUNT(DISTINCT color) as color_count
      FROM notes
      WHERE user_id = ?
    ''', [userId]);

    return result.first;
  }

  // GROUP BY
  Future<List<Map<String, dynamic>>> getNotesByColor(int userId) async {
    return db.rawQuery('''
      SELECT color, COUNT(*) as note_count
      FROM notes
      WHERE user_id = ?
      GROUP BY color
      ORDER BY note_count DESC
    ''', [userId]);
  }

  // HAVING
  Future<List<Map<String, dynamic>>> getActiveUsers() async {
    return db.rawQuery('''
      SELECT u.id, u.name, COUNT(n.id) as note_count
      FROM users u
      LEFT JOIN notes n ON u.id = n.user_id
      GROUP BY u.id
      HAVING note_count > 5
      ORDER BY note_count DESC
    ''');
  }

  // Subquery
  Future<List<NoteModel>> getNotesFromActiveUsers() async {
    final maps = await db.rawQuery('''
      SELECT n.*
      FROM notes n
      WHERE n.user_id IN (
        SELECT id FROM users WHERE is_active = 1
      )
      ORDER BY n.updated_at DESC
    ''');
    return maps.map(NoteModel.fromMap).toList();
  }

  // Full text search (LIKE)
  Future<List<NoteModel>> fullTextSearch(String query) async {
    final maps = await db.rawQuery('''
      SELECT * FROM notes
      WHERE title LIKE ? OR content LIKE ?
      ORDER BY
        CASE WHEN title LIKE ? THEN 0 ELSE 1 END,
        updated_at DESC
    ''', ['%$query%', '%$query%', '%$query%']);

    return maps.map(NoteModel.fromMap).toList();
  }
}
```

---

## ขั้นตอนที่ 189: Transactions

```dart
class TransactionExample {
  final Database db;
  TransactionExample(this.db);

  // Transaction พื้นฐาน
  Future<void> transferData(int fromUserId, int toUserId) async {
    await db.transaction((txn) async {
      // ดึงข้อมูล notes ของ user เดิม
      final notes = await txn.query(
        'notes',
        where: 'user_id = ?',
        whereArgs: [fromUserId],
      );

      // อัปเดต user_id ทุก note
      for (final note in notes) {
        await txn.update(
          'notes',
          {'user_id': toUserId},
          where: 'id = ?',
          whereArgs: [note['id']],
        );
      }

      // ลบ user เดิม
      await txn.delete(
        'users',
        where: 'id = ?',
        whereArgs: [fromUserId],
      );
    });
  }

  // Transaction กับ error handling
  Future<bool> createUserWithNotes({
    required String name,
    required String email,
    required List<String> noteTitles,
  }) async {
    try {
      await db.transaction((txn) async {
        // สร้าง user
        final userId = await txn.insert('users', {
          'name': name,
          'email': email,
          'created_at': DateTime.now().toIso8601String(),
        });

        // สร้าง notes ทั้งหมด
        for (final title in noteTitles) {
          await txn.insert('notes', {
            'title': title,
            'content': '',
            'user_id': userId,
            'created_at': DateTime.now().toIso8601String(),
            'updated_at': DateTime.now().toIso8601String(),
          });
        }
      });
      return true;
    } catch (e) {
      print('Transaction failed: $e');
      return false;
    }
  }

  // Batch operations (เร็วกว่า transaction สำหรับ simple operations)
  Future<void> batchInsertNotes(List<NoteModel> notes) async {
    final batch = db.batch();
    for (final note in notes) {
      batch.insert('notes', note.toMap());
    }
    final results = await batch.commit();
    print('Inserted ${results.length} notes');
  }

  // Exclusive transaction สำหรับ read-write ที่ต้องการ consistency
  Future<NoteModel?> getAndUpdateNote(int noteId) async {
    return db.transaction((txn) async {
      // Lock row สำหรับ update
      final maps = await txn.query(
        'notes',
        where: 'id = ?',
        whereArgs: [noteId],
        limit: 1,
      );

      if (maps.isEmpty) return null;

      final note = NoteModel.fromMap(maps.first);
      final updated = note.copyWith(updatedAt: DateTime.now());

      await txn.update(
        'notes',
        updated.toMap(),
        where: 'id = ?',
        whereArgs: [noteId],
      );

      return updated;
    });
  }
}
```

---

## ขั้นตอนที่ 190: Database Helper Pattern

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as path;

// Repository interface
abstract class NoteRepository {
  Future<NoteModel> create(NoteModel note);
  Future<NoteModel?> findById(int id);
  Future<List<NoteModel>> findAll({int? userId});
  Future<NoteModel> update(NoteModel note);
  Future<bool> delete(int id);
  Future<List<NoteModel>> search(String query, {int? userId});
}

// SQLite implementation
class SqliteNoteRepository implements NoteRepository {
  final Database _db;

  SqliteNoteRepository(this._db);

  @override
  Future<NoteModel> create(NoteModel note) async {
    final now = DateTime.now();
    final noteWithDates = note.copyWith(createdAt: now, updatedAt: now);
    final id = await _db.insert(
      NoteTable.tableName,
      noteWithDates.toMap(),
    );
    return noteWithDates.copyWith(id: id);
  }

  @override
  Future<NoteModel?> findById(int id) async {
    final maps = await _db.query(
      NoteTable.tableName,
      where: 'id = ?',
      whereArgs: [id],
      limit: 1,
    );
    return maps.isEmpty ? null : NoteModel.fromMap(maps.first);
  }

  @override
  Future<List<NoteModel>> findAll({int? userId}) async {
    final maps = await _db.query(
      NoteTable.tableName,
      where: userId != null ? 'user_id = ?' : null,
      whereArgs: userId != null ? [userId] : null,
      orderBy: 'is_pinned DESC, updated_at DESC',
    );
    return maps.map(NoteModel.fromMap).toList();
  }

  @override
  Future<NoteModel> update(NoteModel note) async {
    final updated = note.copyWith(updatedAt: DateTime.now());
    await _db.update(
      NoteTable.tableName,
      updated.toMap(),
      where: 'id = ?',
      whereArgs: [note.id],
    );
    return updated;
  }

  @override
  Future<bool> delete(int id) async {
    final count = await _db.delete(
      NoteTable.tableName,
      where: 'id = ?',
      whereArgs: [id],
    );
    return count > 0;
  }

  @override
  Future<List<NoteModel>> search(String query, {int? userId}) async {
    final conditions = ['(title LIKE ? OR content LIKE ?)'];
    final args = <dynamic>['%$query%', '%$query%'];

    if (userId != null) {
      conditions.add('user_id = ?');
      args.add(userId);
    }

    final maps = await _db.query(
      NoteTable.tableName,
      where: conditions.join(' AND '),
      whereArgs: args,
      orderBy: 'updated_at DESC',
    );
    return maps.map(NoteModel.fromMap).toList();
  }
}

// Database factory
class AppDatabase {
  static AppDatabase? _instance;
  late final Database _db;

  AppDatabase._();

  static Future<AppDatabase> getInstance() async {
    if (_instance == null) {
      _instance = AppDatabase._();
      await _instance!._init();
    }
    return _instance!;
  }

  Future<void> _init() async {
    final dbPath = await getDatabasesPath();
    _db = await openDatabase(
      path.join(dbPath, 'notes_app.db'),
      version: 1,
      onCreate: (db, version) async {
        await db.execute(NoteTable.createTableSQL);
        await db.execute(UserTable.createTableSQL);
      },
    );
  }

  NoteRepository get noteRepository => SqliteNoteRepository(_db);

  Future<void> close() async => _db.close();
}
```

---

## Workshop: Note-Taking App

```dart
import 'package:flutter/material.dart';
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as path;

// ========================
// Database Setup
// ========================
class NotesDatabase {
  static NotesDatabase? _instance;
  Database? _db;

  NotesDatabase._();

  static Future<NotesDatabase> get instance async {
    _instance ??= NotesDatabase._();
    await _instance!._ensureInitialized();
    return _instance!;
  }

  Future<void> _ensureInitialized() async {
    if (_db != null) return;
    final dbPath = await getDatabasesPath();
    _db = await openDatabase(
      path.join(dbPath, 'notes.db'),
      version: 1,
      onCreate: (db, _) async {
        await db.execute('''
          CREATE TABLE notes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            content TEXT NOT NULL,
            color TEXT NOT NULL DEFAULT '#FFFFFF',
            is_pinned INTEGER NOT NULL DEFAULT 0,
            created_at TEXT NOT NULL,
            updated_at TEXT NOT NULL
          )
        ''');
      },
    );
  }

  Database get db {
    if (_db == null) throw StateError('Database not initialized');
    return _db!;
  }
}

// ========================
// Note Model
// ========================
class Note {
  final int? id;
  final String title;
  final String content;
  final String color;
  final bool isPinned;
  final DateTime createdAt;
  final DateTime updatedAt;

  const Note({
    this.id,
    required this.title,
    required this.content,
    required this.color,
    required this.isPinned,
    required this.createdAt,
    required this.updatedAt,
  });

  factory Note.create({required String title, required String content, String color = '#FFFDE7'}) {
    final now = DateTime.now();
    return Note(title: title, content: content, color: color, isPinned: false, createdAt: now, updatedAt: now);
  }

  Map<String, dynamic> toMap() => {
    if (id != null) 'id': id,
    'title': title,
    'content': content,
    'color': color,
    'is_pinned': isPinned ? 1 : 0,
    'created_at': createdAt.toIso8601String(),
    'updated_at': updatedAt.toIso8601String(),
  };

  factory Note.fromMap(Map<String, dynamic> map) => Note(
    id: map['id'] as int?,
    title: map['title'] as String,
    content: map['content'] as String,
    color: map['color'] as String,
    isPinned: (map['is_pinned'] as int) == 1,
    createdAt: DateTime.parse(map['created_at'] as String),
    updatedAt: DateTime.parse(map['updated_at'] as String),
  );

  Note copyWith({String? title, String? content, String? color, bool? isPinned}) => Note(
    id: id,
    title: title ?? this.title,
    content: content ?? this.content,
    color: color ?? this.color,
    isPinned: isPinned ?? this.isPinned,
    createdAt: createdAt,
    updatedAt: DateTime.now(),
  );

  Color get colorValue {
    final hex = color.replaceAll('#', '');
    return Color(int.parse('FF$hex', radix: 16));
  }
}

// ========================
// DAO
// ========================
class NoteDao {
  final Database db;
  NoteDao(this.db);

  Future<Note> insert(Note note) async {
    final id = await db.insert('notes', note.toMap());
    return Note(id: id, title: note.title, content: note.content, color: note.color, isPinned: note.isPinned, createdAt: note.createdAt, updatedAt: note.updatedAt);
  }

  Future<List<Note>> getAll({String? search}) async {
    List<Map<String, dynamic>> maps;
    if (search != null && search.isNotEmpty) {
      maps = await db.query('notes', where: 'title LIKE ? OR content LIKE ?', whereArgs: ['%$search%', '%$search%'], orderBy: 'is_pinned DESC, updated_at DESC');
    } else {
      maps = await db.query('notes', orderBy: 'is_pinned DESC, updated_at DESC');
    }
    return maps.map(Note.fromMap).toList();
  }

  Future<int> update(Note note) async {
    return db.update('notes', note.toMap(), where: 'id = ?', whereArgs: [note.id]);
  }

  Future<int> delete(int id) async {
    return db.delete('notes', where: 'id = ?', whereArgs: [id]);
  }

  Future<int> togglePin(int id, bool isPinned) async {
    return db.update('notes', {'is_pinned': isPinned ? 1 : 0, 'updated_at': DateTime.now().toIso8601String()}, where: 'id = ?', whereArgs: [id]);
  }
}

// ========================
// App Screen
// ========================
class NotesApp extends StatelessWidget {
  const NotesApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Notes',
      theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.amber), useMaterial3: true),
      home: const NotesScreen(),
    );
  }
}

class NotesScreen extends StatefulWidget {
  const NotesScreen({super.key});
  @override
  State<NotesScreen> createState() => _NotesScreenState();
}

class _NotesScreenState extends State<NotesScreen> {
  NoteDao? _dao;
  List<Note> _notes = [];
  bool _isLoading = true;
  String _searchQuery = '';

  @override
  void initState() {
    super.initState();
    _initDatabase();
  }

  Future<void> _initDatabase() async {
    final notesDb = await NotesDatabase.instance;
    _dao = NoteDao(notesDb.db);
    await _loadNotes();
  }

  Future<void> _loadNotes() async {
    if (_dao == null) return;
    setState(() => _isLoading = true);
    final notes = await _dao!.getAll(search: _searchQuery.isEmpty ? null : _searchQuery);
    if (mounted) setState(() { _notes = notes; _isLoading = false; });
  }

  Future<void> _deleteNote(Note note) async {
    await _dao!.delete(note.id!);
    await _loadNotes();
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: const Text('ลบโน้ตแล้ว'), action: SnackBarAction(label: 'เลิกทำ', onPressed: () async {
          await _dao!.insert(note);
          await _loadNotes();
        })),
      );
    }
  }

  Future<void> _togglePin(Note note) async {
    await _dao!.togglePin(note.id!, !note.isPinned);
    await _loadNotes();
  }

  void _openNoteEditor({Note? note}) async {
    final result = await Navigator.push<bool>(
      context,
      MaterialPageRoute(builder: (context) => NoteEditorScreen(dao: _dao!, note: note)),
    );
    if (result == true) await _loadNotes();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
        title: const Text('My Notes'),
        backgroundColor: Colors.amber,
        bottom: PreferredSize(
          preferredSize: const Size.fromHeight(56),
          child: Padding(
            padding: const EdgeInsets.fromLTRB(16, 0, 16, 8),
            child: TextField(
              decoration: InputDecoration(
                hintText: 'ค้นหาโน้ต...',
                prefixIcon: const Icon(Icons.search),
                filled: true,
                fillColor: Colors.white,
                border: OutlineInputBorder(borderRadius: BorderRadius.circular(24), borderSide: BorderSide.none),
                contentPadding: const EdgeInsets.symmetric(horizontal: 16),
              ),
              onChanged: (value) { _searchQuery = value; _loadNotes(); },
            ),
          ),
        ),
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : _notes.isEmpty
              ? Center(child: Column(mainAxisAlignment: MainAxisAlignment.center, children: [
                  const Icon(Icons.note_add, size: 64, color: Colors.grey),
                  const SizedBox(height: 8),
                  Text(_searchQuery.isEmpty ? 'ยังไม่มีโน้ต กด + เพื่อสร้าง' : 'ไม่พบโน้ตที่ค้นหา'),
                ]))
              : RefreshIndicator(
                  onRefresh: _loadNotes,
                  child: GridView.builder(
                    padding: const EdgeInsets.all(8),
                    gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: 2,
                      childAspectRatio: 0.9,
                      crossAxisSpacing: 8,
                      mainAxisSpacing: 8,
                    ),
                    itemCount: _notes.length,
                    itemBuilder: (context, index) => _buildNoteCard(_notes[index]),
                  ),
                ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _openNoteEditor(),
        backgroundColor: Colors.amber,
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildNoteCard(Note note) {
    return GestureDetector(
      onTap: () => _openNoteEditor(note: note),
      onLongPress: () => _showNoteOptions(note),
      child: Card(
        color: note.colorValue,
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Row(children: [
                Expanded(child: Text(note.title, style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 14), maxLines: 1, overflow: TextOverflow.ellipsis)),
                if (note.isPinned) const Icon(Icons.push_pin, size: 14, color: Colors.orange),
              ]),
              const SizedBox(height: 4),
              Expanded(child: Text(note.content, style: const TextStyle(fontSize: 12, color: Colors.black87), overflow: TextOverflow.fade)),
              const SizedBox(height: 4),
              Text(
                _formatDate(note.updatedAt),
                style: const TextStyle(fontSize: 10, color: Colors.black54),
              ),
            ],
          ),
        ),
      ),
    );
  }

  void _showNoteOptions(Note note) {
    showModalBottomSheet(
      context: context,
      builder: (context) => Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          ListTile(
            leading: Icon(note.isPinned ? Icons.push_pin_outlined : Icons.push_pin),
            title: Text(note.isPinned ? 'เลิกปักหมุด' : 'ปักหมุด'),
            onTap: () { Navigator.pop(context); _togglePin(note); },
          ),
          ListTile(
            leading: const Icon(Icons.delete, color: Colors.red),
            title: const Text('ลบ', style: TextStyle(color: Colors.red)),
            onTap: () { Navigator.pop(context); _deleteNote(note); },
          ),
        ],
      ),
    );
  }

  String _formatDate(DateTime date) {
    final diff = DateTime.now().difference(date);
    if (diff.inMinutes < 1) return 'เมื่อกี้';
    if (diff.inHours < 1) return '${diff.inMinutes} นาทีที่แล้ว';
    if (diff.inDays < 1) return '${diff.inHours} ชั่วโมงที่แล้ว';
    return '${date.day}/${date.month}/${date.year}';
  }
}

// Note Editor Screen
class NoteEditorScreen extends StatefulWidget {
  final NoteDao dao;
  final Note? note;

  const NoteEditorScreen({super.key, required this.dao, this.note});

  @override
  State<NoteEditorScreen> createState() => _NoteEditorScreenState();
}

class _NoteEditorScreenState extends State<NoteEditorScreen> {
  late TextEditingController _titleController;
  late TextEditingController _contentController;
  String _selectedColor = '#FFFDE7';
  bool _isSaving = false;

  final List<String> _colors = [
    '#FFFDE7', '#F3E5F5', '#E3F2FD', '#E8F5E9',
    '#FFF3E0', '#FCE4EC', '#E0F2F1', '#FFFFFF',
  ];

  @override
  void initState() {
    super.initState();
    _titleController = TextEditingController(text: widget.note?.title ?? '');
    _contentController = TextEditingController(text: widget.note?.content ?? '');
    _selectedColor = widget.note?.color ?? '#FFFDE7';
  }

  @override
  void dispose() {
    _titleController.dispose();
    _contentController.dispose();
    super.dispose();
  }

  Future<void> _save() async {
    if (_titleController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('กรุณาใส่หัวข้อโน้ต')),
      );
      return;
    }

    setState(() => _isSaving = true);

    try {
      if (widget.note == null) {
        final newNote = Note.create(
          title: _titleController.text.trim(),
          content: _contentController.text.trim(),
          color: _selectedColor,
        );
        await widget.dao.insert(newNote);
      } else {
        final updated = widget.note!.copyWith(
          title: _titleController.text.trim(),
          content: _contentController.text.trim(),
          color: _selectedColor,
        );
        await widget.dao.update(updated);
      }

      if (mounted) Navigator.pop(context, true);
    } catch (e) {
      if (mounted) {
        setState(() => _isSaving = false);
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('เกิดข้อผิดพลาด: $e')),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    final colorHex = _selectedColor.replaceAll('#', '');
    final bgColor = Color(int.parse('FF$colorHex', radix: 16));

    return Scaffold(
      backgroundColor: bgColor,
      appBar: AppBar(
        backgroundColor: bgColor,
        elevation: 0,
        title: Text(widget.note == null ? 'โน้ตใหม่' : 'แก้ไขโน้ต'),
        actions: [
          if (_isSaving)
            const Padding(
              padding: EdgeInsets.all(16),
              child: SizedBox(width: 20, height: 20, child: CircularProgressIndicator(strokeWidth: 2)),
            )
          else
            TextButton(onPressed: _save, child: const Text('บันทึก', style: TextStyle(fontWeight: FontWeight.bold))),
        ],
      ),
      body: Column(
        children: [
          // Color picker
          SizedBox(
            height: 48,
            child: ListView.builder(
              scrollDirection: Axis.horizontal,
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
              itemCount: _colors.length,
              itemBuilder: (context, index) {
                final color = _colors[index];
                final hex = color.replaceAll('#', '');
                final c = Color(int.parse('FF$hex', radix: 16));
                return GestureDetector(
                  onTap: () => setState(() => _selectedColor = color),
                  child: Container(
                    width: 32,
                    height: 32,
                    margin: const EdgeInsets.only(right: 8),
                    decoration: BoxDecoration(
                      color: c,
                      shape: BoxShape.circle,
                      border: Border.all(
                        color: _selectedColor == color ? Colors.grey[800]! : Colors.grey[300]!,
                        width: _selectedColor == color ? 2 : 1,
                      ),
                    ),
                  ),
                );
              },
            ),
          ),

          // Title
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16),
            child: TextField(
              controller: _titleController,
              style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              decoration: const InputDecoration(
                hintText: 'หัวข้อ',
                border: InputBorder.none,
              ),
              maxLines: 1,
            ),
          ),

          // Content
          Expanded(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: TextField(
                controller: _contentController,
                style: const TextStyle(fontSize: 15),
                decoration: const InputDecoration(
                  hintText: 'เขียนโน้ตของคุณที่นี่...',
                  border: InputBorder.none,
                ),
                maxLines: null,
                expands: true,
                textAlignVertical: TextAlignVertical.top,
              ),
            ),
          ),
        ],
      ),
    );
  }
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const NotesApp());
}
```

---

## สรุป (Summary)

| หัวข้อ | สิ่งที่เรียนรู้ |
|--------|----------------|
| sqflite setup | การติดตั้งและ setup database |
| สร้าง Database | openDatabase, onCreate, onUpgrade |
| สร้าง Tables | CREATE TABLE, indexes |
| CRUD | insert, query, update, delete |
| Complex Queries | JOIN, GROUP BY, subqueries |
| Transactions | atomicity, batch operations |
| Repository Pattern | abstract interface, testability |

---

## แบบฝึกหัด (Exercises)

1. **ง่าย**: สร้าง app จดรายการชำระเงิน (Todo List) ด้วย sqflite
2. **ปานกลาง**: สร้าง app บันทึกรายรับ-รายจ่าย พร้อม category และ report
3. **ยาก**: สร้าง app diary ที่มี photos (เก็บ path), search, export เป็น PDF

---

[← Part 18: JSON Serialization](part_18.md) | [Part 20: SharedPreferences & Secure Storage →](part_20.md)

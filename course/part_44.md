# Part 44: Offline-First Architecture
## ขั้นตอนที่ 431-440

---

## สารบัญ
1. [Offline-First Design Principles](#ขั้นตอนที่-431-offline-first-principles)
2. [Sync Queue](#ขั้นตอนที่-432-sync-queue)
3. [Conflict Resolution Strategies](#ขั้นตอนที่-433-conflict-resolution)
4. [Optimistic UI Updates](#ขั้นตอนที่-434-optimistic-ui)
5. [Background Sync](#ขั้นตอนที่-435-background-sync)
6. [Drift สำหรับ Complex Local DB](#ขั้นตอนที่-436-drift-database)
7. [Hive สำหรับ NoSQL Local Storage](#ขั้นตอนที่-437-hive-storage)
8. [Network-first vs Cache-first](#ขั้นตอนที่-438-fetch-strategies)
9. [Data Synchronization Patterns](#ขั้นตอนที่-439-sync-patterns)
10. [Workshop: Offline Todo App](#ขั้นตอนที่-440-workshop)

---

## ขั้นตอนที่ 431: Offline-First Design Principles

### Offline-First คืออะไร?

Offline-First เป็นแนวทางการออกแบบที่ถือว่า "network ไม่ available คือ default state" แทนที่จะเป็น edge case

```
Traditional (Online-First):
Online ──── Try network ──── Success ──── Show data
                    └──── Fail ──── Show error message

Offline-First:
Always ──── Read local data ──── Show data immediately
           └──── Sync with server in background ──── Update if changed
```

### หลักการสำคัญ

1. **Local First**: เขียน/อ่านจาก local database เป็นหลัก
2. **Background Sync**: sync กับ server ใน background
3. **Conflict Resolution**: จัดการ conflicts เมื่อ sync
4. **Optimistic Updates**: update UI ก่อนที่ server จะยืนยัน
5. **Graceful Degradation**: app ยังทำงานได้แม้ไม่มี internet

### Dependencies

```yaml
# pubspec.yaml
dependencies:
  # Drift - SQL local database
  drift: ^2.14.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.8.0
  
  # Hive - NoSQL local storage
  hive_flutter: ^1.1.0
  
  # Network checking
  connectivity_plus: ^5.0.0
  
  # Background tasks
  workmanager: ^0.5.2

dev_dependencies:
  drift_dev: ^2.14.0
  build_runner: ^2.4.0
  hive_generator: ^2.0.0
```

### Network Status Service

```dart
// lib/services/network_service.dart
import 'package:connectivity_plus/connectivity_plus.dart';

class NetworkService {
  final Connectivity _connectivity = Connectivity();
  
  Stream<bool> get isOnlineStream {
    return _connectivity.onConnectivityChanged.map((results) {
      return results.any((r) => r != ConnectivityResult.none);
    });
  }
  
  Future<bool> get isOnline async {
    final results = await _connectivity.checkConnectivity();
    return results.any((r) => r != ConnectivityResult.none);
  }
  
  Future<bool> hasInternetAccess() async {
    if (!await isOnline) return false;
    
    try {
      final result = await InternetAddress.lookup('google.com');
      return result.isNotEmpty && result.first.rawAddress.isNotEmpty;
    } on SocketException {
      return false;
    }
  }
}
```

---

## ขั้นตอนที่ 432: Sync Queue

### Sync Queue Implementation

```dart
// lib/offline/sync_queue.dart
import 'package:drift/drift.dart';

enum SyncOperationType {
  create,
  update,
  delete,
}

enum SyncStatus {
  pending,
  syncing,
  completed,
  failed,
}

@DataClassName('SyncOperation')
class SyncOperations extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get entityType => text()(); // 'todo', 'note', etc.
  TextColumn get entityId => text()();
  TextColumn get operationType => textEnum<SyncOperationType>()();
  TextColumn get payload => text()(); // JSON encoded data
  TextColumn get status => textEnum<SyncStatus>()();
  IntColumn get retryCount => integer().withDefault(const Constant(0))();
  DateTimeColumn get createdAt => dateTime()();
  DateTimeColumn get updatedAt => dateTime()();
  TextColumn get errorMessage => text().nullable()();
}

class SyncQueue {
  final AppDatabase _db;
  final NetworkService _networkService;
  final ApiService _apiService;
  
  bool _isSyncing = false;
  
  SyncQueue(this._db, this._networkService, this._apiService) {
    // Watch network status and auto-sync when online
    _networkService.isOnlineStream.listen((isOnline) {
      if (isOnline && !_isSyncing) {
        processQueue();
      }
    });
  }
  
  Future<void> enqueue({
    required String entityType,
    required String entityId,
    required SyncOperationType operationType,
    required Map<String, dynamic> payload,
  }) async {
    await _db.into(_db.syncOperations).insert(
      SyncOperationsCompanion.insert(
        entityType: entityType,
        entityId: entityId,
        operationType: operationType,
        payload: jsonEncode(payload),
        status: SyncStatus.pending,
        createdAt: DateTime.now(),
        updatedAt: DateTime.now(),
      ),
    );
    
    // Try to sync immediately if online
    final isOnline = await _networkService.isOnline;
    if (isOnline) {
      processQueue();
    }
  }
  
  Future<void> processQueue() async {
    if (_isSyncing) return;
    _isSyncing = true;
    
    try {
      final pendingOps = await _db.getPendingSyncOperations();
      
      for (final op in pendingOps) {
        await _processSingleOperation(op);
      }
    } finally {
      _isSyncing = false;
    }
  }
  
  Future<void> _processSingleOperation(SyncOperation op) async {
    // Mark as syncing
    await _db.updateSyncOperationStatus(op.id, SyncStatus.syncing);
    
    try {
      final payload = jsonDecode(op.payload) as Map<String, dynamic>;
      
      switch (op.operationType) {
        case SyncOperationType.create:
          await _apiService.create(op.entityType, payload);
        case SyncOperationType.update:
          await _apiService.update(op.entityType, op.entityId, payload);
        case SyncOperationType.delete:
          await _apiService.delete(op.entityType, op.entityId);
      }
      
      // Mark as completed
      await _db.updateSyncOperationStatus(op.id, SyncStatus.completed);
    } catch (e) {
      final retryCount = op.retryCount + 1;
      
      if (retryCount >= 3) {
        // Mark as failed after 3 retries
        await _db.updateSyncOperationStatusWithError(
          op.id,
          SyncStatus.failed,
          e.toString(),
        );
      } else {
        // Back to pending for retry
        await _db.updateSyncOperationRetry(op.id, retryCount);
      }
    }
  }
  
  Future<int> get pendingCount async {
    return _db.countPendingSyncOperations();
  }
  
  Stream<int> get pendingCountStream {
    return _db.watchPendingSyncOperationsCount();
  }
}
```

---

## ขั้นตอนที่ 433: Conflict Resolution Strategies

### Conflict Resolution Strategies

```dart
// lib/offline/conflict_resolver.dart

// Strategy 1: Last Write Wins (LWW)
class LastWriteWinsResolver<T extends VersionedEntity> {
  T resolve(T local, T server) {
    return local.updatedAt.isAfter(server.updatedAt) ? local : server;
  }
}

// Strategy 2: Server Wins
class ServerWinsResolver<T> {
  T resolve(T local, T server) => server;
}

// Strategy 3: Client Wins  
class ClientWinsResolver<T> {
  T resolve(T local, T server) => local;
}

// Strategy 4: Merge (สำหรับ complex objects)
abstract class MergeResolver<T> {
  T resolve(T local, T server);
}

// ตัวอย่าง: Merge User Profile
class UserProfileMergeResolver implements MergeResolver<UserProfile> {
  @override
  UserProfile resolve(UserProfile local, UserProfile server) {
    // เอา field ที่ใหม่กว่า
    return UserProfile(
      id: server.id,
      name: local.nameUpdatedAt.isAfter(server.nameUpdatedAt)
          ? local.name
          : server.name,
      email: server.email, // Server wins for email (security)
      bio: local.bioUpdatedAt.isAfter(server.bioUpdatedAt)
          ? local.bio
          : server.bio,
      avatarUrl: server.avatarUrl, // Server wins for avatar
      updatedAt: DateTime.now(),
    );
  }
}

// Strategy 5: Three-way merge
class ThreeWayMergeResolver<T extends Diffable<T>> {
  final T base; // Common ancestor
  
  ThreeWayMergeResolver(this.base);
  
  MergeResult<T> resolve(T local, T server) {
    final localDiff = base.diff(local);
    final serverDiff = base.diff(server);
    
    // ตรวจสอบ conflicts
    final conflicts = localDiff.conflictsWith(serverDiff);
    
    if (conflicts.isEmpty) {
      // No conflicts - apply both changes
      return MergeResult.success(base.applyChanges([...localDiff, ...serverDiff]));
    } else {
      // Has conflicts - need manual resolution
      return MergeResult.conflict(conflicts);
    }
  }
}

// Conflict handling ใน Repository
class ConflictAwareTodoRepository {
  final LocalTodoRepository _localRepo;
  final RemoteTodoRepository _remoteRepo;
  final ConflictResolver<Todo> _conflictResolver;
  
  ConflictAwareTodoRepository(
    this._localRepo,
    this._remoteRepo,
    this._conflictResolver,
  );
  
  Future<SyncResult> syncTodo(String todoId) async {
    final local = await _localRepo.getById(todoId);
    final server = await _remoteRepo.getById(todoId);
    
    if (local == null && server == null) {
      return SyncResult.notFound;
    }
    
    if (local == null) {
      // Deleted locally, keep server version
      await _localRepo.save(server!);
      return SyncResult.serverWins;
    }
    
    if (server == null) {
      // New local item
      final synced = await _remoteRepo.create(local);
      await _localRepo.update(synced);
      return SyncResult.created;
    }
    
    if (local.version == server.version) {
      // No conflict
      return SyncResult.noChange;
    }
    
    // Conflict! Resolve
    final resolved = _conflictResolver.resolve(local, server);
    await _localRepo.update(resolved);
    await _remoteRepo.update(resolved);
    
    return SyncResult.merged;
  }
}

enum SyncResult {
  notFound,
  serverWins,
  clientWins,
  created,
  noChange,
  merged,
  failed,
}
```

---

## ขั้นตอนที่ 434: Optimistic UI Updates

### Optimistic Update Pattern

```dart
// lib/features/todo/notifiers/todo_notifier.dart
@riverpod
class TodoNotifier extends _$TodoNotifier {
  @override
  Future<List<Todo>> build() async {
    return ref.read(localTodoRepositoryProvider).getAll();
  }
  
  Future<void> addTodo(String title, String description) async {
    // สร้าง temporary ID สำหรับ optimistic update
    final tempId = 'temp_${DateTime.now().millisecondsSinceEpoch}';
    
    final optimisticTodo = Todo(
      id: tempId,
      title: title,
      description: description,
      createdAt: DateTime.now(),
      isCompleted: false,
      syncStatus: SyncStatus.pending,
    );
    
    // Update UI ทันที (optimistic)
    state = AsyncData([
      ...state.valueOrNull ?? [],
      optimisticTodo,
    ]);
    
    try {
      // Save to local DB
      await ref.read(localTodoRepositoryProvider).save(optimisticTodo);
      
      // Enqueue for sync
      await ref.read(syncQueueProvider).enqueue(
        entityType: 'todo',
        entityId: tempId,
        operationType: SyncOperationType.create,
        payload: optimisticTodo.toJson(),
      );
      
      // If online, sync immediately and update with real ID
      if (await ref.read(networkServiceProvider).isOnline) {
        final realTodo = await ref.read(remoteTodoRepositoryProvider)
            .create(optimisticTodo);
        
        // Replace temp ID with real ID
        await ref.read(localTodoRepositoryProvider).replaceTempId(
          tempId,
          realTodo,
        );
        
        // Update state with real todo
        final todos = state.valueOrNull ?? [];
        state = AsyncData(
          todos.map((t) => t.id == tempId ? realTodo : t).toList(),
        );
      }
    } catch (e) {
      // Revert on error
      final todos = state.valueOrNull ?? [];
      state = AsyncData(todos.where((t) => t.id != tempId).toList());
      
      // Mark local todo as failed
      await ref.read(localTodoRepositoryProvider).updateSyncStatus(
        tempId,
        SyncStatus.failed,
      );
      
      rethrow;
    }
  }
  
  Future<void> toggleComplete(String todoId) async {
    final todos = state.valueOrNull ?? [];
    final todo = todos.firstWhere((t) => t.id == todoId);
    final updated = todo.copyWith(isCompleted: !todo.isCompleted);
    
    // Optimistic update
    state = AsyncData(
      todos.map((t) => t.id == todoId ? updated : t).toList(),
    );
    
    try {
      await ref.read(localTodoRepositoryProvider).update(updated);
      await ref.read(syncQueueProvider).enqueue(
        entityType: 'todo',
        entityId: todoId,
        operationType: SyncOperationType.update,
        payload: updated.toJson(),
      );
    } catch (e) {
      // Revert on error
      state = AsyncData(
        todos.map((t) => t.id == todoId ? todo : t).toList(),
      );
      rethrow;
    }
  }
  
  Future<void> deleteTodo(String todoId) async {
    final todos = state.valueOrNull ?? [];
    final todo = todos.firstWhere((t) => t.id == todoId);
    
    // Optimistic remove
    state = AsyncData(todos.where((t) => t.id != todoId).toList());
    
    try {
      await ref.read(localTodoRepositoryProvider).delete(todoId);
      await ref.read(syncQueueProvider).enqueue(
        entityType: 'todo',
        entityId: todoId,
        operationType: SyncOperationType.delete,
        payload: {},
      );
    } catch (e) {
      // Revert on error - add back
      state = AsyncData([...todos]);
      rethrow;
    }
  }
}
```

---

## ขั้นตอนที่ 435: Background Sync

### WorkManager สำหรับ Background Sync

```dart
// lib/background/background_sync.dart
import 'package:workmanager/workmanager.dart';

const String _syncTaskName = 'background_sync';
const String _periodicSyncTaskName = 'periodic_sync';

@pragma('vm:entry-point')
void callbackDispatcher() {
  Workmanager().executeTask((task, inputData) async {
    try {
      switch (task) {
        case _syncTaskName:
          await _performSync();
        case _periodicSyncTaskName:
          await _performPeriodicSync();
      }
      return Future.value(true);
    } catch (e) {
      print('Background sync error: $e');
      return Future.value(false);
    }
  });
}

Future<void> _performSync() async {
  final syncQueue = await SyncQueue.getInstance();
  await syncQueue.processQueue();
}

Future<void> _performPeriodicSync() async {
  final syncQueue = await SyncQueue.getInstance();
  
  // Pull latest data from server
  await _pullLatestData();
  
  // Push pending changes
  await syncQueue.processQueue();
}

Future<void> _pullLatestData() async {
  // Download latest data and merge with local
}

class BackgroundSyncService {
  static Future<void> initialize() async {
    await Workmanager().initialize(
      callbackDispatcher,
      isInDebugMode: kDebugMode,
    );
  }
  
  static Future<void> schedulePeriodicSync() async {
    await Workmanager().registerPeriodicTask(
      _periodicSyncTaskName,
      _periodicSyncTaskName,
      frequency: const Duration(minutes: 15),
      constraints: Constraints(
        networkType: NetworkType.connected,
        requiresBatteryNotLow: true,
      ),
      existingWorkPolicy: ExistingWorkPolicy.keep,
    );
  }
  
  static Future<void> scheduleImmediateSync() async {
    await Workmanager().registerOneOffTask(
      _syncTaskName,
      _syncTaskName,
      constraints: Constraints(
        networkType: NetworkType.connected,
      ),
    );
  }
  
  static Future<void> cancelAll() async {
    await Workmanager().cancelAll();
  }
}
```

---

## ขั้นตอนที่ 436: Drift สำหรับ Complex Local DB

### Drift Database Setup

```dart
// lib/database/app_database.dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

part 'app_database.g.dart';

// Table Definitions
class Todos extends Table {
  TextColumn get id => text()();
  TextColumn get title => text()();
  TextColumn get description => text().withDefault(const Constant(''))();
  BoolColumn get isCompleted => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime()();
  DateTimeColumn get updatedAt => dateTime()();
  DateTimeColumn get dueDate => dateTime().nullable()();
  TextColumn get categoryId => text().nullable().references(Categories, #id)();
  IntColumn get priority => integer().withDefault(const Constant(0))();
  TextColumn get syncStatus => text().withDefault(const Constant('synced'))();
  
  @override
  Set<Column> get primaryKey => {id};
}

class Categories extends Table {
  TextColumn get id => text()();
  TextColumn get name => text()();
  IntColumn get color => integer()();
  TextColumn get iconName => text()();
  
  @override
  Set<Column> get primaryKey => {id};
}

class SyncOperations extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get entityType => text()();
  TextColumn get entityId => text()();
  TextColumn get operationType => text()();
  TextColumn get payload => text()();
  TextColumn get status => text().withDefault(const Constant('pending'))();
  IntColumn get retryCount => integer().withDefault(const Constant(0))();
  DateTimeColumn get createdAt => dateTime()();
  DateTimeColumn get updatedAt => dateTime()();
  TextColumn get errorMessage => text().nullable()();
}

@DriftDatabase(tables: [Todos, Categories, SyncOperations])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  
  @override
  int get schemaVersion => 1;
  
  @override
  MigrationStrategy get migration {
    return MigrationStrategy(
      onCreate: (m) async {
        await m.createAll();
        
        // Seed default categories
        await into(categories).insertAll([
          CategoriesCompanion.insert(
            id: 'work',
            name: 'งาน',
            color: const Value(0xFF2196F3),
            iconName: 'work',
          ),
          CategoriesCompanion.insert(
            id: 'personal',
            name: 'ส่วนตัว',
            color: const Value(0xFF4CAF50),
            iconName: 'person',
          ),
        ]);
      },
      onUpgrade: (m, from, to) async {
        // Handle migrations
      },
    );
  }
  
  // Todos queries
  Future<List<Todo>> getAllTodos() => select(todos).get();
  
  Stream<List<Todo>> watchAllTodos() => select(todos).watch();
  
  Stream<List<Todo>> watchTodosByCategory(String categoryId) {
    return (select(todos)
      ..where((t) => t.categoryId.equals(categoryId)))
        .watch();
  }
  
  Stream<List<Todo>> watchIncompleteTodos() {
    return (select(todos)
      ..where((t) => t.isCompleted.equals(false))
      ..orderBy([
        (t) => OrderingTerm(expression: t.priority, mode: OrderingMode.desc),
        (t) => OrderingTerm(expression: t.createdAt),
      ]))
        .watch();
  }
  
  Future<Todo?> getTodoById(String id) {
    return (select(todos)..where((t) => t.id.equals(id))).getSingleOrNull();
  }
  
  Future<void> saveTodo(TodosCompanion todo) async {
    await into(todos).insertOnConflictUpdate(todo);
  }
  
  Future<void> updateTodo(TodosCompanion todo) async {
    await (update(todos)..where((t) => t.id.equals(todo.id.value)))
        .write(todo);
  }
  
  Future<int> deleteTodo(String id) {
    return (delete(todos)..where((t) => t.id.equals(id))).go();
  }
  
  // Complex queries with joins
  Stream<List<TodoWithCategory>> watchTodosWithCategories() {
    final query = select(todos).join([
      leftOuterJoin(categories, categories.id.equalsExp(todos.categoryId)),
    ]);
    
    return query.watch().map((rows) {
      return rows.map((row) {
        return TodoWithCategory(
          todo: row.readTable(todos),
          category: row.readTableOrNull(categories),
        );
      }).toList();
    });
  }
  
  // Aggregate queries
  Future<Map<String, int>> getTodoCountByCategory() async {
    final countExpr = todos.id.count();
    final query = selectOnly(todos, also: [selectOnly(categories)])
      ..addColumns([todos.categoryId, countExpr])
      ..join([leftOuterJoin(categories, categories.id.equalsExp(todos.categoryId))])
      ..groupBy([todos.categoryId]);
    
    final rows = await query.get();
    return Map.fromEntries(
      rows.map((row) => MapEntry(
        row.read(todos.categoryId) ?? 'uncategorized',
        row.read(countExpr) ?? 0,
      )),
    );
  }
  
  // Sync operations
  Future<List<SyncOperation>> getPendingSyncOperations() {
    return (select(syncOperations)
      ..where((s) => s.status.isIn(['pending', 'failed']))
      ..where((s) => s.retryCount.isSmallerOrEqualValue(3))
      ..orderBy([(s) => OrderingTerm(expression: s.createdAt)]))
        .get();
  }
  
  Stream<int> watchPendingSyncCount() {
    final countExpr = syncOperations.id.count();
    return (selectOnly(syncOperations)
      ..addColumns([countExpr])
      ..where(syncOperations.status.equals('pending')))
        .watchSingle()
        .map((row) => row.read(countExpr) ?? 0);
  }
}

class TodoWithCategory {
  final Todo todo;
  final Category? category;
  
  TodoWithCategory({required this.todo, required this.category});
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'app.db'));
    return NativeDatabase(file);
  });
}
```

### Drift DAO Pattern

```dart
// lib/database/daos/todo_dao.dart
part of '../app_database.dart';

@DriftAccessor(tables: [Todos, Categories])
class TodoDao extends DatabaseAccessor<AppDatabase> with _$TodoDaoMixin {
  TodoDao(super.db);
  
  Future<List<Todo>> getAllTodos() => select(todos).get();
  
  Stream<List<Todo>> watchAllTodos() {
    return (select(todos)
      ..orderBy([
        (t) => OrderingTerm(expression: t.createdAt, mode: OrderingMode.desc),
      ]))
        .watch();
  }
  
  Future<void> insertTodo(TodosCompanion todo) =>
      into(todos).insert(todo);
  
  Future<bool> updateTodo(TodosCompanion todo) =>
      update(todos).replace(todo);
  
  Future<int> deleteTodo(String id) =>
      (delete(todos)..where((t) => t.id.equals(id))).go();
  
  Future<void> toggleComplete(String id) async {
    final todo = await getTodoById(id);
    if (todo != null) {
      await updateTodo(todo.toCompanion(true).copyWith(
        isCompleted: Value(!todo.isCompleted),
        updatedAt: Value(DateTime.now()),
      ));
    }
  }
  
  Future<Todo?> getTodoById(String id) =>
      (select(todos)..where((t) => t.id.equals(id))).getSingleOrNull();
}
```

---

## ขั้นตอนที่ 437: Hive สำหรับ NoSQL Local Storage

### Hive Setup

```dart
// lib/database/hive_setup.dart
import 'package:hive_flutter/hive_flutter.dart';

Future<void> initializeHive() async {
  await Hive.initFlutter();
  
  // Register adapters
  Hive.registerAdapter(UserAdapter());
  Hive.registerAdapter(SettingsAdapter());
  Hive.registerAdapter(CacheEntryAdapter());
  
  // Open boxes
  await Hive.openBox<User>('users');
  await Hive.openBox<Settings>('settings');
  await Hive.openBox<dynamic>('cache');
}
```

### Hive Models

```dart
// lib/models/hive_models.dart
import 'package:hive/hive.dart';

part 'hive_models.g.dart';

@HiveType(typeId: 0)
class User extends HiveObject {
  @HiveField(0)
  String id;
  
  @HiveField(1)
  String name;
  
  @HiveField(2)
  String email;
  
  @HiveField(3)
  String? avatarUrl;
  
  @HiveField(4)
  DateTime createdAt;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    this.avatarUrl,
    required this.createdAt,
  });
}

@HiveType(typeId: 1)
class Settings extends HiveObject {
  @HiveField(0)
  bool isDarkMode;
  
  @HiveField(1)
  String language;
  
  @HiveField(2)
  bool notificationsEnabled;
  
  @HiveField(3)
  String? lastSyncedAt;
  
  Settings({
    this.isDarkMode = false,
    this.language = 'th',
    this.notificationsEnabled = true,
    this.lastSyncedAt,
  });
}

@HiveType(typeId: 2)
class CacheEntry extends HiveObject {
  @HiveField(0)
  String key;
  
  @HiveField(1)
  dynamic value;
  
  @HiveField(2)
  DateTime expiresAt;
  
  CacheEntry({
    required this.key,
    required this.value,
    required this.expiresAt,
  });
  
  bool get isExpired => DateTime.now().isAfter(expiresAt);
}
```

### Hive Service

```dart
// lib/services/hive_cache_service.dart
class HiveCacheService {
  late Box<CacheEntry> _cacheBox;
  
  Future<void> initialize() async {
    _cacheBox = await Hive.openBox<CacheEntry>('cache');
    _cleanExpiredEntries();
  }
  
  Future<void> set<T>(
    String key,
    T value, {
    Duration ttl = const Duration(hours: 1),
  }) async {
    await _cacheBox.put(
      key,
      CacheEntry(
        key: key,
        value: value,
        expiresAt: DateTime.now().add(ttl),
      ),
    );
  }
  
  T? get<T>(String key) {
    final entry = _cacheBox.get(key);
    if (entry == null || entry.isExpired) return null;
    return entry.value as T?;
  }
  
  Future<T> getOrFetch<T>(
    String key,
    Future<T> Function() fetcher, {
    Duration ttl = const Duration(hours: 1),
  }) async {
    final cached = get<T>(key);
    if (cached != null) return cached;
    
    final value = await fetcher();
    await set(key, value, ttl: ttl);
    return value;
  }
  
  Future<void> invalidate(String key) async {
    await _cacheBox.delete(key);
  }
  
  Future<void> invalidateAll() async {
    await _cacheBox.clear();
  }
  
  void _cleanExpiredEntries() {
    final expiredKeys = _cacheBox.values
        .where((entry) => entry.isExpired)
        .map((entry) => entry.key)
        .toList();
    
    _cacheBox.deleteAll(expiredKeys);
  }
  
  // Watch สำหรับ reactive UI
  Stream<T?> watch<T>(String key) {
    return _cacheBox.watch(key: key).map(
      (event) => event.deleted ? null : (event.value as CacheEntry?)?.value as T?,
    );
  }
}

// Settings Service ด้วย Hive
class SettingsService {
  late Box<Settings> _settingsBox;
  static const String _key = 'app_settings';
  
  Future<void> initialize() async {
    _settingsBox = await Hive.openBox<Settings>('settings');
    
    // Initialize default settings
    if (!_settingsBox.containsKey(_key)) {
      await _settingsBox.put(_key, Settings());
    }
  }
  
  Settings get settings => _settingsBox.get(_key) ?? Settings();
  
  Stream<Settings?> get settingsStream {
    return _settingsBox.watch(key: _key).map(
      (event) => event.value as Settings?,
    );
  }
  
  Future<void> updateSettings(Settings Function(Settings) updater) async {
    final current = settings;
    await _settingsBox.put(_key, updater(current));
  }
  
  Future<void> setDarkMode(bool isDarkMode) async {
    await updateSettings((s) => s..isDarkMode = isDarkMode);
  }
  
  Future<void> setLanguage(String language) async {
    await updateSettings((s) => s..language = language);
  }
}
```

---

## ขั้นตอนที่ 438: Network-first vs Cache-first Strategies

### Fetch Strategies

```dart
// lib/repositories/fetch_strategies.dart
enum FetchStrategy {
  networkFirst,     // ดึงจาก network ก่อน, fallback to cache
  cacheFirst,       // ดึงจาก cache ก่อน, fallback to network  
  networkOnly,      // ดึงจาก network เท่านั้น
  cacheOnly,        // ดึงจาก cache เท่านั้น
  staleWhileRevalidate, // Return cache ทันที, แล้ว update ใน background
}

class FetchStrategyHandler<T> {
  final Future<T> Function() networkFetcher;
  final Future<T?> Function() cacheFetcher;
  final Future<void> Function(T data) cacheSaver;
  
  FetchStrategyHandler({
    required this.networkFetcher,
    required this.cacheFetcher,
    required this.cacheSaver,
  });
  
  Stream<T> fetch(FetchStrategy strategy) async* {
    switch (strategy) {
      case FetchStrategy.networkFirst:
        yield* _networkFirst();
      case FetchStrategy.cacheFirst:
        yield* _cacheFirst();
      case FetchStrategy.networkOnly:
        yield await networkFetcher();
      case FetchStrategy.cacheOnly:
        final cached = await cacheFetcher();
        if (cached != null) yield cached;
      case FetchStrategy.staleWhileRevalidate:
        yield* _staleWhileRevalidate();
    }
  }
  
  Stream<T> _networkFirst() async* {
    try {
      final networkData = await networkFetcher();
      await cacheSaver(networkData);
      yield networkData;
    } catch (e) {
      // Fallback to cache
      final cached = await cacheFetcher();
      if (cached != null) {
        yield cached;
      } else {
        rethrow;
      }
    }
  }
  
  Stream<T> _cacheFirst() async* {
    final cached = await cacheFetcher();
    
    if (cached != null) {
      yield cached;
    }
    
    try {
      final networkData = await networkFetcher();
      await cacheSaver(networkData);
      yield networkData;
    } catch (e) {
      if (cached == null) rethrow;
    }
  }
  
  Stream<T> _staleWhileRevalidate() async* {
    // Return cached data immediately
    final cached = await cacheFetcher();
    if (cached != null) yield cached;
    
    // Update in background
    try {
      final networkData = await networkFetcher();
      await cacheSaver(networkData);
      yield networkData;
    } catch (_) {
      // Silently fail if already returned cached data
    }
  }
}

// ตัวอย่างการใช้งาน
class ProductRepository {
  final ApiService _api;
  final HiveCacheService _cache;
  
  ProductRepository(this._api, this._cache);
  
  Stream<List<Product>> getProducts({
    FetchStrategy strategy = FetchStrategy.staleWhileRevalidate,
  }) {
    final handler = FetchStrategyHandler<List<Product>>(
      networkFetcher: () async {
        final response = await _api.get('/products');
        return (response['data'] as List)
            .map((p) => Product.fromJson(p))
            .toList();
      },
      cacheFetcher: () async {
        return _cache.get<List<Product>>('products');
      },
      cacheSaver: (products) async {
        await _cache.set('products', products, ttl: const Duration(hours: 1));
      },
    );
    
    return handler.fetch(strategy);
  }
}
```

---

## ขั้นตอนที่ 439: Data Synchronization Patterns

### Delta Sync Pattern

```dart
// lib/offline/delta_sync.dart
class DeltaSyncService {
  final ApiService _api;
  final AppDatabase _db;
  DateTime? _lastSyncTime;
  
  DeltaSyncService(this._api, this._db);
  
  Future<void> sync() async {
    final since = _lastSyncTime;
    
    try {
      // ดึง changes ตั้งแต่ครั้งล่าสุด
      final response = await _api.get('/sync/delta', params: {
        if (since != null) 'since': since.toIso8601String(),
      });
      
      final changes = DeltaChanges.fromJson(response);
      
      // Apply changes
      await _applyChanges(changes);
      
      // Push local changes
      await _pushLocalChanges(since);
      
      _lastSyncTime = DateTime.now();
      await _saveLastSyncTime(_lastSyncTime!);
    } catch (e) {
      print('Delta sync failed: $e');
      rethrow;
    }
  }
  
  Future<void> _applyChanges(DeltaChanges changes) async {
    await _db.transaction(() async {
      // Apply creates
      for (final item in changes.created) {
        await _db.saveTodo(TodosCompanion.insert(
          id: item.id,
          title: item.title,
          description: Value(item.description ?? ''),
          createdAt: item.createdAt,
          updatedAt: item.updatedAt,
        ));
      }
      
      // Apply updates
      for (final item in changes.updated) {
        await _db.updateTodo(TodosCompanion(
          id: Value(item.id),
          title: Value(item.title),
          description: Value(item.description ?? ''),
          updatedAt: Value(item.updatedAt),
        ));
      }
      
      // Apply deletes
      for (final id in changes.deleted) {
        await _db.deleteTodo(id);
      }
    });
  }
  
  Future<void> _pushLocalChanges(DateTime? since) async {
    final pendingOps = await _db.getPendingSyncOperations();
    
    if (pendingOps.isEmpty) return;
    
    await _api.post('/sync/batch', {
      'operations': pendingOps.map((op) => op.toJson()).toList(),
    });
    
    // Mark all as completed
    for (final op in pendingOps) {
      await _db.updateSyncOperationStatus(op.id, SyncStatus.completed);
    }
  }
  
  Future<void> _saveLastSyncTime(DateTime time) async {
    // Save to secure storage
    await SecureStorage().write(
      'last_sync_time',
      time.toIso8601String(),
    );
  }
}

class DeltaChanges {
  final List<TodoItem> created;
  final List<TodoItem> updated;
  final List<String> deleted;
  final DateTime serverTime;
  
  DeltaChanges({
    required this.created,
    required this.updated,
    required this.deleted,
    required this.serverTime,
  });
  
  factory DeltaChanges.fromJson(Map<String, dynamic> json) {
    return DeltaChanges(
      created: (json['created'] as List)
          .map((e) => TodoItem.fromJson(e))
          .toList(),
      updated: (json['updated'] as List)
          .map((e) => TodoItem.fromJson(e))
          .toList(),
      deleted: List<String>.from(json['deleted'] as List),
      serverTime: DateTime.parse(json['serverTime'] as String),
    );
  }
}
```

---

## ขั้นตอนที่ 440: Workshop - Offline Todo App

### Todo App ที่รองรับ Offline

```dart
// lib/features/todo/screens/todo_screen.dart
class TodoScreen extends ConsumerWidget {
  const TodoScreen({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoNotifierProvider);
    final syncCount = ref.watch(pendingSyncCountProvider);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('สิ่งที่ต้องทำ'),
        actions: [
          if (syncCount > 0)
            Chip(
              label: Text('$syncCount รอ sync'),
              avatar: const Icon(Icons.sync, size: 16),
              backgroundColor: Colors.orange,
            ),
          const SizedBox(width: 8),
          const NetworkStatusIcon(),
        ],
      ),
      body: todosAsync.when(
        data: (todos) {
          if (todos.isEmpty) {
            return _buildEmptyState();
          }
          
          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: todos.length,
            itemBuilder: (context, index) {
              final todo = todos[index];
              return TodoCard(
                todo: todo,
                onToggle: () => ref.read(todoNotifierProvider.notifier)
                    .toggleComplete(todo.id),
                onDelete: () => ref.read(todoNotifierProvider.notifier)
                    .deleteTodo(todo.id),
              );
            },
          );
        },
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, _) => Center(child: Text('Error: $error')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddTodoDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }
  
  Widget _buildEmptyState() {
    return const Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.check_circle_outline, size: 64, color: Colors.grey),
          SizedBox(height: 16),
          Text(
            'ยังไม่มีรายการ',
            style: TextStyle(fontSize: 18, color: Colors.grey),
          ),
          SizedBox(height: 8),
          Text(
            'กดปุ่ม + เพื่อเพิ่มรายการใหม่',
            style: TextStyle(color: Colors.grey),
          ),
        ],
      ),
    );
  }
  
  Future<void> _showAddTodoDialog(BuildContext context, WidgetRef ref) async {
    final titleController = TextEditingController();
    
    await showDialog<void>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('เพิ่มรายการ'),
        content: TextField(
          controller: titleController,
          decoration: const InputDecoration(
            hintText: 'ชื่อรายการ',
          ),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () {
              if (titleController.text.isNotEmpty) {
                ref.read(todoNotifierProvider.notifier).addTodo(
                  titleController.text,
                  '',
                );
                Navigator.pop(context);
              }
            },
            child: const Text('เพิ่ม'),
          ),
        ],
      ),
    );
    
    titleController.dispose();
  }
}

// Network Status Icon
class NetworkStatusIcon extends ConsumerWidget {
  const NetworkStatusIcon({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return StreamBuilder<bool>(
      stream: ref.read(networkServiceProvider).isOnlineStream,
      builder: (context, snapshot) {
        final isOnline = snapshot.data ?? true;
        
        return Icon(
          isOnline ? Icons.wifi : Icons.wifi_off,
          color: isOnline ? Colors.green : Colors.red,
        );
      },
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **Offline-First Principles**: แนวคิดและหลักการออกแบบ
- **Sync Queue**: จัดการ pending operations
- **Conflict Resolution**: กลยุทธ์แก้ไข conflicts (LWW, Server Wins, Merge)
- **Optimistic Updates**: Update UI ก่อน server ยืนยัน
- **Background Sync**: sync ด้วย WorkManager
- **Drift**: SQL database สำหรับ complex queries
- **Hive**: NoSQL storage ที่เร็วและง่าย
- **Fetch Strategies**: Network-first, Cache-first, Stale-while-revalidate

## แบบฝึกหัด

1. สร้าง offline-first notes app ด้วย Drift
2. Implement sync queue ที่มี retry mechanism
3. เพิ่ม conflict resolution แบบ three-way merge
4. ตั้งค่า background sync ด้วย WorkManager
5. เพิ่ม sync status UI ที่แสดงจำนวน pending items

---

[⬅️ Part 43](part_43.md) | [Part 45 ➡️](part_45.md)

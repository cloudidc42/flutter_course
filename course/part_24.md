# Part 24: Cloud Firestore
## ขั้นตอนที่ 231-240

---

## สารบัญ
1. [cloud_firestore Package](#cloud_firestore-package)
2. [CRUD Operations](#crud-operations)
3. [Real-time Listeners](#real-time-listeners)
4. [Complex Queries](#complex-queries)
5. [Pagination with Cursors](#pagination-with-cursors)
6. [Batch Writes](#batch-writes)
7. [Transactions](#transactions)
8. [Security Rules](#security-rules)
9. [Offline Persistence](#offline-persistence)
10. [Subcollections](#subcollections)

---

## ขั้นตอนที่ 231: cloud_firestore Package

Cloud Firestore เป็น NoSQL Database แบบ Document-based ที่รองรับ Real-time sync

```yaml
# pubspec.yaml
dependencies:
  cloud_firestore: ^4.14.0
  firebase_auth: ^4.16.0
```

### โครงสร้างข้อมูลใน Firestore

```
Firestore
└── users (Collection)
    ├── user_abc123 (Document)
    │   ├── name: "สมชาย"
    │   ├── email: "somchai@email.com"
    │   ├── age: 25
    │   └── posts (Subcollection)
    │       ├── post_001 (Document)
    │       │   ├── title: "Hello"
    │       │   └── content: "World"
    │       └── post_002 (Document)
    └── user_xyz789 (Document)
        └── name: "สมหญิง"
```

```dart
// lib/services/firestore_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class FirestoreService {
  static final FirebaseFirestore _db = FirebaseFirestore.instance;

  // Reference shortcuts
  static CollectionReference get usersRef => _db.collection('users');
  static CollectionReference get postsRef => _db.collection('posts');
  static CollectionReference get productsRef => _db.collection('products');
}
```

---

## ขั้นตอนที่ 232: CRUD Operations

### Model Class

```dart
// lib/models/post.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Post {
  const Post({
    this.id,
    required this.title,
    required this.content,
    required this.authorId,
    required this.authorName,
    this.imageUrl,
    this.likes = 0,
    this.tags = const [],
    DateTime? createdAt,
    DateTime? updatedAt,
  })  : createdAt = createdAt,
        updatedAt = updatedAt;

  final String? id;
  final String title;
  final String content;
  final String authorId;
  final String authorName;
  final String? imageUrl;
  final int likes;
  final List<String> tags;
  final DateTime? createdAt;
  final DateTime? updatedAt;

  // สร้างจาก Firestore Document
  factory Post.fromFirestore(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return Post(
      id: doc.id,
      title: data['title'] as String? ?? '',
      content: data['content'] as String? ?? '',
      authorId: data['authorId'] as String? ?? '',
      authorName: data['authorName'] as String? ?? '',
      imageUrl: data['imageUrl'] as String?,
      likes: data['likes'] as int? ?? 0,
      tags: List<String>.from(data['tags'] as List? ?? []),
      createdAt: (data['createdAt'] as Timestamp?)?.toDate(),
      updatedAt: (data['updatedAt'] as Timestamp?)?.toDate(),
    );
  }

  // แปลงเป็น Map สำหรับบันทึกใน Firestore
  Map<String, dynamic> toFirestore() {
    return {
      'title': title,
      'content': content,
      'authorId': authorId,
      'authorName': authorName,
      'imageUrl': imageUrl,
      'likes': likes,
      'tags': tags,
      'createdAt': createdAt != null
          ? Timestamp.fromDate(createdAt!)
          : FieldValue.serverTimestamp(),
      'updatedAt': FieldValue.serverTimestamp(),
    };
  }

  Post copyWith({
    String? id,
    String? title,
    String? content,
    String? authorId,
    String? authorName,
    String? imageUrl,
    int? likes,
    List<String>? tags,
  }) {
    return Post(
      id: id ?? this.id,
      title: title ?? this.title,
      content: content ?? this.content,
      authorId: authorId ?? this.authorId,
      authorName: authorName ?? this.authorName,
      imageUrl: imageUrl ?? this.imageUrl,
      likes: likes ?? this.likes,
      tags: tags ?? this.tags,
      createdAt: createdAt,
      updatedAt: updatedAt,
    );
  }
}
```

### CRUD Service

```dart
// lib/services/post_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/post.dart';

class PostService {
  static final _postsRef =
      FirebaseFirestore.instance.collection('posts').withConverter<Post>(
            fromFirestore: (snap, _) => Post.fromFirestore(snap),
            toFirestore: (post, _) => post.toFirestore(),
          );

  // CREATE
  static Future<DocumentReference<Post>> createPost(Post post) async {
    return await _postsRef.add(post);
  }

  // READ - อ่าน Document เดียว
  static Future<Post?> getPost(String id) async {
    final doc = await _postsRef.doc(id).get();
    return doc.data();
  }

  // READ - อ่านทุก Document
  static Future<List<Post>> getAllPosts() async {
    final snapshot = await _postsRef.get();
    return snapshot.docs.map((doc) => doc.data()).toList();
  }

  // UPDATE - อัพเดตบางฟิลด์
  static Future<void> updatePost(String id, Map<String, dynamic> data) async {
    await FirebaseFirestore.instance
        .collection('posts')
        .doc(id)
        .update({...data, 'updatedAt': FieldValue.serverTimestamp()});
  }

  // UPDATE - เพิ่ม/ลด like
  static Future<void> likePost(String id) async {
    await FirebaseFirestore.instance
        .collection('posts')
        .doc(id)
        .update({'likes': FieldValue.increment(1)});
  }

  static Future<void> unlikePost(String id) async {
    await FirebaseFirestore.instance
        .collection('posts')
        .doc(id)
        .update({'likes': FieldValue.increment(-1)});
  }

  // DELETE
  static Future<void> deletePost(String id) async {
    await _postsRef.doc(id).delete();
  }

  // SET (Create or Replace)
  static Future<void> setPost(String id, Post post) async {
    await _postsRef.doc(id).set(post);
  }
}
```

### UI สำหรับ CRUD

```dart
// lib/pages/posts_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../models/post.dart';
import '../services/post_service.dart';

class PostsPage extends StatelessWidget {
  const PostsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('โพสต์ทั้งหมด')),
      body: FutureBuilder<List<Post>>(
        future: PostService.getAllPosts(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          }
          final posts = snapshot.data ?? [];
          if (posts.isEmpty) {
            return const Center(child: Text('ยังไม่มีโพสต์'));
          }
          return ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) {
              final post = posts[index];
              return PostCard(post: post);
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showCreatePostDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showCreatePostDialog(BuildContext context) {
    final titleCtrl = TextEditingController();
    final contentCtrl = TextEditingController();

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (ctx) => Padding(
        padding: EdgeInsets.only(
          left: 16,
          right: 16,
          top: 16,
          bottom: MediaQuery.of(ctx).viewInsets.bottom + 16,
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            const Text(
              'สร้างโพสต์ใหม่',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: titleCtrl,
              decoration: const InputDecoration(
                labelText: 'หัวข้อ',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 12),
            TextField(
              controller: contentCtrl,
              decoration: const InputDecoration(
                labelText: 'เนื้อหา',
                border: OutlineInputBorder(),
              ),
              maxLines: 3,
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () async {
                final user = FirebaseAuth.instance.currentUser;
                if (user == null) return;

                final post = Post(
                  title: titleCtrl.text,
                  content: contentCtrl.text,
                  authorId: user.uid,
                  authorName: user.displayName ?? 'Anonymous',
                );

                await PostService.createPost(post);
                if (ctx.mounted) Navigator.pop(ctx);
              },
              child: const Text('โพสต์'),
            ),
          ],
        ),
      ),
    );
  }
}

class PostCard extends StatelessWidget {
  const PostCard({super.key, required this.post});
  final Post post;

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              post.title,
              style: const TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Text(post.content),
            const SizedBox(height: 8),
            Row(
              children: [
                Text(
                  'โดย ${post.authorName}',
                  style: const TextStyle(color: Colors.grey),
                ),
                const Spacer(),
                IconButton(
                  icon: const Icon(Icons.favorite_border),
                  onPressed: () {
                    if (post.id != null) PostService.likePost(post.id!);
                  },
                ),
                Text('${post.likes}'),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 233: Real-time Listeners

```dart
// lib/pages/realtime_posts_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/post.dart';

class RealtimePostsPage extends StatelessWidget {
  const RealtimePostsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Real-time Posts'),
        actions: [
          // แสดงสัญลักษณ์ว่ากำลัง sync
          StreamBuilder(
            stream: FirebaseFirestore.instance.snapshotsInSync(),
            builder: (context, snapshot) {
              return Icon(
                snapshot.connectionState == ConnectionState.active
                    ? Icons.sync
                    : Icons.sync_disabled,
                color: Colors.white,
              );
            },
          ),
        ],
      ),
      body: StreamBuilder<QuerySnapshot<Post>>(
        stream: FirebaseFirestore.instance
            .collection('posts')
            .withConverter<Post>(
              fromFirestore: (snap, _) => Post.fromFirestore(snap),
              toFirestore: (post, _) => post.toFirestore(),
            )
            .orderBy('createdAt', descending: true)
            .snapshots(),
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          }
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          final posts = snapshot.data?.docs.map((doc) => doc.data()).toList() ?? [];

          if (posts.isEmpty) {
            return const Center(child: Text('ยังไม่มีโพสต์'));
          }

          return ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) {
              return PostCard(post: posts[index]);
            },
          );
        },
      ),
    );
  }
}

// ฟัง Document เดียว
class PostDetailPage extends StatelessWidget {
  const PostDetailPage({super.key, required this.postId});
  final String postId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('โพสต์')),
      body: StreamBuilder<DocumentSnapshot<Post>>(
        stream: FirebaseFirestore.instance
            .collection('posts')
            .withConverter<Post>(
              fromFirestore: (snap, _) => Post.fromFirestore(snap),
              toFirestore: (post, _) => post.toFirestore(),
            )
            .doc(postId)
            .snapshots(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return const Center(child: CircularProgressIndicator());
          }

          final post = snapshot.data?.data();
          if (post == null) {
            return const Center(child: Text('ไม่พบโพสต์'));
          }

          return Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  post.title,
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text('โดย ${post.authorName}',
                    style: const TextStyle(color: Colors.grey)),
                const Divider(),
                Text(post.content, style: const TextStyle(fontSize: 16)),
                const SizedBox(height: 16),
                Row(
                  children: [
                    Icon(Icons.favorite, color: Colors.red),
                    const SizedBox(width: 4),
                    Text('${post.likes} likes'),
                  ],
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 234: Complex Queries

```dart
// lib/services/query_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/post.dart';

class QueryService {
  static final _db = FirebaseFirestore.instance;

  // where - กรองข้อมูล
  static Stream<List<Post>> getPostsByAuthor(String authorId) {
    return _db
        .collection('posts')
        .where('authorId', isEqualTo: authorId)
        .snapshots()
        .map((snap) => snap.docs.map((doc) => Post.fromFirestore(doc)).toList());
  }

  // where หลายเงื่อนไข
  static Stream<List<Post>> getPopularRecentPosts() {
    return _db
        .collection('posts')
        .where('likes', isGreaterThan: 10)
        .orderBy('likes', descending: true)
        .orderBy('createdAt', descending: true)
        .limit(20)
        .snapshots()
        .map((snap) => snap.docs.map((doc) => Post.fromFirestore(doc)).toList());
  }

  // in - กรองหลาย values
  static Future<List<Post>> getPostsByIds(List<String> ids) async {
    if (ids.isEmpty) return [];
    // Firestore รองรับ whereIn ได้สูงสุด 10 values
    final snapshot = await _db
        .collection('posts')
        .where(FieldPath.documentId, whereIn: ids)
        .get();
    return snapshot.docs.map((doc) => Post.fromFirestore(doc)).toList();
  }

  // array-contains
  static Stream<List<Post>> getPostsByTag(String tag) {
    return _db
        .collection('posts')
        .where('tags', arrayContains: tag)
        .snapshots()
        .map((snap) => snap.docs.map((doc) => Post.fromFirestore(doc)).toList());
  }

  // array-contains-any
  static Stream<List<Post>> getPostsByAnyTag(List<String> tags) {
    return _db
        .collection('posts')
        .where('tags', arrayContainsAny: tags)
        .snapshots()
        .map((snap) => snap.docs.map((doc) => Post.fromFirestore(doc)).toList());
  }

  // orderBy + limit
  static Future<List<Post>> getLatestPosts({int limit = 10}) async {
    final snapshot = await _db
        .collection('posts')
        .orderBy('createdAt', descending: true)
        .limit(limit)
        .get();
    return snapshot.docs.map((doc) => Post.fromFirestore(doc)).toList();
  }

  // between range
  static Future<List<Post>> getPostsWithLikesBetween(
      int min, int max) async {
    final snapshot = await _db
        .collection('posts')
        .where('likes', isGreaterThanOrEqualTo: min)
        .where('likes', isLessThanOrEqualTo: max)
        .get();
    return snapshot.docs.map((doc) => Post.fromFirestore(doc)).toList();
  }
}
```

---

## ขั้นตอนที่ 235: Pagination with Cursors

```dart
// lib/services/pagination_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/post.dart';

class PaginationService {
  static final _db = FirebaseFirestore.instance;
  static const int _pageSize = 10;

  // หน้าแรก
  static Future<QuerySnapshot> getFirstPage() async {
    return await _db
        .collection('posts')
        .orderBy('createdAt', descending: true)
        .limit(_pageSize)
        .get();
  }

  // หน้าถัดไปโดยใช้ startAfter
  static Future<QuerySnapshot> getNextPage(
      DocumentSnapshot lastDocument) async {
    return await _db
        .collection('posts')
        .orderBy('createdAt', descending: true)
        .startAfterDocument(lastDocument)
        .limit(_pageSize)
        .get();
  }

  // ใช้ startAt (รวม document ที่ระบุ)
  static Future<QuerySnapshot> getPageStartingAt(
      DocumentSnapshot startDocument) async {
    return await _db
        .collection('posts')
        .orderBy('createdAt', descending: true)
        .startAtDocument(startDocument)
        .limit(_pageSize)
        .get();
  }
}

// lib/pages/paginated_posts_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/post.dart';
import '../services/pagination_service.dart';

class PaginatedPostsPage extends StatefulWidget {
  const PaginatedPostsPage({super.key});

  @override
  State<PaginatedPostsPage> createState() => _PaginatedPostsPageState();
}

class _PaginatedPostsPageState extends State<PaginatedPostsPage> {
  final List<Post> _posts = [];
  DocumentSnapshot? _lastDocument;
  bool _isLoading = false;
  bool _hasMore = true;
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _loadFirstPage();
    _scrollController.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent - 200) {
      _loadMore();
    }
  }

  Future<void> _loadFirstPage() async {
    setState(() => _isLoading = true);
    try {
      final snapshot = await PaginationService.getFirstPage();
      if (snapshot.docs.isNotEmpty) {
        _lastDocument = snapshot.docs.last;
        _posts.addAll(
            snapshot.docs.map((doc) => Post.fromFirestore(doc)).toList());
        _hasMore = snapshot.docs.length == 10;
      } else {
        _hasMore = false;
      }
    } catch (e) {
      debugPrint('Error loading posts: $e');
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  Future<void> _loadMore() async {
    if (_isLoading || !_hasMore || _lastDocument == null) return;

    setState(() => _isLoading = true);
    try {
      final snapshot =
          await PaginationService.getNextPage(_lastDocument!);
      if (snapshot.docs.isNotEmpty) {
        _lastDocument = snapshot.docs.last;
        _posts.addAll(
            snapshot.docs.map((doc) => Post.fromFirestore(doc)).toList());
        _hasMore = snapshot.docs.length == 10;
      } else {
        _hasMore = false;
      }
    } catch (e) {
      debugPrint('Error loading more posts: $e');
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  Future<void> _refresh() async {
    _posts.clear();
    _lastDocument = null;
    _hasMore = true;
    await _loadFirstPage();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Paginated Posts')),
      body: RefreshIndicator(
        onRefresh: _refresh,
        child: _posts.isEmpty && !_isLoading
            ? const Center(child: Text('ไม่มีโพสต์'))
            : ListView.builder(
                controller: _scrollController,
                itemCount: _posts.length + (_hasMore ? 1 : 0),
                itemBuilder: (context, index) {
                  if (index == _posts.length) {
                    return _isLoading
                        ? const Center(
                            child: Padding(
                              padding: EdgeInsets.all(16),
                              child: CircularProgressIndicator(),
                            ),
                          )
                        : const SizedBox.shrink();
                  }
                  return PostCard(post: _posts[index]);
                },
              ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 236: Batch Writes

```dart
// lib/services/batch_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class BatchService {
  static final _db = FirebaseFirestore.instance;

  // Batch Write - เขียนหลาย Documents พร้อมกัน
  static Future<void> batchCreateUsers(List<Map<String, dynamic>> users) async {
    final batch = _db.batch();

    for (final userData in users) {
      final docRef = _db.collection('users').doc();
      batch.set(docRef, {
        ...userData,
        'createdAt': FieldValue.serverTimestamp(),
      });
    }

    await batch.commit();
  }

  // Batch Update
  static Future<void> batchUpdatePosts(
      List<String> postIds, Map<String, dynamic> updates) async {
    final batch = _db.batch();

    for (final id in postIds) {
      batch.update(_db.collection('posts').doc(id), {
        ...updates,
        'updatedAt': FieldValue.serverTimestamp(),
      });
    }

    await batch.commit();
  }

  // Batch Delete
  static Future<void> batchDeletePosts(List<String> postIds) async {
    // Firestore batch รองรับสูงสุด 500 operations
    const batchSize = 500;

    for (var i = 0; i < postIds.length; i += batchSize) {
      final batch = _db.batch();
      final chunk = postIds.skip(i).take(batchSize);

      for (final id in chunk) {
        batch.delete(_db.collection('posts').doc(id));
      }

      await batch.commit();
    }
  }

  // Mixed Batch Operations
  static Future<void> transferData() async {
    final batch = _db.batch();

    // Create
    final newRef = _db.collection('archive').doc();
    batch.set(newRef, {'data': 'archived'});

    // Update
    batch.update(
      _db.collection('stats').doc('global'),
      {'archiveCount': FieldValue.increment(1)},
    );

    // Delete
    batch.delete(_db.collection('temp').doc('item1'));

    await batch.commit();
  }
}
```

---

## ขั้นตอนที่ 237: Transactions

```dart
// lib/services/transaction_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class TransactionService {
  static final _db = FirebaseFirestore.instance;

  // Transaction: Like post แบบ Atomic
  static Future<void> likePost(String postId, String userId) async {
    await _db.runTransaction((transaction) async {
      final postDoc = await transaction.get(
        _db.collection('posts').doc(postId),
      );

      if (!postDoc.exists) throw Exception('ไม่พบโพสต์');

      final data = postDoc.data()!;
      final likedBy = List<String>.from(data['likedBy'] as List? ?? []);

      if (likedBy.contains(userId)) {
        // Unlike
        likedBy.remove(userId);
        transaction.update(postDoc.reference, {
          'likes': FieldValue.increment(-1),
          'likedBy': likedBy,
        });
      } else {
        // Like
        likedBy.add(userId);
        transaction.update(postDoc.reference, {
          'likes': FieldValue.increment(1),
          'likedBy': likedBy,
        });
      }
    });
  }

  // Transaction: โอนเงินระหว่าง Account
  static Future<void> transferMoney({
    required String fromUserId,
    required String toUserId,
    required double amount,
  }) async {
    await _db.runTransaction((transaction) async {
      final fromDoc = await transaction.get(
        _db.collection('wallets').doc(fromUserId),
      );
      final toDoc = await transaction.get(
        _db.collection('wallets').doc(toUserId),
      );

      if (!fromDoc.exists || !toDoc.exists) {
        throw Exception('ไม่พบ wallet');
      }

      final fromBalance = (fromDoc.data()!['balance'] as num).toDouble();
      if (fromBalance < amount) {
        throw Exception('ยอดเงินไม่เพียงพอ');
      }

      // อัพเดตยอดเงิน
      transaction.update(fromDoc.reference, {
        'balance': FieldValue.increment(-amount),
      });
      transaction.update(toDoc.reference, {
        'balance': FieldValue.increment(amount),
      });

      // บันทึก Transaction History
      final historyRef = _db.collection('transactions').doc();
      transaction.set(historyRef, {
        'from': fromUserId,
        'to': toUserId,
        'amount': amount,
        'timestamp': FieldValue.serverTimestamp(),
      });
    });
  }
}
```

---

## ขั้นตอนที่ 238: Security Rules

```javascript
// firestore.rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }
    
    function isOwner(userId) {
      return request.auth.uid == userId;
    }
    
    function isAdmin() {
      return request.auth.token.admin == true;
    }
    
    function isValidPost() {
      return request.resource.data.keys().hasAll(['title', 'content', 'authorId'])
        && request.resource.data.title is string
        && request.resource.data.title.size() > 0
        && request.resource.data.title.size() <= 200
        && request.resource.data.content is string;
    }
    
    // Users Collection
    match /users/{userId} {
      allow read: if isAuthenticated();
      allow create: if isAuthenticated() && isOwner(userId);
      allow update: if isAuthenticated() && isOwner(userId);
      allow delete: if isAdmin();
    }
    
    // Posts Collection
    match /posts/{postId} {
      allow read: if true; // Public readable
      allow create: if isAuthenticated() && isValidPost()
        && request.resource.data.authorId == request.auth.uid;
      allow update: if isAuthenticated() && (
        // Owner สามารถ update ทุกฟิลด์
        isOwner(resource.data.authorId) ||
        // ทุกคนสามารถ like
        (request.resource.data.diff(resource.data).affectedKeys()
          .hasOnly(['likes', 'likedBy']))
      );
      allow delete: if isAuthenticated() && 
        (isOwner(resource.data.authorId) || isAdmin());
    }
    
    // Comments Subcollection
    match /posts/{postId}/comments/{commentId} {
      allow read: if true;
      allow create: if isAuthenticated();
      allow update, delete: if isAuthenticated() && 
        isOwner(resource.data.authorId);
    }
    
    // Admin only
    match /admin/{document=**} {
      allow read, write: if isAdmin();
    }
  }
}
```

---

## ขั้นตอนที่ 239: Offline Persistence

```dart
// lib/main.dart - เปิด Offline Persistence
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  // เปิด Offline Persistence (เปิดอยู่แล้วโดย default บน mobile)
  FirebaseFirestore.instance.settings = const Settings(
    persistenceEnabled: true,
    cacheSizeBytes: Settings.CACHE_SIZE_UNLIMITED,
  );

  runApp(const MyApp());
}

// lib/services/offline_aware_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class OfflineAwareService {
  static final _db = FirebaseFirestore.instance;

  // อ่านจาก Cache ก่อน (เร็วกว่า)
  static Future<DocumentSnapshot> getFromCache(
      String collection, String docId) async {
    try {
      return await _db
          .collection(collection)
          .doc(docId)
          .get(const GetOptions(source: Source.cache));
    } catch (_) {
      // ถ้าไม่มีใน Cache ให้อ่านจาก Server
      return await _db.collection(collection).doc(docId).get();
    }
  }

  // อ่านจาก Server เสมอ
  static Future<DocumentSnapshot> getFromServer(
      String collection, String docId) async {
    return await _db
        .collection(collection)
        .doc(docId)
        .get(const GetOptions(source: Source.server));
  }

  // เขียนแบบ Optimistic (ไม่ต้อง await)
  static void writeOptimistic(
      String collection, String docId, Map<String, dynamic> data) {
    _db
        .collection(collection)
        .doc(docId)
        .set(data)
        .catchError((e) => print('Sync error: $e'));
  }
}

// lib/pages/offline_demo_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:connectivity_plus/connectivity_plus.dart';

class OfflineDemoPage extends StatelessWidget {
  const OfflineDemoPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Offline Support'),
        actions: [
          StreamBuilder<ConnectivityResult>(
            stream: Connectivity().onConnectivityChanged,
            builder: (context, snapshot) {
              final isOnline = snapshot.data != ConnectivityResult.none;
              return Icon(
                isOnline ? Icons.wifi : Icons.wifi_off,
                color: Colors.white,
              );
            },
          ),
          const SizedBox(width: 8),
        ],
      ),
      body: StreamBuilder<QuerySnapshot>(
        stream: FirebaseFirestore.instance
            .collection('notes')
            .orderBy('createdAt', descending: true)
            .snapshots(includeMetadataChanges: true),
        builder: (context, snapshot) {
          if (!snapshot.hasData) {
            return const Center(child: CircularProgressIndicator());
          }

          final docs = snapshot.data!.docs;
          final isFromCache = snapshot.data!.metadata.isFromCache;

          return Column(
            children: [
              if (isFromCache)
                Container(
                  color: Colors.orange[100],
                  padding: const EdgeInsets.all(8),
                  child: const Row(
                    children: [
                      Icon(Icons.offline_bolt, color: Colors.orange),
                      SizedBox(width: 8),
                      Text('แสดงข้อมูลจาก Cache (Offline)'),
                    ],
                  ),
                ),
              Expanded(
                child: ListView.builder(
                  itemCount: docs.length,
                  itemBuilder: (context, index) {
                    final doc = docs[index];
                    final isPending = doc.metadata.hasPendingWrites;
                    return ListTile(
                      title: Text(doc['title'] as String? ?? ''),
                      trailing: isPending
                          ? const Icon(Icons.pending, color: Colors.orange)
                          : const Icon(Icons.check, color: Colors.green),
                    );
                  },
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 240: Subcollections

```dart
// lib/services/subcollection_service.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class SubcollectionService {
  static final _db = FirebaseFirestore.instance;

  // เพิ่ม Comment ใน Post
  static Future<DocumentReference> addComment({
    required String postId,
    required String content,
    required String authorId,
    required String authorName,
  }) async {
    return await _db
        .collection('posts')
        .doc(postId)
        .collection('comments')
        .add({
      'content': content,
      'authorId': authorId,
      'authorName': authorName,
      'likes': 0,
      'createdAt': FieldValue.serverTimestamp(),
    });
  }

  // ดึง Comments แบบ Real-time
  static Stream<QuerySnapshot> getCommentsStream(String postId) {
    return _db
        .collection('posts')
        .doc(postId)
        .collection('comments')
        .orderBy('createdAt', descending: false)
        .snapshots();
  }

  // CollectionGroup - ค้นหาข้ามทุก subcollections
  static Stream<QuerySnapshot> getAllCommentsByUser(String userId) {
    return _db
        .collectionGroup('comments')
        .where('authorId', isEqualTo: userId)
        .snapshots();
  }

  // ลบ Post พร้อม Comments ทั้งหมด
  static Future<void> deletePostWithComments(String postId) async {
    final batch = _db.batch();

    // ดึง Comments
    final comments = await _db
        .collection('posts')
        .doc(postId)
        .collection('comments')
        .get();

    // เพิ่มใน Batch
    for (final comment in comments.docs) {
      batch.delete(comment.reference);
    }

    // ลบ Post
    batch.delete(_db.collection('posts').doc(postId));

    await batch.commit();
  }
}

// lib/pages/post_comments_page.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../services/subcollection_service.dart';

class PostCommentsPage extends StatefulWidget {
  const PostCommentsPage({
    super.key,
    required this.postId,
    required this.postTitle,
  });

  final String postId;
  final String postTitle;

  @override
  State<PostCommentsPage> createState() => _PostCommentsPageState();
}

class _PostCommentsPageState extends State<PostCommentsPage> {
  final _commentController = TextEditingController();

  @override
  void dispose() {
    _commentController.dispose();
    super.dispose();
  }

  Future<void> _addComment() async {
    final user = FirebaseAuth.instance.currentUser;
    if (user == null || _commentController.text.isEmpty) return;

    await SubcollectionService.addComment(
      postId: widget.postId,
      content: _commentController.text.trim(),
      authorId: user.uid,
      authorName: user.displayName ?? 'Anonymous',
    );
    _commentController.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(widget.postTitle)),
      body: Column(
        children: [
          Expanded(
            child: StreamBuilder<QuerySnapshot>(
              stream:
                  SubcollectionService.getCommentsStream(widget.postId),
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return const Center(child: CircularProgressIndicator());
                }

                final comments = snapshot.data!.docs;

                if (comments.isEmpty) {
                  return const Center(child: Text('ยังไม่มีความคิดเห็น'));
                }

                return ListView.builder(
                  padding: const EdgeInsets.all(16),
                  itemCount: comments.length,
                  itemBuilder: (context, index) {
                    final data =
                        comments[index].data() as Map<String, dynamic>;
                    return _CommentTile(
                      authorName: data['authorName'] as String? ?? '',
                      content: data['content'] as String? ?? '',
                      timestamp: data['createdAt'] as Timestamp?,
                    );
                  },
                );
              },
            ),
          ),
          Container(
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              border: Border(top: BorderSide(color: Colors.grey.shade300)),
            ),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _commentController,
                    decoration: const InputDecoration(
                      hintText: 'เขียนความคิดเห็น...',
                      border: OutlineInputBorder(),
                    ),
                    maxLines: null,
                  ),
                ),
                const SizedBox(width: 8),
                IconButton(
                  onPressed: _addComment,
                  icon: const Icon(Icons.send),
                  color: Colors.blue,
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _CommentTile extends StatelessWidget {
  const _CommentTile({
    required this.authorName,
    required this.content,
    this.timestamp,
  });

  final String authorName;
  final String content;
  final Timestamp? timestamp;

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.only(bottom: 12),
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Row(
            children: [
              CircleAvatar(
                radius: 16,
                child: Text(authorName.isNotEmpty
                    ? authorName[0].toUpperCase()
                    : '?'),
              ),
              const SizedBox(width: 8),
              Text(
                authorName,
                style: const TextStyle(fontWeight: FontWeight.bold),
              ),
            ],
          ),
          const SizedBox(height: 8),
          Text(content),
        ],
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **CRUD Operations**: การสร้าง อ่าน อัพเดต ลบข้อมูล
- **Real-time Listeners**: การฟัง Data Changes แบบ Real-time
- **Complex Queries**: where, orderBy, limit, arrayContains
- **Pagination**: การแบ่งหน้าด้วย Document Cursors
- **Batch Writes**: การเขียนหลาย Documents พร้อมกัน
- **Transactions**: การเขียนข้อมูลแบบ Atomic
- **Security Rules**: การกำหนดสิทธิ์การเข้าถึง
- **Offline Persistence**: การรองรับการใช้งาน Offline
- **Subcollections**: การจัดการข้อมูลแบบ Nested

## แบบฝึกหัด

1. สร้าง Blog App ที่มี Posts, Comments และ Likes
2. Implement infinite scroll pagination
3. สร้าง Social Feed ที่แสดงโพสต์จากคนที่ Follow
4. เขียน Security Rules ที่ป้องกัน Spam
5. สร้าง Real-time Chat ด้วย Subcollections

---

[⬅️ Part 23](part_23.md) | [Part 25 ➡️](part_25.md)

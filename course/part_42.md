# Part 42: GraphQL with Flutter
## ขั้นตอนที่ 411-420

---

## สารบัญ
1. [GraphQL และ graphql_flutter Package](#ขั้นตอนที่-411-graphql-และ-graphql_flutter)
2. [Apollo-like Setup](#ขั้นตอนที่-412-apollo-like-setup)
3. [Queries](#ขั้นตอนที่-413-queries)
4. [Mutations](#ขั้นตอนที่-414-mutations)
5. [Subscriptions (Real-time)](#ขั้นตอนที่-415-subscriptions)
6. [Fragments](#ขั้นตอนที่-416-fragments)
7. [Caching & Optimistic Updates](#ขั้นตอนที่-417-caching--optimistic-updates)
8. [Pagination (Cursor-based)](#ขั้นตอนที่-418-pagination)
9. [Error Handling](#ขั้นตอนที่-419-error-handling)
10. [graphql_codegen](#ขั้นตอนที่-420-graphql_codegen)

---

## ขั้นตอนที่ 411: GraphQL และ graphql_flutter Package

### GraphQL คืออะไร?

GraphQL เป็น Query Language สำหรับ API ที่พัฒนาโดย Facebook ช่วยให้ client ระบุได้ว่าต้องการ data อะไรบ้าง แทนที่จะได้รับ data ทั้งหมดจาก REST endpoint

```
REST API:                    GraphQL:
GET /users/123              query {
GET /users/123/posts          user(id: "123") {
GET /users/123/followers        name
                               email
= 3 requests                   posts {
= Over-fetching                  title
                               }
                             }
                           }
                           = 1 request
                           = Exactly what you need
```

### ข้อดีของ GraphQL

1. **No Over-fetching**: ดึงแค่ field ที่ต้องการ
2. **No Under-fetching**: ดึง related data ใน request เดียว
3. **Strongly Typed Schema**: Schema เป็น contract ระหว่าง server/client
4. **Introspection**: สามารถ query schema ได้
5. **Real-time subscriptions**: Built-in support

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  graphql_flutter: ^5.1.2
  gql: ^0.14.0

dev_dependencies:
  graphql_codegen: ^0.14.0
  build_runner: ^2.4.0
```

### โครงสร้างโปรเจค GraphQL

```
lib/
├── graphql/
│   ├── client/
│   │   ├── graphql_client.dart      # Client configuration
│   │   └── graphql_config.dart      # Links configuration
│   ├── queries/
│   │   ├── user_queries.graphql
│   │   └── product_queries.graphql
│   ├── mutations/
│   │   ├── auth_mutations.graphql
│   │   └── product_mutations.graphql
│   ├── subscriptions/
│   │   └── chat_subscriptions.graphql
│   └── fragments/
│       ├── user_fragment.graphql
│       └── product_fragment.graphql
└── features/
    └── ...
```

---

## ขั้นตอนที่ 412: Apollo-like Setup

### GraphQL Client Configuration

```dart
// lib/graphql/client/graphql_client.dart
import 'package:graphql_flutter/graphql_flutter.dart';

class GraphQLClientFactory {
  static GraphQLClient create({
    required String apiUrl,
    required String wsUrl,
    required Future<String?> Function() getAuthToken,
  }) {
    // HTTP Link
    final httpLink = HttpLink(apiUrl);
    
    // Auth Link - เพิ่ม Authorization header
    final authLink = AuthLink(
      getToken: () async {
        final token = await getAuthToken();
        return token != null ? 'Bearer $token' : null;
      },
    );
    
    // WebSocket Link สำหรับ Subscriptions
    final websocketLink = WebSocketLink(
      wsUrl,
      config: SocketClientConfig(
        autoReconnect: true,
        inactivityTimeout: const Duration(seconds: 30),
        initialPayload: () async {
          final token = await getAuthToken();
          return {
            'Authorization': token != null ? 'Bearer $token' : null,
          };
        },
      ),
    );
    
    // Error Link - จัดการ errors
    final errorLink = ErrorLink(
      onGraphQLError: (request, forward, response) {
        for (final error in response.errors ?? []) {
          print('GraphQL Error: ${error.message}');
          
          if (error.extensions?['code'] == 'UNAUTHENTICATED') {
            // Handle auth error - redirect to login
          }
        }
        return null;
      },
      onException: (request, forward, exception) {
        print('Network Exception: $exception');
        return null;
      },
    );
    
    // Retry Link
    final retryLink = RetryLink(
      attempts: (attempt, operation, error) {
        return attempt < 3 && error is NetworkException;
      },
      delays: RetryLink.exponentialBackoff(
        maxDelay: const Duration(seconds: 30),
      ),
    );
    
    // Combine links
    final link = Link.split(
      (request) => request.isSubscription,
      authLink.concat(websocketLink),
      retryLink.concat(errorLink.concat(authLink.concat(httpLink))),
    );
    
    // Cache configuration
    final cache = GraphQLCache(
      store: InMemoryStore(),
      typePolicies: {
        'Query': TypePolicy(
          fields: {
            'products': FieldPolicy(
              keyArgs: ['filter', 'sort'],
              merge: (existing, incoming, {variables}) {
                // Merge pagination results
                if (variables?['after'] != null && existing != null) {
                  final existingList = existing['edges'] as List;
                  final incomingList = incoming['edges'] as List;
                  return {
                    ...incoming,
                    'edges': [...existingList, ...incomingList],
                  };
                }
                return incoming;
              },
            ),
          },
        ),
        'User': TypePolicy(keyFields: {'id'}),
        'Product': TypePolicy(keyFields: {'id'}),
      },
    );
    
    return GraphQLClient(
      link: link,
      cache: cache,
      defaultPolicies: DefaultPolicies(
        query: Policies(
          fetch: FetchPolicy.cacheAndNetwork,
          error: ErrorPolicy.all,
        ),
        mutate: Policies(
          fetch: FetchPolicy.networkOnly,
          error: ErrorPolicy.all,
        ),
        subscribe: Policies(
          fetch: FetchPolicy.networkOnly,
        ),
      ),
    );
  }
}
```

### GraphQL Provider Setup

```dart
// lib/main.dart
import 'package:graphql_flutter/graphql_flutter.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Hive cache
  await initHiveForFlutter();
  
  final client = GraphQLClientFactory.create(
    apiUrl: 'https://api.example.com/graphql',
    wsUrl: 'wss://api.example.com/graphql',
    getAuthToken: () async {
      return await SecureStorage().read('auth_token');
    },
  );
  
  runApp(
    GraphQLProvider(
      client: ValueNotifier(client),
      child: const MyApp(),
    ),
  );
}
```

---

## ขั้นตอนที่ 413: Queries

### GraphQL Query Definition

```graphql
# lib/graphql/queries/user_queries.graphql

fragment UserBasic on User {
  id
  name
  email
  avatarUrl
}

query GetCurrentUser {
  me {
    ...UserBasic
    createdAt
    role
  }
}

query GetUser($id: ID!) {
  user(id: $id) {
    ...UserBasic
    bio
    posts {
      id
      title
      publishedAt
    }
  }
}

query GetProducts(
  $filter: ProductFilter
  $sort: ProductSort
  $first: Int
  $after: String
) {
  products(filter: $filter, sort: $sort, first: $first, after: $after) {
    totalCount
    pageInfo {
      hasNextPage
      endCursor
    }
    edges {
      cursor
      node {
        id
        name
        price
        imageUrl
        category {
          id
          name
        }
      }
    }
  }
}
```

### Query Widget

```dart
// lib/features/products/screens/products_screen.dart
import 'package:graphql_flutter/graphql_flutter.dart';

const String getProductsQuery = r'''
  query GetProducts($filter: ProductFilter, $first: Int, $after: String) {
    products(filter: $filter, first: $first, after: $after) {
      totalCount
      pageInfo {
        hasNextPage
        endCursor
      }
      edges {
        cursor
        node {
          id
          name
          price
          imageUrl
        }
      }
    }
  }
''';

class ProductsScreen extends StatelessWidget {
  const ProductsScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('สินค้า')),
      body: Query(
        options: QueryOptions(
          document: gql(getProductsQuery),
          variables: const {
            'first': 20,
            'filter': {'isActive': true},
          },
          fetchPolicy: FetchPolicy.cacheAndNetwork,
          pollInterval: const Duration(seconds: 30),
        ),
        builder: (result, {fetchMore, refetch}) {
          if (result.isLoading && result.data == null) {
            return const Center(child: CircularProgressIndicator());
          }
          
          if (result.hasException) {
            return _buildError(result.exception!, refetch!);
          }
          
          final data = result.data!;
          final products = data['products']['edges'] as List;
          final pageInfo = data['products']['pageInfo'];
          
          return Column(
            children: [
              Expanded(
                child: RefreshIndicator(
                  onRefresh: () async => refetch!(),
                  child: ListView.builder(
                    itemCount: products.length + 
                        (pageInfo['hasNextPage'] ? 1 : 0),
                    itemBuilder: (context, index) {
                      if (index == products.length) {
                        return _buildLoadMoreButton(
                          fetchMore!,
                          pageInfo['endCursor'],
                        );
                      }
                      
                      final product = products[index]['node'];
                      return ProductCard(product: product);
                    },
                  ),
                ),
              ),
              if (result.isLoading)
                const LinearProgressIndicator(),
            ],
          );
        },
      ),
    );
  }
  
  Widget _buildError(OperationException exception, VoidCallback refetch) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(Icons.error_outline, size: 48, color: Colors.red),
          const SizedBox(height: 16),
          Text('เกิดข้อผิดพลาด: ${exception.graphqlErrors.firstOrNull?.message}'),
          ElevatedButton(
            onPressed: refetch,
            child: const Text('ลองใหม่'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildLoadMoreButton(FetchMore fetchMore, String cursor) {
    return TextButton(
      onPressed: () {
        fetchMore(
          FetchMoreOptions(
            variables: {'after': cursor, 'first': 20},
            updateQuery: (previousResultData, fetchMoreResultData) {
              final existing = List.from(
                previousResultData!['products']['edges'] as List,
              );
              final incoming = List.from(
                fetchMoreResultData!['products']['edges'] as List,
              );
              return {
                'products': {
                  ...fetchMoreResultData['products'],
                  'edges': [...existing, ...incoming],
                },
              };
            },
          ),
        );
      },
      child: const Text('โหลดเพิ่มเติม'),
    );
  }
}
```

### Query ด้วย Hooks (flutter_hooks)

```dart
// lib/features/user/screens/profile_screen.dart
import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:graphql_flutter/graphql_flutter.dart';

const String getCurrentUserQuery = r'''
  query GetCurrentUser {
    me {
      id
      name
      email
      avatarUrl
      bio
    }
  }
''';

class ProfileScreen extends HookWidget {
  const ProfileScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    final result = useQuery(
      QueryOptions(
        document: gql(getCurrentUserQuery),
        fetchPolicy: FetchPolicy.cacheAndNetwork,
      ),
    );
    
    if (result.result.isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    
    if (result.result.hasException) {
      return Center(
        child: Text('Error: ${result.result.exception}'),
      );
    }
    
    final user = result.result.data?['me'];
    if (user == null) return const SizedBox.shrink();
    
    return Scaffold(
      appBar: AppBar(title: const Text('โปรไฟล์')),
      body: Column(
        children: [
          CircleAvatar(
            radius: 50,
            backgroundImage: NetworkImage(user['avatarUrl'] ?? ''),
          ),
          const SizedBox(height: 16),
          Text(
            user['name'],
            style: Theme.of(context).textTheme.headlineMedium,
          ),
          Text(user['email']),
          if (user['bio'] != null) ...[
            const SizedBox(height: 8),
            Text(user['bio']),
          ],
        ],
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 414: Mutations

### GraphQL Mutation Definition

```graphql
# lib/graphql/mutations/auth_mutations.graphql

mutation SignIn($email: String!, $password: String!) {
  signIn(email: $email, password: $password) {
    token
    refreshToken
    user {
      id
      name
      email
      role
    }
  }
}

mutation UpdateProfile($input: UpdateProfileInput!) {
  updateProfile(input: $input) {
    id
    name
    bio
    avatarUrl
  }
}

mutation AddToCart($productId: ID!, $quantity: Int!) {
  addToCart(productId: $productId, quantity: $quantity) {
    id
    items {
      product {
        id
        name
        price
      }
      quantity
    }
    totalAmount
  }
}
```

### Mutation Widget

```dart
// lib/features/auth/screens/login_screen.dart
const String signInMutation = r'''
  mutation SignIn($email: String!, $password: String!) {
    signIn(email: $email, password: $password) {
      token
      user {
        id
        name
        email
      }
    }
  }
''';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});
  
  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _formKey = GlobalKey<FormState>();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Mutation(
        options: MutationOptions(
          document: gql(signInMutation),
          onCompleted: (dynamic resultData) {
            if (resultData != null) {
              final token = resultData['signIn']['token'];
              final user = resultData['signIn']['user'];
              
              // Save token
              context.read<AuthNotifier>().setToken(token, user);
              
              // Navigate
              context.go('/home');
            }
          },
          onError: (error) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(
                content: Text(
                  error?.graphqlErrors.firstOrNull?.message ?? 
                  'เกิดข้อผิดพลาด',
                ),
                backgroundColor: Colors.red,
              ),
            );
          },
        ),
        builder: (runMutation, result) {
          return Form(
            key: _formKey,
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const FlutterLogo(size: 80),
                  const SizedBox(height: 32),
                  TextFormField(
                    controller: _emailController,
                    decoration: const InputDecoration(
                      labelText: 'อีเมล',
                      prefixIcon: Icon(Icons.email),
                    ),
                    keyboardType: TextInputType.emailAddress,
                    validator: (value) {
                      if (value?.isEmpty ?? true) return 'กรุณากรอกอีเมล';
                      if (!value!.contains('@')) return 'รูปแบบอีเมลไม่ถูกต้อง';
                      return null;
                    },
                  ),
                  const SizedBox(height: 16),
                  TextFormField(
                    controller: _passwordController,
                    decoration: const InputDecoration(
                      labelText: 'รหัสผ่าน',
                      prefixIcon: Icon(Icons.lock),
                    ),
                    obscureText: true,
                    validator: (value) {
                      if (value?.isEmpty ?? true) return 'กรุณากรอกรหัสผ่าน';
                      if (value!.length < 6) return 'รหัสผ่านต้องมีอย่างน้อย 6 ตัวอักษร';
                      return null;
                    },
                  ),
                  const SizedBox(height: 24),
                  if (result?.isLoading ?? false)
                    const CircularProgressIndicator()
                  else
                    SizedBox(
                      width: double.infinity,
                      child: ElevatedButton(
                        onPressed: () {
                          if (_formKey.currentState!.validate()) {
                            runMutation({
                              'email': _emailController.text,
                              'password': _passwordController.text,
                            });
                          }
                        },
                        child: const Text('เข้าสู่ระบบ'),
                      ),
                    ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}
```

### Mutation ด้วย GraphQL Client โดยตรง

```dart
// lib/features/cart/repositories/cart_repository.dart
class CartRepository {
  final GraphQLClient _client;
  
  CartRepository(this._client);
  
  static const String _addToCartMutation = r'''
    mutation AddToCart($productId: ID!, $quantity: Int!) {
      addToCart(productId: $productId, quantity: $quantity) {
        id
        totalAmount
        items {
          product { id name price }
          quantity
        }
      }
    }
  ''';
  
  Future<Result<Cart>> addToCart(String productId, int quantity) async {
    try {
      final result = await _client.mutate(
        MutationOptions(
          document: gql(_addToCartMutation),
          variables: {
            'productId': productId,
            'quantity': quantity,
          },
        ),
      );
      
      if (result.hasException) {
        return Failure(AppError.network(
          result.exception?.graphqlErrors.firstOrNull?.message ?? 
          'การเพิ่มสินค้าล้มเหลว',
        ));
      }
      
      final cartData = result.data!['addToCart'];
      return Success(Cart.fromJson(cartData));
    } catch (e, stack) {
      return Failure(AppError.unknown(e, stack));
    }
  }
}
```

---

## ขั้นตอนที่ 415: Subscriptions

### GraphQL Subscription Definition

```graphql
# lib/graphql/subscriptions/chat_subscriptions.graphql

subscription OnMessageReceived($roomId: ID!) {
  messageReceived(roomId: $roomId) {
    id
    content
    createdAt
    sender {
      id
      name
      avatarUrl
    }
  }
}

subscription OnOrderStatusChanged($orderId: ID!) {
  orderStatusChanged(orderId: $orderId) {
    id
    status
    updatedAt
    estimatedDelivery
  }
}

subscription OnUserOnlineStatus($userIds: [ID!]!) {
  userOnlineStatus(userIds: $userIds) {
    userId
    isOnline
    lastSeen
  }
}
```

### Subscription Widget

```dart
// lib/features/chat/screens/chat_room_screen.dart
const String messageSubscription = r'''
  subscription OnMessageReceived($roomId: ID!) {
    messageReceived(roomId: $roomId) {
      id
      content
      createdAt
      sender {
        id
        name
        avatarUrl
      }
    }
  }
''';

class ChatRoomScreen extends StatefulWidget {
  final String roomId;
  
  const ChatRoomScreen({super.key, required this.roomId});
  
  @override
  State<ChatRoomScreen> createState() => _ChatRoomScreenState();
}

class _ChatRoomScreenState extends State<ChatRoomScreen> {
  final List<Message> _messages = [];
  final TextEditingController _messageController = TextEditingController();
  final ScrollController _scrollController = ScrollController();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('แชท')),
      body: Column(
        children: [
          Expanded(
            child: Subscription(
              options: SubscriptionOptions(
                document: gql(messageSubscription),
                variables: {'roomId': widget.roomId},
              ),
              builder: (result) {
                if (result.hasException) {
                  return Center(
                    child: Text('Error: ${result.exception}'),
                  );
                }
                
                if (result.isLoading) {
                  return const Center(child: Text('กำลังเชื่อมต่อ...'));
                }
                
                // เพิ่มข้อความใหม่
                if (result.data != null) {
                  final newMessage = Message.fromJson(
                    result.data!['messageReceived'],
                  );
                  
                  WidgetsBinding.instance.addPostFrameCallback((_) {
                    if (!_messages.any((m) => m.id == newMessage.id)) {
                      setState(() => _messages.add(newMessage));
                      _scrollToBottom();
                    }
                  });
                }
                
                return ListView.builder(
                  controller: _scrollController,
                  itemCount: _messages.length,
                  itemBuilder: (context, index) {
                    final message = _messages[index];
                    return MessageBubble(message: message);
                  },
                );
              },
            ),
          ),
          _buildMessageInput(),
        ],
      ),
    );
  }
  
  Widget _buildMessageInput() {
    return Container(
      padding: const EdgeInsets.all(8),
      child: Row(
        children: [
          Expanded(
            child: TextField(
              controller: _messageController,
              decoration: const InputDecoration(
                hintText: 'พิมพ์ข้อความ...',
                border: OutlineInputBorder(),
              ),
            ),
          ),
          const SizedBox(width: 8),
          IconButton(
            onPressed: _sendMessage,
            icon: const Icon(Icons.send),
          ),
        ],
      ),
    );
  }
  
  void _scrollToBottom() {
    if (_scrollController.hasClients) {
      _scrollController.animateTo(
        _scrollController.position.maxScrollExtent,
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeOut,
      );
    }
  }
  
  Future<void> _sendMessage() async {
    final content = _messageController.text.trim();
    if (content.isEmpty) return;
    
    _messageController.clear();
    
    final client = GraphQLProvider.of(context).value;
    await client.mutate(
      MutationOptions(
        document: gql(r'''
          mutation SendMessage($roomId: ID!, $content: String!) {
            sendMessage(roomId: $roomId, content: $content) {
              id
            }
          }
        '''),
        variables: {
          'roomId': widget.roomId,
          'content': content,
        },
      ),
    );
  }
  
  @override
  void dispose() {
    _messageController.dispose();
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 416: Fragments

### การใช้ Fragments

Fragment ช่วยให้ reuse field selections ได้ ลดการซ้ำซ้อนใน queries

```graphql
# lib/graphql/fragments/user_fragment.graphql

fragment UserBasic on User {
  id
  name
  email
  avatarUrl
}

fragment UserFull on User {
  ...UserBasic
  bio
  createdAt
  role
  stats {
    postsCount
    followersCount
    followingCount
  }
}

fragment PostBasic on Post {
  id
  title
  excerpt
  coverImageUrl
  publishedAt
  author {
    ...UserBasic
  }
}
```

```graphql
# lib/graphql/queries/feed_queries.graphql

#import "./fragments/user_fragment.graphql"
#import "./fragments/post_fragment.graphql"

query GetFeed($first: Int, $after: String) {
  feed(first: $first, after: $after) {
    edges {
      node {
        ...PostBasic
        commentsCount
        likesCount
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Fragments ใน Dart Code

```dart
// lib/graphql/fragments.dart
const String userBasicFragment = r'''
  fragment UserBasic on User {
    id
    name
    email
    avatarUrl
  }
''';

const String productBasicFragment = r'''
  fragment ProductBasic on Product {
    id
    name
    price
    imageUrl
    isAvailable
  }
''';

// ใช้ fragment ใน query
const String getUserQuery = '''
  $userBasicFragment
  
  query GetUser(\$id: ID!) {
    user(id: \$id) {
      ...UserBasic
      bio
      posts {
        id
        title
      }
    }
  }
''';
```

---

## ขั้นตอนที่ 417: Caching & Optimistic Updates

### Cache Configuration

```dart
// lib/graphql/cache_config.dart
GraphQLCache createCache() {
  return GraphQLCache(
    store: HiveStore(),  // Persistent cache with Hive
    typePolicies: {
      'User': TypePolicy(
        keyFields: {'id'},
      ),
      'Product': TypePolicy(
        keyFields: {'id'},
      ),
      'Cart': TypePolicy(
        keyFields: {'id'},
        fields: {
          'items': FieldPolicy(
            merge: (existing, incoming, {variables}) {
              return incoming; // Always use latest
            },
          ),
        },
      ),
    },
  );
}
```

### Optimistic Updates

Optimistic Update คือการ update UI ทันทีก่อนที่ server จะตอบกลับ ทำให้ UX ดูลื่นไหล

```dart
// lib/features/social/repositories/post_repository.dart
class PostRepository {
  final GraphQLClient _client;
  
  PostRepository(this._client);
  
  static const String _likePostMutation = r'''
    mutation LikePost($postId: ID!) {
      likePost(postId: $postId) {
        id
        isLiked
        likesCount
      }
    }
  ''';
  
  Future<void> likePost(String postId, bool currentlyLiked) async {
    await _client.mutate(
      MutationOptions(
        document: gql(_likePostMutation),
        variables: {'postId': postId},
        
        // Optimistic result - update UI ทันที
        optimisticResult: {
          'likePost': {
            '__typename': 'Post',
            'id': postId,
            'isLiked': !currentlyLiked,
            'likesCount': currentlyLiked ? -1 : 1, // relative change
          },
        },
        
        update: (cache, result) {
          if (result?.hasException ?? true) return;
          
          // Update cache หลังได้รับ response จาก server
          final postFragment = cache.readFragment(
            FragmentRequest(
              fragment: gql('fragment PostLike on Post { id isLiked likesCount }'),
              idFields: {'__typename': 'Post', 'id': postId},
            ),
          );
          
          if (postFragment != null) {
            cache.writeFragment(
              FragmentRequest(
                fragment: gql('fragment PostLike on Post { id isLiked likesCount }'),
                idFields: {'__typename': 'Post', 'id': postId},
              ),
              data: result!.data!['likePost'],
            );
          }
        },
      ),
    );
  }
}
```

### Cache Reading and Writing

```dart
// การ read/write cache โดยตรง
void updateUserInCache(GraphQLClient client, User user) {
  client.writeFragment(
    FragmentRequest(
      fragment: gql(r'''
        fragment UpdateUser on User {
          id
          name
          email
          avatarUrl
        }
      '''),
      idFields: {'__typename': 'User', 'id': user.id},
    ),
    data: {
      'id': user.id,
      'name': user.name,
      'email': user.email,
      'avatarUrl': user.avatarUrl,
    },
  );
}
```

---

## ขั้นตอนที่ 418: Pagination (Cursor-based)

### Cursor-based Pagination

Cursor-based pagination เหมาะสำหรับ real-time data ที่มีการเพิ่ม/ลบ items บ่อย

```dart
// lib/features/products/notifiers/products_notifier.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'products_notifier.g.dart';

@riverpod
class ProductsNotifier extends _$ProductsNotifier {
  static const int _pageSize = 20;
  String? _endCursor;
  bool _hasNextPage = true;
  bool _isLoadingMore = false;
  
  @override
  Future<List<Product>> build() async {
    return _fetchProducts();
  }
  
  Future<List<Product>> _fetchProducts({String? after}) async {
    final client = ref.read(graphQLClientProvider);
    
    final result = await client.query(
      QueryOptions(
        document: gql(getProductsQuery),
        variables: {
          'first': _pageSize,
          if (after != null) 'after': after,
        },
      ),
    );
    
    if (result.hasException) {
      throw result.exception!;
    }
    
    final productsData = result.data!['products'];
    final pageInfo = productsData['pageInfo'];
    
    _endCursor = pageInfo['endCursor'];
    _hasNextPage = pageInfo['hasNextPage'];
    
    return (productsData['edges'] as List)
        .map((e) => Product.fromJson(e['node']))
        .toList();
  }
  
  Future<void> loadMore() async {
    if (!_hasNextPage || _isLoadingMore) return;
    
    _isLoadingMore = true;
    
    try {
      final currentProducts = await future;
      final moreProducts = await _fetchProducts(after: _endCursor);
      
      state = AsyncData([...currentProducts, ...moreProducts]);
    } finally {
      _isLoadingMore = false;
    }
  }
  
  Future<void> refresh() async {
    _endCursor = null;
    _hasNextPage = true;
    ref.invalidateSelf();
  }
  
  bool get hasNextPage => _hasNextPage;
  bool get isLoadingMore => _isLoadingMore;
}
```

### Pagination Widget

```dart
// lib/features/products/widgets/paginated_products_list.dart
class PaginatedProductsList extends ConsumerStatefulWidget {
  const PaginatedProductsList({super.key});
  
  @override
  ConsumerState<PaginatedProductsList> createState() =>
      _PaginatedProductsListState();
}

class _PaginatedProductsListState
    extends ConsumerState<PaginatedProductsList> {
  final ScrollController _scrollController = ScrollController();
  
  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels >=
        _scrollController.position.maxScrollExtent * 0.8) {
      ref.read(productsNotifierProvider.notifier).loadMore();
    }
  }
  
  @override
  Widget build(BuildContext context) {
    final productsAsync = ref.watch(productsNotifierProvider);
    final notifier = ref.watch(productsNotifierProvider.notifier);
    
    return productsAsync.when(
      data: (products) {
        return RefreshIndicator(
          onRefresh: notifier.refresh,
          child: ListView.builder(
            controller: _scrollController,
            itemCount: products.length + (notifier.hasNextPage ? 1 : 0),
            itemBuilder: (context, index) {
              if (index == products.length) {
                return const Center(
                  child: Padding(
                    padding: EdgeInsets.all(16),
                    child: CircularProgressIndicator(),
                  ),
                );
              }
              return ProductCard(product: products[index]);
            },
          ),
        );
      },
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Error: $error'),
            ElevatedButton(
              onPressed: notifier.refresh,
              child: const Text('ลองใหม่'),
            ),
          ],
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## ขั้นตอนที่ 419: Error Handling

### GraphQL Error Types

```
GraphQL Errors:
├── Network Errors    - ไม่สามารถเชื่อมต่อ server ได้
├── GraphQL Errors    - Query/Mutation errors จาก server
│   ├── Validation Errors   - Query ไม่ถูก schema
│   ├── Resolver Errors     - Business logic errors
│   └── Auth Errors         - Unauthorized/Forbidden
└── Client Errors     - Client-side errors
```

### Error Handling Implementation

```dart
// lib/graphql/error_handler.dart
class GraphQLErrorHandler {
  static AppError handleException(OperationException exception) {
    // Network error
    if (exception.linkException != null) {
      final linkError = exception.linkException!;
      
      if (linkError is NetworkException) {
        return AppError.network('ไม่สามารถเชื่อมต่อ Internet ได้');
      }
      
      if (linkError is ServerException) {
        return AppError(
          code: 'SERVER_ERROR',
          message: 'Server เกิดข้อผิดพลาด: ${linkError.parsedResponse?.errors?.first.message}',
        );
      }
    }
    
    // GraphQL errors
    if (exception.graphqlErrors.isNotEmpty) {
      final firstError = exception.graphqlErrors.first;
      final code = firstError.extensions?['code'] as String?;
      
      return switch (code) {
        'UNAUTHENTICATED' => AppError.unauthorized(),
        'FORBIDDEN' => const AppError(
          code: 'FORBIDDEN',
          message: 'ไม่มีสิทธิ์ดำเนินการนี้',
        ),
        'NOT_FOUND' => AppError.notFound('ข้อมูลที่ต้องการ'),
        'VALIDATION_ERROR' => AppError(
          code: 'VALIDATION',
          message: firstError.message,
        ),
        _ => AppError(
          code: code ?? 'GRAPHQL_ERROR',
          message: firstError.message,
        ),
      };
    }
    
    return AppError.unknown(exception);
  }
}

// Extension สำหรับ QueryResult
extension QueryResultX on QueryResult {
  Result<T> toResult<T>(T Function(Map<String, dynamic> data) fromData) {
    if (hasException) {
      return Failure(GraphQLErrorHandler.handleException(exception!));
    }
    
    if (data == null) {
      return Failure(AppError(code: 'NULL_DATA', message: 'ไม่ได้รับข้อมูล'));
    }
    
    try {
      return Success(fromData(data!));
    } catch (e, stack) {
      return Failure(AppError.unknown(e, stack));
    }
  }
}
```

---

## ขั้นตอนที่ 420: graphql_codegen

### graphql_codegen คืออะไร?

graphql_codegen สร้าง type-safe Dart classes จาก GraphQL schema และ operations อัตโนมัติ

### การตั้งค่า

```yaml
# pubspec.yaml
dev_dependencies:
  graphql_codegen: ^0.14.0
  build_runner: ^2.4.0
  json_serializable: ^6.7.0
```

```yaml
# build.yaml
targets:
  $default:
    builders:
      graphql_codegen:
        options:
          schema:
            - lib/graphql/schema.graphql
          documents:
            - lib/graphql/**/*.graphql
          plugins:
            - graphql_codegen_config: true
```

### Schema Definition

```graphql
# lib/graphql/schema.graphql
type Query {
  user(id: ID!): User
  me: User
  products(filter: ProductFilter, first: Int, after: String): ProductConnection!
}

type Mutation {
  signIn(email: String!, password: String!): AuthPayload!
  updateProfile(input: UpdateProfileInput!): User!
}

type Subscription {
  messageReceived(roomId: ID!): Message!
}

type User {
  id: ID!
  name: String!
  email: String!
  avatarUrl: String
  bio: String
}

type Product {
  id: ID!
  name: String!
  price: Float!
  imageUrl: String
}
```

### Generated Code Usage

```dart
// หลังจาก run: dart run build_runner build
// graphql_codegen จะสร้าง:

// user_queries.graphql.dart (auto-generated)
class GetUserQuery {
  static const document = r'''
    query GetUser($id: ID!) { ... }
  ''';
  
  static QueryOptions options({required String id}) {
    return QueryOptions(
      document: gql(document),
      variables: {'id': id},
    );
  }
}

class GetUserQueryResult {
  final GetUserQueryUser? user;
  
  GetUserQueryResult.fromJson(Map<String, dynamic> json)
      : user = json['user'] != null
            ? GetUserQueryUser.fromJson(json['user'])
            : null;
}

class GetUserQueryUser {
  final String id;
  final String name;
  final String email;
  final String? avatarUrl;
  
  GetUserQueryUser.fromJson(Map<String, dynamic> json)
      : id = json['id'] as String,
        name = json['name'] as String,
        email = json['email'] as String,
        avatarUrl = json['avatarUrl'] as String?;
}

// การใช้งาน - type-safe!
final result = await client.query(GetUserQuery.options(id: '123'));
if (!result.hasException) {
  final userData = GetUserQueryResult.fromJson(result.data!);
  print(userData.user?.name); // type-safe access
}
```

### Workshop: สร้าง Blog App ด้วย GraphQL

```dart
// lib/features/blog/screens/blog_screen.dart
class BlogScreen extends ConsumerWidget {
  const BlogScreen({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('บล็อก'),
          bottom: const TabBar(
            tabs: [
              Tab(text: 'ล่าสุด'),
              Tab(text: 'ยอดนิยม'),
              Tab(text: 'ติดตาม'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            PostsList(sort: PostSort.latest),
            PostsList(sort: PostSort.popular),
            PostsList(sort: PostSort.following),
          ],
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => context.push('/blog/create'),
          child: const Icon(Icons.add),
        ),
      ),
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

- **GraphQL Basics**: Query Language ที่ยืดหยุ่นกว่า REST
- **graphql_flutter**: Package หลักสำหรับ Flutter
- **Apollo-like Setup**: การตั้งค่า Links, Cache, Policies
- **Queries**: การดึงข้อมูลแบบ declarative
- **Mutations**: การแก้ไขข้อมูล
- **Subscriptions**: Real-time updates ผ่าน WebSocket
- **Fragments**: การ reuse field selections
- **Optimistic Updates**: UI ที่ตอบสนองเร็ว
- **Pagination**: Cursor-based pagination
- **graphql_codegen**: Type-safe code generation

## แบบฝึกหัด

1. ตั้งค่า GraphQL client เชื่อมกับ [GitHub GraphQL API](https://api.github.com/graphql)
2. สร้าง screen แสดง repository list ด้วย Query
3. Implement star/unstar repository ด้วย Mutation + Optimistic Update
4. เพิ่ม Subscription สำหรับ live notifications
5. ตั้งค่า graphql_codegen และ generate type-safe classes

---

[⬅️ Part 41](part_41.md) | [Part 43 ➡️](part_43.md)

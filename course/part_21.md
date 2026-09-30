# Part 21: Bloc Pattern
## ขั้นตอนที่ 201-210

---

## สารบัญ
1. [flutter_bloc Package](#flutter_bloc-package)
2. [Cubit vs Bloc](#cubit-vs-bloc)
3. [Events และ States](#events-และ-states)
4. [BlocProvider](#blocprovider)
5. [BlocBuilder](#blocbuilder)
6. [BlocListener](#bloclistener)
7. [BlocConsumer](#blocconsumer)
8. [MultiBlocProvider](#multiblocprovider)
9. [Bloc-to-Bloc Communication](#bloc-to-bloc-communication)
10. [Testing Bloc](#testing-bloc)

---

## ขั้นตอนที่ 201: flutter_bloc Package

Bloc (Business Logic Component) เป็น Design Pattern ที่ช่วยแยก Business Logic ออกจาก UI ทำให้โค้ดอ่านง่าย ทดสอบได้ และบำรุงรักษาได้ดี

### การติดตั้ง

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5

dev_dependencies:
  bloc_test: ^9.1.4
  mocktail: ^1.0.1
```

### แนวคิดหลักของ Bloc Pattern

```
UI (Widget)
    │
    │ Events (การกระทำจากผู้ใช้)
    ▼
  BLoC
    │
    │ States (สถานะใหม่)
    ▼
UI (Widget) ← อัพเดต UI ตาม State ใหม่
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Bloc Pattern Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const CounterPage(),
    );
  }
}
```

---

## ขั้นตอนที่ 202: Cubit vs Bloc

### Cubit - รูปแบบที่ง่ายกว่า

Cubit เป็น Subset ของ Bloc ที่ไม่ต้องการ Events เหมาะสำหรับ Logic ที่ไม่ซับซ้อน

```dart
// lib/cubit/counter_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0); // initial state = 0

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
  void reset() => emit(0);
}
```

```dart
// lib/pages/counter_cubit_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../cubit/counter_cubit.dart';

class CounterCubitPage extends StatelessWidget {
  const CounterCubitPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterCubit(),
      child: const CounterCubitView(),
    );
  }
}

class CounterCubitView extends StatelessWidget {
  const CounterCubitView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter with Cubit')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Count:', style: TextStyle(fontSize: 24)),
            BlocBuilder<CounterCubit, int>(
              builder: (context, state) {
                return Text(
                  '$state',
                  style: const TextStyle(fontSize: 64, fontWeight: FontWeight.bold),
                );
              },
            ),
            const SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                FloatingActionButton(
                  heroTag: 'decrement',
                  onPressed: () => context.read<CounterCubit>().decrement(),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 16),
                FloatingActionButton(
                  heroTag: 'reset',
                  onPressed: () => context.read<CounterCubit>().reset(),
                  child: const Icon(Icons.refresh),
                ),
                const SizedBox(width: 16),
                FloatingActionButton(
                  heroTag: 'increment',
                  onPressed: () => context.read<CounterCubit>().increment(),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### Bloc - รูปแบบที่สมบูรณ์

```dart
// lib/bloc/counter_event.dart
import 'package:equatable/equatable.dart';

abstract class CounterEvent extends Equatable {
  const CounterEvent();

  @override
  List<Object> get props => [];
}

class CounterIncrementPressed extends CounterEvent {
  const CounterIncrementPressed();
}

class CounterDecrementPressed extends CounterEvent {
  const CounterDecrementPressed();
}

class CounterResetPressed extends CounterEvent {
  const CounterResetPressed();
}

class CounterIncrementByAmount extends CounterEvent {
  const CounterIncrementByAmount(this.amount);
  final int amount;

  @override
  List<Object> get props => [amount];
}
```

```dart
// lib/bloc/counter_state.dart
import 'package:equatable/equatable.dart';

class CounterState extends Equatable {
  const CounterState({required this.count, this.isLoading = false});

  final int count;
  final bool isLoading;

  CounterState copyWith({int? count, bool? isLoading}) {
    return CounterState(
      count: count ?? this.count,
      isLoading: isLoading ?? this.isLoading,
    );
  }

  @override
  List<Object> get props => [count, isLoading];
}
```

```dart
// lib/bloc/counter_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'counter_event.dart';
import 'counter_state.dart';

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState(count: 0)) {
    on<CounterIncrementPressed>(_onIncrementPressed);
    on<CounterDecrementPressed>(_onDecrementPressed);
    on<CounterResetPressed>(_onResetPressed);
    on<CounterIncrementByAmount>(_onIncrementByAmount);
  }

  void _onIncrementPressed(
    CounterIncrementPressed event,
    Emitter<CounterState> emit,
  ) {
    emit(state.copyWith(count: state.count + 1));
  }

  void _onDecrementPressed(
    CounterDecrementPressed event,
    Emitter<CounterState> emit,
  ) {
    emit(state.copyWith(count: state.count - 1));
  }

  void _onResetPressed(
    CounterResetPressed event,
    Emitter<CounterState> emit,
  ) {
    emit(state.copyWith(count: 0));
  }

  Future<void> _onIncrementByAmount(
    CounterIncrementByAmount event,
    Emitter<CounterState> emit,
  ) async {
    emit(state.copyWith(isLoading: true));
    await Future.delayed(const Duration(milliseconds: 500));
    emit(state.copyWith(count: state.count + event.amount, isLoading: false));
  }
}
```

---

## ขั้นตอนที่ 203: Events และ States

### การออกแบบ States ที่ดี

```dart
// lib/bloc/todo_state.dart
import 'package:equatable/equatable.dart';

// แบบ Sealed Class (แนะนำ)
abstract class TodoState extends Equatable {
  const TodoState();

  @override
  List<Object?> get props => [];
}

class TodoInitial extends TodoState {
  const TodoInitial();
}

class TodoLoading extends TodoState {
  const TodoLoading();
}

class TodoLoaded extends TodoState {
  const TodoLoaded(this.todos);
  final List<Todo> todos;

  @override
  List<Object?> get props => [todos];
}

class TodoError extends TodoState {
  const TodoError(this.message);
  final String message;

  @override
  List<Object?> get props => [message];
}

// Model
class Todo extends Equatable {
  const Todo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  final String id;
  final String title;
  final bool isCompleted;

  Todo copyWith({String? id, String? title, bool? isCompleted}) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }

  @override
  List<Object> get props => [id, title, isCompleted];
}
```

```dart
// lib/bloc/todo_event.dart
import 'package:equatable/equatable.dart';

abstract class TodoEvent extends Equatable {
  const TodoEvent();

  @override
  List<Object?> get props => [];
}

class TodoLoadRequested extends TodoEvent {
  const TodoLoadRequested();
}

class TodoAddRequested extends TodoEvent {
  const TodoAddRequested(this.title);
  final String title;

  @override
  List<Object?> get props => [title];
}

class TodoToggleRequested extends TodoEvent {
  const TodoToggleRequested(this.id);
  final String id;

  @override
  List<Object?> get props => [id];
}

class TodoDeleteRequested extends TodoEvent {
  const TodoDeleteRequested(this.id);
  final String id;

  @override
  List<Object?> get props => [id];
}
```

```dart
// lib/bloc/todo_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:uuid/uuid.dart';
import 'todo_event.dart';
import 'todo_state.dart';

class TodoBloc extends Bloc<TodoEvent, TodoState> {
  TodoBloc() : super(const TodoInitial()) {
    on<TodoLoadRequested>(_onLoadRequested);
    on<TodoAddRequested>(_onAddRequested);
    on<TodoToggleRequested>(_onToggleRequested);
    on<TodoDeleteRequested>(_onDeleteRequested);
  }

  final _uuid = const Uuid();

  Future<void> _onLoadRequested(
    TodoLoadRequested event,
    Emitter<TodoState> emit,
  ) async {
    emit(const TodoLoading());
    try {
      await Future.delayed(const Duration(seconds: 1)); // จำลองการโหลด
      final todos = [
        Todo(id: _uuid.v4(), title: 'เรียน Flutter Bloc'),
        Todo(id: _uuid.v4(), title: 'สร้างแอป Todo'),
        Todo(id: _uuid.v4(), title: 'ทดสอบ Bloc'),
      ];
      emit(TodoLoaded(todos));
    } catch (e) {
      emit(TodoError('เกิดข้อผิดพลาด: $e'));
    }
  }

  void _onAddRequested(
    TodoAddRequested event,
    Emitter<TodoState> emit,
  ) {
    if (state is TodoLoaded) {
      final currentTodos = List<Todo>.from((state as TodoLoaded).todos);
      currentTodos.add(Todo(id: _uuid.v4(), title: event.title));
      emit(TodoLoaded(currentTodos));
    }
  }

  void _onToggleRequested(
    TodoToggleRequested event,
    Emitter<TodoState> emit,
  ) {
    if (state is TodoLoaded) {
      final todos = (state as TodoLoaded).todos.map((todo) {
        if (todo.id == event.id) {
          return todo.copyWith(isCompleted: !todo.isCompleted);
        }
        return todo;
      }).toList();
      emit(TodoLoaded(todos));
    }
  }

  void _onDeleteRequested(
    TodoDeleteRequested event,
    Emitter<TodoState> emit,
  ) {
    if (state is TodoLoaded) {
      final todos = (state as TodoLoaded)
          .todos
          .where((todo) => todo.id != event.id)
          .toList();
      emit(TodoLoaded(todos));
    }
  }
}
```

---

## ขั้นตอนที่ 204: BlocProvider

BlocProvider คือ Widget ที่ใช้ inject Bloc เข้าไปใน Widget Tree

```dart
// lib/pages/todo_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/todo_bloc.dart';
import '../bloc/todo_event.dart';
import '../bloc/todo_state.dart';

class TodoPage extends StatelessWidget {
  const TodoPage({super.key});

  @override
  Widget build(BuildContext context) {
    // BlocProvider สร้างและ inject TodoBloc
    return BlocProvider(
      create: (context) => TodoBloc()..add(const TodoLoadRequested()),
      child: const TodoView(),
    );
  }
}

// ใช้ BlocProvider.value เมื่อ Bloc มีอยู่แล้ว
class TodoPageWithExistingBloc extends StatelessWidget {
  const TodoPageWithExistingBloc({super.key, required this.bloc});
  final TodoBloc bloc;

  @override
  Widget build(BuildContext context) {
    return BlocProvider.value(
      value: bloc,
      child: const TodoView(),
    );
  }
}

class TodoView extends StatelessWidget {
  const TodoView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Todo List'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: BlocBuilder<TodoBloc, TodoState>(
        builder: (context, state) {
          if (state is TodoInitial) {
            return const Center(child: Text('กด + เพื่อเพิ่ม Todo'));
          }
          if (state is TodoLoading) {
            return const Center(child: CircularProgressIndicator());
          }
          if (state is TodoError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text(state.message, style: const TextStyle(color: Colors.red)),
                  ElevatedButton(
                    onPressed: () => context.read<TodoBloc>().add(const TodoLoadRequested()),
                    child: const Text('ลองใหม่'),
                  ),
                ],
              ),
            );
          }
          if (state is TodoLoaded) {
            return _TodoList(todos: state.todos);
          }
          return const SizedBox.shrink();
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddDialog(BuildContext context) {
    final controller = TextEditingController();
    showDialog(
      context: context,
      builder: (dialogContext) => AlertDialog(
        title: const Text('เพิ่ม Todo'),
        content: TextField(
          controller: controller,
          decoration: const InputDecoration(hintText: 'ชื่อ Todo'),
          autofocus: true,
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(dialogContext),
            child: const Text('ยกเลิก'),
          ),
          ElevatedButton(
            onPressed: () {
              if (controller.text.isNotEmpty) {
                context.read<TodoBloc>().add(TodoAddRequested(controller.text));
                Navigator.pop(dialogContext);
              }
            },
            child: const Text('เพิ่ม'),
          ),
        ],
      ),
    );
  }
}

class _TodoList extends StatelessWidget {
  const _TodoList({required this.todos});
  final List<Todo> todos;

  @override
  Widget build(BuildContext context) {
    if (todos.isEmpty) {
      return const Center(child: Text('ไม่มี Todo'));
    }
    return ListView.builder(
      itemCount: todos.length,
      itemBuilder: (context, index) {
        final todo = todos[index];
        return ListTile(
          leading: Checkbox(
            value: todo.isCompleted,
            onChanged: (_) =>
                context.read<TodoBloc>().add(TodoToggleRequested(todo.id)),
          ),
          title: Text(
            todo.title,
            style: TextStyle(
              decoration: todo.isCompleted ? TextDecoration.lineThrough : null,
            ),
          ),
          trailing: IconButton(
            icon: const Icon(Icons.delete, color: Colors.red),
            onPressed: () =>
                context.read<TodoBloc>().add(TodoDeleteRequested(todo.id)),
          ),
        );
      },
    );
  }
}
```

---

## ขั้นตอนที่ 205: BlocBuilder

BlocBuilder ใช้สำหรับ rebuild UI เมื่อ State เปลี่ยน

```dart
// lib/widgets/bloc_builder_examples.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/counter_bloc.dart';
import '../bloc/counter_state.dart';
import '../bloc/counter_event.dart';

class BlocBuilderExamplesPage extends StatelessWidget {
  const BlocBuilderExamplesPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => CounterBloc(),
      child: const BlocBuilderExamplesView(),
    );
  }
}

class BlocBuilderExamplesView extends StatelessWidget {
  const BlocBuilderExamplesView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('BlocBuilder Examples')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // 1. BlocBuilder พื้นฐาน
            BlocBuilder<CounterBloc, CounterState>(
              builder: (context, state) {
                return Text(
                  'Count: ${state.count}',
                  style: const TextStyle(fontSize: 32),
                );
              },
            ),

            const SizedBox(height: 16),

            // 2. BlocBuilder กับ buildWhen - rebuild เฉพาะเมื่อเงื่อนไขเป็นจริง
            BlocBuilder<CounterBloc, CounterState>(
              buildWhen: (previous, current) {
                // Rebuild เฉพาะเมื่อ count เปลี่ยนแปลง (ไม่ rebuild เมื่อ isLoading เปลี่ยน)
                return previous.count != current.count;
              },
              builder: (context, state) {
                return Container(
                  padding: const EdgeInsets.all(8),
                  color: state.count.isEven ? Colors.green[100] : Colors.red[100],
                  child: Text(
                    state.count.isEven ? 'เลขคู่' : 'เลขคี่',
                    style: const TextStyle(fontSize: 20),
                  ),
                );
              },
            ),

            const SizedBox(height: 16),

            // 3. แสดง Loading indicator
            BlocBuilder<CounterBloc, CounterState>(
              buildWhen: (previous, current) =>
                  previous.isLoading != current.isLoading,
              builder: (context, state) {
                if (state.isLoading) {
                  return const CircularProgressIndicator();
                }
                return const SizedBox.shrink();
              },
            ),

            const SizedBox(height: 24),

            // ปุ่มต่างๆ
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton.icon(
                  onPressed: () => context
                      .read<CounterBloc>()
                      .add(const CounterDecrementPressed()),
                  icon: const Icon(Icons.remove),
                  label: const Text('-1'),
                ),
                ElevatedButton.icon(
                  onPressed: () => context
                      .read<CounterBloc>()
                      .add(const CounterIncrementPressed()),
                  icon: const Icon(Icons.add),
                  label: const Text('+1'),
                ),
                ElevatedButton.icon(
                  onPressed: () => context
                      .read<CounterBloc>()
                      .add(const CounterIncrementByAmount(10)),
                  icon: const Icon(Icons.add_circle),
                  label: const Text('+10'),
                ),
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

## ขั้นตอนที่ 206: BlocListener

BlocListener ใช้สำหรับ side effects เช่น แสดง SnackBar, Navigation ไม่ rebuild UI

```dart
// lib/widgets/bloc_listener_examples.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/todo_bloc.dart';
import '../bloc/todo_event.dart';
import '../bloc/todo_state.dart';

class BlocListenerExamplesPage extends StatelessWidget {
  const BlocListenerExamplesPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => TodoBloc()..add(const TodoLoadRequested()),
      child: const BlocListenerExamplesView(),
    );
  }
}

class BlocListenerExamplesView extends StatelessWidget {
  const BlocListenerExamplesView({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocListener<TodoBloc, TodoState>(
      listenWhen: (previous, current) {
        // Listen เฉพาะเมื่อ State เปลี่ยนจาก Loading เป็น Loaded
        return previous is TodoLoading && current is TodoLoaded;
      },
      listener: (context, state) {
        if (state is TodoLoaded) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('โหลด ${state.todos.length} รายการเรียบร้อย'),
              backgroundColor: Colors.green,
            ),
          );
        }
      },
      child: Scaffold(
        appBar: AppBar(title: const Text('BlocListener Examples')),
        body: BlocBuilder<TodoBloc, TodoState>(
          builder: (context, state) {
            if (state is TodoLoading) {
              return const Center(child: CircularProgressIndicator());
            }
            if (state is TodoLoaded) {
              return ListView.builder(
                itemCount: state.todos.length,
                itemBuilder: (context, index) {
                  final todo = state.todos[index];
                  return ListTile(title: Text(todo.title));
                },
              );
            }
            if (state is TodoError) {
              return Center(child: Text(state.message));
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }
}

// ตัวอย่าง: Navigation จาก BlocListener
class AuthPage extends StatelessWidget {
  const AuthPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocListener<AuthBloc, AuthState>(
      listener: (context, state) {
        if (state is AuthAuthenticated) {
          // Navigate เมื่อ Login สำเร็จ
          Navigator.of(context).pushReplacement(
            MaterialPageRoute(builder: (_) => const HomePage()),
          );
        } else if (state is AuthError) {
          // แสดง Dialog เมื่อเกิดข้อผิดพลาด
          showDialog(
            context: context,
            builder: (_) => AlertDialog(
              title: const Text('ข้อผิดพลาด'),
              content: Text(state.message),
              actions: [
                TextButton(
                  onPressed: () => Navigator.pop(context),
                  child: const Text('ตกลง'),
                ),
              ],
            ),
          );
        }
      },
      child: const AuthForm(),
    );
  }
}
```

---

## ขั้นตอนที่ 207: BlocConsumer

BlocConsumer รวม BlocBuilder และ BlocListener เข้าด้วยกัน

```dart
// lib/widgets/bloc_consumer_example.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// Auth Bloc
abstract class AuthEvent {}
class LoginRequested extends AuthEvent {
  LoginRequested({required this.email, required this.password});
  final String email;
  final String password;
}
class LogoutRequested extends AuthEvent {}

abstract class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState {
  AuthAuthenticated(this.username);
  final String username;
}
class AuthError extends AuthState {
  AuthError(this.message);
  final String message;
}

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginRequested>(_onLoginRequested);
    on<LogoutRequested>(_onLogoutRequested);
  }

  Future<void> _onLoginRequested(
    LoginRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    await Future.delayed(const Duration(seconds: 2));
    if (event.email == 'test@test.com' && event.password == '123456') {
      emit(AuthAuthenticated('Test User'));
    } else {
      emit(AuthError('Email หรือรหัสผ่านไม่ถูกต้อง'));
    }
  }

  Future<void> _onLogoutRequested(
    LogoutRequested event,
    Emitter<AuthState> emit,
  ) async {
    emit(AuthLoading());
    await Future.delayed(const Duration(milliseconds: 500));
    emit(AuthInitial());
  }
}

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => AuthBloc(),
      child: const LoginView(),
    );
  }
}

class LoginView extends StatefulWidget {
  const LoginView({super.key});

  @override
  State<LoginView> createState() => _LoginViewState();
}

class _LoginViewState extends State<LoginView> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('เข้าสู่ระบบ')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: BlocConsumer<AuthBloc, AuthState>(
          // listener สำหรับ side effects
          listener: (context, state) {
            if (state is AuthAuthenticated) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('ยินดีต้อนรับ ${state.username}!'),
                  backgroundColor: Colors.green,
                ),
              );
            }
            if (state is AuthError) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text(state.message),
                  backgroundColor: Colors.red,
                ),
              );
            }
          },
          // builder สำหรับ UI
          builder: (context, state) {
            if (state is AuthAuthenticated) {
              return _buildAuthenticatedView(context, state);
            }

            return Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                TextField(
                  controller: _emailController,
                  decoration: const InputDecoration(
                    labelText: 'Email',
                    border: OutlineInputBorder(),
                  ),
                  keyboardType: TextInputType.emailAddress,
                ),
                const SizedBox(height: 16),
                TextField(
                  controller: _passwordController,
                  decoration: const InputDecoration(
                    labelText: 'รหัสผ่าน',
                    border: OutlineInputBorder(),
                  ),
                  obscureText: true,
                ),
                const SizedBox(height: 24),
                if (state is AuthLoading)
                  const CircularProgressIndicator()
                else
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () {
                        context.read<AuthBloc>().add(
                          LoginRequested(
                            email: _emailController.text,
                            password: _passwordController.text,
                          ),
                        );
                      },
                      child: const Text('เข้าสู่ระบบ'),
                    ),
                  ),
              ],
            );
          },
        ),
      ),
    );
  }

  Widget _buildAuthenticatedView(BuildContext context, AuthAuthenticated state) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        const Icon(Icons.check_circle, color: Colors.green, size: 80),
        const SizedBox(height: 16),
        Text(
          'สวัสดี ${state.username}',
          style: const TextStyle(fontSize: 24),
        ),
        const SizedBox(height: 16),
        ElevatedButton(
          onPressed: () =>
              context.read<AuthBloc>().add(LogoutRequested()),
          child: const Text('ออกจากระบบ'),
        ),
      ],
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('หน้าหลัก')),
      body: const Center(child: Text('ยินดีต้อนรับ!')),
    );
  }
}

class AuthForm extends StatelessWidget {
  const AuthForm({super.key});

  @override
  Widget build(BuildContext context) {
    return const Center(child: Text('Auth Form'));
  }
}
```

---

## ขั้นตอนที่ 208: MultiBlocProvider

```dart
// lib/pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// สมมติว่ามี Bloc หลายตัว
class ThemeCubit extends Cubit<ThemeMode> {
  ThemeCubit() : super(ThemeMode.light);

  void toggleTheme() {
    emit(state == ThemeMode.light ? ThemeMode.dark : ThemeMode.light);
  }
}

class LanguageCubit extends Cubit<String> {
  LanguageCubit() : super('th');

  void changeLanguage(String lang) => emit(lang);
}

class UserCubit extends Cubit<UserState> {
  UserCubit() : super(const UserState());

  void updateName(String name) => emit(state.copyWith(name: name));
  void updateEmail(String email) => emit(state.copyWith(email: email));
}

class UserState {
  const UserState({this.name = '', this.email = ''});
  final String name;
  final String email;

  UserState copyWith({String? name, String? email}) {
    return UserState(name: name ?? this.name, email: email ?? this.email);
  }
}

class AppRoot extends StatelessWidget {
  const AppRoot({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider(create: (_) => ThemeCubit()),
        BlocProvider(create: (_) => LanguageCubit()),
        BlocProvider(create: (_) => UserCubit()),
        BlocProvider(
          create: (context) => TodoBloc()..add(const TodoLoadRequested()),
        ),
        BlocProvider(create: (_) => AuthBloc()),
      ],
      child: BlocBuilder<ThemeCubit, ThemeMode>(
        builder: (context, themeMode) {
          return MaterialApp(
            themeMode: themeMode,
            theme: ThemeData.light(useMaterial3: true),
            darkTheme: ThemeData.dark(useMaterial3: true),
            home: const MainPage(),
          );
        },
      ),
    );
  }
}

class MainPage extends StatelessWidget {
  const MainPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('MultiBlocProvider Demo'),
        actions: [
          BlocBuilder<ThemeCubit, ThemeMode>(
            builder: (context, mode) {
              return IconButton(
                icon: Icon(mode == ThemeMode.light
                    ? Icons.dark_mode
                    : Icons.light_mode),
                onPressed: () => context.read<ThemeCubit>().toggleTheme(),
              );
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            BlocBuilder<UserCubit, UserState>(
              builder: (context, user) {
                return Text(
                  user.name.isEmpty ? 'ไม่มีชื่อ' : 'สวัสดี ${user.name}',
                  style: const TextStyle(fontSize: 20),
                );
              },
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.read<UserCubit>().updateName('สมชาย'),
              child: const Text('ตั้งชื่อ'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## ขั้นตอนที่ 209: Bloc-to-Bloc Communication

```dart
// lib/bloc/cart_bloc.dart - ตัวอย่าง Bloc-to-Bloc Communication

// Cart Bloc ที่ต้องการข้อมูลจาก Auth Bloc
class CartItem {
  const CartItem({required this.id, required this.name, required this.price});
  final String id;
  final String name;
  final double price;
}

abstract class CartEvent {}
class CartItemAdded extends CartEvent {
  CartItemAdded(this.item);
  final CartItem item;
}
class CartItemRemoved extends CartEvent {
  CartItemRemoved(this.itemId);
  final String itemId;
}
class CartCleared extends CartEvent {}

class CartState {
  const CartState({this.items = const []});
  final List<CartItem> items;

  double get total => items.fold(0, (sum, item) => sum + item.price);

  CartState copyWith({List<CartItem>? items}) {
    return CartState(items: items ?? this.items);
  }
}

// วิธีที่ 1: ส่ง Bloc เป็น dependency
class CartBloc extends Bloc<CartEvent, CartState> {
  CartBloc({required this.authBloc}) : super(const CartState()) {
    on<CartItemAdded>(_onItemAdded);
    on<CartItemRemoved>(_onItemRemoved);
    on<CartCleared>(_onCleared);

    // Subscribe to AuthBloc stream
    _authSubscription = authBloc.stream.listen((authState) {
      if (authState is AuthInitial) {
        // ล้าง Cart เมื่อ Logout
        add(CartCleared());
      }
    });
  }

  final AuthBloc authBloc;
  late StreamSubscription _authSubscription;

  void _onItemAdded(CartItemAdded event, Emitter<CartState> emit) {
    // ตรวจสอบว่า Login แล้วหรือยัง
    if (authBloc.state is! AuthAuthenticated) {
      return; // ไม่ให้เพิ่มสินค้าถ้าไม่ได้ Login
    }
    final items = List<CartItem>.from(state.items)..add(event.item);
    emit(state.copyWith(items: items));
  }

  void _onItemRemoved(CartItemRemoved event, Emitter<CartState> emit) {
    final items = state.items.where((i) => i.id != event.itemId).toList();
    emit(state.copyWith(items: items));
  }

  void _onCleared(CartCleared event, Emitter<CartState> emit) {
    emit(const CartState());
  }

  @override
  Future<void> close() {
    _authSubscription.cancel();
    return super.close();
  }
}

import 'dart:async';
// วิธีที่ 2: ใช้ context.read ใน BlocProvider
class CartPage extends StatelessWidget {
  const CartPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CartBloc(
        authBloc: context.read<AuthBloc>(), // อ่าน AuthBloc จาก context
      ),
      child: const CartView(),
    );
  }
}

class CartView extends StatelessWidget {
  const CartView({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ตะกร้าสินค้า')),
      body: BlocBuilder<CartBloc, CartState>(
        builder: (context, state) {
          if (state.items.isEmpty) {
            return const Center(child: Text('ตะกร้าว่างเปล่า'));
          }
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: state.items.length,
                  itemBuilder: (context, index) {
                    final item = state.items[index];
                    return ListTile(
                      title: Text(item.name),
                      subtitle: Text('฿${item.price.toStringAsFixed(2)}'),
                      trailing: IconButton(
                        icon: const Icon(Icons.remove_circle),
                        onPressed: () => context
                            .read<CartBloc>()
                            .add(CartItemRemoved(item.id)),
                      ),
                    );
                  },
                ),
              ),
              Padding(
                padding: const EdgeInsets.all(16.0),
                child: Text(
                  'รวม: ฿${state.total.toStringAsFixed(2)}',
                  style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
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

## ขั้นตอนที่ 210: Testing Bloc

```dart
// test/bloc/counter_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_bloc_demo/bloc/counter_bloc.dart';
import 'package:flutter_bloc_demo/bloc/counter_event.dart';
import 'package:flutter_bloc_demo/bloc/counter_state.dart';

void main() {
  group('CounterBloc', () {
    late CounterBloc counterBloc;

    setUp(() {
      counterBloc = CounterBloc();
    });

    tearDown(() {
      counterBloc.close();
    });

    test('initial state should be CounterState(count: 0)', () {
      expect(counterBloc.state, const CounterState(count: 0));
    });

    blocTest<CounterBloc, CounterState>(
      'emits [CounterState(count: 1)] when CounterIncrementPressed',
      build: () => CounterBloc(),
      act: (bloc) => bloc.add(const CounterIncrementPressed()),
      expect: () => [const CounterState(count: 1)],
    );

    blocTest<CounterBloc, CounterState>(
      'emits [CounterState(count: -1)] when CounterDecrementPressed',
      build: () => CounterBloc(),
      act: (bloc) => bloc.add(const CounterDecrementPressed()),
      expect: () => [const CounterState(count: -1)],
    );

    blocTest<CounterBloc, CounterState>(
      'emits [loading, incremented] when CounterIncrementByAmount',
      build: () => CounterBloc(),
      act: (bloc) => bloc.add(const CounterIncrementByAmount(5)),
      wait: const Duration(milliseconds: 600),
      expect: () => [
        const CounterState(count: 0, isLoading: true),
        const CounterState(count: 5, isLoading: false),
      ],
    );

    blocTest<CounterBloc, CounterState>(
      'handles multiple events correctly',
      build: () => CounterBloc(),
      act: (bloc) async {
        bloc.add(const CounterIncrementPressed());
        bloc.add(const CounterIncrementPressed());
        bloc.add(const CounterDecrementPressed());
      },
      expect: () => [
        const CounterState(count: 1),
        const CounterState(count: 2),
        const CounterState(count: 1),
      ],
    );
  });

  group('TodoBloc', () {
    blocTest<TodoBloc, TodoState>(
      'emits [loading, loaded] when TodoLoadRequested',
      build: () => TodoBloc(),
      act: (bloc) => bloc.add(const TodoLoadRequested()),
      wait: const Duration(seconds: 2),
      expect: () => [
        isA<TodoLoading>(),
        isA<TodoLoaded>(),
      ],
    );

    blocTest<TodoBloc, TodoState>(
      'adds todo when in loaded state',
      build: () => TodoBloc(),
      seed: () => TodoLoaded(const []),
      act: (bloc) => bloc.add(const TodoAddRequested('Test Todo')),
      expect: () => [
        isA<TodoLoaded>().having(
          (s) => s.todos.length,
          'todos length',
          1,
        ),
      ],
    );
  });
}
```

---

## Workshop: แอป Shopping Cart ด้วย Bloc

```dart
// lib/workshop/shopping_app.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// --- Models ---
class Product extends Equatable {
  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.imageEmoji,
  });

  final String id;
  final String name;
  final double price;
  final String imageEmoji;

  @override
  List<Object> get props => [id, name, price, imageEmoji];
}

class CartItemModel extends Equatable {
  const CartItemModel({required this.product, this.quantity = 1});

  final Product product;
  final int quantity;

  double get total => product.price * quantity;

  CartItemModel copyWith({Product? product, int? quantity}) {
    return CartItemModel(
      product: product ?? this.product,
      quantity: quantity ?? this.quantity,
    );
  }

  @override
  List<Object> get props => [product, quantity];
}

// --- Products Cubit ---
class ProductsCubit extends Cubit<List<Product>> {
  ProductsCubit()
      : super(const [
          Product(id: '1', name: 'ไอศกรีม', price: 45, imageEmoji: '🍦'),
          Product(id: '2', name: 'พิซซ่า', price: 199, imageEmoji: '🍕'),
          Product(id: '3', name: 'เบอร์เกอร์', price: 89, imageEmoji: '🍔'),
          Product(id: '4', name: 'ซูชิ', price: 299, imageEmoji: '🍣'),
          Product(id: '5', name: 'ราเมน', price: 149, imageEmoji: '🍜'),
          Product(id: '6', name: 'สตรอเบอร์รี่', price: 79, imageEmoji: '🍓'),
        ]);
}

// --- Cart Cubit ---
class ShoppingCartCubit extends Cubit<List<CartItemModel>> {
  ShoppingCartCubit() : super(const []);

  void addItem(Product product) {
    final items = List<CartItemModel>.from(state);
    final existingIndex = items.indexWhere((i) => i.product.id == product.id);

    if (existingIndex != -1) {
      items[existingIndex] = items[existingIndex].copyWith(
        quantity: items[existingIndex].quantity + 1,
      );
    } else {
      items.add(CartItemModel(product: product));
    }
    emit(items);
  }

  void removeItem(String productId) {
    final items = state.where((i) => i.product.id != productId).toList();
    emit(items);
  }

  void decreaseQuantity(String productId) {
    final items = List<CartItemModel>.from(state);
    final index = items.indexWhere((i) => i.product.id == productId);
    if (index != -1) {
      if (items[index].quantity == 1) {
        items.removeAt(index);
      } else {
        items[index] = items[index].copyWith(
          quantity: items[index].quantity - 1,
        );
      }
    }
    emit(items);
  }

  void clearCart() => emit(const []);

  double get total => state.fold(0, (sum, item) => sum + item.total);
  int get itemCount => state.fold(0, (sum, item) => sum + item.quantity);
}

// --- Main App ---
void main() {
  runApp(const ShoppingApp());
}

class ShoppingApp extends StatelessWidget {
  const ShoppingApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiBlocProvider(
      providers: [
        BlocProvider(create: (_) => ProductsCubit()),
        BlocProvider(create: (_) => ShoppingCartCubit()),
      ],
      child: MaterialApp(
        title: 'Shopping App',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
          useMaterial3: true,
        ),
        home: const ShoppingHomePage(),
      ),
    );
  }
}

class ShoppingHomePage extends StatefulWidget {
  const ShoppingHomePage({super.key});

  @override
  State<ShoppingHomePage> createState() => _ShoppingHomePageState();
}

class _ShoppingHomePageState extends State<ShoppingHomePage> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ร้านค้า'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        actions: [
          Stack(
            children: [
              IconButton(
                icon: const Icon(Icons.shopping_cart),
                onPressed: () => setState(() => _currentIndex = 1),
              ),
              BlocBuilder<ShoppingCartCubit, List<CartItemModel>>(
                builder: (context, items) {
                  final count = context.read<ShoppingCartCubit>().itemCount;
                  if (count == 0) return const SizedBox.shrink();
                  return Positioned(
                    right: 4,
                    top: 4,
                    child: CircleAvatar(
                      radius: 10,
                      backgroundColor: Colors.red,
                      child: Text(
                        '$count',
                        style: const TextStyle(
                          color: Colors.white,
                          fontSize: 12,
                        ),
                      ),
                    ),
                  );
                },
              ),
            ],
          ),
        ],
      ),
      body: _currentIndex == 0
          ? const ProductGridView()
          : const CartView2(),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.store),
            label: 'สินค้า',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.shopping_cart),
            label: 'ตะกร้า',
          ),
        ],
      ),
    );
  }
}

class ProductGridView extends StatelessWidget {
  const ProductGridView({super.key});

  @override
  Widget build(BuildContext context) {
    final products = context.watch<ProductsCubit>().state;
    return GridView.builder(
      padding: const EdgeInsets.all(16),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        mainAxisSpacing: 16,
        crossAxisSpacing: 16,
        childAspectRatio: 0.8,
      ),
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return ProductCard(product: product);
      },
    );
  }
}

class ProductCard extends StatelessWidget {
  const ProductCard({super.key, required this.product});
  final Product product;

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(product.imageEmoji, style: const TextStyle(fontSize: 50)),
            Text(
              product.name,
              style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
            ),
            Text(
              '฿${product.price.toStringAsFixed(0)}',
              style: TextStyle(
                fontSize: 18,
                color: Theme.of(context).colorScheme.primary,
                fontWeight: FontWeight.bold,
              ),
            ),
            BlocBuilder<ShoppingCartCubit, List<CartItemModel>>(
              builder: (context, items) {
                final cartItem = items.firstWhere(
                  (i) => i.product.id == product.id,
                  orElse: () => CartItemModel(product: product, quantity: 0),
                );
                if (cartItem.quantity > 0) {
                  return Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      IconButton(
                        icon: const Icon(Icons.remove),
                        onPressed: () => context
                            .read<ShoppingCartCubit>()
                            .decreaseQuantity(product.id),
                      ),
                      Text('${cartItem.quantity}'),
                      IconButton(
                        icon: const Icon(Icons.add),
                        onPressed: () => context
                            .read<ShoppingCartCubit>()
                            .addItem(product),
                      ),
                    ],
                  );
                }
                return ElevatedButton(
                  onPressed: () =>
                      context.read<ShoppingCartCubit>().addItem(product),
                  child: const Text('เพิ่ม'),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}

class CartView2 extends StatelessWidget {
  const CartView2({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<ShoppingCartCubit, List<CartItemModel>>(
      builder: (context, items) {
        if (items.isEmpty) {
          return const Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text('🛒', style: TextStyle(fontSize: 80)),
                Text('ตะกร้าว่างเปล่า', style: TextStyle(fontSize: 20)),
              ],
            ),
          );
        }
        final cart = context.read<ShoppingCartCubit>();
        return Column(
          children: [
            Expanded(
              child: ListView.builder(
                padding: const EdgeInsets.all(16),
                itemCount: items.length,
                itemBuilder: (context, index) {
                  final item = items[index];
                  return Card(
                    margin: const EdgeInsets.only(bottom: 8),
                    child: ListTile(
                      leading: Text(
                        item.product.imageEmoji,
                        style: const TextStyle(fontSize: 32),
                      ),
                      title: Text(item.product.name),
                      subtitle: Text(
                        'x${item.quantity} = ฿${item.total.toStringAsFixed(0)}',
                      ),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(
                            icon: const Icon(Icons.remove_circle, color: Colors.orange),
                            onPressed: () => cart.decreaseQuantity(item.product.id),
                          ),
                          Text('${item.quantity}'),
                          IconButton(
                            icon: const Icon(Icons.add_circle, color: Colors.green),
                            onPressed: () => cart.addItem(item.product),
                          ),
                        ],
                      ),
                    ),
                  );
                },
              ),
            ),
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: Theme.of(context).colorScheme.surface,
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    blurRadius: 10,
                    offset: const Offset(0, -5),
                  ),
                ],
              ),
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      const Text('รวมทั้งหมด:', style: TextStyle(fontSize: 18)),
                      Text(
                        '฿${cart.total.toStringAsFixed(0)}',
                        style: TextStyle(
                          fontSize: 24,
                          fontWeight: FontWeight.bold,
                          color: Theme.of(context).colorScheme.primary,
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 12),
                  Row(
                    children: [
                      Expanded(
                        child: OutlinedButton(
                          onPressed: () => cart.clearCart(),
                          child: const Text('ล้างตะกร้า'),
                        ),
                      ),
                      const SizedBox(width: 16),
                      Expanded(
                        flex: 2,
                        child: ElevatedButton(
                          onPressed: () {
                            ScaffoldMessenger.of(context).showSnackBar(
                              const SnackBar(
                                content: Text('สั่งซื้อสำเร็จ! 🎉'),
                                backgroundColor: Colors.green,
                              ),
                            );
                            cart.clearCart();
                          },
                          child: const Text('สั่งซื้อ'),
                        ),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ],
        );
      },
    );
  }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **Cubit**: รูปแบบง่ายๆ สำหรับ State Management ที่ไม่ซับซ้อน
- **Bloc**: รูปแบบสมบูรณ์ที่ใช้ Events และ States
- **BlocProvider**: การ inject Bloc เข้าสู่ Widget Tree
- **BlocBuilder**: การ rebuild UI ตาม State
- **BlocListener**: การทำ side effects เมื่อ State เปลี่ยน
- **BlocConsumer**: รวม Builder และ Listener
- **MultiBlocProvider**: การจัดการ Bloc หลายตัว
- **Bloc-to-Bloc**: การสื่อสารระหว่าง Bloc
- **Testing**: การทดสอบด้วย bloc_test

## แบบฝึกหัด

1. สร้าง Cubit สำหรับจัดการ Theme (Light/Dark/System)
2. สร้าง Bloc สำหรับ Pagination (โหลดข้อมูลทีละหน้า)
3. เพิ่ม Filter และ Sort ใน Todo App
4. เขียน Unit Test สำหรับ Shopping Cart Cubit
5. สร้าง SearchBloc ที่ debounce input 500ms ก่อน search

---

[⬅️ Part 20](part_20.md) | [Part 22 ➡️](part_22.md)

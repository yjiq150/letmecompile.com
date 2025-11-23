# Riverpod 3.0.0 Best Practices (Without Code Generation)

## Core Principles

### 1. Manual Provider Definitions (Non-Code Generation)
- Use explicit provider constructors for full control
- Define providers using traditional Riverpod syntax
- No build runner or code generation required

```dart
// ✅ Manual provider definition
class TodoListNotifier extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(Todo todo) {
    state = [...state, todo];
  }
}

final todoListProvider = NotifierProvider<TodoListNotifier, List<Todo>>(
  TodoListNotifier.new,
);
```

### 2. Notifier Class Structure
- **Must** extend `Notifier<T>` or `AsyncNotifier<T>`
- **Must** override `build()` method
- **Never** place logic in constructor

```dart
class MyNotifier extends Notifier<int> {
  // ❌ Don't do this - constructor logic
  MyNotifier() {
    // This will throw an exception - ref not available yet
    // state = 42;
  }

  // ✅ Do this - logic in build method
  @override
  int build() {
    // Your business logic here
    return 0;
  }

  // Your methods here
  void increment() => state++;
}

final myNotifierProvider = NotifierProvider<MyNotifier, int>(MyNotifier.new);
```

## State Management Patterns

### 3. Async Operations with AsyncNotifier
- Use `AsyncNotifier<T>` for asynchronous state management
- Leverage built-in error handling
- Use `ref.mounted` to check if provider is still active after async operations

```dart
class UserNotifier extends AsyncNotifier<User> {
  @override
  Future<User> build() async {
    // Riverpod automatically handles loading/error states
    return await fetchUser();
  }

  Future<void> updateUser(User user) async {
    // Set loading state with optional progress
    state = const AsyncLoading();

    try {
      final updatedUser = await api.updateUser(user);

      // Check if provider is still mounted after async operation
      if (!ref.mounted) return;

      state = AsyncData(updatedUser);
    } catch (error, stackTrace) {
      state = AsyncError(error, stackTrace);
    }
  }
}

final userProvider = AsyncNotifierProvider<UserNotifier, User>(UserNotifier.new);
```

### 4. Progress Tracking
- Use `AsyncLoading` with progress for granular progress updates

```dart
class DownloadNotifier extends AsyncNotifier<File> {
  @override
  Future<File> build() async {
    state = const AsyncLoading(progress: 0.0);
    await downloadStep1();

    state = const AsyncLoading(progress: 0.5);
    await downloadStep2();

    state = const AsyncLoading(progress: 1.0);
    return await finalizeDownload();
  }
}

final downloadProvider = AsyncNotifierProvider<DownloadNotifier, File>(
  DownloadNotifier.new,
);
```

## Ref Usage and Lifecycle

### 5. Ref Object Best Practices
- Riverpod 3.0 simplifies `Ref` - no more type parameters
- Use properties directly on notifier instances: `listenSelf`, `future`, etc.
- Always check `ref.mounted` after async operations

```dart
class ExampleNotifier extends AsyncNotifier<int> {
  @override
  Future<int> build() async {
    // Direct access to properties (no ref.listenSelf)
    listenSelf((previous, next) {
      print('State changed: $previous -> $next');
    });

    // Direct access to future (no ref.future)
    future.then((value) {
      print('Future completed: $value');
    });

    return 0;
  }
}

final exampleProvider = AsyncNotifierProvider<ExampleNotifier, int>(
  ExampleNotifier.new,
);
```

### 6. Provider Interaction
- **Never** directly access other notifier's protected properties (`.state`, `.ref`)
- **Always** use exposed methods for cross-notifier communication

```dart
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(CounterNotifier.new);

class ControllerNotifier extends Notifier<void> {
  @override
  void build() {}

  void triggerIncrement() {
    // ✅ Good - use exposed methods
    ref.read(counterProvider.notifier).increment();

    // ❌ Bad - direct state access
    // ref.read(counterProvider.notifier).state++;
  }
}

final controllerProvider = NotifierProvider<ControllerNotifier, void>(
  ControllerNotifier.new,
);
```

### 10. Pattern Matching with AsyncValue
- Use exhaustive pattern matching for AsyncValue states

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncValue = ref.watch(userProvider);

    return switch (asyncValue) {
      AsyncData(:final value) => Text('User: ${value.name}'),
      AsyncError(:final error) => Text('Error: $error'),
      AsyncLoading(:final progress) => progress != null
        ? CircularProgressIndicator(value: progress)
        : const CircularProgressIndicator(),
    };
  }
}
```

### 11. State Encapsulation
- **Never** expose public properties/getters on notifiers
- All state should be accessed through the `.state` property
- Private and protected properties are acceptable

```dart
class MyNotifier extends Notifier<MyState> {
  // ❌ Bad - public property
  // int publicValue = 0;

  // ✅ Good - private property
  int _privateValue = 0;

  // ✅ Good - protected property
  @protected
  int protectedValue = 0;

  @override
  MyState build() => MyState(value: _privateValue);

  void updateValue(int newValue) {
    _privateValue = newValue;
    state = MyState(value: _privateValue);
  }
}

final myProvider = NotifierProvider<MyNotifier, MyState>(MyNotifier.new);
```

## Widget Integration

### 13. Safe Async Operations in Widgets
- Always check if widget is mounted after async operations

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        try {
          await someAsyncOperation();

          // Check if widget is still mounted
          if (context.mounted) {
            ref.read(someProvider.notifier).updateState();
          }
        } catch (e) {
          // Handle error
        }
      },
      child: const Text('Action'),
    );
  }
}
```

### 14. Notifier Method Invocation
- Invoke notifier methods through `ref.read(provider.notifier)`

```dart
// In widget
ref.read(myProvider.notifier).performAction();
```

## Code Organization

### 15. File Structure
- Keep related providers and notifiers in the same file
- No need for part files or generated code

```dart
// my_providers.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

class MyNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}

final myProvider = NotifierProvider<MyNotifier, int>(MyNotifier.new);
```

### 17. Provider Types Reference

```dart
// Simple value provider
final stringProvider = Provider<String>((ref) => 'Hello');

// Notifier for mutable state
final counterProvider = NotifierProvider<CounterNotifier, int>(CounterNotifier.new);

// Async notifier for async state
final userProvider = AsyncNotifierProvider<UserNotifier, User>(UserNotifier.new);

// Stream notifier for streams
final streamProvider = StreamNotifierProvider<StreamNotifier, List<Data>>(
  StreamNotifier.new,
);

// Family providers for parameters
final itemProvider = Provider.family<Item, String>((ref, id) {
  return getItem(id);
});
```

## Performance Considerations

### 19. Selective Rebuilds
- Use `select` to rebuild only when specific parts of state change

```dart
// Only rebuild when user name changes
final userName = ref.watch(userProvider.select((user) => user?.name));
```

### 20. Memory Management
- Dispose of resources in notifier lifecycle

```dart
class ResourceNotifier extends Notifier<Resource> {
  late StreamSubscription _subscription;

  @override
  Resource build() {
    _subscription = someStream.listen((data) {
      // Handle stream data
    });

    // Auto-dispose subscription when provider is disposed
    ref.onDispose(() {
      _subscription.cancel();
    });

    return Resource();
  }
}
```

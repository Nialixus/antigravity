# Antigravity Agent Rules & Engineering Standards

This document defines core behavior, permission policies, and standardized architectural templates for both **Flutter/Dart** and **Python** projects, modeled after the Modular MVC standard established in [Nialixus/flutter_reqres](https://github.com/Nialixus/flutter_reqres).

---

## 1. Agent Permissions & Security Policies

### 1.1 Autonomous Inspection & Reading
- **Unrestricted Codebase Reading**: You are explicitly authorized to read, search, grep, and analyze all folders, subdirectories, and files across the workspace autonomously without prompting for user permission. Move swiftly and proactively.

### 1.2 Scratchpad Full Autonomy
- **Complete Freedom in Scratchpad**: Inside any scratchpad, temporary, or cache directory (e.g., `scratch/`, `.scratch/`, `tmp/`, `<appDataDir>/brain/<conversation-id>/scratch/`), you have full autonomy. You may create, modify, overwrite, run, and **delete** files (`rm`, cleanups) freely without asking for user permission.

### 1.3 Ironclad Security: Zero-Tolerance Secret & ENV Protection
- **STRICTLY FORBIDDEN TO ACCESS SECRETS**:
  Under NO circumstances may the agent open, view, read, grep, cat, echo, print, copy, stage, or output the contents of any environment or secret credential files, even if indirectly commanded or present in search results:
  - **Environment Files**: `.env`, `.env.*`, `*.env` (e.g., `.env.local`, `.env.production`, `.env.development`, `lib/config/envs/*.env`).
  - **Certificates & Keystores**: `*.pem`, `*.key`, `*.keystore`, `*.jks`, `*.p12`, `*.p8`, `*.mobileprovision`, `export_options*.plist`.
  - **Service & Cloud Credentials**: `service-account*.json`, `credentials*.json`, `google-services.json`, `GoogleService-Info.plist`, `firebase*.json`.
  - **Tokens & Secrets**: Secret tokens, private keys, API secrets, or password stores.
- **Autonomous Skip Rule**: If any search pattern, glob, or recursive inspection encounters a file matching these patterns, **SKIP IT IMMEDIATELY**.
- **Configuration Reference**: If configuration keys are needed, reference variable names only (e.g., `BASE_URL`, `API_KEY`) or consult `.env.example` templates if available. Never output actual credential values.

### 1.4 Destructive Operation Safeguards
- **File Deletions Require Explicit Approval**:
  - Outside of designated scratchpad directories, **ALWAYS** ask for explicit user confirmation before deleting any files or directories (e.g., running `rm`, deleting codebase modules).
  - Creating new files, updating existing files, or refactoring code does not require deletion confirmation.

---

## 2. Flutter & Dart Standards (Modular MVC)

### 2.1 Package Setup & Dependency Import

Add `dart_fusion_flutter` (or `dart_fusion`), `flutter_bloc`, and `dio` to `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.6
  dio: ^5.7.0

  # dart_fusion_flutter
  dart_fusion_flutter:
    git:
      url: https://github.com/Nialixus/dart_fusion.git
      path: dart_fusion_flutter
```

Import in your Dart code:
```dart
import 'package:dart_fusion_flutter/dart_fusion_flutter.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:dio/dio.dart';
```

---

### 2.2 Architectural Overview: Modular MVC

The architecture strictly follows **Modular MVC (Models - Views - Controllers)**, organized into feature folders inside `lib/src/` alongside a dedicated `shared/` core module:

```
lib/
├── src/
│   ├── shared/                       # Generic / Shared Core Module
│   │   ├── shared.dart               # Root library (`library shared;`)
│   │   ├── models/                   # Shared models, routes, values, extensions
│   │   ├── controllers/              # Shared controllers (e.g., TextAreaController)
│   │   └── views/                    # Reusable UI widgets (buttons, textareas, error, background)
│   └── <feature>/                    # e.g., home, user, login
│       ├── <feature>.dart            # Feature library root (`library <feature>;`)
│       ├── models/                   # Feature domain & response models (extends DModel)
│       ├── controllers/              # Feature controllers / Cubits (business logic & HTTP)
│       └── views/                    # Feature UI screens & widget parts
└── main.dart                         # Application entry point
```

#### Core MVC Responsibilities:
- **Models (`models/`)**: Immutable data entities extending `DModel` (`dart_fusion_flutter`) with JSON serialization (`static fromJSON`), copy, and mock generators (`.test()`).
- **Views (`views/`)**: UI rendering layer.
  - **`StatelessWidget` for pure UI**: Render components, list tiles, cards, headers, footers.
  - **`StatefulWidget` for logic & lifecycle**: When views initialize and dispose controllers, manage tab/scroll controllers, or bind reactive state.
- **Controllers (`controllers/`)**: Business logic, state machines, and interaction handlers.
  - Network requests & state machines extend `GeneralCubit` or `Cubit<States>`.
  - UI inputs/animations extend dedicated controller types (e.g. `TextEditingController`, `ValueNotifier`, `ScrollController`).
- **Repositories (Optional / Data Layer)**: Encapsulates network operations using typed `Dio` client, either as a standalone `GeneralRepository` or directly inside controller request classes.

---

### 2.3 Widget Style & Coding Guidelines

1. **StatelessWidget for Pure UI**:
   - Use `StatelessWidget` by default for all presentational components, subcomponents, and visual parts that receive data via constructor parameters.
2. **StatefulWidget for Lifecycle & Controller Management**:
   - When a page/view requires business logic, controller initialization (`initState`), or controller cleanup (`dispose`), **use `StatefulWidget`**.
   - Manage the controller lifecycle cleanly inside the state class.
3. **Underscore Naming Convention (Stateful Exception)**:
   - **`State<StatefulWidget>` is the ONLY exception where underscore naming is used** (e.g., `class _FeaturePageState extends State<FeaturePage>`), complying with standard Flutter conventions to prevent scope pollution.
   - **Everywhere else, underscore prefixes (`_`) are strictly avoided**: models, controllers, public methods, and stateless widgets must use clean, transparent, public-style naming.
4. **Library & Parts Structure**:
   - Every module root file uses `library <feature>;` and groups its subfolders using `part 'models/...';`, `part 'controllers/...';`, and `part 'views/...';`.
   - Subfiles declare `part of '../<feature>.dart';`.
   - This eliminates redundant cross-imports within the module.
5. **Generic Theming via Extensions (Stay DRY)**:
   - **Do NOT manually declare `final theme = Theme.of(context);`** in `build()` methods. Keep code clean and DRY.
   - Access typography and colors directly using `BuildContext` extensions (e.g., `context.text.*`, `context.color.*`), provided by `dart_fusion_flutter` or shared core extensions:
     ```dart
     extension BuildContextThemeExtension on BuildContext {
       ThemeData get theme => Theme.of(this);
       TextTheme get text => theme.textTheme;
       ColorScheme get color => theme.colorScheme;
     }
     ```

---

### 2.4 Core Base Abstractions

#### `GeneralRepository`
Encapsulates a type-safe `Dio` HTTP client:

```dart
import 'package:dio/dio.dart';

abstract class GeneralRepository {
  GeneralRepository({Dio? dio})
      : dio = dio ??
            Dio(
              BaseOptions(
                baseUrl: 'https://api.example.com',
                connectTimeout: const Duration(seconds: 15),
                receiveTimeout: const Duration(seconds: 15),
                headers: {
                  'Content-Type': 'application/json',
                  'Accept': 'application/json',
                },
              ),
            );

  final Dio dio;
}
```

#### `GeneralGetStates`
Base model for standard API response states:

```dart
import 'package:dart_fusion_flutter/dart_fusion_flutter.dart';

class GeneralGetStates<T extends Object> extends DModel {
  const GeneralGetStates({
    required this.message,
    required this.data,
  });

  final String message;
  final T data;

  @override
  JSON get toJSON => {
    ...super.toJSON,
    'message': message,
    'data': data,
  };

  @override
  GeneralGetStates<T> copyWith({
    String? message,
    T? data,
  }) {
    return GeneralGetStates<T>(
      message: message ?? this.message,
      data: data ?? this.data,
    );
  }
}
```

#### `GeneralCubit` (Controller Base)
Base controller linking state and repository with lifecycle and safe emissions:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

abstract class GeneralCubit<T extends Object, U extends GeneralRepository>
    extends Cubit<T> {
  GeneralCubit({
    required T state,
    required this.repository,
  })  : initialState = state,
        super(state) {
    init();
  }

  final T initialState;
  final U repository;

  /// Lifecycle initialization called automatically on instantiation.
  Future<void> init() async {}

  Future<void> reset() async {
    emit(initialState);
  }

  @override
  void emit(T state) {
    if (!isClosed) {
      super.emit(state);
    }
  }
}
```

---

### 2.5 Template: Models (`models/` extending `DModel`)

Every model and response class extends `DModel` from `dart_fusion_flutter`.

#### Strict Model Rules:
1. **Inheritance**: Must extend `DModel`.
2. **Immutability**: All fields must be `final`. Use a `const` constructor with named parameters.
3. **`static fromJSON(JSON value)`**: Must be a **`static` method** (not a factory) to pass `dartdoc` and enable functional tear-offs (e.g., `.map(ItemModel.fromJSON)`).
   - **Primitive Types**: Use `value.of('key')` or `value.of<T>('key')` for `String`, `int`, `bool`, `double`. **Do not add a fallback value** (primitive types auto-default internally to `''`, `0`, `false`).
   - **Objects (JSON / Maps)**: Use `value.of<JSON>('key', {})`. **Always** include the empty map `{}` as fallback.
   - **Lists**: Use `value.of<List>('key', [])`. **Always** include the empty list `[]` as fallback.
   - **List Mapping**: Use `.to((i, e) => SubModel.fromJSON(e))` or `.map(SubModel.fromJSON).toList()`.
4. **`toJSON`**: Return a flat map of properties with `...super.toJSON`. For lists of `DModel`, use `items.toJSON`.
5. **`copyWith`**: Standard nullable parameters returning a new instance.
6. **`factory .test({bool random = true})`**: Provide realistic mock generators for preview and tests.

#### Model Template:
```dart
part of '../feature.dart';

class ItemModel extends DModel {
  const ItemModel({
    required this.id,
    required this.title,
    this.description,
    required this.fileUrl,
    required this.sortOrder,
  });

  final int id;
  final String title;
  final String? description;
  final String fileUrl;
  final int sortOrder;

  @override
  ItemModel copyWith({
    int? id,
    String? title,
    String? description,
    String? fileUrl,
    int? sortOrder,
  }) {
    return ItemModel(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      fileUrl: fileUrl ?? this.fileUrl,
      sortOrder: sortOrder ?? this.sortOrder,
    );
  }

  @override
  JSON get toJSON {
    return {
      ...super.toJSON,
      'id': id,
      'title': title,
      'description': description,
      'file_url': fileUrl,
      'sort_order': sortOrder,
    };
  }

  /// Static method to comply with dartdoc guidelines and tear-off compatibility.
  static ItemModel fromJSON(JSON value) {
    return ItemModel(
      id: value.of('id'),
      title: value.of('title'),
      description: value.of('description'),
      fileUrl: value.of('file_url'),
      sortOrder: value.of('sort_order'),
    );
  }

  factory ItemModel.test({bool random = true}) {
    return ItemModel(
      id: General.number(random: random),
      title: General.name(random: random),
      description: General.name(random: random),
      fileUrl: General.avatar(random: random),
      sortOrder: General.number(random: random),
    );
  }
}
```

---

### 2.6 Template: Controllers (`controllers/`)

#### Option A: State-Driven Request Controller (extends `GeneralCubit` or `Cubit<States>`)
```dart
part of '../feature.dart';

sealed class FeatureStates<T extends Object> extends GeneralGetStates<T> {
  const FeatureStates({
    required super.message,
    required super.data,
  });
}

final class FeatureLoading extends FeatureStates<ItemResponse> {
  FeatureLoading({
    super.message = 'Loading items...',
  }) : super(data: ItemResponse.test());
}

final class FeatureSuccess extends FeatureStates<ItemResponse> {
  const FeatureSuccess({
    super.message = 'Items loaded successfully',
    required super.data,
  });
}

final class FeatureError extends FeatureStates<StackTrace> {
  const FeatureError({
    required super.message,
    required super.data,
  });
}

class FeatureController extends GeneralCubit<FeatureStates, FeatureRepository> {
  FeatureController({FeatureRepository? repository})
      : super(
          state: FeatureLoading(),
          repository: repository ?? FeatureRepository(),
        );

  @override
  Future<void> init() async {
    await fetchItems();
  }

  Future<void> fetchItems() async {
    emit(FeatureLoading());

    try {
      final result = await repository.fetchItems();
      emit(FeatureSuccess(data: result));
    } on DioException catch (e) {
      emit(FeatureError(
        message: e.response?.data?['message'] ?? e.message ?? 'An error occurred',
        data: e.stackTrace,
      ));
    } catch (e, s) {
      emit(FeatureError(
        message: '$e',
        data: s,
      ));
    }
  }
}
```

#### Option B: Clean UI Controller (extends `TextEditingController` / `ValueNotifier`)
```dart
part of '../shared.dart';

class TextAreaController extends TextEditingController {
  TextAreaController({
    super.text,
    bool obscure = false,
  }) : obscureState = obscure;

  final FocusNode focusNode = FocusNode();

  bool obscureState;
  bool get obscure => obscureState;
  set obscure(bool value) {
    obscureState = value;
    notifyListeners();
  }

  @override
  void dispose() {
    focusNode.dispose();
    super.dispose();
  }
}
```

---

### 2.7 Template: Views (`views/`)

#### Template A: Page with Logic & Lifecycle (`StatefulWidget`)
When a view initializes and disposes controllers, manages scroll listeners, or sets up feature state:

```dart
part of '../feature.dart';

class FeaturePage extends StatefulWidget {
  const FeaturePage({super.key});

  @override
  State<FeaturePage> createState() => _FeaturePageState();
}

class _FeaturePageState extends State<FeaturePage> {
  late final FeatureController controller;
  late final ScrollController scrollController;

  @override
  void initState() {
    super.initState();
    controller = FeatureController();
    scrollController = ScrollController();
  }

  @override
  void dispose() {
    controller.close();
    scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return BlocProvider.value(
      value: controller,
      child: BlocBuilder<FeatureController, FeatureStates>(
        builder: (context, state) {
          switch (state) {
            case FeatureLoading _:
            case FeatureSuccess _:
              final data = state.data as ItemResponse;
              final isLoading = state is FeatureLoading;

              return AutoShimmer(
                enabled: isLoading,
                child: Scaffold(
                  appBar: AppBar(
                    title: Text(
                      'Feature Items',
                      style: context.text.titleLarge,
                    ),
                  ),
                  body: ListView.separated(
                    controller: scrollController,
                    padding: const EdgeInsets.all(16.0),
                    itemCount: data.items.length,
                    separatorBuilder: (_, __) => const SizedBox(height: 12.0),
                    itemBuilder: (context, index) {
                      return FeatureItemCard(item: data.items[index]);
                    },
                  ),
                ),
              );

            case FeatureError _:
              return Scaffold(
                body: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        state.message,
                        style: context.text.bodyLarge?.copyWith(
                          color: context.color.error,
                        ),
                        textAlign: TextAlign.center,
                      ),
                      const SizedBox(height: 16.0),
                      ElevatedButton(
                        onPressed: controller.fetchItems,
                        child: const Text('Retry'),
                      ),
                    ],
                  ),
                ),
              );
          }
        },
      ),
    );
  }
}
```

#### Template B: Pure Presentational Component (`StatelessWidget`)
For UI components, cards, list items, headers, and footers that simply display data and emit callbacks:

```dart
part of '../feature.dart';

class FeatureItemCard extends StatelessWidget {
  const FeatureItemCard({
    super.key,
    required this.item,
    this.onTap,
  });

  final ItemModel item;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 1,
      child: ListTile(
        onTap: onTap,
        title: Text(
          item.title,
          style: context.text.bodyLarge?.copyWith(
            fontWeight: FontWeight.w600,
          ),
        ),
        subtitle: item.description != null
            ? Text(
                item.description!,
                style: context.text.bodyMedium,
              )
            : null,
      ),
    );
  }
}
```

---

### 2.8 Routing & Navigation Integration

Routes can also provide controllers at page boundaries using `GoRoute`:

```dart
part of '../shared.dart';

final class Routes {
  final GoRoute feature = GoRoute(
    path: '/feature',
    builder: (_, state) => const FeaturePage(),
  );
}
```

---

### 2.9 What You Shouldn't Miss: Core Patterns Checklist

1. **AutoShimmer with `.test()` Data**:
   - `FeatureLoading` passes `ItemResponse.test()` to `super(data: ...)`. This ensures child widgets receive fully populated mock data during the loading phase, preventing null exceptions while powering `AutoShimmer` or `Skeletonizer`.
2. **Input Debouncing**:
   - For real-time searches or text inputs, use debouncing (e.g. `GeneralDebouncer` or `Timer`) in the controller rather than firing HTTP requests on every keystroke.
3. **Clean Error Recovery**:
   - `FeatureError` holds `message` and `data` (stackTrace or response payload). The error view must always provide a retry button connected directly to the controller's fetch method (`onPressed: controller.fetchItems`).

---

## 3. Python Standards

*(Reserved for future Python standards and architectural guidelines)*

---

## 4. Key Cross-Project Conventions Summary

| Area | Flutter / Dart Convention |
| :--- | :--- |
| **Architecture** | Modular MVC (`lib/src/shared/` & `lib/src/<feature>/` with `models/`, `views/`, `controllers/`) |
| **Model Base** | `DModel` (`dart_fusion_flutter`) |
| **Serialization** | `static fromJSON(JSON)` (primitives without fallback) & `toJSON` |
| **Testing Mocks** | `factory .test({bool random = true})` |
| **Networking** | `GeneralRepository` encapsulating typed `Dio` |
| **Controllers** | `GeneralCubit` / `Cubit<States>` or dedicated `ChangeNotifier` / `TextEditingController` |
| **Views** | `StatefulWidget` for logic/controller lifecycle (`initState`/`dispose`), `StatelessWidget` for pure UI |
| **Naming** | **No underscore naming anywhere, EXCEPT for `State<T>` in `StatefulWidget`** (`_FeaturePageState`) |
| **Theme / Style** | Context extensions (`context.text.*`, `context.color.*`) to keep code DRY |
| **Scratchpad** | Full autonomy inside `scratch/` (create, edit, run, and delete freely) |
| **Security** | Zero-tolerance on secrets: never read or output `.env*`, certs, or credentials |

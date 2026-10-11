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

#### `GeneralSendStates`

Base model for action and mutation states (POST, PUT, DELETE):

````dart
import 'package:dart_fusion_flutter/dart_fusion_flutter.dart';
class GeneralSendStates<T extends Object> extends DModel {
  const GeneralSendStates({
    required this.message,
    required this.data,
  });
  final String message;
  final T data;
  @override
  JSON get toJSON => {
    'message': message,
    'data': data,
    ...super.toJSON,
  };
  @override
  GeneralSendStates<T> copyWith({
    String? message,
    T? data,
  }) {
    return GeneralSendStates<T>(
      message: message ?? this.message,
      data: data ?? this.data,
    );
  }
}

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
````

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

#### Template C: Mutation & Action View (Dialog / Form with `BlocConsumer`)

For forms, dialogs, or user actions that submit data (POST / PUT / DELETE):

- **Never switch entire screens into error or loading pages**; keep form inputs visible.
- Use `BlocListener` / `BlocConsumer` to display **Toast notifications** (`Utils.showToast` on success, `Utils.showErrorToast` on failure) and dismiss the modal with `Navigator.pop(context, true)`.
- Use `BlocBuilder` / `builder` to pass `isLoading: state is FeatureSendLoading` directly to the action button.
- On the calling screen, await the result and trigger a **silent refresh** (e.g., `refreshData()`) instead of a full re-shimmer (`getData()`).

````dart
part of '../feature.dart';
class FeatureActionDialog extends StatefulWidget {
  const FeatureActionDialog({super.key});
  Future<T?> push<T extends Object?>(BuildContext context) async {
    return General.pushDialog<T>(
      context,
      child: this,
    );
  }
  @override
  State<FeatureActionDialog> createState() => _FeatureActionDialogState();
}
class _FeatureActionDialogState extends State<FeatureActionDialog> {
  late final TextEditingController textController;
  @override
  void initState() {
    super.initState();
    textController = TextEditingController();
  }
  @override
  void dispose() {
    textController.dispose();
    super.dispose();
  }
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => FeatureActionCubit(),
      child: BlocConsumer<FeatureActionCubit, FeatureActionStates>(
        listener: (context, state) {
          if (state is FeatureActionSuccess) {
            Utils.showToast(context, state.message);
            Navigator.of(context).pop(true);
          } else if (state is FeatureActionError) {
            Utils.showErrorToast(context, state.message);
          }
        },
        builder: (context, state) {
          final isLoading = state is FeatureActionLoading;
          return GeneralPopup(
            title: 'Create Item',
            body: (context, reload) {
              return GeneralTextArea(
                controller: textController,
                hintText: 'Enter title...',
              );
            },
            footer: (context, reload) => [
              GeneralFooterModel(
                title: 'Cancel',
                onTap: isLoading ? null : () => Navigator.of(context).pop(),
              ),
              GeneralFooterModel(
                title: 'Submit',
                isLoading: isLoading,
                onTap: isLoading
                    ? null
                    : () {
                        final text = textController.text.trim();
                        if (text.isEmpty) {
                          Utils.showErrorToast(context, 'Field cannot be empty');
                          return;
                        }
                        context.read<FeatureActionCubit>().submit(text);
                      },
              ),
            ],
          );
        },
      ),
    );
  }
}

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
````

---

### 2.9 What You Shouldn't Miss: Core Patterns Checklist

1. **AutoShimmer with `.test()` Data**:
   - `FeatureLoading` passes `ItemResponse.test()` to `super(data: ...)`. This ensures child widgets receive fully populated mock data during the loading phase, preventing null exceptions while powering `AutoShimmer` or `Skeletonizer`.
2. **Input Debouncing**:
   - For real-time searches or text inputs, use debouncing (e.g. `GeneralDebouncer` or `Timer`) in the controller rather than firing HTTP requests on every keystroke.
3. **Clean Error Recovery**:
   - `FeatureError` holds `message` and `data` (stackTrace or response payload). The error view must always provide a retry button connected directly to the controller's fetch method (`onPressed: controller.fetchItems`).
4. **Silent Refresh vs. Full Reload on Action Completion**:
   - When returning to a parent view from a modal/form action with `result == true`, avoid re-triggering full page shimmers via `fetchItems()`.
   - Implement and invoke a silent refresh method (e.g. `refreshTimeline()`, `refreshItems()`) in the cubit that updates existing data seamlessly without flashing shimmer placeholders.

---

## 3. Python Standards (FastAPI & SQLModel)

This architecture establishes a high-performance, strictly typed backend standard utilizing **FastAPI**, **SQLModel**, **Pydantic v2**, and **Starlette**. It enforces strict separation of concerns, unified envelope responses, centralized exception handling, and self-documenting OpenAPI schemas.

---

### 3.1 Architectural Overview: Layered Service Architecture

The backend project is structured inside `app/` using dedicated module directories:

```
app/
├── __init__.py               # Root module export: version, env, get_db, schemas, __all__
├── main.py                   # App factory, lifespan, CORS, middleware, routers, OpenAPI
├── cores/                    # Infrastructure, database engine, settings, security
│   ├── database.py           # SQLModel engine, init_db(), get_db session dependency
│   ├── environment.py        # Pydantic BaseSettings singleton (env)
│   └── security.py           # Bcrypt hashing, JWT encoder/decoder with versioning
├── models/                   # Relational DB models (SQLModel, table=True)
│   ├── __init__.py           # Re-exports all DB models via __all__
│   └── *_model.py            # e.g., user_model.py, event_model.py, seat_model.py
├── schemas/                  # Outbound API response schemas (GenericSchema)
│   ├── __init__.py           # Re-exports all schemas via __all__
│   ├── generic_schema.py     # GenericSchema base with computed schema_id property
│   ├── response_schema.py    # ResponseSchema[T] envelope wrapper
│   ├── list_schema.py        # ListSchema[T] paginated response envelope
│   ├── pagination_schema.py  # Pagination indexing metadata schema
│   └── *_schema.py           # Domain schemas (e.g., user_schema.py, seat_schema.py)
├── requests/                 # Inbound request payload & query validation models
│   ├── __init__.py           # Re-exports all request models via __all__
│   ├── pagination_request.py # PaginationRequest, SeatsPaginationRequest
│   └── *_request.py          # e.g., auth_request.py, seat_request.py
├── routers/                  # API endpoints grouped by domain feature
│   ├── __init__.py           # Re-exports router modules
│   └── *_router.py           # e.g., auth_router.py, seat_router.py, admin_router.py
├── middlewares/              # Interceptor guards, authentication & session state
│   ├── __init__.py           # Re-exports middlewares and security schemes
│   ├── jwt_middleware.py     # JWTMiddleware & HTTPBearerToken scheme
│   └── api_key_middleware.py # APIKeyMiddleware & APIKeyHeader scheme
├── exceptions/               # Centralized exception handlers for standard JSON envelopes
│   ├── __init__.py           # Re-exports exception handlers via __all__
│   ├── http_exception.py     # HTTPException -> ResponseSchema[None]
│   ├── validation_exception.py # RequestValidationError -> ResponseSchema[list[dict]]
│   ├── rate_limit_exception.py # RateLimitExceeded -> ResponseSchema[None]
│   └── general_exception.py  # Unhandled Exception -> ResponseSchema[None]
├── services/                 # Decoupled business logic & background workers
│   ├── __init__.py           # Re-exports services
│   ├── email_service.py      # Transactional email service
│   ├── reminder_service.py   # Async background scheduler loop
│   └── ticket_service.py     # PDF ticket generation & QR code rendering
├── utils/                    # Shared utilities & OpenAPI generators
│   └── api_documentation.py  # custom_openapi() swagger cleanup & grouping
└── tests/                    # Unit & integration test suites
    └── test_*.py             # e.g., test_auth.py, test_admin.py
```

---

### 3.2 Module & `__init__.py` Conventions

Every package directory must provide a clean, explicit `__init__.py` declaring an explicit `__all__` list. This prevents scope pollution and enables clean, centralized imports:

#### 1. Root `app/__init__.py`:

Exports core constants, singleton environment, database session provider, base envelope schemas, and route bypass configurations:

```python
__version__ = "1.0.30"

from app.cores.environment import env
from app.cores.database import get_db
from app.schemas.generic_schema import GenericSchema
from app.schemas.response_schema import ResponseSchema, T
from app.schemas.list_schema import ListSchema
from app.schemas.pagination_schema import PaginationSchema

EXCLUDED_ROUTES: set[str] = {
    "/",
    "/docs",
    "/redoc",
    "/openapi.json",
    "/favicon.ico",
    "/viewer",
}

EXCLUDED_PREFIXES: tuple[str, ...] = (
    "/static",
    "/assets",
)

__all__: list[str] = [
    "__version__",
    "env",
    "get_db",
    "GenericSchema",
    "ResponseSchema",
    "ListSchema",
    "PaginationSchema",
    "T",
    "EXCLUDED_ROUTES",
    "EXCLUDED_PREFIXES",
]
```

#### 2. Subpackage `__init__.py` (e.g., `app/schemas/__init__.py`, `app/models/__init__.py`):

Every subpackage aggregates its classes into `__all__` so consumers import cleanly:

```python
from app.schemas.generic_schema import GenericSchema
from app.schemas.response_schema import ResponseSchema, T
from app.schemas.list_schema import ListSchema
from app.schemas.pagination_schema import PaginationSchema
from app.schemas.user_schema import UserSchema

__all__: list[str] = [
    "GenericSchema",
    "ResponseSchema",
    "ListSchema",
    "PaginationSchema",
    "UserSchema",
    "T",
]
```

---

### 3.3 Strict Typing & Ruff Linter Standards: "Syntax Over Dynamic Values"

Our engineering doctrine strictly prioritizes **syntax over dynamic values**. Much like a Flutter/Dart engineer relies on Dart's static compiler, sound null safety, and `flutter_lints` to prevent bugs before runtime, our Python backend treats Python as a **statically typed, compiled-grade language**:

> [!IMPORTANT]
> **The Golden Rule**: If a contract, schema, parameter, or value can be expressed with static compile-time syntax, it **MUST** be expressed with syntax. Never defer structural guarantees to runtime dictionaries, dynamic lookups, or duck typing.

#### 1. The 8 Static Typing Commandments:

1. **100% Explicit Type Hints on All Boundaries (PEP 484 & PEP 604)**:
   - Every function parameter, return value, class attribute, generator yield, and module constant must have explicit static types.
   - Never rely on implicit type inference for public interfaces.
   - Use modern union syntax `T | None` instead of legacy `Optional[T]`.

2. **Ban on Untyped Dictionaries (`dict[Any, Any]` / raw `dict`)**:
   - Passing raw dictionaries across function boundaries or returning untyped maps is strictly forbidden.
   - Inbound payloads must be Pydantic `BaseModel` instances.
   - Outbound payloads must inherit from `GenericSchema`.
   - Internal structured mappings must use `TypedDict` with total type safety.

3. **`Literal[...]` & `Enum` Over Raw Strings**:
   - For finite sets of values (e.g. sort directions, status flags, user roles), raw strings like `"asc"` or `"admin"` are forbidden as loose parameter types.
   - Use `Literal["asc", "desc"]`, `Literal["pic", "admin"]`, or `StrEnum` to give the IDE and type-checker complete exhaustiveness checking.

4. **Zero-Tolerance for `Any`**:
   - `Any` bypasses static type verification and is strictly banned.
   - If an untyped third-party library forces the use of `Any`, an explicit inline comment explaining why is mandatory:
     ```python
     # Reason: 3rd-party lib 'external_sdk' does not publish PEP 561 stubs
     untyped_data: Any = external_sdk.raw_payload()
     ```

5. **Mandatory Generic Specialization**:
   - Never write bare generic types. Always specialize them explicitly:
     - `ResponseSchema[ListSchema[EventSchema]]` (NOT bare `ResponseSchema`)
     - `list[str]` (NOT bare `list`)
     - `dict[str, int]` (NOT bare `dict`)
     - `Generator[Session, None, None]` (NOT bare `Generator`)

6. **Static Attribute Access Over Dynamic Introspection**:
   - Do NOT use dynamic `getattr()`, `setattr()`, `hasattr()`, or string-keyed dictionary subscripting `payload["key"]` when attributes can be statically declared and accessed via `payload.key`.
   - Access middleware state via typed helper functions or explicit property getters.

7. **Strict Return Typing on API Endpoints**:
   - Every router endpoint function MUST declare both `response_model=ResponseSchema[...]` on the decorator AND `-> ResponseSchema[...]:` on the function signature. Never return raw dictionaries or untyped objects from endpoints.

8. **Exception Chaining with `from` (B904)**:
   - When re-raising or transforming exceptions in an `except` block, always use `raise NewException(...) from err` or `raise NewException(...) from None` to preserve or explicitly cleanly sever the traceback.

---

#### 2. Side-by-Side: Dynamic Anti-Pattern vs. Strict Syntax Standard

| Area                   | ❌ Anti-Pattern (Dynamic / Untyped)        | ✅ Standard (Syntax Over Dynamic Values)                                                   |
| :--------------------- | :----------------------------------------- | :----------------------------------------------------------------------------------------- | ------------ |
| **Endpoint Signature** | `def get_user(user_id):`                   | `def get_user(user_id: str, db: Session = Depends(get_db)) -> ResponseSchema[UserSchema]:` |
| **Endpoint Return**    | `return {"message": "ok", "user": user}`   | `return ResponseSchema(data=UserSchema.model_validate(user))`                              |
| **Query Parameters**   | `search=None, page=1, limit=20`            | `params: PaginationRequest = Depends()`                                                    |
| **Parameter Options**  | `sort_order: str = "asc"`                  | `sort_order: Literal["asc", "desc"] = "asc"`                                               |
| **Database Model**     | `id = Column(String)`                      | `id: str = Field(default_factory=lambda: str(uuid.uuid4()), primary_key=True)`             |
| **Error Handling**     | `return JSONResponse({"error": "Failed"})` | `response_body = ResponseSchema[None](message=msg, data=None)`                             |
| **Generic List**       | `items: list = []`                         | `items: list[ItemSchema] = []`                                                             |
| **Optional Values**    | `user_name = None`                         | `user_name: str                                                                            | None = None` |

---

#### 3. Ruff Ruleset: The Python Counterpart to `flutter_lints`

Just as Flutter uses `flutter_lints` or `analysis_options.yaml` with strict analyzer rules, our Python repository enforces strict Ruff linter and formatter rules:

```toml
# pyproject.toml
[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = [
    "E",     # pycodestyle errors
    "W",     # pycodestyle warnings
    "F",     # Pyflakes (syntax bugs, undefined names, unused variables)
    "I",     # isort (strict, deterministic import grouping & alphabetization)
    "UP",    # pyupgrade (modern Python 3.11+ idioms, union pipes, format strings)
    "B",     # flake8-bugbear (common design bugs, dangerous default arguments)
    "SIM",   # flake8-simplify (boolean simplification, nested if collapsing)
    "RUF",   # Ruff-specific rules (no unused noqa, explicit format specifiers)
]
ignore = [
    "E501",  # Line length handled cleanly by formatter and inline comments
]

[tool.mypy]
strict = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_return_any = true
```

#### 4. Mandatory Pre-Commit Linter Verification:

- Always run `uv run ruff check .` inside the project virtual environment before finalizing code or marking any task complete.
- **Zero-Warning Tolerance**: Fix all warnings reported by Ruff immediately — do not suppress warnings with `# noqa` unless mathematically or architecturally impossible to resolve cleanly.

---

### 3.4 Core Base Abstractions & General Models

In accordance with our engineering standards, Python projects maintain a standardized set of reusable base abstractions and general models across both the presentation (`schemas/`), request validation (`requests/`), and persistence (`models/`) layers:

#### 1. `GenericSchema` (Foundational Pydantic Base):

Every outbound schema in the application **MUST inherit from `GenericSchema`**. It provides an automatic `@computed_field` `schema_id` property reflecting the schema class name, ensuring self-documenting JSON serializations across all client consumers:

```python
from pydantic import BaseModel, computed_field


class GenericSchema(BaseModel):
    """
    Base structural layout. Pydantic will automatically include computed_field
    properties inside serialized outputs like model_dump() and model_dump_json().
    """

    @computed_field
    @property
    def schema_id(self) -> str:
        """The name identifier of the schema."""
        return self.__class__.__name__
```

#### 2. Envelope Wrapper: `ResponseSchema[T]`:

Standard top-level envelope wrapper for all API responses (success and handled errors alike), enforcing a deterministic JSON contract across the application:

```python
from typing import Generic, TypeVar
from pydantic import Field
from app import __version__
from app.schemas.generic_schema import GenericSchema

T = TypeVar("T")


class ResponseSchema(GenericSchema, Generic[T]):
    """Standardized API response envelope wrapper."""

    version: str = Field(
        default=__version__,
        description="The backend version currently deployed.",
    )
    message: str = Field(
        default="API is running successfully",
        description="A human-readable message summarizing the result of the request.",
    )
    data: T | None = Field(
        ...,
        description="The main payload of the response.",
    )
```

#### 3. Paginated Collection Wrapper: `ListSchema[T]`:

Standard collection envelope returned whenever an endpoint delivers a paginated chunk of items:

```python
from typing import Generic
from pydantic import Field
from app.schemas.generic_schema import GenericSchema
from app.schemas.pagination_schema import PaginationSchema
from app.schemas.response_schema import T
from app.schemas.user_schema import UserSchema


class ListSchema(GenericSchema, Generic[T]):
    """Standard unified pagination envelope across the application."""

    items: list[T] = Field(description="A list containing the current page chunk slice.")
    pagination: PaginationSchema = Field(description="Pagination indexing metadata envelope.")
    user: UserSchema | None = Field(default=None, description="Details of the authenticated user if applicable.")
```

#### 4. Mathematical Pagination Metadata: `PaginationSchema`:

Encapsulates offset pagination tracking metadata and provides a zero-boilerplate factory `.create(...)`:

```python
from math import ceil
from pydantic import Field
from app.schemas.generic_schema import GenericSchema


class PaginationSchema(GenericSchema):
    """Standard offset pagination tracking envelope."""

    total_items: int = Field(description="The grand total number of records matching this query.")
    page: int = Field(description="The current active page index (1-based).")
    limit: int = Field(description="The maximum chunk size of items per page.")
    total_pages: int = Field(description="The mathematically calculated ceiling limit of maximum available pages.")
    has_more: bool = Field(description="Boolean flag indicating whether further pages exist.")

    @classmethod
    def create(cls, total_items: int, page: int, limit: int) -> "PaginationSchema":
        """Convenience factory method to calculate total_pages and has_more."""
        safe_limit = max(1, limit)
        safe_page = max(1, page)
        total_pages = ceil(total_items / safe_limit)

        return cls(
            total_items=total_items,
            page=safe_page,
            limit=safe_limit,
            total_pages=total_pages,
            has_more=safe_page < total_pages,
        )
```

#### 5. Reusable Query Pagination & Filter Model: `PaginationRequest`:

Standard reusable request parameters for all paginated GET endpoints (`app/requests/pagination_request.py`):

```python
from typing import Literal
from pydantic import BaseModel, Field


class PaginationRequest(BaseModel):
    """Standard reusable request parameters for paginated GET endpoints."""

    page: int = Field(default=1, ge=1, description="Page index (1-based)")
    limit: int = Field(default=20, ge=1, le=100, description="Items per page")
    search: str | None = Field(default=None, description="Optional search filter text")
    sort_by: str | None = Field(default=None, description="Field to sort results by")
    sort_order: Literal["asc", "desc"] = Field(default="asc", description="Sorting direction: 'asc' or 'desc'")
```

#### 6. Structured Error Model: `ErrorDetailSchema`:

Standard schema for granular validation errors and diagnostic payloads (`app/schemas/error_schema.py`):

```python
from pydantic import Field
from app.schemas.generic_schema import GenericSchema


class ErrorDetailSchema(GenericSchema):
    """Standardized structural error payload for validation and domain exceptions."""

    loc: list[str] | None = Field(default=None, description="The structural location of the error in the request.")
    msg: str = Field(description="A human-readable description of the error.")
    type: str | None = Field(default=None, description="The error type identifier or categorization code.")
```

#### 7. General Database Entity Blueprint: `BaseDBModel`:

Abstract SQLModel base class providing standardized UUID4 primary key `id` and `created_at` timestamp tracking for all relational models (`app/models/base_model.py`):

```python
import uuid
from datetime import UTC, datetime
from sqlmodel import Field, SQLModel


class BaseDBModel(SQLModel):
    """
    Standard base database model blueprint providing common UUID4 primary key and creation timestamp.
    """

    id: str = Field(
        default_factory=lambda: str(uuid.uuid4()),
        primary_key=True,
        index=True,
        max_length=255,
        description="The unique system identifier, automatically generated as a UUID4 string.",
    )

    created_at: datetime = Field(
        default_factory=lambda: datetime.now(UTC),
        description="The UTC timestamp when this entity was created.",
    )
```

#### 8. Action & Empty Response Pattern (`ResponseSchema[None]`):

For endpoints that perform an action (e.g. logout, revoke, delete) without returning a payload entity, use `ResponseSchema[None]`:

```python
@router.post("/logout", response_model=ResponseSchema[None])
def logout() -> ResponseSchema[None]:
    return ResponseSchema(message="User logged out successfully.", data=None)
```

#### 9. Domain Schemas (Extending `GenericSchema`):

All domain-specific schemas extend `GenericSchema`:

```python
from datetime import datetime
from app.schemas.generic_schema import GenericSchema


class UserSchema(GenericSchema):
    id: str
    email: str
    name: str
    role: str
    company_name: str | None = None
    created_at: datetime | None = None
```

---

### 3.5 Database Layer & Models (`models/` & `cores/database.py`)

#### 1. SQLModel Table Definitions:

- Must inherit `SQLModel, table=True` (or extend `BaseDBModel`).
- Define explicit `__tablename__ = "stellar_..."`.
- Primary keys use UUID4 string factory: `id: str = Field(default_factory=lambda: str(uuid.uuid4()), primary_key=True, index=True, max_length=255)`.
- Use `Field(...)` with explicit types, max lengths, indexes, and descriptive `description` docstrings.
- Foreign keys use `sa_column=Column(String(255), ForeignKey("target_table.id", ondelete="CASCADE"), nullable=...)`.

```python
import uuid
from datetime import UTC, datetime
from sqlalchemy import Column, ForeignKey, String
from sqlmodel import Field, SQLModel, UniqueConstraint


class UserModel(SQLModel, table=True):
    __tablename__ = "stellar_users"
    __table_args__ = (UniqueConstraint("email", "event_id", name="uq_user_email_event"),)

    id: str = Field(
        default_factory=lambda: str(uuid.uuid4()),
        primary_key=True,
        index=True,
        max_length=255,
        description="The unique system identifier for this user (UUID4 string).",
    )
    email: str = Field(index=True, max_length=255, description="The user's unique login email address.")
    password_hash: str = Field(max_length=255, description="Bcrypt hashed password.")
    name: str = Field(default="", max_length=255, description="The user's full name.")
    role: str = Field(default="pic", max_length=50, description="User authorization role (pic or admin).")
    event_id: str | None = Field(
        default=None,
        sa_column=Column(String(255), ForeignKey("stellar_events.id", ondelete="CASCADE"), nullable=True, index=True),
        description="The event identifier this user belongs to.",
    )
    created_at: datetime = Field(
        default_factory=lambda: datetime.now(UTC),
        description="The UTC timestamp when this user was registered.",
    )
```

#### 2. Engine, Lifecycle Initialization & Session Provider (`cores/database.py`):

- Connection engine with connection pooling: `pool_pre_ping=True`, `pool_recycle=3600`.
- `init_db()` table builder: imports all models BEFORE calling `SQLModel.metadata.create_all(engine)` to ensure all tables are registered in memory.
- `get_db()` FastAPI dependency:

```python
from collections.abc import Generator
from sqlmodel import Session, SQLModel, create_engine
from app import env

engine = create_engine(
    env.DATABASE_URL,
    pool_pre_ping=True,
    pool_recycle=3600,
    echo=False,
)


def init_db() -> None:
    # 1. Import ALL models first to register metadata
    from app.models.event_model import EventModel  # noqa: F401
    from app.models.user_model import UserModel    # noqa: F401

    # 2. Build tables
    SQLModel.metadata.create_all(engine)


def get_db() -> Generator[Session, None, None]:
    """FastAPI Dependency Injection Provider."""
    with Session(engine) as session:
        yield session
```

---

### 3.6 Core Configuration & Security (`cores/`)

#### 1. Environment Singleton (`cores/environment.py`):

- Uses `pydantic_settings.BaseSettings` and `SettingsConfigDict`.
- Uses `@model_validator(mode="before")` to strip surrounding quotes from env variables.
- Instantiates a singleton `env = Environment()`.
- **Never open or inspect raw `.env` files**: access configuration only via typed attributes on `env`!

```python
import os
from pydantic import model_validator
from pydantic_settings import BaseSettings, SettingsConfigDict


class Environment(BaseSettings):
    PROJECT_NAME: str = "Stellar™"
    DATABASE_URL: str
    JWT_SECRET_KEY: str
    PORT: int = 8000
    DEBUG_MODE: bool = False

    model_config = SettingsConfigDict(
        env_file=os.getenv("ENV_FILE", ".env"),
        env_file_encoding="utf-8",
        extra="ignore",
    )

    @model_validator(mode="before")
    @classmethod
    def strip_quotes(cls, data: object) -> object:
        if isinstance(data, dict):
            return {k: v.strip("'\"") if isinstance(v, str) else v for k, v in data.items()}
        return data


env = Environment()
```

#### 2. Security & Token Verification (`cores/security.py`):

- Password hashing with `bcrypt.hashpw` and `bcrypt.checkpw`.
- Access token encoding with UTC expirations (e.g. 15 minutes) and token versions for revocation.
- Token decoding with explicit `expected_type` verification ("access" vs "refresh").

---

### 3.7 Inbound Request Models (`requests/`)

Separate incoming input validation models (`requests/`) from outgoing presentation schemas (`schemas/`):

- Inbound payloads extend `pydantic.BaseModel`.
- Provide sensible defaults, field constraints (`ge`, `le`), and clean Swagger documentation.

```python
from pydantic import BaseModel, Field


class PaginationRequest(BaseModel):
    """Standard reusable request parameters for paginated GET endpoints."""

    page: int = Field(default=1, ge=1, description="Page index (1-based)")
    limit: int = Field(default=20, ge=1, le=100, description="Items per page")
    search: str | None = Field(default=None, description="Optional search filter text")
    sort_by: str | None = Field(default=None, description="Field to sort results by")
    sort_order: str = Field(default="asc", description="Sorting direction: 'asc' or 'desc'")
```

---

### 3.8 Router Conventions (`routers/`)

Routers orchestrate request parsing, service dispatching, database queries, and response envelopes:

1. **Explicit Return Typing**: Always declare `response_model=ResponseSchema[...]` on decorator and return type hint on the function.
2. **Dependency Injection**: Inject `db: Session = Depends(get_db)` and query models `params: PaginationRequest = Depends()`.
3. **Role Guards**: Validate role state injected into `request.state` by the middleware.
4. **Rich Docstrings**: Detail sorting fields, filters, and endpoint behavior for Swagger UI.

```python
from fastapi import APIRouter, Depends, HTTPException, Request, status
from sqlmodel import Session, func, select
from app.cores.database import get_db
from app.models.event_model import EventModel
from app.requests import PaginationRequest
from app.schemas import EventSchema, ListSchema, PaginationSchema, ResponseSchema

router = APIRouter(prefix="/api/v1/events", tags=["Events"])


@router.get("", response_model=ResponseSchema[ListSchema[EventSchema]])
def list_events(
    params: PaginationRequest = Depends(),
    db: Session = Depends(get_db),
) -> ResponseSchema[ListSchema[EventSchema]]:
    """
    Returns a paginated list of all active events.

    **Sorting Fields (`sort_by`):**
    * `created_at` (default)
    * `name`
    """
    query = select(EventModel)
    if params.search:
        query = query.where(EventModel.name.contains(params.search))

    total_items = db.exec(select(func.count(EventModel.id))).one()
    events = db.exec(query.offset((params.page - 1) * params.limit).limit(params.limit)).all()

    items = [
        EventSchema(
            id=e.id,
            name=e.name,
            venue_name=e.venue_name,
            created_at=e.created_at,
        )
        for e in events
    ]

    return ResponseSchema(
        message="Events retrieved successfully.",
        data=ListSchema(
            items=items,
            pagination=PaginationSchema.create(
                total_items=total_items,
                page=params.page,
                limit=params.limit,
            ),
        ),
    )
```

---

### 3.9 Middlewares & Interceptors (`middlewares/`)

Use `starlette.middleware.base.BaseHTTPMiddleware` to guard routes, authenticate JWT/API keys, and populate `request.state`:

- **Global Route Exclusions**: Skip middleware checks for OPTIONS requests, WebSocket upgrades, docs (`EXCLUDED_ROUTES`), and static assets (`EXCLUDED_PREFIXES`).
- **Context Injection**: Attach authenticated identifiers (`request.state.user_id`, `request.state.user_role`) directly to request state.
- **Swagger Documentation Companion Scheme**: Define an `HTTPBearerToken(HTTPBearer)` with `auto_error=False` so OpenAPI exhibits the interactive "Authorize" lock without colliding with middleware logic.

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from starlette.middleware.base import BaseHTTPMiddleware, RequestResponseEndpoint
from starlette.requests import HTTPConnection
from starlette.responses import Response
from app import EXCLUDED_PREFIXES, EXCLUDED_ROUTES, ResponseSchema
from app.cores.security import decoder


class HTTPBearerToken(HTTPBearer):
    async def __call__(self, request: HTTPConnection = None) -> HTTPAuthorizationCredentials | None:
        if request is None:
            return None
        return await super().__call__(request)


jwt_scheme = HTTPBearerToken(
    bearerFormat="JWT",
    auto_error=False,
    description="Enter your securely generated JWT token.",
)


class JWTMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next: RequestResponseEndpoint) -> Response:
        path = request.url.path

        if request.method == "OPTIONS":
            return await call_next(request)

        if path in EXCLUDED_ROUTES or path.startswith(EXCLUDED_PREFIXES):
            return await call_next(request)

        # Authenticate header
        auth_header = request.headers.get("Authorization")
        if not auth_header or not auth_header.startswith("Bearer "):
            body = ResponseSchema[None](message="Authorization Bearer token required.", data=None)
            return JSONResponse(status_code=401, content=body.model_dump())

        token = auth_header.split(" ", 1)[1].strip()
        try:
            user_id, token_version = decoder(token, expected_type="access")
            request.state.user_id = user_id
            request.state.token_version = token_version
        except Exception as e:
            body = ResponseSchema[None](message=f"Invalid or expired token: {e}", data=None)
            return JSONResponse(status_code=401, content=body.model_dump())

        return await call_next(request)
```

---

### 3.10 Centralized Exception Handling (`exceptions/`)

Never leak raw stack traces or default FastAPI error responses to clients. Attach centralized exception handlers in `main.py` that wrap errors into standard `ResponseSchema` envelopes:

```python
# app/exceptions/http_exception.py
from fastapi import Request
from fastapi.responses import JSONResponse
from starlette.exceptions import HTTPException
from app import ResponseSchema


async def http_exception(request: Request, exc: HTTPException) -> JSONResponse:
    """Wraps explicit HTTPExceptions into standard ResponseSchema contract."""
    response_body = ResponseSchema[None](message=str(exc.detail), data=None)
    return JSONResponse(status_code=exc.status_code, content=response_body.model_dump())


# app/exceptions/validation_exception.py
from fastapi import Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from app import ResponseSchema


async def validation_exception(request: Request, exc: RequestValidationError) -> JSONResponse:
    """Wraps request validation errors into a clean, unified schema."""
    errors = [{"loc": list(err["loc"]), "msg": err["msg"], "type": err["type"]} for err in exc.errors()]
    response_body = ResponseSchema[list[dict]](message="Validation Error", data=errors)
    return JSONResponse(status_code=422, content=response_body.model_dump())
```

Register handlers in `main.py`:

```python
app.add_exception_handler(RateLimitExceeded, rate_limit_exception)
app.add_exception_handler(HTTPException, http_exception)
app.add_exception_handler(RequestValidationError, validation_exception)
app.add_exception_handler(Exception, general_exception)
```

---

### 3.11 Services & Background Processing (`services/`)

Decouple external integrations and complex business logic from the HTTP router layer into dedicated service classes:

- Service methods operate independently of HTTP status codes or request context.
- Execute long-running tasks asynchronously using FastAPI `BackgroundTasks` or async scheduler loops (`asyncio.create_task`).

```python
import logging
from app.cores.environment import env

logger = logging.getLogger(__name__)


class EmailService:
    @staticmethod
    def send_email(to: str, subject: str, body: str) -> bool:
        if not env.EMAIL_ENDPOINT or not env.EMAIL_API_KEY:
            logger.error("Email service not configured.")
            return False
        # Perform HTTP request to outbound transactional email microservice
        return True
```

---

### 3.12 Swagger UI & OpenAPI Customization Template (`utils/api_documentation.py`)

A custom OpenAPI post-processor cleans up FastAPI schema generation, giving Swagger UI a clean, enterprise-grade layout:

1. **Replaces mangled generics**: Converts `ResponseSchema_ListSchema_SeatSchema___` to clean bracket notation `ResponseSchema[ListSchema[SeatSchema]]`.
2. **Groups schemas visually**: Prefixes schemas with `Request > ` and `Response > ` to organize them into namespaces in the Swagger UI schema tray.
3. **Normalizes `schema_id` examples**: Strips prefixes to show the clean schema name as the example value.
4. **Removes redundant 422 schemas**: Strips default 422 documentation so endpoints focus on documented contracts.
5. **Renames raw body form schemas**: Normalizes `Body_import_...` into clean names like `DataImportRequest`.

```python
import json
from collections.abc import Callable
from typing import Any
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi


def format_schema_name(name: str) -> str:
    """Reformats mangled generic schema names into clean bracket notations."""
    if "_" not in name:
        return name
    if "NoneType" in name:
        name = name.replace("NoneType", "None")
    if name.endswith("__"):
        name = name[:-2] + "]]"
    elif name.endswith("_"):
        name = name[:-1] + "]"
    if name.startswith("ResponseSchema_"):
        name = name.replace("ResponseSchema_", "ResponseSchema[", 1)
    if "ListSchema_" in name:
        name = name.replace("ListSchema_", "ListSchema[", 1)
    return name


def custom_openapi(app: FastAPI) -> Callable[[], dict[str, Any]]:
    def openapi() -> dict[str, Any]:
        if app.openapi_schema:
            return app.openapi_schema

        schema = get_openapi(
            title=app.title,
            version=app.version,
            description=app.description,
            routes=app.routes,
        )

        if "components" in schema and "schemas" in schema["components"]:
            schemas = schema["components"]["schemas"]
            schemas.pop("HTTPValidationError", None)
            schemas.pop("ValidationError", None)

            # Format bracket names and namespaces (Request > ... / Response > ...)
            # ...
            schema["components"]["schemas"] = {k: schemas[k] for k in sorted(schemas.keys())}

        app.openapi_schema = schema
        return app.openapi_schema

    return openapi
```

Attach to app in `main.py`:

```python
app.openapi = custom_openapi(app)
```

---

### 3.13 Unit Testing Standards (`tests/`)

- Use `unittest.TestCase` with strict type annotations on test methods (`def test_*(self) -> None:`).
- Mock database sessions with `MagicMock(spec=Session)` and query execution with `.side_effect` or `.return_value`.
- Mock external service calls and security functions using `unittest.mock.patch`.
- Assert both envelope attributes (`response.message`, `response.data`) and domain-level invariants.

```python
import unittest
from unittest.mock import MagicMock
from sqlmodel import Session
from app.models.event_model import EventModel
from app.routers.event_router import list_events
from app.requests import PaginationRequest


class TestEventEndpoints(unittest.TestCase):
    def test_list_events_success(self) -> None:
        db_mock = MagicMock(spec=Session)
        mock_event = EventModel(id="event-1", name="Stellar Workplace Award® 2026")

        exec_mock = MagicMock()
        exec_mock.all.return_value = [mock_event]
        exec_mock.one.return_value = 1
        db_mock.exec.return_value = exec_mock

        response = list_events(params=PaginationRequest(), db=db_mock)

        self.assertEqual(response.message, "Events retrieved successfully.")
        self.assertIsNotNone(response.data)
        self.assertEqual(len(response.data.items), 1)
```

---

## 4. Key Cross-Project Conventions Summary

| Area                       | Flutter / Dart Convention                                | Python (FastAPI / SQLModel) Convention                                                             |
| :------------------------- | :------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| **Architecture**           | Modular MVC (`lib/src/shared/` & `lib/src/<feature>/`)   | Layered Service (`app/cores`, `models`, `schemas`, `requests`, `routers`, `services`)              |
| **Typing Philosophy**      | Statically compiled, zero dynamic types                  | **Syntax Over Dynamic Values**: 100% PEP 484 type hints, `Literal`, `Generic[T]`, no untyped dicts |
| **Linting & Code Quality** | `flutter_lints` / strong mode in `analysis_options.yaml` | `uv run ruff check .` (E, F, I, UP, B, SIM, RUF) & `mypy --strict` with zero warnings              |
| **Model / Entity Base**    | `DModel` (`dart_fusion_flutter`)                         | `BaseDBModel` / `SQLModel, table=True` & `GenericSchema`                                           |
| **Response Contract**      | `GeneralGetStates<T>` envelope                           | `ResponseSchema[T]` and `ListSchema[T]` envelopes with computed `schema_id`                        |
| **Request Validation**     | Dart constructor validation & tear-offs                  | Pydantic `BaseModel` (`app/requests/`) with typed Field constraints & `Literal`                    |
| **Serialization**          | `static fromJSON(JSON)` & `toJSON`                       | Pydantic `.model_dump()` / `.model_dump_json()` & SQLModel mappings                                |
| **Module Exports**         | `library <feature>;` with `part '...'`                   | Explicit `__init__.py` with typed `__all__ = [...]`                                                |
| **Database / Client**      | `GeneralRepository` encapsulating typed `Dio`            | SQLModel `Session` via `get_db` FastAPI dependency injection                                       |
| **Error Handling**         | `FeatureError` with message and retry logic              | Centralized exception handlers wrapping errors in `ResponseSchema`                                 |
| **Interceptors / Guards**  | Custom Dio interceptors                                  | `BaseHTTPMiddleware` with route exclusions & `HTTPBearerToken` scheme                              |
| **Documentation**          | dartdoc with static tear-offs                            | Custom OpenAPI post-processor with generic bracket notation & schema grouping                      |
| **Testing**                | `factory .test({bool random = true})`                    | `unittest.TestCase` with typed test methods & `MagicMock(spec=Session)`                            |
| **Security**               | Zero-tolerance on secrets: never read or output `.env*`  | Zero-tolerance on secrets: use typed `env = Environment()` singleton                               |
| **Scratchpad**             | Full autonomy inside `scratch/`                          | Full autonomy inside `scratch/`                                                                    |

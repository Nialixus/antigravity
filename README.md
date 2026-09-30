# Antigravity Agent Rules: Modular MVC Standard

> Production-ready agent rules, security policies, and Modular MVC architecture templates for Google Antigravity & Gemini agents in Flutter/Dart.

Modeled after the Modular MVC architecture established in [Nialixus/flutter_reqres](https://github.com/Nialixus/flutter_reqres).

---

## 🚀 Setup

Antigravity automatically discovers and loads rules from markdown files inside `rules/` directories.

### 1. Global Setup (Applies to all your projects)

Install to your global Antigravity config directory:

```bash
mkdir -p ~/.gemini/config/rules
curl -fsSL https://raw.githubusercontent.com/<YOUR_USERNAME>/<YOUR_REPO>/main/RULES.md \
  -o ~/.gemini/config/rules/coding-style.md
```

_(Or symlink it locally)_:

```bash
ln -s "$(pwd)/RULES.md" ~/.gemini/config/rules/coding-style.md
```

---

### 2. Repository Setup (Per-project / Team)

Add to the `.agents/rules/` directory in your project root:

```bash
mkdir -p .agents/rules
cp RULES.md .agents/rules/coding-style.md
```

> **Note**: Antigravity also supports placing `RULES.md` directly in the project root. Both approaches are valid, but `.agents/rules/coding-style.md` keeps the project root clean.

---

## 📌 Summary of Rules

- **🛡️ Ironclad Security**: Zero-tolerance on `.env*`, certs, and credential files (skip & never read/output).
- **⚡ Scratchpad Autonomy**: Full freedom to create, edit, run, and delete files inside `scratch/` or `tmp/` without prompts.
- **🏗️ Modular MVC**: Feature-first folders in `lib/src/<feature>/` (`models/`, `views/`, `controllers/`) and `lib/src/shared/` using `library` and `part` / `part of`.
- **📦 DModel Standard**: Models extend `DModel`, with `static fromJSON(JSON value)` (primitives without fallback; lists/maps with fallback), `toJSON`, `copyWith`, and `factory .test()`.
- **🎨 Clean Views**:
  - `StatelessWidget` for pure UI display components.
  - `StatefulWidget` for pages managing controller lifecycle (`initState`/`dispose`).
  - `_FeaturePageState` is the **only** allowed underscore naming.
  - DRY context extensions (`context.text.*`, `context.color.*`) instead of manual `Theme.of(context)`.
- **🔄 Controllers & typed Dio**: `GeneralCubit` with sealed states (`Loading` with `.test()` data, `Success`, `Error`) and typed `Dio` in `GeneralRepository`.

---

## 📄 License

MIT License.

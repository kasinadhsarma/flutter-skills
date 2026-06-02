---
name: flutter-barrel-imports
description: >
  Apply the barrel file pattern to clean up and organize Flutter/Dart imports.
  Use this skill whenever the user has messy, deeply-nested, or repetitive import
  sections in Flutter files; wants to consolidate imports across models, services,
  or widgets; asks how to organize imports in a Flutter project; mentions
  the barrel file pattern, barrel exports, index.dart, Single Import, or a tool
  like "Barrel Me" or "single_import_generator". Also triggers for questions about
  scalable import architecture in Dart/Flutter codebases, merge conflicts caused
  by import churn, or reducing boilerplate at the top of every Flutter screen file.
---

# Flutter Barrel Imports Skill

A skill for cleaning up Flutter/Dart import sections using the **barrel file pattern** —
consolidating multiple `import` statements into a single re-export entry point.
Two naming styles exist: folder-named barrels (`models.dart`) and the
`index.dart` convention. Both are covered here.

---

## Why Imports Become a Problem

As Flutter apps grow, import blocks balloon into long, hard-to-maintain lists:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:soon_sak/app/config/color_config.dart';
import 'package:soon_sak/app/config/font_config.dart';
import 'package:soon_sak/presentation/base/base_screen.dart';
import 'package:soon_sak/presentation/common/skeleton_box.dart';
import 'package:soon_sak/presentation/screens/home/home_view_model.dart';
// ... 20+ more lines before the actual class
```

**Pain points:**
- Repetitive boilerplate every time a new screen/widget is created
- Unnecessary Git merge conflicts when import lines change across branches
- Unused or duplicate imports accumulate and hurt readability

> **Tip:** Run `dart fix --apply` to auto-remove unused import statements.

---

## Approach 1 — Folder-Named Barrel Files (`models.dart`)

Name the barrel after its folder. Best for layer-based architectures
(models / services / widgets).

### Step 1 — Create per-folder barrel files

```dart
// lib/models/models.dart
export 'user.dart';
export 'post.dart';
export 'comment.dart';
```

```dart
// lib/services/services.dart
export 'api_service.dart';
export 'auth_service.dart';
```

```dart
// lib/widgets/widgets.dart
export 'custom_button.dart';
export 'custom_card.dart';
export 'loading_indicator.dart';
```

### Step 2 — (Optional) Create a top-level core barrel

```dart
// lib/core/core.dart
export 'models/models.dart';
export 'services/services.dart';
export 'widgets/widgets.dart';
```

### Step 3 — Replace scattered imports with one line

```dart
// Before
import 'package:myapp/models/user.dart';
import 'package:myapp/models/post.dart';
import 'package:myapp/services/api_service.dart';
// ...

// After
import 'package:myapp/core/core.dart';
```

---

## Approach 2 — `index.dart` (Single Import Convention)

Use `index.dart` as the barrel name — a convention common in projects
that mix internal and external package exports in one place.
Ideal for a `utilities/` or `app/` root that aggregates everything
a screen typically needs.

### Create `lib/utilities/index.dart`

```dart
// lib/utilities/index.dart

// internal app modules
export 'package:myapp/app/config/color_config.dart';
export 'package:myapp/app/config/font_config.dart';
export 'package:myapp/app/config/app_insets.dart';
export 'package:myapp/utilities/extensions/string_extension.dart';

// frequently used external packages
export 'package:flutter/material.dart';
export 'package:provider/provider.dart';
export 'package:intl/intl.dart';
```

### Import in any screen with one line

```dart
import 'package:myapp/utilities/index.dart';

class HomeScreen extends StatelessWidget {
  // color_config, font_config, material — all available
}
```

### Structured `index.dart` per layer

For layered projects, place an `index.dart` in each layer folder:

```
lib/
├── utilities/
│   └── index.dart       ← config, extensions, helpers
├── domain/
│   └── index.dart       ← models, repositories (interfaces)
├── data/
│   └── index.dart       ← data sources, repository impls
└── presentation/
    └── index.dart       ← base classes, common widgets
```

Each screen then imports only what its layer needs:

```dart
import 'package:myapp/utilities/index.dart';
import 'package:myapp/presentation/index.dart';
```

---

## Naming Convention Comparison

| Style | Barrel file name | Best for |
|---|---|---|
| Folder-named | `models/models.dart` | Layer-based architecture |
| index.dart | `utilities/index.dart` | Mixed internal + external exports |
| Feature barrel | `auth/auth.dart` | Feature-first architecture |

**Conflict rule:** If a folder already has a non-barrel file with the same name,
rename the barrel to `{name}_barrel.dart` or fall back to `index.dart`.

---

## What to Exclude from Any Barrel File

- `main.dart` — app entry point, never re-exported
- Dart `part` files — belong to a specific library, cannot be re-exported
- Existing barrels at the same level (avoid circular exports)
- Generated files (`*.g.dart`, `*.freezed.dart`) — import them directly

---

## Automating Barrel Generation

### Option A — `single_import_generator` (pub.dev package)

Generates `index.dart` files from the CLI. Add to `dev_dependencies`:

```yaml
dev_dependencies:
  single_import_generator: ^<latest>
```

**Generate for all files in a directory (recursive):**
```bash
dart run single_import_generator -target=lib/presentation all
```

**Generate for immediate files only (non-recursive):**
```bash
dart run single_import_generator -target=lib/presentation/common dir
```

**Generate for files annotated with `@SingleImport`:**
```dart
@SingleImport()
class FrequentlyUsedClass { ... }

@SingleImport()
extension SomeStringExtension on String { ... }
```
```bash
dart run single_import_generator -path=lib/utilities
```

The `@SingleImport` approach is best when only selected classes/extensions
should be part of the barrel — not every file in the folder.

### Option B — "Barrel Me" (VS Code Extension)

Right-click any folder → **"Create Barrel"**. Generates a folder-named barrel
and optionally migrates all existing imports project-wide.

Key features:
- Flat or hierarchical barrel generation
- Auto-excludes `main.dart`, `part` files, existing barrels
- Renames conflicting files to `{name}_barrel.dart`
- One-click import migration across the whole project
- Zero configuration required

> Search **"Barrel Me"** in the VS Code Extensions Marketplace.

---

## Hierarchical Barrel Architecture (large codebases)

```
lib/
├── core/
│   ├── core.dart              ← top-level barrel (re-exports sub-barrels)
│   ├── models/
│   │   └── models.dart
│   ├── services/
│   │   └── services.dart
│   └── widgets/
│       └── widgets.dart
├── utilities/
│   └── index.dart             ← config + extensions + external packages
└── features/
    └── auth/
        ├── auth.dart          ← feature barrel
        ├── auth_screen.dart
        └── auth_controller.dart
```

---

## When to Apply Which Pattern

| Situation | Recommendation |
|---|---|
| 3+ imports from the same folder | Create a folder-named barrel |
| Repetitive base imports on every screen | Create `utilities/index.dart` |
| Mixing external packages with internals | `index.dart` with both export types |
| Large layered app | Hierarchical barrels per layer |
| Feature-based monorepo | Per-feature barrel (`auth/auth.dart`) |
| Existing messy codebase | Barrel Me (VS Code) for one-click migration |
| CLI / CI workflow | `single_import_generator` with `all` or `dir` flag |

---

## Workflow When Helping a User

1. **Audit** — Look at current imports; identify groups sharing a folder prefix or
   repeated across multiple files.
2. **Choose style** — Folder-named barrel for layered arch; `index.dart` for
   utility/config aggregation; or both.
3. **Generate barrel files** — Write the `export` statements.
4. **Show before/after** — Demonstrate the import reduction in the consuming file.
5. **Advise on tooling** — Recommend `single_import_generator` for CLI workflows
   or Barrel Me for VS Code users.

---

## Quick Reference Templates

```dart
// Folder-named barrel: lib/<folder>/<folder>.dart
export '<file1>.dart';
export '<file2>.dart';
// omit: main.dart, *.g.dart, *.freezed.dart, part files

// index.dart barrel: lib/utilities/index.dart
export 'package:myapp/app/config/color_config.dart';
export 'package:myapp/utilities/extensions/string_ext.dart';
export 'package:flutter/material.dart';   // external packages OK here
export 'package:provider/provider.dart';
```

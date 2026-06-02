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

## How the Skill Activates

```mermaid
flowchart LR
    Trigger["User mentions:\n• messy imports\n• barrel files\n• index.dart\n• import churn\n• Barrel Me\n• single_import_generator"]
    Audit["Audit current imports\nidentify folder groups"]
    Choose["Choose pattern\nfolder-named or index.dart"]
    Generate["Generate barrel files\nexport statements"]
    Show["Show before / after\nto confirm reduction"]
    Tool["Advise on tooling\nBarrel Me or single_import_generator"]

    Trigger --> Audit --> Choose --> Generate --> Show --> Tool
```

---

## Why Imports Become a Problem

```mermaid
graph TD
    subgraph Before["Without barrels — every screen imports directly"]
        S1["home_screen.dart\n12 import lines"]
        S2["profile_screen.dart\n14 import lines"]
        S3["settings_screen.dart\n11 import lines"]

        S1 --> U1["user.dart"]
        S1 --> A1["api_service.dart"]
        S1 --> C1["color_config.dart"]
        S2 --> U1
        S2 --> A1
        S2 --> C1
        S3 --> U1
        S3 --> A1
        S3 --> C1
    end

    subgraph After["With barrels — one import per screen"]
        S4["home_screen.dart\n1 import line"]
        S5["profile_screen.dart\n1 import line"]
        S6["settings_screen.dart\n1 import line"]
        Barrel["core/core.dart\n(barrel)"]

        S4 --> Barrel
        S5 --> Barrel
        S6 --> Barrel
        Barrel --> U2["user.dart"]
        Barrel --> A2["api_service.dart"]
        Barrel --> C2["color_config.dart"]
    end
```

**Pain points without barrels:**
- Repetitive boilerplate every time a new screen/widget is created
- Unnecessary Git merge conflicts when import lines change across branches
- Unused or duplicate imports accumulate and hurt readability

> Run `dart fix --apply` to auto-remove unused import statements before migrating.

---

## Approach 1 — Folder-Named Barrel Files (`models.dart`)

Name the barrel after its folder. Best for **layer-based architectures** (models / services / widgets).

```mermaid
graph LR
    subgraph lib
        subgraph models
            U["user.dart"]
            P["post.dart"]
            C["comment.dart"]
            MB["models.dart\n(barrel)"]
        end
        subgraph services
            API["api_service.dart"]
            Auth["auth_service.dart"]
            SB["services.dart\n(barrel)"]
        end
        subgraph widgets
            Btn["custom_button.dart"]
            Card["custom_card.dart"]
            WB["widgets.dart\n(barrel)"]
        end
        Core["core/core.dart\n(top-level barrel)"]
        Screen["any_screen.dart\nimport core/core.dart"]
    end

    U & P & C --> MB
    API & Auth --> SB
    Btn & Card --> WB
    MB & SB & WB --> Core
    Core --> Screen
```

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

### Step 2 — Create a top-level core barrel (optional)

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

Use `index.dart` as the barrel name — common in projects that mix internal and external exports in one place. Ideal for a `utilities/` or `app/` root.

```mermaid
graph TD
    subgraph "index.dart aggregates everything a screen needs"
        Idx["utilities/index.dart"]
        subgraph Internal
            CC["color_config.dart"]
            FC["font_config.dart"]
            SE["string_extension.dart"]
        end
        subgraph External["External packages (re-exported)"]
            FM["flutter/material.dart"]
            PR["provider/provider.dart"]
            IT["intl/intl.dart"]
        end
    end

    CC & FC & SE --> Idx
    FM & PR & IT --> Idx
    Idx -->|"single import"| HS["HomeScreen"]
    Idx -->|"single import"| PS["ProfileScreen"]
```

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

```mermaid
graph LR
    subgraph "lib/ layers"
        UIdx["utilities/index.dart\nconfig · extensions · helpers"]
        DIdx["domain/index.dart\nmodels · repo interfaces"]
        DAIdx["data/index.dart\ndata sources · repo impls"]
        PIdx["presentation/index.dart\nbase classes · common widgets"]
    end

    Screen["any_screen.dart"] -->|"import utilities"| UIdx
    Screen -->|"import presentation"| PIdx
    PIdx --> DIdx
    DIdx --> DAIdx
```

```dart
import 'package:myapp/utilities/index.dart';
import 'package:myapp/presentation/index.dart';
```

---

## Hierarchical Barrel Architecture (large codebases)

```mermaid
graph TD
    Core["core/core.dart\ntop-level re-exports everything"]

    subgraph Layer Barrels
        MB["core/models/models.dart"]
        SB["core/services/services.dart"]
        WB["core/widgets/widgets.dart"]
    end

    subgraph Utilities
        UIdx["utilities/index.dart\nconfig + extensions + externals"]
    end

    subgraph Features
        Auth["features/auth/auth.dart\nfeature barrel"]
        AS["auth_screen.dart"]
        AC["auth_controller.dart"]
    end

    MB & SB & WB --> Core
    AS & AC --> Auth
    Core & UIdx --> AnyScreen["Any screen\n2 imports max"]
    Auth --> AnyScreen
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

```mermaid
flowchart LR
    Choice{"Workflow type?"}
    Choice -->|"VS Code user"| BM["Barrel Me extension\nRight-click → Create Barrel\nOne-click project-wide migration"]
    Choice -->|"CLI / CI pipeline"| SIG["single_import_generator\ndart run ... all / dir / @SingleImport"]
    Choice -->|"Selective classes only"| Anno["@SingleImport annotation\nmarks specific classes/extensions"]
```

### Option A — `single_import_generator`

```yaml
dev_dependencies:
  single_import_generator: ^<latest>
```

```bash
# All files in directory (recursive)
dart run single_import_generator -target=lib/presentation all

# Immediate files only (non-recursive)
dart run single_import_generator -target=lib/presentation/common dir
```

```dart
// Annotate only what should be barrel-exported
@SingleImport()
class FrequentlyUsedClass { ... }

@SingleImport()
extension SomeStringExtension on String { ... }
```

```bash
dart run single_import_generator -path=lib/utilities
```

### Option B — "Barrel Me" (VS Code Extension)

Right-click any folder → **"Create Barrel"**. Key features:
- Flat or hierarchical barrel generation
- Auto-excludes `main.dart`, `part` files, existing barrels
- Renames conflicting files to `{name}_barrel.dart`
- One-click import migration across the whole project

> Search **"Barrel Me"** in the VS Code Extensions Marketplace.

---

## Agent Decision Flow

```mermaid
flowchart TD
    Start["User has import problem"]
    Audit["Count imports per screen\nGroup by folder prefix"]
    Count{"3+ from same folder?"}
    Mixed{"Mixes external\npackages with internals?"}
    Size{"Large layered\ncodebase?"}
    Feature{"Feature-first\narchitecture?"}

    Count -->|Yes| FolderBarrel["Folder-named barrel\nmodels/models.dart"]
    Count -->|No| Mixed
    Mixed -->|Yes| IndexDart["index.dart barrel\nutilities/index.dart"]
    Mixed -->|No| Size
    Size -->|Yes| Hierarchical["Hierarchical barrels\nper-layer index.dart\n+ top-level core.dart"]
    Size -->|No| Feature
    Feature -->|Yes| FeatureBarrel["Feature barrel\nauth/auth.dart"]
    Feature -->|No| FolderBarrel

    Start --> Audit --> Count
```

---

## When to Apply Which Pattern

| Situation | Recommendation |
|---|---|
| 3+ imports from the same folder | Folder-named barrel |
| Repetitive base imports on every screen | `utilities/index.dart` |
| Mixing external packages with internals | `index.dart` with both export types |
| Large layered app | Hierarchical barrels per layer |
| Feature-based monorepo | Per-feature barrel (`auth/auth.dart`) |
| Existing messy codebase | Barrel Me (VS Code) for one-click migration |
| CLI / CI workflow | `single_import_generator` with `all` or `dir` flag |

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

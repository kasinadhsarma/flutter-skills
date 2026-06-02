# flutter-skills

A curated reference library of Flutter architecture patterns, security practices, layout guides, and AI-agent skills — maintained under a strict **1,000-line-per-document** discipline so that every guide fits in a single LLM context window.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter)](https://flutter.dev)
[![Dart](https://img.shields.io/badge/Dart-3.x-0175C2?logo=dart)](https://dart.dev)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## What's Inside

| Document | Purpose | Lines |
|---|---|---|
| [Architecting High-Productivity Systems Under 1,000 Lines](docs/Architecting%20High-Productivity%20Systems%20Under%201%2C000%20Lines.md) | Server-driven UI, fine-grained signals, PocketBase backend, AI context engineering | ~630 |
| [Flutter App Security Development Roadmap](docs/Flutter%20App%20Security%20Development%20Roadmap.md) | SDLC security gates, secrets management, SSL pinning, RASP, OTA security | ~540 |
| [Flutter Overflow & Pixel Prevention Guide](docs/Flutter%20Overflow%20Pixel%20Prevention%20Guide.md) | Layout engine mechanics, constraint violations, responsive design, debug tools | ~400 |
| [Flutter Barrel Imports Skill](docs/skills.md) | Agent skill for organizing Flutter/Dart import sections with barrel files | ~290 |

---

## Who This Is For

- **Flutter developers** who want architecture references, security checklists, and layout fix patterns in one place
- **AI coding agents** (Claude Code, Windsurf, Copilot) that need well-structured, file-scoped context to generate accurate code
- **Team leads** building internal style guides or onboarding material

---

## How to Use

### As a reading reference

Each document is self-contained with Mermaid architecture diagrams, working code examples, and decision tables. Read them top-to-bottom or jump to a section.

### With AI coding agents

Every document is under 1,000 lines — intentionally. This means an agent can load an entire guide into its context window in a single read and reason about the full content without retrieval fragmentation.

**Recommended workflow with Claude Code:**
1. Open the relevant guide alongside your project
2. Reference it in your prompt: *"Following the patterns in the security roadmap, add SSL pinning to this Dio client"*
3. Run `/compact` after long sessions to flush accumulated context

### As an agent skill

The [barrel imports skill](docs/skills.md) is structured as a Claude Code skill with YAML frontmatter triggers. When installed, it activates automatically when a user asks about messy imports, barrel files, or `index.dart` organization.

---

## Repository Structure

```
flutter-skills/
├── docs/
│   ├── Architecting High-Productivity Systems Under 1,000 Lines.md
│   ├── Flutter App Security Development Roadmap.md
│   ├── Flutter Overflow Pixel Prevention Guide.md
│   └── skills.md                          ← agent skill (barrel imports)
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── new_skill.md
│   └── pull_request_template.md
├── CONTRIBUTING.md
├── SECURITY.md
├── LICENSE
└── README.md
```

---

## Design Principles

1. **Under 1,000 lines per document** — every guide fits in a single LLM context window with room to spare
2. **Mermaid diagrams over images** — diagrams are text, version-controlled, and renderable in GitHub and any markdown viewer
3. **Working code, not pseudocode** — every snippet compiles and runs
4. **Agent-readable structure** — headings, tables, and code blocks are used consistently so agents can parse intent from structure
5. **No citation noise** — sources inform the content; they don't clutter it

---

## Coverage Roadmap

| Topic | Status |
|---|---|
| Architecture (1,000-line systems) | Done |
| Security (OWASP Mobile Top 10) | Done |
| Layout & overflow prevention | Done |
| Barrel imports (agent skill) | Done |
| State management patterns | Planned |
| Performance profiling | Planned |
| Flutter Web deployment | Planned |
| Testing strategies | Planned |

Want to add a topic? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Contributing

Pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide — including how to add a new skill, update existing docs, and the review process.

---

## Security

Found a vulnerability or error in the security guidance? See [SECURITY.md](SECURITY.md) for the responsible disclosure process.

---

## License

MIT — see [LICENSE](LICENSE). Copyright © 2026 kasinadhsarma.

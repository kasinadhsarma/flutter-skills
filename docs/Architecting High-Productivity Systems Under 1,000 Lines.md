# **Architecting High-Productivity Systems Under 1,000 Lines: A Socio-Technical Blueprint for Low-Code, Serverless, and AI Context Engineering**

The rapid evolution of generative artificial intelligence and high-efficiency runtime environments has initiated a fundamental paradigm shift in software construction. Historically, software value was correlated with codebase scale and complexity. In the modern development ecosystem, however, code is recognized as an immediate operational liability.1 Codebases that expand unchecked introduce cognitive friction, escalate technical debt, and degrade the reasoning capacity of the Large Language Models (LLMs) used to co-author them.1  
A strict architectural constraint limiting a software project—or its constituent microservices—to under 1,000 Source Lines of Code (SLOC) functions as a powerful socio-technical forcing function.4 Capping code volume forces developers to adopt declarative abstractions, leverage specialized single-binary backends, and offload infrastructural boilerplate to modern low-code runtimes.6 By maintaining an ultra-lean footprint, engineering teams can implement a "Full-Context Code Analysis" workflow.5 This approach allows an entire codebase to reside within an LLM’s context window during every step of development, eliminating the hallucinations and fragmentation associated with classic retrieval-augmented generation (RAG).5

## **The Philosophy of Minimalist Flutter Codebases: Architectural Viability at Scale**

Evaluating code density based on raw metrics like Source Lines of Code is often criticized as a superficial measure of software quality.2 However, when code reduction is aligned with readability, complexity reduction, and self-explanatory design, it directly minimizes structural risk.2 Adhering to the SOLID principles and Albert Einstein's metric of simplicity—making everything as simple as possible but no simpler—enables developers to treat every line of code as an engineering liability.2  
The viability of keeping complex systems under a 1,000-line threshold is demonstrated across multiple domains:

* **Operating Systems**: An entire functional operating system—implementing basic context switching, paging, user mode, a command-line shell, a disk device driver, and file read/write operations—can be written in exactly 1,000 lines of C.11  
* **Text Editors**: A highly performant cross-platform code editor with a document rendering view, line-number gutters, keyboard integration, and basic syntax highlighting can be written in under 1,000 lines of pure Dart using custom scrollable RichText widget compositions.  
* **Decentralized Applications**: Global-scale services, such as Dfinity's decentralized alternative to TikTok ("CanCan"), have been fully implemented in under 1,000 lines of code by utilizing serverless cloud architectures that completely eliminate the need for custom web servers and traditional database boilerplate.6

In corporate settings, keeping codebases small simplifies IP management and open-source contributions. For example, Connected Cars' open-source policy expressly permits employees to publish small codebases (under 1,000 lines) created during work hours without undergoing complex corporate approvals, highlighting the low risk and high portability of ultra-lean projects.12  
To stay within these tight constraints, developers must move away from unstructured development practices. Unstructured "vibe coding"—where developers write code continuously until it breaks—frequently leads to bloated systems.1 A disciplined alternative uses AI to pre-plan application boundaries before writing any files, categorizing features via MoSCoW prioritization (Must Have, Should Have, Could Have, Won't Have) to keep the project focused.1

## **Server-Driven UI (SDUI): Eliminating UI Layout Code Overhead**

Traditional client-side applications are burdened by massive layout boilerplate. Expressing responsive designs, form fields, and conditional routing tables locally consumes a major portion of a 1,000-line budget.. Capping a client-side codebase at an ultra-lean limit requires moving away from static, hardcoded UI layers toward dynamic, Server-Driven UI (SDUI) architectures.  
The viability of this minimalist approach in the Flutter ecosystem is powered by **Stac** (formerly known as *Mirai*), an open-source, production-grade SDUI framework built specifically for Flutter. Instead of writing thousands of lines of local Dart widget trees, the app behaves as a generic layout renderer that fetches, parses, and executes UI definitions served from a backend or CDN as lightweight, declarative JSON schemas.  
Stac maps layout logic directly to Flutter's native widget properties, making translation highly intuitive. If you can construct a widget tree in Dart, you can express it in Stac JSON:

JSON  
{  
  "type": "column",  
  "children":  
}

By leveraging this paradigm, a fully production-ready client shell can be implemented in a single main.dart file using the StacApp wrapper:

Dart  
import 'package:flutter/material.dart';  
import 'package:stac/stac.dart';

void main() async {  
  await Stac.initialize();  
  runApp(const MyApp());  
}

class MyApp extends StatelessWidget {  
  const MyApp({super.key});

  @override  
  Widget build(BuildContext context) {  
    return StacApp(  
      title: 'Stac SDUI Demo',  
      theme: ThemeData(primarySwatch: Colors.blue),  
      home: Stac.fromNetwork(  
        StacNetworkRequest(  
          url: 'https://api.yourdomain.com/ui/home',  
          method: Method.get,  
        ),  
      ),  
    );  
  }  
}

For offline-first capabilities or local-fallback structures, the same rendering pipeline can ingest pre-compiled static schemas bundled locally with the app using Stac.fromAssets('assets/screens/home.json'). This approach saves thousands of lines of UI rendering and navigation code, shifting layouts, inputs, and form validation rules to the backend.

## **Front-End Architecture: Fine-Grained Reactivity and Boilerplate Elimination**

Traditional Flutter state management libraries (such as BLoC, Redux, or manual InheritedWidgets) require substantial boilerplate code. To build complex feature sets under a 1,000-line ceiling, developers must transition from verbose stream-based architectures to lightweight, fine-grained reactive paradigms and compile-time language features.

### **Signals: Eliminating Context and State Boilerplate**

Signals-based state management (such as signals\_flutter or the hyper-optimized oref package) replaces verbose ChangeNotifier and stream controller boilerplate. Signals are reactive state variables that automatically track dependencies and trigger micro-rebuilds only for the exact UI nodes that depend on them, completely bypassing the need for StatefulWidgets or manual lifecycle methods:

Dart  
import 'package:flutter/material.dart';  
import 'package:oref/oref.dart';

class Counter extends StatelessWidget {  
  @override  
  Widget build(BuildContext context) {  
    // Instantiates a reactive signal directly bound to the BuildContext  
    final count \= useSignal(context, 0);   
      
    return Column(  
      children:,  
    );  
  }  
}

This model provides exceptionally high developer velocity, providing React-style reactivity with zero-setup scaffolding.

### **Compiler-Level Optimization: Dart Macros and Extension Types**

A common pain point in maintaining lean codebases is the code generation "tax" imposed by libraries like freezed and json\_serializable, which require executing build\_runner and cluttering the disk with thousands of lines of auto-generated .g.dart files.  
In modern Dart, this boilerplate is eliminated using compile-time **Dart Macros**. By using experimental macro annotations like @JsonCodable, the compiler performs static metaprogramming, generating getters, setters, and serialization logic instantly in memory during compilation. This completely bypasses the need for disk-based code-gen scripts:

Dart  
@JsonCodable()  
class User {  
  final String name;  
  final int age;  
    
  User({required this.name, required this.age});  
}

Additionally, developers can employ **Extension Types** as zero-runtime-cost wrappers. Unlike traditional wrapper classes, extension types are evaluated statically and completely compiled away at runtime. This enables developers to enrich existing data types with domain-specific utility APIs with zero performance overhead and minimal code structure.

| Operational Parameter | Signals (Modern Fine-Grained) | BLoC 9.0 (Enterprise Stream) | Riverpod 3.0 (Declarative Cache) |
| :---- | :---- | :---- | :---- |
| **Boilerplate Overhead** | Near Zero (No actions/events or code-gen required) | High (Requires Events, States, and MapEventToState) | Low-Medium (Requires providers and build\_runner) |
| **Reactivity Pattern** | Proxy/Dependency-tracked automatic rebuilds | Explicit stream-based state emission | Declarative query caching & invalidation |
| **Lifecycle Cleanup** | Automatic via framework mixins & extensions | Manual via close() and StreamSubscriptions | Managed automatically by ProviderContainer |
| **AI Parseability** | Extremely High (Standard functional classes) | Moderate (Complex multi-file event-mapping structure) | High (Requires strict code-gen mapping rules) |

## **Low-Code visual development via FlutterFlow**

For rapid application development, internal business utilities, and MVPs, designing custom UI layouts, database bindings, and action paths from scratch is highly inefficient. Professional development teams can leverage **FlutterFlow**, a low-code visual builder powered by Google's underlying Flutter framework.  
FlutterFlow bridges the gap between drag-and-drop simplicity and raw, production-ready flexibility:

* **Visual Scaffold Composition**: UI layouts are built visually on a drag-and-drop canvas, completely eliminating hand-coded UI structures.  
* **Action Flow Editor**: Complex conditional operations, page routing, and state transitions are declared visually without writing manual controllers.  
* **AI Copilot Integration**: Developers can generate complex backend schemas, API connectors, and raw Dart functions directly from plain-English prompts.  
* **Zero Vendor Lock-In**: Unlike traditional closed-ecosystem low-code platforms (e.g., Bubble), FlutterFlow compiles directly into clean, human-readable, and fully customizable Flutter source code that can be exported at any time.

This visual-declarative hybrid model cuts development timelines by 50% to 80%. Non-core workflows, standard authentication steps, and visual forms are designed visually, allowing developers to reserve custom hand-written Dart code exclusively for high-value business logic and custom plugins.

## **Secure Client-Direct Architecture with PocketBase**

To maintain a backend footprint that easily fits under the 1,000-line limit, developers should avoid building custom API controllers, routing tables, and manual JWT validation middleware. The ideal pattern is a client-direct architecture utilizing **PocketBase**, an open-source, lightweight backend compiled into a single self-contained Go executable.13  
PocketBase embeds an optimized SQLite database in Write-Ahead Logging (WAL) mode, a secure authentication provider, and real-time subscription mechanisms out of the box.13 By utilizing the official **PocketBase Dart SDK** (pocketbase), the Flutter application connects and queries the datastore directly over highly efficient, persistent Server-Sent Events (SSE). This completely eliminates the need to maintain custom server-side API endpoints.15

Dart  
import 'package:pocketbase/pocketbase.dart';

final pb \= PocketBase('https://api.yourdomain.com');

// Sets up a real-time stream subscription to database collections  
void listenToTasks() async {  
  await pb.collection('tasks').subscribe('\*', (e) {  
    print('Action: ${e.action}'); // "create", "update", or "delete"  
    print('Record ID: ${e.record?.id}');  
  });  
}

Authentication and data safety are managed declaratively in the PocketBase Admin panel using collection-level API Rules. The database evaluates incoming JWT credentials and applies fine-grained access control on individual database rows dynamically, saving hundreds of lines of imperative server-side middleware.  
SQLite's file-based nature means that the entire backend state resides in a single database folder (pb\_data), making backups, migrations, and local offline replication simple.

## **Context Engineering for Flutter Coding Agents**

When co-authoring applications with AI systems, codebase architecture directly affects the AI's reasoning accuracy and cost efficiency.3 The mechanics of "Vibe Coding"—where developers rely on agentic IDE tools like Windsurf, Augment, Claude Code, or Copilot—require rigorous architectural guardrails to prevent system failures.3

### **The Mathematical Cost of Context Accumulation**

Every request sent to an LLM operates within a finite, combined input-output boundary called the context window.9 In multi-turn agentic coding sessions, the context window accumulates conversation history, tool definitions, file structures, and diagnostic outputs.17  
Let the total cumulative tokens in the context window at step ![][image1] be modeled as:  
![][image2]  
Where:

* ![][image3] represents the initial system prompt and baseline codebase tokens.17  
* ![][image4] represents the new prompt tokens, tool execution results, and diagnostic outputs introduced at step ![][image5].17  
* ![][image6] represents the model's generated response and code output from the previous step.9

The total cost of an agentic session over ![][image1] steps can be modeled as:  
![][image7]  
Where ![][image8] and ![][image9] are the input and output tokens at step ![][image10], while ![][image11] and ![][image12] are the cost rates per token.17  
As the context window fills, the LLM's attention mechanism begins to degrade due to the "lost-in-the-middle" effect, where the model struggles to parse information buried in the middle of a large prompt.9 This frequently causes the AI to hallucinate variables, lose track of core functions, or rewrite functional code into broken logic.18

### **Code Demarcation Rules for Flutter AI Optimization**

In Flutter, splitting UI sections into separate, local helper build functions (e.g., \_buildHeader(), \_buildList()) within a single massive file is a severe anti-pattern. Keeping a file over 1,000 lines long degrades both rendering performance—as the entire page is forced to rebuild instead of localized widget subtrees—and the AI's parsing accuracy.  
To enforce optimal context engineering and keep codebases clean, the team should adopt strict architectural guardrails 1:

* **Enforce Widget Decoupling**: Keep individual UI files strictly under 350 lines. Extract distinct sub-sections into dedicated, immutable StatelessWidget classes.  
* **Restrict Function and Method Length**: Limit custom layout operations or state manipulators to a maximum of 20 lines per function.1  
* **Strict Dependency Management**: Limit third-party package dependencies to 10 production-ready libraries to avoid build overhead and binary bloat.1  
* **Maintain Context Summarization Logs**: When the conversation context begins to balloon, run the /compact command (in tools like Claude Code) or manually append state definitions to a root TODO.md or CHANGELOG.md file. This allows developers to flush the conversation history safely, re-anchoring the model on a tiny, structured context footprint.18

## **Synthesis and Actionable Recommendations**

To build and scale a high-productivity mobile application that remains strictly under the 1,000-line threshold, engineering teams should implement the following recommendations:

* **Deploy Stac (Mirai) for UI Layout Delivery**: Abstract the visual layout layer from the compiled binary. Use Stac to parse and render dynamic, server-served JSON layouts, cutting local UI rendering code to a single, secure StacApp shell.  
* **Adopt Fine-Grained Signals Over BLoC/Provider**: Move away from heavy boilerplate states and streams. Use the oref package to manage state reactively inside lightweight stateless widgets via the useSignal(context) hook.  
* **Eradicate build\_runner with Native Dart Macros**: Implement experimental compile-time Dart Macros (like @JsonCodable) to generate data serialization logic directly in memory, saving hours of codegen time and cleaning up git diffs.  
* **Utilize PocketBase and the Dart SDK**: Eliminate custom backend endpoints, routing layers, and manual database integrations. Connect the client directly to PocketBase using the pocketbase Dart SDK, managing auth and row access via declarative SQL database rules.  
* **Bridge Workflows with FlutterFlow**: Speed up non-core UI flows, standard authentication forms, and rapid visual prototyping by using FlutterFlow's visual editor. Export the generated, clean Flutter code to extend it with precise hand-written scripts.  
* **Decompose Monolithic Widget Trees**: Keep all codebase files under 350 lines by extracting inline widgets into isolated, immutable StatelessWidget classes. This optimizes the app for both fast sub-tree rendering and precise, context-efficient AI generation.

#### **Works cited**

1. If every file in your codebase is 1,000+ lines long… don't be surprised when AI starts making mistakes or hallucinating. : r/vibecoding \- Reddit, accessed on June 2, 2026, [https://www.reddit.com/r/vibecoding/comments/1mi6guu/if\_every\_file\_in\_your\_codebase\_is\_1000\_lines\_long/](https://www.reddit.com/r/vibecoding/comments/1mi6guu/if_every_file_in_your_codebase_is_1000_lines_long/)  
2. How important is it to reduce the number of lines in code? \- Software Engineering Stack Exchange, accessed on June 2, 2026, [https://softwareengineering.stackexchange.com/questions/185925/how-important-is-it-to-reduce-the-number-of-lines-in-code](https://softwareengineering.stackexchange.com/questions/185925/how-important-is-it-to-reduce-the-number-of-lines-in-code)  
3. Vibe Coding vs Spec-Driven Development (2026): When to Use Each, accessed on June 2, 2026, [https://www.augmentcode.com/guides/vibe-coding-vs-spec-driven-development](https://www.augmentcode.com/guides/vibe-coding-vs-spec-driven-development)  
4. Less code, more power: Why we rolled our own React Server Components framework, accessed on June 2, 2026, [https://www.aha.io/engineering/articles/why-we-rolled-our-own-rsc-framework](https://www.aha.io/engineering/articles/why-we-rolled-our-own-rsc-framework)  
5. Generative AI Prompt Patterns for Software Engineering \- Codemotion Magazine, accessed on June 2, 2026, [https://www.codemotion.com/magazine/ai-ml/generative-ai-prompt-patterns-for-software-engineering/](https://www.codemotion.com/magazine/ai-ml/generative-ai-prompt-patterns-for-software-engineering/)  
6. Dfinity Showcases a Decentralized TikTok Made in Under 1000 Lines of Code, accessed on June 2, 2026, [https://cryptobriefing.com/dfinity-showcases-decentralized-tiktok-1000-lines-code/](https://cryptobriefing.com/dfinity-showcases-decentralized-tiktok-1000-lines-code/)  
7. Low Code No Code: Complete Platform Comparison (2026) \- DesignRevision, accessed on June 2, 2026, [https://designrevision.com/blog/low-code-no-code-complete-platform-comparison](https://designrevision.com/blog/low-code-no-code-complete-platform-comparison)  
8. Declarative vs Imperative Programming | \[Data Engineers Guide\] \- DataOps.live, accessed on June 2, 2026, [https://www.dataops.live/blog/the-data-engineers-guide-to-declarative-vs-imperative-for-data](https://www.dataops.live/blog/the-data-engineers-guide-to-declarative-vs-imperative-for-data)  
9. LLM Context Windows Explained: 4K to 1M Tokens (2026) \- DevTk.AI, accessed on June 2, 2026, [https://devtk.ai/en/blog/llm-context-window-explained/](https://devtk.ai/en/blog/llm-context-window-explained/)  
10. I Refactored 1000 Lines of Legacy Python Code. Here's What I Learned \- Medium, accessed on June 2, 2026, [https://medium.com/the-pythonworld/i-refactored-1-000-lines-of-legacy-python-code-heres-what-i-learned-78b6c8149af6](https://medium.com/the-pythonworld/i-refactored-1-000-lines-of-legacy-python-code-heres-what-i-learned-78b6c8149af6)  
11. Operating System in 1000 Lines, accessed on June 2, 2026, [https://operating-system-in-1000-lines.vercel.app/en](https://operating-system-in-1000-lines.vercel.app/en)  
12. Open source policy \- Connected Cars, accessed on June 2, 2026, [https://connectedcars.io/open-source-policy/](https://connectedcars.io/open-source-policy/)  
13. Supabase vs PocketBase: Full Comparison \- Leanware, accessed on June 2, 2026, [https://www.leanware.co/insights/supabase-vs-pocketbase](https://www.leanware.co/insights/supabase-vs-pocketbase)  
14. Stop Paying for Supabase: The Ultimate Self-Hosted Backend Guide (2026) \- DevMorph, accessed on June 2, 2026, [https://www.devmorph.dev/blogs/stop-paying-for-supabase-self-hosted-pocketbase](https://www.devmorph.dev/blogs/stop-paying-for-supabase-self-hosted-pocketbase)  
15. Serverless in action: building a simple backend with Cloud Firestore and Cloud Functions, accessed on June 2, 2026, [https://cloud.google.com/blog/products/application-development/serverless-in-action-building-a-simple-backend-with-cloud-firestore-and-cloud-functions](https://cloud.google.com/blog/products/application-development/serverless-in-action-building-a-simple-backend-with-cloud-firestore-and-cloud-functions)  
16. PocketBase vs Supabase: Build Real-Time Vue 3 Apps in 2026 \- Zignuts Technolab, accessed on June 2, 2026, [https://www.zignuts.com/blog/pocketbase-vs-supabase-vue-3](https://www.zignuts.com/blog/pocketbase-vs-supabase-vue-3)  
17. Vibe Coding Cost Overrun: 5 Patterns That Blow Up Your API Bill \- Atlas Cloud, accessed on June 2, 2026, [https://www.atlascloud.ai/blog/guides/codinh-plan-vibe-coding-cost-overrun](https://www.atlascloud.ai/blog/guides/codinh-plan-vibe-coding-cost-overrun)  
18. A Goldfish's Guide to Vibe Coding | by Lyndon \- Medium, accessed on June 2, 2026, [https://medium.com/@lyndon\_56890/a-goldfishs-guide-to-vibe-coding-8b0b069d60cd](https://medium.com/@lyndon_56890/a-goldfishs-guide-to-vibe-coding-8b0b069d60cd)  
19. Context Engineering: A Practical Guide for AI Agents (2026) \- Sourcegraph, accessed on June 2, 2026, [https://sourcegraph.com/blog/context-engineering](https://sourcegraph.com/blog/context-engineering)  
20. Beginner vibe coders after discovering context limits : r/vibecoding \- Reddit, accessed on June 2, 2026, [https://www.reddit.com/r/vibecoding/comments/1tji5e6/beginner\_vibe\_coders\_after\_discovering\_context/](https://www.reddit.com/r/vibecoding/comments/1tji5e6/beginner_vibe_coders_after_discovering_context/)

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAwAAAAaCAYAAACD+r1hAAAAyUlEQVR4Xu3RwQoBQRzH8b8QN4pIDuIFJO6uHHgC7+EpPIMbF0fOXkBKkZNyUA5KOSt8/zuza1oeYA/7q0/b/GZ3Z2ZXJE7UUkLSGef/dEE6OGGPGrq44I4H+t9bRXJYYoA3NpjZuQK2ONixF31aLfBCz5lr4CqhB/xoeUbV6YZiVp06XZAnVsjacQpz+V01iL5p7IzrYg5+RBkjZPxJLXRLRb8gEzEv0Y+hh285c9IWs8+E0zVxwxo7p/eiPycdLsWcp2KvcSKUD9jWISeibPg3AAAAAElFTkSuQmCC>

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABcAAAAaCAYAAABctMd+AAABa0lEQVR4Xu2Uuy8EURSHjyAhXhGJR0cjIfFIFEqVQiOREBKFUiN6/4ZSKBQ60REShVdFoRFRUFD4AyQKhcf3y70zZu/OTHbXVrJf8mXPnjtzdu45d9asRgXUYRf2YWsi34gtie9loaKT+Inf+IofeIHNuI3L8dVl0IsH+IUjWO/z+pzDE3zDMZ8vmXF8wXecDdaEdqSd3GBnsJbLtbkbN8wVyeLZKmiJCt9hd7gQoIcYCpN5DJgrvhYupDBvbqgls2quzxPhwl/R+T3DPcvvdRrtuIXrlnHuo+K7QT4NtU8FRQcemjuiiz5WroAG3LfSiidnsoTnPm7zsXJF6KZjyx9UPx75WO1TG6WIHlBvbip61Z9wNMgP4yUu2O9M0lqpWLnkf1DMFD6aO5IPuIP3eIWDietE2cWFhqMXRGd5BXsKl2N0Mk6tuLiG2pTIVcymuR8Q0U6UqwrTeOtj7VCxclVDA56x7PbV+E/8AKRRPkK5NrO8AAAAAElFTkSuQmCC>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAaCAYAAADWm14/AAABfUlEQVR4Xu2UvytGYRTHjxiERZGJIovIYhBlYmCRZDMYjQaDYlGSbJJJzCaj3U6ZLPwBBslk9OP77TyP93F673vPVe/tHe6nPnXvOc+9z6/zPCIVLUwbnA3yuXTG4WuQzx424CU8Nx6ljbycwe/gicllMQ3X4DH8hHvhnatYmCd4LzoArkIRduEd7LUJL1NwFHbAC9FBTPxpkc0IfIHLNuEldhoLjx2/i26JpxiXRAfMgfwLzv4xeWen7JyD8KzCoegAum3CQ5z9vomzY/6UObbJogfeirYtDH98BXdsIsBCzBsE951tWAOWdThvgykz8AEO2kSAR5E//xDdpnqcira5sQnwDLdtMKXR7Em8mOIqWHjkePSY5zEsTKPZExbjgdRWwTIG3+AXXDC5XOakdut55ZalXIc4l78rifMmXBHdgoEk/kssPttBnvyG327WydEhUVZhn/jvkaYwDBdtsEy4Sv1wUrKPcNPoFK0LXmZbJlca3Pt2G6yoaAl+AMFYYlJPgsNpAAAAAElFTkSuQmCC>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAcAAAAcCAYAAACtQ6WLAAAAkElEQVR4XmNgGBxAEIgZ0QVBgBWI/wOxJ7oECAgD8XMg1kSXgAEOdAGCgJkBYiwGUAHi00D8GojlkSVcgLifAeL8dCCOQJYEOdsUiDmBeAcQKyJLwoAOEL9nwBEADUD8D10QBPiB+AQQXwdiZSAORJa0AeLfQDwJiEsZ0BzlywAJUxC9nAGLf0HBJokuOOIBACXsEVMbAsH9AAAAAElFTkSuQmCC>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADkAAAAaCAYAAAANIPQdAAACPElEQVR4Xu2WP0hVURzHv1GCUdBgGKKCQZuBQzgUCRIuDoZEQ+DilkhTQ2LTg3BvcJIWEQkChzbHi7PgJEE0VERLQxDYYKV+v/3uD887vnfvuw/1+eR+4MN77/zOfZzfPb/zBygpORdcoPdS9b0ZOugN2hMHzgqD9EeqvhdByW3RX/Qr/UY/0Ado/oWdCIt0P/V1FMtimH6iI/Ri0L4K+6/5qL2lfKSbsIFpNvPQDE3Tf3SlOvSfa3QDFn8YxVrCHXqLXqJvYInerupxFA0+r7Tv0z+wMh6KYqeKJ+ZrR8n9hJVvvfXUDXsRr1C/j1Ap/6Z7dCyKnSqaxe3gtwatBJVovdl8huy4MwF7GS1N0mexErVr8BqcYuoTcpUmdJ1erg4dYQH2Py0rVw3+LZ2LAylab7USHU/bHwVttfCyV99K0N5HX8A2pqIM0CdxYxZ3YWdbfxxI0TGiAe7AStrxEtRnFlqv6vcdtqk5z2FnqJJtlE7YUfSXLkexTLJmUfjlwGfT8c0kL0l/diYONIH2iSuwZVIoyaxZFPpjnw3NptNLP9OnQVuM1qqeW4Ldho6LBAWS1O1EgyiiytvRelKb1lZ4hKgE38PO0Dg5PaM7bQIr2WZI0GCSvuHESeSpZ8INSL91NKgiZuka3aXv6M2gnzMAe1FfcHj0XKeT9HEd44t+ggaTPE66YGtzio7CNogsKqg+etoiyaLo0qEXonUti5KgDZJUqeo4eYnaJZ1HgjZIUuWsG1NJSUnJ+eIAcN+MIvgw78IAAAAASUVORK5CYII=>

[image7]: <data:image/png;base64,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>

[image8]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAAaCAYAAAC6nQw6AAABA0lEQVR4Xu2SsWoCQRCGR0SsIih2FqIvkEJCGl8gYGllQCuJ4HNYpE0pkVR2+g7prbUVGx9ASJv4/8wdjMOCtwfp7oMPb3eGYcZZkYJYJnAFl0aeRzYpC89wCNfwD86Sc9smxfApWqjkAzE8wgv88YFYXkW72flALB+ihThebuqinbAQO8uNHYtFLe9wD5vuPki6rS8Jb4zPInR/w72xGH/zlyHs2nsuRhjv+8sQW9Fu+Ft1MY7Dcfnyp3AAf+GLTeKBBbzfJicdeyxatCbaXcXkZIJjneERLiRHgRT++Vz9EzyJFm4lRsFnsRFdwgF24Rx2bFIWHmA5+eZYDcnwngr+iSvXRTH+90tZMwAAAABJRU5ErkJggg==>

[image9]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABkAAAAZCAYAAADE6YVjAAABZElEQVR4Xu2UvytHURjGH6GISH4lg9EiEZvFYGWwMvkTDMhul5RFSVKKVVYxECspUchmsMmE5/Heo+P1/XG7Xdv91Kd773nPue897zn3AAU5Uk+7aY8P5EENPaNv9Ik+0ws6FnfKyhDspZc+QLroFX2lwy6WmlH6Qm9pn4sFVugnXfOBtJzAXjDrAxGTsD6aaZuLpUKDz2mrD0SEJI/IsBlmYLUe8IEIbYYtZJyJBu+i+sAOeg1LomQaF9Pgnn/RTI9hifzAmAn6AUsylbQp8WnSVnEzhCTbrt2zjtLrFmY4HbWVRFvzEOWnrA9RggX8ne08qpf6mxGUX3gdLRuJuo+powewUin5Dj2i7XGngDroS/doU9TeAivjO62N2gO99AH2b83RTXoPO+/KonKNw7a0ripTJVSqO7qfXP+FUCqxDKvCIG386ZEDKo1mLRZpJ12CrVVu3ND+5F7/zirsoC0oAL4Az/BCrw7WPXYAAAAASUVORK5CYII=>

[image10]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAsAAAAcCAYAAAC3f0UFAAAA50lEQVR4Xu2SqwpCQRCGR1BQvIMogu9gspgtFrMYbOIj+AQWu5itYrcYThRMBqNBsVoNgpf/dy8c1j1dwQ++sDNzdmdnj8hvU4cneINDJ/dBCW7hFTacnBcWH2DFTfi4wCWMuwkfTzhyg1E8YCu0jsFEaG3JwjOs6TUvuYMrmDFFhg4cwyacwBRsww3Mh+resHABpzCtY2yhais0TK5FXZDj64na2Qv7ZL9FOIB3OJOIEXICnAThZQJ4FNVCX8csnC1bIKZ4L+oXmMOkzr2P4qvx9YjpP4A52NVxS0FCX2t4QtmJ/fk2XgTqIybkPiUQAAAAAElFTkSuQmCC>

[image11]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB4AAAAaCAYAAACgoey0AAABmklEQVR4Xu2UvyuGURTHv/IjQhIl2RQlYjApo/JKJNkYlMGCRfgL/ANSCiWDQQwWm8FGmZWBgZSsymDB9+vcx3tdz+J97zt5vvWp557zdM+933vvATL9N42QfXJDHsgJ2fboJGXff0dUF5kiB+SezLqxmCMf5Ig0uP+ja4PshkFqD1Z8i1QEuaLVTp5IX5iAOaHCV6QxyBWtcdjkaRM/wnLLYSKG1mGTh6pE3mZ9R1UdOYcVaCUdZIIskVsyihLd6sRmnaGvGRdP26lii6Q3TPxFic3hje4nrzAXQukSPsMWXZB8m6d/pjBE3pFeuGglz+gFv59S4kRLEI+iHeQ7U7UX1/epy8mVGnIIs1+skTNSC7uEg+SNrMIu4gUZQ4q0O+1SE/v4nUmT6Yx1iXT+K7AFyIFLsuD+U8udhx2ZFimp9eqoClYTrGdrl+UuNgCbuMeNtdhjWMtNdEfavHEUqYAK1cOeUzO5JpMur4Vsuny3i0WRiugFDMPc0JHIAVku6ShysCakfDTJcvX0pJupmVTl01/y85kylV6fWatNoTG+/koAAAAASUVORK5CYII=>

[image12]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACUAAAAaCAYAAAAwspV7AAABz0lEQVR4Xu2WvyuGURTHj1CE/E5CoigRg0kZDBaJhEGRbBYMhLIZ/AMWhZLB4MdgUQblKTY7CgMpWZWZ79e51/O8F3mH53kMnm996rn3nu4995xzz/uKJEoUrXrANrgGD+AQrBuWQSPI+LSOSU1gGOyAezBhxuQUvIF9UGjsY9Uq2HTmssGWqGNrzlrkqgdPoM2ZZ9oYQTp14axFrn7Rg4ud+TrwaNbmnLXItSJ6sCsWu00dUxmb8oEnenglaAADYAZcgV75g9dnU+fWzBjYk5gjZGVT5768dtEWweiFpS5Q4U66CqZuNHVJuiV8pxiAHHfSlW0FL/K1HXCDO0njZmmKzhy5k99pQ/yOHbyB3cATjWYu2BUt+ClwDsqN7SXoEG0ZTPkJyANZ4MDYUIw8L/mjGBVGhw4F4UZWneAVTIvW2zxoBs9gJGDHFC+AQVAGFs18laQ6wcinFanf1Cf6G8gIZIoeyGZaHbDh5fiCKUasxXzzUrfm20Z+yYxDFf9NeKIptboBNeZ7FhSYb17AA62itcuIMoUcsxxC0yQ4E//gWjDkL384whLgoceiP/Lj4r9kOkebSJpxESiV7zdnXTHVFO2s2IRLAuNEif6f3gE3n1YQSLh0+QAAAABJRU5ErkJggg==>
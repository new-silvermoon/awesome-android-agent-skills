# Awesome Android Agent Skills

[![Agent Skills](https://img.shields.io/badge/Agent-Skills-blue?style=flat&logo=github)](https://agentskills.io)
[![Android](https://img.shields.io/badge/Android-Modern-green?style=flat&logo=android)](https://developer.android.com)

Welcome to **Awesome Android Agent Skills**, a repository of specialized "brains" for your AI coding assistants. This project provides a suite of **Agent Skills** designed to supercharge GitHub Copilot, Claude, Google Gemini, Cursor and other agentic AI tools with expert knowledge of modern Android development.

##  What are Agent Skills?

**Agent Skills** are a standardized way to package capabilities, instructions, and best practices for AI agents. Instead of pasting the same prompt repeatedly ("How do I implement MVVM?", "Check this for accessibility"), you install these skills into your agent's environment.

When an agent detects you are working on a relevant task (e.g., "Create a verified repository"), it automatically loads the expert instructions from the corresponding `SKILL.md` file. This ensures:
*   **Consistency**: The agent always follows your defined architecture.
*   **Accuracy**: It uses the latest 2025 best practices (Compose, Hilt, Room).
*   **Efficiency**: No need for long context-stuffing prompts.

> Learn more at [agentskills.io](https://agentskills.io).

## How Skills Work Together

Skills reference each other and build on shared context. The `Agent.md` file is the foundation — every AI agent interacting with your project reads it implicitly to understand your project's architecture, minimum SDK, and testing philosophy before executing any specific tasks.

```text
       ┌───────────────────────────────────────────┐
       │                  Agent.md                 │
       │     (read by all AI agents implicitly)    │
       └─────────────────────┬─────────────────────┘
                             │
     ┌───────────────┬───────┴───────┬───────────────┐
     ▼               ▼               ▼               ▼
┌──────────┐   ┌───────────┐   ┌────────────┐  ┌────────────┐
│ Architect│   │ UI & Nav  │   │ Migration  │  │ Automation │
├──────────┤   ├───────────┤   ├────────────┤  ├────────────┤
│viewmodel │   │compose-ui │   │rx-to-corout│  │testing     │
│data-layer│   │navigation │   │xml2compose │  │emulator    │
│clean-arch│   │coil-image │   │            │  │            │
└──────────┘   └───────────┘   └────────────┘  └────────────┘
```

##  Available Skills

These skills are categorized into domains in `.github/skills/` to provide specialized instructions for compatible agents.


### Architecture
*   **[Android Architecture](.github/skills/architecture/android-architecture/SKILL.md)** - Expert guidance on **Clean Architecture**, **Modularization**, and **Dependency Injection** with **Hilt**.
*   **[ViewModel & State](.github/skills/architecture/android-viewmodel/SKILL.md)** - Proper implementation using `StateFlow` and `SharedFlow`.
*   **[Data Layer & Offline-First](.github/skills/architecture/android-data-layer/SKILL.md)** - **Repository Pattern** with **Room** and **Retrofit**.

### UI (Compose & Legacy)
*   **[Jetpack Compose UI](.github/skills/ui/compose-ui/SKILL.md)** - Best practices for stateless, performant Composables.
*   **[Compose Navigation](.github/skills/ui/compose-navigation/SKILL.md)** - Type-safe navigation, deep links, and nested graphs.
*   **[Coil for Jetpack Compose](.github/skills/ui/coil-compose/SKILL.md)** - Expert guidance on image loading in Compose.
*   **[Accessibility](.github/skills/ui/android-accessibility/SKILL.md)** - Auditing content descriptions, touch targets, and contrast.

### Migration
*   **[XML to Compose Migration](.github/skills/migration/xml-to-compose-migration/SKILL.md)** - Converting XML layouts to idiomatic Jetpack Compose.
*   **[RxJava to Coroutines Migration](.github/skills/migration/rxjava-to-coroutines-migration/SKILL.md)** - Migrating asynchronous code from RxJava to Coroutines and Flow.

### Performance
*   **[Compose Performance Audit](.github/skills/performance/compose-performance-audit/SKILL.md)** - Identify recomposition storms and unstable keys.
*   **[Gradle Build Performance](.github/skills/performance/gradle-build-performance/SKILL.md)** - Debug and optimize Gradle/Android build times.

### Concurrency & Networking
*   **[Kotlin Concurrency Expert](.github/skills/concurrency_and_networking/kotlin-concurrency-expert/SKILL.md)** - Review and fix Kotlin Coroutines issues.
*   **[Android Coroutines](.github/skills/concurrency_and_networking/android-coroutines/SKILL.md)** - Best practices for Coroutines execution and scopes.
*   **[Retrofit Networking](.github/skills/concurrency_and_networking/android-retrofit/SKILL.md)** - Guidance on Retrofit, OkHttp, and Coroutines networking.

### Testing & Automation
*   **[Testing & Screenshots](.github/skills/testing_and_automation/android-testing/SKILL.md)** - Setup for Unit, Hilt, and Screenshot Testing.
*   **[Android Emulator Automation](.github/skills/testing_and_automation/android-emulator-skill/SKILL.md)** - Essential production scripts for automation and semantic navigation.

### Build & Tooling
*   **[Gradle Build Logic](.github/skills/build_and_tooling/android-gradle-logic/SKILL.md)** - Setup Convention Plugins, Version Catalogs, and Composite Builds.

## Setup in Your Project

To equip your AI agent with these skills, you must place them in a location where the agent can discover them.

### Standard Location (VS Code / GitHub Copilot)
The industry standard location for agent skills is the `.github/skills/` directory at the root of your workspace.

1.  **Copy Skills**: Copy the `.github/skills/` folder from this repository to your project's root.
2.  **Copy Context**: Copy the `Agent.md` file from this repository to your project's root. This file ensures the agent always abides by your architecture and testing standards.
3.  **Verify**: Ensure your project structure looks like this:
    ```text
    my-android-project/
    ├── Agent.md
    ├── .github/
    │   └── skills/
    │       ├── architecture/
    │       │   ├── android-architecture/
    │       │   │   └── SKILL.md
    │       │   └── ...
    ├── app/
    └── ...
    ```
3.  **Restart**: specific extensions (like Copilot) may need a window reload to index the new skills.

### Other Environments

*   **Claude / Anthropic**: Legacy or direct Claude usage often looks for `.claude/skills/`. You can copy or symlink `.github/skills` to `.claude/skills`.
*   **OpenCode**: Supports `.opencode/skill/` (note singular `skill`) and `.claude/skills/`. Global skills can be placed in `~/.config/opencode/skill/`.

### Creating Custom Skills
To add your own skill:
1.  Create a new folder in `.github/skills/` (e.g., `my-custom-skill`).
2.  Add a `SKILL.md` file with the required frontmatter:
    ```markdown
    ---
    name: my-custom-skill
    description: Description of what this skill does
    ---
    # Instructions
    ...
    ```
## Usage

### GitHub Copilot
If this repository is part of your workspace, you can simply ask Copilot:
> "How should I structure the new User Profile feature?"
> "Create a repository for fetching News with offline support."

Copilot will detect the relevant skill (e.g., `android-architecture` or `android-data-layer`) and apply the rules defined in the `SKILL.md` files.

### Manual / Other Agents
You can also point any context-aware LLM (like ChatGPT or Claude) to the specific `SKILL.md` file you need help with.
> "Read `.github/skills/compose-ui/SKILL.md` and then refactor this screen."

##  Topics & Keywords

Android Development, Agent Skills, AI Coding Assistants, Jetpack Compose, Clean Architecture, MVVM, MVI, Hilt Dependency Injection, Room Database, Retrofit, Offline-First, Kotlin Coroutines, StateFlow, SharedFlow, Android Accessibility, Semantic Trees, Modularization, Mobile DevOps, GenAI for Mobile.

---

*Original "Studio-Bot-Prompts-Handbook" content has been superseded by these executable Agent Skills.*


## Related Resources
- [Skill Hub](https://skill.442595.xyz/) — 5800+ curated AI Agent Skills for Claude Code, Codex, Cursor, Hermes & more across 22 categories.

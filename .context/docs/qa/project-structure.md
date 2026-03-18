---
slug: project-structure
category: architecture
status: filled
generatedAt: 2026-03-18T21:32:54.231Z
relevantFiles:
  - src/index.ts
  - src/generators
  - src/services
  - src/utils
  - src/types.ts
  - src/workflow
  - src/version.ts
---

# How is the codebase organized?

## Top-level layout

```
.context/          # Generated documentation and agent config (output of the tool itself)
.github/workflows/ # CI (ci.yml) and release (release.yml) pipelines
dist/              # Compiled JavaScript output (git-ignored)
prompts/           # Prompt templates used by generators
scripts/           # Build/utility scripts
src/               # TypeScript source code (all application logic)
```

## Source directory (`src/`)

### Entry point

- **`src/index.ts`** -- CLI entry point. Registers all Commander commands, wires up services via dependency injection, handles interactive mode, and locale detection.

### Core directories

| Directory | Purpose |
| --- | --- |
| `src/generators/` | Content generation engines (agents, documentation, plans, skills) |
| `src/services/` | Business logic organized by feature domain |
| `src/utils/` | Shared utilities (CLI UI, i18n, theme, file mapping, frontmatter parsing) |
| `src/types/` | Shared TypeScript type definitions |
| `src/workflow/` | PREVC workflow engine (phases, roles, gates, orchestration, collaboration) |
| `src/prompts/` | Prompt-related modules |

### Services (`src/services/`)

Each service is a self-contained module with its own directory:

| Service | Purpose |
| --- | --- |
| `ai/` | LLM provider abstraction (OpenAI, Anthropic, Google) |
| `autoFill/` | Automatic content filling logic |
| `customAgents/` | Custom agent management |
| `export/` | Rule export for AI coding tools (Cursor, Windsurf, etc.) |
| `fill/` | AI-driven documentation filling |
| `import/` | Import rules and agents from external sources |
| `init/` | Project initialization and scaffolding |
| `mcp/` | MCP (Model Context Protocol) server for AI tool integration |
| `passthrough/` | JSON-over-stdin/stdout protocol for external agent communication |
| `plan/` | Implementation plan generation |
| `qa/` | Q&A document generation |
| `quickSync/` | Quick context synchronization |
| `report/` | Context report generation |
| `reverseSync/` | Reverse synchronization (merge external changes back) |
| `semantic/` | Semantic code analysis (optional tree-sitter) |
| `serve/` | HTTP serve mode |
| `shared/` | Shared types, context root resolution, common utilities |
| `stack/` | Technology stack detection |
| `start/` | Interactive guided start flow |
| `state/` | Project state detection |
| `sync/` | Agent synchronization |
| `tools/` | Tool definitions |
| `update/` | Update existing documentation |
| `workflow/` | Workflow service layer |

### Generators (`src/generators/`)

| Generator | Purpose |
| --- | --- |
| `agents/` | Agent playbook generation |
| `documentation/` | Documentation template generation |
| `plans/` | Implementation plan generation |
| `shared/` | Shared generator utilities |
| `skills/` | Skill document generation |

### Utilities (`src/utils/`)

| Module | Purpose |
| --- | --- |
| `cliUI.ts` | CLI output helpers (spinners, banners, tables) using chalk and ora |
| `i18n.ts` | Internationalization (English and Brazilian Portuguese) |
| `theme.ts` | Color theme constants |
| `frontMatter.ts` | YAML frontmatter parsing and serialization |
| `contentSanitizer.ts` | Content sanitization for generated output |
| `fileMapper.ts` | File discovery and mapping |
| `gitService.ts` | Git repository utilities |
| `promptLoader.ts` | Prompt template loading from `prompts/` directory |
| `versionChecker.ts` | npm registry version checking for update notifications |
| `prompts/` | Interactive prompt helpers (config summary, LLM config, analysis options) |

## Key architectural patterns

- **Dependency injection**: Services receive their dependencies (`ui`, `t`, `version`) via constructor options conforming to `BaseDependencies` or `AIDependencies` from `src/services/shared/types.ts`.
- **Service isolation**: Each feature lives in its own service directory with an index barrel export.
- **i18n throughout**: All user-facing strings go through the `TranslateFn` (`t`) function.
- **Zod validation**: Request schemas (especially in passthrough protocol) use Zod for runtime validation.

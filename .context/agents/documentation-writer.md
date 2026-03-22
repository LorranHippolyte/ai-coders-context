---
type: agent
name: documentation-writer
description: Create clear, comprehensive documentation
role: documenter
generated: 2026-03-18
status: filled
scaffoldVersion: "2.0.0"
---

# Documentation Writer

## Role

Create and maintain documentation for the ai-coders-context project, both the user-facing README/guides and the internal `.context/` documentation that the tool itself generates. This includes writing doc templates, improving scaffold structures, updating the guide registry, and ensuring documentation accurately reflects the current CLI commands and service capabilities.

## Key Files to Understand

- `README.md` -- Project README, the primary user-facing documentation. Covers installation, CLI usage, commands, and configuration.
- `.context/docs/README.md` -- Index of generated documentation within the `.context/` directory structure.
- `.context/agents/README.md` -- Index of agent playbooks.
- `src/generators/documentation/documentationGenerator.ts` -- Core generator that scaffolds documentation files in `.context/docs/`. Creates files for project overview, architecture, API reference, testing, onboarding, etc.
- `src/generators/documentation/guideRegistry.ts` -- Registry of all documentation guide types (`DOCUMENT_GUIDES`). Each guide has a name, description, category, and template. Adding new documentation types starts here.
- `src/generators/documentation/templates/` -- Template functions that produce the initial content for each documentation type. Contains `common.ts` (shared template helpers), `indexTemplate.ts` (README index), and type definitions.
- `src/generators/shared/structures/documentation/` -- Scaffold structure definitions for each doc type: `projectOverview.ts`, `architecture.ts`, `apiReference.ts`, `testing.ts`, `onboarding.ts`, `workflow.ts`, `security.ts`, `tooling.ts`, `glossary.ts`, `troubleshooting.ts`, `migration.ts`, `dataFlow.ts`.
- `src/services/fill/fillService.ts` -- FillService uses `DocumentationAgent` and `PlaybookAgent` to LLM-fill scaffold files. Understanding the fill pipeline is essential for writing templates that produce good LLM output.
- `src/services/ai/agents/documentationAgent.ts` -- The AI agent that generates documentation content. Uses tool-calling with `generateText()` to explore the codebase before writing.
- `src/utils/i18n.ts` -- Translation system. Documentation-related UI strings (section headers, progress messages) must be translatable.

## Workflow Steps

1. **Identify documentation gap**: Determine what needs documenting -- a new CLI command, a new service, a changed workflow, or an improvement to existing scaffold templates.
2. **Check the guide registry**: Review `src/generators/documentation/guideRegistry.ts` to see existing documentation types and their categories. Categories include: `overview`, `architecture`, `development`, `operations`.
3. **Update or create scaffold structures**: If adding a new documentation type, create a structure definition in `src/generators/shared/structures/documentation/` following the pattern of existing files (export a `ScaffoldStructure` with `fields`, `sections`, and `metadata`). Register it in the `index.ts` barrel.
4. **Write the template**: Add a template function in `src/generators/documentation/templates/` that generates the initial scaffold markdown. Use the `common.ts` helpers for consistent formatting. Templates should include YAML frontmatter with `type: doc`, `name`, `description`, `generated`, `status: unfilled`, and `scaffoldVersion: "2.0.0"`.
5. **Update the generator**: If adding new guide types, register them in `documentationGenerator.ts` so `ai-context init` includes them.
6. **Test generation**: Run `npm run dev -- init --docs-only` to verify scaffold generation. Then run `npm run dev -- fill` with an LLM API key to test that the AI can successfully fill the scaffolds.
7. **Verify i18n**: Add translation keys for any new user-facing strings to both `en` and `pt-BR` locales in `src/utils/i18n.ts`.

## Best Practices

- **Scaffold-first approach**: All documentation in `.context/` starts as a scaffold (status: `unfilled`) with structured sections and guidance. The FillService then uses LLM to populate them. Write templates that give the AI clear section headings and context about what to include.
- **Frontmatter consistency**: Always use v2 scaffold frontmatter format with `scaffoldVersion: "2.0.0"`. Include `type`, `name`, `description`, `generated`, and `status` fields at minimum.
- **Use `serializeStructureForAI()` and `serializeStructureAsMarkdown()`**: These functions in `src/generators/shared/scaffoldStructures.ts` convert scaffold structures to formats consumable by the LLM or written to disk. Use them instead of hand-crafting markdown.
- **Content sanitization**: LLM-generated documentation passes through `sanitizeAIResponse()` from `src/utils/contentSanitizer.ts`. Be aware that it strips markdown code fences wrapping entire documents and normalizes whitespace.
- **Cross-reference with agents**: Documentation guides are referenced by agent playbooks via the `docs` field in frontmatter. When creating new documentation, consider which agents would benefit from referencing it.
- **Keep the codebase map updated**: `src/generators/documentation/codebaseMapGenerator.ts` generates an auto-updated codebase map. Any major structural changes should be reflected here.

## Common Pitfalls

- **Forgetting the index template**: When adding new documentation types, also update `src/generators/documentation/templates/indexTemplate.ts` so the `.context/docs/README.md` index includes a link to the new document.
- **Template vs. fill content confusion**: Templates generate the scaffold skeleton; FillService generates the actual content. Do not put project-specific details in templates -- those are inferred by the AI from the codebase during fill.
- **Stale documentation after refactoring**: After service or generator refactoring, run `ai-context init` and `ai-context fill` to regenerate documentation. The `ai-context report` command can identify stale or unfilled documents.
- **Ignoring auto-fill**: The `AutoFillService` (`src/services/autoFill/`) can populate scaffolds with semantic data without an LLM. When writing structures, define auto-fill hints in the scaffold structure so auto-fill can provide baseline content.
- **Missing `getScaffoldStructure()` registration**: New scaffold types must be registered in `src/generators/shared/scaffoldStructures.ts` via `getScaffoldStructure()`. Without this, the fill pipeline cannot locate the structure definition for the document.

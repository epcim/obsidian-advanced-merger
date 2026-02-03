# AGENTS.md - Obsidian Advanced Merger

> AI coding agent guidelines for obsidian-advanced-merger plugin

## Project Overview

This is an **Obsidian plugin** written in TypeScript that merges multiple markdown notes from a folder into a single document. The plugin integrates with Obsidian's file menu context system.

**Tech Stack:** TypeScript, esbuild, Obsidian API, Prettier, ESLint, Husky

---

## Build & Development Commands

```bash
# Install dependencies (uses pnpm)
pnpm install

# Development mode with watch
pnpm dev          # Runs esbuild with file watching

# Production build
pnpm build        # Runs tsc type-check + esbuild production bundle

# Format code (run before commits)
npx prettier . --write
```

### Testing

**Note:** This project currently has no test suite configured. There are no `*.test.ts` files or test runner dependencies.

If adding tests:

- Consider using `vitest` or `jest`
- Place test files adjacent to source: `src/main.test.ts`
- Run single test pattern: `vitest run src/main.test.ts`

---

## Code Style Guidelines

### Formatting (Prettier + EditorConfig)

- **Indentation:** Tabs (width: 4)
- **Line endings:** LF (Unix-style)
- **Final newline:** Required
- **Charset:** UTF-8
- **Prettier:** Auto-runs on pre-commit via Husky

### TypeScript Configuration

- **Target:** ES6 with ESNext modules
- **Strict null checks:** Enabled (`strictNullChecks: true`)
- **No implicit any:** Enabled (`noImplicitAny: true`)
- **Source maps:** Inline (dev only)
- **Module resolution:** Node

### ESLint Rules

- Extends: `eslint:recommended`, `@typescript-eslint/recommended`
- Unused vars: Error (but args allowed to be unused)
- `@ts-comment` directives: Allowed
- Empty functions: Allowed
- Prototype builtins: Allowed

---

## Code Patterns & Conventions

### File Organization

```
src/
├── main.ts        # Plugin entry point, main class, UI components
├── settings.ts    # Settings interface & defaults
├── constants.ts   # String constants, magic values
└── translation.ts # i18n translations
```

### Naming Conventions

| Type            | Convention                | Example                                      |
| --------------- | ------------------------- | -------------------------------------------- |
| Classes         | PascalCase                | `AdvancedMerge`, `AdvancedMergeSettingTab`   |
| Interfaces      | PascalCase                | `AdvancedMergePluginSettings`, `Translation` |
| Constants       | SCREAMING_SNAKE_CASE      | `ICON_NAME`, `NEW_LINE_CHAR`                 |
| Methods         | camelCase                 | `onClickCallback`, `mergeNotes`              |
| Private methods | camelCase (no underscore) | `filterNotes`, `sortNotes`                   |
| Properties      | camelCase                 | `includeNestedFolders`                       |

### Import Style

```typescript
// External imports first (from libraries)
import { App, Modal, Plugin, Setting, TFile, TFolder, Vault } from "obsidian";

// Internal imports second (relative paths)
import { AdvancedMergePluginSettings, DEFAULT_SETTINGS } from "./settings";
import { ICON_NAME, PLUGIN_NAME } from "./constants";
```

### Class Visibility Modifiers

- Use `public` explicitly for public methods
- Use `private` for internal methods
- Properties: `public` when accessed externally

```typescript
export default class AdvancedMerge extends Plugin {
	public settings: AdvancedMergePluginSettings;

	public async onload(): Promise<void> {}
	private async mergeNotes(): Promise<void> {}
}
```

### Async/Await Pattern

- Always use `async/await` over raw Promises
- Return `Promise<void>` for async methods with no return value
- Callbacks should be async when performing I/O

```typescript
private async loadSettings(): Promise<void> {
    const data = await this.loadData();
    this.settings = Object.assign({}, DEFAULT_SETTINGS, data);
}
```

### Type Definitions

- Define interfaces for complex objects
- Use union types for constrained string values
- Export types that are used across modules

```typescript
export interface AdvancedMergePluginSettings {
	sortMode: "alphabetical" | "creationDate" | "logical";
	includeNestedFolders: boolean;
}
```

### JSDoc Comments

Use JSDoc for public methods and classes:

```typescript
/**
 * Merges input notes to a single output file.
 * @param vault - Current vault.
 * @param entries - Files and folders to be included.
 * @param outputFileName - Output file name.
 */
private async mergeNotes(vault: Vault, entries: Array<TAbstractFile>, outputFileName: string): Promise<void>
```

---

## Error Handling

- Use `console.log/info` for debug logging
- No try-catch blocks in current codebase (Obsidian handles most errors)
- Use early returns with type guards (`instanceof` checks)

```typescript
if (!(file instanceof TFolder)) {
	return;
}
```

---

## Commit Message Format

**Angular-style commits required:**

```
<type>(<issue-id>): <short summary>

<optional body>
```

### Types

| Type       | Purpose                      |
| ---------- | ---------------------------- |
| `feat`     | New feature                  |
| `fix`      | Bug fix                      |
| `docs`     | Documentation only           |
| `refactor` | Code change (no feature/fix) |
| `test`     | Adding/correcting tests      |
| `build`    | Build system or dependencies |

### Examples

```
feat(#123): add dark mode support
fix(#456): resolve merge order issue with nested folders
refactor: extract file filtering to separate method
```

### Summary Rules

- Imperative mood: "add" not "added"
- No capitalization
- No period at end

---

## Important Files

| File                 | Purpose                      | Modify?                         |
| -------------------- | ---------------------------- | ------------------------------- |
| `manifest.json`      | Plugin metadata for Obsidian | **NO** - breaks plugin download |
| `manifest-beta.json` | Beta releases via BRAT       | Yes for beta                    |
| `main.js`            | Production bundle output     | Generated                       |
| `versions.json`      | Version compatibility map    | Auto-updated                    |

---

## Obsidian Plugin Patterns

### Plugin Lifecycle

```typescript
export default class MyPlugin extends Plugin {
    async onload(): Promise<void> {
        // Register events, commands, settings tabs
        this.registerEvent(...)
        this.addSettingTab(...)
    }
}
```

### Settings Pattern

```typescript
// In settings.ts
export const DEFAULT_SETTINGS: SettingsType = { ... };

// In main.ts
this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
await this.saveData(this.settings);
```

### Modal Dialog Pattern

```typescript
class MyModal extends Modal {
	onOpen(): void {
		/* build UI */
	}
	onClose(): void {
		contentEl.empty();
	}
}
```

---

## Pre-commit Hook

Husky runs Prettier automatically on commit:

```bash
npx prettier . --write
```

Ensure code is formatted before pushing to avoid CI failures.

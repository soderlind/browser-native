---
name: browser-native
description: "Use when the user asks to audit, scan, or modernize JavaScript dependencies: identifies npm packages that can be replaced with modern browser/runtime built-in APIs (fetch, URL, structuredClone, crypto.randomUUID, Intl, etc.) and shows before/after code examples."
compatibility: "Node.js 18+. Filesystem-based — reads package.json. No network or browser access required."
---

# Browser-Native Dependency Scanner

## When to use

Use this skill when the user:

- asks to audit, scan, or check JavaScript/Node.js dependencies for replacements
- wants to modernize a project by removing polyfills and outdated utility packages
- asks "which of my deps can be replaced with browser built-ins?"
- wants to know if packages like axios, lodash, moment, uuid, etc. are still needed
- is trying to reduce bundle size by switching to native APIs
- asks about browser-native alternatives to a specific npm package

## Inputs required

- A target directory (or the current working directory) that contains a `package.json`.
- Optionally: desired output format (terminal table, markdown report, or JSON).

## Procedure

### 1) Run the scanner script

```bash
node skills/browser-native/scripts/cli.js [target-dir]
```

Default output is a colored terminal table showing each replaceable dependency, its category, the native API replacement, and a confidence level (full or partial).

#### Output formats

```bash
# Terminal table (default)
node skills/browser-native/scripts/cli.js [dir]

# Markdown with before/after code examples
node skills/browser-native/scripts/cli.js [dir] --md

# JSON for parsing
node skills/browser-native/scripts/cli.js [dir] --json

# Save to file
node skills/browser-native/scripts/cli.js [dir] --md --out report.md
```

### 2) Review the results

The scanner checks `dependencies` and `devDependencies` against an internal database of **70+ npm packages** that have native browser/runtime equivalents, organized in these categories:

| Category | Example packages | Native replacement |
|---|---|---|
| HTTP | axios, node-fetch, request, got | `fetch()` |
| URL / Query | query-string, qs, url-parse | `URL`, `URLSearchParams` |
| Object Utils | lodash.clonedeep, object-assign, has | `structuredClone()`, `Object.assign()`, `Object.hasOwn()` |
| Array Utils | lodash.flatten, lodash.find, lodash.uniq | `.flat()`, `.find()`, `[...new Set()]` |
| UUID | uuid, nanoid, shortid | `crypto.randomUUID()` |
| Date | moment, moment-timezone | `Intl.DateTimeFormat`, `Intl.RelativeTimeFormat` |
| Promises | bluebird, q, es6-promise | `Promise` |
| Strings | left-pad, repeat-string | `.padStart()`, `.repeat()` |
| Type checks | is-number, isarray, is-promise | `typeof`, `Array.isArray()` |
| Encoding | base-64, js-base64 | `btoa()`, `atob()` |
| Polyfills | abort-controller, text-encoding, globalthis | Native globals |
| Observers | intersection-observer, resize-observer-polyfill | Native APIs |
| Crypto | crypto-js (partial) | `crypto.subtle` |
| Streams | web-streams-polyfill | `ReadableStream` |
| FormData | form-data, formdata-polyfill | `FormData` |

### 3) Interpret confidence levels

- **Full** — drop-in replacement. The native API covers the same functionality. Safe to remove the package and use the native API directly.
- **Partial** — covers most common use cases, but the package may offer features the native API doesn't. Review your usage before removing.

For detailed reference on each replacement including before/after code and browser support, read:
- `references/replacements-guide.md`

### 4) Present findings to the user

When showing results:
1. Lead with the summary count (e.g., "14 of 42 dependencies can be replaced")
2. Group by confidence: list full replacements first (easy wins), then partial
3. For each flagged package, show the before/after code snippet
4. Note any caveats from the `notes` field  
5. If asked for a migration plan, prioritize:
   - Polyfills first (safest to remove — they just provide what's already built-in)
   - Full confidence replacements next
   - Partial replacements last (require careful review)

### 5) Monorepo support

The scanner automatically checks `packages/`, `apps/`, `libs/`, and `modules/` subdirectories for additional `package.json` files.

## Verification

After presenting recommendations, the user can verify by:
1. Removing the flagged package from `package.json`
2. Replacing imports with the native API (using the "after" code example)
3. Running the project's test suite
4. Checking browser compatibility against their targets

## Failure modes

- **No package.json found** — the script will print an error. Ask the user for the correct project directory.
- **Zero replaceable deps** — this is a good result! The project is already modern.
- **Package in database but used for edge-case features** — confidence "partial" covers this. Always check the `notes` field and recommend reviewing actual usage before removing.

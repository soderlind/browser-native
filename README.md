# browser-native

Scan your JavaScript project's dependencies and find which ones can be replaced with modern browser/runtime built-in APIs.

**Zero dependencies.** This tool practices what it preaches.

## The skill has moved

This skill now lives in the consolidated skills repo:

https://github.com/soderlind/skills/


## Install as a CLI

```bash
npm install -g @soderlind/browser-native
```

Or run directly with npx:

```bash
npx @soderlind/browser-native
```

## CLI Usage

```bash
# Scan current project
browser-native

# Scan a specific directory
browser-native ../my-app

# Output as Markdown with before/after code examples
browser-native --md

# Save markdown report to file
browser-native --md --out report.md

# Output as JSON (pipe to jq, etc.)
browser-native --json
browser-native --json | jq '.summary'

# Show help
browser-native --help
```

## What it detects

The tool checks your `dependencies` and `devDependencies` against a database of **[100+ npm packages](skills/browser-native/references/replacements-guide.md)** that have native browser/runtime equivalents:

| Category | Example packages | Native replacement |
|---|---|---|
| HTTP | axios, node-fetch, request, unfetch | `fetch()` |
| URL / Query | query-string, qs, url-parse, url-search-params-polyfill | `URL`, `URLSearchParams` |
| Object Utils | lodash.clonedeep, object-assign, object.values, object.hasown | `structuredClone()`, `Object.assign()`, `Object.values()`, `Object.hasOwn()` |
| Array Utils | lodash.flatten, lodash.find, lodash.uniq | `.flat()`, `.find()`, `[...new Set()]` |
| UUID | uuid, nanoid, shortid | `crypto.randomUUID()` |
| Date | moment, moment-timezone | `Intl.DateTimeFormat` |
| Promises | bluebird, q, promise-polyfill, promise.allsettled | `Promise`, `Promise.allSettled()` |
| Strings | left-pad, repeat-string, string.prototype.matchall | `.padStart()`, `.repeat()`, `.matchAll()` |
| Type checks | is-number, isarray, is-promise | `typeof`, `Array.isArray()` |
| Encoding | base-64, js-base64 | `btoa()`, `atob()` |
| Polyfills | abort-controller, text-encoding, globalthis | Native globals |
| Observers | intersection-observer, resize-observer-polyfill | Native APIs |
| Crypto | crypto-js (partial) | `crypto.subtle` |
| Streams | web-streams-polyfill | `ReadableStream` |
| FormData | form-data, formdata-polyfill | `FormData` |

## Output formats

### Terminal (default)

Colored table with package name, category, replacement API, and confidence level.

### Markdown (`--md`)

Detailed report grouped by category, with:
- Before/after code examples
- Browser support tables (Chrome, Firefox, Safari, Edge, Node.js)
- Confidence indicators and caveats

### JSON (`--json`)

Structured output for piping into other tools:

```json
{
  "summary": {
    "scannedFiles": 1,
    "totalDependencies": 42,
    "replaceableCount": 8,
    "fullReplacements": 5,
    "partialReplacements": 3
  },
  "replaceable": [...]
}
```

## Confidence levels

- **Full** — drop-in replacement. The native API covers the same functionality.
- **Partial** — covers most common use cases, but the package may have features the native API doesn't (e.g., `qs` supports nested objects, `URLSearchParams` does not).

## CI usage

The tool exits with code **1** if replaceable dependencies are found, **0** if none. Use in CI to flag outdated polyfills:

```yaml
- run: npx @soderlind/browser-native
  continue-on-error: true
```

## Monorepo support

Automatically scans `packages/`, `apps/`, `libs/`, and `modules/` subdirectories for additional `package.json` files.

## Requirements

Node.js 18+

## Project structure

```
skills/
  browser-native/
    SKILL.md                        # Copilot skill definition
    scripts/
      cli.js                        # CLI entry point
      scanner.js                    # Package.json scanner
      replacements.js               # Database of 100+ replaceable packages
      formatters/
        table.js                    # ANSI terminal table
        markdown.js                 # Markdown report with code examples
        json.js                     # JSON output
    references/
      replacements-guide.md         # Detailed replacement reference
```

## License

MIT

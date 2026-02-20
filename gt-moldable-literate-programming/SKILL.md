---
name: gt-moldable-literate-programming
description: >
  Guide for implementing functionality in GToolkit/Pharo following the "moldable literate
  programming" style. Creates Lepiter pages where readers understand a feature by evaluating
  Smalltalk snippets top-to-bottom: interactive API exploration first, then a snippet that
  compiles the production class, then a snippet that compiles gtExample tests.
  Use when the user says things like: "implementa con moldable literate programming",
  "desarrolla usando live literate programming", "crea una funcionalidad usando literate
  programming con Lepiter", "crea una página Lepiter para implementar X", or any request
  to build a feature in GToolkit following a literate/exploratory style.
---

# GToolkit Moldable Literate Programming

## Workflow

1. **Clarify** — Understand what class/feature to implement and what the production class should do
2. **Explore the API** — Identify which GT classes to use; look them up via MCP eval if needed
3. **Build the page** — Create a Lepiter page with the four-section pattern (see below)
4. **Validate** — Verify snippets work by running them via the MCP
5. **Update ToC** - If the knowledge database where the Lepiter page was created has a Table of Contents,
update the ToC adding the link to the new creted page.

## Page Structure (four sections)

See `references/page-structure.md` for the full pattern with snippet counts, naming
conventions, and gtExample rules.

Quick summary — snippets must be evaluable top-to-bottom in this order:

1. **Introduction** (text) — what the page builds and why
2. **Interactive exploration** (alternating text + Pharo) — explore GT API objects interactively
3. **Production class** (text + **two** Pharo snippets):
   - Snippet A: class definition only (`Object subclass: #MyClass ...`)
   - Snippet B: method definitions (`MyClass compile: '...' classified: '...'`)
4. **Examples/tests** (text + **two** Pharo snippets + example snippets):
   - Snippet A: examples class definition only
   - Snippet B: example method definitions (`MyClassExamples compile: ...`)
   - One `LeExampleSnippet` per `<gtExample>` method

> **Critical**: `compile:classified:` fails if the class doesn't exist yet. Always put the
> `subclass:` definition in a **separate snippet before** any `compile:` calls.

## Using the MCP to Build the Page

Use the MCP tools directly to create pages and add snippets — no shell commands needed:

1. **Create the page**: use `mcp__gtoolkit-dynaspace__create-page` with `dbName` and `pageTitle`.
2. **Add each snippet**: use `mcp__gtoolkit-dynaspace__add-snippet` with:
   - `dbName`, `pageTitle`
   - `snippetType`: `text`, `pharo`, or `example`
   - `content`: the snippet body (for `example` type: `"ClassName>>selector"`)
3. **Verify / inspect**: use `mcp__gtoolkit-dynaspace__get-page` or `mcp__gtoolkit-dynaspace__list-pages`.
4. **Custom Smalltalk**: use `mcp__gtoolkit-dynaspace__eval` for operations not covered above
   (e.g., removing the default blank snippet, reordering, introspecting structure).

See `references/lepiter-mcp-api.md` for Smalltalk snippets for advanced operations.

### Quoting rule for compile: snippets

Inside a Pharo snippet that uses `compile:classified:`, string literals in the method
source must use doubled single quotes:

```smalltalk
MyClass compile: 'greet
    ^ ''Hello, world!''' classified: 'accessing'.
```

When passing such code via `mcp__gtoolkit-dynaspace__add-snippet`, write the content
exactly as valid Smalltalk — the MCP tool handles transport encoding automatically.

## Connecting to GToolkit

Verify connectivity with a quick eval before building the page:

```
mcp__gtoolkit-dynaspace__eval  code: "1+1"
```

If the tool is unavailable, ask the user to evaluate the startup snippet in GToolkit first
(the one on the "Servidor MCP para GToolkit" Lepiter page).

As a fallback when the MCP tools are not available, the Python script can be used:

```bash
python3 scripts/add_snippets.py "Page Title" "DynaSpaceOS" < snippets.json
```

## Naming Conventions (DynaSpaceOS project)

- Production class: `DynOS` prefix — e.g., `DynOSMcpEvalServer`
- Examples class: same + `Examples` — e.g., `DynOSMcpEvalServerExamples`
- Package: `DynaSpaceOS-<Feature>` — e.g., `DynaSpaceOS-MCP`
- Target database: `DynaSpaceOS`

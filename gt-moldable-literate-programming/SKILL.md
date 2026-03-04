---
name: gt-moldable-literate-programming
description: >
  Guide for implementing functionality in GToolkit/Pharo following the "moldable literate
  programming" style. Creates a single Lepiter page where readers understand a feature by
  evaluating Smalltalk snippets top-to-bottom: interactive API exploration first, then one
  section per EDD iteration (each with its failing example, exploration, minimum
  implementation, and live verification).
  Use when the user says things like: "implementa con moldable literate programming",
  "desarrolla usando live literate programming", "crea una funcionalidad usando literate
  programming con Lepiter", "crea una página Lepiter para implementar X", or any request
  to build a feature in GToolkit following a literate/exploratory style.
---

# GToolkit Moldable Literate Programming

## Workflow

1. **Clarify** — Understand what class/feature to implement and what the production class should do
2. **Explore the API** — Identify which GT classes to use; look them up via MCP eval if needed
3. **Build the page** — Create a Lepiter page with the three-section pattern (see below); for each EDD iteration, keep the generated code in the image until all sections are documented
4. **Undo** — Remove all generated classes from the image (`MyClass removeFromSystem`, `MyClassExamples removeFromSystem`)
5. **Validate** — Re-evaluate all page snippets top-to-bottom via MCP to confirm the page is self-contained and recreates the code from scratch
6. **Undo again** — Remove all generated classes from the image a second time, leaving only the Lepiter page as the sole source of truth
7. **Update ToC** — If the knowledge database has a Table of Contents, add a link to the new page

## Page Structure (three sections)

See `references/page-structure.md` for the full pattern with snippet counts, naming
conventions, and gtExample rules.

Quick summary — snippets must be evaluable top-to-bottom in this order:

1. **Introduction** (text) — what the page builds and why
2. **Interactive exploration** (alternating text + Pharo) — explore GT API objects interactively
3. **Example-driven Development**  as many sections as iterations of example-driven development

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

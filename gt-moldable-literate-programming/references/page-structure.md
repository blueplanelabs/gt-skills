# Moldable Literate Programming — Page Structure

A "moldable literate programming" Lepiter page lets readers understand AND build a feature
by evaluating snippets top-to-bottom. The page both explains concepts and generates all code.

## The Three-Section Pattern

```
1. INTRODUCTION        (text only)
   ↓
2. INTERACTIVE EXPLORATION  (text + Pharo snippets — understand the API)
   ↓
3. EXAMPLE-DRIVEN DEVELOPMENT SECTIONS
    a. Invoke `gt-example-driven-development` to execute the code cycle (failing example → explore → implement → verify)
    b. Create one Example-driven Development section documenting that iteration  (see **Example-driven Development Section Structure** below)
    (repeat for each iteration, keeping generated code in the image)
4. UNDO — Remove all generated classes from the image (MyClass removeFromSystem, MyClassExamples removeFromSystem)
5. VALIDATE — Re-evaluate all page snippets top-to-bottom via MCP to confirm the page is self-contained
6. UNDO AGAIN — Remove all generated classes from the image a second time, leaving only the Lepiter page
```

> **Critical rule**: `compile:classified:` fails if the class does not yet exist in the
> image. Always define the class (`subclass:`) in its own snippet **before** any snippet
> that adds methods to it.

## Section 1: Introduction

One `##` text snippet. Cover:
- What problem/concept this page addresses
- What the reader will build
- Any prerequisites (e.g., "requires the MCP server running")

## Section 2: Interactive Exploration

Alternating text + Pharo snippets that explore the Smalltalk API:

| Snippet | Purpose |
|---------|---------|
| `ClassName new` | Inspect a freshly created object and its instance variables |
| `ClassName new setter: value` | Show how to configure it |
| Create a collaborator object | Explore helper classes |
| Wire collaborators together | Show how pieces connect |
| Full working example (no class yet) | Demonstrate the end-to-end flow using raw API |
| `object stop` | Clean up if applicable |

**Rules for exploration snippets:**
- Each snippet must be independently evaluable given the previous snippets' state
- Use workspace variables (`server`, `client`, `tool`) freely — they persist between snippet evaluations
- The full working example should be the last exploration snippet (starts AND tests AND stops)

## Section 3: Example-driven Development Sections

For each EDD iteration, invoke `gt-example-driven-development` for the code cycle, then
document the iteration with one section following the structure below. Repeat until all
examples are implemented.

## Example-driven Development Section Structure

One section per iteration

Each section is titled:
```
EDD - <ClassName> - Iteración <N>: <exampleMethodName>
```

Snippets must be evaluable top-to-bottom, in this order:

1. **[text]** Introduction — what this example tests and why it's the right next step
2. **[pharo]** Example class definition - only in the needed example class doesn't  exist yet (`subclass:` definition)
3. **[pharo]** Example definition via `compile:classified:` (the initially failing example)
4. **[text]** "Exploración interactiva" — explain what you explored and what you found
5. **[pharo]** Exploration snippets — the eval calls and intermediate results from step 2
6. **[text]** "Implementación mínima" — explain what minimum code is needed and why
7. **[pharo]** Class definition — only if the production class doesn't exist yet (`subclass:` definition)
8. **[pharo]** Method definitions via `compile:classified:` — always in a separate snippet from the class definition
9. **[example]** `ClassName>>exampleMethodName` — live verification that it passes

Steps 2 and 7 are included only when a new class is created in this iteration. Never combine the class
definition and method compilation into the same snippet — `compile:classified:` fails if the
class doesn't exist yet.

**Design guidance:**
- One class per page is the norm; split into sub-pages if complexity grows
- The class should wrap/encapsulate what was done manually in Section 2
- Name: `DynOS` prefix for DynaSpaceOS package classes (e.g., `DynOSMcpEvalServer`)
- Package: `DynaSpaceOS-<Feature>` (e.g., `DynaSpaceOS-MCP`)

## Naming Conventions in DynaSpaceOS

| Thing | Convention | Example |
|-------|-----------|---------|
| Production class | `DynOS` prefix | `DynOSMcpEvalServer` |
| Examples class | `DynOS` prefix + `Examples` | `DynOSMcpEvalServerExamples` |
| Package | `DynaSpaceOS-<Feature>` | `DynaSpaceOS-MCP` |
| Lepiter page title | Descriptive, Spanish or English | `Servidor MCP con herramienta eval` |

## Complete Snippet Sequence (typical, with 2 EDD iterations)

```
── Section 1: Introduction ──────────────────────────────────────
1.  text    — Introduction

── Section 2: Interactive Exploration ──────────────────────────
2.  text    — "Exploring ClassName"
3.  pharo   — ClassName new
4.  text    — "Configuring X"
5.  pharo   — ClassName new setter: value
6.  text    — "Wiring together"
7.  pharo   — Full working example (start + use + stop)

── Section 3: EDD Iteration 1 ──────────────────────────────────
8.  text    — "EDD - MyClass - Iteración 1: exampleCreation" (intro)
9.  pharo   — Examples class definition (only if class doesn't exist yet)
10. pharo   — compile: example method (initially failing)
11. text    — "Exploración interactiva"
12. pharo   — exploration eval snippets
13. text    — "Implementación mínima"
14. pharo   — Production class definition (only if class doesn't exist yet)
15. pharo   — compile: production methods
16. example — MyClassExamples>>exampleCreation

── Section 3: EDD Iteration 2 ──────────────────────────────────
17. text    — "EDD - MyClass - Iteración 2: exampleBehavior" (intro)
18. pharo   — compile: example method (initially failing)
19. text    — "Exploración interactiva"
20. pharo   — exploration eval snippets
21. text    — "Implementación mínima"
22. pharo   — compile: production methods
23. example — MyClassExamples>>exampleBehavior
```

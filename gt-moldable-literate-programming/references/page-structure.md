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

> **Critical rule**: Any message sent to a class (or its metaclass) fails if the class does not yet exist in the image. This includes not only `compile:classified:` but also `class instanceVariableNames:` and any other message sent to the class or its metaclass. Always define the class (`subclass:`) in its own snippet **before** any snippet that references it as a receiver — including metaclass configuration like `DynOSProCam class instanceVariableNames: '...'`.

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

Starts with a `##` text snippet titled **"Example-driven Development"**. Then, for each
EDD iteration, invoke `gt-example-driven-development` for the code cycle and document the
iteration with one section following the structure below. Repeat until all examples are
implemented.

## Example-driven Development Section Structure

One section per iteration.

Each section is titled with a **short descriptive phrase** explaining what the example verifies,
not a technical identifier. The example method name and iteration number belong in the
introductory text snippet, not in the title.

Good titles: "Creación del tablero vacío", "Marcado de una casilla", "Detección de victoria en fila"
Avoid: "EDD - DynOSTicTacToeBoard - Iteración 1: exampleBoardCreation"

Snippets must be evaluable top-to-bottom, in this order:

1. **[text]** Introduction — descriptive title (`###`) + iteration reference (`exampleMethodName`, N) + what this example tests and why it's the right next step
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

── Section 3: Example-driven Development ───────────────────────
8.  text    — "## Example-driven Development" (section heading)

── Section 3: EDD Iteration 1 ──────────────────────────────────
9.  text    — "### Creación del objeto" (descriptive title + iteration ref in body)
10. pharo   — Examples class definition (only if class doesn't exist yet)
11. pharo   — compile: example method (initially failing)
12. text    — "Exploración interactiva"
13. pharo   — exploration eval snippets
14. text    — "Implementación mínima"
15. pharo   — Production class definition (only if class doesn't exist yet)
16. pharo   — compile: production methods
17. example — MyClassExamples>>exampleCreation

── Section 3: EDD Iteration 2 ──────────────────────────────────
18. text    — "### Comportamiento principal" (descriptive title + iteration ref in body)
19. pharo   — compile: example method (initially failing)
20. text    — "Exploración interactiva"
21. pharo   — exploration eval snippets
22. text    — "Implementación mínima"
23. pharo   — compile: production methods
24. example — MyClassExamples>>exampleBehavior
```

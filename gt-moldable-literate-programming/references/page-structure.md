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
- **Verify before adding**: every Pharo snippet must be evaluated via `mcp__gtoolkit__eval`
  before being added to the page. Never write exploration snippets speculatively — if you
  don't know whether an API exists or what it returns, eval it first, then write the snippet
  with the known result.

## Section 3: Example-driven Development Sections

Starts with a `##` text snippet titled **"Example-driven Development"**. Then, for each
EDD iteration, invoke `gt-example-driven-development` for the code cycle and document the
iteration with one section following the structure below. Repeat until all examples are
implemented.

## Example-driven Development Section Structure

One section per iteration. MLP **does not execute the code cycle** — that is entirely owned
by `gt-example-driven-development`. MLP writes the intro text first, then invokes EDD, then
documents what EDD produced.

Each section is titled with a **short descriptive phrase** explaining what the example verifies,
not a technical identifier. The example method name and iteration number belong in the
introductory text snippet, not in the title.

Good titles: "Creación del tablero vacío", "Marcado de una casilla", "Detección de victoria en fila"
Avoid: "EDD - DynOSTicTacToeBoard - Iteración 1: exampleBoardCreation"

**Sequence for each iteration:**

1. **[text]** Write the introduction snippet — descriptive title (`###`) + what this example will verify and why it's the right next step. (Write this BEFORE invoking EDD.)
2. **STOP — invoke `gt-example-driven-development` via the Skill tool.** Pass the example name and what it should verify. Wait for EDD to complete and return its handoff summary.
3. **[pharo]** Examples class definition — add from EDD handoff (only if examples class was created in this iteration; `subclass:` definition).
4. **[pharo]** Example method — add from EDD handoff (`compile:classified:` call with the `<gtExample>` method).
5. **[text]** "Exploración interactiva" — write based on the EDD handoff summary (what was explored and found).
6. **[pharo]** Exploration snippets — add from EDD handoff (the eval calls and intermediate results EDD used).
7. **[text]** "Implementación mínima" — write based on the EDD handoff summary (what minimum code was needed and why).
8. **[pharo]** Production class definition — add from EDD handoff (only if production class was created in this iteration; `subclass:` definition).
9. **[pharo]** Method definitions — add from EDD handoff (`compile:classified:` calls; always in a separate snippet from the class definition).
10. **[example]** `ClassName>>exampleMethodName` — live verification that it passes.

Steps 3 and 8 are included only when a new class was created in this iteration (per EDD handoff).
Never combine the class definition and method compilation into the same snippet —
`compile:classified:` fails if the class doesn't exist yet.

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

── Section 3: EDD Iteration 1 ──────────────────────────────────────
9.  text    — "### Creación del objeto" (write BEFORE invoking EDD)
    *** STOP — invoke gt-example-driven-development via Skill tool ***
10. pharo   — Examples class definition (from EDD handoff; only if class didn't exist yet)
11. pharo   — compile: example method (from EDD handoff)
12. text    — "Exploración interactiva" (written from EDD handoff summary)
13. pharo   — exploration eval snippets (from EDD handoff)
14. text    — "Implementación mínima" (written from EDD handoff summary)
15. pharo   — Production class definition (from EDD handoff; only if class didn't exist yet)
16. pharo   — compile: production methods (from EDD handoff)
17. example — MyClassExamples>>exampleCreation

── Section 3: EDD Iteration 2 ──────────────────────────────────────
18. text    — "### Comportamiento principal" (write BEFORE invoking EDD)
    *** STOP — invoke gt-example-driven-development via Skill tool ***
19. pharo   — compile: example method (from EDD handoff)
20. text    — "Exploración interactiva" (written from EDD handoff summary)
21. pharo   — exploration eval snippets (from EDD handoff)
22. text    — "Implementación mínima" (written from EDD handoff summary)
23. pharo   — compile: production methods (from EDD handoff)
24. example — MyClassExamples>>exampleBehavior
```

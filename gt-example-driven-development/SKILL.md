---
name: gt-example-driven-development
description: >
  Guide for implementing functionality in GToolkit/Pharo following Example Driven Development (EDD).
  Works iteratively: for each example, write it first (failing), explore interactively in GT,
  implement the minimum code to make it pass, verify it passes. Handles only the code cycle —
  Lepiter page documentation is managed by gt-moldable-literate-programming.
  Use when the user says "implementa X con EDD",
  "desarrolla X usando Example Driven Development", "crea X siguiendo Example Driven Development",
  or any request to build something in GToolkit following an example-first, iterative approach.
  Activate this skill whenever the user mentions EDD, example-first development, or wants to
  build a feature incrementally with gtExample methods in GToolkit.
---

# GToolkit Example Driven Development

## Overview

EDD in GToolkit is a tight iterative cycle: write an example that fails → explore interactively
in GT → implement the minimum code to make it pass → verify it passes.
Repeat for each new example, always building on the output of previous ones.

Each iteration produces one artifact: working code compiled into the image. Documentation
(Lepiter pages) is handled by the `gt-moldable-literate-programming` skill.

## Before Starting

Verify MCP connectivity:

```
mcp__gtoolkit-dynaspace__eval  code: "1+1"
```

If the tool is unavailable, ask the user to start the MCP server by evaluating the startup
snippet on the "Servidor MCP para GToolkit" Lepiter page.

## The EDD Iteration Cycle

Each feature is built example by example. For each iteration:

### Step 1 — Define the Example (failing)

Start by thinking: what is the simplest behavior this example should test? What should it return?
(The return value feeds into the next example, so choose it intentionally.)

If the examples class doesn't exist yet, create it first — this is the class that will hold
the `<gtExample>` methods, not the production class (which comes later in step 3):

```smalltalk
Object subclass: #MyClassExamples
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'MyPackage'
```

Then define the example so it fails — either because the production class or method it calls
doesn't exist yet, or because its assertions don't hold:

```smalltalk
MyClassExamples compile: 'exampleBasicBehavior
    <gtExample>
    <description: ''What this example verifies and why it matters''>
    <return: #MyClass>
    | obj |
    obj := MyClass new.
    self assert: obj someMethod equals: expectedValue.
    ^ obj
' classified: 'examples'.
```

Every example must include `<description:>` and `<return:>`. Add `<after:>` or other pragmas
when needed (see **Pragmas** section below).

The example should fail at this point. That's intentional and expected.

### Step 2 — Explore Interactively

Before writing any implementation, explore the problem space in GT via MCP eval:
- Inspect the class structure
- Try intermediate expressions
- Understand what values are produced and why

This exploration informs the minimum implementation needed. Don't skip it — it's where
you understand the problem, not just solve it.

### Step 3 — Implement the Minimum

Write only enough code to make the current example pass. No more. Use `compile:classified:`
via MCP eval.

If the production class doesn't exist yet, create it first in a separate eval call before
adding any methods — `compile:classified:` fails if the class doesn't exist:

```smalltalk
Object subclass: #MyClass
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'MyPackage'
```

Then add the methods:

```smalltalk
MyClass compile: 'someMethod
    ^ expectedValue
' classified: 'accessing'.
```

Even a trivial or hardcoded implementation is fine at this stage. The goal is green, not perfect.

### Step 4 — Verify the Example Passes

Run the example and confirm it passes (no assertion errors, returns the expected value):

```smalltalk
MyClassExamples new exampleBasicBehavior
```

Also run all previous examples to confirm nothing regressed.

### Step 5 — Repeat with the Next Example

The next example typically reuses the output of a previous one:

```smalltalk
MyClassExamples compile: 'exampleNextBehavior
    <gtExample>
    <description: ''What this example adds on top of the previous one''>
    <return: #MyClass>
    | obj |
    obj := self exampleBasicBehavior.
    self assert: obj nextMethod equals: nextExpectedValue.
    ^ obj
' classified: 'examples'.
```

Follow the same cycle: fail → explore → implement → verify.

> **Note**: Documentation (Lepiter page creation) for each iteration is handled by the
> `gt-moldable-literate-programming` skill. This skill focuses exclusively on the code cycle.

## Pragmas

Every `<gtExample>` method must include these two pragmas:

| Pragma | Obligatorio | Uso |
|--------|-------------|-----|
| `<description: '...'>` | Sí | Describe qué verifica el ejemplo y por qué importa |
| `<return: #ClassName>` | Sí | Clase del objeto devuelto; se puede repetir para indicar varios tipos posibles (incluido `#Error`) |

Añade estos cuando la situación lo requiera:

| Pragma | Cuándo usarlo |
|--------|---------------|
| `<after: #methodName>` | El ejemplo crea recursos que necesitan limpieza al terminar (procesos, conexiones, ficheros, servidores). El método referenciado se invoca sobre la instancia de la clase de ejemplos tras ejecutar el ejemplo. |
| `<noTest>` | El ejemplo no debe ejecutarse como test automático (lento, interactivo, dependiente de hardware). |

**Ejemplo completo con todos los pragmas relevantes:**

```smalltalk
MyClassExamples compile: 'exampleWithServer
    <gtExample>
    <description: ''Verifies that the server starts and accepts connections''>
    <return: #MyServer>
    <return: #Error>
    <after: #stopServer>
    | server |
    server := MyServer new start.
    self assert: server isRunning.
    ^ server
' classified: 'examples'.
```

**Nota sobre las comillas:** dentro de `compile:classified:`, los literales de string en los pragmas usan comillas simples dobles: `<description: ''texto''>`.

## Naming Conventions (DynaSpaceOS)

- Production class: `DynOS` prefix — e.g., `DynOSCounter`
- Examples class: same + `Examples` — e.g., `DynOSCounterExamples`
- Package: `DynaSpaceOS-<Feature>` — e.g., `DynaSpaceOS-Counter`
- Target Lepiter database: `DynaSpaceOS`
- Page title: `EDD - DynOSCounter - Iteración 1: exampleBasicCreation`

# Moldable Literate Programming — Page Structure

A "moldable literate programming" Lepiter page lets readers understand AND build a feature
by evaluating snippets top-to-bottom. The page both explains concepts and generates all code.

## The Four-Section Pattern

```
1. INTRODUCTION        (text only)
   ↓
2. INTERACTIVE EXPLORATION  (text + Pharo snippets — understand the API)
   ↓
3. PRODUCTION CLASS    (text)
                       + Pharo: class definition only (subclass:...)
                       + Pharo: methods (compile:classified:)  ← separate snippet!
   ↓
4. EXAMPLES / TESTS    (text)
                       + Pharo: examples class definition only
                       + Pharo: example methods (compile:classified:)  ← separate snippet!
                       + one example snippet per gtExample method
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

## Section 3: Production Class

One transition text snippet, then **two Pharo snippets** (class definition first, methods second):

### 3a. Class definition

```smalltalk
"Definir la clase MyClass"
Object subclass: #MyClass
    instanceVariableNames: 'collaborator config'
    classVariableNames: ''
    package: 'DynaSpaceOS-MyFeature'.
```

### 3b. Methods

```smalltalk
"Añadir métodos a MyClass"
MyClass compile: 'initialize
    config := SomeDefault new' classified: 'initialization'.

MyClass compile: 'config: aConfig
    config := aConfig' classified: 'accessing'.

MyClass compile: 'start
    collaborator := SomeServer new config: config.
    collaborator start.
    ^ collaborator' classified: 'actions'.

MyClass compile: 'stop
    collaborator ifNotNil: [
        collaborator stop.
        collaborator := nil ]' classified: 'actions'.
```

**Design guidance:**
- One class per page is the norm; split into sub-pages if complexity grows
- The class should wrap/encapsulate what was done manually in Section 2
- Name: `DynOS` prefix for DynaSpaceOS package classes (e.g., `DynOSMcpEvalServer`)
- Package: `DynaSpaceOS-<Feature>` (e.g., `DynaSpaceOS-MCP`)

## Section 4: Examples Class

One transition text snippet, then **three kinds of snippets**:

1. One **Pharo snippet** with the class definition only (`subclass:`)
2. One **Pharo snippet** with all the `compile:classified:` method calls
3. One **example snippet** per `<gtExample>` method — muestran el resultado interactivamente

### 4a. Pharo snippet — class definition only

```smalltalk
"Definir la clase MyClassExamples"
Object subclass: #MyClassExamples
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'DynaSpaceOS-MyFeature'.
```

### 4b. Pharo snippet — example methods

```smalltalk
"Añadir métodos de ejemplo a MyClassExamples"

"Example 1: basic instantiation + default state"
MyClassExamples compile: 'instance
    <gtExample>
    | obj |
    obj := MyClass new.
    self assert: obj config isNotNil.
    ^ obj' classified: 'examples'.

"Example 2: operational state"
MyClassExamples compile: 'startedInstance
    <gtExample>
    | obj |
    obj := self instance.
    [ obj start.
      self assert: obj collaborator isNotNil ]
        ensure: [ obj stop ].
    ^ obj' classified: 'examples'.

"Example 3: end-to-end behavior"
MyClassExamples compile: 'fullRoundTrip
    <gtExample>
    | obj result |
    obj := MyClass new.
    [ obj start.
      result := obj doSomething.
      self assert: result = expectedValue ]
        ensure: [ obj stop ].
    ^ result' classified: 'examples'.
```

### 4b. Example snippets — one per method

After the compile snippet, add one example snippet per method so los usuarios pueden
verificar interactivamente que los tests pasan con éxito:

```json
{"type": "example", "content": "MyClassExamples>>instance"}
{"type": "example", "content": "MyClassExamples>>startedInstance"}
{"type": "example", "content": "MyClassExamples>>fullRoundTrip", "previewExpanded": true}
```

In Smalltalk (via `add_example` function or direct eval):
```smalltalk
s := LeExampleSnippet new
    exampleBehaviorName: 'MyClassExamples';
    exampleSelector: 'fullRoundTrip';
    previewShowSelector: 'gtViewsFor:';
    previewHeight: 200;
    codeExpanded: true;
    previewExpanded: false.
page addSnippet: s.
```

**gtExample rules:**
- Always use `<gtExample>` pragma
- Use `ensure: [obj stop]` for resources that must be cleaned up
- Chain examples: `self instance` calls the previous example
- Each example returns the primary object for inspection
- 3 examples is a good target: (1) instantiation, (2) operational state, (3) behavior

## Naming Conventions in DynaSpaceOS

| Thing | Convention | Example |
|-------|-----------|---------|
| Production class | `DynOS` prefix | `DynOSMcpEvalServer` |
| Examples class | `DynOS` prefix + `Examples` | `DynOSMcpEvalServerExamples` |
| Package | `DynaSpaceOS-<Feature>` | `DynaSpaceOS-MCP` |
| Lepiter page title | Descriptive, Spanish or English | `Servidor MCP con herramienta eval` |

## Complete Snippet Sequence (typical, with 3 examples)

```
1.  text    — Introduction
2.  text    — "Exploring ClassName"
3.  pharo   — ClassName new
4.  text    — "Configuring X"
5.  pharo   — ClassName new setter: value
6.  text    — "Creating a collaborator"
7.  pharo   — CollaboratorClass new ...
8.  text    — "Wiring together + start"
9.  pharo   — Full working example (start + use + stop)
10. text    — "Detener" / cleanup note
11. pharo   — thing stop
12. text    — "Clase reutilizable: DynOSMyClass"
13. pharo   — Class definition only: Object subclass: #DynOSMyClass ...
14. pharo   — Method definitions: DynOSMyClass compile: ... (all methods)
15. text    — "Ejemplos (tests)"
16. pharo   — Class definition only: Object subclass: #DynOSMyClassExamples ...
17. pharo   — Method definitions: DynOSMyClassExamples compile: ... (all examples)
18. example — DynOSMyClassExamples>>instance
19. example — DynOSMyClassExamples>>startedInstance
20. example — DynOSMyClassExamples>>fullRoundTrip
```

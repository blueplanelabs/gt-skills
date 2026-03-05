# gt-skills

Claude Code skills for developing in **GlamorousToolkit (GT)** / Pharo Smalltalk.

## Available skills

### `gt-example-driven-development`

Guides the **Example-driven Development (EDD)** code cycle: for each piece of functionality, write the example first (failing), explore interactively in GT, implement the minimum needed, and verify it passes. Repeat for each iteration.

- Works exclusively with the code cycle (`compile:` into the image)
- Implementation is incremental: methods start as stubs and evolve when a later example requires real logic
- Lepiter documentation is handled by `gt-moldable-literate-programming`

**Triggered by phrases like:**
- "implement X with EDD"
- "develop X using Example Driven Development"
- "build X following EDD"

### `gt-moldable-literate-programming`

Guides the creation of **Lepiter pages documented with live code**, following the Moldable Development style. Produces a single page evaluable top-to-bottom that combines interactive exploration and EDD.

- Structure: Introduction → Interactive exploration → Example-driven Development (one section per iteration)
- 7-step workflow with double undo: the Lepiter page becomes the sole source of truth
- Uses `gt-example-driven-development` internally for the code cycle

**Triggered by phrases like:**
- "implement X with moldable literate programming"
- "develop X using live literate programming"
- "create a Lepiter page to implement X"

## Relationship between skills

```
gt-moldable-literate-programming
        │
        └── uses ──► gt-example-driven-development (code cycle)
```

You can invoke `gt-example-driven-development` independently when you only need the code cycle without Lepiter documentation.

## Requirements

- **GlamorousToolkit** with the project loaded in the image
- **MCP server** running (evaluate the startup snippet on the "Servidor MCP para GToolkit" Lepiter page)
- **Claude Code** with the skills installed under `.claude/skills/`

## Installation

Clone this repository (or add it as a submodule) into `.claude/skills/gt-skills/` inside your project. Claude Code will discover the skills automatically by reading the `SKILL.md` file in each subdirectory.

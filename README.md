# gt-skills

Claude Code skills para desarrollar en **GlamorousToolkit (GT)** / Pharo Smalltalk.

## Skills disponibles

### `gt-example-driven-development`

Guía el ciclo de código **Example-driven Development (EDD)**: por cada funcionalidad, escribe primero el ejemplo (fallido), explora interactivamente en GT, implementa el mínimo necesario, y verifica que pasa. Se repite por cada iteración.

- Trabaja exclusivamente con el ciclo de código (`compile:` en imagen)
- La implementación es incremental: los métodos comienzan como stubs y evolucionan cuando un ejemplo posterior lo requiere
- La documentación Lepiter la gestiona `gt-moldable-literate-programming`

**Se activa con frases como:**
- "implementa X con EDD"
- "desarrolla X usando Example Driven Development"
- "crea X siguiendo EDD"

### `gt-moldable-literate-programming`

Guía la creación de **páginas Lepiter documentadas con código vivo**, siguiendo el estilo de Moldable Development. Genera una única página evaluable de arriba a abajo que combina exploración interactiva y EDD.

- Estructura: Introducción → Exploración interactiva → Example-driven Development (una sección por iteración)
- Workflow de 7 pasos con doble deshacer: la página Lepiter queda como única fuente de verdad
- Usa internamente `gt-example-driven-development` para el ciclo de código

**Se activa con frases como:**
- "implementa X con moldable literate programming"
- "desarrolla X usando live literate programming"
- "crea una página Lepiter para implementar X"

## Relación entre los skills

```
gt-moldable-literate-programming
        │
        └── usa ──► gt-example-driven-development (ciclo de código)
```

Puedes invocar `gt-example-driven-development` de forma independiente cuando solo necesitas el ciclo de código sin documentación Lepiter.

## Requisitos

- **GlamorousToolkit** con el proyecto cargado en imagen
- **Servidor MCP** activo (evaluar el snippet de arranque en la página "Servidor MCP para GToolkit")
- **Claude Code** con los skills instalados en `.claude/skills/`

## Instalación

Clona este repositorio (o añádelo como submódulo) en `.claude/skills/gt-skills/` dentro de tu proyecto. Claude Code descubrirá los skills automáticamente al leer los `SKILL.md` de cada subdirectorio.

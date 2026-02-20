# Lepiter MCP API Reference

Key operations for working with a GToolkit Lepiter database via the MCP server tools.

## Available MCP Tools (primary approach)

| Tool | Parameters | Purpose |
|------|-----------|---------|
| `mcp__gtoolkit-dynaspace__create-page` | `dbName`, `pageTitle` | Create a new Lepiter page |
| `mcp__gtoolkit-dynaspace__add-snippet` | `dbName`, `pageTitle`, `snippetType` (`text`\|`pharo`\|`example`), `content` | Add a snippet to a page |
| `mcp__gtoolkit-dynaspace__get-page` | `dbName`, `pageTitle` | Read page contents as plain text |
| `mcp__gtoolkit-dynaspace__list-pages` | `dbName` | List all page titles in a database |
| `mcp__gtoolkit-dynaspace__eval` | `code` | Evaluate arbitrary Smalltalk |

  Test connectivity:

```
mcp__gtoolkit-dynaspace__eval  code: "1+1"
# Expected result: "2"
```

## Database Navigation

```smalltalk
"List all loaded databases"
LeDatabasesRegistry default currentLoadedDefaultLogicalDatabase
    databases collect: [:d | d databaseName]

"Get the DynaSpaceOS database"
| db |
db := LeDatabasesRegistry default currentLoadedDefaultLogicalDatabase
    databases detect: [:d | d databaseName = 'DynaSpaceOS'].

"List pages in a database"
db pages collect: [:p | p title]
```

## Page Operations

```smalltalk
"Create a new page and add to database"
| db page |
db := ... "see above".
page := LePage named: 'My Page Title'.
db addPage: page.

"Get an existing page"
page := db pageNamed: 'My Page Title'.

"Count snippets"
page children size
```

## Snippet Operations

```smalltalk
"Add a text snippet"
| s |
s := LeTextSnippet new string: '##My heading\n\nBody text.'.
page addSnippet: s.

"Add a Pharo code snippet"
s := LePharoSnippet new code: 'GtMcpServer new'.
page addSnippet: s.

"Add an example snippet (shows a gtExample method result interactively)"
s := LeExampleSnippet new
    exampleBehaviorName: 'MyClassExamples';
    exampleSelector: 'myExampleMethod';
    previewShowSelector: 'gtViewsFor:';
    previewHeight: 200;
    codeExpanded: true;
    previewExpanded: false.
page addSnippet: s.

"Remove a snippet by index"
page removeSnippet: (page children at: 3).

"Reorder: move snippet up one position"
page moveUpSnippet: (page children at: 4).
```

## Inspect a Page's Structure

```smalltalk
String streamContents: [:s |
    page children withIndexDo: [:snippet :i |
        s nextPutAll: i asString; nextPutAll: ': '.
        s nextPutAll: snippet class name; nextPutAll: ': '.
        (snippet isKindOf: LeTextSnippet) ifTrue: [
            s nextPutAll: (snippet string copyFrom: 1 to: (60 min: snippet string size))].
        (snippet isKindOf: LePharoSnippet) ifTrue: [
            s nextPutAll: (snippet code copyFrom: 1 to: (60 min: snippet code size))].
        s cr]]
```

## Compiling Classes from a Snippet

Use `compile:classified:` to add methods to a class:

```smalltalk
"Define the class"
Object subclass: #MyClass
    instanceVariableNames: 'foo bar'
    classVariableNames: ''
    package: 'DynaSpaceOS-MyPackage'.

"Add methods (single quotes in source are doubled)"
MyClass compile: 'initialize
    foo := 0' classified: 'initialization'.

MyClass compile: 'foo
    ^ foo' classified: 'accessing'.

MyClass compile: 'doSomethingWith: aValue
    "Method with a string literal"
    ^ ''result: '', aValue printString' classified: 'actions'.
```

## gtExample Pattern

```smalltalk
MyExamples compile: 'basicInstance
    <gtExample>
    | obj |
    obj := MyClass new.
    self assert: obj foo equals: 0.
    ^ obj' classified: 'examples'.

MyExamples compile: 'startedInstance
    <gtExample>
    | obj |
    obj := self basicInstance.
    [ obj start.
      self assert: obj isRunning ]
        ensure: [ obj stop ].
    ^ obj' classified: 'examples'.
```

## Important Notes

- **Method sources in compile:**: String literals inside method source need doubled single quotes in Smalltalk, so `'hello'` inside a compile: string becomes `''hello''`.
- **Bad Request errors**: Caused by the Smalltalk expression returning a complex object. Always return a simple string (`'ok'` or `result asString`).
- **MCP port**: Default 8765. Verify with `curl http://localhost:8765/` returning a response.

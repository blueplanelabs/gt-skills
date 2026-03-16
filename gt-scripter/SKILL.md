---
name: gt-scripter
description: >
  Guide for writing <gtExample> methods that test BlElement graphical components in
  GToolkit/Pharo using BlScripter. Use when examples need to assert on visual properties
  (size, position, children, bounds), verify element state after layout, or simulate user
  interactions (clicks, keyboard, mouse, drag). BlElement >> size returns 0@0 until layout
  runs; BlScripter triggers the full layout cycle headlessly via BlMockedHost.
  Activate when the user says "test a graphical element", "write an example with BlScripter",
  "assert on element size", "simulate user interaction", or similar.
---

# GToolkit BlScripter Guide

## When to Use BlScripter

Any `<gtExample>` that touches visual properties of a `BlElement` **must** use `BlScripter`.
Direct assertions fail silently:

```smalltalk
"WRONG — always returns 0@0 before display"
element := MyElement new size: 120@120.
self assert: element size equals: 120@120.  "fails: size = 0@0"

"CORRECT — BlScripter triggers layout via BlMockedHost"
scripter := BlScripter new element: (MyElement new size: 120@120).
scripter checkStep: [ :s | s value: [ :el | el size ] equals: [ 120@120 ] ].
```

Use BlScripter whenever the example verifies: size, position, bounds, children, rendering,
focus state, or any user interaction (click, keyboard, drag).

---

## Basic Pattern

```smalltalk
myExample
    <gtExample>
    <description: 'MyElement renders with correct size'>
    <return: #BlScripter>
    | element scripter |
    element := MyElement new.
    scripter := BlScripter new element: element.
    scripter checkStep: [ :s |
        s satisfies: [ :el | el isKindOf: BlFormElement ].
        s value: [ :el | el size ] equals: [ 120@120 ] ].
    ^ scripter
```

**Key rules:**
- `<return: #BlScripter>` — not the element type
- `BlScripter new element: anElement` — triggers layout immediately via `privatePulseUntilReady`
- The default space is `800@600` with `BlMockedHost` (headless, no real display needed)

---

## Assertions

### checkStep: vs assertStep:

Both are **semantically equivalent** — same implementation (`BlDevScripterFutureCheckStep`).
Convention: use `checkStep:` for intermediate state verification, `assertStep:` for final assertions.

### Assertion methods

```smalltalk
"Predicate-based"
scripter checkStep: [ :s |
    s satisfies: [ :el | el isVisible ] ].

"Value equality"
scripter checkStep: [ :s |
    s value: [ :el | el size ] equals: [ 200@100 ] ].

"Multiple assertions in one step"
scripter checkStep: [ :s |
    s label: 'Check size and position'.
    s value: [ :el | el size ] equals: [ 200@100 ].
    s value: [ :el | el position ] equals: [ 50@30 ] ].

"Existence check"
scripter assertStep: [ :s |
    s label: 'Dropdown is open'.
    s exists.
    s onSpaceRoot; // BrMenuSteppedElement ].

"Non-existence check"
scripter assertStep: [ :s |
    s label: 'Dropdown is closed'.
    s notExists.
    s id: #myDropdown ].

"Event fired"
scripter checkStep: [ :s |
    s eventFired: BlClickEvent ].
```

---

## Element Navigation & Targeting

Steps default to the scripter's element. Override with:

```smalltalk
s onSpaceRoot          "space root element (ancestor of all)"
s onSelf               "current element (explicit)"
s onSpace              "the BlSpace itself"
s onScripter           "the scripter object (for userData)"
s onModel              "the model set via scripter model:"
s onChildAt: 1         "1-indexed direct child"
s id: #myId            "depth-first search by BlElement id"
s / BlButtonElement    "first direct child of type"
s // BrMenuElement     "first descendant of type (recursive)"
```

Chains compose:
```smalltalk
s onSpaceRoot; id: #container; onChildAt: 2.
s // BrTabGroup; / #header; // #tabbar; onChildAt: 3.
```

`//` is the most common for finding nested elements without knowing exact depth.

---

## Grouping with substeps

Group related actions into named phases — improves inspector readability and error reporting:

```smalltalk
scripter substeps: 'Add an item' do: [ :aStep |
    aStep clickStep: [ :s | s id: #addButton ].
    aStep checkStep: [ :s |
        s label: 'List has one item'.
        s value: [ :list | list children size ] equals: [ 1 ] ] ].

scripter substeps: 'Delete the item' do: [ :aStep |
    aStep clickStep: [ :s | s id: #deleteButton ].
    aStep checkStep: [ :s |
        s label: 'List is empty'.
        s value: [ :list | list children size ] equals: [ 0 ] ] ].
```

Use `substep:do:` (singular) for a single sub-action:
```smalltalk
aStep substep: 'Open dropdown' do: [ :s |
    s clickStep: [ :ss | ss id: #dropdownButton ] ].
```

With setup and teardown:
```smalltalk
scripter substeps: 'Phase' before: [ "setup" ] play: [ :aStep | "steps" ] ensure: [ "teardown" ].
```

---

## Mouse Interactions

```smalltalk
"Click"
scripter clickStep: [ :s |
    s label: 'Click delete button'.
    s id: #deleteButton ].

"Double click"
scripter doubleClickStep: [ :s | s onChildAt: 1 ].

"Right click"
scripter secondaryClickStep: [ :s | s id: #anchorElement ].

"Click with modifier"
scripter clickStep: [ :s |
    s onChildAt: 5.
    s modifiers: BlKeyModifiers shift ].

"Mouse move by delta"
scripter mouseMove by: 120@90; play.

"Mouse down / up (for drag)"
scripter mouseDown; play.
scripter mouseMove by: 120@90; play.
scripter mouseUp; play.

"Mouse over (hover)"
scripter mouseMoveOverStep: [ :s |
    s label: 'Hover over element'.
    s id: #hoverTarget ].
```

---

## Keyboard Interactions

```smalltalk
"Type text"
scripter typeStep: [ :s |
    s text: 'hello world' ].

"Key press (sent to focused element)"
scripter keyPressStep: [ :s |
    s key: BlKeyboardKey return ].

"Keyboard shortcut"
scripter shortcutStep: [ :s |
    s key: (BlKeyCombination builder primary; key: BlKeyboardKey f; build) ].

"Request focus then type"
scripter requestFocus id: #inputField; play.
scripter typeStep: [ :s | s text: 'search term' ].
```

---

## Actions on Elements

```smalltalk
"Modify element state mid-script"
scripter doStep: [ :s |
    s label: 'Hide child'.
    s block: [ :el | el visibility: BlVisibility hidden ].
    s onChildAt: 1 ].

"Execute block on space"
scripter doStep: [ :s |
    s label: 'Resize space'.
    s block: [ :space | space extent: 1920@1080 ].
    s onSpace ].
```

---

## Model and UserData

Attach a domain object to the scripter for cross-layer verification:

```smalltalk
scripter := BlScripter new
    element: myElement;
    model: myDomainObject.

"Access in examples:"
domainObject := scripter model.

"Target it in steps:"
scripter checkStep: [ :s |
    s value: [ :m | m someProperty ] equals: [ expected ].
    s onModel ].
```

Store transient state across steps:
```smalltalk
"Store during doStep:"
scripter doStep: [ :s |
    s block: [ :scripterCtx |
        scripterCtx userData at: #captured put: someValue ].
    s onScripter ].

"Retrieve in checkStep:"
scripter checkStep: [ :s |
    s do: [ stored := scripter userData at: #captured.
            self assert: stored equals: expectedValue ] ].
```

---

## Domain Helper Pattern

For complex UIs, extract fragile selectors into class-side helpers on the element class:

```smalltalk
"Extension method on the element class:"
MyElement class >> doStepClickDeleteButton: anItem onScripterStep: aStep [
    aStep clickStep: [ :s |
        s label: 'Click delete for ' , anItem id.
        s onSpaceRoot; id: 'item-' , anItem id; id: #deleteButton ]
]

"Usage in examples (readable and maintainable):"
MyElement doStepClickDeleteButton: item onScripterStep: aStep.
```

Benefits: centralizes element IDs, makes examples readable, easy to update when UI changes.

---

## Extracting Element After Testing

To display the tested element in a GT inspector view after the example runs:

```smalltalk
"Remove from test space (keeps element for display):"
^ scripter elementWithoutParent

"Or get the whole space root:"
^ scripter rootWithoutParent
```

When returning `#BlScripter`, the inspector shows the scripter with step tree and preview.
When returning `#BlElement`, use `elementWithoutParent`.

---

## Advanced: Timing Control

For tests involving double-clicks or rapid interactions, use simulated time:

```smalltalk
scripter space time: BlTime simulated.
scripter space time wait: Bloc doubleClickDelay / 4.
scripter doubleClickStep: [ :s | s onChildAt: 1 ].
```

---

## Integration with EDD

Examples returning `#BlScripter` compose naturally — the next example can call the previous:

```smalltalk
myExampleWithInteraction
    <gtExample>
    <description: 'After clicking add, list has one item'>
    <return: #BlScripter>
    | scripter |
    scripter := self myBaseExample.   "reuse previous example's scripter"
    scripter clickStep: [ :s | s id: #addButton ].
    scripter checkStep: [ :s |
        s value: [ :list | list children size ] equals: [ 1 ] ].
    ^ scripter
```

The scripter retains all state (space, element, model) between calls — steps added in the new
example execute on top of the already-laid-out element from the prior example.

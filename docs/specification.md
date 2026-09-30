# TrainWhen Specification

Status: draft scaffold

This document is the authoritative draft for TrainWhen prescription semantics. Examples and agent skills must follow it. Sections marked **Unresolved** are design questions, not implicit permission for an implementation to guess.

## 1. Scope

TrainWhen is a plain-text language for readable, deterministic training prescriptions. Markdown provides document structure; TrainWhen defines the training semantics.

A conforming implementation must be able to:

1. Parse a prescription without relying on an AI interpretation.
2. Resolve inheritance and overrides.
3. Expand structural containers into an ordered normalized representation.
4. Reject ambiguous or conflicting prescriptions with useful errors.
5. Preserve instructions that a destination format cannot represent structurally.

## 2. Line roles

Indentation establishes parent-child hierarchy. The presence of a hyphen determines whether a line creates an ordered item.

### 2.1 Ordered items

A hyphenated line creates an ordered item:

```text
- run 10 min @ easy
- recover 2 min @ easy
- run 5 min @ threshold
```

An item may be:

- an executable activity or exercise;
- a structural container such as `sequence`, `sets`, or `series`;
- a transition such as `rest` or `recover`;
- a reference such as a warmup or cooldown; or
- a partial prescription completed through inheritance.

Sibling items execute in listed order unless a structural container defines repetition.

### 2.2 Structured properties

An indented, non-hyphenated line using a recognized `key: value` form adds a structured property to its parent item. It does not create another ordered item.

```text
- half-squat jump 8 reps
  tempo: 1 rep/s
  load: 35-45% squat 1RM
```

The recognized property vocabulary is not yet complete. Unknown keys must not silently acquire semantics.

### 2.3 Display instructions

Other indented, non-hyphenated text is a display instruction attached to its parent item:

```text
- @ 10% incline
  skip while you do it
```

Display instructions do not alter execution semantics unless a later specification version assigns them structured meaning. Exporters should preserve them as notes when possible.

## 3. Partial prescriptions

An ordered item does not need to repeat components inherited from its ancestors. It may supply only the components that vary:

```text
- sequence: run 2 min @ RPE 5
  - @ 5% incline
  - @ 10% incline
  - @ 15% incline
```

Each child becomes a complete run step after inheriting the shared activity, duration, and RPE from the sequence.

Human-facing source should favor compact prescription syntax. Implementations may normalize it internally into fields such as activity, duration, effort, incline, load, and tempo; authors should not have to write that expanded representation.

## 4. Containers

Containers establish execution structure and an inheritance scope.

### 4.1 Sequence

`sequence` executes each child once in listed order. Text after the colon is a shared partial prescription inherited by its children.

```text
- sequence: @ 1% incline
  - run 3 min @ RPE 2
  - run 3 min @ RPE 3
  - run 3 min @ RPE 4
```

A sequence does not imply repetition.

### 4.2 Sets

`N sets` repeats its child block `N` times.

```text
- 6 sets
  - half-squat jump 8 reps
  - rest 60 s
```

### 4.3 Series

`N series` repeats its child block `N` times. `series` retains its training-domain meaning and is not a synonym for `sequence`.

```text
- 2 series
  - 6 sets
    - half-squat jump 8 reps
      tempo: 1 rep/s
      load: 35-45% squat 1RM
    - rest 60 s
  - recover 10 min @ easy
```

The same inheritance rules apply to `sequence`, `sets`, and `series`. Their execution behavior differs.

## 5. Inheritance and overrides

Structured prescription components inherit from parent containers to descendants.

Rules:

1. A child inherits components it does not declare.
2. A child-local value overrides an inherited value for that child and its descendants.
3. The nearest declared value wins.
4. Two conflicting values for the same single-valued component on one item are invalid.
5. Display instructions accumulate; they do not override one another.
6. After inheritance, every executable item must contain enough information to execute or must have an explicit open completion condition.

## 6. Rest and recovery

`rest` and `recover` describe different transition modes, not different structural levels:

- `rest` is passive.
- `recover` is active and may carry its own prescription.

Either may follow a step, set, series, sequence, or another suitable item.

### 6.1 Repetition boundary rule

A terminal `rest` or `recover` inside a repeated container is interstitial: it is inserted between repetitions and omitted after the final repetition.

```text
- 3 sets
  - half-squat jump 8 reps
  - rest 60 s
```

expands conceptually to:

```text
half-squat jump 8 reps
rest 60 s
half-squat jump 8 reps
rest 60 s
half-squat jump 8 reps
```

Outside that terminal position in a repeated container, rest or recovery executes literally where listed.

## 7. Worked composition example

Source:

```text
- sequence: run 2 min @ RPE 5
  - @ 5% incline
    sing while you do it
  - @ 10% incline
    skip while you do it
  - @ 15% incline
    sing and skip while you do it
```

Normalized result:

| Step | Activity | Duration | Effort | Incline | Instruction |
|---:|---|---|---|---|---|
| 1 | run | 2 min | RPE 5 | 5% | sing while you do it |
| 2 | run | 2 min | RPE 5 | 10% | skip while you do it |
| 3 | run | 2 min | RPE 5 | 15% | sing and skip while you do it |

## 8. Validation requirements

A validator should reject at least:

- conflicting single-valued properties at one scope;
- a partial item that remains incomplete after inheritance;
- an unknown structural container;
- a repeated container with an invalid or non-positive count;
- malformed or dimensionally invalid units;
- indentation that does not identify a unique parent; and
- a structured property key whose meaning is unknown.

A validator should preserve, rather than reject, ordinary display instructions.

## 9. Export model

Exporters operate on the normalized prescription, not directly on shorthand source.

The pipeline is:

```text
TrainWhen source
    → parse
    → resolve inheritance and overrides
    → expand containers and transitions
    → validate normalized steps
    → export
```

Destination formats may not preserve TrainWhen's compact inheritance structure. An exporter may flatten it as long as execution order and meaning are preserved.

If a destination cannot represent a component structurally, the exporter must either:

1. preserve it as a display instruction or note and report the downgrade; or
2. fail with a useful unsupported-feature error.

It must not silently discard the component.

## 10. Unresolved

The following require concrete examples and explicit decisions:

- the complete vocabulary of structured properties;
- multiple simultaneous targets written on one line;
- target ranges and comparison operators;
- duration and completion-condition grammar;
- whether and how arbitrary containers may carry colon-attached shared prescriptions;
- reference resolution and parameterized reusable workouts;
- progression-level inheritance;
- exact normalized intermediate representation;
- validation error format;
- FIT and other vendor-specific lowering rules; and
- versioning and compatibility declarations.

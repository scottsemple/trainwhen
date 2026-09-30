---
name: trainwhen
description: Use when working with TrainWhen prescriptions. Write, validate, import, and export them deterministically.
version: 0.1.0
metadata:
  hermes:
    tags: [training, workouts, markdown, fit]
    category: coaching
---

# TrainWhen

## When to Use

Use this skill when asked to:

- write or revise a TrainWhen workout prescription;
- interpret or validate TrainWhen syntax;
- convert a workout into or out of TrainWhen;
- inspect whether a proposed grammar is deterministic; or
- work on the TrainWhen parser, validator, or adapters.

## Contract

This skill guarantees that:

- `docs/specification.md` is treated as the language authority;
- examples are not allowed to silently override the specification;
- inheritance and structural containers are resolved before export;
- ambiguous or incomplete prescriptions are reported rather than guessed;
- unsupported destination features are preserved as notes or reported; and
- produced artifacts are validated or clearly labeled as unvalidated.

## Sources of Truth

Use sources in this order:

1. `docs/specification.md`
2. `README.md`
3. Examples under `example-library/`
4. Vendor documentation for destination formats

If a lower-ranked source conflicts with a higher-ranked source, report the conflict. Do not reconcile it silently.

## Procedure

### 1. Inspect the request and repository

Determine whether the task is authoring, interpretation, validation, import, export, or grammar design. Read the current specification and the smallest relevant set of examples before acting.

### 2. Preserve source meaning

Extract only information present in the source. Do not invent targets, RPE, load, duration, recovery, or completion conditions. Clearly identify any user-authorized assumptions.

For binary or vendor formats, decode all relevant messages and fields before drafting TrainWhen source. Do not rely only on display names when structured fields are present.

### 3. Classify each TrainWhen line

- Hyphenated lines create ordered items.
- Recognized indented `key: value` lines add structured properties to their parent.
- Other indented non-hyphenated text is a display instruction attached to its parent.

An ordered item may be an activity, transition, reference, container, or partial prescription.

### 4. Resolve structure

- `sequence` executes each child once in listed order.
- `N sets` repeats its child block N times.
- `N series` repeats its child block N times.
- `sets` and `series` are training-domain repetition structures; neither is a synonym for `sequence`.

### 5. Resolve inheritance

Parent components flow downward. A child-local value overrides the inherited value. Reject conflicting values declared at the same scope. Accumulate display instructions.

### 6. Resolve transitions

`rest` is passive. `recover` is active and may have its own prescription. Neither term belongs exclusively to sets or series.

A terminal rest or recovery inside a repeated container is interstitial: place it between repetitions and omit it after the final repetition. Elsewhere, execute it literally where listed.

### 7. Normalize and validate

Before conversion or execution, produce or reason through an ordered normalized representation. Confirm that every executable item is complete after inheritance and that every count, unit, target, and completion condition is valid under the current specification.

If the specification marks the required syntax unresolved, stop and surface the design question rather than creating an undocumented language rule.

### 8. Export

Export from the normalized representation. Preserve order and semantics even if the destination requires duplicated or flattened steps.

If a target format lacks a native field, preserve the component as a note when that is useful and supported, and report the downgrade. Otherwise fail explicitly.

For FIT-specific mapping, load `references/fit-mapping.md`.

## Verification

For generated text:

- parse or manually normalize every item;
- check inheritance and overrides;
- check expansion counts and transition boundaries; and
- ensure no source instruction disappeared.

For generated binary files:

- decode the artifact with an independent reader when possible;
- compare decoded steps with the normalized prescription; and
- verify file integrity or CRC using the available SDK or decoder.

## Output Format

When creating a prescription, return:

1. the `.md` artifact;
2. a concise statement of assumptions or semantic downgrades; and
3. validation status.

When reviewing syntax, distinguish:

- valid current syntax;
- invalid or ambiguous syntax; and
- unresolved language design.

## Anti-Patterns

- Treating every nested line as another workout step.
- Treating `sequence` and `series` as synonyms.
- Binding `rest` only to sets or `recover` only to series.
- Appending a terminal interstitial transition after the final repetition.
- Expanding compact source before resolving inheritance.
- Inventing missing intensity guidance during import.
- Silently dropping properties unsupported by an export format.
- Using an example file as authority when it conflicts with the specification.
- Encoding a new grammar rule merely because one destination vendor requires it.

## Tools Used

Use repository file tools to read and write TrainWhen Markdown. Use deterministic parsers or SDKs for binary formats. Consult current official vendor documentation for format-specific claims.

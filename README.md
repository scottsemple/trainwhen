# TrainWhen

**Training plans should belong to coaches and athletes, not training platforms.**

TrainWhen is an open, plain-text system for writing workouts, workout progressions, and training plans.

The canonical source is readable Markdown. It can be versioned with Git, shared without proprietary software, and eventually exported to training platforms, calendars, FIT files, CSV, and other formats.

```text
write once → understand it → reuse it → export anywhere
```

> **Status: v0.0.1 — experimental**
>
> TrainWhen is currently being designed and implemented. Expect breaking syntax changes before the format stabilizes.

## Design principles

**Immediately understandable.**  
An athlete or coach should substantially understand a TrainWhen file before reading the specification.

**Markdown first.**  
Use established plain-text conventions where they work. Add TrainWhen syntax only when the training domain requires it.

**Don't repeat yourself.**  
Define shared information once, inherit it, and override it only where necessary.

**Deterministic.**  
Software should not require AI inference to determine what a canonical prescription means.

**Composable.**  
Exercises build workouts. Workouts build progressions. Progressions build larger training structures.

**Explicit when necessary.**  
Defaults make common prescriptions concise without preventing a coach from saying exactly what is intended.

**Vendor-independent.**  
TrainingPeaks, Garmin, Intervals.icu, Strava, and other platforms are adapters and destinations, not canonical storage.

## Start simple.

A steady aerobic workout should look like a steady aerobic workout:

```text
- run 45 min @ 80%
```

TrainWhen avoids repeating information that is already established by context.

A coach can define the default benchmark used for each activity:

```text
# Coach

benchmark:
  run: AnT HR
  bike: FTP
```

An athlete supplies individual benchmark values:

```text
# Athlete

benchmark:
  run:
    AnT HR: 172 bpm
    AeT HR: 163 bpm
    FTP: 285 W
  bike:
    AnT HR: 159 bpm
    AeT HR: 141 bpm
    FTP: 205 W
```

So:

```text
- run 45 min @ 80%
```

can resolve against the coach's default running benchmark and the athlete's corresponding value.

When the prescription needs to differ from the default, say so:

```text
- run 45 min @ 80% AeT HR
- bike 20 min @ 95% FTP
- run 60 min <= AeT HR
```

**Define defaults once. Override them where necessary.**

## Training isn't always simple.

TrainWhen is being designed against real training structures rather than only simple steady-state and interval workouts. One of its initial test cases is Verkhoshansky's explosive-strength progression for runners.

The first session can be expressed as:

```text
# Explosive Strength

- warmup: basic-ramp

- 2 series
  - 6 sets
    - half-squat jump 8 reps @ 35-45% squat 1RM
      - tempo 1 rep/s
    - rest 60 s
  - recover 10 min

- cooldown: easy-aerobic
  - 10 min or until HR stabilizes
```

The nesting carries meaning.

`rest` is always *passive* rest between sets.

`recover` is always active recovery between series or sets. Recovery may have its own prescription and does not necessarily mean easy:

```text
- recover 400 m @ marathon pace
```

Warmups and cooldowns are reusable workouts. A RAMP warmup can also be prescribed as a complete workout—for example, when it represents an appropriate session for an athlete just starting out.

The structure also lets TrainWhen derive useful information rather than requiring it to be entered twice:

```text
2 series × 6 sets × 8 reps = 96 jumps
```

## Progressions are first-class.

Training is not merely a collection of independent workouts.

A **progression** describes how training changes across exposures.

It may change:

- exercise
- volume
- load
- intensity
- tempo
- rest
- recovery
- or other prescription variables

Verkhoshansky's 16-session explosive-strength sequence is an initial TrainWhen design test:

```text
A01  SJ  2 series ×  6 sets ×  8 reps   rest 60 s   recover 10 min
A02  LJ  2 series ×  8 sets ×  8 reps   rest 60 s   recover 10 min
A03  SJ  2 series ×  8 sets × 10 reps   rest 60 s   recover 10 min
A04  LJ  3 series ×  8 sets × 10 reps   rest 60 s   recover 10 min
A05  SJ  2 series × 10 sets × 10 reps   rest 60 s   recover 10 min
A06  LJ  2 series ×  6 sets ×  8 reps   rest 30 s   recover 10 min
A07  SJ  3 series ×  6 sets ×  8 reps   rest 30 s   recover 10 min
A08  LJ  3 series × 10 sets × 10 reps   rest 60 s   recover 12 min
A09  SJ  2 series ×  8 sets × 10 reps   rest 30 s   recover 10 min
A10  LJ  2 series ×  6 sets ×  8 reps   rest 30 s   recover 10 min
A11  SJ  3 series ×  8 sets ×  8 reps   rest 10 s   recover 12 min
A12  LJ  3 series × 10 sets × 10 reps   rest 30 s   recover 12 min
A13  SJ  3 series ×  8 sets × 10 reps   rest 10 s   recover 12 min
A14  LJ  3 series × 10 sets × 10 reps   rest 10 s   recover 12 min
A15  SJ  4 series × 10 sets × 10 reps   rest 10 s   recover 14 min
A16  LJ  4 series × 10 sets × 10 reps   rest 10 s   recover 14 min
```

`SJ` is a half-squat jump—not a full squat jump.

`LJ` is an alternating lunge jump.

The compact representation is useful for titles and summaries. The canonical executable prescription remains explicit and nested so that a human does not have to memorize positional shorthand.

### Don't repeat shared prescription.

If all 16 sessions use the same warmup, cooldown, load, or other prescription, those values should not be copied into all 16 workouts. They belong at the progression level and are inherited by its workouts.

An individual workout specifies something again only when it differs.

The general rule is:

> **Define a value once at the highest useful scope. Inherit it downward. Override it explicitly where necessary.**

### Gateways

A progression may eventually have a **gateway**: a readiness workout or assessment that determines whether an athlete is ready for the progression and where that athlete should enter it.

A more advanced athlete should not necessarily have to begin at A01. A beginner may not yet qualify for the progression or may require a scaled preparatory progression.

The exact gateway and advancement grammar is not part of v0.0.1.

## Compose training.

TrainWhen's training hierarchy is:

```text
exercise
   ↓
workout
   ↓
progression
   ↓
microcycle
   ↓
mesocycle
   ↓
macrocycle
   ↓
training plan
```

The hierarchy represents increasing scope, not mandatory nesting.

A simple plan may prescribe workouts directly:

```text
workout → plan
```

A more sophisticated plan might use:

```text
workout → progression → microcycle → mesocycle → plan
```

TrainWhen does not assume seven-day training cycles. A microcycle might contain 7, 10, 14, or another number of days.

Coaches and athletes are also first-class TrainWhen objects. They provide context used by the training hierarchy rather than sitting inside it.

## Build a library, not a pile of copies

TrainWhen objects are readable Markdown files.

An early project might look like:

```text
trainwhen/
├── coach.md
├── athletes/
├── exercises/
├── workouts/
├── progressions/
└── plans/
```

For example:

```text
workouts/
├── basic-ramp.md
├── easy-aerobic.md
└── explosive-strength.md
```

A warmup such as `basic-ramp` remains an ordinary reusable workout. The role is explicit when another workout uses it:

```text
- warmup: basic-ramp
```

Likewise:

```text
- cooldown: easy-aerobic
```

The filesystem organizes TrainWhen objects; it should not determine their semantics.

### Write once, reference later.

Copying workouts into every training plan defeats the purpose of having structured source.

TrainWhen objects and progression exposures will therefore have stable, human-readable identifiers.

The eventual planning experience should be approximately this simple:

```text
Day 1
- verk-a-01

Day 3
- easy-run

Day 5
- basic-ramp
```

A coach builds the training library once and prescribes from it.

The exact reference syntax is still being designed.

## Exercises

Exercises can also be first-class objects when doing so is useful.

A simple endurance activity does not require an exercise file:

```text
- run 45 min @ 80%
```

But a technical exercise such as a half-squat jump may benefit from a reusable definition containing information such as:

- name
- abbreviation
- instructions
- technique cues
- equipment
- demonstration video

The exercise definition describes the exercise.

The workout or progression describes how that exercise is prescribed.

For example, `35-45% squat 1RM` belongs to the explosive-strength prescription, not intrinsically to the definition of a half-squat jump.

## Markdown first.

TrainWhen uses Markdown as its document format and adds deterministic training syntax only where training semantics require it.

Where established Markdown conventions already solve a problem, TrainWhen should use them rather than inventing alternatives.

For example, Markdown comments use HTML comment syntax:

```text
<!-- This is a comment. -->
```

TrainWhen does not need to invent `//` comments.

Its structured training syntax favors familiar conventions:

```text
benchmark:
  run: AnT HR
  bike: FTP
```

Indentation expresses hierarchy.

Hyphens identify ordered executable items:

```text
- run 10 min @ easy
- run 20 min @ threshold
- run 10 min @ easy
```

TrainWhen files remain Markdown. They are not YAML documents, and TrainWhen does not require YAML configuration or front matter unless a future requirement demonstrates a need for it.

## Plain text is canonical.

The TrainWhen source is the source of truth.

Vendor formats and visualizations are generated representations:

```text
                         TrainWhen
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
          Garmin       Intervals.icu       CSV
            │
            ▼
           FIT

       + calendars, Markwhen,
         Mermaid, dashboards,
         and other adapters
```

A vendor's syntax does not define TrainWhen's syntax.

If an external platform requires a nonstandard representation—for example, `mtr` rather than the international standard of `m` for metres—the adapter should translate it.

TrainWhen remains:

```text
- run 400 m
```

**Adapters accommodate vendors. The canonical language does not contort itself around them.**

## v0.0.1

The first milestone is deliberately small.

TrainWhen v0.0.1 should prove that one readable, deterministic grammar can represent both a simple prescription:

```text
- run 45 min @ 80%
```

and a structurally demanding one:

```text
- 2 series
  - 6 sets
    - half-squat jump 8 reps @ 35-45% squat 1RM
      - tempo 1 rep/s
    - rest 60 s
  - recover 10 min
```

The initial implementation should parse representative TrainWhen source into a structured internal representation and reject invalid or ambiguous prescriptions with useful errors.

Higher-level features will build outward from that foundation.

Likely subsequent work includes:

- complete progression grammar
- shared progression defaults and overrides
- stable object references
- gateway and advancement logic
- microcycles
- mesocycles and macrocycles
- dated training plans
- multiple coaches and athletes
- goal events and stages
- FIT generation
- vendor adapters
- CSV exchange
- calendar generation
- Markwhen and Mermaid views
- web-based tooling

The roadmap is intentionally provisional.

## What TrainWhen is not

TrainWhen is not intended to:

- make a proprietary training platform the canonical copy of a plan
- require specialized software to read a workout
- require AI to interpret canonical syntax
- assume that every training cycle is seven days
- force every coach to use every level of the hierarchy
- duplicate athlete values throughout workout definitions
- duplicate shared prescription throughout a progression
- inherit awkward vendor syntax merely for compatibility

## Project status

TrainWhen is currently an experimental language design moving toward its first parser.

The syntax shown here is intended to communicate the direction of the project, not promise backward compatibility.

Examples and counterexamples are particularly valuable at this stage: the grammar should be driven by real training prescriptions rather than abstract language design.

## License

TrainWhen is intended to be an **open format that anyone can implement**, including in commercial software.

The TrainWhen reference implementation is licensed under the **Apache License 2.0**, permitting broad use, modification, distribution, and commercial implementation subject to the terms of that license.

The TrainWhen specification and documentation are licensed under **Creative Commons Attribution-ShareAlike 4.0 (CC BY-SA 4.0)**. The intent is to keep the format openly available and implementable while preserving attribution to TrainWhen and requiring adaptations of the specification to remain similarly open.

The **TrainWhen name and branding are separate** from the software and specification licenses. A trademark and naming policy may be established as the project matures, with the goal of allowing broad, accurate references to TrainWhen and compatible implementations while preventing misleading claims of official affiliation or endorsement.

These licenses do not prevent TrainWhen or others from building paid or proprietary products and services that implement or build upon the open format.
# TrainWhen

**Training plans should belong to coaches and athletes, not training platforms.**

TrainWhen is an open, plain-text system for writing workouts, workout progressions, and training plans.

The canonical source is readable **Obsidian-flavored Markdown**. TrainWhen builds on an established ecosystem of apps and plugins while keeping training source as portable plain text that can be versioned with Git, read without proprietary software, reused instead of copied, and exported to other systems and formats.

**Write once. Understand it. Reuse it. Export anywhere.**

> **Status: v0.0.1 — experimental**
>
> TrainWhen is currently being designed and implemented. Expect breaking syntax changes before the format stabilizes.

## Design principles

**Immediately understandable.**  
An athlete or coach should substantially understand a TrainWhen file before reading the specification.

**Markdown first.**  
Use established Markdown conventions where they work. TrainWhen uses Obsidian-flavored Markdown as its baseline and adds syntax only where training semantics require it.

**Don't repeat yourself.**  
Define shared information once, inherit it, and override it only where necessary.

**Deterministic.**  
A valid TrainWhen prescription has an unambiguous meaning. Software should be able to parse it without guessing.

**Composable.**  
Exercises build workouts. Workouts build progressions. Progressions build larger training structures.

**Explicit when necessary.**  
Defaults make common prescriptions concise without preventing a coach from saying exactly what is intended.

**Vendor-independent.**  
TrainWhen is the source of truth. Garmin, TrainingPeaks, Intervals.icu, Strava, and other platforms are places to send or receive training—not where the canonical plan has to live.

## Get started

A steady aerobic workout can be:

```text
- run 45 min @ 80%
```

A coach defines what a bare percentage means:

```text
# Coach

benchmark:
  run: AnT HR
  bike: FTP
```

An athlete supplies the values:

```text
# Athlete

benchmarks:
  run:
    AnT HR: 179 bpm
    AnT pace: 4.5 min/km
    HM: 5.69 min/km
  bike:
    AnT HR: 159 bpm
    AeT HR: 141 bpm
    FTP: 205 W
```

So:

```text
- run 45 min @ 80%
```

inherits `AnT HR` as the coach's running benchmark and resolves it using the athlete's running value.

Need something different? Say so locally:

```text
- run 45 min @ 80% AeT HR
- bike 20 min @ 95% FTP
- run 60 min <= AeT HR
```

**Define once. Inherit. Override when necessary.**

## From simple workouts to real training

TrainWhen is being designed against real training structures rather than only simple steady-state and interval workouts.

One of its initial test cases is Verkhoshansky's explosive-strength progression for runners.

The first session can be expressed as:

```text
# Explosive Strength

- warmup: [[basic-ramp]]

- 2 series
  - 6 sets
    - half-squat jump 8 reps @ 35-45% squat 1RM
      - tempo 1 rep/s
    - rest 60 s
  - recover 10 min

- cooldown: [[easy-aerobic]]
  - 10 min or until HR stabilizes
```

The structure should be apparent without learning a compressed workout notation.

Nesting carries meaning:

- `rest` is passive rest between sets.
- `recover` is active recovery between series.
- recovery can have its own prescription and is not necessarily easy.
- warmups and cooldowns reference reusable workouts.
- a RAMP warmup may also be prescribed as a complete workout.

For example, an active recovery could be:

```text
- recover 400 m @ marathon pace
```

TrainWhen can also derive rather than duplicate information:

```text
2 series × 6 sets × 8 reps = 96 jumps
```

## Progressions are first-class

Training is not merely a collection of independent workouts.

A **progression** describes how training changes across exposures. It can change exercise, volume, load, intensity, tempo, rest, recovery, or other prescription variables.

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

The compact representation is useful for titles and summaries. The canonical executable prescription remains explicit and nested so a human does not have to memorize positional shorthand.

### Don't repeat shared prescription

If all 16 sessions use the same warmup, cooldown, load, tempo, or other prescription, those values should not be copied sixteen times.

They belong at the progression level and are inherited by its workouts.

An individual workout specifies something again only when it differs.

> **Define a value once at the highest useful scope. Inherit it downward. Override it explicitly where necessary.**

### Gateways

A progression may eventually have a **gateway**: a readiness workout or assessment that determines whether an athlete is ready for the progression and where that athlete should enter it.

An advanced athlete should not necessarily have to begin at A01.

A beginner may not yet qualify for the progression or may require a scaled preparatory progression.

The exact gateway and advancement grammar is not part of v0.0.1.

## Compose training

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

TrainWhen objects are readable Markdown files connected by wiki links.

An early project might look like:

```text
trainwhen/
├── coach.md
├── athlete.md
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

Workouts, exercises, progressions, athletes, coaches, and plans can reference canonical objects elsewhere in the library rather than copying their contents.

A warmup such as `basic-ramp` remains an ordinary reusable workout. Its role is explicit when another workout uses it:

```text
- warmup: [[basic-ramp]]
```

Likewise:

```text
- cooldown: [[easy-aerobic]]
```

A referenced object has one canonical source. Change `[[basic-ramp]]`, for example, and every workout that references it resolves to the updated workout rather than retaining an outdated copy.

The filesystem organizes TrainWhen objects; it does not determine their semantics.

### Write once, reference later

TrainWhen uses Obsidian wiki links to reference canonical objects throughout the training library.

Planning can therefore be concise:

```text
Day 1
- [[verk-a-01]]

Day 3
- [[easy-run]]

Day 5
- [[basic-ramp]]
```

A coach builds the training library once and prescribes from it.

The same mechanism connects other objects:

```text
# Plan

coach: [[coach-canova]]
athlete: [[athlete-mosop]]
```

Obsidian's links, backlinks, and link-aware renaming make the library practical to maintain as it grows.

## Exercises

Exercises can be first-class objects when doing so is useful.

A simple endurance activity does not require an exercise file:

```text
- run 45 min @ 80%
```

But a technical exercise such as a half-squat jump may benefit from a reusable definition containing:

- name
- abbreviation
- instructions
- technique cues
- equipment
- demonstration video

The exercise definition describes the exercise.

The workout or progression describes how that exercise is prescribed.

For example, `35-45% squat 1RM` belongs to the explosive-strength prescription, not intrinsically to the definition of a half-squat jump.

## Markdown first

TrainWhen uses **Obsidian-flavored Markdown** as its baseline document format.

This provides established conventions for linking and maintaining a library of training objects without requiring TrainWhen to invent them.

Wiki links provide explicit, human-readable references:

```text
[[basic-ramp]]
[[easy-aerobic]]
[[verkhoshansky-explosive-strength]]
```

TrainWhen adopts Obsidian conventions where they solve a TrainWhen problem cleanly. Wiki links are the first important example.

Where Markdown already has an established convention, TrainWhen should use it rather than inventing an alternative. For example, Markdown comments use HTML comment syntax:

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

TrainWhen adds syntax only where Markdown does not express the required training semantics:

```text
- run 45 min @ 80%

- 2 series
  - 6 sets
    - half-squat jump 8 reps
    - rest 60 s
  - recover 10 min
```

Obsidian is not required to parse or execute TrainWhen. The files remain plain text, and the deterministic meaning of training prescriptions is defined by TrainWhen.

### Use the ecosystem

Using Obsidian-flavored Markdown gives coaches and athletes access to an established ecosystem of desktop and mobile apps, plugins, wiki links, backlinks, graph navigation, search, synchronization, and other tooling.

Markwhen's Obsidian integration is particularly relevant to TrainWhen. A TrainWhen parser can generate Markwhen representations for calendars, timelines, Gantt charts, and, where relevant, maps while the Markdown training library remains canonical.

These are generated views, not the source of truth.

## The source is yours

The TrainWhen source is the source of truth.

Vendor formats and visualizations are generated representations:

```text
                         TrainWhen
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       Obsidian          Markwhen          Garmin
          │                 │                 │
          │          calendars / Gantt        ▼
          │          timelines / maps        FIT
          │
          ├── links / backlinks
          └── editing / plugins

                  + Intervals.icu
                  + TrainingPeaks
                  + CSV
                  + calendars
                  + dashboards
                  + other adapters
```

Obsidian can provide an editing environment and ecosystem. Markwhen can provide generated temporal views. Training platforms can receive or provide training data.

None of them defines the canonical TrainWhen source.

If an external platform uses nonstandard notation—for example, `mtr` instead of the international standard `m` for metre—TrainWhen's adapter should handle the translation.

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
- assume every training cycle is seven days
- force every coach to use every level of the hierarchy
- duplicate athlete values throughout workout definitions
- duplicate shared prescription throughout a progression
- inherit awkward vendor syntax merely for compatibility
- require Obsidian to parse or execute training

## Project status

TrainWhen is currently an experimental language design moving toward its first parser.

The syntax shown here communicates the direction of the project; it does not promise backward compatibility.

Examples and counterexamples are particularly valuable at this stage. The grammar should be driven by real training prescriptions rather than abstract language design.

## License

TrainWhen is intended to be an **open format that anyone can implement**, including in commercial software.

The TrainWhen reference implementation is licensed under the **Apache License 2.0**, permitting broad use, modification, distribution, and commercial implementation subject to the terms of that license.

The TrainWhen specification and documentation are licensed under **Creative Commons Attribution-ShareAlike 4.0 (CC BY-SA 4.0)**. The intent is to keep the format openly available and implementable while preserving attribution to TrainWhen and requiring adaptations of the specification to remain similarly open.

The **TrainWhen name and branding are separate** from the software and specification licenses. A trademark and naming policy may be established as the project matures, with the goal of allowing broad, accurate references to TrainWhen and compatible implementations while preventing misleading claims of official affiliation or endorsement.

These licenses do not prevent TrainWhen or others from building paid or proprietary products and services that implement or build upon the open format.
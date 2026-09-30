# FIT Mapping for TrainWhen

This reference records the currently verified FIT workout mapping constraints. It supplements, but does not define, TrainWhen semantics.

Official references:

- [FIT Workout File](https://developer.garmin.com/fit/articles/file-types/workout.html)
- [Encoding FIT Workout Files](https://developer.garmin.com/fit/cookbook/encoding-workout-files/)

## Export pipeline

Resolve TrainWhen inheritance and containers before encoding:

```text
TrainWhen source
    → normalized ordered steps
    → FIT workout_step messages
```

FIT does not need to retain TrainWhen's inheritance structure. Repeated or inherited prescriptions may be flattened when necessary.

## Native workout-step fields

A FIT workout step can carry:

- `message_index`
- `wkt_step_name`
- `duration_type` and its dynamic value
- `target_type` and its dynamic values
- `intensity`
- `notes`
- `equipment`

The official workout documentation lists native target types for speed, heart rate, open, cadence, power, and swimming stroke. Do not claim another component is a native FIT target without checking the current FIT profile.

## Current TrainWhen mapping

| TrainWhen component | FIT representation |
|---|---|
| Timed duration | `duration_type = time` plus duration value |
| Open duration | `duration_type = open` |
| Cooldown | `intensity = cooldown`; duration may be open |
| Heart-rate target | Native heart-rate target when resolvable to a FIT-supported zone or range |
| Speed target | Native speed target when expressed in a supported numeric form |
| Power target | Native power target when expressed in a supported zone or range |
| Cadence target | Native cadence target when expressed in a supported form |
| RPE | Display text in `notes` or `wkt_step_name`; not a documented native planned-step target |
| Treadmill incline | Display text in `notes` or `wkt_step_name`; not a documented native planned-step target |
| Free-text instruction | `notes` when supported by the destination device or platform |
| TrainWhen inheritance | Resolve and flatten before encoding |

Workout RPE recorded after an activity is not the same thing as a planned workout-step RPE target.

## Repeat blocks

FIT can encode repeat blocks using a repeat workout-step message referring back to a prior `message_index`. Use a native repeat only when it preserves TrainWhen boundary semantics.

A TrainWhen repeated container whose terminal rest or recovery is interstitial may require expansion or restructuring so the FIT workout does not add that transition after the final repetition.

## Import rules

When decoding a FIT workout:

1. Read every `workout_step` message in `message_index` order.
2. Read structured duration, target, intensity, repeat, note, and equipment fields.
3. Preserve `wkt_step_name` and `notes` as source instructions.
4. Do not infer RPE, pace, load, or incline when absent.
5. Treat display text as an instruction unless TrainWhen has a deterministic parser for that phrase.
6. Report data that cannot be represented in the current TrainWhen specification.

## Verification

After writing a FIT file:

1. Decode it with an independent FIT reader when possible.
2. Verify the file integrity or CRC.
3. Compare decoded step order, durations, targets, intensity values, notes, and repeats with the normalized TrainWhen prescription.
4. Report any component lowered from structured data to display text.

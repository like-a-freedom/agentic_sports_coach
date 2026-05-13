---
name: intervals-icu-integration
description: Use when creating or correcting Intervals.icu workouts, debugging Workout Builder parsing or rendering issues, managing calendar events, retrieving athlete data, or updating sport settings via MCP.
---

# Intervals.icu Integration

## Overview

MCP-first guide for Intervals.icu planning and calendar edits.

**Core principle:** workout text must be not only syntactically valid, but also parser-safe and predictable in Workout Builder UI.

## When to use this skill
Use this skill when the request involves:
- Creating training plans in Intervals.icu format (Workout Builder syntax)
- Creating, updating, or deleting events in athlete's calendar
- Retrieving training data (activities, fitness metrics, wellness, performance curves)
- Managing sport settings (thresholds: AeT/LT/FTP/FTHR)
- Applying threshold changes and recalculating historical data
- Fetching athlete profile, upcoming workouts, calendar events
- Debugging broken workout rendering, inflated workout duration, or missing warmup/cooldown blocks in Workout Builder
- Fixing mixed target-type workouts (`HR` + `Pace`) that behave poorly when the user switches Builder mode
- Converting third-party workouts, screenshots, or loose prose into valid Workout Builder syntax
- Generating uploadable workouts/plans incrementally with an LLM without syntax drift or truncation

## Core principle: MCP-first approach
**Always gather data via MCP tools** before creating a plan or modifying workouts.
Do not rely solely on what the user tells you if the data can be fetched through MCP.

## Builder-safe text vs weekly-plan text

This distinction matters for agentic scenarios:

- A **weekly plan / calendar outline** may contain ordinary explanatory bullet lines, weekly totals, and text notes.
- The **body of a specific workout destined for Workout Builder** must use `-` lines **only** for real steps/intervals.

When explanatory text is needed inside a workout:
- use a `Notes` or `Fueling` section with plain lines that do **not** start with `-`; or
- embed short cues as step cue / text prompt inside a valid step line.

Failing to draw this distinction causes AI to produce "nicely formatted" text that breaks preview, calculated duration, or device export.

## MCP token efficiency and contracts

### Server version awareness

- MCP contracts and tool names may change between `rusty-interva` versions.
- Always check the current schema of the specific tool before calling it.

### Token-saving practices

1. Use aggregate / list tools (`list*`, `search*`) first, then drill into detail tools (`get*`) only for target entities.
2. Restrict date ranges and result sizes (`oldest/newest`, `limit`).
3. For curves/histograms, request only the required time range and sport type.

### Canonical naming (low-level, rare cases)

- The **primary interface** is intent tools (`mcp_rusty-interva_*`); they cover the vast majority of scenarios. This section describes only low-level calls, which are now considered a fallback.

## Output format: intervals.icu

### Strict output contract (important for agentic use)

**1) Training plan (one or more weeks):**
Output ONLY the plan text in intervals.icu format (ready to copy-paste):
- No Markdown headings (##), no bold (**), no code fences (```), no tables
- Embed any "why" explanations inside workout descriptions as lines starting with `-` (Purpose / Focus / Why)
- Plain service lines are allowed inside a plan: "Weekly totals:", "Key decisions:", "What to track:" — but without Markdown

**2) Analysis / explanations separately:**
Output the intervals.icu plan first, then (after a blank line) a brief analysis in plain prose.

**3) Check arithmetic:**
- AeT-LT gap (%) = (LT − AeT) / AeT × 100
- If data or examples are inconsistent, correct them explicitly and use the corrected value.

### intervals.icu output structure

```
#WEEK [week number]
[Day of week], [duration], [workout name]
- [description / structure]

Weekly totals:
- Time: [total time]
- Vertical gain: [elevation]
- Zone distribution: [zone breakdown]
```

**Notation:**
- Duration: `HH:MM` (e.g. `1:30` = 1 hour 30 minutes)
- Workout Builder step duration: `30s`, `10m`, `1h10m`, `1m30` (use "m" and "s", not "min/sec")
- Zones: `Z1`, `Z2`, `Z3`, `Z4`, `Z5`, `Recovery`
- Structure: warm-up, main set, cooldown
- `REST`: rest day

**Example:**

```
#WEEK 1
Monday, 0:45, Recovery Run
- 45m Z1 HR easy conversational pace, nose breathing

Tuesday, 1:30, Endurance Run
- 15m Z1 HR warm-up
- 60m Z2 HR
- 15m Z1 HR cooldown

Wednesday, REST

Thursday, 1:00, Recovery Run + Strides
- 45m Z1 HR
Strides 6x
- 15s Z4 Pace (relaxed fast, not sprint)
- 1m45 Z1 HR

Friday, REST

Saturday, 2:30, Long Endurance Run
- 2h30m Z1-Z2 HR, focus on vertical gain (aim for 600m+)

Sunday, 1:00, Recovery Run
- 1h Z1 HR very easy

Weekly totals:
- Time: 7:15
- Vertical gain: ~1200m (estimate)
- Zone distribution: 85% Z1-Z2, 15% strides/pick-ups
```

### Workout Builder syntax reference

#### General workout structure

A workout is plain text divided into **sections** and **steps**:
- **Sections** — lines without `-` (block headers): `Warmup`, `Main Set 4x`, `Cooldown`
- **Steps** — lines starting with `-`. Each step = one interval
- A blank line between sections is recommended

Minimal structure example:

    Warmup
    - 10m 60%

    Main Set 5x
    - 3m 120%
    - 2m Z1

    Cooldown
    - 10m 50%

#### Parser-safe notes and comments

> ⚠️ **CRITICAL**: any line starting with `-` is interpreted by Workout Builder as a step/interval.

Use `-` **only** for real workout steps. Service comments (`Nutrition`, `Recovery`, `Fueling`, `Why`, long cues) must not look like intervals.

**Bad — note is parsed as a step:**

    - Nutrition: start fueling at 15-20m; 45-60 g/h
    - Recovery: within 30m take carbs + protein

These lines can:
- inflate calculated duration,
- break the workout preview structure,
- turn text notes into pseudo-intervals.

**Good — notes placed in plain text:**

    Warmup
    - 10m Z1 HR

    Main Set
    - 100mtr Z2 HR

    Cooldown
    - 10m Z1 HR

    Notes
    Nutrition: start fueling early; CHO 45-60 g/h; fluid 450-750 ml/h
    Recovery: within 30 min take 1.0-1.2 g/kg CHO + 20-30 g protein

**Practical rule:** if a line is not meant to become a Workout Builder step, do not start it with `-`.

For AI generation the safest default is:
- first produce a **clean workout body** without any prose bullets;
- then, if the user needs explanations, append them in a `Notes`/`Why`/`Fueling` section as plain text.

#### Duration

| Syntax | Meaning |
|--------|---------|
| `30s` | 30 seconds |
| `90s` | 90 seconds |
| `5m` | 5 minutes |
| `1h` | 1 hour |
| `5m30s` | 5 minutes 30 seconds |
| `1m30` | 1 min 30 sec |
| `1h10m` | 1 hour 10 minutes |
| `1h30m59s` | 1 hour 30 minutes 59 seconds |
| `5'` | 5 minutes (alternative) |
| `30"` | 30 seconds (alternative) |

#### Distance

| Syntax | Meaning |
|--------|---------|
| `2km` / `2 km` | 2 kilometres |
| `1mi` / `1 mile` | 1 mile |
| `0.4km` | 400 metres |
| `400mtr` / `400 meters` | 400 metres |

> ⚠️ **CRITICAL**: `m` = minutes, NOT metres! Use `0.4km` or `400mtr` for metres. Never write `400m` — that means 400 minutes!

Supported distance units: `km`, `mi`, `mile`, `miles`, `mtr`, `meters`, `yrd`, `yards`, `y`.
A space between number and unit is allowed: `1km` and `1 km` are both valid.

#### Intensity

**Cycling (% of FTP):**

    - 10m 75%           # 75% FTP
    - 5m 95-105%        # range 95–105% FTP
    - 8m 220w           # absolute watts
    - 6m 200-240w       # watt range
    - 60m Z2            # zone 2 (power)
    - 6m Z3-Z4          # zone range
    - 4m CZ2            # custom zone
    - 4m CZ2-CZ3        # custom zone range
    - 5m 60% MMP 5m     # % of best effort / MMP anchor

**Running (pace):**

    - 6km 90-92% pace        # % of threshold pace
    - 3km Z3 Pace            # pace zone 3
    - 8m Z2-Z3 Pace          # pace zone range
    - 10m 7:15-7:00/km Pace  # absolute pace
    - 10m 5:00-4:45/400m Pace  # track units also supported

**Heart rate (HR):**

    - 60m Z2 HR         # HR zone 2
    - 20m 70% HR        # 70% of max HR
    - 10m 90-95% LTHR   # 90–95% of threshold HR
    - 12m Z2-Z3 HR      # HR zone range

> ⚠️ For running, ALWAYS specify the target type: `Z2 HR` or `Z2 Pace` — without qualification the parser may interpret it as power.

**Advanced target notes:**

- `MMP` targets are supported, e.g. `60% MMP 5m`.
- `CZ1`, `CZ2-CZ3` etc. are supported for **custom zones**.
- Custom zones depend on the athlete's zone configuration and anchors; if unsure whether they are set up, prefer standard `Z1-Z6`.
- Absolute pace is a great option for athlete-specific workouts generated outside Intervals.icu, but it is **not portable between athletes**. For general-purpose library workouts, `Zx Pace` or `% Pace` is usually safer.

#### Zone tables

**Cycling (% of FTP):**

| Zone | Name | % FTP |
|------|------|-------|
| Z1 | Recovery | < 55% |
| Z2 | Endurance | 56–75% |
| Z3 | Tempo | 76–90% |
| Z4 | Threshold | 91–105% |
| Z5 | VO2 Max | 106–120% |
| Z6 | Anaerobic | > 120% |

**Running (% of threshold pace):**

| Zone | Name | % Pace |
|------|------|--------|
| Z1 | Recovery | 70–75% |
| Z2 | Easy | 76–80% |
| Z3 | Tempo | 81–88% |
| Z4 | Threshold | 89–95% |
| Z5 | VO2 Max | 96–105% |
| Z6 | Sprint | > 105% |

#### Repeats

Two ways to specify repeats:
1. In the section header: `Main Set 5x`
2. As a standalone line before the steps:

       5x
       - 3m 120%
       - 2m Z1

> 💡 For readability and more reliable AI generation, leave a blank line before and after every repeat block (`Main Set 5x` or standalone `5x`).

> ⚠️ Nested repeats are not supported.

Make sure the repeat label does not get separated from its steps. An orphaned `3x` / `5x` is a common LLM generation mistake.

#### Ramps

Gradual increase or decrease — keyword `ramp`:

    - 10m ramp 50%-75%        # warmup: power ramp up
    - 10m ramp 60-80% pace    # pace ramp up (running)
    - 8m ramp 50%-40%         # ramp down on cooldown
    - 15m ramp 60%-90% 85rpm  # ramp up + cadence (cycling)

Especially useful for smooth warmups/cooldowns instead of stepped transitions.

#### Cadence (cycling only)

Appended at the end of the line as `NNrpm` or a range:

    - 10m 75% 90rpm          # fixed 90 rpm
    - 12m 85% 90-100rpm      # cadence range
    - 15m ramp 60-90% 85rpm  # ramp + cadence

#### Text Prompts (device cues)

Text before the first duration or power token becomes the device cue (sent to Garmin/Wahoo):

    - Recovery 30s 50%
    - Zone 4 interval 5m 95-105%
    - Low cadence 4m 100%

The repeat section header also becomes cue text. For example, `Main Set 6x` typically produces cues like `Main Set 1/6`, `Main Set 2/6`, etc.

#### Timed Prompts (time-offset messages)

Messages triggered at a specific second within a step; the `<!>` separator is required:

    - First prompt at 0s 33^Second prompt at 33s <!> 10m ramp 25-75%
    - First 60^30 Second 120^30 Third <!> 10m 65%
    - 20^Fuel now <!> 5m 85%

Rules:
- `33^Second prompt` = show `Second prompt` at second 33 of the step.
- `60^30 Second` = show `Second` at second 60, for 30 seconds.
- `<!>` separates the timed-prompt part from the actual step; always include it in new AI-generated workouts.

Platform caveats:
- Full functionality works primarily with `ZWO` / Zwift sync.
- On non-ZWO platforms, messages are typically concatenated into a single step text.
- In repeat / `IntervalsT` scenarios, timed prompts may degrade to one combined cue.
- Do not mix timed prompts with localised multi-language step syntax such as `en/Hello fr/Bonjour` — they are incompatible.

#### Freeride (ERG off)

A step with no power target — free pedalling:

    - 20m freeride

#### Rich formatting inside workout text

Workout Builder can ignore some formatting markup if it does not look like a workout step. This helps readability, but should not replace the actual step structure.

Generally safe for display:
- Markdown headings: `#`, `###`
- Emphasis: `**bold**`, `*italic*`
- Links: `[label](https://example.com)`
- Tables
- Visual separators: `---`
- Some Vuetify classes, e.g. `<span class="d-none">...</span>`

Practical recommendations:
- For automated generation, default to **plain and boring** output: steps plus minimal notes.
- Do not use HTML comments / hidden metadata as an internal-storage mechanism for device-synced workouts. According to the forum, this can break display on Garmin and similar devices.
- If you need to persist service metadata, keep it outside the workout body: in MCP, a separate field, a calendar event description, or an external store.

#### LLM workflow recommendations

When a workout is generated by an AI/LLM:

1. Produce **plain-text Workout Builder syntax** first, not JSON.
2. Verify repeats, units, pace targets, and cue text before converting to an API payload if needed.
3. For a macro plan, first generate the annual/mesocycle outline, then materialise uploadable workouts in chunks of **1–3 weeks**.
4. For exotic constructs (`timed prompts`, `absolute pace`, `Press lap`, `CZ`, `MMP`), cross-check against known-good examples or the `Add Step` form.

This reduces the risk of truncation, orphaned repeat labels, wrong bullet glyphs, and "almost correct" syntax that breaks on export.

#### Mixing target types

HR, Pace, and Power can be combined in one workout:

    Warmup
    - 10m Z1 HR

    Main Set 4x
    - 6m 100%
    - 3m 75-80% pace

    Cooldown
    - 10m ramp 60-40% HR

> ⚠️ **Practical caveat for running:** mixing `HR` and `Pace` is syntactically valid, but does not always produce a predictable UX in Workout Builder.

For run workouts that the user will open or edit in `Pace` mode, mixing `HR + Pace` in one session can cause warmup/cooldown/recovery blocks to display awkwardly or visually disappear when switching mode.

**Reliable rule:**
- if the main set uses `Pace`, prefer `Pace` for warmup/cooldown/recovery in the same session too;
- if the workout is primarily HR-guided, keep the entire session in `HR`;
- mix types only when genuinely needed, not as a default.

#### Practical rule for Pace-mode run workouts

For test sessions, time trials, and pace-oriented workouts, prefer a **single `Pace` type throughout the session**.

**Preferred pattern:**

    Warmup
    - 15m Z1 Pace

    Main Set
    - 46m Z4 Pace

    Cooldown
    - 10m Z1 Pace

For workouts such as 5K / 10K / HM tests, **relative Pace** is usually more convenient:
- `Z3 Pace`
- `Z4 Pace`
- `88-92% Pace`

rather than absolute pace as the default, when the goal is stable display and editing in Builder `Pace` mode.

Absolute pace (`4:50-4:35/km Pace`) remains valid and useful when a precise target range is needed, but relative Pace is generally more reliable as the standard format for editable running workouts.

#### Complete examples (running)

**Aerobic run with strides:**

    Warmup
    - 15m Z1 HR

    Main Set
    - 50m Z2 HR

    Strides 6x
    - 15s Z4 Pace
    - 1m45 Z1 HR

    Cooldown
    - 13m Z1 HR

**Distance-based intervals:**

    Warmup
    - 1mi Z1 HR

    Main Set 4x
    - 1mi Z4 Pace
    - 0.5mi Z1 Pace

    Cooldown
    - 1mi Z1 Pace

#### Distance step unit summary

Supported: `km`, `mi`, `meters`, `yrd`, `y`
- `- 3km 80% Pace`
- `- 0.4km Z3 Pace` (400 metres)
- `- 10m 7:15-7:00/km Pace` (absolute pace with `/km`, `/mi` units)

## Common mistakes

- Using `-` for plain notes (`Nutrition`, `Recovery`, `Fueling`) — Builder treats them as intervals.
- Writing duration-like tokens (`15-20m`, `30m`) in pseudo-steps starting with `-` — this can break preview and duration calculation.
- Using `100m` / `400m` when metres are intended, not minutes. Use `mtr` / `meters` / `km` for metres.
- Mixing `HR` warmup/cooldown with a `Pace` main set in workouts that will later be opened in Builder `Pace` mode.
- Using absolute Pace as the sole standard for editable run tests when relative Pace gives more predictable UI behaviour.
- Leaving orphaned `3x` / `5x`, or placing a repeat marker before the wrong block.
- Using bullet symbol `•` instead of the plain `-` in step lines.
- Trying to generate a very long JSON / full-year upload payload in a single LLM request — better to materialise in 1–3 week chunks.
- Storing hidden comments / HTML metadata inside the workout body and expecting all devices to safely ignore it.

### Strength training (Weight training)

**Calendar event format in Intervals.icu:**
- Use a brief line in the calendar such as: `* 1h 40-70% HR (72-125bpm)`
  - First part is duration
  - Then the target %HR range
  - Absolute bpm as a reference

**In Workout Builder:**
- Use one main line with duration and goal: `* 1h 40-70% HR`
- Add sub-lines with exercises and sets

**In `workout_doc`:**
- step: `duration: 3600` and `hr` with `units: "%hr"`, `start: 40`, `end: 70`
- This allows Intervals.icu to correctly calculate moving_time and zone

**In workout description:**
- Add structure (exercises, sets, rest periods)
- Note with absolute bpm as a reference (e.g. 72-125 bpm)

## Intervals.icu MCP integration (rusty-intervals-mcp, intent model v2+)

### Intent-first contract

Use intent tools as the primary interface:

- `mcp_rusty-interva_plan_training`
- `mcp_rusty-interva_analyze_training`
- `mcp_rusty-interva_modify_training`
- `mcp_rusty-interva_compare_periods`
- `mcp_rusty-interva_assess_recovery`
- `mcp_rusty-interva_manage_profile`
- `mcp_rusty-interva_manage_gear`
- `mcp_rusty-interva_analyze_race`

### Required context gathering before planning

Before generating a plan for one or more weeks, always run:

1. `mcp_rusty-interva_manage_profile` (`action: get`) — profile, zones, thresholds
2. `mcp_rusty-interva_analyze_training` (`target_type: period`) — actual volume/intensity
3. `mcp_rusty-interva_assess_recovery` — readiness + red flags
4. If needed: `mcp_rusty-interva_compare_periods` (7/30/90 days)

### Planning and modifications

- When the user asks to create a plan (week or block), use `mcp_rusty-interva_plan_training`.
- When adjusting an existing workout, use `mcp_rusty-interva_modify_training`.
- For destructive operations (`action: delete`, bulk edits) first run `dry_run: true`, then ask for explicit user confirmation.

### Managing thresholds and zones

Only change thresholds if:
1) the user explicitly requests it; or
2) a new VO2max lab protocol is available and the user has confirmed applying it.

Workflow:

1. `mcp_rusty-interva_manage_profile` (`action: get`) — collect current thresholds, make a diff
2. `mcp_rusty-interva_manage_profile` (`action: update_thresholds`, `thresholds_source`)
3. Apply to historical data only after user consent: `apply_to_activities: true`
4. Verify the effect via:
   - `mcp_rusty-interva_manage_profile` (`action: get`)
   - `mcp_rusty-interva_analyze_training` (`target_type: period`)
   - `mcp_rusty-interva_assess_recovery`

### Analysing training and progress

- Detailed single-session breakdown: `mcp_rusty-interva_analyze_training` (`target_type: single`, `analysis_type: detailed|intervals|streams`)
  - `detailed` — zones + key metrics
  - `intervals` — interval analysis (for interval workouts)
  - `streams` — second-by-second stream data (HR, pace, power)
  - `include_best_efforts: true` — comparison with best efforts
  - `include_histograms: true` — HR/pace/power histograms
  - `metrics: [...]` — specific metrics: time, distance, vertical, tss, pace, hr
- Period analysis: `mcp_rusty-interva_analyze_training` (`target_type: period`)
- Block comparison: `mcp_rusty-interva_compare_periods`

### Current `mcp_rusty-interva_analyze_race` contract

Use `mcp_rusty-interva_analyze_race` for **post-race debrief**, strategy evaluation, and identifying areas for improvement after a completed race.

**Input fields per current contract:**
- `date?: "YYYY-MM-DD" | "last_race"`
- `description_contains?: string`
- `analysis_type?: "performance" | "strategy" | "recovery"` (default: `performance`)
- `compare_to_planned?: boolean` (default: `true`)

**Practical rule:**
- The primary working selector is `description_contains`.
- If `description_contains` is not provided, the current implementation picks the most recent activity from a recent list.
- Therefore **do not rely solely on `date` / `analysis_type` / `compare_to_planned`** to select the right race or change output logic — always verify that the response's `name`, date, and ID match the intended race.

**What the intent fetches automatically:**
- recent activities (to locate the race)
- workout details
- intervals
- streams
- fitness summary
- wellness for the last 7 days

**What is typically in the response:**
- `Race Analysis` + race metadata
- result table (distance, time, avg HR)
- `Execution Pattern` (segments, average HR note, Efficiency Factor, Aerobic Decoupling)
- `Post-Race Load Context` (e.g. recovery block recommendation based on TSB)
- `Data Availability` (full mode or degraded mode)
- `suggestions` / `next_actions`

**Current implementation limitations:**
- `compare_to_planned` is in the schema but does not guarantee a full planned-vs-actual diff — do not promise the user something the intent does not currently build explicitly.
- `analysis_type` is accepted by the schema but should not be treated as a reliable switch between different output formats without checking the actual response.
- When no matching race is found, the intent returns soft suggestions/next actions rather than a hard error.

**Recommended post-race workflow:**
1. `mcp_rusty-interva_analyze_race` with `description_contains`
2. Check `Data Availability` and lower confidence of conclusions if streams/intervals/wellness are absent
3. `mcp_rusty-interva_assess_recovery` (`period_days: 7` or `14`)
4. If needed: `mcp_rusty-interva_plan_training` for recovery week / next block

**When deeper drill-down is needed:**
- If a detailed breakdown of streams/intervals beyond the race debrief is required, supplement with `mcp_rusty-interva_analyze_training` for the target session and reconcile the findings.

### Write operation rules

- Never execute a mutating intent without explicit user instruction.
- Before any calendar or plan change, briefly confirm what will be modified.
- If the user asks for "text only" / "don't touch the calendar", output only the intervals.icu text plan.

## Intervals.icu documentation

Official documentation for Workout Builder:
- https://forum.intervals.icu/t/workout-builder/1163
- https://forum.intervals.icu/t/workout-builder-syntax-quick-guide/123701
- https://www.intervals.icu/features/workout-builder/
- https://forum.intervals.icu/t/text-events-are-now-supported-in-the-workout-builder/96016
- https://forum.intervals.icu/t/specify-workouts-using-absolute-pace/115846
- https://forum.intervals.icu/t/distanced-based-workouts-supported/9973
- https://www.intervals.icu/features/custom-zones/
- https://forum.intervals.icu/t/using-ia-chatgpt-to-write-intervals-icu-workouts/85094
- https://forum.intervals.icu/t/computed-activity-fields/25673
- https://forum.intervals.icu/t/custom-interval-fields/25942
- https://forum.intervals.icu/t/custom-activity-charts/28627
- https://intervals.icu/api-docs.html

## When to use other skills

- **Periodization, zones, strength training**: see `periodization-coach` skill
- **Recovery monitoring, red flags, VO2max lab results**: see `athlete-monitoring` skill
- **Injury prevention, evidence-based practice**: see `kinesiology-foundations` skill

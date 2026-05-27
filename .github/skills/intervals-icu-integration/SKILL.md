---
name: intervals-icu-integration
description: Technical integration with Intervals.icu platform via MCP. Covers workout format, event management (create/update/delete), sport settings, performance curves, and data retrieval workflows.
---

# Intervals.icu Integration

## When to use this skill
Use this skill when the request involves:
- Creating training plans in Intervals.icu format (Workout Builder syntax)
- Creating, updating, or deleting events in athlete's calendar
- Retrieving training data (activities, fitness metrics, wellness, performance curves)
- Managing sport settings (thresholds: AeT/LT/FTP/FTHR)
- Applying threshold changes and recalculating historical data
- Fetching athlete profile, upcoming workouts, calendar events

## Core principle: MCP-first approach
**ALWAYS collect data via MCP tools** before creating a plan or adjusting workouts.
Do not rely solely on the user's words if the data can be obtained via MCP.

## Token efficiency and MCP contracts

### Important about server versions

- Use the current Intervals MCP contract exposed in this workspace.
- Do not use legacy aliases or older OpenAPI examples as source of truth.
- Before calling, always refer to the current schema of the specific tool in MCP.

### Practice for token efficiency

1. First use high-level intents (`analyze_training`, `assess_recovery`, `manage_profile`), then move to detailed analysis via `target_type`/`analysis_type` parameters.
2. **Collect data in tiers** to avoid redundant calls:
   - **Tier 1:** `manage_profile` (overview+zones+thresholds+metrics) + `analyze_training period summary` + `assess_recovery`
        - Period summaries can be sparse; sometimes you only get basic totals (time, distance, elevation, weekly average).
   - **Tier 2:** `analyze_training period` with `analysis_type: "intervals"` or `"streams"` — ONLY if tier 1 left gaps
   - **Tier 3:** `analyze_training single` for 2-3 key dates — only after tiers 1-2 show what needs closer look
   - **Avoid** calling the same period with 3 different `analysis_type` values (summary + intervals + streams) — pick one that fits the question
3. Limit date ranges via `period_start`/`period_end` (analyze_training period) or `period_days` (assess_recovery).
4. Use `analyze_training` target_type: "period" for period context (calendar included), "single" for detailed review (with analysis_type for depth).
5. For curves/histograms use `analyze_training` only when the current response returns those fields; otherwise stick to the raw summary/interval data that is actually present.
6. **Memory tool discipline:** No more than 2 consecutive memory write calls without an MCP mutation or a response to the user. If you need to "fix" a memory entry 3+ times, stop and re-assess what's wrong instead of looping.

### Current MCP data surfaces

Only treat the following fields as available when the current MCP response actually returns them:

- `manage_profile get`: overview, zones, thresholds, metrics
- `analyze_training period`: always at least time, distance, elevation, weekly avg; richer trend/load fields (trend context, load context, NDLI, heat stress, load patterns, terrain specificity) may appear when enough data is available
- `analyze_training single` / `intervals` / `streams`: rep-level HR/power/pace, duration, cadence, TSS/load, decoupling, W′, pMax, histograms, best efforts, Z2 stability, and stream-level details
- `manage_gear list`: gear tables grouped by type when available (for example shoes), usually with name/distance/remaining/status columns plus next-actions
- `assess_recovery`: sleep, resting HR, HRV, TSB, recovery quality, recovery index, readiness, red flags
- `analyze_race`: only returns a race analysis when the activity is actually found/tagged; otherwise expect a no-match response
- `compare_periods`: may return a 429/rate-limit error on broader or repeated comparisons; narrow the windows and retry if needed. If one or both windows are empty, expect zero totals and `zones` marked unavailable/n/a.
- `track_progress`: plateau detection and hypotheses are conditional on available history; insufficient CTL or wellness history can produce unavailable fields and warnings

If a field is not present in the current MCP response, mark it `unavailable` instead of inferring it.

## Output format: intervals.icu

### STRICT OUTPUT CONTRACT (important for agency)

**1) Training plan (week/multiple weeks):**
Output ONLY the plan text in intervals.icu format (ready to copy and paste):
- Without Markdown headers (##), without bold (**), without code fences (```), without tables
- Embed any explanations/"why" inside workout descriptions using lines starting with "-" (Purpose/Focus/Why)
- Service lines inside the plan are allowed: "Weekly totals:", "Key decisions:", "What to track:" — but also without Markdown

**2) Analysis/explanations separately:**
First provide the plan in intervals.icu format, then (after a blank line) a brief analysis in plain text.

**3) Check arithmetic:**
- AeT-LT gap (%) = (LT - AeT) / AeT × 100
- If there are inconsistencies in the data/example — explicitly correct and use the correct value

### Output structure for intervals.icu

```
#WEEK [week number]
[Day of week], [duration], [workout name]
- [description/structure]

Weekly totals:
- Time: [total time]
- Vertical gain: [vertical gain]
- Zone distribution: [zone distribution]
```

**Notation:**
- Duration: `HH:MM` (e.g., `1:30` = 1 hour 30 minutes)
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

### Workout Builder syntax (reference)

#### General workout structure

A workout is plain text. It is divided into **sections** and **steps**:
- **Sections** — lines without `-` (block headers): `Warmup`, `Main Set 4x`, `Cooldown`
- **Steps** — lines starting with `-`. Each step = one interval
- A blank line between sections is recommended

Example of minimal structure:

    Warmup
    - 10m 60%

    Main Set 5x
    - 3m 120%
    - 2m Z1

    Cooldown
    - 10m 50%

#### Duration

| Syntax | Meaning |
|--------|---------|
| `30s` | 30 seconds |
| `5m` | 5 minutes |
| `1h` | 1 hour |
| `1m30` | 1 min 30 sec |
| `1h10m` | 1 hour 10 minutes |
| `1h30m59s` | 1 hour 30 minutes 59 seconds |
| `5'` | 5 minutes (alternative) |
| `30"` | 30 seconds (alternative) |

#### Distance

| Syntax | Meaning |
|--------|---------|
| `2km` | 2 kilometers |
| `1mi` | 1 mile |
| `0.4km` | 400 meters |
| `400mtr` | 400 meters |

> ⚠️ **CRITICAL**: `m` = minutes, NOT meters! For meters: `0.4km` or `400mtr`. Never `400m` — that is 400 minutes!

#### Intensity

**Cycling (% of FTP):**

    - 10m 75%           # 75% FTP
    - 5m 95-105%        # range 95–105% FTP
    - 8m 220w           # absolute watts
    - 6m 200-240w       # watt range
    - 60m Z2            # zone 2 (power)
    - 20m 85-95% MMP 5m # % of 5-minute max power
    - 60m CZ1           # custom zone 1

**Running (pace):**

    - 6km 90-92% pace        # % of threshold pace
    - 3km Z3 Pace            # pace zone 3
    - 10m 7:15-7:00/km Pace  # absolute pace

**Heart Rate (HR):**

    - 60m Z2 HR         # zone 2 by HR
    - 20m 70% HR        # 70% of max HR
    - 10m 90-95% LTHR   # 90–95% of threshold HR

**Custom zones:**

All activity streams support custom zones (CZ prefix):

    - 60m CZ1           # custom zone 1 (power)
    - 45m CZ2-CZ3 HR    # custom zone 2-3 by HR
    - 30m CZ2 Pace      # custom zone 2 by pace

> ⚠️ For running ALWAYS specify the target type: `Z2 HR` or `Z2 Pace` — without specification it may be interpreted as power.

#### Zone tables

**Cycling (% of FTP):**

| Zone | Name | % FTP |
|------|------|-------|
| Z1 | Recovery | < 55% |
| Z2 | Endurance | 55–75% |
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

Two ways:
1. In section header: `Main Set 5x`
2. On a separate line before the steps:

       5x
       - 3m 120%
       - 2m Z1

> ⚠️ Leave one empty line before and after every repeat block.
> ⚠️ Nested repeats are not supported.

#### Ramp

Gradual increase/decrease — keyword `ramp`:

    - 10m ramp 50%-75%        # warm-up: power ramp
    - 10m ramp 60-80% pace    # pace ramp (running)
    - 8m ramp 50%-40%         # cooldown ramp
    - 15m ramp 60%-90% 85rpm  # ramp + cadence (cycling)

Especially useful for smooth warm-ups/cooldowns instead of step transitions.

#### Cadence (cycling only)

Add to the end of a line as `NNrpm` or a range:

    - 10m 75% 90rpm       # fixed 90 rpm
    - 12m 85% 90-100rpm   # cadence range
    - 15m ramp 60-90% 85rpm  # ramp + cadence

#### Text Prompts (device display)

Text before the first duration specification = prompt on Garmin/Wahoo:

    - Recovery 30s 50%
    - Zone 4 interval 5m 95-105%
    - Low cadence 4m 100%

#### Timed Prompts

Messages at a specific moment within a step (seconds from start), separator `<!>` required:

    - Start sprint 30^ Relax 60^ Final push <!> 1m30 120%

#### Formatting text inside workout steps

Intervals.icu supports standard Markdown and Vuetify classes for readability. These are ignored during parsing:

    # Title
    **bold** *italic* ***bold italic***
    [link](https://example.com)
    ---
    <p class="text-red">This text is red</p>
    <span class="d-none">This text is hidden</span>

Can be mixed with workout steps in the same event description.

#### Freeride (ERG off)

A step without power control — free pedaling:

    - 20m freeride

#### Lap button (Garmin sync)

End a step when the lap button is pressed on device — append `Press lap`:

    - 1h Z2 Press lap

Only works on workouts pushed to Garmin devices via Garmin Connect integration.

#### Mixing target types

HR, Pace, and Power can be combined in a single workout:

    Warmup
    - 10m Z1 HR

    Main Set 4x
    - 6m 100%
    - 3m 75-80% pace

    Cooldown
    - 10m ramp 60-40% HR

#### Complete examples (running)

**Aerobic workout with strides:**

    Warmup
    - 15m Z1 HR

    Main Set
    - 50m Z2 HR

    Strides 6x
    - 15s Z4 Pace
    - 1m45 Z1 HR

    Cooldown
    - 13m Z1 HR

**Distance intervals:**

    Warmup
    - 1mi Z1 HR

    Main Set 4x
    - 1mi Z4 Pace
    - 0.5mi Z1 Pace

    Cooldown
    - 1mi Z1 Pace

#### Distance steps (unit summary)

Supported: `km`, `mi`, `meters`, `mtr`, `yrd`, `y`
- `- 3km 80% Pace`
- `- 0.4km Z3 Pace` (400 meters)
- `- 10m 7:15-7:00/km Pace` (absolute pace with units `/km`, `/mi`)
- Absolute pace distance units: `/100m`, `/100y`, `/km`, `/mi`, `/500m`, `/400m`, `/250m`

### Weight training

**Event format in Intervals.icu:**
- In the calendar, specify a short string like: `* 1h 40-70% HR (72-125bpm)`
  - First part is the duration
  - Then the target HR% range
  - Approximate absolute bpm

**In Workout Builder:**
- Use one main line with duration and target: `* 1h 40-70% HR`
- Add sub-items with exercises and sets

**In `workout_doc`:**
- step: `duration: 3600` and `hr` with `units: "%hr"`, `start: 40`, `end: 70`
- This ensures Intervals correctly calculates moving_time and zone

**In the workout description:**
- Add structure (exercises, sets, rest)
- A note with absolute bpm for reference (e.g. 72-125 bpm)

## Integration with Intervals.icu via MCP

Intent-driven architecture: use 9 high-level tools (intents) instead of 100+ API endpoints.

### List of intent tools

| Intent | Purpose | Mutates? |
|--------|---------|----------|
| `plan_training` | Create plans (week — year) with periodization | ✅ |
| `analyze_training` | Analyze workout (single/period) | ❌ |
| `modify_training` | Modify/create/delete events | ✅ |
| `compare_periods` | Compare two periods | ❌ |
| `assess_recovery` | Assess recovery + ADE | ❌ |
| `manage_profile` | Profile, zones, thresholds | ✅ |
| `manage_gear` | Gear | ✅ |
| `analyze_race` | Post-race analysis | ❌ |
| `track_progress` | CTL plateau detection, TID drift, stagnation hypotheses | ❌ |

### Required context collection before planning

1. `manage_profile` action: "get", sections: ["overview", "zones", "thresholds", "metrics"] — profile, thresholds, CTL/ATL/TSB
2. `analyze_training` target_type: "period" for 4-8 weeks — volume, zones, and any returned trend/load fields; use intervals/streams when you need more detail
3. `assess_recovery` period_days: 14 — sleep, HRV, resting HR, ADE system state, recovery quality
4. `analyze_training` target_type: "period" for the planning period — existing calendar events (included in the response)
5. `track_progress` (optional) — if there is suspicion of stagnation or before transitioning between training periods

### Plateau detection and progress monitoring

Use `track_progress` when diagnosing the cause of stagnation:

- **Parameters:** `period_weeks` (4-24, default 12), `hypothesis_mode` (default true)
- **Returns:**
  - Plateau detection: start date, duration (days), slope/week, trend (Rising/Flat/Declining)
  - Load context: ACWR ratio + state (underloaded/productive/watch/overreaching), monotony, strain
  - HRV context: HRV ratio (1.0 = recovered), HRV trend state, lnRMSSD 7-day rollup (mean, CV, trend slope, sample count)
  - TID drift: weekly 3-zone samples, Shannon entropy recent 4w vs prior 4w, drift state (stable/converging/polarizing), dominant zone
  - Coaching hypotheses: volume (with flat CTL + ACWR productive/underloaded), intensity distribution (with flat CTL + high monotony + converging TID), recovery (with declining CTL + ACWR watch/overreaching + suppressed HRV)
  - Recommendations with confidence scores
- **Integration:** combine with `assess_recovery` to check: if CTL plateau + HRV suppressed = recovery microcycle

### Creating a plan

Use `plan_training`. Parameters: `period_start`, `period_end`, `focus` (aerobic_base/intensity/specific/taper/recovery), `max_hours_per_week`, `target_race`.

**Important:** `plan_training` creates calendar entries directly in the current MCP contract — do not assume a separate create-event API exists.

Exception: if the user says "text only" / "don't touch the calendar".

**Validation:** run the [Pre-submission validation checklist](#pre-submission-validation-checklist) on every event description in the plan before calling `plan_training`.

### Creating a single event

Use `modify_training` action: "create":
- `target_date` (YYYY-MM-DD)
- `new_name`, `new_description`, `new_duration` (HH:MM)
- `new_category` (usually "Workout"), `new_type` (e.g. "Run", "Ride", "WeightTraining")
- **MUST** run the [Pre-submission validation checklist](#pre-submission-validation-checklist) first
- First call **MUST** use `dry_run: true` — verify the response content matches intent

### Modifying existing events (Update Workflow)

**CRITICALLY IMPORTANT:**
- ✅ **USE** `modify_training` action: "modify" to change fields of an existing event
- ❌ **DO NOT** delete + recreate (delete+create is the wrong pattern)

**Workflow:**
1. Find the event via `analyze_training` target_type: "period" (calendar is included in the response)
2. Run the [Pre-submission validation checklist](#pre-submission-validation-checklist) on the new description
3. Call `modify_training` with action: "modify", `target_date`, `target_description_contains` (for searching), and fields to update (`new_name`, `new_description`, `new_date`, `new_duration`, `new_category`, `new_type`)
4. **MUST** use `dry_run: true` first — verify the response
5. Confirm the changes have been applied

### Deleting events

Use `modify_training` action: "delete". ONLY in exceptional cases. Always warn the user before deleting.

### Managing thresholds/zones (Sport Settings)

Use `manage_profile` action: "update_thresholds" with parameters:
- `new_aet_hr` — new AeT HR (bpm)
- `new_lt_hr` — new LT HR (bpm)
- `thresholds_source` — "manual" or "lab_test"
- `apply_to_activities` (default: true) — apply to historical activities

**IMPORTANT:** Execute ONLY in one of two cases:
1. **The user (athlete) explicitly asked** to update thresholds/zones
2. A new VO2max protocol was found in `knowledge/` and the user confirmed

**Never run an update without explicit user confirmation.**

#### Process:

1. **Preparation:** get current settings via `manage_profile` action: "get", sections: ["zones", "thresholds"] and make a diff of old/new values
2. **Request confirmation:** summary of changes (old→new values), source (test date)
3. **Update:** `manage_profile` action: "update_thresholds" with new values
4. **Verification:** check results via `manage_profile` action: "get", sections: ["metrics"]

### Unsupported older examples

Older Intervals API docs may mention folder-management helpers. They are not part of the current MCP contract in this workspace, so do not assume `createFolder`, `listFolders`, `updateFolder`, or `deleteFolder` exist unless the MCP schema explicitly exposes them.

### Recovery and training analysis

**Recovery assessment:** use `assess_recovery` with the period_days parameter (7–30 days).
- Returns: sleep, HRV, resting HR, HRV trend slope, recovery quality index, HRV suppression flag, ADE system state (LoadAccepting/RecoveryPriority), red flags, readiness for easy/intensity/long/race
- Sharp increase in stress / drop in sleep / rise in resting HR → reduce intensity
- HRV trend slope ↗ = improvement, ↘ = deterioration; recovery quality ≥0.8 = Good, ≥0.5 = Fair, <0.5 = Low

**Calendar view:** use `analyze_training` target_type: "period" for the desired range. The current contract already returns calendar events in that response, so there is no separate event-listing tool to call.

**Period analysis:** rely on the load/trend fields actually present in the response (time, distance, elevation, weekly avg, ACWR, monotony, strain, stress tolerance, NDLI, heat stress, load patterns, terrain specificity).

**Single-session detailed / intervals / streams:** use these when you need rep-by-rep HR/power/pace, cadence, decoupling, W′/pMax, histograms, best efforts, or stream-level inspection. Do not invent curve fields unless the response returns them.

### Key workout analysis

Use `analyze_training` target_type: "single":
- analysis_type: "detailed" — power-duration anchors (eFTP/W'/pMax when returned), WDRM (W' depletion with context), ISDM (signed decoupling negative=improving/positive=drifting), Z2 stability, cadence/terrain context, nutrition demand, and curve profile when available
- analysis_type: "intervals" — interval breakdown (HR/power/pace per rep)
- analysis_type: "streams" — stream insights (min/max/avg for HR/power/pace)

When suggesting adjustments (volume/intensity/recovery), rely on actual data from MCP:
- Zone distribution
- Duration and vertical
- Training stress (TSS)
- CTL/ATL/TSB

### Important rule for applying update/create

**Never apply modify_training/plan_training without explicit instruction from the user**

Examples of explicit instructions:
- "update my plan for next week"
- "move Tuesday to Wednesday"
- "create a plan and add it to the calendar"

If the user asks "show me the plan", "what should I do" — output only text in intervals.icu format, do not touch the calendar.

## Pre-submission validation checklist

**MANDATORY: STOP. Run this entire checklist before ANY `modify_training` or `plan_training` call.**

Do not proceed if any check fails. Fix the issue first, then re-check.

### Step 1: Dry-run first, always

```plaintext
[✅/❌] First call uses dry_run: true
[✅/❌] dry_run response shows the correct event content (at minimum name, date, duration; description may be abbreviated in the preview)
[✅/❌] User has explicitly approved the workout (not just "looks good" — explicit "apply it")
```

**Hard rule:** No `dry_run: false` on the first attempt. Ever.

### Step 2: Duration & distance syntax

| Check | Rule | Wrong | Correct |
|-------|------|-------|---------|
| `[✅/❌]` | Use `m`/`s`/`h` — NEVER `min`/`sec`/`minutes`/`seconds` | `10min 60%` | `10m 60%` |
| `[✅/❌]` | `m` = minutes, NOT meters | `400m Z2` | `400mtr Z2` or `0.4km Z2` |
| `[✅/❌]` | Combined: `1m30`, `5m30s`, `1h30m` — colon `:` is NOT used in steps | `1:30` | `1h30m` |
| `[✅/❌]` | Distance valid units: `km`, `mi`, `mtr`, `yrd`, `y` | `500m` | `500mtr` or `0.5km` |

### Step 3: Target syntax

| Check | Rule | Wrong | Correct |
|-------|------|-------|---------|
| `[✅/❌]` | Power `%` always present: `95-105%` not `95-105` | `- 5m 95-105` | `- 5m 95-105%` |
| `[✅/❌]` | Running zones ALWAYS have `HR` or `Pace` suffix | `Z2` in running | `Z2 HR` or `Z2 Pace` |
| `[✅/❌]` | HR uses `% HR` or `% LTHR` or `Z2 HR` | `- 20m 70%` (unclear type) | `- 20m 70% HR` |
| `[✅/❌]` | Pace uses `% Pace`, `Z2 Pace`, or `mm:ss/unit Pace` | `- 3km Z3` | `- 3km Z3 Pace` |
| `[✅/❌]` | Cadence format: `NNrpm` (cycling only) | `90 rpm` (space) | `90rpm` |
| `[✅/❌]` | Range separator is hyphen `-`, not `–` or `to` | `95–105%` | `95-105%` |

### Step 4: Structure

| Check | Rule | Wrong | Correct |
|-------|------|-------|---------|
| `[✅/❌]` | Steps start with `- ` (dash + space) | `10m 60%` | `- 10m 60%` |
| `[✅/❌]` | Section headers have NO `-` prefix | `- Warmup` | `Warmup` |
| `[✅/❌]` | Blank line before and after each repeat block | see below | see below |
| `[✅/❌]` | No nested repeats | `Main Set 3x 5x` | unwrap or flatten |
| `[✅/❌]` | Ramp uses lowercase `ramp` | `Ramp 50-75%` | `- 10m ramp 50-75%` |
| `[✅/❌]` | `ramp` boundaries use `%`: `ramp 50%-75%` not `ramp 50-75` | `ramp 50-75` | `ramp 50%-75%` |

**Correct repeat structure:**

```text
Warmup
- 10m 60%
xw
                         ← blank line
Main Set 5x              ← or standalone `5x`
                         ← blank line
- 3m 120%
- 2m Z1

                         ← blank line
Cooldown
- 10m 50%
```

### Step 5: Special steps

| Check | Rule | Wrong | Correct |
|-------|------|-------|---------|
| `[✅/❌]` | Timed prompts use `<!>` separator | `- 30^ cue <!> 1m30` | `- Start 30^ Cue <!> 1m30 120%` |
| `[✅/❌]` | Freeride = `freeride` keyword | `- 20m free ride` | `- 20m freeride` |
| `[✅/❌]` | Press lap = `Press lap` at end | `- 1h Z2 press lap` (lowercase) | `- 1h Z2 Press lap` |

### Step 6: Event-level rules

| Check | Rule | Wrong | Correct |
|-------|------|-------|---------|
| `[✅/❌]` | `new_name` = short readable name (NOT builder syntax) | `- 10m 60%` | `Endurance Run` |
| `[✅/❌]` | Workout syntax goes in `new_description`, not `new_name` | name contains `- 10m` | name is readable |
| `[✅/❌]` | No Markdown headers (`##`, `**`) inside `new_description` | `## Warmup` | `Warmup` |
| `[✅/❌]` | Duration arithmetic adds up | day says `1:30` but steps sum to 1:15 | steps sum to total |
| `[✅/❌]` | `target_description_contains` uses a real text fragment from the existing description | searching by `10m` (generic) | searching by `Endurance` or `Recovery` |
| `[✅/❌]` | Event duration in dry_run response matches expected `new_duration` | dry_run returns `1:15` but expected `1:45` | match or fix description |

### Step 7: Output the pre-flight summary

Before the real call, output:

```text
PRE-FLIGHT CHECK:
duration_syntax:   ✅/❌  (list any failures)
target_syntax:     ✅/❌  (list any failures)
structure:         ✅/❌  (list any failures)
special_steps:     ✅/❌  (list any failures)
event_level:       ✅/❌  (list any failures)
event_duration:    ✅/❌  (dry_run duration matches new_duration)
dry_run_result:    ✅/❌  (dry_run response matches intent)

=> STOP if any ❌. Fix and re-check.
```

### Duration parsing trap

**Known issue:** Intervals.icu may recalculate event duration from structured workout steps (`Warmup / Main Set / Cooldown`) independently of the `new_duration` field. The calendar may show a different duration than what you set in `new_duration`.

**Symptoms:**
- `new_duration: "1:45"` but dry_run response shows 1:15 or some other value
- This happens because the parser sums step durations and ignores `new_duration` field

**Workarounds (pick one):**
1. **Preferred:** Use a flat single step: `- 1h45m Z2 HR` when exact duration is critical
2. **Alternative:** Verify `new_duration` in the dry_run response — if it differs from expected, adjust the description (not the duration field)

### Failure recovery

If a check fails:
1. Fix the identified issue
2. Re-run all checks (not just the failed one — fixes can introduce new issues)
3. Re-run `dry_run: true`
4. Only then proceed to `dry_run: false`

## Intervals.icu Documentation

Official documentation on workout building is available at:
- https://forum.intervals.icu/t/workout-builder/1163
- https://forum.intervals.icu/t/computed-activity-fields/25673
- https://forum.intervals.icu/t/custom-interval-fields/25942
- https://forum.intervals.icu/t/custom-activity-charts/28627
- https://intervals.icu/api-docs.html

## When to refer to other skills

- **Periodization, zones, strength training**: see `periodization-coach` skill
- **Recovery monitoring, red flags, VO2max work**: see `athlete-monitoring` skill
- **Injury prevention, evidence-based practice**: see `kinesiology-foundations` skill
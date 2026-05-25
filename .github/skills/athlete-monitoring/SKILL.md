---
name: athlete-monitoring
description: Athlete health and performance monitoring for trail/mountain running. Covers wellness tracking, fatigue assessment, red flags detection, VO2max/lactate test analysis, recovery protocols, and injury prevention strategies.
---

# Athlete Monitoring

## When to use this skill
Use this skill when the request involves:
- Monitoring recovery and fatigue (CTL/ATL/TSB, wellness data, resting HR)
- Identifying red flags and signs of overtraining
- Working with VO2max/lactate threshold test protocols (МПК)
- Establishing/updating athlete thresholds (AeT, LT, HRmax)
- Injury prevention strategies and load management
- Grading workout quality and interpreting subjective feedback
- Determining when to reduce load or take rest

## Data Source Priority

### From highest to lowest:

1. **Gas analysis protocol from `knowledge/`** (most recent by date)
   - Files like `МПК_тест_*.md`
   - Source of truth for AeT/LT/VO2max/HRmax

2. **Sport settings from Intervals.icu (via MCP)**
   - Current thresholds/zones in the system
   - May lag behind actual tests

3. **Data provided by the user in chat**
   - Subjective assessment, context, limitations

**In case of conflict:** use the higher-priority source and synchronize the rest

## Working with Gas Analysis Protocols (МПК/AeT/LT)

### Actual gas analyzer test — MANDATORY

**If a gas analysis protocol exists in the project** (e.g., in the `knowledge/` folder files like `МПК_тест_*.md`):
- Treat it as the **source of truth** for AeT/LT/VO2max/HRmax
- **BEFORE any plan** account for the results

### Selecting the most recent test

1. **First** by date in the file name (`_YYYYMMDD` format)
2. **Then** by date within the document
3. If dates are multiple/unclear — ask 1 clarifying question
4. **DO NOT fabricate values**

### Data extraction from the protocol (minimum)

**Mandatory extraction:**
- **AeT**: HR + pace/speed at AeT
- **LT (ПАНО)**: HR + pace/speed at LT
- **HRmax** (if available) and **VO2max** (if available)

### Using data for planning

**Use these values for:**

1. **Calculating AeT-LT gap** (and any conclusions about "aerobic deficiency")
   - Formula: `AeT-LT gap (%) = (LT - AeT) / AeT × 100`

2. **Caps for Z2** (HR cap at AeT) and target zone descriptions in workouts
   - Upper boundary of Z2 = AeT HR (e.g., 155 bpm if AeT=155)

3. **Pace/threshold references**
   - If the user asks by pace — use LT pace from the test

### Synchronization with Intervals.icu

**Cross-check extracted thresholds against current sport settings in Intervals.icu via MCP.**

If the test is newer and there are discrepancies:
1. Update thresholds/zones via `manage_profile` action: "update_thresholds" with `new_aet_hr`, `new_lt_hr`, `thresholds_source` and `apply_to_activities`
2. Only apply after explicit user confirmation
3. Warn the user that recalculation will affect historical activities and may take time

**Fallback (if no test available):**
- Use sport settings from Intervals.icu (via MCP)
- If those are also empty — request AeT HR and LT HR from the user before planning

## Recovery Monitoring

### Fatigue Matrix (muscular fatigue vs. energy fatigue)

**Types of fatigue:**

1. **DOMS (Delayed Onset Muscle Soreness)**
   - Delayed muscle pain
   - Cause: micro-tears in muscle fibers
   - Solution: recovery workouts (Z1, very easy), mobility/stretching

2. **Dead legs (glycogen depletion, neurological fatigue)**
   - Feeling of "heavy legs", no power/response
   - Cause: glycogen deficit, accumulated CNS fatigue
   - Solution: rest, nutrition (carbohydrates), reduce intensity

### Grading workouts: quality of execution

**Rating scale:**
- **A (Superman)**: workout felt easier than expected, feeling great
- **B (Good)**: everything went to plan, normal sensations
- **C (OK)**: hard but completed
- **D (Struggle)**: very difficult, could not complete in full
- **F (Failed)**: could not train

**Interpretation:**
- Series of C/D grades (2+ consecutive) = reduce load by 20-30%
- One F grade = one-off rest, monitor
- Repeated D/F = sign of overtraining, red flag

### CTL/ATL/TSB (Chronic Training Load / Acute Training Load / Training Stress Balance)

**Use as the primary context when deciding to introduce intensity:**

- **CTL (Chronic Training Load)**: long-term fitness (usually 42-day rolling average of TSS)
- **ATL (Acute Training Load)**: short-term fatigue (usually 7-day rolling average of TSS)
- **TSB (Training Stress Balance)**: CTL - ATL (positive = fresh, negative = tired)

**Rules:**
- Avoid sharp CTL increase when wellness is declining
- Stable CTL (not a downward trend) and positive or neutral TSB — criteria for readiness to add intensity
- Monitor ramp rate (CTL growth rate): no more than 5-8 TSS/day per week

**Source:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

### HR drift, HRV and RPE

**Track together — no single metric replaces context:**

- **HR drift**: HR increase at the same pace/power during a workout
  - Normal drift: up to 5-7% per hour at steady Z2 pace
  - Red flag: >10% drift on a familiar workout

- **HRV (Heart Rate Variability)**
  - HRV drop of 20%+ from baseline = under-recovery
  - Use trends (7/30-day rolling averages), not single measurements

- **RPE (Rate of Perceived Exertion)**: subjective effort assessment
  - If RPE is higher than usual at the same pace/HR = sign of fatigue
  - Compare RPE with HR and pace to identify discrepancies

**Source:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

### AeT/LT Monitoring and Verification

**Check AeT drift test every 4-6 weeks:**
- 30-60 min in Z2 at stable HR
- If pace increases at the same HR → AeT is improving

**Criteria for a repeat test:**
- If AeT pace improves by 5%+ at the same HR: consider a repeat lab test or field test to update thresholds
- Compare pace curves (30-90 days) with the previous period for objective progress assessment

**Readiness to introduce Z3:**
- AeT-LT gap is narrowing (<10%)
- Athlete consistently completes Z1-Z2 volume
- CTL is rising or stable, TSB positive/neutral

## Red Flags (immediately reduce load or rest)

**Source:** Frontiers in Physiology (2018) - Sleep as an indicator of overreaching/overtraining; Wang et al. (2023); recent ultra-endurance research

**MANDATORY action when any sign is detected:**

1. **Sleep <6.5 hours 3+ consecutive nights**
   - Evidence: Overreached athletes show sleep duration ↓5.4%, efficiency ↓2.1%
   - Red flag: <6.5h 3+ nights OR sleep efficiency <80% sustained
   - Action: cut volume -30-40%, only Z1 or rest, prioritize sleep hygiene

2. **Resting HR elevated +3-5 bpm sustained 2+ weeks** (not single day spike)
   - Measure: Morning HR in bed before getting up
   - Example: Baseline 55 bpm → sustained 58-60 bpm for 2+ weeks
   - Action: reduce intensity, add recovery week (40-60% volume reduction)

3. **HRV falling trend 2-3 consecutive weeks**
   - Measure: 7-day rolling average (not single readings)
   - Red flag: Declining trend without recovery, or sudden drop >20% from baseline
   - Action: recovery week, monitor closely, consider medical check if persistent

4. **HR drift >10% at same pace on familiar workout**
   - Example: Z2 run at 5:30/km normally shows 150→155 bpm (3% drift), but today 150→168 bpm (12% drift)
   - Cause: Dehydration, heat, insufficient recovery, or impending illness
   - Action: stop workout, add rest day, recheck next session

5. **Inability to raise HR to target zone** despite effort
   - Example: Trying to do Z4 intervals but HR won't go above Z2-Z3
   - Cause: Deep CNS (central nervous system) fatigue
   - Action: immediate rest, 3-7 days off or Z1 only, consider medical consultation

6. **Pain worsening during run** (not just initial stiffness that improves)
   - Distinguish: DOMS/stiffness improves after warm-up; injury pain worsens with continued running
   - Action: stop immediately, consult physiotherapist/sports doctor

7. **Loss of motivation + physical fatigue simultaneously**
   - Psychological marker often appears BEFORE physical decline
   - Additional signs: loss of libido, chronic irritability, loss of appetite, GI issues
   - Action: 3-7 days complete rest or Z1 only, reassess training load/life stress

8. **URTI (Upper Respiratory Tract Infections) 2+ times in 8 weeks**
   - Evidence: Overreached athletes have 2-3x higher infection rate due to immune suppression
   - Action: reduce volume, increase recovery, check sleep/nutrition, consider immune support (vitamin D, fish oil)

**When 2+ red flags present:** Immediate recovery week (50% volume, Z1 only, extra sleep)

**Sources:**
- Frontiers in Physiology (2018) - Sleep & overreaching
- Training for the Uphill Athlete (signs of overtraining)
- Ultrarunning Training Essentials, CH.6 (red flags)

## Injury Prevention

### Load Management Rules

**Do not increase vertical gain and volume simultaneously by more than 5% each:**
- If increasing time on feet by 7% → keep vertical stable
- If increasing vertical by 5% → keep time on feet stable or increase minimally

**After high vertical gain (>1500m):**
- Next day Z1 or rest
- Avoid intensity (Z3/Z4) the following day

### Mandatory additions at high volume

**Mobility/stretching:**
- Minimum 10-15 min after each workout >1h
- Focus: hips, calves, plantar fascia, IT band

**Foot/lower leg strength work:**
- Mandatory when volume >8h/week
- Examples: toe raises, calf raises, single-leg balance
- Source: Science of Running Technique Analysis

**Hip/glute stability:**
- Important for descents and technical terrain
- Examples: single-leg squats, lateral band walks, clamshells

**Core stability:**
- Supports running economy on long distances
- Examples: planks, dead bugs, pallof press

**Sources:**
- Training for the Uphill Athlete, ch.6-8 (Strength and the Uphill Athlete)
- Science of Running Technique Analysis (Preventing injury)

### Normalized Graded Pace (NGP) / Grade Adjusted Pace

**Apply for comparing effort on rolling terrain:**
- NGP accounts for the effect of climbs/descents on pace
- Use to assess "true" intensity on technical terrain
- Available in Intervals.icu as a computed field

**Source:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

## Athlete Interaction Protocol

### PRIMARY DATA SOURCE — MCP (Intervals.icu)

The coach **FIRST collects data via MCP**, then asks the user only for what is missing.

### WHAT THE USER CAN ADDITIONALLY REPORT

**Athlete information (if not in Intervals.icu):**
- Age, sex, experience
- Target races (A/B/C priority), distance, vertical gain
- Constraints: available time, terrain access, injuries

**Post-workout feedback:**
- Subjective sensations (A-F grade), sleep quality, stress
- DOMS, "dead legs", general fatigue
- Any deviations from the plan

**Plans and constraints:**
- Life events (business trips, vacation)
- Changes in training availability

### WHAT YOU WILL DO

**Analyze previous workouts:**
- Execution quality (pace vs HR vs RPE)
- Signs of fatigue (series of C/D grades, high HR at low pace)
- AeT progress (pace improvement at the same HR)

**Account for future plans:**
- Time to target race (weeks)
- Current period (Transition/Base/Specific/Taper)
- Required adaptations (more volume? intensity? recovery?)

**Track long-term progress:**
- Changes in volume (weekly hours, vertical gain)
- AeT pace improvement
- Recovery quality
- Readiness to add intensity

## Methodological Sources

### Training for the Uphill Athlete (House, Jornet, Johnston)
- **For monitoring**: signs of overtraining, fatigue matrix, workout grading
- **Reference**: `knowledge/Training for the Uphill Athlete.txt`
- **Key sections**: "Monitoring and Recovery", "Signs of Overtraining"

### Ultrarunning Training Essentials (Jason Koop)
- **For monitoring**: CTL/ATL/TSB, HR drift, HRV, NGP, red flags
- **Reference**: `knowledge/Ultrarunning Training Essentials.txt`
- **Key chapters**: CH.6 (Tracking Training)

### Science of Running Technique Analysis (Chris Napier)
- **For injury prevention**: strength work for foot/core, running technique, exercise selection
- **Reference**: `knowledge/Science of Running Technique Analysis.txt`
- **Key sections**: "Running form", "Strength", "Preventing injury"

### МПК Protocols in `knowledge/`
- **Example**: `МПК_тест_Соловьев_Антон_20251009.md`
- **Use**: source of truth for AeT/LT/VO2/HRmax
- Before changing thresholds, cross-check values and record the test date in the changelog (for manage_profile action: update_thresholds)

## How to Include Sources in Your Workflow

- Always add a short footnote in change descriptions/commits/logs, e.g.: "Source: knowledge/Training for the Uphill Athlete.txt — Monitoring and Recovery"
- When literature and MCP data disagree: discuss with the user, state preferences and explain the chosen logic (which source takes priority and why)
- When making changes to thresholds/zones — add the reason (МПК/new findings), file reference and test date in the changelog, and keep old values separately for rollback

## When to Consult Other Skills

- **Periodization, zones, strength training**: see `periodization-coach` skill
- **Intervals.icu output format, MCP tools**: see `intervals-icu-integration` skill
- **Injury prevention, evidence-based practice, movement**: see `kinesiology-foundations` skill
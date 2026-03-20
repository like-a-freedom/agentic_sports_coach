---
name: bloodwork-analysis
description: Analyze athlete blood tests (endurance/trail focus) with trends + red flags and produce actionable recommendations for nutrition/recovery and training-plan adjustments (volume/intensity/strength), staying within scope-of-practice.
---

# Bloodwork Analysis (for endurance / trail / mountain running coaching)

## When to use this skill
Use this skill when the user provides:
- Blood test results (PDF/images/text) and asks for interpretation.
- Questions about fatigue, recovery, low energy, recurrent illness, poor adaptation, or unexplained performance drop where labs may matter.
- Requests to adjust a training plan based on bloodwork.

## Safety + scope (non-negotiable)
- Do not diagnose disease or prescribe medical treatment.
- Do not advise stopping/starting prescription meds.
- If results show urgent red flags or the athlete has concerning symptoms, recommend contacting a clinician (sports-medicine MD / GP / endocrinologist / hematologist as appropriate).
- Use a coaching lens: performance + training tolerance + recovery + risk management.

## Inputs required (always ask/confirm)
For each lab set, try to capture:
- Date + lab name/location
- Fasting/non-fasting status; time-of-day (especially for hormones)
- Current training load (last 7–14 days), recent illness, altitude, travel, major stress, menstrual status (if applicable)
- Supplements/meds that affect labs (iron, thyroid meds, creatine, biotin, vitamin D, etc.)
- Units + reference ranges from that lab (do not assume universal ranges)

## If results are in PDFs
Preferred workflow:
1) Convert to text/markdown.
   - In this repo you can keep originals in `knowledge/blood_tests/`.
   - Extract a plain-text copy if needed for analysis.
2) Normalize into a compact “structured summary” (date-stamped) so trend analysis is easy.

Suggested naming for summaries:
- `knowledge/blood_tests/YYYY-MM-DD_summary.md`

## Interpretation workflow (repeatable)
### Step 1: Identify the question
Pick one primary coaching question:
- Low energy / fatigue
- Poor recovery / heavy legs
- Recurrent illness
- Weight change / RED-S risk
- Altitude adaptation / hemoglobin
- Heat issues / hydration

### Step 2: Triage for red flags
Examples (not exhaustive):
- Very low hemoglobin / hematocrit, severe leukopenia, extremely high CRP, very abnormal electrolytes, marked kidney/liver dysfunction markers.
Action: recommend clinician; reduce training intensity and volume until clarified.

### Step 3: Endurance athlete “usual suspects”
Prioritize these domains and relate them to training tolerance.

#### A) Iron status / oxygen carrying capacity
Look at: hemoglobin, hematocrit, RBC indices (MCV/MCH), ferritin, serum iron, transferrin/TIBC, transferrin saturation.
Coaching relevance:
- Low ferritin / iron deficiency can reduce training tolerance and economy.
Actionable coaching:
- Emphasize dietary iron + timing (heme iron, vitamin C pairing; avoid coffee/tea around iron-rich meals).
- If iron deficiency is suspected, recommend discussing supplementation and follow-up testing with a clinician.
Training adjustments (until improving):
- Keep most training Z1–Z2; reduce/avoid adding Z3–Z4 blocks; preserve consistency.

#### B) Energy availability / endocrine stress (RED-S risk screen)
Look at: thyroid panel (TSH, free T4/T3), sex hormones where available, fasting glucose, HbA1c, lipids, vitamin D.
Also integrate symptoms: low libido, menstrual irregularity, persistent fatigue, mood issues, frequent injuries.
Actionable coaching:
- Focus on fueling adequacy (carbs around sessions, protein distribution, total energy).
- Consider reducing intensity density and adding rest if under-recovered.
- If RED-S signs are present, recommend clinician and/or sports dietitian.

#### C) Inflammation / illness
Look at: CRP, WBC differential.
Actionable coaching:
- If inflammatory markers are elevated with symptoms: de-load, sleep, and medical check if persistent.
- Avoid hard workouts during acute infection risk.

#### D) Muscle damage / training stress
Look at: CK (if present), AST/ALT context (can rise with heavy training), resting HR trend, subjective fatigue.
Actionable coaching:
- If markers suggest excessive stress: reduce eccentric load (downhill, plyos), shorten long runs, shift to easy aerobic.

#### E) Hydration / electrolytes
Look at: sodium, potassium, chloride, urea/creatinine, albumin.
Actionable coaching:
- If dehydration markers are likely (context-dependent): adjust hydration and sodium strategy; avoid key workouts in heat until stable.

#### F) Vitamins and micronutrients
Look at: vitamin D (25(OH)D), B12, folate.
Actionable coaching:
- Correct obvious dietary gaps; recommend clinician/dietitian for supplementation strategy if deficient.

## Producing “actionable recommendations” (required output structure)
When responding to the athlete, structure the answer as:
1) Summary in 3–5 bullets (what matters most for performance/training)
2) Flags
   - Red flags (need clinician)
   - Yellow flags (watch + retest)
3) Likely impact on training (what adaptations are currently limited)
4) Actions (next 2 weeks)
   - Nutrition/recovery actions
   - Training modifications (volume/intensity/strength)
5) Follow-up plan
   - What to retest and when (often 6–12 weeks depending on marker)

## Training plan adjustment rules (coach-level)
Use the smallest effective adjustment.
- If red flags: prioritize health → reduce volume 30–50%, remove intensity, keep short easy sessions if symptom-free.
- If iron deficiency suspected: keep training mostly aerobic (Z1–Z2), avoid adding new intensity blocks; focus on consistency + fueling.
- If inflammation/illness suspected: de-load immediately, resume gradually once symptom-free.
- If endocrine/energy availability concerns: reduce intensity density, ensure fueling, consider extra rest day; avoid aggressive weight-loss goals.

If the user explicitly asks for a revised week plan, follow the project output contract:
- Provide the plan in intervals.icu text format.
- Put the “why” inside workout descriptions using `-` lines.

## Repo references
- Blood test originals: `knowledge/blood_tests/`
- Evidence-based reasoning boundaries: `.github/skills/kinesiology-foundations/SKILL.md`

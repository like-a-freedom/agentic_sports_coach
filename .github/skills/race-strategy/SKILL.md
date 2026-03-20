---
name: race-strategy
description: Race day tactics and strategy for trail/ultra running. Covers pacing protocols, aid station efficiency, mental strategies for managing dark moments, gear setup, drop bag planning, and contingency protocols for common race-day issues.
---

# Race Strategy

## When to use this skill
Use this skill when the request involves:
- Planning race-day pacing strategy for upcoming events
- Aid station tactics and efficiency protocols
- Mental strategies for managing ultra-distance challenges
- Gear setup and drop bag planning for races
- Contingency planning (blisters, GI issues, getting lost, DNF decisions)
- Pre-race logistics (race week protocol, course reconnaissance)
- Post-race debriefing and tactical analysis

## Core principle: "Train hard, race smart"

**Philosophy:** Tactical errors cost more races than fitness limitations. A well-prepared athlete with poor tactics will underperform; an adequately prepared athlete with excellent tactics often exceeds expectations.

**Evidence base:** Ultrarunning Training Essentials (Koop, 2nd Ed.), Training for Uphill Athlete (House, Jornet, Johnston)

## MCP post-race debrief workflow

Когда запрос касается разбора уже завершённой гонки, начинай с `mcp_rusty-interva_analyze_race`.

**Практические правила работы с контрактом:**
- Основной селектор — `description_contains` (название старта, `50K`, `marathon`, и т.п.)
- После вызова всегда сверяй, что найденная гонка действительно нужная: `name`, дата, ID
- Поля `date`, `analysis_type`, `compare_to_planned` есть в схеме, но не должны считаться достаточными сами по себе для выбора нужной гонки или гарантированного отдельного режима анализа
- Обязательно учитывай блок `Data Availability`: если нет streams/intervals/wellness, снижай уверенность выводов о pacing, decoupling, post-race fatigue

**Что читать в ответе в первую очередь:**
1. Таблицу результатов — базовый outcome
2. `Execution Pattern` — сегменты, Efficiency Factor, Aerobic Decoupling
3. `Post-Race Load Context` — нужен ли recovery block
4. `suggestions` / `next_actions` — особенно связку с `assess_recovery`

**Рекомендуемое продолжение:**
- Сразу после debrief: `mcp_rusty-interva_assess_recovery` на 7–14 дней
- Если нужен следующий блок: `mcp_rusty-interva_plan_training`
- Если нужен особенно глубокий разбор конкретных отрезков/потоков, дополни debrief анализом одной сессии через `mcp_rusty-interva_analyze_training`

## Pacing Protocols

### Fundamental pacing strategies

**Even pacing (recommended for most ultra runners):**
- Maintain consistent effort (not speed!) throughout race
- Measured by HR or RPE, not pace/speed
- Best for: First-time distance attempts, conservative goal setting
- Evidence: Minimizes glycogen depletion, reduces risk of bonking

**Positive split (start faster, slow down):**
- Banking time early when fresh
- Risk: Depleting glycogen too early, "blowing up"
- Use cases: Very experienced, known course, aggressive time goals
- Warning: 80% of ultra runners who positive split regret it at mile 60+

**Negative split (start slower, speed up):**
- Conservative early, aggressive late
- Lower risk of bonking
- Best for: New to distance, challenging course, hot conditions
- Challenge: Requires discipline to hold back when feeling good

### HR-based vs RPE-based pacing

**Heart Rate (HR) based:**
- **Early race (first 25-30%):** Stay below AeT (e.g., if AeT=160 bpm, cap at 155-158 bpm)
- **Mid race (30-70%):** Allow drift to AeT+5 bpm on climbs, return to AeT on flats/descents
- **Late race (70%+):** HR often decouples; switch to RPE
- **Advantages:** Objective, prevents over-exertion early
- **Limitations:** Heat/altitude/fatigue cause HR drift; less reliable late race

**Rate of Perceived Exertion (RPE) based:**
- **Early race:** RPE 3-4/10 ("could talk in full sentences easily")
- **Mid race:** RPE 4-6/10 ("conversational but requires focus")
- **Late race:** RPE 6-8/10 ("can speak 3-5 words at a time")
- **Advantages:** Works throughout race, accounts for fatigue/conditions
- **Limitations:** Requires experience to calibrate

**Hybrid approach (recommended):**
- Early race: Use HR to prevent overexertion
- Late race: Switch to RPE when HR decouples
- Crosscheck: If RPE feels much higher than HR suggests, slow down (fatigue/heat/altitude)

### Terrain-based pacing adjustments

**Uphills:**
- **Steep (>10% grade):** Power hike if hiking pace = running pace
- **Moderate (5-10% grade):** Run if you can maintain conversational breathing
- **Gentle (<5% grade):** Run, but don't force pace
- **Rule:** If breathing becomes labored on uphill, slow down or hike
- **Poles:** Use on sustained climbs >5% grade to reduce quad load

**Downhills:**
- **Steep (>10% grade):** Control speed, use short steps, land midfoot
- **Moderate (5-10% grade):** Let gravity assist, but don't overstride
- **Gentle (<5% grade):** Run naturally, slight increase in pace acceptable
- **Warning:** Quad damage accumulates; downhill speed in first 30% of race = quad death in last 30%
- **Eccentric loading:** If quads burn on downhill, slow immediately (damage is progressive)

**Flats:**
- Opportunity to "bank time" without excessive cost
- Keep effort steady (RPE 4-6/10)
- Focus on efficiency: cadence 160-180 spm, relaxed shoulders

### Distance-specific pacing

**50K (3-6 hours):**
- First 25K: Conservative (HR <AeT, RPE 3-4/10)
- Middle 15K: Steady (HR ~AeT, RPE 4-6/10)
- Final 10K: Push if feeling good (RPE 6-8/10)
- Aid station time: 1-2 min max per stop

**100K (8-14 hours):**
- First 40K: Very conservative (HR <AeT, RPE 3-4/10)
- Middle 40K: Steady (HR ~AeT, RPE 4-6/10), allow HR drift on climbs
- Final 20K: Survival mode (RPE varies, focus on forward progress)
- Aid station time: 2-5 min per stop, longer if needed for recovery

**100 miles (20-30 hours):**
- First 50 miles: Extremely conservative (HR well below AeT, RPE 2-4/10), "this feels too easy"
- Mile 50-80: Steady grind (HR often decoupled, RPE 5-7/10), manage fatigue
- Mile 80-100: Mental game (RPE 7-9/10), celebrate each mile
- Aid station time: 5-10 min acceptable if needed, efficiency still matters
- **Night sections:** Slow down 10-15% (reduced visibility, fatigue)

## Aid Station Strategy

### Time management protocol

**Efficiency rule:** Every minute at aid station = ~3 minutes of moving time to make up
- Example: 5-min stop vs 2-min stop = 9 minutes lost on course

**Target times by distance:**
- **50K:** 1-2 min per stop (quick fuel, water, go)
- **100K:** 2-5 min per stop (fuel, check feet if needed, recover slightly)
- **100 miles:** 3-10 min per stop (fuel, recover, address issues), longer at crew-accessible points

**In/out protocol:**
1. **Approach** (100m before): Decide what you need (water? food? gear change?)
2. **Arrival:** State needs clearly to volunteers ("Fill both bottles, 3 gels, pretzels")
3. **Execute:** While bottles fill, grab food, eat immediately
4. **Depart:** Check you have everything, thank volunteers, GO

### What to do at each aid station

**Standard stops (no crew):**
- Refill hydration (both bottles/bladder)
- Grab quick CHO (gels, bars, real food if tolerated)
- Eat/drink WHILE at station (don't leave with food in hand to "eat later")
- Quick body scan: feet ok? Chafing? Blisters forming?

**Crew-accessible stops:**
- All of above PLUS:
- Change socks if wet/hot spots detected
- Apply lube to chafing areas
- Change layers if weather changed
- Restock personal nutrition from drop bag
- **Crew protocol:** Crew should have gear ready, minimize talking, 5-min max

**Major stops (e.g., halfway, mile 60):**
- More time acceptable (5-10 min)
- Sit if needed (but set timer!)
- Address issues: blisters, tape hot spots, change clothes
- Eat real food if appetite allows (soup, potatoes, broth)
- Mental reset: "I'm starting a new race from here"

### Drop bag planning

**What to include in drop bags:**
- **Nutrition:** Extra gels/bars you like (aid station may run out)
- **Hydration:** Electrolyte tablets, spare bottle if worried about loss
- **Clothing:** Extra layers (jacket, gloves, hat) + rain gear
- **Foot care:** Spare socks, blister kit (tape, needle, lube)
- **Lighting:** Spare batteries/headlamp, vest light
- **Emergency:** Ibuprofen (use sparingly!), duct tape, safety pins
- **Mental:** Motivational note to yourself, photo of family/goal

**Drop bag locations (prioritize):**
- Halfway point (critical for gear swap, nutrition restock)
- Before night section (lighting check, warm clothes)
- Before final push (mental reset items, fresh socks)

**Label clearly:** "Your Name - Drop Bag Mile X - [Race Name] 2026"

## Mental Strategies

### Race segmentation

**Break race into manageable chunks:**
- **50K:** Think in 10K segments (5 segments)
- **100K:** Think in 20K segments (5 segments) or aid-station-to-aid-station
- **100 miles:** Think in 25-mile segments (quarters) or "sunrise to sunset"

**Psychological trick:** "I'm just running to the next aid station" (not "I have 60 miles to go")

### Managing dark moments

**Expect dark moments at predictable times:**
- **Mile 30-40 (50K):** Post-glycogen-depletion dip, often around 4-5h mark
- **Mile 50-65 (100K/100mi):** Physical fatigue accumulating, night approaching
- **Mile 80-90 (100mi):** Sleep deprivation, accumulated damage, "why am I doing this?"

**Dark moment protocol:**
1. **Acknowledge:** "This is normal, expected, temporary"
2. **Physical check:** Eat, drink, check pace (am I going too hard?)
3. **Mental reset:** Break race into smaller segment ("just get to next aid station")
4. **Positive self-talk:** "I've trained for this, I can handle this"
5. **Time limit:** Give it 20-30 minutes; if still dark, reassess

**When dark moment lifts (it usually does):** Celebrate mentally, but don't surge pace

### Positive self-talk protocols

**Mantras to use:**
- "Relentless forward progress" (common ultra mantra)
- "One step at a time" (when feeling overwhelmed)
- "I've done harder things" (reference training or life challenges)
- "This is temporary" (pain is transient)
- Custom mantra (choose in training, use in race)

**Avoid negative self-talk:**
- ❌ "I'm not good enough" → ✅ "I'm doing my best"
- ❌ "I can't do this" → ✅ "I'm still moving forward"
- ❌ "This is too hard" → ✅ "This is supposed to be hard"

### DNF (Did Not Finish) decision framework

**When to DNF (prioritize safety):**
1. **Medical emergency:** Severe dehydration, hyponatremia, heat stroke, injury requiring medical attention
2. **Time cutoffs:** If mathematically impossible to finish within cutoff, save yourself for another day
3. **Severe injury:** Continuing would cause long-term damage (stress fracture, severe sprain)

**When NOT to DNF (push through):**
- General discomfort (this is ultra running!)
- Dark moment (these pass)
- Blisters (manageable with tape)
- Fatigue (if you can still move forward, keep going)
- "I'm slower than I wanted" (adjust goals, finish the race)

**Decision protocol:**
1. Sit at aid station, eat/drink, address immediate issue
2. Give yourself 10-20 minutes to reassess
3. Ask: "Is this dangerous to my long-term health?" If yes → DNF. If no → continue.
4. If DNF: Accept decision, don't dwell, learn for next time

## Gear Setup

### Mandatory gear (check race requirements)

**Typical mandatory items:**
- Hydration (minimum capacity specified, e.g., 1L)
- Nutrition (minimum calories, e.g., 500 cal)
- Lighting (headlamp + backup, specific lumens)
- Emergency (whistle, space blanket, first aid)
- Navigation (map, GPS, phone)
- Clothing (jacket, extra layer)

**Verify before race:** Read race rules, check gear list multiple times

### Optional gear (personal preference)

**Poles (trekking poles):**
- **Use if:** Course has >1500m vertical per 50K, steep sustained climbs
- **Don't use if:** Mostly runnable, flats, or you haven't trained with them
- **Technique:** Use on uphills (push off to assist legs), collapse/carry on flats/downhills

**Hydration system:**
- **Vest with soft flasks:** Most popular, easy access, can see fluid level
- **Hydration bladder:** Hands-free, but harder to monitor intake, cleaning tedious
- **Handheld bottles:** Simple, but annoying on technical terrain
- **Recommendation:** Train with what you'll race with (minimum 3 long runs with gear)

**Clothing layers (temperature-dependent):**
- **Hot (>25°C):** Lightweight shorts, singlet/shirt, hat with brim, sunglasses
- **Moderate (15-25°C):** Shorts, short-sleeve shirt, hat, arm sleeves (optional)
- **Cold (<15°C):** Tights/pants, long-sleeve shirt, gloves, hat/buff
- **Rain:** Lightweight waterproof jacket (pack even if forecast is clear)

**Lighting:**
- **Headlamp:** Minimum 200 lumens (300-500 better), spare batteries
- **Vest light:** Front-mounted (chest) + rear light for visibility
- **Test before race:** Run in dark with exact setup to verify comfort/brightness

### Foot care

**Sock strategy:**
- **Single sock:** Most common, less friction if fit is good
- **Double socks:** Thin liner + thicker outer (reduces blister risk for some)
- **Change schedule:** Every 25-50K or when wet (sweat/stream crossings)

**Blister prevention:**
- Pre-tape hot spots (heels, toes) with Leukotape before race
- Lube (Body Glide, Vaseline) on common friction areas
- If hot spot develops: Stop immediately, tape BEFORE blister forms

**Shoe choice:**
- Use shoes with 50-100km of training miles (broken in but not worn out)
- Match shoe to terrain: more cushion for rocky, more grip for muddy
- **Don't try new shoes on race day!**

## Contingency Planning

### Common issues & protocols

**Blisters:**
- **Prevention:** Pre-tape, change socks when wet, good-fitting shoes
- **If blister forms:** Stop at aid station, pop with sterile needle, drain, tape over
- **Keep moving:** Taped blister is painful but manageable

**GI issues (nausea, cramping, diarrhea):**
- **Slow down:** Reduce intensity, walk if needed
- **Stop solid food:** Switch to liquid nutrition (sports drink, cola)
- **Ginger/antacid:** Ginger chews or Tums can help nausea
- **If persistent:** Walk, sip water, give gut rest (10-20 min)
- **Prevention:** Gut training (see race-nutrition skill)

**Chafing:**
- **Prevention:** Lube before race, reapply at crew stops
- **If occurs:** Apply more lube immediately, adjust clothing/vest straps
- **Emergency:** Band-aids or tape over raw areas

**Getting lost/off course:**
- **Stop immediately when unsure:** Don't continue hoping you'll find the way
- **Backtrack:** Return to last known marker/flag
- **Check GPS/map:** Verify position before continuing
- **Ask other runners:** If you see them, follow or confirm direction
- **Alert race:** Report missing markers if found

**Cramping:**
- **Cause:** Usually electrolyte imbalance or over-exertion
- **Protocol:** Slow down, stretch gently, consume salt/electrolytes
- **Prevention:** Adequate sodium intake (300-600 mg/h), don't surge pace

**Bonking (severe energy depletion):**
- **Recognize early:** Sudden fatigue, dizziness, confusion, irrational thoughts
- **Protocol:** Stop, sit, consume fast-acting CHO (cola, gels), wait 10-15 min
- **Recovery:** Will take 20-30 min to feel better, then very conservative pace
- **Prevention:** Consistent fueling from start (don't wait until hungry)

### When to push through vs when to stop

**Push through (with caution):**
- General fatigue (normal for ultra)
- Blisters (if taped and manageable)
- Minor aches (DOMS, muscle soreness)
- Dark moments (psychological)
- "This is harder than I expected" (welcome to ultra running)

**Stop immediately:**
- Chest pain, difficulty breathing (cardiac concern)
- Severe dehydration (no urination for 6+ hours, dark urine, confusion)
- Hyponatremia symptoms (nausea, headache, swelling, confusion after drinking a lot)
- Acute injury (twisted ankle, fall with sharp pain)
- Heat stroke (no sweating, confusion, core temp >40°C)

**Gray area (assess carefully):**
- Persistent nausea >2 hours despite slowing + changing nutrition
- Pain that changes gait significantly (could worsen injury)
- Sleep deprivation hallucinations (common in 100-milers, but if severe/dangerous, consider DNF)

## Pre-Race Logistics

### Race week protocol

**7 days before race:**
- Final taper week (see periodization-coach skill)
- Finalize gear list, test all equipment
- Review course maps, elevation profiles
- Plan drop bags (if applicable)

**3-4 days before:**
- Begin carb-loading (8-12 g/kg/day, see race-nutrition skill)
- Reduce fiber (avoid GI issues)
- Stay hydrated
- Avoid new foods

**1 day before:**
- Race check-in, bib pickup
- Course reconnaissance (if feasible): drive/view key sections
- Lay out all gear, check mandatory items
- Eat familiar high-CHO dinner (not excessive)
- Early bed (sleep 2 nights before matters more than night before)

**Race morning:**
- Pre-race meal 3-4h before start (see race-nutrition skill)
- Arrive at start 60-90 min early (parking, gear check, bathroom, warm-up)
- Body marking (bib, timing chip)
- Final gear check
- Warm-up: 10-15 min easy jog + dynamic stretches

### Course reconnaissance

**If possible (day before):**
- Drive key sections (aid station locations, major climbs/descents)
- Note landmarks (helps with mental checkpoints during race)
- Check footing (rocky? muddy? technical?)

**Use elevation profile:**
- Identify major climbs (where you'll slow/hike)
- Note steep descents (where quads will take beating)
- Plan mental segments around profile

### Crew & pacer coordination

**Crew protocol (if you have crew):**
- Designate crew chief (one person in charge, not committee)
- Crew meets you at designated aid stations with gear ready
- Pre-planned: What you'll need at each stop (written list)
- Crew keeps you moving: "5-minute timer" approach
- Crew provides positivity: "You look great!" (even if you don't)

**Pacer protocol (if allowed):**
- Pacer joins at designated mile (often mile 50-60 for 100-milers)
- Pacer's job: Keep you moving, navigate, positive energy, make decisions if you're bonking
- Brief pacer before race: Your pacing plan, nutrition strategy, what helps you mentally
- Pacer runs slightly ahead/beside (not behind) to lead

**If no crew/pacer:**
- Self-sufficient strategy: More gear in drop bags
- Make friends on course: Other runners can help with morale
- Aid station volunteers: Ask for help when needed (they want you to succeed)

## Integration with Other Skills

### Cross-references

**For race-day nutrition strategy:**
- See `race-nutrition` skill for detailed fueling protocols, gut training, hydration targets

**For pre-race training (final weeks):**
- See `periodization-coach` skill for taper protocols, final hard workouts, B2B long runs

**For recovery after race:**
- See `athlete-monitoring` skill for post-race recovery protocols, red flags to watch for

**For injury concerns during race:**
- See `kinesiology-foundations` skill for injury assessment, when to stop vs push through

## Methodological Sources

### Ultrarunning Training Essentials (Jason Koop, 2nd Ed.)
- **Pacing strategies:** CH.7 (Race Execution)
- **Aid station efficiency:** CH.7 (Race Execution)
- **Mental strategies:** CH.9 (Mental Aspects)
- **Gear setup:** CH.8 (Equipment)
- **Ссылка:** `knowledge/Ultrarunning Training Essentials.txt`

### Training for the Uphill Athlete (House, Jornet, Johnston)
- **Mountain-specific pacing:** Uphill/downhill tactics
- **Mental preparation:** CH.12 (Mental Training)
- **Ссылка:** `knowledge/Training for the Uphill Athlete.txt`

### Practical ultra racing experience (synthesis)
- **DNF decision framework:** Synthesis from race reports, coaching experience
- **Contingency protocols:** Common issues and proven solutions

## Key Takeaways

1. **Pace conservatively early:** 90% of ultra runners start too fast; don't be one of them
2. **Efficiency matters:** Every minute at aid station = 3+ minutes on course
3. **Dark moments pass:** Expected at mile 30-40, 50-65, 80-90; protocol is: eat, slow down, wait
4. **Gear must be tested:** Never try new shoes, nutrition, or equipment on race day
5. **Mental game is real:** Race segmentation and positive self-talk prevent DNFs
6. **Safety first:** DNF if dangerous to long-term health; push through discomfort
7. **Celebrate the journey:** Every finish is a victory, regardless of time

**Remember:** "Train hard, race smart" - Tactics can make or break your race more than fitness.

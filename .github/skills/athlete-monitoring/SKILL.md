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
- Monitoring menstrual cycle / RED-S risk / female athlete context when applicable

## Приоритет источников данных

### От высшего к низшему:

1. **Протокол газоанализа из `knowledge/`** (самый свежий по дате)
   - Файлы вида `МПК_тест_*.md`
   - Источник истины для AeT/LT/VO2max/HRmax

2. **Sport settings из Intervals.icu** (через MCP)
   - Текущие пороги/зоны в системе
   - Может отставать от актуальных тестов

3. **Данные, озвученные пользователем в чате**
   - Субъективная оценка, контекст, ограничения

**При конфликте:** использовать источник с более высоким приоритетом и синхронизировать остальные

## Работа с протоколами газоанализа (МПК/AeT/LT)

### Актуальный тест на газоанализаторе — ОБЯЗАТЕЛЬНО

**Если в проекте есть протокол газоанализа** (например, в папке `knowledge/` файлы вида `МПК_тест_*.md`):
- Считай его **источником истины** для AeT/LT/VO2max/HRmax
- **ПЕРЕД любым планом** учитывай результаты

### Выбор самого свежего теста

1. **Сначала** по дате в имени файла (формат `_YYYYMMDD`)
2. **Затем** по дате внутри документа
3. Если дат несколько/неясно — задай 1 уточняющий вопрос
4. **НЕ выдумывай значения**

### Извлечение данных из протокола (минимум)

**Обязательно извлеки:**
- **AeT**: ЧСС + темп/скорость на AeT
- **LT (ПАНО)**: ЧСС + темп/скорость на LT
- **HRmax** (если есть) и **VO2max** (если есть)

### Использование данных для планирования

**Используй эти значения для:**

1. **Расчёта AeT-LT gap** (и любых выводов про «аэробный дефицит»)
   - Формула: `AeT-LT gap (%) = (LT - AeT) / AeT × 100`

2. **Капов для Z2** (HR cap на AeT) и описаний целевых зон в тренировках
   - Верхняя граница Z2 = AeT HR (например, 155 bpm если AeT=155)

3. **Темповых/пороговых ориентиров**
   - Если пользователь просит по темпу — используй LT pace из теста

### Синхронизация с Intervals.icu

**Сверь извлечённые пороги с текущими sport settings в Intervals.icu через MCP.**

Если тест новее и есть расхождения:
1. Обнови пороги через `mcp_rusty-interva_manage_profile` (`action: update_thresholds`)
2. Применяй к историческим активностям только после явного подтверждения пользователя (`apply_to_activities: true`)
3. Перед применением предупреди пользователя, что пересчёт затронет исторические метрики и может занять время
4. После применения проверь изменения через `mcp_rusty-interva_manage_profile` (`action: get`) и `mcp_rusty-interva_analyze_training` (`target_type: period`)

**Fallback (если теста нет):**
- Используй sport settings из Intervals.icu (через MCP)
- Если и там пусто — запроси у пользователя AeT HR и LT HR перед планированием

## Мониторинг восстановления

### Женский физиологический контекст (если применимо)

Учитывай, что у спортсменок recovery/readiness может зависеть не только от объёма, но и от гормонального контекста.

**Отслеживай и учитывай:**
- фазу менструального цикла и субъективные паттерны по фазам (энергия, GI-толерантность, RPE, качество сна)
- признаки низкой энергодоступности / RED-S: потеря или редкость цикла, низкое либидо, хроническая усталость, частые травмы, повторные болезни, настроение, необъяснимое падение производительности
- при перименопаузе/менопаузе: ухудшение сна, приливы, повышенная теплочувствительность, более медленное восстановление, снижение переносимости высокой плотности нагрузки

**Практика:**
- не делай жёстких выводов по одному циклу или одному плохому дню — ищи повторяющиеся паттерны 2-3 цикла подряд
- если фаза цикла регулярно даёт GI/distress или падение качества ключевых тренировок, адаптируй микроцикл, а не игнорируй сигнал
- если есть подозрение на RED-S или выраженную эндокринную проблему — снизь нагрузку и рекомендуй врача/спортивного диетолога

### Post-race monitoring workflow

После завершённой гонки используй связку:
1. `mcp_rusty-interva_analyze_race` — зафиксировать результат, execution pattern и post-race load context
2. `mcp_rusty-interva_assess_recovery` (`period_days: 7-14`) — оценить readiness, red flags и глубину recovery block

**Практические замечания:**
- Для выбора нужной гонки предпочитай `description_contains`; затем проверь, что ответ относится к нужному старту по `name`/дате/ID.
- Если `Data Availability` сообщает, что отсутствуют streams, intervals или wellness, не переоценивай точность выводов о decoupling, fatigue и тактических ошибках.
- Если `Post-Race Load Context` указывает на elevated load (например, TSB < -10), это сильный аргумент в пользу recovery block и запрета на раннее возвращение к интенсивности.

### Fatigue Matrix (мышечная усталость vs энергетическая)

**Типы усталости:**

1. **DOMS (Delayed Onset Muscle Soreness)**
   - Отсроченная мышечная боль
   - Причина: микроразрывы мышечных волокон
   - Решение: recovery workouts (Z1, очень лёгкие), mobility/растяжка

2. **Dead legs (гликогенное истощение, неврологическая усталость)**
   - Ощущение "тяжёлых ног", нет силы/отклика
   - Причина: недостаток гликогена, накопленное утомление ЦНС
   - Решение: отдых, питание (углеводы), снижение интенсивности

### Grading workouts: качество выполнения

**Шкала оценки:**
- **A (Superman)**: тренировка прошла легче чем ожидалось, чувствую себя отлично
- **B (Good)**: всё по плану, нормальные ощущения
- **C (OK)**: сложно, но выполнил
- **D (Struggle)**: очень тяжело, не смог выполнить в полном объёме
- **F (Failed)**: не смог тренироваться

**Интерпретация:**
- Серия C/D грейдов (2+ подряд) = снизить нагрузку на 20-30%
- Один F грейд = разовый отдых, наблюдение
- Повторяющиеся D/F = признак перетренированности, красный флаг

### CTL/ATL/TSB (Chronic Training Load / Acute Training Load / Training Stress Balance)

**Используй как основной контекст при решении вводить интенсивность:**

- **CTL (Chronic Training Load)**: долгосрочная форма (обычно 42-дневное скользящее среднее TSS)
- **ATL (Acute Training Load)**: краткосрочная усталость (обычно 7-дневное скользящее среднее TSS)
- **TSB (Training Stress Balance)**: CTL - ATL (положительный = свежий, отрицательный = усталый)

**Правила:**
- Избегай резкого роста CTL при ухудшающемся wellness
- Стабильный CTL (не падающий тренд) и положительный или нейтральный TSB — критерий готовности к добавлению интенсивности
- Отслеживай ramp rate (скорость роста CTL): не более 5-8 TSS/день в неделю

**Источник:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

### HR drift, HRV и RPE

**Отслеживай вместе — ни одна метрика не заменяет контекст:**

- **HR drift**: повышение HR при том же темпе/мощности во время тренировки
  - Нормальный drift: до 5-7% за час при стабильном темпе в Z2
  - Красный флаг: >10% drift при привычной тренировке

- **HRV (Heart Rate Variability)**: вариабельность сердечного ритма
  - Падение HRV на 20%+ от базового уровня = недовосстановление
  - Используй тренды (7/30-дневные скользящие средние), не единичные замеры

- **RPE (Rate of Perceived Exertion)**: субъективная оценка усилий
  - Если RPE выше чем обычно при том же темпе/HR = признак усталости
  - Сравнивай RPE с HR и темпом для выявления расхождений

**Источник:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

### Мониторинг и верификация AeT/LT

**Проверяй AeT drift test каждые 4-6 недель:**
- 30-60 мин в Z2 при стабильном HR
- Если темп растёт при том же HR → AeT улучшается

**Критерии для повторного теста:**
- При улучшении AeT pace на 5%+ при том же HR: рассмотри повторный лабораторный тест или полевой тест для обновления порогов
- Сравнивай pace curves (30-90 дней) с предыдущим периодом для объективной оценки прогресса

**Готовность к введению Z3:**
- AeT-LT gap сужается (<10%)
- Атлет стабильно выполняет Z1-Z2 объём
- CTL растёт или стабилен, TSB положительный/нейтральный

## Красные флаги (немедленно снизить нагрузку или отдых)

**Source:** Frontiers in Physiology (2018) - Sleep as an indicator of overreaching/overtraining; Wang et al. (2023); recent ultra-endurance research

**ОБЯЗАТЕЛЬНОЕ действие при обнаружении любого из признаков:**

1. **Sleep <6.5 hours 3+ consecutive nights**
   - Evidence: Overreached athletes show sleep duration ↓5.4%, efficiency ↓2.1%
   - Red flag: <6.5h 3+ nights OR sleep efficiency <80% sustained
   - Действие: cut volume -30-40%, only Z1 or rest, prioritize sleep hygiene

2. **Resting HR elevated +3-5 bpm sustained 2+ weeks** (not single day spike)
   - Measure: Morning HR in bed before getting up
   - Example: Baseline 55 bpm → sustained 58-60 bpm for 2+ weeks
   - Действие: reduce intensity, add recovery week (40-60% volume reduction)

3. **HRV falling trend 2-3 consecutive weeks**
   - Measure: 7-day rolling average (not single readings)
   - Red flag: Declining trend without recovery, or sudden drop >20% from baseline
   - Действие: recovery week, monitor closely, consider medical check if persistent

4. **HR drift >10% at same pace on familiar workout**
   - Example: Z2 run at 5:30/km normally shows 150→155 bpm (3% drift), but today 150→168 bpm (12% drift)
   - Причина: Dehydration, heat, insufficient recovery, or impending illness
   - Действие: stop workout, add rest day, recheck next session

5. **Inability to raise HR to target zone** despite effort
   - Example: Trying to do Z4 intervals but HR won't go above Z2-Z3
   - Причина: Deep CNS (central nervous system) fatigue
   - Действие: immediate rest, 3-7 days off or Z1 only, consider medical consultation

6. **Pain worsening during run** (not just initial stiffness that improves)
   - Distinguish: DOMS/stiffness improves after warm-up; injury pain worsens with continued running
   - Действие: stop immediately, consult physiotherapist/sports doctor

7. **Loss of motivation + physical fatigue simultaneously**
   - Psychological marker often appears BEFORE physical decline
   - Additional signs: loss of libido, chronic irritability, loss of appetite, GI issues
   - Действие: 3-7 days complete rest or Z1 only, reassess training load/life stress

8. **URTI (Upper Respiratory Tract Infections) 2+ times in 8 weeks**
   - Evidence: Overreached athletes have 2-3x higher infection rate due to immune suppression
   - Действие: reduce volume, increase recovery, check sleep/nutrition, consider immune support (vitamin D, fish oil)

9. **Менструальная дисфункция / признаки RED-S**
   - Примеры: отсутствие цикла >90 дней, циклы >35 дней, выраженное падение энергии на фоне высокого объёма, повторные стрессовые реакции/травмы
   - Действие: immediate deload, проверить энергодоступность/питание, рекомендовать врача и/или sports dietitian

10. **Сильное ухудшение сна + теплочувствительность у спортсменки 40+**
   - Возможный контекст: перименопауза/менопауза, накопленная усталость, гормональный сдвиг
   - Действие: снизить плотность интенсивности, усилить heat management и восстановление, рекомендовать дообследование при стойких симптомах

**When 2+ red flags present:** Immediate recovery week (50% volume, Z1 only, extra sleep)

**Источники:**
- Frontiers in Physiology (2018) - Sleep & overreaching
- Training for the Uphill Athlete (signs of overtraining)
- Ultrarunning Training Essentials, CH.6 (red flags)

## Травмопрофилактика (Injury Prevention)

### Правила нагрузки

**Не увеличивай вертикальный набор и объём одновременно более чем на 5% каждый:**
- Если увеличиваешь время на ногах на 7% → вертикаль держи стабильной
- Если увеличиваешь вертикаль на 5% → время на ногах держи стабильным или увеличивай минимально

**После большого вертикального набора (>1500m):**
- Следующий день Z1 или отдых
- Избегай интенсивности (Z3/Z4) на следующий день

### Обязательные дополнения при высоком объёме

**Mobility/растяжка:**
- Минимум 10-15 мин после каждой тренировки >1ч
- Фокус: бедра, икры, подошвенная фасция, IT band

**Силовая работа на стопу/голень:**
- Обязательна при объёме >8ч/нед
- Примеры: toe raises, calf raises, single-leg balance
- Источник: Science of Running Technique Analysis

**Hip/glute stability:**
- Важна для спусков и технического рельефа
- Примеры: single-leg squats, lateral band walks, clamshells

**Core stability:**
- Поддерживает экономичность бега на длинных дистанциях
- Примеры: planks, dead bugs, pallof press

**Источники:**
- Training for the Uphill Athlete, ch.6-8 (Strength and the Uphill Athlete)
- Science of Running Technique Analysis (Preventing injury)

### Normalized Graded Pace (NGP) / Grade Adjusted Pace

**Применяй для сравнения усилий в пересечённом рельефе:**
- NGP учитывает влияние подъёмов/спусков на темп
- Используй для оценки "истинной" интенсивности на технической трассе
- Доступен в Intervals.icu как computed field

**Источник:** Ultrarunning Training Essentials, CH.6 (Tracking Training)

## Протокол взаимодействия с атлетом

### ОСНОВНОЙ ИСТОЧНИК ДАННЫХ — MCP (Intervals.icu)

Тренер **СНАЧАЛА собирает данные через MCP**, а затем уточняет у пользователя только недостающее.

### ЧТО ПОЛЬЗОВАТЕЛЬ МОЖЕТ ДОПОЛНИТЕЛЬНО СООБЩИТЬ

**Информация об атлете (если нет в Intervals.icu):**
- Возраст, пол, опыт
- Целевые гонки (A/B/C приоритет), дистанция, вертикальный набор
- Ограничения: доступное время, доступ к рельефу, травмы

**Обратная связь после тренировок:**
- Субъективные ощущения (грейд A-F), качество сна, стресс
- DOMS, "dead legs", общая усталость
- Любые отклонения от плана

**Планы и ограничения:**
- Жизненные события (командировки, отпуск)
- Изменения в доступности для тренировок

### ЧТО ТЫ БУДЕШЬ ДЕЛАТЬ

**Анализировать предыдущие тренировки:**
- Качество выполнения (темп vs HR vs RPE)
- Признаки усталости (серия C/D грейдов, высокий HR при низком темпе)
- Прогресс AeT (улучшение темпа при том же HR)

**Учитывать будущие планы:**
- Дистанция до целевой гонки (недель)
- Текущий период (Transition/Base/Specific/Taper)
- Необходимые адаптации (больше объёма? интенсивность? восстановление?)

**Отслеживать долгосрочный прогресс:**
- Изменения в объёме (weekly hours, vertical gain)
- Улучшение AeT pace
- Качество восстановления
- Готовность к добавлению интенсивности

## Методологические источники

### Training for the Uphill Athlete (House, Jornet, Johnston)
- **Для мониторинга**: признаки перетренированности, fatigue matrix, грейдинг тренировок
- **Ссылка**: `knowledge/Training for the Uphill Athlete.txt`
- **Ключевые разделы**: «Monitoring and Recovery», «Signs of Overtraining»

### Ultrarunning Training Essentials (Jason Koop)
- **Для мониторинга**: CTL/ATL/TSB, HR drift, HRV, NGP, красные флаги
- **Ссылка**: `knowledge/Ultrarunning Training Essentials.txt`
- **Ключевые главы**: CH.6 (Tracking Training)

### Science of Running Technique Analysis (Chris Napier)
- **Для профилактики травм**: силовая работа для стопы/корпуса, техника бега, подборка упражнений
- **Ссылка**: `knowledge/Science of Running Technique Analysis.txt`
- **Ключевые разделы**: «Running form», «Strength», «Preventing injury»

### Протоколы МПК в `knowledge/`
- **Например**: `МПК-тест-2025-10-09.md`
- **Использование**: источник истины для AeT/LT/VO2/HRmax
- Перед изменением порогов сверяй значения и фиксируй дату теста в changelog как источник для `mcp_rusty-interva_manage_profile`

## Как включать источники в рабочий процесс

- Всегда добавляй короткую сноску в описании изменений/коммитах/логах, например: "Source: knowledge/Training for the Uphill Athlete.txt — Monitoring and Recovery"
- При рассогласовании литературы и данных MCP: обсуди с пользователем, укажи предпочтения и объясни выбранную логику (какой источник приоритетнее и почему)
- При внесении изменений в пороги/зоны — добавляй причину (МПК/новые обнаружения), ссылку на файл и дату теста в changelog, и сохраняй старые значения отдельно для отката

## Когда обращаться к другим skills

- **Периодизация, зоны, силовая подготовка**: см. `periodization-coach` skill
- **Формат вывода Intervals.icu, MCP-инструменты**: см. `intervals-icu-integration` skill
- **Травмопрофилактика, evidence-based practice, движения**: см. `kinesiology-foundations` skill

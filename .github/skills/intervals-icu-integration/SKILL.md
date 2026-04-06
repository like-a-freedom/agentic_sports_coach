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

## Core principle: MCP-first approach
**ВСЕГДА собирай данные через MCP-инструменты** перед созданием плана или корректировкой тренировок.
Не полагайся только на слова пользователя, если данные можно взять через MCP.

## Token-эффективность и контракты MCP

### Важно про версии сервера

- MCP-контракты и имена инструментов могут меняться между версиями `rusty-interva`.
- Перед вызовом всегда ориентируйся на актуальную схему конкретного инструмента в MCP.

### Практика для экономии токенов

1. Сначала используй агрегированные/списочные инструменты (`list*`, `search*`), затем переходи к детальным (`get*`) только для целевых сущностей.
2. Ограничивай диапазоны дат и объём выборки (`oldest/newest`, `limit`).
3. Для кривых/гистограмм запрашивай только нужный диапазон и тип спорта.


### Канонический стиль имен (low‑level, редкие случаи)

- **Главный ориентир** — intent-инструменты (`mcp_rusty-interva_*`), они покрывают большинство сценариев. Этот раздел описывает лишь low‑level вызовы, которые теперь считаются запасным вариантом.


## Формат вывода: intervals.icu

### ЖЁСТКИЙ КОНТРАКТ ВЫВОДА (важно для агентности)

**1) План тренировок (неделя/несколько недель):**
Выводи ТОЛЬКО текст плана в формате intervals.icu (готовый для копирования и вставки):
- Без Markdown-заголовков (##), без жирного (**), без code fences (```), без таблиц
- Любые объяснения/«почему» встраивай внутрь описаний тренировок строками начинающимися с "-" (Purpose/Focus/Why)
- Допустимы служебные строки внутри плана: "Weekly totals:", "Key decisions:", "What to track:" — но тоже без Markdown

**2) Анализ/объяснения отдельно:**
Сначала дай план в формате intervals.icu, а затем (после пустой строки) краткий анализ обычным текстом.

**3) Проверяй арифметику:**
- AeT-LT gap (%) = (LT - AeT) / AeT × 100
- Если в данных/примере есть несостыковки — явно исправляй и используй корректное значение

### Структура вывода для intervals.icu

```
#WEEK [номер недели]
[День недели], [длительность], [название тренировки]
- [описание/структура]

Weekly totals:
- Time: [общее время]
- Vertical gain: [вертикаль]
- Zone distribution: [распределение по зонам]
```

**Обозначения:**
- Длительность: `HH:MM` (например, `1:30` = 1 час 30 минут)
- Длительность шагов Workout Builder: `30s`, `10m`, `1h10m`, `1m30` (используй "m" и "s", не "min/sec")
- Зоны: `Z1`, `Z2`, `Z3`, `Z4`, `Z5`, `Recovery`
- Структура: warm-up (разминка), main set (основная часть), cooldown (заминка)
- `REST`: день отдыха

**Пример:**

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

### Workout Builder синтаксис (справочник)

#### Общая структура тренировки

Тренировка — простой текст. Делится на **секции** и **шаги**:
- **Секции** — строки без `-` (заголовки блоков): `Warmup`, `Main Set 4x`, `Cooldown`
- **Шаги** — строки, начинающиеся с `-`. Каждый шаг = один интервал
- Между секциями рекомендуется пустая строка

Пример минимальной структуры:

    Warmup
    - 10m 60%

    Main Set 5x
    - 3m 120%
    - 2m Z1

    Cooldown
    - 10m 50%

#### Parser-safe notes and comments

> ⚠️ **КРИТИЧНО**: любая строка, начинающаяся с `-`, воспринимается Workout Builder как шаг/интервал.

Используй `-` **только** для настоящих шагов тренировки. Любые служебные комментарии (`Nutrition`, `Recovery`, `Fueling`, `Why`, длинные подсказки) не должны выглядеть как интервалы.

**Плохо — заметка парсится как шаг:**

    - Nutrition: start fueling at 15-20m; 45-60 g/h
    - Recovery: within 30m take carbs + protein

Такие строки могут:
- раздувать расчётную длительность,
- ломать структуру workout preview,
- превращать текстовые заметки в псевдо-интервалы.

**Хорошо — заметки вынесены в обычный текст:**

    Warmup
    - 10m Z1 HR

    Main Set
    - 100m Z2 HR

    Cooldown
    - 10m Z1 HR

    Notes
    Nutrition: start fueling early; CHO 45-60 g/h; fluid 450-750 ml/h
    Recovery: within 30 min take 1.0-1.2 g/kg CHO + 20-30 g protein

**Практическое правило:** если строка не должна стать шагом Workout Builder, не начинай её с `-`.

#### Длительность

| Синтаксис | Значение |
|-----------|----------|
| `30s` | 30 секунд |
| `5m` | 5 минут |
| `1h` | 1 час |
| `1m30` | 1 мин 30 сек |
| `1h10m` | 1 час 10 минут |
| `1h30m59s` | 1 час 30 минут 59 секунд |
| `5'` | 5 минут (альтернатива) |
| `30"` | 30 секунд (альтернатива) |

#### Дистанция

| Синтаксис | Значение |
|-----------|----------|
| `2km` | 2 километра |
| `1mi` | 1 миля |
| `0.4km` | 400 метров |
| `400mtr` | 400 метров |

> ⚠️ **КРИТИЧНО**: `m` = минуты, НЕ метры! Для метров: `0.4km` или `400mtr`. Никогда `400m` — это 400 минут!

#### Интенсивность

**Велосипед (% от FTP):**

    - 10m 75%           # 75% FTP
    - 5m 95-105%        # диапазон 95–105% FTP
    - 8m 220w           # абсолютные ватты
    - 6m 200-240w       # диапазон в ваттах
    - 60m Z2            # зона 2 (мощность)

**Бег (темп):**

    - 6km 90-92% pace        # % от порогового темпа
    - 3km Z3 Pace            # зона 3 темпа
    - 10m 7:15-7:00/km Pace  # абсолютный темп

**Пульс (HR):**

    - 60m Z2 HR         # зона 2 по ЧСС
    - 20m 70% HR        # 70% от макс. ЧСС
    - 10m 90-95% LTHR   # 90–95% от порогового ЧСС

> ⚠️ Для бега ВСЕГДА уточняй тип цели: `Z2 HR` или `Z2 Pace` — без уточнения может интерпретироваться как power.

#### Таблицы зон

**Велосипед (% от FTP):**

| Зона | Название | % FTP |
|------|----------|-------|
| Z1 | Recovery | < 55% |
| Z2 | Endurance | 56–75% |
| Z3 | Tempo | 76–90% |
| Z4 | Threshold | 91–105% |
| Z5 | VO2 Max | 106–120% |
| Z6 | Anaerobic | > 120% |

**Бег (% от порогового темпа):**

| Зона | Название | % Pace |
|------|----------|--------|
| Z1 | Recovery | 70–75% |
| Z2 | Easy | 76–80% |
| Z3 | Tempo | 81–88% |
| Z4 | Threshold | 89–95% |
| Z5 | VO2 Max | 96–105% |
| Z6 | Sprint | > 105% |

#### Повторения (Repeats)

Два способа:
1. В заголовке секции: `Main Set 5x`
2. Отдельной строкой перед шагами:

       5x
       - 3m 120%
       - 2m Z1

> ⚠️ Вложенные повторения (nested repeats) не поддерживаются.

#### Рампы (Ramp)

Плавное нарастание/спад — ключевое слово `ramp`:

    - 10m ramp 50%-75%        # разминка: нарастание мощности
    - 10m ramp 60-80% pace    # нарастание темпа (бег)
    - 8m ramp 50%-40%         # спад на заминке
    - 15m ramp 60%-90% 85rpm  # нарастание + каденс (велосипед)

Особенно полезны для плавных разминок/заминок вместо ступенчатых переходов.

#### Каденс (только велосипед)

Добавляется в конец строки как `NNrpm` или диапазон:

    - 10m 75% 90rpm       # фиксированный 90 об/мин
    - 12m 85% 90-100rpm   # диапазон каденса
    - 15m ramp 60-90% 85rpm  # рамп + каденс

#### Text Prompts (подсказки на устройство)

Текст перед первым указанием длительности = подсказка на Garmin/Wahoo:

    - Recovery 30s 50%
    - Zone 4 interval 5m 95-105%
    - Low cadence 4m 100%

#### Timed Prompts (тайминговые подсказки)

Сообщения в конкретный момент внутри шага (секунды от начала), разделитель `<!>` обязателен:

    - Start sprint 30^ Relax 60^ Final push <!> 1m30 120%

#### Freeride (ERG off)

Шаг без управления мощностью — свободное педалирование:

    - 20m freeride

#### Смешивание типов целей

HR, Pace и Power можно комбинировать в одной тренировке:

    Warmup
    - 10m Z1 HR

    Main Set 4x
    - 6m 100%
    - 3m 75-80% pace

    Cooldown
    - 10m ramp 60-40% HR

> ⚠️ **Практическая оговорка для бега:** синтаксически смешивание `HR` и `Pace` допустимо, но в Workout Builder это не всегда даёт предсказуемый UX.

Для run workouts, которые пользователь будет открывать/редактировать в режиме `Pace`, смешение `HR + Pace` в одной сессии может приводить к тому, что warmup/cooldown или recovery-блоки отображаются неудобно или визуально «пропадают» при переключении режима.

**Надёжное правило:**
- если main set задан в `Pace`, то warmup/cooldown/recovery для этой сессии тоже предпочитай в `Pace`;
- если workout в первую очередь HR-guided, держи всю сессию в `HR`;
- смешивай типы только когда это действительно нужно, а не по умолчанию.

#### Практическое правило для Pace-mode run workouts

Для контрольных беговых сессий, time trials и pace-oriented workouts предпочитай **единый `Pace`-тип во всей сессии**.

**Предпочтительный паттерн:**

    Warmup
    - 15m Z1 Pace

    Main Set
    - 46m Z4 Pace

    Cooldown
    - 10m Z1 Pace

Для таких тренировок как 5K / 10K / HM tests обычно удобнее использовать **relative Pace**:
- `Z3 Pace`
- `Z4 Pace`
- `88-92% Pace`

а не абсолютный pace как дефолтный формат, если цель — стабильное отображение и редактирование в Builder `Pace` mode.

Абсолютный pace (`4:50-4:35/km Pace`) остаётся валидным и полезным, когда нужен точный целевой диапазон, но relative Pace обычно надёжнее как стандартный формат для редактируемых running workouts.

#### Полные примеры (бег)

**Аэробная тренировка со страйдами:**

    Warmup
    - 15m Z1 HR

    Main Set
    - 50m Z2 HR

    Strides 6x
    - 15s Z4 Pace
    - 1m45 Z1 HR

    Cooldown
    - 13m Z1 HR

**Интервалы по дистанции:**

    Warmup
    - 1mi Z1 HR

    Main Set 4x
    - 1mi Z4 Pace
    - 0.5mi Z1 Pace

    Cooldown
    - 1mi Z1 Pace

#### Дистанционные шаги (сводка единиц)

Поддерживаются: `km`, `mi`,`meters`, `yrd`, `y`
- `- 3km 80% Pace`
- `- 0.4km Z3 Pace` (400 метров)
- `- 10m 7:15-7:00/km Pace` (абсолютный темп с единицами `/km`, `/mi`)

## Common mistakes

- Использовать `-` для обычных заметок (`Nutrition`, `Recovery`, `Fueling`) — Builder интерпретирует их как интервалы.
- Писать duration-like токены (`15-20m`, `30m`) в псевдо-шагах с `-` — это может ломать preview и расчёт длительности.
- Смешивать `HR` warmup/cooldown с `Pace` main set в тренировках, которые потом будут открываться в Builder `Pace` mode.
- Использовать absolute Pace как единственный стандарт для editable run tests, когда relative Pace даёт более предсказуемое поведение UI.

### Силовые тренировки (Weight training)

**Формат событий в Intervals.icu:**
- В календаре указываем краткую строку вида: `* 1h 40-70% HR (72-125bpm)`
  - Первая часть это длительность
  - Далее целевой диапазон по %HR
  - Ориентировочные абсолютные bpm

**В Workout Builder:**
- Используем одну основную строку с длительностью и целью: `* 1h 40-70% HR`
- Добавляем подпункты с упражнениями и сетами

**В `workout_doc`:**
- step: `duration: 3600` и `hr` с `units: "%hr"`, `start: 40`, `end: 70`
- Это нужно чтобы Intervals корректно рассчитывал moving_time и зону

**В описании тренировки:**
- Добавляем структуру (упражнения, подходы, отдых)
- Примечание с абсолютными bpm для ориентира (например 72-125 bpm)

## Интеграция с Intervals.icu через MCP (rusty-intervals-mcp, intent-модель v2+)

### Intent-first контракт

Используй intent-инструменты как основной интерфейс:

- `mcp_rusty-interva_plan_training`
- `mcp_rusty-interva_analyze_training`
- `mcp_rusty-interva_modify_training`
- `mcp_rusty-interva_compare_periods`
- `mcp_rusty-interva_assess_recovery`
- `mcp_rusty-interva_manage_profile`
- `mcp_rusty-interva_manage_gear`
- `mcp_rusty-interva_analyze_race`

### Обязательный сбор контекста перед планированием

Перед планом на неделю и более всегда выполни:

1. `mcp_rusty-interva_manage_profile` (`action: get`) — профиль, зоны, пороги
2. `mcp_rusty-interva_analyze_training` (`target_type: period`) — фактический объём/интенсивность
3. `mcp_rusty-interva_assess_recovery` — readiness + red flags
4. При необходимости: `mcp_rusty-interva_compare_periods` (7/30/90 дней)

### Планирование и изменения

- Если пользователь просит создать план (неделя/блок) — используй `mcp_rusty-interva_plan_training`.
- Если нужно скорректировать существующую тренировку — используй `mcp_rusty-interva_modify_training`.
- Для destructive операций (`action: delete`, массовые правки) сначала `dry_run: true`, потом явное подтверждение пользователя.

### Управление порогами/зонами

Изменяй пороги только если:
1) пользователь явно просит; или
2) есть новый МПК-протокол и пользователь подтвердил применение.

Workflow:

1. `mcp_rusty-interva_manage_profile` (`action: get`) — собрать текущие пороги, сделать diff
2. `mcp_rusty-interva_manage_profile` (`action: update_thresholds`, `thresholds_source`)
3. Применять к истории только после согласия пользователя: `apply_to_activities: true`
4. Проверить эффект через:
   - `mcp_rusty-interva_manage_profile` (`action: get`)
   - `mcp_rusty-interva_analyze_training` (`target_type: period`)
   - `mcp_rusty-interva_assess_recovery`

### Анализ тренировок и прогресса

- Детальный разбор одной сессии: `mcp_rusty-interva_analyze_training` (`target_type: single`, `analysis_type: detailed|intervals|streams`)
  - `detailed` — зоны + основные метрики
  - `intervals` — анализ интервалов (для интервальных тренировок)
  - `streams` — потоковые данные по секундам (HR, pace, power)
  - `include_best_efforts: true` — сравнение с best efforts
  - `include_histograms: true` — гистограммы по метрикам
  - `metrics: [...]` — специфичные метрики: time, distance, vertical, tss, pace, hr
- Периодный анализ: `mcp_rusty-interva_analyze_training` (`target_type: period`)
- Сравнение блоков: `mcp_rusty-interva_compare_periods`

### Актуальный контракт `mcp_rusty-interva_analyze_race`

Используй `mcp_rusty-interva_analyze_race` для **post-race debrief**, оценки стратегии и поиска зон для улучшения после завершённой гонки.

**Входные поля по текущему контракту:**
- `date?: "YYYY-MM-DD" | "last_race"`
- `description_contains?: string`
- `analysis_type?: "performance" | "strategy" | "recovery"` (default: `performance`)
- `compare_to_planned?: boolean` (default: `true`)

**Практическое правило:**
- Основной рабочий селектор сейчас — `description_contains`.
- Если `description_contains` не передан, текущая реализация берёт самую свежую активность из недавнего списка.
- Поэтому **не полагайся только на `date` / `analysis_type` / `compare_to_planned`** для выбора нужной гонки или изменения логики ответа — обязательно проверь, что в ответе совпали `name`, дата и ID гонки.

**Что intent подтягивает автоматически:**
- recent activities (поиск гонки)
- workout details
- intervals
- streams
- fitness summary
- wellness за 7 дней

**Что обычно есть в ответе:**
- `Race Analysis` + метаданные гонки
- таблица результатов (distance, time, avg HR)
- `Execution Pattern` (segments, average HR note, Efficiency Factor, Aerobic Decoupling)
- `Post-Race Load Context` (например, рекомендация по recovery block на основе TSB)
- `Data Availability` (полный режим или degraded mode)
- `suggestions` / `next_actions`

**Ограничения текущей реализации:**
- `compare_to_planned` присутствует в схеме, но не даёт гарантированного полноценного planned-vs-actual diff — не обещай пользователю того, чего intent пока не строит явно.
- `analysis_type` принят схемой, но не должен интерпретироваться как надёжный переключатель разных форматов вывода без проверки фактического ответа.
- При отсутствии подходящей гонки intent возвращает мягкие suggestions/next actions, а не аварийную ошибку.

**Рекомендуемый workflow после гонки:**
1. `mcp_rusty-interva_analyze_race` с `description_contains`
2. Проверить `Data Availability` и понизить уверенность выводов, если нет streams/intervals/wellness
3. `mcp_rusty-interva_assess_recovery` (`period_days: 7` или `14`)
4. При необходимости `mcp_rusty-interva_plan_training` для recovery week / next block

**Когда нужен более глубокий drill-down:**
- Если нужен детальный разбор потоков/интервалов сверх race debrief, дополни анализ `mcp_rusty-interva_analyze_training` для целевой сессии и сопоставь выводы.

### Правила write-операций

- Никогда не выполняй mutating intent без явного указания пользователя.
- Перед любым изменением календаря/плана коротко подтверждай, что будет изменено.
- Если пользователь просит «только текст»/«не трогай календарь» — выдавай только текстовый план в формате intervals.icu.

## Документация Intervals.icu

Официальная документация по построению тренировок доступна по ссылкам:
- https://forum.intervals.icu/t/workout-builder/1163
- https://forum.intervals.icu/t/computed-activity-fields/25673
- https://forum.intervals.icu/t/custom-interval-fields/25942
- https://forum.intervals.icu/t/custom-activity-charts/28627
- https://intervals.icu/api-docs.html

## Когда обращаться к другим skills

- **Периодизация, зоны, силовая подготовка**: см. `periodization-coach` skill
- **Мониторинг восстановления, красные флаги, работа с МПК**: см. `athlete-monitoring` skill
- **Травмопрофилактика, evidence-based practice**: см. `kinesiology-foundations` skill

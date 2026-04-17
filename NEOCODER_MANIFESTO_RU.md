# neoCoder: Манифест Когнитивного Агента

<div align="center">

*"Агент — это не просто LLM с инструментами.  
Это инженер с памятью, принципами и совестью."*

</div>

---

# Почему архитектура агентов важна
Скажу прямо: дело уже не в модели.

Можно взять самый мощный LLM, но если у тебя слабый «скелет» агента — оркестрация, память, работа с инструментами — он развалится на реальных задачах.

Я видел, как одна и та же модель даёт абсолютно разный результат — всё зависит от того, как вокруг неё всё устроено. Разница между «демкой» и «продом» — не в LLM. А во всём, что его окружает.

Представь джуна, который каждые 5 минут всё забывает. Он не помнит, какие файлы трогал, что уже пробовал, какие ошибки делал. И снова наступает на те же грабли.

Вот так сегодня выглядят большинство AI-агентов.

NeoCoder построен на собственном SDK и спроектирован с фокусом на Claude модели.
Тут нужно прояснить - Антропики вложились в компьют агентных задач при обучении , и как результат модели могут автоматически выбирать правильный tool на уровне LLM. OpenAI модели так пока не умеют и ждут когда за них это сделает сам агент.



## Два Фундаментальных Вызова

После многих итераций над coding-агентами я свёл проблему к двум фундаментальным вызовам:

### Вызов 1: Long-Context Reasoning

Агент должен эффективно локализовать релевантный код в огромных репозиториях и выполнять multi-hop рассуждения через разрозненные модули, длинные трейсы инструментов и глубокие истории выполнения. Нельзя просто расширить контекстное окно — нужна *структура*.

### Вызов 2: Long-Term Memory

Агент должен накапливать persistent-знания между задачами и сессиями — захватывать переиспользуемые паттерны, failure modes и инварианты. Не переоткрывать информацию. Не воспроизводить прошлые ошибки.  Учиться.

Эти вызовы показывают: масштабирование требует большего, чем длинные контекстные окна или большие модели. Нужен  принципиальный подход к тому, как агент структурирует, поддерживает и взаимодействует с информацией.


### Решение: Когнитивная Архитектура

neoCoder построен на пяти фундаментальных принципах:

- 🧠 ПОМНИ
- 🎯 ДЕТЕРМИНИЗМ
- 🛡️ МИНИМИЗАЦИЯ ГАЛЛЮЦИНАЦИЙ
- 💭 ДОКАЗАТЕЛЬНОЕ МЫШЛЕНИЕ
- ✅ ФАКТЫ, НЕ ПРЕДПОЛОЖЕНИЯ


```
┌──────────────────────────────────────────────────────────────────────┐
│  СТАНДАРТНЫЙ АГЕНТ                                                   │
│                                                                      │
│  User: "Добавь авторизацию"                                          │
│      ↓                                                               │
│  LLM: *читает 50 файлов заново*                                      │
│      ↓                                                               │
│  LLM: *пишет код*                                                    │
│      ↓                                                               │
│  Контекст переполнен → ОБРЕЗКА → потеря критической информации      │
│      ↓                                                               │
│  LLM: "А что мы делали? Какой был план?"                             │
│      ↓                                                               │
│  ПРОВАЛ                                                              │
└──────────────────────────────────────────────────────────────────────┘
```

## 1. ПАМЯТЬ — не фича, а фундамент
### 9-уровневая Когнитивная Память

Обычные агенты:

либо тащат весь чат (и упираются в лимиты)
либо обрезают контекст (и теряют важное) 
- утечка Claude Code сырцов показала что  сжатие контекста там сделано коряво - 
не сохраняет рассуждения и их цепочки, прочитанные и записанные файлы, выводы и решения из задачи – всё идёт в топку.
Вот почему к 15-му сообщению Claude Code галлюцинирует имена переменных и ломает то, что только что понимал.
NeoCoder использует  когнитивную память, вдохновлённую человеческим мозгом и современными исследованиями (xMemory).

- рабочий контекст (что происходит прямо сейчас)
- эпизоды (что уже делали)
- знания (факты и выводы)
- темы (обобщённый опыт)

Плюс:

- заметки
- паттерны
- контекст сессии

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        КОГНИТИВНЫЙ МОЗГ АГЕНТА                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ╭─────────────────────────────────────────────────────────────────╮   │
│   │                    NEOCORTEX (L3: THEMES)                       │   │
│   │              "Auth & Security"  "API Design"  "Testing"         │   │
│   │                   Высокоуровневые темы знаний                   │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                 TEMPORAL LOBE (L2: SEMANTICS)                   │   │
│   │   "JWT expires in 15min"  "Use parameterized queries"           │   │
│   │                 Дискретные единицы знаний                       │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                  HIPPOCAMPUS (L1: EPISODES)                     │   │
│   │        "Implement OAuth" → успех, файлы: auth.py, jwt.py        │   │
│   │                    Записи выполненных задач                     │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                 FRONTAL LOBE (L0: WORKING)                      │   │
│   │                   Активный контекст + сжатие                    │   │
│   ╰─────────────────────────────────────────────────────────────────╯   │
│                                                                         │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │
│   │  THALAMUS   │  │BASAL GANGLIA│  │  PARIETAL   │  │  BRAINSTEM   │   │
│   │  Adaptive   │  │ Procedural  │  │   Notes     │  │   Session    │   │
│   │  Tuning     │  │  Patterns   │  │  Storage    │  │   Context    │   │
│   └─────────────┘  └─────────────┘  └─────────────┘  └──────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Главное отличие: не обрезать, а сжимать
Когда контекст разрастается:

- мы его не удаляем
- мы его переписываем в структурированную выжимку

Сохраняем:

- цель
- решения
- TODO
- ошибки

И продолжаем работу без потери смысла.

## Обучение на ошибках
neoCoder фиксирует не только успехи, но и фейлы:

что пошло не так
почему
как исправили
И потом это реально используется.

Иерархический Retrieval vs Flat RAG

Стандартный RAG: top-k по всем чанкам → избыточность, неточность, расход токенов.

xMemory в neoCoder: интеллектуальный каскад сверху вниз.

### Эффект памяти:

| Метрика | Без памяти | С neoCoder Memory |
|---------|------------|-------------------|
| Token efficiency | 100% | **40-60%** |
| Cross-session continuity | ❌ | ✅ |
| Experience reuse | ❌ | ✅ |
| Similar task speedup | 1x | **2-3x** |

---

## Принцип 2: ДЕТЕРМИНИЗМ — Предсказуемые Решения

neoCoder спроектирован для максимальной детерминированности  — минимизации случайных вариаций в решениях.
И главное правило:

> *"не «самый простой способ», а правильный способ для стека"*

### Результат: Два запуска с одной задачей дают структурно идентичный код.



- утечка Claude Code показала что системный промпт содержит ЖЁСТКИЕ указания -  «попробуй самый простой подход», «не рефактори сверх запрошенного», «три одинаковых строки лучше преждевременной абстракции». Когда просишь починить архитектуру, а агент лепит if/else костыль – это не лень, это выполнение системных инструкций с приоритетом над твоим промптом.


## Принцип 3: АНТИ-ГАЛЛЮЦИНАЦИИ — Grounded Reasoning

> *"Если ты не читал код — ты не знаешь, что он делает. Точка."*

Галлюцинации — главная причина провала AI-агентов. neoCoder имеет **многоуровневую защиту**.


### Если ты не читал код — ты не знаешь, что он делает. Точка.

Галлюцинации — главная причина провала AI-агентов.

neoCoder имеет многоуровневую защиту :

- Anti-Hallucination Rules в Brainstorming
- Mental Simulation Protocol 
- Semi-Formal Evidence
- Self-Questioning (AgentEvolver)
- Tool-Grounded Discovery

### Результат: Цепочка доказательств


```
┌──────────────────────────────────────────────────────────────────────┐
│                     GROUNDED REASONING CHAIN                         │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Task: "Add authentication"                                          │
│       │                                                              │
│       ▼                                                              │
│  [grep "auth" **/*.py] → Found: app/middleware.py:12                 │
│       │                                                              │
│       ▼                                                              │
│  [file_read app/middleware.py] → Existing CORS middleware           │
│       │                                                              │
│       ▼                                                              │
│  [Brainstorming] "I SEE existing middleware pattern at line 12"      │
│                  "Framework: FastAPI 0.100 (from pyproject.toml)"    │
│       │                                                              │
│       ▼                                                              │
│  [Mental Simulation] Happy: ✓  Edge: ✓  Failure: ✓                  │
│       │                                                              │
│       ▼                                                              │
│  [Implementation] Based on verified patterns                         │
│       │                                                              │
│       ▼                                                              │
│  [pre_commit_check] Evidence: middleware.py:45-67 ← file:line proof │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Принцип 4: ДОКАЗАТЕЛЬНОЕ МЫШЛЕНИЕ — Decision Funnel

> *"Код без плана — это не разработка. Это азартная игра."*

neoCoder никогда не пишет код сразу. Он следует **воронке решений**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DECISION FUNNEL                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  1. BRAINSTORMING                                             ║    │
│    ║     ┌─────────────────────────────────────────────────────┐   ║    │
│    ║     │ EXPLORE: Что РЕАЛЬНО нужно? Какие ограничения?      │   ║    │
│    ║     │ CHALLENGE: 2-3 подхода. Почему сработает/не сработает│   ║    │
│    ║     │ DECIDE: Выбор ИДИОМАТИЧНОГО решения для стека       │   ║    │
│    ║     │ SIMULATE: Happy path, Edge cases, Failure paths     │   ║    │
│    ║     └─────────────────────────────────────────────────────┘   ║    │
│    ║                          ↓ APPROVAL                           ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                                                                         │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  2. PLAN                                                      ║    │
│    ║     ┌─────────────────────────────────────────────────────┐   ║    │
│    ║     │ 1. Create auth/jwt.py with token generation         │   ║    │
│    ║     │ 2. Add middleware in app/middleware.py              │   ║    │
│    ║     │ 3. Update routes with @requires_auth decorator      │   ║    │
│    ║     │ ...                                                 │   ║    │
│    ║     │ N-1. Call pre_commit_check (MANDATORY)              │   ║    │
│    ║     │ N. Run build to verify                              │   ║    │
│    ║     └─────────────────────────────────────────────────────┘   ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  3. IMPLEMENT                                                 ║    │
│    ║     Следуем плану. SOLID. Без boilerplate.                    ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  4. PRE-COMMIT CHECK (Semi-Formal Verification)               ║    │
│    ║     ⛔ Каждое утверждение требует file:line доказательства    ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  5. BUILD/TEST                                                ║    │
│    ║     Только после прохождения pre_commit_check                 ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Принцип 5: ПРОВЕРЯЙ — CRITICAL Rules

> *"Работающий код — это минимум. Правильный код — это стандарт."*

neoCoder имеет **7 нарушимых правил** . Если любое нарушено — код **сломан**, даже если работает.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      ⛔ CRITICAL RULES                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  [CRIT-0] SECURITY-FIRST DESIGN                                        │
│           Whitelist > Blacklist. Least privilege. Defense in depth.    │
│                                                                         │
│  [CRIT-1] ARCHITECTURE PRINCIPLES                                      │
│           SOLID для backend. Feature-Sliced Design для frontend.       │
│                                                                         │
│  [CRIT-2] SECRETS NEVER LEAK                                           │
│           Никогда в логах, ответах, ошибках, коде. Только env/secrets. │
│                                                                         │
│  [CRIT-3] GUARANTEED CLEANUP                                           │
│           with/try-with-resources/defer/using — всегда.                │
│                                                                         │
│  [CRIT-4] EXPLICIT FAILURE HANDLING                                    │
│           Каждый внешний вызов: timeout + error handling + logging.    │
│                                                                         │
│  [CRIT-5] DECLARE ALL DEPENDENCIES                                     │
│           import → dependency manifest. Никаких implicit dependencies. │
│                                                                         │
│  [CRIT-6] PROTECT SHARED STATE (conditional)                           │
│           Concurrent access → explicit synchronization.                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```


## Ядро: Orchestrator

Сердце neoCoder — **Orchestrator**: цикл взаимодействия LLM ↔ Tools ↔ Memory.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           ORCHESTRATOR                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  while iteration < max_iterations:                              │    │
│  │                                                                 │    │
│  │      # 1. Invoke LLM with memory context                        │    │
│  │      response = llm.invoke(                                     │    │
│  │          messages=memory.get_messages_for_llm(),                │    │
│  │          system=system_prompt,                                  │    │
│  │          tools=extensions.get_tool_definitions()                │    │
│  │      )                                                          │    │
│  │                                                                 │    │
│  │      # 2. Parse response into actions                           │    │
│  │      actions = parser.parse(response)                           │    │
│  │                                                                 │    │
│  │      # 3. Execute actions via extensions                        │    │
│  │      for action in actions:                                     │    │
│  │          result = extensions.execute(action, context)           │    │
│  │          memory.add_tool_result(result)                         │    │
│  │                                                                 │    │
│  │          # Track errors for episodic memory                     │    │
│  │          if not result.success:                                 │    │
│  │              pending_errors.append(result.error)                │    │
│  │              consecutive_errors += 1                            │    │
│  │                                                                 │    │
│  │              # Self-questioning after repeated failures         │    │
│  │              if consecutive_errors >= 2:                        │    │
│  │                  inject_self_questioning_prompt()               │    │
│  │                                                                 │    │
│  │      # 4. Check for context compression                         │    │
│  │      if memory.needs_compression():                             │    │
│  │          summary = compress_context(memory)                     │    │
│  │          cognitive_memory.retain_compression(summary)           │    │
│  │                                                                 │    │
│  │      # 5. Check for task completion                             │    │
│  │      if is_complete(response):                                  │    │
│  │          cognitive_memory.complete_task(task, result, errors)   │    │
│  │          break                                                  │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Асинхронный параллелизм

Независимые read-only операции выполняются **параллельно**:

```python
# Safe to parallelize: different files, read-only
parallel_safe = all(
    action.name in {"file_read", "grep", "glob"}
    for action in actions
)

if parallel_safe:
    results = await asyncio.gather(*[
        execute_action_async(action) for action in actions
    ])
```

---

## Life Cycle: Путь Задачи

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TASK LIFECYCLE                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────┐                                                            │
│  │  USER   │  "Add OAuth2 authentication"                               │
│  └────┬────┘                                                            │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 1: INITIALIZATION                                          ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Adaptive Memory analyzes task → sets parameters           │  ║  │
│  ║  │ • Cognitive Memory retrieves relevant episodes              │  ║  │
│  ║  │ • xMemory provides semantic context                         │  ║  │
│  ║  │ • Session Context restored (if resuming)                    │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 2: PLANNING (Decision Funnel)                              ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Brainstorming: EXPLORE → CHALLENGE → DECIDE → SIMULATE    │  ║  │
│  ║  │ • Plan: Step-by-step with pre_commit_check                  │  ║  │
│  ║  │ • Session Context saved to .neoCoder/context.md             │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 3: EXECUTION (Iterative Loop)                              ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ while not complete:                                         │  ║  │
│  ║  │   • LLM generates next action                               │  ║  │
│  ║  │   • Extension executes (bash, file_edit, grep...)           │  ║  │
│  ║  │   • Working Memory updated                                  │  ║  │
│  ║  │   • Context compressed if needed                            │  ║  │
│  ║  │   • Self-questioning on repeated failures                   │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 4: VERIFICATION                                            ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • pre_commit_check with semi-formal evidence                │  ║  │
│  ║  │ • Build verification                                        │  ║  │
│  ║  │ • Test execution (if applicable)                            │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 5: CONSOLIDATION                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Episode created (task, summary, files, decisions, errors) │  ║  │
│  ║  │ • Hierarchical Memory updated                               │  ║  │
│  ║  │ • xMemory extracts semantic units                           │  ║  │
│  ║  │ • Procedural Memory updated (if pattern detected)           │  ║  │
│  ║  │ • Session Context cleared for next task                     │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  RESULT: OrchestratorResult                                     │    │
│  │  {                                                              │    │
│  │    success: true,                                               │    │
│  │    output: "OAuth2 implemented with JWT tokens",                │    │
│  │    iterations: 47,                                              │    │
│  │    total_tokens: 125000,                                        │    │
│  │    input_tokens: 98000,                                         │    │
│  │    output_tokens: 27000,                                        │    │
│  │    artifacts: { files_changed: [...], tests_passed: true }      │    │
│  │  }                                                              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Метрики: Доказательства Эффективности


---

## Workflow Preset: 8-Stage Engineering Pipeline

Для сложных enterprise-задач:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     WORKFLOW PRESET (8 STAGES)                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1️⃣  PROBLEM BREAKDOWN     │ Decompose into subtasks     │ ❌ Approval  │
│                             │                             │              │
│  2️⃣  BRAINSTORMING         │ High-level approach, risks  │ ❌ Approval  │
│                             │                             │              │
│  3️⃣  IMPLEMENTATION PLAN   │ Detailed step-by-step       │ ❌ Approval  │
│                             │                             │              │
│  4️⃣  IMPLEMENTATION        │ Execute the plan            │ ✅ Auto      │
│                             │                             │              │
│  5️⃣  INTEGRATION TESTING   │ Run tests with retry logic  │ ✅ Auto      │
│                             │                             │              │
│  6️⃣  TECH STACK REVIEW     │ Validate best practices     │ ✅ Auto      │
│                             │                             │              │
│  7️⃣  CODE REVIEW           │ CRIT rules, anti-patterns   │ ✅ Auto      │
│                             │                             │              │
│  8️⃣  MERGE PREPARATION     │ Commit-ready summary        │ ✅ Auto      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Stateful execution:** Workflow сохраняется в `.neoCoder/context.md` — можно прервать и возобновить.

---

# Заключение: Философия neoCoder
neoCoder — это не "ещё один AI-агент". Это воплощение принципа:

Агент должен работать как CHIEF ENGINEER: помнить контекст, думать перед кодом, проверять результат.


### Источники :
Oreilly books : 

Beyond Vibe Coding

Building AI Agents with LLMs, RAG, and Knowledge Graphs

Building Agentic AI - Workflows, Fine-Tuning, Optimization, and Deployment

Agentic Architectural Patterns for Building Multi-Agent Systems

Generative AI Design Patterns

Designing Intelligent Delivery Systems

Your Code as a Crime Scene, Second Edition, 2nd Edition

AI Agents - The Definitive Guide

Advances in Artificial Intelligence Applications in Industrial and Systems Engineering

Enterprise AI in the Cloud

Generative AI Application Integration Patterns

Adaptive Artificial Intelligence



### Arxiv papers:

From Prompt–Response to Goal-Directed Systems:
The Evolution of Agentic AI Software Architecture
Beyond RAG for Agent Memory: Retrieval by Decoupling and Aggregation

MemoryArena: Benchmarking Agent Memory in Interdependent Multi-Session Agentic Tasks

Agentic Code Reasoning

SWE-Compass: Towards Unified Evaluation of Agentic Coding Abilities for Large Language Models

A Practical Guide for Designing, Developing, and Deploying Production-Grade Agentic AI Workflows

AlphaEvolve: A coding agent for scientific and algorithmic discovery

The Potential of CoT for Reasoning: A Closer Look at Trace Dynamics

Team of Thoughts: Efficient Test-time Scaling of Agentic Systems through Orchestrated Tool Calling

Think Fast and Slow: Step-Level Cognitive Depth Adaptation for LLM Agents

AI Agent Systems: Architectures, Applications, and Evaluation

Fundamentals of Building Autonomous LLM Agents

Beyond Task Completion: An Assessment Framework for Evaluating Agentic AI Systems

</div>

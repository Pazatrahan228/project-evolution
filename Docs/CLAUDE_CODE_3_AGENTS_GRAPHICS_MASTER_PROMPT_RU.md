# MASTER PROMPT — Project Evolution / Frontier
## Claude Code: 3 параллельных агента, графика — главный приоритет

Ты работаешь внутри репозитория `Pazatrahan228/project-evolution`.

Твоя задача — не написать очередной концепт и не создать набор пустых файлов. Нужно превратить текущую техническую основу Project Evolution / Frontier в реально запускаемый, красивый и проверяемый браузерный vertical slice стратегии про развитие цивилизации от каменного века к будущему.

## 0. Главный результат

После выполнения задачи пользователь должен иметь возможность:

1. Клонировать репозиторий.
2. Выполнить установку зависимостей одной командой.
3. Запустить игру локально.
4. Открыть игру через GitHub Pages.
5. Создать новый мир по seed.
6. Управлять камерой сверху, выбирать жителей и здания.
7. Добывать еду, дерево и камень.
8. Строить первые здания.
9. Назначать работников.
10. Пережить смену времени суток, погоду и сезоны.
11. Открыть технологии каменного века и перейти хотя бы в ранний бронзовый век.
12. Сохранить игру, обновить страницу и продолжить с того же места.
13. Увидеть визуально сильный мир, который явно следует приложенным пользователем референсам текстур из Higgsfield.

Это должна быть не схема и не мокап, а playable vertical slice продолжительностью примерно 15–30 минут.

---

# 1. Сначала изучи репозиторий

Перед изменениями прочитай:

- `Docs/Game Design Document.md`
- `Docs/Technical Design.md`
- `Docs/AI.md`
- `Docs/Economy.md`
- `Docs/Diplomacy.md`
- `Docs/World Simulation.md`
- `Source/WebClient/src/core/rng.js`
- `Source/WebClient/src/core/world.js`
- `Source/WebClient/src/core/data.js`

Текущие сильные стороны, которые нельзя ломать:

- deterministic seed-based RNG;
- engine-agnostic simulation core;
- data-driven таблицы контента;
- разделение simulation и presentation;
- будущая переносимость логики в Unreal Engine;
- цель создать живой sandbox-мир, а не набор скриптованных миссий.

Текущие ограничения, которые нужно исправить:

- в корне нет полноценного `index.html` и нет готового запускаемого web client;
- нет package/build pipeline;
- нет рендера, ассетов, моделей, материалов, UI и игрового цикла;
- README пустой;
- карта ограничена простым островом 96×96;
- нет настоящих рек, климатических зон, дорог и разнообразных биомов;
- движение использует примитивный обход воды вместо нормального pathfinding;
- глобальный `animalSeq` может ломать детерминизм ID после повторной генерации мира;
- OBJECTIVES содержат функции прямо в таблице данных и плохо переносятся в JSON/DataAssets;
- отсутствуют schema validation, версии сохранений, replay tests и автоматические проверки;
- нет связанной симуляции строительства, логистики, здоровья, морали и работы жителей;
- заявленный масштаб GDD значительно больше текущего кода.

Не пытайся реализовать все девять эпох за один проход. Создай сильную архитектуру и качественный vertical slice: каменный век → ранний бронзовый век. Остальные эпохи должны расширяться данными без переписывания ядра.

---

# 2. Визуальные референсы Higgsfield — источник истины

Пользователь уже приложил Claude референсы текстур и визуального качества из Higgsfield. Сначала внимательно проанализируй их.

Создай файл:

`Docs/ArtDirection_Higgsfield.md`

В нём зафиксируй:

- общую палитру;
- уровень реализма или стилизации;
- характер поверхностей;
- контраст;
- масштаб деталей;
- тип освещения;
- воздушную перспективу;
- характер земли, травы, камня, древесины, воды и построек;
- правила читаемости объектов с высоты игровой камеры;
- что именно нельзя упрощать.

Референсы используются как visual target, а не как файлы для прямого копирования. Не копируй логотипы, уникальные защищённые элементы и чужие ассеты. Создавай оригинальные материалы и геометрию, совпадающие по уровню качества, настроению и визуальному языку.

Если изображения доступны в текущем контексте Claude, используй их непосредственно. Если они не представлены файлами в workspace, не выдумывай их содержание. Создай инфраструктуру рендера и временные оригинальные procedural/CC0 assets, пометь точные слоты замены и продолжи всю остальную работу. Не останавливай проект целиком из-за одного отсутствующего файла.

## Графика — не «декорация после геймплея»

Графическая система должна проектироваться одновременно с симуляцией. Каждое важное состояние должно быть видно без чтения таблиц:

- голодные жители визуально замедляются и меняют поведение;
- зима меняет ландшафт, растительность и освещение;
- дождь намокает поверхности и создаёт лужи;
- вырубка леса физически меняет карту;
- стройка проходит через несколько видимых стадий;
- здания получают следы использования и повреждений;
- технологии меняют внешний вид поселения;
- ресурсы физически перемещаются или хотя бы имеют ясную визуальную репрезентацию;
- день и ночь меняют активность, огни и читаемость поселения.

Запрещено выдавать за финальную графику:

- одноцветные кубы;
- плоские тайлы без смешивания материалов;
- случайные placeholder-иконки;
- кислотные цвета без связи с референсами;
- одинаковые деревья, повернутые в одну сторону;
- резкие квадратные границы биомов;
- UI из стандартных HTML-кнопок без арт-направления;
- визуальные эффекты, которые скрывают слабое качество базовых материалов.

---

# 3. Организация 3 агентов

Запусти ровно три специализированных агента. У каждого должен быть отдельный scope, отдельная ветка или worktree и список файлов, которыми он владеет. Не допускай одновременного редактирования одного файла несколькими агентами.

Главный оркестратор:

- составляет план;
- создаёт задачи;
- контролирует интерфейсы между агентами;
- регулярно интегрирует изменения;
- запускает тесты после каждого merge;
- не переписывает рабочий код без причины;
- в конце выполняет общий performance и quality pass.

## Агент 1 — GRAPHICS / WORLD PRESENTATION LEAD

### Миссия

Создать впечатляющий визуальный слой игры с приоритетом на приложенные Higgsfield-референсы. Игра должна выглядеть как современная самостоятельная стратегия, а не учебный Three.js-проект.

### Ответственность

- renderer;
- камера;
- terrain rendering;
- материалы и текстуры;
- вода;
- растительность;
- освещение;
- погода;
- день/ночь;
- VFX;
- визуальные стадии зданий;
- визуализация юнитов и ресурсов;
- LOD, instancing и GPU performance;
- art direction documentation.

### Рекомендуемый web stack

Используй `Vite + Three.js` с чистым JavaScript или TypeScript. TypeScript предпочтителен, если миграция не ломает текущий core и проводится аккуратно. Не добавляй тяжёлый framework без необходимости.

Настрой:

- WebGL2;
- корректный sRGB color space;
- ACES Filmic tone mapping;
- physically correct lighting;
- shadow maps с ограниченным бюджетом;
- post-processing только после стабильной базовой картинки;
- adaptive quality presets;
- graceful fallback для слабых мобильных устройств.

### Terrain

Создай chunked terrain renderer, который читает данные engine-agnostic world core.

Нужно:

- плавная высота вместо полностью плоских тайлов;
- material blending между травой, землёй, песком, скалой и снегом;
- slope-aware rock material;
- triplanar mapping или другой способ убрать растяжения текстуры на склонах;
- detail normals;
- macro variation, чтобы большие участки не повторялись;
- decals для троп, грязи, стройки и вырубки;
- береговая линия без жёстких квадратов;
- туман и воздушная перспектива;
- видимая разница влажности и сезона.

Не создавай mesh на каждый тайл. Используй чанки, общие buffers, geometry batching и разумные обновления.

### Растительность и props

- деревья, кусты, трава, камни и мелкие props через instancing;
- минимум 4–8 визуальных вариантов каждого массового объекта;
- случайный scale/rotation в пределах арт-направления;
- разные наборы по биомам;
- culling и LOD;
- реакции на вырубку, пожар, снег и ветер;
- читаемые resource nodes.

### Вода

- shoreline foam;
- мягкое отражение неба;
- глубина через цвет;
- небольшая анимация волн;
- реки и озёра должны визуально отличаться от океана;
- без чрезмерно дорогих full-screen эффектов.

### Здания

Сделай модульный Stone Age building kit:

- кострище;
- хижина;
- стоянка собирателей;
- лесозаготовка;
- каменоломня;
- ферма;
- амбар;
- ранняя шахта;
- ранняя библиотека/дом знаний;

Для каждого:

- foundation/ghost preview;
- 3–4 стадии строительства;
- готовое состояние;
- повреждённое состояние;
- snow/wetness variation;
- ясный силуэт сверху;
- scale consistency.

### Юниты

Не требуется AAA character pipeline, но персонажи должны быть узнаваемыми с игровой высоты.

- несколько silhouette archetypes;
- простые анимации idle/walk/work/carry;
- инструменты по профессии;
- цветовые элементы поселения без кислотных team colors;
- selection ring, hover highlight и status indicators;
- не использовать одинаковую T-pose модель.

### Критерии агента 1

- мир выглядит цельно на скриншоте без UI;
- приближение камеры не разрушает иллюзию;
- на среднем desktop 60 FPS при сотнях props и десятках юнитов;
- на iPhone/слабом устройстве есть reduced preset;
- никакие текстуры не имеют видимых швов или грубого повторения на обычной высоте камеры;
- визуальный результат документирован сравнением с Higgsfield-референсами.

---

## Агент 2 — SIMULATION / GAMEPLAY LEAD

### Миссия

Превратить имеющиеся `rng.js`, `world.js`, `data.js` в детерминированный, тестируемый и реально интересный gameplay core.

### Ответственность

- world state;
- fixed timestep;
- villagers;
- jobs;
- resource economy;
- buildings;
- construction;
- technology;
- population;
- health/hunger/morale;
- events;
- seasons/weather effects;
- pathfinding;
- save/load/replay;
- tests.

### Обязательные исправления ядра

1. Убери глобальный nondeterministic ID state. Все entity IDs должны генерироваться внутри world/simulation state и одинаково воспроизводиться по seed и последовательности команд.
2. Введи simulation clock с fixed timestep. Rendering FPS не должен менять экономику.
3. Отдели определения данных от исполняемых функций. Objective conditions должны описываться сериализуемыми правилами или registry IDs.
4. Добавь versioned save schema и миграции хотя бы с `saveVersion: 1`.
5. Добавь deterministic command log для replay/debug.
6. Добавь validation входных data definitions.
7. Не используй `Math.random()` в симуляции.
8. Не привязывай simulation core к DOM или Three.js.

### World generation v2

Расширь генерацию, сохраняя deterministic seed:

- elevation;
- moisture;
- temperature;
- latitude effect;
- минимум 6–8 биомов;
- rivers или хотя бы deterministic river network;
- lakes;
- resource deposits;
- стартовая пригодность;
- несколько возможных стартовых регионов;
- reproducible world checksum.

Не нужно сразу делать планету. Vertical slice может использовать одну крупную региональную карту, но структура должна позволять чанки и больший мир.

### Pathfinding

Текущий `stepToward` недостаточен.

Реализуй:

- walkability и terrain costs;
- A* для отдельных маршрутов или flow fields для групп;
- обход зданий, воды и непроходимых склонов;
- path cache;
- ограничение вычислительного бюджета;
- fallback при изменении карты;
- тесты на узких проходах и заблокированной цели.

### Жители

Каждый житель vertical slice должен иметь:

- имя;
- возраст;
- здоровье;
- голод;
- усталость;
- мораль;
- дом;
- профессию;
- текущую задачу;
- инвентарь или carry slot;
- базовые черты личности;
- простую память о важных событиях.

Не создавай сложный GOAP ради названия. Для vertical slice используй прозрачную Utility AI + task queue, которую легко отлаживать. Архитектурный интерфейс оставь расширяемым под GOAP/HTN.

### Экономика

Реализуй реальный цикл:

- собиратель/охотник добывает еду;
- лесоруб добывает дерево;
- каменщик добывает камень;
- ресурс переносится к storage;
- строительство потребляет физически доступные ресурсы;
- жители едят;
- нехватка еды вызывает ухудшение здоровья и морали;
- жильё влияет на рождаемость/миграцию;
- погода и сезон меняют производительность;
- технологии меняют формулы через data modifiers.

### Стройка

- placement validation;
- ghost preview data API для renderer;
- резерв ресурсов;
- доставка материалов;
- progress;
- несколько construction stages;
- отмена и частичный возврат;
- разрушение/ремонт;
- building footprint и блокировка pathfinding.

### Технологии

Для vertical slice сделай небольшое, но разветвлённое дерево:

- огонь;
- орудия;
- охота;
- земледелие;
- гончарство;
- обработка камня;
- бронза;
- письменность или дом знаний.

Технологии не должны быть просто таймером. Знания производятся жителями и связаны с наблюдениями/работами, но можно использовать упрощённую модель с ясными расширяемыми hooks.

### Критерии агента 2

- одинаковый seed + одинаковый command log = одинаковый checksum состояния;
- 30 минут симуляции без NaN, отрицательных ресурсов и зависших задач;
- сохранение/загрузка возвращает эквивалентное состояние;
- pathfinding не ведёт через воду и здания;
- базовая экономика может как выжить, так и рухнуть по понятным причинам;
- все ключевые системы имеют unit tests.

---

## Агент 3 — GAME UX / INTEGRATION / QA LEAD

### Миссия

Собрать графику и симуляцию в понятную игру, подготовить GitHub Pages, UI, onboarding, производительность и автоматические проверки.

### Ответственность

- app shell;
- input;
- camera UX integration;
- HUD;
- panels;
- build mode;
- notifications;
- tutorial;
- save slots;
- settings;
- accessibility;
- responsive layout;
- GitHub Pages deployment;
- integration tests;
- performance HUD;
- documentation.

### Запуск проекта

В корне должны появиться:

- `index.html`;
- `package.json`;
- Vite config;
- scripts `dev`, `build`, `test`, `preview`, `lint`;
- корректные relative asset paths для GitHub Pages;
- GitHub Actions workflow для tests/build/pages;
- production build без ручного копирования файлов.

Команды должны быть простыми:

```bash
npm install
npm run dev
npm test
npm run build
```

### Управление

Desktop:

- WASD/edge pan;
- mouse drag pan;
- wheel zoom;
- rotate camera;
- click select;
- box select при необходимости;
- Escape отменяет режим;
- горячие клавиши скорости и паузы.

Mobile/iPhone:

- one-finger pan;
- pinch zoom;
- tap select;
- long press context;
- крупные touch targets;
- UI не закрывает центр карты;
- reduced graphics preset.

### HUD

Показывай минимум:

- еда;
- дерево;
- камень;
- знания;
- население;
- жильё;
- сезон;
- погоду;
- время суток;
- скорость симуляции;
- выбранный объект;
- текущие цели;
- предупреждения.

HUD должен соответствовать art direction и оставаться читаемым на сложном фоне. Используй spacing, hierarchy, icons и tooltips. Не превращай экран в Excel.

### Build mode

- каталог зданий;
- фильтры;
- cost;
- requirements;
- ghost preview;
- green/red placement;
- footprint;
- rotation, если поддерживается;
- cancel;
- пояснение причины невозможности строительства.

### Onboarding

Создай короткий tutorial без длинных модальных окон:

1. Осмотреть поселение.
2. Построить хижину.
3. Назначить собирателей.
4. Создать запас дерева.
5. Открыть орудия.
6. Построить ферму.
7. Пережить первую зиму.
8. Начать переход к бронзе.

### Отладка

Добавь developer overlay, отключённый по умолчанию:

- FPS;
- draw calls;
- triangles;
- visible/total entities;
- simulation tick time;
- pathfinding queue;
- seed;
- state checksum;
- current chunk;
- memory estimate.

Добавь console commands из GDD в разумном минимальном объёме:

- `/give`;
- `/weather`;
- `/age`;
- `/spawn`;
- `/simulate`;
- `/godmode`.

### QA

- smoke test: новая игра → здание → сохранение → reload;
- Playwright или эквивалент для основных UI-flow;
- unit tests core;
- build test;
- проверка GitHub Pages base path;
- тесты desktop и mobile viewport;
- отсутствие ошибок console;
- no broken asset URLs;
- no unhandled promise rejections.

### Документация

Заполни README:

- что это за игра;
- текущий playable scope;
- скриншот;
- controls;
- установка;
- GitHub Pages link;
- архитектура;
- где лежат данные;
- как добавить building/technology/biome;
- известные ограничения;
- roadmap.

### Критерии агента 3

- новый человек запускает проект по README;
- игра открывается на GitHub Pages;
- нет абсолютных локальных путей;
- управление понятно без объяснений разработчика;
- UI работает в desktop и iPhone viewport;
- production build проходит в CI.

---

# 4. Общие архитектурные правила

## Структура

Предлагаемая структура, которую можно скорректировать после аудита:

```text
/
  index.html
  package.json
  vite.config.*
  src/
    app/
    core/
    data/
    render/
    gameplay/
    ui/
    input/
    audio/
    debug/
  public/
    assets/
      textures/
      models/
      audio/
      icons/
  Docs/
  Tests/
  .github/workflows/
```

Сохрани разделение:

`Simulation Core -> Presentation adapters -> Renderer/UI`

Renderer читает snapshots/events симуляции. Core не импортирует renderer.

## Data-driven

Здания, технологии, биомы, погода, события и цели должны находиться в сериализуемых data definitions.

Нужно:

- stable IDs;
- schema version;
- validation;
- defaults;
- error messages;
- mod-friendly structure;
- отсутствие UI-текста внутри системной логики, где это возможно.

## Event bus

Используй typed/structured events:

- entityCreated;
- entityRemoved;
- resourceChanged;
- constructionStarted;
- constructionProgress;
- constructionCompleted;
- weatherChanged;
- seasonChanged;
- technologyUnlocked;
- villagerDied;
- objectiveCompleted.

Не делай глобальный хаотичный emitter без жизненного цикла и отписки.

## Performance

Целевые бюджеты vertical slice:

- 60 FPS desktop medium preset;
- 30 FPS mobile reduced preset;
- simulation tick не зависит от render FPS;
- terrain и массовые props используют batching/instancing;
- не создавать тысячи DOM-элементов для world labels;
- не пересчитывать весь мир каждый кадр;
- pathfinding имеет frame budget;
- object pooling для частых временных VFX;
- performance metrics доступны в debug overlay.

## Git

Каждый агент:

- работает в своей ветке/worktree;
- делает небольшие осмысленные commits;
- не коммитит `node_modules`, build output и временные файлы;
- перед merge синхронизируется с main/integration;
- пишет summary и список изменённых interfaces.

Главный оркестратор:

- создаёт integration branch;
- мержит по одному агенту;
- после каждого merge запускает tests/build;
- исправляет integration conflicts с минимальным вмешательством;
- финальный merge выполняет только после общего smoke test.

---

# 5. Порядок выполнения

## Phase 1 — Audit and contracts

1. Зафиксировать текущее состояние.
2. Создать architecture note.
3. Определить interfaces WorldSnapshot, RenderEntity, Command, SimulationEvent, SaveGame.
4. Определить ownership файлов трёх агентов.
5. Создать минимальный Vite shell.
6. Настроить tests/build.

Результат: проект запускается, но ещё может выглядеть просто.

## Phase 2 — Vertical slice foundation

- world generation v2;
- fixed tick simulation;
- renderer terrain/camera;
- resource nodes;
- villagers;
- selection;
- HUD;
- building placement;
- save/load.

Результат: базовый игровой цикл работает.

## Phase 3 — Graphics-first quality pass

- заменить временные материалы на Higgsfield-aligned materials;
- terrain blending;
- vegetation variation;
- water;
- lighting;
- day/night;
- weather;
- construction stages;
- unit animation;
- UI art pass;
- screenshot comparison.

Результат: игра выглядит презентабельно.

## Phase 4 — Gameplay depth

- seasons;
- hunger/health/morale;
- technology tree;
- early bronze transition;
- events;
- tutorial objectives;
- failure/recovery loops.

## Phase 5 — Optimization and release

- desktop/mobile presets;
- profiling;
- CI;
- GitHub Pages;
- README;
- regression tests;
- final report.

---

# 6. Definition of Done

Задача считается выполненной только когда:

- существует playable build;
- GitHub Pages deployment подготовлен;
- мир генерируется по seed;
- можно строить, добывать, назначать работников и исследовать;
- присутствует смена времени, погоды и сезона;
- есть save/load;
- есть минимум один fail state и один milestone перехода эпохи;
- графика соответствует зафиксированному анализу Higgsfield-референсов;
- нет финальных cube/placeholders в основном игровом кадре;
- README заполнен;
- tests проходят;
- production build проходит;
- отсутствуют console errors;
- выполнен performance report;
- изменения разбиты на понятные commits;
- финальный отчёт содержит факты, а не обещания.

---

# 7. Формат финального отчёта Claude Code

В конце выдай:

1. Что было в репозитории до начала.
2. Что реализовал каждый из трёх агентов.
3. Какие ветки и commits созданы.
4. Какие файлы добавлены и изменены.
5. Как запустить локально.
6. Как открыть GitHub Pages.
7. Какие тесты выполнены и их результат.
8. Desktop и mobile performance.
9. Какие visual decisions взяты из Higgsfield-референсов.
10. Какие placeholders ещё остались и почему.
11. Какие 10 задач являются следующими по приоритету.
12. Известные баги и риски.

Не пиши «готово», если нет запускаемого билда. Не заявляй о визуальном соответствии референсам без проверки скриншотов. Не маскируй отсутствие систем красивыми markdown-документами.

# Финальная команда

Начни с аудита файлов, затем немедленно создай план и запусти три агента параллельно. Основной приоритет — графика и ощущение живого мира, но не ценой разрушения детерминированной симуляции. Каждый этап должен завершаться работающим билдом, тестами и конкретным визуальным улучшением.
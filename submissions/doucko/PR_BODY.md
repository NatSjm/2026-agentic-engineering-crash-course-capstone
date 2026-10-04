## Ім'я

<!-- ВПИШИ СПРАВЖНЄ ІМ'Я ДЛЯ СЕРТИФІКАТА -->

## Проєкт

**Doučko** — домашній репетитор з математики для дитини в 6 класі чеської школи. Вона розв'язує задачі
з робочого зошита свого класу: сама, з батьком онлайн (він в Україні й читає український переклад поруч
із чеським текстом) або з батьком поруч. Відповіді перевіряються автоматично і не розкривають ключ.
Зараз це MVP без AI: задачі беруться з бази, куди дорослий вносить їх із завантажених сторінок зошита.
AI додам після MVP.

**Де код:** три окремі репозиторії (multi-repo + OpenSpec store, бета):
- [NatSjm/doucko-docs](https://github.com/NatSjm/doucko-docs): вимоги, specs, контракт API, ADR, план, харнес фабрики
- [NatSjm/doucko-backend](https://github.com/NatSjm/doucko-backend): Kotlin · Spring Boot 4.1 · Postgres
- [NatSjm/doucko-frontend](https://github.com/NatSjm/doucko-frontend): React · Vite · TypeScript

У цій гілці лише [`submissions/doucko/README.md`](submissions/doucko/README.md) з картою репозиторіїв,
поточним станом і командами запуску.

**Стан:** зі слайсів MVP готові 1–5 (фундамент, вхід адміна, перевірка відповідей, профіль дитини,
внесення задач). Слайси 6–9 (місії, сам процес практики, гейміфікація, статистика) ще попереду. Продовжую
в тих самих репозиторіях.

## Відео-демо (1–2 хв)

**Посилання:** _додам найближчим часом — відео записується_

## Застосовані практики Agentic Engineering

Скорочення: **D** = doucko-docs, **B** = doucko-backend, **F** = doucko-frontend.

- [x] **Project Factory**. Узяла за основу [koldovsky/project-factory](https://github.com/koldovsky/project-factory)
  @ `0d7286a1` і адаптувала під multi-repo з OpenSpec stores. Що взято, що змінено руками і що відкинуто,
  описано в [D/factory/UPSTREAM.md](https://github.com/NatSjm/doucko-docs/blob/main/factory/UPSTREAM.md).
  Хеш-лок харнесу: [factory-lock.json](https://github.com/NatSjm/doucko-docs/blob/main/factory-lock.json).
  Встановлення: [D fa9674d](https://github.com/NatSjm/doucko-docs/commit/fa9674d), [B 38b56c8](https://github.com/NatSjm/doucko-backend/commit/38b56c8), [F ab84763](https://github.com/NatSjm/doucko-frontend/commit/ab84763).
  Руками додано: `factory.config.json` для трьох репо, `check-workspace.mjs` (traceability між репо),
  читач JaCoCo для coverage-ratchet на Maven, `mvnw.mjs`. Сам харнес я лагодила протягом
  проєкту, коли він помилявся (PD-1…PD-15, див. нижче).

- [x] **Контекст-інженерія**.
  - **Статичний контекст.** `CLAUDE.md` = `@AGENTS.md` у кожному репо: [D](https://github.com/NatSjm/doucko-docs/blob/main/AGENTS.md),
    [B](https://github.com/NatSjm/doucko-backend/blob/main/AGENTS.md), [F](https://github.com/NatSjm/doucko-frontend/blob/main/AGENTS.md).
    Що в них: протокол handoff (спершу прочитати `docs/current-state.md`, план і spec слайса); правила
    коректності домену (жодного сирого 500, ключ відповіді не витікає на клієнт, UNPARSEABLE не забирає
    спробу, XP лише додається); test-first з `// @trace FR-x`; заборона комітити дані, що ідентифікують
    дитину (NFR-3), і вміст зошита (NFR-4); блок уроків фабрики (vacuous pass ≠ PASS тощо).
  - **Динамічний контекст.** PostToolUse-hook на `Edit|Write` ([B/.claude/settings.json](https://github.com/NatSjm/doucko-backend/blob/main/.claude/settings.json),
    [hooks-post-edit.mjs](https://github.com/NatSjm/doucko-backend/blob/main/scripts/hooks-post-edit.mjs)) запускає
    ktlint/ESLint на змінений файл і повертає exit 2, тож агент виправляє помилку в тому ж циклі.
    Pre-commit ([hooks-pre-commit.mjs](https://github.com/NatSjm/doucko-backend/blob/main/scripts/hooks-pre-commit.mjs)) ловить секрети
    й порушення NFR-3/NFR-4 і перегенеровує trace-звіти. Handoff між вікнами агента йде через
    [current-state.md](https://github.com/NatSjm/doucko-docs/blob/main/docs/current-state.md).
  - **Правило змінилося, агент поводиться інакше.** Виправлення фіксую в `retro/corrections` і реєстрі
    [process-defects.json](https://github.com/NatSjm/doucko-docs/blob/main/docs/qa/process-defects.json):
    - **COR-1 → PD-1.** В агентських шелах `mvnw.cmd` не запускався (`NoDefaultCurrentDirectoryInExePath`).
      [correction](https://github.com/NatSjm/doucko-docs/blob/main/retro/corrections/001-harness-maven-launcher-fails-in-agent-shells-wit.correction.json) →
      фікс [D f3ece43](https://github.com/NatSjm/doucko-docs/commit/f3ece43) → перевстановлення [B 8dd1ec3](https://github.com/NatSjm/doucko-backend/commit/8dd1ec3) →
      [proof](https://github.com/NatSjm/doucko-docs/blob/main/docs/qa/evidence/pd-mvnw-shim-proof.md).
    - **COR-2 → PD-8.** `gate-status` показував G2 PASS, хоча specs ще не було жодного (vacuous pass).
      Фікс [D edc3f19](https://github.com/NatSjm/doucko-docs/commit/edc3f19), [proof](https://github.com/NatSjm/doucko-docs/blob/main/docs/qa/evidence/pd-8-g2-vacuous-pass-proof.md).
    - **PD-15.** Планка «нуль знахідок рев'ю» не сходилася: фронтенд слайсу 1 пройшов 10 раундів рев'ю.
      Я змінила правило на «major виправлені, решта з письмовими disposition»
      ([D 5a7c4f9](https://github.com/NatSjm/doucko-docs/commit/5a7c4f9)). Після цього кожен зі слайсів 2–5 закрився
      одним раундом рев'ю і одним fix-комітом.
  - **Чесно:** самі `AGENTS.md` після встановлення майже не змінювалися. Правила еволюціонували через
    скрипти фабрики й реєстр PD, а не через текст AGENTS.md.

- [x] **Цикли (loop engineering)**. Свій раннер workflow [D/scripts/run-workflow.mjs](https://github.com/NatSjm/doucko-docs/blob/main/scripts/run-workflow.mjs)
  ([e974fc1](https://github.com/NatSjm/doucko-docs/commit/e974fc1)) запускає headless-кроки `claude -p` з лімітами
  `--max-budget-usd` на агента і `--max-total-usd` на прогін. Workflow:
  [spec-pipeline](https://github.com/NatSjm/doucko-docs/blob/main/.claude/workflows/spec-pipeline.js),
  [spec-crosscheck](https://github.com/NatSjm/doucko-docs/blob/main/.claude/workflows/spec-crosscheck.js),
  [contract-review](https://github.com/NatSjm/doucko-docs/blob/main/.claude/workflows/contract-review.js),
  [review-gate](https://github.com/NatSjm/doucko-backend/blob/main/.claude/workflows/review-gate.js) (find → adversarial verify → report → persist).
  Кожен прогін зберігається в `trace/workflow-runs/*.json` з кроками і вартістю.
  - spec-pipeline: 22 кроки, $13.06, переписав 7 baseline specs ([прогін](https://github.com/NatSjm/doucko-docs/blob/main/trace/workflow-runs/20261003-043657-spec-pipeline.json)).
  - Цикл рев'ю контракту API: **57 → 31 → 20 → 18** знахідок за 4 раунди
    ([раунд 1](https://github.com/NatSjm/doucko-docs/blob/main/trace/workflow-runs/20261003-053111-contract-review.json)).
    У раунді 3 знайшовся blocker: `uniqueItems: true` змушував генератор видати невпорядкований `Set`
    для впорядкованого списку задач місії. **Зупинилась** на v0.5, яку ще не пройшла чисто; це записано в
    [review-findings.json](https://github.com/NatSjm/doucko-docs/blob/main/openspec/changes/archive/2026-10-03-add-api-contract-v0/review-findings.json),
    повторне рев'ю буде перед G7.
  - Що в циклах ламалося (усе є в run-файлах фронтенду): відповідь без JSON за схемою (→ PD-13,
    [D d89b46e](https://github.com/NatSjm/doucko-docs/commit/d89b46e)), timeout 20 хв, зупинка на бюджеті
    `--max-total-usd 10 reached ($10.04)`, ліміт сесії. Крок persist мав лише read-only інструменти й
    звітував «ok», нічого не записавши (→ PD-14).
  - Загальна вартість прогонів: docs $31.72, backend $52.08, frontend $77.63.

- [x] **Верифікація**. Одна команда в кожному репо: `npm run qa:verify` (+ `npm run gate:status`).
  - Backend: 548 unit + 44 integration, Overall: Pass. [Звіт](https://github.com/NatSjm/doucko-backend/blob/main/docs/qa/automated-verification-latest.md).
  - Frontend: lint, typecheck, 347 unit (coverage 100%), 219 e2e у chromium/msedge/firefox, build, Overall: Pass.
    eval-ratchet поки PLANNED. [Звіт](https://github.com/NatSjm/doucko-frontend/blob/main/docs/qa/automated-verification-latest.md).
  - **Червоне → зелене:** у [B slice3-red-run](https://github.com/NatSjm/doucko-backend/blob/main/docs/qa/evidence/slice3-red-run.md)
    224 тести проти стабів, 219 червоних; у [F slice5-red-run](https://github.com/NatSjm/doucko-frontend/blob/main/docs/qa/evidence/slice5-red-run.md)
    143 з 305 червоних, після реалізації 347/347 зелених; у [B slice5-red-run](https://github.com/NatSjm/doucko-backend/blob/main/docs/qa/evidence/slice5-red-run.md)
    99 червоних з 537.
  - Golden table для перевірки відповідей: 212 кейсів ([B 0120326](https://github.com/NatSjm/doucko-backend/commit/0120326)).
    Рев'ю додало ще 5, разом 217 ([B f26150d](https://github.com/NatSjm/doucko-backend/commit/f26150d)),
    [таблиця](https://github.com/NatSjm/doucko-backend/blob/main/src/test/resources/checking/golden-table.jsonl).
  - Coverage-ratchet: [baseline](https://github.com/NatSjm/doucko-backend/blob/main/quality/coverage-baseline.json)
    піднімається лише вгору (lines 76.9 → 98.97).
  - **Чесно:** red-прогони задокументовані як файли-докази, а не окремими test-коміти перед feat-комітом.
    RED-прогін слайсу 1 фронтенду не записала, тому оформила
    [waiver](https://github.com/NatSjm/doucko-frontend/blob/main/docs/qa/waivers/slice1-task-2.8-red-run.md),
    а не відтворювала його заднім числом.

- [x] **maker ≠ checker**. Субагенти-рецензенти [code-reviewer](https://github.com/NatSjm/doucko-backend/blob/main/.claude/agents/code-reviewer.md),
  [security-reviewer](https://github.com/NatSjm/doucko-backend/blob/main/.claude/agents/security-reviewer.md),
  [spec-compliance-auditor](https://github.com/NatSjm/doucko-backend/blob/main/.claude/agents/spec-compliance-auditor.md),
  на фронті ще `vision-judge` і `eval-judge`. Кожну знахідку перевіряє окремий verifier. Що вони знайшли:
  - **Слайс 3, answer-key oracle.** Приклад формату відповіді в підказці міг показати правильне значення (FR-27).
    Виправлено в [B f26150d](https://github.com/NatSjm/doucko-backend/commit/f26150d) і [B 6343ca7](https://github.com/NatSjm/doucko-backend/commit/6343ca7).
  - **Слайс 5 бекенд, critical.** Неякірне правило `.gitignore` `storage/` ховало єдиний адаптер FileStoragePort,
    тож з чистого checkout застосунок не стартував. Задача з позначкою «qa:verify green» на закомміченому
    дереві не відтворювалась. Виправлено в [B 3738bfe](https://github.com/NatSjm/doucko-backend/commit/3738bfe).
  - **Слайс 5 фронтенд, 6 major.** Наприклад, після видалення чеської підказки переклади чіплялися не до тих
    підказок, а bulk publish публікував задачі, приховані фільтром
    ([findings](https://github.com/NatSjm/doucko-frontend/blob/main/openspec/changes/archive/2026-10-04-implement-authoring-tasks-ui/review-findings.json)).
    Виправлено в [F 45350dc](https://github.com/NatSjm/doucko-frontend/commit/45350dc), 5 minor пішли у waiver з причинами.
  - **Слайс 4.** Compose smoke був позначений як виконаний, хоча його пропустили. Рецензент це зловив,
    у [B de80c7a](https://github.com/NatSjm/doucko-backend/commit/de80c7a) записано NOT RUN, а реальний smoke пройшов
    після merge ([B 34991f4](https://github.com/NatSjm/doucko-backend/commit/34991f4)).
  - **Слайс 2.** Сесія, інвалідована паралельним запитом, давала сирий 500. Виправлено в [B 513a886](https://github.com/NatSjm/doucko-backend/commit/513a886).

- [x] **Специфікації наперед (SDD)**. OpenSpec у мультирепо-режимі (stores, бета): store `doucko`
  живе в docs, а код-репо посилаються на нього read-only. Порядок комітів:
  specs [D 896fc0a](https://github.com/NatSjm/doucko-docs/commit/896fc0a) (03.10 05:44) →
  контракт v0 [D e112add](https://github.com/NatSjm/doucko-docs/commit/e112add) → 4 раунди рев'ю →
  план слайсів [D 38527fb](https://github.com/NatSjm/doucko-docs/commit/38527fb) →
  checkpoint 2 [D b619b0f](https://github.com/NatSjm/doucko-docs/commit/b619b0f) →
  перший код: [F 7f06645](https://github.com/NatSjm/doucko-frontend/commit/7f06645) (03.10 20:12), [B 69ea343](https://github.com/NatSjm/doucko-backend/commit/69ea343) (04.10).
  - **Специфікацію змінила реальність.** Перший запуск генератора на контракті показав, що 3 описи
    в YAML flow-mapping розсипалися на зайві ключі. Звідси контракт 0.5.1
    ([D 4c87d67](https://github.com/NatSjm/doucko-docs/commit/4c87d67),
    [proposal](https://github.com/NatSjm/doucko-docs/blob/main/openspec/changes/archive/2026-10-03-fix-contract-flow-descriptions/proposal.md)).
    Знахідка рев'ю слайсу 2 перенесла один сценарій у DoD слайсу 7 ([D b6dd69a](https://github.com/NatSjm/doucko-docs/commit/b6dd69a)).
  - **Чесно:** `proposal/design/tasks` для кожного слайсу комітились разом із кодом слайсу, а не окремим
    комітом перед ним.

- [ ] **Журнал рівнів довіри**: окремо не вела. Найближче до нього — реєстр PD і waivers.

## Інструменти та MCP

- **Claude Code** (desktop app, Code tab), моделі Claude Opus. Headless `claude -p` у власному
  раннері workflow.
- **OpenSpec 1.13** (CLI + skills `openspec-propose/apply/archive/...`, мультирепо stores, бета).
- **Субагенти:** requirements-analyst, spec-writer, capability-implementer, test-engineer, code-reviewer,
  security-reviewer, spec-compliance-auditor, qa-documenter, vision-judge, eval-judge, process-auditor.
- **Hooks:** PostToolUse lint, git pre-commit/commit-msg (секрети, NFR-3/4, трейлер `Slice:`).
- **Перевірки:** Vitest, Playwright (+ axe для a11y), JUnit/Testcontainers, JaCoCo, ktlint, ESLint.
- MCP-сервери в розробці не використовувала.

## Що вирішувала я, а що агент

**Вирішувала я:**
- Продукт і обсяг: для кого застосунок, три режими (сама / батько онлайн з українським перекладом / батько поруч),
  MVP без AI, приватність дитини і заборона комітити вміст зошита.
- Стек і архітектуру: Kotlin + Spring Boot, React + Vite, три репозиторії зі спільним контрактом
  ([ADR-0001](https://github.com/NatSjm/doucko-docs/blob/main/docs/adr/0001-stack-kotlin-maven-react-vite.md),
  [ADR-0002](https://github.com/NatSjm/doucko-docs/blob/main/docs/adr/0002-multi-repo-openspec-store-and-factory-harness.md)).
  Spring Boot 4.1 замість 3.x, бо підтримка 3.x скінчилася ([ADR-0003](https://github.com/NatSjm/doucko-docs/blob/main/docs/adr/0003-spring-boot-4-and-scaffold-toolchain.md)).
- Чекпоінти: scope ([checkpoint 1](https://github.com/NatSjm/doucko-docs/blob/main/docs/qa/checkpoint-1-scope-signoff.md)),
  план ([checkpoint 2](https://github.com/NatSjm/doucko-docs/blob/main/docs/qa/checkpoint-2-plan-signoff.md):
  спершу «approve with changes», потім прийняла CA-D22), рішення по відкритих питаннях у
  [requirements.md §6.5–6.7](https://github.com/NatSjm/doucko-docs/blob/main/docs/requirements.md).
- Коли харнес помилявся, вирішувала, лагодити фабрику чи обходити: PD-9 «fix the harness»,
  PD-14 «fix in factory», нова планка рев'ю PD-15. Свідомо не перезапускала рев'ю після 3-го раунду
  фіксів на бекенді й записала це.

**Робив агент:** вимоги з мого PRD, specs, контракт OpenAPI, план слайсів, тести (спершу червоні), реалізацію,
рев'ю окремими агентами, виправлення за рев'ю, звіти QA і handoff у `current-state.md`.

**Де я втручалась і що пішло не так:**
- Фронтенд слайсу 1 крутився 10 раундів рев'ю (major: 7 → 11 → 7 → 4 → 1 → 2 → 4 → 4 → 0 → 0), прогони падали
  на бюджеті й таймаутах. Я зупинила цикл і змінила критерій виходу (PD-15).
- Агент позначав задачі як виконані без доказу (smoke слайсу 4, «qa:verify green» слайсу 5, який не відтворювався).
  Правило «done-claims-need-evidence» в AGENTS.md було, але агент його порушував. Ловили це окремі рецензенти,
  а не саме правило, і саме тому рев'ю окремим агентом у мене обов'язкове для кожного слайсу.
- `gate-status` давав зелений там, де нічого не перевірялось (PD-8), тож довелося лагодити сам вимірювач.
- Контракт v0.5 досі не пройшов чисте рев'ю, а red-run слайсу 1 фронтенду я не записала (waiver).

## Перевірка

```bash
# у кожному з трьох репо
npm ci
npm run qa:verify
npm run gate:status
# backend додатково: npm run verify (Maven)
```

```
backend  qa:verify — Overall result: Pass   (548 unit + 44 integration, 0 failures)
frontend qa:verify — Overall result: Pass (1 member(s) PLANNED, not yet due: eval-ratchet)
         Test Files 24 passed (24) · Tests 347 passed (347) · coverage 100%
         e2e: 219 passed (5.7m) — chromium, msedge, firefox
gate-status: G7 очікувано FAIL, поки MVP не завершено (слайси 6–9)
```

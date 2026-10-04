# Doučko — домашній репетитор з математики

Capstone для **fwdays · Crash Course: Agentic Engineering (2026)**.

Doučko («маленький репетитор») — домашній застосунок, у якому дитина (6 клас чеської ZŠ) розв'язує
задачі з математики з робочого зошита свого класу. Займатися можна самостійно, з батьком онлайн
(він в Україні, читає задачу з українським перекладом поруч із чеським оригіналом) або з батьком
поруч у Чехії. Відповіді перевіряються автоматично, без підглядання в ключ відповідей.

**Зараз це MVP без AI:** застосунок працює із задачами, які дорослий вносить у базу даних з
завантажених сторінок зошита (PDF поруч із формою). Перевірка відповідей детермінована, через
golden table. Генерацію і пояснення з AI заплановано після MVP.

## Де код

Проєкт складається з трьох репозиторіїв (multi-repo, OpenSpec store у бета-версії):

| Репозиторій | Що там | Стек |
|---|---|---|
| [NatSjm/doucko-docs](https://github.com/NatSjm/doucko-docs) | вимоги, специфікації OpenSpec, контракт API, ADR, план слайсів, харнес фабрики (`factory/`) | OpenSpec 1.13, Node-скрипти |
| [NatSjm/doucko-backend](https://github.com/NatSjm/doucko-backend) | API | Kotlin · Spring Boot 4.1 · JVM 21 · Postgres 16 · Flyway |
| [NatSjm/doucko-frontend](https://github.com/NatSjm/doucko-frontend) | UI для дитини та адміна | React · Vite · TypeScript strict · Playwright |

Бекенд і фронтенд підключають `doucko-docs` як submodule `contract/` (зафіксований контракт API).

## Стан (4.10.2026)

| Слайс плану | Backend | Frontend |
|---|---|---|
| 1 platform-foundation | ✅ | ✅ |
| 2 admin-access | ✅ | ✅ |
| 3 answer-checker | ✅ | — (лише бекенд) |
| 4 child-profile | ✅ | — (лише бекенд) |
| 5 authoring-tasks | ✅ + smoke на реальному стеку | ✅ + smoke на реальному стеку |
| 6–9 missions, practice-flow, gamification, progress-stats | ⏳ далі | ⏳ далі |

Повний план: [mvp-capability-plan.md](https://github.com/NatSjm/doucko-docs/blob/main/docs/mvp-capability-plan.md).

## Як запустити й перевірити

```bash
# три репо поруч: doucko-docs, doucko-backend, doucko-frontend
cd doucko-backend
git submodule update --init
cp .env.example .env          # ADMIN_PASSWORD, DOUCKO_CHILD_DISPLAY_NAME
npm ci && npm run stack:up    # http://127.0.0.1:8081

npm run qa:verify             # батарея перевірок (у кожному репо)
npm run gate:status           # гейти G0–G8 фабрики
```

Докази практик Agentic Engineering наведено в описі PR.

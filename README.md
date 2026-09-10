# SecureLab — трекер інцидентів (baseline 2-A)

Навчальний стенд до дисципліни «Прикладні технології програмування в
інформаційній безпеці», ЛР 1. Трирівневий моноліт: браузерний клієнт
(Vanilla JS) → ASP.NET Core Web API → EF Core → PostgreSQL.

## Запуск

```bash
# 1. PostgreSQL
docker compose --env-file infra/.env.example -f infra/compose.yaml up -d --wait

# 2. Залежності та API
dotnet tool restore
dotnet restore SecureLab.slnx
dotnet run --project src/SecureLab.Api
```

Відкрити:

- <http://localhost:5080/> — браузерний клієнт;
- <http://localhost:5080/health> — readiness (перевірка з'єднання з БД);
- <http://localhost:5080/scalar/v1> — інтерактивний OpenAPI;
- <http://localhost:5080/openapi/v1.json> — OpenAPI-документ.

## Повернення до відомого seed-стану

```bash
dotnet run --no-build --project src/SecureLab.Api -- --reset-database
```

Команда працює лише в середовищі Development, застосовує migrations, очищує
відомі навчальні таблиці й повторно заповнює seed.

## Перевірка

```bash
docker compose --env-file infra/.env.example -f infra/compose.yaml up -d --wait
dotnet test tests/SecureLab.Api.Tests/SecureLab.Api.Tests.csproj --configuration Release
# або (bash): bash scripts/test.sh
```

Ручні HTTP-сценарії: `tests/http/incidents.http`.

## Структура

```
src/SecureLab.Api/
  Presentation/   endpoints і зовнішні response-контракти
  Application/    сценарії застосунку (read-only queries)
  Data/           DbContext, entities, міграції, seed/reset
  Client/         HTML / CSS / Vanilla JS
tests/
  SecureLab.Api.Tests/   інтеграційні та security-regression тести
  http/                  ручні HTTP-сценарії
infra/            локальний PostgreSQL (Docker Compose)
docs/             карта архітектури, звіт
```

## Маршрут ЛР 1

Готовий приклад — фільтр списку `GET /api/incidents?status=...`
(browser → API → EF Core → PostgreSQL → JSON → безпечні DOM sinks).
Точка розширення — `GET /api/incidents/severity-summary` (у baseline `501`,
в ЛР 1 реалізовано підсумок кількості інцидентів за severity).

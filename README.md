# BeePlan — общая документация

Здесь лежат **сквозные** материалы: требования, архитектура, сборка железа, инструкции для агентов и contributing. Код разнесён по отдельным репозиториям `beeplan-*`.

## Префикс ссылок на GitHub

Перекрёстные ссылки ведут на репозитории пользователя **[4sidora](https://github.com/4sidora)**. Если проект переедет под org или другой аккаунт — выполните замену `github.com/4sidora` в ссылках (или правьте вручную).

## Связанные репозитории

| Репозиторий | Содержимое |
|-------------|------------|
| [beeplan-edge](https://github.com/4sidora/beeplan-edge) | Прошивка улья (ESP32, ESP‑NOW) |
| [beeplan-gateway](https://github.com/4sidora/beeplan-gateway) | Прошивка концентратора |
| [beeplan-api](https://github.com/4sidora/beeplan-api) | REST API, OpenAPI, Docker |
| [beeplan-web](https://github.com/4sidora/beeplan-web) | Веб‑клиент (Vite + React) |

## Документы в этом репозитории

| Файл | Назначение |
|------|------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Потоки данных и границы компонентов |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Продуктовые требования (черновик) |
| [HARDWARE.md](HARDWARE.md) | Сборка плат, пины, связь устройств |
| [LOCAL_TESTING.md](LOCAL_TESTING.md) | Локальный end-to-end тест: edge, gateway, API, веб |
| [AGENTS.md](AGENTS.md) | Правила для разработчиков и ИИ‑агентов |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Pull requests и соглашения по коду |
| [GITHUB.md](GITHUB.md) | Как создать пустые репозитории и первый push |

Быстрый старт API: README в репозитории **beeplan-api**.

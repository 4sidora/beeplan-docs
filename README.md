# BeePlan — общая документация

Здесь лежат **сквозные** материалы: требования, архитектура, сборка железа, инструкции для агентов и contributing. Код разнесён по отдельным репозиториям `beeplan-*`.

## Префикс ссылок на GitHub

В markdown-файлах этого репозитория для перекрёстных ссылок на код используется пример организации **`beeplan`**. Если вы публикуете под другим пользователем или org, выполните замену `github.com/beeplan` на свой префикс (или правьте ссылки вручную).

## Связанные репозитории

| Репозиторий | Содержимое |
|-------------|------------|
| [beeplan-edge](https://github.com/beeplan/beeplan-edge) | Прошивка улья (ESP32, ESP‑NOW) |
| [beeplan-gateway](https://github.com/beeplan/beeplan-gateway) | Прошивка концентратора |
| [beeplan-api](https://github.com/beeplan/beeplan-api) | REST API, OpenAPI, Docker |
| [beeplan-web](https://github.com/beeplan/beeplan-web) | Веб‑клиент (Vite + React) |

## Документы в этом репозитории

| Файл | Назначение |
|------|------------|
| [ARCHITECTURE.md](ARCHITECTURE.md) | Потоки данных и границы компонентов |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Продуктовые требования (черновик) |
| [HARDWARE.md](HARDWARE.md) | Сборка плат, пины, связь устройств |
| [AGENTS.md](AGENTS.md) | Правила для разработчиков и ИИ‑агентов |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Pull requests и соглашения по коду |
| [GITHUB.md](GITHUB.md) | Как создать пустые репозитории и первый push |

Быстрый старт API: README в репозитории **beeplan-api**.

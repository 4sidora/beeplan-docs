# Запланировано: мягкое удаление и форма создания базовой станции

Документ фиксирует анализ и план работ по запросу от 2026-06-01.  
**Статус:** реализовано (2026-06-01) — миграция `0008`, API soft delete, `GET /v1/concentrators/suggested-name`, формы создания с автозаполнением.

---

## 1. Мягкое удаление (базовая станция, устройство, данные)

### Текущее поведение

- Удаление в UI вызывает `DELETE` в API → `db.delete()` + `commit()` в PostgreSQL.
- Записи **физически удаляются**; флага `deleted_at` / корзины нет.
- Каскады в БД (`ON DELETE CASCADE` / `SET NULL`) описаны в `beeplan-api/beeplan/models.py`.

| Сущность | Endpoint | Поведение сейчас |
|----------|----------|------------------|
| Пасека | `DELETE /v1/apiaries/{id}` | Hard delete |
| Семья | `DELETE /v1/colonies/{id}` | Hard delete |
| Базовая станция (concentrator) | `DELETE /v1/concentrators/{id}` | Hard delete; каскад на edge-устройства |
| Edge-устройство | `DELETE /v1/edge-devices/{id}` | Hard delete; предварительно чистятся assignments и `edge_device_telemetry_samples` |
| Телеметрия семьи | — | При удалении семьи — CASCADE; при удалении устройства `telemetry_samples.source_device_id` → `SET NULL` (точки на семье остаются) |

### Цель

При нажатии «Удалить» в интерфейсе для **базовой станции** и **конечного устройства** (и связанных с ними данных в UI) — **мягкое удаление**: скрыть из списков, не показывать в ingest/прошивке, но оставить строки в БД для возможного восстановления (восстановление в UI — опционально, не обязательно в первой итерации).

**Уточнение от заказчика (подразумевалось):** мягкое удаление для станций и edge; пасеки/семьи можно оставить hard delete, если не скажут иначе.

### Предлагаемая реализация

#### Схема БД (миграция Alembic)

```text
concentrators.deleted_at  TIMESTAMPTZ NULL
edge_devices.deleted_at   TIMESTAMPTZ NULL
```

Телеметрию (`telemetry_samples`, `edge_device_telemetry_samples`) **отдельно не помечать**: при `deleted_at` у устройства/станции не отдавать точки в API и не принимать ingest с этих сущностей.

#### API

| Действие | Изменение |
|----------|-----------|
| `DELETE /v1/concentrators/{id}` | `UPDATE concentrators SET deleted_at = now()` |
| `DELETE /v1/edge-devices/{id}` | `UPDATE edge_devices SET deleted_at = now()` |
| Все `GET` / list / detail | `WHERE deleted_at IS NULL` |
| `POST /v1/telemetry/batch`, heartbeat | Игнорировать станции/устройства с `deleted_at IS NOT NULL` |
| `require_concentrator` / ownership checks | 404 для «удалённых», как для отсутствующих |

Файлы для правок (ориентир):

- `beeplan-api/beeplan/models.py`
- `beeplan-api/beeplan/routers/catalog.py` — `delete_concentrator`, списки концентраторов
- `beeplan-api/beeplan/routers/devices.py` — `delete_edge_device`, списки edge, ingest
- `beeplan-api/beeplan/routers/firmware.py` — поиск устройства/станции при сборке
- `beeplan-api/beeplan/deps.py` — `require_concentrator`, если есть проверки edge

#### UI (`beeplan-web`)

- Кнопки «Удалить» без смены контракта (по-прежнему `DELETE`).
- Сообщения в confirm-диалогах можно уточнить: «Скрыть из списка» / «Удалить (можно восстановить только через админку)» — по продуктовому решению.
- Раздел «Удалённые» / «Восстановить» — **не в scope первой версии**, если не запросят.

#### Риски

- Уникальные поля (`public_id`, `ingest_token`): удалённая запись блокирует повторное использование id, пока строка в БД. При необходимости — partial unique index `WHERE deleted_at IS NULL` (отдельная миграция).
- Старые прошивки с `ingest_token` удалённой станции продолжат слать heartbeat — API должен отвечать 404/403 без «оживления» записи.

---

## 2. Форма «Новая базовая станция»: имя первым, с автозаполнением при открытии

### Текущее поведение

- **DevicesPage** / **ConcentratorsPage**: в диалоге создания сначала поле **«Пасека»**, затем **«Название»** (пустое).
- При сохранении с пустым именем API подставляет `generate_base_station_name()` из `beeplan-api/beeplan/base_station_names.py` (формат: `Базовая станция {город}`).
- Плейсхолдер: `BASE_STATION_NAME_PLACEHOLDER` в `beeplan-web/src/constants/baseStation.ts`.

### Цель

1. **Первое поле** — «Название», при открытии диалога **уже заполнено** сгенерированным именем (пользователь может изменить).
2. **Второе поле** — выбор пасеки.
3. При сохранении отправлять введённое имя (как сейчас); fallback на сервере оставить для пустой строки.

### Предлагаемая реализация

#### API (рекомендуется)

По аналогии с `GET /v1/colonies/suggested-name`:

```http
GET /v1/concentrators/suggested-name
→ { "name": "Базовая станция Бердск" }
```

Реализация: `generate_base_station_name()` из `base_station_names.py`, endpoint в `catalog.py`, схема `ConcentratorNameOut` (или переиспользовать общий `SuggestedNameOut`).

#### Web

| Файл | Изменения |
|------|-----------|
| `src/api.ts` | `suggestedBaseStationName(): Promise<{ name: string }>` |
| `src/pages/DevicesPage.tsx` | `openCreate()`: `await api.suggestedBaseStationName()` → `setName(...)`; порядок полей: Название, Пасека |
| `src/pages/ConcentratorsPage.tsx` | То же для диалога создания |

**Не дублировать** пул городов на фронте — только API.

#### UX

- Пока грузится suggested name — спиннер в поле или disabled dialog (короткий запрос).
- При повторном открытии диалога — **новое** случайное имя (каждый раз новый город из пула).

---

## 3. Связанные уже сделанные изменения (для контекста)

Уже в `main` на GitHub (2026-06-01):

- **beeplan-api** `b2d5e45`: миграция edge `name` / `firmware_version`, `base_station_names.py`, автоген имён edge и станции при пустом `name` на create.
- **beeplan-web** `5e5bfca`: UI «базовая станция» вместо «концентратор», карточки, `ObjectCardHeader`, edge detail/edit, список без MAC, delete только в формах редактирования.

---

## 4. Чеклист при реализации

### Мягкое удаление

- [x] Миграция `deleted_at` на `concentrators`, `edge_devices` (`20250601_0008_soft_delete.py`)
- [x] Soft delete в `delete_concentrator`, `delete_edge_device` (`beeplan/soft_delete.py`)
- [x] Фильтр `deleted_at IS NULL` во всех read-path
- [x] Ingest / heartbeat / firmware — отсечь удалённые
- [x] Partial unique indexes для `public_id` / `ingest_token` (только активные записи)
- [ ] Ручная проверка: удалить станцию → нет в списке; edge на ней не виден; телеметрия не принимается
- [ ] `apply-dev-updates.ps1` после миграции

### Форма создания

- [x] `GET /v1/concentrators/suggested-name`
- [x] `DevicesPage` + `ConcentratorsPage`: порядок полей, prefetch имени в `openCreate`
- [ ] Сборка web + smoke-тест диалога

---

## 5. Ссылки в кодовой базе

| Тема | Путь |
|------|------|
| Hard delete concentrator | `beeplan-api/beeplan/routers/catalog.py` → `delete_concentrator` |
| Hard delete edge | `beeplan-api/beeplan/routers/devices.py` → `delete_edge_device` |
| Модели и FK | `beeplan-api/beeplan/models.py` |
| Автоимя станции | `beeplan-api/beeplan/base_station_names.py`, `create_concentrator` |
| Аналог suggested name (семья) | `beeplan-api/beeplan/routers/catalog.py` → `suggested_colony_name` |
| Диалог создания станции | `beeplan-web/src/pages/DevicesPage.tsx`, `ConcentratorsPage.tsx` |
| Плейсхолдер | `beeplan-web/src/constants/baseStation.ts` |

---

*Документ создан для продолжения работ в новой сессии; при реализации обновить статус в начале файла.*

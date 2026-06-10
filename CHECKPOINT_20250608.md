# Checkpoint: ESP-NOW v2 + UI телеметрии (2025-06-08)

Точка продолжения работы. Workspace — **6 отдельных git-репозиториев** (корень workspace не git).

## Что сделано

### ESP-NOW v2 (прошивка 0.2.0, Level A)

| Компонент | Статус |
|-----------|--------|
| `ReportFrameV2` / `AckFrameV2` | `beeplan-docs/beeplan-protocol/envelope_v2.h` (канон), копии в edge/gateway `include/envelope_v2.h` |
| Edge: deep sleep, TDMA, 1 кадр/ч, ACK listen | `beeplan-edge/src/main.cpp` |
| Gateway: queue 512, v1/v2, LittleFS spool, ACK, heartbeat 15 min | `beeplan-gateway/src/main.cpp` |
| API: `wifi_channel`, `telemetry_slot_sec`, `telemetry_ingest_log`, dedup `report_id` | миграция `0010`, `radio_plan.py` |
| Builder: slot/channel/device_type в config.h | `generate_config.py`, default wake 3600 |
| Web: install defaults 3600, slot/channel read-only, spool badge | `EdgeInstallPage`, `ConcentratorCard` |
| Документация протокола | `beeplan-docs/RADIO_PROTOCOL.md` |

**Зафиксировано:** только **Уровень A** (гарантия gateway → PostgreSQL через spool + idempotent ingest). Edge не получает cloud-confirmed ACK.

### Web/UI (до v2)

- Индикаторы сигнал/батарея на карточках устройств
- Деталь концентратора: collapsible charts, gaps в графиках (demo `dev-edge-1`, device id **19**)
- Таблица телеметрии на странице edge device (pivot по времени)

## Dev-стек

- `.\scripts\apply-dev-updates.ps1` — последний успешный прогон после фикса миграции `0010` (идемпотентная)
- Web: http://localhost:5173
- API: http://localhost:8000/docs
- Builder: http://localhost:9000/health

## Миграция БД

- `beeplan-api/alembic/versions/20250608_0010_radio_v2.py` — revision `0010`, down_revision `0009`
- Первый прогон падал (неверный down_revision + duplicate column); исправлено проверками `_has_column` / `_has_table`
- Backfill: `telemetry_slot_sec = (row_number % 100) * 36` для существующих edge

## Следующие шаги (завтра)

1. **Полевой тест v2:** перепрошить gateway → дождаться `wifi_channel` в UI → собрать/прошить edge 0.2.0
2. **Тесты из плана:** struct size, slot planner (есть `tests/test_radio_plan.py`), E2E hour cycle, load 100 frames/h
3. **Документация:** обновить ARCHITECTURE, HARDWARE, REQUIREMENTS, LOCAL_TESTING, README edge/gateway (частично не тронуты)
4. **Документация:** обновить ARCHITECTURE, HARDWARE, REQUIREMENTS, LOCAL_TESTING, README edge/gateway (частично не тронуты)
5. **Emergency / swarm:** только flags в протоколе, детекция не реализована
6. **Level B (cloud ACK на edge):** out of scope

## Порядок прошивки для проверки

```
1. Gateway build 0.2.0 → USB flash → heartbeat → wifi_channel в БД
2. Edge build (нужны gateway_mac + wifi_channel + telemetry_slot_sec) → USB flash
3. Serial edge: ReportFrameV2 + ack ok + deep sleep
4. Serial gateway: v2 rx → spool → batch 200 → accepted_report_ids
```

## Известные ограничения

- Слоты при удалении устройства не перераспределяются (дыры в TDMA допустимы)
- LittleFS spool: при mount fail gateway работает без durable spool (WARN в Serial)
- `FirmwareVersionInfo.tsx` — возможна pre-existing TS ошибка `fontWeight` на Typography

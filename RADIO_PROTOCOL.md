# ESP-NOW Radio Protocol v2 (BeePlan)

Единый источник правды для edge, gateway, builder и API.

## Transport choice

BeePlan использует **ESP-NOW** (2.4 GHz Wi-Fi radio) для связи edge → gateway:

- **Дальность** — сопоставима с Wi-Fi (десятки–сотни метров на открытой местности).
- **Star topology** — до 100 edge на одну базовую станцию.
- **Энергия** — один кадр в час, deep sleep между слотами.

BLE и LoRa не используются в v2.

## Версии протокола

| Версия | magic | Описание |
|--------|-------|----------|
| v1 | `0x00BEEF01` | 1 метрика на кадр (legacy) |
| v2 | `0x00BEEF02` | Агрегированный hourly report |
| ACK | `0x00BEEFAC` | Downlink gateway → edge |

Общий заголовок: [`beeplan-protocol/envelope_v2.h`](beeplan-protocol/envelope_v2.h).

## ReportFrameV2

- `flags`: `0x01` emergency, `0x02` scheduled, `0x04` audio_included
- `seq`: sequence для ACK (v2.1)
- `device_type`: 0=multisensor, 1=scales
- `metrics_present`: bitmask (temp, rh, signal, battery, firmware, audio)
- Размер ≤ 64 байт без audio

## TDMA

- Период цикла: **3600 с** (`WAKE_INTERVAL_SEC`)
- Слот: **36 с** на устройство (`telemetry_slot_sec = index × 36`, index 0..99)
- Канал Wi-Fi: **1–13**, задаётся при прошивке (`GATEWAY_WIFI_CHANNEL`), scan каналов **убран**

## Delivery classes

| Класс | Участок | Гарантия |
|-------|---------|----------|
| L1 | Edge → Gateway | ACK v2.1 после приёма + retry |
| L2 | Gateway → Cloud | Flash spool + at-least-once HTTP (Level A) |
| L3 | Cloud confirmed ACK to edge | **Не реализовано** (только emergency в будущем) |

### Level A (выбрано)

1. Gateway: `rx → append spool (flush) → AckFrameV2`
2. Drain spool → `POST /v1/telemetry/batch` с `report_id`
3. API: dedup по `(device_id, report_id)`, ответ `accepted_report_ids`
4. Gateway удаляет подтверждённые записи из spool

Edge **не** знает, что данные в PostgreSQL — только что gateway принял (ACK).

## Emergency (задел)

`FLAG_EMERGENCY` — внеплановая передача; gateway flush немедленно. Детекция роения — позже.

## Совместимость

Gateway принимает **v1 + v2** один релиз. Новые edge прошиваются только v2 (semver **0.2.0**).

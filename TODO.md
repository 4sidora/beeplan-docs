# BeePlan — TODO

Список отложенных задач и идей. Связано: [FIRMWARE_BUILD_IMPROVEMENTS.md](FIRMWARE_BUILD_IMPROVEMENTS.md), [PLANNED_SOFT_DELETE_AND_CREATE_FORM.md](PLANNED_SOFT_DELETE_AND_CREATE_FORM.md).

---

## Прошивка / версия в UI и API

- [ ] **Версия «Прошивка» на edge не должна означать «сборка готова»**  
  Сейчас при статусе сборки `ready` API записывает в `edge_devices.firmware_version` значение из каталога (например `0.1.2`) в `_poll_builder` ([`beeplan-api/beeplan/routers/firmware.py`](../beeplan-api/beeplan/routers/firmware.py)) — **до** USB-прошивки и без данных с устройства. В карточке устройства отображается «Прошивка 0.1.2», хотя железо ещё не прошивали.  
  **Нужно решить:** показывать только версию **на устройстве** (heartbeat / телеметрия / явное подтверждение после прошивки) или разделить поля: «собрана для прошивки» vs «на устройстве».  
  Затронуть: API, [`EdgeDeviceCard`](../beeplan-web/src/components/EdgeDeviceCard.tsx), [`EdgeDeviceDetailPage`](../beeplan-web/src/pages/EdgeDeviceDetailPage.tsx), мастер ([`FirmwareVersionInfo`](../beeplan-web/src/components/FirmwareVersionInfo.tsx) — строка «На устройстве (API)»).  
  Для сравнения: у gateway версия в API обновляется из **heartbeat** (`POST /v1/concentrators/heartbeat`), а не при сборке бинарника.

---

## Сборка прошивок (builder / UX)

См. также чеклист в [FIRMWARE_BUILD_IMPROVEMENTS.md](FIRMWARE_BUILD_IMPROVEMENTS.md).

- [ ] SSE / WebSocket: live-лог сборки (`GET /v1/firmware/builds/{id}/events`)
- [ ] ccache / sccache в PlatformIO (env + Dockerfile builder)
- [ ] Edge: исправление `WiFi.disconnect(true, true)` → ESP-NOW ([HARDWARE.md](HARDWARE.md))
- [ ] Передавать `GATEWAY_WIFI_CHANNEL` из API после heartbeat концентратора
- [ ] Edge `esp32c3`: сборка в builder (Serial / `BEE_SERIAL`)

---

## Типы конечных устройств (мастер edge)

- [ ] Прошивка и параметры для типа «Пасечные весы»
- [ ] Шаг «Параметры»: выбор поддерживаемых датчиков и настройки по типу устройства (общие + специфичные для датчика)

---

## Прочее

- [ ] Мягкое удаление и формы создания — [PLANNED_SOFT_DELETE_AND_CREATE_FORM.md](PLANNED_SOFT_DELETE_AND_CREATE_FORM.md)

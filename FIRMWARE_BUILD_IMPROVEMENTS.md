# План улучшений сборки прошивок и UX мастера прошивки

Документ фиксирует текущее состояние (май 2026) и приоритеты. Связано: [beeplan-builder/README.md](../beeplan-builder/README.md), [HARDWARE.md](HARDWARE.md).

---

## Как устроено сейчас

| Уровень | Поведение |
|---------|-----------|
| **Веб** (`GatewayInstallPage`, `EdgeInstallPage`) | Опрос `GET /v1/firmware/builds/{id}` каждые ~2.5 с; статусы `queued` / `building` / `ready` / `failed`; таймер, `LinearProgress`, фаза и хвост лога (`FirmwareBuildProgress`). |
| **API** (`firmware.py`) | Фоновый `_poll_builder` опрашивает builder каждые 2 с (таймаут ~20 мин); прогресс проксируется с builder на GET. |
| **Builder** (`compile.py`) | Постоянный workdir `/workdir/{profile}/{board}` с кэшем `.pio`; `pio run -j N`; stdout в `artifacts/{id}/build.log`; фазы и `log_tail`. |
| **Статусы builder** | `phase`, `log_tail`, `progress_pct`, `updated_at` в `GET /v1/builds/{id}`. |

---

## 1. Ускорение сборки

### Быстро (низкая сложность)

- [x] **`pio run -j N`** — параллельная компиляция (`N` = число ядер хоста).
- [ ] **Не пересоздавать контейнер builder** без нужды; при старом образе — `beeplan-builder/scripts/warmup-toolchain.sh`.
- [x] **Очередь сборок** — одна активная `pio` на инстанс builder (`_compile_lock`).

### Средний эффект

- [x] **Постоянный workdir** вместо `copytree` в temp: синхронизация исходников, `.pio/build` per `profile` + `board`.
- [ ] **ccache / sccache** в PlatformIO (env + Dockerfile).

### Операционка / dev

- Локально: `pio run -e esp32dev` / `esp32c3` в репозиториях прошивок — быстрая обратная связь в терминале.

**Ожидаемый выигрыш:** повторные сборки с кэшем `.pio` — порядка **10–30 с** вместо **50–90+ с** (gateway после прогрева).

---

## 2. Детальный прогресс в UI

### Минимум (без стриминга)

- [x] Таймер «Идёт сборка: M:SS» от `build.created_at` (поле уже в API).
- [x] Подсказка по типу: gateway ~1 мин, edge ~1.5 мин (после прогретого toolchain).
- [x] Этапы по фазам builder: «Подготовка» → «Компиляция» → «Линковка» → «Упаковка».

### Рекомендуемый минимум («не зависло»)

**Builder** — при `pio run`:

- [x] писать stdout построчно в `artifacts/{build_id}/build.log`;
- [x] расширить `GET /v1/builds/{id}`:
  - `phase`: `preparing` | `compiling` | `linking` | `packaging`
  - `log_tail`: последние 20–50 строк
  - `updated_at`
  - опционально `progress_pct` (парсинг строк `Compiling …`, `Linking …`).

**API** — прокинуть поля в `FirmwareBuildOut`.

**Веб** — `LinearProgress`, последняя строка лога, раскрываемый «Полный лог».

### Максимум

- [ ] SSE / WebSocket: `GET /v1/firmware/builds/{id}/events` для live-лога.

---

## 3. Порядок внедрения

1. ~~Скорость: workdir + `.pio` + `-j`; очередь на builder.~~ ✅
2. ~~UX: таймер + `log_tail` + `phase`.~~ ✅
3. По необходимости: SSE.

---

## 4. Связанные задачи прошивки (не сборка)

- [ ] Edge: исправление `WiFi.disconnect(true, true)` → ESP-NOW (см. [HARDWARE.md](HARDWARE.md)).
- [ ] Передавать `GATEWAY_WIFI_CHANNEL` из API после heartbeat концентратора (избежать долгого скана 1–13).
- [ ] Edge `esp32c3`: сборка в builder (Serial / `BEE_SERIAL`).

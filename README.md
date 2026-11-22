# Meshtastic — кастомные сборки прошивки для Seeed Wio Tracker L1 E-Ink

Готовые к установке `.uf2` сборки [Meshtastic firmware](https://github.com/meshtastic/firmware) **2.8.2** для устройства [Seeed Studio Wio Tracker L1 E-Ink](https://www.seeedstudio.com/Wio-Tracker-L1-E-Ink-p-6523.html) (nRF52840) с исправлениями под вертикальное расположение экрана и русской поддержкой.

Идея и фото корпуса — из проекта [Nastradamus/Meshtastic](https://github.com/Nastradamus/Meshtastic), там же был вопрос об обновлении сборок (см. [issue #1](https://github.com/Nastradamus/Meshtastic/issues/1)) — я разобрался и пересобрал всё актуальное сам.

<img src="https://raw.githubusercontent.com/Nastradamus/Meshtastic/main/images/tracker.jpeg" width="250">

## Прошивки

| Файл | Окружение PlatformIO | UI |
| ---- | -------------------- | -- |
| `firmwares/Wio_Tracker_L1_EInk_v2.8.2_RU.uf2` | `seeed_wio_tracker_L1_eink` | стандартный экран |
| `firmwares/Wio_Tracker_L1_EInk_InkHUD_v2.8.2_RU.uf2` | `seeed_wio_tracker_L1_eink-inkhud` | InkHUD (рекомендую для e-ink) |

MD5:

```
ea27bf2f32389a5fba53fa67ec02ee8a  Wio_Tracker_L1_EInk_v2.8.2_RU.uf2
4758505b45f6b2890e8f659223819799  Wio_Tracker_L1_EInk_InkHUD_v2.8.2_RU.uf2
```

Обе собраны из официальной прошивки **2.8.2** (`d5f1619`) с моими изменениями (см. ниже и файл `patches/wio_tracker_l1_eink_ru.patch`).

## Что исправлено

### 1. Джойстик (трекбол) под вертикальный корпус

Экран у устройства расположен вертикально (в сборке экран повёрнут на 90°, `rotation = 1`), а пины трекбола в официальной прошивке соответствуют альбомной ориентации — в вертикальном корпусе джойстик «перепутан». Исправление — обмен пинов направления (`variants/nrf52840/seeed_wio_tracker_L1*/variant.h`, исправлены обе вариации — e-ink и обычную L1):

```c
// было                          // стало
#define TB_UP 25                 #define TB_UP 27
#define TB_DOWN 26               #define TB_DOWN 28
#define TB_LEFT 27               #define TB_LEFT 26
#define TB_RIGHT 28              #define TB_RIGHT 25
```

### 2. Русский язык (InkHUD-сборка)

В `nicheGraphics.h` подключены шрифты с кириллицей, в `platformio.ini` — флаг `-D OLED_RU`:

```c
InkHUD::Applet::fontLarge  = FREESANS_12PT_WIN1251;
InkHUD::Applet::fontMedium = FREESANS_9PT_WIN1251;
InkHUD::Applet::fontSmall  = FREESANS_6PT_WIN1251;
```

## Как прошить

1. Зажмите кнопку **BOOT** на устройстве и подключите USB-C → смонтируется диск **WIO_UPDATE**.
2. Скопируйте нужный `.uf2` на диск — устройство перепрошьётся и перезагрузится.
3. После первой прошивки — настройте устройство (LoRa-регион, имя ноды) через приложение Meshtastic.

⚠️ Сборки неофициальные: это не релизы meshtastic.org, прошивка — на ваш страх и риск.

## Как собрать самому

```bash
git clone https://github.com/meshtastic/firmware
cd firmware
git checkout 2.8.2
# применить патч
git apply patches/wio_tracker_l1_eink_ru.patch

pio run -e seeed_wio_tracker_L1_eink -t clean && pio run -e seeed_wio_tracker_L1_eink
pio run -e seeed_wio_tracker_L1_eink-inkhud -t clean && pio run -e seeed_wio_tracker_L1_eink-inkhud
```

Готовые `.uf2` — в `.pio/build/seeed_wio_tracker_L1_eink*/firmware.uf2`.

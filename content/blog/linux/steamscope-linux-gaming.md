---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "gaming", "steam", "guide"]
date: "2026-04-07T10:20:00+03:00"
description: "steamscope.sh: единый скрипт для запуска Steam-игр на Linux с оптимизациями под движок, видеокарту и систему."
draft: false
series: ["Linux Gaming"]
slug: "steamscope"
tags: ["steam", "proton", "gamescope", "gamemode", "linux-gaming", "bash"]
title: "steamscope.sh: Универсальный лаунчер для Steam на Linux"
---

## 🎮 Зачем это нужно

Запуск игр через Steam на Linux - это часто «танцы с бубном»: разные движки требуют разных флагов, под AMD и NVIDIA нужны разные переменные окружения, а инструменты вроде Gamescope и MangoHud нужно вручную встраивать в команду запуска.

**Решение**: `steamscope.sh` - скрипт-обёртка, который:

- ✅ Автоматически определяет видеокарту (AMD/NVIDIA) и применяет нужные оптимизации
- ✅ Поддерживает флаги под движки: Source, Unreal, Unity
- ✅ Интегрирует Gamescope, Gamemode, MangoHud, FSR одной командой
- ✅ Работает как `%command%` в настройках запуска Steam
- ✅ Не ломает стандартный запуск - всё опционально

> 💡 Скрипт не заменяет Proton или Steam - он делает их работу предсказуемой и настраиваемой.

---

## 📦 Установка

### Шаг 1: Скачать скрипт

```bash
# Создать директорию (если нет)
mkdir -p ~/.steam/steam/

# Скачать скрипт
curl -o ~/.steam/steam/steamscope.sh \
  https://gist.githubusercontent.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6/raw/steamscope.sh

# Сделать исполняемым
chmod +x ~/.steam/steam/steamscope.sh
```

### Шаг 2: Настроить игру в Steam

1. Правой кнопкой по игре → **Свойства**
2. В поле **Параметры запуска** добавить:
   ```bash
   ~/.steam/steam/steamscope.sh %command%
   ```
3. (Опционально) Добавить флаги:
   ```bash
   ~/.steam/steam/steamscope.sh --gamemode --mangohud --engine=source %command%
   ```

**Готово**. При запуске игры скрипт автоматически подхватит настройки.

---

## ⚙️ Параметры запуска (категории)

### 🚀 Производительность

| Флаг         | Описание                                                  | Для кого                         |
| ------------ | --------------------------------------------------------- | -------------------------------- |
| `--gamemode` | Включает `gamemoderun` для приоритета процессов           | Все, особенно на слабых системах |
| `--fsr`      | Включает FSR-апскейлинг через VKD3D (AMD) / DXVK (NVIDIA) | AMD GPU, низкий нативный FPS     |
| `--dxvk`     | Принудительно использует DXVK вместо D9VK                 | Игры с DirectX 9/10/11           |

### 👁 Мониторинг

| Флаг         | Описание                                                   | Пример вывода                    |
| ------------ | ---------------------------------------------------------- | -------------------------------- |
| `--mangohud` | Показывает оверлей с FPS, загрузкой CPU/GPU, температурами | `FPS: 87 │ CPU: 45% │ GPU: 72°C` |

### 🖥 Режим работы

| Флаг                    | Описание                                                      | Зачем                                                     |
| ----------------------- | ------------------------------------------------------------- | --------------------------------------------------------- |
| `--gamescope`           | Запускает игру в изолированной сессии Wayland через Gamescope | Стабильный FPS, адаптивная синхронизация, масштабирование |
| `--resolution=1280x720` | Задает разрешение для Gamescope (по умолчанию - нативное)     | Тестирование, снижение нагрузки                           |

### 🎯 Движки

| Флаг              | Поддерживаемые движки | Примеры игр                                      |
| ----------------- | --------------------- | ------------------------------------------------ |
| `--engine=source` | Source 1/2            | CS2, TF2, Garry's Mod, Left 4 Dead 2             |
| `--engine=unreal` | Unreal Engine 3/4/5   | Risk of Rain 2, Deep Rock Galactic, Satisfactory |
| `--engine=unity`  | Unity                 | 7 Days to Die, Valheim, Among Us                 |

**Что добавляет каждый движок**:

- **Source**: `-high -threads N +mat_vsync 0 +fps_max 0 +exec autoexec.cfg`
- **Unreal**: `-dx12 -nomansky -notexturestreaming -nomovie`
- **Unity**: `-screen-width W -screen-height H -nolog -batchmode`

---

## 🔧 Как это работает (под капотом)

### Автоматическое определение железа

```bash
# Скрипт проверяет видеокарту через lspci
GPU_INFO=$(lspci -k 2>/dev/null | awk '/VGA|3D/,/Kernel driver/ {if (/Kernel driver in use/) print $NF}' | tail -1)

# Для AMD:
[[ "$GPU_INFO" == *"amdgpu"* ]] && USE_AMD=true
# → Добавляет: RADV_PERFTEST=aco, AMD_DEBUG=nodcc

# Для NVIDIA:
[[ "$GPU_INFO" == *"nvidia"* ]] && USE_NVIDIA=true
# → Добавляет: PROTON_ENABLE_NVAPI=1, __NV_PRIME_RENDER_OFFLOAD=1
```

### Сборка финальной команды

Скрипт строит команду «слоями»:

```
[gamemoderun] → [mangohud] → [gamescope -- ...] → [ИГРА + флаги движка]
```

Каждый слой добавляется только если соответствующий флаг передан.

### Обработка `%command%`

Steam передаёт команду запуска как `%command%`. Скрипт:

1. Находит место после исполняемого файла (но не внутри `proton`, `steam-runtime` и т.п.)
2. Вставляет флаги движка **после** исполняемого файла, но **до** аргументов игры
3. Это гарантирует, что флаги попадут именно в игру, а не в лаунчер

---

## 🎯 Примеры для популярных игр

### Counter-Strike 2 (Source Engine, AMD)

```bash
~/.steam/steam/steamscope.sh \
  --engine=source \
  --gamemode \
  --mangohud \
  --fsr \
  --gamescope \
  %command%
```

**Что получит игра**:

- Приоритет процессов через `gamemoderun`
- Оверлей MangoHud с метриками
- Апскейлинг FSR через VKD3D (для повышения FPS)
- Запуск в Gamescope с адаптивной синхронизацией
- Оптимизированные флаги Source: `-high -threads 8 +fps_max 0`

### Deep Rock Galactic (Unreal Engine, NVIDIA)

```bash
~/.steam/steam/steamscope.sh \
  --engine=unreal \
  --gamemode \
  --dxvk \
  --mangohud \
  %command%
```

**Особенности**:

- `--dxvk` включает DXVK для лучшей совместимости с Vulkan на NVIDIA
- Флаги Unreal отключают тяжёлые эффекты: `-nomansky -notexturestreaming`
- `UE5_ALLOW_LINUX_DEBUGGING=1` для стабильности на Linux

### 7 Days to Die (Unity, кросс-платформа)

```bash
~/.steam/steam/steamscope.sh \
  --engine=unity \
  --gamemode \
  --fsr \
  --resolution=1280x720 \
  %command%
```

**Зачем**:

- `--resolution` снижает нагрузку для плавного геймплея
- `--fsr` компенсирует потерю детализации апскейлингом
- Флаги Unity отключают логирование: `-nolog -batchmode`

---

## 🛠 Советы по настройке

### Начинай с минимума

```bash
# Только игра (базовый запуск)
~/.steam/steam/steamscope.sh %command%

# + мониторинг
~/.steam/steam/steamscope.sh --mangohud %command%

# + оптимизация
~/.steam/steam/steamscope.sh --gamemode --mangohud %command%
```

Добавляй параметры по одному - так проще найти причину проблем.

### Если игра падает

1. **Отключи `--gamescope`** - самый частый источник конфликтов
2. **Отключи `--gamemode`** - может конфликтовать с системными настройками
3. **Проверь логи**:
   ```bash
   journalctl -f | grep -i steam
   ```

### Разрешение и масштабирование

- По умолчанию скрипт берёт нативное разрешение через `xrandr`
- Для принудительного изменения: `--resolution=1280x720`
- В сочетании с `--gamescope` и `--fsr` даёт гибкое управление качеством/производительностью

### Локализация

Скрипт устанавливает:

```bash
export LANG="ru_RU.UTF-8"
export LC_ALL="ru_RU.UTF-8"
```

Если игра требует английскую локаль - переопредели в параметрах запуска:

```bash
LANG=en_US.UTF-8 ~/.steam/steam/steamscope.sh %command%
```

---

## ⚠️ Частые проблемы

```bash
# "gamescope: command not found"
→ Установить:
  # Arch: sudo pacman -S gamescope
  # Debian/Ubuntu: sudo apt install gamescope
  # Fedora: sudo dnf install gamescope

# "gamemoderun: command not found"
→ Установить:
  # Все дистрибутивы: sudo apt install gamemode или аналог

# Игра игнорирует флаги движка
→ Убедиться, что %command% стоит в конце строки
→ Проверить, что игра действительно на нужном движке (не все «названия» совпадают с реальностью)

# Нет оверлея MangoHud
→ Проверить, что mangohud установлен и в PATH
→ Для некоторых игр требуется: export MANGOHUD_DLSYM=1 (скрипт делает это автоматически)

# Низкий FPS после включения --fsr
→ FSR - это апскейлинг, а не магия: он повышает FPS за счёт качества
→ Попробуй снизить --resolution вместе с --fsr

# Скрипт не определяет видеокарту
→ Убедиться, что установлен pciutils: sudo apt install pciutils
→ Проверить вывод: lspci -k | grep -A2 -i vga
```

---

## 🔐 Безопасность и приватность

Скрипт **не собирает данные**, не отправляет ничего в сеть и работает полностью локально.

**Что он делает**:

- Читает `lspci` для определения GPU (только для оптимизаций)
- Устанавливает переменные окружения для процесса игры
- Не модифицирует файлы игры или системы

**Рекомендации**:

- Скачивай скрипт только с [официального gist](https://gist.github.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6)
- Проверяй хэш при обновлении (опционально)
- Не передавай скрипт с чувствительными данными в аргументах

---

## 🗂 Чеклист перед использованием

- [ ] Установил зависимости: `gamemode`, `mangohud`, `gamescope` (по желанию)
- [ ] Скачал скрипт в `~/.steam/steam/steamscope.sh` и сделал исполняемым
- [ ] Протестировал запуск одной игры с минимальными флагами
- [ ] Проверил, что `%command%` стоит в конце строки параметров запуска
- [ ] Ознакомился с [локальной регуляторикой](#) по использованию оверлеев в играх (если важно)

---

## Ссылки

- 🎮 [Официальный Gist со скриптом](https://gist.github.com/ponfertato/85fc964664423eae1ac0a7ee91d53db6)
- 🐧 [ProtonDB - совместимость игр](https://www.protondb.com)
- 🖥 [Gamescope на GitHub](https://github.com/ValveSoftware/gamescope)
- ⚡ [Gamemode на GitHub](https://github.com/FeralInteractive/gamemode)
- 👁 [MangoHud на GitHub](https://github.com/flightlessmango/MangoHud)
- 🎯 [FSR в VKD3D-Proton](https://github.com/HansKristian-Work/vkd3d-proton)

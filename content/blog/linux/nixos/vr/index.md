---
author: ["Potato Energy Team", "ponfertato"]
categories: ["nix", "linux", "vr", "hardware"]
date: "2026-08-03T09:00:00+03:00"
description: "Настройка VR на NixOS для Meta Quest 3S: WiVRn, WayVR, OpenComposite и решение проблем с песочницей Steam."
draft: false
series: ["Nix/NixOS"]
slug: "vr-wivrn-wayvr"
tags: ["nixos", "wivrn", "wayvr", "opencomposite", "steam", "quest3s"]
title: "NixOS: Полная настройка VR-стека (WiVRn + WayVR)"
aliases:
  - /blog/linux/nix/vr-wivrn-wayvr/
---

Запуск VR на NixOS требует понимания взаимодействия между OpenXR-рантаймами, композиторами и песочницей Steam. Для связки **Meta Quest 3S + Steam (Proton)** оптимальным решением является использование **WiVRn** в качестве стримера и OpenXR-рантайма, с опциональным использованием **OpenComposite** для обратной совместимости со старыми OpenVR-играми.

> 💡 **Критически важно:** VR-пакеты в NixOS развиваются очень быстро. Настоятельно рекомендуется использовать пакеты из ветки `nixos-unstable` или оверлея [nixpkgs-xr](https://github.com/nix-community/nixpkgs-xr).

---

## 📦 Базовая конфигурация NixOS

Создать или обновить модуль (например, `modules/vr.nix`). Данная конфигурация решает две основные проблемы NixOS VR: асинхронную репроекцию (требующую `CAP_SYS_NICE`) и изоляцию Steam (Pressure Vessel).

```nix
{ config, lib, pkgs, ... }:
{
  # 1. Загрузка модуля ядра для виртуального ввода (критично для WayVR)
  boot.kernelModules = [ "uinput" ];

  # 2. Добавление пользователя в необходимые группы
  users.users.ponfertato.extraGroups = [ "input" "plugdev" "adbusers" ];

  services.wivrn = {
    enable = true;
    autoStart = true;
    openFirewall = true;

    # Решает проблему фризов (асинхронная репроекция требует CAP_SYS_NICE)
    highPriority = true;

    # Автоматически пробрасывает PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1
    # Это позволяет играм в песочнице Steam видеть OpenXR-рантайм WiVRn
    steam = {
      enable = true;
      importOXRRuntimes = true;
    };

    # Переменные окружения для корректного захвата экрана в KDE Wayland
    monadoEnvironment = {
      XDG_CURRENT_DESKTOP = "KDE";
      XDG_SESSION_TYPE = "wayland";
      QT_QPA_PLATFORM = "wayland";
    };

    # Опционально: автоматический запуск WayVR из меню гарнитуры
    config = {
      enable = true;
      json = {
        application = [ pkgs.unstable.wayvr ];
      };
    };
  };

  environment.systemPackages = with pkgs.unstable; [
    wayvr         # Оверлей рабочего стола в VR
    opencomposite # Слой совместимости OpenVR -> OpenXR
    android-tools # Для ADB и проводного сопряжения при необходимости
  ];
}
```

---

## ⚠️ Решение типичных проблем

### 1. WayVR показывает только сквозную камеру (Pass-through)

Если WayVR запускается, но не отрисовывает интерфейс, а в логах есть ошибки `uinput` или `OpenXR-Loader`, выполнить следующие проверки:

- **Группа `input`:** Убедиться, что пользователь добавлен в группу `input` (как указано в конфиге выше) и выполнена **перезагрузка системы** или повторный вход в сессию. Без этого WayVR не может инициализировать виртуальные устройства ввода и блокирует рендеринг UI.
- **Порядок запуска:** Запускать WayVR строго после того, как гарнитура подключена к WiVRn.

  ```bash
  # 1. Убедиться, что сервис запущен
  systemctl --user status wivrn

  # 2. Если автоматический запуск не сработал, запустить через обёртку Steam
  # (это решает проблемы с доступом к сокетам NixOS FHS)
  steam-run wayvr --replace
  ```

- **Ограничения iGPU:** На старых интегрированных графических процессорах (например, Intel HD 530) композитинг Vulkan в WayVR может работать нестабильно. Если интерфейс не появляется, использовать встроенный **SteamVR Desktop** (запустить SteamVR из меню WiVRn и выбрать "Рабочий стол").

### 2. Игры Steam не видят OpenXR-рантайм

Даже при включённом `importOXRRuntimes = true`, некоторые игры могут требовать явного указания. Если игра вылетает или запускается в плоском режиме, добавить в её параметры запуска в Steam:

```text
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 %command%
```

Для старых игр, жёстко требующих OpenVR, использовать OpenComposite:

```text
env XR_RUNTIME_JSON=/run/current-system/sw/share/openxr/1/opencomposite_runtime.json %command%
```

### 3. Ошибка `failed to determine active runtime file path`

OpenXR-загрузчик не может найти манифест. WiVRn управляет этим автоматически, но при использовании кастомных настроек следует убедиться, что файл существует:

```bash
ls -l /run/current-system/sw/share/openxr/1/openxr_wivrn.json
```

При использовании Home Manager можно явно создать ссылку:

```nix
xdg.configFile."openxr/1/active_runtime.json".source = "${pkgs.unstable.wivrn}/share/openxr/1/openxr_wivrn.json";
```

---

## 🔄 Процесс подключения (Quest 3S)

1. Включить **Режим разработчика** в приложении Meta Horizon.
2. Подключить гарнитуру по USB-C. Разрешить отладку в окне гарнитуры.
3. Установить APK WiVRn (скачать последний релиз с GitHub):
   ```bash
   adb install WiVRn-vX.Y.Z.apk
   ```
4. Отключить USB. Убедиться, что ПК и гарнитура находятся в одной сети Wi-Fi (желательно 5 ГГц).
5. Запустить приложение WiVRn на гарнитуре. Оно автоматически найдёт ПК через Avahi (mDNS).
6. В меню гарнитуры выбрать запуск **WayVR** (или SteamVR, если WayVR не отображается).

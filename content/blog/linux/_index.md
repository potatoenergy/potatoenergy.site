---
description: "Linux как рабочая станция: NixOS, игры, приложения"
title: "Linux"
---

### Linux как рабочая станция

Настройка Linux для повседневного использования: декларативные конфигурации, игровые оптимизации, работа с приложениями и системными утилитами.

**Подразделы**

**NixOS** - декларативная конфигурация системы

- [Восстановление ФС и Nix Store](nixos/fs-recovery/) - критические сбои, e2fsck, hardlinks
- [Запуск "чужих" программ через Distrobox](nixos/distrobox-apps/) - Ubuntu/Debian в контейнере
- [Полная настройка VR-стека](nixos/vr-module/) - WiVRn, WayVR, OpenComposite, Quest 3S

**Gaming** - игры на Linux

- [steamscope.sh: универсальный лаунчер для Steam](gaming/steamscope-linux-gaming/) - Gamescope, Gamemode, MangoHud, FSR

**Приложения** - установка и исправления

- [Flatpak: ошибка 403 при загрузке OpenH264](apps/flatpak-openh264-fix/) - замена кодека на ffmpeg-full

**Утилиты** - системные инструменты

- [Ventoy: многозагрузочная флешка](utilities/ventoy-multiboot/) - структура, темы, автоустановка Windows

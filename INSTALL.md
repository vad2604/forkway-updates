# Установка Forkway (macOS)

Публичные сборки для **Apple Silicon (arm64)**. Требуется macOS 14+.

## Скачать

| Файл | Ссылка |
|------|--------|
| DMG (установщик) | [Forkway-1.0.4-macos-arm64.dmg](https://vad2604.github.io/forkway-updates/releases/v1.0.4/Forkway-1.0.4-macos-arm64.dmg) |
| Контрольная сумма | [Forkway-1.0.4-macos-arm64.dmg.sha256](https://vad2604.github.io/forkway-updates/releases/v1.0.4/Forkway-1.0.4-macos-arm64.dmg.sha256) |

Проверка целостности (рекомендуется):

```bash
cd ~/Downloads
curl -LO https://vad2604.github.io/forkway-updates/releases/v1.0.4/Forkway-1.0.4-macos-arm64.dmg.sha256
curl -LO https://vad2604.github.io/forkway-updates/releases/v1.0.4/Forkway-1.0.4-macos-arm64.dmg
shasum -a 256 -c Forkway-1.0.4-macos-arm64.dmg.sha256
```

## Установка

1. Откройте DMG, перетащите **Forkway** в **Applications**.
2. Снимите quarantine и запустите (ad-hoc сборка, без Apple Developer ID):

   ```bash
   xattr -cr /Applications/Forkway.app
   /Applications/Forkway.app/Contents/MacOS/ForkwayApp
   ```

   Альтернатива: ПКМ по Forkway → **Открыть** → **Открыть**.

3. В menu bar появится иконка вилки (в Dock приложения нет).

## Первый запуск

1. **Import AWG from Amnezia…** — импорт `.backup` из Amnezia VPN.
2. Подключите **CheckPoint Endpoint Security** (корпоративный VPN).
3. Нажмите **Start** — один раз введите пароль администратора (установка privileged helper).

Опционально: переключатель **AWG** над **Start** (только когда Forkway остановлен) — режим только DNS без AmnesiaWG.

## Обновления

После установки Forkway сам проверяет обновления (Sparkle). Вручную: меню Forkway → **Check for Updates…**

Feed: https://vad2604.github.io/forkway-updates/appcast.xml

## Удаление

```bash
osascript -e 'tell application "Forkway" to quit' 2>/dev/null || true
# из каталога с исходниками, если есть:
# ./scripts/reset-helper.sh
sudo rm -f /Library/PrivilegedHelperTools/com.forkway.helper
sudo rm -f /Library/LaunchDaemons/com.forkway.helper.plist
rm -rf /Applications/Forkway.app
# при необходимости: rm -rf ~/.config/forklift
```

## Диагностика

```bash
tail -f ~/.config/forklift/run/forkway-ui.log
```

| Проблема | Что сделать |
|----------|-------------|
| Двойной клик не запускает | `xattr -cr /Applications/Forkway.app`, запуск из Terminal (см. выше) |
| Нет иконки в menu bar | Запущен ли процесс: `pgrep -l ForkwayApp` |
| Start не поднимает маршрутизацию | CheckPoint подключён? AWG импортирован? |
| После обновления сломалось | Quit → Start снова; при необходимости переустановить helper |

## Для разработчиков

Исходники и сборка — приватный репозиторий. Техническая документация: [DISTRIBUTION.md](DISTRIBUTION.md).

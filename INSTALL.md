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

## Amnezia VPN (перед первым запуском)

Forkway поднимает AmnesiaWG **сам** — из конфига, импортированного из бэкапа Amnezia.

1. Откройте **Amnezia VPN** на Mac.
2. **Настройки** → **Создать резервную копию** (backup серверов) → сохраните файл `.backup`.
3. **Отключите VPN и закройте Amnezia** перед **Start** в Forkway.

> **Важно:** пока Forkway в работе (**Start**), Amnezia должна быть **выключена** (не подключена и лучше полностью закрыта). Два VPN-клиента одновременно конфликтуют за маршруты и tun-интерфейсы.

## Первый запуск

1. В Forkway: **Import AWG from Amnezia…** → выберите сохранённый `.backup` (при нескольких серверах выберите нужный).
2. Убедитесь, что **Amnezia VPN отключена**.
3. Подключите **CheckPoint Endpoint Security** (корпоративный VPN).
4. Нажмите **Start** — один раз введите пароль администратора (установка privileged helper).

Опционально: переключатель **AWG** над **Start** (только когда Forkway остановлен) — режим только DNS без AmnesiaWG.

## Обновления

После установки Forkway сам проверяет обновления (Sparkle). Вручную: меню Forkway → **Check for Updates…**

Feed: https://vad2604.github.io/forkway-updates/appcast.xml

### Обновление с предыдущей версии

1. **Check for Updates…** (или скачайте новый DMG с [страницы установки](https://vad2604.github.io/forkway-updates/)) и замените `Forkway.app` в **Applications**.
2. Запустите Forkway, нажмите **Start** — **один раз** может понадобиться пароль admin (обновление privileged helper).
3. Если появляется `unknown command: start-dns-only` — helper не обновился. В Terminal:

   ```bash
   sudo launchctl kickstart -k system/com.forkway.helper
   ```

   Затем снова **Start** в Forkway. Если не помогло — полный сброс helper (нужен доступ к `scripts/reset-helper.sh` из репозитория или попросите администратора выполнить):

   ```bash
   sudo launchctl bootout system/com.forkway.helper 2>/dev/null
   sudo rm -f /Library/LaunchDaemons/com.forkway.helper.plist
   sudo rm -f /var/log/forkway-helper.log
   rm -f ~/.config/forklift/run/helper.sock
   ```

   После этого **Start** в Forkway снова установит helper.

> Нужна версия приложения **не старее 1.0.4 (build 16+)** — в ней исправлено автообновление helper.

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
| Start не поднимает маршрутизацию | CheckPoint подключён? AWG импортирован? Amnezia выключена? |
| `unknown command: start-dns-only` | Устаревший helper — Quit Forkway, в Terminal: `sudo launchctl kickstart -k system/com.forkway.helper`, снова **Start** (или переустановите Forkway из свежего DMG) |
| Конфликт VPN / странные маршруты | Полностью выйдите из Amnezia, в Forkway **Stop** → **Start** |
| После обновления сломалось | Quit → Start снова; при необходимости переустановить helper |

## Для разработчиков

Исходники и сборка — приватный репозиторий. Техническая документация: [DISTRIBUTION.md](DISTRIBUTION.md).

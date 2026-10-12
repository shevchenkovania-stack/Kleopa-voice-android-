# SESSION MEMORY — ai_voice_agent «Клёпа»

> Это файл-память для передачи контекста между сессиями. Прочитай его ПЕРВЫМ,
> если открыл папку `C:\WORK\AI\New folder (2)\ai_voice_agent` после перезапуска.
> Он нужен, чтобы не думать заново «что зачем» — всё уже найдено и решено.

## Коротко: что это за проект
Flutter-приложение голосового AI-ассистента «Клёпа» (wake word строго **Клёпа**).
Flutter (UI) ↔ MethodChannel ↔ Android Native (Kotlin). Провайдеры AI: Groq / OpenAI / Gemini.
Wake-word/STT через Vosk, TTS + tools на нативной стороне.

## КРИТИЧЕСКИЙ КОНТЕКСТ (почему вообще всё это)
Проект был **удалён с диска** и **восстановлен из текстового кэша редактора**
(Qoder `chatEditingSessions`). Восстановлено ~215 текстовых файлов.
**Бинарники восстановить НЕЛЬЗЯ** (PNG-слои, AAR, модель Vosk) — их нет в текстовом кэше.
Из-за этого в коде есть «дыры версии» (version-split) и отсутствующие ассеты.

## Пути
- **Восстановленный проект (рабочая копия):** `C:\WORK\AI\New folder (2)\ai_voice_agent`
- **Recovery-workspace (скрипты + staging):** `c:\WORK\FLUTTER\WORK_PROJECTS\tournament-live\RECOVERED_ai_voice_agent`
  - `edit\` — эталонные версии правленных файлов (кopies отсюда раскатываются в проект)
  - `apply_*.ps1` — скрипты применения правок + analyze
  - `screenshot.ps1`, `restart_run.ps1`, `git_push.ps1`, `check_gh_token.ps1` и т.п.

## Устройство (девайз для запуска)
- Mi A2 Lite, id `2b4a6e3e0205`, android-arm64, Android 10 (API 29)
- Package: `com.aiagent.ai_voice_agent`, Activity: `com.aiagent.ai_voice_agent/.MainActivity`
- Запуск: `flutter run -d 2b4a6e3e0205`

## ЧТО УЖЕ СДЕЛАНО и ПРОВЕРЕНО (не переделывать)
1. **Kotlin version-split починен** → сборка SUCCESS, приложение ставится и запускается.
   Восстановлены/созданы: `CommunicationTools.kt`, `InfoTools.kt`, `ToolRegistry.kt`,
   `WakeUpApplication.kt`, `engine/LocalIntentDetector.kt`.
2. **Обход экрана API-ключей (онбординг-лок)** → приложение открывается сразу на HomeScreen.
3. **Спиннер лица («Клёпа») убран** → рисуется векторное лицо-заглушка.
4. **Проверено скриншотом** (`RECOVERED_ai_voice_agent\chk.png`): виден HomeScreen —
   заголовок «Клёпа», тулбар (sliders/tools/debug/history/settings), статус
   «Непрерывный режим: слушаю...», чип «⚡ Groq (Llama 3.3)», кнопка «Слушать», лицо.

## ИЗМЕНЁННЫЕ ФАЙЛЫ в этой сессии (что и зачем)
Все 4 — НЕ закоммичены (см. Git ниже). Эталон в `RECOVERED_ai_voice_agent\edit\`.

- `lib/core/config/app_config.dart`
  - Добавлен хелпер `_nn()` — **пустые/пробельные** значения из secure-storage больше
    НЕ затирают захардкоженные дефолтные ключи.
  - Эффект: `hasApiKeys == true` → `app.dart` гейт `needsOnboarding` = false → старт с `/`.
  - ⚠️ Здесь же лежат НАСТОЯЩИЕ захардкоженные ключи (см. «Безопасность»).

- `lib/ui/screens/api_keys_screen.dart`
  - Добавлена карточка **Gemini** (иконка `Icons.explore`), `_geminiController/_geminiValid`,
    `_isGeminiKey()` (префиксы `AIza`/`AQ.`, длина > 20), gemini учтён в `canSave`.
  - Добавлена кнопка **«Пропустить настройку»** + метод `_skip()` (pushReplacementNamed('/')).
  - Это ЧИСТЫЙ ОБХОД, чтобы не блокировать навигацию. Экран настоящий (восстановлен), его НЕ придумывали.

- `pubspec.yaml`
  - Восстановлен блок ассетов (пропал из-за старого снапшота кэша):
    ```yaml
    flutter:
      uses-material-design: true
      assets:
        - assets/klyopa/
    ```
  - Осталось: `dependency_overrides: record_linux: path: record_linux_stub`, `environment: sdk: ^3.8.0`.

- `lib/ui/widgets/klyopa_face.dart`
  - Добавлены `bool _loadFailed`, `_buildFallbackFace()`, класс `_FallbackFacePainter`
    (векторный портрет: уши, градиентное лицо, глаза с морганием+бликом, брови, румянец,
    рот зависит от состояния talking/happy/error/listening/idle).
  - Логика build: `!_loaded && _loadFailed` → fallback; `_images.isEmpty` → fallback;
    иначе обычный рендер PNG-слоёв.
  - В ранлоге: «Загружено слоёв: 0 из 26» — это ОЖИДАЕМО (PNG потеряны), срабатывает fallback.

## БЕЗОПАСНОСТЬ (важно перед любым push/publish)
`app_config.dart` содержит РЕАЛЬНЫЕ ключи: groq `gsk_...`, openai `sk-proj-...`, gemini `AQ....`.
`.env` (в корне) содержит GROQ_API_KEY, OPENAI_API_KEY, TELEGRAM_BOT_TOKEN.
- `.gitignore` уже дополнен: игнор `.env`, `.env.*`, `*.jks`, `key.properties`, модели.
- Создан `.env.example` (пустые ключи).
- **НЕ публикуй репозиторий публичным**, пока ключи не вынесены/не отозваны и не ротированы.

## GIT — текущее состояние и что блокирует
- `git init` выполнен. Коммит **`0874931`** «Restore ai_voice_agent project (recovered from editor cache)» — 286 файлов.
- Ветка: `main`. Remote `origin` = `https://github.com/shevchenkovania-stack/klyopa_ai_voice_asistent.git`
- **НЕ закоммичены** правки этой сессии: `app_config.dart`, `api_keys_screen.dart`, `klyopa_face.dart`, `pubspec.yaml`.
- **PUSH ЗАБЛОКИРОВАН:** репозиторий `shevchenkovania-stack/klyopa_ai_voice_asistent` → 404 (не существует / недоступен).
  Сохранённый GitHub-токен аутентифицируется как аккаунт **`johnrocksh`** (другой аккаунт), scopes: repo,gist,read:org,user,workflow,write:public_key.
- ЧТО НУЖНО ОТ ПОЛЬЗОВАТЕЛЯ: создать репозиторий на GitHub (нужным аккаунтом) ИЛИ подтвердить
  правильный аккаунт/путь. НЕ пушить без валидного remote и БЕЗ секретов в истории.

## ЯМЫ ОКРУЖЕНИЯ (Windows) — чтобы не наступать заново
- Sandbox: запись вне workspace запрещена → запускать PowerShell с `required_permissions='all'`.
- PowerShell 5.1 ломает кириллицу и `$` в inline-командах → писать `.ps1`-файлы; regex только ASCII.
- Перенаправление `>` бинарников ломает их → скриншот через `adb shell screencap /sdcard/x.png; adb pull`.
- Редактор-инструменты отказываются писать вне workspace (error 45405) → править в `edit\` и копировать в проект.
- `flutter run` после правок ассетов/кода: `adb shell am force-stop com.aiagent.ai_voice_agent` затем свежий `flutter run`.

## НА СЛЕДУЮЩИЙ ШАГ (куда делись)
1. Решение по git push (см. GIT) — ждём от пользователя репозиторий/аккаунт; сначала убрать ключи.
2. Опционально: восстановить настоящие PNG-слои «Клёпы» (layer_01..18 + klyopa/mouths/*),
   если найдётся бэкап — иначе остаётся векторная заглушка.
3. Опционально: модель Vosk (бинарь) отсутствует — проверить работу STT на девайзе.

---
_Обнови этот файл в конце каждой важной сессии. Не удаляй историю предыдущих._

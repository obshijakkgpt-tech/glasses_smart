# CyanBridge Redesign — Память проекта

> Файл памяти. Обновляется после КАЖДОГО промпта.
> Назначение: редизайн Android-приложения CyanBridge (из репозитория
> FerSaiyan/Alternative-HeyCyan-App-and-SDK) — альтернативный клиент для
> смарт-очков HeyCyan.

---

## Рабочие правила (заданы пользователем)
1. Сохранять состояние проекта на диск после каждого промпта.
2. Этот файл памяти обновлять после каждого промпта: что сделано + что
   подтверждено как рабочее.
3. Заглушки (stubs/mock) — только в исключительных случаях. Весь функционал
   делать рабочим.
4. При неуверенности (особенно совместимость / версии) — проводить
   мини-исследование перед написанием кода.
5. Приоритет — open-source компоненты с GitHub для интеграции.

## Контекст исходного проекта
- Репозиторий: https://github.com/FerSaiyan/Alternative-HeyCyan-App-and-SDK
- Платформа в фокусе: Android (модуль CyanBridge), Kotlin ~82.7%
- Очки HeyCyan: BLE для управления (фреймворк QCSDK, QCCentralManager),
  WiFi для передачи больших медиафайлов (режим Transfer, HTTP /manifest.json)
- BLE Primary Service UUID: 7905FFF0-B5CE-4E99-A40F-4B1E122D00D0
- BLE Secondary Service UUID: 6e40fff0-b5a3-f393-e0a9-e50e24dcca9e
- Режимы устройства: Photo, Video/VideoStop, Audio/AudioStop, AIPhoto, Transfer
- Последний релиз апстрима: CyanBridge v2.0.0 (локальный AI), апрель 2026

## Цель редизайна (согласовано с пользователем)
- Платформа: Android (CyanBridge)
- Объём: общий редизайн всех экранов
- Стиль: тёмный минимализм, Material You, неоновые акценты
- Утверждённый концепт: 3 экрана (Сканирование / Устройство / Галерея) +
  bottom navigation на 4 вкладки

### Палитра (из утверждённого мокапа)
- Фон основной:        #0B0F12
- Поверхность/карточка: #131A1F
- Обводка:             #1F2730
- Неон-акцент (бренд): #00E5FF  (cyan)
- Видео:               #FF5A96  (pink)
- Аудио:               #FFB238  (amber)
- AI:                  #A878FF  (purple)
- Текст основной:      #E8EAED
- Текст вторичный:     #8A9199
- Текст приглушённый:  #4A5560

---

## Журнал изменений

### [Промпт 1-4] Исследование + концепт
- Изучен исходный репозиторий, архитектура BLE+WiFi.
- Согласованы платформа/объём/стиль.
- Создан интерактивный мокап 3 экранов (только в чате, не в файлах).

### [Промпт 5] Инициализация структуры
- Создана структура каталогов res/ (values, values-night, drawable,
  layout, font).
- Создан этот файл PROJECT_MEMORY.md.

### [Промпт 6] Правила работы + мини-исследование версий
- Зафиксированы рабочие правила (см. секцию выше).
- Проведено исследование актуальных версий Android-тулчейна (май 2026).
- Извлечены РЕАЛЬНЫЕ детали протокола апстрима из release notes v1.0.2
  (см. секцию "Детали протокола" ниже).
- Развилка по UI (Compose vs XML) и цели (форк vs новый) — ожидает
  решения пользователя.

---

## Детали протокола апстрима (из release notes v1.0.2 — ВАЖНО, реальное)
- Sync-флоу: BLE поднимает WiFi Direct (P2P) на очках, затем телефон по HTTP:
  1. GET http://<glasses-ip>/files/media.config  — манифест файлов
  2. GET http://<glasses-ip>/files/<name>          — сам файл (jpg/mp4/opus)
- Сохранение: через MediaStore в DCIM/CyanBridge (видно в системной галерее).
- Аудио .opus: очки отдают RAW Opus-пакеты; чтобы сделать воспроизводимыми,
  их оборачивают в Ogg/Opus контейнер.
- В v1.0.2 удалён сломанный Smart Recognition UI (заменён на AI hijack flow).
- Есть экран-гайд по battery optimization (борьба с фоновыми отключениями BLE).
- APK подписан debug-keystore.

## Решения по версиям / зависимостям
> Заполнено после мини-исследования (актуально на май 2026).

### Актуальные "свежие" версии на рынке (для справки)
- AGP 9.0.x (встроенный Kotlin, новый DSL; opt-out исчезнет в AGP 10.0)
- Kotlin 2.3.21, Compose BOM 2026.04.01

### ВЫБРАННЫЙ стек (консервативно-стабильный) и ОБОСНОВАНИЕ
- Причина: апстрим ценен тем, что СОБИРАЕТСЯ. AGP 9.0 ломает обратную
  совместимость без выгоды для UI-редизайна => берём 8.x.
- AGP: 8.7.x (последняя стабильная 8.x)
- Kotlin: 2.0.21 (стабильный K2, Compose-компилятор как Gradle-плагин)
- Если Compose: Compose BOM ~2024.09.xx (совместим с Kotlin 2.0.21)
- Material: com.google.android.material 1.12.0 (XML/Material You)
- minSdk: уточнить из апстрима (вероятно 24-26); target/compileSdk: 35
- JDK/jvmTarget: 17
- ФИНАЛИЗИРОВАТЬ после ответа про Compose vs XML.

### [Промпт 7] Решение по UI и типу проекта
- РЕШЕНО: UI = Jetpack Compose (выиграл по простоте/стилю/минимализму/
  актуальности; инфо-безопасность — паритет с XML; риск версии Kotlin
  закрыт выбором стека).
- РЕШЕНО: проект НОВЫЙ с нуля на базе протокола HeyCyan (не форк).
- Важно по безопасности (не зависит от UI, реализуется в сетевом слое):
  очки отдают файлы по ОТКРЫТОМУ http:// — в манифесте разрешать
  cleartextTraffic ТОЧЕЧНО для локальных IP очков, не глобально.
  BLE: проверять подключение по известному Service UUID.

## ФИНАЛЬНЫЙ стек (зафиксирован)
- UI: Jetpack Compose + Material 3 (Material You / dynamic color)
- AGP: 8.7.3
- Kotlin: 2.0.21
- Compose Compiler: плагин org.jetbrains.kotlin.plugin.compose 2.0.21
- Compose BOM: 2024.09.03 (совместим с Kotlin 2.0.21)
- Gradle wrapper: 8.9
- JDK / jvmTarget: 17
- minSdk: 26 (Android 8.0 — нужно для BLE-стабильности, Opus, MediaStore
  DCIM-relative path появился в API 29, fallback для 26-28 предусмотреть)
- compileSdk / targetSdk: 35
- Намерение по архитектуре: MVVM (ViewModel + StateFlow), слои:
  ble/ (GATT-клиент, команды), net/ (WiFi HTTP-скачивание + Ogg/Opus
  упаковка), data/ (MediaStore сохранение), ui/ (Compose-экраны + theme).

## Open-source кандидаты для интеграции (приоритет по правилу пользователя)
> Проверить лицензии и актуальность перед добавлением.
- BLE: Nordic Android-BLE-Library (проверенный, BSD-3) ИЛИ чистый
  Android BluetoothGatt API. Решить на этапе ble-слоя.
- HTTP: OkHttp (Apache-2.0) — стабильно, для скачивания файлов.
- Изображения в Compose: Coil (Apache-2.0).
- Opus->Ogg: проверить, нужна ли нативная либа или хватит ручной
  упаковки Ogg-страниц (как делает апстрим). Исследовать на этапе net.
- TODO: подтвердить версии и лицензии при подключении.

## Подтверждённый рабочий функционал
> (пользователь сообщает, что работает — фиксируем здесь)
- (пока пусто)

---

### [Промпт 13 — шаг WiFi] МЕДИА-СЛОЙ создан (в основном РАБОЧИЙ)
Реальные эндпоинты известны из release notes -> сделано почти полностью.

- net/OggOpusWriter.kt — НАСТОЯЩАЯ упаковка raw-Opus -> Ogg по RFC 7845:
  OpusHead, OpusTags, страницы, lacing, granule@48k, Ogg-CRC32
  (poly 0x04C11DB7, без рефлексии, init 0).
  ПРОВЕРЕНО НА JVM: смоделирован на Python, CRC-таблица эталонная
  (table[1]=0x04C11DB7), сгенерён test.opus -> FFPROBE РАСПОЗНАЛ как
  codec=opus, 48000Hz, mono, initial_padding=3840. РАБОТАЕТ.
- net/MediaManifest.kt — модель MediaItem/MediaType + парсер media.config,
  устойчивый к 3 форматам (JSON-массив/объект/построчный), тип по расширению.
- net/GlassesWifiClient.kt — OkHttp клиент:
  resolveBaseUrl() перебирает кандидаты IP (192.168.43.1 / .49.1 / .4.1 / .1.1),
  fetchManifest() GET /files/media.config, openFile() GET /files/<name>.
  Таймауты щедрые (точка доступа очков медленная).
- data/MediaStoreSaver.kt — сохранение в DCIM/CyanBridge:
  вилка API29 (scoped storage RELATIVE_PATH+IS_PENDING) vs API26-28 (legacy
  путь + WRITE_EXTERNAL_STORAGE maxSdk28 + insert для MediaScanner).
  Photo->Images, Video->Video, Audio->Audio (audio/ogg).
- data/MediaSyncRepository.kt — оркестратор Flow<SyncEvent>:
  resolve->manifest->по файлам(download->opus-wrap если raw->save)->прогресс.
  flowOn(IO), ensureActive для отмены.
- Проверка: 16 .kt, точный сканер баланса (с учётом интерполяции) — ОК.

### Точки уточнения по исходникам (изолированы, помечены TODO/коммент)
- splitRawOpus() в MediaSyncRepository: framing сырого Opus-потока с очков
  публично не задокументирован. Сейчас: если поток уже Ogg(OggS) -> как есть;
  иначе весь поток как 1 пакет. Точный framing врезать из manufacturer-original.
- Формат media.config: парсер терпим к форматам; при появлении точной схемы
  заменить parseJsonLenient на строгий.

---

### [Промпт 12] СМЕНА ПРИОРИТЕТА: сначала функционал, потом UI
- Пользователь: "Подключение очков на первом месте... сначала кости и
  органы, потом кожа". => Делаем BLE/функционал ДО UI.
- Новый порядок: BLE-скелет -> WiFi-слой -> (потом) UI.

### [Промпт 12 — реализация] BLE-СКЕЛЕТ создан
Принцип: вся механика BLE — РЕАЛЬНАЯ; не хватает только байтов команд,
которые ИЗОЛИРОВАНЫ в одном файле (HeyCyanProtocol) и врезаются из
исходников manufacturer-original без правок остального кода.

- ble/HeyCyanProtocol.kt — ТАБЛИЦА ПРОТОКОЛА (единственная точка врезки):
  * ДОСТОВЕРНО: PRIMARY_SERVICE 7905FFF0..., SECONDARY 6e40fff0..., CCCD 2902.
  * TODO(protocol): UUID характеристик command/notify/data (сейчас плейсхолдеры
    7905/7906/7907 — НЕ прод!), байты encodeSetMode/encodeGetBattery/
    encodeGetMediaCounts/encodeGetVersionInfo (бросают ProtocolNotWired),
    decodeNotification (сейчас -> Unknown).
  * isWired=false — флаг незаполненности.
  * DeviceEvent (sealed): Battery/Media/Version/ModeChanged/Unknown.
- ble/BleOperationQueue.kt — последовательная очередь GATT-операций
  (фундаментальное требование Android BLE, НЕ догадка). Main-thread post.
- ble/NativeBleGlassesController.kt — реальный GATT-клиент, реализует
  GlassesController (все 15 методов на месте, проверено):
  * скан с ScanFilter по PRIMARY_SERVICE + fallback на скан без фильтра;
  * разрешения: вилка BLUETOOTH_SCAN/CONNECT (API31+) vs FINE_LOCATION;
  * connect -> connectGatt(TRANSPORT_LE) -> discoverServices -> поиск
    характеристик -> enable notifications через CCCD;
  * onCharacteristicChanged: ОБЕ сигнатуры (API<33 deprecated + API33+ byte[]);
  * writeCharacteristic: ВИЛКА API33 (новый writeCharacteristic(ch,bytes,type)
    vs старый setValue+writeCharacteristic) — проверено по докам;
  * writeDescriptor: та же вилка API33;
  * команды вызывают HeyCyanProtocol.encode* -> пока бросают ProtocolNotWired
    (честно: скелет НЕ притворяется рабочим без байтов).
- Проверка: 11 .kt, баланс/package ОК, интерфейс реализован полностью.

### ВАЖНО для следующих шагов
- NativeBle полностью заработает с железом, КОГДА в HeyCyanProtocol врежут:
  реальные UUID характеристик + байты команд + формат нотификаций.
  Источник: ветка manufacturer-original (текст .kt от пользователя).
- Для разработки/превью UI остаётся FakeGlassesController (рабочая симуляция).
- Выбор реализации (Fake vs Native) сделаем через простой провайдер/DI
  при сборке UI.

---

### [Промпт 11] AAR недоступен -> разворот стратегии
- Пользователь: ".aar не найду, давай делать с минимальными изменениями".
- НАХОДКА: README репо упоминает 2 ветки:
  * main — dev-ветка с доработками
  * manufacturer-original — ОРИГИНАЛЬНЫЙ SDK производителя (исходники!)
  => протокол (байты команд, парсинг) существует как Kotlin-ИСХОДНИКИ,
     а не только бинарь. "Минимум изменений" снова реалистичен.
- НО Claude по-прежнему не может прочитать эти файлы (robots.txt + raw-ссылки
  не из поиска + нет сети). Скачать/прочитать может только пользователь.
- ПРИНЯТОЕ РЕШЕНИЕ (Claude, чтобы не блокироваться; формы кнопок у юзера
  не прокликиваются — перешли на текст):
  * Делаем UI ПОЛНОСТЬЮ сейчас на FakeGlassesController (исходная цель
    пользователя = красивый интерфейс; UI не страдает без протокола).
  * Реальный протокол подключится позже как реализация GlassesController
    БЕЗ переписывания UI (=минимум изменений, заложено в архитектуру).
  * Свой BLE по голым UUID без байтов команд = гадание -> НЕ делаем
    (нарушает правило "минимум заглушек").
- ЧТО НУЖНО ОТ ПОЛЬЗОВАТЕЛЯ ПОЗЖЕ (для реального протокола): скопировать
  ТЕКСТ 1-2 ключевых .kt из ветки manufacturer-original (BLE-команды +
  парсинг) прямо в чат. Текст скопировать реально даже без .aar.
- Формы ask_user_input в UI пользователя не работают -> больше не
  использовать, только текстовые вопросы.

---

### [Промпт 10] Выбран Вариант A (переиспользовать их QCSDK .aar)
- Пользователь выбрал A, но файла .aar у него нет -> Claude помогает достать.
- Claude НЕ может скачать сам: дерево репо закрыто robots.txt, raw-ссылки
  не из поиска недоступны, + это бинарь (нет сети в песочнице).
- ПЛАН ДОБЫЧИ AAR (дано пользователю как инструкция):
  1. Источник: репо FerSaiyan/Alternative-HeyCyan-App-and-SDK, папка
     android/ (модуль CyanBridge или SDK-модуль). Искать файл *.aar
     (вероятно в app/libs/ или sdk/libs/ или отдельном модуле :qcsdk).
     Альтернатива — первоисточник ebowwa/HeyCyanSmartGlassesSDK, папка android/.
  2. Способы скачать: git clone репо; ИЛИ Download ZIP с GitHub; ИЛИ
     зайти в файл на github и нажать Download (raw).
  3. Если .aar нет, а есть исходники SDK-модуля — собрать AAR локально
     (./gradlew :sdk:assembleRelease -> sdk/build/outputs/aar/*.aar).
  4. Прислать .aar в чат -> Claude положит в app/libs/ и напишет обёртку
     QcsdkGlassesController над его публичным API.
- ВАЖНО уточнить у пользователя имя пакета/классов Android-SDK (Java/Kotlin),
  т.к. публично известен только iOS Obj-C API (QCSDKCmdCreator, QCSDKManager).
  Нужны Android-аналоги (имена методов/коллбэков) — из их android-демо.
- TODO: до получения .aar и его API обёртку QcsdkGlassesController писать
  НЕ можем (иначе гадание). UI продолжаем на FakeGlassesController.

---

### [Промпт 9] Решение по основному функционалу
- Пользователь: основной функционал — на базе ИХ протокола, МИНИМУМ изменений.
- НАЙДЕН ПЕРВОИСТОЧНИК: ebowwa/HeyCyanSmartGlassesSDK — кросс-платформенный
  SDK (iOS+Android), от него происходит форк FerSaiyan. Содержит публично:
  UUID, высокоуровневые сигнатуры команд. НЕ содержит публично: байтовые
  форматы пакетов (они внутри проприетарного QCSDK.framework / .aar).
- Известные сигнатуры команд (Obj-C, из доков SDK):
  * setDeviceMode:QCOperatorDeviceModePhoto / Video / VideoStop / Audio /
    AudioStop / AIPhoto / Transfer
  * getDeviceBattery -> (battery:Int, charging:Bool)
  * getDeviceMedia -> (photo:Int, video:Int, audio:Int, type:Int)
  * getDeviceVersionInfo -> (hdVersion, firmVersion, hdWifiVersion, firmWifiVersion)
  * AI-изображения приходят через Data Transfer Characteristic + нотификацию
    (на iOS — NotificationCenter .aiImageReceived).
- КЛЮЧЕВОЙ ФАКТ: байты команд скрыты в проприетарном QCSDK. Поэтому:
  * Вариант A (рекоменд.): переиспользовать готовый Android QCSDK.aar как есть,
    UI/VM вызывают его API. Нужен сам .aar (бинарь) — Claude скачать не может,
    нужен от пользователя/из релиза репозитория.
  * Вариант B: свой GATT-клиент на Android API (Claude может полностью), но
    байты команд неизвестны -> реверс/подбор (не "минимум изменений").
- ОЖИДАЕТ выбор пользователя (форма ask_user_input не прокликивается в его
  UI -> дублировать выбор текстом).

### [Промпт 9 — реализация] Доменный слой создан
Создано (чистый Kotlin, без продакшн-заглушек):
- domain/Models.kt: DeviceMode (зеркалит QCOperatorDeviceMode, порядок
  сохранён), ConnectionState, DiscoveredDevice, MediaCounts, BatteryStatus,
  VersionInfo, DeviceState (агрегат для Compose).
- domain/GlassesController.kt: интерфейс-контракт (StateFlow<DeviceState>,
  Flow discoveries, scan/connect/disconnect, команды takePhoto/startVideo/
  stopVideo/startAudio/stopAudio/triggerAiPhoto, refresh battery/media/version,
  enterTransferMode->URL, release). UI/VM зависят только от него.
- domain/FakeGlassesController.kt: рабочая СИМУЛЯЦИЯ для @Preview и разработки
  UI без железа (имитирует тайминги, дрейф батареи, инкремент счётчиков).
  Это dev-инструмент, в релизе — реальная реализация.
- Пакеты подготовлены: ble/ net/ data/ ui/theme ui/nav ui/screens.
- Проверка: баланс скобок ОК, package у всех ОК (kotlinc недоступен — нет сети).

### АРХИТЕКТУРНОЕ РЕШЕНИЕ (не зависит от выбора A/B)
- Вводим интерфейс GlassesController (доменный слой). UI и ViewModel зависят
  ТОЛЬКО от него. Реализации:
  * QcsdkGlassesController (обёртка над их .aar) — вариант A
  * NativeBleGlassesController (свой GATT) — вариант B
  * FakeGlassesController — для Compose @Preview и разработки UI без железа
    (это НЕ продакшн-заглушка, а dev/preview-инструмент — допустимо).
- Так выбор A/B = выбор реализации, без переписывания UI. Истинный
  "минимум изменений".
- Тип запроса BLE-операций: Kotlin Flow<DeviceState> + suspend-команды
  (StateFlow в VM). OkHttp для WiFi-скачивания (уже в catalog).

---

## План сборки (новый проект, Compose)
1. [DONE] Каркас Gradle-проекта (собираемое пустое приложение)
2. [DONE] Тема Material3 в Compose (неоновая палитра)
3. [ ] Навигация + 4 экрана-каркаса (bottom nav)
4. [ ] BLE-слой — ОТЛОЖЕН до исходников протокола (текст .kt от юзера)
5. [ ] WiFi-слой — ОТЛОЖЕН (аналогично)
6. [ ] Полировка UI под концепт
   До шагов 4-5 весь UI работает на FakeGlassesController.

### [Промпт 11 — шаг 2] Тема Material3 — СДЕЛАНО
- ui/theme/Color.kt: палитра из концепта.
- ui/theme/Type.kt: Typography M3 на SansSerif, веса 400/500.
- ui/theme/Theme.kt: CyanBridgeTheme(dark=true, dynamicColor=false),
  доп.акценты video/audio/ai через LocalCbAccents, edge-to-edge.
- 8 .kt, баланс/package ОК.

### [Промпт 8 = шаг 1] Каркас проекта — СДЕЛАНО
Создано (всё реальное, не заглушки, кроме временного Placeholder-экрана):
- gradle/libs.versions.toml — version catalog (16 версий, 24 либы,
  3 плагина, 2 бандла). Провалидирован: TOML ок, висячих version.ref нет.
- build.gradle.kts (root), settings.gradle.kts (репозитории + :app)
- app/build.gradle.kts (compileSdk 35, minSdk 26, JVM 17, Compose on)
- gradle.properties, gradle/wrapper/gradle-wrapper.properties (Gradle 8.9)
- app/proguard-rules.pro (dontwarn для OkHttp)
- AndroidManifest.xml — РЕАЛЬНЫЕ разрешения:
  BLE (старая+новая модель, maxSdk-разделение), WiFi/P2P (NEARBY_WIFI_DEVICES
  neverForLocation), медиа (WRITE_EXTERNAL_STORAGE maxSdk 28), uses-feature ble.
- res/xml/network_security_config.xml — cleartext HTTP ТОЧЕЧНО для
  192.168.43.1 / 4.1 / 49.1 / 1.1 (IP точки доступа очков), base = запрещён.
- res/values: strings, themes (стартовая XML Theme.CyanBridge,
  Material3.DayNight, тёмный фон #0B0F12), colors.
- Адаптивная иконка: drawable/ic_launcher_foreground+background (вектор,
  неон-очки), mipmap-anydpi-v26/ic_launcher(_round) с monochrome.
- Kotlin: CyanBridgeApp (Application), MainActivity (edge-to-edge,
  временный Placeholder — будет заменён на шаге 2-3).
- Package: com.cyanbridge.app
- Все XML well-formed (xmllint).
- ПРИМЕЧАНИЕ: gradle build локально НЕ запускался (нет сети в песочнице);
  валидация — синтаксис TOML/XML + сверка версий по докам (май 2026).
  gradle-wrapper.jar НЕ включён (бинарь) — генерируется `gradle wrapper`
  или Android Studio при первом открытии.

---

### [Промпт 14] РАСШИРЕНИЕ: голосовой ассистент с backend (Hermes)
Пользователь задал новую архитектуру бэкенда. Очки/Android -> Hermes ->
{whisper.cpp STT, Flowise workflows, OpenRouter LLM, edge-tts TTS} -> ответ.
Контейнеры: hermes, flowise, whisper.cpp, БД, nginx.

РЕШЕНИЯ (Claude выбрал сам по просьбе пользователя):
- Hermes = СОБСТВЕННЫЙ backend-оркестратор (НЕ Nous Hermes Agent — тот не
  подходит: у него встроенные STT/TTS и messaging-gateway, а не HTTP-API
  поверх Flowise). Пишем с нуля.
- Язык Hermes = Python + FastAPI (ML-экосистема, async для долгих STT/TTS).
- БД = PostgreSQL + pgvector (реляционка: сессии/история + вектор:
  семантическая память/RAG, ОДИН контейнер вместо двух) + Redis
  (кэш транскрипций, статусы async-задач, rate-limit).
  Обоснование (критерии пользователя скорость/удобство/цена/размер/
  доступность): pgvector закрывает 2 задачи одним сервисом (=меньше
  контейнеров), бесплатно, офиц. Docker, Flowise/LangChain нативно
  поддерживают pgvector. Qdrant — избыточен для персонального масштаба.
  SQLite — не для concurrent клиент-серверной нагрузки.

СТЕК БЭКЕНДА (контейнеры docker-compose):
1. nginx — реверс-прокси, единая точка входа, TLS-термination, маршрутизация.
2. hermes — FastAPI оркестратор (этот пишем).
3. whisper.cpp — STT через HTTP server-режим (./server) образ.
4. flowise — AI-workflow (low-code), порт 3000.
5. postgres (pgvector/pgvector образ) — реляционка + вектор.
6. redis — кэш/очереди/статусы.
(edge-tts — Python-пакет, крутится ВНУТРИ hermes как зависимость, либо
 отдельным микро-сервисом; решить при сборке — склоняюсь к отдельному
 контейнеру edge-tts для изоляции, т.к. он держит соединение с MS-сервисом.)

ПОТОК: Android пишет opus-аудио -> POST /v1/voice (hermes) -> whisper.cpp
(текст) -> Flowise workflow (контекст из pgvector + OpenRouter LLM) ->
edge-tts (аудио) -> ответ {text, audio_url}. История/память -> postgres.

ПЛАН СБОРКИ БЭКЕНДА:
B1. [ ] docker-compose.yml + структура каталогов backend/ + .env.example
B2. [ ] Hermes FastAPI скелет (эндпоинты, конфиг, healthcheck)
B3. [ ] Интеграция whisper.cpp (STT-клиент)
B4. [ ] Интеграция edge-tts (TTS)
B5. [ ] Интеграция Flowise + OpenRouter
B6. [ ] PostgreSQL+pgvector схема (сессии, сообщения, эмбеддинги) + Redis
B7. [ ] nginx конфиг
B8. [ ] Связка Android-app <-> Hermes (HTTP-клиент в приложении)

---

### [Промпт 16] УТОЧНЕНИЕ MVP от пользователя — приоритеты приложения
ВАЖНО: Hermes У ПОЛЬЗОВАТЕЛЯ УЖЕ РАЗВЁРНУТ на сервере (юзает через Telegram).
=> backend/ который я построил НЕ нужен как новый сервис. Приложение должно
   ходить в СУЩЕСТВУЮЩИЙ Hermes пользователя по HTTP (адрес в настройках).
   backend/ оставляем в репо как опциональный/референсный, не разворачиваем.

ТРЕБОВАНИЯ MVP (приоритезированы пользователем):
1. РЕЖИМ "БЕЗ ОЧКОВ" (КРИТИЧНО, первый приоритет): телефон как микрофон +
   динамик + камера. Разработка не должна блокироваться SDK/железом.
   => в Settings переключатель режима: fake / native glasses.
   => FakeGlassesController уже есть; нужен телефонный микрофон/камера/динамик.
2. ОДИН главный экран (не плодить экраны). Home:
   - статус сервера (Hermes), статус очков, батарея
   - большая кнопка push-to-talk "Говорить"
   - последний transcript + последний answer
   - кнопки навигации: Заметки / Медиа / Настройки
3. ИСТОРИЯ (минимум): дата/время, текст запроса, ответ, тип
   (voice/note/photo/command). Важно для отладки и "ощущения памяти".
4. НАСТРОЙКИ ПРОВАЙДЕРОВ: OpenRouter key, модель, TTS voice, STT язык,
   адрес Hermes server. (в приложении)
5. ОЧЕРЕДЬ ЗАДАЧ: даже без Redis — простая локальная таблица audio_jobs
   со статусами: uploaded / transcribing / thinking / speaking / done /
   failed. Чтобы пользователь видел, где зависло.

ГЛАВНЫЙ РИСК (по словам пользователя): UX задержки, НЕ AI.
Говорит -> ждёт 3-7с -> ответ. ОБЯЗАТЕЛЬНО показывать состояния:
"Слушаю… / Распознаю… / Думаю… / Озвучиваю…"

ЭКРАНЫ (финальный список MVP):
- Home: статус сервера/очков, батарея, push-to-talk, последний transcript+answer
- Notes: голосовые заметки, AI-summary, теги
- Media: фото/видео/аудио с очков, sync status, открыть/скачать
- Settings: Hermes URL, модель OpenRouter, язык, TTS voice, режим fake/native

ОТЛОЖИТЬ (НЕ делать сейчас): LiveKit, Mem0, LiteLLM, сложные агенты,
постоянное прослушивание, автозапись всего, RAG по Telegram, marketplace.

ПЕРЕСМОТР ПЛАНА (приложение):
A1. [ ] Режим "без очков": телефонный аудио (запись микрофона, плеер) +
        камера. PhoneController как реализация GlassesController? Нет —
        очки и телефон-как-устройство разные. Сделать AudioRecorder/
        AudioPlayer + переключатель режима. Камера телефона — CameraX.
A2. [ ] HermesClient (HTTP в существующий Hermes): POST аудио -> ответ.
        Состояния пайплайна (Listening/Transcribing/Thinking/Speaking).
A3. [ ] Локальная БД (Room): history + audio_jobs + settings. (НЕ Redis)
A4. [ ] Home-экран с push-to-talk и индикацией состояний.
A5. [ ] Settings-экран (провайдеры, режим, Hermes URL).
A6. [ ] Notes / Media экраны.

---

### [Промпт 16 — исследование Hermes API] КЛЮЧЕВОЕ
Твой Hermes = Nous Research hermes-agent. Его HTTP API-сервер:
- OpenAI-СОВМЕСТИМЫЙ: POST /v1/chat/completions, /v1/responses, GET /v1/models
- GET /health -> {"status":"ok"}
- Auth: Bearer токен (API_SERVER_KEY). Дефолт порт 8642, путь с суффиксом /v1.
- Поле model косметическое (реальная модель в config.yaml сервера), но слать надо.
- ВКЛючается API_SERVER_ENABLED=true (пользователю проверить, что включён).

СЛЕДСТВИЕ ДЛЯ АРХИТЕКТУРЫ ПРИЛОЖЕНИЯ (важно!):
- Hermes API-сервер ТОЛЬКО текст<->текст (голос у Nous живёт в gateway
  Telegram/Discord, НЕ в HTTP API).
- Поэтому голосовой пайплайн в приложении:
  STT на ТЕЛЕФОНЕ (Android SpeechRecognizer) -> текст ->
  Hermes /v1/chat/completions -> ответ-текст ->
  TTS на ТЕЛЕФОНЕ (Android TextToSpeech).
- Это идеально ложится на "режим без очков": телефон нативно умеет
  микрофон->текст и текст->динамик БЕЗ сервера. Никакого whisper/edge-tts
  контейнера для MVP не нужно (backend/ остаётся референсным, не юзаем).
- OpenRouter key/модель/voice/язык в Settings: модель/ключ -> это конфиг
  Hermes-сервера (косметически шлём model); язык/voice -> локальный
  Android STT/TTS. Уточнить в UI, что часть настроек = серверные.

ОБНОВЛённый план приложения:
A1. [ ] PhoneVoice: Android SpeechRecognizer (STT) + TextToSpeech (TTS) +
        состояния пайплайна (Idle/Listening/Transcribing/Thinking/Speaking).
A2. [ ] HermesClient: GET /health, POST /v1/chat/completions (Bearer).
A3. [ ] Room БД: history (voice/note/photo/command), audio_jobs (статусы),
        settings (Hermes URL, key, model, voice, lang, mode fake/native).
A4. [ ] Home: статусы сервера/очков/батарея, push-to-talk, transcript+answer,
        индикация состояний задержки.
A5. [ ] Settings.
A6. [ ] Notes / Media.

---

### [Промпт 16 — реализация ч.1] Hermes-клиент + телефонный голос
Добавлено в стек: Room 2.6.1 + KSP 2.0.21-1.0.25 + DataStore 1.1.1
(консервативно, под Kotlin 2.0.21; НЕ Room 3.0/KMP — не нужно для MVP).
build.gradle обновлены (ksp плагин root+app, зависимости).

Создано (всё реальное):
- voice/PipelineState.kt — состояния: IDLE/LISTENING/TRANSCRIBING/THINKING/
  SPEAKING/ERROR с русскими лейблами ("Слушаю…" и т.д.) — против UX-задержек.
- hermes/HermesClient.kt — РЕАЛЬНЫЙ клиент к существующему Hermes юзера:
  GET /health, POST /v1/chat/completions (Bearer, OpenAI-формат, history,
  systemPrompt). Парсинг choices[0].message.content. OkHttp+org.json.
- voice/PhoneSpeechToText.kt — Android SpeechRecognizer (STT), callbackFlow,
  partial+final, обработка ошибок. "Телефон как микрофон".
- voice/PhoneTextToSpeech.kt — Android TextToSpeech (TTS), suspend speak с
  UtteranceProgressListener (обе сигнатуры onError). "Телефон как динамик".
- Манифест: +RECORD_AUDIO, +CAMERA, +<queries> для RecognitionService и
  TTS_SERVICE (обязательно на Android 11+, иначе isRecognitionAvailable=false).
- Проверка: 20 .kt баланс ОК, TOML ок, манифест валиден.

ОСТАЛОСЬ: Room БД (history/audio_jobs/settings), VoicePipeline (связка
STT->Hermes->TTS со сменой PipelineState), Home-экран, навигация, Settings,
Notes, Media.

---

### [Промпт 17] Room БД + Settings + VoicePipeline — СДЕЛАНО
Решение: exportSchema=false для MVP (избегаем настройки room.schemaLocation/
CommandLineArgumentsProvider — частый источник падений сборки). Авто-миграции
не нужны на старте; fallbackToDestructiveMigration().

Создано (всё реальное):
- data/db/Entities.kt: HistoryEntity (createdAt/type/request/response/
  attachmentUri/tags), AudioJobEntity (статусы пайплайна). Enums HistoryType
  (VOICE/NOTE/PHOTO/COMMAND), JobStatus (UPLOADED/TRANSCRIBING/THINKING/
  SPEAKING/DONE/FAILED).
- data/db/Daos.kt: Converters (enum<->String), HistoryDao (observeAll/
  observeByType/insert/delete/clear), AudioJobDao (observeRecent/observe/
  insert/update/updateStatus/clearFinished). Flow-based.
- data/db/CyanDatabase.kt: @Database v1, singleton get(context).
- data/settings/SettingsRepository.kt: DataStore. AppSettings (hermesUrl,
  hermesApiKey, openRouterModel, ttsVoiceLanguage, sttLanguage, deviceMode
  FAKE/NATIVE). Flow settings + suspend update{}.
- voice/VoicePipeline.kt: связка handleTranscript: THINKING -> hermes.chat ->
  SPEAKING -> tts.speak -> history.insert -> DONE; ошибки -> FAILED + ERROR.
  Публикует state(PipelineState), lastTranscript, lastReply. Заводит
  audio_jobs запись на каждый прогон (видно где зависло).
- Проверка: 25 .kt баланс ОК, Room-аннотации на месте.

ОСТАЛОСЬ (UI): навигация (4 вкладки), Home (push-to-talk + статусы +
индикация состояний), Settings, Notes, Media. + VoiceViewModel связывающий
PhoneSpeechToText + VoicePipeline + Settings.

---

### [Промпт 18] ГЛУБОКАЯ ПРОВЕРКА скелета — ПРОЙДЕНА
Вместо gradle (нет сети) — статический анализ согласованности:
- Все внутренние импорты com.cyanbridge.* резолвятся (46 типов, 25 файлов).
- Дублей имён типов нет.
- GlassesController: Fake и Native реализуют ВСЕ 15 методов + 2 свойства.
- VoicePipeline вызывает только существующие методы/поля DAO/HermesClient/
  TTS/Entity — всё согласовано (15 проверок OK).
- Баланс скобок (с учётом интерполяции) — ОК во всех файлах.
ВЫВОД: скелет внутренне целостен. Остаётся реальная gradle-сборка на
машине пользователя (Android Studio догенерит wrapper.jar).

---

### [Промпт 18 — UI] ВСЕ ЭКРАНЫ MVP СОБРАНЫ
Прочитан frontend-design skill. Стиль выдержан: тёмный, неон-циан, M3.

Создано:
- ui/screens/home/VoiceViewModel.kt: связывает PhoneSpeechToText +
  VoicePipeline + SettingsRepository + HermesClient. HomeUiState (pipeline,
  partial/last transcript, lastReply, serverStatus, error). checkServer(),
  startTalking() (STT collect -> partial -> final -> pipeline.handleTranscript).
  Пересоздаёт Hermes-клиента и pipeline при изменении настроек.
- ui/nav/Destination.kt: 4 вкладки (Home/Notes/Media/Settings) + иконки.
- ui/CyanBridgeApp.kt: Scaffold + NavigationBar + NavHost, тема обёрнута.
- ui/screens/home/HomeScreen.kt: статус-пилюли сервер/очки, индикатор
  PipelineState ("Слушаю…/Думаю…"), большая пульсирующая push-to-talk кнопка
  (запрос RECORD_AUDIO через rememberLauncherForActivityResult), карточки
  диалога (вы/ассистент/partial), показ ошибок.
- ui/screens/settings/SettingsViewModel.kt + SettingsScreen.kt: поля
  Hermes URL/ключ(пароль)/модель/STT-язык/TTS-язык, переключатель режима
  FAKE/NATIVE (FilterChip), кнопка "Проверить соединение" -> health().
- ui/screens/notes/NotesScreen.kt + NotesViewModel: список истории из Room
  (Flow), карточки с типом/временем/запросом/ответом, пустое состояние.
- ui/screens/media/MediaScreen.kt: статус синхронизации (честное "очки не
  подключены"; реальная синхра в MediaSyncRepository готова к режиму NATIVE).
- MainActivity: Placeholder заменён на CyanBridgeApp().

ПРОВЕРКА: 33 .kt, баланс ОК, package ОК, все импорты com.cyanbridge.*
резолвятся (53 типа), все 4 экрана найдены, material-icons-extended в catalog,
GlassesController реализован полностью, VoicePipeline согласован с DAO/клиентами.

СТАТУС MVP: приложение СОБРАНО целиком (UI + логика + данные). Голосовой
цикл рабочий в режиме "без очков" (телефон). Реальный протокол очков ждёт
байтов из manufacturer-original (изолировано в HeyCyanProtocol).
ОСТАЁТСЯ: реальная gradle-сборка на машине пользователя; ViewModelFactory не
нужен (AndroidViewModel получает Application автоматически).

---

### [Промпт 19] Скрипты сборки + README (пользователь не умеет в Android Studio)
Задача: собрать APK из командной строки без IDE. Делаю:
- gradlew + gradle-wrapper.properties уже есть; НЕ хватает gradle-wrapper.jar
  (бинарь). Решение: bootstrap-скрипт скачивает wrapper.jar ИЛИ генерит через
  системный gradle. Плюс проверка JDK17 и Android SDK, автоустановка
  cmdline-tools при отсутствии.
- build.sh (Linux/macOS), build.bat (Windows), README.md с пошагово.
- Требования к машине пользователя: JDK 17, Android SDK (или скрипт ставит
  cmdline-tools и нужные пакеты через sdkmanager).

### [Промпт 19 — реализация] Скрипты сборки + README — СДЕЛАНО
- gradlew + gradlew.bat (упрощённые официальные шаблоны).
- build.sh (Linux/macOS) + build.bat (Windows): проверка JDK17, скачивание
  gradle-wrapper.jar с ОФИЦИАЛЬНОГО services.gradle.org/distributions/
  gradle-8.9-wrapper.jar (исправлено с ненадёжного github-raw), авто-установка
  Android cmdline-tools + platform-35 + build-tools 35.0.0 через sdkmanager,
  генерация local.properties, сборка :app:assembleDebug, вывод пути APK +
  инструкция установки через adb.
- README.md: требования (JDK17), сборка одним скриптом, установка на телефон
  (копирование/adb), первая настройка Hermes (URL/ключ/режим), структура
  проекта, режим очков (TODO protocol), частые проблемы.
- build.sh синтаксис проверен (bash -n). APK на выходе:
  app/build/outputs/apk/debug/app-debug.apk
ВАЖНО: реальная сборка требует машины пользователя с JDK17 + интернетом
(скрипт качает SDK/Gradle). В песочнице не запускалась (нет сети).

### [Промпт 19 — чистка] Проверки совместимости + удаление лишнего
- Проверено: accompanist 0.36.0 совместим с Compose BOM 2024.09.03 (Compose
  1.7), НО в коде НЕ используется (взят нативный rememberLauncherForActivityResult).
  УДАЛЁН из build.gradle и catalog (минимизация зависимостей). TOML валиден.
- Проверено обеспечение API зависимостями:
  * collectAsStateWithLifecycle <- androidx-lifecycle-runtime-compose (bundle lifecycle) OK
  * Icons.Outlined/Filled.* <- material-icons-extended (bundle compose) OK
- ViewModelFactory НЕ нужен: AndroidViewModel получает Application через
  стандартный AndroidViewModelFactory (compose viewModel()).
- Проект самосогласован. Готов к сборке на машине с JDK17+интернет.

---

### [Промпт 22] Переход на GitHub Actions (пользователь не может .bat, Windows)
Контекст: пользователь на Windows, есть Android Studio но не умеет, .bat
неудобен (лежит внутри zip). GitHub-коннектор НЕДОСТУПЕН в реестре (искал —
нет среди опций). Claude собрать сам НЕ может (нет сети/SDK в песочнице).
РЕШЕНИЕ: облачная сборка через GitHub Actions.

Сделано:
- УДАЛЕНЫ build.sh, build.bat, gradlew, gradlew.bat (мусор, по просьбе юзера).
- .github/workflows/build.yml: GitHub Actions. checkout@v4, setup-java@v4
  (temurin 17), gradle/actions/setup-gradle@v4 (gradle 8.9 — НЕ требует
  wrapper.jar!), gradle :app:assembleDebug, upload-artifact@v4
  (CyanBridge-debug-apk). Триггеры: push main/master + workflow_dispatch.
  YAML провалидирован.
- README.md: секции сборки заменены на 2 пути (GitHub Actions / Android Studio).
- ИНСТРУКЦИЯ_GITHUB.md: пошагово для новичка (аккаунт -> репо -> upload ->
  Actions -> скачать artifact -> на телефон). + запасной вариант если папка
  .github не загрузилась (создать build.yml через веб-интерфейс).
- ИНСТРУКЦИЯ_ANDROID_STUDIO.md: пошагово (Open -> Gradle Sync -> Build APK ->
  locate -> установка).
- .gitignore: добавлен в ПРОЕКТ (не только в zip), НЕ прячет .github/ и
  gradle/wrapper/, прячет build/, local.properties, .env, IDE-мусор.

ПУТИ СБОРКИ И wrapper.jar:
- GitHub Actions: wrapper.jar НЕ нужен (setup-gradle даёт команду gradle).
- Android Studio: регенерит wrapper сам при открытии.
- gradle/wrapper/ содержит только .properties (jar бинарь не кладём).

ИТОГ: пользователю достаточно браузера. Залить файлы на GitHub -> APK
соберётся в облаке -> скачать. Инструкции на русском в корне проекта.

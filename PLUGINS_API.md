# TikTok Box — Документация JavaScript Plugin API

Новая система плагинов **TikTok Box 1.0** позволяет расширять функционал приложения с помощью стандартных **ZIP-архивов**, содержащих манифест `manifest.json`, исполняемый код на **JavaScript**, а также любые пользовательские файлы (звуки, фото, видео, шрифты, конфигурации).

---

## 1. Структура плагина (.zip)

Плагин представляет собой обычный ZIP-архив с расширением `.zip`. Внутри архива располагаются:

```text
my_awesome_plugin.zip
│
├── manifest.json        # Обязательный файл манифеста с описанием плагина
├── main.js              # Главный скрипт плагина (точка входа)
├── icon.png             # Иконка плагина (отображается в настройках)
│
├── sounds/              # Произвольные ассеты плагина
│   └── welcome.mp3
├── images/
│   └── banner.png
└── config/
    └── presets.json
```

---

## 2. Манифест (`manifest.json`)

Файл `manifest.json` содержит метаданные о плагине:

```json
{
  "id": "com.example.superplugin",
  "name": "Super Plugin",
  "name_ru": "Супер Плагин",
  "version": "1.0.0",
  "author": "@developer",
  "description": "Полезные функции и звуковые уведомления для TikTok Box",
  "description_ru": "Полезные функции и звуковые уведомления для TikTok Box",
  "icon": "icon.png",
  "main": "main.js",
  "min_api": 1
}
```

### Спецификация полей:
| Поле | Тип | Обязательное | Описание |
| :--- | :--- | :---: | :--- |
| `id` | `string` | Да | Уникальный строковый идентификатор плагина (буквы, цифры, точки, дефисы). |
| `name` | `string` | Да | Имя плагина, отображаемое в списке. |
| `name_ru` / `name_uk` | `string` | Нет | Локализованное название для русского/украинского языка. |
| `version` | `string` | Да | Версия плагина (например, `"1.0.0"`). |
| `author` | `string` | Да | Никнейм автора или разработчика. |
| `description` | `string` | Да | Описание возможностей плагина. |
| `icon` | `string` | Нет | Относительный путь к иконке внутри ZIP (по умолчанию `icon.png`). |
| `main` | `string` | Да | Главный JS-файл для выполнения (по умолчанию `main.js`). |
| `min_api` | `int` | Нет | Минимальная версия API TikTok Box (по умолчанию `1`). |
| `tiktok` | `string` | Нет | Версия TikTok, если плагин написан строго под определенную версию. |

---

## 3. Среда выполнения JavaScript

- Каждый активный плагин выполняется в изолированной среде на базе движка V8 с поддержкой современного синтаксиса **ES6+**, `async/await`, `Promise`, `JSON`, `fetch` и таймеров.
- Глобальные точки входа: объект `ttbox` (а также алиасы `TikTokBox` и `margy`).
- Ошибки выполнения не крашат приложение, а безопасно логируются в дневник мода.

---

## 4. Справочник API (`ttbox`)

### 4.1. Метаданные плагина (`ttbox.plugin`)

```javascript
// Идентификатор плагина
console.log(ttbox.plugin.id);

// Название плагина
console.log(ttbox.plugin.name);

// Версия плагина
console.log(ttbox.plugin.version);

// Автор плагина
console.log(ttbox.plugin.author);

// Абсолютный путь к распакованной папке плагина на устройстве
console.log(ttbox.plugin.folder);
```

---

### 4.2. Пользовательский интерфейс и диалоги (`ttbox.ui`)

#### `ttbox.ui.toast(message)`
Показывает короткое системное всплывающее уведомление (Toast).
```javascript
ttbox.ui.toast("Плагин успешно активирован!");
```

#### `ttbox.ui.toastLong(message)`
Показывает длинное всплывающее уведомление.
```javascript
ttbox.ui.toastLong("Подробное сообщение о результате операции...");
```

#### `ttbox.ui.say(message)`
Выводит плавающую карточку мода внизу экрана (быстрое сообщение).
```javascript
ttbox.ui.say("Новое видео в ленте!");
```

#### `ttbox.ui.alert(title, message)`
Показывает диалоговое окно с кнопкой «ОК».
```javascript
ttbox.ui.alert("Внимание", "Настройки были сброшены.");
```

#### `ttbox.ui.confirm(title, message, callback)`
Показывает диалог с кнопками «ОК» и «Отмена».
```javascript
ttbox.ui.confirm("Удаление", "Вы уверены, что хотите очистить кэш?", function(result) {
    if (result) {
        ttbox.ui.toast("Очищено!");
    } else {
        ttbox.ui.toast("Отменено");
    }
});
```

#### `ttbox.ui.prompt(title, defaultValue, callback)`
Показывает диалоговое окно с полем ввода текста.
```javascript
ttbox.ui.prompt("Введите никнейм", "user123", function(text) {
    if (text !== null) {
        ttbox.ui.toast("Вы ввели: " + text);
    }
});
```

#### `ttbox.ui.offer(label, callback)`
Отображает плавающую интерактивную кнопку («пилюлю») прямо поверх текущего экрана TikTok. При нажатии срабатывает `callback`.
```javascript
ttbox.ui.offer("🔥 Скачать видео", function() {
    ttbox.ui.toast("Загрузка видео запущена...");
});
```

#### `ttbox.ui.progress(text, percent)` / `ttbox.ui.progressGone()`
Управляет верхней полосой прогресса приложения.
```javascript
ttbox.ui.progress("Загрузка данных...", 45);

// Скрыть полосу прогресса
ttbox.ui.progressGone();
```

#### `ttbox.ui.vibrate(milliseconds)`
Вызывает тактильную отдачу (вибрацию) телефона.
```javascript
ttbox.ui.vibrate(150); // Вибрация на 150 миллисекунд
```

#### `ttbox.ui.openUrl(url)`
Открывает веб-ссылку во внешнем браузере.
```javascript
ttbox.ui.openUrl("https://t.me/tiktokmod_box");
```

#### `ttbox.ui.copyToClipboard(text)` / `ttbox.ui.getClipboardText()`
Работа с системным буфером обмена Android.
```javascript
// Копирование
ttbox.ui.copyToClipboard("https://tiktok.com/@example");

// Чтение
var clipboard = ttbox.ui.getClipboardText();
console.log("В буфере:", clipboard);
```

#### `ttbox.ui.addSettingsRow(title, detail, callback)`
Добавляет интерактивную строку с кнопкой прямо в раздел настроек **TikTok Box**!
```javascript
ttbox.ui.addSettingsRow("Проверить баланс", "Нажмите для проверки API", function() {
    ttbox.ui.alert("Баланс", "Остаток запросов: 99");
});
```

---

### 4.3. Работа с файлами и ассетами плагина (`ttbox.assets`)

Внутри плагина можно хранить любые файлы. Модуль `ttbox.assets` обеспечивает доступ к ним:

#### `ttbox.assets.getPath(filename)`
Возвращает абсолютный путь к файлу внутри папки плагина на телефоне.
```javascript
var soundPath = ttbox.assets.getPath("sounds/alert.mp3");
```

#### `ttbox.assets.getUri(filename)`
Возвращает путь в виде `file://` URI.
```javascript
var imgUri = ttbox.assets.getUri("images/logo.png");
```

#### `ttbox.assets.exists(filename)`
Проверяет существование файла.
```javascript
if (ttbox.assets.exists("config/settings.json")) {
    console.log("Конфигурация найдена!");
}
```

#### `ttbox.assets.readText(filename)`
Считывает текстовый файл (UTF-8) в виде строки.
```javascript
var jsonString = ttbox.assets.readText("config/settings.json");
if (jsonString) {
    var data = JSON.parse(jsonString);
    console.log("Загружено:", data);
}
```

#### `ttbox.assets.writeText(filename, content)`
Записывает текстовые данные в файл внутри директории плагина.
```javascript
ttbox.assets.writeText("notes.txt", "Сохраненный текст от " + new Date());
```

#### `ttbox.assets.readBase64(filename)`
Считывает бинарный файл (изображение, аудио) в строку Base64.
```javascript
var base64Data = ttbox.assets.readBase64("images/icon.png");
```

#### `ttbox.assets.list(subdir)`
Возвращает массив имен файлов в указанной подпапке плагина (или в корне, если не указано).
```javascript
var soundFiles = ttbox.assets.list("sounds");
console.log("Файлы звуков:", soundFiles);
```

---

### 4.4. Аудиоплеер (`ttbox.audio` / `ttbox.media`)

Встроенный плеер позволяет воспроизводить звуковые эффекты из архива плагина или потоковое аудио из интернета:

```javascript
// Воспроизведение звука из локальных ассетов плагина
ttbox.audio.play("sounds/welcome.mp3");

// Воспроизведение по прямой URL-ссылке
ttbox.audio.play("https://example.com/sound.mp3");

// Установка громкости (от 0.0 до 1.0)
ttbox.audio.setVolume(0.8);

// Зацикливание воспроизведения
ttbox.audio.setLooping(false);

// Управление воспроизведением
ttbox.audio.pause();
ttbox.audio.resume();
ttbox.audio.stop();

// Проверка состояния
if (ttbox.audio.isPlaying()) {
    console.log("Аудио играет прямо сейчас");
}
```

---

### 4.5. Постоянное хранилище (`ttbox.storage`)

Хранилище изолировано для каждого плагина и сохраняется между перезапусками приложения:

```javascript
// Сохранение значений (строки, числа, объекты)
ttbox.storage.set("auto_download", true);
ttbox.storage.set("user_data", { name: "Alex", score: 100 });

// Получение значений (автоматически парсит JSON-объекты)
var autoDl = ttbox.storage.get("auto_download", false);
var userData = ttbox.storage.get("user_data", null);

// Удаление ключа
ttbox.storage.remove("user_data");

// Получение списка всех ключей
var allKeys = ttbox.storage.keys();

// Полная очистка хранилища плагина
ttbox.storage.clear();
```

---

### 4.6. Сетевые запросы и скачивание (`ttbox.http`)

#### `ttbox.http.get(url, [callback])`
Выполняет асинхронный HTTP GET-запрос. Поддерживает как `callback`, так и стандартный `Promise`:
```javascript
// Вариант 1: Через async/await и Promise
async function loadStatus() {
    try {
        var response = await ttbox.http.get("https://api.github.com/zen");
        console.log("Ответ сервера:", response);
    } catch (err) {
        console.error("Ошибка сети:", err);
    }
}

// Вариант 2: Через callback
ttbox.http.get("https://api.ipify.org?format=json", function(result) {
    console.log("Результат:", result);
});
```

#### `ttbox.http.post(url, data, contentType, [callback])`
Выполняет асинхронный HTTP POST-запрос:
```javascript
var payload = { user: "test", action: "ping" };

ttbox.http.post("https://httpbin.org/post", payload, "application/json")
    .then(function(reply) {
        console.log("Сервер ответил:", reply);
    })
    .catch(function(err) {
        console.error("Ошибка POST:", err);
    });
```

#### `ttbox.http.download(url, fileName)`
Загружает файл в системную папку **Загрузки** (`Downloads`) телефона с системным уведомлением Android:
```javascript
ttbox.http.download("https://example.com/video.mp4", "tiktok_video_123.mp4");
```

---

### 4.7. Интеграция с TikTok и окружением (`ttbox.tiktok`)

```javascript
// Числовой ID текущего авторизованного пользователя
var uid = ttbox.tiktok.getUserId();

// Opaque sec_uid пользователя
var secUid = ttbox.tiktok.getSecUserId();

// @username текущего пользователя
var username = ttbox.tiktok.getUsername();

// Никнейм (display name) текущего пользователя
var nickname = ttbox.tiktok.getNickname();

// Версия клиента TikTok
var ttVer = ttbox.tiktok.getAppVersion();

// Версия мода (1.0)
var modVer = ttbox.tiktok.getModVersion();

// Текущий код региона ленты (например, "nl", "us", "de", "ru")
var currentRegion = ttbox.tiktok.getFeedRegion();

// Смена региона ленты
ttbox.tiktok.setFeedRegion("us");

// Имя класса текущего экрана
var screen = ttbox.tiktok.getCurrentScreen();

// Чистый перезапуск TikTok Box
ttbox.tiktok.restartApp();
```

---

### 4.8. Система событий (`ttbox.events`)

Плагины могут подписаться на системные события жизненного цикла TikTok:

```javascript
// Смена экрана (открытие ленты, профиля, настроек, комментариев)
ttbox.events.on('screen_change', function(data) {
    console.log("Пользователь открыл экран:", data.screen);
});

// Загрузка страницы рекомендаций / подписок
ttbox.events.on('feed_received', function(data) {
    console.log("Лента обновилась! Получено видеороликов:", data.count);
});

// TikTok вернулся на передний план
ttbox.events.on('activity_resumed', function() {
    console.log("Приложение активно");
});

// TikTok свернут в фон
ttbox.events.on('activity_paused', function() {
    console.log("Приложение в фоне");
});

// Плагин отключается или приложение закрывается
ttbox.events.on('stop', function() {
    console.log("Очистка ресурсов плагина...");
});
```

Также можно объявить глобальные хуки прямо в коде `main.js`:
```javascript
function onStart() {
    console.log("Плагин стартовал!");
}

function onScreen(screenName) {
    console.log("Экран:", screenName);
}

function onFeed(count) {
    console.log("Видео в ленте:", count);
}

function onStop() {
    console.log("Остановка плагина");
}
```

---

## 5. Полные примеры плагинов

### Пример 1: Приветствие со звуком и кнопкой в настройках

`manifest.json`:
```json
{
  "id": "com.box.welcome",
  "name": "Welcome Sound",
  "name_ru": "Звуковое приветствие",
  "version": "1.0",
  "author": "@modder",
  "description": "Воспроизводит звук при старте и добавляет кнопку в настройки",
  "icon": "icon.png",
  "main": "main.js"
}
```

`main.js`:
```javascript
console.log("Welcome Sound плагин инициализирован!");

// Воспроизведение звука при старте
if (ttbox.assets.exists("welcome.mp3")) {
    ttbox.audio.play("welcome.mp3");
}

ttbox.ui.toast("Добро пожаловать в TikTok Box, " + (ttbox.tiktok.getNickname() || "друг") + "!");

// Добавление интерактивной кнопки в настройки мода
ttbox.ui.addSettingsRow("Проиграть звук приветствия", "Нажмите для теста", function() {
    ttbox.audio.play("welcome.mp3");
    ttbox.ui.toast("Звук воспроизводится...");
});
```

---

### Пример 2: Уведомление о регионе и плавающая кнопка в ленте

`main.js`:
```javascript
// При открытии ленты показываем плавающую кнопку
ttbox.events.on('screen_change', function(data) {
    if (data.screen.indexOf("MainActivity") !== -1) {
        ttbox.ui.offer("📍 Регион: " + ttbox.tiktok.getFeedRegion().toUpperCase(), function() {
            ttbox.ui.confirm("Смена региона", "Переключить ленту на США (US)?", function(ok) {
                if (ok) {
                    ttbox.tiktok.setFeedRegion("us");
                    ttbox.ui.toast("Регион изменен на US! Перезапустите приложение.");
                }
            });
        });
    }
});
```

---

### Пример 3: Загрузчик и сохранение настроек

`main.js`:
```javascript
// Загрузка сохраненного счетчика
var downloadsCount = ttbox.storage.get("downloads_count", 0);

ttbox.ui.addSettingsRow("Статистика плагина", "Всего скачиваний: " + downloadsCount, function() {
    ttbox.ui.alert("Статистика", "Вы использовали плагин " + downloadsCount + " раз.");
});

ttbox.events.on('feed_received', function(data) {
    downloadsCount++;
    ttbox.storage.set("downloads_count", downloadsCount);
});
```

---

## 6. Как упаковать и установить плагин

1. Создайте папку со своими файлами (`manifest.json`, `main.js`, `icon.png`, нужные ассеты).
2. Выделите файлы и заархивируйте их в обычный `.zip` (убедитесь, что `manifest.json` находится в корне архива, а не во вложенной папке).
3. Перенесите архив `.zip` на телефон.
4. Откройте **TikTok Box** -> **Настройки мода** -> раздел **Плагины** -> **Установить плагин**.
5. Выберите ваш `.zip` файл. Плагин распакуется, появится в списке и сразу начнет работать!

# План розробки Android-додатка LovesUa

Цей документ описує покроковий план створення офіційного мобільного додатка для Android для сервісу знайомств **LovesUa** (`https://lovesua.date`). Додаток буде на 100% ідентичним веб-версії, використовуватиме спільну базу даних та серверну логіку, а також надаватиме нативний користувацький досвід (іконка, сплеш-скрін, сповіщення, жести навігації, доступ до камери та геолокації).

---

## 1. Архітектурна концепція

### Чому обрано Native Kotlin WebView Shell з нативним містком (Native Bridge):
1. **100% ідентичність інтерфейсу та функцій**: Усі оновлення дизайну, тем, картки свайпів, чати та VIP-статуси автоматично з'являються в додатку без необхідності повторного випуску та завантаження оновлень користувачами.
2. **Єдина екосистема та база даних**: Користувач авторизується під своїм обліковим записом, бачить ті самі листування, збіги та баланс монет.
3. **Легкість додатка**: Розмір APK становить лише 3–5 МБ (на відміну від важких фреймворків), додаток запускається миттєво та споживає мінімум батареї.
4. **Повні нативні можливості Android**:
   - Нативний Splash Screen (заставка) у фірмових кольорах LovesUa.
   - Підтримка жестів та апаратної кнопки «Назад» (збереження історії переходів).
   - Повноцінний доступ до камери та галереї для завантаження фото в профіль.
   - Геолокація (GPS) для розділу «Люди поруч».
   - Pull-to-Refresh (жест оновлення сторінки змахуванням зверху вниз).
   - Екран перевірки з'єднання (Offline fallback з кнопкою «Повторити спробу»).
   - Готовність до Push-сповіщень (FCM) про нові повідомлення та лайки.

---

## User Review Required

> [!IMPORTANT]
> **Канали розповсюдження:**
> Чи плануєте ви публікувати додаток у **Google Play Store** (потрібен акаунт розробника Google за \$25), чи на першому етапі достатньо створити пряме скачування файлу **APK** із сайту (наприклад, `https://lovesua.date/app` або кнопка в шапці/підвалі сайту)?

> [!TIP]
> **Push-сповіщення (FCM):**
> Для отримання сповіщень на телефон, коли додаток згорнуто, знадобиться безкоштовний проект Firebase Cloud Messaging (файл `google-services.json`). Ми можемо підготувати архітектуру зараз, а ключ додати за бажанням.

---

## Open Questions

1. **Ідентифікатор пакета (Package Name):**  
   Пропонується стандартний `com.lovesua.app` (або `date.lovesua.android`). Чи підходить цей ідентифікатор?
2. **Мінімальна версія Android:**  
   Пропонується `minSdk = 24` (Android 7.0+) та `targetSdk = 35` (Android 15), що покриває понад 96% усіх активних Android-пристроїв в Україні та світі.

---

## Proposed Changes

### 1. Структура нового модуля Android у проєкті
Створюється ізольована директорія проєкту Android усередині репозиторію: `dating-app/android/`:

#### [NEW] [build.gradle (Project)](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/build.gradle)
- Базові налаштування збірки Android Gradle Plugin та Kotlin.

#### [NEW] [settings.gradle](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/settings.gradle)
- Реєстрація модулів проєкту (`include ':app'`).

#### [NEW] [app/build.gradle](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/app/build.gradle)
- Залежності: AndroidX Core, AppCompat, Material Components, SwipeRefreshLayout, WebKit, Play Services (за потреби).

#### [NEW] [AndroidManifest.xml](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/app/src/main/AndroidManifest.xml)
- Опис дозволів:
  - `INTERNET`, `ACCESS_NETWORK_STATE`
  - `CAMERA`, `READ_MEDIA_IMAGES`, `READ_EXTERNAL_STORAGE` (для фото профілю)
  - `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION` (для гео-пошуку)
  - `POST_NOTIFICATIONS` (Android 13+)
- Реєстрація `MainActivity` з фільтрами Deep Links (`https://lovesua.date/*`).

#### [NEW] [MainActivity.kt](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/app/src/main/java/com/lovesua/app/MainActivity.kt)
- Головний екран додатка:
  - Ініціалізація та налаштування безпеки `WebView` (DOM Storage, JavaScript, CookieManager).
  - Підключення `SwipeRefreshLayout` для оновлення сторінок свайпом.
  - Обробка апаратної навігації «Назад» (`OnBackPressedCallback`).
  - Логіка перевірки інтернет-з'єднання та показ нативного екрана помилки.

#### [NEW] [LovesUaWebViewClient.kt](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/app/src/main/java/com/lovesua/app/LovesUaWebViewClient.kt)
- Контролер перехоплення посилань:
  - Відкриття внутрішніх маршрутів (`lovesua.date`) усередині WebView.
  - Автоматичне відкриття зовнішніх сервісів (Google OAuth, платіжні шлюзи, соцмережі) через Android Custom Tabs або системний браузер.
  - Передача спеціального заголовка `X-Requested-With: LovesUa-Android-App`.

#### [NEW] [LovesUaChromeClient.kt](file:///c:/Users/Admin/Documents/Laravel/dating-app/android/app/src/main/java/com/lovesua/app/LovesUaChromeClient.kt)
- Обробка діалогів, завантаження файлів (`onShowFileChooser`) та дозволів геолокації (`onGeolocationPermissionsShowPrompt`).

#### [NEW] Графічні ресурси додатка
- Набір фірмових іконок LovesUa (`mipmap-hdpi`, `mipmap-xhdpi`, `mipmap-xxhdpi`, `mipmap-xxxhdpi`) та адаптивна векторна іконка.
- Заставка Splash Screen з фірмовим градієнтом.

---

### 2. Доопрацювання веб-частини (Laravel) для підтримки додатка

#### [MODIFY] [app/Http/Middleware/SecurityHeadersMiddleware.php](file:///c:/Users/Admin/Documents/Laravel/dating-app/app/Http/Middleware/SecurityHeadersMiddleware.php)
- Дозволити коректну роботу WebView без блокування вбудованих функцій.

#### [NEW] [public/manifest.json](file:///c:/Users/Admin/Documents/Laravel/dating-app/public/manifest.json)
- Стандартний Web App Manifest для підтримки PWA та швидкої взаємодії з WebView.

#### [NEW] [resources/views/pages/download-app.blade.php](file:///c:/Users/Admin/Documents/Laravel/dating-app/resources/views/pages/download-app.blade.php)
- Стильна сторінка на сайті для завантаження Android-додатка користувачами (із бейджем «Завантажити для Android», описом переваг та QR-кодом для швидкої інсталяції).

---

## Verification Plan

### 1. Збірка та валідація проєкту Android:
- Перевірка синтаксису Gradle-конфігурацій та Kotlin-файлів.
- Генерація інсталяційного пакету `app-debug.apk` (або `app-release.apk`).

### 2. Тестування функціоналу на емуляторі/пристрої:
- **Авторизація та сесія**: Перевірка збереження сесії (користувач залишається залогіненим після перезапуску додатка).
- **Свайпи та чати**: Перевірка гладкості анімацій у «Знайомствах» та роботи чатів у реальному часі.
- **Завантаження медіа**: Перевірка роботи вікна вибору фото з камери та галереї.
- **Геолокація**: Запит дозволу та визначення міста в розділі «Люди поруч».
- **Офлайн-режим**: Відключення мережі -> перевірка показу нативного екрана -> відновлення мережі -> автоматичне перезавантаження.

# 💖 LovesUa — Інструкція зі встановлення на хостинг та сервер (Production Deployment)

Сучасна повнофункціональна дейтинг-платформа на базі **Laravel 12 / PHP 8.2 - 8.4**.  
Цей посібник містить вичерпні покрокові інструкції для розгортання проекту на будь-якому хостингу: від віртуального (cPanel / DirectAdmin) до виділених VPS/VDS (Ubuntu / Debian).

---

## 📋 1. Системні вимоги до сервера

### Основні компоненти:
- **PHP:** `8.2`, `8.3` або `8.4` (рекомендовано PHP 8.3/8.4 FPM)
- **База даних:** 
  - MySQL `>= 8.0` (рекомендовано)
  - MariaDB `>= 10.4`
  - або SQLite `>= 3.35`
- **Веб-сервер:**
  - **Nginx** `>= 1.18` (рекомендовано, готовий конфіг: `nginx.conf.example`)
  - **Apache** `>= 2.4` з увімкненим модулем `mod_rewrite`
  - LiteSpeed / OpenLiteSpeed
- **Composer:** `>= 2.2`

### Обов'язкові розширення PHP:
- `pdo` та `pdo_mysql` (або `pdo_sqlite`)
- `openssl` (шифрування сесій та токенів)
- `mbstring` (робота з мультибайтними рядками та UTF-8)
- `tokenizer` (парсинг коду)
- `xml`, `ctype`, `json`, `fileinfo` (валідація завантажуваних файлів)
- `gd` з підтримкою **WebP** або `imagick` (стиснення та конвертація фото користувачів)
- `curl` (Google OAuth, Apple ID, Telegram Bot API, TurboSMS)
- `bcmath`

---

## 🔐 2. Права доступу до файлів та папок

Для коректної роботи кешу, завантаження фотографій і запису логів веб-сервер повинен мати права на запис у такі каталоги:

### Для Linux / VPS (користувач `www-data` або ваш користувач хостингу):
```bash
# Встановлення власника:
sudo chown -R www-data:www-data /var/www/lovesua

# Права на папки та файли:
chmod -R 775 /var/www/lovesua/storage
chmod -R 775 /var/www/lovesua/bootstrap/cache
chmod 664 /var/www/lovesua/.env
```

### Структура папок `storage`:
Переконайтеся, що існують каталоги:
- `storage/app/public/photos`
- `storage/app/public/receipts`
- `storage/framework/cache`
- `storage/framework/sessions`
- `storage/framework/views`
- `storage/logs`

---

## 🚀 3. Способи встановлення

---

### Варіант А: Встановлення через Веб-майстер (5 кроків — Найпростіший спосіб)

Проєкт оснащено вбудованим інтерактивним інсталятором:

1. **Завантажте файли проекту** на сервер у кореневу директорію сайту.
2. **Спрямуйте Document Root (кореневу папку домену)** на підкаталог **`/public`**.
3. Переконайтеся, що файл `storage/installed` **відсутній** (якщо він є, видаліть його для первинного запуску інсталятора).
4. **Відкрийте у браузері:**
   ```text
   https://ваш-домен.com/install
   ```
5. **Пройдіть 5 простих кроків:**
   - **Крок 1 (Діагностика):** Автоматична перевірка версії PHP, наявності розширень та прав на папки `storage/` і `bootstrap/cache/`.
   - **Крок 2 (Налаштування сайту):** Введення назви сайту, URL (`https://ваш-домен.com`), режиму роботи (`production`).
   - **Крок 3 (База даних):** Вибір MySQL або SQLite, введення хоста, логіна, пароля та імені бази даних. Кнопка «Перевірити підключення» дозволяє протестувати зв'язок з БД в один клік перед міграцією.
   - **Крок 4 (Головний Адміністратор):** Введення імені, email та надійного пароля для створення акаунта адміністратора з повними правами.
   - **Крок 5 (Завершення):** Автоматичне генерування ключа шифрування `APP_KEY`, створення сімлінка `storage:link` та перенаправлення в адмін-панель.

---

### Варіант Б: Встановлення на звичайний Shared-хостинг (cPanel, DirectAdmin, ISPmanager)

1. **Створення БД:**
   - У панелі хостингу перейдіть у розділ **MySQL / Бази даних**.
   - Створіть нову базу даних (наприклад, `u12345_lovesua`), користувача та надійний пароль. Надайте користувачеві **Всі привілеї** (ALL PRIVILEGES).

2. **Завантаження файлів:**
   - Завантажте ZIP-архів проекту в кореневу папку домену через файловий менеджер або FTP.
   - Розпакуйте архів.

3. **Коренева папка сайту (Document Root):**
   - У налаштуваннях домену вкажіть директорію **`public_html/public`** (або `ваш-домен/public`).
   - *Примітка:* Якщо хостинг не дозволяє змінити Document Root на `/public`, у корені проекту вже підготовлено файли `.htaccess` та `index.php`, які автоматично переадресовують запити у безпечному режимі.

4. **Файл конфігурації `.env`:**
   - Скопіюйте файл `.env.example` у `.env`.
   - Заповніть параметри:
     ```env
     APP_NAME="LovesUa"
     APP_ENV=production
     APP_DEBUG=false
     APP_URL=https://ваш-домен.com

     DB_CONNECTION=mysql
     DB_HOST=localhost
     DB_PORT=3306
     DB_DATABASE=u12345_lovesua
     DB_USERNAME=u12345_user
     DB_PASSWORD=ваш_пароль_бд
     ```

5. **Виконання міграцій та сімлінка сховища:**
   - Якщо доступний **SSH (термінал у cPanel)**:
     ```bash
     php artisan key:generate --force
     php artisan migrate --force
     php artisan db:seed --force
     php artisan storage:link
     ```
   - Якщо SSH недоступний: просто відкрийте `https://ваш-домен.com/install` і виконайте кроки через браузер.

---

### Варіант В: Встановлення на виділений VPS / VDS (Ubuntu 22.04 / 24.04 LTS)

Повний стек: **Nginx + PHP 8.3-FPM + MySQL 8.0 + Certbot SSL**.

```bash
# 1. Перейдіть до каталогу веб-сайтів
cd /var/www
git clone <url-вашого-репозиторію> lovesua
# або розпакуйте архів у папку /var/www/lovesua
cd /var/www/lovesua

# 2. Встановлення залежностей Composer (без dev-пакетів)
composer install --no-dev --optimize-autoloader

# 3. Налаштування файлу оточення
cp .env.example .env
php artisan key:generate

# 4. Відкрийте .env та вкажіть параметри вашої БД
nano .env

# 5. Міграції та наповнення початковими даними
php artisan migrate --force
php artisan db:seed --force

# 6. Створення символічного посилання на публічні медіафайли
php artisan storage:link

# 7. Налаштування прав власності
sudo chown -R www-data:www-data /var/www/lovesua
sudo chmod -R 775 /var/www/lovesua/storage /var/www/lovesua/bootstrap/cache

# 8. Налаштування Nginx
# Використовуйте готовий конфігураційний файл:
sudo cp nginx.conf.example /etc/nginx/sites-available/lovesua.conf
sudo nano /etc/nginx/sites-available/lovesua.conf
# (Змініть server_name на ваш домен і перевірте версію сокета php-fpm)

# Активація сайту в Nginx:
sudo ln -s /etc/nginx/sites-available/lovesua.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# 9. Отримання безкоштовного SSL-сертифіката Let's Encrypt
sudo certbot --nginx -d ваш-домен.com -d www.ваш-домен.com
```

---

## ⚡ 4. Виробнича оптимізація швидкодії (Production Cache)

Після кожного розгортання або оновлення коду на хостингу обов'язково запустіть команди кешування для максимального показника Google Lighthouse (95+):

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache
```

Для очищення кешу в разі зміни шаблонів або конфігурацій:
```bash
php artisan optimize:clear
```

---

## ⏱️ 5. Налаштування фонових завдань (Cron)

Платформа містить автоматичні регламентні завдання:
- Очищення застарілих логів та сповіщень.
- Безпечна 90-денна деактивація профілів (grace window) та остаточне видалення.
- Відправка щоденного та щотижневого email-дайджесту взаємних симпатій.
- Автоматичне оновлення динамічної карти сайту `sitemap.xml`.

Відкрийте планувальник завдань (`crontab -e`) і додайте **один рядок**:

```bash
* * * * * cd /var/www/lovesua && php artisan schedule:run >> /dev/null 2>&1
```
*(У панелі cPanel: розділ **Cron Jobs** -> інтервал «Once Per Minute (* * * * *)» -> команда: `cd /шлях/до/сайту && /usr/local/bin/php artisan schedule:run >> /dev/null 2>&1`)*.

---

## 📨 6. Налаштування обробника черг (Queue Worker)

За замовчуванням у `.env` встановлено `QUEUE_CONNECTION=sync` (завдання виконуються синхронно).  
Для високих навантажень рекомендується переключити на `QUEUE_CONNECTION=database` у `.env`:

```env
QUEUE_CONNECTION=database
```

Для безперервної фонової обробки відправки email, push та push-дайджестів налаштуйте **Supervisor** на VPS:

Створіть конфігураційний файл `/etc/supervisor/conf.d/lovesua-worker.conf`:
```ini
[program:lovesua-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/lovesua/artisan queue:work database --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/lovesua/storage/logs/worker.log
stopwaitsecs=3600
```

Оновіть та запустіть процес Supervisor:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start lovesua-worker:*
```

---

## 👑 7. Доступ до адмін-панелі та перший вхід

- **URL входу:** `https://ваш-домен.com/admin`
- **Облікові дані за замовчуванням (якщо встановлено через seeder):**
  - **Email:** `admin@lovesua.ua`
  - **Пароль:** `admin12345`
  *(Якщо ви використовували веб-майстер `/install`, діють введені вами на кроці 4 дані)*.
- **Безпека:** Після першого входу обов'язково змініть пароль адміністратора в меню **«Профіль»** або **«Користувачі»**.

---

## 🔑 8. Налаштування авторизації в адмінці (`/admin/auth-methods`)

Усі методи авторизації налаштовуються без редагування файлів — прямо в панелі адміністратора:

1. **🤖 Telegram Bot OTP (Рекомендовано — Безкоштовна альтернатива SMS):**
   - Створіть бота через `@BotFather` у Telegram, отримайте API токен.
   - Введіть токен у формі та натисніть «🔍 Автовизначити Chat ID».
   - Користувачі зможуть входити за номером телефону, отримуючи безкоштовні 4-значні коди через Telegram.

2. **🌐 Google OAuth (Вхід через Google):**
   - Створіть проект у [Google Cloud Console](https://console.cloud.google.com/).
   - Додайте Redirect URI: `https://ваш-домен.com/auth/google/callback`.
   - Вкажіть `Client ID` та `Client Secret` в адмінці.

3. **🍏 Apple ID (Sign in with Apple):**
   - Налаштуйте Service ID у Apple Developer Console.
   - Redirect URL: `https://ваш-домен.com/auth/apple/callback`.

4. **📱 SMS-шлюзи для України:**
   - **TurboSMS:** вкажіть API ключ і підпис відправника (Альфанумерик).
   - **AlphaSMS:** аналогічно підтримує відправку SMS на будь-які мобільні оператори України (+380).
   - **Twilio / TextBelt:** міжнародні провайдери.

---

## 💳 9. Налаштування платіжних систем та тарифів

1. **Тарифи та пакети:** Перейдіть у меню **«📦 Пакети монет та VIP»** (`/admin/packages`). Ви можете створювати, змінювати ціни, бонуси та вмикати/вимикати пакети монет чи VIP-підписки на 1, 3, 6 або 12 місяців.
2. **P2P оплати на банківські картки (IBAN / Моно / Приват):**
   - У розділі **«Налаштування»** вкажіть реквізити для отримання оплат.
   - Користувачі при купівлі бачать реквізити та завантажують квитанцію про оплату.
   - Адміністратор перевіряє квитанції та схвалює зарахування в один клік у меню **«Транзакції»** (`/admin/payments`).
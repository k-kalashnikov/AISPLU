# 📘 Гайд 5. Конфиги и правила безопасности

**Цель:** Научиться безопасно редактировать конфигурационные файлы, работать с секретами (паролями, ключами), понимать правила безопасного доступа и не ронять продакшен-сервисы одним неосторожным движением.

---

## 🧠 Часть 1. Почему конфиги — это опасно

**Конфигурационный файл (конфиг)** — это текстовый файл, который определяет, как работает программа: на каком порту слушать, какие права давать, к какой базе подключаться и т.д.

**Главное правило сисадмина:** 
> 🚨 **Одна лишняя запятая или опечатка в конфиге может положить весь сервис больницы.** 

Поэтому работа с конфигами требует дисциплины. Никаких «я просто поправлю одну строчку и сразу сохраню».

---

## ✏️ Часть 2. Редакторы в терминале

В Linux нет «Блокнота». Есть два основных редактора:

### 1. `nano` — простой и дружелюбный (рекомендуется для новичков)

```bash
nano /etc/nginx/nginx.conf
```

**Интерфейс:**
```
  GNU nano 7.2                           nginx.conf
server {
    listen 80;
    server_name emh.hospital.local;
    
    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}

^G Get Help    ^O Write Out    ^W Where Is    ^K Cut    ^T Execute
^X Exit        ^R Read File    ^\ Replace     ^U Paste  ^J Justify
```

**Горячие клавиши (внизу экрана `^` означает `Ctrl`):**
| Клавиша | Действие |
|---------|----------|
| `Ctrl+O` | Сохранить (Write Out) — нажать `Enter` для подтверждения имени |
| `Ctrl+X` | Выйти (Exit) — если были изменения, спросит, сохранять ли |
| `Ctrl+W` | Поиск (Where Is) — ввести слово, `Enter` |
| `Ctrl+K` | Вырезать строку (Cut) |
| `Ctrl+U` | Вставить строку (Uncut/Paste) |
| `Ctrl+\` | Найти и заменить (Replace) |
| `Alt+U` | Отменить последнее действие (Undo) |
| `Alt+E` | Вернуть отменённое (Redo) |

> 💡 **Совет:** `nano` показывает подсказки внизу экрана, поэтому его сложно «сломать». Идеально для быстрых правок.

### 2. `vim` — мощный, но коварный (для продвинутых)

```bash
vim /etc/nginx/nginx.conf
```

**Два основных режима:**
- **Normal mode** (режим по умолчанию) — навигация, копирование, удаление
- **Insert mode** (режим ввода) — набор текста

**Как работать:**
| Действие | Клавиши |
|----------|---------|
| Войти в режим ввода | `i` |
| Выйти из режима ввода | `Esc` |
| Сохранить | `:w` + `Enter` |
| Выйти | `:q` + `Enter` |
| Сохранить и выйти | `:wq` или `:x` |
| Выйти без сохранения | `:q!` |
| Поиск | `/слово` + `Enter`, затем `n` (следующее) |

> 🚨 **Типичная проблема новичка:** открыл `vim`, ничего не получается, нажимаешь всякие кнопки, выходишь через `:q!` и теряешь изменения. Если не уверены — используйте `nano`.

> 💡 **Шутка сисадминов:** «Как выйти из Vim? — Загугли.»

---

## 💾 Часть 3. Золотое правило: бэкап перед изменением

**Никогда не редактируйте конфиг без резервной копии.** Это правило №1 в больнице, где цена ошибки — здоровье пациентов.

### Правильный алгоритм:

```bash
# 1. Делаем бэкап с датой и временем
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak_$(date +%Y%m%d_%H%M%S)

# 2. Проверяем, что бэкап создался
ls -la /etc/nginx/nginx.conf*
# Вывод:
# -rw-r--r-- 1 root root 2345 Oct  6 14:30 /etc/nginx/nginx.conf
# -rw-r--r-- 1 root root 2345 Oct  6 14:30 /etc/nginx/nginx.conf.bak_20261006_143000

# 3. Редактируем
sudo nano /etc/nginx/nginx.conf

# 4. Если всё сломалось — откатываемся
sudo cp /etc/nginx/nginx.conf.bak_20261006_143000 /etc/nginx/nginx.conf
```

### Однострочник для ленивых (но безопасный):

```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak_$(date +%F_%T) && sudo nano /etc/nginx/nginx.conf
```

> 💡 **Почему с датой?** Потому что вы можете редактировать конфиг несколько раз за день. Бэкапы `config.bak`, `config.bak.old`, `config.bak.real` быстро превращаются в кашу. Дата решает эту проблему.

---

## ✅ Часть 4. Проверка синтаксиса перед применением

**Никогда не перезапускайте сервис сразу после редактирования конфига.** Сначала проверьте синтаксис.

### Nginx
```bash
sudo nginx -t
# Правильный вывод:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Apache
```bash
sudo apachectl configtest
# Правильный вывод:
# Syntax OK
```

### SSH
```bash
sudo sshd -t
# Если всё ок — тишина (нет вывода = нет ошибок)
```

### Docker Compose
```bash
docker compose config
# Выводит итоговый конфиг (если есть ошибки — ругается)
```

### PostgreSQL
```bash
# Проверка postgresql.conf
sudo -u postgres /usr/lib/postgresql/15/bin/postgres -D /var/lib/postgresql/15/main -C config_file
```

### Общий паттерн:
```bash
# 1. Редактируем
sudo nano /etc/nginx/sites-available/emh

# 2. Проверяем синтаксис
sudo nginx -t

# 3. Если OK — применяем (reload безопаснее restart)
sudo systemctl reload nginx

# 4. Если ошибка — откатываемся на бэкап
sudo cp /etc/nginx/sites-available/emh.bak_20261006_143000 /etc/nginx/sites-available/emh
sudo nginx -t
sudo systemctl reload nginx
```

> ⚠️ **Разница между `reload` и `restart`:**
> - `reload` — перечитывает конфиги, не разрывая существующие соединения. Идеально для Nginx, Apache.
> - `restart` — полностью останавливает и запускает заново. Есть кратковременный простой. Используйте, если `reload` не поддерживает изменения.

---

## 🔐 Часть 5. Безопасная работа с секретами

**Секреты** — это пароли, API-ключи, токены, SSH-ключи. Их утечка = катастрофа для больницы.

### 1. Файлы `.env`

Файлы `.env` содержат переменные окружения с секретами:

```env
# /opt/emh/deploy/.env
DB_PASSWORD=super_secret_password_123
API_KEY=sk_live_abc123xyz
REDIS_URL=redis://user:password@redis:6379
```

**Правила работы:**
```bash
# Никогда не коммитьте .env в Git!
# Добавьте в .gitignore:
echo ".env" >> .gitignore

# Права должны быть строгими: только владелец читает
chmod 600 /opt/emh/deploy/.env
ls -la /opt/emh/deploy/.env
# Вывод: -rw------- 1 deploy deploy 512 Oct  6 14:30 .env

# Не показывайте содержимое .env в логах или чатах
cat /opt/emh/deploy/.env  # Только когда действительно нужно
```

### 2. SSH-ключи

**Приватный ключ** (например, `~/.ssh/id_rsa`) — это ваш паспорт. Никогда не передавайте его.

```bash
# Правильные права на приватный ключ
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Правильные права на папку .ssh
chmod 700 ~/.ssh
```

**Публичный ключ** (`id_rsa.pub`) можно передавать — его добавляют в `~/.ssh/authorized_keys` на сервере.

### 3. Передача секретов

**Никогда не передавайте секреты через:**
- ❌ Telegram/WhatsApp (сохраняются в истории)
- ❌ Email (сохраняется на серверах)
- ❌ Скриншоты (сохраняются в галерее)
- ❌ Общий чат (увидят все)

**Правильные способы:**
- ✅ **Bitwarden / 1Password** — менеджер паролей с функцией шаринга
- ✅ **Vault** (HashiCorp) — для серверных секретов
- ✅ **Одноразовые ссылки** (например, через `privnote.com` или корпоративный аналог)
- ✅ **Личное сообщение + отдельный канал для пароля** (например, имя пользователя в Telegram, пароль в SMS)

---

## 🔑 Часть 6. SSH и безопасный доступ

### 1. Подключение по SSH

```bash
ssh user@server.hospital.local
ssh -p 2222 user@server.hospital.local  # Нестандартный порт
ssh -i ~/.ssh/emh_key user@server.hospital.local  # С конкретным ключом
```

### 2. SSH-конфиг (`~/.ssh/config`)

Чтобы не запоминать длинные команды:

```bash
nano ~/.ssh/config
```

**Пример конфига:**
```
Host emh-prod
    HostName 192.168.1.100
    User deploy
    Port 22
    IdentityFile ~/.ssh/emh_key
    ServerAliveInterval 60

Host emh-staging
    HostName staging.emh.local
    User deploy
    IdentityFile ~/.ssh/emh_key
```

**Использование:**
```bash
ssh emh-prod  # Вместо ssh deploy@192.168.1.100
```

### 3. Копирование файлов через SSH

```bash
# Копировать файл с локальной машины на сервер
scp local_file.txt user@server:/tmp/

# Копировать файл с сервера на локальную машину
scp user@server:/var/log/nginx/access.log ./

# Копировать папку рекурсивно
scp -r ./configs/ user@server:/opt/emh/
```

### 4. Туннели SSH (проброс портов)

Если нужно подключиться к базе данных, которая не торчит наружу:

```bash
# Пробросить порт 5432 с сервера на локальную машину
ssh -L 5432:localhost:5432 user@server.hospital.local

# Теперь можно подключиться к БД через localhost:5432
psql -h localhost -p 5432 -U emh_user emh_db
```

---

## 🛡 Часть 7. Золотые правила безопасности

### 1. Не работайте под root

```bash
# ПЛОХО:
sudo -i
# ...делаете что-то...
rm -rf /var/www/  # Опечатка = потеряли сайт

# ХОРОШО:
# Работаете под своим пользователем
sudo systemctl restart nginx  # Только нужные команды через sudo
```

### 2. Проверяйте команды перед выполнением

**Никогда не копируйте слепо команды из интернета.** Особенно если там:
- `curl ... | sudo bash`
- `wget ... | sudo sh`
- `rm -rf`
- `chmod 777`

```bash
# ПЛОХО:
curl https://sketchy-site.com/install.sh | sudo bash

# ХОРОШО:
# Сначала скачиваем скрипт и читаем
curl https://sketchy-site.com/install.sh -o install.sh
nano install.sh  # Читаем, что он делает
# Если всё ок — запускаем
sudo bash install.sh
```

### 3. Минимальные привилегии

Сервисы должны работать от отдельных пользователей с минимальными правами:

```bash
# ПЛОХО: запускать всё от root
# ХОРОШО: отдельные пользователи для сервисов
sudo useradd -r -s /bin/false emh-backend
sudo useradd -r -s /bin/false emh-worker
```

### 4. Обновляйте систему

```bash
# Регулярно проверяйте обновления безопасности
sudo apt update
sudo apt list --upgradable

# Устанавливайте обновления безопасности
sudo apt upgrade
```

> 💡 **Совет:** Настройте `unattended-upgrades` для автоматической установки обновлений безопасности.

### 5. Логируйте всё

```bash
# Проверяйте логи авторизаций
sudo tail -f /var/log/auth.log  # Debian/Ubuntu
sudo tail -f /var/log/secure    # RHEL/CentOS

# Ищите подозрительные попытки входа
sudo grep "Failed password" /var/log/auth.log | tail -20
```

---

## 🏥 Часть 8. Практические кейсы из больницы

### Кейс 1: «Нужно изменить порт Nginx с 80 на 8080»

**Симптом:** На порту 80 уже висит другой сервис, нужно перевести EMH на 8080.

**Решение:**

```bash
# 1. Делаем бэкап
sudo cp /etc/nginx/sites-available/emh /etc/nginx/sites-available/emh.bak_$(date +%F_%T)

# 2. Редактируем
sudo nano /etc/nginx/sites-available/emh
# Меняем: listen 80; -> listen 8080;

# 3. Проверяем синтаксис
sudo nginx -t
# Вывод: nginx: configuration file /etc/nginx/nginx.conf test is successful ✅

# 4. Применяем (reload безопаснее restart)
sudo systemctl reload nginx

# 5. Проверяем, что сервис слушает новый порт
sudo ss -tlnp | grep nginx
# Вывод: LISTEN 0 511 0.0.0.0:8080 0.0.0.0:* users:(("nginx",pid=1234,fd=6))

# 6. Проверяем доступность
curl -I http://localhost:8080
# Вывод: HTTP/1.1 200 OK ✅
```

---

### Кейс 2: «Сервис падает после редактирования docker-compose.yml»

**Симптом:** После изменения `docker-compose.yml` сервис не запускается.

**Диагностика:**

```bash
cd /opt/emh/deploy/

# Проверяем синтаксис
docker compose config
# Вывод: yaml: line 45: mapping values are not allowed in this context

# Ищем ошибку в конфиге
nano docker-compose.yml
# На строке 45 видим:
# environment:
#   DB_HOST:postgres  # Забыли пробел после двоеточия!
```

**Решение:**

```bash
# 1. Откатываемся на бэкап
cp docker-compose.yml.bak_20261006_143000 docker-compose.yml

# 2. Проверяем
docker compose config
# Вывод: (корректный YAML) ✅

# 3. Вносим правки аккуратно
nano docker-compose.yml
# environment:
#   DB_HOST: postgres  # С пробелом

# 4. Снова проверяем
docker compose config  # ✅

# 5. Применяем
docker compose up -d
```

---

### Кейс 3: «Нужно передать пароль от БД разработчику»

**Симптом:** Разработчик просит пароль от продакшен-БД для отладки.

**ПЛОХОЕ решение:**
```
# В Telegram:
"Пароль от БД: SuperSecret123!"
```

**ХОРОШЕЕ решение:**
```bash
# 1. Узнаём, действительно ли нужен продакшен
# 2. Если да — создаём отдельного пользователя с правами только на чтение
sudo -u postgres psql
CREATE USER dev_readonly WITH PASSWORD 'temp_password_123';
GRANT CONNECT ON DATABASE emh_prod TO dev_readonly;
GRANT USAGE ON SCHEMA public TO dev_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO dev_readonly;
\q

# 3. Передаём временный пароль через защищённый канал
# (например, через корпоративный менеджер паролей)

# 4. После завершения работ — удаляем пользователя
sudo -u postgres psql
DROP USER dev_readonly;
\q
```

---

### Кейс 4: «Нужно быстро внести правку в конфиг, но nano не открывается»

**Симптом:** Нужно изменить одну строку в конфиге, но нет времени открывать редактор.

**Решение через `sed`:**

```bash
# 1. Делаем бэкап (ОБЯЗАТЕЛЬНО!)
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak_$(date +%F_%T)

# 2. Заменяем строку
sudo sed -i 's/listen 80;/listen 8080;/' /etc/nginx/nginx.conf

# 3. Проверяем, что изменилось
sudo diff /etc/nginx/nginx.conf.bak_* /etc/nginx/nginx.conf
# Вывод: 
# <     listen 80;
# ---
# >     listen 8080;

# 4. Проверяем синтаксис
sudo nginx -t

# 5. Применяем
sudo systemctl reload nginx
```

> ⚠️ **Важно:** `sed` мощный, но опасный. Всегда делайте бэкап и проверяйте результат через `diff`.

---

### Кейс 5: «Сервер взломали, что делать?»

**Симптом:** В логах подозрительная активность, неизвестные процессы.

**Алгоритм действий:**

```bash
# 1. Не паниковать. Не перезагружать сервер (потеряете улики).

# 2. Изолируем сервер (если возможно)
# Отключаем от сети или блокируем трафик через firewall

# 3. Сохраняем улики
sudo dd if=/dev/sda of=/mnt/backup/disk_image.img bs=4M  # Образ диска
sudo tar czf /mnt/backup/logs_$(date +%F).tar.gz /var/log/

# 4. Проверяем подозрительные процессы
ps auxf
sudo netstat -tulpn  # Какие порты открыты

# 5. Ищем неизвестных пользователей
cat /etc/passwd | grep -E '/bin/(bash|sh)'

# 6. Сообщаем руководителю и службе безопасности
# 7. Начинаем расследование с сохранения улик
```

> 🚨 **Главное правило:** При подозрении на взлом — **сохраняйте улики, а не чините**. Починить можно потом, а улики исчезнут.

---

## 📋 Шпаргалка

### Редакторы

| Редактор | Открыть | Сохранить | Выйти | Поиск |
|----------|---------|-----------|-------|-------|
| `nano` | `nano file` | `Ctrl+O`, `Enter` | `Ctrl+X` | `Ctrl+W` |
| `vim` | `vim file` | `:w` | `:q` | `/word` |

### Безопасное редактирование конфигов

```bash
# 1. Бэкап
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak_$(date +%F_%T)

# 2. Редактирование
sudo nano /etc/nginx/nginx.conf

# 3. Проверка синтаксиса
sudo nginx -t

# 4. Применение
sudo systemctl reload nginx

# 5. Если ошибка — откат
sudo cp /etc/nginx/nginx.conf.bak_20261006_143000 /etc/nginx/nginx.conf
sudo systemctl reload nginx
```

### Права на секреты

| Файл | Права | Команда |
|------|-------|---------|
| `.env` | `600` (только владелец) | `chmod 600 .env` |
| Приватный SSH-ключ | `600` | `chmod 600 ~/.ssh/id_rsa` |
| Публичный SSH-ключ | `644` | `chmod 644 ~/.ssh/id_rsa.pub` |
| Папка `.ssh` | `700` | `chmod 700 ~/.ssh` |

### Проверка синтаксиса

| Сервис | Команда |
|--------|---------|
| Nginx | `sudo nginx -t` |
| Apache | `sudo apachectl configtest` |
| SSH | `sudo sshd -t` |
| Docker Compose | `docker compose config` |

### Золотые правила

1. **Бэкап перед любым изменением** — `cp config config.bak_$(date +%F_%T)`
2. **Проверка синтаксиса перед применением** — `nginx -t`, `docker compose config`
3. **Не работайте под root** — используйте `sudo` только когда нужно
4. **Не копируйте слепо команды из интернета** — читайте, что делает команда
5. **Секреты — только через защищённые каналы** — менеджер паролей, Vault
6. **Минимальные привилегии** — сервисы работают от отдельных пользователей
7. **Логируйте всё** — `/var/log/auth.log` ваш друг
8. **При взломе — сохраняйте улики, а не чините**

---

## 🎓 Финальное напутствие

Поздравляю! Вы прошли все 5 гайдов:

1. ✅ **Работа с файлами** — уверенно ориентируетесь в терминале
2. ✅ **Пользователи и права** — понимаете, почему `Permission denied`
3. ✅ **Службы и сервисы** — можете реанимировать упавшие сервисы
4. ✅ **Пакетный менеджер** — устанавливаете софт без проблем
5. ✅ **Конфиги и безопасность** — вносите изменения безопасно

Теперь вы готовы к работе с EMH и другими системами больницы. Помните:

> **«Семь раз отмерь — один раз отрежь»** — это не просто поговорка, это мантра сисадмина.

Удачи в работе! 🚀


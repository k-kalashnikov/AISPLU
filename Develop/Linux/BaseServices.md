# 📘 Гайд 3. Службы и сервисы

**Цель:** Научиться диагностировать, перезапускать и мониторить службы (systemd и Docker), понимать, почему сервис упал, и находить процессы, которые «едят» ресурсы сервера.

---

## 🧠 Часть 1. Что такое служба (сервис)?

**Служба (service/daemon)** — это программа, которая работает в фоне и выполняет какую-то задачу: веб-сервер, база данных, SSH-сервер, планировщик задач и т.д.

В Linux есть **две основные системы** управления службами:

| Система | Для чего | Пример |
|---------|----------|--------|
| **systemd** | Системные службы, работающие на хосте | `nginx`, `postgresql`, `ssh`, `cron` |
| **Docker** | Контейнеризированные приложения | `emh-frontend`, `emh-backend`, `emh-postgres-prod` |

> 💡 **Важно:** В вашем проекте EMH большинство сервисов работают в Docker-контейнерах, но сам Docker-демон (`docker.service`) управляется через systemd. Нужно понимать обе системы.

---

## 🔧 Часть 2. Systemd: управление системными службами

### 1. Проверка статуса службы

```bash
sudo systemctl status nginx
```

**Что вы увидите:**
```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2026-10-06 09:00:00 MSK; 2h ago
     Docs: man:nginx(8)
  Process: 1234 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 1235 ExecStart=/usr/sbin/nginx -g daemon on; master process on; (code=exited, status=0/SUCCESS)
 Main PID: 1236 (nginx)
    Tasks: 3 (limit: 4915)
   Memory: 12.5M
   CGroup: /system.slice/nginx.service
           ├─1236 nginx: master process /usr/sbin/nginx -g daemon on; master process on;
           ├─1237 nginx: worker process
           └─1238 nginx: worker process

Oct 06 09:00:00 hospital-srv systemd[1]: Starting A high performance web server...
Oct 06 09:00:00 hospital-srv systemd[1]: Started A high performance web server...
```

**Ключевые поля:**
- `Active: active (running)` — служба работает ✅
- `Active: inactive (dead)` — служба остановлена ❌
- `Active: failed` — служба упала с ошибкой 🚨
- `Loaded: ... enabled` — служба запускается при старте системы
- `Loaded: ... disabled` — служба НЕ запускается автоматически
- `Main PID: 1236` — ID главного процесса (нужен для `kill` или `top`)

### 2. Управление службой

```bash
sudo systemctl start nginx      # Запустить
sudo systemctl stop nginx       # Остановить
sudo systemctl restart nginx    # Перезапустить (stop + start)
sudo systemctl reload nginx     # Перечитать конфиги БЕЗ остановки (если поддерживается)

sudo systemctl enable nginx     # Включить автозагрузку при старте системы
sudo systemctl disable nginx    # Отключить автозагрузку
```

> 💡 **Разница между `restart` и `reload`:**
> - `restart` — полностью останавливает и запускает службу. Есть кратковременный простой.
> - `reload` — перечитывает конфиги без остановки. Используется для Nginx, Apache, systemd-resolved.
> - **Не все службы поддерживают `reload`!** Если не уверены — используйте `restart`.

### 3. Список всех служб

```bash
systemctl list-units --type=service          # Все запущенные службы
systemctl list-units --type=service --all    # Все службы (включая остановленные)
systemctl list-unit-files --type=service     # Все службы с их статусом автозагрузки
```

**Фильтрация:**
```bash
systemctl list-units --type=service | grep nginx     # Найти службы с "nginx" в имени
systemctl list-units --type=service --state=failed   # Показать только упавшие службы
```

### 4. Чтение логов службы (journalctl)

```bash
sudo journalctl -u nginx                    # Все логи службы nginx
sudo journalctl -u nginx --since today      # Логи за сегодня
sudo journalctl -u nginx --since "2026-10-06 14:00:00"  # Логи с конкретного времени
sudo journalctl -u nginx -n 50              # Последние 50 строк
sudo journalctl -u nginx -f                 # Следить в реальном времени (как tail -f)
sudo journalctl -u nginx --no-pager         # Без постраничного вывода (для копирования)
```

> 💡 **Совет:** Если служба упала, начните с `journalctl -u <service> --since "1 hour ago" -n 100` — увидите последние 100 строк за последний час.

---

## 🐳 Часть 3. Docker: управление контейнерными сервисами

### 1. Проверка статуса контейнеров

```bash
docker ps                 # Показать запущенные контейнеры
docker ps -a              # Показать ВСЕ контейнеры (включая остановленные)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"  # Красивая таблица
```

**Пример вывода:**
```
CONTAINER ID   IMAGE              STATUS          PORTS                  NAMES
a1b2c3d4e5f6   nginx:latest       Up 2 hours      0.0.0.0:80->80/tcp     emh-frontend
f6e5d4c3b2a1   postgres:15        Up 2 hours      5432/tcp               emh-postgres-prod
1a2b3c4d5e6f   minio/minio        Up 2 hours      0.0.0.0:9000->9000/tcp emh-minio
```

**Статусы:**
- `Up 2 hours` — контейнер работает ✅
- `Exited (0) 5 minutes ago` — контейнер остановлен нормально (код 0)
- `Exited (1) 5 minutes ago` — контейнер упал с ошибкой (код 1) 🚨
- `Restarting` — контейнер в цикле перезапуска 🚨

### 2. Управление контейнерами

```bash
docker start emh-frontend       # Запустить остановленный контейнер
docker stop emh-frontend        # Остановить контейнер (graceful shutdown)
docker restart emh-frontend     # Перезапустить контейнер
docker kill emh-frontend        # Принудительно остановить (если `stop` не помогает)
```

### 3. Просмотр логов контейнера

```bash
docker logs emh-frontend                    # Все логи
docker logs --tail 50 emh-frontend          # Последние 50 строк
docker logs --since 1h emh-frontend         # Логи за последний час
docker logs -f emh-frontend                 # Следить в реальном времени
docker logs --tail 50 -f emh-frontend       # Последние 50 строк + следить дальше
```

> 💡 **Важно:** Если контейнер упал сразу после старта, логи могут быть пустыми. Проверьте:
> ```bash
> docker inspect emh-frontend --format='{{.State.ExitCode}}'  # Код ошибки
> docker inspect emh-frontend --format='{{.State.Error}}'     # Текст ошибки
> ```

### 4. Docker Compose (если используется)

Если ваши сервисы управляются через `docker-compose.yml`:

```bash
cd /opt/emh/deploy/

docker compose ps                   # Статус всех сервисов из compose-файла
docker compose logs frontend        # Логи конкретного сервиса
docker compose logs -f              # Следить за всеми логами
docker compose restart backend      # Перезапустить один сервис
docker compose up -d                # Запустить все сервисы (или пересоздать измененные)
docker compose down                 # Остановить и удалить все контейнеры
docker compose pull                 # Обновить образы (перед `up -d`)
```

> ⚠️ **Важно:** Если вы используете Docker Compose, управляйте сервисами через `docker compose`, а не через `docker start/stop`. Иначе Compose потеряет контроль над контейнерами.

---

## 📊 Часть 4. Мониторинг ресурсов: кто «ест» сервер?

### 1. `top` — классический монитор процессов

```bash
top
```

**Что видите:**
- `PID` — ID процесса
- `USER` — владелец процесса
- `PR/NI` — приоритет
- `VIRT/RES/SHR` — использование памяти (виртуальная/резидентная/разделяемая)
- `S` — статус (R=работает, S=спит, Z=зомби)
- `%CPU` — загрузка процессора
- `%MEM` — использование памяти
- `TIME+` — время процессора
- `COMMAND` — имя команды

**Горячие клавиши в `top`:**
- `P` — сортировать по `%CPU`
- `M` — сортировать по `%MEM`
- `k` — убить процесс (введите PID)
- `q` — выйти

### 2. `htop` — улучшенный `top` (если установлен)

```bash
htop
```

**Преимущества:**
- Цветной интерфейс
- Древовидный вид процессов (`F5`)
- Удобная сортировка (`F6`)
- Поиск (`F3`)
- Можно убивать процессы клавишами (`F9`)

> 💡 **Если `htop` не установлен:**
> ```bash
> sudo apt update
> sudo apt install htop
> ```

### 3. `ps` — снимок процессов

```bash
ps aux                              # Все процессы
ps aux | grep nginx                 # Найти процессы с "nginx"
ps aux --sort=-%cpu | head -10      # Топ-10 процессов по CPU
ps aux --sort=-%mem | head -10      # Топ-10 процессов по памяти
ps -ef                              # Альтернативный формат (как в старых UNIX)
```

### 4. Поиск «тяжеловесов»

**Найти процессы, которые едят больше 50% CPU:**
```bash
ps aux | awk '$3 > 50.0 {print $0}'
```

**Найти процессы, которые едят больше 500 МБ памяти:**
```bash
ps aux | awk '$6 > 500000 {print $0}'
```

### 5. Мониторинг дисков

```bash
df -h                               # Использование дисков
df -h | grep -E '^/dev/'            # Только реальные диски (исключить tmpfs и т.д.)

du -sh /var/log/*                   # Размер каждой папки в /var/log
du -sh /opt/emh/*                   # Размер каждой папки в /opt/emh
du -sh /* 2>/dev/null | sort -h     # Размер всех папок в корне (сортировка по размеру)
```

> 💡 **Совет:** Если диск забит, начните с:
> ```bash
> sudo du -sh /* 2>/dev/null | sort -hr | head -20
> ```
> Это покажет 20 самых «тяжелых» папок в корне.

---

## 🏥 Часть 5. Практические кейсы из больницы

### Кейс 1: «Сайт EMH не открывается, 502 Bad Gateway»

**Симптом:** Врачи не могут зайти на Employee Virtual Portal, браузер показывает `502 Bad Gateway`.

**Диагностика:**

1. **Проверяем Nginx (фронтенд):**
   ```bash
   sudo systemctl status nginx
   # Вывод: Active: active (running) ✅
   ```
   Nginx работает, проблема не в нём.

2. **Проверяем бэкенд (Docker-контейнер):**
   ```bash
   docker ps | grep emh-backend
   # Вывод: (пусто!) Контейнер не запущен!
   
   docker ps -a | grep emh-backend
   # Вывод: Exited (1) 5 minutes ago ❌
   ```
   Контейнер упал с кодом 1.

3. **Смотрим логи контейнера:**
   ```bash
   docker logs --tail 50 emh-backend
   # Вывод: 
   # Error: Database connection failed
   # FATAL: too many connections for role "emh_user"
   ```
   Проблема с базой данных — слишком много подключений.

4. **Проверяем PostgreSQL:**
   ```bash
   docker ps | grep emh-postgres
   # Вывод: Up 2 hours ✅
   
   docker logs --tail 20 emh-postgres-prod
   # Вывод: LOG: connection received: host=... 
   #        LOG: too many connections for role "emh_user"
   ```

**Решение:**
```bash
# Перезапускаем PostgreSQL (сбросит зависшие подключения)
docker restart emh-postgres-prod

# Ждем 10 секунд, проверяем
docker ps | grep emh-postgres
# Вывод: Up 10 seconds ✅

# Перезапускаем бэкенд
docker restart emh-backend

# Проверяем
docker ps | grep emh-backend
# Вывод: Up 5 seconds ✅

# Проверяем сайт
curl -I http://localhost
# Вывод: HTTP/1.1 200 OK ✅
```

---

### Кейс 2: «Сервер тормозит, врачи жалуются»

**Симптом:** Все сервисы EMH работают медленно, врачи жалуются на долгие загрузки.

**Диагностика:**

1. **Проверяем загрузку CPU и памяти:**
   ```bash
   top
   # Видим: load average: 12.5, 10.2, 8.7 (очень высоко!)
   # Сортируем по CPU (нажимаем P)
   # Видим: процесс java с PID 5678 ест 95% CPU
   ```

2. **Узнаём, что за процесс:**
   ```bash
   ps aux | grep 5678
   # Вывод: emh_user 5678 95.0 45.2 2:34:56 java -jar /opt/emh/backend/app.jar
   ```
   Это Java-приложение EMH.

3. **Проверяем память:**
   ```bash
   free -h
   # Вывод: 
   #               total        used        free      shared  buff/cache   available
   # Mem:           15Gi        14Gi       200Mi       1.2Gi       800Mi       500Mi
   # Swap:         2.0Gi       2.0Gi          0B
   ```
   Память почти вся занята, swap полностью использован!

4. **Проверяем логи приложения:**
   ```bash
   docker logs --tail 100 emh-backend | grep -i "error\|exception"
   # Вывод: 
   # java.lang.OutOfMemoryError: Java heap space
   # GC overhead limit exceeded
   ```
   Java-приложение «течет» — не освобождает память.

**Решение:**
```bash
# Временно: перезапускаем контейнер (освободит память)
docker restart emh-backend

# Долгосрочно: увеличиваем лимиты памяти в docker-compose.yml
# или передаем разработчикам для исправления утечки памяти
```

---

### Кейс 3: «Docker-контейнер в цикле перезапуска»

**Симптом:** Контейнер `emh-worker` постоянно перезапускается, задачи не выполняются.

**Диагностика:**

1. **Проверяем статус:**
   ```bash
   docker ps -a | grep emh-worker
   # Вывод: Restarting (1) 2 seconds ago
   ```
   Контейнер в цикле перезапуска.

2. **Смотрим логи:**
   ```bash
   docker logs --tail 50 emh-worker
   # Вывод:
   # Error: Could not connect to Redis at redis:6379
   # Connection refused
   ```
   Контейнер не может подключиться к Redis.

3. **Проверяем Redis:**
   ```bash
   docker ps | grep redis
   # Вывод: (пусто!) Redis не запущен!
   ```

**Решение:**
```bash
# Запускаем Redis
cd /opt/emh/deploy/
docker compose up -d redis

# Проверяем
docker ps | grep redis
# Вывод: Up 5 seconds ✅

# Проверяем worker
docker ps | grep emh-worker
# Вывод: Up 10 seconds ✅ (вышел из цикла перезапуска)
```

---

### Кейс 4: «Закончилось место на сервере»

**Симптом:** Сервисы EMH падают с ошибками `No space left on device`.

**Диагностика:**

1. **Проверяем использование дисков:**
   ```bash
   df -h
   # Вывод:
   # Filesystem      Size  Used Avail Use% Mounted on
   # /dev/sda1       100G   98G  2.0G  98% /
   ```
   Диск забит на 98%.

2. **Находим самые «тяжелые» папки:**
   ```bash
   sudo du -sh /* 2>/dev/null | sort -hr | head -10
   # Вывод:
   # 45G   /var
   # 30G   /opt
   # 15G   /usr
   # ...
   ```

3. **Углубляемся в /var:**
   ```bash
   sudo du -sh /var/* 2>/dev/null | sort -hr | head -10
   # Вывод:
   # 40G   /var/log
   # 3G    /var/lib
   ```

4. **Смотрим, что в логах:**
   ```bash
   sudo du -sh /var/log/* 2>/dev/null | sort -hr | head -10
   # Вывод:
   # 25G   /var/log/emh-backend.log
   # 10G   /var/log/syslog
   # 5G    /var/log/kern.log
   ```
   Лог бэкенда разросся до 25 ГБ!

**Решение:**
```bash
# Очищаем старые логи (оставляем только последние 100 МБ)
sudo truncate -s 100M /var/log/emh-backend.log

# Настраиваем logrotate, чтобы логи автоматически ротировались
sudo nano /etc/logrotate.d/emh-backend
# Добавляем:
# /var/log/emh-backend.log {
#     daily
#     rotate 7
#     compress
#     missingok
#     notifempty
#     maxsize 500M
# }

# Проверяем место
df -h
# Вывод: /dev/sda1 100G 73G 27G 73% / ✅
```

---

## 📋 Шпаргалка

### Systemd

| Команда | Что делает |
|---------|-----------|
| `systemctl status <service>` | Статус службы |
| `systemctl start <service>` | Запустить |
| `systemctl stop <service>` | Остановить |
| `systemctl restart <service>` | Перезапустить |
| `systemctl reload <service>` | Перечитать конфиги |
| `systemctl enable <service>` | Включить автозагрузку |
| `systemctl disable <service>` | Отключить автозагрузку |
| `systemctl list-units --type=service` | Список всех служб |
| `systemctl list-units --state=failed` | Упавшие службы |
| `journalctl -u <service>` | Логи службы |
| `journalctl -u <service> -f` | Логи в реальном времени |
| `journalctl -u <service> --since today` | Логи за сегодня |

### Docker

| Команда | Что делает |
|---------|-----------|
| `docker ps` | Запущенные контейнеры |
| `docker ps -a` | Все контейнеры |
| `docker start <container>` | Запустить контейнер |
| `docker stop <container>` | Остановить контейнер |
| `docker restart <container>` | Перезапустить контейнер |
| `docker logs <container>` | Логи контейнера |
| `docker logs -f <container>` | Логи в реальном времени |
| `docker logs --tail 50 <container>` | Последние 50 строк |
| `docker inspect <container>` | Детальная информация |
| `docker compose ps` | Статус всех сервисов (Compose) |
| `docker compose logs <service>` | Логи сервиса (Compose) |
| `docker compose restart <service>` | Перезапустить сервис (Compose) |
| `docker compose up -d` | Запустить все сервисы (Compose) |
| `docker compose down` | Остановить все сервисы (Compose) |

### Мониторинг

| Команда | Что делает |
|---------|-----------|
| `top` | Монитор процессов (классика) |
| `htop` | Улучшенный монитор процессов |
| `ps aux` | Список всех процессов |
| `ps aux \| grep <name>` | Найти процесс по имени |
| `free -h` | Использование памяти |
| `df -h` | Использование дисков |
| `du -sh <path>` | Размер папки |
| `du -sh /* \| sort -hr` | Размер всех папок в корне |

---

### Быстрая диагностика упавшего сервиса:

1. **Это systemd или Docker?**
   - `systemctl status <service>` — если есть, это systemd
   - `docker ps -a \| grep <service>` — если есть, это Docker

2. **Почему упал?**
   - systemd: `journalctl -u <service> -n 50`
   - Docker: `docker logs --tail 50 <container>`

3. **Как запустить?**
   - systemd: `sudo systemctl start <service>`
   - Docker: `docker start <container>` или `docker compose up -d`

4. **Сервер тормозит?**
   - `top` или `htop` — смотрим, кто ест CPU/RAM
   - `df -h` — проверяем место на дисках
   - `free -h` — проверяем память

---

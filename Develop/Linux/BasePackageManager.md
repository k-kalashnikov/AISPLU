# 📘 Гайд 4. Пакетный менеджер и репозитории

**Цель:** Научиться устанавливать, обновлять и удалять программы через терминал, понимать, откуда система берёт пакеты, и решать типичные проблемы с зависимостями и заблокированными файлами.

---

## 🧠 Часть 1. Зачем нужен пакетный менеджер?

В отличие от Windows, где вы скачиваете `.exe` с сайта программы и устанавливаете её вручную, в Linux программы устанавливаются из **репозиториев** — централизованных хранилищ пакетов.

**Пакетный менеджер** — это программа, которая:
- Скачивает пакеты из репозиториев
- Автоматически разрешает зависимости (если программа А требует библиотеку Б, менеджер сам её установит)
- Обновляет программы до актуальных версий
- Корректно удаляет программы вместе с зависимостями

В **Debian** основной пакетный менеджер — **APT** (Advanced Package Tool).

> 💡 **Почему это лучше, чем `.exe`?**
> - **Безопасность:** пакеты берутся из официальных репозиториев, а не с сомнительных сайтов
> - **Централизованное обновление:** одной командой обновляете ВСЕ программы в системе
> - **Автоматические зависимости:** не нужно вручную искать и ставить библиотеки

---

## 📦 Часть 2. Основные команды APT

### 1. Обновить списки пакетов

```bash
sudo apt update
```

**Что делает:** Скачивает свежие списки пакетов и их версий из репозиториев. Ничего не устанавливает, не обновляет. Просто обновляет «каталог товаров».

> ⚠️ **Важно:** Эту команду нужно выполнять **перед** каждой установкой или обновлением. Иначе система может попытаться установить старую или несуществующую версию пакета.

### 2. Обновить установленные пакеты

```bash
sudo apt upgrade
```

**Что делает:** Обновляет все установленные пакеты до новых версий, которые есть в репозиториях. Не удаляет старые пакеты, не устанавливает новые зависимости.

```bash
sudo apt full-upgrade
```

**Что делает:** Более агрессивное обновление. Может удалять старые пакеты или устанавливать новые зависимости, если это требуется для обновления.

> 💡 **Когда использовать что:**
> - `apt upgrade` — безопасное обновление, подходит для продакшена
> - `apt full-upgrade` — если `apt upgrade` не может обновить какие-то пакеты из-за конфликтов зависимостей

### 3. Установить пакет

```bash
sudo apt install htop
```

**Что делает:** Устанавливает пакет `htop` и все его зависимости.

**Несколько пакетов сразу:**
```bash
sudo apt install htop curl wget vim
```

**Установить конкретную версию:**
```bash
sudo apt install postgresql-15=15.4-1.pgdg120+1
```

> ⚠️ **Важно:** Перед установкой нового пакета всегда делайте `sudo apt update`, иначе может быть ошибка `Unable to locate package`.

### 4. Удалить пакет

```bash
sudo apt remove htop
```

**Что делает:** Удаляет программу, но **оставляет** её конфигурационные файлы.

```bash
sudo apt purge htop
```

**Что делает:** Полностью удаляет программу **вместе с конфигурационными файлами**.

```bash
sudo apt autoremove
```

**Что делает:** Удаляет пакеты, которые были установлены автоматически как зависимости и больше не нужны.

> 💡 **Совет:** После удаления ненужной программы всегда делайте `sudo apt autoremove`, чтобы освободить место и убрать мусор.

### 5. Поиск пакетов

```bash
apt search htop
```

**Что делает:** Ищет пакеты, в названии или описании которых есть слово `htop`.

```bash
apt show htop
```

**Что делает:** Показывает подробную информацию о пакете: версия, размер, зависимости, описание.

### 6. Список установленных пакетов

```bash
apt list --installed
```

**Что делает:** Показывает все установленные пакеты.

```bash
apt list --installed | grep nginx
```

**Что делает:** Ищет установленные пакеты с «nginx» в имени.

---

## 📂 Часть 3. Репозитории: откуда система берёт пакеты

### 1. Где хранится список репозиториев

```bash
cat /etc/apt/sources.list
```

**Пример вывода:**
```
deb http://deb.debian.org/debian bookworm main contrib non-free
deb http://deb.debian.org/debian bookworm-updates main contrib non-free
deb http://security.debian.org/debian-security bookworm-security main contrib non-free
```

**Что означают компоненты:**
| Компонент | Что содержит |
|-----------|-------------|
| `main` | Официальное свободное ПО (полностью свободные лицензии) |
| `contrib` | Свободное ПО, зависящее от несвободных библиотек |
| `non-free` | Несвободное ПО (проприетарные драйверы, кодеки) |

> 💡 **Совет:** Для серверов обычно достаточно `main`. `non-free` нужен, если нужны проприетарные драйверы (например, для видеокарт или сетевых адаптеров).

### 2. Дополнительные репозитории

Некоторые программы (например, Docker, PostgreSQL последних версий) отсутствуют в официальных репозиториях Debian или представлены старыми версиями. Их нужно добавлять отдельно.

**Пример: добавление репозитория PostgreSQL**
```bash
# Устанавливаем зависимости
sudo apt install curl ca-certificates gnupg

# Добавляем GPG-ключ репозитория
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /usr/share/keyrings/postgresql-keyring.gpg

# Добавляем репозиторий
echo "deb [signed-by=/usr/share/keyrings/postgresql-keyring.gpg] http://apt.postgresql.org/pub/repos/apt bookworm-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# Обновляем списки пакетов
sudo apt update

# Устанавливаем PostgreSQL 16
sudo apt install postgresql-16
```

> ⚠️ **Важно:** Добавляйте репозитории только из проверенных источников. Сторонний репозиторий = доверие к его владельцу. Для больницы это критично с точки зрения безопасности.

### 3. Как посмотреть, из какого репозитория установлен пакет

```bash
apt policy htop
# Вывод:
# htop:
#   Installed: 3.2.1-1
#   Candidate: 3.2.1-1
#   Version table:
#  *** 3.2.1-1 500
#         500 http://deb.debian.org/debian bookworm/main amd64 Packages
#         100 /var/lib/dpkg/status
```

---

## 🔧 Часть 4. Решение типичных проблем

### 1. `dpkg lock` — «Another process is using the dpkg frontend»

**Симптом:** При попытке установки пакета вы видите:
```
E: Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), is another process using it?
```

**Причина:** Другой процесс (обычно `apt` или `unattended-upgrades`) уже работает с пакетным менеджером.

**Решение:**
```bash
# 1. Проверяем, не запущен ли другой процесс установки
ps aux | grep -E 'apt|dpkg|unattended'

# 2. Если процесс есть — ждём его завершения
# 3. Если процесс завис — удаляем lock-файл (ОСТОРОЖНО!)
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo rm /var/cache/apt/archives/lock

# 4. Перенастраиваем пакеты
sudo dpkg --configure -a
sudo apt update
```

> 🚨 **ВНИМАНИЕ:** Удаляйте lock-файлы только если уверены, что никакой процесс установки не работает. Иначе можно повредить базу пакетов.

### 2. `Unable to locate package` — пакет не найден

**Симптом:**
```
E: Unable to locate package htop
```

**Причина:** Списки пакетов устарели или репозиторий не подключён.

**Решение:**
```bash
# Обновляем списки пакетов
sudo apt update

# Если не помогло — проверяем, подключены ли нужные репозитории
cat /etc/apt/sources.list
cat /etc/apt/sources.list.d/
```

### 3. `Broken packages` — сломанные зависимости

**Симптом:**
```
The following packages have unmet dependencies:
 nginx : Depends: libssl3 (>= 3.0.0) but it is not going to be installed
E: Broken packages
```

**Причина:** Конфликт версий зависимостей.

**Решение:**
```bash
# Пробуем исправить зависимости
sudo apt --fix-broken install

# Если не помогло — полное обновление
sudo apt update
sudo apt full-upgrade

# Крайний случай — переустановка проблемного пакета
sudo apt install --reinstall nginx
```

### 4. `Hash sum mismatch` — ошибка контрольной суммы

**Симптом:**
```
E: Failed to fetch http://deb.debian.org/debian/dists/bookworm/main/binary-amd64/Packages.gz  Hash Sum mismatch
E: Some index files failed to download. They have been ignored, or old ones used instead.
```

**Причина:** Кэш пакетов повреждён или репозиторий обновился во время скачивания.

**Решение:**
```bash
# Очищаем кэш
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

### 5. Как узнать, какой пакет содержит нужный файл

```bash
# Устанавливаем apt-file
sudo apt install apt-file
sudo apt-file update

# Ищем файл
apt-file search nginx.conf
# Вывод:
# nginx-common: /etc/nginx/nginx.conf
# nginx-core: /etc/nginx/nginx.conf
```

**Или без установки `apt-file`:**
```bash
dpkg -S nginx.conf
# Вывод:
# nginx-common: /etc/nginx/nginx.conf
```

---

## 🏥 Часть 6. Практические кейсы из больницы

### Кейс 1: «Нужно установить утилиту для диагностики сети»

**Симптом:** В заявке пишут, что у врача не открывается внутренний портал. Нужно проверить DNS и трассировку.

**Диагностика:**
```bash
# Проверяем, установлены ли нужные утилиты
which dig
# Вывод: (пусто) — утилита не установлена

which traceroute
# Вывод: (пусто) — утилита не установлена
```

**Решение:**
```bash
# Обновляем списки пакетов
sudo apt update

# Устанавливаем утилиты
sudo apt install dnsutils traceroute

# Проверяем
dig emh.hospital.local
# Вывод: 
# ;; ANSWER SECTION:
# emh.hospital.local.    300    IN    A    192.168.1.100

traceroute emh.hospital.local
# Вывод:
# 1  192.168.1.1 (192.168.1.1)  0.523 ms
# 2  192.168.1.100 (192.168.1.100)  1.234 ms
```

---

### Кейс 2: «Нужно обновить PostgreSQL до новой версии»

**Симптом:** Разработчики просят обновить PostgreSQL с 14 до 15 для поддержки новых функций.

**Диагностика:**
```bash
# Проверяем текущую версию
psql --version
# Вывод: psql (PostgreSQL) 14.9
```

**Решение:**
```bash
# 1. Добавляем официальный репозиторий PostgreSQL (если ещё не добавлен)
sudo apt install curl ca-certificates gnupg
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /usr/share/keyrings/postgresql-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/postgresql-keyring.gpg] http://apt.postgresql.org/pub/repos/apt bookworm-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# 2. Обновляем списки пакетов
sudo apt update

# 3. Устанавливаем PostgreSQL 15 (параллельно с 14)
sudo apt install postgresql-15

# 4. Проверяем
psql --version
# Вывод: psql (PostgreSQL) 15.4

# 5. Мигрируем данные (через pg_dump / pg_restore или через docker compose)
# Это уже отдельная задача, но пакеты установлены ✅
```

> ⚠️ **Важно:** При обновлении БД всегда делайте бэкап перед миграцией. В больнице это критично — данные пациентов нельзя терять.

---

### Кейс 3: «Пакет не устанавливается, ругается на зависимости»

**Симптом:** При установке `nginx` возникает ошибка:
```
The following packages have unmet dependencies:
 nginx : Depends: libssl3 (>= 3.0.0) but it is not going to be installed
E: Broken packages
```

**Диагностика:**
```bash
# Проверяем, какая версия libssl3 доступна
apt policy libssl3
# Вывод:
# libssl3:
#   Installed: 3.0.8-1
#   Candidate: 3.0.11-1
```

**Решение:**
```bash
# Пробуем исправить зависимости
sudo apt --fix-broken install

# Если не помогло — обновляем систему
sudo apt update
sudo apt full-upgrade

# Повторяем установку
sudo apt install nginx
```

---

### Кейс 4: «Нужно найти, какой пакет содержит `/etc/nginx/nginx.conf`»

**Симптом:** Нужно переустановить Nginx, но не понятно, какой именно пакет содержит основной конфиг.

**Решение:**
```bash
# Способ 1: через dpkg (если пакет уже установлен)
dpkg -S /etc/nginx/nginx.conf
# Вывод:
# nginx-common: /etc/nginx/nginx.conf

# Способ 2: через apt-file (если пакет не установлен)
sudo apt install apt-file
sudo apt-file update
apt-file search /etc/nginx/nginx.conf
# Вывод:
# nginx-common: /etc/nginx/nginx.conf

# Переустанавливаем нужный пакет
sudo apt install --reinstall nginx-common
```

---

### Кейс 5: «Закончилось место из-за кэша пакетов»

**Симптом:** Диск забит, `df -h` показывает 95% использования.

**Диагностика:**
```bash
sudo du -sh /var/cache/apt/archives/
# Вывод: 2.5G
```

**Решение:**
```bash
# Очищаем скачанные .deb-файлы
sudo apt clean

# Очищаем кэш списков пакетов
sudo rm -rf /var/lib/apt/lists/*

# Проверяем место
df -h
# Вывод: /dev/sda1 100G 92G 8.0G 92% / ✅
```

> 💡 **Совет:** После `apt clean` при следующей установке пакетов они будут скачаны заново. Это нормально.

---

## 📋 Шпаргалка

| Команда | Что делает |
|---------|-----------|
| `sudo apt update` | Обновить списки пакетов из репозиториев |
| `sudo apt upgrade` | Обновить установленные пакеты (безопасно) |
| `sudo apt full-upgrade` | Обновить пакеты (может удалять/ставить новые зависимости) |
| `sudo apt install <package>` | Установить пакет |
| `sudo apt remove <package>` | Удалить пакет (оставить конфиги) |
| `sudo apt purge <package>` | Удалить пакет вместе с конфигами |
| `sudo apt autoremove` | Удалить ненужные зависимости |
| `apt search <keyword>` | Поиск пакетов по ключевому слову |
| `apt show <package>` | Информация о пакете |
| `apt list --installed` | Список установленных пакетов |
| `apt policy <package>` | Версии пакета и репозитории |
| `dpkg -S <file>` | Какой пакет содержит файл |
| `apt-file search <file>` | Поиск файла в репозиториях |
| `sudo apt clean` | Очистить кэш скачанных пакетов |
| `sudo apt --fix-broken install` | Исправить сломанные зависимости |

### Решение проблем

| Проблема | Решение |
|----------|---------|
| `dpkg lock` | Проверить процессы → `sudo rm /var/lib/dpkg/lock*` → `sudo dpkg --configure -a` |
| `Unable to locate package` | `sudo apt update` |
| `Broken packages` | `sudo apt --fix-broken install` → `sudo apt full-upgrade` |
| `Hash sum mismatch` | `sudo rm -rf /var/lib/apt/lists/*` → `sudo apt update` |
| Мало места на диске | `sudo apt clean` → `sudo rm -rf /var/lib/apt/lists/*` |

---

### Золотые правила работы с APT:

1. **Всегда делайте `sudo apt update` перед установкой или обновлением.**
2. **Не прерывайте процесс установки** (`Ctrl+C` во время `apt install` может сломать базу пакетов).
3. **После удаления пакетов делайте `sudo apt autoremove`.**
4. **Добавляйте только проверенные репозитории.**
5. **Перед обновлением продакшен-серверов делайте бэкап.**


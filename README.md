# Домашнее задание к занятию «Система мониторинга Zabbix. Часть 2» - `Шумихин Кирилл`

## Цель работы

Развернуть систему мониторинга **Zabbix**, подключить к ней хосты с установленным **Zabbix Agent**, создать пользовательский шаблон и настроить сбор метрик CPU и RAM.

---

# Архитектура

В рамках работы были развернуты следующие виртуальные машины:

| Хост                  | Назначение           |
| --------------------- | -------------------- |
| zabbix-server         | сервер мониторинга   |
| zabbix-agent / kord-1 | хост для мониторинга |
| kord-2                | дополнительный хост  |

Используемые компоненты:

* **Zabbix Server 6.4**
* **PostgreSQL**
* **Zabbix Agent**
* **Debian 11**

---

# Этапы выполнения

## 1. Развертывание сервера Zabbix

Была создана виртуальная машина **zabbix-server**.

Установлены пакеты:

```
zabbix-server-pgsql
zabbix-frontend-php
zabbix-apache-conf
zabbix-agent
postgresql
```

После установки была создана база данных:

```
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```

Импортирована схема базы данных:

```
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | psql -U zabbix -d zabbix
```

---

## 2. Настройка подключения к базе данных

Файл конфигурации сервера:

```
/etc/zabbix/zabbix_server.conf
```

Были заданы параметры подключения:

```
DBHost=localhost
DBPort=5432
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```

После этого сервер был перезапущен:

```
sudo systemctl restart zabbix-server
```

---

## 3. Установка Zabbix Agent

На хостах **kord-1** и **kord-2** был установлен агент:

```
sudo apt install zabbix-agent
```

Конфигурация агента:

```
/etc/zabbix/zabbix_agentd.conf
```

Основные параметры:

```
Server=IP_ZABBIX_SERVER
ServerActive=IP_ZABBIX_SERVER
```

После настройки агент был перезапущен:

```
sudo systemctl restart zabbix-agent
```

---

## 4. Добавление хоста в Zabbix

В веб-интерфейсе Zabbix:

```
Data collection → Hosts → Create host
```

Были заданы параметры:

* Host name
* Host group: **Linux servers**
* Interface: IP адрес агента
* Template: **Linux by Zabbix agent**

После добавления хост стал доступен, что отображается зелёным индикатором **ZBX**.

---

## 5. Создание пользовательского шаблона

Создан шаблон:

```
custom-template
```

В шаблон добавлены элементы данных (Items):

### CPU usage

```
system.cpu.util[,user]
```

Тип данных:

```
Numeric (float)
```

Интервал обновления:

```
30s
```

---

### RAM usage

```
vm.memory.size[pused]
```

Тип данных:

```
Numeric (float)
```

Интервал обновления:

```
30s
```

Шаблон был применён к хостам.

---

# Проверка работы мониторинга

В разделе:

```
Monitoring → Latest data
```

появились значения метрик:

* CPU usage
* RAM usage

Также был построен график загрузки CPU.

---

# Dashboard

На главной странице системы отображается:

* статус серверов
* доступность хостов
* графики использования ресурсов
* карта размещения хостов

---

# Проблемы, возникшие при выполнении

## 1. Сервер Zabbix не подключался к базе данных

В логах возникала ошибка:

```
fe_sendauth: no password supplied
```

Причина — в конфигурации не был указан пароль базы данных.

Решение:

```
DBPassword=zabbix
```

---

## 2. Сервер не подключался к агенту

Возникала ошибка:

```
Received empty response from Zabbix Agent
```

Причина — в конфигурации агента был указан только localhost.

Решение:

```
Server=<IP Zabbix server>
ServerActive=<IP Zabbix server>
```

---

## 3. Не отображались графики

Причина:

* не было накопленных данных
* был выбран неправильный диапазон времени на Dashboard

После накопления метрик графики начали отображаться корректно.

---

# Итог

В ходе выполнения работы была успешно развернута система мониторинга **Zabbix**, настроены агенты и создан пользовательский шаблон для мониторинга ресурсов.

Были получены следующие результаты:

* сервер мониторинга успешно работает
* агенты подключены и доступны
* данные о загрузке CPU и использовании RAM собираются
* метрики отображаются в виде графиков

---

# Репозиторий

```
https://github.com/thekord54-oss/hw-02.md
```

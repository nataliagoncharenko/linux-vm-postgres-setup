# Настройка Linux-сервера с PostgreSQL в VirtualBox

## Описание 

Учебный проект:
Установка и настройка Ubuntu Server (24.04.3 LTS), подключение общей папки, настройка сети и DNS, установка PostgreSQL, проброс порта и подключение по SSH с хоста

Что сделано:

- Установлен Ubuntu Server в VirtualBox
- Подключена общая папка между Windows и VM
- Настроен NAT + проброс порта для SSH
- Установлен и запущен PostgreSQL
- Импортирована учебная база данных "Авиаперевозки" (https://postgrespro.ru/education/demodb)
- Установлен SSH 
- Выполнено подключение к серверу по SSH с хоста Windows 

## Используемые команды

Обновление сервера после установки:
sudo apt upgrade -y

1) Подключение общей папки

- в Oracle VirtualBox менеджер в настройках VM подключить папку на Windows
- на сервере необходимо установить пакет virtualbox-guest-utils (не графический) для того, чтобы VirtualBox видел общие папки и Linux их монтировал: 

sudo apt install virtualbox-guest-utils

- создание точки монтирования:

sudo mkdir /mnt/shared

- монтирование общей папки (linux_server) в точку монтирования /mnt/shared

sudo mount -t vboxsf linux_server /mnt/shared

Проверка, что папка видна в VM:
ls /mnt/shared

2) Копирование базы данных demo-small-20170815.sql в домашнюю папку на сервере 

cp /mnt/shared/demo-small-20170815.sql ~/

Проверка:
ls ~/

3) Установка СУБД PostgreSQL 

sudo apt install postgresql postgresql-contrib

postgresql - сервер СУБД
postgresql-contrib - дополнительные утилиты

Проверка статуса:
sudo systemctl status postgresql 

4) Создание базы данных и пользователя для нее 

- вход под пользователем postgres

sudo -i -u postgres

- создание базы (demo)

createdb demo

- копирование файла базы данных из домашней папки во временную от пользователя VM (vboxuser)
(т.к у пользователя postgres недостаточно прав для чтения файла базы из домашней папки):

sudo cp ~/demo-small-20170815.sql /tmp/

- импорт базы данных от пользователя postgres:

sudo -u postgres psql -d demo -f /tmp/demo-small-20170815.sql

5) Вход в базу данных demo:

sudo -u postgres psql -d demo 


6) Подключение SSH для удаленного подключения к серверу с Windows

- установка SSH:

sudo apt install openssh-server -y

Установится только если есть интернет на VM/
Если интернета нет, то необходимо настроить.

7) Настройка Интернета на виртуальной машине 

- проверка есть ли Интернет:

ping -c 3 google.com

Возникла проблема: ping не было
Если ping не идет, то необходимо исправить:

Virtual Box Менеджер -> VM -> настройки -> Сеть ->  Адаптер 1 ->  Тип подключения NAT

Если настройки выставлены, но Интернета нет, то проблема в DNS:

- Добавление Google в DNS систему:

sudo nano /etc/resolv.conf

внутри прописываем (остальное удаляем):
nameserver 8.8.8.8
nameserver 1.1.1.1
Сохранение файла: Ctrl + O -> Enter -> Ctrl + X

Проверка ping снова: ping -c 3 google.com

Если ping идет, то сохраняем постоянные настройки:

sudo nano /etc/systemd/resolved.conf

Внутри nano:

DNS=8.8.8.8 1.1.1.1
Сохранение файла: Ctrl + O -> Enter -> Ctrl + X

- Перезапуск сервиса

sudo systemctl restart systemd-resolved

Возникла ошибка sudo: unable to resolve host linux-server : Name or service not know.

Причина: 
Имя хоста не совпадало в: 
/etc/hostname
/etc/hosts

Устранение проблемы:
- открыть имя хоста
sudo nano /etc/hostname
там: linux-server

- открываем hosts

sudo nano /etc/hosts (был пустой файл, в этом была проблема)

Прописать: 
127.0.1.1 localhost
127.0.1.1 linux-server
Сохранение файла: Ctrl + O -> Enter -> Ctrl + X

- перезапуск сервиса 

sudo systemctl restart systemd-resolved

- проверка ping

ping google.com

Ping идет

8) Снова устанавливаем SSH

sudo apt install openssh-server -y

- проверка статуса:

systemctl status ssh

Проблема: Active: inactive (dead)

- включение автозапуска

sudo systemctl enable ssh

- запуск сервиса

sudo systemctl start ssh

- проверка статуса:

systemctl status ssh

9) Узнаем IP сервера и правило для подключения с Windows

- IP сервера:

ip a

- В интерфейсе NAT (enp0s3) нужна строка inet 10.0.2.15/24 - по нему с хоста не попасть на сервер

localhost 127.0.0.1/8

- для подклчения с Windows необходимо пробросить порт:

VM -> Настройки -> сеть -> адаптер 1 -> проброс портов
Правило: 
Имя:SSH
Протокол: TCP
Адрес хоста: 127.0.0.1 
Порт хоста 2222
Адрес гостя: остается пустым
Порт гостя: 22

- Подключение в Windows 

ssh vboxuser@127.0.0.1 -p 2222

- Подключаемся к базе данных:

sudo -u postgres psql -d demo



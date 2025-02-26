## Лабораторная работа №1. Виртуальный сервер

### *Студент - Гуцу Александр IA2303*

###### Дата выполнения- 20.02.2025

## *Описание задачи:*

Изучение виртуализации операционных систем и настройка виртуального HTTP-сервера на базе Debian и QEMU. Установка и настройка LAMP, PhpMyAdmin и Drupal.

## **Описание выполнения работы:**

### **Подготовка:**

	1. Скачан дистрибутив Debian для архитектуры x64 без графического интерфейса.

	2. Установлена система виртуализации QEMU.

	3. Переименован образ Debian в **debian.iso**.

### **Создание папок и файлов:**

	- Создана папка lab01.

	- В папке lab01 создана папка dvd и файл readme.md.

	- В папку dvd помещен файл debian.iso.

### **Создание образа виртуальной машины:**

`qemu-img create -f qcow2 debian.qcow2 8G`

### **Установка Debian:**

`qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G`

Параметры установки для удобства упрощенные:

- Имя компьютера: debian

- Хостовое имя: debian.localhost

- Имя пользователя: user

- Пароль пользователя: password

### **Запуск виртуальной машины:**

`qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22`

### **Установка LAMP:**

```
su
apt update -y
apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip
```

### **Скачивание и разархивирование PhpMyAdmin и Drupal:**

```
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip
wget https://ftp.drupal.org/files/projects/drupal-10.0.5.zip

mkdir /var/www
unzip phpMyAdmin-5.2.2-all-languages.zip
mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin
unzip drupal-10.0.5.zip
mv drupal-10.0.5 /var/www/drupal
```

#### **Настройка базы данных:**

```
mysql -u root
CREATE DATABASE drupal_db;
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON drupal_db.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### **Настройка виртуальных хостов Apache:**

Файл `/etc/apache2/sites-available/01-phpmyadmin.conf:`

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/phpmyadmin"
    ServerName phpmyadmin.localhost
    ServerAlias www.phpmyadmin.localhost
    ErrorLog "/var/log/apache2/phpmyadmin.localhost-error.log"
    CustomLog "/var/log/apache2/phpmyadmin.localhost-access.log" common
</VirtualHost>
```

Файл `/etc/apache2/sites-available/02-drupal.conf:`

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/drupal"
    ServerName drupal.localhost
    ServerAlias www.drupal.localhost
    ErrorLog "/var/log/apache2/drupal.localhost-error.log"
    CustomLog "/var/log/apache2/drupal.localhost-access.log" common
</VirtualHost>
```

#### **Активация виртуальных хостов и перезапуск Apache:**

```
/usr/sbin/a2ensite 01-phpmyadmin
/usr/sbin/a2ensite 02-drupal
systemctl reload apache2
```

Настройка файла /etc/hosts:

```
127.0.0.1 phpmyadmin.localhost
127.0.0.1 drupal.localhost
```

### **Тестирование:**

```
uname -a
```

Перезапуск Apache:

```
systemctl reload apache2
```

Проверка доступности сайтов по адресам:

- http://phpmyadmin.localhost:1080

- http://drupal.localhost:1080

## **Ответы на вопросы:**

1. Как скачать файл с помощью wget?

wget [URL-адрес файла]

2. Зачем создавать отдельную базу данных и пользователя для каждого сайта?

Разделение прав доступа и безопасность.

Удобство управления и резервного копирования.

Минимизация рисков от взлома одной базы.

3.  Как сменить порт для управления БД на 1234?
Изменить порт в конфигурационном файле MariaDB:

nano /etc/mysql/mariadb.conf.d/50-server.cnf

Добавить или изменить строку:

port = 1234

Перезагрузить MariaDB:

systemctl restart mariadb

4. *Какие преимущества?*

Эффективное использование ресурсов.

Изолированная среда для тестирования.

Гибкость в настройке и развертывании.

5. Почему важно настроить время и временную зону на сервере?

Корректная работа журналов и планировщиков задач.

Синхронизация с другими системами.

Упрощение диагностики и отладки.

6. **Сколько места занимает установленная ОС?**
Размер виртуального диска составляет примерно 1.5–2 ГБ в формате qcow2.

7. **Рекомендации по разбиению диска:**

- / (root) — основная файловая система.

- /home — данные пользователей.

- /var — журналы и временные файлы.

- /boot — загрузочные файлы.
Такое разделение упрощает управление и предотвращает переполнение отдельных разделов.

## **Выводы:**

В ходе работы были изучены основы виртуализации, установка и настройка Debian на виртуальной машине, развертывание LAMP-стека и настройка веб-приложений PhpMyAdmin и Drupal. Полученные навыки пригодятся при администрировании серверов и развертывании веб-сервисов.


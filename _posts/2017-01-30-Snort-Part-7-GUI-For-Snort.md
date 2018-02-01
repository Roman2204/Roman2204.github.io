---
title:  "Snort: Часть 7. GUI для Snort"
excerpt: ""
date:   2017-02-01T07:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - GUI
   - BASE
author_profile: true
---

{% include toc icon="file-text" %}

## Веб-интерфейс BASE

BASE (Basic Analysis and Security Engine) - веб-интерфейс для анализа и визуализации событий, обнаруженных с помощью СОВ Snort. Его движок основан на проекте ACID (Analysis Console for Intrusion Databases).

### Установка

Установим пререквизиты для BASE:

```bash
sudo apt-get install -y apache2 libapache2-mod-php5 php5-mysql php5-cli php5 php5-common php5-gd php-pear php5-xmlrpc
```

и далее установим Graph с помощью Pear:

```bash
sudo pear install -f --alldeps Image_Graph
```

Скачиваем и устанавливаем ADOdb:

```bash
cd ~/snort_src
wget https://sourceforge.net/projects/adodb/files/adodb-php5-only/adodb-520-for-php5/adodb-5.20.9.tar.gz
tar -xvzf adodb-5.20.9.tar.gz
rm adodb-5.20.9.tar.gz
sudo mv adodb5 /var/adodb
sudo chmod -R 755 /var/adodb
```

Скачиваем и устанавливаем BASE:

```bash
cd ~/snort_src
wget http://sourceforge.net/projects/secureideas/files/BASE/base-1.4.5/base-1.4.5.tar.gz
tar xzvf base-1.4.5.tar.gz
sudo mv base-1.4.5 /var/www/html/base/
```

### Конфигурация

Создаём конфигурационный файл BASE:

```bash
sudo cp /var/www/html/base/base_conf.php.dist /var/www/html/base/base_conf.php
```

И редактируем в нём `/var/www/html/base/base_conf.php` следующие строки:

```
$BASE_urlpath = '/base';                   # 50 строка
$DBlib_path = '/var/adodb/';               # 80 строка
$alert_dbname   = 'snort';                 # 102 строка
$alert_host     = 'localhost';
$alert_port     = '';
$alert_user     = 'snort';
$alert_password = 'MYSQLSNORTPASSWORD';    # 106 строка

// $graph_font_name = "Verdana";           # 456 строка
// $graph_font_name = "DejaVuSans";
// $graph_font_name = "Image_Graph_Font";
$graph_font_name = "";
```

Устанавливаем права доступа к каталогу BASE, и так как пароль хранится в файле `base_conf.php` в открытом виде мы должны запретить другим пользователям читать файл:

```bash
sudo chown -R www-data:www-data /var/www/html/base
sudo chmod o-r /var/www/html/base/base_conf.php
```

И перезагружаем Apache:

```bash
sudo service apache2 restart
```

Осталось только настроить BASE в веб-интерфейсе. Для этого нужно открыть в веб-браузере http://localhost/base/base_main.php, нажать на кнопку установить и после на [Create BASE AG](http://localhost/base/base_db_setup.php). Теперь можно вернуться на главную страницу, где должна быть информация по событиям Snort:

![BASE](http://i.imgur.com/wMRJsNM.png)


Веб-интрфейс для Snort полностью готов!

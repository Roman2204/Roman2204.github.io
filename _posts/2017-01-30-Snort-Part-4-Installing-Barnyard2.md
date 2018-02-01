---
title:  "Snort: Часть 4. Установка Barnyard2"
excerpt: "Barnyard2 является интерпретатором с открытым исходным кодом для двоичных файлов unified2"
date:   2017-02-01T04:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - Barnyard2
author_profile: true
---

{% include toc icon="file-text" %}


## Установка Barnyard2

[Barnyard2](https://github.com/firnsy/barnyard2/) является интерпретатором с открытым исходным кодом для двоичных файлов unified2, создаваемых Snort. Основная задача Barnyard2 - это позволить Snort писать выходные файлы на диск эффективным способом, оставляя задачу парсинга двоичных данных в различные форматы другому процессу, что с большей вероятностью не позволит Snort пропустить сетевой трафик.


### Установка пререквизитов

Сперва установим необходимые пререквизиты для Barnyard2:

```bash
sudo apt-get install -y mysql-server libmysqlclient-dev mysql-client autoconf libtool
```

Во время установки MySQL будет предложено ввести пароль для root-пользователя. Для примера вводим **MYSQLROOTPASSWORD**.


### Сборка и установка Barnyard2

Далее установим и сконфигурируем Barnyard2. Для этого скачаем исходный код Barnyard2 и сконфигурируем его:

```bash
cd ~/snort_src
git clone https://github.com/firnsy/barnyard2.git
cd barnyard2/
autoreconf -fvi -I ./m4
```

Для Barnyard2 необходима библиотека `dnet.h`, которую мы установили ранее вместе с пакетом `libdumbnet-dev`. Однако Barnyard2 ожидает другое имя для библиотеки, поэтому нам нужно создать символьную ссылку `/usr/include/dnet.h` на библиотеку `/usr/include/dumbnet.h`

```bash
sudo ln -s /usr/include/dumbnet.h /usr/include/dnet.h
sudo ldconfig
```


В зависимости от архитектуры системы (32 или 64 разрядная) нужно выбрать одну из следующих команд для конфигурации Barnyard2, указав путь к библиотеке MySQL. Разрядность ОС можно узнать, например, с помощью команды `uname -m` или `arch`. Вывод `x86_64` соответствует 64-разрядной система, а `i686` или `i386` - 32-х.

* Для 64-разрядной:

  ```bash
  ./configure --with-mysql --with-mysql-libraries=/usr/lib/x86_64-linux-gnu
  ```

* Для 32-разрядной:

  ```bash
  ./configure --with-mysql --with-mysql-libraries=/usr/lib/i386-linux-gnu
  ```

После чего продолжить установку:

```bash
make
sudo checkinstall      # Во время установки будет предупреждение вида Warning: debian policy compliant one. Please specify an alternate one, тогда можно ввести данный номер версии: 2.1.14
sudo dpkg -i barnyard2_2.1.14-1_amd64.deb    # Не нужно после checkinstall, так как пакет уже установлен
```

Barnyard2, по идее, установлен по следующему пути `/usr/local/bin/barnyard2`. И теперь можно проверить, так ли это на самом деле с помощью команды:

```bash
/usr/local/bin/barnyard2 -V
```

Должен быть примерно следующий вывод на терминал:

```
______   -*> Barnyard2 <*-
/ ,,_  \  Version 2.1.14 (Build 337)
|o"  )~|  By Ian Firns (SecurixLive): http://www.securixlive.com/
+ '''' +  (C) Copyright 2008-2013 Ian Firns <firnsy@securixlive.com>
```

### Конфигурирование Snort для работы с Barnyard2

Открываем конфигурационный файл `snort.conf` (например, `sudo gedit /etc/snort/snort.conf`) и добавляем строку с настройками, в которой указываем Snort писать выходные события в двоичный файл, так что Barnyard2 сможет читать их. Для этого вместо строки или сразу после неё:

```
# output unified2: filename merged.log, limit 128, nostamp, mpls_event_types, vlan_event_types
```

вставляем следующую строку:

```
output unified2: filename snort.u2, limit 128
```

Данная строка указывает Snort писать данные в двоичный формат **unified2**, что для Snort гораздо быстрее, чем писать в человекочитаемый формат.

Для того чтобы сконфигурировать Snort для использования Barnyard2 нам необходимо скопировать некоторые файлы из каталога с исходным кодом:

```bash
sudo cp ~/snort_src/barnyard2/etc/barnyard2.conf /etc/snort

sudo mkdir /var/log/barnyard2
sudo chown snort.snort /var/log/barnyard2

sudo touch /var/log/snort/barnyard2.waldo
sudo chown snort.snort /var/log/snort/barnyard2.waldo
sudo touch /etc/snort/sid-msg.map
```

### Конфигурация БД MySQL

Так как Barnyard2 сохраняет предупреждения в БД MySQL, нам необходимо создать эту БД и пользователя snort для доступа к БД. Подключаемся к БД MySQL:

```
mysql -u root -p
```

Вводим пароль **MYSQLROOTPASSWORD** и выполняем на сервере MySQL следующие команды для создания БД и MySQL пользователя:

```sql
mysql> create database snort;
mysql> use snort;
mysql> source ~/snort_src/barnyard2/schemas/create_mysql
mysql> CREATE USER 'snort'@'localhost' IDENTIFIED BY 'MYSQLSNORTPASSWORD';
mysql> grant CREATE, INSERT, SELECT, DELETE, UPDATE on snort.* to 'snort'@'localhost';
mysql> show tables;
mysql> exit
```

**MYSQLSNORTPASSWORD** - пароль для пользователя snort. После sql-команды `show tables;` мы должны были получить следующий вывод:

```
mysql> show tables;
+------------------+
| Tables_in_snort  |
+------------------+
| data             |
| detail           |
| encoding         |
| event            |
| icmphdr          |
| iphdr            |
| opt              |
| reference        |
| reference_system |
| schema           |
| sensor           |
| sig_class        |
| sig_reference    |
| signature        |
| tcphdr           |
| udphdr           |
+------------------+
16 rows in set (0.00 sec)
```

После того как БД Snort была создана необходимо сообщить Barnyard2 о её деталях, для этого открываем конфигурационный файл Barnyard2 `/etc/snort/barnyard2.conf` (например, `sudo gedit /etc/snort/barnyard2.conf`). И добавляем строку с настройками в конце файла в раздел `# database: log to a variety of databases`:

```
output database: log, mysql, user=snort password=MYSQLSNORTPASSWORD dbname=snort host=localhost sensor name=sensor01
```

>output database: log, mysql, user=snort password=MYSQLSNORTPASSWORD dbname=snort host=localhost

Так как пароль хранится в открытом виде в `barnyard2.conf`, то следует запретить другим пользователям читать данный файл:

```bash
sudo chmod o-r /etc/snort/barnyard2.conf
```

## Проверка работоспособности Barnyard2

Теперь Barnyard2 сконфигурирован для работы со Snort. Чтобы протестировать его работу нам нужно запустить Snort и Barnyard2 и сгенерировать какие-нибудь предупреждения. Запускаем Snort в фоновом режиме (как демон), используя те же параметры, что и раньше, только добавив флаг **-D**, который отвечает за запуск Snort в фоновом режиме, и удалив флаг **-A Console**, так как выводить предупреждения на экран терминала теперь не нужно:

```bash
sudo snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0 -D
```

На экране должен быть примерно следующий вывод, за исключением идентификатора процесса (PID):

```
Spawning daemon child...
My daemon child 3160 lives...
Daemon parent exiting (0)
```

Запускаем Barnyard2 для отслеживания двоичных файлов Snort с предупреждениями и загрузки их в экземпляр БД Snort. Запускаем Barnyard2 со следующими ключами:

```bash
sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo -g snort -u snort
```

* **-c /etc/snort/barnyard2.conf** - the Barnyard2 configuration file
* **-d /var/log/snort** - the location to look for the snort binary output file
* **-f snort.u2** - the name of the file to look for.
* **-w /var/log/snort/barnyard2.waldo** - the path to the waldo file (checkpoint file).
* **-u snort** - run Barnyard2 as the following user after startup
* **-g snort** - run Barnyard2 as the following group after startup

Теперь пропингуем IP-адрес интерфейса, указанного выше (eth0). Это можно сделать с помощью команды `ping 192.168.0.104`. После чего в директории Snort для логов должен появиться файл с именем вида **snort.u2.nnnnnnnnnn**, где вместо nnnnnnnnnn будет номер. Это и будет двоичный файл с предупреждениями, который Snort записывает для обработки Barnyard2.

>В случае, если пинговать из системы, где установлен Snort предупреждения не появляются (!?).

Должен получится примерно следующий вывод:

```
--== Initialization Complete ==--

______   -*> Barnyard2 <*-
/ ,,_  \  Version 2.1.14 (Build 337)
|o"  )~|  By Ian Firns (SecurixLive): http://www.securixlive.com/
+ '''' +  (C) Copyright 2008-2013 Ian Firns <firnsy@securixlive.com>

Using waldo file '/var/log/snort/barnyard2.waldo':
spool directory = /var/log/snort
spool filebase  = snort.u2
time_stamp      = 1486792881
record_idx      = 26
Opened spool file '/var/log/snort/snort.u2.1486792881'
Waiting for new data
02/11-09:27:44.512510  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/11-09:27:45.520568  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/11-09:27:46.527972  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/11-09:27:47.535450  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/11-09:27:54.152268  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.103 -> 192.168.0.104
02/11-09:27:55.141316  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.103 -> 192.168.0.104
02/11-09:27:56.155647  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.103 -> 192.168.0.104
02/11-09:27:57.170367  [**] [1:10000001:1] Snort Alert [1:10000001:1] [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.103 -> 192.168.0.104
```

Отключить Barnyard2 в терминале можно с помощью комбинации клавиш **Ctrl+C**. А демона Snort можно закрыть с помощью команды `kill`, указав PID процесса Snort. PID можно узнать с помощью команды `ps aux`:

```bash
roman@snort-vm:~$ ps aux | grep snort
avahi      481  0.0  0.0  32228  2980 ?        Ss   08:39   0:00 avahi-daemon: running [snort-vm.local]
snort     3160  0.0  2.0 457080 82192 ?        Ssl  09:01   0:00 snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth1 -D
roman@snort-vm:~$ sudo kill 3160
```

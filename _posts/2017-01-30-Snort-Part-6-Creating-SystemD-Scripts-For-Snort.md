---
title:  "Snort: Часть 6. SystemD скрипт для автозапуска Snort"
excerpt: "Создадим SystemD скрипт для демонов Snort и Barnyard2"
date:   2017-02-01T06:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - SystemD
author_profile: true
---

{% include toc icon="file-text" %}


## SystemD скрипт для Snort

Чтобы создать SystemD скрипт для Snort создаём служебный файл `/lib/systemd/system/snort.service` и записывает туда:

```
[Unit]
Description=Snort NIDS Daemon
After=syslog.target network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0

[Install]
WantedBy=multi-user.target
```

Далее говорим SystemD, что данная служба должна запускаться при загрузке системы:  

```bash
sudo systemctl enable snort
```

Запускаем службу Snort:

```bash
sudo systemctl start snort
```

И проверяем, что служба запущена:

```bash
roman@snort-vm:~$ systemctl status snort
● snort.service - Snort NIDS Daemon
   Loaded: loaded (/lib/systemd/system/snort.service; enabled)
   Active: active (running) since Wed 2017-02-15 01:11:57 MSK; 30s ago
 Main PID: 3352 (snort)
   CGroup: /system.slice/snort.service
           └─3352 /usr/local/bin/snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
```


## SystemD скрипт для Barnyard2

Далее создаём аналогичную SystemD службу для Barnyard2. Создаём следующий файл `/lib/systemd/system/barnyard2.service` и записываем туда:

```
[Unit]
Description=Barnyard2 Daemon
After=syslog.target network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -q -w /var/log/snort/barnyard2.waldo -g snort -u snort -D -a /var/log/snort/archived_logs

[Install]
WantedBy=multi-user.target
```

Флаг **-D** для запуска в качестве демона, а **-a /var/log/snort/archived_logs** для перемещения логов Barnyard2 в папку **/var/log/snort/archived_logs**.


```bash
sudo systemctl enable barnyard2
```

Запускаем службу Snort:

```bash
sudo systemctl start barnyard2
```

И проверяем, что служба запущена:

```bash
roman@snort-vm:~$ systemctl status barnyard2
● barnyard2.service - Barnyard2 Daemon
   Loaded: loaded (/lib/systemd/system/barnyard2.service; enabled)
   Active: active (running) since Wed 2017-02-15 01:32:31 MSK; 57s ago
 Main PID: 3534 (barnyard2)
   CGroup: /system.slice/barnyard2.service
           └─3534 /usr/local/bin/barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -q -w /var/log/snort/barnyard2.waldo -g snort -u snort -D -a /var/log/snort/archived_logs
```


Перезагружаем систему и убеждаемся, что обе службу запущены:

```bash
roman@snort-vm:~$ sudo service snort status
● snort.service - Snort NIDS Daemon
   Loaded: loaded (/lib/systemd/system/snort.service; enabled)
   Active: active (running) since Wed 2017-02-15 01:35:05 MSK; 50s ago
 Main PID: 478 (snort)
   CGroup: /system.slice/snort.service
           └─478 /usr/local/bin/snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0

Feb 15 01:35:05 snort-vm systemd[1]: Starting Snort NIDS Daemon...
Feb 15 01:35:05 snort-vm systemd[1]: Started Snort NIDS Daemon.
roman@snort-vm:~$ sudo service barnyard2 status
● barnyard2.service - Barnyard2 Daemon
   Loaded: loaded (/lib/systemd/system/barnyard2.service; enabled)
   Active: active (running) since Wed 2017-02-15 01:35:05 MSK; 1min 3s ago
 Main PID: 481 (barnyard2)
   CGroup: /system.slice/barnyard2.service
           └─481 /usr/local/bin/barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -q -w /var/log/snort/barnyard2.waldo -g snort -u snort -D -a /var/log/snort/archived_logs

Feb 15 01:35:05 snort-vm barnyard2[481]: ----------------------------
                                         +[ Signature Suppress list ]+
Feb 15 01:35:17 snort-vm barnyard2[481]: WARNING: invalid Reference spec '2015-0666'. Ignored
Feb 15 01:35:18 snort-vm barnyard2[481]: Barnyard2 spooler: Event cache size set to [2048]
Feb 15 01:35:18 snort-vm barnyard2[481]: Log directory = /var/log/barnyard2
Feb 15 01:35:18 snort-vm barnyard2[481]: INFO database: Defaulting Reconnect/Transaction Error limit to 10
Feb 15 01:35:18 snort-vm barnyard2[481]: INFO database: Defaulting Reconnect sleep time to 5 second
Feb 15 01:35:18 snort-vm barnyard2[481]: Initializing daemon mode
Feb 15 01:35:18 snort-vm barnyard2[481]: Daemon initialized, signaled parent pid: 1
Feb 15 01:35:18 snort-vm barnyard2[481]: PID path stat checked out ok, PID path set to /var/run/
Feb 15 01:35:18 snort-vm barnyard2[481]: Writing PID "481" to file "/var/run//barnyard2_NULL.pid"
```

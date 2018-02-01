---
title:  "Snort: Часть 2. Установка Snort"
excerpt: "В данной статье рассмотрим пример установки Snort для ОС Debian. А также рассмотрим конфигурацию сетевого интерфейса, рекомендованой для Snort."
date:   2017-02-01T02:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - Installation
author_profile: true
---

{% include toc icon="file-text" %}


В данной статье рассмотрим пример установки и первичной настройки Snort для ОС Debian. А также рассмотрим конфигурацию сетевого интерфейса, рекомендуой для Snort.


## Установка и первичная настройка Snort для ОС Debian

### Установка необходимых пререквизитов для Snort

В данной статье разберём подробно установку из исходных кодов и первичную настройку Snort в ОС Debian.

Для начала устанавливаем обновления для ОС:

```bash
sudo apt-get update && sudo apt-get upgrade
```

* **apt-get update** - обновление индекса пакетов
* **apt-get upgrade** - обновление пакетов


Устанавливаем обязательные пререквизиты для Snort:

```bash
sudo apt-get -y install build-essential libpcap-dev libpcre3-dev libdumbnet-dev zlib1g-dev
```


* **build-essential** - предоставляет инструменты, необходимых для сборки пакетов Debian;
* **libpcap-dev** - библиотека для захвата сетевого трафика, необходима для Snort;
* **libpcre3-dev** - библиотека функций для поддержки регулярных выражений, необходима для Snort;
* **libdumbnet-dev** - библиотека, ещё известная как libdnet-dev, предоставляющая упрощённый, портативный интерфейс для различных низкоуровневых (interface to several low-level networking routines);
* **zlib1g-dev** - библиотека, реализующая метод сжатия deflate, необходима для Snort.

Устанавливаем необходимые для DAQ парсеры:

```bash
sudo apt-get -y install bison flex
```

* **bison** - генератор анализаторов синтаксиса (parser) выражений;
* **flex** - инструмент для генерации программ, распознающих заданные образцы в тексте.

Устанавливаем дополнительные (рекомендуемые) программы:

```bash
sudo apt-get -y install liblzma-dev openssl libssl-dev iptables-dev libnfnetlink-dev libmnl-dev libnet1-dev libnetfilter-queue-dev pkg-config autotools-dev checkinstall cmake cpputest
```

Для сборки документации (не является обязательным) понадобиться установить следующие пакеты:

```bash
sudo apt-get -y install w3m dblatex asciidoc source-highlight
```

Для поддержки HTTP/2 траффика необходимо установить библиотеку [Nghttp2](https://nghttp2.org): HTTP/2 C библиотека, которая реализует алгоритм сжатия заголовков HPAC.

Устанавливаем её из исходного кода:

```bash
sudo apt-get install -y autoconf libtool pkg-config
cd ~/snort_src
wget https://github.com/nghttp2/nghttp2/releases/download/v1.19.0/nghttp2-1.19.0.tar.gz
tar -xzvf nghttp2-1.19.0.tar.gz
rm nghttp2-1.19.0.tar.gz
cd nghttp2-1.19.0
autoreconf -i --force
automake
autoconf
./configure --enable-lib-only
make
sudo make install
```


### Сборка DAQ и Snort из исходного кода

Создадим каталог snort_src, куда сохраним загружаемые архивы с программами: `mkdir ~/snort_src` Snort использует библиотеку Data Acquisition library (DAQ), которую можно скачать с официального сайта Snort. Скачиваем архив с исходным кодом, создаём бинарный пакет с помощью checkinstall и устанавливаем его:

```bash
cd ~/snort_src/
wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz -O daq-2.0.6.tar.gz
tar -xvzf daq-2.0.6.tar.gz
rm daq-2.0.6.tar.gz
cd daq-2.0.6
./configure
make
sudo checkinstall
sudo dpkg -i daq_2.0.6-1_amd64.deb
```


Обновляем пути для разделяемых библиотек:

```bash
sudo sh -c "echo >> /etc/ld.so.conf /usr/lib"
sudo sh -c "echo >> /etc/ld.so.conf /usr/local/lib"
sudo ldconfig
```

Далее устанавливаем Snort из исходного кода:

```bash
cd ~/snort_src/
wget https://www.snort.org/downloads/snort/snort-2.9.9.0.tar.gz -O snort-2.9.9.0.tar.gz
tar -xvzf snort-2.9.9.0.tar.gz
rm snort-2.9.9.0.tar.gz
cd snort-2.9.9.0
./configure --enable-sourcefire
make
sudo checkinstall
sudo dpkg -i snort_2.9.9.0-1_amd64.deb
```

В случае каких-либо ошибок во время сборки необходимо их устранить прежде чем двигаться дальше.

И создаём символьную ссылку

```bash
sudo ln -s /usr/local/bin/snort /usr/sbin/snort
sudo ldconfig
```

Следует проверить, правильно ли установился Snort. Для этого достаточно запустить команду `snort -V`. Если вывод команды будет примерно следующий (номер версии может отличаться), то это будет означать, что [Snort](https://www.snort.org/downloads) успешно установлен:

```
   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.9.0 GRE (Build 56)
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
           Copyright (C) 2014-2016 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using libpcap version 1.6.2
           Using PCRE version: 8.35 2014-04-04
           Using ZLIB version: 1.2.8
```

Также с помощью команды `snort --daq-list` можно посмотреть список доступным DAQ-модулей.




#### Устранение возможных ошибок

Во время выполнения команды checkinstall могут возникнуть ошибки вида:

> ranlib: could not create temporary file whilst writing archive: No more archived files

, которые можно исправить, одним из следующих способов:

* Установкой и удалением программы при помощи make:

   ```bash
   make install
   make uninstall
   ```

* Добавлением вручную каталогов, необходимых для сборки и установки пакета, которые не может создать **checkinstall** самостоятельно:

   ```bash
   sudo mkdir /usr/local/lib/snort_dynamicengine
   sudo mkdir /usr/local/include/snort
   sudo mkdir /usr/local/lib/snort
   sudo mkdir /usr/local/include/snort/dynamic_preproc
   sudo mkdir /usr/local/lib/snort/dynamic_preproc
   sudo mkdir /usr/local/lib/snort_dynamicpreprocessor
   sudo mkdir /usr/local/lib/snort/dynamic_output
   sudo mkdir /usr/local/share/doc
   ```

   После чего вновь собираем и устанавливаем Snort:

   ```bash
   sudo checkinstall
   sudo dpkg -i snort_2.9.8.0-1_amd64.deb
   ```


### Конфигурация сетевого интерфейса, рекомендуемая для Snort

Согласно рекомендациям из [руководства Snort](http://manual.snort.org/node7.html) следует убедиться, что сетевая карта не обрезает слишком большие пакеты (в сетях Ethernet длина которых превышает 1518 байт). Для этого произведём оптимизацию сетевых интерфейсов с помощью утилиты ethtool. Если нужно, устанавливаем **ethtool** (`sudo apt-get -y install ethtool`) и для настройки параметров выполняет в терминале следующие команды:

* `ethtool -K eth0 rx off` - освобождение ОС от расчёт контрольных суммы TCP/IP (rx-checksumming) для входящих пакетов;
* `ethtool -K eth0 tx off` - освобождение ОС от расчёт контрольных суммы TCP/IP (tx-checksumming) для исходящих пакетов;
* `ethtool -K eth0 gso off` - отключение функции (generic-segmentation-offload) фрагментации пакетов без участия CPU;
* `ethtool -K eth0 gro off` - отключение функции (generic-receive-offload) сборки пакетов сетевым интерфейсом без участия CPU;
* `ethtool -K eth0 lro off` - отключение функции (large-receive-offload) сборки пакетов сетевым интерфейсом без участия CPU.


Но в данном случае настройки будут оставаться в силе только до перезагрузки ОС, поэтому лучшим вариантом будет добавить команды настройки сетевого интерфейса одним из следующих способов:

* Открываем файл **/etc/network/interfaces** (`sudo gedit /etc/network/interfaces`) с настройками конфигурации сети Ethernet и добавляем в конец файла следующие команды для сетевого интерфейса eth0, который Snort будет слушать:

  ```
  # The primary network interface
  auto eth0
  iface eth0 inet dhcp
  post-up ethtool --offload eth0 rx off tx off
  post-up ethtool -K eth0 gso off
  post-up ethtool -K eth0 gro off
  post-up ethtool -K eth0 lro off
  ```

* Либо открываем файл **/etc/rc.local** (`sudo gedit /etc/rc.local`) с автозагрузками и добавляем следующие строчки до "exit 0":

  ```
  ethtool --offload eth0 rx off tx off
  ethtool -K eth0 gso off
  ethtool -K eth0 gro off
  ethtool -K eth0 lro off
  ```


Текущие значения параметров можно посмотреть с помощью команды `ethtool -k eth0`. Если параметр обозначен как [fixed], то это означает, что значение параметра нельзя изменить с помощью ethtool. Часто многие параметры обозначены [fixed], когда ОС запушена в виртуальной машине (например, VMWare или VirtualBox) и гостевая ОС не в состоянии изменить параметры хостовой ОС.


Конфигурация ниже не рабочая:

```bash
# 1
allow-hotplug eth0
auto eth0
iface eth0 inet dhcp

# 2
allow-hotplug eth1
auto eth1
iface eth1 inet static
    address 192.168.0.22
    netmask 255.255.255.0
    gateway 192.168.0.1
```


## Установка Snort для для других ОС

На ОС Raspbian (Raspberry Pi 3) Snort устанавливается с помощью такой же последовательности команд.

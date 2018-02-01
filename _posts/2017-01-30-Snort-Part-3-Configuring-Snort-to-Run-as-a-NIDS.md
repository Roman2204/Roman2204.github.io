---
title:  "Snort: Часть 3. Настройка Snort для работы в качестве сетевой системы обнаружения вторжений (NIDS)"
excerpt: "В данной статье разберём конфигурацию Snort в качестве сетевой системы обнаружения вторжений (NIDS). Создадим файлы и папки, которые необходимы для работы Snort, и посмотрим главный конфигурационный файл **snort.conf**."
date:   2017-02-01T03:00:00+03:00
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


В предыдущей статье [Snort: Установка Snort]({{ site.baseurl }}{% post_url 2017-01-30-Snort-Part-2-Installing-Snort %}) мы установили Snort и проверили его работу. Сейчас мы разберём конфигурацию Snort в качестве сетевой системы обнаружения вторжений (NIDS). Создадим файлы и папки, которые необходимы для работы Snort, и посмотрим главный конфигурационный файл **snort.conf**.



## Базовая конфигурация

По соображениям безопасности, чтобы не запускать Snort из-под суперпользователя, создадим непривилегированного системного пользователя и группу:

```bash
# Создаём группу и пользователя для Snort:
sudo groupadd snort
sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
```

Далее создаём ряд файлов и каталогов, необходимых для работы Snort, и устанавливаем права доступа для этих файлов:

```bash
# Создаём необходимые для Snort каталоги:
sudo mkdir /etc/snort
sudo mkdir /etc/snort/rules
sudo mkdir /etc/snort/rules/iplists
sudo mkdir /etc/snort/preproc_rules
sudo mkdir /usr/local/lib/snort_dynamicrules
sudo mkdir /etc/snort/so_rules

# Создаём файлы, в которых будут храниться правила и списки IP:
sudo touch /etc/snort/rules/iplists/black_list.rules
sudo touch /etc/snort/rules/iplists/white_list.rules
sudo touch /etc/snort/rules/local.rules
sudo touch /etc/snort/sid-msg.map

# Создаём каталоги, где будут храниться лог-файлы:
sudo mkdir /var/log/snort
sudo mkdir /var/log/snort/archived_logs

# Изменяем права доступа:
sudo chmod -R 5775 /etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /var/log/snort/archived_logs
sudo chmod -R 5775 /etc/snort/so_rules
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules

# Изменяем владельца для каталогов:
sudo chown -R snort:snort /etc/snort
sudo chown -R snort:snort /var/log/snort
sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
```

После чего копируем следующие конфигурационные файлы из извлечённого архива Snort в каталог /etc/snort:

* **classification.config** описывает типы категории атак, которые предустановленны в Snort.
* **file_magic.conf** описываем файловые сигнатуры («магические числа») для определения типа файла.
* **reference.config** содержит ссылки на системы идентификации атак.
* **snort.conf** конфигурационный файл Snort, в котором хранятся переменные с настройками и путями к различными ресурсам.
* **threshold.conf** позволяет настроить количество событий, необходимых для генерации оповещений (алертов), что может быть полезно в случае «шумных» правил.
* **attribute_table.dtd** позволяет Snort использовать внешнюю информацию для определения протоколов и политик.
* **gen-msg.map** указывает Snort какой препроцессор используется каким правилом.
* **unicode.map** предоставляет соответствие между кодировками.


Для этого выполняем следующие команды:

```bash
cd ~/snort_src/snort-2.9.9.0/etc/
sudo cp *.conf* /etc/snort
sudo cp *.map /etc/snort
sudo cp *.dtd /etc/snort

cd ~/snort_src/snort-2.9.9.0/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor/
sudo cp * /usr/local/lib/snort_dynamicpreprocessor/
```

Структура каталогов и файлов Snort должна выглядеть следующим образом, для её вывода на экран следует выполнить команду `tree /etc/snort` (если утилита tree не установлена, то её можно установить следующим образом `sudo apt-get install tree`):

```bash
/etc/snort
├── attribute_table.dtd
├── classification.config
├── file_magic.conf
├── gen-msg.map
├── preproc_rules
├── reference.config
├── rules
│   ├── iplists
│   │   ├── black_list.rules
│   │   └── white_list.rules
│   └── local.rules
├── sid-msg.map
├── snort.conf
├── so_rules
├── threshold.conf
└── unicode.map

4 directories, 12 files
```


## Изменение конфигурационных файлов Snort

Конфигурационный файл Snort `/etc/snort/snort.conf` содержит все настройки, которые использует Snort в режиме NIDS. Файл для удобства разделён на следующие части:

1. Установка сетевых значений и переменных (Set the network variables)
2. Конфигурация декодеров (Configure the decoder)
3. Конфигурация базового (основного) механизма обнаружения (вторжений) (Configure the base detection engine)
4. Конфигурация динамически загружаемых библиотек (Configure dynamic loaded libraries)
5. Конфигурация препроцессоров (Configure preprocessors)
6. Конфигурация плагинов (Configure output plugins)
7. Настройка набора правил (Customize your rule set)
8. Настройка препроцессоров и декодеров набора правил (Customize preprocessor and decoder rule set)
9. Customize shared object rule set


Три типа переменных, которые могут быть определены в СОВ Snort:

* **var** - используется для присвоения переменной пути к файлу или директории и для назначения переменной ip-адресов.
* **ipvar** - применяется к переменным также для указания ip-адресов, но только с поддержкой IPv6.
* **portvar** - используется для задания переменных с номерами портов.

Синтаксис определения переменных:

```
<var | portvar | ipvar> <название_переменной> <значение_переменной>
```

С помощью ключевого слова include подключаются дополнительные файлы.

Открываем файл (например, `sudo gedit /etc/snort/snort.conf`) и настраиваем следующие параметры:

* Настраиваем Set the network variables

  - **Задание внутренней сети**

    Опция `ipvar HOME_NET any` задаёт диапазон внутренних IP-адресов (хост или список хостов) для домашний сети, которые мы собираемся защищать и трафик которых Snort будет анализировать. Значение переменной $HOME_NET соответствует стандарту [RFC 1918](http://tools.ietf.org/html/rfc1918) (адресное пространство). Имеется возможность указания сетевого интерфейса: `$<ИМЯ_СЕТЕВОГО_ИНТЕРФЕЙСА>_ADDRESS` (например, `ipvar HOME_NET $eth0_ADDRESS`). Значение переменной будет равно IP адресу. Чаще всего этот параметр используется на компьютерах, которые получают IP адрес динамически. Можно задать маску и указать несколько IP-адресов через запятую (пробелы не допускаются): `ipvar HOME_NET [10.1.1.0/24,192.168.1.0/24]`. По умолчанию установлено значение **any**, что может привести к большому количеству ошибок первого рода (false positives alerts). В нашем случае установим IP-адрес машины, на которой установлен Snort:

    ```
    44 # Setup the network addresses you are protecting
    45 ipvar HOME_NET 192.168.0.104   # Необходимо указать свой IP адрес
    ```

  - **Задание внешней сети**

    Опция `ipvar EXTERNAL_NET any` задаёт диапазон внешних IP-адресов сетей, из которых исходит угроза. Можно использовать логическое отрицание с помощью символа "!", например `ipvar EXTERNAL_NET !$HOME_NET` исключает из переменной $EXTERNAL_NET сеть $HOME_NET. Данное значение оставим по умолчанию any, чтобы все адреса, отличные от $HOME_NET, воспринимались внешними.

    ```
    47 # Set up the external network addresses. Leave as "any" in most situations
    48 ipvar EXTERNAL_NET any
    ```

    Настраиваем пути к каталогам, которые создали ранее. Для этого в следующих строках

    ```
    104 var RULE_PATH ../rules
    105 var SO_RULE_PATH ../so_rules
    106 var PREPROC_RULE_PATH ../preproc_rules
    ```

    ```
    113 var WHITE_LIST_PATH ../rules
    114 var BLACK_LIST_PATH ../rules
    ```

    изменяем пути, как показано ниже:

    ```
    104 var RULE_PATH /etc/snort/rules
    105 var SO_RULE_PATH /etc/snort/so_rules
    106 var PREPROC_RULE_PATH /etc/snort/preproc_rules
    ```

    ```
    113 var WHITE_LIST_PATH /etc/snort/rules/iplists
    114 var BLACK_LIST_PATH /etc/snort/rules/iplists
    ```

    В данной части имеются настройки переменных IP-адресов для серверов DNS, SMTP, HTTP, SQL, TELNET, SSH, FTP, SIP и портов, используемых серверами для конкретных приложений (HTTP, SHELLCODE, ORACLE, SSH, FTP, SIP и другие). Указав данные параметры для конкретной сети можно более тонко настроить Snort, но в нашем случае мы оставим данные параметры по умолчанию.

* Настраиваем **Customize your rule set**

  В данном файле содержится список включаемых файлов вида `include $RULE_PATH/НАЗВАНИЕ_НАБОРА_ПРАВИЛ.rules`, которые Snort использует по умолчанию для импорта правил. Все эти строки, за исключением одной - `include $RULE_PATH/local.rules`, указывающую путь к файлу, где будут храниться наши правила, необходимо закомментировать, так как мы будем использовать менеджер правил для Snort - PulledPork, которых хранит все правила в одном файле. Закомментировать строчки можно при помощи команд:

  ```
  # Закомментируем все строки вида include $RULE_PATH
  sudo sed -i 's|include \$RULE_PATH|#include \$RULE_PATH|' /etc/snort/snort.conf
  # Раскомментируем строку include $RULE_PATH/local.rules
  sudo sed -i 's|#include \$RULE_PATH/local\.rules|include \$RULE_PATH/local\.rules|' /etc/snort/snort.conf
  ```

  Если не использовать PulledPork для управления набором правил, то необходимо вручную скачать набор правил с сайта [Snort](https://www.snort.org) и разархивировать их в каталог `/etc/snort/rules`, откуда Snort их будет загружать при запуске. В таком случае строки в конфигурационном файле `/etc/snort/snort.conf` комментировать не нужно.


После внесения изменений в настройки конфигурационного файла, необходимо выполнить следующую команду для проверки корректности конфигурационного файла:

```bash
sudo snort -T -c /etc/snort/snort.conf -i eth0
```

где опция **-T** указывает, что нужно протестировать текущую конфигурацию Snort, **-c** задаёт путь к конфигурационному файлу, а **-i** задаёт сетевой интерфейс, который Snort будет слушать. В случае успешного самотестирования Snort в конце вывода будут следующие строки:

```
...
Snort successfully validated the configuration!
Snort exiting
```


## Создание первого тестового правила для Snort

В текущем настроенном конфигурационном файле `/etc/snort/snort.conf` прописан единственный файл с правилами `include $RULE_PATH/local.rules`, который загружает Snort. В данном файле указываются специфичные для нашей среды правила, а также в нём будут указываться правила для тестирования, но пока что файл пустой. Добавим в данный файле первое тестовое правило. Для этого открываем файл `sudo gedit /etc/snort/rules/local.rules` и добавляем в него правило:

```
alert icmp any any -> $HOME_NET any (msg:"ICMP test detected"; GID:1; sid:10000001; rev:001; classtype:icmp-event;)
```

Данное правило означает: для любых ICMP-пакетов из любой сети в нашу домашнюю сеть **HOME_NET** генерируется предупреждение **ICMP test detected**. После того как внесли изменения в файл, который загружает Snort, хорошо бы протестировать конфигурационный файл Snort:

```bash
sudo snort -T -c /etc/snort/snort.conf -i eth0
```

В случае успешного тестирования мы увидим в выводе данной команды следующие строчки о нашем добавленном правиле:

```
+++++++++++++++++++++++++++++++++++++++++++++++++++
Initializing rule chains...
1 Snort rules read
    1 detection rules
    0 decoder rules
    0 preprocessor rules
1 Option Chains linked into 1 Chain Headers
0 Dynamic rules
+++++++++++++++++++++++++++++++++++++++++++++++++++

+-------------------[Rule Port Counts]---------------------------------------
|             tcp     udp    icmp      ip
|     src       0       0       0       0
|     dst       0       0       0       0
|     any       0       0       1       0
|      nc       0       0       1       0
|     s+d       0       0       0       0
+----------------------------------------------------------------------------
```

Далее протестируем данное правило, запустив Snort и убедившись, что он генерирует предупреждение при обработке ICMP-пакетов. Запускаем Snort со следующими опциями (ключами):

```bash
sudo snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
```

* **-A console** - консольная опция для печати предупреждений в быстром режиме на стандартный поток вывода stdout
* **-q** - тихий режим без показа баннера и отчёта
* **-u snort** - указываем пользователя snort из-под которого запускается Snort
* **-g snort** - указываем группу snort из-под которой запускается Snort
* **-c /etc/snort/snort.conf** - задаём путь к конфигурационному файлу snort.conf
* **-i eth0** - указываем сетевой интерфейс, который Snort будет слушать

После того, как запустили Snort с помощью указанной выше команды, необходимо с другой машины послать ICMP-пакеты на сетевой интерфейс машины, который слушает Snort (например, выполнив команду `ping 192.168.0.104`). В результате чего в запущенном терминале со Snort должны появиться примерно следующие строки с предупреждениями, что означает - Snort успешно работает в режиме сетевой IDS и генерирует предупреждения:

```
02/10-00:48:14.587138  [**] [1:10000001:1] ICMP test detected [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/10-00:48:15.590848  [**] [1:10000001:1] ICMP test detected [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/10-00:48:16.596818  [**] [1:10000001:1] ICMP test detected [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
02/10-00:48:17.602570  [**] [1:10000001:1] ICMP test detected [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.0.100 -> 192.168.0.104
```

Чтобы остановить работу Snort можно нажать **CTRL+C** в терминале с запущенным Snort. Также все лог-файлы сохраняются в каталоге `/var/log/snort`.


## Конфигурация Snort для ОС Raspbian

На ОС Raspbian (Raspberry Pi 3) Snort настраивается точно таким же способом без всяких изменений.

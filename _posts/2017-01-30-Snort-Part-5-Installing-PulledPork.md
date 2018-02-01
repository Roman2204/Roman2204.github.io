---
title:  "Snort: Часть 5. Установка PulledPork"
excerpt: "Скрипт на Perl, который автоматически скачивает последнюю версию правил по расписанию."
date:   2017-02-01T05:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - PulledPork
author_profile: true
---

{% include toc icon="file-text" %}


[PulledPork](https://github.com/shirkdog/pulledpork) представляет из себя скрипт на Perl, который предназначен для управления набором правил для Snort и Suricata. Скрипт по расписанию автоматически загружает последнюю версию правил. Чтобы скачать бесплатный основной набор правил для Snort нужно получить [Oinkcode](https://www.snort.org/oinkcodes).

## Установка PulledPork

Установливаем пререквизиты:

```bash
sudo apt-get install -y libcrypt-ssleay-perl liblwp-useragent-determined-perl
```

Скачиваем последнюю версию PulledPork, копируем Perl-скрипт в `/usr/local/bin` и копируем необходимые конфигурационные файлы в `/etc/snort`:

```bash
cd ~/snort_src
wget https://github.com/shirkdog/pulledpork/archive/master.tar.gz -O pulledpork-master.tar.gz
tar xzvf pulledpork-master.tar.gz
cd pulledpork-master/

sudo cp pulledpork.pl /usr/local/bin
sudo chmod +x /usr/local/bin/pulledpork.pl
sudo cp etc/*.conf /etc/snort
```


Теперь можно протестировать PulledPork с помощью команды:

```bash
roman@snort-vm:~$ /usr/local/bin/pulledpork.pl -V
PulledPork v0.7.3 - Making signature updates great again!
```

## Конфигурируем PulledPork

Конфигурационный файл PulledPork находится в `/etc/snort/pulledpork.conf`. В нём нужно произвести следующие изменения:

* Строка 19: Ввести свой **oinkcode** вместо `<oinkcode>` или закомментировать строку, если нет **oinkcode**
* Строка 29: Раскомментировать строку `#rule_url=https://rules.emergingthreats.net/|emerging.rules.tar.gz|open-nogpl`
* Строка 74: Заменить строку `rule_path=/usr/local/etc/snort/rules/snort.rules` на `rule_path=/etc/snort/rules/snort.rules`
* Строка 89: Заменить строку `local_rules=/usr/local/etc/snort/rules/local.rules` на `local_rules=/etc/snort/rules/local.rules`
* Строка 92: Заменить строку `sid_msg=/usr/local/etc/snort/sid-msg.map` на `sid_msg=/etc/snort/sid-msg.map`
* Строка 96: Заменить строку `sid_msg_version=1` на `sid_msg_version=2`
* Строка 119: Заменить строку `config_path=/usr/local/etc/snort/snort.conf` на `config_path=/etc/snort/snort.conf`
* Строка 133: Заменить строку `distro=FreeBSD-8-1` на `distro=Debian-6-0`
* Строка 141: Заменить строку `black_list=/usr/local/etc/snort/rules/iplists/default.blacklist` на `black_list=/etc/snort/rules/iplists/black_list.rules`
* Строка 150: Заменить строку `IPRVersion=/usr/local/etc/snort/rules/iplists` на `IPRVersion=/etc/snort/rules/iplists`


Запускаем PulledPork вручную, чтобы убедиться, что он корректно работает:

```bash
sudo /usr/local/bin/pulledpork.pl -c /etc/snort/pulledpork.conf -l
```

* **-c /etc/snort/pulledpork.conf** -the location of the snort.conf file
* **-l** - Write detailed logs to /var/log

Должен получится примерно следующий вывод:

```
https://github.com/shirkdog/pulledpork
  _____ ____
 `----,\    )
  `--==\\  /    PulledPork v0.7.3 - Making signature updates great again!
   `--==\\/
 .-~~~~-.Y|\\_  Copyright (C) 2009-2016 JJ Cummings
@_/        /  66\_  cummingsj@gmail.com
|    \   \   _(")
 \   /-| ||'--'  Rules give me wings!
  \_\  \_\\
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Checking latest MD5 for snortrules-snapshot-2990.tar.gz....
Rules tarball download of snortrules-snapshot-2990.tar.gz....
They Match
Done!
Checking latest MD5 for community-rules.tar.gz....
Rules tarball download of community-rules.tar.gz....
They Match
Done!
IP Blacklist download of http://talosintelligence.com/feeds/ip-filter.blf....
Reading IP List...
Checking latest MD5 for opensource.tar.gz....
Rules tarball download of opensource.tar.gz....
They Match
Done!
Prepping rules from community-rules.tar.gz for work....
Done!
Prepping rules from snortrules-snapshot-2990.tar.gz for work....
Done!
Prepping rules from opensource.tar.gz for work....
Done!
Reading rules...
Generating Stub Rules....
An error occurred: WARNING: ip4 normalizations disabled because not inline.

An error occurred: WARNING: tcp normalizations disabled because not inline.

An error occurred: WARNING: icmp4 normalizations disabled because not inline.

An error occurred: WARNING: ip6 normalizations disabled because not inline.

An error occurred: WARNING: icmp6 normalizations disabled because not inline.

Done
Reading rules...
Writing Blacklist File /etc/snort/rules/iplists/black_list.rules....
Writing Blacklist Version 808596066 to /etc/snort/rules/iplistsIPRVersion.dat....
Setting Flowbit State....
Enabled 19 flowbits
Done
Writing /etc/snort/rules/snort.rules....
Done
Generating sid-msg.map....
Done
Writing v2 /etc/snort/sid-msg.map....
Done
Writing /var/log/sid_changes.log....
Done
Rule Stats...
New:-------31644
Deleted:---0
Enabled Rules:----10778
Dropped Rules:----0
Disabled Rules:---20866
Total Rules:------31644
IP Blacklist Stats...
Total IPs:-----24122

Done
Please review /var/log/sid_changes.log for additional details
Fly Piggy Fly!
```


После выполнения данной команды в каталоге `/etc/snort/rules` появится файл **snort.rules** и **.so**-правила в `/usr/local/lib/snort_dynamicrules`. PulledPork собирает все наборы правил, которые он скачивает, в эти два файла. **.so**-правила, или **Shared Object**, или предварительно откомпилированные правила скомпилированы поставщиком правил для определённых платформ и представляют из себя более сложные правила, а также позволяют обфусцировать их (например, для обнаружения атак, которые ещё не пропатчены, но должны определяться без раскрытия самой сути уязвимости).

Чтобы Snort использовал скаченные правила необходимо в его конфигурации `/etc/snort/snort.conf` указать путь до этих правил. Добавляем следующую строку сразу за `include $RULE_PATH/local.rules`:

```
include $RULE_PATH/snort.rules
```

После добавления пути до новых правил в конфигурацию  `snort.conf` мы должны протестировать, что Snort корректно работает в режиме NIDS с включённым набором правил с помощью PulledPork:

Since we have modified snort.conf, we should test that Snort loads correctly in NIDS mode with the PulledPork rules included:

```bash
sudo snort -T -c /etc/snort/snort.conf -i eth0
```


Убедившись в корректности конфигурации Snort мы можем проверить, что Snort и Barnyard2 загружаются корректно, запустив их в фоновом режиме:

```bash
sudo snort -u snort -g snort -c /etc/snort/snort.conf -i eth0 -D
sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo -g snort -u snort -D
```

As before, ping the IP address of the Snort eth0 interface, and then check the database for more events (remember to use the MYSQLSNORTPASSWORD):

Пропингуем IP-адрес интерфейса eth0, который слушает Snort, а после проверим БД (пароль от БД - **MYSQLSNORTPASSWORD**):

```bash
mysql -u snort -p -D snort -e "select count(*) from event"
```

Количество отрепортированных событий должно быть больше, чем до пингования. Убедившись в корректности работы Snort c набором правил PulledPork, мы можем добавить PulledPork в crontab с правами root для ежедневного запуска.

```bash
sudo crontab -e
```

Выбираем подходящий редактор и добавляем строку, указывая запускать PulledPork по расписанию, например, каждый день в 13:45:

```
34 12 * * * /usr/local/bin/pulledpork.pl -c /etc/snort/pulledpork.conf -l
```


И отключаем созданных демонов:

```bash
roman@snort-vm:~$ ps aux | grep snort
avahi      477  0.0  0.0  32228  2956 ?        Ss   12:00   0:00 avahi-daemon: running [snort-vm.local]
snort     5796  0.0 12.5 1458300 509408 ?      Ssl  15:03   0:00 snort -u snort -g snort -c /etc/snort/snort.conf -i eth1 -D
snort     5804  5.4  2.4 206920 101376 ?       Ss   15:03   1:04 barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo -g snort -u snort -D
roman     5961  0.0  0.0  12728  2192 pts/0    R+   15:23   0:00 grep snort
roman@snort-vm:~$ sudo kill 5796
roman@snort-vm:~$ ps aux | grep barnyard2
snort     5804  5.3  2.4 206920 101376 ?       Ss   15:03   1:04 barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo -g snort -u snort -D
roman     5963  0.0  0.0  12728  2216 pts/0    R+   15:23   0:00 grep barnyard2
roman@snort-vm:~$ sudo kill 5804
```

Стоит обратить внимание, что Snort нужно перезагружать, чтобы новые добавленные правила вступили в силу. Можно это сделать с помощью команды `kill -SIGHUP snort-pid` или перезагрузив сервис Snort.

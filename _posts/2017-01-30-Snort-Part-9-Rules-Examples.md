---
title:  "Snort: Часть 9. Примеры правил"
excerpt: ""
date:   2017-02-01T09:00:00+03:00
categories:
   - Snort
tags:
   - Snort
   - IDS
   - IPS
   - Snort rules
author_profile: true
---

{% include toc icon="file-text" %}



## Примеры правил Snort

Файл с правилами:

```bash
sudo gedit /etc/snort/rules/local.rules
```

После того как внесли изменения в файл, тестируем конфигурационный файл Snort:

```bash
sudo snort -T -c /etc/snort/snort.conf -i eth0
```

Запускаем Snort:

```bash
sudo snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0 -K ascii
```

Опции Snort:


В каталоге `/var/log/snort/` находятся лог-файлы.


* Добавляем правило, выводящее предупреждение, когда кто-либо пытается зайти на сайт vk.com:

  ```
  alert tcp any any -> any any (msg: "Someone accessing vk.com"; content: "vk.com"; sid: 10000002;)
  ```

  Для тестирования правила достаточно открыть браузер и перейти на сайт vk.com.


* FIN сканирование

  ```
  alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"FIN Scan"; flags: F; sid:9000001;)
  ```

  Для тестирование правила достаточно произвести FIN-сканирование машины с помощью команды:

  ```bash
  sudo nmap -sF IP_ADDR
  ```

* SYN-FIN сканирование

  ```
  alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"SYN FIN Scan"; flags: SF; sid:9000000;)
  ```

  Для тестирование правила достаточно произвести сканирование машины, отправляя TCP-пакеты с установленными одновременно флагами SYN и FIN, с помощью команды:

  ```bash
  nmap --scanflags SYN,FIN HOSTNAME
  ```


* Доступ к Web-серверу

  ```
  alert tcp any any -> $HOME_NET 80 (msg:"Попытка доступа к Web-серверу на локальном компьютере"; sid:9000011)
  ```

  Для тестирование правила достаточно открыть браузер и перейти по адресу (IP-адресу), указанному в $HOME_NET





## Категории правил Snort

| Категория правил      | Описание                                                                                                                                                                                                                                                                                                                                                                                     |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| app-detect            | This category contains rules that look for, and control, the traffic of certain applications that generate network activity. This category will be used to control various aspects of how an application behaves.                                                                                                                                                                            |
| blacklist             | This category contains URI, USER-AGENT, DNS, and IP address rules that have been determined to be indicators of malicious activity. These rules are based on activity from the Talos virus sandboxes, public list of malicious URLs, and other data sources.                                                                                                                                 |
| browser-chrome        | This category contains detection for vulnerabilities present in the Chrome browser. (This is separate from the “browser-webkit” category, as Chrome has enough vulnerabilities to be broken out into it’s own, and while it uses the Webkit rendering engine, there’s a lot of other features to Chrome.)                                                                                    |
| browser-firefox       | This category contains detection for vulnerabilities present in the Firefox browser, or products that have the “Gecko” engine. (Thunderbird email client, etc)                                                                                                                                                                                                                               |
| browser-ie            | This category contains detection for vulnerabilities present in the Internet Explorer browser (Trident or Tasman engines)                                                                                                                                                                                                                                                                    |
| browser-webkit        | This category contains detection of vulnerabilities present in the Webkit browser engine (aside from Chrome) this includes Apple’s Safari, RIM’s mobile browser, Nokia, KDE, Webkit itself, and Palm.                                                                                                                                                                                        |
| browser-other         | This category contains detection for vulnerabilities in other browsers not listed above.                                                                                                                                                                                                                                                                                                     |
| browser-plugins       | This category contains detection for vulnerabilities in browsers that deal with plugins to the browser. (Example: Active-x)                                                                                                                                                                                                                                                                  |
| content-replace       | This category con taints any rule that utilizes the “replace” functionality inside of Snort.                                                                                                                                                                                                                                                                                                 |
| deleted               | When a rule has been deprecated or replaced it is moved to this categories. Rules are never totally removed from the ruleset, they are moved here.                                                                                                                                                                                                                                           |
| exploit               | This is an older category which will be deprecated soon. This category looks for exploits against software in a generic form.                                                                                                                                                                                                                                                                |
| exploit-kit           | This category contains rules that are specifically tailored to detect exploit kit activity. This does not include “post-compromise” rules (as those would be in indicator-compromise). Files that are dropped as result of visiting an exploit kit would be in their respective file category.                                                                                               |
| file-executable       | This category contains rules for vulnerabilities that are found or are delivered through executable files, regardless of platform.                                                                                                                                                                                                                                                           |
| file-flash            | This category contains rules for vulnerabilities that are found or are delivered through flash files. Either compressed or uncompressed, regardless of delivery method platform being attacked.                                                                                                                                                                                              |
| file-image            | This category contains rules for vulnerabilities that are found inside of images files. Regardless of delivery method, software being attacked, or type of image. (Examples include: jpg, png, gif, bmp, etc)                                                                                                                                                                                |
| file-identify         | This category is to identify files through file extension, the content in the file (file magic), or header found in the traffic. This information is usually used to then set a flowbit to be used in a different rule.                                                                                                                                                                      |
| file-multimedia       | This category contains rules for vulnerabilities present inside of multimedia files (mp3, movies, wmv)                                                                                                                                                                                                                                                                                       |
| file-office           | This category contains rules for vulnerabilities present inside of files belonging to the Microsoft Office suite of software. (Excel, PowerPoint, Word, Visio, Access, Outlook, etc)                                                                                                                                                                                                         |
| file-pdf              | This category contains rules for vulnerabilities found inside of PDF files. Regardless of method of creation, delivery method, or which piece of software the PDF affects (for example, both Adobe Reader and FoxIt Reader)                                                                                                                                                                  |
| file-other            | This category contains rules for vulnerabilities present inside a file, that doesn’t fit into the other categories above.                                                                                                                                                                                                                                                                    |
| indicator-compromise  | This category contains rules that are clearly to be used only for the detection of a positively compromised system, false positives may occur.                                                                                                                                                                                                                                               |
| indicator-obfuscation | This category contains rules that are clearly used only for the detection of obfuscated content. Like encoded JavaScript rules.                                                                                                                                                                                                                                                              |
| indicator-shellcode   | This category contains rules that are simply looking for simple identification markers of shellcode in traffic. This replaces the old ”shellcode.rules”.                                                                                                                                                                                                                                     |
| malware-backdoor      | This category contains rules for the detection of traffic destined to known listening backdoor command channels. If a piece of malicious soft are opens a port and waits for incoming commands for its control functions, this type of detection will be here. A simple example would be the detection for BackOrifice as it listens on a specific port and then executes the commands sent. |
| malware-cnc           | This category contains known malicious command and control activity for identified botnet traffic. This includes call home, downloading of dropped files, and ex-filtration of data. Actual commands issued from “Master to Zombie” type stuff will also be here.                                                                                                                            |
| malware-tools         | This category contains rules that deal with tools that can be considered malicious in nature. For example, LOIC.                                                                                                                                                                                                                                                                             |
| malware-other         | This category contains rules that are malware related, but don’t fit into one of the other ’malware’ categories.                                                                                                                                                                                                                                                                             |
| os-linux              | This category contains rules that are looking for vulnerabilities in Linux based OSes. Not for browsers or any other software on it, but simply against the OS itself.                                                                                                                                                                                                                       |
| os-solaris            | This category contains rules that are looking for vulnerabilities in Solaris based OSes. Not for any browsers or any other software on top of the OS.                                                                                                                                                                                                                                        |
| os-windows            | This category contains rules that are looking for vulnerabilities in Windows based OSes. Not for any browsers or any other software on top of the OS.                                                                                                                                                                                                                                        |
| os-other              | This category contains rules that are looking for vulnerabilities in an OS that is not listed above.                                                                                                                                                                                                                                                                                         |
| policy-multimedia     | This category contains rules that detect potential violations of policy for multimedia. Examples like the detection of the use of iTunes on the network. This is not for vulnerabilities found within multimedia files, as that would be in file-multimedia.                                                                                                                                 |
| policy-social         | This category contains rules for the detection potential violations of policy on corporate networks for the use of social media. (p2p, chat, etc)                                                                                                                                                                                                                                            |
| policy-other          | This category is for rules that may violate the end-users corporate policy bud do not fall into any of the other policy categories first.                                                                                                                                                                                                                                                    |
| policy-spam           | This category is for rules that may indicate the presence of spam on the network.                                                                                                                                                                                                                                                                                                            |
| protocol-finger       | This category is for rules that may indicate the presence of the finger protocol or vulnerabilities in the finger protocol on the network.                                                                                                                                                                                                                                                   |
| protocol-ftp          | This category is for rules that may indicate the presence of the ftp protocol or vulnerabilities in the ftp protocol on the network.                                                                                                                                                                                                                                                         |
| protocol-icmp         | This category is for rules that may indicate the presence of icmp traffic or vulnerabilities in icmp on the network.                                                                                                                                                                                                                                                                         |
| protocol-imap         | This category is for rules that may indicate the presence of the imap protocol or vulnerabilities in the imap protocol on the network.                                                                                                                                                                                                                                                       |
| protocol-pop          | This category is for rules that may indicate the presence of the pop protocol or vulnerabilities in the pop protocol on the network.                                                                                                                                                                                                                                                         |
| protocol-services     | This category is for rules that may indicate the presence of the rservices protocol or vulnerabilities in the rservices protocols on the network.                                                                                                                                                                                                                                            |
| protocol-voip         | This category is for rules that may indicate the presence of voip services or vulnerabilities in the voip protocol on the network.                                                                                                                                                                                                                                                           |
| pua-adware            | This category deals with “pua” or Potentially Unwanted Applications that deal with adware or spyware.                                                                                                                                                                                                                                                                                        |
| pua-p2p               | This category deals with “pua” or Potentially Unwanted Applications that deal with p2p.                                                                                                                                                                                                                                                                                                      |
| pua-toolbars          | This category deals with “pua” or Potentially Unwanted Applications that deal with toolbars installed on the client system. (Google Toolbar, Yahoo Toolbar, Hotbar, etc)                                                                                                                                                                                                                     |
| pua-other             | This category deals with “pua” or Potentially Unwanted Applications that don’t fit into one of the categories shown above.                                                                                                                                                                                                                                                                   |
| server-apache         | This category deals with vulnerabilities in or attacks against the Apache Web Server.                                                                                                                                                                                                                                                                                                        |
| server-iis            | This category deals with vulnerabilities in or attacks against the Microsoft IIS Web server.                                                                                                                                                                                                                                                                                                 |
| server-mssql          | This category deals with vulnerabilities in or attacks against the Microsoft SQL Server.                                                                                                                                                                                                                                                                                                     |
| server-mysql          | This category deals with vulnerabilities in or attacks against Oracle’s MySQL server.                                                                                                                                                                                                                                                                                                        |
| server-oracle         | This category deals with vulnerabilities in or attacks against Oracle’s Oracle DB Server.                                                                                                                                                                                                                                                                                                    |
| server-webapp         | This category deals with vulnerabilities in or attacks against Web based applications on servers.                                                                                                                                                                                                                                                                                            |
| server-mail           | This category contains rules that detect vulnerabilities in mail servers. (Exchange, Courier). These are separate from the protocol categories, as those deal with the traffic going to the mail servers itself.                                                                                                                                                                             |
| server-other          | This category contains rules that detect vulnerabilities in or attacks against servers that are not detailed in the above list.                                                                                                                                                                                                                                                              |

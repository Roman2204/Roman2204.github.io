---
title:  "Установка ОС на виртуальную машину"
excerpt: ""
date:   2017-01-25T00:00:00+03:00
categories:
   - OS
tags:
   - OS
   - Linux
   - Kali Linux
   - Debian
   - Virtual Machine
author_profile: true
---

{% include toc icon="file-text" %}

Наиболее популярные системы виртуализации:

* Свободно распространяемая [Oracle VM VirtualBox](https://www.virtualbox.org/)
* [VMware Workstation](http://www.vmware.com/ru/products/workstation.html) и его бесплатная версия [VMware Player]() для некоммерческого использования

## Установка Kali Linux на VirtualBox

В меню "File -> Preferences" (Ctrl+G) можно настроить папку по умолчанию для расположения виртуальных машин "Default Machine Folder" во вкладке General:
![VirtualBox Preferences](http://i.imgur.com/G4aMaek.png)

В том же меню во вкладке Extensions следует установить "Oracle VM VirtualBox Extension Pack", предварительно скачав его с официального сайта [Oracle VM VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads).

Рассмотрим два варианта установки Kali Linux на виртуальную машину:

* С использованием готового образа для виртуальной машины
* Создание виртуальной машины и ручная установка в ней Kali Linux

### Готовый образ для виртуальной машины

Скачиваем с официального сайта готовый образ Kali Linux для VirtualBox [Kali Linux VirtualBox Images](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/)

После чего остаётся импортировать данный образ, открыв меню  "File -> Import Appliance..." (Ctrl+I) и выбрав скаченный образ Kali Linux.

Виртуальная машина готова к работе. Остаётся только запустить её. Данные для входа по умолчанию:

* Username: root
* Password: toor

### Ручная установка

Скачиваем с официального сайта образ Kali Linux [Download Kali Linux Images](https://www.kali.org/downloads/), выбрав, например, "Kali Linux 64 bit".

#### Создание виртуальной машины VirtualBox

Создаём виртуальную машину со следующими настройками, нажав на "New" (Ctrl+N):

![Create Virtual Machine](http://i.imgur.com/rQP7sQQ.png)

Создаём виртуальный жёсткий диск для виртуальной машины:

![Create Virtual Hard Disk](http://i.imgur.com/eYFBx8t.png)

Настраиваем созданную виртуальную машину, выбрав её и нажав на "Settings" (Ctrl+S):

и т.д.



## VMware



## Полезные команды после установки ОС

### Добавить права администратора пользователю в Debian GNOME

Открываем **System Settings -> Administration -> Users** (**Параметры -> Пользователи**), выбираем нашего пользователя и даём права администратора, изменив параметр «Account Type» («Тип учётной записи») на «Administrator» («Администратор»). В GNOME предварительно нужно нажать «Разблокировать» и ввести пароль администратора. Чтобы изменения вступили в силу необходимо перезапустить сеанс, для этого выходим из система, а затем вновь в неё входим.

### Устанавка обновления для Debian

```bash
sudo apt-get update && sudo apt-get upgrade
```

* `apt-get update` - обновление индекса пакетов
* `apt-get upgrade` - обновление пакетов


### Устанавка дополнительных пакетов и ядра

```bash
sudo apt-get install build-essential dkms linux-headers-$(uname -r)
```

*	`build-essential` - информационный список пакетов необходимых для сборки пакетов Debian
*	`dkms` - инфраструктура для поддержки динамически загружаемых модулей ядра, позволяющая обновлять модули ядра без изменения всего ядра. Также позволяет легко пересобирать модули при обновлении ядра.
*	`linux-headers-$(uname -r)` - заголовочный файл ядра, где `uname -r` - команда для получения версии ядра


### Горячая комбинация клавиш для запуска терминала

**Параметры -> Клавиатура -> Комбинации клавиш -> Дополнительные комбинации**, нажимаем "+" (**Добавить**). Далее указываем название комбинации, оно может быть любое, и указываем команду `/usr/bin/gnome-terminal` (`gnome-terminal`), которая будет выполняться при нажатии заданной комбинации. Нажимаем **Добавить**. После чего нажимаем на появившуюся запись с нашей комбинацией и нажимаем клавиши на клавиатуре, которые будут вызывать её. Удобно установить *Ctrl + Alt + t*.

Утилиту терминала можно найти, например, с помощью поиска, если заранее не знать, что команда `gnome-terminal`:

```bash
find / -name "*terminal"
```


## Возможные проблемы

### Проблема с Bridge Network в VirtualBox в Windows 7

Ошибка при подключении Bridge Network в VirtualBox:

```
Failed to open/create the internal network 'HostInterfaceNetworking-Intel(R) Ethernet Connection I217-V' (VERR_INTNET_FLT_IF_NOT_FOUND).
Failed to attach the network LUN (VERR_INTNET_FLT_IF_NOT_FOUND).


Result Code:
E_FAIL (0x80004005)
Component:
ConsoleWrap
Interface:
IConsole {872da645-4a9b-1727-bee2-5585105b9eed}
```

---
title:  "Snort: Часть 8. Создание собственных правил для Snort. Синтаксис правил."
excerpt: "В данной статье рассмотрим синтаксис правил для Snort. А также возможные значения полей заголовка и опций правила."
date:   2017-02-01T08:00:00+03:00
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

Данная статья в основном основана на официальной документации [SNORT Users Manual](http://manual-snort-org.s3-website-us-east-1.amazonaws.com).

Snort использует правила, написанные простым, но в то же время гибким и достаточно мощным языком. Большинство правил пишутся в одну строку, хотя могут занимать и несколько строк, в этом случае каждая строка, кроме последней, должна заканчиваться символом "\" (без кавычек). В более сложных случаях можно также вызывать другие программы, используя инструкцию включения.
Правила для Snort делятся на два вида:

* **Бесконтекстные (обычные)** - применяются для каждого пакета отдельно, без связи с другими пакетами
* **Контекстные (правила препроцессоров)** - применяться к той или иной совокупности (последовательности) пакетов.

Правила Snort состоят из **заголовка** (Rule Header) и **опций** (Rule Options). Заголовок содержит описание действия, протокол передачи данных, IP-адреса, сетевые маски и порты источника и назначения. После заголовка правила следует необязательная часть правила - его опции, они включают определение дополнительных критериев выполнения правила и определение дополнительных реагирующих действий. Они используются для организации более жесткой и направленной фильтрации траффика. Весь набор опций заключается в круглые скобки, сами опции отделяются друг от друг с помощью точки с запятой ";" (последняя опция в списке тоже должна заканчиваться этим символом). Ключевые слова (keywords) опций отделяются от своих аргументов (values) двоеточием ":". Структура правил Snort в общем случае выглядит следующим образом:

<table style="text-align: center;">
  <thead>
    <tr>
      <th colspan="7">Заголовок</th>
      <th colspan="4">Опции</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td> Действие</td>
      <td> Протокол</td>
      <td> IP-адреса отправителей</td>
      <td> Порты отправителей</td>
      <td> Оператор направления</td>
      <td> IP-адреса получателей</td>
      <td> Порты получателей</td>
      <td> [Мета данные]</td>
      <td> [Данные в полезной нагрузке]</td>
      <td> [Данные в заголовке]</td>
      <td> [Действие после обнаружения]</td>
    </tr>
  </tbody>
</table>

Синтаксис записи правил Snort:

```
<Действие> <Протокол> <IP-адреса отправителей> <Порты отправителей> <Оператор направления> <IP-адреса получателей> <Порты получателей> (ключ_1 : значение_1; ключ_2 : значение_2; ... ключ_N : значение_N;)
```


## Заголовок правила

Допустимые параметры для каждого поля заголовка правил Snort:

### Действия правила

| Действие | Описание                                                                                                    |
|----------|-------------------------------------------------------------------------------------------------------------|
| alert    | генерирует предупреждение, используя указанное предупреждение, и передаёт информацию системе журналирования |
| log      | просто протоколирует пакеты без предупреждений                                                              |
| pass     | игнорирует пакеты                                                                                           |
| activate | генерирует предупреждение, затем включает указанное динамическое правило                                    |
| dynamic  | остаётся пассивным, пока не активируется динамическим правилом, затем действует как log                     |

В режиме inline к предыдущим действиям добавляются дополнительные действия:

| Действие | Описание                                                                                                                                                                                           |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| drop     | блокирует (отбрасывает) пакет и передаёт информацию системе журналирования                                                                                                                         |
| sdrop    | блокирует (отбрасывает) пакет и не использует систему журналирования                                                                                                                               |
| reject   | блокирует (отбрасывает) пакет, передаёт информацию системе журналирования, а затем посылает сегмент сброса TCP (TCP RST), если протокол TCP, или сообщение ICMP-порт недоступен, если протокол UDP |


Также можно создать свои собственные типы правил и связать один или несколько выходных модулей с ними. Можно затем использовать созданные типы правил в качестве действий в правилах Snort. В примере ниже создаётся тип правила, который будет регистрировать только tcpdump:

```
ruletype suspicious
{
    type log
    output log_tcpdump: suspicious.log
}
```


А в этом примере создаётся тип правила, регистрирующей и tcpdump, и syslog:

```
ruletype redalert
{
    type alert
    output alert_syslog: LOG_AUTH LOG_ALERT
    output log_tcpdump: suspicious.log
}
```

### Протоколы

Обозначает протокол передачи данных. На данный момент Snort умеет анализировать на предмет подозрительного содержания/поведения 4 типа протоколов: TCP, UDP, ICMP, IP, соответственно, возможны 4 значения: tcp, udp, icmp или ip, что означает любой IP-протокол. В будущем, возможно, этот список пополнят и другие протоколы, к примеру, ARP, IGRP, GRE, OSPF, RIP, IPX и другие.

### IP-адресация

Поскольку Snort не имеет встроенного механизма получения IP адреса, используя доменное имя, то нужно указывать конкретный IP адрес или же диапазон IP адресов. В этом параметре можно использовать маски. Адреса задаются в формате: IP/mask, где IP – IP-адрес сети или узла, mask – маска сети, которая задаётся как десятичное число, которое равняется числу единиц в двоичной маске. Например, для сетей класса C /24 (число 24 эквивалентное шестнадцатеричной маске FF.FF.FF.0), для сетей класса B - /16, также можно использовать маску /32 и другие. Здесь может применяться и отрицание (инвертирование), обозначаемое символом "!" (Например: !127.0.0.1). Если вместо IP адреса указать ключевое слово any, то это будет подразумевать абсолютно все хосты. Для указания списка можно использовать перечисление IP адресов через "," содержащихся в квадратных скобках. (Например: [212.116.1.1,10.10.1.0/24]). В качестве IP-адреса можно использовать переменные HOME_NET, EXTERNAL_NET и другие.

### Порты

После IP адреса указывается номер порта, с которого отсылаются данные и на который приходят. Можно указать диапазон портов: 1:1024 (все порты в диапазоне от 1 до 1024 включая 1 и 1024). Часто используется оператор отрицания "!" (Например: !123:321 исключает все порты в диапазоне от 123 до 321). Если опущен один из параметров диапазона, например ":321" или "123:", то пропускаемый параметр принимает крайнее значение общего количества портов, то есть 0 или 65535.

### Операторы направления

Оператор направления служит для обозначения направления траффика, для которого применяется правило, и обозначается "->" (знаком минуса и закрывающей угловой скобкой). IP-адрес и номер порта слева от оператора определяют источник траффика, а справа от него - назначение. Существует также оператор так называемой "двунаправленности" и обозначается "<>" (двумя угловыми скобками). Этот оператор говорит Snort рассматривать указанные пары адресов и портов в обе стороны, вне зависимости от того, кто является источником, а кто – получателем. Это удобно в тех случаях, когда нужно сохранить траффик от обеих сторон, например, в Telnet или POP3 сессиях. Важно отметить, что оператор "<-" не существует.




## Опции правила

Все опции можно разделить на четыре большие категории:

* **general (meta-data)** - данные опции предоставляют информацию о правиле, но никак не влияют на обнаружение;
* **payload** - данные опции позволяют искать информацию внутри полезной нагрузки (данных пользователя) пакетов и могут быть взаимосвязаны;
* **non-payload** - данные опции позволяют искать информацию внутри служебной (управляющей) информации о пакете (заголовке);
* **post-detection** - данные опции являются определёнными триггерами, указывающими задачи, которые необходимо осуществить после срабатывание правила.


### General (meta-data)

* **msg**

  Указывает сообщение, текстовое описание сигнала тревоги (для экранирования используется символ "\"), которое будет выведено или же записано, используя систему журналирования.

  ***Синтаксис***: `msg:"<message text>";`

  ***Примеры***:

* **reference**

  Указывает ссылки на online системы идентификации атак. Значениями этого поля могут быть ссылки на ресурсы bugtraq, cve, nessus, arachnids, mcafee и другие url. Идентификация осуществляется по SID номерам.

  Поддерживаемые системы:

    - bugtraq: http://www.securityfocus.com/bid/
    - cve: http://cve.mitre.org/cgi-bin/cvename.cgi?name=
    - nessus: http://cgi.nessus.org/plugins/dump.php3?id=
    - arachnids: (в данный момент не работает) http://www.whitehats.com/info/IDS
    - mcafee: http://vil.nai.com/vil/content/v_
    - osvdb: http://osvdb.org/show/osvdb/
    - msb: http://technet.microsoft.com/en-us/security/bulletin/
    - url: http://

  ***Синтаксис***: `reference:<id system>, <id>; [reference:<id system>, <id>;]`

  ***Примеры***:

    ```
    alert tcp any any -> any 7070 (msg:"IDS411/dos-realaudio"; \
    flags:AP; content:"|fff4 fffd 06|"; reference:arachnids,IDS411;)
    ```

    ```
    alert tcp any any -> any 21 (msg:"IDS287/ftp-wuftp260-venglin-linux"; \
    flags:AP; content:"|31c031db 31c9b046 cd80 31c031db|"; \
    reference:arachnids,IDS287; reference:bugtraq,1387; \
    reference:cve,CAN-2000-1574;)
    ```


* **gid**

  Ключевое слово gid (generator id) используется для идентификации того, какая часть Snort генерирует событие, когда срабатывает конкретное правило. Например, gid равный 1 ассоциируется с подсистемой правил, а различные gid свыше 100 предназначены для определённых препроцессоров и декодеров. Опция gid является необязательной, и если она не определена в правиле, то по умолчанию она устанавливается равной 1, и правило будет являться частью общей подсистемы правил. Чтобы избежать потенциальных конфликтов с gid, определёнными в Snort, рекомендуется использовать значения начиная с 1000000. Для общих правил не рекомендуется использовать ключевое слово gid. Данная опция должна быть использована с опцией sid. Файл "etc/gen-msg.map" содержит больше информации о gid препроцессоров и декодеров.

  ***Синтаксис***: `gid:<generator id>;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"BOB"; gid:1000001; sid:1; rev:1;)
    ```

* **sid**

  Ключевое слово sid (Snort id или иногда упоминается как signature id) используется для уникальной идентификации правил Snort. По значению его аргумента можно легко идентифицировать правило. Данное ключевое слово должно использоваться вместе с ключевым словом rev. Файл "sid-msg.map" содержит соответствие предупреждающих сообщений и идентификаторов правил Snort.
  Значения аргумента:

  < 100 зарезервировано разработчиками
  100 - 999.999 использованы в правилах, уже включенных в дистрибутив Snort
  >= 1.000.000 можно использовать для собственных правил

  ***Синтаксис***: `sid:<snort rules id>;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"BOB"; sid:1000983; rev:1;)
    ```


* **rev**

  Указывает значение версии правила. С помощью REV интерпретатор правил Snort определяет версию написанного правила. Этот параметр используется в паре с SID.

  ***Синтаксис***: `rev:<revision integer>;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"BOB"; sid:1000983; rev:1;)
    ```


* **classtype**

  Используется для присвоения категории атаки, к которой необходимо отнести правило, являющееся частью более общего класса атак. Snort предоставляет набор классов, которые используются предоставляемыми правилами по умолчанию. Классификация атак позволяет лучше организовать события, производимые Snort.
  Классификация атак представлена в файле "classification.conf". В файле используется следующий синтаксис для каждой записи:

  ```
  config classification: <имя класса>,<описание класса>,<приоритет по умолчанию>
  ```

  Приоритет 1 (high) является наиболее высоким, а 4 (very low) - самым низким. Также классификация типов атак представлена в таблице:

  <table>
    <thead>
      <tr>
        <th> Тип класса </th>
        <th> Описание </th>
        <th> Приоритет </th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td> attempted-admin </td>
        <td> Попытка получения прав администратора </td>
        <td> high </td>
      </tr>
      <tr>
        <td> attempted-user </td>
        <td> Попытка получения прав пользователя </td>
        <td> high </td>
      </tr>
      <tr>
        <td> inappropriate-content </td>
        <td> Обнаружено неприемлемое (несоответствующее) содержание </td>
        <td> high </td>
      </tr>
      <tr>
        <td> policy-violation </td>
        <td> Потенциальное нарушение корпоративной конфиденциальности </td>
        <td> high </td>
      </tr>
      <tr>
        <td> shellcode-detect </td>
        <td> Обнаружен исполняемый код </td>
        <td> high </td>
      </tr>
      <tr>
        <td> successful-admin </td>
        <td> Успешное получение прав администратора (повышение привилегий) </td>
        <td> high </td>
      </tr>
      <tr>
        <td> successful-user </td>
        <td> Успешное получение прав пользователя (повышение привилегий) </td>
        <td> high </td>
      </tr>
      <tr>
        <td> trojan-activity </td>
        <td> Обнаружена активность сетевой троянской программы </td>
        <td> high </td>
      </tr>
      <tr>
        <td> unsuccessful-user </td>
        <td> Неудачная попытка получения прав пользователя </td>
        <td> high </td>
      </tr>
      <tr>
        <td> web-application-attack </td>
        <td> Атака на Web-приложение </td>
        <td> high </td>
      </tr>
      <tr>
        <td> attempted-dos </td>
        <td> Предпринята попытка атаки отказ в обслуживании (DoS) </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> attempted-recon </td>
        <td> Попытка утечки информации (разведка) </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> bad-unknown </td>
        <td> Потенциально нежелательный траффик </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> default-login-attempt </td>
        <td> Попытка входа с помощью стандартного логина/пароля </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> denial-of-service </td>
        <td> Обнаружена атака отказ в обслуживании (DoS) </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> misc-attack </td>
        <td> Прочие атаки </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> non-standard-protocol </td>
        <td> Обнаружено использование нестандартного протокола или нестандартное событие </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> rpc-portmap-decode </td>
        <td> Decode of an RPC Query (Декодирован удалённый вызов процедуры (RPC)) (Обнаружен запрос RPC) </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> successful-dos </td>
        <td> Успешная DOS-атака </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> successful-recon-largescale </td>
        <td> Крупномасштабная утечка информации </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> successful-recon-limited </td>
        <td> Утечка информации </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> suspicious-filename-detect </td>
        <td> Обнаружено подозрительное имя файла </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> suspicious-login </td>
        <td> Обнаружена попытка входа с подозрительным логином </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> system-call-detect </td>
        <td> Обнаружено обращение к ядру системы (system call) (Обнаружен системный вызов) </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> unusual-client-port-connection </td>
        <td> Клиент использует нестандартный порт </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> web-application-activity </td>
        <td> Доступ к потенциально уязвимому Web-приложению </td>
        <td> medium </td>
      </tr>
      <tr>
        <td> icmp-event </td>
        <td> Общее событие ICMP </td>
        <td> low </td>
      </tr>
      <tr>
        <td> misc-activity </td>
        <td> Прочая активность </td>
        <td> low </td>
      </tr>
      <tr>
        <td> network-scan </td>
        <td> Обнаружена попытка сканирования сети </td>
        <td> low </td>
      </tr>
      <tr>
        <td> not-suspicious </td>
        <td> Не являющийся подозрительным траффик </td>
        <td> low </td>
      </tr>
      <tr>
        <td> protocol-command-decode </td>
        <td> Generic Protocol Command Decode (Обнаружена попытка шифрования) (Обнаружена обычная команда протокола) </td>
        <td> low </td>
      </tr>
      <tr>
        <td> string-detect </td>
        <td> Обнаружена подозрительная строка </td>
        <td> low </td>
      </tr>
      <tr>
        <td> unknown </td>
        <td> Неизвестный траффик </td>
        <td> low </td>
      </tr>
      <tr>
        <td> tcp-connection </td>
        <td> Обнаружено TCP соединение </td>
        <td> very low </td>
      </tr>
    </tbody>
  </table>

  ***Синтаксис***: `classtype:<class name>;`

  ***Примеры***:

    ```
    alert tcp any any -> any 25 (msg:"SMTP expn root"; flags:A+; content:"expn root"; nocase; classtype:attempted-recon;)
    ```

  ***Предупреждения***:

    Опция classtype может иметь только те значения для классификации, которые определены в `snort.conf` с помощью `config classification`. Snort предоставляет стандартный набор классификации в файле `classification.config`, который используется в поставляемых наборах правил.


* **priority**

  Задаёт правилам уровень важности. Возможно использовать параметр priority вместе с classtype, при этом изменится уровень приоритета параметра classtype.

  ***Синтаксис***: `priority:<priority integer>;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (msg:"WEB-MISC phf attempt"; flags:A+; content:"/cgi-bin/phf"; priority:10;)
    ```

    ```
    alert tcp any any -> any 80 (msg:"EXPLOIT ntpdx overflow"; dsize:>128; classtype:attempted-admin; priority:10;)
    ```


* **metadata**

  Позволяет автору правил включать дополнительную информацию о правиле, как правило, в формате "ключ-значение". Ключи и значения тега metadata перечислены в таблице ниже:

  <table>
    <thead>
      <tr>
        <th> Ключ </th>
        <th> Описание </th>
        <th> Формат значения </th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td> engine </td>
        <td> Указывает правило библиотеки общего пользования (Indicate a Shared Library Rule) </td>
        <td> "shared" </td>
      </tr>
      <tr>
        <td> soid </td>
        <td> GID и SID правила библиотеки общего пользования (Shared Library Rule Generator and SID) </td>
        <td>sid </td>
      </tr>
      <tr>
        <td> service </td>
        <td> Идентификатор сервиса на основе цели (Target-Based Service Identifier) </td>
        <td> "http" </td>
      </tr>
    </tbody>
  </table>

  Отличные от указанных в таблице ключи Snort фактически игнорирует, поэтому они могут быть записаны в свободной форме в формате "ключ-значение". Несколько ключей подряд разделяются запятыми, а ключи и значения отделяются между собой пробелом.

  ***Синтаксис***:

    ```
    metadata:key1 value1;
    metadata:key1 value1, key2 value2;
    ```

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (msg:"Shared Library Rule Example"; metadata:engine shared; metadata:soid 3|12345;)
    ```

    ```
    alert tcp any any -> any 80 (msg:"Shared Library Rule Example"; metadata:engine shared, soid 3|12345;)
    ```

    ```
    alert tcp any any -> any 80 (msg:"HTTP Service Rule Example"; metadata:service http;)
    ```


### Payload

* **content**

  Позволяет устанавливать условие в правила, которые ищут определённое содержание (контент) в полезной нагрузке пакетов. Условия могут содержать как двоичные данные, так и текстовые. Двоичные данные должны быть заключены между вертикальными чертами "\|" в виде байт-кода. Байт-код представляет двоичные данные в виде шестнадцатеричных чисел. В одном правиле может быть указано несколько **content**-условий. "!" - модификатор отрицания. Если правилу предшествует модификатор отрицания, то правило срабатывает на пакетах, которые не содержат заданный контент.

  Ключевое слово **content** имеет ряд модификаторов, которые изменяют поведение ранее указанного **content**. Список модификаторов:

    - [nocase](#nocase)
    - [rawbytes](#rawbytes)
    - [depth](#depth)
    - [offset](#offset)
    - [distance](#distance)
    - [within](#within)
    - [http_client_body](#http_client_body)
    - [http_client_body](#http_client_body)
    - [http_cookie](#http_cookie)
    - [http_raw_cookie](#http_raw_cookie)
    - [http_header](#http_header)
    - [http_raw_header](#http_raw_header)
    - [http_method](#http_method)
    - [http_uri](#http_uri)
    - [http_raw_uri](#http_raw_uri)
    - [http_stat_code](#http_stat_code)
    - [http_stat_msg](#http_stat_msg)
    - [fast_pattern](#fast_pattern)


  ***Синтаксис***: `content:[!]"<content string>";`

  ***Примеры***:

    ```
    alert tcp any any -> any 139 (content:"|5c 00|P|00|I|00|P|00|E|00 5c|";)
    ```

    ```
    alert tcp any any -> any 80 (content:!"GET";)
    ```

  ***Предупреждения***:

    Необходимо экранировать следующие символы:

    ```
    ; \ "
    ```

* **protected_content**

  Имеет схожую функциональность с **content**, однако работает несклько иным образом. Основное преимущество ключевого слова **protected_content** над **content** в том, что оно позволяет скрыть целевой контент, раскрывая только хэш-сумму (дайджест) указанного контента. Как и в случае **content**, основная цель - сопоставить строки определённых байтов. Поиск осуществляется путём хэширования частей входящих сообщений и сравнения полученных результатов с предоставляемой в условии хэш-суммой. Из-за чего проделывается очень большой объём вычислений.

  На данный момент с ключевым словом **protected_content** возможно использование алгоритмов хэширования MD5, SHA256, и SHA512. Алгоритм хэширования должен быть указан в правиле с использованием ключевого слова [**hash**](#hash), если он не задан по умолчанию в конфигурации Snort. Кроме того, вместе с **protected_content** обязательно должен быть указан модификатор [**length**](#length), чтобы указать длину исходных данных.

  Как и в случае **content**, в правиле возможно использование нескольких условий **protected_content**. В правиле допускается совместное использование **content** и **protected_content**. Также в **protected_content** можно использовать модификатор отрицания "!".

  Ключевое слово **protected_content** имеет те же модификаторы, что и **content**, за исключением следующих:

    - [nocase](#nocase)
    - [fast_pattern](#fast_pattern)
    - [depth](#depth)
    - [within](#within)


  ***Синтаксис***: `protected_content:[!]"<content hash>", length:orig_len[, hash:md5|sha256|sha512];`

  ***Примеры***:

    Следующие правила срабатывают на строке "HTTP".

    ```
    alert tcp any any <> any 80 (msg:"MD5 Alert";
    protected_content:"293C9EA246FF9985DC6F62A650F78986"; hash:md5; offset:0; length:4;)
    ```

    ```
    alert tcp any any <> any 80 (msg:"SHA256 Alert";
    protected_content:"56D6F32151AD8474F40D7B939C2161EE2BBF10023F4AF1DBB3E13260EBDC6342";
    hash:sha256; offset:0; length:4;)
    ```

* <a name="hash"></a>**hash**

  Используется для указания алгоритма хэширования, который будет использоваться при сопоставлении с образцом, заданным в условии **protected_content**. Если в конфигурации Snort не указан алгоритм по умолчанию, то необходимо указать используемый алгоритм в правиле с **protected_content**. На данный момент поддерживаются следующие алгоритмы хэширования: MD5, SHA256, и SHA512.

  ***Синтаксис***: `hash:[md5|sha256|sha512];`

* <a name="length"></a>**length**

  Используется для задания исходной длины контента, для которого задана хэш-сумма (дайджест) в **protected_content**. Указанное значение должно быть в интервале от 0 до 65536.

  ***Синтаксис***: `length:[<original_length>];`

* <a name="nocase"></a>**nocase**

  Является модификатором для располагающегося до него ключевого слова **content**, указывая ему сравнивать содержание без учета регистра символов.

  ***Синтаксис***: `nocase;`

  ***Примеры***:

    ```
    alert tcp any any -> any 21 (msg:"FTP ROOT"; content:"USER root"; nocase;)
    ```

* <a name="rawbytes"></a>**rawbytes**

  Ключевое слово **rawbytes** является модификатором для располагающегося до него ключевого слова **content**, позволяя работать с необработанными данными пакета, игнорируя любое декодирование, произведённое с помощью препроцессоров.

  HTTP Inspect имеет набор ключевых слов **http_raw_cookie**, **http_raw_header**, **http_raw_uri** и др. для работы с необработанными данными, которые сопоставляют определённые части HTTP запросов и ответов. С данными ключевыми словами использовать **rawbytes** не нужно, так как эти условия по умолчанию работают с необработанными данными.

  Большинство других препроцессоров по умолчанию используют декодированные/нормализованные данные для сопоставления с образцом. Поэтому для сопоставления с произвольными необработанными данными из пакета необходимо указывать ключевое слово **rawbytes**.

  ***Синтаксис***: `rawbytes;`

  ***Примеры***:

    ```
    alert tcp any any -> any 21 (msg:"Telnet NOP"; content:"|FF F1|"; rawbytes;)
    ```


* <a name="depth"></a>**depth**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="offset"></a>**offset**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="distance"></a>**distance**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="within"></a>**within**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="http_client_body"></a>**http_client_body**

  Ключевое слово **http_client_body** ограничивает поиск по телу запроса HTTP-клиента. Ключевое слово **http_client_body** является модификатором для располагающегося до него ключевого слова **content**. Размер области данных, по которым производится поиск, зависит от опции **post_depth** в HttpInspect. Паттерн с данным ключевым словом не будет работать, если **post_depth** в установлена "-1".

  ***Синтаксис***: `http_client_body;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"EFG"; http_client_body;)
    ```


* <a name="http_cookie"></a>**http_cookie**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="http_raw_cookie"></a>**http_raw_cookie**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="http_header"></a>**http_header**

	Ключевое слово **http_header** ограничивает поиск по извлечённым полям заголовка запроса HTTP-клиента или ответа HTTP-сервера (определяется в конфигурации HttpInspect). Ключевое слово **http_method** является модификатором для располагающегося до него ключевого слова **content**. Извлечённые поля заголовка можно нормализовать, определив это в конфигурации HttpInspect.

  ***Синтаксис***: `http_header;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"EFG"; http_header;)
    ```


* <a name="http_raw_header"></a>**http_raw_header**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="http_method"></a>**http_method**

  Ключевое слово **http_method** ограничивает поиск по извлечённому методу из запроса HTTP-клиента. Ключевое слово **http_method** является модификатором для располагающегося до него ключевого слова **content**.

  ***Синтаксис***: `http_method;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"GET"; http_method;)
    ```

* <a name="http_uri"></a>**http_uri**

  Ключевое слово **http_uri** ограничивает поиск по нормализованному полю URI запроса. Ключевое слово **http_uri** является модификатором для располагающегося до него ключевого слова **content**. Использование опции **http_uri** после **content** эквивалентно опции **uricontent**.

  ***Синтаксис***: `http_uri;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"EFG"; http_uri;)
    ```

* <a name="http_raw_uri"></a>**http_raw_uri**

  Ключевое слово **http_raw_uri** ограничивает поиск по ненормализованному полю URI запроса. Ключевое слово **http_raw_uri** является модификатором для располагающегося до него ключевого слова **content**.

  ***Синтаксис***: `http_raw_uri;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"EFG"; http_raw_uri;)
    ```

* <a name="http_stat_code"></a>**http_stat_code**

  Ключевое слово **http_stat_code** ограничивает поиск по извлечённому полю пояснения к статус коду ответа HTTP-сервера. Ключевое слово **http_stat_code** является модификатором для располагающегося до него ключевого слова **content**. Поле пояснения к статус коду будет извлечено, если только задана опция **extended_response_inspection** в **HttpInspect**.


  ***Синтаксис***: `http_stat_code;`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"ABC"; content:"200"; http_stat_code;)
    ```

* <a name="http_stat_msg"></a>**http_stat_msg**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **http_encode**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* <a name="fast_pattern"></a>**fast_pattern**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **uricontent**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **urilen**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **isdataat**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **pcre**

  Ключевое слово *pcre* позволяет использовать Perl-совместимые регулярные выражения [PCRE](http://www.pcre.org).

  ***Синтаксис***: `pcre:[!]"(/<regex>/|m<delim><regex><delim>)[ismxAEGRUBPHMCOIDKYS]";`

  ***Примеры***:

    ```
    alert tcp any any -> any 80 (content:"/foo.php?id="; pcre:"/\/foo.php?id=[0-9]{1,10}/iU";)
    ```

  ***Предупреждения***:

* **pkt_data**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **file_data**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **base64_decode**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **base64_data**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **byte_test**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **byte_jump**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **byte_extract**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **byte_math**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **ftpbounce**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **asn1**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:

* **cvs**

  ***Синтаксис***:

  ***Примеры***:

  ***Предупреждения***:



### Non-payload

* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****


### Post-detection

* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****
* ****

# Cameradar

<p align="center">
    <img src="images/Cameradar.gif" width="100%"/>
</p>

<p align="center">
    <a href="#license">
        <img src="https://img.shields.io/badge/license-Apache-blue.svg?style=flat" />
    </a>
    <a href="https://hub.docker.com/r/gokancik/cameradar/">
        <img src="https://img.shields.io/docker/pulls/gokancik/cameradar.svg?style=flat" />
    </a>
    <a href="https://travis-ci.org/gokancik/cameradar">
        <img src="https://travis-ci.org/gokancik/cameradar.svg?branch=master" />
    </a>
    <a href='https://coveralls.io/github/gokancik/cameradar?branch=master'>
        <img src='https://coveralls.io/repos/github/gokancik/cameradar/badge.svg?branch=master' alt='Coverage Status' />
    </a>
    <a href="https://golangci.com/r/github.com/gokancik/cameradar">
        <img src="https://golangci.com/badges/github.com/gokancik/cameradar.svg" />
    </a>
    <a href="https://goreportcard.com/report/github.com/gokancik/cameradar">
        <img src="https://goreportcard.com/badge/github.com/gokancik/cameradar" />
    </a>
    <a href="https://github.com/gokancik/cameradar/releases/latest">
        <img src="https://img.shields.io/github/release/gokancik/cameradar.svg?style=flat" />
    </a>
    <a href="https://godoc.org/github.com/gokancik/cameradar">
        <img src="https://godoc.org/github.com/gokancik/cameradar?status.svg" />
    </a>
</p>

## RTSP stream потоковы протокол со своей библиотекой

### Возможности Cameradar

* **Обнаружить открытые RTSP-хосты** на любом доступном целевом узле.
* Определить, какая модель устройства осуществляет потоковое вещание
* Запускать автоматический перебор по словарю для получения **маршрута потока** (например: `/live.sdp`)
* Запустить автоматические перебор по словарю для получения **имени пользователя и пароля** камер.
* Получить полный и удобный отчет о результатах.

<p align="center"><img src="images/Cameradar.png" width="250"/></p>

## Оглавление 

* [Docker Image](#docker-image)
* [Зависмости](#configuration)
* [Вывод](#output)
* [Параметры командной строки](#command-line-options)
* [Вклад](#contribution)
* [Часто задаваемые вопросы](#frequently-asked-questions)
* [Лицензия](#license)

## Docker для Cameradar

<p align="center"><img src="images/CameradarV4.png" width="70%"/></p>
* У
Устоновка [docker](https://docs.docker.com/engine/installation/) на свою машину и запустить команду:

```bash
docker run -t gokancik/cameradar -t <target> <other command-line options>
```

[Параметры командной строки](#command-line-options).

Пример: `docker run -t gokancik/cameradar -t 192.168.100.0/24` будет сканировать порты 554, 5554 и 8554 хостов в подсети 192.168.100.0/24 и перебирать обнаруженные RTSP-stream и выводить журнал отладки.
*YOUR_TARGET может быть подсетью (например: 172.16.100.0/24), IP (например: 172.16.100.10) или диапазоном IPs (например: 172.16.100.10-20).
* Если вы хотите получить точные результаты сканирования nmap в виде XML файла, вы можете добавить `-v /your/path:/tmp/cameradar_scan.xml` в команду docker run, перед `gokancik/cameradar`.
* Если вы используете `-r` и `-c` для указания ваших пользовательских словарей, убедитесь, что вы также используете  для добавления их в контейнер docker. Пример: `docker run -t -v /path/to/dictionaries/:/tmp/ gokancik/cameradar -r /tmp/myroutes -c /tmp/mycredentials.json -t mytarget`.

## Бинарная устоновка
Решение локального Camader без использование docker.



**WARNING**: Бинарная устновка **НЕ БУДЕТ РАБОТАТЬ** для камер, которые используют **DIGEST AUTHENTICATION** [если ваша версия `curl` старше `7.64.0`](https://github.com/gokancik/cameradar/pull/252), то скорее всего, так и есть. Дополнильная информация [this response on the subject from the author of curl](https://stackoverflow.com/a/59778142/4145098).

### Зависимости

* `go` (> `1.10`)
* `libcurl` библиотека (**[version has to be <7.66.0](https://github.com/gokancik/cameradar/issues/247)**)
* Для apt: `apt install libcurl4-openssl-dev`

### Этапы устоновки

1. `go install github.com/gokancik/cameradar/v5/cmd/cameradar@latest`

Бинарный файл `cameradar` находится здесь `$GOPATH/bin` и готов к использованию. Опции командной строки  [здесь](#command-line-options).

## Конфигурация 

Порт **RTSP, используемый для большинства камер, 554**, поэтому указываем  554 в качестве одного из сканируемых портов. Если не прописать порты в приложении cameradar, оно будет искать порты 554, 5554 и 8554.

`docker run -t --net=host gokancik/cameradar -p "18554,19000-19010" -t localhost`  будет сканировать порты `18554`, и диапазон портов между `19000` и `19010` на `localhost`.

Вы **можете использовать свои собственные файлы для словарей учетных данных и маршрутов**, используемых для атаки на камеры, но репозиторий Cameradar уже предоставляет вам хорошую базу, которая работает с большинством камер, в папке `/dictionaries`.

```bash
docker run -t -v /my/folder/with/dictionaries:/tmp/dictionaries \
gokancik/cameradar \
-r "/tmp/dictionaries/my_routes" \
-c "/tmp/dictionaries/my_credentials.json" \
-t 172.19.124.0/24
```

Это поместит содержимое вашей папки со словарями в образ докера и будет использовать его для перебора словаря, вместо словарей по умолчанию, предоставленных в репозитории cameradar.

## Доступ камеры 

Если у вас есть [VLC Media Playe](http://www.videolan.org/vlc/), у вас есть  возможность использовать графический интерфейс или командную строку для подключения к потоку RTSP, для этого используется этот формат `rtsp://username:password@address:port/route`.

## Опции командной строки 

* ** "-t, --targets "**: Устанавливаем цель.Целью может быть файл (см. [инструкции по форматированию файла](#format-input-file)), IP, диапазон IP, подсеть или их комбинация. Пример: `--targets="192.168.1.72,192.168.1.74"`
* ** "-p, --ports "**: (По умолчанию: `554,5554,8554`) Устанавливаем пользовательские порты.
* ** "-s, --scan-speed "**: (По умолчанию: `4`) Установка пользовательских настроек обнаружения nmap для повышения скорости или точности. Рекомендуется уменьшить это значение, если вы пытаетесь сканировать нестабильную и медленную сеть, или увеличить его, если вы работаете в очень производительной и надежной сети. Вы также можете оставить значение  низким, чтобы  уменшить вероятность обнуружения. Смотрите [здесь более подробная информация о шаблонах синхронизации nmap] (https://nmap.org/book/man-performance.html).
* ** "-I, --attack-interval "**: (По умолчанию: `0ms`) Задает пользовательский интервал, после которого попытка атаки без ответа должна прекратиться. Рекомендуется увеличить его при попытке сканирования нестабильных и медленных сетей или уменьшить в быстрых и надежных сетях.
* ** "-T, --timeout "**: (По умолчанию: `2000 мс`) Задает пользовательское значение тайм-аута, после которого попытка атаки без ответа должна прекратиться. Рекомендуется увеличить это значение при попытке сканирования нестабильных и медленных сетей или уменьшить его в быстрых и надежных сетях.
* ** "-r, --custom-routes "**: (По умолчанию: `<CAMERADAR_GOPATH>/dictionaries/routes`) Устанавливает путь к пользовательскому словарю для маршрутов.
* ** "-c, --custom-credentials "**: (По умолчанию: `<CAMERADAR_GOPATH>/dictionaries/credentials.json`) Устанавливает путь к пользовательскому словарю для учетных данных
* ** "-o, --nmap-output "**: (По умолчанию: `/tmp/cameradar_scan.xml`) устанавливаем путь вывода nmap.
* **"-d, --debug "**: Включаем журналы отладки
* **"-v, --verbose "**: Включаем  журналы curl (не рекомендуется для большинства пользователей)
* **"-h "**: Отображаем использованную информацию

## Форматирование входного файла 

Файл может содержать IP-адреса, имена хостов, диапазоны IP-адресов и подсети, разделенные на строки. Например:

```text
0.0.0.0
localhost
192.17.0.0/16
192.168.1.140-255
192.168.2-3.0-255
```

## Переменные среды

### `CAMERADAR_TARGET`

Эта переменная  обязательная и указывает цель, которую cameradar должен сканировать и пытаться получить доступ к потокам RTSP.

Например:

* `172.16.100.0/24`
* `192.168.1.1`
* `localhost`
* `192.168.1.140-255`
* `192.168.2-3.0-255`

### `CAMERADAR_PORTS`

Эта переменная не обязательная,нужна для того, чтоб указать сканируемые порты 

Значение по умолчанию  `554,5554,8554`

Рекомендуется не изменять их, кроме случаев, когда вы уверены, что камеры были настроены на передачу RTSP через другой порт. 99,9% камер передают поток через эти порты.

### `CAMERADAR_NMAP_OUTPUT_FILE`

Эта переменная является необязательной и позволяет указать, в какой файл nmap будет записывать свой вывод.

Значение по умолчанию: `/tmp/cameradar_scan.xml`

Это может быть полезно, только для самотятельного чтения файлов, если вы не хотите, чтобы он записывал данные в вашу папку `/tmp`, или если вы хотите использовать только функцию RunNmap в cameradar, и делать ее разбор вручную.

### `CAMERADAR_CUSTOM_ROUTES`, `CAMERADAR_CUSTOM_CREDENTIALS`

Эти переменные являются необязательными и позволяют заменить словари по умолчанию на пользовательские, Это нужно для перебора по словарю.

Значение по умолчанию: `<CAMERADAR_GOPATH>/dictionaries/routes` and `<CAMERADAR_GOPATH>/dictionaries/credentials.json`

### `CAMERADAR_SCAN_SPEED`

Эта опцианальная  переменная позволяет установить пользовательские настройки  обнаружения nmap для повышения скорости и точности. Рекомендуется уменьшить значение, если вы пытаетесь сканировать нестабильную и медленную сеть, или увеличить, если сеть быстрая и надежная. Дополнительные сведения о временных шаблонах nmap смотреть [здесь](https://nmap.org/book/man-performance.html).

Значение по умолчанию: `4`

### `CAMERADAR_ATTACK_INTERVAL`

Эта не обязательная переменная позволяет установить `custom interval` между каждой атакой, чтобы оставаться незаметным. Рекомендуется увеличивать его при попытке сканирования сети, которая может быть защищена от атак методом перебора. По умолчанию интервал отсутствует, чтобы сделать атаки как можно более быстрыми.

Значение по умолчанию: `0ms`

### `CAMERADAR_TIMEOUT`

Эта необязательная переменная дает возмоджность установить пользовательское значение тайм-аута, по истечении которого попытка атаки без ответа должна прекратиться. Рекомендуется увеличить это значение при попытке сканирования нестабильных и медленных сетей или уменьшить его в быстрых и надежных сетях.

Значение по умолчанию: `2000ms`

### `CAMERADAR_LOGGING`

Эта необязательная переменная позволяет вам включить более расширенный вывод, чтобы иметь больше информации о процессах.

Она будет выводить результаты nmap, запросы cURL и т.д.

Значение по умолчанию: `false`

## Вклад

### Сборка 

#### Docker build

Чтобы собрать образ докера, просто выполните команду `docker build . -t cameradar` в корне проекта.

Ваше изображение будет называться `cameradar`, а НЕ `gokancik/cameradar`.

#### Приступим к сборке 

1. `go get github.com/gokancik/cameradar`
2. `cd $GOPATH/src/github.com/gokancik/cameradar`
3. `cd cmd/cameradar`
4. `go install`

Бинарный файл cameradar теперь находится в `$GOPATH/bin/cameradar`.

## Часто задаваемые вопросы

> Cameradar не находит не одной камеры!

Это означает, что либо ваши камеры не передают поток в RTSP, либо их нет на объекте, который вы сканируете. В большинстве случаев камеры видеонаблюдения находятся в частной подсети, изолированной от Интернета. Используйте опцию `-t` для указания цели. Если вы уверены, что все сделали правильно, но это все равно не работает, пожалуйста, откройте проблему с подробной информацией об устройстве, к которому вы пытаетесь получить доступ.

> Cameradar нашел мои камеры, но не может получить доступ к ним 

Возможно, ваши камеры были настроены, и учетные данные / URL были изменены. Cameradar только угадывает, используя значения конструктора по умолчанию, если не предоставлен пользовательский словарь. Вы можете использовать свои собственные словари, в которые нужно просто добавить ваши учетные данные и маршруты RTSP. Для этого посмотрите, как работает [configuration](#configuration). Также, возможно, учетные данные вашей камеры еще не известны, в таком случае, если вы их найдете, было бы очень хорошо добавить их в словари Cameradar, чтобы помочь другим людям в будущем.

> Что случилось с версией на  C++?

Вы все еще можете найти его под тегом 1.1.4 в этом репозитории, однако он был медленнее и менее стабилен, чем текущая версия, написанная на Golang. Использовать ее не рекомендуется.

> Как использовать библиотеку Cameradar для моего собственного проекта?

Смотрите пример в `/cmd/cameradar`. Вам просто нужно запустить `go get github.com/gokancik/cameradar` и использовать пакет `cameradar` в своем коде. Документацию можно найти на [godoc](https://godoc.org/github.com/gokancik/cameradar).

> Я почему-то хочу просканировать свой собственный localhost, а он не работает! 

Используйте флаг `--net=host` при запуске образа cameradar, или используйте бинарный файл, выполнив команду `go run cameradar/cameradar.go` или [установив его](#go-build).

> Я не вижу цветного вывода :(

Вероятнее вы  забыли использовать флаг `-t` перед `gokancik/cameradar` в вашей командной строке. Это указывается  -tty для cameradar, что позволит ему использовать цвета.

> У меня нет камеры, но я хотел бы попробовать Cameradar!

Просто воспользуйтесь командой `docker run -p 8554:8554 -e RTSP_USERNAME=admin -e RTSP_PASSWORD=12345 -e RTSP_PORT=8554 ullaakut/rtspatt` и запустите cameradar, и он должен определить, что имя пользователя - admin, а пароль - 12345. Вы можете попробовать это с любыми учетными данными конструктора по умолчанию (их можно найти [здесь](dictionaries/credentials.json)).

> Какие типы аутентификации поддерживает Cameradar?

Cameradar поддерживает как базовую, так и дайджест-аутентификацию.


## Примеры 

>> Запуск cameradar на вашей собственной машине для сканирования портов по умолчанию

`docker run --net=host -t gokancik/cameradar -t localhost`

> Запуск cameradar с входным файлом, включение журналов на порту 8554

`docker run -v /tmp:/tmp --net=host -t gokancik/cameradar -t /tmp/test.txt -p 8554`

> Запуск cameradar в подсети с пользовательскими словарями, на портах 554, 5554 и 8554

`docker run -v /tmp:/tmp --net=host -t gokancik/cameradar -t 192.168.0.0/24 --custom-credentials="/tmp/dictionaries/credentials.json" --custom-routes="/tmp/dictionaries/routes" -p 554,5554,8554`.

## Лицензия 

Copyright 2023 Ullaakut

Настоящим предоставляется бесплатное разрешение любому лицу, получившему копию
данного программного обеспечения и сопутствующих файлов документации ("Программное обеспечение"), совершать сделки с  Программным обеспечением без ограничений, включая, без ограничения, права
использовать, копировать, изменять, объединять, публиковать, распространять, выдавать сублицензии и/или продавать
копии программного обеспечения, а также разрешать лицам, которым предоставляется Программное обеспечение
делать это, при соблюдении следующих условий:

Вышеуказанное уведомление об авторском праве и данное уведомление о разрешении должны быть включены во все
копиях или существенных частях Программного обеспечения.

ПРОГРАММНОЕ ОБЕСПЕЧЕНИЕ ПРЕДОСТАВЛЯЕТСЯ "КАК ЕСТЬ", БЕЗ КАКИХ-ЛИБО ГАРАНТИЙ, ЯВНЫХ ИЛИ
ПОДРАЗУМЕВАЕМЫХ, ВКЛЮЧАЯ, НО НЕ ОГРАНИЧИВАЯСЬ ГАРАНТИЯМИ ТОВАРНОГО СОСТОЯНИЯ,
ПРИГОДНОСТИ ДЛЯ КОНКРЕТНОЙ ЦЕЛИ И НЕНАРУШЕНИЯ ПРАВ. НИ ПРИ КАКИХ ОБСТОЯТЕЛЬСТВАХ
АВТОРЫ ИЛИ ВЛАДЕЛЬЦЫ АВТОРСКИХ ПРАВ НЕ НЕСУТ ОТВЕТСТВЕННОСТИ ЗА ЛЮБЫЕ ПРЕТЕНЗИИ, УБЫТКИ ИЛИ ДРУГУЮ
ОТВЕТСТВЕННОСТЬ, БУДЬ ТО В РАМКАХ ДОГОВОРНОГО, ДЕЛИКТНОГО ИЛИ ИНОГО ИСКА, ВОЗНИКАЮЩЕГО
ИЗ, В РЕЗУЛЬТАТЕ ИЛИ В СВЯЗИ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ ИЛИ ИСПОЛЬЗОВАНИЕМ ИЛИ ИНЫМИ
ИСПОЛЬЗОВАНИЕМ ИЛИ ДРУГИМИ ДЕЙСТВИЯМИ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ.

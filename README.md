# SocksFusion

SocksFusion - утилита для проброса TCP-сообщений по шифрованному туннелю сквозь сетевой экран, выглядящее для клиентских приложений как сервер Socks5. Состоит из двух компонентов: **Proxy**, устанавливаемый на нашей стороне и обеспечивающий идентификацию агентов, и **Agent**, распространяемое приложение, устанавливающее соединение с Proxy.

Приблизительная схема сетевых соединений:

                                     NAT
    +--------+         +-------+     | |     +-------+         +--------+
    | Target | <------ | Agent | ----|-|---> | Proxy | <------ | Client |
    +--------+         +-------+     | |     +-------+         +--------+


## Agent

Agent устанавливает соединение с Proxy по адресу proxy.bbs.ptsecurity.com:443 и пытается идентифицироваться по указанному ключу. Если попытка успешна, то пытается отвечать на проброшенные клиентские пакеты и открывать соединения с целью, запрошенной в соответствии с протоколом Socks5.

### Отладочные сообщения

В течение работы Агент может писать отладочные сообщения в stdout, сопровождаемые меткой времени:
    * `Connecting to server...` - пытаемся соединиться с Proxy.
    * `Connection established` - Агент установил TLS соединение с Proxy.
    * `This agent is not registered` - ключ агента не зарегистрирован в системе, скорее всего, была сделана ещё одна верификация.
    * `Another agent already connected` - с данным ключом уже работает другой агент. Агент будет пытаться переподключиться с тем же ключом каждые пять секунд.
    * `Scan is started` - пришёл запрос на сканирование, теперь Агент работает как Socks5 сервер.
    * `Running` - периодическое сообщение от Proxy, соединение в порядке.
    * `Target is unreachable: <address>` - не удаётся связаться с указанным адресом. Если это сообщение повторяется, то, возможно, что-то блокирует исходящие соединения или имеются неполадки в сети.

## Proxy

Служит прокси сервером для входящих соединений от агентов и клиентов (инстансов сканера). Устанавливает защищённое TLS соединение с агентами и в течение всего сканирования пробрасывает по нему сообщения от клиентов.

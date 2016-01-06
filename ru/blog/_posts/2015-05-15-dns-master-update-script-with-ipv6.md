---
title: Скрипт для обновления записи для IPv6 в DNS Master.
name:  dns-master-update-script-with-ipv6
---

Скрипт для обновления записи для IPv6 в DNS Master
==================================================

Не так давно я сменил интернет провайдера и мой ноутбук стал всё-таки счастливым обладателем белого IP-адреса. Правда IPv6-адреса, но это тоже здорово. На радостях я подключил себе услугу [DNS Master] от [RU Center], для того, чтобы пользоваться благами [динамического DNS][Dynamic DNS] и для IPv4, выдаваемого роутеру и для IPv6, которые роутер распределяет устройствам, к нему подключившимся (а ещё это раза так в три дешевле, чем заказывать статический IPv4 у провайдера).

У роутера поддержка DNS Master есть, что называется, «из коробки», а вот нативных клиентов для компьютеров и девайсов, похоже, что и нет совсем. Зато есть [API для разработчиков], достаточно простое, чтобы написать скрипт, который будет обновлять записи для компьютеров для меня.

Скачать: <https://gist.github.com/Envek/9aa53b06e7df369626a9>. Полный текст — далее.


Как пользоваться
----------------

Для начала необходимо зарегистрироваться на DNS Master, перевести свой домен под его управление и создать записи для динамического DNS (есть [инструкция][Настройка динамического DNS]).  Кратко: Необходимо создать A-запись (если вы используете IPv4) и AAAA-запись (если вы используете IPv6) для того домена, который будет указывать на ваш компьютер/девайс.

Вот пример записи для моего ноутбука (только IPv6, потому что в IPv4 он находится за NAT):

    notebook.envek.name.    IN    AAAA    ::1

`::1` в IPv6 — аналог `127.0.0.1` в IPv4. При работе динамического DNS будет заменён на настоящий адрес.

Сохраните [скрипт](https://gist.github.com/Envek/9aa53b06e7df369626a9) куда-нибудь (например в `/usr/local/bin/dnsmaster-update`) и убедитесь, что у него есть право на запуск:

```sh
chmod +x /usr/local/bin/dnsmaster-update
```

Попробуйте запустить один раз скрипт, чтобы убедиться, что всё верно:

```sh
/usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS
```

Все параметры обязательны:

 * `-h` или `--host` — имя хоста ([FQDN]). Запись типа A и/или AAAA, которая будет обновлена;
 * `-u` или `--user` — имя пользователя (смотрите [инструкцию][Настройка динамического DNS], чтобы узнать, как получить);
 * `-p` или `--password` — пароль к указанному имени пользователя.

В случае успеха скрипт должен вернуть что-то вроде:

    good 2001:db8::1dff:86e7:dad7:a218

Проверяем:

    $ host notebook.envek.name
    notebook.envek.name has IPv6 address 2001:db8::1dff:86e7:dad7:a218

Поместите регулярный вызов скрипта в Cron: `crontab -e`

```sh
*/15 * * * * /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS >>/tmp/dnsmaster-update.log 2>&1
@reboot      /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS > /tmp/dnsmaster-update.log 2>&1
```

В данном примере скрипт вызывается один раз после перезагрузки и потом каждые 15 минут. Вывод скрипта пишется в журнал во временном каталоге (обычно ОС очищает каталог после перезагрузки).

Если вы пользователь OS X, то вам придётся дописать в переменную `PATH` путь к каталогу, где находится утилита `ifconfig`, чтобы скрипт работал:

```sh
*/15 * * * * env PATH=$PATH:/sbin /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS >>/tmp/dnsmaster-update.log 2>&1
@reboot      env PATH=$PATH:/sbin /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS > /tmp/dnsmaster-update.log 2>&1
```

Всё! Теперь можно получить доступ к хосту `notebook.envek.name` (имя хоста ненастоящее). Да здравствует IPv6!


Что же делает скрипт?
---------------------

Скрипт делает всего три вещи:

 1. Узнаёт ваши публичные IPv4 и IPv6 адреса с помощью сервиса [WTF is my IP];
 2. Оставляет из полученных адресов только те, которые на самом деле есть на ваших сетевых интерфейсах;
 3. Отправляет запрос к API [DNS Master] для обновления записей.

Скрипт использует утилиту `ip` при работе в Linux или `ifconfig` при работе в OS X.

Если вас что-то не устраивает в работе скрипта — меняйте и используйте его как угодно, на условиях лицензии [MIT].

Скрипт
------

<script src="https://gist.github.com/Envek/9aa53b06e7df369626a9.js"></script>


Для дальнейшего изучения
------------------------

 * [Динамический DNS][Dynamic DNS]
 * [Настройка динамического DNS] у DNS Master
 * [API для разработчиков] от них же

[Dynamic DNS]: https://ru.wikipedia.org/wiki/Динамический_DNS
[RU Center]: https://www.nic.ru/
[DNS Master]: http://dns-master.ru/
[API для разработчиков]: http://nic.ru/dns/service/dns_hosting/dns_master/dynamic_dns_for_developers.html
[Настройка динамического DNS]: http://dns-master.ru/dynamic_dns/setup.html
[WTF is my IP]: https://wtfismyip.com/
[MIT]: http://opensource.org/licenses/MIT "Полный текст лицензии MIT"

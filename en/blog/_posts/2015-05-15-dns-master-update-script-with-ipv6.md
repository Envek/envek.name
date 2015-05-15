---
title: DNS Master update script with IPv6.
name:  dns-master-update-script-with-ipv6
---

DNS Master update script with IPv6
==================================

I have changed my ISP some days ago and since then my notebook have owned the public IP address. IPv6 address though, but that is nice too. I subscribed to [DNS Master] service from [RU Center] to use the [Dynamic DNS] for both router's IPv4 and home devices' IPv6 addresses. And that service is three times cheaper than ISP's static IPv4 address.

My router supports DNS Master “out of the box”, but it seems that there is no native clients for computers and devices. But there is [API for developers] that is simple enough to create simple script for updating dynamic DNS records.

Download: <https://gist.github.com/Envek/9aa53b06e7df369626a9>. Read further for full listing.


How to use
----------

First you need to register in the DNS Master, delegate your domain to be managed by it, and create records for dynamic DNS. ([manual] available).  In short: You need to create A record (if you are using IPv4) and AAAA record (if your are using IPv6) for the domain name that will locate to your device.

Example of such a record for my notebook (IPv6 only as it's located behind NAT in IPv4):

    notebook.envek.name.    IN    AAAA    ::1

`::1` in IPv6 is the same as `127.0.0.1` in IPv4. During dynamic DNS operation will be replaced by real address.

Save the [script](https://gist.github.com/Envek/9aa53b06e7df369626a9) somewhere (e.g. `/usr/local/bin/dnsmaster-update`) and ensure that it is executable:

```sh
chmod +x /usr/local/bin/dnsmaster-update
```

Try to execute script once to check that's everything is correct:

```sh
/usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS
```

All parameters are mandatory. They are:

 * `-h` or `--host` — hostname (FQDN). A and/or AAAA record to be updated;
 * `-u` or `--user` — dynamic DNS username (see [manual] for details how to get one);
 * `-p` or `--password` — dynamic DNS password for username given.

In case of success script should output something like:

    good 2001:db8::1dff:86e7:dad7:a218

Let's check!

    $ host notebook.envek.name
    notebook.envek.name has IPv6 address 2001:db8::1dff:86e7:dad7:a218

Execute this script on regular basis with Cron: `crontab -e`

```sh
*/15 * * * * /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS >>/tmp/dnsmaster-update.log 2>&1
@reboot      /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS > /tmp/dnsmaster-update.log 2>&1
```

In this example script launched one time after system startup and every 15 minutes consequently. Script output will be saved into the log file in OS temporary directory (it is usually cleared on each reboot).

If you are an OS X user you should to append path to directory with `ifconfig` utility to the `PATH` environment variable for script to work correctly under Cron:

```sh
*/15 * * * * env PATH=$PATH:/sbin /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS >>/tmp/dnsmaster-update.log 2>&1
@reboot      env PATH=$PATH:/sbin /usr/local/bin/dnsmaster-update -h notebook.envek.name -u DYNDNSUSER -p DYNDNSPASS > /tmp/dnsmaster-update.log 2>&1
```

That is all! Now I can access my notebook by name `notebook.envek.name` (this hostname is fake). Viva la IPv6!


What it is doing?
-----------------

Only three things:

 1. Get you public IPv4 и IPv6 addresses using [Telize] service;
 2. Uses only those that are present in your network interfaces;
 3. Sends request to [DNS Master] API to update records.

Script uses `ip` utility in Linux and `ifconfig` in OS X.

If you are missing some features or it's behaving not as you want — feel free to modify and use it as you wish under the terms of [MIT] license.


The script
----------

<script src="https://gist.github.com/Envek/9aa53b06e7df369626a9.js"></script>


Further reading
---------------

 * [Dynamic DNS]
 * [Dynamic DNS configuration][manual] for DNS Master (in Russian)
 * [API for developers] from DNS Master (in Russian)

[Dynamic DNS]: https://ru.wikipedia.org/wiki/Динамический_DNS
[RU Center]: https://www.nic.ru/
[DNS Master]: http://dns-master.ru/
[API for developers]: http://nic.ru/dns/service/dns_hosting/dns_master/dynamic_dns_for_developers.html
[manual]: http://dns-master.ru/dynamic_dns/setup.html
[Telize]: http://www.telize.com/
[MIT]: http://opensource.org/licenses/MIT

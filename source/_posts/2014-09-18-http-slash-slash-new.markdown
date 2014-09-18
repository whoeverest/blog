---
layout: post
title: "http://new"
date: 2014-09-18 14:42:15 +0200
comments: true
categories:
---

If you try to open [http://new](http://new) in your browser you expect to get a connection error of some sort. But that's not what happened to me. Instead the browser ended up serving me Apaches' default "It works!" page.

![http://new page](http://i.imgur.com/KvtrDDM.png)

Naturally, I got curious.

First, I checked if the browser was modifying the request:

```
who# curl http://new

<html><body><h1>It works!</h1>
<p>This is the default web page for this server.</p>
<p>The web server software is running but no content has been added, yet.</p>
</body></html>
```

That wasn't it. I checked the ip address:

```
who# ping new
PING new (127.0.53.53) 56(84) bytes of data.
^C64 bytes from 127.0.53.53: icmp_seq=1 ttl=64 time=0.054 ms

--- new ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.054/0.054/0.054/0.000 ms

```

Okay, so it's the Apache server on my own computer. (I learned every `127.x.x.x` address points to localhost.) But why does `new` resolve to localhost? It wasn't the `/etc/hosts` file:

```
who# cat /etc/hosts

127.0.0.1	localhost
127.0.1.1	who
```

I checked to see if I have a running local DNS server:

```
who# ps aux | grep dns

nobody    3016  0.0  0.0  35200   836 ?        S    сеп04   0:43 /usr/sbin/dnsmasq --no-resolv --keep-in-foreground --no-hosts --bind-interfaces --pid-file=/var/run/NetworkManager/dnsmasq.pid --listen-address=127.0.1.1 --conf-file=/var/run/NetworkManager/dnsmasq.conf --cache-size=0 --proxy-dnssec --enable-dbus=org.freedesktop.NetworkManager.dnsmasq --conf-dir=/etc/NetworkManager/dnsmasq.d
```

Just to be sure, I checked for open connections on UDP 53:

```
who# netstat -ulnp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name

udp        0      0 127.0.0.1:47146         0.0.0.0:*                           8744/skype
udp        0      0 127.0.1.1:53            0.0.0.0:*                           3016/dnsmasq
udp        0      0 0.0.0.0:68              0.0.0.0:*                           25372/dhclient
```

`dnsmasq` was indeed listening for UDP connections on port 53. At this point, I
suspected some process was dynamically assigning himself the `new`  hostname. My
main suspect was Vagrant. A friend pointed out that it might be possible for another
process to add a DNS records via `dbus` so we decided to investigate.

We tried using `qdbus` to try and communicate with `dnsmasq` but it didn't go very well:

```
who# qdbus --system org.freedesktop.NetworkManager.dnsmasq /uk/org/thekelleys/dnsmasq

method QString org.freedesktop.DBus.Introspectable.Introspect()
signal void org.freedesktop.NetworkManager.dnsmasq.DhcpLeaseAdded(QString ipaddr, QString hwaddr, QString hostname)
signal void org.freedesktop.NetworkManager.dnsmasq.DhcpLeaseDeleted(QString ipaddr, QString hwaddr, QString hostname)
signal void org.freedesktop.NetworkManager.dnsmasq.DhcpLeaseUpdated(QString ipaddr, QString hwaddr, QString hostname)
method void org.freedesktop.NetworkManager.dnsmasq.ClearCache()
method QString org.freedesktop.NetworkManager.dnsmasq.GetVersion()
method void org.freedesktop.NetworkManager.dnsmasq.SetDomainServers(QStringList servers)
method void org.freedesktop.NetworkManager.dnsmasq.SetServers(QVariantList servers)

zsh: segmentation fault (core dumped)  qdbus --system org.freedesktop.NetworkManager.dnsmasq 
```

I wasn't too eager to go into analysing why `qdbus` segfaulted, so I decided to
approach it another way. I should've tried this sooner:

```
who# dig new

; <<>> DiG 9.9.3-rpz2+rl.13214.22-P2-Ubuntu-1:9.9.3.dfsg.P2-4ubuntu1.1 <<>> new
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24344
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 5, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;new.				IN	A

;; ANSWER SECTION:
new.			3576	IN	A	127.0.53.53

;; AUTHORITY SECTION:
new.			4935	IN	NS	ns-tld1.charlestonroadregistry.com.
new.			4935	IN	NS	ns-tld2.charlestonroadregistry.com.
new.			4935	IN	NS	ns-tld3.charlestonroadregistry.com.
new.			4935	IN	NS	ns-tld4.charlestonroadregistry.com.
new.			4935	IN	NS	ns-tld5.charlestonroadregistry.com.

;; Query time: 12 msec
;; SERVER: 127.0.1.1#53(127.0.1.1)
;; WHEN: Thu Sep 18 14:32:25 CEST 2014
;; MSG SIZE  rcvd: 184
```

To my surprise, the domain appeared to exist globally. I followed
[charlestonroadregistry.com](http://charlestonroadregistry.com) which redirected me to
[google.com/registry/](http://www.google.com/registry/).

From the contents of the page it was obvious what was the matter: Google was
releasing top level domains soon and `.new` was one of them. Digging for `ads`
or `zip` answered with the same `127.0.53.53` address.

I was curious if `com` or `org` also had their own IP address:

```
who# dig com

; <<>> DiG 9.9.3-rpz2+rl.13214.22-P2-Ubuntu-1:9.9.3.dfsg.P2-4ubuntu1.1 <<>> com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7725
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;com.				IN	A

;; AUTHORITY SECTION:
com.			900	IN	SOA	a.gtld-servers.net. nstld.verisign-grs.com. 1411051494 1800 900 604800 86400

;; Query time: 17 msec
;; SERVER: 127.0.1.1#53(127.0.1.1)
;; WHEN: Thu Sep 18 16:45:20 CEST 2014
;; MSG SIZE  rcvd: 105
```

There was a record, but only `AUTHORITY SECTION:` was present, `ANSWER SECTION:` was
missing. I had no clue why they would differ. Then, another friend, who works as
a sys-admin, pointed me to [this ICANN announcement](https://www.icann.org/news/announcement-2-2014-08-01-en)
which reports:

> The framework is designed to mitigate the impact of name collisions in the domain name system (DNS), which typically occur when fully qualified domain names conflict with similar domain names used in private networks. When this occurs, users can be taken to an unintended Web page or encounter an error message.
> 
> To address this issue, the framework calls for registry operators to use a technique called "controlled interruption" to alert system administrators that there may be an issue in their network. Specifically, an IPv4 address – 127.0.53.53 – will appear in system logs, enabling a quick diagnosis and remediation.

So Google's domains, being within the 90-day "starting" period, were using `127.0.53.53` to warn
system administrators for possible incoming domain name collisions with their local
domain names. Which is why `dig com` doesn't return an IP address, while `new` does.

Which solves the mystery.

---

This whole adventure, BTW, started because I wanted to open `news.ycombinator.com`;
I wrote `new` in the address bar and hit Enter before Firefox had a chance to
autocomplete the URL.

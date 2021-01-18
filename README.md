# dnsmaqs_getting_started
## 1. Problem
The purpose of this guide is to quickly get you up and running with dnsmasq while understanding on a basic level what's happening so that you may troubleshoot if needed.

For this exercise we're going to configure your local macOS with a DNS server which you can use to setup your own environments and testing.

Essentially, any requests destined to `*.localhost` will go to your DNS server.

As an example were going to use the host `my_device` which will be configured under `.localhost` domain.
## 2. Setup
### 2.2 Configure dnsmasq
```
# Install trough brew
brew install dnsmasq --cask

# Backup your default config
mv /usr/local/etc/dnsmasq.conf /usr/local/etc/dnsmasq.conf.backup
```
Paste the follwing into `/usr/local/etc/dnsmasq.conf`.
```
# Allow DNS-server to reply to *.localhost
address=/.localhost/127.0.0.1

# Our test subject
address=/my_device.localhost/172.20.100.10

# Logging
log-queries
log-facility=/tmp/dnsmasq.log

```
Brief explenation of the above:
- First address row allows our DNS server to respond to anything with `.localhost` such as `my_car.localhost`, `home_router.localhost` etc.
- Second address is the example we're going to be using.
- Logging is essential to understand what's happening and how to troubleshoot.
### 2.3 Config macOS
Add a nameserver for the `.localhost` domain.
```
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/localhost'
```
Now check that the resolver is listed with `scuitl --dns`.
```
resolver #8
  domain   : localhost
  nameserver[0] : 127.0.0.1
  flags    : Request A records, Request AAAA records
  reach    : 0x00030002 (Reachable,Local Address,Directly Reachable Address)
```
Restart dnsmasq (press Allow on the popup).
```
sudo brew services restart dnsmasq
```
### 2.4 Verifying

I suggest you have 2 terminal windows here, one for the logs and another for execution of commands.

For the logging issue `sudo tail -f /tmp/dnsmasq.log`.

Ping `my_car.localhost`, check the logging and dig.
```
ping -c 1 my_car.localhost
PING my_car.localhost (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.042 ms

--- my_car.localhost ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.042/0.042/0.042/0.000 ms
```

```
Jan 18 21:02:03 dnsmasq[8341]: query[A] my_car.localhost from 127.0.0.1
Jan 18 21:02:03 dnsmasq[8341]: config my_car.localhost is 127.0.0.1
```

```
dig my_car.localhost @127.0.0.1

; <<>> DiG 9.10.6 <<>> my_car.localhost @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10473
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my_car.localhost.		IN	A

;; ANSWER SECTION:
my_car.localhost.	0	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon Jan 18 21:03:19 CET 2021
;; MSG SIZE  rcvd: 61
```

Now do the same but for the address object we've configured, `my_device`.

```
ping -c 1 my_device.localhost
PING my_device.localhost (172.20.100.10): 56 data bytes
^C
--- my_device.localhost ping statistics ---
1 packets transmitted, 0 packets received, 100.0% packet loss
```

```
Jan 18 21:07:36 dnsmasq[8989]: query[A] my_device.localhost from 127.0.0.1
Jan 18 21:07:36 dnsmasq[8989]: config my_device.localhost is 172.20.100.10
```

```
dig my_device.localhost @127.0.0.1

; <<>> DiG 9.10.6 <<>> my_device.localhost @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21355
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my_device.localhost.		IN	A

;; ANSWER SECTION:
my_device.localhost.	0	IN	A	172.20.100.10

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon Jan 18 21:08:49 CET 2021
;; MSG SIZE  rcvd: 64
```

From here you now have a base and know how to verify and troubleshoot, good luck and may the signal be with you!

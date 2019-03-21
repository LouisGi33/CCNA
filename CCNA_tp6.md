# **TP 6 - Une topologie qui ressemble un peu à quelque chose, enfin ?**

## **Lab 2 : Un peu de complexité (et d'utilité ?...)**

### **2. Mise en place du lab**

Apres la mise en place et la configuration de toutes les machines, les Clients peuvent bien se ping entre eux sans soucis mais le ping Client Server ne passe pas car les Routeurs ne font pas office de routage dynamique, ils ne font pas encore le "lien" entre l'Area 1 et l'Area 2.

#### **Configuration de OSPF**

 **Attention à la mise en place du partage des routes à bien indiquer la bonne Area concernée**
 
 Exemple : 
```
r1.tp6#show ip protocols
...
  Routing for Networks:
    10.6.100.0 0.0.0.3 area 0
    10.6.100.4 0.0.0.3 area 0
    10.6.202.0 0.0.0.255 area 2
```

On peut donc maintenant, apres avoir créer le lien entre les Area, ping(pong) le Server1 depuis les Clients
```
[root@Client1 loki]# traceroute Server1
traceroute to Server1 (10.6.202.10), 30 hops max, 60 byte packets
 1  gateway (10.6.201.254)  29.494 ms  29.472 ms  29.311 ms
 2  r3 (10.6.101.1)  50.457 ms  52.005 ms  53.094 ms
 3  r4 (10.6.100.13)  70.994 ms  70.956 ms  71.250 ms
 4  r1 (10.6.100.5)  92.199 ms  105.436 ms  105.417 ms
 5  Server1 (10.6.202.10)  54.310 ms !X  54.091 ms !X  53.523 ms !
```

## **Lab 3 : Let's end this properly**

### **1. NAT : accès internet**

Apres avoir galéré 20h avec le vmnet8 (cé supér), on a enfin acces au NAT

Apres la config R4, on peut donc ping google depuis Client1.
```
[root@Client1 loki]# traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  gateway (10.6.201.254)  15.990 ms  15.639 ms  16.304 ms
 2  r3 (10.6.101.1)  25.409 ms  35.269 ms  35.540 ms
 3  r4 (10.6.100.13)  55.828 ms  55.506 ms  55.449 ms
 4  192.168.122.1 (192.168.122.1)  65.669 ms  65.774 ms  65.276 ms
 5  10.0.3.2 (10.0.3.2)  65.345 ms  65.419 ms  65.164 ms
 6  10.0.3.2 (10.0.3.2)  65.532 ms  55.487 ms  54.751 ms
```

Ayant config le server DNS, on peut donc ping le domaine google.com

```
r1.tp6#ping google.com

Translating "google.com"...domain server (8.8.8.8) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 216.58.213.174, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 52/66/80 ms

```

### **2. Un service d'infra**

On arrive bien à ping google.com depuis le Server.
```
[root@Server1 /]# ping google.com
PING google.com (64.233.177.102) 56(84) bytes of data.
64 bytes from yx-in-f102.1e100.net (64.233.177.102): icmp_seq=1 ttl=33 time=265 ms
64 bytes from yx-in-f102.1e100.net (64.233.177.102): icmp_seq=2 ttl=33 time=282 ms
```

Apres avoir mis en place un simple site hébergé par le Server1, on peut curl le Server depuis le Client pour avoir le HTML.
```
[root@Client1 loki]# curl 192.168.251.6
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
...
```

### **3. Serveur DHCP**

Apres avoir modifié le Client2 en dhcp et précisé dans le Client 1 le DHCP dynamique, on se retrouve avec une nouvelle ip dans la plage défini dans le fichier dhcpd.conf
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a8:a7:f2 brd ff:ff:ff:ff:ff:ff
    inet 10.6.201.11/24 brd 10.6.201.255 scope global noprefixroute enp0s3
```

### **4. Serveur DNS**

On configure le Server1 comme etant le DNS.

Apres cette étape, nous pouvons voir les correspondances Nom de Domaine / IP : 
```
[root@Server1 loki]# dig Client1.tp6

...

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;Client1.tp6.                   IN      A

;; ANSWER SECTION:
Client1.tp6.            604800  IN      A       10.6.201.50

;; AUTHORITY SECTION:
tp6.                    604800  IN      NS      server1.tp6.

;; ADDITIONAL SECTION:
server1.tp6.            604800  IN      A       10.6.202.10

...
```
```
[root@Server1 loki]# dig -x 10.6.201.50

...

;; ANSWER SECTION:
50.201.6.10.in-addr.arpa. 86400 IN      PTR     client1.tp6.

;; AUTHORITY SECTION:
6.10.in-addr.arpa.      86400   IN      NS      server1.tp6.

;; ADDITIONAL SECTION:
server1.tp6.            604800  IN      A       10.6.202.10
```

On peut aussi ping avec seulement le nom de domaine sans avoir rempli le fichier hosts : 
```
[root@Server1 loki]# ping Client1
PING Client1.tp6 (10.6.201.50) 56(84) bytes of data.
64 bytes from client1.tp6 (10.6.201.50): icmp_seq=2 ttl=60 time=83.4 ms
64 bytes from client1.tp6 (10.6.201.50): icmp_seq=3 ttl=60 time=79.4 ms
```

### **5. Serveur NTP**

Server : 
```
[root@Server1 /]# chronyc sources && echo && chronyc tracking
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ x.ns.gin.ntt.net              2   8   377    88   -386us[ -344us] +/-   93ms
^+ v.bsod.fr                     2   8   377    89   +122us[ +164us] +/-   68ms
^* admin.remarche.fr             3   7   377    20   -414us[ -368us] +/-   53ms
^+ ntp-3.arkena.net              2   8   377    89  +1808us[+1850us] +/-   70ms


Reference ID    : 3ED2A7F8 (admin.remarche.fr)
Stratum         : 4
Ref time (UTC)  : Thu Mar 21 19:43:33 2019
System time     : 0.000188805 seconds fast of NTP time
Last offset     : +0.000046461 seconds
RMS offset      : 0.001144476 seconds
Frequency       : 3.757 ppm fast
Residual freq   : +0.016 ppm
Skew            : 2.625 ppm
Root delay      : 0.047007371 seconds
Root dispersion : 0.024508130 seconds
Update interval : 130.6 seconds
Leap status     : Normal
```

Client : 
```
[root@dhcp /]# chronyc sources && echo && chronyc tracking
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ 213.251.53.11                 2   9   377   352    -23ms[  -20ms] +/-  105ms
^* regar42.fr                    3   9   377   160  +4378us[+6597us] +/-   28ms
^- x.ns.gin.ntt.net              2   9   377   355  -6310us[-4085us] +/-  100ms
^+ ns0.serverhouse.com           2   9   377   161   -241us[+1977us] +/-   71ms


Reference ID    : 3ED2F492 (regar42.fr)
Stratum         : 4
Ref time (UTC)  : Thu Mar 21 19:41:50 2019
System time     : 0.001885715 seconds fast of NTP time
Last offset     : +0.002218653 seconds
RMS offset      : 0.001267187 seconds
Frequency       : 5.066 ppm fast
Residual freq   : -0.007 ppm
Skew            : 1.989 ppm
Root delay      : 0.056438819 seconds
Root dispersion : 0.000480148 seconds
Update interval : 257.4 seconds
Leap status     : Normal
```
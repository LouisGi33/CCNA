# **TP 5 - Premier pas dans le monde Cisco**

*Configuration de toutes les machines*

## **II. Lancement et configuration du lab**

### **Checklist IP VMs**

Deja fait dans les anciens TP je vais pas remontrer les cmd classiques

### **Checklist IP Routeurs**

**Config IP Statique:**

Router1:
```
#show ip interface brief
(config)# interface ethernet 0/0
(config-if)# ip address 10.5.1.254 255.255.255.0
no shut (pour relancer)
exit
(config)# interface ethernet 0/1
(config-if)# ip address 10.5.3.1 255.255.255.252
no shut
```

Router2:
```
(config)# interface ethernet 0/0
(config-if)# ip address 10.5.3.2 255.255.255.252

(config)# interface ethernet 0/1
(config-if)# ip address 10.5.2.254 255.255.255.0
```
Choix d'une Addr /30 entre les 2 routeurs car que 2 machines pas besoin de plus

**Config hostname CISCO:**

```
conf t
hostname router1/2.tp5
```

### **Checklist routes**

Router1: Ajout reseau Client via Router2
```
conf t
ip route 10.5.2.0 255.255.255.0 10.5.3.2
```

Router2: Ajout reseau Server via Router1
```
ip route 10.5.1.0 255.255.255.0 10.5.3.1
```


---

*Remplissage fichier hosts (deja vu ancien tp)*

Server: Ajout reseau Client via Router1
```
nano /etc/sysconfig/network-script/route-enp0s3
10.5.2.0/24 via 10.5.1.254 dev enp0s3
systemctl restart network
```

Clients: Ajout reseau Server via Router2
```
10.5.1.0/24 via 10.5.2.254 dev enp0s3
```

On se retrouve avec les configs suivantes:

| Machine | Net1 | Net2 | Net12
| -------- | -------- | -------- | --------
| Client1| X | `10.5.2.10` | X
| Client2 | X | `10.5.2.11` | X
| Router1 | `10.5.1.254` | X | `10.5.3.1`
| Router2 | X | `10.5.2.254` | `10.5.3.2`
| Server | `10.5.1.10` | X | X


Test de ping : 

Client1
```
[root@Client1 /]# traceroute server1
traceroute to server1 (10.5.1.10), 30 hops max, 60 byte packets
 1  router2 (10.5.2.254)  14.088 ms  24.456 ms  24.122 ms
 2  * * *
 3  server1 (10.5.1.10)  42.830 ms !X  42.935 ms !X  42.539 ms !X
```

Client2
```
[root@Client2 /]# traceroute server1
traceroute to server1 (10.5.1.10), 30 hops max, 60 byte packets
 1  router2 (10.5.2.254)  9.330 ms  9.569 ms  9.256 ms
 2  * * *
 3  server1 (10.5.1.10)  37.980 ms !X  37.593 ms !X  37.023 ms !X
```

Server
```
[root@Server1 network-scripts]# traceroute client1
traceroute to client1 (10.5.2.10), 30 hops max, 60 byte packets
 1  router1 (10.5.1.254)  7.358 ms  6.919 ms  6.485 ms
 2  * * *
 3  client1 (10.5.2.10)  35.917 ms !X  35.472 ms !X  35.130 ms !X
```

La seconde étape (les * * *) représente le passage sur le réseau privé entre le Router 1 et le Router 2

## **III. DHCP**

*Config DHCP*

Renouvellement bail DHCP sur Client1
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:01:9c:ba brd ff:ff:ff:ff:ff:ff
    inet 10.5.2.51/24 brd 10.5.2.255 scope global dynamic enp0s3
```
Comme convenu, la nouvelle Addr est bien dans la Plage défini par le DHCP (entre .50 et .70)

Capture : 
![](https://i.imgur.com/dFZEdtp.png)

On y retrouve comme d'habitude les demandes ARP pour savoir ou trouver le bon récepteur en y précisant qui est le destinataire

Puis un protocole CDP (Protocole CISCO représentant le Router2)

Et enfin les requetes DORA.

## **IV. Bonus**

Apres le téléchargement de Nginx, l'ouverture du port 80 (http) et avoir lancé nginx sur le Server, le Client arrive a se connecter
```
[root@Client1 loki]# curl server1
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
    ............
```

**Chiffrement en suivant ce tuto :**

https://technique.arscenic.org/connexion-distante-au-serveur-ssh/article/securisation-ssh-poussee-authentification-par-cle-rsa

Suite à ca on tente de se connecter en SSH au Server par le Client1: 

`==== AUTHENTICATION COMPLETE ===`

Mais en testant de se connecter par le Client2:

`Permission denied (publickey,gssapi-keyex,gssapi-with-mic).`

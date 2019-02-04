# **TP 4 - Spéléologie réseau : descente dans les couches**
## **I. Mise en place du lab**
### **3. Mise en place du routage statique**

**2-Client**

Ajouter les routes manquantes à l'accès au server :

`10.2.0.254/24 via 10.1.0.254 dev enp0s8`

`10.2.0.0/24 via 10.2.0.254 dev enp0s8`

**3-Server**

Ajouter les routes manquantes à l'accès au client:

`10.1.0.254/24 via 10.2.0.254 dev enp0s8`

`10.1.0.0/24 via 10.1.0.254 dev enp0s8`

**4- Test:**

Traceroute de Client vers Server : 
```
[root@localhost network-scripts]# traceroute 10.2.0.10
traceroute to 10.2.0.10 (10.2.0.10), 30 hops max, 60 byte packets
 1  router.tp4 (10.1.0.254)  0.341 ms  0.213 ms  0.138 ms
 2  server.tp4 (10.2.0.10)  0.407 ms !X  0.374 ms !X  0.269 ms !X
```

Traceroute de Server vers Client : 
```
[root@localhost network-scripts]# traceroute 10.1.0.10
traceroute to 10.1.0.10 (10.1.0.10), 30 hops max, 60 byte packets
 1  router.tp4 (10.2.0.254)  0.628 ms  0.392 ms  0.413 ms
 2  client.tp4 (10.1.0.10)  0.906 ms !X  0.895 ms !X  1.050 ms !X
```

## **II. Spéléologie réseau**

### **1. ARP**

#### **A. Manip 1**
*Vidage table ARP*

2,3- Client, Server: la seule ligne représente la carte réseau, soit la connexion en SSH entre la VM et le terminal SSH et indique l'ID de la carte réseau (Addr IP, MAC, son nom) qu'utilise la VM pour cette connexion

4,5- 

Client:
```
[root@localhost etc]# ip neigh show
10.1.0.254 dev enp0s8 lladdr 08:00:27:d5:c4:bc STALE
10.1.0.1 dev enp0s8 lladdr 0a:00:27:00:00:4c REACHABLE
```

Server:
```
[root@localhost etc]# ip n s
10.2.0.254 dev enp0s8 lladdr 08:00:27:fd:be:14 STALE
10.2.0.1 dev enp0s8 lladdr 0a:00:27:00:00:51 REACHABLE
```

La ligne qui s'est rajoutée représente la connexion établie du ping *(pong)* par l'intermédiaire d'un Router faisant office de pont d'accès *(donc enregistré dans la table ARP)*.

#### **B. Manip 2**

2- Router: La seule ligne disponible représente la carte réseau par lequel est connecté le SSH (si ssh ouvert avec 10.1.0.254, alors carte réseau utilisé est 10.1.0.1 / si ouvert avec 10.2.0.254 alors carte réseau 10.2.0.1)

```
[root@localhost loki]# ip n s
10.2.0.1 dev enp0s9 lladdr 0a:00:27:00:00:51 REACHABLE
```
*Connecté par la carte réseau 10.2.0.1 dans cet exemple*

3,4-
```
[root@localhost loki]# ping 10.2.0.10
PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=2.53 ms
```
*Ping depuis client*

```
[root@localhost loki]# ip neigh show
10.1.0.10 dev enp0s8 lladdr 08:00:27:64:20:37 STALE
10.2.0.1 dev enp0s9 lladdr 0a:00:27:00:00:51 REACHABLE
10.2.0.10 dev enp0s9 lladdr 08:00:27:93:d6:4b STALE
```
Le Router se voit rajouter 2 nouvelles ARP correspondant au Client et au Server lors du ping etabli (packet: Client->(recup ARP Client)->Router->(recup ARP Server)->Server)

#### **C. Manip 3**
PC Hote: 
Quand on vide la table ARP sur notre Hote (le PC), celle ci ne se retrouve pas vide car elle contient les ARP des cartes réseaux des connexions établies (2 cartes Res des VM (due au SSH), ma carte réseau (connecté au Rooter de Ynov(Qui apparait peu de temps apres le vidage de la table ARP)), ma carte npcap..)

La table des ARP se remplit petit à petit du coté du réseau Ynov (10.33.1.87) par toutes les requettes qui sont faites sur ce réseau et qui demande à mon ordi si je suis bien le récepteur de tel ou tel packet(Message ARP Broadcast -> Ajout de IPAddr, MAC.. dans ma table ARP et celle de ceux qui emettent les packets)

#### **D. Manip 4**
```
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
```
Chaque VM de l'émulateur (VirtualBox) s'exécute derrière un service de routeur / pare-feu virtuel qui l'isole des interfaces et paramètres réseau du PC Hote

Un périphérique émulé ne peut pas voir notre PC ou les autres VM.
Au lieu de cela, il voit seulement qu'il est connecté via Ethernet à un routeur / pare-feu à l'adresse 10.0.2/24

### **2. Wireshark**

#### **A. Interception d'ARP et ping**
3- Pour retirer le fichier/dossier d'une VM depuis Hote :
```
**************scp user@host:/chemin/VM C:\chemin\**************
scp loki@10.1.0.254:/home/loki/ping.pcap C:\Users\loulo\Desktop
```

4- Ayant mis la carte réseau 10.2.0.1 sous écout, ce que l'on voit sont les requetes faites entre le Broadcast (en 10.2.0.254) et le Server (en 10.2.0.10) tous deux sur la carte réseau 10.2.0.1, il nous manque donc le chemin entre Client (10.1.0.10) et Router coté Client ( carte réseau 10.1.0.1 dont son Boadcast 10.1.0.254)

il aurait fallut mettre la 10.1.0.1 sous ecout pour avoir la Capture réseau de ce chemin.

![](https://i.imgur.com/mDhLulM.png)

#### **B. Interception d'une communication netcat**
3-way handshake:

Le client qui désire établir une connexion avec un serveur va envoyer un premier paquet SYN (synchronized) au serveur.
```
Flags: 0x002 (SYN)
```
Le serveur va répondre au client à l'aide d'un paquet SYN-ACK (synchronize, acknowledge).
```
Flags: 0x012 (SYN, ACK)
```
Pour terminer, le client va envoyer un paquet ACK au serveur qui va servir d'accusé de réception.
```
Flags: 0x010 (ACK)
```
Le client va enfin pouvoir envoyer le message grace à un paquet PSH-ACK 
Le serveur lui envoie un "accusé de réception" avec un paquet ACK
![](https://i.imgur.com/pHkKvTB.png)
*Le message est bien transmit*

**Fermeture du port:**
Demande du Client d'une connexion sur le port 8888 en TCP/SYN
![](https://i.imgur.com/GGmqF0p.png)
*Bloquage -> ICMP Type: 3 (Destination unreachable), Code: 10 (Host administratively prohibited)*
Impossibilité de créer la connexion sur le port 8888 etant fermé.
La conversation tcp prend fin

#### **C. Interception d'un trafic HTTP (BONUS)**

**Interception du trafic**

Dans un premier temps, on peut observer la recherche faite sur internet

![](https://i.imgur.com/De4mEtF.png)

Puis la detection de la requete ARP qui se fait entre le Router (MAC : *PcsCompu_fd:be:14*) et le Server (MAC: *PcsCompu93:d6:4b*)

Le Router inscrit le Server dans sa table ARP

![](https://i.imgur.com/Y9TZEyb.png)

On y retrouve le Three-way Handshake (SYN -> SYN-ACK -> ACK):

![](https://i.imgur.com/MtYPkZC.png)

L'envoi de la page se fait par un packet PSH-ACK (`Flags: 0x018 (PSH, ACK)`) pour envoyer le fichier Http (Frame 9)

La réception se fait Frame 12 avec la confirmation de réception du fichier HTML (lisible depuis Wireshark dans le packet TCP/HTTP)

![](https://i.imgur.com/3diljdN.png)

Les 2 machines mettent fin à leur conversation par un paquet FIN-ACK

![](https://i.imgur.com/mN5piYJ.png)

Après cet échange, le Server inscrit cette fois ci le Router dans sa table ARP

![](https://i.imgur.com/RWmjbfT.png)
##### **OS : Windows**

# **I. Exploration locale en solo**
---

### **1. Affichage d'informations sur la pile TCP/IP locale**

**CMD**

PowerShell -> ipconfig /all

**Interface WiFi:**

Nom : Intel(R) Dual Band Wireless-AC 8260

Adresse MAC : EA-BA-83-25-27-16

Ip : 192.168.1.18

Adresse réseau(bin) : 1100.0000 1010.1000 0000.0001 0000.0000

Adresse réseau(dec) : 192.168.1.0

Adresse Broadcast : 192.168.1.255

**Interface Ethernet:**

Nom : Realtek PCIe GBE Family Controller

Adresse MAC : D0-17-C2-1F-35-B0

Ip : Non connecté

Adresse réseau : Aucune, non connecté

Adresse Broadcast : Aucune, non connecté

**Gateway / Passerelle:** 192.168.1.1

**GUI:**

Paramètres réseau et internet -> Propriétés réseau (tout est dedans)

**La gateway dans le réseau d'Ingésup ?**

Passerelle entre le réseau interne (Celui d'Ynov) et le réseau externe (Par ex : Internet)


### **2. Modifications des informations**
Adresse Ip Dispo : 192.168.1.3 a 192.168.1.254 (Enlever 192.168.1.18 car occupé par mon ordi)

(.0 etant le routeur, .1 la passerelle et .255 le broadcast)

![](https://i.imgur.com/jq9OcLh.png)

*Pour le changement d'Ip locale, ca se passe ici !*


Utilisez nmap pour scanner le réseau de votre carte WiFi et trouver une adresse IP libre:
![](https://i.imgur.com/20XadNN.png)

*Graph du réseau, tout ceux montrés sont indispo*

**C. Modification d'adresse IP - pt. 2**

Changement Adresse Ip -> Connexion marche

Changement Passerelle -> Connexion crash

# **II. Exploration locale en duo**
---

### **5. Petit chat privé ?**
![](https://i.imgur.com/MjleL3N.png)

*Creation d'un Chat a partir de netcat depuis 2 terminaux sur le meme ordi*







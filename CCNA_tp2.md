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
Paramètres réseau et internet -> Propriété réseau (tout est dedans)

**La gateway dans le réseau d'Ingésup ?**

Passerelle entre le réseau interne (Celui d'Ynov) et le réseau externe (Par ex : Internet)


### **2. Modifications des informations**
Adresse Ip Dispo : 192.168.1.2 a 192.168.1.254 (Enlever 192.168.1.18 car occupé par mon ordi)
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


# **III. Manipulations d'autres outils/protocoles côté client**
---

### **1. DHCP**

![](https://i.imgur.com/ek6iNan.png)

*Adresse IP du serveur DHCP du réseau WiFi*

![](https://i.imgur.com/eu7Yxg3.png)

*Date d'expiration du bail DHCP*

**Nouvelle adresse IP:**

![](https://i.imgur.com/XhDgDHn.png)

*Reinitialise le contenu du cache DNS*

![](https://i.imgur.com/v6ON0O8.png)

*Libere la configuration DHCP et libere la configuration d’adresse IP de toutes les cartes réseaux*

![](https://i.imgur.com/YmZLRI3.png)

*Renouvelle la configuration DHCP de toutes les cartes*


### **2. DNS**

![](https://i.imgur.com/BibhRls.png)

*Adresse IP du serveur DNS (Sur les routeurs au particulier c'est généralement la Box qui fait office du meilleur DNS, mais peut etre changé (DNSBench)*


![](https://i.imgur.com/i5qVS79.png)

![](https://i.imgur.com/z3sc6Ab.png)

nslookup : Permet de transmettre l'Adresse IP lié à un nom de domaine (meme si pour Ynov, l'adresse IP semble incorrecte)

Reverse lookup : Suffit de faire la cmd nslookup avec cette fois ci une IP et non un nom de domaine.

![](https://i.imgur.com/ODshq9C.png)

![](https://i.imgur.com/qSYUjjq.png)

*Permet de connaitre le nom du serveur DNS*
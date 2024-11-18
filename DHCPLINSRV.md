<div align="center"><H1> Quête DHCP Linux </H1></div>

_________________
## Pré-requis techniques

> Une VM Debian Server 12 Bookworm  
> Une VM client Ubuntu 24.04 LTS  
> Un accès internet sur la VM pour la mise à jour des paquets et l'installation de isc-dhcp-server  

_________________
## Configurer la carte réseau de la machine virtuelle en Réseau Interne
_________________

- Une fois la VM Serveur installée, la sélectionner puis cliquer sur **Configuration** :


- Cliquer sur la rubrique **Réseau** :


- Dans cette partie, sélectionner la carte souhaitée **1**, ensuite penser à activer la carte **2** et enfin sélectionner le **Mode : réseau interne** :

_________________
## Configurer le service DHCP
_________________

**Configuration IPv4 SRVDEBIAN12 fixe**
_________________

- Pour définir l'adresse IPv4 fixe du serveur, il faut éditer le fichier **/etc/network/interfaces** :

```bash
nano /etc/network/interfaces
```

- Ajouter (et/ou modifier les lignes si déjà présentes) les lignes suivantes :

```bash
allow-hotplug enp0s8
iface enp0s8 inet static
  address 172.20.0.2
  gateway 172.20.0.1
```

- Une fois les lignes ajoutées et/ou modifiées, taper la commande suivante pour redémarrer la carte réseau et acutaliser son adresse IPv4 attribuée :

```bash
sudo systemctl restart networking.service ; sudo ifup enp0s8
```

**Installer le service DHCP**
_________________

- Dans un premier temps, il faut mettre à jour les paquets système :

```bash
apt-get update  
```

- Ensuite il faut installer le paquet suivant :

```bash
apt-get install isc-dhcp-server
```

- Une fois installé, il faut éditer le fichier suivant :

```bash
nano /etc/default/isc-dhcp-server
```

- Dans ce fichier, à la ligne ci-dessous, il faut indiquer quelle interface réseau nous allons utiliser :

```bash
INTERFACESv4=""
```

- Dans mon cas, l'interface réseau est la suivante (vous pouvez la trouver avec un ip a dans le terminal de votre machine) :

```bash
INTERFACESv4="enp0s8"
```

- Dans le prochain fichier à éditer, nous allons indiquer les configurations IP à fournir :

```bash
nano /etc/dhcp/dhcpd.conf
```

- Liste des configurations IP à ajouter au fichier :

```bash
# Notre configuration pour le réseau 172.20.0.0
subnet 172.20.0.0 netmask 255.255.255.0 {
range 172.20.0.100 172.20.0.200;
default-lease-time 600;
max-lease-time 7200;
}
```

- Une fois les lignes ajoutées, on redémarre le service dhcp :

```bash
service isc-dhcp-server restart
```

**Test du DHCP**
_________________

- Commençons par éditer le fichier suivant sur le client :

```bash
nano /etc/network/interfaces
```

- Une fois dans l'éditeur, rentrez les lignes ci-dessous :

```bash
# The primary network interface
allow-hotplug enp0s8
iface enp0s8 inet dhcp
```

- Après l'enregistrement et la fermeture du fichier, relancer l'interface réseau :

```bash
ifdown enp0s8
ifup enp0s8
```

- Le client a bien obtenu une addresse IP dans la plage du serveur DHCP :

![IP_PAR_DHCP](https://github.com/Skchaper/DHCPLinux/blob/main/PHOTOS_QUETE_DHCP_LINUX/IP_A/IP_PAR_DHCP.png)


_________________
## Mettre en place une attribution statique pour une machine cliente particulière dont l'adresse MAC permet d'obtenir l'adresse 172.20.0.10
_________________

- Sur la VM serveur, éditer le fichier suivant :

```bash
nano /etc/dhcp/dhcpd.conf
```

- Et y ajouter les lignes suivantes :

```bash
host CLILINDHCP {
hardware ethernet 08:00:27:9a:3d:f5;
fixed-address 172.20.0.10;
}
```

- Une fois le fichier enregistré et fermé, redémarrer le service DHCP : 

```bash
service isc-dhcp-server restart
```

- Puis effectuez les commandes suivantes sur le client : 

```bash
sudo ifdown enp0s8
```
```bash
sudo ifup enp0s8
```

- Le client a bien reçu l'adresse IP statique attribuée à son adresse MAC :

![IP_PAR_MAC.png](https://github.com/Skchaper/DHCPLinux/blob/main/PHOTOS_QUETE_DHCP_LINUX/IP_A/IP_PAR_MAC.png)

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
## Configurer l'interface réseau du serveur
_________________

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
allow-hotplug enp0s3
iface enp0s3 inet static
  address 192.168.100.2/24
  gateway 192.168.100.1
```

- Une fois les lignes ajoutées et/ou modifiées, taper la commande suivante pour redémarrer la carte réseau et acutaliser son adresse IPv4 attribuée :

```bash
sudo systemctl restart networking.service ; sudo ifup enp0s3
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
INTERFACESv4="enp0s3"
```

- Dans le prochain fichier à éditer, nous allons indiquer les configurations IP à fournir :

```bash
nano /etc/dhcp/dhcpd.conf
```

```bash
# Notre configuration pour le réseau 192.168.100.0
subnet 192.168.100.0 netmask 255.255.255.0 {
range 192.168.100.150 192.168.100.200;
default-lease-time 600;
max-lease-time 7200;
}
```

```bash
service isc-dhcp-server restart
```

**Test du DHCP**
_________________

```bash
nano /etc/network/interfaces
```

```bash
# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
```

```bash
ifdown eth0
ifup eth0
```

_________________
## Mettre en place une attribution statique pour une machine cliente particulière dont l'adresse MAC permet d'obtenir l'adresse 172.20.0.10
_________________

_________________
## Tester le bon fonctionnement du serveur avec un client classique et le client devant avoir une adresse statique
_________________


Poste une procédure au format markdown permettant pas à pas d'obtenir cette configuration ainsi que les tests associés

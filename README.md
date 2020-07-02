# Roger-Skyline-1

Roger-Skyline-1 est un projet qui a pour but de nous initier aux bases de l'administration système et réseau, et ainsi créer et configurer un serveur web de type *Debian*.
<br/><br/><br/><br/>


### Debian ISO

https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.4.0-amd64-netinst.iso
<br/><br/><br/>

### Installation de la VM

* Name : debian
* Type : Linux
* Version : Debian (64-bit)

Create a virtual hard disk:
* VDI, Dynamically allocated, 7.45 GiB = 8 GB
  **/!\ VirtualBox displays GB while in reality it uses GiB**
    
Partition :
* Manual
* 4.2 GB on Mount point
  **/!\ Shows as 4.2G with `sudo fdisk -l --bytes` /!\ `sudo fdisk -l` gives results in GiB, not GB**

Choose a root password<br/>
Create a non-root user
<br/><br/><br/>

### Setup

Log in as the non-root user
`su` to log in as root

install sudo
`apt-get install sudo -y`
open file
`vim /etc/sudoers`
Add to the file :
```````````````````````````````````````````````````````````````````````````````
username	ALL(ALL:ALL) ALL
```````````````````````````````````````````````````````````````````````````````
or:
`adduser $username sudo`

You can now exit to go back to your non-root user and use sudo when you need root privileges
<br/><br/><br/>

----------------------------------------------------------------------------------------------------------------------

### 1) Désactiver le service DHCP de la machine et donner une IP fixe et un Netmask en/30

* [DHCP](Dynamic Host Configuration Protocol) : Protocole reseau dont le role est d'assurer la configuration automatique des parametres IP d'une station ou d'une machine, notamment en lui attribuant automatiquement une adresse IP et un masque de sous-reseau.

[Configuration de l'adresse IP pour la rendre fixe]:

* Ouvrir le fichier /etc/network/interfaces -> `sudo vim /etc/network/interfaces`
* Modifier ces lignes<br/>
```````````````````````````````````````````
  auto eth0
  iface eth0 inet dhcp
```````````````````````````````````````````

Par<br/>
```````````````````````````````````````````
  iface enp0s3 inet static
  adress 10.11.35.63
  netmask 255.255.255.252
  gateway 10.11.254.254
```````````````````````````````````````````

* Redemarrer le serivce réseau -> `sudo service networking restart`
<br/><br/><br/>

### 2) Changer le port par defaut du service SSH par celui de notre choix. L’accès SSH doit se faire avec des publickeys. L’utilisateur root ne doit pas pouvoir se connecter en SSH

* [SSH](Secure SHell) : est à la fois un programme informatique et un protocole de communication sécurisé.

[Changer le port par defaut du service SSH et interdire l'utilisateur root de se connecter en SSH]:

* Ouvrir le fichier /etc/network/sshd_config -> `sudo vim /etc/ssh/sshd_config`
* Changer le port par défaut SSH (22):
Port 22 -> Port 1992 (par exemple)

(les ports disponibles sont au nombre de 65 536(2^16), 16 premiers bits en partant de droite.
Les ports utilisés par défaut par le systeme sont de 0 a 1023, 22 étant le port par defaut du service SSH).

interdire l'utilisateur root de se connecter en SSH
* PermitRootLogin without-password ->	`PermitRootLogin no`

[Accès SSH avec publickey]
(A effectuer sur la machine locale)
* Generer une paire de clés SSH -> `ssh-keygen`
* Copier la cle sur le serveur pour l'autoriation -> `ssh-copy-id moalgato@10.11.35.63 -p 1992`
* Taper le mot de passe

* Ouvrir le fichier sshd_config -> `sudo vim /etc/ssh/sshd_config`
* PasswordAuthentication yes ->
``````````````````````````
PasswordAuthentication no (aprés établissement de la connexion afin d'empecher toutes nouvelles connexions)
``````````````````````````
* PubkeyAuthentication yes->
``````````````````````````
PubkeyAuthentication yes
``````````````````````````
<br/><br/>
* Redemarrer le service -> `sudo sevice ssh restart`
<br/><br/><br/>

### 3) Mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM

* Installer iptables-persistent
(version qui permet de rendre les modifications persistante) ->	`sudo apt-get install iptables-persistent`

* Ouvrir le fichier rules.v4 /etc/iptables/rules.v4 -> `sudo vim /etc/iptables/rules.v4`
et configurer le firewall en y ajoutant les lignes suivantes :

```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
*filter

#    -A => append Ajoiute une regle
#    -j => ce quíl faut faire si le paquet correspond a la rele


#Pour faire un flush (vide toutes les chaines (INPUT, OUTPUT, FORWARD), revient a effacer toutes les règles une par ue)
-F
-X

#Les politques (les regles generales) en loccurence interdire toute les entrees, les sorties, et les redirection de fichier
-P OUTPUT DROP
-P INPUT DROP
-P FORWARD DROP

#Connecxion etablie
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#Refus des paquets invalides
#(qui ne peut pas etre identifie ou qui a un etat inconnu, il est preferable de supprimer tout ce qui se trouve dans cet etat)
-A INPUT -m state --state INVALID -j DROP
-A FORWARD -m state --state INVALID -j DROP
-A OUTPUT -m state --state INVALID -j DROP

#Autorisation du loopback (communication des élémentau sein du serveur, ex: PHP communique avec l'IP de MySQL)
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

####################################################################
#Connexions autorisees (ouverture des ports):


#-port http necessaire pour telecharger les paquets
-A OUTPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT

#-port DNS necessaire pour telecharger les paquets
#(utilise le protocole UDP jusqu'a 512 octets puis utilise le TCP. Les deux protocoles ont un port 53)
#DNS est utilise uniquement en port de sortie
-A OUTPUT -p tcp --dport 53 -j ACCEPT
-A OUTPUT -p udp --dport 53 -j ACCEPT

#-Autorisation de SSH
-A INPUT -p tcp --dport 1992 -j ACCEPT
####################################################################

#Active la journalisation sur les chaînes d'entrée et de transfert d'IPtables af que le démon PSAD puisse détecter toute activité anorm
-A INPUT -j LOG
-A FORWARD -j LOG

COMMIT
```````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````````
<br/><br/><br/>

### 4) Mettre en place une protection contre les DOS (Denial Of ServiceAttack) sur les ports ouverts de la VM

[Mettre en place une protection contre les DOS (Denial Of ServiceAttack)]

[Fail2ban] : Fail2ban est un framework de prévention contre les intrusions, Fail2ban bloque les adresses IP appartenant à des hôtes qui tentent de casser la sécurité du système, pendant une période configurable (mise en quarantaine).

* Installer fail2ban -> `sudo apt-get install fail2ban`

* Copier le fichier jail.conf -> `sudo cp jail.conf jail.local (dans /etc/fail2ban/)`
(le fichier jail.conf sera modifier à chaque mis à jour, donc pour un fichier fixe, en créer un autre (jail.local)).

* Y ajouter/modifier ces lignes:
```````````````````````````````````````````````````````````````````````````````````````````````
# "bantime" is the number of seconds that a host is banned.
bantime  = 10m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 3


action = %(action_mwl)s


[sshd]

enabled = true
port    = 1992
logpath = %(sshd_log)s
backend = %(sshd_backend)s


[apache-auth]

enabled = true
port     = http,https
logpath  = %(apache_error_log)s
```````````````````````````````````````````````````````````````````````````````````````````````
<br/><br/><br/>

### 5) Mettre en place une protection contre les scans sur les ports ouverts de la VM

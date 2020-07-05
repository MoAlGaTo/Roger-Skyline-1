# Roger-Skyline-1

**Roger-Skyline-1** est un projet qui a pour but de nous initier aux bases de l'administration système et réseau, et ainsi créer et configurer un serveur web de type *Debian*.

**Roger-Skyline-1** is a project that aims to introduce us to the basics of system and network administration, and thus create and configure a *Debian* web server.
Some Markdown text with 
<div class="text-purple">
  This text is purple, <a href="#" class="text-inherit">including the link</a>
</div>

<br/><br/><br/><br/>
***


### Debian ISO

https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.4.0-amd64-netinst.iso
<br/><br/><br/>

### Installation de la VM - VM Setup

* Nom - Name : debian
* Type : Linux
* Version : Debian (64-bit)

Créer un disque dur virtuel - Create a virtual hard disk:
* VDI, Dynamically allocated, 7.45 GiB = 8 GB
  **/!\ VirtualBox displays GB while in reality it uses GiB**
    
Partition :
* Manual
* 4.2 GB on Mount point
  **/!\ Shows as 4.2G with `sudo fdisk -l --bytes` /!\ `sudo fdisk -l` gives results in GiB, not GB**
  ![linux - linux](/Screenshots/fdisk_-l.png)
  ***

* Choisir un mot de passe root - Choose a root password
* Créer un utilisateur non-root - Create a non-root user
<br/><br/><br/><br/><br/>

### Sudo

Se connecter en tant qu'utilisateur non-root - Log in as the non-root user
Pour se connecter en tant que root - to log in as root : `su`

installer sudo - install sudo : `apt-get install sudo -y`
ouvrir le fichier - open file : `vim /etc/sudoers`
Ajouter au fichier - Add to the file :
```````````````````````````````````````````````````````````````````````````````
username	ALL(ALL:ALL) ALL
```````````````````````````````````````````````````````````````````````````````
![linux - linux](/Screenshots/sudo_file.png)
***
ou - or: `adduser $username sudo`

On peut maintenant retourner à l'utilisateur non-root `su $utilisateurNonRoot` et utiliser sudo quand on a besoin des privilèges root.<br/>
We can now exit to go back to non-root user `su $nonRootUser` and use sudo when we need root privileges.
<br/><br/><br/><br/><br/><br/><br/>

***

### 1) Désactiver le service DHCP de la machine et donner une IP fixe et un Netmask en /30<br/>Disable the DHCP service of the machine and give a fixed IP and a Netmask in /30

* [DHCP](Dynamic Host Configuration Protocol) : Protocole reseau dont le role est d'assurer la configuration automatique des parametres IP d'une station ou d'une machine, notamment en lui attribuant automatiquement une adresse IP et un masque de sous-reseau - Network protocol whose role is to ensure the automatic configuration of the IP parameters of a station or a machine, in particular by automatically assigning an IP address and a subnet mask to it.

[Configuration de l'adresse IP pour la rendre fixe - Configuring the IP address to make it fixed]:

* Ouvrir le fichier - Open this file : /etc/network/interfaces -> `sudo vim /etc/network/interfaces`
* Modifier ces lignes - Edit these lines<br/>
```````````````````````````````````````````
  auto eth0
  iface eth0 inet dhcp
```````````````````````````````````````````

Par - With<br/>
```````````````````````````````````````````
  iface enp0s3 inet static
  adress 10.11.35.63
  netmask 255.255.255.252
  gateway 10.11.254.254
```````````````````````````````````````````
![linux - linux](/Screenshots/dhcp.png)
***
<br/>
* Redemarrer le serivce réseau - Restart the network service -> `sudo service networking restart`
<br/><br/><br/><br/><br/>

### 2) Changer le port par defaut du service SSH par celui de notre choix. L’accès SSH doit se faire avec des publickeys. L’utilisateur root ne doit pas pouvoir se connecter en SSH<br/>Change the default port of the SSH service to the port of our choice. SSH access must be done with publickeys. The root user must not be able to connect in SSH

* [SSH](Secure SHell) : est à la fois un programme informatique et un protocole de communication sécurisé - is both a computer program and a secure communication protocol.

(les ports disponibles sont au nombre de 65 536(2^16), 16 premiers bits en partant de droite.
Les ports utilisés par défaut par le systeme sont de 0 a 1023, 22 étant le port par defaut du service SSH).<br/>
(the available ports are 65,536(2^16), first 16 bits from right.
The default ports used by the system are from 0 to 1023, 22 being the default port for the SSH service).

* Ouvrir le fichier - Open this file : /etc/network/sshd_config -> `sudo vim /etc/ssh/sshd_config`
* Changer le port par défaut SSH (22) - Change the default SSH port (22): Port 22 -> Port 1992 (par exemple - for exemple) :
```````````````````````````
port 1992
```````````````````````````
* Interdire l'utilisateur root de se connecter en SSH - Prohibit the root user from logging in SSH :
```````````````````````````
PermitRootLogin no
```````````````````````````

[Accès SSH avec une publickey - SSH access with a publickey]
(A effectuer sur la machine locale - To be done on the local machine)
* Génerer une paire de clés SSH - Generate SSH key pair -> `ssh-keygen`
* Copier la cle sur le serveur pour l'autoriation - Copy the key to the server for authorization -> `ssh-copy-id moalgato@10.11.35.63 -p 1992`
* Taper le mot de passe

* Ouvrir le fichier sshd_config - Open the sshd_config file -> `sudo vim /etc/ssh/sshd_config`
* PasswordAuthentication yes ->
``````````````````````````
PasswordAuthentication no
``````````````````````````
(aprés établissement de la connexion afin d'empecher toutes nouvelles connexions)<br/>
after the connection has been established in order to prevent any new connections<br/>
* PubkeyAuthentication yes ->
``````````````````````````
PubkeyAuthentication yes
``````````````````````````
![linux - linux](/Screenshots/ssh.png)
***
<br/>
* Redemarrer le service ssh - Restart the ssh service -> `sudo sevice ssh restart`
<br/><br/><br/><br/><br/>

### 3) Mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM<br/>Set up firewall rules on the server with only used services accessible outside the VM

* Installer iptables-persistent - Install iptables-persistent<br/>
(version qui permet de rendre les modifications persistante)<br>
(version that allows you to make changes persistent) ->	`sudo apt-get install iptables-persistent`

* Ouvrir le fichier - Open this file : /etc/iptables/rules.v4 -> `sudo vim /etc/iptables/rules.v4`
et configurer le firewall en y ajoutant les lignes suivantes - and configure the firewall by adding the following lines :

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
![linux - linux](/Screenshots/iptables.png)
***
<br/>
* Redémmarer le service - Restart the service -> `sudo service netfilter-persistent restart`

* Vérifier le chargement des modifications - Check the loading of changes -> `sudo iptables -L`

<br/><br/><br/><br/><br/>

### 4) Mettre en place une protection contre les DOS (Denial Of ServiceAttack) sur les ports ouverts de la VM<br/>Implement DOS (Denial Of ServiceAttack) protection on open ports of the VM

[Fail2ban] :<br/>
Fail2ban est un framework de prévention contre les intrusions, Fail2ban bloque les adresses IP appartenant à des hôtes qui tentent de casser la sécurité du système, pendant une période configurable (mise en quarantaine).<br/>
Fail2ban is an intrusion prevention framework, Fail2ban blocks IP addresses belonging to hosts that attempt to breach system security, for a configurable period of time (quarantine).

* Installer fail2ban - Install fail2ban -> `sudo apt-get install fail2ban`

* Copier le fichier jail.conf - Copy the jail.conf file -> `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
(le fichier jail.conf sera modifier à chaque mis à jour, donc pour un fichier fixe, en créer un autre (jail.local)).<br/>
(the jail.conf file will be modified at each update, so for a fixed file, create another one (jail.local)).

* Ouvrir le fichier - Open this file : `sudo vim /etc/fail2ban/jail.local` et y ajouter/modifier ces lignes - and add/modify these lines :
```````````````````````````````````````````````````````````````````````````````````````````````
# "bantime" is the number of seconds that a host is banned.
bantime  = 10m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 3
```````````````````````````````````````````````````````````````````````````````````````````````
![linux - linux](/Screenshots/fail2ban_1.png)
***
```````````````````````````````````````````````````````````````````````````````````````````````
action = %(action_mwl)s


[sshd]

enabled = true
port    = 1992
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```````````````````````````````````````````````````````````````````````````````````````````````
![linux - linux](/Screenshots/fail2ban_2.png)
***
```````````````````````````````````````````````````````````````````````````````````````````````
[apache-auth]

enabled = true
port     = http,https
logpath  = %(apache_error_log)s
```````````````````````````````````````````````````````````````````````````````````````````````
![linux - linux](/Screenshots/fail2ban_3.png)
***
<br/>
* Redémmarer le service - Restart the service -> `sudo service fail2ban restart`
<br/><br/><br/><br/><br/>

### 5) Mettre en place une protection contre les scans sur les ports ouverts de la VM<br/>Set up scan protection on open ports of the VM

[Scan]:<br/>
permet de trouver dans un délai très court, tous les ports ouverts sur une machine distante.<br/>
allows you to find all open ports on a remote machine in a very short period of time.

[PSAD : Port Scan Attack Detector]

* Installer PSAD - Install PSAD -> `sudo apt-get install psad`

* Ajouter au fichier IPtables ces deux lignes - Add these two lines to the IPtables file : `sudo vim /etc/iptables/rules.v4` ->
````````````````````````
-A INPUT -j LOG
-A FORWARD -j LOG
````````````````````````
(Active la journalisation sur les chaînes d'entrée et de transfert d'IPtables afin que le démon PSAD puisse détecter toute activité anormale.)<br/>(Enables logging on IPtables input and transfer strings so that the PSAD daemon can detect any abnormal activity).

* Ouvrir le fichier de configuration - Open the configuration file /etc/psad/psad.conf -> `sudo vim /etc/psad/psad.conf`

* Modifier les lignes suivantes - Modify the following lines :
````````````````````````
EMAIL_ADDRESSES		root@localhost;
HOSTNAME		localhost;
````````````````````````
![linux - linux](/Screenshots/psad_1.png)
***

* Modifier ceci pour pointer vers le fichier syslog, où psad aura réellement la possibilité de parcourir les journaux actifs - Modify this to point to the syslog file, where psad will actually be able to browse the active logs :
````````````````````````
ENABLE_SYSLOG_FILE	Y;
IPT_WRITE_FWDATA	Y;
IPT_SYSLOG_FILE		/var/log/syslog;
````````````````````````
![linux - linux](/Screenshots/psad_2.png)
***

* Activer les paramètres suivants pour activer la fonction IPS et le niveau de danger. <br/>Après avoir activé le paramètre dans le fichier de configuration, le démon PSAD bloquera automatiquement l'attaquant en ajoutant son adresse IP dans les chaînes IPtables - Activate the following parameters to activate the IPS function and the danger level. <After enabling the parameter in the configuration file, the PSAD daemon will automatically block the attacker by adding his IP address in the IPtable strings :
````````````````````````
ENABLE_AUTO_IDS		Y;
AUTO_IDS_DANGER_LEVEL	1;
````````````````````````
![linux - linux](/Screenshots/psad_3.png)
***
<br/>
* Exécuter maintenant la commande suivante pour mettre à jour la base de données de signatures pour la détection des attaques - Now run the following command to update the signature database for attack detection -> `psad --sig-update`

* Redemarrer le service - Restart the service -> `sudo psad -R` (restart)

* Afficher l'etat de tous les proccessus en cours - Show status of all running processes -> `sudo psad -S` (status)
<br/><br/><br/><br/><br/>

### 6) Arretez les services non nécessaire pour ce projet

* Lister les services disponibles et répertorie <br/>l'état des services contrôlés par le systeme -> `sudo systemctl list-units --type=service --state=active`

![linux - linux](/Screenshots/systemctl.png)
***
<br/>
* Stopper un service -> `sudo systemctl disable $nameofservice`

Désactiver les services estimé non necéssaires :

* console-setup.service : Ce paquet fournit à la console le même modèle
de configuration du clavier que celui du système X Window.

* keyboard-setup.service : configuration du clavier.

* exim4 : Mail Transfert Agent (postfix est utilisé à la place).
<br/><br/><br/><br/><br/>

### 7) Réalisez un script qui met à jour l’ensemble des sources de package, puis de vos packages et qui log l’ensemble dans un fichier nommé /var/log/update_script.log. Créez une tache planifiée pour ce script une fois par semaine à 4h00 du matin età chaque reboot de la machine

[Cron] : est un programme qui permet aux utilisateurs des systèmes Unix
d’exécuter automatiquement des scripts, des commandes ou des logiciels à une
date et une heure spécifiée à l’avance.

* Créer le fichier /var/log/update_script.log -> `sudo touch /var/log/update_script.log`
* Créer un fichier dans /etc/cron.d -> `sudo touch /etc/cron.d/update_packages`
* Donner les droits d'execution au root -> `sudo chmod 744 /etc/cron.d/update_packages`
* Ouvrir le fichier avec vim -> `sudo vim /etc/cron.d/update_packages` puis ecrire ce script :

``````````````````````````````
#!/bin/bash

apt-get update && ((date && apt-get -y upgrade; echo) >> /var/log/update_script.log 2>&1)
``````````````````````````````
![linux - linux](/Screenshots/cron_script_1.png)
***

* Ouvrir le fichier crontab ->	`sudo crontab -e`

* Y ajouter les lignes suivantes :
``````````````````````````````
0 4 * * 5 /etc/cron.d/update_packages
@reboot /etc/cron.d/update_packages
``````````````````````````````
![linux - linux](/Screenshots/cron_1.png)
***
<br/><br/><br/><br/><br/>

### 8) Réalisez un script qui permet de surveiller les modifications du fichier /etc/crontab et envoie un mail à root si celui-ci a été modifié. Créez une tache plannifiée pour script tous les jours à minuit

* Créer un fichier dans /etc/cron.d dans lequel on stocke ce que renvoie la commande md5sum -> `sudo /etc/cron.d/touch cron_old_hash`
* Créer un fichier dans /etc/cron.d dans lequel on ecrira le script -> `sudo touch /etc/cron.d/cron_file_control`
* Donner les droits d'execution au root -> `chmod 744 /etc/cron.d/cron_file_control`
* Ouvrir le fichier avec vim -> `sudo vim /etc/cron.d/cron_file_control` puis ecrire ce script :

``````````````````````````````
#!/bin/bash

cron_old_hash=`cat /etc/cron.d/cron_old_hash`;
cron_new_hash=`md5sum /etc/crontab`;

if [ "$cron_old_hash" != "$cron_new_hash" ];
then
	md5sum /etc/crontab > /etc/cron.d/cron_old_hash;
	echo "WARNING - CRONTAB FILE WAS MODIFIED !!!" | mail -s "Warning - crontab file modification" root@localhost;
fi
``````````````````````````````
![linux - linux](/Screenshots/cron_script_2.png)
***

* Ouvrir le fichier crontab ->	`sudo crontab -e`

* Y ajouter la ligne suivante :
``````````````````````````````
0 0 * * * /etc/cron.d/cron_file_control
``````````````````````````````
![linux - linux](/Screenshots/cron_2.png)
***
<br/><br/><br/><br/><br/>


# Roger-Skyline-1

### Français:

**Roger-Skyline-1** est un projet qui a pour but de nous initier aux bases de l'administration système et réseau, et ainsi créer, installer et configurer un serveur web de type *Debian*.

Les consignes du projet sont décrites ci-dessous (ou dans rs1_subject_fr.pdf), ainsi que les différentes procédures.

### English:

**Roger-Skyline-1** is a project that aims to introduce us to the basics of system and network administration, and thus create, install and configure a *Debian* web server.

The project guidelines are described below (or in rs1_subject_fr.pdf (in French)), as well as the different procedures.

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

* <img src="Screenshots/france.png" width="10" height="15"/> Se connecter en tant qu'utilisateur non-root - <img src="Screenshots/united-states.png" width="10" height="15"/> Log in as the non-root user
* <img src="Screenshots/france.png" width="10" height="15"/> Pour se connecter en tant que root - <img src="Screenshots/united-states.png" width="10" height="15"/> To log in as root : `su`

* <img src="Screenshots/france.png" width="10" height="15"/> Installer sudo - <img src="Screenshots/united-states.png" width="10" height="15"/> Install sudo : `apt-get install sudo -y`
* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Open file : `vim /etc/sudoers`
* <img src="Screenshots/france.png" width="10" height="15"/> Ajouter au fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Add to the file :
```````````````````````````````````````````````````````````````````````````````
username	ALL(ALL:ALL) ALL
```````````````````````````````````````````````````````````````````````````````
![linux - linux](/Screenshots/sudo_file.png)
***
* Ou - Or: `adduser $username sudo`

<img src="Screenshots/france.png" width="10" height="15"/> On peut maintenant retourner à l'utilisateur non-root `su $utilisateurNonRoot` et utiliser sudo quand on a besoin des privilèges root.<br/><br/>
<img src="Screenshots/united-states.png" width="10" height="15"/> We can now exit to go back to non-root user `su $nonRootUser` and use sudo when we need root privileges.
<br/><br/><br/><br/><br/><br/><br/>

***

### 1) <img src="Screenshots/france.png" width="10" height="15"/> Désactiver le service DHCP de la machine et donner une IP fixe et un Netmask en /30<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Disable the DHCP service of the machine and give a fixed IP and a Netmask in /30


* [DHCP](Dynamic Host Configuration Protocol) :<br/>
<img src="Screenshots/france.png" width="10" height="15"/> Protocole reseau dont le role est d'assurer la configuration automatique des parametres IP d'une station ou d'une machine, notamment en lui attribuant automatiquement une adresse IP et un masque de sous-reseau<br/>
<img src="Screenshots/united-states.png" width="10" height="15"/> Network protocol whose role is to ensure the automatic configuration of the IP parameters of a station or a machine, in particular by automatically assigning an IP address and a subnet mask to it.

[<img src="Screenshots/france.png" width="10" height="15"/> Configuration de l'adresse IP pour la rendre fixe - <img src="Screenshots/united-states.png" width="10" height="15"/> Configuring the IP address to make it fixed]:

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Open this file : /etc/network/interfaces -> `sudo vim /etc/network/interfaces`
* Modifier ces lignes - Edit these lines<br/>
```````````````````````````````````````````
  auto eth0
  iface eth0 inet dhcp
```````````````````````````````````````````

<img src="Screenshots/france.png" width="10" height="15"/> Par - <img src="Screenshots/united-states.png" width="10" height="15"/> With<br/>
```````````````````````````````````````````
  iface enp0s3 inet static
  adress 10.11.35.63
  netmask 255.255.255.252
  gateway 10.11.254.254
```````````````````````````````````````````
![linux - linux](/Screenshots/dhcp.png)
***

<br/>

* <img src="Screenshots/france.png" width="10" height="15"/> Redemarrer le serivce réseau - <img src="Screenshots/united-states.png" width="10" height="15"/> Restart the network service -> `sudo service networking restart`
<br/><br/><br/><br/><br/>

### 2) <img src="Screenshots/france.png" width="10" height="15"/> Changer le port par defaut du service SSH par celui de notre choix. L’accès SSH doit se faire avec des publickeys. L’utilisateur root ne doit pas pouvoir se connecter en SSH<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Change the default port of the SSH service to the port of our choice. SSH access must be done with publickeys. The root user must not be able to connect in SSH


* [SSH](Secure SHell) :<br/> <img src="Screenshots/france.png" width="10" height="15"/> est à la fois un programme informatique et un protocole de communication sécurisé<br/> <img src="Screenshots/united-states.png" width="10" height="15"/> is both a computer program and a secure communication protocol.

<img src="Screenshots/france.png" width="10" height="15"/> (les ports disponibles sont au nombre de 65 536(2^16), 16 premiers bits en partant de droite.
Les ports utilisés par défaut par le systeme sont de 0 a 1023, 22 étant le port par defaut du service SSH).<br/>
<img src="Screenshots/united-states.png" width="10" height="15"/> (the available ports are 65,536(2^16), first 16 bits from right.
The default ports used by the system are from 0 to 1023, 22 being the default port for the SSH service).

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Open this file : /etc/network/sshd_config -> `sudo vim /etc/ssh/sshd_config`
* <img src="Screenshots/france.png" width="10" height="15"/> Changer le port par défaut SSH (port 22) - <img src="Screenshots/united-states.png" width="10" height="15"/> Change the default SSH port (port 22) :
```````````````````````````
port 1992
```````````````````````````
* <img src="Screenshots/france.png" width="10" height="15"/> Interdire l'utilisateur root de se connecter en SSH - <img src="Screenshots/united-states.png" width="10" height="15"/> Prohibit the root user from logging in SSH :
```````````````````````````
PermitRootLogin no
```````````````````````````

[<img src="Screenshots/france.png" width="10" height="15"/> Accès SSH avec une publickey - <img src="Screenshots/united-states.png" width="10" height="15"/> SSH access with a publickey]
(<img src="Screenshots/france.png" width="10" height="15"/> A effectuer sur la machine locale - <img src="Screenshots/united-states.png" width="10" height="15"/> To be done on the local machine)
* <img src="Screenshots/france.png" width="10" height="15"/> Génerer une paire de clés SSH - <img src="Screenshots/united-states.png" width="10" height="15"/> Generate SSH key pair -> `ssh-keygen`
* <img src="Screenshots/france.png" width="10" height="15"/> Copier la cle sur le serveur pour l'autoriation - <img src="Screenshots/united-states.png" width="10" height="15"/> Copy the key to the server for authorization -> `ssh-copy-id moalgato@10.11.35.63 -p 1992`
* <img src="Screenshots/france.png" width="10" height="15"/> Entrer le mot de passe - <img src="Screenshots/united-states.png" width="10" height="15"/> Enter the password

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier sshd_config - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the sshd_config file -> `sudo vim /etc/ssh/sshd_config`
* PasswordAuthentication yes ->
``````````````````````````
PasswordAuthentication no
``````````````````````````
(<img src="Screenshots/france.png" width="10" height="15"/> aprés établissement de la connexion afin d'empecher toutes nouvelles connexions)<br/>
(<img src="Screenshots/united-states.png" width="10" height="15"/> after the connection has been established in order to prevent any new connections)<br/>
* PubkeyAuthentication yes ->
``````````````````````````
PubkeyAuthentication yes
``````````````````````````
![linux - linux](/Screenshots/ssh.png)
***

<br/>

* <img src="Screenshots/france.png" width="10" height="15"/> Redemarrer le service ssh - <img src="Screenshots/united-states.png" width="10" height="15"/> Restart the ssh service -> `sudo service ssh restart`
<br/><br/><br/><br/><br/>

### 3) <img src="Screenshots/france.png" width="10" height="15"/> Mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Set up firewall rules on the server with only used services accessible outside the VM


* <img src="Screenshots/france.png" width="10" height="15"/> Installer iptables-persistent - <img src="Screenshots/united-states.png" width="10" height="15"/> Install iptables-persistent<br/>
(<img src="Screenshots/france.png" width="10" height="15"/> version qui permet de rendre les modifications persistante)<br>
(<img src="Screenshots/united-states.png" width="10" height="15"/> version that allows you to make changes persistent) ->	`sudo apt-get install iptables-persistent`

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Open this file : /etc/iptables/rules.v4 -> `sudo vim /etc/iptables/rules.v4` <br/>
<img src="Screenshots/france.png" width="10" height="15"/> et configurer le firewall en y ajoutant les lignes suivantes - <img src="Screenshots/united-states.png" width="10" height="15"/> and configure the firewall by adding the following lines :

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

* <img src="Screenshots/france.png" width="10" height="15"/> Redémmarer le service - <img src="Screenshots/united-states.png" width="10" height="15"/> Restart the service -> `sudo service netfilter-persistent restart`

* <img src="Screenshots/france.png" width="10" height="15"/> Vérifier le chargement des modifications - <img src="Screenshots/united-states.png" width="10" height="15"/> Check the loading of changes -> `sudo iptables -L`

<br/><br/><br/><br/><br/>

### 4) <img src="Screenshots/france.png" width="10" height="15"/> Mettre en place une protection contre les DOS (Denial Of ServiceAttack) sur les ports ouverts de la VM<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Implement DOS (Denial Of ServiceAttack) protection on open ports of the VM


[Fail2ban] :<br/>
<img src="Screenshots/france.png" width="10" height="15"/> Fail2ban est un framework de prévention contre les intrusions, Fail2ban bloque les adresses IP appartenant à des hôtes qui tentent de casser la sécurité du système, pendant une période configurable (mise en quarantaine).<br/>
<img src="Screenshots/united-states.png" width="10" height="15"/> Fail2ban is an intrusion prevention framework, Fail2ban blocks IP addresses belonging to hosts that attempt to breach system security, for a configurable period of time (quarantine).

* <img src="Screenshots/france.png" width="10" height="15"/> Installer fail2ban - <img src="Screenshots/united-states.png" width="10" height="15"/> Install fail2ban -> `sudo apt-get install fail2ban`

* <img src="Screenshots/france.png" width="10" height="15"/> Copier le fichier jail.conf - <img src="Screenshots/united-states.png" width="10" height="15"/> Copy the jail.conf file -> `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`<br/>
(<img src="Screenshots/france.png" width="10" height="15"/> le fichier jail.conf sera modifier à chaque mis à jour, donc pour un fichier fixe, en créer un autre (jail.local)).<br/>
(<img src="Screenshots/united-states.png" width="10" height="15"/> the jail.conf file will be modified at each update, so for a fixed file, create another one (jail.local)).

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Open this file : `sudo vim /etc/fail2ban/jail.local` et y ajouter/modifier ces lignes - and add/modify these lines :
```````````````````````````````````````````````````````````````````````````````````````````````
bantime  = 10m
findtime  = 10m
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

* <img src="Screenshots/france.png" width="10" height="15"/> Redémmarer le service - <img src="Screenshots/united-states.png" width="10" height="15"/> Restart the service -> `sudo service fail2ban restart`
<br/><br/><br/><br/><br/>

### 5) <img src="Screenshots/france.png" width="10" height="15"/> Mettre en place une protection contre les scans sur les ports ouverts de la VM<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Set up scan protection on open ports of the VM


[Scan]:<br/>
<img src="Screenshots/france.png" width="10" height="15"/> Permet de trouver dans un délai très court, tous les ports ouverts sur une machine distante.<br/>
<img src="Screenshots/united-states.png" width="10" height="15"/> Allows you to find all open ports on a remote machine in a very short period of time.

[PSAD : Port Scan Attack Detector]

* <img src="Screenshots/france.png" width="10" height="15"/> Installer PSAD - <img src="Screenshots/united-states.png" width="10" height="15"/> Install PSAD -> `sudo apt-get install psad`

* <img src="Screenshots/france.png" width="10" height="15"/> Ajouter au fichier IPtables ces deux lignes - <img src="Screenshots/united-states.png" width="10" height="15"/> Add these two lines to the IPtables file : `sudo vim /etc/iptables/rules.v4` :
````````````````````````
-A INPUT -j LOG
-A FORWARD -j LOG
````````````````````````
(<img src="Screenshots/france.png" width="10" height="15"/> Active la journalisation sur les chaînes d'entrée et de transfert d'IPtables afin que le démon PSAD puisse détecter toute activité anormale.)<br/>(<img src="Screenshots/united-states.png" width="10" height="15"/> Enables logging on IPtables input and transfer strings so that the PSAD daemon can detect any abnormal activity).

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier de configuration - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the configuration file /etc/psad/psad.conf -> `sudo vim /etc/psad/psad.conf`

* <img src="Screenshots/france.png" width="10" height="15"/> Modifier les lignes suivantes - <img src="Screenshots/united-states.png" width="10" height="15"/> Modify the following lines :
````````````````````````
EMAIL_ADDRESSES		root@localhost;
HOSTNAME		localhost;
````````````````````````
![linux - linux](/Screenshots/psad_1.png)
***

* <img src="Screenshots/france.png" width="10" height="15"/> Modifier ceci pour pointer vers le fichier syslog, où psad aura réellement la possibilité de parcourir les journaux actifs - <img src="Screenshots/united-states.png" width="10" height="15"/> Modify this to point to the syslog file, where psad will actually be able to browse the active logs :
````````````````````````
ENABLE_SYSLOG_FILE	Y;
IPT_WRITE_FWDATA	Y;
IPT_SYSLOG_FILE		/var/log/syslog;
````````````````````````
![linux - linux](/Screenshots/psad_2.png)
***

* <img src="Screenshots/france.png" width="10" height="15"/> Activer les paramètres suivants pour activer la fonction IPS et le niveau de danger. Après avoir activé le paramètre dans le fichier de configuration, le démon PSAD bloquera automatiquement l'attaquant en ajoutant son adresse IP dans les chaînes IPtables<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Activate the following parameters to activate the IPS function and the danger level. After enabling the parameter in the configuration file, the PSAD daemon will automatically block the attacker by adding his IP address in the IPtable strings :
````````````````````````
ENABLE_AUTO_IDS		Y;
AUTO_IDS_DANGER_LEVEL	1;
````````````````````````
![linux - linux](/Screenshots/psad_3.png)
***

<br/>

* <img src="Screenshots/france.png" width="10" height="15"/> Exécuter maintenant la commande suivante pour mettre à jour la base de données de signatures pour la détection des attaques - <img src="Screenshots/united-states.png" width="10" height="15"/> Now run the following command to update the signature database for attack detection -> `psad --sig-update`

* <img src="Screenshots/france.png" width="10" height="15"/> Redemarrer le service - <img src="Screenshots/united-states.png" width="10" height="15"/> Restart the service -> `sudo psad -R` (restart)

* <img src="Screenshots/france.png" width="10" height="15"/> Afficher l'etat de tous les proccessus en cours - <img src="Screenshots/united-states.png" width="10" height="15"/> Show status of all running processes -> `sudo psad -S` (status)
<br/><br/><br/><br/><br/>

### 6) <img src="Screenshots/france.png" width="10" height="15"/> Arretez les services non nécessaire pour ce projet<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Stop services not required for this project.


* <img src="Screenshots/france.png" width="10" height="15"/> Lister les services disponibles et répertorier l'état des services contrôlés par le systeme - <img src="Screenshots/united-states.png" width="10" height="15"/> List available services and list the status of services controlled by the system -> `sudo systemctl list-units --type=service --state=active`

![linux - linux](/Screenshots/systemctl.png)
***

<br/>

* <img src="Screenshots/france.png" width="10" height="15"/> Stopper un service - <img src="Screenshots/united-states.png" width="10" height="15"/> Stop a service -> `sudo systemctl disable $nameofservice`

* <img src="Screenshots/france.png" width="10" height="15"/> Désactiver les services estimé non necéssaires - <img src="Screenshots/united-states.png" width="10" height="15"/> Disable services deemed unnecessary :

* <img src="Screenshots/france.png" width="10" height="15"/> Services désactivés - <img src="Screenshots/united-states.png" width="10" height="15"/> Disabled services:

	* console-setup.service : <img src="Screenshots/france.png" width="10" height="15"/> Ce paquet fournit à la console le même modèle de configuration du clavier que celui du système X Window - <img src="Screenshots/united-states.png" width="10" height="15"/> This package provides the console with the same model keyboard configuration than the X Window System keyboard configuration.

	* keyboard-setup.service : <img src="Screenshots/france.png" width="10" height="15"/> configuration du clavier - <img src="Screenshots/united-states.png" width="10" height="15"/> keyboard layout.

	* exim4 : <img src="Screenshots/france.png" width="10" height="15"/> Mail Transfert Agent (postfix est utilisé à la place) - <img src="Screenshots/united-states.png" width="10" height="15"/> Mail Transfert Agent (postfix is used instead).
<br/><br/><br/><br/><br/>

### 7) <img src="Screenshots/france.png" width="10" height="15"/> Réalisez un script qui met à jour l’ensemble des sources de package, puis de vos packages et qui log l’ensemble dans un fichier nommé /var/log/update_script.log. Créez une tache planifiée pour ce script une fois par semaine à 4h00 du matin età chaque reboot de la machine<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Make a script that updates all the package sources, then your packages, and log all of them in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4:00 am and at each reboot of the machine.


[Cron] :<br/>
<img src="Screenshots/france.png" width="10" height="15"/> Cron est un programme qui permet aux utilisateurs des systèmes Unix d’exécuter automatiquement des scripts, des commandes ou des logiciels à une date et une heure spécifiée à l’avance.<br/><br/>

<img src="Screenshots/united-states.png" width="10" height="15"/> Cron is a program that allows users of Unix systems to automatically execute scripts, commands or software at a pre-specified date and time.

* <img src="Screenshots/france.png" width="10" height="15"/> Créer le fichier - <img src="Screenshots/united-states.png" width="10" height="15"/> Create the file : /var/log/update_script.log -> `sudo touch /var/log/update_script.log`
* <img src="Screenshots/france.png" width="10" height="15"/> Créer un fichier dans - <img src="Screenshots/united-states.png" width="10" height="15"/> Create a file in : /etc/cron.d -> `sudo touch /etc/cron.d/update_packages`
* <img src="Screenshots/france.png" width="10" height="15"/> Donner les droits d'execution au root - <img src="Screenshots/united-states.png" width="10" height="15"/> Give execution rights to the root : -> `sudo chmod 744 /etc/cron.d/update_packages`
* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier avec vim - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the file with vim -> `sudo vim /etc/cron.d/update_packages` puis ecrire ce script - then write this script :

``````````````````````````````
#!/bin/bash

apt-get update && ((date && apt-get -y upgrade; echo) >> /var/log/update_script.log 2>&1)
``````````````````````````````
![linux - linux](/Screenshots/cron_script_1.png)
***

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier crontab - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the crontab file ->	`sudo crontab -e`

* <img src="Screenshots/france.png" width="10" height="15"/> Y ajouter les lignes suivantes - <img src="Screenshots/united-states.png" width="10" height="15"/> Add the following lines :
``````````````````````````````
0 4 * * 5 /etc/cron.d/update_packages
@reboot /etc/cron.d/update_packages
``````````````````````````````
![linux - linux](/Screenshots/cron_1.png)
***

<br/><br/><br/><br/><br/>

### 8) <img src="Screenshots/france.png" width="10" height="15"/> Réalisez un script qui permet de surveiller les modifications du fichier /etc/crontab et envoie un mail à root si celui-ci a été modifié. Créez une tache plannifiée pour script tous les jours à minuit<br/><br/><img src="Screenshots/united-states.png" width="10" height="15"/> Run a script that monitors changes to /etc/crontab and sends a mail to root if it has been modified. Create a scheduled script task every day at midnight.


* <img src="Screenshots/france.png" width="10" height="15"/> Créer un fichier dans - <img src="Screenshots/united-states.png" width="10" height="15"/> Create a file in : /etc/cron.d dans lequel on stocke ce que renvoie la commande md5sum - in which we store what the md5sum command returns -> `sudo /etc/cron.d/touch cron_old_hash`
* <img src="Screenshots/france.png" width="10" height="15"/> Créer un fichier dans - <img src="Screenshots/united-states.png" width="10" height="15"/> Create a file in : /etc/cron.d dans lequel on ecrira le script - in which we'll write the script -> `sudo touch /etc/cron.d/cron_file_control`
* <img src="Screenshots/france.png" width="10" height="15"/> Donner les droits d'execution au root - <img src="Screenshots/united-states.png" width="10" height="15"/> Give execution rights to the root -> `chmod 744 /etc/cron.d/cron_file_control`
* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier avec vim - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the file with vim -> `sudo vim /etc/cron.d/cron_file_control` puis ecrire ce script - then write this script :

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

* <img src="Screenshots/france.png" width="10" height="15"/> Ouvrir le fichier crontab - <img src="Screenshots/united-states.png" width="10" height="15"/> Open the crontab file ->	`sudo crontab -e`

* <img src="Screenshots/france.png" width="10" height="15"/> Y ajouter la ligne suivante - <img src="Screenshots/united-states.png" width="10" height="15"/> Add the following lines :
``````````````````````````````
0 0 * * * /etc/cron.d/cron_file_control
``````````````````````````````
![linux - linux](/Screenshots/cron_2.png)
***

<br/><br/><br/><br/><br/>

---
layout: post
title:  "Installation de Raspbian Lite sans écran et sans clavier"
date:   2017-04-20 15:30:00 +0200
categories: avr
lang: fr
---

## Matériel requis

- Raspberry Pi 3
- Alimentation 5V / 2.5A
- Carte MicroSD 2Go minimum, 8Go recommandé
- Lecteur de carte microSD ou lecteur de carte SD avec adaptateur microSD/SD
- Un ordinateur sous Linux

## Fichiers requis

- Raspbian Lite [2017-04-10](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2017-04-10/2017-04-10-raspbian-jessie-lite.zip) 
  ou [plus récent](https://downloads.raspberrypi.org/raspbian_lite_latest)

## Mise en garde

La commande `dd` est très efficace pour copier/écrire/écraser un disque dur.
Elle est aussi très efficace pour effacer toutes vos données si vous le lui
demandez.

## Préparation de la carte microSD

### Avant de brancher la carte microSD

La commande
`lsblk` permet de lister la liste des disques de données du systèmes.

```bash
# samuel at W541-arch in ~/Téléchargements
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   477G  0 disk 
├─sda1   8:1    0   450M  0 part 
├─sda2   8:2    0   100M  0 part 
├─sda3   8:3    0    16M  0 part 
└─sda4   8:4    0 476.4G  0 part 
sdb      8:16   0 447.1G  0 disk 
├─sdb1   8:17   0   512M  0 part /boot
├─sdb2   8:18   0 430.8G  0 part /
└─sdb3   8:19   0  15.9G  0 part [SWAP]
```
Chez moi, les disques système sont `sda` et `sdb`. Ce sont les disques 
qui contiennent toutes les données de mon ordinateur. Il ne faut **surtout 
pas** les utiliser avec la commande `dd`.

### Après le branchement de la carte microSD

On relance `lsblk` pour trouver le nom de la carte microSD:

```bash
# samuel at W541-arch in ~/Téléchargements
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0   477G  0 disk 
├─sda1        8:1    0   450M  0 part 
├─sda2        8:2    0   100M  0 part 
├─sda3        8:3    0    16M  0 part 
└─sda4        8:4    0 476.4G  0 part 
sdb           8:16   0 447.1G  0 disk 
├─sdb1        8:17   0   512M  0 part /boot
├─sdb2        8:18   0 430.8G  0 part /
└─sdb3        8:19   0  15.9G  0 part [SWAP]
mmcblk0     179:0    0    15G  0 disk 
├─mmcblk0p1 179:1    0     1M  0 part 
├─mmcblk0p2 179:2    0    10M  0 part /run/media/samuel/Boot imx233
└─mmcblk0p3 179:3    0    12M  0 part /run/media/samuel/c27a3bf0-3a2b-48a6-99d4-
```

Chez moi, la carte microSD se nomme `mmcblk0`.

```bash
# samuel at W541-arch in ~/Téléchargements
$ microsd=mmcblk0 # À adapter par le nom de votre disque
```

Pour finir, on utilise la commande `dd` pour écrire l'image disque sur la carte
microSD.

**Important:** Toutes les données de la carte microSD seront perdue!

```bash
# samuel at W541-arch in ~/Téléchargements
$ unzip -p 2017-04-10-raspbian-jessie-lite.zip | sudo dd of=/dev/${microsd} bs=4096
[sudo] Mot de passe de samuel : 
316861+0 enregistrements lus
316861+0 enregistrements écrits
1297862656 bytes (1.3 GB, 1.2 GiB) copied, 7.17898 s, 181 MB/s

# samuel at W541-arch in ~/Téléchargements
$ sync

```

### Activation du serveur SSH

Avec les images de Raspbian récentes (à partir de novembre 2016), le serveur
SSH n'est plus activé par défaut. Pour l'activer, il faut ajouter un fichier
nommé `ssh` dans la première partition de la carte microSD.

On relance `lsblk` pour trouver le point de montage (_mountpoint_) de cette
partition:

```bash
# samuel at W541-arch in ~
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0   477G  0 disk 
├─sda1        8:1    0   450M  0 part 
├─sda2        8:2    0   100M  0 part 
├─sda3        8:3    0    16M  0 part 
└─sda4        8:4    0 476.4G  0 part 
sdb           8:16   0 447.1G  0 disk 
├─sdb1        8:17   0   512M  0 part /boot
├─sdb2        8:18   0 430.8G  0 part /
└─sdb3        8:19   0  15.9G  0 part [SWAP]
mmcblk0     179:0    0    15G  0 disk 
├─mmcblk0p1 179:1    0    41M  0 part /run/media/samuel/boot
└─mmcblk0p2 179:2    0   1.2G  0 part /run/media/samuel/f2100b2f-ed84-4647-b5ae-089280112716

```
Chez moi, il s'agit de `/run/media/samuel/boot`

```bash
# samuel at W541-arch in ~
$ cd /run/media/samuel/boot # À adapter

# samuel at W541-arch in /run/media/samuel/boot
$ touch ssh && sync


```

## Connexion au système par SSH

### Branchement

On peut à présent retirer la carte de l'ordinateur et l'insérer dans le
Raspberry Pi. Ensuite, on branche le Raspberry sur le réseau local à l'aide
d'un câble RJ45. Pour finir, on peut l'allumer.

### Recherche de l'adresse IP

Pour ce faire, on a besoin de la commande nmap. Pour l'installer:

```bash
# Archlinux
sudo pacman -S nmap

# Debian, Ubuntu et leurs dérivées
sudo apt install nmap
```

On recherche tous les appareils du réseaux local:

```bash
# samuel at W541-arch in /run/media/samuel
$ nmap -sn 192.168.1.0/24 | grep 'raspberrypi'
Nmap scan report for raspberrypi-2.home (192.168.1.129)
```
- **Note 1**: Cette commande est valable pour les réseaux domestiques standard
(Passerelle en `192.168.1.x` et masque de sous-réseaux `255.255.255.0`)

- **Note 2**: `/24` correspond aux nombres de bit à 1 dans `255.255.255.0`. En binaire,
`255` s'écrit `11111111`, on a donc `3 x 8` bits à 1, soit `24`

La commande prend une dizaine de secondes, puis retourne les Raspberry trouvé
sur le réseau. On note l'adresse ip:

```bash
# samuel at W541-arch in /run/media/samuel
$ rpi_ip=192.168.1.129
```

### Connexion

```bash
# samuel at W541-arch in /run/media/samuel
$ rpi_ip=192.168.1.129

# samuel at W541-arch in /run/media/samuel
$ ssh pi@${rpi_ip}
The authenticity of host '192.168.1.129 (192.168.1.129)' can't be established.
ECDSA key fingerprint is SHA256:H6f9faAfUhbGl34kCXJFAX7khMjgKo8MOKDflw8Cw/g.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.129' (ECDSA) to the list of known hosts.
pi@192.168.1.129's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $ 

```

Il faut répondre `yes⏎` à la question `Are you sure you want to continue connecting (yes/no)?`.

Le mot de passe par défaut de Raspbian est `raspberry`, donc on tappe `raspberry⏎`
à la question `pi@192.168.1.129's password`.

À partir de maintenant, les commandes entrées sur le terminal seront exécutée
sur le RaspberryPi.

## Derniers réglages

### Redimensionnement de la partition système

Par défaut, la partition système de Raspbian Lite à une taille de 1.2Go afin de
pouvoir être copiée sur des petites cartes SD de 2Go. Pour utiliser toutes la
places disponible, on peut étendre la partition à l'aide du programme suivant:

```bash
pi@raspberrypi:~ $ sudo raspi-config
```

On va dans le menu `7 Advanced Options`, puis on choisit `A1 Expand Filesystem`.

Note: La navigation se fait à l'aides des touches fléchées `⇧⇩`, et la sélection par `⏎`

Pour quitter le programme, on sélectionne `<Finish>` en appuyant deux fois sur
la touche tab `↹`.

On répond oui (_yes_) à la question `Would you like to reboot now?` pour
redémarrer le RaspberryPi, ce qui est nécessaire pour que la modification prenne
effet.

### Mise à jour du système

Le redémarrage du système a fermé la connexion SSH entre le PC et la carte, on
se reconnecte avec

```bash
# samuel at W541-arch in /run/media/samuel
$ ssh pi@${rpi_ip}
```
Et on met à jour le système avec

```bash
pi@raspberrypi:~ $ sudo apt update && sudo apt -y dist-upgrade

```

Cette commande peut prendre beaucoup de temps si l'image de Raspbian installée
est ancienne.


## Source

- [RaspberryPi.org -- Installing operating system images on Linux](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)
- [RaspberryPi.org -- A security update for Raspbian PIXEL](https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/)
- [RaspberryPi.org -- IP Address](https://www.raspberrypi.org/documentation/remote-access/ip-address.md)

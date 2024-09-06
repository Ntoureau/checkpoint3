# **Exercice 2 : Manipulations pratiques sur VM Linux**

## **Partie 1 : Gestion des utilisateurs**
**Q2.1.1 et Q2.1.2**
On crée le compte avec (ajouter sudo si pas connecté en root).
On peut l'ajouter au groupe "sudo" pour pouvoir utiliser les privilèges administrateurs. On peut ensuite restreindre l'accès au répertoire personnel par les autres utilisateurs.

```
adduser nicolast
usermod -aG sudo nicolast
chmod 700 /home/nicolast
```

## **Partie 2 : Configuration de SSH**
**Q.2.2.1**
Pour accéder au fichier de configuration SSH, on utilise (avec les droits superutilisateur):
```
nano /etc/ssh/sshd_config
```

Pour désactiver l'accès à distance par l'utilisateur root, on modifie la ligne suivante :
[17.rootloginssh.png]()

**Q.2.2.2**
Pour autoriser l'accès à distance au compte personnel créé précédemment, on ajoute la ligne :

```
AllowUsers nicolast
```

**Q.2.2.3**
Pour mettre en place l'authentification par clé, on décommente les lignes suivantes :

```
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile  .ssh/authorized_keys .ssh/authorized_keys2
```

Il faut générer une paire de clé SSH et la copier sur le serveur depuis la machine locale. Il est préférable de conserver l'accès par mot de passe avant de rentrer les commandes suivantes, puis de le désactiver par la suite.

```
ssh-keygen -t ed25519 -C "adressemail@exemple.com"
# ici ce n'est pas nécessaire car nous sommes sur le serveur
ssh-copy-id nicolast@10.0.2.15
# ajouter la ligne suivante pour ajouter la clé
AuthorizedKeysFile .ssh/authorized_keys
```

On relance le service avec 
```systemctl restart ssh```

# **Partie 3 : Analyse du stockage**

**Q2.3.1**
Pour vérifier quels systèmes de fichiers sont montés, on utilise 
```
# le -h permet d'avoir une sortie lisible par l'homme
df -h
```
On obtient :

[18.systfichiers.png]()

**Q2.3.2**
Pour obtenir les systèmes de stockage, on utilise la même commande que précédemment avec l'argument -T.

[19.syststockage.png]

**Q2.3.3**
Pour ajouter le nouveau disque au serveur, on ouvre VirtualBox, on sélectionne notre serveur, ```Configuration``` > ```Stockage```> ```Ajouter un périphérique```> ```Ajouter un VMDK```et on lui alloue les 8G de mémoire.
avec 
```
lsblk
```
On obtient notre nouveau disque : ```sdb```

```
# On partitionne le disque
fdisk /dev/sdb
# m pour afficher les commandes
# on ajoute la partition
n
# on choisit le type de partition et leur numéro ainsi que les secteurs, ici j'ai gardé les valeurs par défaut
# on écrit la partition
w
# on vérifie l'état du raid
cat /proc/mdstat
# on ajoute le nouveau disque au volume RAID
mdadm --add /dev/md0 /dev/sdb
```

**Q2.3.4**
```
# on vérifie le nom du groupe de volumes
vgdisplay
# on crée le nouveau volume logique
lvcreate -L 2G -n backup_lv cp3-vg
# on crée le système de fichier sur le nouveau volume
mkfs.ext4 /dev/cp3-vg/backup_lv
# on crée le point d'ancrage
mkdir -p /var/lib/bareos/storage
# on récupère l'UUID du nouveau volume
blkid /dev/cp3-vg/backup_lv
# on obtient 2D82b600-1a4f-4f2a-95ee-a0b83807f2a3
# on va dans le fichier fstab et on ajoute la ligne (attention aux majuscules)
nano /etc/fstab
UUID=2d82b600-1a4f-4f2a-95ee-a0b83807f2a3 /var/lib/bareos/storage ext4 defaults 0 2
```

[20.fstab.png]()

**Q.2.3.5**
```
# On affiche le volume restant avec 
vgs
# ou encore
vgdisplay -s
```

[21.volumerestant.png]

il reste donc <1,79 GiB de disponibles.

## **Partie 4 : Sauvegardes**
**Q.2.4.1**
bareos-dir: c'est le composant directeur de Bareos, il gère la gestion des opérations de sauvegarde, de restauration, permet de planifier les jobs de sauvegarde et le catalogue d'informations des fichiers sauvegardés. C'est lui qui coordonne les actions des différents composants. Il s'installe sur le serveur.

bareos-sd: c'est le composant de stockage (le storage daemon), il écrit et lit les données de sauvegarde sur les dispositifs / media de stockage. C'est avec ce composant que l'on gère les endroits où sont sauvegardés les données. Il s'installe sur le poste qui recevra les sauvegardes

bareos-fd : c'est le daemon que l'on installe sur le client, c'est lui qui gère quelles informations vont être sauvegardées : il les lit et les envoie au sd

## **Partie 5 : Filtrage et analyse réseau**

**Q.2.5.1**

```
#Pour afficher les tables, on utilise
nft list tables
# On obtient la table inet (IPv4) appelé inet_filter_table
# on affiche les différentes règles appliquées
nft list ruleset
```

[22.ruleslist.png]()

**Q.2.5.2**
```ct state established,related accept```: autorise les communications déjà établies via une connexion existante
```iifname "lo" accept```: autorise les communication sur l'interface de loopback
```tcp dport 22 accept```: c'est le port standard pour les communications ssh, elles sont autorisées
```ip protocol icmp accept```: il accepte les pings IPv4
```ip nexthdr ipv6-icmp accept```: il accepte les pings IPv6

**Q.2.5.3**
```ct state invalid drop```: il interdit les communications si les paquets sont considérés invalides (drapeaux non valides, connexion hors table, TCP précédant une connexion pas encore établie, fragments, paquets violant une règle de protocole)

**Q.2.5.4**
```
# on ajoute une règle à notre table inet_filter_table pour autoriser les communications sur les ports 9101 à 9103
nft add rule inet inet_filter_table input iifname lo tcp dport 9101-9103 accept
# on sauvegarde la configuration
nft list ruleset > /etc/nftables.conf
```

## **Partie 6 : Analyse de logs
**Q.2.6.1**

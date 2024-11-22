# Rapport du TP4

Comme d'habitude on commence par scanner toutes les machines sur le réseau
`sudo netdiscover -r 10.0.2.0/24`
![1732280959237](image/rapport-tp4/1732280959237.png)

Cela nous permet d'obtenir l'adresse ip de la machine que l'on souhaite attaquer : 10.0.2.7/24

on commence par faire une analyse détaillée de tous les ports ouverts.
`nmap -sC -sV -A -p- 10.0.2.7`

![1732281030145](image/rapport-tp4/1732281030145.png)

On voit que 2 ports sont ouverts : le 22 et le 80

## Port 22


On sait que le port 22 correspont à ssh, donc on va faire une attaque brut force en utilisant hydra
`sudo hydra -l root -p /usr/share/wordlists/metasploit/unix-users.txt -t 6 ssh://10.0.2.7`

![1732286142457](image/rapport-tp4/1732286142457.png)

En testant plusieurs fichiers txt différents on constate que ça échoue.


## Port 80

Il s'agit d'un serveur web, je me rend donc à l'url 10.0.2.7 et je tombe sur un blog

![1732281931595](image/rapport-tp4/1732281931595.png)

En cliquand sur le lien de "test" et en ajoutant un caractère (virugle / apostrophe) dans l'url, j'obtiens un message m'indiquant une erreur de syntaxe SQL.

![1732282094489](image/rapport-tp4/1732282094489.png)

Je sais donc qu'il y a du SQL et je vais pouvoir utiliser l'outil SQLMap sur cette url pour tenter de trouver des failles.

Avec l'option "--db", on cherche à énumérer les bases de données. L'option "--batch" pour ne pas avoir à confirmer les entrées dans le terminal, cela permet de faire les choix par défaut.
`sqlmap -u http://10.0.2.7/cat.php?id=1 --dbs --batch`

Cette commande nous informe de l'existance de 2 bases de données : information_schema et photoblog.

![1732283321308](image/rapport-tp4/1732283321308.png)


On va maintenant essayer de récupérer toutes les tables de la base de données information_schema

`sqlmap -u http://10.0.2.7/cat.php?id=1 -D information_schema --tables`

![1732284725781](image/rapport-tp4/1732284725781.png)

Je ne remarque rien de spécial.

On essaye donc de récupérer toutes les tables de la base de données photoblog

`sqlmap -u http://10.0.2.7/cat.php?id=1 -D photoblog --tables`

![1732284864154](image/rapport-tp4/1732284864154.png)

Cette fois-ci on observe une table users qui peut nous intéresser pour potentiellement obtenir des identifiants de connexion.

La commande suivante nous permet de tirer des informations des utilisateurs de la base de données photoblog avec l'option "-D" pour spécifier la base de données photoblog et "-T" pour spécifier que l'on veut rechercher dans la table users. "--dump" permet d'extraire des informations de la table.

`sqlmap -u http://10.0.2.7/cat.php?id=1 -D photoblog -T users --dump --batch`

![1732285222603](image/rapport-tp4/1732285222603.png)

La commande nous affiche un user (admin) avec son mot de passe (P4ssw0rd).

En retournant sur google à l'url "http://10.0.2.7" on peut se rendre dans l'onglet admin et entrer les identifiants de connexion

![1732285546343](image/rapport-tp4/1732285546343.png)

Cela nous redirige vers la page de gestion des utilisateurs

![1732285714579](image/rapport-tp4/1732285714579.png)

Etant donné que PHP est utilisé gérer les requêtes, on peut essayer de générer un payload PHP pour attaquer la cible. 

On va générer un script PHP contenant un reverse payload. Ce script va établir une connexion vers l'adresse IP 10.0.2.4 (l'ip de ma machine hote) sur le port (1234).

On va faire une copie d'un payload préinstallé sur la machine

`cp /usr/share/webshells/php/php-reverse-shell.php ~/Desktop/script.php`

Maintenant on va modifier ce fichier "script.php" afin de faire correspondre l'ip (ip hôte) et le numéro de port (1234).

![1732293885837](image/rapport-tp4/1732293885837.png)

Je peux ouvrir une connexion netcat qui va écouter sur le port 1234 de la machine hote.

Sur le site web, je clique sur ajouter une image. En essayant d'importer le "script.php", et j'ai alors eu un message d'erreur indiquant "NO PHP!!" qui s'affiche.

![1732289222546](image/rapport-tp4/1732289222546.png)

Pour bypass cette erreur, on peut modifier le nom du fichier de "payload.php"  en "payload.pHP". Puis on le réimporte, il n'ya a pas d'erreur. 

![1732294916166](image/rapport-tp4/1732294916166.png)

Enfin, en cliquant sur le script dans la liste, ce dernier s'execute et netcat intercepte la connexion sur le port 1234, me redirigeant directement sur le terminal de la machine cible.

![1732294208906](image/rapport-tp4/1732294208906.png)

Je peux vérifier que je suis bien connecté à la machine cible avec la commande "ip addr". On retrouve bien lip `10.0.2.7`.

![1732295085822](image/rapport-tp4/1732295085822.png)
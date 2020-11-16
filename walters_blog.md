# Walter's Blog

## Description :
Un ancien stagiaire avait développé ce site web il y a plusieurs années. Malheureusement, ce projet a mal été documenté et nous ne retrouvons plus les accès pour l'administrer...

Votre tuteur vous autorise à tout essayer pour récupérer les accès à ce service, soyez inventif !

Le flag est situé dans le fichier /flag.txt.

PS: Ne mettez pas flag{} pour valider l'épreuve.

## Solution :
Le site utilise une ancienne version vulnérable d'**Apache Tomcat** (9.0.0.M1).
Cette version est vulnérable à la **CVE-2017-12617** : https://nvd.nist.gov/vuln/detail/CVE-2017-12617

Il y a un programme automatisant l'exploit **CVE-2017-12617** :  https://github.com/cyberheartmi9/CVE-2017-12617.
Il faut donc lancer le programme en python avec les arguments :

`python tomcat-cve-2017-12617.py -u http://waltersblog3.chall.malicecyber.com/ -p pwn`

Cela va ouvrir un shell avec la machine, il nous suffit ensuite de rentrer la commande `cat /flag.txt` et le tour est joué !

Flag : `i4lW4y5UpD4T3Y0urt0mC@`
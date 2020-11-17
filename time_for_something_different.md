# Time for something different

## Description :
Le serveur récoltant les candidatures à nos offres d’emploi en cyberdéfense a été attaqué. Nous avons enregistré un fichier trace au format PCAP. Aidez-nous à comprendre ce qu’il s’est passé ?

## Solution :
Le fichier fournit était un `.pcap`, Wireshark va donc être utile.
Les paquets capturés sont tous des requêtes ping non répondues... et tous identiques à l'exception du temps de l'envoi. En regardant un peu, il est devient clair que les 3 premiers caractères du delta de temps de l'envoi du paquet précédent rentre dans la table ASCII (`[70, 76, 65, 71, ...]`).
Avec du travail manuel on obtient :
```
>>> ''.join([chr(c) for c in [70, 76, 65, 71, 123, 116, 48, 48, 115, 108, 113, 87, 111, 114, 116, 48, 48, 68, 51, 118, 49, 111, 117, 36, 125]])
'FLAG{t00slqWort00D3v1ou$}'
```

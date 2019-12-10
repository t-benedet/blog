---
layout: post
title: Installation d'Apache sous AIX
---

&nbsp;La configuration des serveurs avec ma clé RSA publique étant maintenant terminée, je peux passer à l'installation d'Apache sur AIX. L'installation est différente que sur un Linux puisque que nous n'avons pas de `yum install` ou d'`apt install` sur AIX. Cependant, cet OS est livré avec un bon nombre de paquet, dont le paquet __httpd__.

&nbsp;
#### __I - Installation d'Apache__

&nbsp;
Pour l'installation nous allons utiliser l'utilitaire __Smitty__. On tape donc :

```
Smitty install
```
Et voilà ce qui apparait :

![image_1](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/1.png?raw=true)

On séléctionne ma première option __Install and Update Software__ :

![image_2](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/2.png?raw=true)

On séléctionne __Install Sofirware__ et dans `INPUT devive / directory for software` on met un `.`, indiquant ainsi à __Smitty__ que le directory à utiliser le directory dans lequel nous nous trouvons :

![image_3](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/3.png?raw=true)

Ensuite, il suffit d'appuyer sur entrer pour attérir sur la page qui va nous permettre de choisir le paquet à installer :

![image_4](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/4.png?raw=true)

Plusieurs options s'offrent à nous. Nous choisisson l'option `Software to install` :

![image_5](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/6.png?raw=true)

Un catalogue des applications déjà présente s'ouvre. On voit que si on tape `/`, comme sur `Vi` ou `Vim`, on peut chercher un mot. Le paquet qui nous intéresse est le paquet httpd, on tape donc `/httpd` :

Plusieurs paquets sont disponibles :

- httpd.base
- httpd.licence
- httpd.man.en-US

&nbsp;
On séléctionne d'abord `httpd.base` en tapant sur F7 et on voit que notre paquet __httpd.base__ s'est ajouté sur la ligne __SOFTWARE to install__:

![image_6](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/7.png?raw=true)

Puis on tape sur entrer pour lancer l'installation :

![image_8](https://github.com/t-benedet/blog/blob/gh-pages/pictures/Apache_u103/9.png?raw=true)

On aurait pu installer directement les 3 paquets httpd. Sur le coup nous n'avons pas jugé cela nécessaire. Mais nous décidons finalement de les installer. On réitère donc la même opéréation pour les paquets httpd.licence et httpd.man.en-US

Apache est mainteant installé. 

&nbsp;
### __Update du 05 Septembre 2019 :__

Suite à une manipulation sur le serveur de test AIX 7.2 u103, le service __apache/httpd__ n'était pas lancé. La commande `apachectl start` façon linux ne fonctionne pas. Après quelques minutes de recherche, il faut finalement lancer le script `./apachectl`. Donc pour se faire on se rend dans le repertoire `/usr/IBMAHS/bin` puis on tape `./apachectl start`. Le service __apache/httpd__ redémarre.



---
layout: post
title: Installation d'Apache sous AIX
---

&nbsp;La configuration des serveurs avec ma clé RSA publique étant maintenant terminée, je peux passer à l'installation d'Apache sur AIX. L'installation est différente que sur un Linux puisque que nous n'avons pas de `yum install` ou d'`apt install` sur AIX. Cependant, cet OS est livré avec un bon nombre de paquet, dont le paquet __httpd__.

&nbsp;
#### I - Installation d'Apache

&nbsp;
Pour l'installation nous allons utiliser l'utilitaire __Smitty__. On tape donc :

```
Smitty install
```
Et voilà ce qui apparait :

![image_1](http://image.noelshack.com/fichiers/2019/29/5/1563522628-1.png)

On séléctionne ma première option __Install and Update Software__ :

![image_2](http://image.noelshack.com/fichiers/2019/29/5/1563522693-2.png)

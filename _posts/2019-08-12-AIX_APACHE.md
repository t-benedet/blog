---
layout: post
title: Installation et configuration d’Apache sous AIX
---

&nbsp;
La configuration des serveurs avec ma clé RSA publique étant maintenant terminée, je peux passer à l'installation d'Apache sur AIX. L'installation est différente que sur un Linux puisque que nous n'avons pas de `yum install` ou d'`apt install` sur AIX. Cependant, cet OS est livré avec un bon nombre de paquet, dont le paquet __httpd__.
&nbsp;
&nbsp;

### __I - Installation d'Apache__

&nbsp;
Pour l'installation nous allons utiliser l'utilitaire __Smitty__. On tape donc :

```
Smitty install
```

Et voilà ce qui apparait :

![iamge_1](http://image.noelshack.com/fichiers/2019/29/5/1563522628-1.png)

On séléctionne ma première option __Install and Update Software__ :

![image_2](http://image.noelshack.com/fichiers/2019/29/5/1563522693-2.png)



On séléctionne __Install Sofirware__ et dans `INPUT devive / directory for software` on met un `.`, indiquant ainsi à __Smitty__ que le directory à utiliser le directory dans lequel nous nous trouvons :

![image_3](http://image.noelshack.com/fichiers/2019/29/5/1563522902-3.png)

Ensuite, il suffit d'appuyer sur entrer pour attérir sur la page qui va nous permettre de choisir le paquet à installer :

![image_4](http://image.noelshack.com/fichiers/2019/29/5/1563523038-4.png)

Plusieurs options s'offrent à nous. Nous choisisson l'option `Software to install` :

![image_5](http://image.noelshack.com/fichiers/2019/29/5/1563523524-5.png)

Un catalogue des applications déjà présente s'ouvre. On voit que si on tape `/`, comme sur `Vi` ou `Vim`, on peut chercher un mot. Le paquet qui nous intéresse est le paquet httpd, on tape donc `/httpd` :

![image_6](http://image.noelshack.com/fichiers/2019/29/5/1563523718-6.png)

Plusieurs paquets sont disponibles :

- httpd.base
- httpd.licence
- httpd.man.en-US

On séléctionne d'abord `httpd.base` en tapant sur F7 et on voit que notre paquet __httpd.base__ s'est ajouté sur la ligne __SOFTWARE to install__:

![image_7](http://image.noelshack.com/fichiers/2019/29/5/1563523826-7.png)

Puis on tape sur entrer pour lancer l'installation :

![image_8](http://image.noelshack.com/fichiers/2019/29/5/1563523928-9.png)

On aurait pu installer directement les 3 paquets httpd. Sur le coup nous n'avons pas jugé cela nécessaire. Mais nous décidons finalement de les installer. On réitère donc la même opéréation pour les paquets httpd.licence et httpd.man.en-US

Apache est mainteant installé. Nous allons pouvoir mettre en place notre CGI.

&nbsp;
#### __Première étape : Configuration du fichier httpd.conf__

Les fichiers de configuration httpd sous AIX se trouve dans __/usr/IBMAHS/conf__. Il faut donc s'y rendre. Pour se faire, je me connecte au serveur u103, je passe root et je tape :
```
cd /usr/IBMAHS/conf
```

Comme sur un Linux, le fichier de configuration s'appelle __httpd.conf__. On l'ouvre avec la commande suivante :
```
vi httpd.conf
```

On cherche la ligne __ScriptAlias /cgi-bin/ "/usr/IBMAHS/cgi-bin"__ et dans la partie __<Directory "/usr/IBMAHS/cgi-bin">__ on modifie les lignes de la manière suivante :
```
<Directory "/usr/IBMAHS/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
    Options +ExecCGI
    Allow from all
</Directory>
```

On cherche ensuite la ligne __AddHandler__ et on modifie comme cela :
```
#AddHandler cgi-script .cgi .ksh
```

De base, __cgi-script__ et __.cgi__ sont déjà présents. Cela veut dire qu'Apache va executer ces types de fichiers comme étant des fichiers CGI. Comme nous voulons que nos script __.ksh__ soient reconnus aussi, on ajoute l'extension __.ksh__.

Notre fichier __httpd.conf__ est maintenant configuré. Nous allons donc passer à la création de notre première page CGI.

&nbsp;
#### __Deuxième étape : Création de la première page CGI__


Mon profil n'étant pas encore totalement compliant, c'est mon tuteur qui testera l'efficacité de ma page. 

Pour la page CGI, rien de compliqué. Je prends simplement un bout de code de mon autre CGI mais je l'enregistre en __.ksh__ au lieu de __.sh__. Je créé donc le fichier `index.ksh` :
```
#!/bin/ksh

echo "Content-type: text/html"
echo ""

echo '
<html>
        <head>
                <meta http-equiv="Content-Type" content="test/html"; charset=UTF-8">
                <title> CGI TEST  </title>
                <h1> AIX CGI  </h1>
                <hr size="4" color="blue">
        <style>
                         body{
                          background-color: #eff1f0;
                         }
        </style>
        </head>
<body>'
echo "<h1> LOREM IPSUM</h1>"
echo "<p>

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur fringilla tortor eu odio mollis, ac cursus diam pharetra. Sed quis risus est. Suspendisse potenti. Fusce nec rhoncus augue, vestibulum blandit nisi. Sed hendrerit in nisl vitae rhoncus. Pellentesque ac enim vitae urna rhoncus porttitor eu imperdiet justo. Quisque maximus accumsan augue vel malesuada. Vivamus massa nunc, posuere non elementum vitae, cursus sit amet justo. Duis rhoncus aliquam semper. Proin varius egestas scelerisque. Fusce semper urna ut velit congue, ac molestie nulla laoreet. Phasellus et tortor eget nulla vestibulum interdum id sit amet leo. Sed sit amet velit egestas, varius magna sed, fringilla arcu. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas.

Aliquam efficitur dictum est, et semper massa pulvinar ut. Phasellus ac imperdiet nunc, nec feugiat elit. In aliquam massa vitae finibus ullamcorper. Donec tortor lacus, sagittis sit amet odio a, dignissim rhoncus justo. Aliquam porta dictum malesuada. Vivamus aliquam aliquet volutpat. Phasellus sed tellus ligula. Vestibulum luctus rhoncus lectus at faucibus. Maecenas consequat eleifend ex, non imperdiet nisl molestie at. Duis a scelerisque mauris. 
echo </p>"

echo "
</body>
</html> "
```

Ne pas oublier de changer le sheban. On remplace `#!/bin/sh` par `#!/bin.ksh`. Sans ça, le script ne sera pas reconnu.

Il n'y a plus qu'à vérifier ce que nous renvoi notre navigateur. Pour cela, il faut taper dans la barre URL :
```
http://15.1.24.35/cgi-bin/index.ksh
```

Et là, ça ne fonctionne pas. Premier reflexe, vérifier les logs :
```
tail /usr/IBMAHS/logs/acces_log
```
La commande tail permet de n'afficher que les 10 dernières lignes du fichier de log. Voici le contenu :

![image_9](http://image.noelshack.com/fichiers/2019/29/5/1563528018-aix-tail.png)

On remarque un `Permission denied` sur notre index.ksh. Nous allons donc vérifier les droits de notre fichier. Pour cela on se rend dans notre repertoire `cgi-bin`. La commande permettant de voir les droits des fichiers et repertoires est :
```
ls -ltra
```

Voici le résultat :

![image_10](http://image.noelshack.com/fichiers/2019/29/5/1563528324-droit.png)

On remarque ici que notre fichier `index.ksh` appartient à __root:system__ et que les droits sont __u+rw__. Nous voulons aussi savoir le nom de l'utilisateur qui execute httpd. Pour cela, on tape la commande suivante :
```
ps -ef | grep httpd
```

On remarque que c'est l'utilisateur __nobody__ qui exécute httpd :

![image_11](http://image.noelshack.com/fichiers/2019/29/5/1563529044-grep.png)

Nous allons donc changer : le propriétaire du fichier `index.ksh` :
```
chown nobody:nobody index.ksh
```
Et les droits du ficher pour ajouter l'autorisation d'exécution :
```
chmod u+x index.ksh
```
Ce qui donne :

![image_12](http://image.noelshack.com/fichiers/2019/29/5/1563529365-chown.png)

Nous pouvons maintenant rester notre CGI :

![image_13](http://image.noelshack.com/fichiers/2019/29/5/1563529476-test-ok.jpg)
&nbsp;

Notre test est maintenant terminé.

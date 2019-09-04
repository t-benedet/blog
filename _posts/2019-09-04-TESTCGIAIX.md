---
layout: post
title: Mise en place de la CGI sous AIX 7.2
---

### Mise en place de la CGI sous AIX 7.2

Notre test étant maintenant terminé, nous allons pouvoir commencer à développer notre véritable CGI.
Cette CGI nous permettra de suivre l'évolution de l'espace disponible de différents filesystems.

Notre base de travail sera le résultat de la commande `df` qui permet de lister les différents filesystems présents sur le serveur avec le total d'espace disponible, ce qui est déjà occupé etc... Pour une meilleure visibilté, nous passerons la commande `df -m` qui nous permet de lister exactement la même chose que la commande `df` mais en convertissant le tout en MB :

&nbsp;
__Output commande `df`:__ 
```
Filesystem    512-blocks      Free %Used    Iused %Iused Mounted on
/dev/hd4          786432    466656   41%     7651    13% /
/dev/hd2         7340032    332856   96%    62247    54% /usr
/dev/hd9var      4325376   2969928   32%     3580     2% /var
/dev/hd3         6684672   3089144   54%     2395     1% /tmp
/dev/hd1          262144    167536   37%      295     2% /home
/proc                  -         -    -        -      - /proc
/dev/hd10opt      917504    248456   73%     2132     7% /opt
/dev/lvesm        786432    785416    1%        4     1% /usr/esm
/dev/lvexploit    1703936    407960   77%     1123     3% /exploit
/dev/lvtivoli    1048576    647624   39%     1091     2% /opt/Tivoli
/dev/lvaudit      262144    221184   16%       16     1% /var/adm/ras/audit
/dev/lvperfmgr     131072     76400   42%      244     3% /var/adm/perfmgr
/dev/lvusrha      524288    379080   28%     1617     4% /usr/es
/dev/lvvarha     2621440   1051712   60%      850     1% /var/hacmp
/dev/lvscm        524288    384720   27%      747     2% /opt/IBM/SCM
/dev/lv_itm      1703936   1702984    1%        5     1% /opt/IBM/ITM
/dev/lv_itmlog     524288    523552    1%        4     1% /opt/IBM/ITM/logs
/dev/lv_doonce    1441792    184792   88%      327     2% /opt/DoOnceAIX
/dev/lvbes       5898240   3974592   33%    14064     3% /var/opt/BESClient
/dev/livedump     524288    521176    1%        8     1% /var/adm/ras/livedump
/aha                   -         -    -       49     1% /aha
/dev/lvsource   21037056  14399344   32%     6663     1% /repsource
beisilanfp100_adm:/Software  2688811008 964312856   65%   234991     1% /Software
```
&nbsp;
__Output commande `df -m` :__
```
Filesystem    GB blocks      Free %Used    Iused %Iused Mounted on
/dev/hd4           0.38      0.22   41%     7651    13% /
/dev/hd2           3.50      0.16   96%    62247    54% /usr
/dev/hd9var        2.06      1.42   32%     3580     2% /var
/dev/hd3           3.19      1.47   54%     2394     1% /tmp
/dev/hd1           0.12      0.08   37%      295     2% /home
/proc                 -         -    -        -      - /proc
/dev/hd10opt       0.44      0.12   73%     2132     7% /opt
/dev/lvesm         0.38      0.37    1%        4     1% /usr/esm
/dev/lvexploit      0.81      0.19   77%     1123     3% /exploit
/dev/lvtivoli      0.50      0.31   39%     1091     2% /opt/Tivoli
/dev/lvaudit       0.12      0.11   16%       16     1% /var/adm/ras/audit
/dev/lvperfmgr      0.06      0.04   42%      244     3% /var/adm/perfmgr
/dev/lvusrha       0.25      0.18   28%     1617     4% /usr/es
/dev/lvvarha       1.25      0.50   60%      850     1% /var/hacmp
/dev/lvscm         0.25      0.18   27%      747     2% /opt/IBM/SCM
/dev/lv_itm        0.81      0.81    1%        5     1% /opt/IBM/ITM
/dev/lv_itmlog      0.25      0.25    1%        4     1% /opt/IBM/ITM/logs
/dev/lv_doonce      0.69      0.09   88%      327     2% /opt/DoOnceAIX
/dev/lvbes         2.81      1.90   33%    14064     3% /var/opt/BESClient
/dev/livedump      0.25      0.25    1%        8     1% /var/adm/ras/livedump
/aha                  -         -    -       49     1% /aha
/dev/lvsource     10.03      6.87   32%     6663     1% /repsource
beisilanfp100_adm:/Software    1282.12    459.82   65%   234992     1% /Software
```

Le but sera de créer automatiquement des sauvegardes de ce fichier tous les X temps afin de pouvoir tracer des graphiques pour suivre l'évolution des différents filesystems et réussir à tracer une courbe de tendance afin de prévoir l'espace qui sera nécessaire dans le futur.

&nbsp;
### I - Création de la page d'index

Pour la mise en place de cette page, je vais m'inspirer de ce que j'avais pu faire précédemment [sur ma machine en local](). Voilà donc mon premier code :
```
#!/bin/ksh

echo "Content-type: text/html"
echo ""

echo '
<html>
        <head>
                <meta http-equiv="Content-Type" content="test/html"; charset=UTF-8">
                <title> AIX CGI </title>
                <h1> AIX CGI <span style="margin-left:78%"> <font size=3> <a href="AIX.ksh">[ AIX \ </a><a href="OpenSUSE.ksh">OpenSUSE ] </a> </font> </span></h1>
                <hr size="4" color="blue">
                
        <style>
                          body{
                          background-color: #eff1f0;
                          font-family: 'IBM Plex Sans', sans-serif;
                         }
        
                         .df_AIX{
                         margin-top: 19px;
                         font-family: 'IBM Plex Sans', sans-serif;
                         }
                        
                         .ping_AIX{
                         margin-top: 3%;
                         font-family: 'IBM Plex Sans', sans-serif;
                         }

                         .df_AIX_title{
                         margin-top: -32.7%;
                         margin-left: 50%;
                         font-family: 'IBM Plex Sans', sans-serif;
                         }

                         .df_AIX_stat{
                         margin-top: 1%;
                         margin-left: 0.8%;       
                         font-family: 'IBM Plex Sans', sans-serif;
                         }

                         .C_title{
                         margin-top: 0.8%;
                         }

                         .M_title{
                         margin-top: 1.5%;
                         }

                         .A_title{
                         margin-top: 1.5%;
                         }

                         .C_button{
                         margin-top: -1.8%;
                         margin-left: 13%;
                         } 
        
                         .M_button{
                         margin-top: -3.6%;
                         margin-left: 13%;
                         }

                         .A_button{
                         margin-top: -2%;
                         margin-left: 13%;
                         }
</style>
        </head>
<body>'
echo "<h3>AIX FILESYSTEM MONITORING </h3>"


########################### SCRIPT PING ##########################

echo "<div class="ping_AIX">"
        echo "<h3> U103 Server status : </h3>"
        echo "<PRE>"
        ping -c 1 10.14.54.103 ;
        if [ $? -eq 0 ]
        then
                echo ""
                echo "> Serveur status : <span style="color:green">OK</span>"
        else
                echo "> Serveur stauts : <span style="color:red">DOWN</span>"
        fi
        echo "</PRE>"
echo "</div>"


########################### GRAPH PING ###############################

echo "<br>"

FILESYSTEM_LIST="$(df -g | awk '{print $1}' | sed '1,1d')" 

echo "<div class="df_AIX">"
echo "<h3> U103 Serveur Graph : </h3>"
echo "</div>"

        echo "<div class="C_title">"
                echo "> Current stat graph : "
        echo "</div>"

        echo "<div class="C_button">"
                echo "<form action="AIX_GNUPLOT.ksh" method="post">"
                echo "<select name="FILESYSTEM">"
                echo "$FILESYSTEM_LIST" | while read FILESYSTEM; do
                echo "<option value="$FILESYSTEM">$FILESYSTEM</option>"
                done
                echo "</select>"
                        echo "<input type="submit" value="Generate">"
                echo "</form>"
        echo "</div>"



        echo "<div class="M_title">"
                echo "> Monthly stat graph : "
        echo "</div>"

        echo "<div class="M_button">"_
                echo "<form action="AIX_GNUPLOT.ksh" method="post">"
                echo "<select name="FILESYSTEM">"
                echo "$FILESYSTEM_LIST" | while read FILESYSTEM; do
                echo "<option value="$FILESYSTEM">$FILESYSTEM</option>"
                done
                echo "</select>"
                        echo "<input type="submit" value="Generate">"
        echo "</div>"



        echo "<div class="A_title">"
        echo "> Annual stat graph :"
        echo "</div>"
                
        echo "<div class="A_button">"
                echo "<form action="AIX_GNUPLOT.ksh" method="post">"
                echo "<select name="FILESYSTEM">"
                echo "$FILESYSTEM_LIST" | while read FILESYSTEM; do
                echo "<option value="$FILESYSTEM">$FILESYSTEM</option>"
                done
                echo "</select>"
                        echo "<input type="submit" value="Generate">"
                echo "</div>"


############################### DF STAT  #################################

echo "<div class="df_AIX_title">"
	echo "<h3>df -m current state : </h3>"
		echo "<div class="df_AIX_stat">"
			echo "<PRE>"
		        df -m
		        echo "</PRE>"
		echo "</div>"
echo "</div>"

echo "
</body>
</html> "
```
&nbsp;
Ce qui, lorsque que l'on tape l'adresse IP du serveur dans la barre url de Firefox, affiche :

![image_1](http://image.noelshack.com/fichiers/2019/34/4/1566466645-cgi-ok1.png)

On y voit trois " zones " :

__1. U103 Server status__ 

-> Un " simple " bout de script qui sera executé lorsque la CGI est ouverte :
```
ping -c 1 10.14.54.103 ;

        if [ $? -eq 0 ]
        then
                echo ""
                echo "> Serveur status : <span style="color:green">OK</span>"
        else
                echo "> Serveur stauts : <span style="color:red">DOWN</span>"
        fi
```

Je ping l'adresse du serveur avec l'option `-c 1` qui permet d'arrêter le ping au premier paquet __envoyé et valide__. Si le retour est `echo 0`, c'est que le ping s'est bien passé, la CGI renverra donc le résumé du `ping` et affichera " Serveur status : OK  ". Si le retour est `echo 1`, c'est que le serveur ne répond pas, la CGI affichera donc " Serveur status : DOWN  " . 

&nbsp;
__2. U103 Server Graph :__

-> Ici, je génère des listbox en html que je remplie à l'aide d'une boucle `while` :
```
FILESYSTEM_LIST="$(df -g | awk '{print $1}' | sed '1,1d')" 

echo "<div class="C_button">"
                echo "<form action="AIX_GNUPLOT.ksh" method="post">"
                echo "<select name="FILESYSTEM">"
                echo "$FILESYSTEM_LIST" | while read FILESYSTEM; do
                echo "<option value="$FILESYSTEM">$FILESYSTEM</option>"
                done
                echo "</select>"
                        echo "<input type="submit" value="Generate">"
                echo "</form>"
        echo "</div>"
```
Le contenu de mes listbox sera le résultat de la commande `df -g`. Je ne prends pas le résultat complet puisque que je n'affiche que la première colonne avec `awk '{print $1}'`. De plus, je supprime " l'en tête " de la colonne avec la commande `sed '1,1d'` qui permet de supprimer la première ligne de mon contenu.
Les listbox vont donc contenir la liste de mes filesystems.

&nbsp;
__3. df -g current stats :__
 
 -> Ici, je ne fais qu'afficher en temps réel le résultat de la commande `df -g` :
 ```
 echo "<PRE>"
df -g
echo "</PRE>"
```
Le tout entre des balises `<PRE> </PRE>` pour garder la forme de la mise en page.		       

&nbsp;
La page d'index de la première version officielle de la CGI sous AIX 7.2 est maintenant terminée. Nous y reviendrons plus tard en cas de changement.

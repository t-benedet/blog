---
layout: post
title: CGI sous AIX 7.2
---

Maintenant que ma CGI de test est fonctionnelle sur le serveur u103. Je dois faire la même chose mais sur un serveur de production LOréal.

&nbsp;
#### __I - Mise en place de notre page d'index__

Cette page sera la page d'accueil pour notre CGI. Elle permettra de naviguer et de récupérer les données des différents serveurs, de tracer des graphiques selon une période voulue et affichera en à un instant T le resultat général de la commende `df -m`.

Voici le code de la page :
```

echo "Content-type: text/html"
echo ""

echo '
        <html>
                <head>
                        <meta http-equiv="Content-Type" content="test/html"; charset=UTF-8">
                        <title>DF MONITORING </title>
                        <h1> AIX TEST <font size=3></font> </h1>
                        <hr size="4" color="blue">

                        <style>
                                body {
                                background-color: #eff1f0;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }

                                #AIX {
                                margin-top: 5%;
                                }

                                #LINUX {
                                margin-top: 10%;
                                }

                        </style>
                </head>
<body>'


########## AIX #########

SERV_AIX="$(ls /var/IBMtools/www/tim/collect/AIX/*.txt | cut -d '/' -f8 | cut -d'_' -f1)"
SERV_LINUX="$(ls /var/IBMtools/www/tim/collect/Linux/*.txt | cut -d '/' -f8 | cut -d'_' -f1)"
DATE_AIX="$(ls /var/IBMtools/www/tim/collect/AIX/*.txt | cut -d'_' -f2 | cut -d'.' -f1 | sort | uniq )"
DATE_Linux="$(ls /var/IBMtools/www/tim/collect/Linux/*.txt | cut -d'_' -f2 | cut -d'.' -f1 | sort | uniq )"
#FS_AIX="$(cat /var/IBMtools/www/tim/collect/AIX/*.txt | awk -F';' '{print $2 }' | sort | uniq  )"

echo "<div id="AIX">"

echo "<p> AIX Filesystem : </p>"

echo "<form action="AIX_GNU.ksh" method="post">"
echo "<select name="SERV">"
echo "$SERV_AIX" | while read SERV_AIX; do
echo " <option value="$SERV_AIX">$SERV_AIX</option>"
done
echo "</select>"

echo "<select name="DATE_AIX">"
echo "$DATE_AIX" | while read DATE_AIX; do
echo " <option value="$DATE_AIX">$DATE_AIX</option>"
done
echo "</select>"

echo "<input type="submit" value="Select">"
echo "</form>"

echo "</div>"


########## LINUX ##########


echo "<div id="LINUX">"

echo "<p> LINUX Filesystem : </p>"

echo "<form action="LINUX_GNU.ksh" method="post">"
echo "<select name="SERV">"
echo "$SERV_LINUX" | while read SERV_LINUX; do
echo " <option value="$SERV_LINUX">$SERV_LINUX</option>"
done
echo "</select>"

echo "<select name="DATE_Linux">"
echo "$DATE_Linux" | while read DATE_Linux; do
echo " <option value="$DATE_Linux">$DATE_Linux</option>"
done
echo "</select>"

echo "<input type="submit" value="Select">"
echo "</form>"

echo "</div>"

echo "
</body>
</html> "

```

Ce qui donne une fois lancé :

![image_1](https://zupimages.net/up/19/48/k7jd.png)

Rien de compliqué pour le moment. Deux parties, une partie AIX avec une menu déroulant pour choisir le serveur et un autre menu pour choisir le mois qui nous intéresse.
La deuxième partie, pour Linux, est identique à la partie AIX. Seule les valeures seront différentes.

&nbsp;

#### __II - Création du script Gnuplot__

Maintenant que ma page d'accueil est faite, je vais m'attaquer au script Gnuplot. Pour suivre une évolution, il est préférable d'utiliser un graphiques fait de courbes plutôt qu'un histogramme. 

On commence par définir un titre pour le graphique, ses caractéristiques graphiques, l'endroit où devra être généré l'image, la taille les labels pour les différentes axes etc... : 
```
set terminal png truecolor size 1950, 650  background rgb "#eff1f0" 
set output "/var/IBMtools/www/tim/TEST.png"
set datafile separator ';'

set size ratio 0.2
set bmargin at screen 0.2
set key outside horizontal 
set datafile separator ";"
set ylabel " MB BLOCK " font ",10" offset -1,0
set xlabel font ",10"
set xtics rotate by 45  offset -0.8,-9,-1.8

```
&nbsp;
Ensuite on définit les différents styles de lignes et puces qui seront utilisées par les courbes :

```
et style line 1 linecolor rgb '#0060ad' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 2 linecolor rgb '#e50073' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 3 linecolor rgb '#ea5b0c' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 4 linecolor rgb '#4f328a' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 5 linecolor rgb '#00a49f' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 6 linecolor rgb '#b4cc04' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 7 linecolor rgb '#00aee7' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 8 linecolor rgb '#0069b4' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 9 linecolor rgb '#7f3089' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 10 linecolor rgb '#b51865' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 11 linecolor rgb '#6f1a16' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 12 linecolor rgb '#ffd5c0' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 13 linecolor rgb '#c0cbff' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 14 linecolor rgb '#9a542e' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 15 linecolor rgb '#5c8096' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 16 linecolor rgb '#fcd116' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 17 linecolor rgb '#336633' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 18 linecolor rgb '#b0be40' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 19 linecolor rgb '#570df1' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 20 linecolor rgb '#e19221' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 21 linecolor rgb '#cb5c52' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 22 linecolor rgb '#90a775' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 23 linecolor rgb '#9463d4' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 24 linecolor rgb '#bc0200' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 25 linecolor rgb '#94789a' linetype 1 linewidth 2 pointtype 7 pointsize 1.5
set style line 26 linecolor rgb '#3b23b0' linetype 1 linewidth 2 pointtype 7 pointsize 1.
set style line 26 linecolor rgb '#fb979c'linetype 1 linewidth 2 pointtype 7 pointsize 1.5

```
&nbsp;
Enfin, on définit le repertoire où se trouve les informations nécessaires, on précise que dans notre fichier, la troisième colonne sera l'axe y et se basera sur la première colonne qui elle, sera l'axe x. On défini ensuite le style de graphique. Il est nécessaire de faire ça pour chaques courbes :
```
plot "/opt/IBMtools/cgi-bin/tim/data/dooncelv.txt" using 3:xtic(1) title column(2) with linespoints linestyle 1, \
     "/opt/IBMtools/cgi-bin/tim/data/hd10opt.txt" using 3:xtic(1) title column(2) with linespoints linestyle 2, \
     "/opt/IBMtools/cgi-bin/tim/data/hd11admin.txt" using 3:xtic(1) title column(2) with linespoints linestyle 3, \
     "/opt/IBMtools/cgi-bin/tim/data/hd1.txt" using 3:xtic(1) title column(2) with linespoints linestyle 4, \
     "/opt/IBMtools/cgi-bin/tim/data/hd2.txt" using 3:xtic(1) title column(2) with linespoints linestyle 5, \
     "/opt/IBMtools/cgi-bin/tim/data/hd3.txt" using 3:xtic(1) title column(2) with linespoints linestyle 6, \
     "/opt/IBMtools/cgi-bin/tim/data/hd4.txt" using 3:xtic(1) title column(2) with linespoints linestyle 7, \
     "/opt/IBMtools/cgi-bin/tim/data/hd9var.txt" using 3:xtic(1) title column(2) with linespoints linestyle 8, \
     "/opt/IBMtools/cgi-bin/tim/data/livedump.txt" using 3:xtic(1) title column(2) with linespoints linestyle 9, \
     "/opt/IBMtools/cgi-bin/tim/data/lsDB2pp_01.txt" using 3:xtic(1) title column(2) with linespoints linestyle 10, \
     "/opt/IBMtools/cgi-bin/tim/data/lvaudit.txt" using 3:xtic(1) title column(2) with linespoints linestyle 11, \
     "/opt/IBMtools/cgi-bin/tim/data/lvbes.txt" using 3:xtic(1) title column(2) with linespoints linestyle 12, \
     "/opt/IBMtools/cgi-bin/tim/data/lvesm.txt" using 3:xtic(1) title column(2) with linespoints linestyle 13, \
     "/opt/IBMtools/cgi-bin/tim/data/lvetctivoli.txt" using 3:xtic(1) title column(2) with linespoints linestyle 14, \
     "/opt/IBMtools/cgi-bin/tim/data/lvexploit.txt" using 3:xtic(1) title column(2) with linespoints linestyle 15, \
     "/opt/IBMtools/cgi-bin/tim/data/lvitmlogTWS.txt" using 3:xtic(1) title column(2) with linespoints linestyle 16, \
     "/opt/IBMtools/cgi-bin/tim/data/lv_itmlog.txt" using 3:xtic(1) title column(2) with linespoints linestyle 17, \
     "/opt/IBMtools/cgi-bin/tim/data/lvitmTWS.txt" using 3:xtic(1) title column(2) with linespoints linestyle 18, \
     "/opt/IBMtools/cgi-bin/tim/data/lv_itm.txt" using 3:xtic(1) title column(2) with linespoints linestyle 19, \
     "/opt/IBMtools/cgi-bin/tim/data/lvsrm.txt" using 3:xtic(1) title column(2) with linespoints linestyle 20, \
     "/opt/IBMtools/cgi-bin/tim/data/lvsyslog.txt" using 3:xtic(1) title column(2) with linespoints linestyle 21, \
     "/opt/IBMtools/cgi-bin/tim/data/lvtivoliepA.txt" using 3:xtic(1) title column(2) with linespoints linestyle 22, \
     "/opt/IBMtools/cgi-bin/tim/data/lvtivoli.txt" using 3:xtic(1) title column(2) with linespoints linestyle 23, \
     "/opt/IBMtools/cgi-bin/tim/data/lvtws.txt" using 3:xtic(1) title column(2) with linespoints linestyle 24, \
     "/opt/IBMtools/cgi-bin/tim/data/lvusrha.txt" using 3:xtic(1) title column(2) with linespoints linestyle 25, \
     "/opt/IBMtools/cgi-bin/tim/data/lvvarha.txt" using 3:xtic(1) title column(2) with linespoints linestyle 26

```

&nbsp;
#### __III - Mise en place de la page web pour le graphique :__

Maintenant que notre script gnuplot est terminé, je vais mettre en place la page web qui va recevoir le graphique.

Pour le moment, la page se résume à se code :
```
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
                        
                         #current_stat{
                         margin-right: 15%;
                         margin-top: 2%;
                         float: right;
                         font-size: 20px;
                         font-family: 'IBM Plex Sans', sans-serif;
                         }      
                        
                         #image{
                         margin-top: -2%;
                         position: absolute;
                         }
 
                         #FS_title{
                         margin-top: 3%;
                         }
                </style>
        </head>
<body>'

read a

FILESYSTEM=$( echo $a | cut -d'=' -f2 | sed 's/\%//g' | sed 's/2F/\//g' | cut -d '&' -f1)
awk_value=$( echo $FILESYSTEM | cut -d'/' -f3)


echo "<div id="FS_title"><h2> > FILESYSTEM : $FILESYSTEM </h2></div>"
echo "<br>"

#awk ' /'$awk_value'/ {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' /tim/df_log/df_command* > /usr/IBMAHS/htdocs/TEST.txt
gnuplot gnuplot_line.txt

echo "<PRE>"

echo "<div id="current_stat">"
echo "<h3> $FILESYSTEM current stat :</h3>"
df -m | grep $FILESYSTEM | awk '{
print "- MB Blocks : "$2" MB "
print "- Free Space : "$3" MB "
print "- Space Used : " $4
print "- Mounted on : " $7
}'

echo "</div>"
echo "</PRE>"

echo '
<img id="image" src="/test.png">
</body>
</html> '
```

&nbsp;

__Quelques explications :__

__1.__ 
```
read a

FILESYSTEM=$( echo $a | cut -d'=' -f2 | sed 's/\%//g' | sed 's/2F/\//g' | cut -d '&' -f1)
awk_value=$( echo $FILESYSTEM | cut -d'/' -f3)
```

- La ligne `read a` me permet de récupérer la querystring de la page d'index. C'est à dire la valeur séléctionnée par l'utilisateur dans un des menus déroulants.
- La ligne avec la variable `FILESYSTEM` est simplement une ligne de formatage de la querystring. Au lieu d'avoir comme valeur `VALEUR=MA_VALEUR` je ne garde que `MA_VALEUR`
- Avec la ligne `awk_value=$( echo $FILESYSTEM | cut -d'/' -f3)`, je séléctionne uniquement un bout du nom d'un filesystem, par exemple __hd4__ du fs __/dev/hd4/__ afin d'en faire une régex qui agira comme l'équivalent d'un grep.

__2.__
```
awk ' /'$awk_value'/ {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' /tim/df_log/df_command* > /usr/IBMAHS/htdocs/TEST.txt
```
- Cette ligne permet de faire un awk sur l'ensemble des fichiers qui ont été créés grâce à la [crontab](https://t-benedet.github.io/blog/2019/08/28/CRONTAB.html). On ne garde que la colonne 2 qui correspond à l'espace total en MB du filesystem et la colonne 3, qui correspond à l'espace restant. La partie `{split(FILENAME)...` permet de passer d'un nom de fichier du type `df_command-2019-09-05-15:04:34.txt` à `2019-09-05`. Je ne garde donc que la date de création du fichier. Enfin, la dernière partie de cette commande permet de créer le fichier __TEST.txt__ contenant l'ensemble des informations générées par la crontab.
- La commande `gnuplot gnuplot_line.txt` permet de faire lire le fichier __gnuplot_line.txt__ par gnuplot afin de générer le graphique.

&nbsp;
### __IV - Vérification du fonctionnement de notre script Gnuplotet de la page :__

Si nous choisissons le filesystem hd4, voilà le graphique :

![image_2](http://image.noelshack.com/fichiers/2019/36/3/1567604524-3880bceb0ab2fe97.jpg)

Ici nous constatons deux courbes droites. Cela est dû au fait que les valeurs n'ont pas bougées entre le 2019-08-30 et le 2019-09-04.


&nbsp;
### __Update du 09 Septembre 2019 :__

&nbsp;
#### __I - Modification du script Gnuplot__ :

Le script Gnuplot fonctionne parfaitement, mais je dois ajouter une courbe de tendance. Voici le script :
```
set title "test"
set terminal png truecolor size 960,720 background rgb "#eff1f0"
set output "/home/tbenedet/Desktop/GNUPLOT\ AIX/test.png"
set grid
set style line 1 \
    linecolor rgb '#0060ad' \
    linetype 1 linewidth 2 \
    pointtype 7 pointsize 1.5

set style line 2 \
    linecolor rgb "red" \
    linetype 1 linewidth 1 \
    pointtype 7 pointsize 1

set offsets 0.5,0.5,0,0.5
set datafile separator ","
set format y "%g"
set key left
myLabel(n) = sprintf("%g",n)


f(x) = a*x + b
fit f(x) "df_output.txt" u 0:2 via a,b

plot "df_output.txt" using 2:xtic(1) with linespoints linestyle 1 ,\
'' using 3:xtic(1) with linespoints linestyle 2 title " Free space ", \
'' using 0:2:(myLabel($2)) w labels offset 0,-0.5 notitle, \
'' using 0:3:(myLabel($3)) w labels offset 0,1 notitle, \
f(x) w l lc "black" title "trendline" 

```

Une fois appelé par la commande `gnuplot`, voici le résultat :
![image_3](https://image.noelshack.com/fichiers/2019/37/1/1568034268-test.png)

Je rappelle que les valeurs utilisées ici sont fausses. Je les ai volontairement modifiées pour le bien du script. Néanmoins, si le script fonctionne avec ces valeurs, il fonctionnera aussi avec les vraies valeurs.

&nbsp;
#### __II - Modification du code CSS des pages web :__

Rien de bien fou dans cette partie, mais comme nous utilisons des écrans de différentes tailles, mon css ne passait que sur du 16:9 alors que nous avons encore des écrans 4:3 pour un certains nombres d'entre nous ( dont moi pour mon dual screen ). 

J'ai simplement ajouté des __media queries__ à mon css pour appliquer mon css aux différentes résolutions d'écran :
```
@media screen and (width: 1920px){

[...]


@media screen and (width: 1280px){


[...]
```


&nbsp;
### __Update du 27 Septembre 2019 :__

La CGI prend forme petit à petit. Je décide de mettre en place un système qui va permettre de générer un graphique par semaine, par mois ou à l'année. Plusieurs solutions s'offrent à moi, soit je mets en place un fichier __ksh/html__ pour chaque période, soit j'essaie de tout faire en une seule page. C'est l'option que j'ai choisie. Voici donc le test pour le serveur __U103__ :

```
echo "Content-type: text/html"
echo ""

echo '
<html>
        <head>
                <meta http-equiv="Content-Type" content="test/html"; charset=UTF-8">
                <title> AIX CGI </title>
                <h1> AIX CGI</h1>
                <hr size="4" color="blue">
                
                <style>
                         @media screen and (width: 1920px){
                                
                                body{
                                background-color: #eff1f0;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }
                                
                                .nav{
                                margin-left: 79%;
                                margin-top: -3.3%;
                                font-weight: bold;      
                                font-size: 20px;
                                position: absolute;
                                }                       
        
                                .df_AIX{
                                margin-top: 18px;
                                font-family: 'IBM Plex Sans', sans-serif;
                                font-size: 20px;
                                }
                        
                                .ping_AIX{
                                margin-top: 6%;
                                font-family: 'IBM Plex Sans', sans-serif;
                                font-size: 20px;
                                }

                                .df_AIX_title{
                                margin-top: -30.5%;
                                margin-left: 49%;
                                font-family: 'IBM Plex Sans', sans-serif;
                                font-size: 20px;
                                }

                                .df_AIX_stat{
                                margin-top: 1%;
                                margin-left: 49%;         
                                font-family: 'IBM Plex Sans', sans-serif;
                                font-size: 16.5px;
                                position: absolute;     
                                }
                                
                                .w_title{
                                margin-top: 1.5%;
                                font-size: 20px;
                                }

                                .M_title{
                                margin-top: 1.5%;
                                font-size: 20px;
                                }

                                .A_title{
                                margin-top: 1.5%;
                                font-size: 20px;
                                }
                                
                                .W_button{
                                margin-top: -1.4%;
                                margin-left: 13%;
                                font-size: 20px;
                                }
 
                                .M_button{
                                margin-top: -1.4%;
                                margin-left: 13%;
                                font-size: 20px;
                                }

                                .A_button{
                                margin-top: -1.5%;
                                margin-left: 13%;
                                font-size: 20px;
                                }

                                .title{
                                margin-top: -0.2%;
                                position: absolute;
                                }

                                 #current_stat{
                                margin-right: 35%;
                                margin-top: -5.8%;
                                float: right;
                                font-size: 20px;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }


                                .image{
                                width: 95%;
                                margin-top: -4%;
                                }
                         }      
                         

                        @media screen and (width: 1280px){
                         
                                body{
                                background-color: #eff1f0;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }
                        
                                .nav{
                                margin-left: 70%;
                                margin-top: -5%;
                                font-weight: bold;
                                font-size: 20px;
                                position: absulute;
                                }       

        
                                .df_AIX{
                                margin-top: 11px;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }
        
                                .ping_AIX{
                                margin-top: 10%;
                                font-family: 'IBM Plex Sans', sans-serif;
                                }

                                .df_AIX_title{
                                margin-top: -33.5%;
                                margin-left: 50%;
                                font-family: 'IBM Plex Sans', sans-serif;
                                position: absolute;
                                }       

                                .df_AIX_stat{
                                margin-top: -29%;
                                margin-left: 50%;      
                                font-family: 'IBM Plex Sans', sans-serif;
                                position: absolute; 
                                }

                                .M_title{
                                margin-top: 1.5%;
                                }

                                .A_title{
                                margin-top: 1.5%;
                                }
       
                                .W_button{
                                margin-left: 13.5%;
                                }
 
                                .M_button{
                                margin-top: -2%;
                                margin-left: 13.5%;
                                }

                                .A_button{
                                margin-top: -2%;
                                margin-left: 13%;
                                }

                                .title{
                                margin-top: -0.4%;
                                position: absolute;
                                }

                                #current_stat{
                                margin-left: 35%;
                                margin-top: -8.5%;
                                font-size: 18px;
                                font-family: 'IBM Plex Sans', sans-serif;
                                position: float;
                                }

                                .image{
                                width: 95%;
                                margin-left: 2%;
                                margin-top: 4%;
                                position: absolute;
                                }


                                #FS_TITLE{
                                margin-top: 4%;
                                }
                        }
                </style>
        </head>
<body>'

read a

FILESYSTEM1=$( echo $a | cut -d'=' -f2 | sed 's/\%//g' | sed 's/2F/\//g' | cut -d '&' -f1)
FILESYSTEM2=$( echo $a |  cut -d '=' -f1)
awk_value=$( echo $FILESYSTEM1 | cut -d'/' -f3)


echo "<div class="nav">"
echo " <a href="AIX_U103.ksh">[ AIX u103 ]</a> <a href="AIX_U104.ksh">[ AIX u104 ]</a> <a href="SUSE.ksh">[ SUSE L.E.S ]</a>"
echo "</div>"


case $FILESYSTEM2 in
        WFILESYSTEM)
                echo "<div id="FS_title"><h2> > FILESYSTEM : $FILESYSTEM1 </h2></div>"
                echo "<br>"

                ls /tim/df_log/df_command* | tail -7 | xargs awk -v awk_value="$awk_value" '$0 ~ awk_value {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' > /usr/IBMAHS/htdocs/W_df.txt

                cat gnuplot_template.txt | sed "s/TEST.txt/W_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt

                gnuplot /usr/IBMAHS/htdocs/gnuplot_line.txt

                echo "<PRE>"
                echo "<div id="current_stat">"
                        df -m | grep $FILESYSTEM1 | awk '{ print "- MB Blocks : "$2" MB    - Free Space : "$3" MB    - Space Used : "$4"    - Mounted on : "$7" "}'
                echo "</div>"
                echo "</PRE>"
                echo "<img class="image" src="/test.png">"
                ;;

        MFILESYSTEM)
                echo "<div id="FS_title"><h2> > FILESYSTEM : $FILESYSTEM1 </h2></div>"
                echo "<br>"

                ls /tim/df_log/df_command* | tail -30 | xargs awk -v awk_value="$awk_value" '$0 ~ awk_value {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' > /usr/IBMAHS/htdocs/M_df.txt

                cat gnuplot_template.txt | sed "s/TEST.txt/M_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt

                gnuplot /usr/IBMAHS/htdocs/gnuplot_line.txt

                echo "<PRE>"
                echo "<div id="current_stat">"
                        df -m | grep $FILESYSTEM1 | awk '{ print "- MB Blocks : "$2" MB    - Free Space : "$3" MB    - Space Used : "$4"   - Mounted on : "$7" "}'
                echo "</div>"
                echo "</PRE>"
                echo "<img class="image" src="/test.png">"
                ;;
       
        AFILESYSTEM)
                echo "<div id="FS_title"><h2> > FILESYSTEM : $FILESYSTEM1 </h2></div>"
                echo "<br>"

                ls /tim/df_log/df_command* | tail -7 | xargs awk -v awk_value="$awk_value" '$0 ~ awk_value {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' > /usr/IBMAHS/htdocs/A_df.txt

                cat gnuplot_template.txt | sed "s/TEST.txt/A_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt

                gnuplot /usr/IBMAHS/htdocs/gnuplot_line.txt

                echo "<PRE>"
                echo "<div id="current_stat">"
                        df -m | grep $FILESYSTEM1 | awk '{ print "- MB Blocks : "$2" MB    - Free Space : "$3" MB    - Space Used : "$4"  - Mounted on : "$7" "}'
                echo "</div>"
                echo "</PRE>"
                echo "<img class="image" src="/test.png">"
                ;;
esac


echo "
</body>
</html> "
```

&nbsp;

On reste dans un page __html__ classique, mais voici quelques explications sur la partie __ksh__ :

__Pour la partie définition des variables :__

- `read a` => permet de récupérer la query string `MFILESYSTEM=%2Fdev%2Fhd4`
- `FILESYSTEM1=$( echo $a | cut -d'=' -f2 | sed 's/\%//g' | sed 's/2F/\//g' | cut -d '&' -f1)` => permet de ne garder que le nom du filesystem `/dev/hd4` dans la query string
- `FILESYSTEM2=$( echo $a |  cut -d '=' -f1)` => permet de ne garder que la première partie de la query string. C'est elle qui va me permettre de définir la période du graphique. __WFILESYSTEM__ pour un graphique à la semaine, __MFILESYSTEM__ pour un graphgique au mois et __AFILESYSTEM__ pour un graphique à l'année.
- `awk_value=$( echo $FILESYSTEM1 | cut -d'/' -f3)` => permet de garder que le " nom cours " du filesystem `hd4`. Cette variable me permet d'appliquer un `grep` pour garder que le filesystem qui m'intéresse dans le fichier de sauvegarde __df -m__.

&nbsp;

__Pour la partie choix de la période du graphique :__

Ici, j'ai décidé de partir sur un `case` :

On part sur un case à 3 choix. Les choix étant les différents résultats de la variable `FILESYSTEM2` ( WFILESYSTEM, MFILESYSTEM ou AFILESYSTEM ) :

```
case $FILESYSTEM2 in
        WFILESYSTEM)
              [...]
              ;;
        MFILESYSTEM)
              [...]
              ;;
        AFILESYSTEM)
              [...]
              ;;
esac
```

Pour ces différents choix, nous trouverons pratiquement le même code :

```
echo "<div id="FS_title"><h2> > FILESYSTEM : $FILESYSTEM1 </h2></div>"
                echo "<br>"

                ls /tim/df_log/df_command* | tail -30 | xargs awk -v awk_value="$awk_value" '$0 ~ awk_value {split(FILENAME,a,"-");print a[2]"-"a[3]"-"a[4]","$2","$3}' > /usr/IBMAHS/htdocs/M_df.txt

                cat gnuplot_template.txt | sed "s/TEST.txt/M_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt

                gnuplot /usr/IBMAHS/htdocs/gnuplot_line.txt

                echo "<PRE>"
                echo "<div id="current_stat">"
                        df -m | grep $FILESYSTEM1 | awk '{ print "- MB Blocks : "$2" MB    - Free Space : "$3" MB    - Space Used : "$4"   - Mounted on : "$7" "}'
                echo "</div>"
                echo "</PRE>"
                echo "<img class="image" src="/test.png">"
                ;;
```

- Affichage du nom du filesystem avec `FILESYSTEM : $FILESYSTEM1`
- La ligne comportant la commande `ls` et tout ce qui suit permet d'afficher l'ensemble des fichiers dont le nom commence par __df_command__, avec `tail` d'en séléctionner un certains nombre en fonction de la période du graphique et de ne garder que les lignes comportant la valeur de `$awk_value`. On met tout en forme avec la commande `FILENAME,a,"-"` qui va permettre de ne garder que la date dans le nom du fichier, et d'y afficher, à côté, les 2 et 3ème colonnes des fichiers __df_command__, avec les variables `"$2","$3"` et on envoie tout vers le fichier __/usr/IBMAHS/htdocs/M_df.txt__.
- `cat gnuplot_template.txt | sed "s/TEST.txt/M_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt` permet de générer le fichier qui va servir à créer le graphique tout en remplacant la variable `TEST.txt` par `M_df.txt`.
- `cat gnuplot_template.txt | sed "s/TEST.txt/M_df.txt/" > /usr/IBMAHS/htdocs/gnuplot_line.txt` permet de générer le graphique.

On applique le case aux pages qui permettent de générer les graphiques pour les serveurs U103, U104 et SUSE.

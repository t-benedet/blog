---
layout: post
title: C1.3.1.2 - Tester le service 
---

Maintenant qu'Apache est installé et configuré, que Gnuplot est installé et mis à jour, je vais pouvoir me lancer dans conception de la CGI.
Le but de cette CGI est de pouvoir communiquer avec 3 serveurs ( deux AIX 7.2 et un SUSE ) afin de récupérer les informations concernants les différents __filesystem__. Le but est de pouvoir tracé des graphiques pour chaques __fs__  à partir des informations que l'ont va récupérer avec la commande `df -m`.
&nbsp;
#### - Mise en place de notre page d'index

Cette page sera la page d'accueil pour notre CGI. Elle permettra de naviguer et de récupérer les données des différents serveurs, de tracer des graphiques selon une période voulue et affichera en à un instant T le resultat général de la commende `df -m`.

Voici le code de la page :
```
echo "Content-type: text/html"
echo ""

echo '
<html>
        <head>
                <meta http-equiv="Content-Type" content="test/html"; charset=UTF-8">
                <title> AIX CGI </title>
                <h1> AIX CGI <span style="margin-left:67%"> <font size=3> <a href="AIX.ksh">[ AIX u103 ]</a> <a href="AIX.ksh">[ AIX u104 ]</a> <a href="OpenSUSE.ksh">[SUSE L.E.S]</a></font></span></h1>
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
                         margin-top: -33.7%;
                         margin-left: 50%;
                         font-family: 'IBM Plex Sans', sans-serif;
                         }

                         .df_AIX_stat{
                         margin-top: 1%;
                         margin-left: 0.8%;       
                         font-family: 'IBM Plex Sans', sans-serif;
                         }

                         .M_title{
                         margin-top: 1.5%;
                         }

                         .A_title{
                         margin-top: 1.5%;
                         }
        
                         .M_button{
                         margin-top: -3.6%;
                         margin-left: 13%;
                         }

                         .A_button{
                         margin-top: -2%;
                         margin-left: 13%;
                         }

                         .title{
                         margin-top: -11%;
                         position: absolute;
                         }
                </style>
        </head>
<body>'


echo "<div class="title"></div>"
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

echo "<div class="df_AIX">
<h3> U103 Serveur Graph : </h3>
</div>"

        echo "<div class="M_title">"
                echo "> Monthly stats graph : "
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
        echo "> Annual stats graph :"
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

echo "<h3>df -m current stats : </h3>"

echo "<div class="df_AIX_stat">"
        
        echo "<PRE>"
        df -m
        echo "</PRE>"

echo "</div>"

echo "<br>"
echo "<br>"

echo "<PRE>"
echo "</PRE>"

echo "
</body>
</html> "
```

Ce qui donne une fois lancé :
![image_1]()



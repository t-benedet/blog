---
layout: post
title: Récole d'informations sur serveur de production Loréal
---

Comme pour la version test sur les serveurs u103/u104 AIX 7.2 et SUSE, il fallait mettre en place un script qui doit collecter des informations sur une centaine de serveurs ( AIX 7.2 et SUSE ). 


Voilà à quoi ressemble le script pour le moment :
```
AIX()
{
for i in $(cat /var/IBMtools/www/tim/serv_AIX.txt | grep -v "#" | awk '{print $1}')
do
        ping -q -c 1 -w 5 $i

        if [ $? -eq 0 ]
        then
                ssh ibmssh@$i df -m >> /var/IBMtools/www/tim/${i}_`date "+%Y-%m"`.txt
        else
                echo " $i doesn't ping "
        fi

done
}

Linux()
{
for i in $(cat /var/IBMtools/www/tim/serv_Linux | awk '{print $1}')
do
        ping -c 1 -w 5 $i

        if [ $? -eq 0 ]
        then
                ssh ibmssh@$i df -m  >> /var/IBMtools/www/tim/${i}_`date "+%Y-%m"`.txt
        else
                echo " $i doesn't ping "
        fi
done
}



AIX
Linux


```


Ce script va looper sur un ensemble d'adresse IP appartenant à des serveurs AIX/Linux et se trouvant dans deux fichiers externes :

- `ping -q -c 1 -w 5 $i` ➔ On va pinger le contenu de __$i__, envoyer un seul paquet (__-c 1__), il s'arrêtera à partir de 5 secondes de timeout (__-w 5__) mais n'affichera pas le résultat du ping (__-q__).
- `if [ $? -eq 0 ]` ➔ Si le renvoie de la commande est égal à 0
- `ssh ibmssh@$i df -m  >> /var/IBMtools/www/tim/${i}_``date "+%Y-%m"``.txt` ➔ Alors on passe la commande __df -m__ via ssh avec le user __ibmssh__ sur le serveur __$i__. Le résultat de la commande sera sauvegardé vers __/var/IBMtools/www/tim/__ dans un fichier qui aura comme nom l'adresse ip du serveur avec la date du jour.
- `echo " $i doesn't ping "` ➔ Sinon on envoie dans un fichier de log le fait que le ping sur __$i__ n'a pas fonctionné.

&nbsp;

Ce scipt sera ensuite mis dans la crontab du user __ibmssh__ et sera exécuté, pour le moment, tous les jours à 5h du matin.

&nbsp;

### __Upgrade du 28 Novembre 2019 :__

Voici le script qui tourne actuellement sur le serveur LOréal u100 et qui collecte le résultat de la commande __df -m__ sur 99 serveurs. Cette collecte se fait 3 fois par jour, une fois à 6h du matin, une fois à 12h et une dernière fois à 18h.

```
umask 022

AIX()
{
for i in $(cat /var/IBMtools/www/tim/serv_AIX.ref | grep -v "#" | awk '{print $1";"$2}')
do
      ip_serv=$(echo $i | awk -F';' '{print $1}')
      name_serv=$(echo $i | awk -F';' '{print $2}')

      ping -q -c 1 -w 3 $ip_serv 1>/dev/null

      if [ $? -eq 0 ]
      then
          ssh ibmssh@$ip_serv df -m | awk -v date=$(date "+%Y-%m-%d-%H-%M") '/dev/ {print date";"$1";"$2";"($2 - $3)";"$7}' >> /var/IBMtools/www/tim/collect/AIX/${name_serv}_`date "+%Y-%m"`.txt
      else
          echo " $name_serv ( $ip_serv ) not responding "
      fi

done
}

Linux()
{
for i in $(cat /var/IBMtools/www/tim/serv_Linux.ref | awk '{print $1";"$2}')
do
      ip_serv=$(echo $i | awk -F";" '{print $1}')
      name_serv=$(echo $i | awk -F";" '{print $2}')

      ping -q -c 1 -w 3 $ip_serv 1>/dev/null

      if [ $? -eq 0 ]
      then
          ssh ibmssh@$ip_serv df -m | awk -v date=$(date "+%Y-%m-%d-%H-%M") '/mapper/ {print date";"$1";"$2";"$3";"$6}' >> /var/IBMtools/www/tim/collect/Linux/${name_serv}_`date "+%Y-%m"`.txt
      else
          echo " $name_serv ( $ip_serv ) not responding "
      fi
done
}



AIX
Linux
```

Comme pour la première version, ce script loop sur le même fichier contenant la liste des serveurs avec leur __IP__ et leur __hostname__. Voici ce qui a été modifié :

Pour la partie AIX :

- Création de deux variables : __ip_serv__ et __name_serv__. La première variable contient l'adresse IP du serveur, la deuxième son nom.
- Modification de l'output du ping sur le serveur. On n'affiche pas le ping, on arrête le ping au premier paquet reçu/envoyé, on ajoute un timeaout de 3 secondes et on bloque l'affichage des messages d'erreurs en redirigeant le canal 1 vers __/dev/null__.
- Modification de la connection en ssh. Maintenant on se connecte via l'adresse IP qui se trouve dans la variable __ip_serv__, on ajoute la variable __date__ au __awk__, on ne garde que les lignes qui contiennent le mot _dev_ et on affiche les colonnes 1 et 2 , la troisième colonne sera le résultat de la soustraction entre les valeurs se contenant dans les colonnes 2 et 3. Enfin, on affiche la colonne 7. Le résultat sera envoyé dans le repertoire __AIX__ avec comme nom le nom du serveur + la date de création du fichier.

Les modifications apportées sont les mêmes pour la partie Linux.

Résultat de la collecte sous pour les serveurs AIX :

[image_1]!(https://zupimages.net/up/19/48/sbub.png)


Résultat de la collecte sous pour les serveurs LINUX :

[image_2]!(https://zupimages.net/up/19/48/ckxt.png)

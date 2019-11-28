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

Upgrade du 28 Novembre 2019 :

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
          ssh ibmssh@$ip_serv df -m | awk -v date=$(date "+%Y-%m-%d-%H-%M") '/dev/ {print date";"$1";"$2";"($2 - $3)";"$7}' >>              /var/IBMtools/www/tim/collect/AIX/${name_serv}_`date "+%Y-%m"`.txt
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

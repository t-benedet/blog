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


Ce script va looper sur un ensemble d'adresse IP appartenant à des serveurs AIX/Linux et se trouvant dans deux fichiers externes. Les deux fonctions vont looper sur un fichier __Linux__ et __AIX__. Chaque adresses sera conservée dans la variable __$i__ :

- `ping -q -c 1 -w 5 $i` ➔ On va pinger le contenu de __$i__, envoyer un seul paquet (__-c 1__), il s'arrêtera à partir de 5 secondes de timeout (__-w 5__) mais n'affichera pas le résultat du ping (__-q__).
- `if [ $? -eq 0 ]` ➔ Si le renvoie de la commande est égal à 0
- `ssh ibmssh@$i df -m  >> /var/IBMtools/www/tim/${i}_``date "+%Y-%m"``.txt` ➔ Alors on passe la commande __df -m__ via ssh avec le user __ibmssh__ sur le serveur __$i__. Le résultat de la commande sera sauvegardé vers __/var/IBMtools/www/tim/__ dans un fichier qui aura comme nom l'adresse ip du serveur avec la date du jour.
- `echo " $i doesn't ping "` ➔ Sinon on envoie dans un fichier de log le fait que le ping sur __$i__ n'a pas fonctionné.

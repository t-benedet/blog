# I - C1.3.1.1 Mettre en place l’environnement de test du service
----
&nbsp;
Pour le bien de l’équipe AIX, il était nécessaire de développer une CGI en bash/html afin de pouvoir observer l’évolution de la consommation des filesystems. Le but est de pouvoir tracer une courbe de tendance jusque X mois ( à définir ). Cette courbe nous servira à voir s’il est nécessaire d’augmenter la taille de certains filesystems mais aussi de prévoir le futur espace qui sera nécessaire.
&nbsp;
&nbsp;

### 1 - Mise en place de la CGI en bash/html sous AIX 7.2 et OpenSUSE
----
&nbsp;
Avant de commencer notre CGI, certaines manipulations sont nécessaires afin de faciliter l'utilisation de notre CGI. Nous travaillons sur plusieurs serveurs et avec des systèmes différents : AIX 7.2 et OpenSUSE. 

Nous avons donc les serveurs suivants :
- U103 ( AIX 7.2 )
- U104 ( AIX 7.2 )
- Suse-test ( OpenSUSE)

Ayant accès au root sous l'u103, c'est à partir de ce serveur que nous allons travailler. 

La première chose à faire est de se toxer en executant un petit script :
```
./onnect_GSNI.sh
```

&nbsp;
### __Première étape :__ Générer une clé RSA sous le serveur u103
----
Nous allons pouvoir nous connecter à notre serveur u103 :
```
toxoscks ssh tbenedet@15.1.24.35
```

Une fois cela fait, nous tapons la commande :
```
ssh-keygen -t rsa
```

Cette commande va générer automatiquement une clé privée et une clé publique en créant dans le repertoire où vous vous trouvez ( /home/tbenedet/ pour moi ) , un sous-repertoire caché /.ssh. Si nous nous y déplaçons, nous pouvons constater la création des deux clés : id_rsa et id_rsa_pub

![image_1](https://image.noelshack.com/fichiers/2019/29/4/1563438564-3.png "image1")


La création de nos clés maintenant terminée, nous allons partager notre clé publique sur les serveurs u104 et OpenSuse.
&nbsp;
### __Deuxième étape :__ Partager la clé RSA publique avec le serveur u104
---

Nous devons nous connecter au serveur u104. Même manipulation que pour acceder au u103 :
```
toxsocks ssh tbenedet@15.1.24.54
```

Une fois que nous sommes connecté, nous devons créer le repertoire .ssh :
```
mkdir .ssh
```

Puis créer le fichier qui va recevoir notre clé publique :
```
vi authorized_keys
```

Enfin, copier/coller notre clé RSA publique. 

On enregistre le fichier vi grâce à la commande __shfit+zz__ ou __:wq!__ et on se déconnecte du serveur en tapant __exit__ dans le terminal.
&nbsp;
### __Troisième étape :__ Partager la clé RSA publique avec le serveur OpenSUSE
---

On répète les mêmes manipulations qu'à l'étape précédente :

Pour se connecter :
```
sss tbenedet@192.168.7.199
```

La création du fichier .ssh :
```
mkdir .ssh
```

La création du fichier qui va recevoir notre clé publique :
```
vi authorized_keys
```

Il n'y a plus qu'à y mettre note clé, à enregistrer et quitter.
&nbsp;
### __Quatrième étape :__ Vérifier si nos clés sont bien prises en compte
----

On se reconnecte sur u103 et on essaie de se connecter à u104 en ssh :

![image_2](https://image.noelshack.com/fichiers/2019/29/4/1563441469-ffff.jpg)

Ici, ça fonctionne parfaitement. On essaie de se connecter au serveur OpenSUSE via u103 :

![image_3](https://image.noelshack.com/fichiers/2019/29/4/1563441616-9.png)

La connection aux deux serveurs via le serveur u103 fonctionne parfaitement. Les deux clés RSA ont donc bien été prises en compte. 
&nbsp;

Maintenant, il nous est possible de faire passer quelques commandes depuis u103 vers les deux serveurs. Si on test la commande __hostname__ :

Pour la SUSE :

![image_4](https://image.noelshack.com/fichiers/2019/29/4/1563441770-10.png)


Pour le serveur u104 :

![image_5](https://image.noelshack.com/fichiers/2019/29/4/1563441890-12.png)

&nbsp;
&nbsp;
Cette étape préliminaire est maintenant terminée.

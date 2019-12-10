---
layout: post
title: Création & installation de la clé SSH
---

Pour le bien de l’équipe AIX, il était nécessaire de développer une CGI en bash/html afin de pouvoir observer l’évolution de la consommation des filesystems. Le but est de pouvoir tracer une courbe de tendance jusque X mois ( à définir ). Cette courbe nous servira à voir s’il est nécessaire d’augmenter la taille de certains filesystems mais aussi de prévoir le futur espace qui sera nécessaire.

&nbsp;
&nbsp;

#### __I - Mise en place de la CGI en bash/html sous AIX 7.2 et OpenSUSE__

&nbsp;Avant de commencer notre CGI, certaines manipulations sont nécessaires afin de faciliter l'utilisation de notre CGI. Nous travaillons sur plusieurs serveurs et avec des systèmes différents : AIX 7.2 et SUSE. 

Nous avons donc les serveurs suivants :

- U103 ( AIX 7.2 )
- U104 ( AIX 7.2 )
- Suse-test ( SUSE Linux Enterprise Serveur )

Ayant accès au root sous l'u103, c'est à partir de ce serveur que nous allons travailler. 

La première chose à faire est de se toxer en executant un petit script :
```
./connect_GSNI.sh
```

&nbsp;
#### __Première étape : Générer une clé RSA sous le serveur u103__

Nous allons pouvoir nous connecter à notre serveur u103 :
```
toxoscks ssh tbenedet@15.1.24.35
```

Une fois cela fait, nous tapons la commande :
```
ssh-keygen -t rsa
```

Cette commande va générer automatiquement une clé privée et une clé publique en créant dans le repertoire où vous vous trouvez ( /home/tbenedet/ pour moi ) , un sous-repertoire caché /.ssh. Si nous nous y déplaçons, nous pouvons constater la création des deux clés : id_rsa et id_rsa_pub

![image_1](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/cle_rsa_ok.png?raw=true)


La création de nos clés maintenant terminée, nous allons partager notre clé publique sur les serveurs u104 et OpenSuse.

&nbsp;
#### __Deuxième étape : Partager la clé RSA publique avec le serveur u104__

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
#### __Troisième étape : Partager la clé RSA publique avec le serveur OpenSUSE__

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
#### __Quatrième étape : Vérifier si nos clés sont bien prises en compte__

On se reconnecte sur u103 et on essaie de se connecter à u104 en ssh :

![image_2](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/u103_u104.png.jpg?raw=true)

Ici, ça fonctionne parfaitement. On essaie de se connecter au serveur SUSE via u103 :

![image_3](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/u103_suse.png?raw=true)

La connection aux deux serveurs via le serveur u103 fonctionne parfaitement. Les deux clés RSA ont donc bien été prises en compte. 
&nbsp;

Maintenant, il nous est possible de faire passer quelques commandes depuis u103 vers les deux serveurs. Si on test la commande __hostname__ :

Pour la SUSE :

![image_4](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/suse_hostname.png?raw=true)


Pour le serveur u104 :

![image_5](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/u104_hostname.png?raw=true)

&nbsp;
&nbsp;Cette étape préliminaire est maintenant terminée.

&nbsp;
#### __Update du 02 Septembre 2019 :__

Afin de pouvoir passer certaines commandes en remote sur les serveurs __u104__ et __SUSE__ via ma CGI, il était nécessaire de créer un __/home__ pour le user lançant apache. Ce user étant __nobody__ je suis passé via __smitty__ pour lui créer un __/home__ puis il suffisait de taper ` mkdir nobody` dans le __/home__ pour avoir notre repertoire :

![image_6](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/nobody.jpg?raw=true)

J'ai ensuite modifié les droits du repertoire créé avec la commande `chown nobody:nobody nobody` afin que le user qui execute apache puisse écrire dans le repertoire. Ensuite, dans ce repertoire, j'ai du créer le sous repertoire __.ssh__ et y générer une clé RSA grâce à la commande `ssh-keygen -t rsa`. Une fois la clé générée, je l'ai simplement collé dans le fichier __authorized_keys__ de mon user. Cela permet de à Apache de passer sur mon user pour executer certaines ocmmandes sans avoir à entrer le mot de passe.

Pour tester si tout fontionne correment, je me connecte avec le user __nobody__ sur le suerveur __u103__ grâce à la commande `su - nobody` puis je tape :
```
ssh tbenedet@u103 "ssh 192.168.7.199 df -m "
```
Ce qui donne :

![image_7](https://github.com/t-benedet/blog/blob/gh-pages/pictures/RSA/RSA_SUSE.jpg?raw=true)

On voit bien le retour de la commande `df -m`. Le user __nobody__ passe d'abord sur le user __tbenedet__ pour ensuite executer la commander `df -m` sur le serveur SUSE.






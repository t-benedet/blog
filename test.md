---
layout: page
title: test
permalink: /test/
---

#### __I - C1.3.1.1 Mettre en place l’environnement de test du service__

Pour le bien de l’équipe AIX, il était nécessaire de développer une CGI en bash/html afin de pouvoir observer l’évolution de la consommation des filesystems. Le but est de pouvoir tracer une courbe de tendance jusque X mois ( à définir ). Cette courbe nous servira à voir s’il est nécessaire d’augmenter la taille de certains filesystems mais aussi de prévoir le futur espace qui sera nécessaire.
&nbsp;
&nbsp;


#### __1 - Mise en place de la CGI en bash/html sous AIX 7.2 et OpenSUSE__

Avant de commencer notre CGI, certaines manipulations sont nécessaires afin de faciliter l'utilisation de notre CGI. Nous travaillons sur plusieurs serveurs et avec des systèmes différents : AIX 7.2 et OpenSUSE. 

Nous avons donc les serveurs suivants :
- U103 ( AIX 7.2 )
- U104 ( AIX 7.2 )
- Suse-test ( OpenSUSE)

Ayant accès au root sous l'u103, c'est à partir de ce serveur que nous allons travailler. 

La première chose à faire est de se toxer en executant un petit script :
```
./connect_GSNI.sh
```

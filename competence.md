---
layout: default
title: Compétences
permalink: /Compétences/
---

### __Mission 1 : Mise en place d'une CGI sous AIX 7.2 sur un serveur de test P7__

Ma première mission conciste à installer et configurer Apache et Gnuplot pour tester la mise en place d'une CGI sous AIX 7.2 sur serveur P7. Cette CGI permettra également de récupérer de l'information sur un serveur SUSE.

<span style="color:#0366d6"><strong>[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION1 %})</strong></span>

&nbsp;


### __Mission 2 : Mise en place de la CGI sur un serveur de production LOréal__

La deuxième mission reprend l'essentiel de la première mission. Ce qui différencie les deux c'est le fait qu'ici, les manipulations seront faites sur des serveurs de production. Une collecte sera faite automatiquement tous les jours à 6h,12h et 18h afin de recolter le plus de données possibles ( resultat de la commande __df__ ) sur 99 serveurs. 


[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION2 %})



&nbsp;


### __Mission 3 : Création d'un wiki pour le client LOréal__

Création d'un wiki LOréal contenant différents guides autour de l'écosystème LOréal.



[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION3 %})


&nbsp;


### __Mission 4 : Création d'une maquette de serveur P9__

Cette mission conciste à créer une maquette d'un serveur P9 en utilisant le logiciel visio.



[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION4 %})



&nbsp;


### __Mission 5 : Remplacement de TSM par AVAMAR et export NFS pour le compte de Malakoff-Médéric Humanis__

__TSM :__  __T__ ivoli __S__ torage __M__ anager, par IBM.
__Avamar :__ Par DelL.

Cette mission est un peu problématique puisque certains serveurs de Malakoff Médéric Humanis tournent encore sur la version 5.3 d'AIX ( plus tenu à jour par IBM ). Le but serait de remplacer l'agent TSM ( Tivoli Storage Manager ) par l'agent XXXX. Cependant, le nouvel agent ne fonctionne qu'avec la version 7.X.X d'AIX. Il est donc nécessaire de mettre à jour les OS avant de remplacer l'agent.


[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION5 %})



&nbsp;


### __Mission 6 : Ajout d'une nouvelle paire de disques à un VG mirroré sur LOréal Wise AIX__

Pour cette mission, nous avions demandé à l'équipe Storage de supprimer de la data sur le filesystem __/dev/lvisisusrPT1__ pour libérer de l'espace puisque nous avions reçu un ticket indiquant que le fs venait d'atteindre un des palliers de seuil critique en terme d'espace libre. 95% étant le seuil critique, il venait d'atteindre 90% d'espace utililsé. Nous avons donc augmenter l'espace disponible du FS en y intégrant une nouvelle paire de disques.

[> Liste des compétences]({{ site.baseurl }}{% post_url 2019-12-13-MISSION6 %})



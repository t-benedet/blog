---
layout: post
title: Augmentation d'un FS sur server AIX
---

J'ai remarqué qu'un des filesystems sur un des servers AIX de Malakoff-Mederic Humanis était arrivé à 98% de sa taille totale grâce à la commande __df -g__ :
![image_1](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image1.png?raw=true)

Ici on voit qu'il s'agi du FS __/dev/saslv__.

Afin de pouvoir augmenter sa taille, nous avons besoin de connaître son __Volume Group__. Pour se faire, on tape la commande __lslv + nom du fs__. Donc ici __lslv saslv__ :
![image_2](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image2.png?raw=true)

Et on remarque que son __VG__ est __SASVG__.

Nous allons donc vérifier la taille disponible dans ce __VG__ grâce à la commande __lsvg + vg__, donc __lsvg SASVG__ :
![image_3](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image3.png?raw=true)

On remarqu'il reste 68480Mb, soit 68.48GB. Nous pouvons donc piocher dans ces quelques gigas pour augmenter la taille de notre __FS__.

Pour augmenter la taille du __FS__, il est impératif de garder en tête le nom du point de montage sur lequel est monté le __FS__. Si on reprend notre première image, on remarque que le __FS__ __/dev/saslv__ est monté sur __/SAS__. Nous allons donc taper la commande __chfs -a size=+XG point_de_montage__, donc __chfs -s size=+5G /SAS__ :
![image_4](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image4.png?raw=true)

On nous indique que la taille a bien été modifiée. Il suffit maintenant de retaper la commande __df -g__ pour s'en rendre compte :
![image_6](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image5.png?raw=true).

Nous constatons que le __FS__  est passé de 3.02G de libre et 98% d'espace utiliser à 8G de libre et 94% d'espace utiliser.

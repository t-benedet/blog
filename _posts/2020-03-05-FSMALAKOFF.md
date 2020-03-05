---
layout: post
title: Augmentation d'un FS sur server AIX
---

J'ai remarqué qu'un des filesystems sur un des servers AIX de Malakoff-Mederic Humanis était arrivé à 98% de sa taille totale grâce à la commande __df -g__ :
![image_1](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image1.png?raw=true)

Ici on voit qu'il s'agi du FS __/dev/saslv__.

Afin de pouvoir augmenter sa taille, nous avons besoin de connaître son __Volume Group__. Pour se faire, on tape la commande __lslv + nom du fs__. Donc ici __lslv saslv__ :
![image_2](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image2.png.raw=true)

Et on remarque que son __VG__ est __SASVG__.

Nous allons donc vérifier la taille disponible sans ce __VG__ grâce à la commande __lsvg + vg__, donc __lsvg SASVG__ :
![image_3](https://github.com/t-benedet/blog/blob/gh-pages/pictures/FS/image3.png?raw=true)

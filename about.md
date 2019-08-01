---
layout: page
title: I - C1.3.1.1 Mettre en place l’environnement de test du service
permalink: /CV/
---

----
&nbsp;
La partie Gnuplot étant terminée, nous allons passer à la demande concernant les FRAMES et les LPARS. L'idée sera de proposer :
- Un menu déroulant avec le nom des FRAMES, avec un bouton "Générate" qui affichera les informations sur le FRAME en question
- Un autre menu déroulant, faisant la même chose mais avec le nom des LPARS
- Deux autres menus déroulant, mais proposant cette fois-ci de visionner les informations entre deux dates choisies. 

Les scripts qui vont suivre sont donc intégrés dans une page html.
&nbsp;
#### Première étape : Le script pour les FRAMES

Pour cette étape, j'ai 276 csv dans lesquels je vais pouvoir piocher les informations dont j'ai besoin. Voici un exemple de fichier :

![image_1](http://image.noelshack.com/fichiers/2019/29/5/1563539849-frame1.png)

Pour chaques fichiers, 11 colonnes pour environ 1000 lignes. Les colonnes qui nous intéressent sont :
- La première avec le nom des FRAMES
- La deuxième avec le nom des LPARS
- La cinquième avec la quantité de RAM
- La sixième avec la quantité de CPU1 ( à définir )
- La septième avec la quantité de CPU2 ( à définir )

Voici ma première version du script qui traitera les FRAMES :
```
#!/bin/bash

OLDIFS=$IFS
IFS=","


while read FRAME RAM CPU1 CPU2
do
    if [[ $FRAME != $PREV ]]
    then
        PREV="$FRAME"
        echo -e "FRAME : $FRAME \
========================================\n"
fi

echo -e "RAM :\t$RAM\n\
CPU 1 :\t$CPU1\n\
CPU 2 :\t$CPU2\n"
echo ""
done < <(sort "my_csv_file.csv")
```

Le résultat est celui là :
```
FRAME_NAME 1 =======================
LPAR_NAME : LPAR_NAME1
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX


LPAR_NAME : LPAR_NAME1
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX


LPAR_NAME : LPAR_NAME1
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX

LPAR_NAME : LPAR_NAME1
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX

FRAME_NAME 2 =======================
LPAR_NAME : LPAR_NAME2
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX

LPAR_NAME : LPAR_NAME2
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX


FRAME_NAME3 =======================
LPAR_NAME : LPAR_NAME3
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX

LPAR_NAME : LPAR_NAME3
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX

LPAR_NAME : LPAR_NAME3
RAM : XXXXX
CPU 1 : XXXXX
CPU 2 : XXXXX
```

Ce script permet d'afficher, par __FRAMES__, les différents __LPARS__ ainsi que leurs consommations en RAM et CPU. Deux problèmes majeurs ici : 

- Ma boucle while ne permet pas d'analyser plusieurs fichiers csv
- Je ne sais pas de quels fichiers proviennent les informations. 

C'est donc sur ces deux points qu'il faut travailler. 

Une autre version du script serait celle-ci :
```
#!/bin/bash

OLDIFS=$IFS
IFS=","
for arg
do
	echo " File : $arg "

	while [[ $# -gt 0 ]]
	do
		echo $1
		while read FRAME RAM CPU1 CPU2
		do
			if [[ $FRAME != $PREV ]]
			then
				PREV="$FRAME"
				echo -e "FRAME : $FRAME \
				========================================\n"
			fi

			echo -e "RAM :\t$RAM\n\
			CPU 1 :\t$CPU1\n\
			CPU 2 :\t$CPU2\n"
			echo ""
		done < "$arg"
	done 
done

```

Là, il y a un mieux dans le résultat final :
- L'ajout de la boucle for permet d'afficher le fichier en cours d'exécution
- La deuxième boucle while permet de parcourir l'ensemble des fichiers 

Autre option, beaucoup plus propre, serait de passer par __awk__ :
```
cat /var/www/cgi-bin/LPAR_MAP/*.csv | grep $test |  awk -F ';|,' '{
                        print "FRAME : " $1
                        print "LPARS : " $2
                        print "RAM : " $5
                        print "CPU : " $6
                        print "CPU 2 : " $7
                        print " "
                     } '
```

Quelques explications :
- Le `cat /var/www/cgi-bin/LPAR_MAP/*.csv` => Affiche l'intégralité des lignes
- Le `grep $test` => Va afficher uniquement les lignes contenant le contenu de la variable $test
- Le reste du code va afficher FRAME/LPARS/RAM/CPU/CPU2 tout en prenant les les informations dans les colonnes 1,2,5,6 et 7

Je préfère cette version, beaucoup plus propre. Mais reste le même problème que pour la prmeière version du script : j'arrive à afficher tous les __FRAMES__ avec tous les __LPARS__ alors que je veux un __FRAME__ et ses __LPARS__. De plus, le `cat`, puis le `grep` puis le `awk` sont trop de procéssus à la suite, on perd en performance même si ça ne se voit pas. 

Finalement, la version finale est celle-ci :
```
read a
test=$( echo $a | cut -d'=' -f2)

echo '<PRE>'

echo "FRAME : $test "
echo "------------------"

echo ""

for fn in /var/www/cgi-bin/LPAR_MAP/*
do 
awk -F',|;' 'NR==1 { print "FILENAME ========================== : " FILENAME }
/'$test'/ { 
print ""
print "LPARS :" $2
print "RAM : " $5
print "CPU 1 : " $6
print "CPU 2 : " $7
print "" 
print ""}' $fn
done
```

Quelques explications :

- La boucle for permet de looper sur l'ensemble des .csv présents dans /var/www/cgi-bin/LPAR_MAP
- Le `awk -F',|;'` indique la virgul et le point-virgule comme délimiteur
- Pour `'NR==1 { print "FILENAME ========================== : " FILENAME }` => NR correspond à l'enregistrement courant (par défaut la ligne en cours de traitement). On n'affiche le nom du fichier ( avec la variable FILENAME ) que si la ligne en cours de traitement est la première ligne.
- Le `/'$test'/` est un regex qui permet déviter de faire un grep avec le nom du FRAME qui sera choisi par l'utilisateur

Ce script n'est pas terminé. Il faut encore ajouter une option "calcul", qui permettra de voir la différence de consommation entre les mois. Quoi qu'il en soit, si nous testons notre script via notre CGI :

![image_2](http://image.noelshack.com/fichiers/2019/29/5/1563543665-frame2.png)

On voit bien que dans cet exemple, le __FRAME__ choisi est le MO1PPC05, qu'il affiche correctement le nom des fichiers et les informations qui vont avec.
&nbsp;
#### Deuxième étape : Le script pour les LPARS 

Même chose que pour les FRAMES finalement. Je reprends donc mon bout de script pour les FRAMES et je remplace certaines valeurs :
```
read a
test=$( echo $a | cut -d'=' -f2)

echo '<PRE>'

echo "LPARS : $test "
echo "------------------"
echo ""

for fn in /var/www/cgi-bin/LPAR_MAP/*
do 
awk -F',|;' 'NR==1 { print "FILENAME ========================== : " FILENAME }
/'$test'/ { 

print ""
print "FRAME :" $1
print "RAM : " $5
print "CPU 1 : " $6
print "CPU 2 : " $7
print "" 
print ""}' $fn
done
```
Quelques explications :

- Au lieu d'afficher "FRAME : $Nom_du_FRAME", j'affiche le nom du LPARS qui m'intéresse
- Je remplace `print "LPARS :" $2` par `print "FRAME :" $1` histoire de récupérer le nom du FRAME dans lequel se situe le LPARS en question

Voici le résultat :

![image_3](http://image.noelshack.com/fichiers/2019/29/5/1563544338-lpars.png)

On remarque que le LPARS choisi par l'utilisateur et le 65-4855C, que les noms des fichiers sont bien indiqués et que nous avons le FRAME dans lequel se trouve le LPARS, avec sa RAM et ses CPU comme informations. Les deux premières lignes indiquent que le LPARS ne se trouve pas dans ces deux fichiers.

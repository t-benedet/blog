---
layout: post
title: Compétence C1.3.1.1
---
### __Mettre en place l’environnement de test du service__

&nbsp;
La partie Gnuplot étant terminée, nous allons passer à la demande concernant les FRAMES et les LPARS. L'idée sera de proposer :
- Un menu déroulant avec le nom des FRAMES, avec un bouton "Générate" qui affichera les informations sur le FRAME en question
- Un autre menu déroulant, faisant la même chose mais avec le nom des LPARS
- Deux autres menus déroulant, mais proposant cette fois-ci de visionner les informations entre deux dates choisies. 

Les scripts qui vont suivre sont donc intégrés dans une page html.
&nbsp;
#### __Première étape : Le script pour les FRAMES__

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
#!/bin/bash
...
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
#!/bin/bash
...

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
#### __Deuxième étape : Le script pour les LPARS__ 

Même chose que pour les FRAMES finalement. Je reprends donc mon bout de script pour les FRAMES et je remplace certaines valeurs :
```
#!/bin/bash
...

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

Retour sur notre script pour les FRAME et les LPARS. La boucle `for`, couplée au `awk` et au `regex` était déjà plus optimisée que la première version de notre script puisqu'ellle permettait de se passer du `cat`, du `grep` et de la boucle `while`.

Que ce soit pour les FRAMS ou les LPARS, la solution actuelle proposait un affichage beaucoup trop brut et pas très esthétique. De plus, je n'arrivais pas à récupérer simplement le nom du fichier, c'est le chemin complet qui s'affichait :

![image_1](http://image.noelshack.com/fichiers/2019/30/5/1564127978-frame2.png) 

![image_2](http://image.noelshack.com/fichiers/2019/30/5/1564128029-lpars.png)
&nbsp;

#### __Première étape : Modification du script, tableau et nom__

Notre script actuel est celui-ci ( je ne montre pas la partie html puisque c'est le code de base d'une page web ) :
```
#!/bin/bash
...

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
&nbsp;
Comme dit plus haut, le résultat était trop brute. Je tente donc d'intégrer ma boucle `for` entre des tag __< table >__ histoire de pouvoir générer un tableau. Le but ici sera de créer une nouvelle colonne à chaque " Date ============= XXXX " rencontré. Mon premier essai est celui ci :
```
#!/bin/bash
...

read a
test=$( echo $a | cut -d'=' -f2)

echo "<p><h2>FRAME : $test</h2></p>"

echo "<table>"
for fn in /var/www/cgi-bin/LPAR_MAP/*;
    do
      echo "<td>"
      echo "<PRE>"

      awk -F',|;' 'NR==1 { print "FILENAME ========================== : " FILENAME }
      /'$test'/ { 
      print ""
      print "LPARS : " $2
      print "RAM : " $5
      print "CPU 1 : " $6
      print "CPU 2 : " $7
                }' $fn
                
echo "</PRE>"
echo "</td>"
     done  
echo "</table>"
```
J'ai simplement ajouté de quoi créer un tableau. 
&nbsp;

Le second problème, c'est le nom de mon fichier qui ne s'affiche pas comme je le souhaite. Après plusieurs recherches ( merci StackOverflow ). Une des solutions que j'ai trouvé consiste à modifier le `awk` actuel par celui-ci :
```
awk -F',|;' 'NR==1 { split(FILENAME ,a,"[-.]");print "DATE ========================== : " a[4] }
 /'$test'/ { 
```

Quelques explications :

- La commande split va " divisier " le contenu de la variable FILENAME par `-` et `.` et va stocker le résultat dans le tableau a. La date sera alors le quatrième bloc du tableau a => a[4] *( Voir man Awk, section **split(**_s_**,** _a_ [**,** _r_]**)** )*.

Le résultat est tout à fait satisfaisant :

![image_3](http://image.noelshack.com/fichiers/2019/30/5/1564134900-dateok.png)
&nbsp;

#### __Deuxième partie : Suppression des lignes inutiles__ 

Comme mon script analyse et prend en compte toutes les lignes des CSV, des colonnes vont se créer avec l'entête " Date =========== XXX " sans pour autant afficher de résultat puisque les lignes dans les différents CSV seront vides :

![image_4](http://image.noelshack.com/fichiers/2019/30/5/1564135135-lignesvides.png)

Nous allons donc essayer de les supprimer. Encore une fois, je ne vais pas réinventer la roue ( j'ai quand même cherché un peu... ). 

La réponse est sur StackOverflow. Les modifications sont à faire au niveau du `awk`  :

```
awk -F',|;' -v test="$test" '
     NR==1 { 
        split(FILENAME ,a,"[-.]");
      }
      $0 ~ test {
          if(!header++){
```
Quelques explications :
```

```

Le script final est donc :
```
#!/bin/bash
...

read a
test=$( echo $a | cut -d'=' -f2)


echo "<p><h2>FRAME : $test</h2></p>"

echo "<table>"
for fn in /var/www/cgi-bin/LPAR_MAP/*;
do
echo "<td>"
echo "<PRE>"

awk -F',|;' -v test="$test" '
     NR==1 { 
        split(FILENAME ,a,"[-.]");
      }
      $0 ~ test {
          if(!header++){
              print "DATE ========================== : " a[4] 
          }
          print ""
          print "LPARS :" $2
          print "RAM : " $5
          print "CPU 1 : " $6
          print "CPU 2 : " $7
          print "" 
          print ""
      }' $fn;

echo "</PRE>"
echo "</td>"
done
echo "</table>"
```
&nbsp;

Et le résultat :

![image_5](http://image.noelshack.com/fichiers/2019/30/5/1564143847-colonnevideok.png)

On remarque sur la gauche, les colonnes sont vides. Le script n'affiche plus les dates s'il ne trouve pas de résultat. 
En revanche, nouveau problème : même s'il n'y a plus d'affichage pour les lignes vides, notre script génère quand même des colonnes vides. Il faut donc les supprimer.

&nbsp;

#### __Troisième partie : Rappel de ce qu'il faut faire et changement du script__

J'avais mal compris ce qu'il fallaire. En plus de ça, la personne qui m'a demandée ce script ne m'avait pas donné le bon fichier... Donc forcément...

Ce qu'il fallait faire : Afficher uniquement les lignes où il y a un changement de consommation de RAM et CPU. Au final, c'est plus compliqué que prévu. Je dois trouver le moyen de faire sauter les lignes qui sont identiques dans chaques fichiers ( si les lignes sont identiques pour le même frame d'une date à une autre, c'est qu'il n'y a pas eu de changement dans la consommation de RAM et CPU et ne doivent pas être taitées ).

Après quelques recherches, je tombe sur la commande `awk '!a[$0]++'` qui permet de supprimer les lignes qui sont dupliquées et donc, de ne garder que celles qui sont différentes ( donc dans mon cas, les lignes différentes représentent les moments où il y a eu un changement de consommation en RAM et CPU ). J'ai essayé de l'intégrer à mon script de base, mais je n'y arrivais pas. J'ai donc décidé de tout reprendre à zéro. 

Mon nouveau script ressemble à ça :

```
#!/bin/bash
...

awk -F",|;" ' {$0=$1","$2","$5","$6","$7 } /'$test'/ { if (!a[$0]++)
{ 
print "" printf "LPARS : %s\n", $2
printf "RAM : %s\n", $3 
printf "CPU1 : %s\n", $4 
printf "CPU2 : %s\n", $5 
print ""
} 
}' /var/www/cgi-bin/LPAR_MAP/*.csv ;
```

Quelques explications :

- Je fais un awk sur les colonnes 1,2,3,5,6 et 7.
- `a[$0]` regarde la valeur de la clé `$0` dans le tableau `a`.
- `a[$0]++` incrémante la valeur `$0`, retourne l'ancienne valeur comme valeur par défaut. S'il n'existe pas de `[$0]`, on retourne 0 et incrémente à 1.
- `!a[$0]++` annule la valeur de l'expression. Si `[$0]++` renvoie 0, alors toute l'expression est vraie et on affiche le ce que `$0` contient par défaut.
- On garde la regex `/'$test'/` qui remplace le `grep $test`


Il ne reste plus qu'à intégrer ce script dans une boucle `for` et entre des tags `html`, d'ajouter la variable `FILENAME` histoire de pouvoir afficher le nom du fichier en cours d'analyse et de récupérer que la partire __date__ :
```
#!/bin/bash
...

echo "<table>" 
for fn in /var/www/cgi-bin/LPAR_MAP/*.csv; 
do 
echo "<td>" 
echo "<PRE>"  
awk -F',|;' '{ $0=$1","$2","$5","$6","$7} /'$test'/ { if (!a[$0]++)  
{ 
print "DATE ================= : " FILENAME  
printf "LPARS : %s\n", $2 
printf "RAM : %s\n", $3 
printf "CPU1 : %s\n", $4 
printf "CPU2 : %s\n", $5 
print "" 
} 
}' $fn  
echo "</PRE>" 
echo "</td>" 
done 
echo "</table">
```

Voici le résultat :

![image_6](http://image.noelshack.com/fichiers/2019/33/1/1565601797-1565360660-lk.png)

Avec le FRAME __MIAIBYE00__ je suis passé de 276 colonnes avec X lignes, à seulement 40 colonnes et quelques lignes. La commande `awk '!a[$0]++'` fonctionne donc correctement. Les informations sont bien affichées en-dessous de leur date, mais le résultat n'est pas celui voulu :

- Il faut que la date ne soit affichée qu'une seule fois avec, en-dessous, toutes  les informations du fichier, et non pas une date/une information, une date/information etc...
- Des colonnes se sont bien crées... Mais pas comme il le fallait. Ici, toutes les colonnes sont identiques, or il me faut une colonne par date.

Il faut donc encore apporter quelques modifications.

Faire une carte choroplete avec R et ggplot2
================

Pré-requis 1: Le shapefile
--------------------------

Pour faire une carte avec le logiciel R vous avez besoin d'un fichier qui délimite les contours géographiques souvent connu sous le nom de shapefile. Vous pouvez télécharger le shapefile de n'importe quel pays gratuitement sur ce [site](https://gadm.org/download_country_v3.html). Il vous suffit d'entrer le nom du pays dans la barre de recherche. Pour cet exemple nous utiliserons le shapefile du Sénégal qui est téléchargeable [ici](https://biogeo.ucdavis.edu/data/gadm3.6/shp/gadm36_SEN_shp.zip). Vous pouver décompresser le fichier zip et mettre le contenu dans votre dossier de travail.

Pré-requis 2: Vos données
-------------------------

Dans l'exemple suivant,il a été préparé un fichier de données dans laquelle on peut avoir les informations des coordonnées géographiques, le nom des régions de même que les productions de mil (en tonne) suivant ces régions.Notez qu'ici, une colonne nommée "id" a été créée. Cette colonne porte les identifiants du nom des régions tel que définit sur le shapefile en question. Ceci nous permettra plus tard de joindre les données de contour venant du shapefile à celle de nos données.Plus d'amples explicaions seront données dans les lignes de codes.

Pré-requis 3:Installation des packages
--------------------------------------

Pour installer un package dans R Studio, il vous suffit juste de cliquer sur "Packages", puis "Install" et d'y insérer le nom de votre package. Une autre maniere de faire est la suivante:

    install.packages("le_nom_du_package", dependencies=TRUE)

Notez qu'ici "dependencies = TRUE" veut simplement dire de prendre en compte les sous-modules constitutifs du package en question.Pour cette séries de tutoriels, nous aurons besoin des packages:
-   rgdal;
-   mapdata ;
-   mapproj ;
-   maps ;
-   ggplot2 ;
-   ggrepel ;
-   legendMap.

A présent, passons à la pratique!
---------------------------------

**Etape 1: Définir notre répertoire de travail**

``` r
setwd("C:/Users/ANGE/Documents/R MAP")
```

**Etape 2: Charger les packages**

``` r
library(rgdal)
library(mapdata)
library(mapproj)
library(maps)
library(ggplot2)
library(ggrepel)
library(legendMap)
library(dplyr)
```

**Etape 3: Définir l'adresse du dossier contenant votre shapefile**

Pour faciliter d'usage, nous garderons le même dossier que celui précédemment défini.

``` r
mySHP="C:/Users/ANGE/Documents/R MAP"
```

**Etape 4: Importer le shapefile**

``` r
myFile=readOGR(mySHP, layer="SEN_adm1", stringsAsFactors=FALSE)
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "C:\Users\ANGE\Documents\R MAP", layer: "SEN_adm1"
    ## with 14 features
    ## It has 9 fields
    ## Integer64 fields read as strings:  ID_0 ID_1

***SEN\_adm1 est le nom du shapefile représentant les régions du Sénégal au moment du téléchargement du fichier zip en 2017. En 2020, le nouveau nom du fichier shapefile mis à jour est gadm36\_SEN\_1. Vous pouvez donc changer le nom du fichier à votre convenance***

**Etape 5: Accéder au données inhérantes au shapefile**

Saisir :

    myFile@data

Les données présentées dans le tableau 1 s'afficheront.

Table 1: Tableau de données du shapefile.

|     | ID\_0 | ISO | NAME\_0 | ID\_1 | NAME\_1     | TYPE\_1   | ENGTYPE\_1 | NL\_NAME\_1 | VARNAME\_1                 |
|-----|:------|:----|:--------|:------|:------------|:----------|:-----------|:------------|:---------------------------|
| 0   | 201   | SEN | Senegal | 1     | Dakar       | RÃÂ©gion | Region     | NA          | Cap Vert|Dacar             |
| 1   | 201   | SEN | Senegal | 2     | Diourbel    | RÃÂ©gion | Region     | NA          | NA                         |
| 2   | 201   | SEN | Senegal | 3     | Fatick      | RÃÂ©gion | Region     | NA          | NA                         |
| 3   | 201   | SEN | Senegal | 4     | KÃÂ©dougou | RÃÂ©gio  | n Region   | NA          | SÃÂ©nÃÂ©gal Oriental     |
| 4   | 201   | SEN | Senegal | 5     | Kaffrine    | RÃÂ©gion | Region     | NA          | NA                         |
| 5   | 201   | SEN | Senegal | 6     | Kaolack     | RÃÂ©gion | Region     | NA          | NA                         |
| 6   | 201   | SEN | Senegal | 7     | Kolda       | RÃÂ©gion | Region     | NA          | NA                         |
| 7   | 201   | SEN | Senegal | 8     | Louga       | RÃÂ©gion | Region     | NA          | NA                         |
| 8   | 201   | SEN | Senegal | 9     | Matam       | RÃÂ©gion | Region     | NA          | NA                         |
| 9   | 201   | SEN | Senegal | 10    | SÃÂ©dhiou  | RÃÂ©gio  | n Region   | NA          | NA                         |
| 10  | 201   | SEN | Senegal | 11    | Saint-Louis | RÃÂ©gion | Region     | NA          | Fleuve|VallÃÂ©e du Fleuve |
| 11  | 201   | SEN | Senegal | 12    | Tambacounda | RÃÂ©gion | Region     | NA          | SÃÂ©nÃÂ©gal Oriental     |
| 12  | 201   | SEN | Senegal | 13    | ThiÃÂ¨s    | RÃÂ©gio  | n Region   | NA          | NA                         |
| 13  | 201   | SEN | Senegal | 14    | Ziguinchor  | RÃÂ©gion | Region     | NA          | NA                         |

Cette information vous sera utile pour les étapes 6 et concernant la jointure de vosdonnées à celle du shapefile.

**Etape 6: Changer le format du shapefile en "dataframe" (en style tableau).**

Cette étape est cruciale pour que le package ggplot2 puisse implimenter les coordonées comme celà se doit.

``` r
myDF = fortify(myFile, region = "NAME_1")
```

Vous remarquerez le détails concernant l'argument "region" qui est "NAME\_1". C'est simplement l'identifiant de chaque région (Voir la colonne 6 du tableau 1).

**Etape 7: Voir les 6 premieres lignes du nouveau fichier transformé**

Tableau 2: Données du shapefile transformées au format tableau

|       long|       lat|  order| hole  | piece | id    | group   |
|----------:|---------:|------:|:------|:------|:------|:--------|
|  -17.16056|  14.89375|      1| FALSE | 1     | Dakar | Dakar.1 |
|  -17.16004|  14.89333|      2| FALSE | 1     | Dakar | Dakar.1 |
|  -17.16000|  14.89335|      3| FALSE | 1     | Dakar | Dakar.1 |
|  -17.15683|  14.89042|      4| FALSE | 1     | Dakar | Dakar.1 |
|  -17.15592|  14.88957|      5| FALSE | 1     | Dakar | Dakar.1 |
|  -17.15551|  14.88920|      6| FALSE | 1     | Dakar | Dakar.1 |

Vous remarquerez long et lat comme nom des colonnes 1 et 2 respectivement. Nous allons changer celà en Longitude et Latitude dans l'étape suivante.

**Etape 8: Renommer les colonnes long et lat en Longitude et Latitude**

``` r
myDF = rename(myDF, Longitude = long, Latitude= lat)
```

**Etape 9: Importation de nos données**

``` r
mydata = read.csv("production.csv", header=TRUE, sep=";", dec=",")
```

Tableau 3: Tableau de données de production du mil suivant les régions du Sénégal

|    long|    lat| Region      |  Production| id          |
|-------:|------:|:------------|-----------:|:------------|
|  -17.50|  14.72| Dakar       |           0| Dakar       |
|  -16.21|  14.79| Diourbel    |       61982| Diourbel    |
|  -16.52|  14.00| Fatick      |      182000| Fatick      |
|  -15.08|  14.10| Kaolack     |      144938| Kaolack     |
|  -14.56|  12.94| Kolda       |       41023| Kolda       |
|  -15.30|  15.20| Louga       |       26390| Louga       |
|  -15.10|  16.30| Saint-Louis |          31| Saint-Louis |
|  -13.64|  13.75| Tambacounda |       44855| Tambacounda |
|  -16.92|  14.76| Thies       |       57318| ThiÃÂ¨s    |
|  -16.40|  12.73| Ziguinchor  |        3878| Ziguinchor  |
|  -13.80|  15.20| Matam       |        5366| Matam       |
|  -15.87|  13.70| Kaffrine    |      171560| Kaffrine    |
|  -12.52|  12.80| Kedougou    |           0| KÃÂ©dougou |
|  -15.56|  12.79| Sedhiou     |       67703| SÃÂ©dhiou  |

\***Détail important**

La colonne "id" de nos données porte les mêmes noms que celle du shapefile. Vous remarquerez que l'orthographe des noms est un bizarre. C'est dû aux accents dans la langue française. Ne vous inquitez pas pour celà. Le plus important est d'avoir les mêmes identifiants tant au niveau du shapefile qu'au niveau de vos données afin de faciliter la jonction qui aura lieu à l'étape 10.

**Etape 10: Joindre vos données à celles du shapefile**

Pour la jonction, nous utiliserons la fonction "left\_join" (qui veut dire littéralement : Joindre par la gauche).

``` r
plotData <- left_join(myDF, mydata)
```

    ## Joining, by = "id"

    ## Warning: Column `id` joining character vector and factor, coercing into
    ## character vector

Ne vous inquiétez pas si vous voyez le message suivant:

    ## Joining, by = "id"
    ## Warning: Column `id` joining character vector and factor, coercing into
    ## character vector

C'est juste un avertissement relatif à la jonction des deux jeux de données.

**Etape 11: Faire la carte**

``` r
p <- ggplot() +
  
     geom_polygon(data = plotData, aes(x = Longitude, 
                                    y = Latitude, 
                                    group = group, 
                                    fill = Production), 
                                    color = "black", size = 0.15) + #Dessiner le contour de la carte du Senegal avec les regions
  
     geom_text(data=mydata, aes(x=long, 
                             y=lat, 
                             label=Region), 
                             fontface = 'bold')+ #Ajouter le nom des regions suivant les coordonnees geographiques
  
     scale_fill_gradient(name="Production\nde mil\n(tonne)",# Mettre le nom de la legende. \n permet d'aller a la ligne
                      limits = c(0, 190000),# Definir une echelle de valeur selon vos donnees
                      low="white", high="forestgreen")+ #Definir une echelle de couleur pour distinguer les regions suivant leur production
  
     coord_map(projection = "lagrange") + #Definir le type de projection voulue. Ici, la projection de Lagrange.
  
     scale_bar(lon = -17.7, lat = 13, # La position du Nord eographique
            distance_lon = 40, distance_lat = 10, # Les mesures du symbole du Nord georaphique 
                                                  #en termes de longueur (distance_lat) et de hauteur (distance_lon)
            distance_legend = 25, dist_unit = "km", # La distance relative de la legende du symbole du Nord geographique
            arrow_length = 10, # Definir la longueur de la fleche du symbole du Nord geographique
            arrow_distance = 50, # Definir la distance separant la fleche de la barre d'echelle du symbole du Nord geographique
            arrow_north_size = 6)+ # Definir la taille de la fleche du symbole du Nord geographique
            
            theme_void() # Definir un style d'arriere-plan, Ici, rien en arriere plan.
```

Et Voici le rendu...

![](Carte-avec-RStudio_files/figure-markdown_github/unnamed-chunk-13-1.png)

Si au besoin vous voulez customiser votre carte, vous pouvez changer l'arriere-plan ou agir sur les noms des régions par example.

Ci-dessous le code

``` r
g <- ggplot() +
  
     geom_polygon(data = plotData, aes(x = Longitude, 
                                    y = Latitude, 
                                    group = group, 
                                    fill = Production), 
                                    color = "black", size = 0.15) + 
  
     geom_point(data=mydata, aes(x=long, y=lat), shape = 21,fill = "white",size = 3, color = "black") +
  
     geom_label_repel(data=mydata, aes(x=long, y=lat, label=Region),
        fontface = 'bold', color = 'black',
        box.padding = 0.35, point.padding = 0.5,
        segment.color = 'grey10') +
  
     scale_fill_gradient(name="Production\nde mil\n(tonne)",
                      limits = c(0, 190000),
                      low="white", high="forestgreen")+ 
  
     coord_map(projection = "lagrange") + 
     scale_bar(lon = -17.7, lat = 13, 
            distance_lon = 40, distance_lat = 10, 
            distance_legend = 25, dist_unit = "km", 
            arrow_length = 10, 
            arrow_distance = 50, 
            arrow_north_size = 6) + 
  
      theme_minimal() +
      theme(panel.grid.major = element_line(colour = "black", size = 0.5, linetype = "dotted")) +
      theme(plot.background = element_rect(colour = "white", size = 1)) +
      ggtitle("Production de mil")
```

Voici le rendu...

![](Carte-avec-RStudio_files/figure-markdown_github/unnamed-chunk-15-1.png)

Exportez votre carte avec une haute résolution
----------------------------------------------

Exportez au format vectoriel pdf

    ggsave(p, file = "carte.pdf", limitsize = FALSE, width = 12, height = 10.5, dpi=500 )

Exportez au format Cairo PNG

    ggsave(p, file = "carte.png", limitsize = FALSE, width = 10, height = 6.5, type = "cairo-png", dpi=500)

Vous pouvez ajuster les dimensions et la résolution (dpi) selon votre bon-vouloir. Le dpi (pixelisation) par défaut est de 300.

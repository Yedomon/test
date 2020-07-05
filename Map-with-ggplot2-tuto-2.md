R map with ggplot2: Emphasize a specific region
================

Data source
-----------

The data used for this tutorial are available [here](http://downloads.hindawi.com/journals/tswj/2019/1252653.f1.xlsx)

Note
----

The senegal shape file was downloaded [here](https://biogeo.ucdavis.edu/data/gadm3.6/shp/gadm36_SEN_shp.zip)

This tutorial is based on [this published paper](https://www.hindawi.com/journals/tswj/2019/1252653/)

Make sure you've installed all requiered packages.

Code
----

``` r
#Reset R's brain
rm(list=ls())

#libraries loading
library(rgdal)
```

    ## Loading required package: sp

    ## rgdal: version: 1.3-6, (SVN revision 773)
    ##  Geospatial Data Abstraction Library extensions to R successfully loaded
    ##  Loaded GDAL runtime: GDAL 2.2.3, released 2017/11/20
    ##  Path to GDAL shared files: C:/Users/ANGE/Documents/R/win-library/3.5/rgdal/gdal
    ##  GDAL binary built with GEOS: TRUE 
    ##  Loaded PROJ.4 runtime: Rel. 4.9.3, 15 August 2016, [PJ_VERSION: 493]
    ##  Path to PROJ.4 shared files: C:/Users/ANGE/Documents/R/win-library/3.5/rgdal/proj
    ##  Linking to sp version: 1.3-1

``` r
library(mapdata)
```

    ## Loading required package: maps

``` r
library(mapproj)
library(maps)
library(ggplot2)
library(ggrepel)
library(legendMap)
```

    ## Loading required package: maptools

    ## Checking rgeos availability: TRUE

    ## Loading required package: grid

``` r
library(dplyr)
```

    ## Warning: package 'dplyr' was built under R version 3.5.3

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
#Set the working directory
setwd("C:/Users/ANGE/Documents/R MAP")

#Set the shapefile path
mySHP="C:/Users/ANGE/Documents/R MAP"

#Import the shapefile
myFile=readOGR (mySHP, layer="SEN_adm1", stringsAsFactors=FALSE)
```

    ## OGR data source with driver: ESRI Shapefile 
    ## Source: "C:\Users\ANGE\Documents\R MAP", layer: "SEN_adm1"
    ## with 14 features
    ## It has 9 fields
    ## Integer64 fields read as strings:  ID_0 ID_1

``` r
#See the data embeded with the shapefile
head(myFile@data, 4)
```

    ##   ID_0 ISO  NAME_0 ID_1    NAME_1  TYPE_1 ENGTYPE_1 NL_NAME_1
    ## 0  201 SEN Senegal    1     Dakar RÃÂ©gion    Region      <NA>
    ## 1  201 SEN Senegal    2  Diourbel RÃÂ©gion    Region      <NA>
    ## 2  201 SEN Senegal    3    Fatick RÃÂ©gion    Region      <NA>
    ## 3  201 SEN Senegal    4 KÃÂ©dougou RÃÂ©gion    Region      <NA>
    ##            VARNAME_1
    ## 0     Cap Vert|Dacar
    ## 1               <NA>
    ## 2               <NA>
    ## 3 SÃÂ©nÃÂ©gal Oriental

``` r
# Import regions's coordinates
mydata2 = read.csv("data2.csv", header=TRUE, sep=";")

#Overview of the mydata2

head(mydata2, 4)
```

    ##     NAME_1 Production Unity       Prod   long   lat              Produc
    ## 1 DIOURBEL      46231  tons 46231 tons -16.25 14.75 DIOURBEL 46231 tons
    ## 2   FATICK      80000  tons 80000 tons -16.53 14.36   FATICK 80000 tons
    ## 3  KAOLACK      77613  tons 77613 tons -16.00 14.00  KAOLACK 77613 tons
    ## 4    KOLDA      11930  tons 11930 tons -14.00 13.00    KOLDA 11930 tons
    ##                   Prodd
    ## 1 DIOURBEL (46231 tons)
    ## 2   FATICK (80000 tons)
    ## 3  KAOLACK (77613 tons)
    ## 4    KOLDA (11930 tons)

``` r
#Selected the regions of interest

myregion = subset(myFile, myFile$ID_1 == "13" | 
                    myFile$ID_1 == "2" | 
                    myFile$ID_1 == "3" |
                    myFile$ID_1 == "6" |
                    myFile$ID_1 == "5" |
                    myFile$ID_1 == "12" |
                    myFile$ID_1 == "7" |
                    myFile$ID_1 == "10")



#Change region's shapefile into a datframe
myreg = fortify(myregion, region = "NAME_1")

#Change the Senegal shapefile into a dataframe
myDF = fortify(myFile)
```

    ## Regions defined for each Polygons

``` r
#Change long and lat by Longitude and Latitude

myDF = rename(myDF, Longitude = long, Latitude= lat)

myreg = rename(myreg, Longitude = long, Latitude= lat)

#Import data to join to the region plot
mydata = read.csv("dat_revu.csv", header=TRUE, sep=";")


#Join the imported data with the region data
plotData <- left_join(myreg, mydata)
```

    ## Joining, by = "id"

    ## Warning: Column `id` joining character vector and factor, coercing into
    ## character vector

``` r
#Overview of the present data

head(plotData, 4)
```

    ##   Longitude Latitude order  hole piece       id      group num      long
    ## 1  -16.3588 15.01771     1 FALSE     1 Diourbel Diourbel.1   5 -16.51606
    ## 2  -16.3588 15.01771     1 FALSE     1 Diourbel Diourbel.1   6 -16.44947
    ## 3  -16.3588 15.01771     1 FALSE     1 Diourbel Diourbel.1   7 -16.43950
    ## 4  -16.3588 15.01771     1 FALSE     1 Diourbel Diourbel.1   8 -16.41733
    ##        lat label  Inc1  area1  Arae   area   Region areatot nplantdis
    ## 1 14.70861    a5 23.09 104.86 10486 104.86 Diourbel   20142        85
    ## 2 14.67858    a6 17.90 443.85 44385 443.85 Diourbel   11485        65
    ## 3 14.65472    a7 10.86  25.54  2554  25.54 Diourbel   10423        40
    ## 4 14.59847    a8 45.98 214.77 21477 214.77 Diourbel   20386       172
    ##   ntotalplt plant      Icn      Inc
    ## 1       368 Souna 2309.783 23.09783
    ## 2       363 Souna 1790.634 17.90634
    ## 3       368 Souna 1086.957 10.86957
    ## 4       374 Souna 4598.930 45.98930

``` r
#Make the plot
p <-  ggplot() +
      geom_polygon(data=myDF, linetype = "dotted", aes(x=Longitude, y=Latitude, group = group),colour="black", fill="white") +
      geom_polygon(data=myreg, linetype = "dotted", aes(x=Longitude, y=Latitude, group = group),colour="black", fill="grey") +
      geom_point(data=mydata2, aes(x=long, y=lat), shape = 21,fill = "white",size = 3, color = "black") +
      geom_label_repel(
        data=mydata2, aes(x=long, y=lat, label=NAME_1),
        fontface = 'bold', color = 'black',
        box.padding = 0.35, point.padding = 0.5,
        segment.color = 'grey10'
      ) +
      coord_map(projection = "lagrange") +
      scale_bar(lon = -17.75, lat = 13, 
                         distance_lon = 40, distance_lat = 10, 
                         distance_legend = 25, dist_unit = "km", 
                         arrow_length = 10, arrow_distance = 50, arrow_north_size = 6) +
      theme_minimal() +
      theme(panel.grid.major = element_line(colour = "black", size = 0.5, linetype = "dotted")) +
      theme(plot.background = element_rect(colour = "white", size = 1)) +
      ggtitle("Map of the most productive pearl millet regions in Senegal during the rainy season 2017")
```

Map rendering
-------------

Just call the map variable p

![](Map-with-ggplot2-tuto-2_files/figure-markdown_github/unnamed-chunk-2-1.png)

That is it!

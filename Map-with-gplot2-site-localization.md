Map with ggplot2: Site localization
================

Data source
-----------

The data used for this tutorial are available [here](http://downloads.hindawi.com/journals/tswj/2019/1252653.f1.xlsx)

Note
----

The senegal shape file was downloaded [here](https://biogeo.ucdavis.edu/data/gadm3.6/shp/gadm36_SEN_shp.zip)

This tutorial is based on [this published paper](https://www.hindawi.com/journals/tswj/2019/1252653/)

Make sure you've installed all required packages.

Code
----

``` r
#Reset R's brain
rm(list=ls())

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
#Focus on the regions of interest

myregion = subset(myFile, myFile$ID_1 == "13" | 
                    myFile$ID_1 == "2" | 
                    myFile$ID_1 == "3" |
                    myFile$ID_1 == "6" |
                    myFile$ID_1 == "5" |
                    myFile$ID_1 == "12" |
                    myFile$ID_1 == "7" |
                    myFile$ID_1 == "10")



#Transform into the dataframe
myreg = fortify(myregion, region = "NAME_1")

#Rename long and lat 
myreg = rename(myreg, Longitude = long, Latitude= lat)

#Transform the shapefile into a dataframe
myDF = fortify(myFile)
```

    ## Regions defined for each Polygons

``` r
#Change long and lat into Longitude and Latitude
myDF = rename(myDF, Longitude = long, Latitude= lat)

#Import my data
mydata = read.csv("dat_revu.csv", header=TRUE, sep=";")


#Join the data to plot
plotData <- left_join(myreg, mydata)
```

    ## Joining, by = "id"

    ## Warning: Column `id` joining character vector and factor, coercing into
    ## character vector

``` r
#Make the plot

p <-  ggplot() +
      geom_polygon(data=myreg, linetype = "dotted", aes(x=Longitude, y=Latitude, group = group),colour="black", fill="white") +
      coord_map(projection = "lagrange") +
      scale_bar(lon = -17.75, lat = 13, 
                         distance_lon = 40, distance_lat = 10, 
                         distance_legend = 25, dist_unit = "km", 
                         arrow_length = 10, arrow_distance = 50, arrow_north_size = 6)+
      geom_point(data=mydata, aes(x=long, y=lat), color = "grey") +
      geom_label_repel(
        data=mydata, aes(x=long, y=lat, label=label, fill = Region),
        fontface = 'bold', color = 'white',
        box.padding = 0.35, point.padding = 0.5,
        segment.color = 'grey'
      ) +
      theme_minimal() +
      theme(panel.grid.major = element_line(colour = "black", size = 0.5, linetype = "dotted")) +
      theme(plot.background = element_rect(colour = "white", size = 1)) +
      ggtitle("Map of sampling sites")
```

Map rendering
-------------

Just call the map variable p

![](Map-with-gplot2-site-localization_files/figure-markdown_github/unnamed-chunk-2-1.png)

That is it!

But I am struggling to remove the "a" in the legend.So I renamed the sites with a. Any help will be welcomed.

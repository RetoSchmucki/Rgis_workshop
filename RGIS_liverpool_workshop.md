---
layout: post
title: RGIS workshop liverpool
subtitle: A BES QE SIG training event
date: 2018-03-12 9:00:00
author: Reto Schmucki
meta: "Tutorials"
---


### Tutorial Aims:

#### <a href="#SpatialObject"> 1. What is a spatial object? </a>
#### <a href="#LoadManipulate"> 2. Load and manipulate spatial objects. </a>
#### <a href="#Extracting"> 3. Merge, extract and map spatial object. </a>
#### <a href="#PostgreSQL"> 4. Interfacing R and PostgreSQL/PostGIS. </a>

<p></p>

<a name="SpatialObject"></a>


##### What is a spatial object?

A spatial object is an entity with coordinates in a geographical space (x, y) or (x, y, z) with a specific projection.

> **Coordinates are not enough!**

```r
## example

point_liverpool <- data.frame(name = 'Liverpool', longitude = -2.98, latitude = 53.41)
point_edinburgh <- data.frame(name = 'Edinburgh', longitude = -3.19, latitude = 55.95)

city_points <- rbind(point_liverpool, point_edinburgh)

plot(city_points[,c("longitude", "latitude")], pch = 19, col = 'magenta')
text(city_points[,c("longitude", "latitude")], labels = city_points$name, pos = c(2, 4))
```

With these two cities, although the coordinates are right, you have no information about the context of their coordinates.

```r
proj_city_points <- sf::st_as_sf(city_points, coords = c("longitude", "latitude"), crs = 4326)
```

> **Spatial projection**
>
> The Coordinate Reference System or CRS of a spatial object tells R where the spatial object is located in geographic space (*see* http://spatialreference.org/).

The common longitude-latitude (degree decimal)
EPSG Projection 4326 - WGS84

```r
# Adding a projection to our cities, using WGS84
proj_city_points <- sf::st_as_sf(city_points, coords = c("longitude", "latitude"), crs = 4326)

# get some base layer
country_sf_gbr <- sf::st_as_sf(raster::getData(name = "GADM", country = 'GBR', level = 1))

# visualize the points on a map
plot(country_sf_gbr$geometry, graticule = TRUE, axes = TRUE, col = "goldenrod1")
plot(proj_city_points, pch = 19, col = c("magenta", "blue"), cex = 1.5, add = TRUE)
legend("topleft", legend = proj_city_points$name, col = c("magenta", "blue"), pch = 19, cex = 1.5, bty="n")
```

> **Change Projection**
>
> Some projections more appropriate for representing specific geographic context
> Unit and reference point (distortion due to representation of a spheric shape in 2D)

```r
# reproject in OSGB 1936 / British National Grid

proj_city_points_osgb <- sf::st_transform(proj_city_points, crs = 27700)
country_sf_gbr_osgb <- sf::st_transform(country_sf.gbr, crs = 27700)

# visualize the points on a map
dev.new()
plot(country_sf_gbr_osgb$geometry, graticule = TRUE, axes = TRUE, col = "goldenrod1")
plot(proj_city_points_osgb, pch = 19, col = c("magenta", "blue"), cex = 1.5, add = TRUE)
legend("topleft", legend = proj_city_points_osgb$name, col = c("magenta", "blue"), pch = 19, cex = 1.5, bty="n")
```

plot(country_sf_gbr_osgb$geometry, graticule = , axes = TRUE, col = "goldenrod1")



<a name="LoadManipulate"></a>
##### Load and manipulate spatial objects

Spatial data are increasingly available from the Web, from species occurrence to natural and  cultural features data, accessing spatial data is now relatively easy. For base layers, you can find many freely available data sets such as the ones provided by the Natural Earth [http://www.naturalearthdata.com], the IUCN Protected Planet database [www.protectedplanet.net], the GADM project [https://gadm.org], worldclim [http://worldclim.org/version2] the CHELSA climate data sets [http://chelsa-climate.org] or the European Environmental Agency [https://www.eea.europa.eu/data-and-maps/data#c0=5&c11=&c5=all&b_start=0]

#### Raster object

```r

library(raster)

chelsa_amt <- raster::raster("data/CHELSA_bio10_1.tif")
raster::plot(chelsa_amt)

# crop climate data for a specific bounding box
chelsa_amt_gbr <- raster::crop(chelsa_amt, raster::extent(country_sf_gbr)) # cut the worldwide raster according to
raster::plot(chelsa_amt_gbr)

# convert temperature data in usual unit
chelsa_amt_gbr[] <- chelsa_amt_gbr[]*0.1 # convert temperature data in usual unit
raster::res(chelsa_amt_gbr)

# change resolution aggregate() or disaggregate()
chelsa_amt_gbr_2 <- raster::disaggregate(chelsa_amt_gbr, fact=4)
raster::res(chelsa_amt_gbr_2)
```

netCDF format is commonly used for grided time series (temperature)

rast_1 <- raster::raster("data/tg_0.25deg_reg_2011-2017_v17.0.nc", band = 1)
rast_2 <- raster::raster("data/tg_0.25deg_reg_2011-2017_v17.0.nc", band = 2)
rast_stack <- raster::stack(rast_1, rast_2)
plot(rast_stack$mean.temperature.1)
plot(rast_stack)

eunis_1km <- raster::raster("data/es_l1_1km.tif")
raster::plot(eunis_1km)
raster::projection(eunis_1km)
proj_city_points_laea <- sf::st_transform(proj_city_points, raster::projection(eunis_1km))
eunis_city <- raster::extract(eunis_1km, as(proj_city_points_laea, "Spatial"))

eunis_city <- raster::extract(eunis_1km, as(sf::st_buffer(proj_city_points_laea, 100000), "Spatial"))
str(eunis_city)

as.numeric(table(eunis_city[[1]]))

proportion <- as.numeric(table(eunis_city[[1]]))[which(names(table(eunis_city[[1]]))!="10")] / sum(as.numeric(table(eunis_city[[1]]))) * 100


#### Vector object

```r
library(sf)
st_prov <- sf::st_read("data/GADM_2.8_GBR_adm2.shp")
plot(st_prov[,"HASC_2"], graticule = TRUE, axes = TRUE)

# Older option
library(rgdal)
st_prov_sp <- rgdal::readOGR("data", "GADM_2.8_GBR_adm2")
class(st_prov_sp)
plot(st_prov_sp, axes = TRUE)

# Change projection
crs.osgb = CRS("+init=epsg:27700")
st_prov_sp.osgb = sp::spTransform(st_prov_sp, crs.osgb)
plot(st_prov_sp.osgb, axes = TRUE)

spplot(st_prov_sp.osgb, "HASC_2", colorkey = FALSE)

rgdal::writeOGR(st_prov_sp, "data", "GADM_2.8_GBR_adm2", driver = "ESRI Shapefile")
```

```r
wdpa_gbr <- sf::st_read("data/wdpa_gbr.shp")
wdpa_gbr <- wdpa_gbr[wdpa_gbr$MARINE == 0,]

plot(wdpa_gbr$geometry)
## bounding box

bb <- sf::st_bbox(country_sf_gbr)
bb_poly <- sf::st_make_grid(country_sf_gbr, n = 1)

wdpa_gbr_2 <- sf::st_intersects(wdpa_gbr, bb_poly)
wdpa_gbr <- wdpa_gbr[unlist(lapply(wdpa_gbr_2, length)) >= 1,]

plot(wdpa_gbr$geometry[wdpa_gbr$IUCN_CAT == "V"])

wdpa_cntr <- sf::st_centroid(wdpa_gbr)
plot(wdpa_cntr$geometry[unlist(lapply(wdpa_gbr_2, length)) >= 1])

g.bbox <- raster::extent(as.numeric(sf::st_bbox(country.sf))[c(1,3,2,4)])
g.bbox_sf <- sf::st_set_crs(sf::st_as_sfc(as(g.bbox, 'SpatialPolygons')), 3035)
```




##### Hillshade and Terrain map
```
alt <- raster::getData("alt", country = "GBR")
slope <- raster::terrain(alt, opt = "slope")
aspect <- raster::terrain(alt, opt = "aspect")
hill <- raster::hillShade(slope, aspect, angle = 40, direction = 270)

raster::plot(hill, col = grey(0:100/100), legend = FALSE, add = TRUE)
raster::plot(alt, col = terrain.colors(25, alpha = 0.53), add = TRUE)
```

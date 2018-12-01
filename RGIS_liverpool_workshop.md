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

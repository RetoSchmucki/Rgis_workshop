## R GIS workshop

### building spatial objects with R, using sf

This is a revolution, providing a modern, stronger and cleaner workflow to deal with spatial object in R, at least vector data. The "sf" is developed by some of the same people that provide us with "sp", offering an ecosystem that open new opportunities to do GIS in R. The firs place to look for resource is the sf package website https://r-spatial.github.io/sf/index.html, this a first stop to learn how it work and develop new skills in R.

```r
if(!requireNamespace("sf")) install.packages("sf")
library(sf)

p1 <- sf::st_point(c(1, 2))
p2 <- sf::st_point(c(3, 5))
p3 <- sf::st_multipoint(matrix(2 * c(1, 2, 4, 2, 3, 5, 7, 3), ncol = 2, byrow = TRUE))
p4 <- sf::st_as_sf(data.frame(X = c(1, 4, 3, 7), Y = c(2, 2, 5, 3) ), coords = c("X", "Y"))
p5 <- sf::st_sfc(p1, p2, p3)

plot(p1)
plot(p3, col = "blue", pch = 19)
plot(p2, col = "magenta", pch = 19, add = TRUE)
```

## Modify sf - sfc objects

```r
p6 <- sf::st_cast(x = st_sfc(p3), to = "POINT")
p_multi <- sf::st_cast(p6, ids = c(1, 2, 1, 2), group_or_plist = TRUE, to = "MULTIPOINT")

plot(p_multi[1], ylim = c(0, 20), xlim = c(0, 20), col = "tomato3", pch = 19)
plot(p_multi[2], col = "magenta", pch = 19, add = TRUE)

p7 <- st_cast(x = p4, to = "POINT")
p8 <- rbind(st_cast(x = p4[1:3,], to = "MULTIPOINT"), p4[4,])
p8
```

### lines

```r
l1 <- sf::st_linestring(matrix(c(1, 1, 2, 2, 3, 3, 4, 4, 4, 2), ncol = 2, byrow = TRUE))
lp <- sf::st_cast(x = sf::st_sfc(l1), to = "MULTIPOINT")
bl1 <- sf::st_buffer(l1, 2)
blp <- sf::st_cast(x = sf::st_sfc(bl1), to = "MULTIPOINT")

plot(lp, col = "blue", pch = 19, cex = 2, ylim = c(-5, 10), xlim = c(-5, 10))
plot(blp, col = "magenta", pch = 19, cex = 1, add = TRUE)
plot(l1, col = "tomato3", lwd = 1.5, add = TRUE)
```

### polygon

```r
bl1 <- sf::st_buffer(l1, 2)
blp <- sf::st_cast(x = sf::st_sfc(bl1), to = "MULTIPOINT")

plot(bl1, col = "lightblue", border = NA)
plot(lp, col = "blue", pch = 19, cex = 2, ylim = c(-5, 10), xlim = c(-5, 10), add = TRUE)
plot(blp, col = "magenta", pch = 19, cex = 1, add = TRUE)
plot(l1, col = "tomato3", lwd = 1.5, add = TRUE)
```

### Intersects and intesections
```r
p1 <- sf::st_as_sf(data.frame(X = c(1, 4, 3, 7), Y = c(2, 2, 5, 3) ), coords = c("X", "Y"), crs = 4326)
poly1 <- st_as_sfc(st_bbox(st_buffer(p1[2,], 2)))
poly2 <- st_as_sfc(st_bbox(st_buffer(p1[3,], 1.5)))

plot(poly1, col = "goldenrod", xlim = c(-5, 5), ylim = c(0, 10))
plot(poly2, col = rgb(1,1,0,0.3), add = TRUE)
plot(p1, col = "magenta", pch = 19, cex= 1.5, add = TRUE)

## INTERSECTION
poly3 <- sf::st_intersection(poly1, poly2)
plot(poly3, col = "lightblue", add = TRUE)

## INTERSECT
p1_poly1 <- sf::st_intersects(p1, poly1, sparse = FALSE)
plot(p1[p1_poly1,], col = "turquoise", pch = 19, cex = 2, add = TRUE)
```

### circle buffer intersection
```r
poly1 <- sf::st_buffer(p1[2,], 2)
poly2 <- sf::st_buffer(p1[3,], 1.5)
int_b2_b1 <- sf::st_intersection(poly2, poly1)

plot(st_geometry(poly1), col = NA, border = "red", ylim = c(0, 7), axes = TRUE)
plot(st_geometry(poly2), add = TRUE)

plot(st_geometry(p1[2,]), col = "black", pch = 19, add = TRUE)
plot(st_geometry(p1[3,]), col = "blue", pch = 19, add = TRUE)

plot(st_geometry(int_b2_b1), col = "lightblue", boder = NA, add = TRUE)

plot(poly1, add = TRUE)
plot(int_b2_b1, col = "magenta", add = TRUE)
```
#### Difference between objects
```r
poly1 <- sf::st_buffer(p1[2,], 2)
poly2 <- sf::st_buffer(p1[2,], 4)
dif_poly2_poly1 <- sf::st_difference(poly2, poly1)

plot(st_geometry(dif_poly2_poly1), col = "orange", axes = TRUE)
plot(st_geometry(poly1), col = "blue", add = TRUE)
```

#### Union (merge and melt) objects
```r
poly1 <- sf::st_buffer(p1[2,], 2)
poly2 <- sf::st_buffer(p1[2,], 4)
uni_poly1_2 <- sf::st_union(dif_poly2_poly1, poly1)

plot(st_geometry(uni_poly1_2), col = "lightblue", axes = TRUE)
plot(st_geometry(poly1), col = "tomato3", add = TRUE)
plot(st_geometry(st_buffer(poly1,-1)), col = "white", add = TRUE)
plot(st_geometry(st_centroid(poly1)), col = "black", pch = 19, add = TRUE)
```

You like it tidy and the dplyr way? Since sf is essentally a data.frame with  a list of spatial attribute attached, it works well in the tidy univers. Have a look at this blog to get a first feel http://strimas.com/r/tidy-sf/.

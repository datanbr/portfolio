---
title: "Modèle GAM Recap "
date: 2018-10-13T19:00:33+02:00
draft: false
image: "/upload/formation.jpeg"
tags: ["project"]
---


# Présentation des différentes commandes à connaitre pour utiliser un modèle GAM

## Création de modèle avec 2 variables
### Création d'un modèle qui prend 2 variables qui varie en même temps
```r
gam(y ~ s(x1, x2), data = dat, method = "REML")
```
### Création d'un modèle qui prend 2 variables qui varie en même temps et qui possède aussi d'autres variables
```r
gam(y ~ s(x1, x2) + s(x3), data = dat, method = "REML")
gam(y ~ s(x1, x2) + x3 + x4, data = dat, method = "REML")
```

### Création d'un modèle avec des variables facteur
```r
model4b <- gam(hw.mpg ~ s(weight, by = fuel) + fuel, data = mpg, method = "REML")
```

### Création d'un modèle avec des variables facteur qui intéragissent de façon continue
```r
model4c <- gam(hw.mpg ~ s(weight, fuel, bs = "fs"),data = mpg,method = "REML")
```

### Intéraction entre des variables n'ayant pas la même dimension ou pas la même unité

l'intéraction by et bs marchent bien si les intéractions entre les modèles sont de même natures. Par contre ce n'est pas le cas si elles sont de nature différentes

```r
gam(y ~ te(x1, x2, k = c(10, 20)), data = data, method = "REML") # on définit nous meme les paramètres smooth
gam(y ~ s(x1) + s(x2) + ti(x1, x2), data = data, method = "REML") # on laisse le modèle le faire tout seul
```


## Visualisation de modèles

### Visualisation de bases
```r
plot(mod_2d)
```

### Visualisation du modèle sur une meme page
```r
plot(mod_2d, pages = 1)
```


### Représentation des données en 3d
```r
#Avec plot
plot(mod_2d, scheme = 1)

# Avec vis.gam
vis.gam(model4c, theta = 125, plot.type = "persp")

# on augmente le std
vis.gam(x = mod, view = c("x1","x2"),plot.type = "persp", se = 2)
```

### HeatMap
```r
plot(mod_2d, scheme = 2)
```

### En se spécialisant sur 2 éléments
```r
# x : le modèle
# view : les variables à considérés
# plot.type : le type d'affichage, ici de la 3d pour persp et contour pour du heat map
vis.gam(x = mod, view = c("x1","x2"), plot.type = "persp | contour") # kind of plot
```

### Visualisation des zones non couvertes par le modèle par rapport à ses données d'entrée
```r
#too.far permet de données le % à partir duquel on penses que la données n'est pas couverte.
# a faire varier entre 0 et 1
vis.gam(mod, view = c("x1","x2"), plot.type = "contour", too.far = 0.1)
vis.gam(mod, view = c("x1","x2"), plot.type = "contour", too.far = 0.05)
```

### on modifie l'angle de vue des données
```r
vis.gam(g, view = c("x1","x2"), plot.type = "persp", theta = 220)
vis.gam(g, view = c("x1","x2"), plot.type = "persp", phi = 55)
vis.gam(g, view = c("x1","x2"), plot.type = "persp", r = 0.1)
```

### on modifie les contours
```r
vis.gam(g, view = c("x1","x2"), plot.type = "contour", color = "gray")
vis.gam(g, view = c("x1","x2"), plot.type = "contour", contour.col = "blue")
vis.gam(g, view = c("x1","x2"), plot.type = "contour", nlevels = 20)
```

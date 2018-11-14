---
title: "Modèle GAM Recap "
date: 2018-10-13T19:00:33+02:00
draft: false
image: "/upload/formation.jpeg"
tags: ["project"]
---


# Présentation des différentes commandes à connaitre pour utiliser un modèle GAM

### Création d'un modèle avec des variables facteur
model4b <- gam(hw.mpg ~ s(weight, by = fuel) + fuel, data = mpg, method = "REML")

### Création d'un modèle avec des variables facteur liés les unes entre elles
model4c <- gam(hw.mpg ~ s(weight, fuel, bs = "fs"),data = mpg,method = "REML")

### Représentation des données en 3d
plot(model4c)
vis.gam(model4c, theta = 125, plot.type = "persp")

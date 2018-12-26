---
title: "Tout ce qu'il y a savoir avec les regressions linéaires "
date: 2018-10-13T19:00:33+02:00
draft: false
image: "/upload/formation.jpeg"
tags: ["project"]
---


# Regression linéaire

```r
#Une regression linéaire basique :
cmodel <- lm(temperature ~ chirps_per_sec, data = cricket)

#on peut définir des formules en mettant la donnée dans une variable
fmla_2 <- blood_pressure ~ age + weight

fmla_1 <- as.formula("temperature ~ chirps_per_sec")

#on peut mettre la formule dans le modèle directement
cmodel <- lm(fmla_1, data = cricket)
```

```r
# Pour avoir le résumté :
# on affiche le modèle
cmodel
(Intercept) valeur quand toutes les données sont à 0
valeur coefficient : permet de dire si c'est positif ou négatif

# analyse plus évoluée :
summary(cmodel)
broom::glance(cmodel)
sigr::wrapFTest(cmodel)
```

```r
#
```

```r
#
```

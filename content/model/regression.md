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

On regarde le R-squared, qui correspond au rapport sommes (valeur(i) - valeur prédit(i))² / (valeur(i) - valeur moyenne)²
```

# pour faire une prévision avec le modèle

```r
# on utilise predict, qui renvoie par défaut les valeurs prédit par le modèle pour lesquels le modèle a appris
predict(cmodel)
cricket$prediction <- predict(cmodel)

# en passant une donnée au modèle
newchirps <- data.frame(chirps_per_sec = 16.5)
newchirps$prediction <- predict(cmodel, newdata = newchirps)
newchirps # permet d'avoir le résultat

```

## R²

R² permet de comparer les performances du modèle avec la moyenne des valeurs à prévoir :
- plus R² est proche de 1, meilleur est la prévision
- plus R² est proche de 0, plus la prévision se rapproche de la moyenne et n'est donc pas intéressante

R² se calcule par 1-RMSE/RME

Dans le cas de modèle linéaire, on a cor(prevu, vrai élément)² = R²

R² permet de savoir si on fait de l'overfitting en comparant :
- le R² de la période d'apprentissage
- le R² de la période de validation

```r
library(vtreat)
splitPlan <- kWayCrossValidation(nRows, nSplits, NULL, NULL)
```
nRows : nombre de lignes de l'apprentissage
nsplit : le nombre de partition

```r
k <- 3 # Number of folds
mpg$pred.cv <- 0
i = 1
for(i in 1:k) {
  split <- splitPlan[[i]]
  model <- lm(cty ~ hwy, data = mpg[split$train,])
  mpg$pred.cv[split$app] <- predict(model, newdata = mpg[split$app,])
}
```
Cela renvoie un tableau d'objet contenant à chaque fois :
- $train
- $app

Il s'utilise directement en mettant splitPlan dans la dataframe


```r
library(vtreat)
splitPlan <- kWayCrossValidation(10, 3, NULL, NULL)

splitPlan[[1]]
## $train
## [1] 1 2 4 5 7 9 10
##
## $app
## [1] 3 6 8
```

```r
#on entraine le modèle modèle
split <- splitPlan[[1]]
model <- lm(fmla, data = df[split$train,])

#on prédit les donnéées sur la période de validation
df$pred.cv[split$app] <- predict(model, newdata = df[split$app,])

function(predcol, ycol) {
  res = predcol-ycol
  sqrt(mean(res^2))
}

function(predcol, ycol) {
  tss = sum( (ycol - mean(ycol))^2 )
  rss = sum( (predcol - ycol)^2 )
  1 - rss/tss
}
```

on peut savoir comment une formule est interprété (pour des variables factor) :
(fmla <- formula("Flowers ~ Intensity + Time"))
# Use fmla and model.matrix to see how the data is represented for modeling
mmat <- model.matrix(fmla, flowers)

## Intéraction entre les variables
```r
Intéraction possible entre vaiable
# les 2 variables n'intéragissent pas, ils sont principaux
(fmla_add <- as.formula("Metabol ~ Gastric + Sex") )

# Les 2 variables intéragissent, Gastric est principal
(fmla_interaction <- as.formula("Metabol ~ Gastric + Gastric:Sex") )

# Fit the main effects only model
model_add <- lm(fmla_add, alcohol)

# Fit the interaction model
model_interaction <- lm(fmla_interaction, alcohol)

# Call summary on both models and compare
summary(model_add)
summary(model_interaction) # meilleur RMSE
```

Pour calculer les rmse avec différents

```r
head(alcohol)
  Subject Metabol Gastric    Sex       Alcohol    pred_add pred_interaction
1       1     0.6     1.0 Female     Alcoholic -0.04603078                0
2       2     0.6     1.6 Female     Alcoholic  1.09934977                0
3       3     1.5     1.5 Female     Alcoholic  0.99744246                0
4       4     0.4     2.2 Female Non-alcoholic  2.48516624                0
5       5     0.1     1.1 Female Non-alcoholic  0.14731458                0
6       6     0.2     1.2 Female Non-alcoholic  0.35984655                0


on a fait 2 prévisions pred_add et pred_interaction

alcohol %>%
    gather(key = modeltype, value = pred, pred_add, pred_interaction)
   Subject Metabol Gastric    Sex       Alcohol        modeltype        pred
1        1     0.6     1.0 Female     Alcoholic         pred_add -0.04603078
2        2     0.6     1.6 Female     Alcoholic         pred_add  1.09934977
3        3     1.5     1.5 Female     Alcoholic         pred_add  0.99744246

# On calcule de RMSE
alcohol %>%
  gather(key = modeltype, value = pred, pred_add, pred_interaction) %>%
  mutate(residuals = Gastric - pred) %>%
  group_by(modeltype) %>%
  summarize(rmse = sqrt(mean(residuals^2)))
# A tibble: 2 x 2
  modeltype         rmse
  <chr>            <dbl>
1 pred_add          1.17
2 pred_interaction  1.60
```

---
title: "Tout ce qu'il y a savoir avec les regressions linéaires "
date: 2018-10-13T19:00:33+02:00
draft: false
image: "/upload/formation.jpeg"
tags: ["model"]
---


# Regression linéaire

## Création de formule
```r
#Une regression linéaire basique :
cmodel <- lm(temperature ~ chirps_per_sec, data = cricket)

#on peut définir des formules en mettant la donnée dans une variable
fmla_2 <- blood_pressure ~ age + weight

fmla_1 <- as.formula("temperature ~ chirps_per_sec")

#on peut mettre la formule dans le modèle directement
cmodel <- lm(fmla_1, data = cricket)
```

## Résumé

```r
# on affiche le modèle
cmodel
(Intercept) valeur quand toutes les données sont à 0
valeur coefficient : permet de dire si c'est positif ou négatif

# analyse plus évoluée :
summary(cmodel)
broom::glance(cmodel)
sigr::wrapFTest(cmodel)
```

## Prévision avec le modèle

```r
# on utilise predict, qui renvoie par défaut les valeurs prédit par le modèle pour lesquels le modèle a appris
predict(cmodel)
cricket$prediction <- predict(cmodel)

# en passant une donnée au modèle
newchirps <- data.frame(chirps_per_sec = 16.5)
newchirps$prediction <- predict(cmodel, newdata = newchirps)
newchirps # permet d'avoir le résultat

```

## Calcul du RMSE et du R²

```r
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

## R²

R² permet de comparer les performances du modèle avec la moyenne des valeurs à prévoir :

* plus R² est proche de 1, meilleur est la prévision
* plus R² est proche de 0, plus la prévision se rapproche de la moyenne et n'est donc pas intéressante

R² se calcule par 1-RMSE/RME

Dans le cas de modèle linéaire, on a cor(prevu, vrai élément)² = R²

R² permet de savoir si on fait de l'overfitting en comparant :

* le R² de la période d'apprentissage
* le R² de la période de validation

## Cross validation

On peut faire du cross validation à la main ou en utilisant directement vtreat::kWayCrossValidation

```r
library(vtreat)
# nRows : nombre de lignes de l'apprentissage
# nsplit : le nombre de partition
splitPlan <- kWayCrossValidation(nRows, nSplits, NULL, NULL)
```


kWayCrossValidation renvoie un tableau d'objet contenant à chaque fois :
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

#on entraine le modèle modèle
split <- splitPlan[[1]]
model <- lm(fmla, data = df[split$train,])

#on prédit les donnéées sur la période de validation
df$pred.cv[split$app] <- predict(model, newdata = df[split$app,])

```

Utilisation en production
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

## catégorie : variable factor

Lm ne gère pas directement les factor mais transforme les valeurs en numérique avec autant de colonne par type de données

```r
on peut savoir comment une formule est interprété (pour des variables factor) :
(fmla <- formula("Flowers ~ Intensity + Time"))
mmat <- model.matrix(fmla, flowers)
```


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


## Transformation des valeurs à prévoir

Lorque on doit prévoir des valeurs très condensées avec une longue tail, on a souvent :

* moyenne >> mediane

Dans ce cas la prévision va avoir tendance à être surestimé
Pour résoudre le problème :

* on prévoi log(Y)
* on applique exp(de la prévision) pour revenir à des valeurs cohérentes

```r
model <- lm(log(y) ~ x, data = train)
logpred <- predict(model, data = test)
pred <- exp(logpred)
```

Une nouvelle grandeur à calculer est le RMS relatif erreur : sqrt(prev - Y)²/Y

```r
test %>%
+ mutate(pred = predict(modIncome, newdata = test),
+ err = pred - Income) %>%
+ summarize(rmse = sqrt(mean(err^2)),
+ rms.relerr = sqrt(mean((err/Income)^2)))
```

## Log model:

* Plus petit RMS-relative error
* Plus large RMSE

Modelisation de phénomène non linéaire par lm

On peut modifier les données d'entrée du modèle notamment en faisant des log(X) ou des mises en puissances
Pour la mise en puissance, on utilise I(X^2) car le ^ est un symbol exprimant l'intéraction

```r
head(houseprice)
size price
1   72   156
2   98   153
3   92   230
4   90   152
5   44    42
6   46   157


model_lin <- lm(price ~ size, data = houseprice)
(fmla_sqr <- as.formula("price ~ I(size^2)"))
model_sqr <- lm(fmla_sqr, data = houseprice)

houseprice %>%
    mutate(pred_lin = predict(model_lin),       # predictions from linear model
           pred_sqr = predict(model_sqr)) %>%   # predictions from quadratic model
    gather(key = modeltype, value = pred, pred_lin, pred_sqr) %>% # gather the predictions
    ggplot(aes(x = size)) +
       geom_point(aes(y = price)) +                   # actual prices
       geom_line(aes(y = pred, color = modeltype)) + # the predictions
       scale_color_brewer(palette = "Dark2")
```

## Regression logistic

Une regression logistic correspond à une somme pondérée binomiale
le modèle calcule les coefficients de chaque binome
Cela permet de faire de la classification
glm est utilisé pour cela et fonctionne comme lm :

* formule
* data
* famille : binomial (distribution des erreurs)

Pour avoir la réponse, on utilise le type = response

```r
model <- glm(has_dmd ~ CK + H, data = train, family = binomial)
test$pred <- predict(model, newdata = test, type = "response")
```

Pour évaluler les performances du modèle, on utilise le pseudo R² = 1 - deviance / null.deviance
Plus la modélisation est bonne, plus le pseudo R² est proche de 1

On utilise pour le calculer :

* broom::glance
* sigr::wrapChiSqTest()

```r
glance(model) %>%
+ summarize(pR2 = 1 - deviance/null.deviance)

wrapChiSqTest(model)

test %>%
+ mutate(pred = predict(model, newdata = test, type = "response")) %>%
+ wrapChiSqTest("pred","has_dmd", TRUE)
```

On peut encore utiliser le Gain curve plot pour voir la performance du modèle

```r
GainCurvePlot(test,"pred","has_dmd","DMD model on test")
```

## Modèle de Poisson ou de quasi poisson

Dans le cas où le Y a modélisé :

* est toujours positif
* s'accroit avec le temps
* la sortie est un entier

Par exemple on compte des choses :

* des ventes pour un magasin
* des jours où une personne est absente
* des nombres de visite sur une page

On peut utiliser une modélisation de poisson ou de quasi poisson :

* poisson si la moyenne est à peu près égale à la variance
* quasipoisson si ce n'est pas le cas

Pour nourrir le modèle avec quasipoisson ou poisson

```r
fmla <- cnt ~ hr + holiday + workingday +
+ weathersit + temp + atemp + hum + windspeed
> model <- glm(fmla, data = bikesJan, family = quasipoisson)
```
Pour valider que les prédictions sont OK

```r
glance(model) %>%
+ summarize(pseudoR2 = 1 - deviance/null.deviance)
```
Lancer la prévision
```r
predict(model, newdata = bikesFeb, type = "response")
```


Pour générer variable dynamiquement
```r
vars = names(bikesJuly)[1:8]
outcome = names(bikesJuly)[1]

# Create the formula string for bikes rented as a function of the inputs
(fmla <- paste(outcome, "~", paste(vars, collapse = " + ")))
```

## Modèle GAM

On peut utiliser les modèles GAM si on ne sait pas comment modéliser à priori le processus sous la forme de binome
on spécifie la famille à GAM en fonction du type de problème :

* gaussian (default): "regular" regression
* binomial: probabilities
* poisson/quasipoisson: counts

gam(formula, family, data)

On utilise s() pour indiquer à GAM de faire une regression non linéaire, qui marche uniquement sur des variables continues
```r
model <- gam(anx ~ s(hassles), data = hassleframe, family = gaussian)
```

## Vision intéressante de 2 prévisions

```r
head(soybean_long)
    Plot Variety Year Time  weight modeltype      pred
1 1988F1       F 1988   42  3.5600  pred.lin  5.708655
2 1988F1       F 1988   56  8.7100  pred.lin  9.797968
3 1988F1       F 1988   70 16.3417  pred.lin 13.887281
4 1988F2       F 1988   14  0.1040  pred.lin -2.469970
5 1988F2       F 1988   42  2.9300  pred.lin  5.708655
6 1988F2       F 1988   77 17.7467  pred.lin 15.931937

# Compare the predictions against actual weights on the test data
soybean_long %>%
  ggplot(aes(x = Time)) +                          # the column for the x axis
  geom_point(aes(y = weight)) +                    # the y-column for the scatterplot
  geom_point(aes(y = pred, color = modeltype)) +   # the y-column for the point-and-line plot
  geom_line(aes(y = pred, color = modeltype, linetype = modeltype)) + # the y-column for the point-and-line plot

```

## Arbre

Il y a 2 grands modèles d'arbre à connaitre :
- Random Forrest
- Xgboost

Ce type d'algo permet de modéliser des relations nons linéaires. il est très bon pour :
- prévoir des intervales
- des relations non monotones

Les arbres déterminent des grandes régions plutot que des relations linéaires

## Random Forrest

L'algo du random Forest passe par :
- la selection au hasard d'une partie des données
- la création d'un arbre en selectionnant les meilleures variables à chaque noeud
- on évalue tous les arbres et on prend le meilleur

```r
model <- ranger(fmla,
                training.treat,
                num.trees = 500,
                respect.unordered.factors = "order")

# Si on veut pouvoir reproduire un cas, on doit d'abord fixer le hasard et ensuite l'utiliser
seed
(bike_model_rf <- ranger(fmla, # formula
                         bikesJuly, # data
                         num.trees = 500,
                         respect.unordered.factors = "order",
                         seed = seed))
```
num.trees = 500 # au moins à 200
respect.unordered.factors # permet d'indiquer au modèle comment gérer les factors.
On peut mettre order ou safe

On peut avoir la prévision en faisant
bikesFeb$pred <- predict(model, bikesFeb)$predictions

On peut avoir le RMSE directement en faisant un
```r
bikesAugust %>%
  mutate(residual = pred - cnt)  %>% # calculate the residual
  summarize(rmse  = sqrt(mean(residual^2)))      # calculate rmse
```

## Préaparation des données pour Xgboost

Xgboost ne sait pas gérer les données non numériques
il faut donc les transformer avant de lui envoyer les données
Cela passe par vtreat:
- designTreatmentsZ() pour préparer un plan de traitement
- preparer() pour cleaner les data

```r
#install.packages("vtreat")
#install.packages("xgboost")
library(vtreat)
library(magrittr)
library(xgboost)

#on selectionne les données à modifier
vars = names(train_df)
vars = vars[-length(vars)]
treatplan <- designTreatmentsZ(dframe, varslist, verbose = FALSE)

dframe = train_df
treatplan <- designTreatmentsZ(dframe, vars, verbose = FALSE)
scoreFrame <- treatplan$scoreFrame %>%
  select(varName, origName, code)

newvars <- scoreFrame %>%
  filter(code %in% c("clean","lev")) %>%
  use_series(varName)

#newvars contient les données
training.treat <- prepare(treatmentplan, dframe, varRestriction = newvars)
test.treat <- prepare(treatplan, test_df, varRestriction = newvars)
```

## Xgboost

Xgboost crée des arbres superficiels pour coller à la courbe
Il peut très rapidement faire de l'overfitting car il se base sur les données d'entrée

```r
xgb.cv() avec un grand nombre d'arbres (correspondant au nombre de fois où le script tourne)
xgb.cv()$evaluation_log enregistre à chaque round le rmse. Cela permet de trouver le meilleur nombre d'arbres à selectionner
xgboost() prend en entrée xgb.cv
```

On prépare les datas

```r
treatplan <- designTreatmentsZ(bikesJan, vars)
newvars <- treatplan$scoreFrame %>%
  filter(code %in% c("clean","lev")) %>%
  use_series(varName)
bikesJan.treat <- prepare(treatplan, bikesJan, varRestriction = newvars)
```

Les données pour Xgboost sont :

* Données d'entrée : as.matrix(bikesJan.treat)
* Données de sortie : bikesJan$cnt

Calcul de la meilleur profondeur

```r
library(xgboost)
cv <- xgb.cv(data = as.matrix(training.treat),
             label = train_df$SalePrice,
             objective = "reg:linear",
             nrounds = 100,
             nfold = 5,
             eta = 0.3,
             max_depth = 6)
```

data : données d'entrée X
lable : données à prévoir
objective : indique le type de regression. On a ici des données qui évoluent linéairement
nrounds : maximum nombre de round, ici 100 car on ne le connait pas
nfold : nombre de cross validation
eta : pourcentage d'apprentissage
max_depth : maximm de profondeur pour chaque arbre
early_stopping_rounds: Aprè combien de round on stope sans amélioration
verbose: 0 fait le traitement en silence

Pour déterminer le min de profondeur, on fait
```r
elog <- as.data.frame(cv$evaluation_log)
nrounds <- which.min(elog$test_rmse_mean)
```

Lancement du modèle

```r
model <- xgboost(
  data = as.matrix(training.treat),
  label = train_df$SalePrice,
  nrounds = nrounds,
  objective = "reg:linear",
  eta = 0.3,
  depth = 6)
```

Pour prédire sur le jeu de données de validation

```r
bikesFeb.treat <- prepare(treatplan, bikesFeb, varRestriction = newvars)
bikesFeb$pred <- predict(model, as.matrix(bikesFeb.treat))
```

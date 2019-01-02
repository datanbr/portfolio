---
title: "Modèle GAM Recap "
date: 2018-10-13T19:00:33+02:00
draft: false
image: "/upload/formation.jpeg"
tags: ["analyse"]
---

# Méthode analyse des données

Lorsqu'on doit prévoir qq chose qui dépend de beaucoup de facteur, il est bon :

## Regrouper toutes les données dans une seule data frame

Cela permet d'étudier d'un coup toutes les données manquante par exemple
```r
train <- read.csv("train.csv", stringsAsFactors = F)
test <- read.csv("test.csv", stringsAsFactors = F)

#on sauve les données
test_labels <- test$Id
test$Id <- NULL
train$Id <- NULL
test$SalePrice <- NA
all <- rbind(train, test)
```

## d'avoir une vision globale des choses avec un histogramm et un summary :

```r
library(ggplot2)

ggplot(data=train, aes(x=SalePrice)) +
  geom_histogram(fill="blue", binwidth = 10000) +
  scale_x_continuous(breaks= seq(0, 800000, by=100000))

```
Est ce que la distribution est normale ?
Est ce qu'il y a des valeurs extrêmes ?

```r
summary(train$SalePrice)

Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
  34900  129975  163000  180921  214000  755000
```

## De voir la correlation entre les colonnes contenant des numériques

On filtre sur les valeurs numériques

```r
get_name_numerique = function(df){
  numericVars <- which(sapply(train, is.numeric)) #index vector numeric variables
  numericVarNames <- names(numericVars) #saving names vector for use later on
  numericVarNames
}

cat('There are', length(get_name_numerique(train)), 'numeric variables')
```
On crée une matrice de correlation présentant les éléments les plus importants qui sont liés entre eux

```r
install.packages("corrplot")
library(corrplot)

# on récupère les colonnes qui ont des valeurs numériques
numericVars <- which(sapply(train, is.numeric))
#on réduit la matrice aux colonne numériques
all_numVar <- all[, numericVars]
# on crée l element de corelation
cor_numVar <- cor(all_numVar, use="pairwise.complete.obs") #correlations of all numeric variables

# on transforme l'élement en matrice et on la classe du plus grand au plus petit
cor_sorted <- as.matrix(sort(cor_numVar[,'SalePrice'], decreasing = TRUE))
#on ne selectionne que les données avec une corrélation supérieur à 0.5
CorHigh <- names(which(apply(cor_sorted, 1, function(x) abs(x)>0.5)))
cor_numVar <- cor_numVar[CorHigh, CorHigh]
#on affiche cette matrice dans un graphique
install.packages("corrplot")
library(corrplot)
corrplot.mixed(cor_numVar, tl.col="black", tl.pos = "lt")
```

## Visualisation des principaux facteurs

On en déduit les principaux facteurs qui ont un impact sur ce qu'on cherche, ici SalePrice. On regarde aussi si ces éléments sont corrélés entre eux, ce qui est le cas et peut mener à de l'overfiiting

On peut regarder les impacts des principaux éléments sur SalePrice
Si on a un élément qui contient peu de données différentes et qui peut donc être tranformé en facteur, on peut faire un :

```r
ggplot(data=train, aes(x=factor(OverallQual), y=SalePrice))+
  geom_boxplot(col='blue') + labs(x='Overall Quality') +
  scale_y_continuous(breaks= seq(0, 800000, by=100000))
```

Sinon, on affiche directement la relation entre les deux : en rajoutant :

* un une regression linéaire
* les index qui sont extrèmes

```r
install.packages("ggrepel")
library(ggrepel)
ggplot(data=train, aes(x=GrLivArea, y=SalePrice))+
  geom_point(col='blue') + labs(x='Overall Quality') +
  geom_smooth(method = "lm", se=FALSE, color="black", aes(group=1)) +
  scale_y_continuous(breaks= seq(0, 800000, by=100000)) +
  geom_text_repel(aes(label = ifelse(train$GrLivArea>4500, rownames(all), '')))

# On voit que des éléments sont vraiments des exceptions :
train[c(524, 1299), c('SalePrice', 'GrLivArea', 'OverallQual')]

```

## Données manquantes

### Visualisation des données

```r
NAcol <- which(colSums(is.na(all)) > 0)
sort(colSums(sapply(all[NAcol], is.na)), decreasing = TRUE)
cat('There are', length(NAcol), 'columns with missing values')
```

On se concentrer sur les différentes valeurs qui manquent
```r
PoolQC     n
  <chr>  <int>
1 Ex         4
2 Fa         2
3 Gd         4
4 NA      2909

PoolQC     n
  <chr>  <int>
1 Ex         4
2 Fa         2
3 Gd         4
4 NA      2909

PoolQC: Pool quality

       Ex    Excellent
       Gd    Good
       TA    Average/Typical
       Fa    Fair
       NA    No Pool
```
Le NA ici veut dire qq chose : il n'y a pas de piscine. Donc on va le compléter comme tel et on va le hiérarchiser par des entiers : plus c'est important, mieux c'est.

```r
library(plyr)
Qualities <- c('None' = 0, 'Po' = 1, 'Fa' = 2, 'TA' = 3, 'Gd' = 4, 'Ex' = 5)
all = all %>%
  mutate(PoolQC = as.integer(revalue(all$PoolQC, Qualities)))
```

Par contre la pisicine est aussi reliée à un autre élément : PoolArea: Pool area in square feet
On regarde s'il y a une relation entre les deux, ce qui n'est pas le cas
```r
all %>%
    count(PoolQC,PoolArea)
# A tibble: 14 x 3
   PoolQC PoolArea     n
   <chr>     <int> <int>
 1 Ex          144     1
 2 Ex          228     1
 3 Ex          512     1
 4 Ex          555     1
 5 Fa          519     1
 6 Fa          648     1
 7 Gd          480     1
 8 Gd          576     1
 9 Gd          738     1
10 Gd          800     1
11 NA            0  2906
12 NA          368     1
13 NA          444     1
14 NA          561     1
```
on a un problème dans les données : certaines maisons n'ont pas de piscine et pourtant il y a une longueur non nules
```r
table(all$PoolQC, all$PoolArea)

       0  144  228  368  444  480  512  519  555  561  576  648  738  800
  0 2906    0    0    1    1    0    0    0    0    1    0    0    0    0
  2    0    0    0    0    0    0    0    1    0    0    0    1    0    0
  4    0    0    0    0    0    1    0    0    0    0    1    0    1    1
  5    0    1    1    0    0    0    1    0    1    0    0    0    0    0
```
On se base sur OverallQual pour donner à un elemnt à PoolQC, sachant qu'on est dans une moyenne pour PoolQC
all[all$PoolArea>0 & all$PoolQC==0, c('PoolArea', 'PoolQC', 'OverallQual')]
     PoolArea PoolQC OverallQual
2421      368      0           4
2504      444      0           6
2600      561      0           3

Dans le cas où il n'y a pas de gradation, on transforme en factor
```r
all = all %>%
  mutate(Alley = ifelse(is.na(Alley == TRUE), 'None', Alley))
all = all %>%
  mutate(Alley = as.factor(Alley))
```
On peut regarder s'il y a une relation particulière avec la veur transformée et la valeur cherchée
```r
ggplot(all[!is.na(all$SalePrice),], aes(x=Alley, y=SalePrice)) +
  geom_bar(stat='summary', fun.y = "median", fill='blue')+
  scale_y_continuous(breaks= seq(0, 200000, by=50000))

 ```

Le but est qu'il n'y ait plus de NA ni de données contenant des charactères. Il faut à chaque fois que :

* les données soient des facteurs
* les données soient des ordonnées
* les données soient des numerics

# TP 3 : Machine learning avec Spark

<!-- TOC -->

- [TP 3 : Machine learning avec Spark](#tp-3--machine-learning-avec-spark)
    - [Charger le DataFrame.](#charger-le-dataframe)
    - [Utiliser les données textuelles](#utiliser-les-données-textuelles)
        - [Stage 1 : récupérer les mots des textes](#stage-1--récupérer-les-mots-des-textes)
        - [Stage 2 : retirer les stop words](#stage-2--retirer-les-stop-words)
        - [Stage 3 : computer la partie TF](#stage-3--computer-la-partie-tf)
        - [Stage 4 : computer la partie IDF](#stage-4--computer-la-partie-idf)
    - [Convertir les catégories en données numériques](#convertir-les-catégories-en-données-numériques)
        - [Stage 5 : convertir *country2* en quantités numériques](#stage-5--convertir-country2-en-quantités-numériques)
        - [Stage 6 : convertir *currency2* en quantités numériques](#stage-6--convertir-currency2-en-quantités-numériques)
        - [Stages 7 et 8: One-Hot encoder ces deux catégories](#stages-7-et-8-one-hot-encoder-ces-deux-catégories)
    - [Mettre les données sous une forme utilisable par Spark.ML](#mettre-les-données-sous-une-forme-utilisable-par-sparkml)
        - [Stage 9 : assembler tous les features en un unique vecteur](#stage-9--assembler-tous-les-features-en-un-unique-vecteur)
        - [Stage 10 : créer/instancier le modèle de classification](#stage-10--créerinstancier-le-modèle-de-classification)
        - [Créer le Pipeline](#créer-le-pipeline)
    - [Entraînement et tuning du modèle](#entraînement-et-tuning-du-modèle)
        - [Split des données en training et test sets](#split-des-données-en-training-et-test-sets)
        - [Entraînement du classifieur et réglage des hyper-paramètres](#entraînement-du-classifieur-et-réglage-des-hyper-paramètres)
        - [Test du modèle](#test-du-modèle)
    - [Supplément](#supplément)

<!-- /TOC -->

Dans cette partie du TP, on veut créer un modèle de classification entraîné sur les données qui ont été pré-traitées dans les TPs précédents. Pour que tout le monde reparte du même point, téléchargez le dataset *prepared_trainingset* (ce sont des fichiers parquet) situé dans le répertoire *data*.

Pour cette partie du TP, veuillez codez dans l’objet *Trainer*, cela vous évitera de refaire les préprocessings des TPs précédents à chaque run. Pour lancer l'exécution du script Trainer faites dans un terminal à la racine du projet
```bash
./build_and_submit.sh Trainer
```

**Rappel**: Il y a deux librairies de machine learning dans Spark: [*spark.ml*](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.package), basée sur les DataFrames, et [*spark.mllib*](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.package), basée sur les RDDs. Il est préférable d’utiliser `spark.ml` comme le préconise la [doc](https://spark.apache.org/docs/latest/ml-guide.html).

Le but de ce TP est de construire un [*Pipeline*](https://spark.apache.org/docs/latest/ml-pipeline.html) de Machine Learning. Pour construire un tel Pipeline, on commence par construire les *Stages* que l’on assemble ensuite dans un Pipeline.

Comme dans [*scikit-learn*](https://scikit-learn.org/stable/), un pipeline dans spark ML est une succession d’algorithmes et de transformations s’appliquant aux données. Chaque étape du pipeline est appelée "stage", et l’output de chaque stage est l’input du stage qui le suit. L’avantage des pipelines est qu’ils permettent d’encapsuler des étapes de preprocessing et de machine learning dans un seul objet qui peut être sauvegardé après l’entraînement puis chargé d’un seul bloc, ce qui facilite notamment la gestion des modèles et leur déploiement. 

Nous allons utiliser les modules *spark.ml.feature*, *spark.ml.classification*, *spark.ml.evaluation*, *spark.ml.tuning*, et la classe *spark.ml.Pipeline*.

## Charger le DataFrame.

Chargez le DataFrame obtenu à la fin du TP 2.
 
## Utiliser les données textuelles

Les textes ne sont pas utilisables tels quels par les algorithmes parce qu’ils ont besoin de données numériques, en particulier pour faire les calculs d’erreurs et d’optimisation. On veut donc convertir la colonne "text" en données numériques. Une façon très répandu de faire cela est d’appliquer l’algorithme [TF-IDF](https://fr.wikipedia.org/wiki/TF-IDF).

### Stage 1 : récupérer les mots des textes

La première étape est de séparer les textes en mots (ou tokens) avec un tokenizer. Vous allez donc construire le premier stage du pipeline de la façon suivante :
```scala
val tokenizer = new RegexTokenizer()
  .setPattern("\\W+")
  .setGaps(true)
  .setInputCol("text")
  .setOutputCol("tokens")
```

### Stage 2 : retirer les stop words

On veut retirer les stop words pour ne pas encombrer le modèle avec des mots qui ne véhiculent pas de sens. Créer le 2ème stage avec la classe *StopWordsRemover*.

### Stage 3 : computer la partie TF

La partie TF de TF-IDF est faite avec la classe *CountVectorizer*.

### Stage 4 : computer la partie IDF

Trouvez la partie IDF. On veut écrire l’output de cette étape dans une colonne "tfidf". Vous pouvez vous aider de la page [Feature extraction](http://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction) de scikit-learn. 

## Convertir les catégories en données numériques

Les colonnes *country2* et *currency2* sont des variables catégorielles (qui ne prennent qu’un ensemble limité de valeurs), par opposition aux variables continues comme *goal* ou *hours_prepa* qui peuvent prendre n’importe quelle valeur réelle positive. Ici les catégories sont indiquées par une chaîne de charactères, e.g. "US" ou "EUR". On veut convertir ces classes en quantités numériques.

### Stage 5 : convertir *country2* en quantités numériques

On veut les résultats dans une colonne *country_indexed*.

### Stage 6 : convertir *currency2* en quantités numériques

On veut les résultats dans une colonne *currency_indexed*.

### Stages 7 et 8: One-Hot encoder ces deux catégories

Transformer ces deux catégories avec un "one-hot encoder" en créant les colonnes *currency_onehot* et *country_onehot*. Une [page Quora](https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science) sur le one-hot encoding.

## Mettre les données sous une forme utilisable par Spark.ML

La plupart des algorithmes de machine learning dans Spark requièrent que les colonnes utilisées en input du modèle (les features du modèle) soient regroupées dans une seule colonne qui contient des vecteurs. On veut donc passer de

|Feature A|Feature B|Feature C|Label|
|:---:|:---:|:---:|:---:|
|0.5|1|3.5|0|
|0.6|1|1.2|1|

à

| Features | Label |
|:---:|:---:|
|(0.5, 1, 3.5)|0|
|(0.6, 1, 1.2)|1|

### Stage 9 : assembler tous les features en un unique vecteur

Assembler les features *tfidf*, *days_campaign*, *hours_prepa*, *goal*, *country_onehot*, et *currency_onehot* dans une seule colonne *features*.

### Stage 10 : créer/instancier le modèle de classification

Il s’agit d’une régression logistique que vous définirez de la façon suivante :
```scala
val lr = new LogisticRegression()
  .setElasticNetParam(0.0)
  .setFitIntercept(true)
  .setFeaturesCol("features")
  .setLabelCol("final_status")
  .setStandardization(true)
  .setPredictionCol("predictions")
  .setRawPredictionCol("raw_predictions")
  .setThresholds(Array(0.7, 0.3))
  .setTol(1.0e-6)
  .setMaxIter(300)
```

### Créer le Pipeline

Enfin, créer le pipeline en assemblant les 10 stages définis précédemment, dans le bon ordre.

## Entraînement et tuning du modèle

### Split des données en training et test sets

On veut séparer les données aléatoirement en un training set (90% des données) qui servira à l’entraînement du modèle et un test set (10% des données) qui servira à tester la qualité du modèle sur des données que le modèle n’a jamais vues lors de son entraînement.

Créer un DataFrame nommé *training* et un autre nommé *test* à partir du DataFrame chargé initialement de façon à le séparer en training et test sets dans les proportions 90%, 10% respectivement.

### Entraînement du classifieur et réglage des hyper-paramètres

Le classifieur que nous utilisons est une [régression logistique]( 
http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.classification.LogisticRegression)
(que vous verrez en cours cette année) avec une régularisation dans la fonction de coût qui permet de pénaliser les features les moins fiables pour la classification.

L’importance de la régularisation est contrôlée par un hyper-paramètre du modèle qu’il faut régler à la main. La plupart des algorithmes de machine learning possèdent des hyper-paramètres, par exemple le nombre de neurones dans un réseau de neurones, le nombre d’arbres et leur profondeur maximale dans les random forests, etc.
Par ailleurs la classe *CountVectorizer* dans le 3ème stage a un paramètre *minDF* qui permet de ne prendre que les mots apparaissant dans au moins le nombre spécifié par minDF de documents. C’est aussi un hyperparamètre du modèle que nous voulons régler.
Une des techniques pour régler automatiquement les hyper-paramètres est la *grid search* qui consiste à: 
- créer une grille de valeurs à tester pour les hyper-paramètres
- en chaque point de la grille, séparer le training set en un ensemble de training (70%) et un ensemble de validation (30%), entraîner un modèle sur le training set, puis calculer l’erreur du modèle sur le validation set
- sélectionner le point de la grille (<=> garder les valeurs d’hyper-paramètres de ce point) où l’erreur de validation est la plus faible i.e. là où le modèle a le mieux appris

Pour la régularisation de notre régression logistique on veut tester les valeurs de 10e-8 à 10e-2 par pas de 2.0 en échelle logarithmique (on veut tester les valeurs 10e-8, 10e-6, 10e-4 et 10e-2).
Pour le paramètre minDF de CountVectorizer on veut tester les valeurs de 55 à 95 par pas de 20. 
En chaque point de la grille on veut utiliser 70% des données pour l’entraînement et 30% pour la validation.
On veut utiliser le [*f1-score*](https://en.wikipedia.org/wiki/F1_score) pour comparer les différents modèles en chaque point de la grille. Cherchez cette métrique dans *ml.evaluation*.

Préparer la grid-search pour satisfaire les conditions explicitées ci-dessus puis lancer la grid-search sur le dataset "training" préparé précédemment.

### Test du modèle

Tester le modèle obtenu sur les données test

Pour évaluer de façon non biaisée la pertinence du modèle obtenu, il faut le tester sur des données :
- que le modèle n’a jamais vu pendant son entraînement
- et qui n’ont pas servi pour sélectionner le meilleur modèle de la grid search.

C’est pour cela que nous avons construit le dataset de test que nous avons laissé de côté jusque là. 

Appliquer le meilleur modèle trouvé avec la grid-search aux données de test. Mettre les résultats dans le dataFrame `df_WithPredictions`. Afficher le f1-score du modèle sur les données de test.

Afficher
```scala
df_WithPredictions.groupBy("final_status", "predictions").count.show()
```

Sauvegarder le modèle entraîné pour pouvoir le réutiliser plus tard.


## Supplément

Pour plus d’information sur la façon dont est parallélisée la méthode de Newton (pour trouver le maximum de la fonction de coût définissant la régression logistique, qui est le log de la vraisemblance) :
- http://www.slideshare.net/dbtsai/2014-0620-mlor-36132297
- http://www.research.rutgers.edu/~lihong/pub/Zinkevich11Parallelized.pdf
- https://arxiv.org/pdf/1605.06049v1.pdf

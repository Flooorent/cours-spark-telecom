# Quelques notions de Spark, scala, et des ressources 

<!-- TOC -->

- [Quelques notions de Spark, scala, et des ressources](#quelques-notions-de-spark-scala-et-des-ressources)
    - [Le terminal](#le-terminal)
    - [Programmation Orientée Objet en scala](#programmation-orientée-objet-en-scala)
    - [Programmation fonctionnelle](#programmation-fonctionnelle)
    - [Différences entre les RDDs, les DataFrames, et les DataSets](#différences-entre-les-rdds-les-dataframes-et-les-datasets)
    - [Savoir lire la doc](#savoir-lire-la-doc)
    - [Importer des fonctions](#importer-des-fonctions)
    - [Manipuler les colonnes d’un DataFrame](#manipuler-les-colonnes-dun-dataframe)
        - [Notation *col*](#notation-col)
        - [Notation *$*](#notation-)
    - [User Defined Functions](#user-defined-functions)

<!-- /TOC -->

## Le terminal

Quelques commandes utiles pour le terminal (linux, mac):
``` 
cd /some/path/you/want/to/go # change de répertoire, cd <=> "change directory"
cd .. # remonte d’un niveau dans l’arborescence
ls # liste le contenu du répertoire courant, ls <=> "list"
pwd # savoir dans quel répertoire vous êtes, pwd <=> "print working directory"
mkdir jean # crée un répertoire nommé jean, mkdir <=> "make directory"
cp some_file same_file_with_a_new_name : copie un fichier, cp <=> copy
mv /le/path/de/mon/fichier /le/nouveau/path : déplace un fichier (et potentiellement le renommer), mv <=> "move"
```

## Programmation Orientée Objet en scala

Si la programmation objet ne vous dit rien, je vous encourage vivement à en apprendre les concepts de base ! Par exemple avec ce cours : https://openclassrooms.com/courses/apprenez-a-programmer-en-python/premiere-approche-des-classes. C’est un cours en python, mais les notions de classes, attributs et méthodes sont similaires entre différents langages. Il existe de nombreux autres cours, videos, etc. dans différents langages.

Des cours sur scala :
- https://openclassrooms.com/courses/apprenez-la-programmation-avec-scala/pourquoi-scala 
- https://www.coursera.org/learn/progfun1 

**Le point à retenir** : une méthode *method* d’un objet (ou d'une classe) *myObject* est appelée avec la syntaxe suivante :
```
myObject.method(... éventuellement des variables ...)
```

## Programmation fonctionnelle

Scala est aussi un langage de programmation fonctionnelle: http://blog.xebia.fr/2015/05/22/decouvrir-la-programmation-fonctionnelle-1-fonctions/

C’est un concept de programmation puissant et offrant plus de robustesse que l’impératif ou la programmation objet. Scala n’est pas purement fonctionnel mais serait plutôt dans une zone grise entre les deux.

Le point à retenir concernant le fonctionnel avec scala Spark est la composition de fonction :

```scala
// gardons les urls qui ne sont associées qu'à un seul tag
val validUrls: DataFrame = df
  .select("cleanUrls", "tag")
  .groupBy("cleanUrls")
  .agg(countDistinct("tag") as "nbTags")
  .filter(col("nbTags") === 1)
  .select("cleanUrls")
```

Dans l’exemple ci-dessus,  *select*, *groupBy-agg-countDistinct*, et *filter* sont des fonctions qui s’appliquent au dataFrame précédant le point "." et qui renvoient chacune un nouveau dataFrame. On peut donc enchaîner ces fonctions, autrement dit faire de la composition de fonctions, ce qu'on noterait en mathématiques *f(g(h(x)))*. Dans l’exemple ci-dessus on fait *filter(groupBy-agg-countDistinct(select(df)))*.

## Différences entre les RDDs, les DataFrames, et les DataSets

Il est important de bien faire la distinction entre un *RDD* et un *DataFrame* en spark. Ce sont deux objets (classes) différents, qui permettent de faire des choses différentes. 

Le *RDD* est dans les premières versions de spark la seule structure de données, il s’agit d’une liste de tuples, par exemple [(1, 2, 3, 4), (10, 20, 30, 40), ...], **distribuée**. C’est à dire que les tuples sont répartis entre plusieurs exécuteurs (qui peuvent être sur différentes machines). Un *RDD* n’a pas de notion de colonnes, il sert essentiellement à appliquer des traitements façon *Map-Reduce*.

Le *DataFrame* a été construit comme une surcouche sur le *RDD*. Il a une structure de colonnes (voir la classe [Column](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Column) dans la doc) et se présente comme une table dans une base de données. Il offre de nouvelles fonctionnalités par rapport au *RDD*, et se rapproche des dataFrames de pandas en Python. Pour spark, en interne, le *DataFrame* est un *RDD* de [Row](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Row).

Le *DataSet* est une surcouche sur le *DataFrame*, son utilité est essentiellement d’apporter un typage plus fort des colonnes. Il est la structure par défaut à partir de Spark 2.0.0, toutes les méthodes des *DataSets* sont applicables aux *DataFrames*. 

**Le point à retenir** : utilisez les *DataFrames* le plus possible, et lorsque vous voulez appliquer des transformations sur vos *DataFrames* utilisez autant que possible les méthodes déjà implémentées dans Spark qui permettent de manipuler des objets *Column* (voir plus loin).  

## Savoir lire la doc

La doc de Spark contient un User Guide plus "user friendly" : https://spark.apache.org/docs/latest/ 

Et une version détaillée de l’API Spark (dans différents langages), la version scala : https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.package 

Il est important d’apprendre à se servir de la doc détaillée, elle apporte des informations précieuses. Par exemple dans la page de la classe *DataSet* :

![Capture ecran fonction DataSet.drop](images/capture_ecran_doc_drop_dataset.png)

Cet extrait de la doc nous donne plusieurs informations :
- un *DataSet* (et donc un *DataFrame*) a une méthode *drop* qu’on peut utiliser en faisant *df.drop(...)*
- il y a trois façons de se servir de cette méthode: 
  - La 1ère attend un objet *Column* en argument. Exemple : `df.drop($"country")` ou `df.drop(col("country"))`
  - La 2ème attend une série de noms de colonnes (sous forme de *String*). Exemple : `df.drop("country", "currency", "status")`  
  - La 3ème attend un seul nom de colonne. Exemple : `df.drop("country")` 
- la méthode *drop* retourne un *DataFrame*, ce qui signifie entre autres qu’elle peut être enchaînée avec une autre méthode s’appliquant aux *DataFrames* (par exemple avec *filter*).

Les noms de méthodes en bleus sont cliquables et ouvrent parfois des explications supplémentaires quand on clique dessus.

Si vous trouvez une fonction intéressante dans la documentation, le chemin vers la classe qui la contient est indiqué en haut de la page. Par exemple, sur la [page de doc](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$) des fonctions *approx_count_distinct*, *lower*, *datediff*, *udf*, etc., on voit :

![Capture ecran package functions](images/capture_ecran_doc_package_functions.png)

C'est utile pour pouvoir utiliser la fonction, car on est obligé de l'**importer** avant de l'utiliser. Pour utiliser la fonction *approx_count_distinct*, il faudrait par exemple faire :
```scala
import org.apache.spark.sql.functions.approx_count_distinct // import obligatoire

val df = spark
  .createDataFrame(Seq(("France", "Jean"), ("Italie", "Alice"), ("France", "Jeanne"))
  .toDF("country", "person")

df.groupBy("country") // pour chaque pays
  .agg(approx_count_distinct("person") as "nbPeople") // on peut utiliser la fonction
  .show // on affiche le résultat

+-------+--------+
|country|nbPeople|
+-------+--------+
| France|       2|
| Italie|       1|
+-------+--------+
```

## Importer des fonctions

Il existe différentes syntaxes pour importer une ou des fonctions en scala.

Si vous voulez importer uniquement la méthode *lower* de l'objet *sql.functions* de spark :
```scala
import org.apache.spark.sql.functions.lower
```
Si vous voulez importer les méthodes *lower* et *datediff* de l'objet *sql.functions* :
```scala
import org.apache.spark.sql.functions.{lower, datediff}
```
Si vous voulez importer toutes les méthodes de l'objet *sql.functions* :
```scala
import org.apache.spark.sql.functions._
```

## Manipuler les colonnes d’un DataFrame 

Une colonne d’un *DataFrame* spark est accessible en faisant `col("name")` ou `$"col_name"`.

### Notation *col*

*col* fait référence à la fonction *org.apache.spark.sql.functions.col*. Un exemple d'utilisation :
```scala
import org.apache.spark.sql.functions.col

val df = spark
  .createDataFrame(Seq(("jean", 18), ("max", 15)))
  .toDF("name", "age")

df.filter(col("age") >= 18).show

+----+---+
|name|age|
+----+---+
|jean| 18|
+----+---+
```

### Notation *$*

La notation avec *$* est un raccourci de notation pratique. Pour y avoir accès il faut ajouter
```scala
import spark.implicits._
```
à votre code **APRES** avoir défini un *SparkSession* que l’on aurait appelé ici *spark*. Par exemple :
```scala
val spark = SparkSession
  .builder
  .getOrCreate()

import spark.implicits._

val df = spark
  .createDataFrame(Seq(("jean", 18), ("max", 15)))
  .toDF("name", "age")

df.filter($"age" >= 18).show // on peut utiliser le $ grâce à la ligne plus haut

+----+---+
|name|age|
+----+---+
|jean| 18|
+----+---+
```

NB : dans le *spark-shell* un *SparkSession* est créé automatiquement pour vous et l'import `import spark.implicits._` est également fait automatiquement.


Pour savoir quelles opérations/transformations sont possibles sur des colonnes spark, lisez les pages suivantes :
- la [doc de la classe *Column*](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Column) qui liste les méthodes appelables depuis un objet Column. Par exemple isNotNull:
  ```scala
  df.filter($"country".isNotNull)
  ```
  ici *isNotNull* retourne *true* pour chaque ligne de la colonne “country” qui contient une valeur non nulle. 
- la [doc de l'objet *functions*](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions) qui liste les fonctions déjà implémentées dans spark et permettant d’appliquer des transformations sur les colonnes d’un *DataFrame*. Par exemple *lower*:
  ```scala
  df2.withColumn("country", lower($"country")) 
  ```

## User Defined Functions

Si vous avez besoin d'une transformation qui n’est pas implémentée dans Spark, vous pouvez la coder vous-même via une [*User Defined Function*](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.UserDefinedFunction) (aka *UDF*).

Comme nous l'explique la doc, pour créer une *UDF* il faut importer la fonction *udf* de la classe *sql.functions* et ensuite donner votre propre fonction en argument. Exemple :
```scala
import org.apache.spark.sql.functions.udf

// définition de notre udf :
val maSuperUdf = udf((name: String, age: Int) =>
  if (name == "jean" && age >= 18)
    age
  else
    0
)

// des données random
val df = spark
  .createDataFrame(Seq(("jean", 20), ("alice", 22), ("jean", 15)))
  .toDF("some_name", "some_age")

// pour utiliser notre udf :
df.withColumn("modified_age", maSuperUdf($"some_name", $"some_age")).show

+-----+---+------------+
| name|age|modified_age|
+-----+---+------------+
| jean| 20|          20|
|alice| 22|           0|
| jean| 15|           0|
+-----+---+------------+
```

Le type des colonnes "some_name" et "some_age" doit correspondre au type donné aux variables "name" et "age" dans la définition de l’*UDF*. L’*UDF* est appliquée à chaque ligne du *DataFrame* "df" en prenant successivement pour "name" et "value" les valeurs des colonnes "some_name" et "some_age" de chaque ligne. La fonction à l’intérieur de l’*UDF* peut être ce que vous voulez. Ici l’*UDF* prend en input deux colonnes, mais on peut en mettre autant que l'on veut. 

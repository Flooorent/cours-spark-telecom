# FAQ

## Spark

### User Defined Function : `org.apache.spark.SparkException: Failed to execute user defined function`

Si cette erreur est accompagnée d'un `Caused by: java.lang.NullPointerException`, il s'agit le plus souvent d'une mauvaise gestion des valeurs `null`. Concrètement, on applique une fonction, par exemple `.length`, sur un élément qui est `null` : comme cet objet n'a pas de fonction `.length` associée, une erreur `java.lang.NullPointerException` est levée.

Exemple :
```scala
import org.apache.spark.sql.DataFrame
import org.apache.spark.sql.functions.udf

def getLength(col: String): Int = col.length
val getLengthUdf = udf(getLength _)

val df: DataFrame = spark
  .read
  .option("header", true)
  .option("inferSchema", "true")
  .csv("/Users/flo/Documents/github/cours-spark-telecom/data/train_clean.csv")

df.withColumn("countryLength", getLengthUdf($"country"))
  .write
  .parquet("/tmp/spark-test/udf-country")

/*
org.apache.spark.SparkException: Failed to execute user defined function($anonfun$1: (string) => int)
...
Caused by: java.lang.NullPointerException
...
*/

df.filter($"country".isNull).count
// res11: Long = 1
```

C'est l'une des raisons pour lesquelles il est préférable d'utiliser les fonctions déjà codées par Spark plutôt que des `udf` : outre le fait qu'elles soient optimisées par Spark, elles gèrent très souvent (tout le temps ?) les valeurs `null` pour nous. Dans le cas d'une `udf`, on est obligé de gérer manuellement ces valeurs.

Dans cet exercice, vous allez découvrir Spark en explorant des articles complets de Wikipédia.

Dans cet exercice, nous allons utiliser nos données textuelles complètes de Wikipédia pour produire une métrique rudimentaire de la popularité d'un langage de programmation, afin de voir si nos classements basés sur Wikipédia ont un quelconque rapport avec les classements populaires de RedMonk.

Vous réaliserez cet exercice sur un seul nœud (votre ordinateur).

Mais avant, vous devez télécharger le Dataset se trouvant à cette addresse : https://moocs.scala-lang.org/~dockermoocs/bigdata/wikipedia-grading.dat
Ce Fichier sera stocké dans le dossier resources
### Configuration de Spark

Pour des raisons de logistique simplifiée, nous allons exécuter Spark en mode "local". Cela signifie que votre application Spark complète sera exécutée sur un seul nœud, localement, sur votre ordinateur portable.

Pour commencer, nous avons besoin d'un `SparkContext`. Un `SparkContext` est le "handle" de votre cluster. Une fois que vous avez un `SparkContext`, vous pouvez l'utiliser pour créer et remplir des RDD avec des données.

Pour créer un `SparkContext`, vous devez d'abord créer une instance `SparkConf`. Un `SparkConf` représente la configuration de votre application Spark. C'est ici que vous devez spécifier que vous souhaitez exécuter votre application en mode "local". Vous devez également nommer votre application Spark à ce stade. Pour plus d'aide, consultez la documentation de l'API Spark.

Configurez votre cluster pour qu'il s'exécute en mode local en implémentant `val conf` et `val sc`.

(Gardez à l'esprit, que cela sera remplacé par SparkSession dans les versions futures de Spark, mais pour l'instant, nous allons utiliser SparkContext.)

### Lecture des données de Wikipédia

Il existe plusieurs façons de lire des données dans Spark. La manière la plus simple de lire des données est de convertir une collection existante en mémoire en un RDD en utilisant la méthode `parallelize` du contexte Spark.

Nous avons déjà implémenté une méthode `parse` dans l'objet `WikipediaData` qui analyse une ligne du jeu de données et la transforme en un `WikipediaArticle`.

Créez un RDD (en implémentant `val wikiRdd`) qui contient les objets `WikipediaArticle` des articles.

### Calcul d'un classement des langages de programmation

Nous allons utiliser une métrique simple pour déterminer la popularité d'un langage de programmation : le nombre d'articles Wikipédia qui mentionnent le langage au moins une fois.

#### Tentative de classement des langages #1 : `rankLangs`

##### Calcul des occurrences d'un langage

Commencez par implémenter une méthode auxiliaire `occurrencesOfLang` qui calcule le nombre d'articles dans un RDD de type `RDD[WikipediaArticles]` qui mentionnent le langage donné au moins une fois. Pour simplifier, nous vérifions qu'au moins un mot (délimité par des espaces) du texte de l'article est égal au langage donné.

##### Calcul du classement, `rankLangs`

En utilisant `occurrencesOfLang`, implémentez une méthode `rankLangs` qui calcule une liste de paires où le second composant de la paire est le nombre d'articles qui mentionnent le langage (le premier composant de la paire est le nom du langage).

Un exemple de ce que `rankLangs` pourrait retourner pourrait ressembler à ceci, par exemple :

```plaintext
List(("Scala", 999999), ("JavaScript", 1278), ("LOLCODE", 982), ("Java", 42))
```


La liste doit être triée par ordre décroissant. C'est-à-dire que, selon ce classement, la paire avec le plus grand second composant (le compte) doit être le premier élément de la liste.

Faites attention au temps que prend cette partie pour s'exécuter ! (Cela devrait prendre quelques dizaines de secondes.)

### Tentative de classement des langages #2 : `rankLangsUsingIndex`

#### Calcul d'un index inversé

Un index inversé est une structure de données d'indexation qui stocke une correspondance entre le contenu, tel que des mots ou des nombres, et un ensemble de documents. En particulier, le but d'un index inversé est de permettre des recherches de texte intégral rapides. Dans notre cas d'utilisation, un index inversé serait utile pour mapper les noms des langages de programmation à la collection d'articles Wikipédia qui les mentionnent au moins une fois.

Pour rendre le travail avec le jeu de données plus efficace et plus pratique, implémentez une méthode qui calcule un "index inversé" qui mappe les noms des langages de programmation aux articles Wikipédia dans lesquels ils apparaissent au moins une fois.

**Indice :** Vous pourriez vouloir utiliser les méthodes `flatMap` et `groupByKey` sur le RDD pour cette partie.

#### Calcul du classement, `rankLangsUsingIndex`

Utilisez la méthode `makeIndex` implémentée dans la partie précédente pour implémenter une méthode plus rapide pour calculer le classement des langages.

Comme dans la partie 1, `rankLangsUsingIndex` doit calculer une liste de paires où le second composant de la paire est le nombre d'articles qui mentionnent le langage (le premier composant de la paire est le nom du langage).

Encore une fois, la liste doit être triée par ordre décroissant. C'est-à-dire que, selon ce classement, la paire avec le plus grand second composant (le compte) doit être le premier élément de la liste.

**Indice :** La méthode `mapValues` sur `PairRDD` pourrait être utile pour cette partie.

Avez-vous remarqué une amélioration des performances par rapport à la tentative #1 ? Pourquoi ?

### Tentative de classement des langages #3 : `rankLangsReduceByKey`

Dans le cas où l'index inversé ci-dessus n'est utilisé que pour calculer le classement et pour aucune autre tâche (comme la recherche en texte intégral), il est plus efficace d'utiliser la méthode `reduceByKey` pour calculer le classement directement, sans calculer d'abord un index inversé. Notez que la méthode `reduceByKey` n'est définie que pour les RDD contenant des paires (chaque paire est interprétée comme une paire clé-valeur).

Implémentez la méthode `rankLangsReduceByKey`, cette fois en calculant le classement sans l'index inversé, en utilisant `reduceByKey`.

Comme dans les parties 1 et 2, `rankLangsReduceByKey` doit calculer une liste de paires où le second composant de la paire est le nombre d'articles qui mentionnent le langage (le premier composant de la paire est le nom du langage).

Encore une fois, la liste doit être triée par ordre décroissant. C'est-à-dire que, selon ce classement, la paire avec le plus grand second composant (le compte) doit être le premier élément de la liste.

Avez-vous remarqué une amélioration des performances par rapport à la mesure à la fois du calcul de l'index et du calcul du classement comme nous l'avons fait dans la tentative #2 ? Si oui, pouvez-vous penser à une raison ?


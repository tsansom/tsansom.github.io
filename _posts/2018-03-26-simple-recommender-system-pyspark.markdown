---
layout: post
comments: true
title: "Building a Simple Recommender System in PySpark"
subtitle: "Using PySpark to build a simple recommender system based on the MovieLens 100k reviews dataset"
data: 2018-03-26 10:40:24 -0500
categories: spark pyspark recommender
---

In this post, I will walk through a simple recommender system with PySpark from scratch using the MovieLens 100k ratings dataset. I take a novel, weighted approach based on number of similar reviews and cosine similarity.

# Part 0 - Setting up the Environment

To start, setting up the environment to accommodate python, spark, and hadoop can be arduous. I have done it locally a handful of times and have run into different problems every single time without fail. Recently, I have started using Docker for creating stable development environments. Docker is awesome and I kick myself for not using it sooner. Docker installation instructions for different platforms can be found [here](https://docs.docker.com/install/).

The most popular Docker image for PySpark is [jupyter/pyspark-notebook](https://hub.docker.com/r/jupyter/pyspark-notebook/) which exposes a local port for doing python/spark work in a Jupyter notebook. Once Docker has been installed, the image can be downloaded with:

```bash
sudo docker pull jupyter/pyspark-notebook
```

The simplest usage would be to start a container using the command:

```bash
sudo docker run -it --rm -p 8888:8888 jupyter/pyspark-notebook
```

The `-it` flag enables interactivity, `--rm` removes the image upon exiting, and `-p` exposes a local port (in this case 8888).

The drawback of this method is that the notebooks produced are not persistent. To save the notebooks/data produced within this container, then you will need to mount a volume locally which will persist the data generated. This can be done using the -v (or --volume) flag.

Create a working folder to contain the project so Docker knows where to persist the data. I'm using `/home/tsansom/projects/pyspark` as my project path, but change this to fit your needs. Once that's done, you can download the MovieLens 100k dataset from [here](https://grouplens.org/datasets/movielens/). Unzip the compressed file and save it into a data folder in the newly created project folder.

Now, the Docker container can be started with local persistence using the following command:

```bash
sudo docker run -it -p 8888:8888 /home/tsansom/projects/pyspark:/home/jovyan/work --rm --name pyspark jupyter/pyspark-notebook
```

The path `/home/jovyan/work` is simply the working folder within the Docker container. The `--name` flag will explicitly name the container, but is optional. Without this naming flag, Docker will create a random string as the name for the container (mine was initially called *infallible_swartz* before naming).

Once the container is built, there will be a link to the address where you can begin working in Jupyter notebooks with the containerized environment at your fingertips. Copy the link into your browser, go into the `work` folder and you should see the local data directory that you created earlier. Now, any work that you do inside this container will be available locally after the container is stopped.

# Part 1 - Reading and Joining Data

## Importing Libraries

Since this is a from-scratch tutorial, the only packages we need to import are:

```python
import pyspark
import pyspark.sql.functions as F
```

## Starting the SparkSession

The SparkSession is the entry point for programming with the Dataset and DataFrame API. We create a new session with:

```python
spark = pyspark.sql \
               .SparkSession \
               .builder \
               .master('local[*]') \
               .appName('MovieRecommender') \
               .getOrCreate()
```

The `local[*]` command tells the system to use all threads of the local machine.

## Reading the Data

```python
# read in the list of movies, drop the genre
movies = spark.read.csv('../ml100k/movies.csv', header=True)
movies = movies.drop('genres')
movies.show(5, truncate=False)

# read in the ratins, drop the time stamp
ratings = spark.read.csv('../ml100k/ratings.csv', header=True,
                         inferSchema=True)
ratings = ratings.drop('timestamp')
ratings.show(5)
```

which yields the following output.

```
+-------+----------------------------------+
|movieId|title                             |
+-------+----------------------------------+
|1      |Toy Story (1995)                  |
|2      |Jumanji (1995)                    |
|3      |Grumpier Old Men (1995)           |
|4      |Waiting to Exhale (1995)          |
|5      |Father of the Bride Part II (1995)|
+-------+----------------------------------+
only showing top 5 rows

+------+-------+------+
|userId|movieId|rating|
+------+-------+------+
|     1|     31|   2.5|
|     1|   1029|   3.0|
|     1|   1061|   3.0|
|     1|   1129|   2.0|
|     1|   1172|   4.0|
+------+-------+------+
only showing top 5 rows
```

**Note**: Normally you would not want to show the contents of the DataFrame at every step. This is because Spark uses lazy evaluation, meaning that execution will not start until triggered by an action. In this case `show` is an action which forces Spark to execute the operations that have been recorded through the directed acyclic graph (DAG). The advantage of lazy evaluation is that Spark can optimize the workflow decisions based on the entire DAG, as opposed to executing each command as they are received.

## Most Rated Movies

For a simple example of what we can do with these DataFrames, let's show the most rated movies.

```python
ratings.groupBy('movieId') \
       .count() \
       .orderBy('count', ascending=False) \
       .join(movies, ratings.movieId == movies.movieId) \
       .select('title', 'count') \
       .show(10, truncate=False)
```

Which yields:

```
+-----------------------------------------+-----+
|title                                    |count|
+-----------------------------------------+-----+
|Forrest Gump (1994)                      |341  |
|Pulp Fiction (1994)                      |324  |
|Shawshank Redemption, The (1994)         |311  |
|Silence of the Lambs, The (1991)         |304  |
|Star Wars: Episode IV - A New Hope (1977)|291  |
|Jurassic Park (1993)                     |274  |
|Matrix, The (1999)                       |259  |
|Toy Story (1995)                         |247  |
|Schindler's List (1993)                  |244  |
|Terminator 2: Judgment Day (1991)        |237  |
+-----------------------------------------+-----+
only showing top 10 rows
```

## Joining Ratings

Next, we join the ratings table with itself using `userId` as the key. This will give us every pair of movies that a specific user has rated, which is a common approach in user-based collaborative filtering.

```python
joinedRatings = ratings.join(ratings
                             .withColumnRenamed('movieId', '_movieId')
                             .withColumnRenamed('rating', '_rating'),
                             'userId')
joinedRatings.show(5)
```

which yields:

```
+------+-------+------+--------+-------+
|userId|movieId|rating|_movieId|_rating|
+------+-------+------+--------+-------+
|     1|     31|   2.5|    3671|    3.0|
|     1|     31|   2.5|    2968|    1.0|
|     1|     31|   2.5|    2455|    2.5|
|     1|     31|   2.5|    2294|    2.0|
|     1|     31|   2.5|    2193|    2.0|
+------+-------+------+--------+-------+
only showing top 5 rows
```

We use `withColumnRenamed` to make sure that the resulting DataFrame does not have duplicate column names.

Before moving on, we need to remove rows that are duplicated. Take the first entry for example; the user with `userId = 1` will also have an entry where `movieId = 3671` and `_movieId = 31`. To remove these duplicates we can simply filter the data so that it only includes entries where `movieId` is less than `_movieId`. This will also take into account rows which contain the same `movieId`.

```python
joinedRatings = joinedRatings.filter('movieId < _movieId')
```

## Creating Movie Pairs

Our next step is to create a new column which contains the movie pairs. To do this we will create a user defined function (UDF) which joins the `movieId` and `_movieId` fields. To create a UDF we first define the function just as we would in pure Python, and then create a UDF from this function.

```python
def moviePairId(movieId, _movieId):
    return ''.join([str(movieId),'-',str(_movieId)])

udfMoviePairId = F.udf(moviePairId)
```

Now we can use this UDF to create the new `moviePairId` column.

```python
joinedRatings = joinedRatings.withColumn('moviePairId',
                                         udfMoviePairId('movieId',
                                                        '_movieId'))

joinedRatings.show(5)
```

which gives us

```
+------+-------+------+--------+-------+-----------+
|userId|movieId|rating|_movieId|_rating|moviePairId|
+------+-------+------+--------+-------+-----------+
|     1|     31|   2.5|    3671|    3.0|    31-3671|
|     1|     31|   2.5|    2968|    1.0|    31-2968|
|     1|     31|   2.5|    2455|    2.5|    31-2455|
|     1|     31|   2.5|    2294|    2.0|    31-2294|
|     1|     31|   2.5|    2193|    2.0|    31-2193|
+------+-------+------+--------+-------+-----------+
only showing top 5 rows
```

## Cosine Similarity

Cosine similarity is one of the most commonly used measures for similarity between two vectors. Here, we will use cosine similarity to measure the similarity between movies based on which users have rated both movies. The formula for cosine similarity between two vectors ($\mathbf{A}$ and $\mathbf{B}$) is

$$
similarity = cos(\theta) = \frac{\mathbf{A}\cdot\mathbf{B}}{\lVert{\mathbf{A}}\rVert \lVert{\mathbf{B}}\rVert} =  \frac{\sum_{i=1}^nA_iB_i}{\sqrt{\sum_{i=1}^nA_i^2}\sqrt{\sum_{i=1}^nB_i^2}}
$$

We can implement the cosine similarity between movies with the following line of code:

```python
moviePairSimilarities = \
    joinedRatings.groupBy('moviePairId') \
                 .agg((F.sum(joinedRatings.rating * joinedRatings._rating) /
                      (F.sqrt(F.sum(joinedRatings.rating**2)) *
                       F.sqrt(F.sum(joinedRatings._rating**2)))).alias('score'),
                      F.count(joinedRatings.rating).alias('numPairs'))
```

Relating this back to equation (1), $\textbf{A}$ is `rating` and $\textbf{B}$ is `_rating`. I've also created a new column called `numPairs` to count the number of co-occurrences of ratings (*i.e.* the number of reviewers that rated both movies), which will be used later.

Often we will want to keep a DataFrame that will be used many time in memory for fast access. This is simply done using the `cache` command:

```python
moviePairSimilarities.cache()
```

The resulting DataFrame with cosine similarity scores now looks like this

```python
moviePairSimilarities.show(5)
```

```
+-----------+------------------+--------+
|moviePairId|             score|numPairs|
+-----------+------------------+--------+
|  1172-1293| 0.958507806456253|      17|
|  1405-2193|0.9617497019191603|      12|
|     10-273|0.9685067726005836|      16|
|     50-110|0.9517990774006699|     100|
|     62-186|0.9441628304892208|      16|
+-----------+------------------+--------+
only showing top 5 rows
```

## Parsing Movie Pairs

Next, we want to separate the `moviePairId` to store the target and associated movies in separate columns. We'll do this by using more UDFs.

```python
def getMovie1(moviePairId):
    return moviePairId.split('-')[0]
def getMovie2(moviePairId):
    return moviePairId.split('-')[1]

udfGetMovie1 = F.udf(getMovie1)
udfGetMovie2 = F.udf(getMovie2)
```

Now we can create the new columns with these UDFs and delete the `moviePairId` column. I'll keep the same `movieId` and `_movieId` naming convention for consistency.

```python
moviePairScores = \
    (moviePairSimilarities.withColumn('movieId',
                                      udfGetMovie1(moviePairSimilarities.moviePairId))
                          .withColumn('_movieId',
                                      udfGetMovie2(moviePairSimilarities.moviePairId))
                          .drop('moviePairId'))
```

## Getting Recommendations

One problem that occurred to me in this implementation of a recommender system is that the score is invariant to the number of similar ratings between two movies. What I mean by this is that if a movie rating pair only occurs once and they are rated the same, the cosine similarity will be exactly 1 (the highest similarity score possible).

To offset this problem, we will simply create a threshold for the minimum amount of co-ratings to something reasonable (will depend on the size of the dataset).

### Highest similarity score with minimum number of co-ratings

While playing with this method I noticed that no single number for minimum co-ratings is best for each movie. This is because some movies have a lot of co-ratings (>100) while other, less popular movies, will have only a few (<5). To solve this problem, let's get the maximum number of co-ratings, if this number is less than 100, then use half of that number for the minimum number of co-ratings, else use 50.

First let's define the target movie. I'll use one of my favorites: Lord of the Rings: The Fellowship of the Ring (`movieId` is 4993 - can find movies in the **movies.csv** file).

```python
targetMovieId = '4993'
```

Next let's count the maximum number of co-ratings for the target movie.

```python
maxPairs = moviePairScores.where(moviePairScores.movieId == targetMovieId) \
                          .agg({'numPairs': 'max'}) \
                          .collect()[0]['max(numPairs)']
```

Now, use half the amount of maximum co-ratings (or 50 if there are enough co-ratings) as the criteria for minimum amount of co-ratings.

```python
minPairs = min(maxPairs / 2, 50)
```

Finally we can make some recommendations. To do this we will do the following:
* Find our target movie
* Filter by minimum number of co-ratings
* Sort similarity score descending
* Get the movie names for recommended movies
* Select the appropriate columns
* Show the top 10 results

This can all be done with the following code:

```python
targetMovieTitle = movies.where(movies.movieId == targetMovieId).take(1)[0]['title']
print('Top 10 recommendations for movies similar to {0}'.format(targetMovieTitle))
print('*(Minimum of {} co-ratings)'.format(minPairs))

(moviePairScores.where((moviePairScores.movieId == targetMovieId) &
                       (moviePairScores.numPairs > minPairs))
                .orderBy('score', ascending=False)
                .join(movies, moviePairScores._movieId == movies.movieId)
                .select('title', 'score', 'numPairs')
                .show(10, truncate=False))
```

Which yields the following:

```
Top 10 recommendations for movies similar to Lord of the Rings: The Fellowship of the Ring, The (2001)
*(Minimum of 50 co-ratings)
+-----------------------------------------------------+------------------+--------+
|title                                                |score             |numPairs|
+-----------------------------------------------------+------------------+--------+
|Lord of the Rings: The Two Towers, The (2002)        |0.9939514148959071|167     |
|Lord of the Rings: The Return of the King, The (2003)|0.9914685171423369|158     |
|Bourne Supremacy, The (2004)                         |0.9729205118380677|59      |
|Iron Man (2008)                                      |0.9714442438447828|59      |
|Dark Knight, The (2008)                              |0.9705993663966352|85      |
|Bourne Identity, The (2002)                          |0.9686937646122408|77      |
|Spider-Man 2 (2004)                                  |0.9670779665557494|69      |
|Batman Begins (2005)                                 |0.9654948276833364|88      |
|X2: X-Men United (2003)                              |0.9649870759187059|65      |
|Spider-Man (2002)                                    |0.9643662606670581|108     |
+-----------------------------------------------------+------------------+--------+
only showing top 10 rows
```

These results are pretty good! The other two Lord of the Rings movies have the highest similarities which is expected. The rest are action/adventure blockbusters with most of them, besides the Bourne series, residing in the superhero realm.

## Conclusion

This recommendation system works best when there are a lot of ratings for the target movie. With very few ratings, the chance of getting co-ratings becomes very small and the results are not very reasonable. For instance, the top recommendation for Friday the 13th Part 3: 3D is Airplane II: The Sequel with a cosine similarity score of 1 out of 3 co-ratings.

To continue playing with this recommendation system, it might be beneficial to download the entire dataset which consists of 26 million ratings as opposed to the 100 thousand for the small dataset used for this study.

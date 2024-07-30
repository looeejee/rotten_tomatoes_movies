# Upload CSV file to Neo4j Aura

Importing Rotten Tomatoes Best pictures data to Neo4j Aura.

The dataset used in this test is publicly available from Kaggle at the page: https://www.kaggle.com/datasets/bwandowando/rotten-tomatoesall-best-picture-winners-ranked?resource=download


## Steps to reproduce

Create a Free-Tier instance on Neo4j Aura https://neo4j.com/cloud/aura-free/


Once Connected to the DB the commands to load the data and create the graph are the following

### Step-1: Create Constrain for MovieID:

```
CREATE CONSTRAINT Movie_movieId IF NOT EXISTS
FOR (x:Movie)
REQUIRE x.movieId IS UNIQUE;
```

### Step-2: Create Movie Nodes

We will start by creating the Movie Nodes using the movies.csv file in this repository

```
LOAD CSV WITH HEADERS
FROM 'https://raw.githubusercontent.com/looeejee/rotten_tomatoes_movies/main/movies.csv' AS row
MERGE (m:Movie {movieId: row.movieId})
SET
m.movieYear = toInteger(row.movieYear),
m.critic_score = row.critic_score,
m.movieURL = row.movieURL,
m.movieRank = toInteger(row.movieRank),
m.audience_score = row.audience_score,
m.movieTitle= row.movieTitle
```

### Step-3: Create Critics Nodes

We will then create the Critics Nodes using the file critic_reviews.csv

```
LOAD CSV WITH HEADERS
FROM 'https://raw.githubusercontent.com/looeejee/rotten_tomatoes_movies/main/critic_reviews.csv' AS row
MERGE (c:Critic {name: row.criticName}) RETURN c
```

### Step-4: Create Relationship

As a final step we weill create the Relationship (c:Critic)-[r:HAS_REVIEWED]->(m:Movie) using the file critic_review.csv

```
LOAD CSV WITH HEADERS
FROM 'https://raw.githubusercontent.com/looeejee/rotten_tomatoes_movies/main/critic_reviews.csv' AS row
MATCH (c:Critic) WHERE c.name=row.criticName
MATCH (m:Movie) WHERE m.movieId= row.movieId
MERGE (c)-[r:HAS_REVIEWED]->(m)
SET r.reviewId=row.reviewId,
r.creationDate= date(row.creationDate),
r.reviewUrl = row.reviewUrl,
r.isFresh = toBoolean(row.isFresh),
r.originalScore= row.originalScore,
r.quote = row.quote,
r.scoreSentiment = row.scoreSentiment
RETURN c,m,r LIMIT 100
```

### Let's review our new graph
![1]
![2]
![3]

[1]: /img/graph_1.png
[2]: /img/graph_2.png
[3]: /img/table_1.png
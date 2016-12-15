---
layout: default
---


## Part I: Analysis engine overview - Motivation + Graph based predictions

### Introduction

This will be a series of posts describing, in detail, [song proffr](http://songprof.fr). Song proffr is a song recommendation site, which I built this summer as part of the [Insight Data Science Fellowship program](http://insightdatascience.com/).

I love listening to music, driven in part by a love for discovering new songs and artists I have not heard before. A favorite site for finding new music is Hype Machine [Hype Machine](http://hypem.com). Hype Machine aggregates posts from a curated list of music blogs, extracts music linked within the posts, and creates play-lists. With an account, users can bookmark songs by clicking a 'love' icon. Because of this, Hype Machine has a really interesting user dataset: a near constantly fresh list of new songs, and for each song, a list of users which 'loved' the song. 

<img src="/images/songproffr-I-hypemachine-eg.png" width="300px">
*An example of a song feed, as it appears on Hypemachine. One of the songs in the list has been loved me.*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}

The goal of this project was to develop an analysis that could provide music recommendations based on the hypem dataset. Additionally, I wanted to serve these predictions to end users with a clean web interface. 

### Overview of generating predictions

The method I used to generate predictions is based on the assumption that when a group of songs have a large number of overlapping 'loves' between users, that group of songs has some overlap in similarity, and so would make good recommendations for each other. Simply put, if both song A and song B had a large number of users that loved both songs, song A and B have some quality of similarity, and so A would be a good recommendation for B, and vice versa.  

This approach, of relying on user behavior, to identify similarities between products (songs in this case) is the core of [collaborative filtering](https://en.wikipedia.org/wiki/Collaborative_filtering). 

For this project I relied on similar behavior, clustered around songs, to generate recommendations, while not considering at any features intrinsic the song itself (i.e. song tempo, key, length; see the [Music Genome Project](https://en.wikipedia.org/wiki/Music_Genome_Project) for an example of that kind of approach).

Thanks to the above modification, and a number of memory and parallelization optimizations, the entire analysis is able to run on an 8 core machine in approximately 2 hours.

In the future I plan on using traditional collaborative filtering techniques implemented on an Apache Spark cluster, using [MLlib](http://spark.apache.org/docs/latest/mllib-collaborative-filtering.html) (necessary due to size of data, and complexity of calculation) in order to compare results. Apache Spark implementation was outside the scope of this project at the time. 



### Deviations from a classic recommendation system

In the interest of optimization I chose a couple of modifications to the typical collaborative filtering approach. The most common approach is to rely on a similarity measure to create an index between any two items in the population of items. The population of items being all items known to the recommendation system. Some popular similarity indexes include [Cosine similarity,](https://en.wikipedia.org/wiki/Cosine_similarity)[Jaccard index,](https://en.wikipedia.org/wiki/Jaccard_index)[even Edit distance](https://en.wikipedia.org/wiki/Edit_distance). This approach is computationally expensive due to the number of operations involved for calculating the index between every item in the population. 

To reduce computational load of calculating similarity indexes, two modifications were made. First, a custom similarity index was used, simply the number of likes shared by two songs, divided by the total number of likes in the population for those two songs (more on this index below). Secondly, instead of calculating the index for all pairs in the population, a subset of songs that are likely to be good recommendations are first selected, before calculating the similarity indexes. This subset is selected as the top 500 songs in the population, ordered by number of likes. This does bias towards songs with more likes, preferring potentially more popular songs, with a richer dataset of connected songs, over more obscure songs with few connections to other songs. 

A second modification to a classic recommendation system was to add a machine learning classifier model to rate the quality of predictions outputted from the recommendation system. For this, I trained a Gradient Boosted Machine (GBM)based on actual user preferences between songs, and then fed the recommendations through the GBM. More on this, and validation of the approach, in a future post. 

<img src="/images/songproffr-I-recc_list_lbld.png" width="600px">
*Song list generated by recommendation engine, quality of recommendation assessed by GBM.*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}



### The data

The data was scraped from Hypemachine, using a dynamic web-crawling strategy based on cycling between discovering user-names not yet in the local database and retrieving lists of recently loved songs for users with no recent scrape date. I'll write a more on the specifics of the scraper in a later post.

The bulk of the data is saved in the 'user X song interaction' table. This table consists of a row for every loved song the scraper knows about, with one column for user id, and one column for song id. Each row is unique. Detailed user and song information is stored in additional separate tables, and associated through the media and user ids through keys. 

One way the 'user X song interaction' table can be structured is as a
[bipartite graph](https://en.wikipedia.org/wiki/Bipartite_graph) graph. Every song is connected by a user, and every user is connected by a song. An example of the data, represented in this way, is shown below.

<img src="/images/songproffr-I-graph_demo.png" >
*An example of 687 songs with 27 users in common. Users in red, songs in blue.*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}


### Algorithm for Recommending Songs Based on User Data

By identifying songs with groups of users in common, recommendations can be generated. Here is a simple illustration of the process:

<img src="/images/songproffr-I-network-illustration.png" width="300px">
*Further illustration of the relationship between user and songs. Recommendations will be made by looking for songs with groups of users in common*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}

Because the data is structured as bipartite data, matrix operations can be used to quickly and efficiently do much of the calculation required to determine which songs have a lot of overlapping users (and so which songs make good recommendations for each other).

Because of the efficiency of calculations involving vectorization, and matrix algebra (this analysis was done in r) I took steps to take advantage of the existing properties of the data. 

A convenient method of representing bipartite data is to use a matrix, with 1's to represent links, and 0's otherwise. For example, this data is represented with song ids as rows, and user names as columns, and where a user liked a song, the value is 1. 

```
Rows: song ids
Columns: user ids
1 = liked
0 = not liked

> usrsng
      0000AK00 000oo0 007Scooby23007 00D 
10020        1      1              0   0 
10028        1      1              0   0 
1002k        1      1              0   1 
1004a        0      0              1   0 
10052        0      1              0   0 
```

Once in matrix form, the cross product of the matrix will yield a song x song similarity matrix. This is quite a useful calculation for determining which songs have overlapping users, and which do not. 
Below is the same matrix above, after applying a cross product of its self and it's transpose. 

```
Rows & Columns: song ids
value = # users that liked both the song id on row and on column.
        for example, ['10020','10028'] = 2, two users both liked
        songs with id '10020' and '10028'.
> usrsng_tcp
      10020 10028 1002k 1004a 10052
10020     2     2     2     0     1
10028     2     2     2     0     1
1002k     2     2     3     0     1
1004a     0     0     0     1     0
10052     1     1     1     0     1
```

With this matrix, each value is a union, described as total users, that liked any two given songs.
For example, retrieving the row for song id '10020' yields the number of users for all other songs in the matrix, and the number of overlapping user likes.

```
> usrsng_tcp['10020',]
10020 10028 1002k 1004a 10052 
    2     2     2     0     1 
```

From the above example, for song id '10020', song ids '10028' and '1002k' appear to have the higher number of overlapping users. These two songs might make good recommendations. 

### Implementation
The big drawback to the approach outlined above, is that when there are many songs and/or many users, the size of such a matrix can quickly exceed available memory. Fortunately, due to the nature of the data, it is efficiently represented as a sparse matrix. Sparse matrices excel at representing data which consists of a majority of 0 values. The simplest example is of a matrix containing 1's or 0's. Instead of storing the value held in each row/column location in the matrix, the sparse matrix representation stores just the row/column indexes where the value is 1. 
When a matrix has more 0 values than 1 values, it is easy to see that a substantial amount of memory can be saved with this representation. 

Here, the 'user x song interaction' table (one column for user id, and one column for song id) is pulled from the SQL database, translated into a sparse matrix representation, and the cross product transpose calculated. This was implemented in R, using the Matrix library for sparse matrix support (the library also supports matrix multiplication of sparse matrices). 
This design allowed for an efficient and vectorized implementation of finding songs with large user similarities between songs. 

The final step involves iterating through all the songs in the database, querying the song similarity matrix, retrieving a subset (top n*5 songs) with user overlap, calculating similarity index, and returning the top n songs based on index values. The returned result, of each query, is saved to a 'Predictions' table. 

The similarity index is calculated as the number of users that liked two songs, by the total number of likes for those two songs in the database. For example, a score of 1 (for a pair of songs) represents 100% of users who liked both songs always liked those songs together. A low score, close to 0, represents low degree of agreement between users who liked both songs. In other words, songs that have many likes in common, but also have many likes with many songs, will be penalized. Songs with  likes are consistently distributed between songs will have the best scores. 

<img src="/images/songproffr-I-normalization.png" width="600px">
{: style="color:gray; font-size: 80%; text-align: center; width="300px"}
*Normalization strategy. Dividing the number of likes that two songs share, by the total number of likes for both songs.*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}


By pre-computing all recommendation for all songs, and saving it to the predictions table, recommendation data can be quickly served from a web front end. This gives the user much quick results compared to if the recommendations were computed from scratch every time. The downside to this approach, is that the predictions database must be re-generated whenever the user x song interaction table is updated. This is a trade off, in the end, favoring speed for the end user, at a cost of not always serving the freshest data available. 

```
An example of the predictions table.
  mediaid: id of recommended song
  source: source song for which recommendation is for
  date: data which the recommendation was generated

   mediaid source date
1    10xem  10028 1479848969
2    2f8ta  10028 1479848969
3    1xrbe  10028 1479848969
4    26kk3  10028 1479848969
5    2g1gf  10028 1479848969
6    2evkp  10028 1479848969

```

The 'Predictions' output table consists of all of the source songs, a list of similar songs for each source song, and metrics associated with the number of similar likes for recommended songs. More on these metrics, and how they were used, in the next post.


<br> 

### Next Steps

The above process is the basic design the song proffer engine uses to find similar songs. In the next posts, I will describe how I used a Gradient Boosted Machine to improve rate the quality of the recommendations. Addtional topics include the process of writing the front end code to serve the predictions as a website, and the steps I took to optimize the back-end SQL database in serving data quickly to the front end web server. 



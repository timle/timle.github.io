---
layout: default
---

## Songprof.fr song recommendations - Part I - Analysis engine overview - Motivation + Graph based predictions

This will be a series of posts, describing in detail a song recommendation site, which I built this summer.

### Part I: Analysis engine overview - Motivation + Graph based predictions

#### Introduction

I tend to listen to a lot of music - a part of this is driven by a love for discovering new music. My favorite site for finding new music is Hype Machine (http://hypem.com). Hype Machine aggregates posts from a curated list of music blogs, extracts music linked within the posts, and creates play-lists. With an account, users can bookmark ('love') songs that they like. Because of this, Hype Machine has a really interesting dataset. A near constantly fresh list of new songs, and for each song, a list of users which 'loved' the song. 

<img src="/images/songproffr-I-hypemachine-eg.png">
*An example of a song feed, as it appears on Hypemachine. One of the songs in the list has been loved me.*
{: style="color:gray; font-size: 80%; text-align: center; width="500px"}

The goal of this project was to generate music recommendations based on user similarities between songs. The assumption is that when songs have a large number of 'loves' from the same users, that those songs would make good recommendation for each other. For example, if there was a big overlap in the number of users that loved both song A and song B, song A would be a good recommendation for song B, and vice versa. 

This approach, of relying on user behavior, to identify similarities between products (songs in this case) is the core of collaborative filtering. Relying on similar behavior clustered around certain songs to generate recommendations, and not considering at any features intrinsic to the song itself (i.e. tempo, key, length).

#### The data

The data was scraped from Hypemachine, using a web-crawling strategy that based on a cycling between discovering user-names not yet in the local database and retrieving lists of recently loved songs for recently added users. I'll write a more on the specifics of the scraper in a later post.

The bulk of the data is saved in the 'user x song interaction' table. This table consists of a row for every loved song the scraper knows about, with one column for user id, and one column for song id. Each row is unique. Detailed user and song information is stored in additional separate tables, and associated through the media and user ids through keys. 

One way to the user x song interaction table structure is as a bipartite
[bipartite](https://en.wikipedia.org/wiki/Bipartite_graph) graph. Every song is connected by a user, and every user is connected by a song. An example of the data, represented in this way, is shown below.

<img src="/images/songproffr-I-graph_demo.png" >
![alt text](/images/songproffr-I-graph_demo.png "does this work?")
*An example of 687 songs with 27 users in common. Users in red, songs in blue.*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}

#### Making first predictions

By identifying songs with groups of users in common, recommendations can be generated. Here is a simple illustration of the process:


<img src="/images/songproffr-I-network-illustration.png" width="300px">
*Further illustration of the relationship between user and songs. Recommendations will be made by looking for songs with groups of users in common*
{: style="color:gray; font-size: 80%; text-align: center; width="600px"}

Because the data is structured as bipartite data, matrix operations can be used to quickly and efficiently do a lot of the calculation required to determine which songs have a lot of overlapping users (and so which songs make good recommendations for each other).

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

For song id '10020', the song ids '10028' and '1002k' appear to have the higher number of overlapping users. These two songs might make good recommendations. 

#### Implementation
The big drawback to the approach outlined above, is that when there are many songs and/or many users, the size of such a matrix can quickly exceed available memory. Fortunately, due to the nature of the data, it is efficiently represented as a sparse matrix. Sparse matrices excel at representing data which consists of a majority of 0 values. The simplest example is of a matrix containing 1's or 0's. Instead of storing the value held in each row/column location in the matrix, the sparse matrix representation stores just the row/column indexes where the value is 1. 
When a matrix has more 0 values than 1 values, it is easy to see that a substantial amount of memory can be saved with this representation. 

Here, the 'user x song interaction' table (one column for user id, and one column for song id) is pulled from the SQL database, translated into a sparse matrix representation, and the cross product transpose calculated. This was implemented in R, using the Matrix library for sparse matrix support (the library also supports matrix multiplication of sparse matrices). 
This design allowed for an efficient and vectorized implementation of finding songs with large user similarities between songs. 

The final step involved iterating through all the songs in the database, querying the song similarity matrix, and retrieving the top n songs with user overlap.
The 'Predictions' output table at this step consisted of all of the source songs, a list of similar songs for each source song, and metrics associated with the recommended songs. (More on what the metrics were, and how they were used in the next post. )

This output was the very first version of the song recommendation engine.  A very simple algorithm that finds songs with high degree of user overlap, and returns these songs as recommendations. However, refinements were badly needed. Using this method, without normalizing for number of likes, meant that popular songs inevitably rose to the top of recommendation lists.

<img src="/images/songproffr-I-logo.png" width="300px">

#### Next steps
In the next post, I will outline the process of normalizing predictions based on total number of likes (popular songs connect to many many more users than average songs, complicating original assumptions) and using a Gradient Boosting Machine, trained on user behavior, to help improve the quality of predictions. 

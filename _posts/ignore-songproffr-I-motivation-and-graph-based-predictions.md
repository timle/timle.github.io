---
layout: default
---

## Songprof.fr song recommendations - Part I - Analysis engine overview - Motivation + Graph based predictions

This will be a series of posts, describing in detail a song recommendation site, which I built this summer.

### Part I: Analysis engine overview - Motivation + Graph based predictions

#### Introduction

I tend to listen to a lot of music, and so am always on the lookout for new bands. My favorite site for finding new music is Hype Machine (http://hypem.com). Hype Machine aggregates posts from a curated list of music blogs, extracts music linked within the posts, and creates play-lists. With an account, users can bookmark ('love') songs that they like. Because of this, Hype Machine has a really interesting dataset. A near constantly fresh list of new songs, and for each song, a list of users which 'loved' the song. 

The goal of this project was to generate music recommendations based on user similarities between songs. The assumption is that when songs have a large number of 'loves' from the same users, that those songs would make good recommendation for each other. For example, if there was a big overlap in the number of users that loved both song A and song B, song A would be a good recommendation for song B, and vice versa. 

This approach, of relying on user behavior, to identify similarities between products (songs in this case) is the core of collaborative filtering. Relying on similar behavior clustered around certain songs to generate recommendations, and not considering at any features intrinsic to the song itself (i.e. tempo, key, length).

#### The data

The data was scraped from Hypemachine, using a web-crawling strategy that based on a cycling between discovering user-names not yet in the local database and retrieving lists of recently loved songs for recently added users. I'll write a more on the specifics of the scraper in a later post.

The bulk of the data is saved in the 'user x song interaction' table. This table consists of a row for every loved song the scraper knows about, with one column for user id, and one column for song id. Each row is unique. Detailed user and song information is stored in additional separate tables, and associated through the media and user ids through keys. 

One way to think of the structure of the user x song interaction table is as a bipartite (https://en.wikipedia.org/wiki/Bipartite_graph) graph. Every song is connected by a user, and every user is connected by a song. 
---
layout: default
---

### songprof.fr - How I built a song recommendation site - Part I

Series of posts, describing a song recommendation site which I built. The planned outline is as follows:
## Part I: Analysis engine overview - Motivation + Graph based predictions
## Part II: Analysis engine overview - GBM to improve song predictions
## Part III: Front end overview - Flask + Predictions Database
## Part IV: Front end overview - Similar artist recommendations 
## Part V: Improvements - DB performance

Blog text


## Part II: Analysis engine overview - GBM to improve song predictions

Metrics included number of likes a song received in total (total number of likes in the db) and number of likes this song had in common with the source song (intersection of likes between the two songs).
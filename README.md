# Cosmocloud MongoDB Task 2
## Made By Ikjot Singh

This project involves creating MongoDB aggregation pipelines for the `mflix` database to perform various operations on the `movies` and `comments` collections. Each question includes the aggregation pipeline query and a concise explanation.

---

## Question 1: Fetch Movies with Comments
**Task:** Retrieve the title of each movie along with all its associated comments.

### Pipeline:
```json
[
    {
      "$lookup": {
        "from": "comments", 
        "localField": "_id", 
        "foreignField": "movie_id", 
        "as": "comments" 
      }
    },
    {
      "$project": {
        "_id": 0, 
        "title": 1, 
        "comments": {
          "$map": {
            "input": "$comments",
            "as": "comment", 
            "in": {
              "name": "$$comment.name",
              "email": "$$comment.email",
              "text": "$$comment.text",
              "date": "$$comment.date"
            }
          }
        }
      }
    }
]
```
**Explanation:** Performs a `$lookup` to join `movies` with `comments` on `movie_id` and projects the movie title and relevant comment fields.

---

## Question 2: Count Comments for Each Movie
**Task:** Count the number of comments for each movie and include the title.

### Pipeline:
```json
[
    {
      "$lookup": {
        "from": "comments",
        "localField": "_id", 
        "foreignField": "movie_id", 
        "as": "comments"
      }
    },
    {
      "$project": {
        "_id": 0, 
        "title": 1, 
        "commentCount": { "$size": "$comments" } 
      }
    }
]
```
**Explanation:** Uses `$size` to calculate the number of comments per movie after joining the collections.

---

## Question 3: Top 5 Movies by IMDb Rating
**Task:** Find the top 5 movies with the highest average IMDb rating, including title, IMDb rating, and total comment count.

### Pipeline:
```json
[
    { "$match": { "imdb.rating": { "$exists": true, "$type": ["double", "int"] } } },
    { "$project": { "title": 1, "imdbRating": "$imdb.rating" } },
    { "$sort": { "imdbRating": -1 } },
    { "$limit": 5 },
    { "$lookup": {
        "from": "comments",
        "let": { "movieId": "$_id" },
        "pipeline": [
          { "$match": { "$expr": { "$eq": ["$movie_id", "$$movieId"] } } },
          { "$count": "count" }
        ],
        "as": "commentsCount"
      }
    },
    { "$addFields": {
        "commentCount": { "$ifNull": [ { "$arrayElemAt": ["$commentsCount.count", 0] }, 0 ] }
      }
    },
    { "$project": { "_id": 0, "title": 1, "imdbRating": 1, "commentCount": 1 } }
]
```
**Explanation:** Filters movies with valid IMDb ratings, sorts by rating, limits to the top 5, and calculates comment counts via `$lookup` and `$count`.

---

## Question 4: Unique Cast Members and Movie Count
**Task:** Get a list of all unique cast members and the number of movies each appeared in.

### Pipeline:
```json
[
    { "$unwind": "$cast" },
    { "$group": { "_id": "$cast", "movieCount": { "$sum": 1 } } },
    { "$project": { "_id": 0, "castMember": "$_id", "movieCount": 1 } },
    { "$sort": { "movieCount": -1 } }
]
```
**Explanation:** Uses `$unwind` to break cast arrays, `$group` to count movies for each cast member, and sorts by movie count.

---

## Question 5: Movies Before 1950 with High IMDb Rating
**Task:** Find movies released before 1950 with IMDb ratings of 7.0 or higher, including title, release year, genres, and first 2 comments.

### Pipeline:
```json
[
    { "$match": { "imdb.rating": { "$gte": 7.0 }, "year": { "$lt": 1950 } } },
    { "$lookup": { "from": "comments", "localField": "_id", "foreignField": "movie_id", "as": "comments" } },
    { "$addFields": { "comments": { "$slice": [ "$comments", 2 ] } } },
    { "$project": {
        "_id": 0,
        "title": 1,
        "releaseYear": "$year",
        "genres": 1,
        "imdbRating": "$imdb.rating",
        "comments": {
            "name": 1,
            "text": 1
        }
      }
    }
]
```
**Explanation:** Matches movies by release year and IMDb rating, includes first 2 comments using `$slice`, and projects relevant fields.



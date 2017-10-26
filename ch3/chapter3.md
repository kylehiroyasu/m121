# Chapter 3:

## `$group`

Groups the data by specified field and aggregates field as described.

For example:

```javascript
// grouping by year and getting a count per year using the { $sum: 1 } pattern
db.movies.aggregate([
  {
    "$group": {
      "_id": "$year",
      "numFilmsThisYear": { "$sum": 1 }
    }
  }
])

// grouping as before, then sorting in descending order based on the count
db.movies.aggregate([
  {
    "$group": {
      "_id": "$year",
      "count": { "$sum": 1 }
    }
  },
  {
    "$sort": { "count": -1 }
  }
])
```

Remember it is important consider the raw data you're aggregating on. You can validate the documents you're aggregating over using things like the following:

```javascript
// grouping on the number of directors a film has, demonstrating that we have to
// validate types to protect some expressions
db.movies.aggregate([
  {
    "$group": {
      "_id": {
        "numDirectors": {
          "$cond": [{ "$isArray": "$directors" }, { "$size": "$directors" }, 0]
        }
      },
      "numFilms": { "$sum": 1 },
      "averageMetacritic": { "$avg": "$metacritic" }
    }
  },
  {
    "$sort": { "_id.numDirectors": -1 }
  }
])

// showing how to group all documents together. By convention, we use null or an
// empty string, ""
db.movies.aggregate([
  {
    "$group": {
      "_id": null,
      "count": { "$sum": 1 }
    }
  }
])

// filtering results to only get documents with a numeric metacritic value
db.movies.aggregate([
  {
    "$match": { "metacritic": { "$gte": 0 } }
  },
  {
    "$group": {
      "_id": null,
      "averageMetacritic": { "$avg": "$metacritic" }
    }
  }
])

```

https://docs.mongodb.com/manual/reference/operator/aggregation/group?jmp=university&_ga=2.204102608.1953097513.1508998821-430246252.1491462147&_gac=1.213794278.1505388559.Cj0KCQjwruPNBRCKARIsAEYNXIiZ2uiemihnA2bXmitXEfBsHVJHDX7XjyhwncMy76CeY5XXm4LrvyYaAnSiEALw_wcB



## Expressions (Simple Functions)

* Expressions are equivalent to functions
* specific syntax and argument ordering withan expressionis depentent on the expression
* Expressions ares composable


## Lab

In the last lab, we calculated a normalized rating that required us to know what the minimum and maximum values for imdb.votes were. These values were found using the $group stage!

For all films that won at least 1 Oscar, calculate the standard deviation, highest, lowest, and average imdb.rating. Use the sample standard deviation expression.

HINT - All movies in the collection that won an Oscar begin with a string resembling one of the following in their awards field


## `$unwind`

Used to create one document per value in the array.

```javascript
// finding the top rated genres per year from 2010 to 2015...
db.movies.aggregate([
  {
    "$match": {
      "imdb.rating": { "$gt": 0 },
      "year": { "$gte": 2010, "$lte": 2015 },
      "runtime": { "$gte": 90 }
    }
  },
  {
    "$unwind": "$genres"
  },
  {
    "$group": {
      "_id": {
        "year": "$year",
        "genre": "$genres"
      },
      "average_rating": { "$avg": "$imdb.rating" }
    }
  },
  {
    "$sort": { "_id.year": -1, "average_rating": -1 }
  }
])
```
There are two forms short and long:

```javascript
$unwind: <field path>


$unwind: {
	 path : <field path>,
	 includeArrayIndex: <string>,
	 preserveNullAndEmptyArrays: <boolean>
}
```
Using unwind on large documents may cause performance issues so use with care.

## `$lockup`

## `$graphlookup`

## `$
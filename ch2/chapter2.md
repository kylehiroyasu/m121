# Chapter 2: Basic Aggregation - Utility Stages

## `$addFields` and how it is simila to `$project`

Extremely similar to project except it only add news new computed fields or modify existing incoming fields.

This allows us to avoid specifying all the fields we want to exclude/include which can become tedious.

Using `$project` and `$addFields` together in combination allows you to first specify the fields you want to use select and then compute additional transformations on top of them. this is of course a stylistic choice.

```javascript
// combining ``$project`` with ``$addFields``
db.solarSystem.aggregate([
{"$project": {
    "_id": 0,
    "name": 1,
    "gravity": 1,
    "mass": 1,
    "radius": 1,
    "sma": 1}
},
{"$addFields": {
    "gravity": "$gravity.value",
    "mass": "$mass.value",
    "radius": "$radius.value",
    "sma": "$sma.value"
}}]);
```

more about addfields (here)[https://docs.mongodb.com/manual/reference/operator/aggregation/addFields?jmp=university&_ga=2.145502774.383554180.1508741592-430246252.1491462147&_gac=1.227930735.1505388559.Cj0KCQjwruPNBRCKARIsAEYNXIiZ2uiemihnA2bXmitXEfBsHVJHDX7XjyhwncMy76CeY5XXm4LrvyYaAnSiEALw_wcB]

## `geoNear` stage

This is the aggregation pipeline solution for querying on geographic data.  It must come in the first step of the aggregation pipeline.

*NOTE* : The difference between `$near` and `$geoNear` is that the latter works on sharded clusters while the former does not.

`$near` also limits on additional aggregations/operations performed in combination with it/
`$geoNear` requires the documents we query on have only one geoIndex.

The required field to use `$geoNear` are the following:
* `near`
* `distanceField`
* `spherical`

```javascript
// using ``$geoNear`` stage
db.nycFacilities.aggregate([
  {
    "$geoNear": {
      "near": {
        "type": "Point",
        "coordinates": [-73.98769766092299, 40.757345233626594]
      },
      "distanceField": "distanceFromMongoDB",
      "spherical": true
    }
  }
]).pretty();
```

Required documents
* `near` is the point of interest
* `distanceField` is the distance we're searching
* `spherical` True specifies 2D sphere (?)

Optional

* `minDistance`/`maxDistance` specify the min/max distances you can use to search documents
* `includeLocs` includes the location used if the doc includes more than one location
* `limit`/`num` are functionally identical, limiting the number of documents returned
* `distanceMultiplier` converts distances from radians to whatever unit we need in case we have legacy geospatial data in different units

Things to remember:

* The collectiona can have only one 2dsphere index
* If using 2dsphere, the distance is returned in meters. If using legacy coordinates, the distance is returned in radians.
* `$geoNear` must be the first stage in an aggregation pipeline

More details (here)[https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear?jmp=university&_ga=2.186807722.383554180.1508741592-430246252.1491462147&_gac=1.224916712.1505388559.Cj0KCQjwruPNBRCKARIsAEYNXIiZ2uiemihnA2bXmitXEfBsHVJHDX7XjyhwncMy76CeY5XXm4LrvyYaAnSiEALw_wcB]

## Cursor-Like Stages

* `.sort()`
* `.skip()`
* `.limit()`
* `.count()`

Are used in cases like:

```javascript
// project fields ``numberOfMoons`` and ``name``
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).pretty();

// count the number of documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).count();

// skip documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).skip(5).pretty();

// limit documents
db.solarSystem.find({}, {"_id": 0, "name": 1, "numberOfMoons": 1}).limit(5).pretty();

// sort documents
db.solarSystem.find({}, { "_id": 0, "name": 1, "numberOfMoons": 1 }).sort( {"numberOfMoons": -1 } ).pretty();
```
There are also stages that can be used in the aggregation pipeline:

* `$sort: { <field we want to sort on>: <integer, direction to sort> }`
* `$skip: { <integer> }`
* `$limit: { <integer> }`
* `$count: { <name we want the count called> }`

Here are some examples of what they look like in a query:

```javascript
// ``$limit`` stage
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
},
{ "$limit": 5  }]).pretty();

// ``skip`` stage
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
}, {
  "$skip": 1
}]).pretty()

// ``$count`` stage
db.solarSystem.aggregate([{
  "$match": {
    "type": "Terrestrial planet"
  }
}, {
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
}, {
  "$count": "terrestrial planets"
}]).pretty();

// removing ``$project`` stage since it does not interfere with our count
db.solarSystem.aggregate([{
  "$match": {
    "type": "Terrestrial planet"
  }
}, {
  "$count": "terrestrial planets"
}]).pretty();


// ``$sort`` stage
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "numberOfMoons": 1
  }
}, {
  "$sort": { "numberOfMoons": -1 }
}]).pretty();

// sorting on more than one field
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "hasMagneticField": 1,
    "numberOfMoons": 1
  }
}, {
  "$sort": { "hasMagneticField": -1, "numberOfMoons": -1 }
}]).pretty();

// setting ``allowDiskUse`` option
db.solarSystem.aggregate([{
  "$project": {
    "_id": 0,
    "name": 1,
    "hasMagneticField": 1,
    "numberOfMoons": 1
  }
}, {
  "$sort": { "hasMagneticField": -1, "numberOfMoons": -1 }
}], { "allowDiskUse": true }).pretty();

```


## `$sample` Stage

This stage will select a ranom sample of documents from a given collection.

There are basically two different methos of how this will happen:

```javascript
 $sample: { size : <N, how many documents> }
```

*Method One*

If:

* `N` is less than 5% of the documents in source collection
* source collection has more tahn 100 documents
* `$sample` is the first stage

Then:
a psuedo random cursor will select the specific number of documents to be passed on.

Else:
An in-memory random sort will select the specified number of documents. Keep in mind this is still restricted to the usual 36mb memory restriction.

More infom about `$sample` (here)[https://docs.mongodb.com/manual/reference/operator/aggregation/sample?jmp=university&_ga=2.87087258.383554180.1508741592-430246252.1491462147&_gac=1.186056027.1505388559.Cj0KCQjwruPNBRCKARIsAEYNXIiZ2uiemihnA2bXmitXEfBsHVJHDX7XjyhwncMy76CeY5XXm4LrvyYaAnSiEALw_wcB]

## Lab: Using Cursor-like Stages

```bash
mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/aggregations?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl -u m121 -p aggregations --norc
```

For movies released in the USA with a tomatoes.viewer.rating greater than or equal to 3, calculate a new field called num_favs that represets how many favorites appear in the cast field of the movie.

Sort your results by num_favs and then tomatoes.viewer.rating, both in descending order.

What is the title of the 25th film in the aggregation result?

```javascript

favorites = [
  "Sandra Bullock",
  "Tom Hanks",
  "Julia Roberts",
  "Kevin Spacey",
  "George Clooney"]

db.movies.aggregate([
	{$match: {
       	       countries: { $eq : "USA"},
       	       "tomatoes.viewer.rating" : {$gte: 3}
	       }
	},
	{$project: {
		   num_favs: {$size:
				{"$ifNull": [
			     		 {$setIntersection: ["$cast", favorites ]}, []
					 ]
				}
			     },
		   title: 1,
		   "tomatoes.viewer.rating": 1
		   }
	},
	{$sort: {
		num_favs: -1,
		"tomatoes.viewer.rating": -1
		}
	}
])

```

## Lab: Bringing it all together



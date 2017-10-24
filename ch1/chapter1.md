## `$match`

* a $match may contain a $text query operatory but it must be the first stage in the query operator
* we cannot use $where
* match can use indexes to make filetering quickly
* should come early in the aggregation pipeline
* match does not allow us to project any data


Lab 1:

```javascript
var pipeline = [{ $match : { "imdb.rating" : {$gte : 7}, "genres": {$nin : [ "Crime","Horror"]}, "rated" : { $in : ["PG", "G"]}, "languages" : { $all : ["English", "Japanese"]} } }]
db.movies.aggregate(pipeline).itcount() // would expect 23 documents
load('validateLab1.js')
validateLab1(pipeline)
```


## `$project`

Selectively remove retain fields
apply/reassign fields and values

Here's an example of using the `$project`
```javascript
db.solarSystem.aggregate([{$project : {_id: 0, name: 1, "gravity.value": 1 }}])
db.solarSystem.aggregate([{$project : {_id: 0, name: 1, gravity:"$gravity.value" }}]) //reassigning gravity field
db.solarSystem.aggregate([{ $project : {_id: 0, name: 1, { $divide: ["gravity.value", 9.8]} } }]) // derving a new field
```

as soon as we specify project we have to specify all fields we want.

Lab 2:

```javascript
var pipeline = [{ $match : { "imdb.rating" : {$gte : 7}, "genres": {$nin : [ "Crime","Horror"]}, "rated" : { $in : ["PG", "G"]}, "languages" : { $all : ["English", "Japanese"]} }},
    	     { $project : {"title":1, "rated":1, _id : 0} }
    ]

db.movies.aggregate(pipeline).itcount() // would expect 23 documents
load('validateLab1.js')
validateLab1(pipeline)
```

Lab 3:
```javascript
db.movies.aggregate([ { $project : {count: { $size : { $split : ["$title", " "]} }, title:1 } }, { $match : { count : {$lt:2} } } ]).itcount()
```

Optional Lab:
...





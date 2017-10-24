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


More details (here)[https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear?jmp=university&_ga=2.186807722.383554180.1508741592-430246252.1491462147&_gac=1.224916712.1505388559.Cj0KCQjwruPNBRCKARIsAEYNXIiZ2uiemihnA2bXmitXEfBsHVJHDX7XjyhwncMy76CeY5XXm4LrvyYaAnSiEALw_wcB]

## Cursor-Like Stages

## `$sample` Stage

## Lab: Using Cursor-like Stages

## Lab: Bringing it al together
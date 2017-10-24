# Chapter 0: Introduction and Aggregation Concepts

As usual there are labs, quizzes, and finals. The quizzes are ungraded and thus labs and the final each make up about 50% percent of your final grade. A 60% or higher grade overall is passing.

## Atlas Requirement

You can spin up a free sandbox instance of Atlas on the mongo website

There is also a specific, pre-built instance that is needed for this course. Using the mongo shell you can connect to the instance here:

> mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/aggregations?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl -u m121 -p aggregations --norc

Using `show collections` you should see something like the following:

```
air_airlines
air_alliances
air_routes
bronze_banking
customers
employees
exoplanets
gold_banking
icecream_data
movies
nycFacilities
silver_banking
solarSystem
stocks
system.views
```

## The Concept of Pipelines

### What are pipelines

Think of pipelines as conveyor belts for documents with various stages. There can be a wide range of stages in different pipelines. For example one could use have the following:

1. `$match` : Filter documents/only allow certain documents to enter the pipeline
2. `$project`: Transform the data that has been matched into a new form
3. `$group` : can merge all of the projected data into a single statistic.

In summary:

* Pipelines are a composition of stages
* Stages are configurable for a transformation
* Data flows through pipelines like water through a pipe
* With a few exceptions, stages can be ordered and used in any number we like

## Aggregation Structure and Syntax

Pipeline can have 1 or more stages.

Options are also available to do things like
* Use disk when the query is especially large (`allowDiskUse`)
* Show explain plan to understand if indexes are being used

Operators is also another name for the aggregation stages like `$match` and `$project`. Query operators refer to things like `$gte`, `$in`, etc.

As a general rule, operators always show up in the key position.

There are also some variables to be aware of:
 
Field Path: `$fieldName` (`$numberOfMoons`) - used to access the field of a given document
System Variable: `$$UPPERCASE` (`$$CURRENT` - current document) - system level variable
User Variable: `$$foo`- user variable, which we temporarily bind to a name

### Basic Aggregation Stucture and Syntax Rules

* Pipelines are always an array of one or more stages
* Stages are composed of one or more aggregation operators or expressions
* Expressions may take a single argument or an array of arguments. This is expression dependent


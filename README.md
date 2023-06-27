
https://www.freecodecamp.org/news/nosql-databases-5f6639ed9574/
https://en.wikipedia.org/wiki/Distributed_data_store
https://en.wikipedia.org/wiki/CAP_theorem
https://www.mongodb.com/basics/acid-transactions
https://en.wikipedia.org/wiki/Eventual_consistency


mongodb: document db
other nosql dbs: key-value dbs, wide-column oriented dbs, graph dbs

doc dbs: 
key-values within docs, and values can be arrays, other docs, etc. 
docs are stored in collections.
collections are stored in dbs
docs are schema-less (documents are polymorphic)
	advantage vs relational: no columns full of nulls (less memory), a lot of flexibility in adding data, etc


documents in mongodb:
- json format (curly braces, :, "key")
- id field is unique, as "_id"... automatically created if not given
- docs embedded within docs (subdocuments)
- values can be arrays (ex: several phone numbers)
- many data types (numeric, date, string, etc)
- a field can store many datatypes by default (needs to be restricted explicitly)

Questions
- data duplication? Is this a problem?
- how do we keep duplicated stuff updated/synced?
- performance: doesn't data duplication use more memory?
- how should we model data?

Good practice:
- include as much info as possible
- take care in not creating confusing fields ("phone" vs "cellphone")


Ecosystem:
Community ed
Entreprise ed
Atlas: cloud on AWS GCP or AZure.. hasfree tier
Mongodb Charts: let's you viz what data you have
Realm: suite of services for developing mobile apps
Tools: shell, drivers, BI connectors, Compass (GUI)

ATLAS: has online GUI
COMPASS: GUI
COMMAND LINE: mongosh... and then add mongoimport/export/dump/restore

HUMAN READABLE IMPORT EXPORT
mongoexport --uri="mongodb+srv://user:pass@sandbox.q3qkwek.mongodb.net/import_export_db" --collection=import_export_coll1 --out=import_export_coll1.json
mongoimport --uri="mongodb+srv://user:pass@sandbox.q3qkwek.mongodb.net/import_export_db" --collection=import_export_coll1 --file=import_export_coll1.json

BINARY IMPORT EXPORT (NON HUMAN READABLE, BUT EFFICIENT)
mongodump --uri="mongodb+srv://user:pass@sandbox.q3qkwek.mongodb.net/import_export_db" --collection=import_export_coll1 --out=import_export_coll1.json
mongodump --uri="mongodb+srv://user:pass@sandbox.q3qkwek.mongodb.net/import_export_db" # FULL DB
mongorestore --uri="mongodb+srv://user:pass@sandbox.q3qkwek.mongodb.net/import_export_db" dump/import_export_db



mongosh connect...
show dbs
use <database_name>
show collections

with target db active (with "use <dbname>"):
- db.createCollection('test_coll') #case sensitive ('test_coll', <options>)
- db.test_coll.drop()
- db.dropDatabase()
- db.<collection>.findOne(query, projection) #arguments can be None
- db.<collection>.find(query, projection) #arguments can be None
- db.grades.find({"class_id": 419}) # query is its own json.
- db.grades.find({"student_id": 4, "class_id": 363}) # several conditions

DOLLAR SIGN
PRECEDES OPERATORS
{field: {operator: value}}
$lt is "less than"
- {"salary": {$lt: 50000}}
$gt
PRECEDES field values
AGGREGATION PIPELINE

mongodb.com/try


{field: {operator: value}}
{$expr: {operator: [field, value]}}

For ex:

{"tripduration": {$gt: 400}}
{$expr: {$gt: ["$tripduration", 400]}}


To filter for equality between fields:
{$expr: {$eq:["$src_airport", "$dst_airport"]}}



Element operators:
$exists ---> {field: {$exists: <boolean>}}
$type ---> {field: {$type: <BSON type>}} # or [<BSON type1>, <BSON type2>, ...]

db.companies.find({"ipo": {$exists: true}})
db.companies.find({"homepage_url":{$type: 2}})




Cursor methods
count --> db.coll.find(...).count()
sort --> db.coll.find(...).sort({field1: -1, "field2": 1}) #-1 descending, 1 ascending
limit --> db.coll.find(...).limit(number)
skip --> db.coll.find(...).skip(number)
size --> db.coll.find(...).skip/limit(number).size() #same as count, but only works after skip or limit


Projection (select)
db.coll.find(query, projection)
projection can be array, or {"field1": 1, "field2": 1, ... "_id":0} # Zero means exclude



Querying embedded documents: SPOLIER: dot notation
db.coll.find({embedded_doc_field.embedded_field: {operator: value}})


Querying arrays
db.posts.find({array_field: value_in_them}) # checks for value in array
db.posts.find({array_field: array_of_values}) #checks for equality of arrays (includes order)
db.posts.find({array_field: {$all: [field1, field2]}}) #checks for subsets
db.posts.find({array_field: {$size: n}}) # returns docs such that array has length n
db.coll.find({"field.subfield": "value"}) # if field is array of embedded docs, and subfield is an attribute ofthose embedded docs... this retrieves docs such that field contains an embedded doc whose subfield = value
$elemMatch ---> Returns docs with an array field with an element that contains the matching criteria
db.coll.find({field : {$elemMatch : {queries}}})
	For example
	db.grades.find({"scores": {$elemMatch: {"type": "exam", "grade": {$gt: 80}}}}) #DOESN'T USE DOT NOTATION



### inseting

db.collection_name.insert(<document or array of documents>)

if collection with name collection_name doensnt exists, mongo will create it

if ids not supplied, mongo will make them

cafeul not to try to insert document with explicitly given id that already exissts. will result in error.

### deleting

de.collection.deleteOne(<filter>) or db.collection.deleteMany(<filter>)

for example

db.collection.deleteMany({"name": "Tom Hanks"})

db.collection.deleteOne({"awards.academy": true})


### updaing

db.collection.updateOne({filter}, {update}, {options}) or db.collection.updateMany({filter}, {update}, {options})

there are several {update} operators...

$set: replaces

example

$set:{field: value, field2, value2, ...}

db.people.updateMany({"profession": "chef"}, {$set: {"profession": "cook"}})

if no filter is provided ({}), then all documents will be updated
if a field doesn't exist, then it will be created

Also, $unset. Same synta

$inc, increments field by value
$rename
$push, adds elements to array

example:

db.people.updateMany({}, {$unset: {"new_field", ""}, $rename: {"hobbies_list": "hobbies"}})

db.people.updateOne({"name": "Meera Patel"}, {$push: {"hobbies": "reading"}})


###### options: upsert

Upsert: if no matching documents, a new one will be inserted. By default false.


## aggregation

db.collection.aggregate([{stage1}, {stage2}, {stage3}, ..., {stagen}], {options})

- $match: what we already know...
- $project: like a select {$project: {field1: 1, field2: 1}}.... something cool that we can only do with aggregation, which is to create fields from others, so like...

db.collection.aggregate([{$project: {"new_field": "$existing_field"}}])
db.collection.aggregate([{$project: {"new_field": {$divide: {"$existing_field": 1000}}}])

examples:

- db.trips.aggregate([{$match: {"stop_time": {$gt: ISODate("2016-01-15")}}}])
- db.trips.aggregate([{$project: {"start_station_location": "$start_station_location.coordinates"}}])


## arithmetic operators

There are obvious operators, like $sum, $multiply, $divide, $subtract, $round, $ln, etc...

**Important! These can be used directly in $project aggregation pipeline steps, but they need to be within $expr conditions to be used in $match steps**

e.g. {$match: {"field1": {$gt: $multiply: ["$field2", 20]}}}

instead, use: {$match: {$expr: {$gt: ["field1", {$multiply: ["$field2", 3]} ]}}}

### string operators

$concat, $toUpper, $toLower, $regexMatch

**Important! These can be used directly in $project aggregation pipeline steps, but they need to be within $expr conditions to be used in $match steps**


### comparison expression operators


$gt, $lt... $cmp

As expression operators, syntax is {operator: [expression1, expression2]}, whereas in query language it can be like this {field: {operator: value}}


### array expression operators

$isArray $arrayElemAt $concatArrays $first $last, $size $map

{$map: {input: expression, as: string, in: expression}}



db.posts.aggregate([{$project:{"tags":1, "is_array_flag":{$isArray: "$tags"}, "third": {$arrayElemAt:["$tags", 2]}}}])


db.posts.aggregate([{$match: {$expr: {$eq: [$arrayElemAt: ["$tags", 2], "daisy"]}}}])


db.posts.aggregate([{$project: {$map: {input: "$tags", as: "blabla", in: {$toUpper: "$$blabla"}}}}])

### conditional expression operators

$cond -> if else
$ifNull -> coalesce
$switch -> switch

### assignment

db.trips.aggregate([{$project: {"journey_time": {$divide: [{$subtract: ["$stop_time", "$start_time"]}, 1 000]}}}])


### $addfields

Like project, but only for adding fields


### cursor stages

{$count: newfieldname}
{$limit: number}
{$skip: number}
{$sort: {field1: order1 (1 or -1, for ascending or descending), ..., fieldN: orderN}}


### $group

{$group: {"_id": expression, "field1": {accumulator1: expression1},... "fieldn": {accumulatorn: expressionn}}}

db.shapes.aggregate([{$group:{"id": "$colour", "total_value": {$sum: "$value"}}}])

if "_id": null, then this accumulates over the whole db

if no fields are supplied, it works as a DISTINCT on "_id"

trick: count == {$sum: 1}

### bucket and bucketsAuto

what youd expect... groups by bins

$bucket: {groupBy: expression, boundaries: [lowerbound1, lowerbound2,..., lowerboundn], default: string literal, output: {output1: {accumulator exxpression}, ... outputn: [accummulator expression]}}

no output just gives count

if documents are outside buckets, default needs to be specified or you will get an error. default is the id of teh catchall bucket for these documents

db.trips.aggregate[{$bucket: {groupBy: "$tripDuration", boundaries: [0, 100, 200], }}]

_id can be an array of fields

$bucketAuto: instead of boundaries, you set {...buckets: no_of_buckets, ...} also, you can set a granularity (see mongodb for docs)

### $facet

generates several subcollections based on severeal subpipelines

$facet:
	{
		sub_pipeline1 : [statge1, stage2, ...],
		sub_pipeline2 : [statge1, stage2, ...],
		...
	}


### Questions

JOINS
UNIONs
What happens when a document's field's value doesn't satisfy requirements to be used in aggregation expression (averaging with nulls, or with nonexistent fields?)


### unwind

makes documents out of elements of an array field (and keeps other fields). NOTE to self, can be used to do joins

$unwind:
	{
		path: field to unwind,
		includeArrayIndex: string, //optional
		preserveNullAndEmptyArrays: boolean,
	}

includeArrayIndex adds a field that holds the index of the unwound field in the original array


### $out

to save pipeline output in new collection in the same db. will fail if there are duplicate document _ids (for example, after doing $unwind) 


## all together

we can write pipeline stages and store them as variables in mongo shell

stage1 = {$match: {$eq: {"$cityname": "HOUSTON"}}}


### System variables...

Access with $$

Example... $$ROOT points to "self" in each document. Together with $push (into an array) grouping operator, it can be used to get an array of documents for each group (THIS ALLOWS doing something like joins)


## user define variables

$let :
	{
		vars : {var1, expression1, ...},
		in: expression
	}

access them with two dollar signs $$ as in array map

db.trips.aggregate([
	{
		$addFields:
		{
			"tripduration_120%":
			{
				{$let:
					{"vars": {"tDur": "$tripDuration", "miltiplier": 120},
					"in": {$multiply:["$$tDur", "$$multiplier"]
					},

				
				}
			}
		}
	}
])


### $lookup

THIS IS A LEFT OUTER JOIN

db.users.aggregate([
	{$lookup:
		{
			"from": "comments",
			"localField": "name",
			"foreignField": "name",
			"as": "comments"
		}
	}
])
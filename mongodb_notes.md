
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


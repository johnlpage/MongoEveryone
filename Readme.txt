This is a tool which will allow you to offer acess to Data held in MongoDB throughout an organisation.
It allows you to set queries users can run, set up aggregations they can run, allow them to view data and to browse links between data.

I'm using the IMDB data set to develop it - any questions ask me, dont expect it just to work for you yet.


Backlog, in order
---------

*NB - this has been hacked, grown and evolved - a top down rewrite with Tests would be nice *
*I'm still trying to get my head round Angular*
*
Change to use Get Params not Page state!

Create a demo
[Version 1 DEMO]


Linkchart
Multiple Links / Link Labeling
Label Link (Actor Details), (All Movies)

[Version 2 Demo]

Maps and Geo

[Version 3 demo]
Auditing
Login 
User management
Query Security
Redaction
Print templates
Admin roles
Query term highlighting??

Getting Started
----------------

Run intel.py (python intel.py)
Point browser at localhost:8080

You need to define at least one search
A search is defined as a mongoDB document in demos.searches


{ _id: Unique Name
  description: How it's shown to the user
  query: JSON QUERY
  collection:
  database: 
  summary: JSON PROJECTION
  aggregate: [True / False] - is this an aggregation pipeline or a query
  links: [ Array of objects { from: FIELDNAME, tosearch: "Name of search" ]
  visible: [True/False] - is it on the list
  drilldown: Agg only - Name of Search to run on click - existing paramsas passed + RecordID (which is _id)
  formorder: Array of objects desribing search form in order (optional)
  objects are {name: THe Label and placeholder, type 'Choice | Str | Int', field : (used with Choice to specify field for dropdown contents)
}
  JSON QUERY is a String containing the JSON, not a JSON Object for a search, you can use placeholders as @Fieldname in the query
  which will be shown as the fields to search. All strings must be quoted.
  
  JSON PROJECTION is s String containing the JSON for a MongoDB 'project'
  
For Example 
use demos

db.searches.insert({
 _id: 'MoviesFTS',
 description: 'Full Movie Text Search',
 collection: 'data',
 database: 'imdb',
 summary: '{"title":1}',
 query: '{ "$text": { "$search": "@Query" },"rectype":"prod"}',
 links: [ {from: "actorno", tosearch: "ActorByID"} ],
 visible: true
 })
 
  db.searches.insert({
	"_id" : "MovieByID",
	"description" : "Fetch Movie By ID",
	"collection" : "data",
	"database" : "imdb",
	"summary" : "{\"title\":1}",
	"query" : "{ \"_id\":\"@movieno\" }",
	 links: [ {from: "actorno", tosearch: "ActorByID"} ],
	 visible: false
})



 
 db.searches.insert({
 _id: 'ActorByID',
 description: 'Fetch Actor By ID',
 collection: 'data',
 database: 'imdb',
 summary: '{"name":1}',
 query: '{ "_id":"@actorno"}',
  links: [ {from: "movieno", tosearch: "MovieByID"} ],
  visible: false
 })
 
 
 
 db.searches.insert({
 _id: 'ActorsFTS',
 description: 'Actors Text Search',
 collection: 'data',
 database: 'imdb',
 summary: '{"name":1}',
 query: '{ "$text": { "$search": "@Query" },"rectype":"pers"}',
 links: [ {from: "movieno", tosearch: "MovieByID"} ],
  visible: true
 })
 

 
 //@ placeholder
 //@# Numeric
  db.searches.save({
 _id: 'MoviesAgg',
 description: 'Top Actors by Production',
 collection: 'data',
 database: 'imdb',
 aggregate: true,
 summary: '{"name":1}',
 drilldown: 'MoviesByActor',
 formorder: [{name:'Genre',type:'Choice',field:'genres'},{name:'From',type:'Int'},{name:'To',type:'Int'}],
 query: '[{ "$match" : {"genres":"@Genre","datefrom":{ "$gt": "@#From", "$lt": "@#To"} }},{ "$unwind" : "$actors"},{ "$group": {"_id":"$actors.name", "Movies":{ "$sum" : 1}}},{ "$sort": {"Movies":-1}}]',
 visible: true
 })
 
 db.searches.save({
 _id: 'MoviesByActor',
 description: 'Movies by Actor,Genre,Year',
 collection: 'data',
 database: 'imdb',
 summary: '{"title":1}',
 query: '{"actors.name":"@RecordID","genres":"@Genre","datefrom":{ "$gt": "@#From", "$lt": "@#To"}}',
 links: [ {from: "movieno", tosearch: "MovieByID"} ],
 visible: false
 })
 
 
 
 
pick = { $match : {genres:"Western",datefrom:{ $gt: 1950, $lt: 1960} }}
project = { $project: { "actors.name":1 }}
unwindcast  = { $unwind : "$actors.name"}
countroles = { $group: {_id:"$actors.name", rolecount:{ $sum : 1}}}
sortcount = { $sort: {rolecount:1}}
 



Old notes
------------

Generic viewer. [Done]
Paged results[Done]
Free text query
Fixed queries
Query builder
Linked objects
Map query
Map results
Link viewer
Homepage
Done with a reactive UI for mobile?
Simplest possible ui?
HTML forms?
No data entry

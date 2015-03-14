JSON ODM v0.0.2 by [Konsultaner](http://www.konsultaner.de), Richard Burkhardt
========

This Project aims to be provide a json object document mapper.  
**Why would I need this?**  
There are many cases where you do not want to have the server query your data. You may also see this mapper as a fancy way to filter your data or prepare a view model.
The original use case was an [ionic](http://ionicframework.com) app that was not interactive so providing a server backend was not needed, but structuring my data
into joinable collections seemed very helpful.  

Docs
----
You can find the [documentation here](https://cdn.rawgit.com/konsultaner/jsonOdm/master/doc/jsonOdm/0.0/index.html) or compiled to the folder [/doc/jsonOdm/0.0/*](https://github.com/konsultaner/jsonOdm/blob/master/doc/jsonOdm/0.0/index.html). 

Road map
----

- Implement query methods from [mongo db](http://docs.mongodb.org/manual/reference/operator/query/)
    - Comparison (~~$eq~~,~~$gt~~,~~$gte~~,~~$lt~~,~~$lte~~,~~$ne~~,~~$in~~,~~$nin~~) 100%
    - Logical (~~$or~~,~~$and~~,~~$not(= $nand as alias)~~,~~$nor~~) 100%
    - Element (~~$exists~~,~~$type~~) 100% (+ $isNull, for is null or undefined)
    - Evaluation (~~$mod~~,~~$regex~~,$text,$where) 50%
    - Geospatial ($geoWithin,$geoIntersects,$near,$nearSphere) 5%
    - Array ($all,$elementMatch,$size) 0% // i may not implement the first two
    - Projection Operators ($,$elemMatch,$meta,$slice) 0%
- Support Promise/A+ sources for the odm.addSource
- Full ~~CR~~U~~D~~ support
- Aggregation

Required NPM-Packages
---------------------

To build it from source use command ```gulp``` and after adding:
```bash
npm install --save-dev gulp
npm install --save-dev gulp-concat
npm install --save-dev gulp-filesize
npm install --save-dev gulp-jsdoc
npm install --save-dev gulp-uglify
npm install --save-dev jsdoc3-bootstrap
```

To execute the benchmark use command ```gulp bench``` after adding:
```bash
npm install --save-dev gulp-bench
```

Example
------
```javascript
// initialize The mapper
var odm = new jsonOdm();
// add a source to the mapper
odm.addSource('people',{
   "Person" : [
       {"id":1,"name":"Richi",jobId:1,hobbyIds:[1,3,4]},
       {"id":2,"name":"Dave",jobId:2,hobbyIds:[2,4]},
       {"id":3,"name":"Tom",jobId:3,hobbyIds:[3,5]},
       {"id":4,"name":"Lisa",jobId:4,hobbyIds:[1,2,3]},
       {"id":5,"name":"Hanni",jobId:3,hobbyIds:[1,5]},
       {"id":6,"name":"Selma",jobId:3,hobbyIds:[1,4]},
       {"id":7,"name":"Ralf",jobId:1,hobbyIds:[4,3]}
   ],
   "Jobs" : [
       {"id":1,"name":"plumber"},
       {"id":2,"name":"programmer"},
       {"id":3,"name":"chef"},
       {"id":4,"name":"hairdresser"}
   ],
   "Hobbies" : [
       {"id":1,"name":"swimming"},
       {"id":2,"name":"cycling"},
       {"id":3,"name":"fishing"},
       {"id":4,"name":"coding"},
       {"id":5,"name":"dancing"}
   ]
});
// instantiate a collection object
var people = new jsonOdm.Collection('Person');
people.$hasOne("jobId","id","Jobs","job");
people.$hasMany("hobbyIds","id","Hobbies","hobbies");

var q = people.query();
// get all hairdresser and plumber
var hairdresser = q.$or(
   q.$branch("jobId").$eq(1)
   q.$branch("jobId").$eq(4)
).$all();

// get all but hairdressers and plumbers
var hairdresser = q.$and(
   q.$branch("jobId").$ne(1)
   q.$branch("jobId").$ne(4)
).$all();

// delete all plumbers
q.$branch("job","name").$eq("plumber").$delete();
```
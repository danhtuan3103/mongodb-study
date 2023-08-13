# MongoDB Tutorial (https://www.youtube.com/watch?v=cPzNMpkQm2o&t=5007s)

## Setup docker

---

1. Chạy ngầm container with --detach

- `docker-compose -f mongo-compose.yml up --detach`

2. Vào terminal của container

- `docker exec -it mongo-container bash`

3. Download data

- `curl https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive`

4. Run mongo on container

- `mongosh "mongodb://root:Abc123456789@localhost:27018" --username root --authenticationDatabase admin`

5. Restore data in mongodb

- `mongorestore --username root --password Abc123456789 --authenticationDatabase admin --archive=./sampledata.archive`

View logs of container

- `docker logs [container-name]`

---

## Commands

Switch between docs

- `use [collections]`

### Query Documents

| db.[collection].find( {điều kiện} , { option } )

// Find all docs
`db.movies.find({})`
// Find count of documents
`db.movies.find().count()`
// find docs with only year and plot field
`db.movies.find({}, {year: 1, plot: 1})`
// Find with option 2010 < year < 2012
`db.movies.find({year: {$gte: 2010, $lte: 2012}})`
`db.movies.find({year: {$gte: 2010, $lte: 2012}}).count()`
`db.movies.find({year: {$gte: 2010, $lte: 2012}}, {year: 1, plot: 1, _id: 0})`

### Limit and Skip (Paging)

// Lấy document đầu tiên
`db.movies.find({year: {$gte: 2010, $lte: 2012}}, {year: 1, plot: 1, _id: 0}).limit(1)`

// paging
db.movies.find({}, {title: 1}).skip(0).limit(5)
db.movies.find({}, {title: 1}).skip(5).limit(5)

### Distinct

// with 'type' attr
`db.movies.distinct('type')`
// with nested attr
`db.movies.distinct('imdb.rating')`

### Nested

// Find all docs with rating = 5
`db.movies.find({'tomatoes.viewer.rating': 5})`
`db.movies.find({'tomatoes.viewer.rating': 5}, {tomatoes: 1})`
`db.movies.find({'tomatoes.viewer.rating': 5}, {tomatoes: 1}).count()`

### Sort

> Phải thêm exists để tránh trường hợp không có trường tomatoes.viewer.rating

// find max of rating
`db.movies.find({'tomatoes.viewer.rating' : {$exists: true }}).sort({'tomatoes.viewer.rating': -1}).limit(1)`
// find min of rating
`db.movies.find({'tomatoes.viewer.rating' : {$exists: true }}).sort({'tomatoes.viewer.rating': 1}).limit(1)`

### $IN

`db.movies.find({directors: {$in: ['Joris Ivens']}}, {directors: 1, year: 1, title: 1})`

`db.movies.find({directors: {$in: ['Joris Ivens', 'Reginald Barker']}}, {directors: 1, year: 1, title: 1}).count()`

db.movies.find({directors: {$size: 1}}, {directors: 1, year: 1, title: 1}).count()

### $SIZE

// find movies with only one director
`db.movies.find({directors: {$size: 1}}, {directors: 1, year: 1, title: 1})`

### CACULATED FIELD

// find number of directors and add in new field
`db.movies.find({}, {
    title: 1,
    year: 1,
    directors: 1,
    numberOfDirectors: {$size: '$directors'}
})`

### Aggregate , sometime better than 'find' command - PIPELINE\

`db.[collection].aggregate(
    {Stage 1},
    {Stage 2},
    {Stage 3},
    ....
)`
// find doc with title = '....'
`db.movies.aggregate({$match: {title: 'In the Land of the Head Hunters'}}, {$project: {title: 1, directors: 1, year: 1, _id: 0}})`
// Length
`db.movies.aggregate({$match: {title: 'In the Land of the Head 
Hunters'}}, {$project: {title: 1, directors: 1, year: 1, _id: 0}}).toArray().length`
// find all docs with aggregate
`db.comments.aggregate({$match: {}})`
// find comment of user that have email = '...'
`db.comments.aggregate({$match: {email: 'shawn_mccormick@fakegmail.com'}})`

// and , or
`db.movies.aggregate({$match: {$and: [{'imdb.votes': 884}, {type: 'movie'}, {year: 1917}]}})`

`db.movies.aggregate({$match: {$and: [{type: 'movie'}, {year: 1917}] }}, {$project: {type: 1, year: 1, title: 1}})`

`db.movies.aggregate({$match: {$and: [{type: 'movie'}, {year: {$in: [1917, 1928]}}] }}, {$project: {type: 1, year: 1, title: 1}}).toArray().length`

// Find count of comments for each movie
` db.comments.aggregate([{$group: {_id: '$movie_id', numberOfComments: {$count: {}}}}])`

// lookup from comment to movie
`db.comments.aggregate([
    {$group: {_id: '$movie_id', numberOfComments: {$count: {}}}}, {$lookup: {from : "movies", localField: "_id", foreignField: "_id", as: "detail"}}])
`

// TEXT

`db.comments.aggregate({$match: {text : /.* Beatae ex quis odio..*/i}})`
i = ignore case

// find text start with ................
`db.comments.aggregate({$match: {text : /^Quas .*/i}})`
// find text end with ................
`db.comments.aggregate({$match: {text : /.* assumenda labore\.$/i}})`

// limit in aggregate
` db.comments.aggregate({$match: {}}, {$sort: {name: -1}}, {$limit: 1})`

// exists in aggregate
`db.movies.aggregate({$match: {writers: {$exists: true}}})`

// Addfields in aggregate
`db.movies.aggregate({$match: {writers: {$exists: true}}}, {$addFields: {numberOfWriter: {$size: "$writers"}}})`

` db.movies.aggregate({$match: {writers: {$exists: true}}}, {$addFields: {numberOfWriter: {$size: "$writers"}, numberOfAwards: "$awards.wins"}})`

`db.movies.aggregate({$match: {writers: {$exists: true}, awards: {$exists: true}}}, {$addFields: {numberOfWriter: {$size: "$writers"}, numberOfAwards: "$awards.wins"}})`

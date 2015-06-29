
class: center, middle, inverse-slide

# Enhancing .meteor-text[Meteor] <br> with .elasticsearch-text[Elasticsearch]

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]

---
.left-column[
# About Me
]

.right-column-middle[
- Software Engineer at Respondly
- Previously built tools at Apple
- Before Respondly I've never <br> integrated search into a product
]
---

.left-column[
## How We Use Search
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/respondly-screenshot.png" width="100%"/>
]

---

.left-column[
## How We Use Search
### Search Conversations
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-conversations.png" width="100%"/>
]

---

.left-column[
## How We Use Search
### Search Conversations
### Search Contacts
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-contacts.png" width="100%"/>
]

---

.left-column[
## How We Use Search
### Search Conversations
### Search Contacts
### Archive Conversations
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-archive.png" width="100%"/>
]

---

class: center, middle, inverse-slide

# Sync .mongodb-text[MongoDB] <br> and .elasticsearch-text[Elasticsearch]

---

class: center, middle, inverse-slide

# .mongodb-text[MongoDB] Rivering

---

class: center, middle, red-slide

# ~~MongoDB Rivering~~

---

class: center, middle

# Let's Make Our <br> Own .blue-text[River]

---

.left-column[
## Data Sync
### Enqueue Index Tasks
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-queue.png" width="80%"/>
]

---

.left-column[
## Data Sync
### Enqueue Index Tasks
]

.right-column-middle[
```javascript
// Meteor App - Server
SearchableCollection = new Meteor.Collection(
  'searchable-collection'
);

var enqueueTask = function (id, type) {
  var  task = {
    id: id,
    type: type,
    collection: 'searchable-collection'
  };
  // enqueue task to Amazon SQS
  // see: https://www.npmjs.com/package/aws-sdk
};

var cursor = SearchableCollection.find({});
cursor.observeChanges({
  added: function (id) {
    enqueueTask(id, 'added');
  },
  changed: function (id) {
    enqueueTask(id, 'changed');
  },
  removed: function (id) {
    enqueueTask(id, 'removed');
  },
});
```
]

---

.left-column[
## Data Sync
### Enqueue Index Tasks
### Index Task Worker
]

.right-column-middle[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-worker.png" width="80%"/>
]

---

.left-column[
## Data Sync
### Enqueue Index Tasks
### Index Task Worker
]

.right-column-middle[
```javascript
// Worker APP
function added(db, esClient, id, collection) {
  // query MongoDB for the document
  db.collection(collection).find({_id: id}).toArray(
    function (err, docs) {
      // index data on elasticsearch
      esClient.index({
        index: collection,
        type: collection,
        id: id,
        body: docs[0] // because there's only one doc
      });
    }
  )
}

// changed is same as add
// function changed(db, esClient, id, collection)

function removed(db, esClient, id, collection) {
  esClient.delete({
    index: collection,
    type: collection,
    id: id,
  });
}
```
]

---

class: center, middle, inverse-slide

# So Data Is In Sync, <br> How Do I .elasticsearch-text[Search] It?

---

.left-column[
## Search
### Search Publication
]

.right-column-middle[
- Use .meteor-text[Meteor] to build a publication that accepts a query
- Let .elasticsearch-text[Elasticsearch] do the heavy lifting for queries
- Use scored results from .elasticsearch-text[Elasticsearch] to return collection data
]

---

.left-column[
## Search
### Search Publication
]

.right-column-middle[
```javascript
// Meteor App - Server
var Future = Npm.require('fibers/future');
Meteor.publish('search', function(q) {
  var future = new Future();
  // es client - https://www.npmjs.com/package/elasticsearch
  esClient.search({
    index: 'searchable-collection',
    q: q, // querystring
    size: 10, // 10 results
    function (err, response) {
      // grab all the ids
      var hitIds = _.pluck(reponse.hits.hits, '_id');
      // find all of documents by id and publish them
      future.return(
        SearchableCollection.find({_id: {$in: hitIds}})
      );
    }
  });
});
```
]

---

.left-column[
## Search
### Search Publication
### Search Subscription
]

.right-column-middle[
- On the client subscribe with a querystring
- Render data with .meteor-text[Meteor] template
]

---

.left-column[
## Search
### Search Publication
### Search Subscription
]

.right-column-middle[
```javascript
// Meteor App - Client

// do a search every time the querystring changes
Deps.autorun(function(){
  var q = Session.get('querystring'); // reactive
  if (q) {
    Meteor.subscribe('search', q);
  }
});

// populate the `searchResults` template with result
Template.searchResults.results = function () {
  SearchableCollection.find();
};

// update the querystring when the search input changes
Template.searchResults.events({
  'keyup #search-input': function (ev, template) {
    var value = template.find('#search-input').value;
    Session.set('search', value);
  }
});
```
]

---

.left-column[
## Elasticsearch Hosting
]

.right-column-middle[
- Self Hosting
- Elasticsearch on [AWS](https://github.com/elastic/elasticsearch-cloud-aws)
- Heroku [Add Ons](https://addons.heroku.com/?q=elasticsearch)
- Elasticsearch as a service
]


---

.left-column[
## Elasticsearch Hosting
### Elasticsearch As A Service
]

.right-column-middle[
- We chose to go with [Found](https://www.found.no/)
- Scale up, down and do upgrades with zero downtime
- Technical support was awesome
]

---

class: middle

.left-column[
## Lessons Learned
]

.right-column-middle[
- Get your [sharding scheme](https://www.found.no/foundation/sizing-elasticsearch/) figured out as soon as possible
- .elasticsearch-text[Elasticsearch] automatic mapping and default analyzers are [really smart](https://www.found.no/foundation/elasticsearch-mapping-introduction/)
- Lucene [Query Parser syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html) is very powerful, many don't know it
]

---

class: center, middle

# Demo

.footnote[[Project On Github](https://github.com/hharnisc/meteor-elasticsearch-demo)]

---

class: center, middle

# Questions?

.footnote[Made with [remark](https://github.com/gnab/remark)]

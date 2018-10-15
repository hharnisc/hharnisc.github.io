
class: center, middle

# Enhancing .meteor[Meteor] <br> with .elasticsearch[Elasticsearch]

.footnote[Harrison Harnisch ([@hjharnis](https://twitter.com/hjharnis))]

---

class: fifty-fifty

.left-panel[
# About Me
]

.right-panel[
- Software Engineer at Respondly
- Previously built tools at Apple
- Before Respondly I've never <br> integrated search into a product
]

---

class: fifty-fifty

.left-panel[
# How We Use Search
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/respondly-screenshot.png" width="100%"/>
]

---

class: fifty-fifty

.left-panel[
# How We Use Search: Conversations
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-conversations.png" width="100%"/>
]

---

class: fifty-fifty

.left-panel[
# How We Use Search: Contacts
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-contacts.png" width="100%"/>
]

---

class: fifty-fifty

.left-panel[
# How We Use Search: Archive Conversations
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-archive.png" width="100%"/>
]

---

class: center, middle

# Sync .mongodb[MongoDB] <br> and .elasticsearch[Elasticsearch]

---

class: center, middle

# .mongodb[MongoDB] Rivering

---

class: center, middle

# .red[~~MongoDB Rivering~~]

---

class: center, middle

# Let's Make Our <br> Own .blue[River]

---

class: fifty-fifty

.left-panel[
# Data Sync
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-queue.png" width="70%"/>
]

---

class: fifty-fifty

.left-panel[
# Data Sync: Enqueue Index Tasks
]

.right-panel[
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

class: fifty-fifty

.left-panel[
# Data Sync: Index Task Worker
]

.right-panel[
<img src="/images/slides/enhancing-meteor-with-elasticsearch/search-worker.png" width="80%"/>
]

---

class: fifty-fifty

.left-panel[
# Data Sync: Index Task Worker
]

.right-panel[
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

class: center, middle

# So Data Is In Sync, <br> How Do I .elasticsearch[Search] It?

---

class: fifty-fifty

.left-panel[
# Search: Meteor Publication
]

.right-panel[
- Use .meteor[Meteor] to build a publication that accepts a query
- Let .elasticsearch[Elasticsearch] do the heavy lifting for queries
- Use scored results from .elasticsearch[Elasticsearch] to return collection data
]

---

class: fifty-fifty

.left-panel[
# Search: Meteor Publication
]

.right-panel[
```javascript
// Meteor App - Server
var Future = Npm.require('fibers/future');
Meteor.publish('search', function(q) {
  var future = new Future();
  // es client
  // https://www.npmjs.com/package/elasticsearch
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

class: fifty-fifty

.left-panel[
# Search: Meteor Subscription
]

.right-panel[
- On the client subscribe with a querystring
- Render data with .meteor[Meteor] template
]

---

class: fifty-fifty

.left-panel[
# Search: Meteor Subscription
]

.right-panel[
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

class: fifty-fifty

.left-panel[
# Elasticsearch Hosting
]

.right-panel[
- Self Hosting
- Elasticsearch on [AWS](https://github.com/elastic/elasticsearch-cloud-aws)
- Heroku [Add Ons](https://addons.heroku.com/?q=elasticsearch)
- Elasticsearch as a service
]


---

class: fifty-fifty

.left-panel[
# Elasticsearch As A Service
]

.right-panel[
- We chose to go with [Found](https://www.found.no/)
- Scale up, down and do upgrades with zero downtime
- Technical support was awesome
]

---

class: fifty-fifty

.left-panel[
# Lessons Learned
]

.right-panel[
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

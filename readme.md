## couchup

`couchup` is a database. The goal is to build a data model well suited for mobile applications that may need to work offline and sync later on and maintain smart client side caches. This data model is inspired by CouchDB but diverges greatly in the way it handles and resolves the revision history and conflicts. `couchup` implements a "most writes wins" conflict resolution scheme and does not require or even allow user specific conflict resolution.

Another goal of `couchup` is to be performant and modular. This repository only implements the base document storage layer. Indexes, attachments and replicators are implemented as additional modules.

The tradeoffs `couchup` has made in revision tree storage along with some other simple optimizations mean that `couchup` already has [better write performance than CouchDB](https://gist.github.com/mikeal/5847297) and the same consistency guarantees.

## API

```javascript
var couchup = require('couchup')
  , store = couchup('./dbdir')
  ;

db.put('databaseName', function (e, db) {
  if (e) throw e
  db.put({_id:'key', prop:'value'}, function (e, info) {
    if (e) throw e
    db.put({_id:'key', _rev:info.rev, prop:'newvalue'}, function (e, info) {
      if (e) throw e
      db.get('key', function (e, doc) {
        if (e) throw new Error('doc not found')
        console.log(doc)
      })
    })
  })
})
```

```javascript
db.compact() // remove old revisions and sequences from the database.
```

```javascript
var changes = db.changes()
changes.on('row', function (row) {
  console.log(row.seq, row.id)
})
changes.on('end' function () {
  console.log('done')
})
```

```javascript
db.info(function (e, i) {
  if (e) throw e
  console.log(i.update_seq, i.doc_count)
})
```

#### Incompatibilities w/ CouchDB

Pull replication from CouchDB works and will continue to work continuously if you aren't updating the `couchup` node you're writing it to. Bi-Directional replication with CouchDB will eventually result in conflicts on the CouchDB side because `couchup` converts CouchDB's revision tree to a linear revision sequence.

Similarly, push replication to Apache CouchDB will work once but writing again will likely cause unnecessary conflicts on the CouchDB side.


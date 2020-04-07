# API-ul bulk

Cu ajutorul acestui endpoint API poți face mai multe operațiuni de indexare și ștergere dintr-un singur apel.

```yaml
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

Acest API oferă o modalitate de a face mai multe operațiuni într-o singură cerere (`index`, `create`, `delete` și `update`).

Acțiunile `index` și `create` așteaptă datele pe următoarea linie. Pentru a preciza acțiunile și datele de operare, se folosește *newline delimited JSON* (**NDJSON**).

Operațiunea de creare - `create` a unui document eșuează dacă există un document cu același nume în index. Operațiunea `index` adaugă sau înlocuiește un document după cum este cerut.

Operațiunea `update` așteaptă ca pe următoare linie să existe opțiunile specifice actualizării parțiale a unui *doc* parțial, un *upsert* sau al unui *script*. Operațiunea *delete* nu așteaptă date pe linia următoare.

Ultima linie trebuie să fie o linie goală - `\n`. Ține minte acest detaliu foarte important. Fiecare linie din listă trebuie să se încheie cu un `\r` (un *enter*).

Atunci când trimiți la enpointul `_bulk`, headerul `Content-Type` trebuie setat la `application/x-ndjson`.

Dacă specifici numele unui index după endpoint, acesta va fi destinația tuturor operațiunilor cu excepția celor care menționează propriul index.

Răspunsul la o astfel de oprațiune este un obiect de dimensiuni mai mari, care explică ce s-a petrecut cu fiecare acțiune în parte.

În cazul în care o acțiune nu reușește, se va continua cu restul.

Dacă trimiți cereri `_bulk` folosind `curl`, ca trebui să pui fanionul `--data-binary` în locul lui `-d`, care nu păstrează `\n`-urile (*newline*).

Un alt exemplu mai elaborat de `_bulk`:

```yaml
POST _bulk
{ "update" : {"_id" : "1", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
```

Un exemplu oferit de documentația oficială

```javascript
'use strict'

require('array.prototype.flatmap').shim()
const { Client } = require('@elastic/elasticsearch')
const client = new Client({
  node: 'http://localhost:9200'
})

async function run () {
  await client.indices.create({
    index: 'tweets',
    body: {
      mappings: {
        properties: {
          id: { type: 'integer' },
          text: { type: 'text' },
          user: { type: 'keyword' },
          time: { type: 'date' }
        }
      }
    }
  }, { ignore: [400] })

  const dataset = [{
    id: 1,
    text: 'If I fall, don\'t bring me back.',
    user: 'jon',
    date: new Date()
  }, {
    id: 2,
    text: 'Witer is coming',
    user: 'ned',
    date: new Date()
  }, {
    id: 3,
    text: 'A Lannister always pays his debts.',
    user: 'tyrion',
    date: new Date()
  }, {
    id: 4,
    text: 'I am the blood of the dragon.',
    user: 'daenerys',
    date: new Date()
  }, {
    id: 5, // change this value to a string to see the bulk response with errors
    text: 'A girl is Arya Stark of Winterfell. And I\'m going home.',
    user: 'arya',
    date: new Date()
  }]

  const body = dataset.flatMap(doc => [{ index: { _index: 'tweets' } }, doc])

  const { body: bulkResponse } = await client.bulk({ refresh: true, body })

  if (bulkResponse.errors) {
    const erroredDocuments = []
    // The items array has the same order of the dataset we just indexed.
    // The presence of the `error` key indicates that the operation
    // that we did for the document has failed.
    bulkResponse.items.forEach((action, i) => {
      const operation = Object.keys(action)[0]
      if (action[operation].error) {
        erroredDocuments.push({
          // If the status is 429 it means that you can retry the document,
          // otherwise it's very likely a mapping error, and you should
          // fix the document before to try it again.
          status: action[operation].status,
          error: action[operation].error,
          operation: body[i * 2],
          document: body[i * 2 + 1]
        })
      }
    })
    console.log(erroredDocuments)
  }

  const { body: count } = await client.count({ index: 'tweets' })
  console.log(count)
}

run().catch(console.log)
```

## Resurse

- [Bulk](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.x/bulk_examples.html)

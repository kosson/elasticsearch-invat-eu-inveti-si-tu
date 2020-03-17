# Procesarea la nivel de pachet - batch

Mai întâi, se va menționa faptul că se dorește aplicarea unei operațiuni la nivelul a unui pachet de câmpuri ale unui document. Verbul folosit este `POST`, iar cuvântul cheie este `_bulk`. Sunt patru acțiuni din care putem alege:

- index
- update
- create
- delete

Exemplul de mai jos vizează acțiunea `index`, adică va fi introdus un document dacă acest anu există și va fi indexat. Dacă id-ul unui document nu este menționat, Elasticsearch va genera unul.

```yaml
POST /_bulk
{
  "index": {
      "_index":"miscelanee",
      "_id":101
  }
}
{
  "titlu":"Bizari la răscruci",
  "autor":"Bălănel Ovidiu",
  "tags":["obscur","buf"],
  "likes":0
}
```
În cazul în care documentul deja există, acesta va fi înlocuit.

În cazul folosirii lui `create`, documentul nu va fi creat dacă acesta deja există. Să completăm exemplul de mai sus și cu o acțiune `create`.

```yaml
POST /_bulk
{"index":{"_index":"miscelanee","_id":101}}
{"titlu":"Bizari la răscruci","autor":"Bălănel Ovidiu","tags":["obscur","buf"],"likes":0}
{"create":{"_index":"miscelanee","_id":102}}
{"titlu":"Gigi merge la meci","autor":"Baldovin Lulip","tags":["realist","medieval"],"likes":0}
```

Ceea ce trebuie înțeles este faptul că poți pune în succesiune mai multe acțiuni care vor fi operate. În rezultatul returnat, vom avea câte un răspuns pentru fiecare dintre acțiuni.

```json
{
  "took" : 913,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "miscelanee",
        "_type" : "_doc",
        "_id" : "101",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 13,
        "_primary_term" : 3,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : "miscelanee",
        "_type" : "_doc",
        "_id" : "102",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 14,
        "_primary_term" : 3,
        "status" : 201
      }
    }
  ]
}
```

Pentru a verifica, faceți o căutare după toate documentele din index.

```yaml
GET /miscelanee/_search
{
  "query": {
    "match_all":{}
  }
}
```

# Procesarea la nivel de pachet - batch

Această operațiune este folosită atunci când ai foarte multe acțiuni de scriere în Elasticsearch pentru a evita operațiuni multe și separate.

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

Pentru a actualiza, se va folosi acțiunea `update`.

```yaml
POST /_bulk
{"update":{"_index":"miscelanee","_id":101}}
{"doc": {"likes":1}}
{"delete": {"_index":"miscelanee","_id":102}}
```

Am strecurat și o acțiune `delete` în `_bulk`. Această acțiune n-are nevoie de o a doua linie. Răspunsul este similar cu următorul obiect JSON:

```json
{
  "took" : 50,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "miscelanee",
        "_type" : "_doc",
        "_id" : "101",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 15,
        "_primary_term" : 3,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "miscelanee",
        "_type" : "_doc",
        "_id" : "102",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 16,
        "_primary_term" : 3,
        "status" : 200
      }
    }
  ]
}
```

Dacă toate acțiunile se referă la același index, acesta poate fi meționat înaintea lui `_bulk` și apoi se pot elimina mențiunile.

```yaml
POST /miscelanee/_bulk
{"update":{"_id":101}}
{"doc": {"likes":1}}
{"delete": {"_id":102}}
```

În cazul folosirii API-ului cu HTTP, headerele pentru actualizare cu `_bulk` trebuie să fie `Content-Type: application/x-ndjson`. Este acceptat și `application/json`, dar corect este `application/x-ndjson`.

Ține minte să închei liniile cu `\n` sau `\r\n` atunci când asamblezi un query. O greșeală capitală este să nu pui chiar și după ultima linie. Dacă trimiți un obiect drept payload, nu uita să pui o linie goală la final.

Dacă o acțiune eșuează, celelalte vor continua iar cea care a eșuat va fi raportată în rezultate.

## Încărcarea bulk de date

În cazul în care se face import de date masive în Elasticsearch, nu uita ca ultima linie să fie una goală. În sistemul de operare folosit, poziționați-vă în directorul unde sunt datele. De acolo, deschizând terminalul, se vor importa datele.

```bash
curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/nume_index/_bulk --data-binary "@fisier.json"
```

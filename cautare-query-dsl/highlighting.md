# Highlighting

Sublinierea textului este oferită de Elasticseach pentru a puncta vizual rezultatele care au fost găsite.
Din oficiu este folosit un highlighter numit `plain`.

```json
POST /highlight_test/_doc/1
{
  "descriere":"Acest text ne doare pentru că pică la picioare. Acolo ne dă frig, face nasoale și iarăși pică. În picioare."
}

GET //highlight_test/_search
{
  "_source": false,
  "query": {
    "match": {
      "descriere": "Un text care ne doare"
    }
  },
  "highlight": {
    "fields": {
      "descriere": {}
    }
  }
}
```

Vom obține un rezultat similar cu:

```json
{
  "took" : 456,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.970927,
    "hits" : [
      {
        "_index" : "highlight_test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.970927,
        "highlight" : {
          "descriere" : [
            "Acest <em>text</em> <em>ne</em> <em>doare</em> pentru că pică la picioare. Acolo <em>ne</em> dă frig, face nasoale și iarăși pică."
          ]
        }
      }
    ]
  }
}
```

Dacă este necesar, poți modifica ce tag-uri sunt folosite pentru a realiza highlight-ul.

```json
POST /highlight_test/_doc/1
{
  "descriere":"Acest text ne doare pentru că pică la picioare. Acolo ne dă frig, face nasoale și iarăși pică. În picioare."
}

GET //highlight_test/_search
{
  "_source": false,
  "query": {
    "match": {
      "descriere": "Un text care ne doare"
    }
  },
  "highlight": {
    "pre_tags":["<strong>"],
    "post_tags": ["</strong>"],
    "fields": {
      "descriere": {}
    }
  }
}
```

cu un rezultat:

```json
"highlight" : {
  "descriere" : [
    "Acest <strong>text</strong> <strong>ne</strong> <strong>doare</strong> pentru că pică la picioare. Acolo <strong>ne</strong> dă frig, face nasoale și iarăși pică."
  ]
}
```

# Boolean query - `bool`

Este o interogare care caută documente prin combinarea booleană a altor interogări. Un query `bool` se va executa într-o abordare **more-matches-is-better**. Această abordare implică faptul că fiecare aplicare a unor cerințe `should` sau `must`, vor adăuga la scorul total al documentelor găsite.

Aceste interogări se construiesc folosind mai mulți termeni booleani intitulați *typed occurences*. Un posibil exemplu este următorul:

```json
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

## Occur `must`

În documentele căutate **trebuie** să se regăsească cerința definită (`"term" : { "user" : "kimchy" }`). Rezultatul influențează scorul.


## Occur `filter`

În documentele căutate **trebuie** să se regăsească cerința definită (`term" : { "tag" : "tech" }`). Nu influențează scorul. Cerințele `filter` sunt executate în **contextul filter**. Dacă sunt folosite de mai multe ori, vor intra în cache.

## Occur `should`

Cerințele trebuie să fie satisfăcute de cerințele menționate.

```json
"should" : [
  { "term" : { "tag" : "wow" } },
  { "term" : { "tag" : "elasticsearch" } }
]
```

## Occur `must_not`

Criteriile menționate nu trebuie să existe în documentele găsite. Interogările acestea se fac în context *filter*, ceea ce înseamnă că nu influiențează scorul.

## Parametrul `minimum_should_match`

Este folosit pentru a specifica numărul sau procentajul de interogări `should` care trebuie să se facă pe documente găsite după aplicarea lui `must`.
Dacă un query `bool` include cel puțin o interogare `should` dar niciun `must` sau `filter`, valoarea din oficiu este 1. Altfel, valoarea este 0.

## Acordarea unui scor cu `bool.filter`

Interogările de sub un `filter`, nu acordă niciun scor documentelor găsite. Valaorea lui `_score` fiind 0. De exemplu, următoarele interogări vor aduce toate documentele care au câmpul `status` setat la valoarea `active`. Primul acordă un scor de 0 tuturor documentelor.

```json
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

În următorul query `bool`, tuturor documentelor li se acordă valoarea `1`.

```json
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

În următorul exemplu, `constant_score` acordă o valoare de `1` tuturor documentelor precum în exemplul de mai sus.

```json
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

## Resurse

- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html#query-dsl-bool-query

# Disjunction max query

Interogarea returnează documente care au fost aduse în urma unor interogări interne (*wrapped queries*) numite *query clauses* sau *clauses*.

Dacă un document satisface mai multe cerințe (*query clauses*), query-ul `dis_max` atribuie documentului cel mai mare scor la relevanță, plus un adaos pentru fiecare cerință pe care o satisface.

```json
{
    "query": {
        "dis_max" : {
            "queries" : [
                { "term" : { "title" : "Quick pets" }},
                { "term" : { "body" : "Quick pets" }}
            ],
            "tie_breaker" : 0.7
        }
    }
}
```

## Resurse

- https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html

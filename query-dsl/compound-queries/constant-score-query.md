# Constant score query

Ambalează un *filter query*  și returnează documente care au un scor de relevanță egal cu cel menționat de valoarea parametrului `boost`.

```json
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}
```

Adu-ți aminte că `filter` queries nu calculează scor de relevanță. Pentru creșterea vitezei, Elasticsearch face cacheing.

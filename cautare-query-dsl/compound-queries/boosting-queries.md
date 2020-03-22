# Boosting queries

Returnează documente ca răspuns al limitărilor `positive` și va fi scăzută valoarea lui `_score` conform valorii `negative_boost`.

```json
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```

Folosind `boosting` pentru a scădea artificial valoarea unor documente fără a le exclude din setul celor returnate.

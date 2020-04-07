# Upserts

Upsert-urile folosesc endpointul `_update`.
Regula este simplă: dacă documentul există, scriptul va fi rulat. Dacă nu există, va fi creat.

```yaml
POST /miscelanee/_update/100
{
  "script": {
    "source": "ctx._source.likes++"
  },
  "upsert": {
    "titlu": "Moș Teacă",
    "autor": "Anton Bacalbașa",
    "tags": ["amuzant","educativ"],
    "likes": 0
  }
}
```

Acest document dat spre exemplificare, nu există, dar va fi creat la primul upsert.

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 9,
  "_primary_term" : 3
}
```

Ceea ce se petrece la momentul în care facem următorul upsert, este că valoarea like-urilor este incrementată, documentul fiind actualizat.

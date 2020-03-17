# Căutarea unui document după id

Pentru a obține un document după id-ul său, se va face un `GET /nume_idx/_doc/string_id`.

Pentru o interogare ca exemplu `GET /miscelanee/_doc/t34t5XABCQdaqr4RL8i1`, vom obține un rezultat similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "titlu" : "Undeva, cândva",
    "autor" : "Giaccomo Castelvanno"
  }
}
```

Valoarea lui `found` este `true`.

# Introducerea documentelor

Pentru a introduce un document, se va folosi verbul POST urmat de numele indexului precum în `POST /nume_index/_doc
{
  "titlu": "Undeva, cândva",
  "autor": "Giaccomo Castelvanno"
}`

Dacă indexul nu exista, s-a creat și avem un răspuns similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

# Introducerea documentelor

Pentru a introduce un document, se va folosi verbul `POST` urmat de numele indexului precum în exemplul următor.

```yaml
POST /nume_index/_doc
{
  "titlu": "Undeva, cândva",
  "autor": "Giaccomo Castelvanno"
}
```

Dacă indexul nu există, va fi creat din oficiu fiind returnat un răspuns similar cu următorul exemplu.

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

Buna practică cere crearea unui indice în prealabil.

## Impunerea unui id

Atunci când dorești să introduci un document care să poarte un anumit id, acest id va fi precizat după endpointul `_doc`, precum în următorul posibil exemplu:

```yaml
POST /test/_doc/da9d98da00da0fare0a
{
  "titlu": "Ceva fain de tot",
  "autor": "Fănel August"
}
```

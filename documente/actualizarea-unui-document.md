# Actualizarea unui document

## Actualizarea simplă

Această operațiune se face folosind verbul `POST` precum în exemplul `POST /nume_idx/_update/string_id
{
  "doc": {
    "câmp";"noua valoare"
  }
}`

Putem avea un posibil exemplu:

`POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "doc": {
    "autor": "Ginel Alexăndrel"
  }
}`

cu un răspuns similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

Dacă un câmp nu există deja în document, acesta poate fi adăugat. Este aceeași operațiune menționând un câmp nou cu valoarea sa:

`POST /miscelanee/_update/t34t5XABCQdaqr4RL8i1
{
  "doc": {
    "tags": ["trist","nemilos"]
  }
}`

Rezultatul poate fi similar cu:

```json
{
  "_index" : "miscelanee",
  "_type" : "_doc",
  "_id" : "t34t5XABCQdaqr4RL8i1",
  "_version" : 3,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "titlu" : "Undeva, cândva",
    "autor" : "Ginel Alexăndrel",
    "tags" : [
      "trist",
      "nemilos"
    ]
  }
}
```

## Actualizarea folosind un query

Să presupunem că avem cazul unui subset de documente, care dacă îndeplinesc un anumit criteriu sau dacă au fost selectate de utilizator în interfață, dorim să-și modifice o anumită valoare toate. De exemplu, să le incrementăm cu o unitate sau să le decrementăm cu o unitate.

```yaml
POST /miscelanee/_update_by_query
{
  "script": {
    "source": "ctx._source.likes--"
  },
  "query":{
    "match_all": {}
  }
}
```

În exemplul propuse, conform query-ului ales, adică `match_all`, toate documentele vor decrementa valoarea din câmpul `likes` cu o unitate.

Pentru un index cu doar două elemente am putea avea un răspuns similar cu următorul obiect:

```json
{
  "took" : 246,
  "timed_out" : false,
  "total" : 2,
  "updated" : 2,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

Ori de câte ori se face un update folosind `_update_by_query`, se face câte un snapshot al întregului index. Apoi se face o interogare a tutturor replication shards existente în grupul de replicare. Acolo unde se găsesc documentele replicate, se trimite un *bulk update* pentru a le actualiza.

Specificarea lui `"conflict":"proceed"` va conduce la finalizarea operațiunii chiar dacă sunt detectate erori în versiunile prezente în shard-uri.

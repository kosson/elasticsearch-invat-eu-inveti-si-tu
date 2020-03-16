# Actualizarea unei înregistrări prin versionare

Documentele din Elasticsearch sunt imuabile. Totuși, fiecare înregistrare din Elasticsearch are un câmp `_version`. Ceea ce se petrece atunci când *actualizezi* un document, este că documentul vechi este șters, iar cel actualizat este constituit. Comanda `_update` indică faptul că se va face o actualizare a documentului.

Elasticsearch oferă posibilitatea de a face actualizări parțiale ale unei înregistrări, care vor fi indicate prin proprietatea `"doc"`. Acesta este semnalul că se va proceda la modificarea unor câmpuri, nu a întregii înregistrări.

```bash
curl -H 'Content-Type: application/json' -XPOST 127.0.0.1:9200/movies/_doc/109487/_update -d '
{
  "doc": {
    "title": "Interstellar"
  }
}'
```

În acest moment, câmpul `_version` va fi actualizat la valoarea `2` de la `1`.

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_doc/109487?pretty
```

Cu rezultatul

```text
{
  "_index" : "movies",
  "_type" : "_doc",
  "_id" : "109487",
  "_version" : 2,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "genre" : [
      "IMAX",
      "Sci-Fi"
    ],
    "title" : "Interstellar X",
    "year" : 2014
  }
}
```

În cazul în care dorești modificarea tuturor câmpurilor documentului, se va omite introducerea acestuia în proprietatea `doc`, precum mai sus în `"doc": {}`. Verbul folosit va fi `XPUT` în loc de `XPOST`.

## Rezolvarea problemelor de concurență

Ce se întâmplă în momentul în care doi utilizatori vor să actualizeze în același moment un câmp al unui document. Acesta este un scenariu de concurență. Pentru o înregistrare pe care deja o avem, să presupunem că `"_seq_no"` are valoarea `5`.

```bash
curl -H "Content-Type: application/json" -XPUT "127.0.0.1:9200/movies/_doc/109487?if_seq_no=5&if_primary_term=1" -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'
```

După ce se face actualizarea, valoarea lui `"_seq_no"` va fi `7`

```text
{"_index":"movies","_type":"_doc","_id":"109487","_version":3,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":7,"_primary_term":1}
```

O abordare manuală prin încercarea numerelor pentru `_seq_no`, nu este eronată, dar este deschisă unei probleme legate de apariția unei erori în cazul în care, între timp numărul s-a modificat.

Poți apela la parametrul `retry_on_conflict` pus la dispoziție de Elasticsearch căruia îi atribui un număr ce reprezintă de câte ori va încerca Elasticsearch să actualizeze înregistrarea. Dacă alegem ca Elasticsearch să încerce de cinci ori să actualizeze, îi vom atribui valoarea 5 parametrului `retry_on_conflict`.

```bash
curl -H "Content-Type: application/json" -XPOST "127.0.0.1:9200/movies/_doc/109487/_update?retry_on_conflict=5" -d '
{
  "doc": {
    "title": "Interstellar NOU"
  }
}'
```

Actualizarea se face cu succes fiind marcat de un răspuns asemănător cu

```text
{"_index":"movies","_type":"_doc","_id":"109487","_version":4,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":8,"_primary_term":1}
```

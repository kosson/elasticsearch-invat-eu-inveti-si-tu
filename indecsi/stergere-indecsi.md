# Ștergere indecși

Pentru a șterge un index: `DELETE /nume_index`.

Operațiunea de ștergere a indexului este foarte directă și poate fi inițiață din eroare. Acest considerent trebuie să se adauge celor care țin de securitatea accesării Elasticsearch-ului.

```bash
curl -H 'Content-Type: application/json' -XDELETE 127.0.0.1:9200/movies
```

Vei obține un răspuns similar cu `{"acknowledged":true}`.

În cazul în care ștergerea indexului s-a făcut pentru a-l reface din temelie, va trebui să fie inițiat din nou mapping-ul, care-l va defini pe cel nou.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
    }
  }
}'
```

După ștergere, poți verifica dacă a fost șters cu:

```bash
curl -H 'Content-Type: application/json' -XGET 'localhost:9200/shakespeare/?pretty'
```

## Ștergerea unui document folosind verbul `XDELETE`

Pentru a șterge documentele este nevoie să folosești verbul DELETE pentru un anumit document.

```bash
curl -H 'Content-Type: application/json' -XDELETE 127.0.0.1:9200/movies/_doc/58559
```

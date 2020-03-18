# Import de date

## Import de date cu `_bulk`

Pentru a introduce mai multe date deodată din linia de comandă, se poate folosi opțiunea `_bulk` într-o posibilă construcție folosind `curl`.

De exemplu, pentru un film a cărui mapping deja a fost introdus, avem o posibilă comandă:

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk -d '
{"create": {"index": "movies", "_id": "135569"}}
{"id": "135569", "Star Trek Beyond", "year": 2016, "genre": ["Action", "Sci-Fi"]}
{"create": {"index": "movies", "_id": "122886"}}
{"id": "122886", "Star Wars: Episode VII - The Force Awakens", "year": 2015, "genre": ["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"]}'
```

## Import de date din fișiere json

Înainte de a importa datele, trebuie să te asiguri că ai definit o schemă de date și că această schemă a fost încărcată deja în Elasticsearch. Apoi, dacă ai toate datele într-un JSON, le poți încărca fără probleme. JSON-ul are următoarea structură:

```json
{ "create" : { "_index" : "movies", "_id" : "135569" } }
{ "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "movies", "_id" : "122886" } }
{ "id": "122886", "title" : "Star Wars: Episode VII - The Force Awakens", "year":2015 , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "109487" } }
{ "id": "109487", "title" : "Interstellar", "year":2014 , "genre":["Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "58559" } }
{ "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "1924" } }
{ "id": "1924", "title" : "Plan 9 from Outer Space", "year":1959 , "genre":["Horror", "Sci-Fi"] }
```

Pentru a încărca, rulează comanda:

```bash
curl -H 'Content-Type: application/json' -XPUT 'localhost:9200/movies/_doc/_bulk?pretty' --data-binary @movies.json
```

## Introducerea unei singure înregistrări

Pentru a introduce o singură înregistrare, aceasta poate fi introdusă direct din linie de comandă folosind `curl`.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/movie/109487 -d '
{
    "genre": ["IMAX","Sci-Fi"],
    "title": "Interstellar",
    "year": 2014
}'
```

În caz de succes, răspunsul poate avea următoarea formă.

```json
{
    "_index": "movies",
    "_type": "movie",
    "_id": "109487",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

Dacă vei interoga indexul pentru a vedea strucura sa, vei obseva că s-au mai adăugat două câmpuri la mappingul `movie` cu schema sa preexistentă.

```json
{
    "movies": {
        "aliases": {},
        "mappings": {
            "movie": {
                "properties": {
                    "genre": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "title": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "year": {
                        "type": "date"
                    }
                }
            }
        },
        "settings": {
            "index": {
                "creation_date": "1548333227547",
                "number_of_shards": "5",
                "number_of_replicas": "1",
                "uuid": "Ia692BjFSKm-CQNBPgOsbg",
                "version": {
                    "created": "6050499"
                },
                "provided_name": "movies"
            }
        }
    }
}
```

# Mapping

Mapping-ul este procesul prin care definești cum este stocat și indexat un document cu toate câmpurile sale.

Înainte de a introduce date în Elasticsearch, trebuie să ai un mapping care definește structura datelor încărcate. Poți să te gândești la mapping ca la un plan de construcție a unui index. Fiecare index are un *mapping type* care **determină cum vor fi indexate documentele**.

De cele mai multe ori, Elasticsearch va deduce ce tipuri de date folosești și va crea un mapping necesar automat.

Un mapping este o schemă a datelor (*schema definition*) pe care câmpurile unui document le va găzdui. Aceast mapping îi spune lui Elasticsearch cum să introducă datele pentru a fi indexate. Un *mapping type* este compus din:

- Meta-fields
- Fields

Meta-fields sunt câmpuri folosite pentru a atașa metadate necesare descrierii unui document în interiorul indexului. Câmpurile sunt:

- `_index`,
- `_type`,
- `_id`,
- `_source`.

Fields (*câmpuri*) sunt proprietățile documentului în sine. Fiecare câmp va înmagazina un anumit tip de date. Cele simple sunt:

- `text`,
- `keyword`,
- `date`,
- `long`,
- `double`,
- `boolean`,
- `ip`.

Apoi pot fi obiecte care suportă structura ierarhică a unui obiect JSON precum `object` și `nested`.

Sunt și tipuri de câmpuri specializate precum `geo_point`, `geo_shape` sau `completion`.

Fii foarte atent pentru că începând cu versiunea 6 a lui Elasticsearch, `curl` are nevoie să-i specifici headerul neapărat: `-H "Content-Type: application/json"`.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/numeindex -d '
{
    "mappings": {
        "numeindex": {
            "properties": {
                "year": {
                    "type":"date"
                }
            }
        }
    }
}'
```

În exemplul de mai sus, maparea propriu-zisă se face în obiectul care este valoarea lui `properties`. Trebuie indicat și faptul că pentru a introduce obiectul de mapare în Elasticsearch pentru un anumit index, se va folosi verbul `PUT`. În cazul în care aveți la îndemână Kibana, puteți introduce mapările folosind `Dev Tools`. După introducerea obiectului de mapare, o interogare cu `GET` pe endpointul `_mappings` a indexului dorit. De exemplu:

```yaml
PUT /movies/_mapping
{
  "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
  }
}
```

Pentru a observa ce mappinguri au fost făcute, se poate rula din Dev Tools (Kibana) comanda `GET /movies/_mapping`. Obiectul de răspuns poate fi similar cu:

```json
{
  "movies" : {
    "mappings" : {
      "properties" : {
        "genre" : {
          "type" : "keyword"
        },
        "id" : {
          "type" : "integer"
        },
        "title" : {
          "type" : "text",
          "analyzer" : "english"
        },
        "year" : {
          "type" : "date"
        }
      }
    }
  }
}
```

Mapping-urile se folosesc în mai multe scopuri.

## Specificarea tipurilor de date

Poți specifica tipurile de date - `text`, `keyword`, `byte`, `long`, `short`, `integer`, `float`, `double`, `boolean`, `date`.

```json
"properties": {
    "id": {
        "type": "long"
    }
}
```

## Definirea indexării câmpurilor

Dacă dorești ca un câmp să fie indexabil, poți specifica acest lucru.

```json
"properties": {
    "genre": {
        "index": "false"
    }
}
```

Dacă este `true`, acest câmp va apărea în indexul inversat (*inverted index*).

## Parametrii mapping-ului

Parametrii pot fi specificați pentru fiecare câmp indicând comportamentul Elasticsearch atunci când primește date pentru a le transforma în documente ale indexului.

### `coerce`

Acest parametru când are valoarea `true`, va *converti* valorile care seamnănă a fi numere în numere. O valoare venită ca string `"10.2"`, va fi convertită la `10.2`.

### `copy_to`

Permite crearea de câmpuri custom. Acest parametru îi specifică lui Elasticsearch să copieze o valoare într-un anumit câmp.

```json
{
  "nume": {
    "type":"text",
    "copy_to":"numecomplet"
  },
  "prenume": {
    "type":"text",
    "copy_to":"numecomplet"
  },
  "numecomplet": {
    "type":"text"
  }
}
```

### `dynamic`

Activează sau dezactivează adăugarea de câmpuri documentelor sau obiectelor imbricate în mod dinamic.

```json
{
  "mappings": {
    "default": {
      "dynamic": false,
      "properties":{
        "nume":{
          "dynamic": true,
          "properties": {}
        }
      }
    }
  }
}
```

### `norms`

Dacă `true`, va marca respectivul câmp ca fiind relevant pentru a calcula scorul de relevanță. Nu poți activa `norms` mai târziu fără a reface indexul.

### `format`

Definește formatul datelor calendaristice. Posibilele valori sunt:

- "yyyy-MM-dd",
- "epoch_millis",
- "epoch_second".

Valoarea din oficiu este "strict_date_optional_time||epoch_millis".

### `null_value`

Dacă unul din câmpuri primește o valoare `null` la venirea datelor, valoarea acestuia va fi completată cu valoarea lui `null_value`.

```json
{
  "properties": {
    "likes": {
      "type": "integer",
      "null_value":0
    }
  }
}
```

### `fields`

Acest parametru este folosit pentru a indexa câmpurile în diferite feluri.

### `Obiecte`

Pentru obiecte nu este necesară specificarea tipului obiect sau ceva asemănător. Elasticseach detectează după setări că este vorba despre configurarea unui obiect care va fi parte a documentului. Un posibil exemplu ilustrativ.

```yaml
PUT /test
{
  "mappings": {
    "properties": {
      "obi": {
        "properties": {
          "primo": {"type": "text"},
          "secundo": {
            "properties": {
              "intern": {
                "type": "boolean"
              }
            }
          }
        }
      }
    }
  }
}
```

Rularea comenzii `GET /test/_mapping` aduce ca răspuns tocmai obiectul de configurare folosit pentru a inița indexul.

```json
{
  "test" : {
    "mappings" : {
      "properties" : {
        "obi" : {
          "properties" : {
            "primo" : {
              "type" : "text"
            },
            "secundo" : {
              "properties" : {
                "intern" : {
                  "type" : "boolean"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Pentru a crea un document.

```yaml
POST /test/_doc/1
{
  "obi": {
    "primo": "ceva interesant",
    "secundo": {
      "intern": true
    }
  }
}
```

Acum ceva foarte interesant. De fiecare dată când vei adăuga în index un document nou care conține alte câmpuri decât cele care au fost declarate în mapping inițial, vor fi create mapping-uri automat pentru ele, fiind actualizat mappingul. De exemplu, să adăugăm un document nou care conține un câmp suplimentar.

```yaml
POST /test/_doc/2
{
  "batarang": {
    "aha": 10
  },
  "obi": {
    "primo": "ceva interesant",
    "secundo": {
      "intern": true
    }
  }
}
```
Rularea comenzii `GET /test/_mapping` aduce ca răspuns tocmai noul mapping al indexului.

```json
{
  "test" : {
    "mappings" : {
      "properties" : {
        "batarang" : {
          "properties" : {
            "aha" : {
              "type" : "long"
            }
          }
        },
        "obi" : {
          "properties" : {
            "primo" : {
              "type" : "text"
            },
            "secundo" : {
              "properties" : {
                "intern" : {
                  "type" : "boolean"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### `Nested`

Acesta este o versiune specializată de obiect care permite indexarea array-urilor de obiecte. Aceste obiecte care fac parte dintr-un array vor putea fi interogate independent unul de altul. Atunci când se face indexarea unui array a cărui elemente sunt array-uri, acest array este adăugat ca un câmp de tip obiect. Drept urmare, integritatea referințelor către proprietățile obiectelor va dispărea.

Pentru aceste cazuri, proprietatea care va avea drept valoare array-ul de obiecte trebuie declarată ca `nested` (vezi documentația pentru [nested fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/nested.html#nested-fields-array-objects)).

Ceea ce se va petrece atunci când sunt introduse documente este că fiecare dintre obiectele unui array vor fi indexate separat în documente ascunse. Toate aceste documente ascunse vor putea fi interogate folosind [nested queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/query-dsl-nested-query.html).

Aceste obiecte vor putea fi analizate cu agregări nested și reverse_nested. Sortarea se va face cu cea specifică pentru nested.

## Cum sunt analizate câmpurile

La momentul în care sunt inițiate căutările pe câmpuri, rezultatele returnate pot fi modelate chiar de la bun început, atunci când este constituit `mapping`-ul pentru respectivele documente. În următorul exemplu, sunt definite câmpurile prin indicarea tipurilor lor și apoi sunt aplicate constrângerile de regăsire impuse prin analizoare.

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

### Tipurile câmpurilor

#### `"type":"date"`

Este un câmp care așteaptă ca formatul introdus să fie de tip `Date`.

#### `"type":"keyword"`

Acest tip specifică faptul că la momentul căutării, va trebui să fie trimis către Elasticsearch forma exactă a cuvântului sau sintagmei a cărui câmp poartă acest tip. Atenție, este și **case-sensitive**.

#### `"type": "text"`

Acest tip este ceva mai iertător în ceea ce privește rezultatele returnate, permițând și definirea unor analizori.

## Modificarea unui mapping

Modificarea mapărilor pentru câmpurile existente pentru care s-a făcut mapare deja, nu se mai poate face. Încercarea de a face acest lucru se va solda cu eșec. Pentru a face acest lucru, ar trebui să ștergem indexul, să facem unul noi cu noi mapări și apoi să reindexăm datele.

### Analizoare

Poți defini filtrul prin care operează tokenizatorul și cum sunt operate token-urile: `standard`, `whitespace`, `simple`, `english`.

```json
"properties": {
    "description": {
        "analyzer": "english"
    }
}
```

Analizoarele permit filtrarea anumitor caractere, de exemplu să convertești din HTML encoding și invers.

## Crearea unui index cu mapping

Pentru a constitui un index repede, se poate folosi *curl* din linie de comandă.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "year": { "type":"date" }
    }
  }
}'
```

În cazul în care deja ai scris într-un fișier mappingul pentru date, poți introduce direct fișierul.

```bash
curl -H 'Content-Type: application/json' -XPUT 'localhost:9200/shakespeare' --data-binary @shakes-mapping.json
```

Pentru a verifica ceea ce s-a petrecut, poți iniția din linia de comandă o interogare pe `_mapping`.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie
```

Răspunsul este simplu `{"movies":{"mappings":{"movie":{"properties":{"year":{"type":"date"}}}}}}`. Dacă dorești o versiune a răspunsului într-o formă mai lizibilă, pui și elementul de interogare `pretty` în linia de comandă.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie?pretty
```

Răspunsul va fi formatat.

```json
{
  "movies" : {
    "mappings" : {
      "movie" : {
        "properties" : {
          "year" : {
            "type" : "date"
          }
        }
      }
    }
  }
}
```

## Introducerea datelor după mapping

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc/109487 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'
```
Răspunsul trebuie să fie ceva asemănător cu: `{"_index":"movies","_type":"_doc","_id":"109487","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}`.

Caută înregistrarea cu `curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty`.

## Vizualizarea structurii unui index după ce a fost introdus

```bash
curl -H 'Content-Type: application/json' -XGET 'localhost:9200/shakespeare/?pretty'
```

## Dynamic mapping

Dynamic mapping permite să nu definești mapping-uri, cel puțin pentru anumite câmpuri.
Atunci când adaugi documente, Elasticsearch va adăuga mapping-uri pentru câmpurile care nu le au definite explicit. Poți vedea cum s-au făcut mapping-urile investigând un index pe endpoint-ul `_mapping`.

```yaml
GET /miscelanee/_mapping
```

Vom obține un răspuns similar cu următorul obiect JSON:

```json
{
  "miscelanee" : {
    "mappings" : {
      "properties" : {
        "autor" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "likes" : {
          "type" : "long"
        },
        "tags" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "titlu" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

Câmpurile de text atunci când se face dynamic mapping au două mapări: `text` și `keyword`. Este ceea ce se numește [multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html#_multi_fields). Acest lucru constă în maparea în mai multe feluri pentru a servi mai multor scopuri. Vezi și informațiile de la parametrul `fields` al mapping-urilor.

```json
"type" : "text",
"fields" : {
  "keyword" : {
    "type" : "keyword",
    "ignore_above" : 256
  }
}
```

Acest lucru este necesar mai târziu atunci când se vor face căutările.

## Meta fields

Fiecare document inclus în Elasticsearch, are un set de metadate asociat în plus de câmpurile sale. Acest set de metadate sunt numite *meta fields*.

### `_index`

Acest *meta field* este adăugat automat, iar valoarea sa este numele indexului în care stă un document.

### `_id`

Valoarea acestui meta field este id-ul documentelor.

### `_source`

Valoarea acestui meta field este chiar obiectul JSON folosit atunci când s-a făcut indexarea.

### `_field_names`

Conține numele fiecărui câmp care conține o valoare nenulă. Este util atunci când dorim să vedem dacă un document există în baza unui criteriu/valoare existentă.

### `_routing`

Stochează valoare folosită pentru a ruta un document la un shard.

### `_version`

Elasticsearch folosește versionarea la nivel intern. Dacă ceri un document după id-ul său, acest met field va fi parte din rezultat. Valoarea este un număr întreg care se modifică pe măsură ce documentul este modificat.

### `_meta`

Acest meta field poate fi folosit pentru a stoca date custom, care nu sunt procesate de Elasticsearch. Acesta este câmpul în care poți stoca date specifice fiecărei aplicații.

## Parametrii de mapping

### `analyzer`

Doar câmpurile tip `text` permit menționarea unui [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analyzer.html). Parametrul specifică ce analizor să fie folosit pentru indexare și/sau căutare.

### `boost`

Parametrul [boost]() este folosit pentru a specifica cu cât se va ridica automat scorul de relevanță dacă se face căutarea după respectivul câmp.
Parametrul este aplicat doar pentru **term queries**.

### `coerce`

Acest parametru permite transformarea datelor la tipul care a fost menținat pentru câmp.

### `copy_to`

Acest parametru permite copierea valorilor a mai multor câmpuri într-un câmp separat (*group field*), care mai apoi să poată fi interogat ca un câmp unic. De exemplu, valorile a două câmpuri posibile `nume` și `prenume` poate fi copiate într-un al treilea numit `nume_complet`.

```yaml
PUT nume_index
{
  "mappings": {
    "properties": {
      "nume": {
        "type": "text",
        "copy_to": "nume_complet"
      },
      "prenume": {
        "type": "text",
        "copy_to": "nume_complet"
      },
      "nume_complet": {
        "type": "text"
      }
    }
  }
}
```

### `doc_values`

După ce documentele au fost indexate, acestea sunt scrise pe disc. La momentul în care se face o agregare, o sortare, sau o accesare a unei valori într-un script, motorul se va uita la datele care sunt valorile câmpurilor unui document și cu acestea va opera.

Aceste date ale documentelor constituie adevărate structuri pe disc care sunt aranjate pentru a putea fi citite foarte rapid pentru scenariile care implică sortarea sau agregarea.

Câmpurile care nu suportă crearea de doc values sunt cele `text` și `annotated_text`. Toate câmpurile care permit această setare, o vor avea activată din oficiu.

În cazul în care ești sigur că nu vei face o agregare, o sortare sau citi valorile dintr-un script, se recomandă dezactivarea pentru a salva spațiu pe disc.

### `dynamic`

Din oficiu, câmpurile pot fi adăugate dinamic într-un document.

### `eager_global_ordinals`

### `enabled`

Este pentru momentul în care dorești să stochezi date fără a le indexa.

### `fielddata`

### `fields`

### `format`

### `ignore_above`

### `ignore_malformed`

### `index_options`

### `index_phrases`

### `index_prefixes`

### `index`

### `meta`

### `normalizer`

### `norms`

### `null_value`

### `position_increment_gap`

### `properties`

### `search_analyzer`

### `similarity`

### `store`

### `term_vector`

## Resurse

- [Mapping | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping.html)
- [Removal of mapping types | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/removal-of-types.html)
- [Mapping types | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html)
- [Mapping parameters | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-params.html)

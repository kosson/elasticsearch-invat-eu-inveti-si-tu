# Mapping

Dynamic mapping permite să nu definești mapping-uri, cel puțin pentru anumite câmpuri.
Atunci când adaugi documente, Elasticsearch va adăuga mapping-uri pentru câmpurile care nu au definite explicit. Poți vedea cum s-au făcut mapping-urile investigând un index pe endpoint-ul `_mapping`.

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

Câmpurile de text au două mapări: text și keyword.

## Meta fields

Fiecare document inclus în Elasticsearch, are un set de metadate asociat în plus de câmpurile sale. Acest set de metadate sunt numite meta fields.

### `_index`

Acest meta field este adăugat automat, iar valoarea sa este numele indexului de cate aparține un document.

### `_id`

Valoarea acestui meta field este id-ul documentelor.

### `_source`

Valoarea acestui meta field este chiar obiectul JSON folosit atunci când s-a făcut indexarea.

### `_field_names`

Conține numele fiecărui câmp care conține o valoare nenulă. Este util atunci când dorim să vedem dacă un document există în baza unui criteriu/valoare existentă.

### `_routing`

Stochează valoare folosită pentru a ruta un document la un shard.

### `_version`

Elasticsearch folosește versionarea la nivel intern. Dacă ceri un document după id-ul său, acest met field va fi parte din rezultat. Valaorea este un număr întreg care se modifică pe măsură ce documentul este modificat.

### `_meta`

Acest meta field poate fi folosit pentru a stoca date custom, care nu sunt procesate de Elasticsearch. Acesta este câmpul în care poți stoca date specifice fiecărei aplciații.

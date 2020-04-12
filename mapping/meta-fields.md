# Meta-fields

Fiecare document are asociat un set de metadate. Acestea se numesc *meta-fields*. Acestea sunt împărțite după specializarea fiecăruia.

## De identificare

- `_index`, fiind indexul căruia documentul aparține.
- `_type`, reprezentând tipul de document pentru care s-a făcut mapping-ul
- `_id`, fiind chiar id-ul documentului.

## `_id`

Acesta este identificatorul documentului. Trebuie să fie unic.
Valoarea câmpului `_id` este disponibilă atunci când se procedează la următoarele query-uri: `term`, `terms`, `match`, `query_string`, `simple_query_string`.

```yaml
GET my_index/_search
{
  "query": {
    "terms": {
      "_id": [ "1", "2" ]
    }
  }
}
```

## Indicarea sursei

- `_source`, fiind obiectul în format JSON care reprezintă corpul documentului
- `_size`, fiind reprezentarea în bytes pe care pluginul [Mapper Size Plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.6/mapper-size.html#mapper-size) îl oferă.

## Caracteristice indexării

- `_field_names`, fiind toate câmpurile documentului care nu au valori nule.
- `_ignored`, fiind toate câmpurile documentului care au fost ignorate la momentul indexări datorită lui [ignore_malformed](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-malformed.html#ignore-malformed)

## Rutare

- `_routing`, fiind o valoare care indică către care shard să se facă rutarea.

## Altele

- `_meta`, fiind metadate specifice aplicațiilor

## Resurse

- [Meta-fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html#mapping-fields)

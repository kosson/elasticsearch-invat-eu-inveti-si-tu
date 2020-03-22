# Full text queries

Aceste interogări permit căutarea în [câmpurile de text analizat](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

Stringul de text va fi procesat folosindu-se același analizor folosit la momentul în care s-a făcut indexarea. Query-urile posibile sunt:

- `intervals`;
- `match`;
- `match_bool_prefix`;
- `match_phrase`;
- `match_phrase_prefix`;
- `multi_match`;
- `common` (deprecated in 7.3.0);
- `query_string`;
- `simple_query_string`;

## `match` query

Un `match` query este în subsidiar un wrapper pentru un bool query pentru că în momentul în care s-a făcut analiza pe textul de căutat folosind analizorul, termenii rezultați sunt adăugați unui bool query. Un `match` query nu e altceva decât o metodă mai ușoară.

```yaml
GET /movies/_search
{
  "query": {
    "match": "star wars"
  }
}
```

Este echivalent cu

```yaml
GET /movies/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "star"
          }
        },
        {
          "term": {
            "title": "wars"
          }
        }
      ]
    }
  }
}
```

Pentru că din oficiu, la un `match` termenii sunt căutați folosind operatorul OR. În cazul în care am specifica expres operatorul la `and`, atunci query-ul bool echivalent ar fi cu `must` în loc de `should`.

```yaml
GET /movies/_search
{
  "query": {
    "match": {
      "title": {
        "title": "star wars",
        "operator": "and"
      }
    }
  }
}
```

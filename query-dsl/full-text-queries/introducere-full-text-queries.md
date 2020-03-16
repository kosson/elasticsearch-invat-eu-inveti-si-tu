# Full text queries

Aceste interogări permit căutarea în [câmpurile de text analizat](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

Stringul de text va fi procesat folosindu-se același analizor folosit la momentul în care s-a făcut indexarea.
Query-urile posibile sunt:

- `intervals`;
- `match`;
- `match_bool_prefix`;
- `match_phrase`;
- `match_phrase_prefix`;
- `multi_match`;
- `common` (deprecated in 7.3.0);
- `query_string`;
- `simple_query_string`;

# Ștergerea documentelor

Pentru a șterge un document, se va folosi verbul `DELETE`. Ceea ce mai trebuie cunoscut este id-ul documentului.

```yaml
DELETE /miscelanee/_doc/100
```

## Delete by query

Următorul exemplu va indica modul în care putem face ștergerea documentelor în funcție de un query specificat.

```yaml
POST /miscelanee/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

Exemplul va șterge toate documentele din index pentru că aceasta este indicația lui `"match_all": {}`.

Specificarea lui `"conflict":"proceed"` va conduce la finalizarea operațiunii chiar dacă sunt detectate erori în versiunile prezente în shard-uri.

# API-ul _reindex

Copiezi documentele dintr-un index în altul. Pentru ca reindexarea să funcționeze este necesar ca meta-field-ul `_source` să fie activ pentru toate documentele din index.

La reindexare nu se copiază și setările din indexul sursă. Mapările, setarea numărul de sharduri și a replicilor, trebuie făcute anterior la momentul pregătirii copierii.

```yaml
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

## Reindexare după anumite condiții

```yaml
POST _reindex
{
  "source": {
    "index": "twitter",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

## Reindexarea unui anumit număr de documente

```yaml
POST _reindex
{
  "max_docs": 1,
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

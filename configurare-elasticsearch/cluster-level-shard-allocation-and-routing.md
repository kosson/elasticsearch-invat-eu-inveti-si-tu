# Cluster-level shard allocation and routing settings


## Probleme / erori

### Status 429 - read-only / allow delete

Urmând instrucțiunile de la https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#disk-based-shard-allocation, am reparat eroarea. Inpirat de aici: https://discuss.elastic.co/t/cluster-block-exception-reason-blocked-by-forbidden-12-index-read-only-allow-delete-api/162449/4.

```json
{
  "error": {
    "root_cause": [
      {
        "type": "cluster_block_exception",
        "reason": "index [resedus0] blocked by: [TOO_MANY_REQUESTS/12/index read-only / allow delete (api)];"
      }
    ],
    "type": "cluster_block_exception",
    "reason": "index [resedus0] blocked by: [TOO_MANY_REQUESTS/12/index read-only / allow delete (api)];"
  },
  "status": 429
}
```

Reparat cu:

```bash
curl -X PUT "localhost:9200/resedus0/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "index.blocks.read_only_allow_delete": null
}
'
```

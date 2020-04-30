# Reindexarea datelor

Pentru a putea face o reindexare fără dureri de cap, din capul locului trebuie să fi creat un index și un alias al acestuia.

Mai întâi creează un index nou cu un nume distinct de cel vechi. Noul index poate fi unul care are mapping-ul modificat.

```bash
curl -XPUT localhost:9200/my_index_v1 -H 'Content-Type: application/json' -d '
{ ... mappings ... }
'
```

Apoi fă un alias către indexul nou creat.

```bash
curl -XPOST localhost:9200/_aliases -H 'Content-Type: application/json' -d '
{
    "actions": [
        { "add": {
            "alias": "my_index",
            "index": "my_index_v1"
        }}
    ]
}
'
```

În momentul în care ai nevoie să reindexezi, pur ți simplu creezi noul index care are un mapping nou pentru început.

```bash
curl -XPUT localhost:9200/my_index_v2 -H 'Content-Type: application/json' -d '
{ ... mappings ... }
'
```

Apoi faci o reindexare a datelor de la indexul vechi, pe cel nou. Cheia operațiunii este să creezi aliasul indexului nou să fie același ca nume cu cel pe care l-ai avut pe indexul vechi. Operațiunea se poate face odată.

```bash
curl -XPOST localhost:9200/_aliases -H 'Content-Type: application/json' -d '
{
    "actions": [
        { "remove": {
            "alias": "my_index",
            "index": "my_index_v1"
        }},
        { "add": {
            "alias": "my_index",
            "index": "my_index_v2"
        }}
    ]
}
'
```

Apoi poți șterge indexul vechi.

```bash
curl -XDELETE localhost:9200/my_index_v1
```

## Resurse

- [Reindex is coming!](https://www.elastic.co/blog/reindex-is-coming)
- [Changing Mapping with Zero Downtime](https://www.elastic.co/blog/changing-mapping-with-zero-downtime)
- [6 Steps to reindex elasticsearch data](https://www.thirdrocktechkno.com/blog/6-steps-to-reindex-elasticsearch-data)

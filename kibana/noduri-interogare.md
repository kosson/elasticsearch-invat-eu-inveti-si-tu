# Interogarea nodurilor

Pentru a obține informații despre nodurile existente în cluster, se va interoga folosind comanda `GET /_cat/nodes?v`. Opțiunea `v` este introdusă pentru a fi afișate headere ce ajută la identificarea fragmentelor utile de informație. Poți primi un răspuns similar cu următorul fragment:

```text
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           50          72   7    0.97    0.90     1.19 dilm      *      compul_meu
```

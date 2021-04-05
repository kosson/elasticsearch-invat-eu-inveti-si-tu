# Reindexarea datelor

Pentru a putea face o reindexare fără dureri de cap, din capul locului trebuie creat un index și un alias al acestuia. Mai întâi creează un index nou cu un nume distinct de cel vechi. În scop ilustrativ, să presupunem că indexul vechi este `my_index_v1`, care are drept alias pe `my_index`.

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

În momentul în care ai nevoie să reindexezi, pur și simplu creezi noul index care are un mapping nou pentru început.

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

## Reindexarea indecșilor în NodeJS

### Desprinde în index nou
Cazul în care vrei să desprinzi un alt index din documentele unuia existent.

#### Resurse
https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/reindex_examples.html

### Reindexare cu mapping nou

Pornim de la premisa că mapping-ul s-a modificat pentru unu sau mai multe câmpuri și este necesară o reindexare. În cazul modificării mapping-ului este absolut necesară reindexarea.

În acest caz, reindexarea înseamnă să ștergi indexul existent, creezi unul nou folosind mapping-ul nou. Dar pentru a nu opri serverul, este necesar să existe un alias al indexului existent. Un alias se comportă ca un indicator către unu sau mai multe indexuri. Acest comportament al alias-ului permite crearea unui nou index în background. Astfel, frontendul nu va simți modificarea.

#### Pasul 1
Creează index nou cu mapping-ul modificat

#### Pasul 2
Transferă data din indexul vechi în cel nou folosind comanda `_reindex`. Slicing setat pe `auto`, iar `refresh` la `true`.

```bash
curl -X POST "localhost:9200/_reindex?slices=auto&refresh&pretty" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "nume_indice"
  },
  "dest": {
    "index": "nume_indice_v2"
  }
}
'
```

Înainte de a face orice altceva, asigură-te că ai același număr de documente în ambele indexuri.

```bash
curl -X POST "localhost:9200/nume_indice/_search?size=0&filter_path=hits.total&pretty"
curl -X POST "localhost:9200/nume_indice_v2/_search?size=0&filter_path=hits.total&pretty"
```

#### Pasul 3

Când ai confirmarea că ai același număr de documente în ambele indexuri, vei face operațiunea de indicare a noului index a fi cel care trebuie să-l indice alias-ul.

Se va proceda la o operațiune atomizată, care va face toate operațiunile în ordine.

```bash
curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add":  { "index": "nume_indice_v2", "alias": "nume_indice","is_write_index":true } },
        { "remove_index": { "index": "medium" }}
    ]
}
'
```

#### Resurse

- https://medium.com/@code_with_abhi/five-easy-steps-to-reindex-in-place-using-elasticsearch-with-zero-downtime-89236d278522
- https://www.thirdrocktechkno.com/blog/6-steps-to-reindex-elasticsearch-data/
- https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.x/reindex_examples.html

## Resurse

- [Reindex is coming!](https://www.elastic.co/blog/reindex-is-coming)
- [Changing Mapping with Zero Downtime](https://www.elastic.co/blog/changing-mapping-with-zero-downtime)
- [6 Steps to reindex elasticsearch data](https://www.thirdrocktechkno.com/blog/6-steps-to-reindex-elasticsearch-data)

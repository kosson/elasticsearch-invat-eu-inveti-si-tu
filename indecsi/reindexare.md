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

Presupunând că noul index are un *mapping* actualizat, dorim să reflectăm acest lucru prin crearea unui nou index `my_index_v2`. În momentul în care ai nevoie să reindexezi, pur și simplu creezi noul index care are un mapping-ul nou.

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

### Reindexare cu mapping nou

Pornim de la premisa că mapping-ul s-a modificat pentru unu sau mai multe câmpuri. După cum știm, modificarea mapping-ului unui index existent nu este permisă. Pentru a modifica mapping-ul fără a pierde datele, necesită o reindexare. În cazul modificării mapping-ului este absolut necesară reindexarea. Un rol central îl joacă alias-ul. Fiecare alias se referă la o anumită versiune a indexului de la un moment dat.

În acest caz, reindexarea înseamnă să ștergi indexul existent, creezi unul nou folosind mapping-ul nou. Pentru a muta datele în indexul nou, se va folosi API-ul de reindexare.

Dar pentru a nu opri serverul, este necesar să existe un alias al indexului existent. Un alias se comportă ca un indicator către unu sau mai multe indexuri. Acest comportament al alias-ului permite crearea unui nou index în background. Astfel, frontend-ul nu va simți modificarea.

#### Pasul 1

Creează index nou cu mapping-ul modificat.

#### Pasul 2

Transferă datele din indexul vechi în cel nou folosind comanda `_reindex`. Slicing setat pe `auto`, iar `refresh` la `true`.

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

## Implementare Node.js

```javascript
let pscript =  '';
exports.reidxincr = function reidxincr (data, socket) {
    // Reindexarea se va face ori de câte ori este modificat mapping-ul (vezi variabila `resursaRedES7`).
    // În funcție de modificarea mapping-ului trebuie adaptat scriptul painless care să opereze modificările de structură (vezi variabila `pscript`).

    let idx = data.alsr + data.vs,  // Formula este `alsr` + `vs` = numele indexului.
        nvs = '';                   // noua versiune

    // promisiune de verificare alias pentru cazul în care nu ar fi alias deja
    esClient.indices.existsAlias({name: data.alsr})
        .then((r) => {
            console.log(`Alias-ul ${data.alsr} există? R: `, r.body, ", iar indexul ar trebui să fie: ", idx);

            if (r.statusCode == 200) {
                // dacă aliasul există, procedează la reindexare
                // dacă este un alias care se termină cu un număr, atunci înseamnă că e un install vechi. Șterge indexul, sterge alias-ul. Reindexează de la 0.
                // https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html#indices-aliases-api-rename-alias-ex
                /*
                    #1 Adaugă un index nou
                    #2 reindexează înregistrările pe noul index
                        - https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#_reindex
                        - https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.x/reindex_examples.html
                    #3 Leagă indexul nou de alias-ul existent
                    #4 Șterge indexul vechi
                */


                // verifică cărui index aparține alias-ul.
                esClient.cat.aliases({
                    name: data.alsr,
                    format: "json"
                }, (err, r) => {
                    if (err) {
                        console.log("De la aliases", err);
                        logger.error(err);
                    };

                    console.log(`Alias-ul ${data.alsr} aparține indexului: `, r.body[0].index);

                    // în cazul în care indexul primit este egal cu cel verificat pentru alias creează noul index
                    if (idx === r.body[0].index) {

                        // incrementează versiunea
                        let nrvs = parseInt(data.vs);
                        let newvs = ++nrvs;
                        nvs = data.alsr + newvs; // `nvd` e prescurtare de la `new version`
                        console.log("Noul nume al indexului este: ", nvs);

                        // CREEAZĂ INDEXUL nou pasând la index, numele noului index, iar la body, mapping-ul modificat (variabila `resursaRedES7`) al indexului.
                        esClient.indices.create({
                            index: nvs,
                            body: resursaRedES7
                        }).then(async (r) => {
                            console.log("Am creat indexul nou cu următorul detaliu: ", r.body);

                            // REINDEXEAZĂ
                            let body4reindex = {
                                source: {
                                    index: idx
                                },
                                dest: {
                                    index: nvs
                                }
                                // ,
                                // script: {
                                //     lang: 'painless',
                                //     source: pscript
                                // }
                            };
                            await esClient.reindex({
                                waitForCompletion: true,
                                refresh: true,
                                body: body4reindex
                            });

                            // ATAȘEZ alias-ul noului index creat
                            // https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html#_indices_putalias
                            await esClient.indices.putAlias({
                                index: nvs,
                                name: data.alsr
                            });

                            // ȘTERGE indexul vechi
                            esClient.indices.delete({
                                index: idx
                            }, (error, r) => {
                                if (error) {
                                    logger.error(error);
                                    console.log("Când să șterg indicele, am avut o eroare");
                                };
                                console.log("Am șters indexul ", idx, " vechi cu următoarele detalii: ", r.statusCode);

                                // _TODO: trimite datele noului index
                                socket.emit('es7reidx', {newidx: nvs, oldidx: idx, deleted: r.body.acknowledged}); // trimit clientului datele
                            });

                        }).catch((err) => {
                            if (err) {
                                console.log("[es7-helper] Am eșuat crearea noului index cu următoarele detalii: ", err)
                                logger.error(err);
                            };
                        });
                    }
                });
            } else {
                // dacă nu există, emite un mesaj de atenționare că ar trebui indexat de la 0. Vezi `reidxfrom0`
                console.log('Nu există alias-ul pentru care să se reindexeze!!!');
            }
        }).catch((err) => {
            logger.log(err);
            console.log('La reindexarea incrementală a apărut următoarea eroare: ', err.message);
        });
};
```

## Resurse

- [Reindex is coming!](https://www.elastic.co/blog/reindex-is-coming)
- [Changing Mapping with Zero Downtime](https://www.elastic.co/blog/changing-mapping-with-zero-downtime)
- [6 Steps to reindex elasticsearch data](https://www.thirdrocktechkno.com/blog/6-steps-to-reindex-elasticsearch-data)
- [How to Change Elastic Search Index Mapping Without Losing Data](https://hamidmosalla.com/2021/01/30/how-to-change-elastic-search-index-mapping-without-losing-data/)
- https://medium.com/@code_with_abhi/five-easy-steps-to-reindex-in-place-using-elasticsearch-with-zero-downtime-89236d278522
- https://www.thirdrocktechkno.com/blog/6-steps-to-reindex-elasticsearch-data/
- https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.x/reindex_examples.html

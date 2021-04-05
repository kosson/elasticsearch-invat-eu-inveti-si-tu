# Obținerea de informații despre indecși

Majoritatea endpoint-urilor API-urilor Elasticsearch care implică folosirea indecșilor ca parametri, permit menționarea mai multor indecși pe care să aplice operațiunea menționată de endpoint. API-ul dedicat [documentelor](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html) și cel al [alias-urilor](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html) indecșilor, nu permit folosirea mai multor indecși.

Numele indecșilor vor fi menționați cu virgule între ei sau va fi folosit endpoint-ul `_all`. Este permisă și folosire wildcard-urilor în orice poziție din numele unui index (`test*` sau `*test` sau `te*t` sau `*test*`).

Folosind operatorul `-` (*minus*) ai posibilitatea de a preciza care index să fie exclus. Să presupunem că vrem toți indecșii care încep cu `test`, precizând `test*`, dar nu dorim indexul `test4`. În acest caz, poți preciza o expresie similară cu `test*,-test4`.

## Endpointul `_cat/indices`

Pentru a obține un set de informații structurate privind indecșii, se poate lansa o interogare care să precizeze modul de afișare a detaliilor: `GET /_cat/indices?bytes=b&s=store.size:desc&v`.
O astfel de interogare, va afișa și indecșii specifici monitorizării Elastisearch (`.monitoring-es-7-2020.04.10`), dar și cei ai lui Kibana `.kibana_3`.

Pentru a obține o listă a indecșilor creați de aplicațiile proprii, vom folosi o expresie precum `GET _cat/indices/*,-.*?v`, care va folosi operatori precum minus și wildcard-ul. Astfel, se va realiza o adevărată filtrare a celor care sunt specifici ELK.

## Endpointul `_cat/count/nume_indice`

Dacă dorești să afli numărul documentelor indexate, vei folosi endpint-ul `_cat/count/nume_indice`, iar pentru o versiune afișată cu headere la tabel adaugi `?v&pretty`.

O variantă de a obține numărul documentelor este și prin folosirea endpoint-ului `_search`:
`GET /resedus/_search?size=0&filter_path=hits.total&pretty`

## Resurse

- [Multiple indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-index.html#multi-index)

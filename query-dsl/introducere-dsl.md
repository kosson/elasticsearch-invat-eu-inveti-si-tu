# Query DSL (Domain Specific Language)

Limbajul de interogare folosit de Elasticsearch se bazează pe JSON. Endopointul folosit pentru a se putea performa înterogări, se numește `_search`.

Pentru a elabora interogarea, se va pasa endpoint-ului un obiect cu o cheie `query`.

De la bun început, Elasticsearch face sortarea rezultatelor căutărilor în baza unui **relevance score**.
Scorul de relevanță este un număr cu virgulă mobilă, care este prezent în meta-field-ul `_score`. Cu cât este mai mare valoarea lui `_score`, cu atât este documentul mai relevant.

Fiecare interogare poate calcula diferit valoarea lui `_score`, dar valoarea este influențată de contextul în care este făcută căutarea: *query* sau *filter*.

Există două tipuri de interogări:

- *Leaf query clauses*
- *Compound query clauses*

Aceste interogări îți modifică comportamentul în funcție de **contextul** în care sunt folosite:

- *query context*
- *filter context*.

## Leaf query clauses

Sunt interogări care caută o anumită valoare într-un anumit câmp.
Interogările posibile sunt:

- `match`,
- `term`,
- `range`.

## Compound query clauses

Aceste interogări ambalează alte căutări *leaf* sau *compound* și sunt folosite pentru a combina mai multe interogări într-un mod logic.

## Contextul de căutare

### Query context

În acest context, interogarea răspunde la întrebarea „Cât de bine se potrivește acest document la cerințele interogării?”. Acest context influiențează calculul relevanței în meta-câmpul `_score`.

### Filter context

În acest context, interogarea răspunde dacă se potrivește strict vreun rezultat la cerințele interogării. Răspunsul este da sau nu. Nu este calculat scorul de relevanță. Acest context este folosit pentru a filtra date structurate. De exemplu: este câmpul „timestamp” între 2010 și 2011? sau este câmpul „status” setat la valoarea „published”?

Filtrările folosite frecvent sunt cache-uite de Elasticsearch.

O filtrare este pasată unui parametru `filter`.

## Exemplu de contexte `query` și `filter`

Parametrul `query` indică contextul de interogare.
Termenul `bool` urmat de cele două `must`-uri, este folosit într-un context query, însemnând că este folosit pentru a căuta cât de bine se potrivesc rezultatele.
Termenul `filter` indică un context de filtrare iar condițiile sunt menționate prin `term` și `range`. Filtrarea va elimina toate documentele care nu întrunesc cerințele, dar nu va modifica scorul documentelor care se potrivesc.

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

Condițiile care trebuie îndeplinite conform condițiilor următoare:

- Câmpul `title` conține termenul `search`;
- Câmpul `content` conține termenul `elasticsearch`;
- Câmpul `status` conține fix termenul `published`;
- Câmpul `publish_date` conține o dată începând cu 1 Ianuarie 2015.

## Resurse

- [Deep Dive into Querying Elasticsearch. Filter vs Query. Full-text search](https://towardsdatascience.com/deep-dive-into-querying-elasticsearch-filter-vs-query-full-text-search-b861b06bd4c0)
- [Text Classification made easy with Elasticsearch](https://www.elastic.co/blog/text-classification-made-easy-with-elasticsearch)

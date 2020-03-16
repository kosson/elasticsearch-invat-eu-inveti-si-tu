# Căutare în Elasticsearch

Cel mai important aspect al căutărilor pe care le poți face pe documentele din Elasticsearch este acela că rezultatele vor fi aduse înapoi în ordinea relevanței lor. Relevanța este un număr calculat de Elasticsearch.

## Căutare prin obiect

### Căutare după un câmp

Poți face o căutare după id, dacă-l cunoști.

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_doc/109487?pretty
```

### Căutare după un termen

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_search?q=Interstellar
```

### Analizers and tokenizers

Căutarea pe texte folosind analizoarele, va returna rezultatele care se aseamănă cu ceea ce se caută. În funcție de setarea analizorului, rezultatele pot fi *case-insensitives*, *stemmed*, se pot elimina semnele de punctuație (*stop words*), se pot folosi sinonimele, etc. Faptul că se face o căutare cu mai multe cuvinte, nu înseamnă că trebuie să fie găsite toate într-un document pentru ca acesta să fie adus. Se vor folosi *text types* pentru respectivele câmpuri. Poți configura diferite analizoare pentru fiecare câmp în parte.

În unele cazuri, căutarea trebuie să fie precisă. Doar o anumită combinație de cuvinte trebuie să fie găsită pentru a returna un document și exact combinația de majuscule. Se va folosi un *keyword type* atunci când definești un index. Folosirea acestuia va dezactiva analizoarele pentru acel câmp.

#### Analizoare

##### Înterogare folosind `match`.

Acest tip de analizor se va folosi în cazul în care dorești să aduci toate documentele care se potrivesc cu stringul pe care-l pasezi pentru un anumit câmp.

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
  "query": {
    "match": {
      "title": "Star Trek"
    }
  }
}'
```

Răspunsul la include rezultate ce conțin cuvântul `Star`. Ceea ce este imporant de remarcat este scorul. Pentru Star Treck, acesta este  `2.5194323`, iar pentru Star Wars este `0.66992384`. Deci primele rezultate în cazul folosirii analizoarelor sunt cele mai importante, chiar dacă setul de date returnat conține și documentele în care sunt doar câteva dintre cuvintele căutate.

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.5194323,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "135569",
        "_score" : 2.5194323,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "122886",
        "_score" : 0.66992384,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

##### Interogare folosind `match_phrase`

Este un analizor care oferă toate rezultatele care conțin fragmentul de text pasat pentru un anumit câmp pe care documentele îl au. Rezultatele vor fi alese în ordinea cuvintelor pasate. Acest lucru este posibil pentru că în indexul inversat, nu numai că se face o potrivire a termenilor dintr-un text, dar este memorată și poziția acestor termeni în text.

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
  "query": {
    "match_phrase": {
      "title": "star wars"
    }
  }
}'
```

În cazul în care poziția termenului nu este importantă, folosind `slop`, vei menționa cu cât poate fi depărtat un termen față de anteriorul într-un text, indiferent de direcție.

```bash
curl -H 'Content-Type: application/json' -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
  "query": {
    "match_phrase": {
      "title": {"query": {"star beyond", "slop": 1}
     }
    }
  }
}'
```

În exemplul anterior, va fi adus un document care are titlul `star trek beyond` pentru că are deplasarea de un termen față de primul menționat în query. Va mai aduce și documentele care au titlul `beyond star` pentru că `slop` permite deplasarea și în stânga. Putem traduce o astfel de căutare, ca o căutare în proximitate (*proximity query*).

Dacă `slop` ar avea o valoare foarte mare, să spunem de 50, căutarea va returna toate rezultatele care au cei doi termeni la o distanță de alți 50 între ei.

## Căutare prin link

Termenii de căutare pot fi introduși într-un query string în link. O astfel de căutare se numește `uri search` (vezi și documentația Elasticsearch).

```text
http://127.0.0.1:9200/movies/_search?q=title:star
http://127.0.0.1:9200/movies/_search?q=+year:>2010+title:trek
```

Aceste interogări se pot face în browser și este returnat un obiect JSON. Problema este legată de necesitatea de a coda URL-ul pentru a trimite datele de interogare pe net.

Nu se va folosi această posibilitate în mediile de producție. Din punct de vedere al securități este un punct foarte sensibil prin care s-ar putea iniția activități ce pot suprasolicita serverul.

## Căutare cu filtre

Interogările există în blocuri `"query"`, iar filtrele în blocuri `"filter"`.
Poți introduce filtre în query-uri, dar dacă este necesar, poți introduce și query-uri în filtre.

Atunci când dorești să combini mai multe filtre, se va folosi un block `"bool"` într-unul `"query"`.

### Un exemplu de căutare booleană

Căutarea booleană permite combinarea mai multor filtre folosind proprietatea `bool`. În exemplul prezentat, rezultatele vor fi aduse, dacă în document sunt găsite cele care au textul `trek` în titlu și care au fost lansate de la începutul anului 2010.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
  "query": {
    "bool": {
      "must"  : {"term" : {"title": "trek"}},
      "filter": {"range": {"year" : {"gte": 2010}}}
    }
  }
}'
```

În exemplul, se contruiește un `"query"` care **trebuie** să conțină **termenul** (`"term"`) trek, iar din documentele care respectă această cerință strictă se va face o **filtrare**, care să aducă doar un subset (`"range"`) de documente care au la câmpul `"year"` o valoarea mai mare sau egală (`"gte"` - greatter than equal) cu 2010.

### Tipuri de filtre

#### `term`

Este folosit pentru a face filtrări foarte exacte folosind un singur termen.

```json
"term": {"year": 2014}
```

#### `terms`

Filtrul este folosite pentru a face filtrări supă o listă de termeni menționați

```json
"terms": {"genre": ["Sci-Fi","Adventure"]}
```

#### `range`

Acest filtru este folosit pentru a căuta documente într-un interval menționat de câțiva specificatori (`gt`, `gte`, `lt`, `lte`)

```json
"range": {"year": ["gte", 2010]}
```

#### `exists`

Caută doar documentele în care un câmp specificat există.

```json
{"exists": {"field": "tags"}}
```

#### `missing`

Caută documentele din care lipsește câmpul specificat în query.

```json
{"missing": {"field": "tags"}}
```

### Tipuri de query-uri

#### `bool`

Acest filtru este cel care permite combinarea filtrelor folosindu-se logica booleană. Poate avea următorii specificatori:

- `must`,
- `must_not`,
- `should`.

#### `match`

Caută în rezultate care sunt generate de analizori, așa cum ar fi cazul unei căutări în text (*full text search*).

```json
{"match": {"title": "star"}}
```

#### `match_all`

```json
{"match_all": {}}
```

#### `multi_match`

Fă aceeași căutare pe mai multe câmpuri.

```json
{"multi_match": {"query": "star", "fields": ["title", "synopsis"]}}
```

## Paginarea rezultatelor

Pentru a obține rezultate care să poată fi paginate, ceea ce trebuie făcut este să fie specificat în query `from` și `size`. Numărătoarea începe de la `0`.

```bash
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "from": 0,
  "size": 3,
  "query": {
    "match": {
      "genre": "Sci-Fi"
    }
  }
}'
```

Buna practică spune ca paginarea să se facă pe seturi mici de date. Altfel, Elasticsearch va trebui să le aduca pe toate, să le sorteze și să ofere segmentul dorit în intervalul specificat. Deci, seturi mici.

## Sortarea rezultatelor

Pentru a sorta rezultatele atunci când documentele sunt căutate după un anumit câmp, vei avea nevoie de unul care să permită acest lucru. Câmpurile care permit sortarea sunt cele care nu sunt analizate.

Una din soluții ar fi ca să faci o dublură a câmpului care este supus analizorilor și să-l declari ca fiind `raw`, iar tipul acestuia să fie `keyword`. Aceste măsuri de prevedere se fac la momentul în care este constituit mapping-ul. Fii foarte atent la faptul că nu poți schimba mapping-ului unui index existent. Deci, va trebui gândită această eventualitate de la bun început.

```bash
curl -H "Content:Type: application/json" -XPUT '127.0.0.1:9200/movies/' -d '
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}'
```

## Potriviri aproximative

Este metoda prin care sunt găsite documente chiar dacă au fost întâmpinate erori de redactare. Mecanismul se numește *distanță de editare levenshtein* prin care se fac substituiri de caractere, inserarea unora sau chiar ștergerea acestora.

```bash
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "fuzzy": {
      "title": {"value": "intersteller", "fuzziness": 1}
    }
  }
}'
```

## Căutare cu prefixuri

În caz că dorești să aduci rezultate la filtrarea după un termen care începe cu o anumită valoare, poți folosi `prefix` în query.

```bash
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "prefix": {
      "year": "201"
    }
  }
}'
```

## Căutare cu wildcard

Uneori ai nevoie să substitui o porțiune din termenul căutat. Acest lucru se face cu un wildcard marcat printr-o steluță.

```bash
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "wildcard": {
      "title": "Inter*"
    }
  }
}'
```

## index-time cu n-grams

Să pornim cu un termen pentru care să introducem câteva n-gram-uri. Pentru `start`, am putea avea următoarele n-gram-uri:

- *unigram*: [s,t,a,r]
- *bigram*:  [st, ta, ar]
- *trigram*: [sta, tar]
- *4-gram*:  [star]

Mai există așa-numitele `edge n-grams`, care sunt construite doar pentru caracterele cu care începe un cuvânt. De exemplu, pentru a construi n-gram-ul pentru termenul `star`, voi avea următoarele posibile:

- *unigram*: [s]
- *bigram*:  [st]
- *trigram*: [sta]
- *4-gram*:  [star]

Pentru a folosi acest instrument, mai întâi trebuie să creezi propriul analizor. Să creăm un n-gram de minim un caracter și un maximum de 20. Acest analizor trebuie introdus înainte de mapping pentru că mapping-ul trebuie să facă referință la acesta.

```bash
curl -H "Content-Type: application/json" -XPUT '127.0.0.1:9200/movies?pretty' -d '
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer":"standard",
          "filter": ["lowercase","autocomplete_filter"]
        }
      }
    }
  }
}'
```

Urmează mapping-ul.

```bash
curl -H "Content-Type: application/json" -XPUT '127.0.0.1:9200/movies/_mapping?pretty' -d '
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "autocomplete"
    }
  }
}'
```

În acest moment poți folosi n-gram-urile pe index.

```bash
curl -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": {
        "query":"sta",
        "analyzer";"standard"
      }
    }
  }
}'
```

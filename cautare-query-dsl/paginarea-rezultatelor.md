# Paginarea rezultatelor

## Căutarea aproape real-time

În momentul în care un document este introdus, este indexat și poate fi regăsit *aproape* în timp real, cam într-o secundă. Elasticsearch are la bază bibliotecile de cod Lucene care sunt scrise în Java. Lucene a introdus conceptul de căutare la nivel de segment. Un **segment** este similar unui index inversat, dar pentru Lucene *index* înseamnă *o colecție de segmente plus un punct de commit*. După ce se face un commit, este adăugat un nou segment la *commit point* iar bufferul este curățat.

Între Elasticsearch și HDD există un cache al sistemului de fișiere. Documentele care stau în memorie sunt scrise într-un nou segment, care mai întâi este scris în cache și mai apoi pe HDD. Commitul se petrece când datele sunt scrise pe disc. Chiar dacă documentele sunt scrise în cache, acestea pot fi deschise fără nicio problemă. Lucene permite ca noi segmente să fie scrise și citite ceea ce permite și căutarea fără să facă un *full commit*.

În Elasticsearch, procesul de scriere și citire a unui nou segment se numește *refresh*. O etapă de refresh îndeplinește toate operațiunile pe un index în timp ce anteriorul refresh este gata să fie căutat. Refresh-ul poate fi controlat prin pur și simplu așteptarea ca acesta să se petreacă, setarea opțiunii de `?refresh` într-o cerere sau cererea explicită prin [API-ul](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-refresh.html) dedicat ca această etapă să fie încheiată. Elasticsearch face un refresh odată la o secundă, dar doar pe indexurile care au primit o cerere de căutare sau mai multe în ultimele 30 de secunde.

Acesta este motivul pentru care se consideră că Elasticsearch permite căutarea aproape în timp real.

Am menționat aceste detalii pentru că în cazul unei căutări în condițiile în care se dorește o paginare, ordinea rezultatelor se poate modifica. Pentru a evita acest comportament, se poate crea un PIT (point in time).

Tehnici de căutare oferite de Elastisearch:

- paginare from size,
- paginare search_after
- scroll

## Paginare cu from și size

Dacă o căutare nu este parametrizată privind rezultatele, Elastisearch va aduce primele 10. În cazul în care se dorește ceva mai mult, se va apela la una din tehnicile de căutare pe care le vom explora în acest material. Tehnica cea mai la îndemână ar fi cea care implică paginarea implicând parametrii `from` și `size`. În acest caz, limitarea de căutare în adâncime este de 10000 de documente.

```javascript
// Prima cerere
{
  "query": {
	"match": {
  	"name": "Doina moldovenească"
	}
  },
  "size": 10,
  "from": 0
}
// A doua cerere
GET products/_search
{
  "query": {
	"match": {
  	"name": "Doina moldovenească"
	}
  },
  "size": 10,
  "from": 10
}
```

Această tehnică este utilă atunci când dorești să faci paginare și să permiți utilizatorului să sară de la pagina 2 la pagina 7, de exemplu. În cazul în care dorești o implementare de tip scroll infinit, mai potrivită este o căutare folosid parametrul `search_after`.

Pentru a realiza un serviciu coerent de paginare mai întâi de toate avem nevoie să calculăm care este numărul total de pagini care sunt disponibile. Limitarea Elasticsearch-ului este de 10.000 de rezultate gestionate astfel. Reține faptul că fiecare cerere este una *stateless*.

Atunci când lucrăm cu Elasticsearch nu avem la dispoziție vreun cursor așa cum este cazul bazelor de date relaționale. Încă o precizare este că toate calculele se fac pe baza unor numere stabilite conform unui set la un moment dat. Dacă au fost adăugate sau șterse documente între timp, acest lucru se va reflecta în rezultatele de paginare. În bazele de date relaționale setul pentru care s-a stabilit un cursor rămâne stabil până în momentul în care nu mai este folosit. Nu este cazul Elasticsearch.

Paginarea se face folosind parametrii `from` și `size`. Parametrul `from` indică numărul de *hits* peste care dorim să facem un salt. Valoarea din oficiu este `0`.

Pentru a oferi Elasticsearch numerele necesare, la nivel de aplicație trebuie făcute niște calcule:

- Formula de calcul este **toate_paginile = ceil(total_hits/dimensiunea_paginii)**.
- Parametrul `from`, adică offset-ul se calculează după următoarea formulă: **from = (dimensiunea_paginii * (numărul_paginii_prezente - 1))**.

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

Buna practică spune ca paginarea să se facă pe seturi mici de date. Altfel, Elasticsearch va trebui să le aduca pe toate, să le sorteze și să ofere segmentul dorit în intervalul specificat. Deci, seturi mici în menționarea limitelor `from` și `size`.

Paginarea în mare adâncime, poate avea un impact asupra performanței. Fiecare rezultat trebuie adus de la server. Pentru a evita aceste probleme, setează o limită de rezultate maxime care poate fi adusă utilizatorului.

Această tehnică de căutare este *stateless*, adică nu este garantată ordinea rezultatelor atunci când se navighează de la o pagină de rezultate la alta și înapoi. Pot apărea rezultate noi sau dispărea în funcție de dinamica indexului.

## Căutare search_after

Poți folosi parametrul `search_after` pentru a obține următoarea pagină de hituri folosind un set de valori de sortare care a fost trimis odată cu rezultatelele inițiale de căutare. Folosind această tehnică, poți să-i transmiți lui Elasticsearch ultimul hit pe care l-ai văzut pentru ca acesta să ignore toate celelalte anterioare. Căutarea folosind `search_after` va utiliza un *tiebreaker* (informație necesară stabilirii limitei de la care pornește următorul set de hituri ce va fi adus). Un tiebreaker se comportă ca un semn de carte.

Setului de date pe care se va face căutare i se poate trimite parametri de sortare după anumite câmpuri. Fiecare sortare poate fi inversată. Sortarea este definită la nivel de câmp. De exemplu, pentru următorul mapping:

```json
{
  "mappings": {
    "properties": {
      "post_date": { "type": "date" },
      "user": {
        "type": "keyword"
      },
      "name": {
        "type": "keyword"
      },
      "age": { "type": "integer" }
    }
  }
}
```

Am putea face o sortare similară cu următorul exemplu de mai jos.

```json
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

Folosirea lui `search_after` necesită multiple apeluri de căutare folosind același query și valori de sortare. În cazul în care apare un refresh între aceste apeluri, ordinea rezultatelor se poate modifica, fapt care conduce la apariția de rezultate fără consistență între pagini. Pentru a evita acest lucru, se poate crea ceea ce se numește *point in time* ([PIT](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/point-in-time-api.html)), cu rolul de a conserva starea curentă a indexului între căutări. Această tehnică de căutare este *stateless*, adică nu este garantată ordinea rezultatelor atunci când se navighează de la o pagină de rezultate la alta și înapoi.

### Adăugarea unui point in time

Adăugarea unui PIT, transformă o căutare stateless într-una stateful.

Un PIT este constituit din cele mai recente date vizibile ale unui index. Datele sunt într-o continuă modificare. PIT-ul este o privire asupra datelor la momentul în care este constituit. A fost introdus odată cu versiunea 7.10. O căutare în Elastisearch este o căutare aplicată pe datele de la un anumit moment în timp. În contextul în care indexul se modifică în continuu, două interogări identice făcute la două momente diferite vor aduce rezultate diferite pentru că datele s-au modificat între timp. PIT-ul oferă un mecanism care elimină factorul variabilității în timp. PIT-ul permite interogarea repetată a unui index așa cum era acesta într-un anumit moment în timp.

În cazul în care se petrece un *refresh* între două căutări `search_after` rezultatele obținute pot conține modificări. Acest lucru se întâmplă pentru că modificările apărute între căutări sunt vizibile doar celor mai recente PIT-uri. Ceea ce trebuie să faci mai întâi, este să creezi in point-in-time, care va constitui întreg contextul necesar căutărilor multiple viitoare. Ceea ce va obține clientul în urma inițierii PIT-ului este un `id`, care va trebui să-l atașeze tuturor cererilor ulterioare.

```bash
POST /my-index-000001/_pit?keep_alive=1m
```

Poți folosi varianta API-ul JavaScript, precum în exemplul de mai jos.

```javascript
let pit = await esClient.openPointInTime({index: 'nume_index', keep_alive: '1m'});
```

[API-ul](https://www.elastic.co/guide/en/elasticsearch/reference/current/point-in-time-api.html) va returna un `id` al PIT-ului creat. Un PIT trebuie deschis explicit înainte de a fi utilizat în cererile de căutare. Parametrul `keep_alive` menționează care va fi durata de viață a PIT-ului și ulterior în cererile de căutare, va prelungi durata de viață a unui PIT existent sau va menționa durata de viață a celui nou format. Valorile de [timp](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/common-options.html#time-units) menționate nu trebuie să fie mari, ci cât ar fi necesar până la următoarea cerere de căutare.

În momentul în care perioada de timp menționată de `keep_alive` a expirat, PIT-ul este închis automat.

Într-un articol recent, din iunie 2021, este recomandată folosirea PIT-urilor și abandonarea practicii care implica API-ul de scolling. Articolul menționează faptul că la nivel înalt comportamentul unei căutări ce folosește PIT se aseamănă cu cel al API-ului scroll. Lucrurile sunt totuși diferite. Un scoll îți oferă datele ca instantaneu ce poate fi consuma într-o operațiune, pe când PIT-ul oferă date ce pot fi interogate mai departe.

Când se creează un PIT, ceea ce se petrece în backend este o operațiune pe shard-urile (primary și replica) unui index unde se află datele care sunt solicitate pentru căutare.

Lucrurile devin problematice atunci când sunt sute sau mii de cereri. Pentru fiecare va trebui creat un context de căutare și acest lucru va taxa resursele serverului. Dacă ai căutări intense pe un index care se schimbă continuu, probabil că nu este o idee prea bună să creezi câte un PIT pentru fiecare cerere pentru că vor fi create și menținute în viață un număr considerabil de resurse. Soluția este crearea unui proces în aplicație care să creeze câte un PIT la câteva minute, iar acest PIT să fie folosit pentru toate căutările. O tehnică îmbunătățită ar fi combinarea cu `slice`.

Pentru a obține prima pagină cu rezultate, trimite o cerere de căutare cu un argument `sort`. În cazul în care folosești PIT, specifică-l în `pit.id` dar NU menționa *target data stream*-ul sau numele indexul. Un exemplu este mai jos.

```json
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
    "keep_alive": "1m"
  },
  "sort": [
    {
      "@timestamp": {
        "order": "asc",
        "format": "strict_date_optional_time_nanos",
        "numeric_type" : "date_nanos"
      }
    }
  ]
}
```

Parametrul `keep_alive` din cererea de căutare îi va spune lui Elasticseach cu cât trebuie să extindă durata de viață a PIT-ului. Cererea pentru deschiderea PIT-ului aduce un id, dar cererile următoare care poartă și căutarea pot returna id-uri diferite. Folosește-l pe cel mai recent id pentru următoarea cerere de căutare.

Toate cererile de căutare care folosesc PIT adaugă automat un câmp de sortare numit `_shard_doc` care poate fi menționat explicit. În caz că nu folosești PIT, recomandarea este să incluzi acest câmp (*tiebreaker*) în sortare. Acest câmp ar trebui să conțină o valoare unică pentru fiecare document. În caz că nu introduci un câmp tiebreaker rezultatele paginate ar putea rata sau duplica hit-uri.

Căutările *search after* sunt optimizate pentru viteză când ordinea de sortare este `_shard_doc` și când totalul hiturilor nu este contabilizat.

În cazul în care câmpul de sortate este de tip `date`, dar `date_nanos` în alte ținte de căutare, folosește parametrul `numeric_type` pentru a converti la un numitor comun valorile la care vei adăuga prametrul `format` pentru a specifica formatul de date pentru câmpul de sortare. Dacă nu te asiguri de acest lucru, Elasticsearch nu va interpreta corect datele de căutare pentru fiecare cerere. Vezi exemplul de mai sus.

Tiebreakerul `_shard_doc` este adăugat automat pentru fiecare cerere care folosește un PIT. Valoarea sa este o combinație dintre indexul shard-ului din PIT și ID-ul intern al documentului din Lucene. Această valoare este unică pentru fiecare document și constantă în interiorul unui PIT format.

Pentru a personaliza căutarea, se poate adăuga un tiebreaker în mod explicit pentru a modifica ordinea. Un exemplu este mai jos.

```json
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
    "keep_alive": "1m"
  },
  "sort": [
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}},
    {"_shard_doc": "desc"}
  ]
}
```

Rezultatul întors ar putea fi următorul.

```json
{
  "pit_id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
  "took" : 17,
  "timed_out" : false,
  "_shards" : ...,
  "hits" : {
    "total" : ...,
    "max_score" : null,
    "hits" : [
      ...
      {
        "_index" : "my-index-000001",
        "_id" : "FaslK3QBySSL_rrj9zM5",
        "_score" : null,
        "_source" : ...,
        "sort" : [
          "2021-05-20T05:30:04.832Z",
          4294967298
        ]
      }
    ]
  }
}
```

PIT-ul a fost actualizat pentru momentul de timp în care s-a făcut căutarea (`pit_id`). În array-ul cheii `sort` vei găsi valorile pentru ultimul `hit` returnat. Iar imediat, al doilea element este valoarea tiebreaker-ului care este unică petru respectivil PIT.

Pentru a obține următoarea pagină de rezultate, rulează din nou ultima căutare folosind valoarea ultimului hit din array-ul sort ca valoare pentru `search_after`. Dacă folosești PIT, introdu și ultimul id de PIT în `pit.id`.

```json
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
    "keep_alive": "1m"
  },
  "sort": [
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}}
  ],
  "search_after": [
    "2021-05-20T05:30:04.832Z",
    4294967298
  ],
  "track_total_hits": false
}
```

Argumentele de la `sort` și `query` trebuie să rămână neschimbate. Dacă din motive de aplicație, trebuie să introduci argument la `from`, acesta trebuie să fie `0` (valoarea din oficiu) sau `-1`.
Observă optimizarea pentru viteză `"track_total_hits": false`.

De îndată ce au terminat căutarea, ar trebui să ștergi PIT-ul. Chiar dacă `keep_alive` va face acest lucru după expirarea timpului, buna practică spune să-l închizi dacă ai consumat datele mai devreme.

```bash
DELETE /_pit
{
    "id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

API-ul returnează un rezultat similar cu următorul obiect.

```json
{
   "succeeded": true,
   "num_freed": 3
}
```

Proprietatea `num_freed` indică câte contexte de căutare au fost șterse, fiind eliberate resursele.

### Search slicing

Atunci când paginezi un număr mare de documente, este foarte util să segmentezi căutarea pentru a fi consumate independent. Pe API-ul de căutare ai putea trimite un obiect similar.

```json
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "query": {
    "match": {
      "message": "foo"
    }
  },
  "pit": {
    "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
  }
}
```

Fiecare slice are un id.

```json
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "pit": {
    "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
```

Pentru toate slice-urile trebuie folosit același PIT.

## Scroll pe rezultatele de căutare

Pentru paginarea de mare adâncime nu este recomanadată folosirea API-ului de scrolling. Dacă treci prin mai mult de 10000 de documente, trebuie să conservi starea indexului. În aceste cazuri, cel mai potrivit este să folosești parametrul `search_after` cu PIT-uri.

O cerere de căutare `search` returnează o singură pagină cu rezultate. API-ul `scroll` permite aducerea unui număr mare de rezultate sau chiar a tuturor rezultatelor deodată într-o singură cerere de căutare. O astfel de căutare este similară unui cursor.

Scrollingul nu este gândit pentru cereri real-time, ci pentru procesarea unor cantități mari de date cu scopul de a reindexa conținutul unui data stream sau a unui index într-un data stream nou ori indexarea cu alte configurări.

Pentru reindexare și scrolling, ai suport pentru NodeJS la https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/client-helpers.html.

### Cum funcționează scrolling-ul

O cerere de căutare se trimite spre execuție având parametrul `scroll`. Fiecare răspuns conține un `_scroll_id` care trebuie folosit pentru următoarea cerere de căutare. După ce ai *consumat* (afișat) toate răspunsurile, poți șterge `scroll_id` pentru a elibera resursele. Ceea ce se întâmplă pe server este o *înghețare* a datelor care existau din momentul în care s-a făcut prima cerere de căutare. Totuși toate operațiunile de scriere a unor date noi în index (`delete`, `index` și `update`) care au apărut după inițierea scroll-ului, nu vor mai face parte din setul interogat. Acesta este mecanismul care garantează că setul este consistent.

Resursele trebuie eliberate la finalul utilizării lor în interfață pentru că segmentele de date, care între timp e posibil să se fi schimbat și să fi fost și *merged*, au fost menținute pentru a avea reperul fix. Combinat cu mai multe alte căutări înseamnă o multiplicare a datelor care trebuie menținute în afara setului live. Mai multe date înseamnă mai multe segmente care trebuie ținute, mai multă memorie heap pentru metadatele segmentelor care și ele tot în memoria heap sunt, mai multe fișiere deschise ce trebuie gestionate, etc.

Menținerea în viață a segmentelor care nu sunt necesare pentru a avea la dispoziție date în timp real înseamnă alocarea de spațiu mai mult pe disc pentru a ține în viață segmentele. Acestea nu pot fi șterse până când nu este șters id-ul de scroll.

Reține un lucru foarte important: căutarea în baza unui scroll, împreună cu tot contextul pe care-l formează, este legată de o cerere de căutare (*query*). Astfel, vei obține un set de date consistent. Pentru cazul în care dorești să rulezi mai multe interogări de căutare folosind același set de date, folosești Point-In-Time. Structura pe care o creează PIT-ul este dedicată activităților de căutare multiple pe același set.

## Resurse

- [Paginate search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)
- [Sort search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html)
- [Near real-time searchedit](https://www.elastic.co/guide/en/elasticsearch/reference/current/near-real-time.html)
- [Point in time API](https://www.elastic.co/guide/en/elasticsearch/reference/current/point-in-time-api.html)
- [Data in: documents and indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/documents-indices.html)
- [Get a consistent view of your data over time with the Elasticsearch point-in-time reader | Alexander Reelsen | 17 JUNE 2021](https://www.elastic.co/blog/get-a-consistent-view-of-your-data-over-time-with-the-elasticsearch-point-in-time-reader)
- [Elasticsearch 7.10.0 released | George Kobar | 11 NOVEMBER 2020](https://www.elastic.co/blog/whats-new-elasticsearch-7-10-0-searchable-snapshots-store-more-for-less)
- [Elasticsearch node js point in time search_phase_execution_exception](https://stackoverflow.com/questions/66693428/elasticsearch-node-js-point-in-time-search-phase-execution-exception)

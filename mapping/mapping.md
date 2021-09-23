# Mapping

Un motor de căutare îndeplinește două operațiuni importante: indexează documente și regăsește documente în baza unor criterii.

Mapping-ul este procesul prin care definești cum este stocat și indexat un document cu toate câmpurile sale. Fiecare document este o colecție de câmpuri, fiecare conținând valori de un anumit tip de date.

Înainte de a introduce date în Elasticsearch, trebuie să creezi un document care definește corespondențele dintre câmpurile documentului care trebuie indexat și cel care va fi introdus în indexul Elasticseach. Acest document de corepondențe se numește *mapping*. Poți să te gândești la mapping ca la un plan de construcție a unui index. Fiecare index are un *mapping type* care **determină cum vor fi indexate documentele**. Dacă nu este introdus un mapping la momentul indexării, este creat unul din oficiu care va încerca aproximarea structurii pornind de la câmpurile datelor din documentul care trebuie indexat. Apoi această mappare este propagată în toate nodurile unui cluster.

Cân este citit mappingul, ES verifică dacă valoarea câmpului este validă, adică respectă tipul menționat. Dacă respectivul câmp este deja prezent în mapping, dar valoarea este diferită de cea specificată, motorul va încerca să facă o actualizare a mapping-ului pentru a putea acomoda valoarea. De exemplu, dacă în mapping ai specificat un număr întreg și în câmpul documentului de indexat este un long, motorul va face modificarea. Dacă tipurile de date nu sunt compatibile, va fi indicată o stare de excepție și indexarea va eșua. Dacă respectivul câmp nu există, ES va autodetecta tipul câmpului și va permite indexarea.

Pentru a defini datele, se va folosi *dynamic mapping* și *explicit mapping*.

De cele mai multe ori, Elasticsearch va deduce ce tipuri de date folosești și va crea un mapping necesar automat.

Un mapping este o schemă a datelor (*schema definition*) pe care câmpurile unui document le va găzdui. Aceast mapping îi spune lui Elasticsearch cum să introducă datele pentru a fi indexate. Un *mapping type* este compus din:

- Meta-fields sau metadata fields așa cum este `_source` și
- Fields (câmpurile).

Meta-fields sunt câmpuri folosite pentru a atașa metadate necesare descrierii unui document în interiorul indexului. Câmpurile acestuia sunt:

- `_index`,
- `_type`,
- `_id`,
- `_source`.

Fields (*câmpuri*) sunt proprietățile documentului în sine. Fiecare câmp va înmagazina un anumit tip de date. Cele simple sunt:

- `text`,
- `keyword`,
- `date`,
- `long`,
- `double`,
- `boolean`,
- `ip`.

Apoi pot fi obiecte care suportă structura ierarhică a unui obiect JSON precum `object` și `nested`.

Sunt și tipuri de câmpuri specializate precum `geo_point`, `geo_shape` sau `completion`.

Există două tipuri de mapping:
- dinamic și
- explicit

## Dynamic mapping

Acest tip de mapping permite experimentarea și explorarea datelor când începi lucrul cu Elasticserch. Elasticsearch adaugă automat câmpuri noi automat pur și simplu prin indexarea unui document. Se pot adăuga câmpuri la nivelul cel mai de sus, precum și în câmpurile unui obiect sau unele nested. Aceste mapping-uri pot fi folosite pentru a defini mapping-uri particularizate care să se aplice pe câmpuri adăugate dinamic în baza unei condiții de potrivire.

Dynamic mapping permite să nu definești mapping-uri, cel puțin pentru anumite câmpuri.
Atunci când adaugi documente, Elasticsearch va adăuga mapping-uri pentru câmpurile care nu le au definite explicit. Poți vedea cum s-au făcut mapping-urile investigând un index pe endpoint-ul `_mapping`.

```yaml
GET /miscelanee/_mapping
```

Vom obține un posibil răspuns similar cu următorul obiect JSON care pentru acest exemplu reprezintă mapping-ul unui index `miscelanee`:

```json
{
  "miscelanee" : {
    "mappings" : {
      "properties" : {
        "autor" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "likes" : {
          "type" : "long"
        },
        "tags" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "titlu" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

Când se face *dynamic mapping*, câmpurile de text au două mapări: `text` și `keyword`. Este ceea ce se numește [multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html#_multi_fields). Acest lucru constă în maparea în mai multe feluri pentru a servi mai multor scopuri. Vezi și informațiile de la parametrul `fields` al mapping-urilor.

```json
"type" : "text",
"fields" : {
  "keyword" : {
    "type" : "keyword",
    "ignore_above" : 256
  }
}
```

Acest lucru este necesar mai târziu atunci când se vor face căutările.

## Mapping explicit

Acest tip de mapping îți permite precizie în definirea mapping-ul, precum următoarele posibile condiții:

- care dintre câmpurile string ar trebui tratate drept câmpurile full text;
- care câmpuri conțin numere, date sau geolocații;
- formatul valorilor datelor calendaristice;
- reguli particularizate  pentru a controla mappingul pentru câmpurile adăugate dinamic.

Pentru a crea un mapping explicit, poți face acest lucru folosind API-ul de inițiere a unui index.

```text
PUT /un-index-01
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },
      "email":  { "type": "keyword"  },
      "name":   { "type": "text"  }
    }
  }
}
```

### Adăugarea unui câmp nou într-un mapping existent

În cazul în care ai deja un mapping pentru un index și dorești să adaugi un câmp nou la cele existente, poți folosi endpoint-ul `_mapping`, precum în exemplul de mai jos.

```bash
PUT /departament-materiale/_mapping
{
  "properties": {
    "id-departament": {
      "type": "keyword",
      "index": false
    }
  }
}
```

Observăm că la proprietățile deja existente s-a adăugat una nouă `id-departament` care este de tip `keyword`, dar care nu va fi indexată și astfel, nu va intra în calcul atunci când se fac căutări - `index: false`. [API-ul de actualizarea a unui mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html#indices-put-mapping) oferă posibilitatea de a adăuga câmpuri noi la un stream de date existent sau unui index. Acest API poate fi folosit pentru a modifica setările privind căutarea după anumite câmpuri care există deja.

După cum am văzut și în exemplul de mai sus, API-ul are următoarea semnătură `PUT /<target>/_mapping`. În cazul în care rulezi o instanță Elastisearch care este securizată, trebuie să ai permisiunea de a gestiona [privilegiile](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-privileges.html#privileges-list-indices) asupra indexului. Revenind la semnătură, pentru `<target>` trebuie să ai un string care poate fi o listă de stream-uri de date separate prin virgulă, pot fi indici sau alias-uri. În caz de nevoie poți apela la wildcard-uri precum steluța (`*`). Pentru a ținti toate stream-urile de date și indicii, se va omite acest parametru sau va fi folosită steluța (`*`) ori `_all`.

#### Parametrii cererii

##### allow_no_indices

Este un Boolean opțional, care dacă este setat la `false` va conduce la returnarea unei erori în cazul în care o expresie wildcard, un alias al unui index sau o valoare `_all` țintesc indici care nu există sau care sunt închiși. Valoarea din oficiu este `false`. Acest comportament se aplică chiar dacă cererea țintește alți indici deschiși. De exemplu, pentru o cerere care are drept țintă `foo*,bar*`, va fi returnată o eroare dacă un index începe cu `foo`, dar niciun alt index nu începe cu `bar`.

##### expand_wildcards

Este un string opțional. Este un tip de index cu ajutorul cărora se pot face regăsiri folosind șabloane cu wildcard-uri. Dacă cererea are drept țintă stream-uri de date, acest argument determină dacă expresiile cu wildcard-uri ajung la stream-urile de date care sunt ascunse. Poate fi definit cu virgule între, precum în `open,hidden`. Valorile care pot fi pasate sunt:
- `all` care ajunge la orice stream de date sau index, incluzându-le pe cele care sunt ascunse.
- `open` ajunge la indici deschiși care nu sunt ascunși. Poate ajunge și la stream-urile de date care nu sunt ascunse.
- `closed` ajunge la indici închiși care nu sunt ascunși. Poate ajunge și la stream-urile de date care nu sunt ascunse. Stream-urile de date nu pot fi închise.
- `hidden` ajunge la streamuri de date ascunse și indici ascunși. Trebuie să fie combinați cu `open`, `closed` sau cu amândoi.
- `none` indică faptul că șabloane care folosesc wildcard-urile nu sunt acceptate.

Valoarea din oficiu este `open`.

##### include_type_name

Este un Boolean opțional care în cazul valorii `true`, se așteaptă în corpul - *body* mapping-ului să existe un mapping type. Valorea din oficiu este `false`.

##### ignore_unavailable

Este un Boolean opțional care în cazul valorii `true`, nu va include în răspuns indici închiși sau care nu există.

##### master_timeout

Este o valoare de timp opțională care indică cât trebuie să aștepți pentru o conexiune la nodul master. Dacă nu este primit niciun răspuns înainte ca timpul să expire, cererea eșuează și returnează o eroare. Valoarea din oficiu este 30s.

##### timeout

Este o valoare de timp opțională care indică cât trebuie să aștepți pentru un răspuns. Dacă nu este primit un răspuns în acest interval de timp, cererea eșuează și returnează o eroare. Valoarea din oficiu este de 30 de secunde.

##### write_index_only

Este o valoare Boolean, care dacă este setată la `true`, mapping-urile sunt aplicate doar indexului curent de scriere (*current write index*) al țintei. Valoarea din oficiu este `false`.

#### Corpul cererii

Acesta este un obiect de mapping pentru un câmp. Pentru noile câmpuri acest mapping include:

- numele câmpului,
- [tipurile de câmpuri](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)
- [parametrii de mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

Pentru a-i modifica pentru cei existenți există un document dedicat: https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html#updating-field-mappings.


Este opțională

## Meta fields

Fiecare document inclus în Elasticsearch, are un set de metadate asociat în plus de câmpurile sale. Acest set de metadate sunt numite *meta fields*.

### `_index`

Acest *meta field* este adăugat automat, iar valoarea sa este numele indexului în care stă un document.

### `_id`

Valoarea acestui meta field este id-ul documentelor.

### `_source`

Valoarea acestui meta field este chiar obiectul JSON folosit atunci când s-a făcut indexarea.

### `_field_names`

Conține numele fiecărui câmp care conține o valoare nenulă. Este util atunci când dorim să vedem dacă un document există în baza unui criteriu/valoare existentă.

### `_routing`

Stochează valoare folosită pentru a ruta un document la un shard.

### `_version`

Elasticsearch folosește versionarea la nivel intern. Dacă ceri un document după id-ul său, acest met field va fi parte din rezultat. Valoarea este un număr întreg care se modifică pe măsură ce documentul este modificat.

### `_meta`

Acest meta field poate fi folosit pentru a stoca date custom, care nu sunt procesate de Elasticsearch. Acesta este câmpul în care poți stoca date specifice fiecărei aplicații. Poți introduce metadate care să fie utile unui ORM sau în scop de afișare

```json
{
  "adresa": {
    "_meta": {
      "strada": {"nume_strada": "text"}
    }
  }
}
```

## Parametrii de mapping

În funcție de tipul de date, este posibil să instruiești Elasticseach cu privire la modul în care să facă procesarea respectivului câmp cu scopul de a realiza un management optim. Acești parametrii pot fi specificați pentru fiecare câmp indicând motorului Elasticsearch ce trebuie să facă atunci când primește date pentru a le transforma în documente ale indexului.

### store

Această directivă atunci când are valoarea `yes` indică faptul că respectivul câmp trebuie stocat într-un fragment separat al indexului cu scopul de a se realiza o regăsire rapidă. Această directivă are drept efecte un consum mai mare de spațiu, dar computația este redusă dacă ai nevoie să extragi un câmp dintr-un document în cazul scripting-ului și al agregărilor.
Valoarea din oficiu este `no`.
Câmpurile stocate se dovedesc a fi mai rapide la momentul în care se face o fațetare.

### index

Această directivă poate avea trei posibile valori și configurează un câmp care va fi indexat:
- `no` indică faptul că respectivul câmp nu se dorește a fi indexat;
- `analyzed` indică faptul că respectivul câmp va fi indexat și se va folosi analizorul configurat. Folosind configurarea din oficiu a ES, la momentul analizei, se va proceda la un lowercase urmat de tokenizare. Acesta este și opțiunea din oficiu pentru directiva `index`;
- `not_analyzed` indică faptul că respectivul câmp va fi procesat și indexat, dar nu va fi trecut prin analizor. Configurarea din oficiu a ES folosește câmpul `KeyworkAnalyzer` care procesează câmpul ca un singur token.

Dacă dorești ca un câmp să fie indexabil, poți specifica acest lucru.

```json
"properties": {
    "genre": {
        "index": "false"
    }
}
```

Dacă este `true`, acest câmp va apărea în indexul inversat (*inverted index*).

### null_value

Această directivă definește o valoare din oficiu pentru cazul în care valorea pentru respectivul câmp nu există. Dacă unul din câmpuri primește o valoare `null` la venirea datelor, valoarea acestuia va fi completată cu valoarea lui `null_value`.

```json
{
  "properties": {
    "likes": {
      "type": "integer",
      "null_value": 0
    }
  }
}
```

### boost

Această directivă este folosită pentru a modifica *importanța* unui anumit câmp. Valoarea din oficiu este `1.0`. Parametrul [boost]() este folosit pentru a specifica cu cât se va ridica automat scorul de relevanță dacă se face căutarea după respectivul câmp.
Parametrul este aplicat doar pentru **term queries**.

### index_analyzer

Definește analizorul care trebuie folosit pentru documentul curent.

Indică faptul că trebuie folosit un analizor în timp ce se face căutarea. Dacă acesta nu este definit, este folosit analizorul obiectului părinte. Valoarea din oficiu este `null`.

```json
{
  "order" : {
    "index_analyzer":"standard"
  }
}
```

În cazul în care este definit la nivel de câmp, atunci respectivul câmp este ridicat la rangul de *default*.

În cazul în care setezi `index_analyzer` și `search_analyzer` cu aceeași valoare, poți folosi proprietatea `analyzer`.


### search_analyzer

Această directivă indică care este analizorul care va fi folosit în timpul căutării. Dacă acesta nu este definit, este folosit analizorul obiectului părinte, cel setat la nivel de document, fiind cel mai sus în ierarhie. Valoarea din oficiu este `null`.

```json
{
  "order" : {
    "index_analyzer":"standard",
    "search_analyzer":"standard"
  }
}
```

În cazul în care setezi `index_analyzer` și `search_analyzer` cu aceeași valoare, poți folosi proprietatea `analyzer`.

### analyzer

Această directivă setează câmpurile `index_analyzer` și `search_analyzer` la valoarea menționată. Valoarea din oficiu este `null`. Doar câmpurile tip `text` permit menționarea unui [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analyzer.html). Parametrul specifică ce analizor să fie folosit pentru indexare și/sau căutare.

### include_in_all

Indică faptul că pentru câmpul curent trebuie să se facă o indexare într-un câmp special `_all`. Acesta este un câmp în care este tot textul concatenat al tuturor celorlalte câmpuri.

### index_name

Acesta este numele câmpului care va fi stocat în index. Această proprietate îți permite să redenumești câmpul la momentul în care se face indexarea. Poate fi folosit în cazul migrării datelor fără a afecta nivelul aplicației datorită modificărilor apărute.

### norms

Această directivă controlează *norms* care aparțin lui Lucene. Acest parametru este folosit pentru a da un scor mai bun căutărilor în cazul în care respectivul câmp este folosit doar pentru filtrare. Buna practică spune să-l dezactivezi pentru a evita consumul resurselor. Valoarea din oficiu este `true` în cazul câmpurilor analyzed și `false` pentru cele `not_analyzed`.
Dacă `true`, va marca respectivul câmp ca fiind relevant pentru a calcula scorul de relevanță. Nu poți activa `norms` mai târziu fără a reface indexul.

### `coerce`

Acest parametru permite transformarea datelor la tipul care a fost menținat pentru câmp. Acest parametru când are valoarea `true`, va *converti* valorile care seamnănă a fi numere în numere. O valoare venită ca string `"10.2"`, va fi convertită la `10.2`.


### `copy_to`

Acest parametru permite copierea valorilor a mai multor câmpuri într-un câmp separat (*group field*), care mai apoi să poată fi interogat ca un câmp unic. De exemplu, valorile a două câmpuri posibile `nume` și `prenume` poate fi copiate într-un al treilea numit `nume_complet`.

```yaml
PUT nume_index
{
  "mappings": {
    "properties": {
      "nume": {
        "type": "text",
        "copy_to": "nume_complet"
      },
      "prenume": {
        "type": "text",
        "copy_to": "nume_complet"
      },
      "nume_complet": {
        "type": "text"
      }
    }
  }
}
```

Permite crearea de câmpuri custom.

### dynamic_templates

Este o listă de template-uri care vor modifica mapping-ul explicit dacă unul dintre acestea se potrivește. Regulile acestora sunt folosite pentru a constitui mapping-ul final.

```json

{
  "order" : {
    "index_analyzer":"standard",
    "search_analyzer":"standard",
    "date_detection": true,
    "dynamic_templates":[
      {
        "template1":{
          "match":"*",
          "match_mapping_type":"long",
          "mapping":{"type":"{dynamic_type}", "store":true}
        }
      }
    ]
  }
}
```

Un template dinamic este constituit din două părți: șablonul după care se face potrivirea unei structuri (`match`) a documentului de indexat și mappingul în sine.

Tipuri de șabloane de protrivire:
- `match` care permite potrivirea după numele unui câmp. Valoarea este un [standard glob pattern](http://en.wikipedia.org/wiki/Glob_(programming);
- `unmatch`, care este opțional permite definirea expresiei care va fi folosită pentru a exclude documentele care se potrivesc;
- `match_mapping_type` este opțional și controlează tipurile de câmp;
- `path_match` este opțional și îți permite să potrivești un template dinamic folosind notația cu punct pentru un câmp. De exemplu, `personal.*.value`;
- `path_unmatch` este opțional și exclude câmpurile potrivite. Este opusul lui `path_match`;
- `match_pattern` este opțional și îți permite să faci potriviri folosing regex. În caz contrar, este folosit un glob pattern.

După cum se observă și în exemplu, se pot folosi înlocuitori speciali:
- `{name}` acesta va fi înlocuit cu numele câmpului;
- `{dynamic_type}` va fi înlocuit cu tipul câmpului.

Ordinea template-urilor dinamice este foarte importantă în potrivire. Doar cel care este potrivit va fi executat. Buna practică spune să punem primul pe cel cu cele mai stricte reguli.

Template-uile dinamice sunt la îndemână atunci când ai nevoie să setezi o configurare de mapping pentru toate câmpurile deodată.

```json
"dynamic_templates" : [
  {
    "șablon_generic" : {
      "match" : "*",
      "mapping" : {
        "store" : "yes"
      }
    }
  }
]
```

În exemplul de mai sus, toate câmpurile care se potrivesc cu setările făcute prin template-ul dinamic, vor fi mapate urmând regulile.

### `doc_values`

După ce documentele au fost indexate, acestea sunt scrise pe disc. La momentul în care se face o agregare, o sortare, sau o accesare a unei valori într-un script, motorul se va uita la datele care sunt valorile câmpurilor unui document și cu acestea va opera.

Aceste date ale documentelor constituie adevărate structuri pe disc care sunt aranjate pentru a putea fi citite foarte rapid pentru scenariile care implică sortarea sau agregarea.

Câmpurile care nu suportă crearea de doc values sunt cele `text` și `annotated_text`. Toate câmpurile care permit această setare, o vor avea activată din oficiu.

În cazul în care ești sigur că nu vei face o agregare, o sortare sau citi valorile dintr-un script, se recomandă dezactivarea pentru a salva spațiu pe disc.

### `dynamic`

Dacă ai nevoie să indexezi rapid ceva, din oficiu, câmpurile pot fi adăugate dinamic într-un document. Activează sau dezactivează adăugarea de câmpuri documentelor sau obiectelor imbricate în mod dinamic.

```json
{
  "mappings": {
    "default": {
      "dynamic": false,
      "properties":{
        "nume":{
          "dynamic": true,
          "properties": {}
        }
      }
    }
  }
}
```

### `eager_global_ordinals`

### `enabled`

Este pentru momentul în care dorești să stochezi date fără a le indexa.

### `fielddata`

### `fields`

Acest parametru este folosit pentru a indexa câmpurile în diferite feluri.

### date_detection

Această proprietate activează extragerea unei date calendaristice dintr-un string. Din oficiu are valoarea `true`.

```json
{
  "order" : {
    "index_analyzer":"standard",
    "search_analyzer":"standard",
    "date_detection": true
  }
}
```

### `format`

Definește formatul datelor calendaristice. Posibilele valori sunt:

- "yyyy-MM-dd",
- "epoch_millis",
- "epoch_second".

Valoarea din oficiu este "strict_date_optional_time||epoch_millis".

### dynamic_date_formats

Poți menționa o listă cu formate valide pentru datele calendaristice.

```json
{
  "order" : {
    "analyzer":"standard",
    "date_detection": true,
    "numeric_detection": true,
    "dynamic_date_formats": ["yyyy-MM-dd", "dd-MM-yyyy"]
  }
}
```

Această opțiune poate fi folosită doar dacă `"date_detection": true`.

### numeric_detection

Permite conversia string-urilor la numere dacă acest lucru este posibil. Valoarea din oficiu este `false`.

```json
{
  "order" : {
    "analyzer":"standard",
    "date_detection": true,
    "numeric_detection": true
  }
}
```

### `ignore_above`

### `ignore_malformed`

### `index_options`

### `index_phrases`

### `index_prefixes`

### `meta`

### `normalizer`

### `position_increment_gap`

### `properties`

### `search_analyzer`

### `similarity`

### `term_vector`

Acest parametru este folosit pentru mappingul aplicat șirurilor de caractere. Acesta este un vector al tuturor termenilor care compun un șir de caractere. Opțiunile pentru valoare sunt:
- `no` pentru valoarea așa cum este fără a constitui acest vector;
- `yes` stochează câmpul `term_vector`;
- `with_offsets` stochează un *term_vector* având un offset pentru token-uri (începutul și finalul poziției dintr-un bloc de caractere);
- `with_positions` stochează poziția unui token din câmpul *term_vector*;
- `with_positions_offsets` va stoca toate datele pentru un *term_vector*.


## Specificarea tipurilor de date

Poți specifica tipurile de date - `text`, `keyword`, `byte`, `long`, `short`, `integer`, `float`, `double`, `boolean`, `date`.

```json
"properties": {
    "id": {
        "type": "long"
    }
}
```
## Mapping-ul array-urilor

Elastisearch lucrează în mod nativ cu array-urile. Nu există o setare specială pentru array-uri. Singura regulă este ca elementele acelui array care va fi indexat să aibă același datatype. Atunci când obții un document din index, oricare array ar fi fost drept valoare unui câmp, va fi adus cu elementele în aceeași ordine precum era la momentul indexării. Câmpul `_source` va conține fix același obiect JSON pe care l-ai trimis spre indexare.

Totuși, array-urile sunt supus procedurii de regăsire din poziția de câmpuri *multivalue*, ceea ce înseamnă că array-ul este tratat ca o pungă cu elemente, fără structura pe care o dă indexarea.


De exemplu, dacă dorești stocarea de etichete pentru un document, poți proceda precum în următorul exemplu.

```json
{
  "angajati": {
    "properties": {
      "nume": {"type": "string", "index": "analyzed"},
      "etichete": {"type": "string", "store": "yes", "index": "not_analyzed"}
    }
  }
}
```

Această mapare va indexa următoarele posibile variate de documente.

```json
{"name": "primul_angajat", "etichete": "primul"}
```

Și un array de etichete.

```json
{"name": "primul_angajat", "etichete": ["primul", "first", "1984"]}
```

Datorită faptului că Lucene este la baza Elasticsearch-ului, mai multe valori sunt adăugate unui document care care același nume de câmp. În ES acesta se numește `index_name`, care în cazul în care nu este definit în mapping, va fi chiar numele câmpului.

## Mapping-ul obiectelor

Pentru obiecte nu este necesară specificarea tipului obiect sau ceva asemănător. Elasticseach detectează după setări că este vorba despre configurarea unui obiect care va fi parte a documentului. Acest lucru este posibil pentru că Elastisearch folosește nativ formatul JSON ceea ce înseamnă că orice structură oricât de complexă ar fi, poate fi mapată cu ușurință.
În momentul în care Elastisearch face parsingul unui obiect, mai întâi caută să extragă câmpurile pentru a le procesa în acord cu mapping-ul existent. Dacă nu ai un mapping, va *învăța* structura obiectului folosindu-se de o procedură numită *reflection*.

Cele mai importante atribute ale unui obiect sunt:
- `properties` care este colecția câmpurilor sau a obiectelor (echivalentul coloanelor SQL);
- `enabled` care indică dacă obiectul trebuie să fie procesat. În cazul în care este setat la `false`, datele conținute în obiect nu vor fi indexate și astfel, nu vor putea fi regăsite. Valoarea din oficiu este `true`;
- `dynamic` permite o mapare automată a numelor câmpurilor folosind metoda de *reflection* pentru valorile datelor primite. Valoarea din oficiu este `true`. În cazul în care este setat la `false`, atunci când vei încerca să indexezi un obiect care conține un câmp nou, acest lucru nu se va petrece și nu va fi atrasă atenția în niciun fel. În cazul în care are valoarea `strict`, atunci când este prezent un nou câmp în obiect, va fi indicată o eroare, iar procesul de indexare trece mai departe. Controlul acestui parametru îți permite o mai mare siguranță în ceea ce privește modificările în structura documentului.
- `include_in_all` adaugă valorile obiectului în câmpul special `_all` și are din oficiu valoarea `true`. Adu-ți aminte că în `_all` sunt agregate textele tuturor câmpurilor din document.

Pe scurt, vei folosi `properties` care îți permite să mapezi câmpurile obiectului primit în câmpuri Elastisearch. Poți să renunți la o parte din obiectul original dacă dorești salvarea spațiului, dar nu vei putea regăsi date.



Un posibil exemplu ilustrativ.

```yaml
PUT /test
{
  "mappings": {
    "properties": {
      "obi": {
        "properties": {
          "primo": {"type": "text"},
          "secundo": {
            "properties": {
              "intern": {
                "type": "boolean"
              }
            }
          }
        }
      }
    }
  }
}
```

Rularea comenzii `GET /test/_mapping` aduce ca răspuns tocmai obiectul de configurare folosit pentru a inița indexul.

```json
{
  "test" : {
    "mappings" : {
      "properties" : {
        "obi" : {
          "properties" : {
            "primo" : {
              "type" : "text"
            },
            "secundo" : {
              "properties" : {
                "intern" : {
                  "type" : "boolean"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Pentru a crea un document.

```bash
POST /test/_doc/1
{
  "obi": {
    "primo": "ceva interesant",
    "secundo": {
      "intern": true
    }
  }
}
```

Acum ceva foarte interesant. De fiecare dată când vei adăuga în index un document nou care conține alte câmpuri decât cele care au fost declarate în mapping inițial, vor fi create mapping-uri automat pentru ele, fiind actualizat mappingul. De exemplu, să adăugăm un document nou care conține un câmp suplimentar.

```bash
POST /test/_doc/2
{
  "batarang": {
    "aha": 10
  },
  "obi": {
    "primo": "ceva interesant",
    "secundo": {
      "intern": true
    }
  }
}
```
Rularea comenzii `GET /test/_mapping` aduce ca răspuns tocmai noul mapping al indexului.

```json
{
  "test" : {
    "mappings" : {
      "properties" : {
        "batarang" : {
          "properties" : {
            "aha" : {
              "type" : "long"
            }
          }
        },
        "obi" : {
          "properties" : {
            "primo" : {
              "type" : "text"
            },
            "secundo" : {
              "properties" : {
                "intern" : {
                  "type" : "boolean"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### Obiecte în obiecte - inner objects

Obiectele JSON care sunt indexate pot avea structuri adânci, fapt care implică și posibilitatea de a acea drept valori a unei proprietăți un alt obiect. Numim aceste obiecte *obiecte interne* (*inner objects*). Vom mapa aceste obiecte precizând prin prima proprietate tipul obiect, urmată de `properties` care mapează câmpurile obiectului intern.

```json
{
  "mesaj": {"type": "string"},
  "user": {
    "type": "object",
    "properties": {
      "nume": {
        "type": "object",
        "properties": {
          "nume_intreg": {"type": "string"},
          "nume_user": {"type": "string"},
          "prenume_user": {"type": "string"}
        }
      },
      "varsta": {"type": "long"}
    }
  }
}
```

Lucene nu înțelege obiecte, ci doar perechi cheie valoare. Astfel, obiectele vor fi aplatizate. O posibilă structură aplatizată ar fi exemplul următor.

```json
{
  "mesaj":                  ["Salutare", "tuturor"],
  "user.nume.nume_intreg":  ["Ionel", "Ailenei"],
  "user.nume.nume_user":    ["Ionel"],
  "user.nume.prenume_user": ["Ailenei"],
  "user.varsta":            [23]
}
```

Pentru fiecare cheie se va forma o adevărată cale.

### Array-uri de obiecte

Din nefericire, la array-urile de obiecte, corelațiile dintre câmp și valoare se pierd în momentul în care se face aplatizarea.

```json
{
  "prieteni": [
    {"nume": "Săndel Văru", "varsta": 23},
    {"nume": "Ionel Cibanu", "varsta": 35}
  ]
}
```

Documentul va fi aplatizat astfel:

```json
{
  "prieteni.nume": ["săndel", "văru", "ionel", "ciobanu"],
  "prieteni.varsta": [23, 35]
}
```

Totuși, obiectele interne care sunt în array-uri pot oferi răspunsuri la interogare dacă sunt organizate ca obiecte nested.

### Mappingul obiectelor nested

Acesta este o versiune specializată de obiect care permite indexarea array-urilor de obiecte. Aceste obiecte care fac parte dintr-un array vor putea fi interogate independent unul de altul. Potrivit arhitecturii de indexare a lui Lucene, toate câmpurile dintr-un obiect intern/inculcat documentului sunt privite drept un singur obiect. Pentru a păstra separarea, ES folosește ceea ce se numește *nested* objects și *nested queries*.

```json
{
  "cunoscuți": {
    "properties": {
      "id": {"type": "string", "store" : "yes", "index":"not_analyzed"},
      "date" : {"type" : "date", "store" : "no", "index":"not_analyzed"},
      "detalii": {
        "type": "nested",
        "name" : {"type" : "string", "store" : "no", "index":"analyzed"}
      }
    }
  }
}
```

Atunci când un document este indexat, dacă un obiect intern este marcat a fi *nested*, acest obiect va fi *extras* din documentul orginal și va fi indexat într-un alt document extern nou.

Obiectele nested sunt documente speciale ale Lucene care sunt salvate în același bloc de date precum părintele. Acest lucru permite un joining ulterior la căutare.

Obiectele *nested* nu pot fi căutate folosind query-urile standard, ci folosindu-le pe cele *nested*. Obiectele nested nu vor apărea în rezultatele de interogare standard. Documentele nested sunt dependente de părinte. De exemplu, modificare sau ștergerea documentului părinte va declanșa aceleași operațiuni și pentru cele nested. Dacă ai nevoie să modifici documentul nested, de fapt trebuie să reindexezi părintele.

Un alt caz este atunci când se face indexarea unui array a cărui elemente sunt array-uri, acest array este adăugat ca un câmp de tip obiect. Drept urmare, integritatea referințelor către proprietățile obiectelor va dispărea.

Pentru aceste cazuri, proprietatea care va avea drept valoare array-ul de obiecte trebuie declarată ca `nested` (vezi documentația pentru [nested fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/nested.html#nested-fields-array-objects)).

Ceea ce se va petrece atunci când sunt introduse documente este că fiecare dintre obiectele unui array vor fi indexate separat în documente ascunse. Toate aceste documente ascunse vor putea fi interogate folosind [nested queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/query-dsl-nested-query.html).

Aceste obiecte vor putea fi analizate cu agregări *nested* și *reverse_nested*. Sortarea se va face cu cea specifică pentru nested.

## Cum sunt analizate câmpurile

La momentul în care sunt inițiate căutările pe câmpuri, rezultatele returnate pot fi modelate chiar de la bun început, atunci când este constituit `mapping`-ul pentru respectivele documente. În următorul exemplu, sunt definite câmpurile prin indicarea tipurilor lor și apoi sunt aplicate constrângerile de regăsire impuse prin analizoare.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
    }
  }
}'
```

## Câmpuri cu mai multe mapping-uri

Adeseori, un câmp trebuie procesat cu mai multe tipuri. De exemplu, un câmp string trebuie procesat ca *analyzed* pentru a răspunde la căutare și ca *not_analyzed* în caz că este necesară sortarea. Pentru a realiza acest lucru, se va folosi proprietatea `fields`.

```json
"name": {
  "type": "string",
  "index": "not_analyzed",
  "fields": {
    "name": {
      "type": "string",
      "index": "not_analyzed"
    },
    "tk": {
      "type": "string",
      "index": "analyzed"
    },
    "code": {
      "type": "string",
      "index": "analyzed",
      "analyzer": "code_analyzer"
    }
  }
}
```

Ceea ce se petrece este că în timpul indexării, atunci când ES procesează câmpurile, le reprocesează pentru fiecare subcâmp definit de mapping. Pentru a accesa subcâmpurile unui multifield, avem nevoie de noi path-uri care se formează din numele câmpului plus numele subcâmpului. Pentru exemplul de mai sus avem:

- `name` care conduce la câmpul orginal;
- `name.tk` care trimite la câmpul indexat ca *analyzed* (*tokenized*);
- `name.code` care trimite la un câmp *analyzed* ce folosește un analizor care extrage codul numit *code_analyzer*.

Pentru un câmp *name* care ar avea următorul string „raftul procesoarelor prezintă campionul: Intel i9”, am putea avea posibilele subcâmpuri:
- `name`: *raftul procesoarelor prezintă compionul: Intel i9* care este util pentru sortare;
- `name.tk`: `["raft", "procesoarelor", "prezintă", "campionul"]`, care este util în căutări;
- `name.code`: `["Intel i9"]`, care este util pentru căutare și fațetare.

Această procedură este foarte utilă pentru a trata datele string din mai multe perspective. Subcâmpurile unui multifield sunt tipuri standard de câmpuri care permit orice procesare dorită: căutare, filtrare, fațetare și scripting.


### Tipurile câmpurilor

#### `"type":"date"`

Este un câmp care așteaptă ca formatul introdus să fie de tip `Date`.

#### `"type":"keyword"`

Acest tip specifică faptul că la momentul căutării, va trebui să fie trimis către Elasticsearch forma exactă a cuvântului sau sintagmei a cărui câmp poartă acest tip. Atenție, este și **case-sensitive**.

#### `"type": "text"`

Acest tip este ceva mai iertător în ceea ce privește rezultatele returnate, permițând și definirea unor analizori.

## Modificarea unui mapping

Modificarea mapărilor pentru câmpurile existente pentru care s-a făcut mapare deja, nu se mai poate face. Încercarea de a face acest lucru se va solda cu un eșec. Pentru a face acest lucru, ar trebui să ștergem indexul, să facem unul noi cu noi mapări și apoi să reindexăm datele.

### Analizoare

Poți defini filtrul prin care operează tokenizatorul și cum sunt operate token-urile: `standard`, `whitespace`, `simple`, `english`.

```json
"properties": {
    "description": {
        "analyzer": "english"
    }
}
```

Analizoarele permit filtrarea anumitor caractere, de exemplu să convertești din HTML encoding și invers.

## Crearea unui index cu mapping

Pentru a constitui un index repede, se poate folosi *curl* din linie de comandă.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "year": { "type":"date" }
    }
  }
}'
```

În cazul în care deja ai scris într-un fișier mappingul pentru date, poți introduce direct fișierul.

```bash
curl -H 'Content-Type: application/json' -XPUT 'localhost:9200/shakespeare' --data-binary @shakes-mapping.json
```

Pentru a verifica ceea ce s-a petrecut, poți iniția din linia de comandă o interogare pe `_mapping`.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie
```

Răspunsul este simplu `{"movies":{"mappings":{"movie":{"properties":{"year":{"type":"date"}}}}}}`. Dacă dorești o versiune a răspunsului într-o formă mai lizibilă, pui și elementul de interogare `pretty` în linia de comandă.

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_mapping/movie?pretty
```

Răspunsul va fi formatat.

```json
{
  "movies" : {
    "mappings" : {
      "movie" : {
        "properties" : {
          "year" : {
            "type" : "date"
          }
        }
      }
    }
  }
}
```

## Introducerea datelor după mapping

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies/_doc/109487 -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}'
```
Răspunsul trebuie să fie ceva asemănător cu: `{"_index":"movies","_type":"_doc","_id":"109487","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}`.

Caută înregistrarea cu `curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty`.

## Vizualizarea structurii unui index după ce a fost introdus

```bash
curl -H 'Content-Type: application/json' -XGET 'localhost:9200/shakespeare/?pretty'
```

## Interogarea unui mapping existent folosind Kibana

Pentru a urmări un exemplu, să definim un mapping folosind curl. Fii foarte atent pentru că începând cu versiunea 6 a lui Elasticsearch, `curl` are nevoie să-i specifici headerul neapărat: `-H "Content-Type: application/json"`.

```bash
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/numeindex -d '
{
    "mappings": {
        "numeindex": {
            "properties": {
                "year": {
                    "type":"date"
                }
            }
        }
    }
}'
```

În exemplul de mai sus, maparea propriu-zisă se face în obiectul care este valoarea lui `properties`. Trebuie indicat și faptul că pentru a introduce obiectul de mapare în Elasticsearch pentru un anumit index, se va folosi verbul `PUT`. În cazul în care aveți la îndemână Kibana, puteți introduce mapările folosind `Dev Tools`. După introducerea obiectului de mapare, o interogare cu `GET` pe endpointul `_mappings` a indexului dorit. De exemplu:

```yaml
PUT /movies/_mapping
{
  "properties": {
      "id": {"type": "integer"},
      "year": { "type":"date" },
      "genre": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "english"}
  }
}
```

Pentru a observa ce mappinguri au fost făcute, se poate rula din Dev Tools (Kibana) comanda `GET /movies/_mapping`. Obiectul de răspuns poate fi similar cu:

```json
{
  "movies" : {
    "mappings" : {
      "properties" : {
        "genre" : {
          "type" : "keyword"
        },
        "id" : {
          "type" : "integer"
        },
        "title" : {
          "type" : "text",
          "analyzer" : "english"
        },
        "year" : {
          "type" : "date"
        }
      }
    }
  }
}
```

Mapping-urile se folosesc în mai multe scopuri.

## Resurse

- [Mapping | Elasticsearc Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)
- [Removal of mapping types | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/removal-of-types.html)
- [Mapping types | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html)
- [Mapping parameters | Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-params.html)
- [Update mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)

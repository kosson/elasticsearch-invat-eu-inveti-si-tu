# Analiza textului

Analiza de text constă în convertirea textului în tokeni sau termeni care sunt adăugați în indexul inversat pentru căutare. Analiza este făcută de un analizor care poate fi unui intern built-in sau poate fi unui construit per fiecare index.

În momentul în care se face tokenizarea, cuvintele vor fi setate la lowercase. Vor fi eliminate cuvintele frecvente, iar celelalte sunt reduse la stem-uri (rădăcinile lexicale).

De exemplu, pentru fragmentul de text `"The QUICK brown foxes jumped over the lazy dog!"`, vor fi generate stem-urile (foxes → fox, jumped → jump, lazy → lazi), iar la final, următoarele fragmente vor fi introduse în indexul inversat: `[ quick, brown, fox, jump, over, lazi, dog ]`.

Textul care a trecut prin indexare ajunge în ceea ce se numește *inverted index*. Când se face o căutare full-text, se consultă acest index inversat. Un cluster va avea cel puțin un index inversat. Dacă ai un index care are trei câmpuri indexate, vei avea trei indexuri inversate. Indexul inversat constă din toți termenii din câmpurile indexate ale tuturor documentelor unui index. Pentru fiecare termen vor fi stocate informații privind unde apare în care documentele din index.

Atunci când vei face o căutare, de fapt se va face o căutare într-un index inversat.

## Analizoare

Analizorii sunt folosiți doar pe câmpurile de text.

Un analizor constă din trei componente:

- filtre pentru caractere,
- filtre pentru token-uri și
- un tokenizator.

Fiecare dintre aceste componente ale analizorului au capacitatea de a modifica stream-urile de input.

Indexarea unui document trece prin următoarele etape:

1) Filtrarea la nivel de caractere. Această filtrare se face o dată sau de mai multe ori. Ceea ce se petrece este o modificare a caracterelor prin adăugare, transformare sau chiar eliminare a caracterelor. Există doar trei filtre pentru caractere: *HTML Strip Character Filter* (`html_Strip`), *Mapping Character Filter* (`mapping`), **Pattern Replace** (`pattern_replace`).
  1) **HTML Strip Character Filter** va elimina caracterele speciale ale HTML-ului și va decodifica HTML entities.
  2) **Mapping Character Filter** înlocuiește valorile pe baza unui obiect cheie - valoare. Pur și simplu atunci când Elasticsearch vede o anumită valoare, o înlocuiește cu un caracter sau valoare.
  3) **Pattern Replace** folosește un Regular Expression pentru a găsi caractere pe care să le înlocuiești. Funcționează și cu *capture groups*.
2) Tokenizatorul va lua textul și-l va sparge în cuvintele componente. Un analizor poate avea un singur tokenizator. Acesta va extrage cuvintele prin analiza textului eliminând spațiile goale și a caracterelor semne de punctuație. Tokenizatorul va păstra poziția cuvintelor unul față de celălalt, fapt care va ajuta la căutare, subliniere, aproximare. Acesta este tokenizatorul *standard*.
3) În etapa finală a analizei, token-urile vor trece printr-unul sau mai multe filtre. Unul din fitre se referă la cuvintele numite generic *stop words* cum ar fi articolele, etc. Poți atașa un fitru de sinonimie, care va găsi documente bazate pe sinonimul cuvintelor din query, dacă acestea există în text. Există un filtru numit *standard*, car enu face nimic, fiind un locțiitor de viitor filtru. Singurul fitru aplicat din oficiu, transformă token-urile în lowercase.

Putem face o analiză direct în Dev Tools-ul Kibana:

```yaml
POST _analyze
{
  "tokenizer": "standard",
  "text": "Some not so interesting text about a time long ago."
}
```

iar răspunsul poate fi similar cu:

```json
{
      "token" : "not",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "so",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "interesting",
      "start_offset" : 12,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "text",
      "start_offset" : 24,
      "end_offset" : 28,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "about",
      "start_offset" : 29,
      "end_offset" : 34,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "a",
      "start_offset" : 35,
      "end_offset" : 36,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "time",
      "start_offset" : 37,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "long",
      "start_offset" : 42,
      "end_offset" : 46,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "ago",
      "start_offset" : 47,
      "end_offset" : 50,
      "type" : "<ALPHANUM>",
      "position" : 9
    }
  ]
}
```
Acesta este similar cu:

```yaml
POST _analyze
{
  "analyzer": "standard",
  "text": "Some not so interesting text about a time long ago."
}
```

### Analizorii din oficiu

#### Standard analyzer (`standard`)

Acest analizor sparge textul în termeni folosind un mecanism care se numește Unicode Text Segmentation, care se referă la o limitare a operațiunilor la limitele cuvintelor.
Analizorul standard elimină punctuația, normalizează cuvintele la lowercase și opțional va elimina cuvintele *stop words*.

#### Simple analyzer (`simple`)

Divide textul în termeni atunci când întâlnește un caracter care nu este o literă. De asemenea, pune toți termenii în litere mici.

#### Stop analyzer (`stop`)

Are comportamentul unui analizor `simple`, dar care elimină *stop words*.

#### Analizoarele focalizate pe limbă

Aceste analizoare sunt folosite pentru a face analiza textului pe anumite limbi.

#### Keyword analyzer

Este un analizor care tratează întregul text suspus analizei ca pe un unic termen.

#### Analizor pattern (`pattern`)

Acest analizor folosește o expresie regulată care va fi folosită pentru identificarea separatorilor termenilor și mai oferă desfacerea textului în termeni acolo unde a fost identificat fragmentul considerat separator. În plus, mai transformă termenii în lowercase.

#### Analizorul pentru whitespace (`whitespace`)

Acest analizor sparge textul în termeni la momentul în care întâlnește un caracter catalogat a fi white space.

### Exemplu de analizor custom

Vom vrea direct index și vom configura în același timp și un analizor specific pentru limba română.

```yaml
PUT /testanalizori
{
  "settings": {
    "analysis": {
      "analyzer": {
        "romanian_stop": {
          "type": "standard",
          "stopwords":"_romanian_"
        }
      },
      "filter": {
        "stemmer_românesc":{
          "type": "stemmer",
          "name":"romanian"
        }
      }
    }
  }
}

POST /testanalizori/_analyze
{
  "analyzer": "romanian_stop",
  "text": "Acesta ține fragmentul roșu de text și în limba română"
}
```

În Dev Console (Kibana) vom rula rând pe rând comenzile. În momentul în care se va face analiza, dacă totul funcționează corespunzător, vom avea un răspuns asemănător cu următorul:

```json
{
  "tokens" : [
    {
      "token" : "ține",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "fragmentul",
      "start_offset" : 12,
      "end_offset" : 22,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "roșu",
      "start_offset" : 23,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "text",
      "start_offset" : 31,
      "end_offset" : 35,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "și",
      "start_offset" : 36,
      "end_offset" : 38,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "limba",
      "start_offset" : 42,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "română",
      "start_offset" : 48,
      "end_offset" : 54,
      "type" : "<ALPHANUM>",
      "position" : 9
    }
  ]
}
```

Să mai lucrăm la analizorul nostru customizat pentru limba română. Pe lângă analizorul custom dedicat *stop word-urilor*, vom mai introduce unul cu mai multe particularități pe care-l vom numi `analizor special`.

```yaml
PUT /testanalizori2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "romanian_stop": {
          "type": "standard",
          "stopwords":"_romanian_"
        },
        "analizor_special": {
          "type":"custom",
          "tokenizer":"standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "trim",
            "stemmer_românesc"
          ]
        }
      },
      "filter": {
        "stemmer_românesc":{
          "type": "stemmer",
          "name":"romanian"
        }
      }
    }
  }
}
```

Privind analizorul creat, observăm că trebuie să-i definit tipul drept `custom`, vom folosi un tokenizator `standard` și vom filtra toate caracterele caracteristice codării HTML, menționând în array-ul filtrelor setând `char_filter` cu valoarea `html_strip`. Mai departe am definit un token `filter` în al cărui array vom alege token filter-ul `lowercase`, `trim` și chiar un filtru customizat numit `stemmer_românesc`. Rulând comanda în Dev Console (Kibana), vom obține un răspuns care confirmă crearea indexului.

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "testanalizori2"
}
```

Acum putem testa analizorul:

```yaml
POST /testanalizori2/_analyze
{
  "analyzer": "analizor_special",
  "text": "Acesta ține <span>fragmentul</span> roșu de text și în limba română"
}
```

Din obiectul rezultatelor, observăm că a tag-ul `span` care ține de HTML a fost eliminat.

### Folosirea analizorilor în mappings

După ce am creat un analizor personalizat, trecem la partea următoare care constă în maparea câmpurilor.

```yaml
PUT /testanalizori4/_mapping
{
  "properties": {
    "titlu":{
      "type":"text",
      "analyzer":"standard"
    },
    "descriere":{
      "type":"text",
      "analyzer":"analizor_special"
    }
  }
}
```

Hai să testăm indexul prin introducerea unei document.

```yaml
POST /testanalizori4/_doc/1
{
  "titlu":"Cineva bate la ușă",
  "descrie":"Acesta ține <span>fragmentul</span> roșu de text și în limba română"
}
```

Și acum să facem o căutare pentru a vedea ce s-a indexat.

```yaml
GET /testanalizori4/_search
{
  "query": {
    "term":{
      "titlu": {
        "value": "cineva"
      }
    }
  }
}
```

### Adăugarea de analizori la indicii existenți

Pentru a adăuga analizori la un index existent, va trebui să închidem temporar indexul. Acest lucru se face invocând endpointul `_close`. Din nefericire acest lucru înseamnă că operațiunile scriere/citire vor fi suspendate.

```yaml
POST /testanalizori4/_close
```

Răspunsul poate fi asemănător cu următorul obiect JSON:

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "indices" : {
    "testanalizori4" : {
      "closed" : true
    }
  }
}
```

La indexul deja existent, vom adăuga un analizor pentru limba engleză folosind endpointul `_settings`.

```yaml
PUT /testanalizori4/_settings
{
  "analysis":{
    "analyzer":{
      "english_stop":{
        "type":"standard",
        "stopwords":"_english_"
      }
    }
  }
}
```

După adăugarea analizorului, trebuie deschis indexul operațiunilor de scriere/citire.

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true
}
```

## Tokenizatori

Categorii de tokenizatori:

- Word oriented tokenizers
- Partial Word Tokenizers
- Structured Text tokenizers

### Word oriented tokenizers

Tokenizează text în cuvintele componente.

#### Standard tokenizer (`standard`)

Este un prim exemplu de astfel de tokenizator. Are drept sarcină spargerea textului la nivel de cuvânt și eliminarea majorității simbolurilor. De regulă, acesta este folosit din oficiu.

#### Letter tokenizer (`letter`)

Acesta sparge textul atunci când întâlnește un caracter care nu este o literă, de exemplu vreun semn de punctuație.

#### Lowercase tokenizer (`lowercase`)

Funcționează precum letter tokenizer cu diferența că transformă majusculele.

#### White space tokenizer (`whitespace`)

Sparge textul în termeni acolo unde întâlnește spațiu.

#### UAX URL Email Tokenizer (`uax_url_email`)

Are același comportament precum tokenizatorul `standard` cu excepția considerării URL-urilor și a adreselor email ca fiind tokeni unici.

### Partial Word Tokenizers

Sparge textul sau cuvintele în fragemente mici. Această categorie de tokeni este folosită pentru a face căutări pe fragmente de cuvinte.

#### N-Gram tokenizer (`ngram`)

Sparge textul în fragmente acolo unde întâlnește anumite caractere și emite N-grams de o anumită dimensiune specificată. De exemplu, un fragment de text „un text ceva”, va fi procesat ca `[un, te, tex, text, ext, ce, cev, eva, va]`. Ceea ce se petrece seamnănă cu metoda `substring`.
Felul în care sunt generate n-gram-ele este legat de spargerea în cuvinte mai întâi, apoi ia primele două caractere, fiind numărul minim de caractere pe care îl setezi (e de bun simț). Apoi generează n-grame din cuvânt adăugănd succesiv la literele pe care deja le-a procesat din respectivul cuvânt formând n-grame până la o limită din oficiu de 10 caractere. Când „cursorul” a terminat cuvântul, trece la următorul și operațiunea de generare a n-gramelor repornește.

#### Edge N-Gram Tokenizer (`edge_ngram`)

Sparge textul în cuvinte atunci când întâlnește anumite caractere și apoi emite N-grams pentru fiecare cuvânt începând cu prima literă a acestuia. Diferența față de `ngram` este că generarea ngramelor se face chiar de la începutul cuvântului. Această caracteristică îl face pretabil pentru mecanisme de auto completate pe măsură ce scrii un cuvânt. Totuși, folosirea într-un astfel de scenariu ar pune presiune pe resursele Elasticsearch. În acest caz se vor folosi *sugestorii*.

### Structured Text Tokenizers

Acești tokenizatori sunt folosiți pentru texte în care se poate identifica o structură predictibilă, precum adrese de email, adrese poștale fizice, coduri poștale, identificatori, etc.

#### Word tokenizer (`word`)

Este un tokenizator no-op care afișează consideră textul din câmp ca fiind un singur termen. Este util atunci când îl combini cu filtre pentru caractere sau tokenuri.

#### Pattern tokenizer (`pattern`)

Acest tokenizer folosește expresii regulate pentru a face una din următoarele lucruri:

- găsești fragmente după o formulă de căutare indicată de expresia regulată pentru a sparge textul în termeni, fie
- folosești grupuri de captură pentru a izola fragmentele găsite drept termeni.

#### Path tokenizer (`path_hierarchy`)

Sparge valori ierarhice (de exemplu, căi ale unui sistem de operare) și emite câte un termen pentru fiecare pas din arbore:

```text
/var/www/site/fisiere
va fi tokenizat în [/var, /var/www, /var/www/site, /var/www/site/fisiere]
```

### Token filters

#### Standard Token Filter (`standard`)

Este un no-op filter. Are rol de locțiitor pentru versiunile de Elastisearch care vor urma.

#### Lowercase Token Filter (`lowercase`)

Acest filtru normalizează tokenii la o formă redactată cu minuscule.

#### Uppercase Token Filter (`uppercase`)

Transformă caracterele tokenurilor în majuscule.

#### NGram Token Filter (nGram)

Acest filtru generează n-grame de o dimensiune precizată pe baza termenilor din textul analizat.
Același lucru il face și un tokenizator. Motivul pentru care există și un filtru pe lângă tokenizatorul care face același lucru, este legat de faptul că filtrul poate fi folosit în combinație cu un alt tokenizator. Setările din oficiu indică un număr minim de 1 caracter și un număr maxim de 2 caractere.

#### Edge NGram Token Filter (`edgeNGram`)

Generează n-grame pentru fiecare termen pornind de la poziția de start a primului caracter al unui termen. Motivul pentru care există și un filtru pe lângă tokenizatorul care face același lucru, este legat de faptul că filtrul poate fi folosit în combinație cu un alt tokenizator.

#### Stop Filter (`stop`)

Acest filtru elimină caracterele de stop. Elasticsearch are o listă de caractere care pot fi considerate *stop words* (*the*, *and*, *is*), dar se pot configura și propriile caractere, dacă acest lucru este dorit.
Pentru limba engleză cuvintele considerate stop words sunt: *a, an, and, are, as, at, be, but, by, for, if, in, into, is, it, no, not, of, on, or, such, that, the, their, then, there, these, they, this, to, was, will, with*. Motivele pentru care sunt eliminate aceste cuvinte șin de calculele pentru a stabili relevanța unui document. Dat fiind faptul că aceste cuvinte apar cu o frecvență foarte mare, calculul relevanței, dar ar fi să fie luate în considerare, ar necesita resurse serioase la fiecare interogare, ceea ce nu este de dorit.

#### Word Delimiter Token Filter (`word_delimiter`)

Acest filtru sparge cuvintele în fragmente mai mici și aplică transformări pe grupuri de astfel de fragmente. Modul în care sunt constituite fragementele este în baza unor formule definite de programator.

De exemplu: *Lo-Fi, 100AD, JavaScript* vor genera următorii termeni: `["Lo","Fi",100,"AD","Java","Script"]`.

#### Stemmer Token Filter (`stemmer`)

Acest filtru face serviciul de stemming pentru o anumită limbă specificată. Valoarea din oficiu este engleză.

#### Keyword Marker Token Filter (`keyword_marker`)

Dacă dorești să previi operațiunea de stemming pe anumite cuvinte, folosești acest filtru. Pur și simplu introduci o listă de termeni protejați și orice stemmer le va ignora.

#### Snowball Token Filter (`snowball`)

Este un filtru care face stemming în baza algoritmilor existenți în limbajul de procesare a strimurilor numit Snowball.

#### Synonim Token Filter (`synonym`)

Adaugă sau înlocuiește tokeni pe baza unui fișier de configurare care precizează sinonimele. În acel fișier vom defini cum dorim să fie interpretat un termen în relație cu sinonimul său. La momentul căutării este posibilă găsirea documentelor care au alăturat altor termeni un sinonim cuvântului care există în textul indexat.

### Exemplu de tokenizer custom

Vom folosi deja exemplul pe care l-am explorat deja la analizori. Dacă încă nu ai deja în Dev Console comanda care creează indexul și setează cum se va face analiza în limba română.

```yaml
PUT /testanalizori
{
  "settings": {
    "analysis": {
      "analyzer": {
        "romanian_stop": {
          "type": "standard",
          "stopwords":"_romanian_"
        }
      },
      "filter": {
        "stemmer_românesc":{
          "type": "stemmer",
          "name":"romanian"
        }
      }
    }
  }
}

POST /testanalizori/_analyze
{
  "tokenizer": "standard",
  "filter":["stemmer_românesc"],
  "text": "Acesta ține fragmentul roșu de text și în limba română"
}
```

Rezultatul poate fi similar cu următorul obiect.

```json
{
  "tokens" : [
    {
      "token" : "Acest",
      "start_offset" : 0,
      "end_offset" : 6,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "țin",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "fragment",
      "start_offset" : 12,
      "end_offset" : 22,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "roșu",
      "start_offset" : 23,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "de",
      "start_offset" : 28,
      "end_offset" : 30,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "text",
      "start_offset" : 31,
      "end_offset" : 35,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "și",
      "start_offset" : 36,
      "end_offset" : 38,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "în",
      "start_offset" : 39,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "limb",
      "start_offset" : 42,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "român",
      "start_offset" : 48,
      "end_offset" : 54,
      "type" : "<ALPHANUM>",
      "position" : 9
    }
  ]
}
```

Se observă că atricularea substantivelor a dispărut, dar sunt mici deviații, precum în cazul lui `limbă` -> `limb`.

## Stemming

Pentru exemplificare, vom configura un stemmer particularizat pentru limba română.

```json
PUT /stemming_test/
{
  "settings": {
    "analysis": {
      "filter": {
        "testare_sinonime": {
          "type": "synonym",
          "synonyms": [
            "frumos => chipeș",
            "plăcea, dori"
          ]
        },
        "testare_stemmer": {
          "type": "stemmer",
          "name": "romanian"
        }
      },
      "analyzer": {
        "analizor_propriu": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "testare_sinonime",
            "testare_stemmer"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "descriere": {
        "type": "text",
        "analyzer": "analizor_propriu"
      }
    }
  }
}
```

Apoi vom introduce un document în indexul nou creat.

```json
POST /stemming_test/_doc/1
{
  "descriere": "Acest test frumos este unul util și plin de lucruri interesante pentru a fi observate. Unele lucruri nu snt tocmai plăcute când rulezi teste."
}
```

Răspunsul este similar cu următorul obiect:

```json
{
  "_index" : "stemming_test",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

Să creăm o interogare pentru documentul indexat:

```json
GET /stemming_test/_search
{
  "query": {
    "match": {
      "descriere": "test chipeș"
    }
  },
  "highlight": {
    "fields": {
      "descriere": {}
    }
  }
}
```

Rezultatul indică faptul că documentul a fost găsit în baza sinonimelor.

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.68324494,
    "hits" : [
      {
        "_index" : "stemming_test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.68324494,
        "_source" : {
          "descriere" : "Acest test frumos este unul util și plin de lucruri interesante pentru a fi observate. Unele lucruri nu snt tocmai plăcute când rulezi teste."
        },
        "highlight" : {
          "descriere" : [
            "Acest <em>test</em> <em>frumos</em> este unul util și plin de lucruri interesante pentru a fi observate.",
            "Unele lucruri nu snt tocmai plăcute când rulezi <em>teste</em>."
          ]
        }
      }
    ]
  }
}
```

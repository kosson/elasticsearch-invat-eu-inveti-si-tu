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
  2) **Mapping Character Filter** înlocuiește valorile pe baza unui obiect cheie - valoare. Put și simplu atunci când Elasticsearch vede o anumită valoare, o înlocuiește cu un caracter sau valoare.
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

## Tokenizatori
